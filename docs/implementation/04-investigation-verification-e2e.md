# ReproDocket Investigation, Verification, and End-to-End Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete the real ReproDocket user workflow from an auditable reproduction plan through a recorded Solari investigation, fresh independent verification, evidence-backed outcome, local report, durable history, cancellation, and full UI projection.

**Architecture:** One `RunCoordinator` owns admission and cancellation. A shared `AttemptEngine` executes the same parsed action semantics for investigation and verification, but each attempt receives a separately created recorded Solari browser. Evidence and run state are persisted through the existing stores, and the final `OutcomeClassifier` can only promote a run to VERIFIED when both independent attempts contain sufficient reproduction evidence.

**Tech Stack:** TypeScript, Fastify, React, Vitest, Playwright Test, `@solarisdk/browser`, existing ReproDocket storage/evidence/report infrastructure.

**Spec:** `docs/reprodocket-design.md`; contracts: `docs/reprodocket-interface-contracts.md`; security/lifecycle: `docs/reprodocket-security-lifecycle.md`; tests: `docs/reprodocket-test-matrix.md`; Solari baseline: `docs/reprodocket-sdk-baseline.md`.

## Global Constraints

* Plans 1 through 3 must be freshly green for their applicable authorities before this work begins.
* Version one executes only the documented reproduction-step grammar. It does not invent arbitrary browser actions from freeform prose.
* The first and second attempts use different Solari session identities and newly created pages/browser state.
* No outcome is inferred from a transport success, a console warning, or a failed subrequest alone.
* VERIFIED requires evidence from both attempts and different non-null Solari session IDs.
* The user-facing workflow must be exercised through the built local UI in end-to-end tests.
* Cancellation is a supported lifecycle path, not a process-kill shortcut.
* Browser/sandbox cleanup remains mandatory even when reproduction or verification fails.
* Every state shown by the UI comes from the same authoritative run manifest or an explicit damaged-run projection.
* A functional result is not enough if persistence, report generation, history, reload, cleanup, or return navigation is disconnected.

---

## Task 1: Implement deterministic reproduction-step execution

**Files:**
- Create: `reprodocket/src/core/ReproductionStepExecutor.ts`
- Create: `reprodocket/src/core/AccessibleLocatorResolver.ts`
- Test: `reprodocket/tests/unit/AccessibleLocatorResolver.test.ts`
- Test: `reprodocket/tests/integration/ReproductionStepExecutor.test.ts`

**Interfaces:**
- Implements `ReproductionStepExecutor.execute(page, step, context)` from `docs/reprodocket-interface-contracts.md`.
- Produces `StepExecutionResult` with exact before/after URLs and timing.
- Uses the exact installed Solari Playwright-compatible page type established in Plan 3 rather than importing an incompatible type by assumption.

- [ ] **Step 1: Write failing resolver tests**

Test the resolution policy independently from Solari with a Playwright-compatible local fixture:

```text
CLICK -> unique accessible button/link name first, then unique visible text fallback
FILL -> unique associated label first
SELECT -> unique associated label first
CHECK/UNCHECK -> unique associated label first
multiple equally valid matches -> AMBIGUOUS_TARGET_ELEMENT
no match -> TARGET_ELEMENT_NOT_FOUND
hidden-only match -> not accepted as the ordinary target
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- AccessibleLocatorResolver.test.ts
```

Expected: FAIL because the resolver does not exist.

- [ ] **Step 3: Implement deterministic accessible resolution**

Use ordinary user-facing semantics. Do not accept arbitrary CSS/XPath from the reproduction grammar. A fallback from accessible name to text must still require one unique visible target.

- [ ] **Step 4: Write failing step-executor integration tests**

Cover every grammar action from `docs/reprodocket-interface-contracts.md`:

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

Use a local deterministic test page and verify observable browser state after each action. Test cancellation through an already-aborted `AbortSignal` and an abort during a wait.

- [ ] **Step 5: Prove RED**

```powershell
npm run test:integration -- ReproductionStepExecutor.test.ts
```

- [ ] **Step 6: Implement the executor**

Before each action, check `AbortSignal`. Capture URL before and after. Use stage-specific timeouts. `OPEN` and `WAIT_FOR_URL` must apply the same target URL/network policy to absolute destinations and enforce same allowed-origin/network boundaries after redirects where authoritative data is available.

`PRESS` accepts only a bounded allowlist of normal keyboard names needed by the supported grammar. It must not become an arbitrary chord/injection language.

- [ ] **Step 7: Add ambiguity, timeout, and blocked-navigation tests**

Require stable error codes:

```text
AMBIGUOUS_TARGET_ELEMENT
TARGET_ELEMENT_NOT_FOUND
ACTION_TIMEOUT
BLOCKED_TARGET_NETWORK
```

- [ ] **Step 8: Prove GREEN and mutation sensitivity**

Temporarily change the resolver to choose the first of two ambiguous matching buttons. Confirm the ambiguity test fails. Restore and rerun.

```powershell
npm run test:unit -- AccessibleLocatorResolver.test.ts
npm run test:integration -- ReproductionStepExecutor.test.ts
npm run typecheck
```

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/core/ReproductionStepExecutor.ts reprodocket/src/core/AccessibleLocatorResolver.ts reprodocket/tests/unit/AccessibleLocatorResolver.test.ts reprodocket/tests/integration/ReproductionStepExecutor.test.ts
git diff --cached --check
git commit -m "feat: execute auditable reproduction steps"
```

---

## Task 2: Implement evidence-backed defect observation

**Files:**
- Create: `reprodocket/src/core/DefectObserver.ts`
- Create: `reprodocket/src/shared/contracts/ObservationModels.ts`
- Test: `reprodocket/tests/unit/DefectObserver.test.ts`

**Interfaces:**
- Produces `observeAttempt(input): AttemptObservation`.
- Consumes completed/failed step results and normalized evidence, not raw target HTML.
- Does not own final two-attempt classification.

- [ ] **Step 1: Define the attempt observation type in a failing test**

Required semantic values:

```ts
export type ObservationStrength = "CONFIRMED" | "NOT_OBSERVED" | "INSUFFICIENT";

export interface AttemptObservation {
  strength: ObservationStrength;
  summary: string;
  supportingEvidenceIds: string[];
  reasonCode: string;
}
```

- [ ] **Step 2: Write failing detector tests**

Initial general detectors:

```text
uncaught page error after the relevant action -> may confirm when the report/fixture expectation declares that cue
main-document navigation to 4xx/5xx when the expected route should load -> may confirm
page closes/crashes during required step -> may confirm
WAIT_FOR_TEXT required postcondition succeeds -> evidence that expected healthy condition exists
WAIT_FOR_TEXT required postcondition fails -> reproduction cue only when the report/fixture observation contract defines it
WAIT_FOR_URL required postcondition fails -> same rule
console warning alone -> never enough
console error alone -> never enough for arbitrary defect
subrequest 4xx/5xx alone -> never enough
successful action sequence alone -> never enough
insufficient cue -> INSUFFICIENT
```

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- DefectObserver.test.ts
```

- [ ] **Step 4: Add explicit observation expectations to the parsed plan model**

Do not infer fixture outcomes from route names inside production code. Extend the run request/plan only with general, user-auditable expectation primitives if needed, such as:

```text
EXPECT_TEXT "Profile saved"
EXPECT_NO_TEXT "Account details"
EXPECT_URL "/billing/error"
EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"
EXPECT_MAIN_STATUS 200
```

If these are necessary, update `docs/reprodocket-interface-contracts.md`, parser tests, UI syntax help, and this plan before implementing them. The parser must remain closed and deterministic.

- [ ] **Step 5: Implement observation truth conservatively**

The observer returns CONFIRMED only when an explicit supported observation contract is satisfied by current evidence. Otherwise it returns NOT_OBSERVED when the relevant healthy expectation was actually tested, or INSUFFICIENT when no authoritative determination can be made.

- [ ] **Step 6: Prove GREEN and anti-false-positive tests**

Use a test containing an unrelated console error plus a successful expected postcondition and require NOT_OBSERVED, not CONFIRMED.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/core/DefectObserver.ts reprodocket/src/shared/contracts/ObservationModels.ts reprodocket/src/core/ReproductionStepParser.ts reprodocket/src/shared/contracts/ReproductionAction.ts reprodocket/tests/unit/DefectObserver.test.ts reprodocket/tests/unit/ReproductionStepParser.test.ts docs/reprodocket-interface-contracts.md
git diff --cached --check
git commit -m "feat: classify defect observations from evidence"
```

---

## Task 3: Build one real attempt engine

**Files:**
- Create: `reprodocket/src/core/AttemptEngine.ts`
- Test: `reprodocket/tests/integration/AttemptEngine.test.ts`
- Test: `reprodocket/tests/live/AttemptEngine.live.test.ts`

**Interfaces:**
- Implements `AttemptEngine.runAttempt(run, role, steps, signal)`.
- Consumes `BrowserProvider`, `ReproductionStepExecutor`, `EvidenceCollector`, `DefectObserver`, `RunStore`, and `ResourceLedger`.

- [ ] **Step 1: Write failing deterministic integration tests with a browser-provider test double**

Require this order:

```text
create recorded browser
register session ID on attempt
create page
attach evidence bridge
capture baseline screenshot
execute step 1..N
capture semantic screenshots after required boundaries/failures
finish evidence collection
produce attempt observation/outcome
close browser in finally
evaluate replay after close
persist attempt
```

A thrown step must not skip browser close.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- AttemptEngine.test.ts
```

- [ ] **Step 3: Implement the engine around structured cleanup**

Use a single attempt-owned `AbortController`/caller signal. Persist enough state before starting external work that a crash leaves an identifiable active attempt.

- [ ] **Step 4: Connect evidence bridge to the real page**

The engine attaches the Plan 3 bridge immediately after page creation and before target navigation so navigation/page errors are observable.

- [ ] **Step 5: Capture screenshots at semantic boundaries**

Initial minimum:

```text
baseline after initial target ready
immediately before the action associated with the claimed failure when determinable
after the relevant action/postcondition
failure state on exception/timeout when a current frame is available
final attempt state
```

Do not take screenshots on arbitrary fixed intervals.

- [ ] **Step 6: Run deterministic integration tests GREEN**

```powershell
npm run test:integration -- AttemptEngine.test.ts
```

- [ ] **Step 7: Write and run one real Solari attempt live test**

Use the Solari-hosted fixture from Plan 3. Execute one healthy and one defective plan through the production `AttemptEngine` and require real evidence IDs/session ID/replay state/cleanup.

```powershell
npm run test:live -- AttemptEngine.live.test.ts
```

- [ ] **Step 8: Verify no unresolved browser resource remains**

A passed attempt test with an unresolved session fails the live test.

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/core/AttemptEngine.ts reprodocket/tests/integration/AttemptEngine.test.ts reprodocket/tests/live/AttemptEngine.live.test.ts
git diff --cached --check
git commit -m "feat: run evidence-backed Solari attempts"
```

---

## Task 4: Implement the run coordinator and atomic active-run admission

**Files:**
- Create: `reprodocket/src/core/RunCoordinator.ts`
- Create: `reprodocket/src/core/ActiveRunGate.ts`
- Test: `reprodocket/tests/integration/RunCoordinator.test.ts`
- Test: `reprodocket/tests/integration/ActiveRunGate.test.ts`

**Interfaces:**
- Implements `RunCoordinator.createAndStart`, `cancel`, `activeRunId`, and `shutdown`.
- One process allows one executing pipeline while history remains readable.

- [ ] **Step 1: Write failing G004/G005 tests for the gate**

Race two create requests with `Promise.allSettled`. Require exactly one admitted run and exactly one `RUN_ALREADY_ACTIVE` rejection.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- ActiveRunGate.test.ts
```

- [ ] **Step 3: Implement an in-process atomic gate**

JavaScript runs on one event loop, but admission may still race across awaits. The gate must reserve the active slot before any asynchronous persistence/provider operation and release it only after terminal cleanup/finalization.

- [ ] **Step 4: Write coordinator lifecycle tests**

Require:

```text
CREATED persisted
PREPARING
INVESTIGATING
first attempt persisted
VERIFYING
second attempt persisted
FINALIZING
report generation/sealing
COMPLETED
active gate released
```

On failures, require truthful FAILED/INCONCLUSIVE policy and cleanup before gate release.

- [ ] **Step 5: Prove RED**

```powershell
npm run test:integration -- RunCoordinator.test.ts
```

- [ ] **Step 6: Implement the coordinator**

The coordinator starts background run execution only after `createAndStart` has persisted the run and returns its authoritative ID. Exceptions are caught at the coordinator boundary, sanitized, persisted, and reflected into the event hub.

- [ ] **Step 7: Connect the two attempts**

After first attempt closes, create verification through a second `AttemptEngine.runAttempt` invocation that causes `BrowserProvider` to create a fresh session. Add an invariant check that the IDs differ before classification.

- [ ] **Step 8: Finalize report/evidence before COMPLETED**

`COMPLETED` is persisted only after required durable evidence, report generation, and integrity sealing succeed. Cleanup summary remains separate and can block FULL validation if incomplete.

- [ ] **Step 9: Prove GREEN and mutation check**

Temporarily reuse the first attempt's session ID in the coordinator test and confirm final VERIFIED assertion fails. Restore.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/src/core/RunCoordinator.ts reprodocket/src/core/ActiveRunGate.ts reprodocket/tests/integration/RunCoordinator.test.ts reprodocket/tests/integration/ActiveRunGate.test.ts
git diff --cached --check
git commit -m "feat: coordinate independent investigation verification"
```

---

## Task 5: Add run-create and cancellation HTTP routes

**Files:**
- Create: `reprodocket/src/server/routes/runsWrite.ts`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/integration/RunWriteRoutes.test.ts`

**Interfaces:**
- Implements `POST /api/runs` and `POST /api/runs/:runId/cancel`.
- Consumes local request guard and coordinator.

- [ ] **Step 1: Write failing G001-G005/G010-G011 tests**

Require validation to finish before any external adapter invocation. Invalid step grammar returns 400 with `INVALID_REPRODUCTION_STEP`. Active duplicate returns 409. Accepted create returns 202 and a run ID. Cancellation response means requested, not completed.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunWriteRoutes.test.ts
```

- [ ] **Step 3: Implement route schemas**

Use Zod/validated types. Enforce documented length/count limits server-side even if the UI also enforces them.

- [ ] **Step 4: Implement cancellation routing**

Cancel only the matching current active run. A terminal run cannot trigger another resource cleanup pass. Unknown run returns `RUN_NOT_FOUND`.

- [ ] **Step 5: Prove GREEN including foreign Origin/nonce guard**

```powershell
npm run test:integration -- RunWriteRoutes.test.ts LocalServerSecurity.test.ts
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/server/routes/runsWrite.ts reprodocket/src/server/createServer.ts reprodocket/tests/integration/RunWriteRoutes.test.ts
git diff --cached --check
git commit -m "feat: expose investigation lifecycle routes"
```

---

## Task 6: Connect the New Investigation UI to the real coordinator path

**Files:**
- Create: `reprodocket/src/client/components/NewInvestigationForm.tsx`
- Create: `reprodocket/src/client/components/ReproductionStepHelp.tsx`
- Modify: `reprodocket/src/client/App.tsx`
- Modify: `reprodocket/src/client/api/ApiClient.ts`
- Test: `reprodocket/tests/ui/NewInvestigation.spec.ts`

**Interfaces:**
- Uses `POST /api/runs` only; it does not call an internal engine directly.
- Navigates to `/runs/:runId` after the server accepts the run.

- [ ] **Step 1: Write failing H004-H008 tests**

Require target/problem/steps validation, optional whitespace normalization, visible step syntax help, one submission despite rapid double click, and server error projection without stack traces.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- NewInvestigation.spec.ts
```

- [ ] **Step 3: Implement the form**

Use one textarea with one reproduction statement per line. Show concise examples from the actual parser grammar. Validate client-side for immediacy but treat the server response as authority.

- [ ] **Step 4: Disable only after submission begins and re-enable on rejected admission**

A client button disable is not the concurrency authority. Server gate remains required.

- [ ] **Step 5: Navigate to the authoritative run**

On 202, route directly to `/runs/<id>` and let the existing run page hydrate from GET/SSE.

- [ ] **Step 6: Prove GREEN and accessibility**

```powershell
npm run build
npm run test:ui -- NewInvestigation.spec.ts
```

Check labels, error associations, keyboard submission, focus after error, and visible step help.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/client/components/NewInvestigationForm.tsx reprodocket/src/client/components/ReproductionStepHelp.tsx reprodocket/src/client/App.tsx reprodocket/src/client/api/ApiClient.ts reprodocket/tests/ui/NewInvestigation.spec.ts
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
- UI -> cancel API -> coordinator -> AbortController -> attempt cleanup -> terminal persistence -> UI.

- [ ] **Step 1: Write deterministic M018-M020/O006 tests**

Use controllable fake browser actions that wait on an AbortSignal. Cancel during investigation and during verification. Require no new step begins after cancellation requested, resources close once, final lifecycle CANCELLED, and final outcome is never promoted to VERIFIED.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunCancellation.test.ts
```

- [ ] **Step 3: Implement coordinator cancellation token ownership**

Only the active run's controller may be aborted. Cancellation request is idempotent.

- [ ] **Step 4: Implement cancel UI**

Show Cancel only when lifecycle is cancellable. After click, show `Cancellation requested` until server reports terminal state. Do not display CANCELLED immediately from client optimism.

- [ ] **Step 5: Run UI test**

```powershell
npm run build
npm run test:ui -- RunCancellation.spec.ts
```

- [ ] **Step 6: Add one bounded live cancellation test**

Use a deterministic fixture action with a long wait. Start a real Solari run, cancel, and require browser cleanup plus terminal CANCELLED. Do not create a deliberately harmful external mutation just to test cancellation.

```powershell
npm run test:live -- RunCancellation.live.test.ts
```

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/core/RunCoordinator.ts reprodocket/src/client/components/CancelRunButton.tsx reprodocket/src/client/pages/RunPage.tsx reprodocket/tests/integration/RunCancellation.test.ts reprodocket/tests/ui/RunCancellation.spec.ts reprodocket/tests/live/RunCancellation.live.test.ts
git diff --cached --check
git commit -m "feat: cancel active investigations safely"
```

---

## Task 8: Add complete fixture plans and prove all four outcome classes

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
- Fixture plan JSON contains only public CreateRunRequest/expectation fields and no privileged product hooks.

- [ ] **Step 1: Define each plan through the public step grammar**

Examples after observation grammar is finalized:

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

Healthy plans must assert the intended postcondition so NOT_REPRODUCED is evidence-backed rather than a default absence state.

- [ ] **Step 2: Write plan parse/oracle tests**

Every plan must parse using production grammar and reference a fixture route that the independent fixture truth tests prove.

- [ ] **Step 3: Prove RED for any missing observation grammar**

Run:

```powershell
npm run test:integration -- FixturePlans.test.ts
```

Update parser/contracts only through the design process recorded in Task 2.

- [ ] **Step 4: Prove all plan files valid**

Require exactly the expected plan count and no unknown fields.

- [ ] **Step 5: Commit**

```powershell
git add reprodocket/fixtures/plans reprodocket/tests/integration/FixturePlans.test.ts
git diff --cached --check
git commit -m "test: define end-to-end reproduction plans"
```

---

## Task 9: Execute the complete product path through real Solari

**Files:**
- Create: `reprodocket/tests/e2e/ReproDocketFullFlow.spec.ts`
- Create: `reprodocket/tests/e2e/E2eHarness.ts`
- Modify: `reprodocket/playwright.config.ts` if a separate e2e project is needed.

**Interfaces:**
- Executes L001-L014 through the real built local UI and real Solari-hosted fixture.
- The harness may prepare the fixture URL, but run creation must occur through the same user-facing form/API path a normal user uses.

- [ ] **Step 1: Write the E2E harness ownership boundary**

The harness:

```text
starts exact built ReproDocket server
creates one owned Solari sandbox fixture
externally verifies fixture version
opens local ReproDocket UI
submits plan through visible form
waits for terminal run state
reads UI result
reads persisted manifest via public/local API
asserts horizontal equality
kills fixture sandbox
stops owned local server
```

Cleanup lives in outer `finally` blocks so a UI assertion failure does not strand Solari resources.

- [ ] **Step 2: Add L001-L006 happy/negative outcome tests**

Require:

```text
account blank -> VERIFIED
billing refresh -> VERIFIED
missing ZIP -> VERIFIED
healthy profile -> NOT_REPRODUCED
healthy login -> NOT_REPRODUCED
ambiguous -> INCONCLUSIVE
```

- [ ] **Step 3: Add L007 nonrepeatable case**

Fixture state must make the first fresh browser reproduce and the second fresh browser not reproduce without ReproDocket knowing the fixture scenario identity. Expected final result REPRODUCED.

- [ ] **Step 4: Assert fresh-session proof L008-L010**

For every verified/nonrepeatable run:

```text
investigation sessionId non-null
verification sessionId non-null
IDs differ
first browser close recorded before verification session creation
first evidence attemptId differs from second
evidence ownership remains correct
```

- [ ] **Step 5: Assert local result parity L011-L013**

Run detail visible outcome, report JSON, history row, and persisted manifest all match. Restart the local ReproDocket server after at least one completed run and prove the history/detail remain available without creating a new Solari browser.

- [ ] **Step 6: Assert cleanup L014**

At suite end, no harness-owned browser or sandbox is unresolved. If provider confirmation is limited, record the strongest authoritative terminal state supported by the current SDK and do not overclaim.

- [ ] **Step 7: Run one scenario first**

```powershell
npx playwright test tests/e2e/ReproDocketFullFlow.spec.ts --grep "account blank"
```

Repair until one complete vertical path passes.

- [ ] **Step 8: Run all E2E scenarios**

```powershell
npx playwright test tests/e2e/ReproDocketFullFlow.spec.ts
```

- [ ] **Step 9: Mutation proof the harness**

Temporarily force the classifier to return VERIFIED for the healthy-profile case. Confirm the E2E suite fails. Restore and rerun healthy profile to PASS.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/tests/e2e reprodocket/playwright.config.ts
git diff --cached --check
git commit -m "test: prove ReproDocket end to end on Solari"
```

---

## Task 10: Close horizontal and vertical connectivity for M5-M7

**Files:**
- Modify: `reprodocket/docs/connectivity-matrix.md`
- Create: `reprodocket/tests/e2e/VerticalConnectivity.spec.ts`
- Modify: `reprodocket/tests/integration/HorizontalConsistency.test.ts`
- Modify: `reprodocket/tests/ui/HorizontalConsistency.spec.ts`
- Modify: `reprodocket/scripts/validate.ps1`

**Interfaces:**
- Adds N001-N015 and O001-O009 applicable complete paths to Full validation.

- [ ] **Step 1: Update every visible feature row from current evidence**

Do not mark COMPLETE based on implementation existence. Each row needs discoverability, entry, prerequisites, action routing, feedback, authority, persistence, reload, recovery, exit/return, regression coverage, and human-QA state.

- [ ] **Step 2: Write full vertical tests O002-O009**

At minimum prove:

```text
new investigation UI -> Solari -> evidence -> verification -> outcome -> UI
screenshot click -> artifact ownership -> integrity -> image response
replay state -> exact local evidence/replay authority
cancel UI -> cleanup -> terminal state
restart -> validated history -> detail
malformed historical artifact -> explicit damaged state without global failure
fixture validation -> sandbox -> browser A -> browser B -> local UI -> cleanup
```

- [ ] **Step 3: Run horizontal consistency on real E2E data**

Do not restrict consistency tests to synthetic local manifests once real run data exists.

- [ ] **Step 4: Add the complete product stages to Full validation**

Ordering after substrate contracts:

```text
fixture truth
attempt engine
coordinator
user-boundary UI
full outcome E2E
horizontal consistency
vertical connectivity
resource reconciliation
report/evidence integrity
```

- [ ] **Step 5: Run Full for the first time with a complete vertical slice**

```powershell
.\reprodocket\scripts\validate.ps1
```

Expected: only PASS if every mandatory implemented authority is current. Human subjective QA remains separately pending/ready rather than silently included in machine PASS.

- [ ] **Step 6: Capture product evidence at defect-resolving scale**

Using the built local app and current run data, capture original-resolution views of:

```text
New Investigation
Active Investigation
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
History
Screenshot evidence
Console evidence
Network evidence
Timeline
Replay state
representative failure/cancellation
```

Use background-safe browser capture. No desktop-wide input/capture is required for a local web UI claim.

- [ ] **Step 7: Inspect and repair objective UI defects before moving to hardening**

Blocking examples:

```text
truncated labels
horizontal overflow
raw stack trace
unreadable small text
result/status contradiction
dead replay/cancel control
history/detail mismatch
wrong run screenshot
```

Each repair receives a regression test.

- [ ] **Step 8: Commit connectivity closure**

```powershell
git add reprodocket/docs/connectivity-matrix.md reprodocket/tests/e2e/VerticalConnectivity.spec.ts reprodocket/tests/integration/HorizontalConsistency.test.ts reprodocket/tests/ui/HorizontalConsistency.spec.ts reprodocket/scripts/validate.ps1
git diff --cached --check
git commit -m "test: close ReproDocket vertical integration"
```

---

## Plan 4 completion gate

Do not begin final hardening while any ordinary supported path is partial or disconnected.

Current evidence must prove:

```text
all supported reproduction grammar executes through real browser semantics
observation rules cannot treat unrelated console/network noise as automatic bug proof
one AttemptEngine path works against real Solari
first and second attempts are fresh independent Solari sessions
all four final outcome classes are demonstrated
one-active-run admission survives duplicate request races
cancellation is end-to-end and resource-aware
New Investigation UI submits the real production path
all deterministic fixture plans are independently truthful
real Solari E2E covers positive, negative, ambiguous, and nonrepeatable cases
completed runs survive local app restart
reports/history/detail/evidence agree with the manifest
all owned Solari resources are reconciled
horizontal and vertical connectivity matrices contain no UNKNOWN/PARTIAL ordinary supported feature
Full machine validation has a current result for the exact source revision
human visual/usability state is explicitly separate
```

Run:

```powershell
.\reprodocket\scripts\validate.ps1
git status --short
git diff main...HEAD --check
```

A machine Full PASS at this point means implementation/integration is ready for the deliberate bug-finding and hardening plan. It does not yet mean publication ready.