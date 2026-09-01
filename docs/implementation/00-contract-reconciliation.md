# ReproDocket Pre-Implementation Contract Reconciliation

Date: August 31, 2026
Status: Mandatory pre-implementation gate

This document resolves specification drift discovered during the final pre-implementation audit. It is not runtime proof and it does not claim that ReproDocket has been implemented.

## Authority

Use this order when implementation documents disagree:

1. `docs/reprodocket-design.md`
2. `docs/reprodocket-interface-contracts.md`
3. `docs/reprodocket-data-handling.md`
4. `docs/reprodocket-security-lifecycle.md`
5. `docs/reprodocket-toolchain-baseline.md`
6. `docs/reprodocket-sdk-baseline.md`
7. `docs/reprodocket-fixture-spec.md`
8. `docs/reprodocket-ui-spec.md`
9. `docs/reprodocket-test-matrix.md`
10. this reconciliation document for the exact transitional plan corrections below
11. numbered implementation plans

Installed Solari package type declarations and live observed behavior remain the execution-time authority for SDK signatures. If they materially contradict the design, update the affected documentation before implementing around the contradiction.

## Resolved version-one request contract

A valid normal run requires:

```text
public HTTP/HTTPS target URL
human-readable problem description
auditable plan with at least one browser action
auditable plan with at least one observation expectation
```

The plan is required. It is not optional.

```ts
export interface CreateRunRequest {
  targetUrl: string;
  problem: string;
  plan: string[];
}
```

Validation:

```text
targetUrl required; maximum 2048 UTF-16 code units before normalization
problem trimmed length 1..10000
plan 2..100 nonblank statements
statement trimmed length 1..1000
at least one ReproductionAction
at least one ObservationExpectation
```

## Resolved parser contract

Plan 1 was drafted before observation expectations were finalized and uses transitional names such as `ReproductionStepParser`, `parseReproductionSteps`, and `ParsedReproductionStep`.

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

and the `ReproductionAction`, `ObservationExpectation`, `PlanStatement`, and `ParsedPlanStatement` contracts in `docs/reprodocket-interface-contracts.md`.

The parser recognizes action and expectation statements from the first implementation slice. Request-level validation, not single-line parsing, enforces that a submitted plan contains at least one action and one expectation.

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

This grammar is final for the version-one implementation baseline. Plan 4 must not treat observation expectations as a future optional extension.

## Corrections to Plan 1

Where Plan 1 Task 1 names `ReproductionAction.ts`, `ReproductionStepParser.ts`, or `ReproductionStepParser.test.ts`, substitute the resolved PlanModels/PlanParser names above.

Where it says `parseReproductionSteps`, substitute `parsePlanStatements`.

The first failing parser test must cover action and expectation syntax. Minimum representative input:

```text
OPEN "/account"
FILL "Email" WITH "QA.User@example.com"
CLICK "Save"
EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"
EXPECT_MAIN_STATUS 500
```

Parser tests also reject unknown keywords, prohibited empty values, malformed syntax, and trailing garbage while preserving quoted case.

Plan 1 request-validation work must reject an omitted/empty plan, an action-only plan, and an expectation-only plan before any external Solari resource is created.

Any later Plan 1 file list or command that names superseded parser files uses the resolved names instead.

## Corrections to Plan 4

Plan 4 Task 1 executes only `ReproductionAction` statements. It receives the already-parsed unified plan and does not execute `ObservationExpectation` statements as browser actions.

Plan 4 Task 2 evaluates the already-final expectation grammar. Its transitional instruction to add expectations "if needed" is superseded. No contract extension is required merely to support `EXPECT_TEXT`, `EXPECT_NO_TEXT`, `EXPECT_URL`, `EXPECT_PAGE_ERROR`, or `EXPECT_MAIN_STATUS`.

The observer evaluates expectations against current browser/evidence state and produces `ExpectationResult[]` and `AttemptObservation` exactly as defined in the interface contract.

Plan 4 Task 6 presents one auditable plan editor containing both actions and expectations. Client validation is convenience only; server validation remains authoritative.

Plan 4 Task 8 fixture plans use the already-final grammar. The observation grammar is not redesigned during that task.

## Test matrix correction status

The acceptance matrix has been directly updated. H006 now requires:

```text
H006 | Plan completeness | Omitted/empty, action-only, and expectation-only plans rejected; complete plan accepted.
```

Do not implement the obsolete earlier idea that reproduction steps may be omitted.

## Outcome semantics

Expectations describe the reported defect condition, not a generic healthy postcondition unless the allegation itself is phrased accordingly.

For a deterministic defect scenario, satisfaction of the defined defect expectation supports attempt-level reproduction.

For a healthy negative control, the plan defines the alleged defect condition so healthy target behavior causes that condition not to be observed. A failed defect expectation is not automatically `NOT_REPRODUCED` unless the required workflow was exercised sufficiently; ambiguity or incomplete authority remains `INCONCLUSIVE`.

Console warnings/errors and failed subrequests remain supporting evidence unless an explicit expectation makes the exact observation relevant.

## Resolved data-handling boundary

The submitted source plan is intentionally persisted so the run remains auditable and reproducible. Therefore arbitrary literal plan values cannot simultaneously be guaranteed secret and preserved exactly.

Version one supports **nonsecret plan data only**.

The UI and public README must tell users:

```text
Plans are saved with the run. Use test data only. Do not enter passwords, API keys, tokens, payment data, or other secrets.
```

Authenticated workflows requiring real secret injection are outside version-one scope until a separate secret-reference mechanism is designed and validated.

Broader statements in earlier design/security prose saying secrets are absent from manifests refer to ReproDocket/provider secrets and authoritatively recognized secret-bearing fields. They do not mean arbitrary user-authored plan strings can be perfectly classified as secret.

The Solari API key remains protected and must never be stored in a source plan, manifest, report, evidence artifact, command line, public source, or normal log.

See `docs/reprodocket-data-handling.md` for the controlling data contract.

## Resolved Node/toolchain floor

Earlier Plan 1 wording used `Node.js 22+` and a package engine of `>=22`. Current Vite 8 requirements make that floor too loose.

Use:

```text
minimum compatible Node: 22.12.0
fresh bootstrap preference: current supported Node LTS
```

At planning time Node 24.20.0 is the current LTS release, but bootstrap must discover the current LTS rather than hard-code that patch forever.

If an existing Node 22.12+ or later supported runtime restores dependencies and passes the real build/test toolchain, do not force an upgrade solely because a newer LTS exists.

Any Plan 1 package metadata must use an engine range that excludes Node 22.0 through 22.11 when Vite 8 remains selected.

See `docs/reprodocket-toolchain-baseline.md`.

## Resolved fixture authority

The deterministic scenarios are fixed in `docs/reprodocket-fixture-spec.md`.

The fixture includes:

```text
account save page error -> deterministic VERIFIED candidate
billing reload main-document 404 -> deterministic VERIFIED candidate
missing ZIP accepted -> deterministic VERIFIED candidate
healthy profile negative control -> NOT_REPRODUCED candidate
healthy login negative control -> NOT_REPRODUCED candidate
ambiguous duplicate control -> INCONCLUSIVE candidate
one-time nonrepeatable save -> REPRODUCED candidate
```

Production code receives only ordinary target URL/problem/plan inputs. It must never receive fixture identity as a classifier shortcut.

## Resolved UI authority

The UI behavior and copy contract is fixed in `docs/reprodocket-ui-spec.md`.

The plan editor is required. Syntax help is functional. Investigation and verification evidence are peers. Lifecycle failure is visually distinct from defect outcome. Cleanup state remains visible even for a positive defect outcome. Empty/pending/unavailable states are explicit. No dead control is accepted.

Version one does not add speculative settings, team, cloud-sync, theme, analytics, or generic browser-console surfaces.

## File naming and type consistency gate

Before Plan 1 code is written, search active plans for transitional identifiers:

```text
ReproductionStepParser
parseReproductionSteps
ParsedReproductionStep
Optional steps omitted
If these are necessary
observation grammar is finalized
Node.js 22+
">=22"
```

Treat matches according to this reconciliation document. New implementation code uses current contract names and current toolchain floors.

As implementation advances, rewrite stale plan prose when the affected plan is already being maintained so future continuation depends less on this reconciliation layer.

## Public-documentation status

This file is a development reconciliation record, not a product capability claim. Before final publication, the outside-reader audit decides whether detailed implementation/reconciliation material improves the public repository. If not, it may be omitted from the release candidate while retaining user/reviewer documentation that remains valuable.

## Gate to begin implementation

Executable work may begin only after the executor has confirmed:

```text
all authoritative documents read in current order
repository/branch/dirty state known
no newer contract supersedes this gate
PlanParser/ParsedPlanStatement names will be used
plan completeness is required
plan data is nonsecret and persisted
Node floor is at least 22.12.0 with current LTS preferred for fresh install
fixture spec is the test oracle
UI spec is the visual/user-boundary contract
no product capability is being claimed as already implemented
```

The first code change still follows the required red-green test cycle. This document is planning evidence, not runtime proof.
