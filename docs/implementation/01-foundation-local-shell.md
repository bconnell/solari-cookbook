# ReproDocket Foundation and Local Shell Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish ReproDocket's reproducible TypeScript project, real test harness, safe input contracts, loopback server, local UI shell, Windows bootstrap, process ownership, and zero-file-edit Solari credential flow.

**Architecture:** One Node process serves a built React application and JSON/SSE API on loopback. Core validation and persistence remain framework-independent. PowerShell scripts own Windows bootstrap/start/stop behavior, while a protected secret-store adapter uses PowerShell/.NET only at the narrow operating-system credential boundary.

**Tech Stack:** Node.js 22+, TypeScript, Fastify, React, Vite, Zod, Vitest, Playwright Test, PowerShell 7, Windows DPAPI, npm lockfile.

**Spec:** `docs/reprodocket-design.md`; contracts: `docs/reprodocket-interface-contracts.md`; security: `docs/reprodocket-security-lifecycle.md`; tests: `docs/reprodocket-test-matrix.md`; SDK baseline: `docs/reprodocket-sdk-baseline.md`.

## Global Constraints

* Work on `feature/reprodocket` or an isolated worktree based on it. Re-read branch/HEAD/dirty state before editing.
* Preserve the upstream `examples/` tree and `LICENSE`.
* No destructive Git cleanup, reset, stash, rebase, or force push.
* New Windows automation must not use WMIC.
* Normal product data must not be written into the Git repository.
* The local service binds to loopback only and rejects cross-origin state mutation.
* The normal user must not create `.env` files, databases, or manual frontend/backend configuration.
* `reprodocket/package-lock.json` must be committed after dependency installation.
* Tests must assert real behavior. No test may exist solely to increase a count.
* A task is not complete until its failing test was observed before the fix, its affected suite passes after the fix, and repository state is reviewed.
* Use current installed Solari SDK type declarations as authority when they differ from historical examples.
* Do not add a second runtime AI credential in this plan.

---

## Task 1: Create the project and establish the first real test harness

**Files:**
- Create: `reprodocket/package.json`
- Create: `reprodocket/tsconfig.json`
- Create: `reprodocket/tsconfig.server.json`
- Create: `reprodocket/vite.config.ts`
- Create: `reprodocket/vitest.config.ts`
- Create: `reprodocket/playwright.config.ts`
- Create: `reprodocket/eslint.config.js`
- Create: `reprodocket/.prettierrc.json`
- Create: `reprodocket/index.html`
- Create: `reprodocket/src/shared/contracts/ReproductionAction.ts`
- Create: `reprodocket/src/core/ReproductionStepParser.ts`
- Test: `reprodocket/tests/unit/ReproductionStepParser.test.ts`

**Interfaces:**
- Produces: `parseReproductionSteps(lines: string[]): ParsedReproductionStep[]`
- Produces: the `ReproductionAction` and `ParsedReproductionStep` types defined in `docs/reprodocket-interface-contracts.md`.

- [ ] **Step 1: Inspect the current package registry and installed Node version before writing package metadata**

Run:

```powershell
node --version
npm --version
npm view @solarisdk/browser version
npm view @solarisdk/sandbox version
npm view fastify version
npm view @fastify/static version
npm view react version
npm view react-dom version
npm view zod version
npm view vite version
npm view vitest version
npm view @playwright/test version
npm view typescript version
```

Expected: each command returns a concrete version. If the Solari versions differ from `docs/reprodocket-sdk-baseline.md`, update that document before proceeding.

- [ ] **Step 2: Create `package.json` with exact direct dependencies**

Use this package/script shape, substituting only newer current package versions that were explicitly verified in Step 1 and keeping `--save-exact` behavior:

```json
{
  "name": "reprodocket",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=22"
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
  },
  "dependencies": {
    "@fastify/static": "10.1.3",
    "@solarisdk/browser": "0.1.1",
    "@solarisdk/sandbox": "0.1.2",
    "fastify": "5.12.1",
    "react": "19.2.8",
    "react-dom": "19.2.8",
    "zod": "4.5.4"
  },
  "devDependencies": {
    "@playwright/test": "1.62.1",
    "@types/node": "24",
    "@types/react": "19.2.18",
    "@types/react-dom": "19.2.5",
    "@vitejs/plugin-react": "6.1.1",
    "eslint": "10.9.0",
    "prettier": "3.9.6",
    "typescript": "7.0.2",
    "typescript-eslint": "8.68.0",
    "vite": "8.2.2",
    "vitest": "4.1.11"
  }
}
```

If `npm view eslint version` proves a different current stable version, use that exact verified version instead of assuming `10.9.0`.

- [ ] **Step 3: Install dependencies and create the lockfile**

Run:

```powershell
Set-Location reprodocket
npm install --save-exact
npm ls --depth=0
```

Expected: install succeeds, `package-lock.json` exists, direct dependency versions are explicit, and `npm ls --depth=0` has no missing peer/dependency errors.

- [ ] **Step 4: Create strict TypeScript configuration**

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

`tsconfig.server.json` extends it, switches to Node-compatible module resolution as required by the actual TypeScript/Node combination, excludes `src/client`, and emits under `dist/server` with `rootDir` set to `src`.

- [ ] **Step 5: Write the first failing parser tests**

Create tests covering at least:

```ts
import { describe, expect, it } from "vitest";
import { parseReproductionSteps } from "../../src/core/ReproductionStepParser.js";

describe("parseReproductionSteps", () => {
  it("parses the supported action grammar without changing quoted case", () => {
    expect(
      parseReproductionSteps([
        'OPEN "/account"',
        'CLICK "Account"',
        'FILL "Email" WITH "QA.User@example.com"',
        'SELECT "Country" VALUE "US"',
        'CHECK "I agree"',
        'UNCHECK "Subscribe"',
        'PRESS "Enter"',
        'WAIT_FOR_TEXT "Saved"',
        'WAIT_FOR_URL "/account"',
        "RELOAD",
        "BACK",
        "FORWARD"
      ])
    ).toHaveLength(12);
  });

  it("rejects arbitrary script or selector execution", () => {
    expect(() => parseReproductionSteps(['EVALUATE "alert(1)"'])).toThrow();
    expect(() => parseReproductionSteps(['CLICK_CSS "#danger"'])).toThrow();
  });

  it("reports the one-based failing step index", () => {
    expect(() => parseReproductionSteps(['CLICK "Save"', "BROKEN"])).toThrow(/step 2/i);
  });
});
```

- [ ] **Step 6: Run the parser test and prove RED**

Run:

```powershell
npm run test:unit -- ReproductionStepParser.test.ts
```

Expected: FAIL because the parser implementation does not yet exist.

- [ ] **Step 7: Implement the minimal parser and action types**

Implement the union from `docs/reprodocket-interface-contracts.md`. Use anchored regular expressions. Reject empty quoted labels, unrecognized commands, and trailing garbage. Preserve original source lines in `ParsedReproductionStep.source`.

Do not use `eval`, `Function`, arbitrary selectors, or arbitrary JavaScript.

- [ ] **Step 8: Run parser tests and static checks**

Run:

```powershell
npm run test:unit -- ReproductionStepParser.test.ts
npm run typecheck
npm run lint
npm run format:check
```

Expected: all exit zero.

- [ ] **Step 9: Verify lockfile tracking**

Run from repository root:

```powershell
git check-ignore -v reprodocket/package-lock.json
if ($LASTEXITCODE -eq 0) { throw "ReproDocket lockfile is still ignored." }
git status --short
```

Expected: lockfile is not ignored; generated `node_modules` is not listed.

- [ ] **Step 10: Commit the validated foundation**

```powershell
git add reprodocket/package.json reprodocket/package-lock.json reprodocket/tsconfig.json reprodocket/tsconfig.server.json reprodocket/vite.config.ts reprodocket/vitest.config.ts reprodocket/playwright.config.ts reprodocket/eslint.config.js reprodocket/.prettierrc.json reprodocket/index.html reprodocket/src/shared/contracts/ReproductionAction.ts reprodocket/src/core/ReproductionStepParser.ts reprodocket/tests/unit/ReproductionStepParser.test.ts
git diff --cached --check
git commit -m "build: establish ReproDocket TypeScript foundation"
```

---

## Task 2: Implement target URL validation and network blocking

**Files:**
- Create: `reprodocket/src/core/TargetUrlPolicy.ts`
- Create: `reprodocket/src/shared/errors/ReproDocketError.ts`
- Test: `reprodocket/tests/unit/TargetUrlPolicy.test.ts`

**Interfaces:**
- Produces: `validateTargetUrl(input: string): URL`
- Produces: `ReproDocketError` carrying a stable error code.

- [ ] **Step 1: Write the failing URL-policy tests**

Cover C001-C022 from `docs/reprodocket-test-matrix.md`, including public HTTPS success, file/data/javascript rejection, credential-in-URL rejection, localhost, RFC1918 IPv4, IPv6 loopback/link-local/unique-local, and `169.254.169.254`.

Representative assertions:

```ts
expect(validateTargetUrl("https://example.com/path").href).toBe("https://example.com/path");
expect(() => validateTargetUrl("javascript:alert(1)")).toThrowError(/INVALID_TARGET_URL/);
expect(() => validateTargetUrl("http://127.0.0.1")).toThrowError(/BLOCKED_TARGET_NETWORK/);
expect(() => validateTargetUrl("http://169.254.169.254/latest/meta-data")).toThrowError(/BLOCKED_TARGET_NETWORK/);
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- TargetUrlPolicy.test.ts
```

Expected: FAIL because `TargetUrlPolicy` does not exist.

- [ ] **Step 3: Implement scheme, hostname, and literal-address rules**

Use `URL` for parsing. Do not attempt to recognize IP ranges with substring matching. Normalize bracketed IPv6 through a dedicated parser/helper and unit-test every blocked range.

Do not perform DNS resolution in this pure function. DNS/redirect revalidation belongs to the browser-navigation adapter where authoritative resolved/final destination data can be obtained.

- [ ] **Step 4: Run the affected unit suite**

```powershell
npm run test:unit -- TargetUrlPolicy.test.ts ReproductionStepParser.test.ts
npm run typecheck
```

Expected: PASS.

- [ ] **Step 5: Mutation proof one critical guard**

Temporarily disable the private IPv4 rejection and rerun the URL test. Confirm at least one test fails. Restore the guard and confirm PASS.

Record the command/result in the task notes; do not commit the deliberate regression.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/core/TargetUrlPolicy.ts reprodocket/src/shared/errors/ReproDocketError.ts reprodocket/tests/unit/TargetUrlPolicy.test.ts
git diff --cached --check
git commit -m "feat: enforce safe ReproDocket target URLs"
```

---

## Task 3: Define versioned run schemas and lifecycle invariants

**Files:**
- Create: `reprodocket/src/shared/contracts/RunModels.ts`
- Create: `reprodocket/src/core/RunLifecycleMachine.ts`
- Create: `reprodocket/src/core/OutcomeClassifier.ts`
- Test: `reprodocket/tests/unit/RunLifecycleMachine.test.ts`
- Test: `reprodocket/tests/unit/OutcomeClassifier.test.ts`

**Interfaces:**
- Produces the run/attempt/evidence/resource types in `docs/reprodocket-interface-contracts.md`.
- Produces `transitionLifecycle(current, next): RunLifecycle`.
- Produces `classify(investigation, verification): RunOutcome`.

- [ ] **Step 1: Write lifecycle tests B001-B010**

Test every legal transition and representative illegal transitions. Terminal states must reject a transition back to active execution.

- [ ] **Step 2: Write classifier tests B011-B019**

Include a helper that creates complete attempt records with explicit evidence IDs. A `VERIFIED` assertion must require different non-null Solari session IDs and reproduction evidence for both attempts.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- RunLifecycleMachine.test.ts OutcomeClassifier.test.ts
```

Expected: FAIL because lifecycle/classifier implementations do not exist.

- [ ] **Step 4: Implement Zod schemas and TypeScript types**

Create a `RunManifestSchemaV1` matching the documented manifest contract. Export inferred types from the schema where practical so persistence validation and TypeScript cannot drift independently.

- [ ] **Step 5: Implement lifecycle transitions**

Use an explicit allowed-transition map. Do not infer legality by string ordering.

- [ ] **Step 6: Implement the classifier**

Implement the decision table from the contracts document. Before returning `VERIFIED`, assert distinct non-null session IDs and nonempty evidence for both attempts. Invariant violation throws a stable internal error; it never silently returns VERIFIED.

- [ ] **Step 7: Prove GREEN and run typecheck**

```powershell
npm run test:unit -- RunLifecycleMachine.test.ts OutcomeClassifier.test.ts
npm run typecheck
```

- [ ] **Step 8: Mutation proof VERIFIED guard**

Temporarily remove the distinct-session check. Rerun `OutcomeClassifier.test.ts`; confirm failure. Restore and rerun PASS.

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/shared/contracts/RunModels.ts reprodocket/src/core/RunLifecycleMachine.ts reprodocket/src/core/OutcomeClassifier.ts reprodocket/tests/unit/RunLifecycleMachine.test.ts reprodocket/tests/unit/OutcomeClassifier.test.ts
git diff --cached --check
git commit -m "feat: define run lifecycle and outcome truth"
```

---

## Task 4: Establish application-owned filesystem roots and basic run persistence

**Files:**
- Create: `reprodocket/src/storage/AppPaths.ts`
- Create: `reprodocket/src/storage/AtomicJsonFile.ts`
- Create: `reprodocket/src/storage/FileRunStore.ts`
- Test: `reprodocket/tests/unit/AppPaths.test.ts`
- Test: `reprodocket/tests/integration/FileRunStore.test.ts`

**Interfaces:**
- Produces `resolveAppPaths(overrides?): AppPaths`.
- Produces the initial `RunStore` methods `create`, `get`, `list`, `update`.

- [ ] **Step 1: Write path tests**

Use a test-only explicit root under the test artifact directory. Assert run paths derive only from generated IDs, reject traversal-like IDs, and never default to Desktop/Documents/Downloads/source tree.

- [ ] **Step 2: Write persistence tests F001-F005 and F012-F014**

Use temporary directories created by the test. Verify unique run creation, fail-if-exists behavior, legal update, malformed-run isolation, and history survival when one run is damaged.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- AppPaths.test.ts
npm run test:integration -- FileRunStore.test.ts
```

- [ ] **Step 4: Implement application paths**

On Windows, default persistent root is based on `LOCALAPPDATA` and ends in `ReproDocket`. If unavailable, fail with a clear application-path error rather than silently writing into the repository.

- [ ] **Step 5: Implement atomic JSON write**

Write sibling temporary JSON, close the file, then rename/replace within the same directory. Parse and validate with `RunManifestSchemaV1` before returning the new authority.

- [ ] **Step 6: Implement basic run store**

`create` generates/receives the server-generated ID, uses exclusive directory creation, writes CREATED state, and returns the persisted result. `list` validates each manifest independently and returns a damaged projection for malformed historical runs through a separate type rather than crashing the entire list.

- [ ] **Step 7: Run tests**

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

## Task 5: Build the loopback Fastify shell and local request guard

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
- Produces `GET /api/health` and `GET /api/bootstrap`.
- Produces header contract `X-ReproDocket-Request` for mutations.

- [ ] **Step 1: Write failing security tests D001-D011**

Use Fastify injection for Host/Origin/nonce behavior where possible and an actual ephemeral loopback listener for the bind-address assertion.

Representative mutation probe:

```ts
const response = await app.inject({
  method: "POST",
  url: "/api/test-mutation",
  headers: {
    host: expectedHost,
    origin: "https://evil.example"
  }
});
expect(response.statusCode).toBe(403);
```

- [ ] **Step 2: Write failing health/bootstrap tests**

Assert `/api/health` returns only status, instance ID, and version. Assert bootstrap returns a nonempty per-process request nonce and never contains local absolute paths or a field named `apiKey`.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- LocalServerSecurity.test.ts HealthRoutes.test.ts
```

- [ ] **Step 4: Implement server creation with no CORS plugin**

Bind address is passed explicitly and production startup uses `127.0.0.1`. Add a request hook that validates Host and, for mutations, same-origin plus request nonce.

- [ ] **Step 5: Add response headers**

Set the CSP and security header baseline from `docs/reprodocket-security-lifecycle.md`. Do not use a permissive wildcard CSP.

- [ ] **Step 6: Implement health/bootstrap routes**

Generate instance ID and request nonce with Node `crypto.randomUUID()` / `randomBytes`. Keep the nonce process-local.

- [ ] **Step 7: Run integration/static checks**

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

## Task 6: Create the built React shell and serve it from the same process

**Files:**
- Create: `reprodocket/src/client/main.tsx`
- Create: `reprodocket/src/client/App.tsx`
- Create: `reprodocket/src/client/styles.css`
- Create: `reprodocket/src/client/api/ApiClient.ts`
- Create: `reprodocket/src/server/static/registerStaticUi.ts`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/ui/AppShell.spec.ts`

**Interfaces:**
- Produces built route `/`.
- Client bootstraps through `GET /api/bootstrap` and stores the request nonce only in runtime memory.

- [ ] **Step 1: Write failing Playwright shell test**

The test starts the built server through Playwright `webServer`, visits `/`, and requires:

```ts
await expect(page.getByRole("heading", { name: "ReproDocket" })).toBeVisible();
await expect(page.getByLabel("Target URL")).toBeVisible();
await expect(page.getByLabel("Problem description")).toBeVisible();
await expect(page.getByLabel("Reproduction steps")).toBeVisible();
```

Do not test exact decorative copy unless it is a requirement.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- AppShell.spec.ts
```

Expected: test fails because UI/static serving is absent.

- [ ] **Step 3: Implement the minimal accessible shell**

Use native labels/textarea/input/button. Reproduction steps are one line per step. Display a concise syntax reference directly beside the field or behind a working disclosure control.

- [ ] **Step 4: Serve built assets with `@fastify/static`**

Configure `dist/client` as the only static root. Add SPA fallback for application routes without intercepting `/api/*`.

- [ ] **Step 5: Connect bootstrap**

On application load, fetch `/api/bootstrap`. Show a genuine loading/error state. Store nonce in memory and pass it to `ApiClient` mutation calls.

- [ ] **Step 6: Run built user-boundary test**

```powershell
npm run build
npm run test:ui -- AppShell.spec.ts
npm run typecheck
```

- [ ] **Step 7: Capture a temporary development screenshot for inspection**

Use Playwright's page screenshot into `reprodocket/.artifacts/targeted/`. Inspect at original resolution for clipping, unreadable text, or debug content. Do not commit the screenshot.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/client reprodocket/src/server/static reprodocket/src/server/createServer.ts reprodocket/tests/ui/AppShell.spec.ts
git diff --cached --check
git commit -m "feat: serve the ReproDocket local interface"
```

---

## Task 7: Implement owned-instance metadata and deterministic port selection

**Files:**
- Create: `reprodocket/src/server/startup/PortAllocator.ts`
- Create: `reprodocket/src/server/startup/InstanceRecord.ts`
- Modify: `reprodocket/src/server/server.ts`
- Test: `reprodocket/tests/integration/InstanceOwnership.test.ts`
- Test: `reprodocket/tests/integration/PortAllocator.test.ts`

**Interfaces:**
- Produces `findLoopbackPort(preferred: number): Promise<number>`.
- Produces a runtime ownership record with PID, process start identity, port, bind address, application/build identity, nonce, and startedAt.

- [ ] **Step 1: Write failing occupied-port test R013/P009**

Start a test TCP listener on the preferred port, call the allocator, assert it returns another available port, and assert the foreign listener remains alive.

- [ ] **Step 2: Write failing instance-record tests**

Assert the record includes more than PID and can distinguish deliberately mismatched process-start identity/instance nonce.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- InstanceOwnership.test.ts PortAllocator.test.ts
```

- [ ] **Step 4: Implement allocator**

Probe a bounded range beginning at 4317, binding only loopback. If no port is found within the bound, throw `LOCAL_PORT_UNAVAILABLE`. Never kill a process during allocation.

- [ ] **Step 5: Implement ownership record write**

Write under the application-owned runtime path. Use atomic write. Include current process start timestamp from a trustworthy Windows/Node source available to both start and stop validation.

- [ ] **Step 6: Run tests and mutation check**

Temporarily change allocator behavior to reuse an occupied port and confirm `PortAllocator.test.ts` fails. Restore.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/server/startup reprodocket/src/server/server.ts reprodocket/tests/integration/InstanceOwnership.test.ts reprodocket/tests/integration/PortAllocator.test.ts
git diff --cached --check
git commit -m "feat: track local ReproDocket instance ownership"
```

---

## Task 8: Build Windows bootstrap, preflight, run, and stop scripts

**Files:**
- Create: `reprodocket/scripts/bootstrap.ps1`
- Create: `reprodocket/scripts/preflight.ps1`
- Create: `reprodocket/scripts/run.ps1`
- Create: `reprodocket/scripts/stop.ps1`
- Create: `reprodocket/scripts/lib/Common.ps1`
- Test: `reprodocket/tests/integration/PowerShellScripts.test.ts`

**Interfaces:**
- Produces one normal user start command: `.\reprodocket\scripts\run.ps1`.
- Produces one ownership-aware stop command: `.\reprodocket\scripts\stop.ps1`.

- [ ] **Step 1: Write source/runtime tests R001-R020 that are applicable before live Solari**

The test launches PowerShell processes with controlled PATH/environment and checks parser/runtime behavior. Include a source scan that fails on `wmic` or `wmic.exe` tokens in new scripts.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- PowerShellScripts.test.ts
```

- [ ] **Step 3: Implement `Common.ps1`**

Functions must resolve script/repository paths from `$PSScriptRoot`, report errors with nonzero exits, query disk space through .NET/CIM rather than WMIC, and avoid touching user folders.

- [ ] **Step 4: Implement bootstrap**

The bootstrap entry must parse in Windows PowerShell 5.1. It checks for PowerShell 7 and a supported Node runtime. When `winget` is available, it may use explicit package IDs to install missing PowerShell 7/Node LTS, then verifies the installed command before continuing. If safe automatic installation is unavailable, stop with exact instructions rather than downloading an unverified binary.

After the lockfile exists, restore with:

```powershell
npm ci
```

Do not run a broad `npm update`.

- [ ] **Step 5: Implement preflight**

Validate PowerShell, Node, npm, disk reserve, application data write access, lockfile presence, dependency tree, build capability, and local port capability. Solari credential/live connectivity checks are added in later plans.

- [ ] **Step 6: Implement run**

If a valid owned instance record points to a live ReproDocket instance, open its URL and exit successfully. Otherwise build if needed, start the server, poll `/api/health` with a bounded timeout, then open the default browser. If browser opening fails, print the local URL and leave the healthy server running.

- [ ] **Step 7: Implement stop**

Read the ownership record, verify PID plus process-start identity/application identity, request graceful shutdown when implemented, and only use process termination as an owned-process fallback. If identity is stale or mismatched, refuse to terminate.

- [ ] **Step 8: Run script tests under both PowerShell hosts when available**

```powershell
powershell.exe -NoProfile -File .\reprodocket\scripts\preflight.ps1 -LocalOnly
pwsh.exe -NoProfile -File .\reprodocket\scripts\preflight.ps1 -LocalOnly
npm run test:integration -- PowerShellScripts.test.ts
```

Expected: parser/runtime tests pass and no unrelated process is terminated.

- [ ] **Step 9: Manual local shell proof**

Run:

```powershell
.\reprodocket\scripts\run.ps1
```

Confirm one browser tab opens to the health-verified owned instance. Run it a second time and confirm a duplicate server is not created. Then run `stop.ps1` and confirm the owned server stops.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/scripts reprodocket/tests/integration/PowerShellScripts.test.ts
git diff --cached --check
git commit -m "feat: automate ReproDocket Windows startup"
```

---

## Task 9: Implement the Windows protected secret store

**Files:**
- Create: `reprodocket/src/security/SecretStore.ts`
- Create: `reprodocket/src/security/WindowsDpapiSecretStore.ts`
- Create: `reprodocket/scripts/protect-secret.ps1`
- Create: `reprodocket/scripts/unprotect-secret.ps1`
- Test: `reprodocket/tests/integration/WindowsDpapiSecretStore.test.ts`

**Interfaces:**
- Produces `SecretStore.get(): Promise<string | null>`.
- Produces `SecretStore.set(value: string): Promise<void>`.
- Produces `SecretStore.delete(): Promise<void>`.

- [ ] **Step 1: Write failing E001-E007 tests**

Use a fake value such as `slr_live_TEST_DO_NOT_USE_4d9f0b8f`. Verify protected bytes do not contain that plaintext and round-trip returns the exact value for the current Windows user.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- WindowsDpapiSecretStore.test.ts
```

- [ ] **Step 3: Implement PowerShell protect helper**

Read all secret text from standard input. Use .NET protected-data APIs with current-user scope. Write only base64 protected bytes to stdout. Diagnostics go to stderr and never echo the input.

- [ ] **Step 4: Implement unprotect helper**

Read protected base64 from stdin and return plaintext only to stdout. Do not accept the secret or protected value as a command-line argument.

- [ ] **Step 5: Implement Node adapter**

Spawn `pwsh -NoProfile -File <helper>` with piped stdin/stdout/stderr. Store protected bytes in the application-owned `secrets` directory. Set/delete operations are atomic where practical.

- [ ] **Step 6: Prove GREEN and inspect process command line behavior**

Run the integration test and inspect that the fake key value never appears in the constructed spawn argument array.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/security reprodocket/scripts/protect-secret.ps1 reprodocket/scripts/unprotect-secret.ps1 reprodocket/tests/integration/WindowsDpapiSecretStore.test.ts
git diff --cached --check
git commit -m "feat: protect Solari credentials on Windows"
```

---

## Task 10: Add provider status, credential management, and real readiness boundary

**Files:**
- Create: `reprodocket/src/solari/SolariCredentialProvider.ts`
- Create: `reprodocket/src/solari/SolariReadinessProbe.ts`
- Create: `reprodocket/src/server/routes/solariProvider.ts`
- Create: `reprodocket/src/client/components/SolariConnection.tsx`
- Modify: `reprodocket/src/client/App.tsx`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/integration/SolariProviderRoutes.test.ts`
- Test: `reprodocket/tests/ui/SolariConnection.spec.ts`

**Interfaces:**
- Produces GET/PUT/DELETE `/api/provider/solari[/credential]` contracts.
- Produces deterministic credential precedence: Environment -> Protected local store -> None.

- [ ] **Step 1: Write failing route tests**

Use a fake readiness-probe dependency for non-live integration tests. Assert environment credentials cannot be overwritten by the UI, failed verification is not persisted as READY, and successful protected-store credential returns source `Protected local store` without returning the key.

- [ ] **Step 2: Write failing UI tests H001-H003**

Assert no-key state asks for the Solari key; a failed save remains visibly unready; a successful configured state exposes the New Investigation form and a working Change/Remove credential path when the source is the protected store.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- SolariProviderRoutes.test.ts
npm run build
npm run test:ui -- SolariConnection.spec.ts
```

- [ ] **Step 4: Implement credential source precedence**

Read `SOLARI_API_KEY` without logging it. Otherwise query the protected store. Return source metadata separately from the secret.

- [ ] **Step 5: Implement readiness-probe abstraction**

Define a method that validates a candidate credential against the real Solari boundary. The concrete live implementation is initially allowed to create a minimal short-lived browser if the installed SDK has no cheaper authoritative key-verification endpoint, but it must close it in `finally`.

- [ ] **Step 6: Implement provider routes behind mutation guard**

Only persist a submitted key after a successful probe. DELETE removes only the protected store. Never mutate process environment.

- [ ] **Step 7: Implement connection UI**

Use a password input with autocomplete disabled appropriately for an API credential. Do not retain the typed key in localStorage/sessionStorage. Clear component state after the request settles.

- [ ] **Step 8: Run local tests**

```powershell
npm run test:integration -- SolariProviderRoutes.test.ts
npm run build
npm run test:ui -- SolariConnection.spec.ts
npm run typecheck
```

- [ ] **Step 9: Add the live credential probe test only after Solari is available on the laptop**

Create `tests/live/SolariCredentialProbe.live.test.ts`. It must skip with an explicit BLOCKED reason when no authorized credential exists; it must not synthesize PASS. With a real credential, require the probe to succeed and any resource it creates to close.

- [ ] **Step 10: Commit**

```powershell
git add reprodocket/src/solari/SolariCredentialProvider.ts reprodocket/src/solari/SolariReadinessProbe.ts reprodocket/src/server/routes/solariProvider.ts reprodocket/src/client/components/SolariConnection.tsx reprodocket/src/client/App.tsx reprodocket/src/server/createServer.ts reprodocket/tests/integration/SolariProviderRoutes.test.ts reprodocket/tests/ui/SolariConnection.spec.ts reprodocket/tests/live/SolariCredentialProbe.live.test.ts
git diff --cached --check
git commit -m "feat: add zero-config Solari connection flow"
```

---

## Task 11: Close M0/M1 with a real local validation entry point

**Files:**
- Create: `reprodocket/scripts/validate.ps1`
- Create: `reprodocket/src/validation/ValidationReport.ts`
- Create: `reprodocket/tests/integration/ValidationScript.test.ts`
- Create: `reprodocket/docs/validation.md`

**Interfaces:**
- Produces `.\reprodocket\scripts\validate.ps1`.
- Produces `-Profile Targeted|Full` with no profile meaning Full.
- At this phase Full truthfully reports later Solari/E2E authorities as BLOCKED until their plans are implemented; it must not return overall PASS while mandatory authorities are absent.

- [ ] **Step 1: Write failing R017/R018 tests around a controlled validation command runner**

Prove that a required child stage returning nonzero makes `validate.ps1` return nonzero and that the summary retains the failed stage name.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- ValidationScript.test.ts
```

- [ ] **Step 3: Implement deterministic stage runner**

Initial local stages:

```text
preflight-local
npm-ci-check
format-check
lint
typecheck
unit
integration
build
ui
secret-scan
repository-hygiene
```

Do not continue to billable live stages after a prerequisite deterministic stage fails.

- [ ] **Step 4: Emit machine and human summaries**

Write ignored artifacts under `reprodocket/.artifacts/validation/<run-id>/` containing `validation.json` and `validation.txt`. Include source revision, profile, started/finished timestamps, stages, exit codes, and explicit BLOCKED authorities.

- [ ] **Step 5: Run current local validation**

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
```

Expected: current implemented M0/M1 local authorities PASS. A Full profile at this phase must explicitly fail/block on not-yet-implemented mandatory live/E2E authorities rather than claiming project completion.

- [ ] **Step 6: Inspect the local UI at normal and enlarged scale**

Use the exact built server from the validation source. Capture original-resolution New Investigation/connection views. Resolve clipping, inaccessible controls, raw stack traces, or debug text before closing M1.

- [ ] **Step 7: Update `reprodocket/docs/validation.md`**

Document what Targeted and Full mean, which live resources they create, where ignored evidence is written, and how BLOCKED differs from FAIL.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/scripts/validate.ps1 reprodocket/src/validation/ValidationReport.ts reprodocket/tests/integration/ValidationScript.test.ts reprodocket/docs/validation.md
git diff --cached --check
git commit -m "test: establish ReproDocket validation authority"
```

---

## Plan 1 completion gate

Do not begin Plan 2 until all of the following are current for the same source revision:

```text
ReproDocket installs from the lockfile
real parser tests are red-green proven
unsafe targets are rejected
run schemas and lifecycle rules are executable
run state persists safely
loopback server security tests pass
built React shell is exercised through Playwright
preferred-port collision leaves the foreign process untouched
run.ps1 starts/reuses one owned local process
stop.ps1 refuses stale/unowned processes
protected credential round-trip works on Windows
normal user can configure/replace/remove the protected Solari key without editing a file
provider readiness never leaks the key
Targeted validation exits correctly and produces current evidence
Full validation remains truthfully blocked on later mandatory authorities
```

Run:

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
```

Then inspect:

```powershell
git status --short
git log --oneline --decorate -12
git diff main...HEAD --check
```

The plan is complete only if the targeted command is freshly green, the worktree contains no unexpected generated output, and the exact next blocker is Plan 2 rather than an unresolved Plan 1 defect.