# ReproDocket Implementation Index

Date: August 31, 2026
Status: Pre-implementation execution map

This directory contains the ordered implementation plans for ReproDocket. They are deliberately split so each phase ends in working, testable software and has an explicit gate before the next phase begins.

## Read first

Before implementation, read these documents in this order:

1. [`../reprodocket-design.md`](../reprodocket-design.md) — product scope and architecture.
2. [`../reprodocket-interface-contracts.md`](../reprodocket-interface-contracts.md) — public/internal component contracts and run model.
3. [`../reprodocket-security-lifecycle.md`](../reprodocket-security-lifecycle.md) — trust boundaries, secrets, target safety, ownership, retries, recovery.
4. [`../reprodocket-sdk-baseline.md`](../reprodocket-sdk-baseline.md) — current Solari assumptions and required execution-time probes.
5. [`../reprodocket-test-matrix.md`](../reprodocket-test-matrix.md) — minimum validation catalog and harness-sensitivity expectations.

Current source, installed SDK type declarations, and live observed behavior outrank stale example assumptions. If a material contract must change, update the relevant design/contract document and the affected plan before continuing.

## Required plan order

### Plan 1 — Foundation and local shell

[`01-foundation-local-shell.md`](01-foundation-local-shell.md)

Builds:

```text
reproducible Node/TypeScript project
real test harness
closed reproduction-step parser
safe target URL policy
versioned run model/lifecycle
filesystem run store
secure loopback Fastify shell
built React shell
owned instance/port handling
Windows bootstrap/preflight/run/stop
protected Solari credential storage
provider readiness UI
first validation entry point
```

Exit gate: M0/M1 local foundation is executable and Targeted validation is green. Full remains truthfully blocked on live/product authorities not yet implemented.

### Plan 2 — Evidence, persistence, and local results

[`02-evidence-persistence-ui.md`](02-evidence-persistence-ui.md)

Builds:

```text
central redaction
run-owned artifacts
SHA256 sealing/integrity
structured console/page/network/timeline evidence
safe static HTML/JSON reports
history/detail/artifact APIs
SSE progress
History and Run Detail UI
active rehydration
interrupted-run recovery
horizontal consistency checks
connectivity matrix
```

Exit gate: M2 local evidence subsystem is complete and all implemented local surfaces agree on authoritative state.

### Plan 3 — Solari browser and sandbox substrate

[`03-solari-browser-sandbox.md`](03-solari-browser-sandbox.md)

Builds/proves:

```text
installed Solari Browser SDK contract
recorded browser create/use/close
clean Node client shutdown
Solari resource ledger
bounded provider retry policy
page evidence bridge
replay readiness/download capability probe
installed Solari Sandbox SDK contract
deterministic defective fixture
Solari-hosted fixture preview
external preview verification
live page evidence capture
live validation stages
```

Exit gate: M3/M4 real Solari substrate is current, resource-owned, and directly exercised.

### Plan 4 — Investigation, verification, and complete vertical product path

[`04-investigation-verification-e2e.md`](04-investigation-verification-e2e.md)

Builds:

```text
deterministic accessible step execution
evidence-backed observation rules
AttemptEngine
atomic one-active-run coordinator
fresh verification session
create/cancel APIs
New Investigation UI wiring
cancellation lifecycle
fixture reproduction/expectation plans
all four outcome classes
real Solari E2E through local UI
horizontal and vertical connectivity closure
```

Exit gate: M5-M7 front-to-back product is functionally complete and has a current Full machine validation result. This is not publication readiness.

### Plan 5 — Bug finding, hardening, and final proof

[`05-bug-finding-hardening-final-proof.md`](05-bug-finding-hardening-final-proof.md)

Audits/hardens:

```text
visible feature completeness
failure injection
remote target/redirect safety
loopback/request/artifact security
untrusted-content rendering
secret leakage
resource lifetime/retries/shutdown
persistence corruption/interruption
fresh Windows setup
accessibility/native-scale readability
harness mutation sensitivity
dependency/public-source hygiene
post-hardening Full validation
human visual/usability QA
escaped-defect regression feedback
final Full validation
```

Exit gate: M8-M10 has no known ordinary-scope hardening blocker, human QA is explicitly recorded, and the final Full result is fresh after all fixes.

### Plan 6 — Publication and challenge submission

[`06-publication-challenge-submission.md`](06-publication-challenge-submission.md)

Produces:

```text
truthful project README
restrained root cookbook README entry
safe reproducible demo material
short technical walkthrough
fresh post-documentation Full validation
complete outside-reader GitHub audit
challenge post draft/demo plan
final integration/PR steps
```

Exit gate: M11 is ready for the repository/social publication decision. Public posting and final merge remain explicit owner-authority boundaries.

## Milestone dependency chain

```text
M0 Foundation
  -> M1 Local Shell
  -> M2 Evidence Core
  -> M3 Solari Browser
  -> M4 Solari Fixture Infrastructure
  -> M5 Investigation
  -> M6 Independent Verification
  -> M7 Complete Local Results UI / Vertical Closure
  -> M8 Bug Finding and Adversarial Audit
  -> M9 Hardening
  -> M10 Final Proof and Human QA
  -> M11 Publication / Challenge Submission
```

Do not skip from M3 to a showcase, or from M7 to publication. The deliberate bug-finding and second-validation sequence is part of product completion.

## Validation vocabulary

`TARGETED` means the smallest sufficient real checks for a bounded change.

`FULL` means the complete current automated/machine authority required for milestone/release claims, including live Solari when applicable.

Human visual/usability acceptance is a separate authority. A machine Full PASS does not silently imply human QA PASS.

`BLOCKED` means a required authority could not be exercised because its prerequisite is genuinely unavailable. It does not count as PASS.

## Test-first task rule

Every behavior-changing implementation task follows this sequence:

```text
identify the authoritative boundary
write the regression/acceptance test
run it and observe the expected failure
implement the smallest complete behavior
run the affected test and observe pass
run adjacent regression/static checks
perform mutation/sensitivity proof when the guard is critical
inspect repository/diff
commit only the validated related slice
```

Tests for live external behavior may first use an interface double to prove local lifecycle handling, but the relevant live Solari authority must still run before that capability is promoted.

## Bug repair rule

When a defect is found:

1. Preserve the failing evidence.
2. Identify the earliest responsible production boundary.
3. Add or strengthen the test that should fail for the defect.
4. Prove the test fails before the fix when safely possible.
5. Repair the root boundary.
6. Prove the targeted test passes.
7. Check sibling paths sharing the same state/store/router/adapter/lifecycle boundary.
8. Run the affected vertical path.
9. Invalidate/re-run wider evidence when shared infrastructure changed.
10. Do not call the defect closed merely because a later screenshot looks correct.

## Repository rules during execution

* Start by reading branch, HEAD, remote HEAD/divergence, dirty/staged/untracked state.
* Preserve existing and user-created work.
* Do not reset, clean, stash, rebase, or force push as a shortcut.
* Stage only files belonging to the validated slice.
* Run `git diff --cached --check` before commit.
* Keep generated validation output, run data, local credentials, reports, screenshots, coverage, browser artifacts, and runtime ownership state out of source control unless a specific small public artifact is intentionally approved.
* Preserve the upstream `examples/` content and `LICENSE` unless a separately justified change is explicitly required.
* Do not push/merge merely because a local task completed; follow the current publication authority.

## Resource rules during execution

* Check output-volume free space before Full/live/evidence-heavy runs.
* Create Solari browser/sandbox resources only after cheaper deterministic prerequisites pass.
* Register external resource ownership immediately.
* Close/kill only resources that can be proven owned by the current operation.
* Never kill another local process just because it occupies the preferred port.
* Full validation fails when required owned-resource cleanup is unresolved.

## Public truth rules

* Documentation describes current implemented behavior, not planned architecture as if already shipped.
* Do not claim arbitrary autonomous bug discovery in version one unless a runtime planner was actually implemented and validated.
* Do not claim formal accessibility/security compliance without a scoped audit supporting it.
* Do not claim screenshots are automatically privacy-safe.
* Do not claim replay download when the installed TypeScript SDK path has not been proven.
* Do not claim a capability Available while ordinary end-to-end, persistence/lifecycle, failure, cleanup, and required validation remain incomplete.

## Completion sequence

The complete project sequence is intentionally:

```text
implement front to back
-> close horizontal/vertical integration
-> run Full
-> deliberately search for bugs/disconnected features
-> repair with regressions
-> harden demonstrated risk boundaries
-> run fresh Full
-> perform human visual/usability QA
-> repair escaped defects + harness gaps
-> run affected checks and final fresh Full
-> audit public docs/repository
-> prepare publication
-> obtain final owner authority for public posting/integration
```

Any executor resuming this project should report the exact current milestone, last current validation result, dirty files, known blockers, live resource state, human QA state, and exact next safe task before continuing.