# ReproDocket Engineering Documentation

ReproDocket is currently in **pre-implementation design and planning** on `feature/reprodocket`.

These documents define intended supported scope, contracts, validation requirements and execution order. They are not evidence that the corresponding product capability has already shipped.

## Current maturity

```text
Product: Planned / pre-implementation
Executable ReproDocket application: not yet implemented
Live Solari product validation: not yet run
Human UI acceptance: not yet run
Public release: not yet prepared
```

Capability status changes only from current implementation and validation evidence. A design section, interface declaration, test case, or implementation-plan checkbox does not make a feature Available.

## Authoritative product documents

Read these first:

1. [`reprodocket-design.md`](reprodocket-design.md) — supported scope, architecture, lifecycle, evidence, integration and completion model.
2. [`reprodocket-interface-contracts.md`](reprodocket-interface-contracts.md) — plan grammar, TypeScript-facing models, API contracts and result semantics.
3. [`reprodocket-data-handling.md`](reprodocket-data-handling.md) — persisted run data, provider-secret boundary, screenshot/replay privacy limits and nonsecret-plan requirement.
4. [`reprodocket-security-lifecycle.md`](reprodocket-security-lifecycle.md) — loopback security, target safety, evidence boundaries, resource ownership, retries and recovery.
5. [`reprodocket-toolchain-baseline.md`](reprodocket-toolchain-baseline.md) — Node, PowerShell and package-selection compatibility policy.
6. [`reprodocket-sdk-baseline.md`](reprodocket-sdk-baseline.md) — current Solari documentation assumptions and execution-time probes.
7. [`reprodocket-fixture-spec.md`](reprodocket-fixture-spec.md) — independent deterministic fixture ground truth for all four product outcomes.
8. [`reprodocket-ui-spec.md`](reprodocket-ui-spec.md) — local interface behavior, states, copy, accessibility baseline and visual evidence requirements.
9. [`reprodocket-test-matrix.md`](reprodocket-test-matrix.md) — minimum deterministic, live, end-to-end, connectivity, lifecycle and harness-sensitivity coverage.

## Execution and handoff documents

[`implementation/`](implementation/) contains detailed development execution plans. They are deliberately more procedural than normal product documentation and may include implementation sequencing irrelevant to ordinary users.

Before executable work begins, [`implementation/00-contract-reconciliation.md`](implementation/00-contract-reconciliation.md) is mandatory because it records final naming, data-handling, toolchain, fixture and UI corrections discovered during the preimplementation audit.

[`pre-codex-readiness.md`](pre-codex-readiness.md) records the exact boundary between preparation already completed and runtime facts that still require the actual Windows/Solari environment.

The numbered implementation phases then execute in order:

```text
01 foundation and local shell
02 evidence, persistence and local results UI
03 real Solari browser and sandbox substrate
04 investigation, independent verification and complete vertical path
05 bug discovery, hardening and final proof
06 publication and challenge submission preparation
```

## Publication planning

[`reprodocket-release-surface-policy.md`](reprodocket-release-surface-policy.md) defines which development artifacts require an explicit KEEP / CONDENSE-MOVE / REMOVE-FROM-RELEASE decision before final publication.

[`reprodocket-demo-acceptance.md`](reprodocket-demo-acceptance.md) defines what the eventual public demonstration must prove from the real current product without turning the demo into a substitute for Full validation.

The root cookbook README is intentionally unchanged during preimplementation. It should surface ReproDocket only after the product is genuinely usable and the wording can be derived from current runtime truth.

## Documentation authority

When documentation disagrees, use the current explicit contract for the affected behavior and the mandatory reconciliation file for already-known transitional plan wording. During Solari integration, installed package type declarations and observed live behavior are stronger authority for SDK signatures than historical example code.

Material implementation-time contract changes must be reflected back into the applicable design/contract document rather than surviving only in source or terminal history.

## Public release policy

The final release audit decides which development-only planning and executor artifacts improve the public repository. They are not automatically retained merely because they were useful during construction.

The final public README, source, walkthrough, validation summary, product UI and demo must describe the same actually proven capability boundary.

The upstream Solari cookbook examples and MIT license remain preserved.
