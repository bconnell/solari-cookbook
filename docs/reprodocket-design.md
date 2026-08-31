# ReproDocket Design Specification

Date: August 31, 2026
Status: Pre-implementation design baseline

## 1. Purpose

ReproDocket is a local, evidence-first web defect reproduction and verification product built on Solari. A user supplies a public web target, a human-readable problem description, and an auditable reproduction/observation plan. ReproDocket executes that plan in a real Solari cloud browser, captures supporting evidence, closes the first browser, repeats the same plan in a fresh Solari browser, and presents the combined result locally.

The product is designed to make browser defect reproduction more trustworthy. It does not treat one automated attempt, one screenshot, or one console error as sufficient proof. Investigation, independent verification, evidence provenance, persistence, and resource cleanup are separate parts of the result.

## 2. Solari integration

The upstream Solari cookbook remains intact. ReproDocket is added as a first-class project under `reprodocket/`. The fork relationship, upstream examples, MIT license, and Pinetree Research copyright remain preserved.

Solari is central to the shipped workflow. Production investigations and independent verification use real Solari cloud browsers. The deterministic full validation fixture is hosted in a real Solari sandbox. Unit and local integration tests may use test doubles where isolation is appropriate, but doubles cannot satisfy live Solari or end-to-end completion gates.

There is no silent local-browser fallback for a production investigation.

## 3. Supported version-one workflow

Version one supports this complete user path:

1. Start ReproDocket locally with one command.
2. Configure a Solari API key through the local interface if no supported credential source is already present.
3. Enter a public HTTP/HTTPS target URL.
4. Enter a problem description.
5. Enter an auditable reproduction and observation plan using the supported plan grammar.
6. Submit the run.
7. ReproDocket validates the request before creating an external resource.
8. ReproDocket creates a recorded Solari browser and executes the plan.
9. It captures semantically useful screenshots, action timing, page errors, selected console evidence, selected network evidence, current URLs/status, and expectation results.
10. It closes the investigation browser and records replay state.
11. It creates a separate fresh recorded Solari browser.
12. It executes the same validated plan as independent verification.
13. It compares evidence from both attempts and produces a bounded final outcome.
14. It writes a durable local manifest, integrity metadata, JSON report, and static HTML report.
15. The local interface shows investigation evidence, verification evidence, final outcome, replay state, cleanup state, and history.
16. Completed runs remain available after ReproDocket restarts.

Version one does not promise arbitrary autonomous bug discovery from prose alone. A future AI planner may generate the same auditable plan, but no runtime planner is required for the initial product.

## 4. Auditable plan model

The plan has two kinds of statements: browser actions and observable expectations.

Initial actions include:

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

Initial expectations include:

```text
EXPECT_TEXT
EXPECT_NO_TEXT
EXPECT_URL
EXPECT_PAGE_ERROR
EXPECT_MAIN_STATUS
```

The exact syntax and contracts are defined in [`reprodocket-interface-contracts.md`](reprodocket-interface-contracts.md).

The grammar intentionally excludes arbitrary JavaScript, arbitrary CSS/XPath selectors, shell execution, direct DOM mutation, filesystem access, and local-process commands.

A problem description explains the defect to a human. The plan explains what the browser should do and which observable condition defines success/failure. Neither one directly sets the final result.

## 5. Result model

Run lifecycle and defect outcome are separate authorities.

Lifecycle:

```text
CREATED
PREPARING
INVESTIGATING
VERIFYING
FINALIZING
COMPLETED
FAILED
CANCELLED
INTERRUPTED
```

Final outcome:

```text
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
```

`VERIFIED` requires reproduction evidence from the investigation and a separate fresh verification session.

`REPRODUCED` means the first attempt observed the defined defect condition but clean independent verification did not confirm it.

`NOT_REPRODUCED` means the intended workflow and observation contract were exercised sufficiently and the defined defect condition was not observed.

`INCONCLUSIVE` means the available evidence is not strong enough for a trustworthy determination.

An infrastructure failure does not become `NOT_REPRODUCED`. A console warning or failed subrequest alone does not become `REPRODUCED`. Uncertainty is preserved rather than promoted to success.

## 6. Technology choices

### 6.1 Runtime and language

ReproDocket uses TypeScript on Node.js. This follows Solari's direct JavaScript/TypeScript browser path and avoids an unnecessary interoperability layer.

Exact dependency versions are locked through `reprodocket/package-lock.json` after installation and live contract validation.

### 6.2 Local server and UI

Fastify provides the local service. React with Vite provides the local interface. The normal built application is served from one loopback Node process.

Server-Sent Events provide live progress updates. Durable run state remains the authority after reconnect or refresh.

### 6.3 Persistence

Version one uses filesystem-based run storage under the current user's local application data directory. No database server, schema deployment, or manual migration step is required.

Large/generated runtime evidence remains outside source control.

### 6.4 Credentials

The normal Windows path stores a Solari credential using the current Windows-user protection boundary. Environment-variable support remains available for automated/developer environments, but a normal user is not required to create an `.env` file.

Secrets are excluded from run manifests, evidence, reports, screenshots generated by ReproDocket itself, logs, process metadata, and public source. Target screenshots may still contain sensitive information visibly rendered by the target site; the product must state that limitation truthfully.

## 7. Repository shape

```text
solari-cookbook/
  examples/                       upstream cookbook preserved
  LICENSE                         upstream license preserved
  README.md                       cookbook plus restrained ReproDocket entry after release
  docs/
    reprodocket-design.md
    reprodocket-interface-contracts.md
    reprodocket-security-lifecycle.md
    reprodocket-sdk-baseline.md
    reprodocket-test-matrix.md
    implementation/
  reprodocket/
    package.json
    package-lock.json
    src/
      client/
      server/
      core/
      solari/
      evidence/
      storage/
      reporting/
      security/
      shared/
      validation/
    fixtures/
    tests/
    scripts/
    docs/
```

Implementation files should have one clear responsibility. Shared bottlenecks are split when they become multi-purpose or create unclear ownership.

## 8. Local startup and zero-configuration goal

The normal Windows start path is:

```powershell
.\reprodocket\scripts\run.ps1
```

The startup path discovers the repository/application location, verifies required tooling, restores/builds when necessary, starts one owned local ReproDocket instance, health-checks it, and opens the default browser.

The user does not manually configure ports, start separate frontend/backend terminals, create a database, edit JSON configuration, or browse artifact folders to understand results.

If the preferred port is already owned by another process, ReproDocket selects another loopback port. It never kills an unrelated process simply to obtain a preferred port.

A separate `bootstrap.ps1` handles missing prerequisites where safe automation is available. A separate `validate.ps1` owns deterministic validation profiles.

New Windows automation does not use WMIC.

## 9. Local service security

The local server binds to loopback only by default. It rejects unexpected host/origin state-changing requests and uses a process-local request nonce as defense in depth for mutation APIs.

The built UI uses a restrictive content policy. Target-derived values render as inert text. The standalone HTML report escapes untrusted values and does not require JavaScript.

Artifact APIs resolve only run-owned artifact identifiers from validated manifests. User-provided filesystem paths are not accepted.

Detailed requirements are in [`reprodocket-security-lifecycle.md`](reprodocket-security-lifecycle.md).

## 10. Target safety

Version one accepts only public HTTP/HTTPS targets. It rejects executable/local schemes, credentials embedded in URLs, loopback, link-local, private address literals, and known metadata-service destinations before execution.

Redirects and DNS destinations are revalidated to the strongest authoritative boundary the installed browser/provider exposes. The product must document any provider-dependent limitation rather than claiming universal SSRF prevention.

## 11. Evidence model

Evidence is captured at meaningful semantic boundaries rather than fixed screenshot intervals.

Durable evidence may include:

* screenshots,
* warning/error console entries,
* uncaught page errors,
* sanitized network method/URL/status/failure information,
* action/expectation timeline,
* Solari session identity,
* replay availability/reference or locally retained replay data when the TypeScript SDK path is proven,
* investigation and verification observations,
* cleanup state.

Authorization/cookie headers, unrestricted bodies, password values, browser-storage dumps, Solari credentials, and arbitrary DOM snapshots are excluded by default.

Text evidence passes through a central redaction boundary before persistence.

## 12. Evidence integrity and provenance

Each finalized durable artifact receives a SHA256 digest. A final manifest records ownership, artifact identity, byte length, digest, timestamps, source revision, application version, relevant package versions, fixture version for harness runs, and validation profile.

A finalized artifact that is missing, substituted, or hash-mismatched is not silently displayed under a valid evidence label.

Historical outputs do not satisfy a fresh validation gate after relevant source changes.

## 13. Independent verification

Investigation and verification are deliberately separate attempts.

The investigation browser is closed before the verification attempt is created. The verification attempt receives a new Solari session identity and new browser/page state. Browser/session reuse is prohibited for the verification claim.

The harness explicitly checks that session IDs differ. A classifier cannot return `VERIFIED` without evidence from both attempts.

## 14. Deterministic Solari fixture

The full harness hosts a controlled test website inside a real Solari sandbox and exposes it using a Solari preview URL. The URL is externally probed before browser tests consume it.

The fixture contains deterministic defect, healthy, ambiguous, and nonrepeatable cases. Its truth is tested independently from ReproDocket so the fixture itself is not a weak oracle.

Production ReproDocket code cannot branch on fixture route names or scenario IDs to manufacture expected outcomes.

## 15. Run storage and recovery

Each run owns a generated directory beneath the application data root. Manifest updates are written atomically where practical.

Malformed historical state is isolated to the affected run rather than crashing history globally.

A previous-process run left in a nonterminal state becomes `INTERRUPTED` on startup unless a future explicitly designed resume mechanism is introduced. Version one does not automatically replay browser actions after a crash.

Captured valid evidence is preserved.

## 16. Resource ownership

Every Solari browser/sandbox created by ReproDocket is registered immediately with explicit ownership. Browser/session and sandbox cleanup occur in structured `finally`-style boundaries.

A functional scenario cannot produce a clean Full lifecycle result while required owned resources remain unresolved.

The local server also records sufficient identity to distinguish its owned process from a recycled or unrelated PID before stop operations.

## 17. Concurrency and cancellation

Version one supports one executing investigation pipeline per local application process. This is explicit supported capacity, not a hidden limit.

A second create request while a run is active receives a visible `RUN_ALREADY_ACTIVE` response. It is not silently dropped or silently queued. Historical browsing remains available.

Cancellation stops new actions, aborts/winds down the current attempt within bounded time, releases owned remote resources, and persists `CANCELLED`. The UI does not optimistically display cancellation completion before the server has persisted it.

Application shutdown follows the same ownership-aware cleanup path for an active run.

## 18. Local results interface

The version-one UI is intentionally small and complete rather than broad and decorative.

Primary surfaces:

```text
provider connection state
new investigation
active investigation
run detail with history navigation
```

Run detail exposes, when applicable:

```text
run lifecycle
investigation result
verification result
final outcome
reproduction/expectation plan
screenshots
console evidence
page errors
network evidence
timeline
replay state
report
cleanup result
limitations/errors
```

Every visible interaction either performs real behavior, is truthfully disabled with a current reason, or is absent. No placeholder/dead controls are used for presentation value.

## 19. Horizontal and vertical integration

### Vertical

A supported run is not complete until the entire path is proven:

```text
local UI
-> request validation
-> run admission/persistence
-> Solari investigation
-> evidence capture
-> first cleanup
-> fresh Solari verification
-> evidence/classification
-> report/integrity finalization
-> local UI/history
-> reload/restart
-> resource reconciliation
-> user return path
```

### Horizontal

Sibling surfaces must agree on the same authority:

```text
active progress vs manifest
history vs run detail
outcome badge vs JSON/HTML report
screenshot/console/network/timeline views vs owned artifacts
replay control vs replay state
cleanup display vs resource ledger
provider readiness vs actual credential path
validation summary vs exact source revision
README capability wording vs current proven maturity
```

A shared router/store/serializer/state/adapter defect triggers review of adjacent consumers rather than a one-screen cosmetic patch.

A maintained connectivity matrix records discoverability, entry, prerequisite truth, action routing, feedback, authority, persistence, reload, recovery, return path, regression coverage, and remaining human QA for every visible feature.

## 20. Validation authorities

ReproDocket uses separate validation authorities:

1. Static/repository quality.
2. Unit behavior.
3. Local integration.
4. Built local UI user-boundary tests.
5. Security/content/persistence failure tests.
6. Real Solari browser contract tests.
7. Real Solari sandbox/fixture contract tests.
8. Full investigation/verification end-to-end tests.
9. Horizontal connectivity checks.
10. Vertical connectivity checks.
11. Resource/lifetime checks.
12. Fresh Windows bootstrap/start/stop checks.
13. Harness sensitivity/mutation checks.
14. Public-source/documentation audit.
15. Human visual/usability acceptance where deterministic tests are insufficient.

The complete catalog is defined in [`reprodocket-test-matrix.md`](reprodocket-test-matrix.md).

## 21. Targeted and Full validation

`TARGETED` validation runs the smallest sufficient real checks for a bounded change.

`FULL` validation is required for broad milestone, hardening, final completion, and publication claims. Full includes live Solari authorities once the product reaches those phases.

A mandatory `BLOCKED` or `FAIL` does not count as PASS.

Human visual/usability acceptance remains a separate gate. Machine Full PASS is not renamed human PASS.

## 22. Harness sensitivity

Critical validators must prove they can fail. Representative deliberate mutations include:

```text
allow VERIFIED without verification evidence
reuse the first session ID
remove private-target blocking
remove local request guard
skip escaping/redaction
serve a cross-run artifact
skip evidence hashing
allow duplicate pipelines
swallow cleanup failure
skip sandbox kill
always classify broken
always classify healthy
hardcode fixture scenario outcome
accept stale source revision evidence
```

A test that remains green under the defect it exists to guard is itself defective and must be strengthened.

## 23. Bug-finding and hardening sequence

Feature implementation does not immediately become release completion.

The required sequence is:

```text
front-to-back implementation
-> horizontal/vertical integration closure
-> Full machine validation
-> deliberate visible-feature and workflow bug audit
-> defect repairs with regressions
-> proportional security/lifecycle/persistence hardening
-> fresh Full validation
-> human visual/usability review
-> escaped-defect repairs plus harness improvements
-> affected validation
-> final fresh Full validation
-> documentation/publication audit
```

Hardening focuses on demonstrated risk and supported scope rather than speculative enterprise infrastructure.

## 24. Visual and usability proof

The ReproDocket interface is part of the product under test. Backend/API success cannot prove the result is understandable.

Automated browser validation checks navigation, critical control availability, state consistency, overflow, basic accessibility semantics, keyboard reachability, responsive layouts, malicious-content inertness, and representative zoom/size behavior.

Original-resolution product screenshots are prepared for human review from the same current build. Background-safe Playwright capture is preferred for this web UI; desktop-wide pointer/keyboard automation is unnecessary for ordinary layout claims.

Human review judges readability, hierarchy, status clarity, evidence comprehensibility, affordance truth, error usefulness, and professional polish. Any escaped defect that automation could reasonably detect creates a regression/harness repair.

## 25. Documentation and publication truth

Public documentation must describe only current supported behavior. Planned architecture does not become a shipped capability claim.

Maturity language may use:

```text
Planned
Foundation
In Progress
Available (Unhardened)
Available
```

`Available` requires the declared scope to be complete, integrated, persistent/recoverable where applicable, failure-aware, resource-clean, validated, documented, and free of known ordinary-scope hardening blockers.

The root cookbook README will surface ReproDocket only after the project is genuinely usable, while preserving the upstream cookbook identity. ReproDocket receives its own README with setup, supported plan grammar, evidence model, validation scope, and limitations.

Any challenge/social submission is prepared from the final validated product and remains a separate public-mutation decision.

## 26. Detailed implementation sequence

The authoritative implementation plan index is [`implementation/README.md`](implementation/README.md).

Ordered plans:

1. Foundation and local shell.
2. Evidence, persistence, and local results.
3. Solari browser and sandbox substrate.
4. Investigation, independent verification, and complete end-to-end path.
5. Deliberate bug finding, hardening, and final proof.
6. Publication and challenge submission.

The plans intentionally build executable validation alongside each capability rather than postponing the harness until the end.

## 27. Completion definition

ReproDocket version one is front-to-back complete only when every visible supported feature is connected horizontally and vertically; the normal user can enter, execute, inspect, reload, and leave the workflow; the two Solari attempts are independent; important evidence is locally persisted and traceable; failures are safe and diagnostic; cancellation/restart behavior is truthful; owned resources close; the harness detects representative deliberate regressions; the complete product survives the deliberate bug-finding/hardening/revalidation sequence; human UI acceptance is recorded; and public documentation agrees with the exact current runtime.

A successful build, a large unit-test count, one working API request, one screenshot, one Solari session, or one happy-path fixture result is insufficient by itself.