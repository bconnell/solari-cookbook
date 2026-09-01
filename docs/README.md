# ReproDocket Engineering Documentation

ReproDocket is currently in **pre-implementation design and planning** on `feature/reprodocket`.

These documents describe the intended supported scope, contracts, validation requirements, and execution plan. They are not evidence that the corresponding product capability has already shipped.

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
3. [`reprodocket-data-handling.md`](reprodocket-data-handling.md) — persisted data, secret boundary, screenshot/replay privacy limits and nonsecret-plan requirement.
4. [`reprodocket-security-lifecycle.md`](reprodocket-security-lifecycle.md) — loopback security, target safety, evidence boundaries, resource ownership, retries and recovery.
5. [`reprodocket-sdk-baseline.md`](reprodocket-sdk-baseline.md) — current Solari documentation assumptions and execution-time probes.
6. [`reprodocket-fixture-spec.md`](reprodocket-fixture-spec.md) — independent deterministic fixture ground truth.
7. [`reprodocket-ui-spec.md`](reprodocket-ui-spec.md) — local interface behavior, states, copy, accessibility baseline and visual evidence requirements.
8. [`reprodocket-test-matrix.md`](reprodocket-test-matrix.md) — minimum deterministic, live, end-to-end, connectivity, lifecycle and harness-sensitivity coverage.

## Implementation plans

[`implementation/`](implementation/) contains development execution plans. They are deliberately more procedural than the public product design and may include implementation sequencing that is irrelevant to normal users.

Before implementation begins, [`implementation/00-contract-reconciliation.md`](implementation/00-contract-reconciliation.md) must be read because it records final naming/contract corrections discovered during the planning audit.

The numbered plans then execute in order:

```text
01 foundation and local shell
02 evidence, persistence and local results UI
03 real Solari browser and sandbox substrate
04 investigation, independent verification and complete vertical path
05 bug discovery, hardening and final proof
06 publication and challenge submission preparation
```

## Documentation authority

When documentation disagrees, use the newest explicit contract for the affected behavior. During actual Solari integration, installed package types and observed live behavior are stronger authority for SDK signatures than historical example code.

Material implementation-time contract changes must be reflected back into the applicable design/contract document rather than surviving only in code or terminal history.

## Public release policy

The final release audit decides which development-only implementation-plan artifacts improve the public repository. They are not automatically retained merely because they were useful during construction.

The final public README, walkthrough, validation summary, product UI and source must describe the same actually proven capability boundary.

The upstream Solari cookbook examples and MIT license remain preserved.
