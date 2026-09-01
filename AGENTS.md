# Repository Execution Guide

This repository is a public fork of the Solari cookbook. Preserve the upstream cookbook and build ReproDocket as the new first-class project under `reprodocket/`.

## Scope

Unless a task explicitly says otherwise:

* ReproDocket implementation belongs under `reprodocket/`.
* ReproDocket architecture and planning documents belong under `docs/`.
* Existing upstream examples under `examples/` are reference material and should remain unchanged.
* The root `LICENSE` must remain preserved.

## Read before implementing ReproDocket

Read in this order:

1. `docs/reprodocket-design.md`
2. `docs/reprodocket-interface-contracts.md`
3. `docs/reprodocket-security-lifecycle.md`
4. `docs/reprodocket-sdk-baseline.md`
5. `docs/reprodocket-test-matrix.md`
6. `docs/implementation/00-contract-reconciliation.md`
7. `docs/implementation/README.md`
8. The currently active numbered implementation plan.

If plan prose conflicts with a newer contract/design document, stop and reconcile the documentation before implementing the conflicting behavior. The mandatory pre-implementation reconciliation file resolves known transitional names and stale assumptions in the numbered plans.

## Required implementation order

Complete `docs/implementation/00-contract-reconciliation.md`, then implement numbered plans 1 through 6 in order.

Do not skip directly to demo/publication work because an intermediate happy path works.

The intended sequence is:

```text
contract reconciliation
-> foundation/local shell
-> local evidence/persistence/UI
-> real Solari browser/sandbox substrate
-> full investigation + fresh verification
-> product-wide bug finding/hardening
-> fresh Full validation + human QA
-> publication preparation
```

## Current product truth

Version one is an evidence-first browser defect reproduction and verification tool.

A normal run accepts:

```text
public HTTP/HTTPS target URL
problem description
auditable reproduction + observation plan
```

The plan is required and must contain at least one browser action and at least one observation expectation.

It then:

```text
creates real recorded Solari browser A
executes the plan
captures/persists evidence
closes browser A
creates fresh recorded Solari browser B
executes the same validated plan
captures/persists independent evidence
classifies the combined result
shows the result locally
```

Solari is not a decorative integration. Production investigation/verification uses real Solari cloud browsers. Full deterministic end-to-end validation uses a real Solari sandbox-hosted fixture.

Do not add a silent local-browser fallback for production investigations.

## Version-one outcome truth

Lifecycle and defect outcome are separate.

Lifecycle:

```text
CREATED
PREPARING
INVESTIGATING
VERIFYING
FINALIZING
COMPLETED
FAILED
CANCELLED
INTERRUPTED
```

Final outcome:

```text
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
```

Never map an infrastructure failure to `NOT_REPRODUCED`.

Never return `VERIFIED` unless both independent attempts contain sufficient reproduction evidence and use distinct non-null Solari session IDs.

A console error, warning, failed subrequest, or successful action sequence is evidence, not automatic proof of the reported defect.

## Version-one plan grammar

Implement only the grammar defined in `docs/reprodocket-interface-contracts.md`.

The first parser implementation uses the unified `PlanStatement` / `ParsedPlanStatement` model and `parsePlanStatements()`. Do not create a competing `ParsedReproductionStep` model from older plan prose.

Do not add arbitrary JavaScript, CSS/XPath selectors, shell execution, local filesystem reads, or generic computer-control commands to the user plan language.

A future AI planner is outside the initial release unless separately designed and validated. Do not introduce a second mandatory AI/API credential to make the current plan work.

## Test-first rule

For every behavior-changing implementation task:

1. Identify the authoritative boundary.
2. Write the smallest real regression/acceptance test that should fail without the behavior.
3. Run it and confirm the expected failure.
4. Implement the complete bounded behavior.
5. Run the affected test and confirm pass.
6. Run adjacent/static checks appropriate to the changed boundary.
7. For critical guards, deliberately prove test sensitivity as specified by the plan.
8. Review the diff and repository state.
9. Commit only the related validated slice when repository policy permits.

Do not claim a test, build, bug fix, or milestone passes without fresh command output from the current source state.

## Validation authority

`TARGETED` validation is allowed for a bounded change.

`FULL` validation is required for broad milestone, hardening, final completion, and publication claims.

A mandatory `BLOCKED` is not a PASS.

Human visual/usability QA is a separate authority from automated Full validation.

When the validation scripts exist, plain:

```powershell
.\reprodocket\scripts\validate.ps1
```

means Full.

## Harness quality

Tests must exercise behavior rather than merely inspect source or assert that a function can be called.

Critical tests must be capable of failing for the defect they guard. Follow the mutation/sensitivity catalog in `docs/reprodocket-test-matrix.md` and the hardening plan.

Mocks/test doubles may isolate unit/local lifecycle behavior, but they cannot satisfy live Solari or end-to-end proof.

The deterministic fixture is an oracle, not a production special case. Production code must not branch on fixture route names/scenario IDs to manufacture expected results.

## Horizontal and vertical completeness

Do not call a visible feature complete from one layer alone.

Vertical proof follows the ordinary product path from user input through validation, state, Solari, evidence, persistence, verification, classification, UI/history/reload, and cleanup.

Horizontal proof requires sibling surfaces describing the same run to agree: active state, history, detail, report, evidence lists, replay, cleanup, provider state, validation provenance, and public capability wording.

If a shared store/router/serializer/state/adapter/lifecycle defect is found, inspect adjacent consumers sharing that boundary.

Every visible button/link/tab/control must perform real behavior, be truthfully disabled with a current reason, or be absent.

## Security and privacy

Follow `docs/reprodocket-security-lifecycle.md`.

Non-negotiable initial boundaries:

* Bind the normal local server to loopback only.
* Protect state-changing localhost APIs from foreign origins and invalid local request nonces.
* Accept only allowed public HTTP/HTTPS targets and reject prohibited local/private/metadata destinations to the strongest enforceable boundary.
* Treat target/user text as untrusted data.
* Do not render target/run data with `dangerouslySetInnerHTML`.
* Escape all untrusted values in static HTML reports.
* Do not persist Authorization/Cookie headers, unrestricted bodies, passwords, Solari keys, or browser-storage dumps.
* Redact before durable persistence, not only at display time.
* Serve artifacts only by validated run ownership and artifact identity.
* Protect the normal Windows credential through the current-user OS boundary; never put it in `.env`, command arguments, logs, manifests, or tracked source.
* Do not overclaim security beyond tested scope.

## Resource ownership

Every created Solari browser/sandbox must be registered immediately and released in structured cleanup.

Full validation fails if a required owned resource remains unresolved.

Never kill or mutate an unrelated local/provider resource merely because it appears old or occupies a desired port.

PID equality alone is not enough to prove local-process ownership.

Retries must be bounded and safe for the operation. Do not blindly retry unknown target mutations or provider capacity failures.

## Windows automation

PowerShell 7 is the substantive scripting target. A tiny bootstrap entry may remain Windows PowerShell 5.1 compatible so it can discover/install/launch PowerShell 7.

Do not use WMIC or `wmic.exe`.

Use `$PSScriptRoot` for script-relative paths.

Do not use Desktop, Documents, or Downloads as scratch/default generated-output locations.

Check output-volume free space before evidence-heavy/live Full runs.

## Git safety

Before edits, inspect:

```powershell
git status --short
git branch --show-current
git rev-parse HEAD
git rev-list --left-right --count main...HEAD
```

Preserve all existing staged, unstaged, untracked, ignored, and user-created work.

Do not use destructive reset/clean/stash/rebase/force-push shortcuts.

Stage only related files.

Before commit:

```powershell
git diff --cached --check
```

Do not push, merge, or publish externally unless the current task/user authority explicitly permits it.

## Generated output

Keep these out of source control unless a specific public artifact is intentionally approved:

```text
node_modules
dist
coverage
test-results
playwright-report
.artifacts
.runtime
.tmp
local run data
local secrets
large screenshots/replays/logs
```

`reprodocket/package-lock.json` is intentionally tracked.

## SDK authority

Do not assume the cookbook's historical SDK signatures are current.

For Solari integration use this authority order:

1. Installed package type declarations and live observed behavior.
2. Current Solari documentation.
3. Current package registry metadata.
4. Existing cookbook examples.
5. Historical assumptions.

If the installed SDK contradicts `docs/reprodocket-sdk-baseline.md`, update the baseline and affected contract/plan before building around the new behavior.

## Documentation truth

Public docs must describe current supported behavior and limitations.

Do not claim:

* arbitrary autonomous bug discovery unless an AI planner is actually shipped and validated,
* formal accessibility/security compliance without a scoped audit,
* replay download if only replay URL access is proven,
* screenshots are automatically privacy-safe,
* universal SSRF prevention when provider/DNS visibility limits the enforceable boundary,
* a feature is Available when its ordinary end-to-end/lifecycle/failure/validation work is incomplete.

Detailed implementation plans are development artifacts rather than capability evidence. Before final publication, explicitly decide whether they improve the public repository; do not let internal execution material become part of the final presentation by accident.

## Stop conditions

Stop broad feature expansion and address the current blocker if:

* a visible ordinary workflow is broken/disconnected,
* a critical test cannot fail for its seeded defect,
* evidence identity/provenance is uncertain,
* a secret may have leaked,
* repository state is unknown and writing risks unrelated work,
* external-resource ownership is uncertain,
* a shared infrastructure defect makes adjacent results untrustworthy,
* required validation fails,
* a completion claim would rely on stale evidence.

Preserve failures and report exact remaining uncertainty instead of converting it into success.
