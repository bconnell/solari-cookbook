# ReproDocket Pre-Codex Readiness Record

Date: August 31, 2026
Status: Planning-prepared; executable implementation intentionally not started

## 1. Purpose

This record identifies what has been resolved before local Codex execution begins and what still requires the actual Windows/Solari runtime. It prevents planning completeness from being confused with product completion.

This document is a handoff boundary, not a release-status claim.

## 2. Repository anchor at preparation checkpoint

Repository:

```text
bconnell/solari-cookbook
```

Working branch:

```text
feature/reprodocket
```

Upstream-derived main baseline used for this planning effort:

```text
d304843f5ea0edb5c27829bb2ca30868645bef7a
```

Checkpoint branch revision before this readiness file was created:

```text
86d423ccece86bec1d054440ea34a37c5b3b32d8
```

Because this file itself creates a later commit, execution must re-read `HEAD` and remote branch state rather than treating the checkpoint SHA above as the implementation start SHA.

## 3. Current maturity truth

At this handoff:

```text
Product design                    prepared
Interface/API contracts           prepared
Data/privacy contract             prepared
Security/lifecycle contract       prepared
Toolchain baseline                prepared
Solari SDK baseline               prepared from current public docs/examples
Deterministic fixture contract    prepared
Local UI contract                 prepared
Acceptance/test matrix            prepared
Implementation plans              prepared
Persistent AGENTS.md guidance     prepared
Executable ReproDocket source     NOT IMPLEMENTED
ReproDocket package-lock          NOT GENERATED
Local build                       NOT RUN
Local tests                       NOT RUN
Live Solari tests                 NOT RUN
Full validation                   NOT RUN
Human product QA                  NOT RUN
Release/publication candidate     NOT READY
```

The project must remain described publicly as Planned/pre-implementation until executable evidence supports a later maturity state.

## 4. Resolved product decisions

The following decisions are not open implementation questions unless current runtime evidence proves a material incompatibility:

```text
ReproDocket remains inside the public Solari cookbook fork.
Upstream examples and MIT license are preserved.
TypeScript/Node is the primary application stack.
Fastify is the local loopback server.
React/Vite is the local UI.
SSE carries live progress; persisted GET state remains authoritative.
Filesystem run storage replaces a database for version one.
Normal Windows credential storage uses the current-user protected OS boundary.
No user-created .env file is required.
Production investigation uses real Solari cloud browsers.
Full deterministic validation hosts its fixture in a real Solari sandbox.
There is no production local-browser fallback.
Investigation and verification use separate Solari browser sessions.
A run plan is required and includes browser actions plus observation expectations.
The source plan is intentionally persisted and must use nonsecret test data.
Arbitrary autonomous planning from prose is not a version-one claim.
One active execution pipeline per local process is the explicit initial capacity.
Cancellation is a product lifecycle, not process termination.
Evidence is locally persisted, minimized, hashed when finalized, and tied to run ownership.
Target screenshots/replays are not claimed automatically privacy-safe.
Real provider/browser/sandbox cleanup is part of Full validation.
Horizontal and vertical feature connectivity are completion gates.
Feature implementation is followed by deliberate bug discovery, proportional hardening, and fresh revalidation.
Human UI/usability acceptance is separate from machine Full validation.
```

## 5. Required execution-time unknowns

These cannot be truthfully resolved from GitHub/docs alone and are the first local-runtime authority checks.

### Windows/toolchain state

Confirm on the actual machine:

```text
Windows edition/build and x64 architecture for the tested support claim
current PowerShell 7 availability/version
Windows PowerShell 5.1 bootstrap parsing path if exercised
Node version
npm version
winget/package-manager availability when installation is needed
free space on project/generated-output volumes
Git version and worktree support
browser/default-browser behavior
local Playwright browser availability after installation
```

### npm compatibility

Run current registry checks before freezing `package.json`:

```text
@solarisdk/browser
@solarisdk/sandbox
fastify
@fastify/static
react
react-dom
zod
vite
@vitejs/plugin-react
vitest
@playwright/test
typescript
typescript-eslint
eslint
prettier
@types/node
@types/react
@types/react-dom
```

The selected set must restore, typecheck, build, and run together. Planning-time version observations are not sufficient evidence.

### Solari Browser SDK

Inspect installed TypeScript declarations and prove live behavior for:

```text
client constructor/authentication shape
launch() recording option
browser/session ID
newPage()
Playwright-compatible Page/locator methods used by the executor
screenshot return behavior
console/pageerror/request/response event compatibility
browser.close() behavior
client close/dispose behavior needed to let Node exit
replay readiness API
replay URL API
whether a supported TypeScript replay-download method exists
provider error/status shapes used for retry mapping
```

Do not infer a method merely from the Python example.

### Solari Sandbox SDK

Inspect installed declarations and prove:

```text
SandboxClient construction
create() options/current template name
sandbox ID
connect() requirement/current semantics
files.write() input shape
commands.run() input/result/exit-code shape
previewUrl() result shape
kill() behavior and errors
whether python3 exists in the chosen base template
whether the uploaded standard-library fixture server remains alive in background
public preview readiness timing
```

### Solari account/runtime state

Using the supplied user credential only at the local secret boundary, prove:

```text
credential accepted
browser session creation permitted
sandbox creation permitted
current concurrency/capacity behavior
preview URL accessible externally
recording/replay enabled for the account/path
normal cleanup succeeds
no existing unrelated resource is touched
```

Do not encode account-specific limits as universal product facts.

### Windows protected-secret boundary

Prove on the machine:

```text
CurrentUser protection helper works
round-trip fake secret succeeds
stored bytes do not contain plaintext
child command line does not contain plaintext
failure messages do not echo plaintext
real Solari key can be verified without entering run evidence
```

## 6. Exact implementation start procedure

The first Codex session should not begin by scaffolding files immediately.

It first performs read-only reconciliation:

```powershell
git status --short
git branch --show-current
git rev-parse HEAD
git remote -v
git fetch origin
git rev-list --left-right --count main...HEAD
git diff --check main...HEAD
```

Then it reads `AGENTS.md` and the authoritative documents in the order listed there.

If local branch state contains user work not represented by the remote planning branch, preserve it and reconcile before writes. Do not reset, clean, stash, rebase, or overwrite it as a convenience.

## 7. First behavior-changing implementation task

After repository/toolchain reconciliation, the first code task is Plan 1's unified plan parser and project foundation, using the corrections in `00-contract-reconciliation.md`.

The first parser names are:

```text
reprodocket/src/shared/contracts/PlanModels.ts
reprodocket/src/core/PlanParser.ts
reprodocket/tests/unit/PlanParser.test.ts
parsePlanStatements(lines: string[]): ParsedPlanStatement[]
```

The first real test cycle must be observable:

```text
create project/test harness
write parser tests before parser implementation
run parser test and confirm expected RED
implement bounded parser
run parser test and confirm GREEN
run typecheck/lint/format checks
review repository state
commit only after the slice is current and validated
```

Do not create a placeholder parser that simply returns fixture-specific expected statements.

## 8. Local environment automation target

The intended user experience remains:

```text
clone/checkout
run bootstrap or run.ps1
approve unavoidable machine installation/elevation only when required
provide Solari API key through the local UI once when needed
use ReproDocket locally
run validate.ps1 for Full validation
```

The user should not manually create databases, environment files, ports, frontend/backend servers, evidence folders, or test harness configuration.

Codex is expected to repair safely automatable prerequisite/configuration issues itself and stop only at genuine OS authority, secret, provider-account, or human-quality boundaries.

## 9. No-preimplementation-code decision

No ReproDocket executable application source was intentionally authored through the GitHub connector during planning.

Reason: this chat environment cannot run the Windows build/test stack or live Solari account. Prewriting large uncompiled/unexecuted feature code would undermine the required red-green and fresh-runtime evidence model.

This is deliberate scope discipline, not missing preparation.

## 10. Name status

Working product name: `ReproDocket`.

A renewed exact-name search during the planning audit found no exact GitHub repository and no obvious exact public software/package result across general web, npm, PyPI, and NuGet searches.

This is a collision screen, not legal trademark clearance. The name remains acceptable for the challenge build unless a later specific conflict is discovered.

## 11. Publication-state boundary

Do not yet:

```text
modify the root cookbook README to claim a working product
create a release/tag
open a final integration PR
publish challenge/social copy
publish product screenshots
claim test/build/Full PASS
claim Windows support beyond the actually tested environment
claim replay download until TypeScript support is proven
claim authenticated/secret-bearing test plans
```

Those are later phases with their own evidence gates.

## 12. Deliberately deferred decisions

These are not blockers for version-one implementation and should remain deferred unless evidence demands them:

```text
runtime AI planner/provider
macOS/Linux zero-configuration bootstrap
multi-run parallel execution
persistent queue
cloud history sync
multi-user/team support
plugin system
custom selectors/JavaScript actions
secret-reference/authenticated workflow support
local replay viewer
automatic screenshot redaction
```

Do not implement them preemptively.

## 13. Pre-Codex handoff acceptance

This planning handoff is acceptable to begin local execution when all of the following remain true at the actual start:

```text
feature/reprodocket exists and can be reconciled with the local checkout
upstream examples/LICENSE are preserved
current design/contracts/plans are readable
00-contract-reconciliation is applied before Plan 1
test matrix contains required plan-completeness acceptance
AGENTS.md contains the current reading/order/safety rules
no executable ReproDocket source is being misrepresented as already tested
user/local machine authority is available for implementation
Solari credential can be supplied privately at runtime
```

If any item changed after this record, re-read current state instead of trusting this checkpoint.

## 14. First expected local report

Before implementing beyond the initial foundation, the local executor should report:

```text
actual starting HEAD and divergence
working-tree state
Node/npm/PowerShell/Git versions
disk-space check
current direct package versions selected
first RED test result
first GREEN test result
static-check results
files changed
commit SHA if validation permits commit
remaining execution-time blockers
```

That becomes the first real runtime evidence for ReproDocket.
