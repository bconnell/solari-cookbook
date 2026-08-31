# ReproDocket Solari Browser and Sandbox Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish the real Solari execution boundary with current SDK contract proof, recorded browser lifecycle, replay handling, deterministic sandbox fixture hosting, and ownership-aware cleanup.

**Architecture:** Production browser work goes through a narrow `BrowserProvider` adapter over the installed `@solarisdk/browser` package. Deterministic end-to-end fixtures use `@solarisdk/sandbox`. Live contract tests are small and explicit so later product logic depends on proven SDK behavior instead of copied examples.

**Tech Stack:** `@solarisdk/browser`, `@solarisdk/sandbox`, TypeScript, Vitest, Node.js, current Solari API, existing evidence/store infrastructure.

**Spec:** `docs/reprodocket-design.md`; external assumptions: `docs/reprodocket-sdk-baseline.md`; contracts: `docs/reprodocket-interface-contracts.md`; tests: `docs/reprodocket-test-matrix.md`.

## Global Constraints

* Plans 1 and 2 must be freshly green before creating billable live resources.
* The authorized Solari credential must come through the product credential provider; do not hardcode or write it into test fixtures.
* Every live-created resource is registered immediately and released in structured cleanup.
* Browser and sandbox cleanup failures fail the lifecycle authority even if functional assertions pass.
* Use the installed SDK's actual exported types and methods. Update the SDK baseline when reality differs.
* Do not install `playwright`, `playwright-core`, or `patchright-core` to drive Solari unless the primary `@solarisdk/browser` launch path is proven insufficient.
* The separate `@playwright/test` dependency remains for ReproDocket's own local UI tests.
* Live tests are serialized initially to minimize resource usage and eliminate avoidable concurrency ambiguity.
* No test may assume a replay is immediately ready after browser close.

---

## Task 1: Probe the installed Solari Browser SDK contract

**Files:**
- Create: `reprodocket/src/solari/SolariSdkContract.ts`
- Create: `reprodocket/tests/live/SolariBrowserContract.live.test.ts`
- Modify: `docs/reprodocket-sdk-baseline.md` only if observed API differs.

**Interfaces:**
- Produces a typed internal record of the installed package version and confirmed lifecycle capabilities.
- Does not yet expose full application browser operations.

- [ ] **Step 1: Inspect the installed package rather than guessing its type surface**

Run:

```powershell
Set-Location reprodocket
npm ls @solarisdk/browser
Get-Content .\node_modules\@solarisdk\browser\package.json -Raw
Get-ChildItem .\node_modules\@solarisdk\browser -Recurse -Include *.d.ts | Select-Object -ExpandProperty FullName
```

Search declarations for:

```text
class Solari
launch(
sessions
getReplayUrl
download
close(
```

Record exact signatures in task notes.

- [ ] **Step 2: Write the minimal live contract test I001-I006, I010-I014**

The test must:

```ts
const client = new Solari({ apiKey });
const browser = await client.launch({ recording: true });
const sessionId = browser.id;
try {
  const page = await browser.newPage();
  await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
  expect(await page.title()).not.toBe("");
  const png = await page.screenshot({ fullPage: true });
  expect(png.length).toBeGreaterThan(100);
} finally {
  await browser.close();
}
```

Then use the actual documented/installed replay API with bounded polling. If the installed client exposes an explicit client `close()` required to release a local handle, call it in a final outer cleanup and test process termination separately.

- [ ] **Step 3: Run with no credential and prove truthful BLOCKED behavior**

The test harness must not show PASS when `SOLARI_API_KEY`/protected credential is absent. It should exit/skip with a machine-readable BLOCKED reason that Full validation treats as not complete.

- [ ] **Step 4: Run with the authorized real credential**

```powershell
npm run test:live -- SolariBrowserContract.live.test.ts
```

Expected: actual browser create, navigate, screenshot, close, and replay readiness path succeed or expose a precise current SDK discrepancy.

- [ ] **Step 5: Verify the test process exits cleanly**

Run the test from a parent process with a bounded timeout. A test that prints PASS but leaves Node alive due to an SDK client handle is a failure.

- [ ] **Step 6: Reconcile the SDK baseline**

If observed signatures differ from `docs/reprodocket-sdk-baseline.md`, update the document with the observed package version/signature and do not retain a historical workaround without evidence.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/solari/SolariSdkContract.ts reprodocket/tests/live/SolariBrowserContract.live.test.ts docs/reprodocket-sdk-baseline.md
git diff --cached --check
git commit -m "test: prove the current Solari browser contract"
```

---

## Task 2: Implement explicit Solari resource ownership tracking

**Files:**
- Create: `reprodocket/src/solari/ResourceLedger.ts`
- Test: `reprodocket/tests/unit/ResourceLedger.test.ts`

**Interfaces:**
- Produces `register`, `releaseRequested`, `releaseConfirmed`, `releaseFailed`, `forRun`, and `unresolved` operations over `OwnedResourceRecord`.

- [ ] **Step 1: Write failing ownership tests**

Require registration before terminal updates, immutable resource ID/type/run ownership, idempotent release-confirmed updates, and visible cleanup error retention.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- ResourceLedger.test.ts
```

- [ ] **Step 3: Implement the ledger using the RunStore authority**

The ledger must not maintain a second durable resource database. It updates the owning run manifest atomically.

- [ ] **Step 4: Test unresolved-resource query**

A run with one confirmed closed browser and one release-failed browser must report exactly one unresolved resource.

- [ ] **Step 5: Prove GREEN and commit**

```powershell
npm run test:unit -- ResourceLedger.test.ts
git add reprodocket/src/solari/ResourceLedger.ts reprodocket/tests/unit/ResourceLedger.test.ts
git diff --cached --check
git commit -m "feat: track Solari resource ownership"
```

---

## Task 3: Build the production Solari browser provider

**Files:**
- Create: `reprodocket/src/solari/SolariBrowserProvider.ts`
- Create: `reprodocket/src/solari/SolariRetryPolicy.ts`
- Test: `reprodocket/tests/unit/SolariRetryPolicy.test.ts`
- Test: `reprodocket/tests/live/SolariBrowserProvider.live.test.ts`

**Interfaces:**
- Implements `BrowserProvider` from `docs/reprodocket-interface-contracts.md` using exact SDK types discovered in Task 1.
- Consumes `SolariCredentialProvider` and `ResourceLedger`.

- [ ] **Step 1: Write retry policy tests**

Assert:

```text
502 safe idempotent operation -> eligible within max attempts
503 safe idempotent operation -> eligible
504 safe idempotent operation -> eligible
429 -> SOLARI_CAPACITY_REACHED, no rapid automatic retry
500 -> not automatically retried
501 -> not automatically retried
unknown mutation outcome -> not retried
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- SolariRetryPolicy.test.ts
```

- [ ] **Step 3: Implement bounded retry policy**

Keep maximum attempts small (initially 3 total attempts) with bounded backoff. Do not retry action-level target mutations here; this policy is for safe provider lifecycle/control operations only.

- [ ] **Step 4: Write provider live tests**

Require `createRecordedBrowser` to register resource ownership before returning the handle. Closing the handle must update release requested/confirmed and be idempotent from the application perspective.

- [ ] **Step 5: Implement browser provider**

Construct the Solari client only with the current credential. Launch `{ recording: true }`. Do not enable stealth, proxy, captcha, or profile by default. Record the actual mode/options in provenance when available.

- [ ] **Step 6: Prove provider live lifecycle**

```powershell
npm run test:live -- SolariBrowserProvider.live.test.ts
```

Require no unresolved browser resources at test end.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/solari/SolariBrowserProvider.ts reprodocket/src/solari/SolariRetryPolicy.ts reprodocket/tests/unit/SolariRetryPolicy.test.ts reprodocket/tests/live/SolariBrowserProvider.live.test.ts
git diff --cached --check
git commit -m "feat: own recorded Solari browser sessions"
```

---

## Task 4: Attach real page evidence capture to the Solari browser

**Files:**
- Create: `reprodocket/src/solari/SolariPageEvidenceBridge.ts`
- Test: `reprodocket/tests/live/SolariPageEvidenceBridge.live.test.ts`
- Create: `reprodocket/fixtures/probe-site/index.html`
- Create: `reprodocket/fixtures/probe-site/probe.js`

**Interfaces:**
- Produces `attach(page, collector): DetachEvidenceBridge` using the exact page event API exposed by the installed browser client.

- [ ] **Step 1: Build a tiny deterministic probe page**

The page exposes buttons that deliberately:

```text
log console warning
log console error
throw uncaught Error("PROBE_PAGE_ERROR")
fetch /known-500
```

This probe is test infrastructure only and is not part of ReproDocket production behavior.

- [ ] **Step 2: Write failing live bridge tests I007-I009**

Host the probe through the fixture infrastructure once Task 7 exists; until then the test may use a stable public data URL only if the Solari browser supports it and doing so does not contradict target policy because the test is adapter-level, not product input. Prefer deferring actual execution until the sandbox probe URL is available rather than adding an unstable public dependency.

- [ ] **Step 3: Implement page event bridge from actual SDK/Playwright-shaped events**

Subscribe only to events needed by the evidence contract. Map console warning/error, pageerror, requestfailed, and response statuses relevant to evidence. Do not persist headers or bodies.

- [ ] **Step 4: Ensure detach is real**

`detach()` removes listeners. A repeated attach/detach cycle must not duplicate evidence events.

- [ ] **Step 5: Run live test after sandbox probe is available**

```powershell
npm run test:live -- SolariPageEvidenceBridge.live.test.ts
```

Require exactly the expected probe evidence cues and redaction behavior.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/solari/SolariPageEvidenceBridge.ts reprodocket/tests/live/SolariPageEvidenceBridge.live.test.ts reprodocket/fixtures/probe-site
git diff --cached --check
git commit -m "feat: capture evidence from Solari pages"
```

---

## Task 5: Implement replay retrieval as a bounded secondary artifact

**Files:**
- Create: `reprodocket/src/solari/SolariReplayService.ts`
- Test: `reprodocket/tests/unit/SolariReplayService.test.ts`
- Test: `reprodocket/tests/live/SolariReplayService.live.test.ts`

**Interfaces:**
- Implements `getReplay(sessionId): Promise<ReplayRecord>`.
- May persist local replay data only when the installed TypeScript SDK exposes and proves a supported byte-download path.

- [ ] **Step 1: Write deterministic replay state-machine tests**

Test:

```text
initial -> PENDING
ready URL -> READY_URL
supported downloaded bytes -> READY_LOCAL
bounded repeated not-ready -> UNAVAILABLE
non-retryable provider error -> UNAVAILABLE with sanitized message
```

Replay failure must not by itself rewrite a sufficiently evidenced run outcome.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- SolariReplayService.test.ts
```

- [ ] **Step 3: Implement bounded polling**

Use the current SDK's replay API and a bounded timeout. Do not busy-loop. Record checkedAt and sanitized status.

- [ ] **Step 4: Probe local-download capability**

Inspect installed declarations for a replay-download method. If present, write a focused live test that verifies returned bytes and media/format assumptions. If absent, do not invent HTTP endpoints from the Python SDK.

- [ ] **Step 5: Run live replay test**

```powershell
npm run test:live -- SolariReplayService.live.test.ts
```

The live test creates one short recorded browser, closes it, then resolves the replay. All resources must close.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/solari/SolariReplayService.ts reprodocket/tests/unit/SolariReplayService.test.ts reprodocket/tests/live/SolariReplayService.live.test.ts
git diff --cached --check
git commit -m "feat: retrieve Solari session replays"
```

---

## Task 6: Probe the installed Solari Sandbox SDK contract

**Files:**
- Create: `reprodocket/src/solari/SolariSandboxContract.ts`
- Create: `reprodocket/tests/live/SolariSandboxContract.live.test.ts`
- Modify: `docs/reprodocket-sdk-baseline.md` if required.

**Interfaces:**
- Confirms exact installed `SandboxClient`, create/connect/files/commands/preview/kill methods.

- [ ] **Step 1: Inspect package declarations**

```powershell
npm ls @solarisdk/sandbox
Get-Content .\node_modules\@solarisdk\sandbox\package.json -Raw
Get-ChildItem .\node_modules\@solarisdk\sandbox -Recurse -Include *.d.ts | Select-Object -ExpandProperty FullName
```

Search exact declarations for create, connect, files, commands, preview URL, timeout/lifecycle, and kill.

- [ ] **Step 2: Write minimal live contract test J001-J009/J011**

The test creates the smallest suitable sandbox, connects, writes `/tmp/reprodocket-probe/index.html`, starts a simple HTTP server, obtains the preview URL using the actual installed method, fetches it from the local test process, then kills the sandbox in `finally`.

Check the command's own exit code. A successful transport with nonzero command exit is a test failure.

- [ ] **Step 3: Run live**

```powershell
npm run test:live -- SolariSandboxContract.live.test.ts
```

- [ ] **Step 4: Confirm explicit kill**

The test must not pass by waiting for idle pause/timeout. The resource ledger must end with the sandbox confirmed terminal according to the available contract.

- [ ] **Step 5: Reconcile SDK baseline and commit**

```powershell
git add reprodocket/src/solari/SolariSandboxContract.ts reprodocket/tests/live/SolariSandboxContract.live.test.ts docs/reprodocket-sdk-baseline.md
git diff --cached --check
git commit -m "test: prove the current Solari sandbox contract"
```

---

## Task 7: Build the deterministic defective fixture application

**Files:**
- Create: `reprodocket/fixtures/defective-site/package.json`
- Create: `reprodocket/fixtures/defective-site/server.mjs`
- Create: `reprodocket/fixtures/defective-site/public/index.html`
- Create: `reprodocket/fixtures/defective-site/public/app.js`
- Create: `reprodocket/fixtures/defective-site/public/styles.css`
- Create: `reprodocket/fixtures/defective-site/fixture-version.json`
- Test: `reprodocket/tests/integration/DefectiveFixtureTruth.test.ts`

**Interfaces:**
- Provides deterministic routes/states for K001-K009.
- The fixture is standalone and contains no imports from ReproDocket production source.

- [ ] **Step 1: Write fixture truth tests before implementation**

Launch fixture locally on an ephemeral port and use `@playwright/test` or HTTP assertions to prove the exact ground truth independently from ReproDocket.

Required cases:

```text
/account: change Email then Save -> account panel becomes empty and page throws PROBE_ACCOUNT_SAVE_ERROR
/billing: RELOAD on /billing/details -> server responds with deterministic 404 page/state
/address: submit with empty ZIP -> target incorrectly shows "Address accepted" confirmation
/profile: valid profile Save -> "Profile saved" remains visible, no page error
/login: invalid credentials -> intended validation message, no uncaught error
/ambiguous: action changes visual text in a way intentionally insufficient for a built-in defect detector
/nonrepeatable: first fresh-session trigger reproduces once according to server-side run token, second independent trigger does not, used only to prove REPRODUCED vs VERIFIED without product hardcoding
/known-500: returns HTTP 500 for network evidence probe
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- DefectiveFixtureTruth.test.ts
```

- [ ] **Step 3: Implement the fixture server**

Use Node built-ins or the smallest existing server dependency already available. Do not add a production dependency solely for the fixture. Fixture state resets for each test/session through an explicit test route or per-fixture server instance, not hidden ReproDocket hooks.

- [ ] **Step 4: Implement accessible fixture markup**

Labels/buttons must be deterministic and accessible so reproduction steps exercise ordinary browser semantics:

```text
Email
Save
Country
ZIP
Submit address
Profile name
Save profile
Username
Password
Sign in
```

- [ ] **Step 5: Prove fixture truth GREEN**

```powershell
npm run test:integration -- DefectiveFixtureTruth.test.ts
```

Record fixture version from `fixture-version.json` in the test output.

- [ ] **Step 6: Mutation proof fixture oracle**

Temporarily remove one seeded defect and confirm its ground-truth test fails. Restore before commit.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/fixtures/defective-site reprodocket/tests/integration/DefectiveFixtureTruth.test.ts
git diff --cached --check
git commit -m "test: add deterministic defect fixture"
```

---

## Task 8: Deploy the fixture into a real Solari sandbox

**Files:**
- Create: `reprodocket/src/solari/SolariSandboxFixtureProvider.ts`
- Test: `reprodocket/tests/live/SolariSandboxFixtureProvider.live.test.ts`

**Interfaces:**
- Implements `FixtureProvider.create(): Promise<FixtureHandle>`.
- Consumes resource ledger and exact sandbox contract.

- [ ] **Step 1: Write failing live test**

The test requires:

```text
sandbox ID nonempty
fixture files copied/written
server start command exit 0
preview URL public HTTP/HTTPS
external GET returns fixture identity/version
kill() updates resource ledger
```

- [ ] **Step 2: Implement fixture upload**

Use sandbox file APIs to create the fixture tree. Do not depend on a GitHub checkout inside the sandbox for the initial harness; upload the exact current fixture files being validated so provenance is direct.

- [ ] **Step 3: Start server with explicit background semantics**

Use a shell only where necessary to background the fixture process. Capture command exit status/log path inside sandbox for diagnostics. Do not leave an interactive PTY running unnecessarily.

- [ ] **Step 4: Verify preview externally before returning handle**

Poll with bounded retries from the local process. Response must contain the exact fixture version. A URL existing is not enough.

- [ ] **Step 5: Ensure setup failure still kills sandbox**

Deliberately fail the fixture server command in a controlled test variant and require sandbox cleanup.

- [ ] **Step 6: Run live**

```powershell
npm run test:live -- SolariSandboxFixtureProvider.live.test.ts
```

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/solari/SolariSandboxFixtureProvider.ts reprodocket/tests/live/SolariSandboxFixtureProvider.live.test.ts
git diff --cached --check
git commit -m "feat: host ReproDocket fixtures on Solari"
```

---

## Task 9: Complete real page evidence bridge against the sandbox fixture

**Files:**
- Modify: `reprodocket/tests/live/SolariPageEvidenceBridge.live.test.ts`
- Modify: `reprodocket/src/solari/SolariPageEvidenceBridge.ts` only as required by proven SDK behavior.

- [ ] **Step 1: Create the fixture through `SolariSandboxFixtureProvider`**

Do not use a public third-party site for console/page/network evidence.

- [ ] **Step 2: Launch recorded browser through `SolariBrowserProvider`**

Navigate to fixture preview, attach evidence bridge, trigger console warning/error, uncaught page error, and `/known-500`.

- [ ] **Step 3: Assert exact evidence cues**

Require expected marker strings/status, correct sequence monotonicity, no headers/bodies, and no fake secret value if fixture injects one for redaction proof.

- [ ] **Step 4: Close browser and kill fixture in nested finally blocks**

At test end, require resource ledger unresolved count zero.

- [ ] **Step 5: Run live**

```powershell
npm run test:live -- SolariPageEvidenceBridge.live.test.ts
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/solari/SolariPageEvidenceBridge.ts reprodocket/tests/live/SolariPageEvidenceBridge.live.test.ts
git diff --cached --check
git commit -m "test: prove Solari browser evidence capture"
```

---

## Task 10: Add Solari live authorities to validation without making them routine local noise

**Files:**
- Modify: `reprodocket/scripts/preflight.ps1`
- Modify: `reprodocket/scripts/validate.ps1`
- Modify: `reprodocket/src/validation/ValidationReport.ts`
- Modify: `reprodocket/docs/validation.md`
- Test: `reprodocket/tests/integration/ValidationScript.test.ts`

**Interfaces:**
- Adds live-browser-contract, live-replay, live-sandbox-contract, live-fixture, and resource-closure stages to Full validation.

- [ ] **Step 1: Write validation orchestration tests**

Assert Targeted local validation does not create Solari sessions unless explicitly selected. Assert Full requires the live stages and returns BLOCKED/nonzero without authorized credential rather than skipping them as PASS.

- [ ] **Step 2: Update preflight**

Full profile checks provider readiness before running billable tests. It does not create all resources just to preflight; it reuses the proven readiness boundary from Plan 1.

- [ ] **Step 3: Order live stages after all cheap deterministic stages**

Required ordering:

```text
local deterministic gates
-> browser contract
-> replay contract
-> sandbox contract
-> sandbox fixture
-> page evidence bridge
-> resource reconciliation
```

- [ ] **Step 4: Reconcile live resource cleanup even after a failed live test**

The validation summary must surface unresolved owned resources by ID/type without enumerating or killing unrelated account resources.

- [ ] **Step 5: Run Plan 3 targeted/live validation**

Run the smallest relevant live suite first, then:

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted -IncludeLiveSolari
```

If the script does not support `-IncludeLiveSolari` yet, add this explicit opt-in to Targeted. Full always includes live Solari.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/scripts/preflight.ps1 reprodocket/scripts/validate.ps1 reprodocket/src/validation/ValidationReport.ts reprodocket/docs/validation.md reprodocket/tests/integration/ValidationScript.test.ts
git diff --cached --check
git commit -m "test: integrate live Solari validation"
```

---

## Plan 3 completion gate

Before Plan 4, all of these must be proven by current execution:

```text
installed @solarisdk/browser contract inspected and recorded
real recorded Solari browser create/navigate/screenshot/close works
Node live-test process exits cleanly
replay readiness uses a proven TypeScript API and bounded polling
local replay download is claimed only if actually supported/proven
installed @solarisdk/sandbox contract inspected and recorded
real sandbox can host exact current fixture
preview URL is externally verified before use
fixture ground truth passes independently from ReproDocket classification
real Solari browser captures expected fixture console/page/network evidence
browser and sandbox resources are ownership-tracked
cleanup failure is a validation failure
all Plan 1 and Plan 2 deterministic authorities remain green
```

Run:

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted -IncludeLiveSolari
git status --short
git diff main...HEAD --check
```

Do not start the product investigation engine until both live substrate contracts are current and clean.