# ReproDocket Toolchain Baseline

Date: August 31, 2026
Status: Pre-implementation environment contract

## 1. Purpose

This document freezes the minimum environment rules that can be established before local execution. Exact patch versions are verified again on the implementation machine because package registries and installed tools can change.

## 2. Node.js

Compatibility floor:

```text
Node.js >= 22.12.0
```

Reason: the selected Vite 8 toolchain requires Node 20.19+ or 22.12+ on the corresponding release lines. `>=22` is therefore too broad.

Fresh bootstrap preference:

```text
install the current supported Node.js LTS release
```

At the time this baseline was written, Node.js 24.20.0 is the current LTS release. Bootstrap must not hard-code that patch forever. It should discover/install the current LTS through the supported package manager and then verify that the resulting runtime satisfies ReproDocket's actual package engines.

If the machine already has a compatible Node 22.12+ or later supported LTS runtime and the full dependency restore/build succeeds, bootstrap does not upgrade merely for novelty.

## 3. PowerShell

Substantive Windows automation targets supported PowerShell 7.

At the time of this baseline, PowerShell 7.6 is the current LTS line and 7.6.5 is the latest stable release surfaced by Microsoft's release channel.

A small bootstrap entry may remain Windows PowerShell 5.1 compatible so a clean Windows machine can discover/install/reroute to PowerShell 7.

Do not hard-code a private installer path. Prefer an available supported system package manager such as `winget`, verify the installed `pwsh`, and emit a clear manual boundary only when elevation/package-manager policy prevents automatic installation.

New scripts do not use WMIC.

## 4. npm

Use the npm version supplied with the selected Node installation unless package compatibility proves a reason to change it.

Before project creation record:

```powershell
node --version
npm --version
```

After `package-lock.json` exists, deterministic restore uses:

```powershell
npm ci
```

Normal implementation does not routinely regenerate the lockfile merely because a newer transitive dependency exists.

## 5. ReproDocket package version policy

Before creating `package.json`, run `npm view <package> version` for every direct package named by Plan 1 and record the resolved versions in task evidence.

Direct dependencies use exact versions in `package.json`; `package-lock.json` is committed.

At the time this planning audit ran, public package sources indicated current releases including:

```text
Fastify 5.12.1
@fastify/static 10.1.3
React 19.2.8
Zod 4.5.4
Vite 8.2.2
Vitest 4.1.11
@playwright/test 1.62.1
Prettier 3.9.6
TypeScript 7.0.2
```

These are planning-time observations, not commands to ignore the execution-time registry check. The Vite React plugin and ESLint ecosystem must be verified as a compatible set on the implementation machine before they are locked.

Do not select a newer major merely because it exists if its surrounding plugin/type ecosystem is not compatible. The goal is a current, stable, reproducible stack, not maximum version numbers.

## 6. Solari packages

Current Solari documentation separates browser and sandbox packages:

```text
@solarisdk/browser
@solarisdk/sandbox
```

The current cookbook browser example uses `@solarisdk/browser`. An older cookbook sandbox example uses a unified SDK package, while current docs recommend the dedicated sandbox package.

Before implementation:

```powershell
npm view @solarisdk/browser version
npm view @solarisdk/sandbox version
```

Then inspect installed type declarations and run a minimal live contract probe before depending on any uncertain method signature.

See `docs/reprodocket-sdk-baseline.md` for the API authority rules.

## 7. Local Playwright use

`@playwright/test` belongs to the ReproDocket local UI test harness.

It is not required to drive Solari's cloud browser through a separately installed local Playwright runtime; the Solari browser SDK exposes a Playwright-shaped remote browser API.

The local UI harness may require Playwright browser binaries. Bootstrap/validation must install or verify only the browsers actually needed for ReproDocket's local UI tests.

Do not install multiple redundant local browser stacks.

## 8. Operating system scope

The initial zero-configuration bootstrap and protected-secret path are Windows-first.

Do not advertise macOS/Linux zero-configuration support until those startup, credential, browser-test, and cleanup paths are implemented and exercised.

The TypeScript application should avoid needless Windows-only logic outside the narrow bootstrap and protected-secret adapters so broader support remains feasible later.

## 9. Architecture

Initial supported local host architecture is Windows x64 unless the actual implementation machine and dependency set prove additional architectures as part of the validation matrix.

Do not claim ARM64 support from source portability alone.

## 10. Fresh environment proof

Before public release, use a clean checkout/worktree with no inherited project `node_modules` and prove:

```text
bootstrap finds or installs prerequisites
package-lock is honored
local browser-test dependency is installed/available
build completes
local app starts
provider setup is reachable
validation entry point works
only scoped generated output is created
```

A developer machine that was already configured does not by itself prove setup reproducibility.

## 11. Toolchain drift rule

When any direct dependency, Node major, PowerShell major, Solari SDK version, or build tool changes after a validated baseline:

```text
re-run package restore
re-run typecheck/build
run affected deterministic tests
run Solari contract tests if the SDK changed
run Full when the release or broad integration claim changes
update public requirement wording if support changed
```

Do not silently reuse evidence produced by a materially different toolchain.
