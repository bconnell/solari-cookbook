# ReproDocket Foundation and Local Shell Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish ReproDocket's reproducible TypeScript project, real test harness, safe request and plan contracts, loopback local application shell, Windows bootstrap, process ownership, protected Solari credential flow, and the first truthful validation authority.

**Architecture:** One Node process serves a built React application and JSON/SSE API on loopback. Core validation and persistence remain framework independent. PowerShell owns Windows bootstrap/start/stop behavior, while a narrow Windows current-user protected-data adapter owns local Solari credential protection.

**Tech Stack:** Node.js 22.12.0 or newer on the supported Node 22 line, with fresh setup preferring the current supported Node LTS; TypeScript; Fastify; React; Vite; Zod; Vitest; Playwright Test; PowerShell 7; Windows current-user protected data; npm lockfile.

**Specs:** `docs/reprodocket-design.md`, `docs/reprodocket-interface-contracts.md`, `docs/reprodocket-data-handling.md`, `docs/reprodocket-security-lifecycle.md`, `docs/reprodocket-toolchain-baseline.md`, `docs/reprodocket-sdk-baseline.md`, `docs/reprodocket-ui-spec.md`, `docs/reprodocket-test-matrix.md`, and `docs/implementation/00-contract-reconciliation.md`.

## Global Constraints

* Work on `feature/reprodocket` or an isolated worktree based on it. Re-read branch, HEAD, divergence, staged, unstaged, untracked, and ignored state before editing.
* Preserve upstream `examples/` and the root `LICENSE`.
* Do not reset, clean, stash, rebase, force push, or overwrite unrelated work as a shortcut.
* New Windows automation must not use WMIC or `wmic.exe`.
* Node.js must satisfy the current Vite/toolchain floor. With Vite 8 selected, the minimum is `22.12.0`; a fresh bootstrap should prefer current Node LTS discovered at execution time.
* Normal product data must not be written into the Git repository.
* The local service binds to loopback only and rejects unauthorized cross-origin state mutation.
* The normal user must not create `.env` files, databases, manually selected ports, or separate frontend/backend terminals.
* `reprodocket/package-lock.json` is intentionally tracked after dependency installation.
* User-authored plan text is durable run data. Version-one plan values must use nonsecret test data.
* Do not add a second mandatory runtime AI/API credential.
* Tests must exercise real behavior and be capable of failing for the defects they guard.
* Behavior-changing work follows observable RED -> implementation -> GREEN, then adjacent/static checks and required mutation proof.
* Use current installed Solari package type declarations and live observed behavior as the highest SDK authority.
* No Plan 1 completion claim is valid without fresh command output from the current source revision.

---

## Task 1: Create the project and implement the unified plan parser

**Files:**
- Create: `reprodocket/package.json`
- Create: `reprodocket/package-lock.json`
- Create: `reprodocket/tsconfig.json`
- Create: `reprodocket/tsconfig.server.json`
- Create: `reprodocket/vite.config.ts`
- Create: `reprodocket/vitest.config.ts`
- Create: `reprodocket/playwright.config.ts`
- Create: `reprodocket/eslint.config.js`
- Create: `reprodocket/.prettierrc.json`
- Create: `reprodocket/index.html`
- Create: `reprodocket/src/shared/contracts/PlanModels.ts`
- Create: `reprodocket/src/core/PlanParser.ts`
- Test: `reprodocket/tests/unit/PlanParser.test.ts`

**Interfaces:**
- Produces `parsePlanStatements(lines: string[]): ParsedPlanStatement[]`.
- Produces `ReproductionAction`, `ObservationExpectation`, `PlanStatement`, and `ParsedPlanStatement` exactly as defined in `docs/reprodocket-interface-contracts.md`.

- [ ] **Step 1: Inspect the actual machine and current registry before freezing package metadata**

Run from repository root:

```powershell
$ErrorActionPreference = "Stop"
node --version
npm --version
pwsh --version
git --version
npm view @solarisdk/browser version
npm view @solarisdk/sandbox version
npm view fastify version
npm view @fastify/static version
npm view react version
npm view react-dom version
npm view zod version
npm view vite version
npm view @vitejs/plugin-react version
npm view vitest version
npm view @playwright/test version
npm view typescript version
npm view typescript-eslint version
npm view eslint version
npm view prettier version
npm view @types/node version
npm view @types/react version
npm view @types/react-dom version
```

Expected: the installed Node runtime satisfies the current toolchain floor and each registry query returns a concrete version. If the current Solari package versions or package APIs materially contradict `docs/reprodocket-sdk-baseline.md`, update that baseline and affected contracts before proceeding.

- [ ] **Step 2: Create the package skeleton with the current runtime floor**

`reprodocket/package.json` begins with this script and engine shape. Dependency versions are added by the exact-install commands in the next step and then become authoritative through the lockfile.

```json
{
  "name": "reprodocket",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=22.12.0"
  },
  "scripts": {
    "clean:dist": "node -e \"require('node:fs').rmSync('dist',{recursive:true,force:true})\"",
    "build:client": "vite build",
    "build:server": "tsc -p tsconfig.server.json",
    "build": "npm run clean:dist && npm run build:client && npm run build:server",
    "typecheck": "tsc -p tsconfig.json --noEmit && tsc -p tsconfig.server.json --noEmit",
    "lint": "eslint .",
    "format:check": "prettier --check .",
    "format": "prettier --write .",
    "test": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:live": "vitest run tests/live",
    "test:ui": "playwright test",
    "start": "node dist/server/server/server.js"
  }
}
```

- [ ] **Step 3: Install current direct dependencies exactly and create the lockfile**

Run:

```powershell
Set-Location reprodocket
npm install --save-exact @solarisdk/browser @solarisdk/sandbox fastify @fastify/static react react-dom zod
npm install --save-dev --save-exact @playwright/test @types/node @types/react @types/react-dom @vitejs/plugin-react eslint prettier typescript typescript-eslint vite vitest
npm ls --depth=0
```

Expected: installation succeeds, every direct package version is exact in `package.json`, `package-lock.json` exists, and `npm ls --depth=0` reports no missing or invalid direct dependency. If the latest independently resolved packages are incompatible, do not randomly downgrade. Resolve the smallest compatible set, record why, and update `docs/reprodocket-toolchain-baseline.md` before continuing.

- [ ] **Step 4: Create strict TypeScript and build configuration**

`tsconfig.json` must include at least:

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "skipLibCheck": false,
    "resolveJsonModule": true,
    "types": ["vitest/globals"]
  },
  "include": ["src", "tests", "vite.config.ts", "vitest.config.ts"]
}
```

`tsconfig.server.json` extends it, uses the Node module/resolution mode required by the actually installed TypeScript/Node combination, excludes `src/client`, sets `rootDir` to `src`, and emits under `dist/server`. Configure Vite to emit the client under `dist/client`.

- [ ] **Step 5: Write the first failing unified parser tests**

The first test must prove that actions and expectations are parsed by the same closed grammar and that quoted case is preserved.

```ts
import { describe, expect, it } from "vitest";
import { parsePlanStatements } from "../../src/core/PlanParser.js";

describe("parsePlanStatements", () => {
  it("parses action and expectation statements while preserving quoted case", () => {
    const result = parsePlanStatements([
      'OPEN "/account"',
      'FILL "Email" WITH "QA.User@example.com"',
      'CLICK "Save"',
      'EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"',
      "EXPECT_MAIN_STATUS 500"
    ]);

    expect(result).toHaveLength(5);
    expect(result[1]?.source).toContain("QA.User@example.com");
  });

  it("rejects arbitrary script and selector execution", () => {
    expect(() => parsePlanStatements(['EVALUATE "alert(1)"'])).toThrow();
    expect(() => parsePlanStatements(['CLICK_CSS "#danger"'])).toThrow();
  });

  it("reports the one-based failing statement index", () => {
    expect(() => parsePlanStatements(['CLICK "Save"', "BROKEN"])).toThrow(/statement 2|step 2/i);
  });
});
```

Also cover every supported action and expectation from the interface contract, empty prohibited quoted values, malformed quoting, invalid status codes, unsupported keys, blank lines according to the request-normalization contract, and trailing garbage.

- [ ] **Step 6: Run the parser test and prove RED**

```powershell
npm run test:unit -- PlanParser.test.ts
```

Expected: FAIL because `PlanParser` does not yet exist.

- [ ] **Step 7: Implement the bounded parser and plan model**

Use anchored deterministic parsing. Preserve the original source line in `ParsedPlanStatement.source`. Return only the documented action/expectation union. Do not use `eval`, `Function`, arbitrary CSS/XPath, arbitrary JavaScript, shell execution, or fixture-specific route logic.

- [ ] **Step 8: Prove GREEN and run static checks**

```powershell
npm run test:unit -- PlanParser.test.ts
npm run typecheck
npm run lint
npm run format:check
```

Expected: all commands exit zero.

- [ ] **Step 9: Verify dependency/output tracking**

From repository root:

```powershell
git check-ignore -v reprodocket/package-lock.json
if ($LASTEXITCODE -eq 0) { throw "ReproDocket lockfile is ignored." }
git status --short
```

Expected: the lockfile is not ignored and `node_modules` is not tracked.

- [ ] **Step 10: Commit the validated foundation slice**

```powershell
git add reprodocket/package.json reprodocket/package-lock.json reprodocket/tsconfig.json reprodocket/tsconfig.server.json reprodocket/vite.config.ts reprodocket/vitest.config.ts reprodocket/playwright.config.ts reprodocket/eslint.config.js reprodocket/.prettierrc.json reprodocket/index.html reprodocket/src/shared/contracts/PlanModels.ts reprodocket/src/core/PlanParser.ts reprodocket/tests/unit/PlanParser.test.ts
git diff --cached --check
git commit -m "build: establish ReproDocket TypeScript foundation"
```

---

## Task 2: Implement target URL validation and literal network blocking

**Files:**
- Create: `reprodocket/src/core/TargetUrlPolicy.ts`
- Create: `reprodocket/src/shared/errors/ReproDocketError.ts`
- Test: `reprodocket/tests/unit/TargetUrlPolicy.test.ts`

**Interfaces:**
- Produces `validateTargetUrl(input: string): URL`.
- Produces `ReproDocketError` carrying a stable error code from the documented taxonomy.

- [ ] **Step 1: Write failing target-policy tests**

Cover C001 through C020 and C023 from `docs/reprodocket-test-matrix.md`: public HTTP/HTTPS success; executable/local scheme rejection; credentials in URL; localhost and `.local`; IPv4 loopback including all of 127/8; RFC1918; IPv4 link-local; metadata address; IPv6 loopback, unique-local, and link-local; malformed URL/host; and absolute plan destinations using the same policy.

Representative assertions:

```ts
expect(validateTargetUrl("https://example.com/path").href).toBe("https://example.com/path");
expect(() => validateTargetUrl("javascript:alert(1)")).toThrowError(/INVALID_TARGET_URL/);
expect(() => validateTargetUrl("http://127.12.2.3")).toThrowError(/BLOCKED_TARGET_NETWORK/);
expect(() => validateTargetUrl("http://169.254.169.254/latest/meta-data")).toThrowError(/BLOCKED_TARGET_NETWORK/);
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- TargetUrlPolicy.test.ts
```

Expected: FAIL because the policy does not exist.

- [ ] **Step 3: Implement URL, hostname, and literal-address rules**

Use the platform `URL` parser plus dedicated IPv4/IPv6 range logic. Do not recognize private ranges with substring matching. This pure policy does not claim DNS resolution or redirect authority; those belong to the later browser navigation boundary.

- [ ] **Step 4: Prove GREEN with parser adjacency**

```powershell
npm run test:unit -- TargetUrlPolicy.test.ts PlanParser.test.ts
npm run typecheck
```

- [ ] **Step 5: Mutation prove one private-target guard**

Temporarily disable a private IPv4 rejection and rerun `TargetUrlPolicy.test.ts`. Confirm a test fails. Restore the guard and rerun to PASS. Do not commit the deliberate regression.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/core/TargetUrlPolicy.ts reprodocket/src/shared/errors/ReproDocketError.ts reprodocket/tests/unit/TargetUrlPolicy.test.ts
git diff --cached --check
git commit -m "feat: enforce safe ReproDocket target URLs"
```

---

## Task 3: Define request validation, versioned run schemas, lifecycle, and outcome invariants

**Files:**
- Create: `reprodocket/src/shared/contracts/RunModels.ts`
- Create: `reprodocket/src/core/CreateRunRequestValidator.ts`
- Create: `reprodocket/src/core/RunLifecycleMachine.ts`
- Create: `reprodocket/src/core/OutcomeClassifier.ts`
- Test: `reprodocket/tests/unit/CreateRunRequestValidator.test.ts`
- Test: `reprodocket/tests/unit/RunLifecycleMachine.test.ts`
- Test: `reprodocket/tests/unit/OutcomeClassifier.test.ts`

**Interfaces:**
- Produces the run, attempt, evidence, resource, replay, provenance, request, and history types in `docs/reprodocket-interface-contracts.md`.
- Produces `validateCreateRunRequest(input): { request: CreateRunRequest; parsedPlan: ParsedPlanStatement[] }`.
- Produces `transitionLifecycle(current, next): RunLifecycle`.
- Produces `classify(investigation, verification): RunOutcome`.

- [ ] **Step 1: Write request completeness tests B006 through B009**

Require all of these:

```text
missing/empty plan -> rejected
action-only plan -> rejected for missing expectation
expectation-only plan -> rejected for missing executable action
complete action + expectation plan -> accepted
```

Also test target/problem/plan length limits from the interface contract and verify invalid input is rejected before any provider interface can be invoked.

- [ ] **Step 2: Write lifecycle tests B010 through B019**

Test every legal transition and representative illegal transition. Terminal states cannot silently return to active execution.

- [ ] **Step 3: Write classifier tests B020 through B028**

A `VERIFIED` result requires nonempty supporting evidence from both attempts and different non-null Solari session IDs. Infrastructure failure never becomes `NOT_REPRODUCED`. Conflicting or insufficient evidence remains uncertain.

- [ ] **Step 4: Prove RED**

```powershell
npm run test:unit -- CreateRunRequestValidator.test.ts RunLifecycleMachine.test.ts OutcomeClassifier.test.ts
```

Expected: FAIL because the validator/lifecycle/classifier do not exist.

- [ ] **Step 5: Implement versioned Zod schemas and inferred TypeScript types**

Create `RunManifestSchemaV1` matching the authoritative manifest contract. Infer TypeScript types from schemas where practical so reload validation and compile-time types do not drift.

- [ ] **Step 6: Implement request validation**

Normalize and validate target/problem/plan, call `parsePlanStatements`, require at least one action and one expectation, and return stable documented errors. Do not create external resources in this layer.

- [ ] **Step 7: Implement explicit lifecycle transitions**

Use an allowed-transition map. Do not infer legality from string order or terminal-name checks alone.

- [ ] **Step 8: Implement the classifier**

Apply the documented two-attempt decision table. Assert the session/evidence invariants before `VERIFIED`; invariant violation becomes a stable internal failure, never a silent positive result.

- [ ] **Step 9: Prove GREEN and mutation sensitivity**

```powershell
npm run test:unit -- CreateRunRequestValidator.test.ts RunLifecycleMachine.test.ts OutcomeClassifier.test.ts
npm run typecheck
```

Then temporarily remove the distinct-session guard and confirm `OutcomeClassifier.test.ts` fails. Restore and rerun.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/src/shared/contracts/RunModels.ts reprodocket/src/core/CreateRunRequestValidator.ts reprodocket/src/core/RunLifecycleMachine.ts reprodocket/src/core/OutcomeClassifier.ts reprodocket/tests/unit/CreateRunRequestValidator.test.ts reprodocket/tests/unit/RunLifecycleMachine.test.ts reprodocket/tests/unit/OutcomeClassifier.test.ts
git diff --cached --check
git commit -m "feat: define ReproDocket request and run truth"
```

---

## Task 4: Establish application-owned paths and durable run persistence

**Files:**
- Create: `reprodocket/src/storage/AppPaths.ts`
- Create: `reprodocket/src/storage/AtomicJsonFile.ts`
- Create: `reprodocket/src/storage/FileRunStore.ts`
- Test: `reprodocket/tests/unit/AppPaths.test.ts`
- Test: `reprodocket/tests/integration/FileRunStore.test.ts`

**Interfaces:**
- Produces `resolveAppPaths(overrides?): AppPaths`.
- Implements initial `RunStore.create`, `get`, `list`, and `update` behavior from the interface contract.

- [ ] **Step 1: Write path-boundary tests**

Use a test-only explicit root. Assert run/artifact paths derive only from generated IDs, reject traversal-like IDs, and never default to Desktop, Documents, Downloads, or the source tree.

- [ ] **Step 2: Write persistence tests F001 through F005 and F012 through F014**

Verify unique creation, fail-if-exists behavior, validated legal updates, atomic state, malformed-run isolation, unsupported schema handling, completed reload, and interrupted-run projection behavior.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- AppPaths.test.ts
npm run test:integration -- FileRunStore.test.ts
```

- [ ] **Step 4: Implement application paths**

On Windows, use the current user's `LOCALAPPDATA` and an application root ending in `ReproDocket`. If a safe application root cannot be determined, fail explicitly rather than writing into the repository.

- [ ] **Step 5: Implement atomic JSON persistence**

Write a sibling temporary file, close/flush where practical, then rename/replace within the same directory. Validate persisted data with the current schema before returning it as authority.

- [ ] **Step 6: Implement the run store**

Use exclusive run-directory creation. Validate each historical manifest independently. One damaged run must not crash the entire history projection.

- [ ] **Step 7: Prove GREEN**

```powershell
npm run test:unit
npm run test:integration -- FileRunStore.test.ts
npm run typecheck
```

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/storage reprodocket/tests/unit/AppPaths.test.ts reprodocket/tests/integration/FileRunStore.test.ts
git diff --cached --check
git commit -m "feat: persist ReproDocket run state safely"
```

---

## Task 5: Build the secure loopback Fastify shell

**Files:**
- Create: `reprodocket/src/server/createServer.ts`
- Create: `reprodocket/src/server/server.ts`
- Create: `reprodocket/src/server/security/LocalRequestGuard.ts`
- Create: `reprodocket/src/server/routes/health.ts`
- Create: `reprodocket/src/server/routes/bootstrap.ts`
- Test: `reprodocket/tests/integration/LocalServerSecurity.test.ts`
- Test: `reprodocket/tests/integration/HealthRoutes.test.ts`

**Interfaces:**
- Produces `createServer(options): FastifyInstance`.
- Implements `GET /api/health` and `GET /api/bootstrap`.
- Establishes `X-ReproDocket-Request` as the process-local mutation nonce header.

- [ ] **Step 1: Write failing D001 through D011 tests**

Use Fastify injection for Host, Origin, nonce, CORS, and security-header behavior and an actual ephemeral listener to prove loopback-only binding rather than only asserting a configuration string.

- [ ] **Step 2: Write failing health/bootstrap tests**

`/api/health` returns only nonsecret health/build identity. `/api/bootstrap` returns the process instance ID and a nonempty request nonce. Neither may return a Solari key, local absolute path, provider-account detail, or raw stack trace.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- LocalServerSecurity.test.ts HealthRoutes.test.ts
```

- [ ] **Step 4: Implement server creation and local request guard**

Production startup passes `127.0.0.1` explicitly. Do not add permissive CORS. Validate Host on requests and require same-origin plus the request nonce for mutations.

- [ ] **Step 5: Add the documented security headers**

Use the current CSP/header baseline from `docs/reprodocket-security-lifecycle.md`. Do not require `unsafe-eval`.

- [ ] **Step 6: Implement health/bootstrap routes**

Generate process identity/nonce using Node crypto. Keep the nonce in process memory only.

- [ ] **Step 7: Prove GREEN**

```powershell
npm run test:integration -- LocalServerSecurity.test.ts HealthRoutes.test.ts
npm run typecheck
npm run lint
```

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/server reprodocket/tests/integration/LocalServerSecurity.test.ts reprodocket/tests/integration/HealthRoutes.test.ts
git diff --cached --check
git commit -m "feat: add secure loopback application shell"
```

---

## Task 6: Build and serve the real local React shell

**Files:**
- Create: `reprodocket/src/client/main.tsx`
- Create: `reprodocket/src/client/App.tsx`
- Create: `reprodocket/src/client/styles.css`
- Create: `reprodocket/src/client/api/ApiClient.ts`
- Create: `reprodocket/src/server/static/registerStaticUi.ts`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/ui/AppShell.spec.ts`

**Interfaces:**
- Serves the built application at `/` from the same loopback Node process.
- Client bootstraps through `GET /api/bootstrap` and retains the request nonce only in runtime memory.

- [ ] **Step 1: Write the failing built-shell Playwright test**

The test starts the built server, visits `/`, and requires the structural shell rather than fake future product controls:

```ts
await expect(page.getByRole("heading", { name: "ReproDocket" })).toBeVisible();
await expect(page.getByRole("navigation")).toBeVisible();
await expect(page.getByText(/Solari/i)).toBeVisible();
```

The test also requires no raw stack trace/debug overlay on normal load.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- AppShell.spec.ts
```

Expected: the shell/static-serving test fails because the UI does not exist.

- [ ] **Step 3: Implement the minimal accessible application shell**

Follow `docs/reprodocket-ui-spec.md` for layout hierarchy, typography, status semantics, focus, and responsive behavior. Do not add dead New Investigation, history, replay, or evidence controls before their real behavior exists.

- [ ] **Step 4: Serve built assets from one static root**

Serve only `dist/client` through `@fastify/static`. Add SPA fallback for application routes without intercepting `/api/*`.

- [ ] **Step 5: Connect bootstrap state**

On application load, call `/api/bootstrap`, show truthful loading/error state, and keep the request nonce in memory for the API client.

- [ ] **Step 6: Prove GREEN and inspect a current screenshot**

```powershell
npm run build
npm run test:ui -- AppShell.spec.ts
npm run typecheck
```

Capture a temporary original-resolution screenshot under `reprodocket/.artifacts/targeted/` and inspect it for clipping, unreadable text, accidental debug content, and misleading controls. Do not commit the screenshot.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/client reprodocket/src/server/static reprodocket/src/server/createServer.ts reprodocket/tests/ui/AppShell.spec.ts
git diff --cached --check
git commit -m "feat: serve the ReproDocket local interface"
```

---

## Task 7: Implement owned local instance identity and deterministic port selection

**Files:**
- Create: `reprodocket/src/server/startup/PortAllocator.ts`
- Create: `reprodocket/src/server/startup/InstanceRecord.ts`
- Modify: `reprodocket/src/server/server.ts`
- Test: `reprodocket/tests/integration/InstanceOwnership.test.ts`
- Test: `reprodocket/tests/integration/PortAllocator.test.ts`

**Interfaces:**
- Produces `findLoopbackPort(preferred: number): Promise<number>`.
- Produces an ownership record containing PID, process start identity, port, bind address, application/build identity, instance nonce, and start timestamp.

- [ ] **Step 1: Write an occupied-port test**

Bind a test listener on the preferred port, call the allocator, require a different available loopback port, and prove the foreign listener remains alive.

- [ ] **Step 2: Write instance-identity tests**

A valid record must contain more authority than PID. Deliberately mismatched process-start identity, application identity, or instance nonce must prevent ownership claims.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- InstanceOwnership.test.ts PortAllocator.test.ts
```

- [ ] **Step 4: Implement bounded port allocation**

Start at preferred port 4317 and probe a bounded loopback range. If no port is available, throw `LOCAL_PORT_UNAVAILABLE`. Port allocation never kills a process.

- [ ] **Step 5: Implement ownership record persistence**

Write under the application-owned runtime root using atomic persistence. Choose a process-start identity that the later stop script can verify on Windows without WMIC.

- [ ] **Step 6: Prove GREEN and mutation sensitivity**

Temporarily make the allocator reuse the occupied port and confirm the test fails. Restore and rerun.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/server/startup reprodocket/src/server/server.ts reprodocket/tests/integration/InstanceOwnership.test.ts reprodocket/tests/integration/PortAllocator.test.ts
git diff --cached --check
git commit -m "feat: track local ReproDocket instance ownership"
```

---

## Task 8: Build Windows bootstrap, preflight, run, and stop automation

**Files:**
- Create: `reprodocket/scripts/bootstrap.ps1`
- Create: `reprodocket/scripts/preflight.ps1`
- Create: `reprodocket/scripts/run.ps1`
- Create: `reprodocket/scripts/stop.ps1`
- Create: `reprodocket/scripts/lib/Common.ps1`
- Test: `reprodocket/tests/integration/PowerShellScripts.test.ts`

**Interfaces:**
- Normal start command: `.\reprodocket\scripts\run.ps1`.
- Owned stop command: `.\reprodocket\scripts\stop.ps1`.

- [ ] **Step 1: Write failing Windows-script tests for R001 through R020 that are applicable at this phase**

Launch PowerShell in controlled environments to verify parser/runtime behavior, script-relative path resolution, unsupported Node rejection, generated-output roots, port handling, ownership checks, disk-reserve decisions, and a source scan that fails if new scripts contain WMIC usage.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- PowerShellScripts.test.ts
```

- [ ] **Step 3: Implement shared script helpers**

Use `$PSScriptRoot` for paths. Query free space through modern PowerShell/.NET/CIM APIs. Emit nonzero failures. Never use Desktop, Documents, or Downloads as default scratch locations.

- [ ] **Step 4: Implement bootstrap**

The small entry path must parse under Windows PowerShell 5.1. Detect PowerShell 7 and Node. Accept an existing compatible Node runtime. If Node is missing or below the supported floor and `winget` is available, install the current Node LTS using the explicit trusted package ID, then verify the installed executable/version. If safe automation is unavailable, stop with precise remediation instead of downloading an unverified binary.

After a lockfile exists, dependency restore uses `npm ci`, not `npm update`.

- [ ] **Step 5: Implement preflight**

Validate PowerShell, Node, npm, Git, disk reserve, application-data write access, lockfile, dependency tree, build capability, and local port behavior. Solari live authority is added in later plans.

- [ ] **Step 6: Implement run**

If a valid ownership record points to a healthy owned ReproDocket instance, open/reuse it and exit. Otherwise build when required, start one server, poll `/api/health` with a bounded timeout, and open the default browser. If browser launch fails, print the local URL and leave a healthy server running.

- [ ] **Step 7: Implement stop**

Verify PID plus process-start/application/instance identity before requesting termination. If identity is stale or mismatched, leave the process untouched and report the condition.

- [ ] **Step 8: Run under available PowerShell hosts**

```powershell
powershell.exe -NoProfile -File .\reprodocket\scripts\preflight.ps1 -LocalOnly
pwsh.exe -NoProfile -File .\reprodocket\scripts\preflight.ps1 -LocalOnly
npm run test:integration -- PowerShellScripts.test.ts
```

If one host is genuinely unavailable, record BLOCKED for that exact host instead of synthesizing PASS.

- [ ] **Step 9: Exercise the real local start/reuse/stop path**

```powershell
.\reprodocket\scripts\run.ps1
.\reprodocket\scripts\run.ps1
.\reprodocket\scripts\stop.ps1
```

Require one owned application instance, reuse on the second start, and termination only of that owned process.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/scripts reprodocket/tests/integration/PowerShellScripts.test.ts
git diff --cached --check
git commit -m "feat: automate ReproDocket Windows startup"
```

---

## Task 9: Implement the Windows protected Solari secret store

**Files:**
- Create: `reprodocket/src/security/SecretStore.ts`
- Create: `reprodocket/src/security/WindowsDpapiSecretStore.ts`
- Create: `reprodocket/scripts/protect-secret.ps1`
- Create: `reprodocket/scripts/unprotect-secret.ps1`
- Test: `reprodocket/tests/integration/WindowsDpapiSecretStore.test.ts`

**Interfaces:**
- `SecretStore.get(): Promise<string | null>`.
- `SecretStore.set(value: string): Promise<void>`.
- `SecretStore.delete(): Promise<void>`.

- [ ] **Step 1: Write failing E001 through E007 tests**

Use a fake test key. Require round trip under the current Windows user, absence of plaintext in stored protected bytes, absence of plaintext from spawn arguments, safe invalid-payload handling, and deterministic missing-value behavior.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- WindowsDpapiSecretStore.test.ts
```

- [ ] **Step 3: Implement the PowerShell protect helper**

Read the secret from standard input only. Use current-user Windows protected-data APIs. Return only the protected representation to stdout. Diagnostics go to stderr and never echo the secret.

- [ ] **Step 4: Implement the unprotect helper**

Read protected data through stdin and return plaintext only to the owning Node process on stdout during a deliberate decrypt operation. Do not accept the secret or protected payload on the command line.

- [ ] **Step 5: Implement the Node adapter**

Spawn `pwsh -NoProfile -File <helper>` with piped stdin/stdout/stderr. Store only protected bytes beneath the application-owned secret root. Make set/delete atomic where practical.

- [ ] **Step 6: Prove GREEN**

```powershell
npm run test:integration -- WindowsDpapiSecretStore.test.ts
```

Inspect the constructed child-process arguments and generated test storage to prove the fake plaintext value is absent.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/security reprodocket/scripts/protect-secret.ps1 reprodocket/scripts/unprotect-secret.ps1 reprodocket/tests/integration/WindowsDpapiSecretStore.test.ts
git diff --cached --check
git commit -m "feat: protect Solari credentials on Windows"
```

---

## Task 10: Add provider state, credential management, and real readiness semantics

**Files:**
- Create: `reprodocket/src/solari/SolariCredentialProvider.ts`
- Create: `reprodocket/src/solari/SolariReadinessProbe.ts`
- Create: `reprodocket/src/server/routes/solariProvider.ts`
- Create: `reprodocket/src/client/components/SolariConnection.tsx`
- Modify: `reprodocket/src/client/App.tsx`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/integration/SolariProviderRoutes.test.ts`
- Test: `reprodocket/tests/ui/SolariConnection.spec.ts`
- Test: `reprodocket/tests/live/SolariCredentialProbe.live.test.ts`

**Interfaces:**
- Implements `GET /api/provider/solari`, `PUT /api/provider/solari/credential`, and `DELETE /api/provider/solari/credential`.
- Credential precedence is `Environment -> Protected local store -> None`.

- [ ] **Step 1: Write failing provider-route tests**

With a fake readiness-probe dependency, require environment credentials to remain authoritative, failed submitted credentials not to become READY or persist as valid, successful protected credentials to report only their source, and no API response to contain the key.

- [ ] **Step 2: Write failing H001 through H003 UI tests**

Require first-launch connection state, sanitized invalid-key failure, configured READY state, and a working change/remove path only when the protected local store is the active credential source.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- SolariProviderRoutes.test.ts
npm run build
npm run test:ui -- SolariConnection.spec.ts
```

- [ ] **Step 4: Implement credential source precedence**

Read `SOLARI_API_KEY` without logging it, otherwise use the protected store, and return source metadata separately from the secret.

- [ ] **Step 5: Implement the readiness probe abstraction**

Use the least expensive authoritative real Solari operation supported by the installed package. If only a short-lived browser proves credential validity, own it explicitly and close it in `finally`.

- [ ] **Step 6: Implement guarded credential routes**

Persist a submitted protected-store credential only after successful verification. DELETE removes only the protected-store value. Never mutate process environment.

- [ ] **Step 7: Implement connection UI**

Use a password input appropriate for an API key. Do not store the typed value in localStorage/sessionStorage. Clear component secret state after the request settles. Normal UI copy follows `docs/reprodocket-ui-spec.md` rather than exposing implementation jargon.

- [ ] **Step 8: Prove local GREEN**

```powershell
npm run test:integration -- SolariProviderRoutes.test.ts
npm run build
npm run test:ui -- SolariConnection.spec.ts
npm run typecheck
```

- [ ] **Step 9: Prove the live credential boundary when an authorized key is available**

```powershell
npm run test:live -- SolariCredentialProbe.live.test.ts
```

No credential available means the live authority is BLOCKED, not PASS. With a credential, the test requires successful readiness and cleanup of any created resource.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/src/solari/SolariCredentialProvider.ts reprodocket/src/solari/SolariReadinessProbe.ts reprodocket/src/server/routes/solariProvider.ts reprodocket/src/client/components/SolariConnection.tsx reprodocket/src/client/App.tsx reprodocket/src/server/createServer.ts reprodocket/tests/integration/SolariProviderRoutes.test.ts reprodocket/tests/ui/SolariConnection.spec.ts reprodocket/tests/live/SolariCredentialProbe.live.test.ts
git diff --cached --check
git commit -m "feat: add zero-config Solari connection flow"
```

---

## Task 11: Establish the first real validation entry point

**Files:**
- Create: `reprodocket/scripts/validate.ps1`
- Create: `reprodocket/src/validation/ValidationReport.ts`
- Create: `reprodocket/tests/integration/ValidationScript.test.ts`
- Create: `reprodocket/docs/validation.md`

**Interfaces:**
- `.\reprodocket\scripts\validate.ps1 -Profile Targeted|Full`.
- Omitting `-Profile` means Full.
- Full cannot return overall PASS while later mandatory Solari/E2E authorities are unimplemented or BLOCKED.

- [ ] **Step 1: Write failing validation-runner tests**

Prove a required child stage returning nonzero makes the wrapper return nonzero and preserves the failed stage name. Prove a mandatory BLOCKED Full authority prevents an overall PASS.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- ValidationScript.test.ts
```

- [ ] **Step 3: Implement deterministic local stages**

Initial stages:

```text
preflight-local
lockfile/dependency restore check
format-check
lint
typecheck
unit
integration
build
ui
secret/public-path scan
repository hygiene
```

Stop before billable external work when a cheaper mandatory prerequisite fails.

- [ ] **Step 4: Emit machine and human validation summaries**

Write ignored output beneath `reprodocket/.artifacts/validation/<run-id>/`, including `validation.json` and `validation.txt`. Record source revision, profile, timestamps, stage names, results, exit codes, and explicit blocked authorities without including credentials or private absolute paths in any artifact intended for publication.

- [ ] **Step 5: Run current Targeted validation**

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
```

Expected: only implemented M0/M1 authorities may pass. Plain Full at this phase must remain non-PASS because the later mandatory live browser/sandbox/E2E authorities are not complete.

- [ ] **Step 6: Inspect current local UI evidence**

Capture original-resolution connection/shell views from the exact built source. Repair objective clipping, unreadable text, broken focus, raw errors, or misleading controls before closing the milestone.

- [ ] **Step 7: Document validation semantics**

`reprodocket/docs/validation.md` explains Targeted versus Full, where ignored evidence is written, when live resources are created, and why BLOCKED is distinct from FAIL and PASS.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/scripts/validate.ps1 reprodocket/src/validation/ValidationReport.ts reprodocket/tests/integration/ValidationScript.test.ts reprodocket/docs/validation.md
git diff --cached --check
git commit -m "test: establish ReproDocket validation authority"
```

---

## Plan 1 Completion Gate

Do not begin Plan 2 until fresh evidence from one source revision establishes all applicable Plan 1 requirements:

```text
current Node/npm/PowerShell/Git state recorded
current direct dependency versions installed and lockfile tracked
unified action + expectation parser red-green proven
complete plan request validation rejects missing action/expectation
unsafe target literals/schemes rejected
run schemas/lifecycle/classifier executable
VERIFIED session/evidence guards mutation-proven
application-owned storage and damaged-run isolation work
loopback/request security tests pass
built React shell is exercised through Playwright
preferred-port collision leaves unrelated process untouched
run.ps1 starts/reuses one owned instance
stop.ps1 refuses stale/unowned process identity
protected credential round trip works on Windows
normal user can configure/change/remove protected Solari credential without file editing
provider state never exposes the key
Targeted validation exits correctly and emits current evidence
Full remains truthfully non-PASS until later mandatory authorities exist
```

Run:

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
git status --short
git log --oneline --decorate -12
git diff main...HEAD --check
```

Plan 1 is complete only if the targeted validation command is freshly green, the worktree has no unexpected generated output, the current diff is clean, and the next blocker is Plan 2 rather than an unresolved Plan 1 defect.