# Repository Execution Guide

This repository is a public fork of the Solari cookbook. Preserve the upstream cookbook and implement ReproDocket as the first-class project under `reprodocket/`.

## Required reading order

Before changing ReproDocket implementation, read:

1. `docs/README.md`
2. `docs/reprodocket-design.md`
3. `docs/reprodocket-interface-contracts.md`
4. `docs/reprodocket-data-handling.md`
5. `docs/reprodocket-security-lifecycle.md`
6. `docs/reprodocket-toolchain-baseline.md`
7. `docs/reprodocket-sdk-baseline.md`
8. `docs/reprodocket-fixture-spec.md`
9. `docs/reprodocket-ui-spec.md`
10. `docs/reprodocket-test-matrix.md`
11. `docs/implementation/00-contract-reconciliation.md`
12. `docs/implementation/README.md`
13. the currently active numbered implementation plan

If any active plan conflicts with the newer authoritative contracts, the reconciliation document controls the known transition and the documentation must be reconciled before new incompatible code is written.

## Scope and preservation

* Product implementation belongs under `reprodocket/`.
* Architecture/planning documents belong under `docs/`.
* Preserve upstream `examples/` and the root `LICENSE` unless an independently justified change is explicitly authorized.
* Keep generated output, runtime state, local runs, credentials, test reports, large evidence, and dependency directories out of source control.
* `reprodocket/package-lock.json` is intentionally tracked.

## Product truth

Version one is an evidence-first browser defect reproduction and independent verification tool.

A normal run requires:

```text
public HTTP/HTTPS target URL
human-readable problem description
auditable plan containing at least one browser action and one observation expectation
```

The source plan is persisted with the run. Version-one plan values must therefore be nonsecret test data. Do not implement or claim authenticated secret injection until a separate secret-reference design exists.

Production investigation and verification use real Solari cloud browsers. The deterministic Full harness uses a real Solari sandbox-hosted fixture. Do not add a silent local-browser production fallback.

## Plan grammar

Use the unified `PlanStatement` / `ParsedPlanStatement` model and `parsePlanStatements()` from the current contracts.

Actions:

```text
OPEN CLICK FILL SELECT CHECK UNCHECK PRESS WAIT_FOR_TEXT WAIT_FOR_URL RELOAD BACK FORWARD
```

Expectations:

```text
EXPECT_TEXT EXPECT_NO_TEXT EXPECT_URL EXPECT_PAGE_ERROR EXPECT_MAIN_STATUS
```

Do not implement the older transitional `ParsedReproductionStep` model.

Do not add arbitrary JavaScript, CSS/XPath selectors, shell execution, local filesystem reads, or generic computer-control commands to the user plan language.

## Outcome truth

Lifecycle:

```text
CREATED PREPARING INVESTIGATING VERIFYING FINALIZING COMPLETED FAILED CANCELLED INTERRUPTED
```

Final outcome:

```text
VERIFIED REPRODUCED NOT_REPRODUCED INCONCLUSIVE
```

Lifecycle and defect outcome are separate authorities.

Never map infrastructure failure to `NOT_REPRODUCED`.

Never return `VERIFIED` unless investigation and verification both contain sufficient matching defect evidence and use distinct non-null Solari session IDs.

A console error, warning, failed subrequest, or successful action sequence is evidence, not automatic proof of the reported defect.

## Test-first execution

Every behavior-changing task follows:

```text
identify authoritative boundary
-> write smallest real failing regression/acceptance test
-> run and observe expected failure
-> implement the bounded behavior
-> run and observe pass
-> run adjacent/static checks
-> perform required mutation/sensitivity proof for critical guards
-> inspect diff/repository state
-> commit only the related validated slice when policy permits
```

Do not claim a test, build, bug fix, milestone, or feature passes without fresh command output from the current source state.

Mocks/doubles may isolate local behavior but never satisfy live Solari or complete end-to-end authority.

The fixture is an oracle, not a production special case. Production code must not branch on fixture routes/scenario IDs to manufacture expected results.

## Implementation order

After the reconciliation gate, execute plans 1 through 6 in order:

```text
foundation/local shell
-> evidence/persistence/local UI
-> live Solari browser/sandbox substrate
-> investigation + fresh independent verification
-> deliberate product-wide bug finding + hardening
-> fresh Full validation + human QA
-> publication preparation
```

Do not jump to showcase work because one happy path works.

## Validation authority

`TARGETED` is the smallest sufficient current evidence for a bounded change.

`FULL` is required for broad milestone, hardening, final completion, and publication claims and includes applicable live Solari authorities.

A required `BLOCKED` is not a PASS.

Human visual/usability QA remains separate from automated Full validation.

When available, plain:

```powershell
.\reprodocket\scripts\validate.ps1
```

means Full.

## Horizontal and vertical completeness

A visible feature is not complete from one layer alone.

Vertical proof traverses the normal user path through request validation, coordinator state, real Solari execution, evidence, persistence, independent verification, classification, UI/history/reload, and resource cleanup.

Horizontal proof requires sibling surfaces describing the same run to agree: active state, history, detail, report, evidence lists, replay, cleanup, provider state, validation provenance, and public capability wording.

Every visible control must perform real behavior, be truthfully disabled with a current reason, or be absent.

A shared storage/router/serializer/state/provider/lifecycle defect triggers adjacent-consumer review.

## Security and privacy

Follow `docs/reprodocket-data-handling.md` and `docs/reprodocket-security-lifecycle.md`.

Non-negotiable boundaries:

* normal server binds to loopback only;
* mutation APIs enforce the local same-origin/request-nonce boundary;
* target policy accepts only allowed public HTTP/HTTPS destinations and blocks prohibited local/private/metadata destinations to the strongest enforceable boundary;
* target/user content is untrusted data, never executable product markup;
* do not use `dangerouslySetInnerHTML` for target/run data;
* static reports escape every untrusted value;
* do not persist Authorization/Cookie/Set-Cookie headers, unrestricted bodies, browser-storage dumps, or the Solari API key;
* central redaction occurs before durable provider-derived text is written;
* user-authored plan text is intentionally persisted and must be nonsecret test data;
* artifact serving requires validated run ownership and artifact identity;
* protect the normal Windows Solari credential with the current-user OS boundary;
* do not overclaim screenshot/privacy or SSRF guarantees beyond tested scope.

## Resource ownership

Register every created Solari browser/sandbox immediately. Release owned resources in structured cleanup and record cleanup truth.

Full validation fails if a required owned resource remains unresolved.

Never kill or mutate an unrelated local/provider resource because it appears old or occupies a preferred port. PID equality alone is not process ownership.

Retries are bounded and operation-aware. Do not blindly retry unknown target mutations or capacity failures.

## Toolchain and Windows

Minimum Node compatibility floor while Vite 8 is selected:

```text
Node.js 22.12.0
```

A fresh bootstrap should prefer the current supported Node LTS discovered at execution time. Do not force-upgrade an already compatible runtime that passes the actual toolchain.

PowerShell 7 is the substantive Windows scripting target. A small Windows PowerShell 5.1-compatible bootstrap may discover/install/reroute to PowerShell 7.

Do not use WMIC or `wmic.exe`.

Use `$PSScriptRoot` for script-relative paths. Do not default scratch/generated output to Desktop, Documents, or Downloads. Check free space before evidence-heavy/live Full runs.

## SDK authority

Do not assume historical cookbook signatures are current.

Use this order for Solari SDK details:

1. installed package type declarations and live observed behavior;
2. current Solari documentation;
3. current package-registry metadata;
4. current cookbook examples;
5. historical assumptions.

If installed behavior materially contradicts the SDK baseline, update the baseline and affected contract/plan before building on the new behavior.

## Git safety

Before edits inspect:

```powershell
git status --short
git branch --show-current
git rev-parse HEAD
git rev-list --left-right --count main...HEAD
```

Preserve existing staged, unstaged, untracked, ignored, and user-created work.

Do not use destructive reset/clean/stash/rebase/force-push shortcuts.

Stage only related files. Before commit run:

```powershell
git diff --cached --check
```

Do not push, merge, or publish externally without current authority.

## Documentation truth

Public docs describe implemented and validated behavior, not architecture as if already shipped.

Do not claim arbitrary autonomous discovery, formal accessibility/security compliance, universal screenshot privacy, universal SSRF prevention, replay download without proven support, or Available capability without its complete ordinary workflow/lifecycle/failure/validation evidence.

Detailed implementation plans are development artifacts. The final outside-reader audit explicitly decides whether retaining them improves the public release.

## Stop conditions

Stop broad feature expansion and fix/reconcile the current boundary when:

* a visible ordinary workflow is broken or disconnected;
* a critical harness test cannot fail for its seeded defect;
* evidence identity/provenance is uncertain;
* a secret may have leaked;
* repository state is unknown and writing could damage unrelated work;
* external-resource ownership is uncertain;
* a shared infrastructure defect makes adjacent results untrustworthy;
* required validation fails;
* a completion claim would rely on stale evidence.

Preserve the failure and remaining uncertainty rather than converting it into success.
