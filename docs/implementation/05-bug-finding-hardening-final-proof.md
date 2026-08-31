# ReproDocket Bug Finding, Hardening, and Final Proof Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Subject the front-to-back ReproDocket product to a deliberate completeness audit, adversarial failure injection, targeted security/lifecycle hardening, harness sensitivity checks, fresh Full validation, and a separate human visual/usability acceptance gate.

**Architecture:** This plan does not add speculative product breadth. It audits the finished supported scope, repairs concrete defects at their earliest responsible boundary, strengthens regression coverage for every escaped defect, and revalidates the entire source-to-runtime-to-evidence path after shared changes.

**Tech Stack:** Existing ReproDocket stack and harness, Vitest, Playwright Test, PowerShell 7, real Solari browser/sandbox resources where the affected boundary requires them.

**Spec:** `docs/reprodocket-design.md`; security/lifecycle: `docs/reprodocket-security-lifecycle.md`; test catalog: `docs/reprodocket-test-matrix.md`; connectivity: `reprodocket/docs/connectivity-matrix.md`.

## Global Constraints

* Plan 4 must have produced a current machine Full validation result before this audit begins.
* A previous Full PASS is baseline evidence, not permission to skip the hardening rerun.
* Every visible feature is inventoried from the actual built application, not from intended architecture alone.
* UNKNOWN, PARTIAL, DISCONNECTED, DEAD, or misleading ordinary-scope behavior blocks publication until fixed or truthfully removed/disabled.
* Repair the earliest responsible boundary. Do not add UI patches that conceal a store, lifecycle, adapter, or evidence defect.
* Shared-boundary changes invalidate dependent evidence and trigger appropriate wider reruns.
* Do not manufacture dangerous external events solely for proof.
* Security hardening remains proportional to a loopback, single-user local application but must cover its real attack surfaces.
* Human visual/usability judgment is distinct from automated Full validation.
* Any human-discovered defect that automation could reasonably detect receives a corresponding harness repair.

---

## Task 1: Perform the product-wide visible-feature completeness audit

**Files:**
- Modify: `reprodocket/docs/connectivity-matrix.md`
- Create: `reprodocket/docs/completeness-audit.md`
- Create: `reprodocket/tests/ui/VisibleFeatureInventory.spec.ts`

**Interfaces:**
- Produces an evidence-backed inventory of every visible screen/control/state in declared version-one scope.

- [ ] **Step 1: Launch the exact built application from a clean current build**

Run:

```powershell
npm ci
npm run build
.\reprodocket\scripts\run.ps1
```

Record the source revision and local build identity used for the audit.

- [ ] **Step 2: Inventory every visible product surface**

At minimum enumerate:

```text
Solari connection/configuration
New Investigation form
reproduction/expectation syntax help
active progress
cancel
history
run detail
investigation result
verification result
final outcome
cleanup result
screenshots
console evidence
page-error evidence
network evidence
timeline
replay state/report
interrupted run
damaged run
not-found route
provider error
run error
```

For every button, link, tab, disclosure, input, status badge, and navigation element, record COMPLETE/PARTIAL/DISCONNECTED/DEAD/STALE/INTENTIONALLY_DISABLED/DEFERRED/UNKNOWN.

- [ ] **Step 3: Write `VisibleFeatureInventory.spec.ts` to enforce discoverability and no dead interactive controls**

The test does not merely count buttons. It opens representative run states and verifies every visible interactive control either produces its documented transition/navigation or exposes a truthful disabled state/reason.

- [ ] **Step 4: Exercise round-trip behavior for each feature**

For each row, verify:

```text
discover -> enter -> satisfy prerequisite -> act -> receive feedback -> authoritative state -> persist -> reload -> recover if relevant -> leave/return
```

- [ ] **Step 5: Repair every ordinary-scope UNKNOWN/PARTIAL/DISCONNECTED/DEAD/STALE row**

Each repair first gets a failing regression test at the lowest meaningful boundary, then the affected vertical path is rerun.

- [ ] **Step 6: Re-run the visible-feature test and relevant connectivity tests**

```powershell
npm run build
npm run test:ui -- VisibleFeatureInventory.spec.ts HorizontalConsistency.spec.ts
```

- [ ] **Step 7: Commit only after the audit has no unresolved ordinary-scope visible gaps**

```powershell
git add reprodocket/docs/connectivity-matrix.md reprodocket/docs/completeness-audit.md reprodocket/tests/ui/VisibleFeatureInventory.spec.ts <repair-files>
git diff --cached --check
git commit -m "test: close ReproDocket visible feature gaps"
```

---

## Task 2: Execute the full failure-injection catalog

**Files:**
- Create: `reprodocket/tests/failure/FailureInjection.test.ts`
- Create: `reprodocket/tests/failure/FailureInjection.live.test.ts`
- Modify: production files only when an injected case reveals a real defect.

**Interfaces:**
- Covers M001-M025 and applicable failure/security cases from `docs/reprodocket-test-matrix.md`.

- [ ] **Step 1: Implement deterministic failure adapters**

Create test-only adapters that can fail at exact boundaries:

```text
credential verification
browser create
page create
navigation
action execution
evidence write
manifest update
verification browser create
replay readiness
sandbox create
fixture start
cleanup
```

They must implement production interfaces rather than introducing test-only branches into production code.

- [ ] **Step 2: Assert lifecycle, outcome, evidence, cleanup, and next-action behavior for every deterministic failure**

A test succeeds only when all relevant dimensions are correct. Merely throwing the expected exception is insufficient.

- [ ] **Step 3: Prove the suite can distinguish infrastructure failure from target result**

Examples:

```text
browser create failure != NOT_REPRODUCED
verification unavailable != VERIFIED
manifest write failure != COMPLETED
cleanup failure != cleanup PASS
```

- [ ] **Step 4: Run deterministic failure suite**

```powershell
npx vitest run tests/failure/FailureInjection.test.ts
```

- [ ] **Step 5: Run safe live failure cases**

Use only safe/cheap live cases that add real authority, such as an intentionally invalid credential, fixture 500, bounded timeout, known nonzero sandbox command, replay-not-ready polling, and live cancellation.

Do not intentionally consume all account concurrency or create an uncontrolled provider failure.

- [ ] **Step 6: Repair defects test-first**

For each discovered bug, preserve the failing case, fix the root boundary, run the smallest affected suite, then run the relevant vertical path.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/tests/failure <repair-files>
git diff --cached --check
git commit -m "test: harden ReproDocket failure behavior"
```

---

## Task 3: Harden target URL and redirect safety against the actual browser path

**Files:**
- Modify: `reprodocket/src/core/TargetUrlPolicy.ts`
- Create: `reprodocket/src/core/NavigationPolicy.ts`
- Modify: `reprodocket/src/core/ReproductionStepExecutor.ts`
- Create: `reprodocket/tests/integration/NavigationPolicy.test.ts`
- Create: `reprodocket/tests/live/NavigationPolicy.live.test.ts`

**Interfaces:**
- Closes C023/C024 where current browser/network APIs make authoritative enforcement possible.

- [ ] **Step 1: Create controlled redirect fixtures**

Use the deterministic fixture to expose:

```text
public -> public redirect
public -> prohibited localhost literal redirect if representable
public -> cloud metadata literal redirect
```

Do not probe a real cloud metadata service. The test needs only to prove policy rejects/aborts the destination before sensitive content is consumed.

- [ ] **Step 2: Write failing navigation-policy tests**

Assert every new main-frame URL is checked and prohibited destinations abort the action/run with `BLOCKED_TARGET_NETWORK` or a documented equivalent.

- [ ] **Step 3: Determine whether DNS resolution is visible authoritatively through the installed browser/API**

If yes, enforce private-resolution blocking. If no, document that DNS rebinding/private resolution cannot be fully established at the application layer and avoid making a false SSRF-complete claim.

- [ ] **Step 4: Run local and live controlled tests**

```powershell
npm run test:integration -- NavigationPolicy.test.ts
npm run test:live -- NavigationPolicy.live.test.ts
```

- [ ] **Step 5: Update security documentation with tested scope**

State precisely what is blocked pre-navigation and what redirect/DNS behavior is enforced or remains provider-dependent.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/core/TargetUrlPolicy.ts reprodocket/src/core/NavigationPolicy.ts reprodocket/src/core/ReproductionStepExecutor.ts reprodocket/tests/integration/NavigationPolicy.test.ts reprodocket/tests/live/NavigationPolicy.live.test.ts docs/reprodocket-security-lifecycle.md
git diff --cached --check
git commit -m "security: harden remote target navigation"
```

---

## Task 4: Run the local-service and untrusted-content security audit

**Files:**
- Create: `reprodocket/tests/security/LocalServiceSecurity.test.ts`
- Create: `reprodocket/tests/security/ContentSafety.spec.ts`
- Modify: production files only for failures.

**Interfaces:**
- Covers D001-D015, E008-E014, F017-F020, H034, and the security cases defined in `docs/reprodocket-security-lifecycle.md`.

- [ ] **Step 1: Verify loopback exposure with an actual listener**

Use OS socket inspection or an attempted connection strategy that can prove the server is bound only to loopback. Do not rely only on a configuration string assertion.

- [ ] **Step 2: Attack local mutation routes**

Test wrong Host, foreign Origin, missing/wrong nonce, duplicate/raced mutation, oversized request, malformed JSON, and unsupported content type.

- [ ] **Step 3: Attack artifact paths**

Test plain and encoded traversal, malformed IDs, run/artifact mismatch, absolute path attempts, symlink/reparse escape where supported.

- [ ] **Step 4: Attack displayed/generated text**

Persist strings containing script tags, event handler attributes, SVG payloads, malformed entities, bidi/control characters, and markup-like console text. Prove the built UI and HTML report render inert text.

- [ ] **Step 5: Inject a known fake key through every likely text channel**

Search all generated run files, reports, validation summaries, and logs for the exact fake secret. The test fails on one occurrence.

- [ ] **Step 6: Inspect response headers from the real built server**

Confirm CSP, nosniff, referrer policy, no wildcard CORS, and no server/debug header leaking unnecessary internals where controllable.

- [ ] **Step 7: Repair and rerun until the complete security suite is green**

```powershell
npx vitest run tests/security/LocalServiceSecurity.test.ts
npx playwright test tests/security/ContentSafety.spec.ts
```

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/tests/security <repair-files>
git diff --cached --check
git commit -m "security: close ReproDocket local attack surfaces"
```

---

## Task 5: Audit resource ownership, retries, cancellation, and shutdown under stress

**Files:**
- Create: `reprodocket/tests/lifecycle/ResourceLifetime.test.ts`
- Create: `reprodocket/tests/lifecycle/ResourceLifetime.live.test.ts`
- Create: `reprodocket/tests/lifecycle/RepeatedRuns.live.test.ts`
- Modify: lifecycle/provider files only as defects require.

**Interfaces:**
- Covers P001-P012 plus repeatability/lifetime behavior not established by one clean run.

- [ ] **Step 1: Execute repeated deterministic local lifecycles**

Run at least 50 coordinator cycles with test doubles covering successful, not-reproduced, inconclusive, cancelled, and injected-failure runs. At every terminal state require the active gate released and no duplicate listener/resource record growth.

- [ ] **Step 2: Execute a bounded repeated live sequence**

Use a small count sufficient to expose session lifecycle leakage without wasting provider resources, initially 5 full browser attempt pairs against one fixture sandbox if the provider supports this economically.

Require unique session IDs and clean closure every iteration.

- [ ] **Step 3: Test shutdown in every active lifecycle stage with deterministic adapters**

At minimum PREPARING, INVESTIGATING, VERIFYING, FINALIZING. Ensure shutdown stops admission, requests cancellation, closes known resources, and persists truthful terminal/uncertain state.

- [ ] **Step 4: Test retry bounds**

Ensure a retryable control-plane failure stops after configured attempts; 429 does not spin; unknown target mutation outcomes do not auto-repeat.

- [ ] **Step 5: Check open handles after suites**

No leaked timer, SSE subscriber, server socket, child process, or Solari client handle may keep Node alive after the owning test/app is closed.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/tests/lifecycle <repair-files>
git diff --cached --check
git commit -m "test: harden ReproDocket resource lifetimes"
```

---

## Task 6: Audit persistence under interruption, corruption, and concurrent reads

**Files:**
- Create: `reprodocket/tests/persistence/PersistenceHardening.test.ts`
- Modify: storage files only as defects require.

**Interfaces:**
- Extends F001-F020 and restart/recovery behavior.

- [ ] **Step 1: Inject process-style interruption around atomic writes**

Simulate failure before temp write completes, after temp write before rename, and after rename. On restart, require either the last valid state or a detectable recovery condition, never silently accepted truncated JSON.

- [ ] **Step 2: Read history repeatedly during writes**

Ensure concurrent readers see valid previous/new snapshots or a bounded explicit transient state, not malformed partial JSON.

- [ ] **Step 3: Corrupt one run among many**

History and other run detail pages remain available. Damaged run remains visible with truthful diagnostic state.

- [ ] **Step 4: Tamper every artifact class**

Screenshot, console JSON, network JSON, timeline JSON, report HTML/JSON, replay data if local. Finalized artifact serving must reject mismatches.

- [ ] **Step 5: Verify retention/cleanup boundaries**

Any cleanup command may remove only known generated application-owned data after explicit scope selection. Source and unrelated user files are never candidates.

- [ ] **Step 6: Run and commit**

```powershell
npx vitest run tests/persistence/PersistenceHardening.test.ts
git add reprodocket/tests/persistence <repair-files>
git diff --cached --check
git commit -m "test: harden ReproDocket persistence recovery"
```

---

## Task 7: Harden Windows bootstrap and fresh-checkout behavior

**Files:**
- Create: `reprodocket/tests/bootstrap/FreshCheckoutHarness.test.ts`
- Modify: `reprodocket/scripts/bootstrap.ps1`
- Modify: `reprodocket/scripts/preflight.ps1`
- Modify: `reprodocket/scripts/run.ps1`
- Modify: `reprodocket/scripts/stop.ps1`
- Modify: `reprodocket/scripts/validate.ps1`

**Interfaces:**
- Proves the public setup path does not depend on the developer's previously configured machine state.

- [ ] **Step 1: Create a temporary clean checkout/copy from the exact Git revision**

Prefer `git worktree` or a fresh clone into scoped test storage. Do not use Desktop/Downloads/Documents. Do not share `node_modules` accidentally.

- [ ] **Step 2: Run bootstrap with no project dependencies installed**

Require dependency restore from the committed lockfile and creation only of ignored/generated state.

- [ ] **Step 3: Run from a non-repository current directory**

Call scripts by absolute/relative script path and require `$PSScriptRoot`-based resolution.

- [ ] **Step 4: Test current prerequisite variants without damaging the host**

Use controlled PATH/environment to simulate missing npm/Node/pwsh where feasible. Installation code paths that would mutate global machine state may be tested through a command-runner seam, then manually exercised only when the host genuinely needs that prerequisite.

- [ ] **Step 5: Verify unsupported old runtime is rejected**

No false PASS merely because `node.exe` exists.

- [ ] **Step 6: Verify disk reserve behavior**

Inject free-space provider values and ensure Full/live artifact-heavy work refuses to start below reserve while normal read-only startup remains proportionate.

- [ ] **Step 7: Verify no WMIC and no user-folder scratch**

Source scan scripts and runtime-created paths.

- [ ] **Step 8: Run a real clean-checkout build/test/start/stop cycle**

```powershell
.\reprodocket\scripts\bootstrap.ps1
.\reprodocket\scripts\preflight.ps1 -LocalOnly
.\reprodocket\scripts\run.ps1
.\reprodocket\scripts\stop.ps1
```

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/tests/bootstrap reprodocket/scripts
git diff --cached --check
git commit -m "test: prove ReproDocket fresh Windows setup"
```

---

## Task 8: Run accessibility and native-scale usability hardening

**Files:**
- Create: `reprodocket/tests/ui/Accessibility.spec.ts`
- Create: `reprodocket/tests/ui/ResponsiveReadability.spec.ts`
- Modify: UI files only as findings require.

**Interfaces:**
- Covers H028-H035 and Q014-Q015 without claiming a formal accessibility standard.

- [ ] **Step 1: Automate semantic checks**

Assert labels/names/roles, keyboard reachability, visible focus, outcome text independent of color, status/live-region behavior, tab/section semantics, disabled-reason availability, and reduced-motion CSS behavior if animation exists.

- [ ] **Step 2: Exercise representative viewport widths**

At minimum:

```text
1280x720
1440x900
1920x1080
1024x768
768x900
```

Use narrower widths only if declared supported. Critical actions/status/evidence must remain visible without accidental horizontal page scrolling.

- [ ] **Step 3: Exercise browser zoom**

Inspect 100%, 125%, and 150% for the primary workflow. Browser automation may simulate equivalent viewport/device scale where direct zoom control is unreliable, but human review must still inspect actual browser zoom at least once.

- [ ] **Step 4: Inspect long content**

Use long URL, 10,000-character problem near the supported limit, long console message, long network URL, and 50 reproduction/expectation steps. Layout may wrap/scroll within intended panels but cannot overlap or make navigation unusable.

- [ ] **Step 5: Fix and rerun**

```powershell
npm run build
npx playwright test tests/ui/Accessibility.spec.ts tests/ui/ResponsiveReadability.spec.ts
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/tests/ui/Accessibility.spec.ts reprodocket/tests/ui/ResponsiveReadability.spec.ts <repair-files>
git diff --cached --check
git commit -m "test: harden ReproDocket usability"
```

---

## Task 9: Prove the harness is sensitive to critical regressions

**Files:**
- Create: `reprodocket/docs/harness-sensitivity.md`
- Modify: tests only when a mutation reveals weak coverage.

**Interfaces:**
- Executes representative S001-S018 mutations and records red/green evidence.

- [ ] **Step 1: Select at least one mutation from each critical class**

Required classes:

```text
classification independence
URL/network safety
local mutation security
XSS/report escaping
secret redaction
artifact ownership/integrity
active-run concurrency
cancellation truth
Solari resource cleanup
fixture anti-cheat/false positive
source revision provenance
```

- [ ] **Step 2: For each selected mutation, deliberately introduce the defect in an uncommitted working tree**

Run the exact guarding test and require FAIL for the intended reason.

- [ ] **Step 3: Restore only the deliberate mutation safely**

Because destructive Git restore is not allowed as a general workflow shortcut, preserve the original content before mutation or use a controlled patch reversal limited to the exact mutation.

- [ ] **Step 4: Rerun the same test and require PASS**

Record:

```text
mutation ID
guarding test
failure observed
restored pass observed
source revision
```

- [ ] **Step 5: Strengthen any test that fails to detect its mutation**

A green test under its target deliberate regression is itself a harness defect.

- [ ] **Step 6: Commit the sensitivity report/tests only after all deliberate mutations are gone**

```powershell
git diff --check
git status --short
git add reprodocket/docs/harness-sensitivity.md <test-repairs>
git diff --cached --check
git commit -m "test: prove ReproDocket harness sensitivity"
```

---

## Task 10: Run dependency, secret, and public-source hygiene audit

**Files:**
- Create: `reprodocket/scripts/audit-public.ps1`
- Create: `reprodocket/tests/integration/PublicAuditScript.test.ts`
- Modify: `.gitignore` and public docs only as findings require.

**Interfaces:**
- Produces a deterministic prepublication scan with nonzero exit on blocking findings.

- [ ] **Step 1: Write failing audit-script test**

Seed a temporary source tree containing a fake API key, absolute developer path, accidental `.env`, generated evidence file, `TODO` in public UI, and `dangerouslySetInnerHTML`; require the audit to reject the appropriate classes without scanning arbitrary user directories.

- [ ] **Step 2: Implement repository-scoped scans**

At minimum scan tracked/staged candidate files for:

```text
realistic Solari key patterns
.env files other than .env.example
absolute Windows developer paths
private machine/user path fragments
internal stack traces accidentally committed
TODO/FIXME/placeholder/stub markers in production UI/public docs at release gate
dangerouslySetInnerHTML in production UI
wmic/wmic.exe in new scripts
tracked generated evidence/runtime folders
fixture scenario IDs referenced from production classification code
```

Allow known intentional occurrences only through narrow documented allowlists.

- [ ] **Step 3: Run `npm audit` without pretending every advisory is exploitable**

Record severity, package, dependency path, fix availability, and whether it reaches shipped runtime. Any high/critical reachable shipped-runtime issue blocks release until fixed or explicitly justified with evidence.

- [ ] **Step 4: Verify license/attribution preservation**

Confirm root MIT license remains unchanged unless an additive licensing file is deliberately needed for new code. Do not remove Pinetree Research copyright from upstream license.

- [ ] **Step 5: Run audit and commit**

```powershell
.\reprodocket\scripts\audit-public.ps1
npm audit
git add reprodocket/scripts/audit-public.ps1 reprodocket/tests/integration/PublicAuditScript.test.ts <repair-files>
git diff --cached --check
git commit -m "test: audit ReproDocket public source hygiene"
```

---

## Task 11: Run the first post-hardening fresh Full validation

**Files:**
- Modify: `reprodocket/scripts/validate.ps1` only if the hardening suites are not yet included.
- Modify: `reprodocket/docs/validation.md`.

- [ ] **Step 1: Add every new hardening authority to Full**

Full ordering should stop cheap failures before live resources and include:

```text
preflight/fresh source identity
format/lint/typecheck/build
unit
integration
security
persistence
bootstrap contract
built UI
accessibility/readability
fixture truth
Solari browser contract
Solari replay
Solari sandbox contract
live evidence bridge
full outcome E2E
failure/recovery applicable live cases
horizontal connectivity
vertical connectivity
resource/lifetime
public audit
final evidence/provenance summary
```

- [ ] **Step 2: Start from a fresh dependency restore**

```powershell
npm ci
.\reprodocket\scripts\validate.ps1
```

- [ ] **Step 3: Do not accept stale artifacts**

Validation output must carry the current source revision. If any shared harness/storage/adapter code changes after this run, rerun affected authorities and ultimately Full again.

- [ ] **Step 4: Review Full result**

Any FAIL or BLOCKED mandatory authority prevents transition to human acceptance. Human-QA status itself is READY/PENDING, not machine PASS.

- [ ] **Step 5: Commit only documentation/orchestration changes that were validated**

```powershell
git add reprodocket/scripts/validate.ps1 reprodocket/docs/validation.md
git diff --cached --check
git commit -m "test: integrate ReproDocket hardening gates"
```

---

## Task 12: Prepare and execute the separate human visual/usability gate

**Files:**
- Create: `reprodocket/docs/human-qa-checklist.md`
- Create: `reprodocket/docs/visual-evidence-manifest.md` or generate it into ignored evidence with a small public template.
- Modify: UI/tests only for findings.

**Interfaces:**
- Human acceptance judges subjective clarity/polish while automated checks remain authoritative for deterministic facts.

- [ ] **Step 1: Produce a fresh evidence catalog from the exact post-hardening source revision**

Required states from Q001-Q015:

```text
connection
new investigation
active investigation
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
history
screenshot evidence
console evidence
network evidence
replay ready/pending/unavailable as applicable
representative error
narrow supported viewport
125/150 percent readability
```

- [ ] **Step 2: Capture background-safe browser screenshots at original resolution**

Because ReproDocket is a web UI, Playwright page screenshots are the preferred product-surface evidence. Do not use broad desktop capture or move the user's real pointer merely to prove web layout.

- [ ] **Step 3: Present exact inspection criteria**

Human reviewer judges:

```text
visual hierarchy
readability
status/outcome distinction
evidence provenance comprehensibility
investigation vs verification distinction
action affordance truth
error usefulness
empty-state quality
clipping/overlap
professional polish
```

- [ ] **Step 4: Record PASS/FAIL/UNCERTAIN per state**

Do not collapse uncertainty into PASS. Preserve failing screenshots.

- [ ] **Step 5: Repair escaped defects**

For every defect automation could have caught, first add/strengthen a deterministic test, prove it fails, then fix the product and prove it passes.

- [ ] **Step 6: Re-run affected validation after each repair**

A UI-only repair may begin with targeted UI/build checks. A shared state/evidence/lifecycle repair requires broader affected validation.

- [ ] **Step 7: Commit repaired product and harness together**

Use descriptive defect-specific commits when multiple independent findings exist rather than one opaque `fix QA` commit.

---

## Task 13: Run the final fresh Full validation after human-QA repairs

**Files:**
- No planned production files; changes indicate a discovered defect.

- [ ] **Step 1: Verify the worktree and exact source state**

```powershell
git status --short
git rev-parse HEAD
```

No unreviewed source mutation or generated evidence should be staged accidentally.

- [ ] **Step 2: Restore dependencies from the committed lockfile**

```powershell
npm ci
```

- [ ] **Step 3: Run Full from the final candidate source**

```powershell
.\reprodocket\scripts\validate.ps1
```

- [ ] **Step 4: Require zero mandatory FAIL/BLOCKED authorities**

Record exact test counts/stage results from output rather than copying historical numbers from this plan.

- [ ] **Step 5: Verify human-QA outcome belongs to the same build/revision**

If any human-facing source changed after the accepted captures, recapture affected states.

- [ ] **Step 6: Verify repository diff/hygiene**

```powershell
git diff --check
git status --short
git diff main...HEAD --stat
.\reprodocket\scripts\audit-public.ps1
```

- [ ] **Step 7: Produce the final validation identity**

The completion evidence records:

```text
source revision
application version
lockfile/package versions
validation profile FULL
validation start/end
test stage results
fixture version
Solari live authority results
resource reconciliation
human-QA result/evidence manifest
known limitations, if any
```

Publication preparation may begin only after this candidate is stable.

---

## Plan 5 completion gate

The product is ready for publication preparation only if current evidence establishes all of the following:

```text
visible feature inventory has no unresolved ordinary-scope partial/dead/disconnected/unknown controls
failure injection covers the real lifecycle and keeps infrastructure failures distinct from target outcomes
remote target policy is tested to the strongest enforceable boundary without overstated SSRF claims
loopback service rejects foreign mutation and unsafe artifact access
malicious target text remains inert in UI and report
known secrets are absent from durable evidence/public output
repeat runs do not leak sessions/listeners/processes
shutdown/cancellation/retry bounds are truthful
persistence survives interruption/corruption without global history failure
fresh Windows setup works from an actual clean checkout
UI is usable at representative native sizes/zoom and critical keyboard semantics work
critical harness guards have proven mutation sensitivity
public audit has no blocking secret/private-path/debug/placeholder findings
post-hardening Full validation is fresh
human visual/usability review is explicitly recorded for the same candidate
any escaped automatable defects have product and harness repairs
final Full validation after all repairs is fresh and mandatory-gate clean
```

Nothing in this plan authorizes a claim of formal security certification, WCAG conformance, universal SSRF prevention, arbitrary-site autonomous bug discovery, or provider-level resource state beyond what the actual tests prove.