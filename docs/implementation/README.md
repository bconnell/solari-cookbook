# ReproDocket Implementation Index

Date: August 31, 2026
Status: Pre-implementation execution map

This directory contains the ordered implementation plans for ReproDocket. Each phase must end in working, testable software with an explicit gate before the next phase begins.

Detailed plans are development artifacts. They describe what must be implemented and proven; they are not evidence that a capability already exists.

## Mandatory read order

Before executable work:

1. [`../README.md`](../README.md) — documentation status and maturity.
2. [`../reprodocket-design.md`](../reprodocket-design.md) — product scope and architecture.
3. [`../reprodocket-interface-contracts.md`](../reprodocket-interface-contracts.md) — plan grammar, run models and HTTP/API contracts.
4. [`../reprodocket-data-handling.md`](../reprodocket-data-handling.md) — persisted plan-data and privacy boundary.
5. [`../reprodocket-security-lifecycle.md`](../reprodocket-security-lifecycle.md) — trust, target safety, ownership, retries and recovery.
6. [`../reprodocket-toolchain-baseline.md`](../reprodocket-toolchain-baseline.md) — Node/PowerShell/package compatibility policy.
7. [`../reprodocket-sdk-baseline.md`](../reprodocket-sdk-baseline.md) — current Solari assumptions and execution-time probes.
8. [`../reprodocket-fixture-spec.md`](../reprodocket-fixture-spec.md) — deterministic validation oracle.
9. [`../reprodocket-ui-spec.md`](../reprodocket-ui-spec.md) — local user-boundary behavior and visual contract.
10. [`../reprodocket-test-matrix.md`](../reprodocket-test-matrix.md) — minimum acceptance and harness-sensitivity catalog.
11. [`00-contract-reconciliation.md`](00-contract-reconciliation.md) — mandatory corrections to transitional wording in earlier numbered plans.
12. This index.
13. The active numbered plan.

Current source, installed SDK type declarations and live observed behavior outrank stale SDK/example assumptions. Material contract changes must be reconciled back into documentation before incompatible code is built around them.

## Gate 0 — contract reconciliation

[`00-contract-reconciliation.md`](00-contract-reconciliation.md)

Before Plan 1, confirm:

```text
plan is required
unified PlanStatement/ParsedPlanStatement contract is used
parsePlanStatements() is the parser entry point
action + expectation grammar is already final for version one
user-authored plan text is persisted and must be nonsecret test data
Node compatibility floor is at least 22.12.0 while Vite 8 is selected
fresh install prefers current supported Node LTS
fixture spec is the independent ground truth
UI spec is the user-boundary contract
```

No product source is written until this gate is reconciled against the current branch state.

## Plan 1 — Foundation and local shell

[`01-foundation-local-shell.md`](01-foundation-local-shell.md)

Builds/proves:

```text
reproducible Node/TypeScript project
real red-green unit harness
unified plan parser and request validation
safe public-target URL policy
versioned run model and lifecycle invariants
initial filesystem run store
secure loopback Fastify shell
built React application shell
owned instance/port handling
Windows bootstrap/preflight/run/stop
protected Solari credential storage
provider readiness UI
first validation entry point
```

Use the names and runtime floor from Gate 0 rather than the transitional parser/Node wording still present in portions of the original Plan 1 text.

Exit gate: M0/M1 foundation is executable through the actual local app boundary and Targeted validation is current. Full remains truthfully incomplete because live/product authorities do not all exist yet.

## Plan 2 — Evidence, persistence and local results

[`02-evidence-persistence-ui.md`](02-evidence-persistence-ui.md)

Builds/proves:

```text
central provider-derived text redaction
run-owned artifact storage
SHA256 final evidence sealing/integrity
structured console/page/network/timeline evidence
safe static HTML and JSON reports
history/detail/artifact APIs
SSE progress with GET rehydration authority
History and Run Detail UI
interrupted-run recovery
horizontal consistency checks
initial connectivity matrix
```

User-authored source plan text is intentionally persisted; the plan itself must use nonsecret test data. Redaction is not represented as a magic arbitrary-secret detector.

Exit gate: M2 evidence subsystem is complete for its declared local scope and all implemented views agree on authoritative run state.

## Plan 3 — Solari browser and sandbox substrate

[`03-solari-browser-sandbox.md`](03-solari-browser-sandbox.md)

Builds/proves:

```text
installed Solari Browser SDK contract
real recorded browser create/use/close
actual Node client shutdown behavior
Solari resource ledger
bounded provider retry policy
page/console/network/screenshot evidence bridge
replay readiness and TypeScript replay capability probe
installed Solari Sandbox SDK contract
versioned deterministic fixture implementation
Solari-hosted fixture preview
external fixture identity probe
live validation stages
```

Fixture behavior must match [`../reprodocket-fixture-spec.md`](../reprodocket-fixture-spec.md), not a convenient implementation that makes ReproDocket pass.

Exit gate: M3/M4 real Solari substrate is current, resource-owned and directly exercised.

## Plan 4 — Investigation, verification and complete vertical path

[`04-investigation-verification-e2e.md`](04-investigation-verification-e2e.md)

Builds/proves:

```text
deterministic accessible action execution
finalized expectation evaluation
AttemptEngine
atomic one-active-run coordinator
fresh independent verification session
create/cancel APIs
New Investigation UI with action + expectation plan editor
cancellation lifecycle
fixture plans for all four result classes
real Solari E2E through the built local UI
horizontal and vertical connectivity closure
```

Expectation grammar is already fixed by the interface contract. Transitional Plan 4 wording that says to add/design expectations later is superseded by Gate 0.

Exit gate: M5-M7 front-to-back product is functionally complete and has a current Full machine-validation result. This is not publication readiness.

## Plan 5 — Bug finding, hardening and final proof

[`05-bug-finding-hardening-final-proof.md`](05-bug-finding-hardening-final-proof.md)

Audits/hardens:

```text
complete visible-feature inventory
horizontal and vertical connection matrix
failure injection
remote target/redirect safety
loopback/request/artifact security
untrusted-content rendering
Solari-key and known-secret leakage
resource lifetime, retries and shutdown
persistence corruption/interruption
fresh Windows setup
accessibility and native-scale readability
harness mutation sensitivity
dependency/public-source hygiene
post-hardening Full validation
human visual/usability QA
escaped-defect regression feedback
final fresh Full validation
```

Exit gate: M8-M10 has no known ordinary-scope hardening blocker, human QA is explicitly recorded, and final Full evidence is fresh after all fixes.

## Plan 6 — Publication and challenge submission

[`06-publication-challenge-submission.md`](06-publication-challenge-submission.md)

Produces:

```text
truthful ReproDocket README
restrained root cookbook README entry
safe reproducible demo material
short technical walkthrough
fresh post-documentation Full validation
outside-reader GitHub audit
challenge post draft/demo plan
final integration/PR steps
```

The outside-reader audit explicitly decides whether detailed implementation-plan/reconciliation files and `AGENTS.md` improve the final public repository. They are development artifacts, not automatically permanent product documentation.

Exit gate: M11 is ready for the final repository/social-publication decision. Public posting and integration remain explicit owner-authority boundaries.

## Milestone dependency chain

```text
Gate 0 Contract reconciliation
  -> M0 Foundation
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

Do not skip from the live Solari substrate to a showcase, or from functional vertical closure to publication. Deliberate bug discovery, hardening and revalidation are part of completion.

## Validation vocabulary

`TARGETED`: smallest sufficient current real checks for a bounded change.

`FULL`: complete current automated/machine authority required for broad milestone/release claims, including live Solari when applicable.

`BLOCKED`: a required authority could not be exercised. It is not PASS.

Human visual/usability acceptance is separate. Machine Full PASS does not imply human QA PASS.

When implemented, plain:

```powershell
.\reprodocket\scripts\validate.ps1
```

means Full.

## Test-first task rule

Every behavior-changing task follows:

```text
identify authoritative boundary
-> write failing regression/acceptance test
-> run and observe expected failure
-> implement smallest complete behavior
-> run and observe pass
-> run adjacent/static checks
-> prove mutation/sensitivity where the guard is critical
-> inspect repository/diff
-> commit only the validated related slice when policy permits
```

Live external behavior may first use a test double for local lifecycle behavior, but the relevant real Solari authority still has to run before the capability is promoted.

## Bug repair rule

For every discovered product defect:

1. Preserve the failing evidence.
2. Identify the earliest responsible production boundary.
3. Add or strengthen the test that should detect it.
4. Prove that test fails for the defect when safely possible.
5. Repair the responsible boundary.
6. Prove the targeted test passes.
7. Check sibling consumers of the same state/store/router/adapter/lifecycle boundary.
8. Run the affected vertical path.
9. Invalidate and rerun wider evidence when shared infrastructure changed.
10. Do not close the defect only because a later screenshot appears correct.

## Repository and resource rules

Before editing, record branch, HEAD, remote/divergence and dirty/staged/untracked state. Preserve user work. Do not use destructive reset, clean, stash, rebase or force push as shortcuts. Stage only related files and run `git diff --cached --check` before commit.

Generated output, local runs, credentials, reports, coverage and runtime state remain untracked unless a small public artifact is deliberately approved.

Check free space before evidence-heavy/live Full work. Create billable Solari resources only after cheaper deterministic prerequisites pass. Register resource ownership immediately and release only resources proven owned by the current operation. Cleanup uncertainty blocks a clean Full lifecycle claim.

## Public truth rules

Until implementation evidence exists, ReproDocket remains Planned/pre-implementation.

Do not claim:

```text
arbitrary autonomous bug discovery
formal accessibility/security compliance
automatic screenshot privacy
universal SSRF prevention
local replay download before the TypeScript path is proven
support for secret-bearing/authenticated plans
Available capability before complete workflow/lifecycle/failure/validation proof
```

## Complete project sequence

```text
pre-implementation contract reconciliation
-> front-to-back implementation
-> horizontal/vertical integration closure
-> Full
-> deliberate bug/disconnected-feature search
-> regression-backed repairs
-> proportional hardening
-> fresh Full
-> human visual/usability QA
-> escaped-defect + harness repair
-> affected checks + final fresh Full
-> public docs/repository audit
-> publication preparation
-> final owner authority for integration/posting
```

A resuming executor reports the exact current milestone, last current validation result, dirty files, known blockers, live-resource state, human-QA state and exact next safe task before continuing.
