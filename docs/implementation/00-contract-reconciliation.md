# ReproDocket Pre-Implementation Contract Reconciliation

Date: August 31, 2026
Status: Mandatory pre-implementation gate

This document resolves specification drift discovered during the final pre-implementation audit. It is not a new product design. It aligns the existing implementation plans with the current authoritative design and interface contracts before executable work begins.

## Authority

Use this order when implementation documents disagree:

1. `docs/reprodocket-design.md`
2. `docs/reprodocket-interface-contracts.md`
3. `docs/reprodocket-security-lifecycle.md`
4. `docs/reprodocket-sdk-baseline.md`
5. `docs/reprodocket-test-matrix.md`
6. this reconciliation document
7. numbered implementation plans

Installed Solari package type declarations and live observed behavior remain the execution-time authority for SDK signatures. If they materially contradict the design, update the affected documentation before implementing around the contradiction.

## Resolved version-one request contract

A valid normal run requires all of the following:

```text
public HTTP/HTTPS target URL
human-readable problem description
auditable plan with at least one browser action
auditable plan with at least one observation expectation
```

The plan is required. It is not optional.

The exact `CreateRunRequest` remains:

```ts
export interface CreateRunRequest {
  targetUrl: string;
  problem: string;
  plan: string[];
}
```

Validation remains:

```text
targetUrl required; maximum 2048 UTF-16 code units before normalization
problem trimmed length 1..10000
plan 2..100 nonblank statements
statement trimmed length 1..1000
at least one ReproductionAction
at least one ObservationExpectation
```

## Resolved parser contract

Plan 1 was drafted before observation expectations were finalized and therefore uses transitional names such as `ReproductionStepParser`, `parseReproductionSteps`, and `ParsedReproductionStep` in its first task.

Those names are superseded before implementation.

Use:

```text
src/shared/contracts/PlanModels.ts
src/core/PlanParser.ts
tests/unit/PlanParser.test.ts
```

with:

```ts
parsePlanStatements(lines: string[]): ParsedPlanStatement[]
```

and the `ReproductionAction`, `ObservationExpectation`, `PlanStatement`, and `ParsedPlanStatement` contracts already defined in `docs/reprodocket-interface-contracts.md`.

The parser recognizes both action and expectation statements from the first implementation slice. Request-level validation, not the single-line parser, enforces that a complete submitted plan contains at least one action and at least one expectation.

Do not create a second incompatible `ParsedReproductionStep` model.

## Resolved initial grammar

Actions:

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

Expectations:

```text
EXPECT_TEXT
EXPECT_NO_TEXT
EXPECT_URL
EXPECT_PAGE_ERROR
EXPECT_MAIN_STATUS
```

This grammar is already final for the version-one implementation baseline. Plan 4 must not treat observation expectations as a future optional extension.

## Corrections to Plan 1

Where Plan 1 Task 1 says to create `ReproductionAction.ts`, `ReproductionStepParser.ts`, or `ReproductionStepParser.test.ts`, substitute the resolved PlanModels/PlanParser names above.

Where it says `parseReproductionSteps`, substitute `parsePlanStatements`.

Its first failing parser test must cover both action and expectation syntax. Minimum representative input:

```text
OPEN "/account"
FILL "Email" WITH "QA.User@example.com"
CLICK "Save"
EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"
EXPECT_MAIN_STATUS 500
```

Parser tests must also reject unknown keywords, empty quoted values where prohibited, malformed syntax, and trailing garbage while preserving quoted case.

Plan 1 request-validation work must reject an omitted/empty plan, a plan containing only actions, and a plan containing only expectations before any external Solari resource is created.

Any later Plan 1 file list or command that names the superseded parser files must use the resolved names instead.

## Corrections to Plan 4

Plan 4 Task 1 executes only `ReproductionAction` statements. It receives the already-parsed unified plan and does not execute `ObservationExpectation` statements as browser actions.

Plan 4 Task 2 evaluates the already-final expectation grammar. Its transitional instruction to add expectations "if needed" is superseded. No contract extension is required merely to support `EXPECT_TEXT`, `EXPECT_NO_TEXT`, `EXPECT_URL`, `EXPECT_PAGE_ERROR`, or `EXPECT_MAIN_STATUS`; they already exist in the authoritative contract.

The observer must evaluate expectations against current browser/evidence state and produce `ExpectationResult[]` and `AttemptObservation` exactly as defined in the interface contract.

Plan 4 Task 6 must present one auditable plan editor containing both actions and expectations. Client validation is convenience only; server validation remains authoritative.

Plan 4 Task 8 fixture plans use the already-final grammar. Remove the transitional idea that the observation grammar still needs to be designed during that task.

## Correction to the test matrix

The earlier test-matrix row `H006 | Optional steps omitted | Submission remains valid` is obsolete and must not be implemented.

The authoritative H006 requirement is:

```text
H006 | Plan completeness | Empty/omitted plan, action-only plan, and expectation-only plan are rejected; a plan with at least one action and one expectation is accepted.
```

All other H-series numbering remains unchanged.

## Outcome semantics

Expectations describe the reported defect condition, not a generic healthy postcondition unless the condition is phrased accordingly.

For a deterministic defect scenario, satisfaction of the defined defect expectation supports `REPRODUCED` at the attempt level.

For a healthy negative-control scenario, the submitted plan must define the alleged defect condition such that the healthy fixture causes that expectation not to be observed. A failed defect expectation is not automatically `NOT_REPRODUCED` unless the required workflow was exercised sufficiently; ambiguity or incomplete authority remains `INCONCLUSIVE`.

Console warnings/errors and failed subrequests remain supporting evidence only unless an explicit expectation makes the exact observation relevant.

## File naming and type consistency gate

Before Plan 1 code is written, search the active plans for these transitional identifiers:

```text
ReproductionStepParser
parseReproductionSteps
ParsedReproductionStep
Optional steps omitted
If these are necessary
observation grammar is finalized
```

Treat matches according to this reconciliation document. New implementation code must use the current contract names, not the transitional names.

As implementation advances, remove or rewrite stale plan prose when the affected file is already being edited so future continuation does not depend forever on this reconciliation layer.

## Public-documentation status

This file is a development reconciliation record, not a product capability claim. Before final publication, the outside-reader audit decides whether detailed implementation planning/reconciliation files improve the public repository. If not, they may be omitted from the release candidate while preserving the public product design, architecture, validation, and attribution documents that remain useful to users and reviewers.

## Gate to begin implementation

Executable work may begin only after the executor has:

```text
read the authoritative documents in order
read this reconciliation file
confirmed repository/branch/dirty state
confirmed no newer contract supersedes this gate
confirmed the first implementation task uses PlanParser/ParsedPlanStatement
confirmed plan completeness is required
confirmed no feature capability is being claimed as already implemented
```

The first code change still follows the required red-green test cycle. This reconciliation document is planning evidence, not runtime proof.