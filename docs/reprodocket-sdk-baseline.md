# ReproDocket SDK Baseline

Date checked: August 31, 2026

This document records the external SDK assumptions ReproDocket is allowed to make before implementation begins. It is intentionally narrow. Runtime behavior and installed package types remain authoritative when they disagree with examples or older documentation.

## Solari browser

Current Solari documentation uses the package:

```text
@solarisdk/browser
```

The documented TypeScript entry point is:

```ts
import { Solari } from "@solarisdk/browser";
```

The primary ReproDocket browser path should use `client.launch()` rather than bringing a separate Playwright client unless a concrete SDK limitation requires otherwise.

Current documented behavior:

* `launch()` creates a fresh cloud browser and returns a Playwright shaped browser object.
* `browser.newPage()` creates a page.
* Page navigation, locators, evaluation, screenshots, and standard Playwright style interaction are available.
* `recording: true` enables session recording.
* `browser.id` identifies the Solari browser session.
* `browser.close()` ends the browser session.
* `client.sessions.getReplayUrl(sessionId)` is documented for recorded sessions after release.
* When using explicit `sessions.create()`, `releaseAndWait()` should complete before replay access.

References:

* https://docs.getsolari.com/quickstart
* https://docs.getsolari.com/browser-api
* https://docs.getsolari.com/sessions
* https://www.npmjs.com/package/@solarisdk/browser

The npm registry reports `@solarisdk/browser` version `0.1.1` as the current published package at the time of this baseline. The implementation must use the package lock as the exact dependency authority after installation.

## Browser client shutdown probe

The existing cookbook TypeScript quickstart contains an additional `await solari.close()` after `browser.close()` because the client version used by that example retained a loopback proxy handle that could keep Node alive.

Current public Solari documentation emphasizes `browser.close()` for session cleanup and does not make the extra client close call part of every example.

This is therefore an explicit execution-time probe, not an assumption.

Before the browser adapter is considered complete, the live contract test must prove all of the following against the installed package:

1. A browser session can be created.
2. The page can navigate and take a screenshot.
3. Closing the browser releases the session.
4. The Node process can terminate cleanly with the documented client shutdown sequence.
5. If the installed `Solari` object exposes a required or useful client `close()` method, the adapter owns and calls it exactly once during application shutdown.
6. No open handle remains solely because the Solari client was not disposed.

The implementation must follow the installed package's actual type surface and observed lifecycle rather than retaining a historical workaround blindly.

## Solari sandbox

Current Solari documentation uses the package:

```text
@solarisdk/sandbox
```

with:

```ts
import { SandboxClient } from "@solarisdk/sandbox";
```

Current documented behavior:

* `SandboxClient.create()` creates an isolated headless microVM.
* The sandbox can run commands and read or write files.
* A sandbox can expose a service through a public preview URL.
* `kill()` permanently tears down the sandbox.
* The default idle behavior may pause rather than destroy the sandbox, so ReproDocket must explicitly kill harness-owned fixture sandboxes.

References:

* https://docs.getsolari.com/sandboxes
* https://docs.getsolari.com/
* https://www.npmjs.com/package/@solarisdk/sandbox

The npm registry reports `@solarisdk/sandbox` version `0.1.2` as the current published package at the time of this baseline.

The existing cookbook `sandbox-port-preview-ts` example uses the unified `@solarisdk/sdk` package. ReproDocket should follow the current direct sandbox package unless live type inspection proves the unified package is materially better for this project.

## Sandbox preview probe

The cookbook demonstrates a `previewUrl(port)` style call, while documentation may evolve with the package surface. The live contract test must compile against the installed package and prove the exact supported preview method before fixture infrastructure is built around it.

The fixture harness must prove that the returned URL is reachable from outside the sandbox before giving it to a Solari browser.

## Region

Current Solari documentation lists `us-west` as the available browser region and the default. ReproDocket should not expose a region selector in version one while only one region is available.

Reference:

* https://docs.getsolari.com/regions

If additional regions become available during implementation, that fact alone does not justify adding configuration UI. Region support remains outside version one unless it materially improves the supported defect investigation workflow.

## Retry behavior

Solari error guidance distinguishes transient gateway failures from conditions that should not be blindly retried.

ReproDocket retry policy must follow these rules:

* Retry only bounded, known transient failures.
* Treat HTTP 502, 503, and 504 as candidates for a bounded retry when the operation is safe to repeat.
* Do not automatically retry 429 concurrency errors as though they were transient server faults. Report the capacity condition or wait only under an explicitly bounded policy.
* Do not classify a 500 or 501 as automatically transient.
* Preserve idempotency for any lifecycle operation where retrying could create duplicate external resources.
* A successful transport response from a sandbox command does not imply the command succeeded; inspect its command exit code.

Reference:

* https://docs.getsolari.com/errors
* https://docs.getsolari.com/api-reference

## Cost and concurrency

Live validation creates billable resources. Current Solari pricing states that browser sessions are billed while open and sandboxes are billed by allocated compute. Plan concurrency limits also apply.

The validation architecture therefore closes resources immediately after the evidence needed from them has been captured. Cost awareness may reduce idle time and redundant live runs, but it must not replace required live validation with mocks.

Reference:

* https://docs.getsolari.com/pricing

## Replays

Current TypeScript documentation guarantees replay URL access for sessions created with recording enabled. The existing Python cookbook example additionally demonstrates downloading replay data as rrweb NDJSON after asynchronous upload.

ReproDocket version one therefore guarantees one of two truthful states based on the installed TypeScript SDK:

1. replay data is downloaded and retained locally when a supported TypeScript download API is present and proven, or
2. the locally stored run retains the replay reference and availability state while all other evidence remains local.

The product must not advertise an offline replay file until the TypeScript path is proven against the real current SDK.

## Local UI testing

ReproDocket's local UI tests may use a separately installed Playwright test runner. That dependency is for testing ReproDocket's own localhost interface and is not required to drive the Solari cloud browser when the Solari Browser SDK already supplies its matching browser client.

This separation must remain clear in package scripts and documentation:

```text
@solarisdk/browser       remote target investigation
@solarisdk/sandbox       deterministic remote fixture hosting
@playwright/test         ReproDocket localhost UI and user-boundary validation
```

## Dependency selection rule

The initial implementation may use current stable releases of React, Fastify, Vite, Vitest, Zod, TypeScript, and supporting tooling, but the exact dependency set becomes authoritative only after:

1. installation on the target development machine,
2. TypeScript compilation,
3. unit and integration execution,
4. local application startup,
5. live Solari browser smoke validation,
6. live Solari sandbox smoke validation, and
7. creation and commit of `reprodocket/package-lock.json`.

No dependency is added solely because it is common or convenient. Every runtime dependency must support a concrete product requirement.

## Authority rule

When these sources disagree, use this order for implementation decisions:

1. Installed package type declarations and observable live behavior.
2. Current Solari product documentation.
3. Current package registry metadata.
4. Current cookbook examples.
5. Historical discussion or assumptions.

Any material SDK contradiction discovered during implementation is recorded in the validation report and resolved before ReproDocket claims the affected capability.