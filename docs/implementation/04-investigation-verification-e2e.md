# ReproDocket Investigation, Verification, and End-to-End Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete the real ReproDocket workflow from a required auditable action-and-expectation plan through a recorded Solari investigation, fresh independent verification, evidence-backed outcome, local report/history, cancellation, and full UI projection.

**Architecture:** One `RunCoordinator` owns admission, lifecycle, cancellation, and finalization. A shared `AttemptEngine` executes only `ReproductionAction` statements and evaluates the already-final `ObservationExpectation` statements through the observer. Investigation and verification each receive a separately created recorded Solari browser. The final classifier may return `VERIFIED` only when both independent attempts contain sufficient matching reproduction evidence and distinct non-null Solari session IDs.

**Tech Stack:** TypeScript, Fastify, React, Vitest, Playwright Test, `@solarisdk/browser`, and the ReproDocket storage/evidence/report infrastructure established in Plans 1 through 3.

**Specs:** `docs/reprodocket-design.md`, `docs/reprodocket-interface-contracts.md`, `docs/reprodocket-data-handling.md`, `docs/reprodocket-security-lifecycle.md`, `docs/reprodocket-sdk-baseline.md`, `docs/reprodocket-fixture-spec.md`, `docs/reprodocket-ui-spec.md`, `docs/reprodocket-test-matrix.md`, and `docs/implementation/00-contract-reconciliation.md`.

## Global Constraints

* Plans 1 through 3 must have current passing evidence for their applicable gates before this plan begins.
* Version one accepts a required unified plan containing at least one `ReproductionAction` and at least one `ObservationExpectation`.
* The expectation grammar is already finalized: `EXPECT_TEXT`, `EXPECT_NO_TEXT`, `EXPECT_URL`, `EXPECT_PAGE_ERROR`, and `EXPECT_MAIN_STATUS`.
* Do not introduce the obsolete `ParsedReproductionStep`, `ReproductionStepParser`, or `INVALID_REPRODUCTION_STEP` naming.
* Version one executes the documented plan. It does not invent arbitrary browser actions from freeform prose.
* User-authored plan values are persisted and must use nonsecret test data.
* The first and second attempts use different Solari session identities and new browser/page state.
* No outcome is inferred merely from transport success, a warning, a console error, or a failed subrequest.
* `VERIFIED` requires sufficient supporting evidence from both attempts plus different non-null Solari session IDs.
* The user-facing workflow is exercised through the built local UI in end-to-end tests.
* Cancellation is a supported lifecycle path, not a process-kill shortcut.
* Browser and sandbox cleanup remain mandatory even when functional reproduction/verification fails.
* Every state shown by the UI comes from the authoritative run manifest or an explicit damaged-run projection.
* A functional result is incomplete if report generation, persistence, history, reload, cleanup, or return navigation is disconnected.
* Critical tests receive an actual sensitivity/mutation proof where the test matrix requires one.

---

## Task 1: Implement deterministic action execution and accessible target resolution

**Files:**
- Create: `reprodocket/src/core/ReproductionStepExecutor.ts`
- Create: `reprodocket/src/core/AccessibleLocatorResolver.ts`
- Test: `reprodocket/tests/unit/AccessibleLocatorResolver.test.ts`
- Test: `reprodocket/tests/integration/ReproductionStepExecutor.test.ts`

**Interfaces:**
- Implements `ReproductionStepExecutor.execute(page, step, context)` from the interface contract.
- Receives only a parsed `ReproductionAction`, never an `ObservationExpectation`.
- Produces `StepExecutionResult` with before/after URL and timing.
- Uses the exact installed Solari Playwright-compatible page type proven in Plan 3.

- [ ] **Step 1: Write failing locator-resolution tests**

Require these semantics:

```text
CLICK -> unique accessible button/link/name first, then one unique visible text fallback
FILL -> unique associated accessible label
SELECT -> unique associated accessible label
CHECK/UNCHECK -> unique associated accessible label
multiple plausible visible matches -> AMBIGUOUS_TARGET_ELEMENT
no valid visible match -> TARGET_ELEMENT_NOT_FOUND
hidden-only match -> not accepted
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- AccessibleLocatorResolver.test.ts
```

Expected: FAIL because the resolver does not exist.

- [ ] **Step 3: Implement accessible resolution**

Use ordinary user-facing semantics and deterministic uniqueness. Do not accept arbitrary CSS/XPath from the plan. Do not silently choose the first ambiguous element.

- [ ] **Step 4: Write failing executor tests for every supported action**

Cover:

```text
OPEN
CLICK
FILL
SELECT
CHECK
UNCHECK
PRESS
WAIT_FOR_TEXT
WAIT_FOR_URL
RELOAD
BACK
FORWARD
```

Use a local deterministic Playwright-compatible page. Verify observable state after each action. Include already-aborted and mid-wait cancellation tests.

- [ ] **Step 5: Prove RED**

```powershell
npm run test:integration -- ReproductionStepExecutor.test.ts
```

- [ ] **Step 6: Implement action execution**

Check the abort signal before each action and during waits. Use stage-specific timeouts. Capture before/after URL. `OPEN` and absolute `WAIT_FOR_URL` destinations pass the same target policy as the initial URL. Main-frame redirects are revalidated at the strongest authority proven by Plan 3 and later hardening.

`PRESS` uses a bounded allowlist of ordinary keys required by supported workflows. It is not arbitrary global keyboard injection.

- [ ] **Step 7: Add explicit error-path tests**

Require stable codes for:

```text
AMBIGUOUS_TARGET_ELEMENT
TARGET_ELEMENT_NOT_FOUND
ACTION_TIMEOUT
BLOCKED_TARGET_NETWORK
ACTION_OUTCOME_UNKNOWN
```

A target-mutating action whose outcome is unknown must not be silently retried.

- [ ] **Step 8: Prove GREEN and resolver sensitivity**

Temporarily change the resolver to choose the first of two ambiguous buttons. Confirm the ambiguity test fails. Restore and rerun:

```powershell
npm run test:unit -- AccessibleLocatorResolver.test.ts
npm run test:integration -- ReproductionStepExecutor.test.ts
npm run typecheck
```

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/core/ReproductionStepExecutor.ts reprodocket/src/core/AccessibleLocatorResolver.ts reprodocket/tests/unit/AccessibleLocatorResolver.test.ts reprodocket/tests/integration/ReproductionStepExecutor.test.ts
git diff --cached --check
git commit -m "feat: execute auditable ReproDocket actions"
```

---

## Task 2: Implement the finalized expectation observer

**Files:**
- Create: `reprodocket/src/core/DefectObserver.ts`
- Test: `reprodocket/tests/unit/DefectObserver.test.ts`

**Interfaces:**
- Consumes `ObservationExpectation`, `ExpectationResult`, `AttemptObservation`, normalized evidence, and current browser state from the existing shared contracts.
- Produces `AttemptObservation` with `CONFIRMED`, `NOT_OBSERVED`, or `INSUFFICIENT` strength.
- Does not own final two-attempt classification.

- [ ] **Step 1: Write failing tests for every expectation primitive**

Cover the already-final grammar:

```text
EXPECT_TEXT
EXPECT_NO_TEXT
EXPECT_URL
EXPECT_PAGE_ERROR
EXPECT_MAIN_STATUS
```

For each expectation test true, false, and insufficient-authority cases where applicable. Every `ExpectationResult` must retain the originating statement index and supporting evidence IDs.

- [ ] **Step 2: Add anti-false-positive tests before implementation**

Require:

```text
unrelated console warning alone -> not CONFIRMED
unrelated console error alone -> not CONFIRMED
unrelated failed subrequest alone -> not CONFIRMED
successful action sequence alone -> not CONFIRMED
missing required observation authority -> INSUFFICIENT
explicit satisfied defect expectation with supporting evidence -> CONFIRMED
complete workflow with tested defect expectation not observed -> NOT_OBSERVED
```

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- DefectObserver.test.ts
```

Expected: FAIL because the observer does not exist.

- [ ] **Step 4: Implement expectation evaluation without fixture identity knowledge**

The observer evaluates general plan expectations only. It may correlate page errors, main-document status, current URL, and visible text to explicit expectations, but it must not inspect fixture route/scenario IDs to manufacture an answer.

`EXPECT_PAGE_ERROR` matches the bounded normalized page-error contract. `EXPECT_MAIN_STATUS` refers to the current relevant main-document navigation response, not any subrequest. `EXPECT_URL` uses normalized allowed URL semantics. Text expectations use visible user-facing page state rather than arbitrary DOM serialization.

- [ ] **Step 5: Implement conservative attempt strength**

Return `CONFIRMED` only when the defect-defining expectation set has sufficient current evidence. Return `NOT_OBSERVED` only when the required workflow completed far enough to make a trustworthy negative determination. Otherwise return `INSUFFICIENT`.

- [ ] **Step 6: Prove GREEN and seed one false-positive mutation**

Temporarily make any console error force `CONFIRMED`. Confirm the anti-false-positive test fails. Restore and rerun:

```powershell
npm run test:unit -- DefectObserver.test.ts
npm run typecheck
```

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/core/DefectObserver.ts reprodocket/tests/unit/DefectObserver.test.ts
git diff --cached --check
git commit -m "feat: evaluate ReproDocket defect expectations"
```

---

## Task 3: Build the production AttemptEngine

**Files:**
- Create: `reprodocket/src/core/AttemptEngine.ts`
- Test: `reprodocket/tests/integration/AttemptEngine.test.ts`
- Test: `reprodocket/tests/live/AttemptEngine.live.test.ts`

**Interfaces:**
- Produces `runAttempt(run, role, parsedPlan, signal)`.
- Executes only action statements through `ReproductionStepExecutor` and evaluates expectation statements through `DefectObserver`.
- Consumes `BrowserProvider`, `EvidenceCollector`, `RunStore`, and resource ownership services established earlier.

- [ ] **Step 1: Write failing deterministic integration tests**

Require this ordering:

```text
create recorded browser
register owned session identity
create page
attach evidence bridge before target navigation
capture baseline semantic evidence
execute actions in source order
evaluate expectations at their defined observation boundaries
capture failure/final semantic evidence
finish collector
produce attempt observation/outcome
close browser in finally
query replay state only after release boundary required by the installed SDK
persist attempt and cleanup truth
```

A thrown action, observer error, or evidence failure must not skip browser cleanup.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- AttemptEngine.test.ts
```

- [ ] **Step 3: Implement structured attempt ownership**

Persist enough active state before external work that a crash leaves an identifiable attempt. Each attempt owns one fresh browser and its evidence collector. Caller cancellation flows through an abort signal.

- [ ] **Step 4: Capture evidence at semantic boundaries**

Minimum useful screenshots:

```text
initial target ready
before a failure-defining action when determinable
after relevant action/observation boundary
current frame on exception/timeout when available
final attempt state
```

Do not capture on arbitrary intervals merely to inflate evidence count.

- [ ] **Step 5: Prove deterministic GREEN**

```powershell
npm run test:integration -- AttemptEngine.test.ts
```

- [ ] **Step 6: Add one healthy and one defective real Solari attempt test**

Use the real sandbox-hosted deterministic fixture from Plan 3 and the actual production AttemptEngine. Require a real session ID, run-owned evidence IDs, correct attempt outcome, replay state, and cleanup record.

```powershell
npm run test:live -- AttemptEngine.live.test.ts
```

- [ ] **Step 7: Require resource reconciliation**

A functional assertion does not pass the live test while its attempt browser remains unresolved.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/core/AttemptEngine.ts reprodocket/tests/integration/AttemptEngine.test.ts reprodocket/tests/live/AttemptEngine.live.test.ts
git diff --cached --check
git commit -m "feat: run evidence-backed Solari attempts"
```

---

## Task 4: Implement atomic run admission and the two-attempt coordinator

**Files:**
- Create: `reprodocket/src/core/RunCoordinator.ts`
- Create: `reprodocket/src/core/ActiveRunGate.ts`
- Test: `reprodocket/tests/integration/ActiveRunGate.test.ts`
- Test: `reprodocket/tests/integration/RunCoordinator.test.ts`

**Interfaces:**
- Implements `RunCoordinator.createAndStart`, `cancel`, `activeRunId`, and `shutdown`.
- One process supports one executing investigation pipeline while history remains readable.

- [ ] **Step 1: Write failing duplicate-admission race tests**

Race two create requests with `Promise.allSettled`. Exactly one obtains the active slot; the other receives `RUN_ALREADY_ACTIVE`. No second browser-provider call may occur.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- ActiveRunGate.test.ts
```

- [ ] **Step 3: Implement the synchronous reservation boundary**

Reserve the active slot before the first asynchronous persistence/provider await. Release it only after terminal persistence and required cleanup/finalization have settled.

- [ ] **Step 4: Write failing coordinator lifecycle tests**

Require the authoritative sequence:

```text
CREATED persisted
PREPARING
INVESTIGATING
investigation attempt persisted and browser released
VERIFYING
fresh verification attempt persisted
FINALIZING
report/integrity finalization
COMPLETED or truthful failure/cancellation state
active gate released
```

- [ ] **Step 5: Prove RED**

```powershell
npm run test:integration -- RunCoordinator.test.ts
```

- [ ] **Step 6: Implement first attempt then fresh verification**

The investigation browser must close before the verification attempt obtains a new browser. Before final classification, require different non-null session IDs. Do not reuse page/context/browser/session state.

- [ ] **Step 7: Finalize durable evidence before COMPLETED**

Required report generation, artifact sealing, and manifest persistence occur before `COMPLETED` becomes authoritative. Cleanup remains a separate visible dimension and can block Full validation.

- [ ] **Step 8: Prove GREEN and mutate session independence**

Temporarily force the verification record to reuse the investigation session ID. Require the test to fail before `VERIFIED`. Restore and rerun.

```powershell
npm run test:integration -- ActiveRunGate.test.ts RunCoordinator.test.ts
```

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/core/RunCoordinator.ts reprodocket/src/core/ActiveRunGate.ts reprodocket/tests/integration/ActiveRunGate.test.ts reprodocket/tests/integration/RunCoordinator.test.ts
git diff --cached --check
git commit -m "feat: coordinate independent ReproDocket verification"
```

---

## Task 5: Expose real run creation and cancellation routes

**Files:**
- Create: `reprodocket/src/server/routes/runsWrite.ts`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/integration/RunWriteRoutes.test.ts`

**Interfaces:**
- Implements `POST /api/runs` and `POST /api/runs/:runId/cancel` from the interface contract.
- Uses the existing local request guard and `RunCoordinator`.

- [ ] **Step 1: Write failing route tests for create, validation, duplicate admission, and cancel**

Require:

```text
invalid request shape -> 4xx stable error
missing/empty plan -> rejected
action-only plan -> rejected
expectation-only plan -> rejected
invalid plan statement -> INVALID_PLAN_STATEMENT
invalid/blocked target -> rejected before provider invocation
valid complete plan -> 202 + authoritative run ID
second active submission -> 409 RUN_ALREADY_ACTIVE
unknown cancel target -> RUN_NOT_FOUND
accepted cancel response -> request acknowledged, not falsely terminal
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunWriteRoutes.test.ts
```

- [ ] **Step 3: Implement validated routes**

Use the server-side `CreateRunRequestValidator`. UI validation is not authority. Enforce documented length/count limits and local mutation guard.

- [ ] **Step 4: Implement cancellation routing**

Cancel only the matching currently owned run. A terminal run cannot trigger a second resource cleanup pass.

- [ ] **Step 5: Prove GREEN with security adjacency**

```powershell
npm run test:integration -- RunWriteRoutes.test.ts LocalServerSecurity.test.ts
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/server/routes/runsWrite.ts reprodocket/src/server/createServer.ts reprodocket/tests/integration/RunWriteRoutes.test.ts
git diff --cached --check
git commit -m "feat: expose ReproDocket run lifecycle routes"
```

---

## Task 6: Connect the New Investigation UI to the real coordinator path

**Files:**
- Create: `reprodocket/src/client/components/NewInvestigationForm.tsx`
- Create: `reprodocket/src/client/components/PlanSyntaxHelp.tsx`
- Modify: `reprodocket/src/client/App.tsx`
- Modify: `reprodocket/src/client/api/ApiClient.ts`
- Test: `reprodocket/tests/ui/NewInvestigation.spec.ts`

**Interfaces:**
- Uses only the public local `POST /api/runs` path.
- Submits target URL, problem description, and one required action-and-expectation plan.
- Navigates to `/runs/:runId` after the server accepts the run.

- [ ] **Step 1: Write failing H004 through H008 user-boundary tests**

Require target, problem, and plan completeness. Omitted plan, action-only plan, and expectation-only plan are rejected. Show working syntax help for both actions and expectations. Rapid double submit creates one server run. Server errors render sanitized text without stack traces.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- NewInvestigation.spec.ts
```

- [ ] **Step 3: Implement the form from the UI specification**

Use labeled target/problem/plan controls. The plan textarea is one statement per line and visibly warns that plan text is persisted and must not contain secrets. Syntax help comes from the actual finalized grammar.

- [ ] **Step 4: Keep server admission authoritative**

Disable the submit control after submission begins for usability, but rely on the server active-run gate for concurrency. Re-enable after rejected admission.

- [ ] **Step 5: Navigate to the authoritative run**

On `202`, route to `/runs/<id>` and hydrate the page through GET/SSE. Do not construct optimistic fake run state in the client.

- [ ] **Step 6: Prove GREEN and accessibility**

```powershell
npm run build
npm run test:ui -- NewInvestigation.spec.ts
```

Verify accessible names, error associations, keyboard operation, focus after rejection, visible plan help, and nonsecret-plan warning.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/client/components/NewInvestigationForm.tsx reprodocket/src/client/components/PlanSyntaxHelp.tsx reprodocket/src/client/App.tsx reprodocket/src/client/api/ApiClient.ts reprodocket/tests/ui/NewInvestigation.spec.ts
git diff --cached --check
git commit -m "feat: connect the ReproDocket investigation form"
```

---

## Task 7: Implement cancellation end to end

**Files:**
- Modify: `reprodocket/src/core/RunCoordinator.ts`
- Create: `reprodocket/src/client/components/CancelRunButton.tsx`
- Modify: `reprodocket/src/client/pages/RunPage.tsx`
- Test: `reprodocket/tests/integration/RunCancellation.test.ts`
- Test: `reprodocket/tests/ui/RunCancellation.spec.ts`
- Test: `reprodocket/tests/live/RunCancellation.live.test.ts`

**Interfaces:**
- UI -> cancel API -> coordinator -> abort signal -> attempt cleanup -> terminal persistence -> UI.

- [ ] **Step 1: Write deterministic cancel tests before implementation**

Cancel during investigation and verification. Require no new action after cancellation request, no false `VERIFIED`, exactly-once cleanup requests per owned resource, and terminal `CANCELLED` only after the server persists it.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunCancellation.test.ts
```

- [ ] **Step 3: Implement coordinator cancellation ownership**

Only the active run's controller may be aborted. Repeated cancellation is idempotent. Unknown/unowned run IDs cannot cancel another run.

- [ ] **Step 4: Implement truthful Cancel UI**

Show Cancel only while the lifecycle is cancellable. After click, show `Cancellation requested` until terminal server state arrives. Do not optimistically display `CANCELLED`.

- [ ] **Step 5: Prove UI GREEN**

```powershell
npm run build
npm run test:ui -- RunCancellation.spec.ts
```

- [ ] **Step 6: Add one bounded real Solari cancellation test**

Use a deterministic fixture wait/action, cancel the real run, and require browser cleanup plus terminal `CANCELLED`. Do not create a harmful external mutation solely to test cancellation.

```powershell
npm run test:live -- RunCancellation.live.test.ts
```

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/core/RunCoordinator.ts reprodocket/src/client/components/CancelRunButton.tsx reprodocket/src/client/pages/RunPage.tsx reprodocket/tests/integration/RunCancellation.test.ts reprodocket/tests/ui/RunCancellation.spec.ts reprodocket/tests/live/RunCancellation.live.test.ts
git diff --cached --check
git commit -m "feat: cancel ReproDocket runs safely"
```

---

## Task 8: Define the complete deterministic fixture plans and all four outcome classes

**Files:**
- Create: `reprodocket/fixtures/plans/account-blank.json`
- Create: `reprodocket/fixtures/plans/billing-refresh.json`
- Create: `reprodocket/fixtures/plans/missing-zip.json`
- Create: `reprodocket/fixtures/plans/healthy-profile.json`
- Create: `reprodocket/fixtures/plans/healthy-login.json`
- Create: `reprodocket/fixtures/plans/ambiguous.json`
- Create: `reprodocket/fixtures/plans/nonrepeatable.json`
- Test: `reprodocket/tests/integration/FixturePlans.test.ts`

**Interfaces:**
- Each fixture plan is an ordinary `CreateRunRequest` using only the public finalized grammar and nonsecret test data.
- Fixture route/scenario identity is not passed as a privileged product instruction.

- [ ] **Step 1: Encode plans against the independent fixture specification**

Representative finalized grammar:

```text
account blank:
OPEN "/account"
FILL "Email" WITH "qa.changed@example.com"
CLICK "Save"
EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"

billing refresh:
OPEN "/billing/details"
RELOAD
EXPECT_MAIN_STATUS 404

missing ZIP:
OPEN "/address"
FILL "Street" WITH "100 Test Ave"
CLICK "Submit address"
EXPECT_TEXT "Address accepted"
```

Healthy negative-control plans define an alleged defect condition that the healthy fixture does not satisfy after the workflow is fully exercised. Do not use an automatically healthy default merely because an error was absent.

- [ ] **Step 2: Write failing plan/oracle consistency tests**

Every plan must parse through `parsePlanStatements`, satisfy request completeness, contain no unknown fields, and refer only to fixture behaviors independently established by `docs/reprodocket-fixture-spec.md` and fixture truth tests.

- [ ] **Step 3: Prove RED while plan files/fixtures are absent**

```powershell
npm run test:integration -- FixturePlans.test.ts
```

- [ ] **Step 4: Implement all seven plan files and prove GREEN**

```powershell
npm run test:integration -- FixturePlans.test.ts
```

Require the expected plan count and the intended mapping to the four result classes without production code branching on plan names.

- [ ] **Step 5: Commit**

```powershell
git add reprodocket/fixtures/plans reprodocket/tests/integration/FixturePlans.test.ts
git diff --cached --check
git commit -m "test: define ReproDocket end-to-end plans"
```

---

## Task 9: Prove the complete real product path through the local UI and Solari

**Files:**
- Create: `reprodocket/tests/e2e/ReproDocketFullFlow.spec.ts`
- Create: `reprodocket/tests/e2e/E2eHarness.ts`
- Modify: `reprodocket/playwright.config.ts` only if a dedicated E2E project is required.

**Interfaces:**
- Exercises the real built local UI, local APIs, production coordinator/attempt engine, real Solari browsers, and the real Solari sandbox-hosted fixture.
- The harness may create the fixture URL, but run creation itself occurs through the normal visible form/API path.

- [ ] **Step 1: Implement outer resource ownership in the E2E harness**

The harness sequence is:

```text
build/start exact ReproDocket source
create one owned Solari sandbox fixture
externally verify fixture version/health
open local ReproDocket UI
configure/use authorized Solari credential without exposing it
submit a normal complete plan through the visible form
wait for terminal authoritative state
read UI result and public/local API projection
assert manifest/report/history/evidence parity
kill fixture sandbox in outer finally
stop owned local server
```

A UI assertion failure must not strand the fixture sandbox.

- [ ] **Step 2: Add all deterministic result scenarios**

Require:

```text
account blank -> VERIFIED
billing refresh -> VERIFIED
missing ZIP -> VERIFIED
healthy profile -> NOT_REPRODUCED
healthy login -> NOT_REPRODUCED
ambiguous -> INCONCLUSIVE
nonrepeatable -> REPRODUCED
```

The fixture truth tests remain a separate oracle authority.

- [ ] **Step 3: Assert fresh-session independence**

For every two-attempt run:

```text
investigation session ID non-null
verification session ID non-null
IDs differ
investigation release occurs before verification creation
evidence attempt ownership differs correctly
no browser/page/context reuse
```

- [ ] **Step 4: Assert local result parity and restart persistence**

Run detail, history, JSON report, HTML report, evidence lists, and manifest agree on the same run. Restart ReproDocket after a completed run and prove it remains visible without creating a new Solari browser.

- [ ] **Step 5: Assert resource cleanup**

At suite end, every harness-owned browser and sandbox must be reconciled to the strongest terminal state the current provider exposes. Unresolved cleanup prevents E2E PASS.

- [ ] **Step 6: Run one vertical scenario first**

```powershell
npx playwright test tests/e2e/ReproDocketFullFlow.spec.ts --grep "account blank"
```

Repair until one complete real vertical path passes.

- [ ] **Step 7: Run all E2E outcome scenarios**

```powershell
npx playwright test tests/e2e/ReproDocketFullFlow.spec.ts
```

- [ ] **Step 8: Mutation prove false-positive sensitivity**

Temporarily force the classifier toward `VERIFIED` for a healthy control. Confirm the healthy E2E case fails. Restore and rerun the affected scenario.

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/tests/e2e reprodocket/playwright.config.ts
git diff --cached --check
git commit -m "test: prove ReproDocket end to end on Solari"
```

---

## Task 10: Close horizontal and vertical connectivity for M5 through M7

**Files:**
- Modify: `reprodocket/docs/connectivity-matrix.md`
- Create: `reprodocket/tests/e2e/VerticalConnectivity.spec.ts`
- Modify: `reprodocket/tests/integration/HorizontalConsistency.test.ts`
- Modify: `reprodocket/tests/ui/HorizontalConsistency.spec.ts`
- Modify: `reprodocket/scripts/validate.ps1`

**Interfaces:**
- Adds the applicable N-series and O-series authorities from `docs/reprodocket-test-matrix.md` to Full validation.

- [ ] **Step 1: Reinventory every visible feature from the actual built application**

A row becomes complete only when discoverability, entry, prerequisites, action routing, feedback, authority, persistence, reload, recovery where applicable, exit/return, regression coverage, and remaining human-QA state are known.

- [ ] **Step 2: Write vertical user-outcome tests**

At minimum prove:

```text
New Investigation -> validated request -> Solari -> evidence -> verification -> outcome -> UI
screenshot click -> artifact ownership -> integrity -> image response
replay control -> authoritative replay state
cancel -> coordinator -> resource cleanup -> terminal state -> UI
restart -> validated history -> detail
one damaged historical run -> explicit diagnostic without global history failure
Full fixture validation -> sandbox -> browser A -> browser B -> local results -> cleanup
```

- [ ] **Step 3: Run horizontal consistency against real E2E data**

Once real runs exist, do not limit consistency checks to synthetic manifests. Compare active/history/detail/report/evidence/replay/cleanup/provider/validation projections against the same durable authority.

- [ ] **Step 4: Add complete product stages to Full validation**

After substrate contracts, Full includes:

```text
fixture truth
attempt engine
run coordinator
built user-boundary UI
all outcome E2E
horizontal consistency
vertical connectivity
resource reconciliation
report/evidence integrity
```

- [ ] **Step 5: Run the first complete machine Full validation**

```powershell
.\reprodocket\scripts\validate.ps1
```

It may return PASS only if every applicable mandatory machine authority is current for the exact source revision. Human visual/usability judgment remains a separate state.

- [ ] **Step 6: Capture objective visual evidence from the same build**

Capture original-resolution views required by `docs/reprodocket-ui-spec.md` and `docs/reprodocket-test-matrix.md`, including New Investigation, active state, all four outcomes, history, screenshot/console/network/timeline evidence, replay state, and representative failure/cancellation.

Use background-safe Playwright capture for this local web UI. Do not require desktop-wide foreground automation for ordinary web layout proof.

- [ ] **Step 7: Repair objective UI defects before hardening**

Blocking examples include clipped labels, horizontal overflow, unreadable small text, raw stack traces, result/status contradiction, dead controls, history/detail mismatch, wrong-run evidence, and misleading success styling. Each repair receives a regression test.

- [ ] **Step 8: Commit connectivity closure**

```powershell
git add reprodocket/docs/connectivity-matrix.md reprodocket/tests/e2e/VerticalConnectivity.spec.ts reprodocket/tests/integration/HorizontalConsistency.test.ts reprodocket/tests/ui/HorizontalConsistency.spec.ts reprodocket/scripts/validate.ps1
git diff --cached --check
git commit -m "test: close ReproDocket vertical integration"
```

---

## Plan 4 Completion Gate

Do not begin the hardening plan while any ordinary supported product path is partial or disconnected.

Current evidence for one source revision must establish:

```text
all supported action grammar executes through real browser semantics
all finalized expectation grammar is evaluated through general evidence rules
unrelated console/network noise cannot automatically prove a defect
one production AttemptEngine path works against real Solari
investigation and verification are fresh independent Solari sessions
all four final outcome classes are demonstrated
one-active-run admission survives duplicate-request races
cancellation works end to end and is resource aware
New Investigation UI submits the real public/local path
all deterministic fixture plans are independently truthful
real Solari E2E covers positive, negative, ambiguous, and nonrepeatable cases
completed runs survive local app restart
reports/history/detail/evidence agree with manifest authority
all required owned Solari resources are reconciled
horizontal and vertical connectivity contain no unresolved ordinary-scope partial/dead path
Full machine validation has a current result for the exact source revision
human visual/usability authority remains explicitly separate
```

Run:

```powershell
.\reprodocket\scripts\validate.ps1
git status --short
git diff main...HEAD --check
```

A machine Full PASS at this point means the implemented front-to-back product is ready for the deliberate bug-finding and hardening plan. It does not mean publication ready.