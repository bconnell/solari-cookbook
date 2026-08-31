# ReproDocket Evidence, Persistence, and Local Results Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make every run a durable, integrity-checked local record and make the complete current record understandable through the actual ReproDocket UI before live Solari orchestration is added.

**Architecture:** `FileRunStore` remains the durable authority. Evidence is written through one sanitizing artifact boundary, sealed with hashes at finalization, projected through safe API routes, and rendered by React from authoritative manifests. SSE accelerates live progress but never replaces manifest rehydration.

**Tech Stack:** TypeScript, Node.js filesystem/crypto, Zod, Fastify, React, Vitest, Playwright Test.

**Spec:** `docs/reprodocket-design.md`; contracts: `docs/reprodocket-interface-contracts.md`; security: `docs/reprodocket-security-lifecycle.md`; test catalog: `docs/reprodocket-test-matrix.md`.

## Global Constraints

* Plan 1 must be freshly validated before this work starts.
* No target-derived text is treated as HTML.
* Evidence artifacts are always scoped to one run and, where applicable, one attempt.
* Durable text evidence is redacted before persistence.
* Finalized evidence is hashed and verified before serving.
* A damaged historical run may not crash or hide unrelated valid history.
* SSE is advisory/live transport; GET run state remains authoritative after reconnect.
* No database is introduced.
* No browser/Solari mock result can be used to claim live Solari completion; this plan only proves local evidence infrastructure.

---

## Task 1: Implement the central redaction boundary

**Files:**
- Create: `reprodocket/src/evidence/Redactor.ts`
- Create: `reprodocket/src/evidence/SensitiveValueRegistry.ts`
- Test: `reprodocket/tests/unit/Redactor.test.ts`

**Interfaces:**
- Produces `redactText(input: string, registry: SensitiveValueRegistry): string`.
- Produces `redactUrl(input: string, registry): string`.
- Produces per-run/process registration of exact sensitive values without persisting the registry.

- [ ] **Step 1: Write failing E008-E014 tests**

Use only fake values. Include:

```ts
const fakeKey = "slr_live_TEST_DO_NOT_USE_4d9f0b8f";
registry.add(fakeKey);
expect(redactText(`key=${fakeKey}`, registry)).not.toContain(fakeKey);
expect(redactText("Authorization: Bearer abc.def.ghi", registry)).not.toContain("abc.def.ghi");
```

Test URL query parameters named `token`, `api_key`, `apikey`, `access_token`, `password`, and `secret` are replaced with a marker while preserving enough URL structure for diagnosis.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- Redactor.test.ts
```

- [ ] **Step 3: Implement exact-value and pattern redaction**

Use a stable marker such as `[REDACTED]`. Escape exact values before building any regular expression. Do not log the registry itself.

- [ ] **Step 4: Prove GREEN and mutation sensitivity**

Disable exact-key replacement temporarily and confirm the fake-key test fails. Restore and rerun.

- [ ] **Step 5: Commit**

```powershell
git add reprodocket/src/evidence/Redactor.ts reprodocket/src/evidence/SensitiveValueRegistry.ts reprodocket/tests/unit/Redactor.test.ts
git diff --cached --check
git commit -m "feat: redact sensitive ReproDocket evidence"
```

---

## Task 2: Implement run-owned artifact writing and sealing

**Files:**
- Create: `reprodocket/src/storage/ArtifactStore.ts`
- Create: `reprodocket/src/evidence/EvidenceHasher.ts`
- Modify: `reprodocket/src/storage/FileRunStore.ts`
- Test: `reprodocket/tests/integration/ArtifactStore.test.ts`
- Test: `reprodocket/tests/unit/EvidenceHasher.test.ts`

**Interfaces:**
- Produces `writeArtifact(runId, attemptId, kind, bytes, mediaType, label)`.
- Produces `sha256(bytes): string`.
- Extends `RunStore.seal(runId)`.

- [ ] **Step 1: Write failing F006-F011 tests**

Require unique artifact IDs, run ownership, byte length, final SHA256, and detection of tampering/missing finalized files.

- [ ] **Step 2: Write path traversal/cross-run tests D012-D015**

Attempt encoded traversal and artifact substitution. Ensure the store cannot be tricked by a user-supplied relative path because paths are generated internally.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:unit -- EvidenceHasher.test.ts
npm run test:integration -- ArtifactStore.test.ts
```

- [ ] **Step 4: Implement artifact layout**

Use a generated artifact ID and kind-specific subdirectory. Example internal layout:

```text
<run>/investigation/screenshots/<artifact-id>.png
<run>/investigation/console/<artifact-id>.json
<run>/verification/screenshots/<artifact-id>.png
<run>/report/<artifact-id>.html
```

The persisted manifest, not the filename, determines ownership/type.

- [ ] **Step 5: Implement sealing**

For each durable artifact, read current bytes, calculate SHA256, update byte length/hash/sealed state atomically, then produce an integrity summary. If a required artifact is missing, sealing fails.

- [ ] **Step 6: Prove GREEN**

```powershell
npm run test:unit -- EvidenceHasher.test.ts
npm run test:integration -- ArtifactStore.test.ts
```

- [ ] **Step 7: Mutation proof**

After a valid sealed run, alter one screenshot byte and prove the verification test returns `EVIDENCE_INVALID`. Restore test fixture state.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/storage/ArtifactStore.ts reprodocket/src/evidence/EvidenceHasher.ts reprodocket/src/storage/FileRunStore.ts reprodocket/tests/integration/ArtifactStore.test.ts reprodocket/tests/unit/EvidenceHasher.test.ts
git diff --cached --check
git commit -m "feat: seal run-owned evidence artifacts"
```

---

## Task 3: Implement structured console, page-error, network, and timeline persistence

**Files:**
- Create: `reprodocket/src/evidence/ConsoleCollector.ts`
- Create: `reprodocket/src/evidence/PageErrorCollector.ts`
- Create: `reprodocket/src/evidence/NetworkCollector.ts`
- Create: `reprodocket/src/evidence/TimelineCollector.ts`
- Create: `reprodocket/src/evidence/EvidenceCollector.ts`
- Test: `reprodocket/tests/unit/EvidenceCollectors.test.ts`

**Interfaces:**
- Produces collector interfaces and entry types defined in the contracts document.
- Consumes `Redactor`, `SensitiveValueRegistry`, and `ArtifactStore`.

- [ ] **Step 1: Write failing collector tests**

Use fake event objects independent of Playwright/Solari. Assert sequence ordering, warning/error filtering, sanitized URLs, excluded headers/bodies, and monotonic timeline sequence.

Example:

```ts
collector.recordNetwork({
  method: "POST",
  url: "https://example.com/api?token=secret-value",
  status: 500,
  failureText: null,
  durationMs: 42
});
expect(await collector.entries()).toEqual([
  expect.objectContaining({ url: "https://example.com/api?token=%5BREDACTED%5D", status: 500 })
]);
```

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- EvidenceCollectors.test.ts
```

- [ ] **Step 3: Implement collectors**

Collectors accept normalized event values and never own browser session lifecycle. The later Solari adapter translates real page events into these contracts.

- [ ] **Step 4: Persist attempt evidence on finish**

`EvidenceCollector.finishAttempt()` writes structured JSON artifacts through `ArtifactStore`, records IDs on the attempt, and becomes idempotent so duplicate finish calls do not create duplicate artifacts.

- [ ] **Step 5: Prove GREEN**

```powershell
npm run test:unit -- EvidenceCollectors.test.ts
npm run typecheck
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/evidence reprodocket/tests/unit/EvidenceCollectors.test.ts
git diff --cached --check
git commit -m "feat: collect structured run evidence"
```

---

## Task 4: Generate safe static JSON and HTML reports from authoritative run state

**Files:**
- Create: `reprodocket/src/reporting/HtmlEscaper.ts`
- Create: `reprodocket/src/reporting/RunReportBuilder.ts`
- Test: `reprodocket/tests/unit/HtmlEscaper.test.ts`
- Test: `reprodocket/tests/integration/RunReportBuilder.test.ts`

**Interfaces:**
- Produces `buildReport(run: RunManifest): { json: Uint8Array; html: Uint8Array }`.
- Report generation consumes a validated manifest only.

- [ ] **Step 1: Write malicious-content tests F015-F020**

Inputs must include:

```text
</script><script>alert(1)</script>
<img src=x onerror=alert(1)>
& < > " '
```

Require literal escaped text in output and zero `<script` elements generated from untrusted data.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:unit -- HtmlEscaper.test.ts
npm run test:integration -- RunReportBuilder.test.ts
```

- [ ] **Step 3: Implement explicit HTML escaping**

Escape `&`, `<`, `>`, `"`, and `'` for every untrusted interpolation. Do not add a Markdown-to-HTML pipeline for target text.

- [ ] **Step 4: Implement report builder**

Report must include target, problem, lifecycle, investigation outcome, verification outcome, final outcome, reproduction steps, evidence listing, replay state, cleanup state, duration/timestamps, and limitations/errors. It must not include local absolute paths or secrets.

HTML is static and contains no JavaScript.

- [ ] **Step 5: Write report through ArtifactStore**

Add report artifact IDs to the run, then include them in final sealing.

- [ ] **Step 6: Prove GREEN and open the report manually**

```powershell
npm run test:unit -- HtmlEscaper.test.ts
npm run test:integration -- RunReportBuilder.test.ts
```

Open the generated test report in a browser and inspect that malicious strings render as text.

- [ ] **Step 7: Commit**

```powershell
git add reprodocket/src/reporting reprodocket/tests/unit/HtmlEscaper.test.ts reprodocket/tests/integration/RunReportBuilder.test.ts
git diff --cached --check
git commit -m "feat: generate safe local run reports"
```

---

## Task 5: Add history, run detail, report, and artifact API routes

**Files:**
- Create: `reprodocket/src/server/routes/runsRead.ts`
- Create: `reprodocket/src/server/routes/artifacts.ts`
- Modify: `reprodocket/src/server/createServer.ts`
- Test: `reprodocket/tests/integration/RunReadRoutes.test.ts`
- Test: `reprodocket/tests/integration/ArtifactRoutes.test.ts`

**Interfaces:**
- Produces `GET /api/runs`.
- Produces `GET /api/runs/:runId`.
- Produces `GET /api/runs/:runId/artifacts/:artifactId`.
- Produces `GET /api/runs/:runId/report`.

- [ ] **Step 1: Write failing G006-G009 and G015-G017 tests**

Seed run directories only through `FileRunStore`; do not hand-edit JSON in tests except explicit malformed/tamper cases.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunReadRoutes.test.ts ArtifactRoutes.test.ts
```

- [ ] **Step 3: Implement list/detail routes**

Clamp limit to 1..200, default 50, newest first. A malformed historical run returns a damaged summary/projection, not a fake valid `RunManifest`.

- [ ] **Step 4: Implement artifact route**

Resolve artifact by ID from the validated requested run. Verify sealed artifacts before serving. Set media type from manifest. Add `Content-Disposition: inline` for screenshots/report and safe headers appropriate to the type.

- [ ] **Step 5: Implement report route**

Serve only the run-owned generated HTML report artifact; do not dynamically render arbitrary query input.

- [ ] **Step 6: Prove GREEN**

```powershell
npm run test:integration -- RunReadRoutes.test.ts ArtifactRoutes.test.ts
```

- [ ] **Step 7: Mutation proof cross-run access**

Temporarily bypass run ownership lookup, confirm cross-run test fails, restore.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/src/server/routes/runsRead.ts reprodocket/src/server/routes/artifacts.ts reprodocket/src/server/createServer.ts reprodocket/tests/integration/RunReadRoutes.test.ts reprodocket/tests/integration/ArtifactRoutes.test.ts
git diff --cached --check
git commit -m "feat: expose validated local run evidence"
```

---

## Task 6: Implement live run events without creating a second truth store

**Files:**
- Create: `reprodocket/src/server/sse/RunEventHub.ts`
- Create: `reprodocket/src/server/routes/runEvents.ts`
- Test: `reprodocket/tests/integration/RunEvents.test.ts`

**Interfaces:**
- Produces `publish(event: RunEvent): void`.
- Produces `subscribe(runId, sink): unsubscribe`.
- Produces `GET /api/runs/:runId/events`.

- [ ] **Step 1: Write failing G012-G014 tests**

Require first event `snapshot`, monotonic event IDs, no event leakage across runs, and clean unsubscribe on connection close.

- [ ] **Step 2: Prove RED**

```powershell
npm run test:integration -- RunEvents.test.ts
```

- [ ] **Step 3: Implement in-memory event hub**

The hub does not persist a second run model. Before subscribing, the route reads the current manifest and sends a snapshot. New events carry sequence IDs from the same run sequencing authority.

- [ ] **Step 4: Handle reconnect truthfully**

Do not promise full event replay in version one. Client reconnect performs GET run state and then resubscribes. Document this behavior in code/tests.

- [ ] **Step 5: Prove GREEN and leak check**

After closing all test subscribers, require subscriber counts to be zero.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/server/sse reprodocket/src/server/routes/runEvents.ts reprodocket/tests/integration/RunEvents.test.ts
git diff --cached --check
git commit -m "feat: stream authoritative run progress"
```

---

## Task 7: Build the History and Run Detail UI from the real APIs

**Files:**
- Create: `reprodocket/src/client/components/HistoryList.tsx`
- Create: `reprodocket/src/client/components/RunHeader.tsx`
- Create: `reprodocket/src/client/components/EvidenceTabs.tsx`
- Create: `reprodocket/src/client/components/ScreenshotEvidence.tsx`
- Create: `reprodocket/src/client/components/ConsoleEvidence.tsx`
- Create: `reprodocket/src/client/components/NetworkEvidence.tsx`
- Create: `reprodocket/src/client/components/TimelineEvidence.tsx`
- Create: `reprodocket/src/client/components/ReplayEvidence.tsx`
- Create: `reprodocket/src/client/pages/RunPage.tsx`
- Modify: `reprodocket/src/client/App.tsx`
- Modify: `reprodocket/src/client/api/ApiClient.ts`
- Test: `reprodocket/tests/ui/RunHistory.spec.ts`
- Test: `reprodocket/tests/ui/RunDetail.spec.ts`

**Interfaces:**
- UI consumes only documented API contracts.
- All evidence selections use run ID + artifact ID, never path strings from query parameters.

- [ ] **Step 1: Write failing H012-H024 and H016-H021 tests using real local FileRunStore fixtures**

Prepare manifests through test helpers that call production persistence APIs. Test at least VERIFIED, NOT_REPRODUCED, INCONCLUSIVE, replay pending/ready/unavailable, and a damaged history entry.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- RunHistory.spec.ts RunDetail.spec.ts
```

- [ ] **Step 3: Implement history**

Show target host, concise problem, lifecycle, outcome when available, timestamp, and cleanup state. Do not display a green success treatment for a FAILED lifecycle merely because an earlier attempt reproduced a defect.

- [ ] **Step 4: Implement run header**

Clearly separate:

```text
Run lifecycle
Investigation result
Verification result
Final outcome
Cleanup
```

Outcome must be communicated by text/iconography in addition to any color.

- [ ] **Step 5: Implement evidence views**

Screenshots render through artifact endpoints. Console/network/timeline values render as text. Replay controls are disabled or absent unless the authoritative `ReplayRecord` permits use.

- [ ] **Step 6: Implement route handling**

Support `/` and `/runs/:runId`. Unknown local routes show a ReproDocket not-found state with a working return action. Browser back/forward must work.

- [ ] **Step 7: Prove GREEN**

```powershell
npm run build
npm run test:ui -- RunHistory.spec.ts RunDetail.spec.ts
```

- [ ] **Step 8: Run malicious text UI test**

Add H034: target/problem/console text that resembles HTML remains text. Confirm no injected element/script exists in DOM.

- [ ] **Step 9: Commit**

```powershell
git add reprodocket/src/client reprodocket/tests/ui/RunHistory.spec.ts reprodocket/tests/ui/RunDetail.spec.ts
git diff --cached --check
git commit -m "feat: show durable run evidence locally"
```

---

## Task 8: Implement active-state rehydration and SSE progress UI

**Files:**
- Create: `reprodocket/src/client/api/RunEventClient.ts`
- Create: `reprodocket/src/client/components/ActiveProgress.tsx`
- Modify: `reprodocket/src/client/pages/RunPage.tsx`
- Test: `reprodocket/tests/ui/ActiveRunProgress.spec.ts`

**Interfaces:**
- Consumes SSE snapshot/events and authoritative GET run API.

- [ ] **Step 1: Write failing H009-H011 tests**

Simulate persisted lifecycle transitions through a controlled server-side test driver. Require UI progress update without refresh, then reload the browser during active state and require rehydration from GET.

- [ ] **Step 2: Prove RED**

```powershell
npm run build
npm run test:ui -- ActiveRunProgress.spec.ts
```

- [ ] **Step 3: Implement event client**

On open, accept snapshot. On error/reconnect, fetch the run manifest before trusting later events. Reject lower-sequence stale client updates.

- [ ] **Step 4: Implement active progress**

Show current lifecycle and timeline summaries. Use an ARIA live region with polite, bounded announcements rather than announcing every low-level network event.

- [ ] **Step 5: Prove GREEN**

```powershell
npm run build
npm run test:ui -- ActiveRunProgress.spec.ts
```

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/client/api/RunEventClient.ts reprodocket/src/client/components/ActiveProgress.tsx reprodocket/src/client/pages/RunPage.tsx reprodocket/tests/ui/ActiveRunProgress.spec.ts
git diff --cached --check
git commit -m "feat: rehydrate live ReproDocket progress"
```

---

## Task 9: Implement restart recovery and interrupted-run presentation

**Files:**
- Create: `reprodocket/src/storage/RunRecovery.ts`
- Modify: `reprodocket/src/server/server.ts`
- Modify: `reprodocket/src/client/pages/RunPage.tsx`
- Test: `reprodocket/tests/integration/RunRecovery.test.ts`
- Test: `reprodocket/tests/ui/InterruptedRun.spec.ts`

**Interfaces:**
- Produces startup recovery that marks stale nonterminal prior-process runs INTERRUPTED without replaying actions.

- [ ] **Step 1: Write failing B009/F013/M017 tests**

Seed a prior-process run in INVESTIGATING with a different instance identity. Start recovery and require INTERRUPTED, preserved evidence IDs, and no adapter/browser invocation.

- [ ] **Step 2: Write UI test**

Interrupted run shows what was captured, what did not finish, cleanup uncertainty if any, and a working path back to New Investigation. It does not show NOT_REPRODUCED.

- [ ] **Step 3: Prove RED**

```powershell
npm run test:integration -- RunRecovery.test.ts
npm run build
npm run test:ui -- InterruptedRun.spec.ts
```

- [ ] **Step 4: Implement recovery**

Scan only the application-owned run root. Validate each run independently. Transition only stale nonterminal runs from a prior process. Do not automatically resume user actions in version one.

- [ ] **Step 5: Prove GREEN**

Run the same commands and require PASS.

- [ ] **Step 6: Commit**

```powershell
git add reprodocket/src/storage/RunRecovery.ts reprodocket/src/server/server.ts reprodocket/src/client/pages/RunPage.tsx reprodocket/tests/integration/RunRecovery.test.ts reprodocket/tests/ui/InterruptedRun.spec.ts
git diff --cached --check
git commit -m "feat: recover interrupted ReproDocket runs"
```

---

## Task 10: Close M2 with horizontal connectivity tests

**Files:**
- Create: `reprodocket/tests/integration/HorizontalConsistency.test.ts`
- Create: `reprodocket/tests/ui/HorizontalConsistency.spec.ts`
- Create: `reprodocket/docs/connectivity-matrix.md`
- Modify: `reprodocket/scripts/validate.ps1`

**Interfaces:**
- Adds N001-N015 local-applicable checks to Targeted validation.
- Creates the maintained visible-feature connectivity matrix.

- [ ] **Step 1: Write consistency tests**

Use one sealed test run and assert history summary, run detail JSON, report JSON, HTML report parsed text, artifact ownership, replay state, and UI outcome all agree on the same run ID/outcome.

- [ ] **Step 2: Prove RED before adding missing production wiring**

```powershell
npm run test:integration -- HorizontalConsistency.test.ts
npm run build
npm run test:ui -- HorizontalConsistency.spec.ts
```

- [ ] **Step 3: Repair any discovered inconsistency in the earliest shared boundary**

Do not patch each UI surface independently when the root defect is serialization/store projection.

- [ ] **Step 4: Create the initial connectivity matrix**

Rows must include at least:

```text
Solari credential setup
New Investigation form
Active run progress
History
Run detail
Screenshot evidence
Console evidence
Network evidence
Timeline evidence
Replay state
HTML report
Interrupted run state
```

Columns are the contract fields defined in the design: discoverability, entry, prerequisites, action routing, feedback, authority, persistence, reload, recovery, exit/return, regression, human QA.

At this phase, live investigation rows may truthfully remain IN_PROGRESS because Solari execution arrives in later plans.

- [ ] **Step 5: Add Plan 2 stages to Targeted validation**

Include redaction, artifact integrity, report safety, history/API, SSE, UI detail, recovery, and local horizontal consistency.

- [ ] **Step 6: Run fresh targeted validation**

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
```

Expected: all Plan 1 and Plan 2 implemented authorities PASS. Full remains blocked on live Solari and complete execution.

- [ ] **Step 7: Original-resolution visual inspection**

Capture and inspect at least New Investigation, representative Run Detail, History, evidence tabs, and Interrupted state. Fix clipping, stale labels, contradictory status treatment, dead controls, or unreadable evidence before M2 is accepted.

- [ ] **Step 8: Commit**

```powershell
git add reprodocket/tests/integration/HorizontalConsistency.test.ts reprodocket/tests/ui/HorizontalConsistency.spec.ts reprodocket/docs/connectivity-matrix.md reprodocket/scripts/validate.ps1
git diff --cached --check
git commit -m "test: close ReproDocket local evidence integration"
```

---

## Plan 2 completion gate

Before Plan 3:

```text
redaction runs before durable text persistence
run artifacts cannot cross ownership boundaries
final evidence is hash-sealed and tampering is detected
malicious text cannot execute in HTML report or React UI
history survives damaged sibling runs
SSE reconnect returns to durable authority
active run UI can rehydrate after refresh
interrupted prior-process runs do not auto-replay actions
history/detail/report/evidence surfaces agree horizontally
all visible local-only controls are connected or truthfully unavailable
Plan 1 and Plan 2 targeted validation is freshly green
```

Verify:

```powershell
.\reprodocket\scripts\validate.ps1 -Profile Targeted
git status --short
git diff main...HEAD --check
```

Do not begin billable live Solari integration while any local deterministic authority is red.