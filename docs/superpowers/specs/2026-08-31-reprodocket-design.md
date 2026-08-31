# ReproDocket Design Specification

Date: August 31, 2026
Status: Approved design baseline
Repository: `bconnell/solari-cookbook`
Branch: `feature/reprodocket`

## 1. Purpose

ReproDocket is a local evidence first web defect investigation product built inside the Solari cookbook fork. A user supplies a target URL, a problem description, and optional known reproduction steps. ReproDocket drives real Solari cloud browsers to investigate the problem, captures evidence, starts a completely fresh Solari browser to independently verify the reproduction, and presents the complete result locally without requiring the user to configure a database, manually wire services, inspect raw artifact folders, or navigate the Solari dashboard to understand what happened.

The product exists to demonstrate a strong, concrete Solari use case for the Solari internship challenge while remaining small enough to finish quickly and rigorously.

## 2. Challenge alignment

The design preserves the upstream Solari cookbook and adds ReproDocket as a first class project under `reprodocket/`. The repository remains a recognizable fork and retains upstream license and attribution. Solari is central to the actual product behavior rather than appearing as a decorative dependency.

The shipped investigation path uses real Solari browser execution. The full validation path also uses real Solari execution. Isolated unit tests may use doubles where isolation is appropriate, but mocks and fixtures cannot satisfy live integration or end to end completion gates.

## 3. Product boundary

Version one supports one complete job exceptionally well:

1. Accept a web target and defect report.
2. Create an actual Solari browser session with recording enabled.
3. Navigate and execute the intended workflow.
4. Capture meaningful screenshots, console failures, page errors, network failures, action timing, URLs, and state observations.
5. Derive or preserve reproduction steps.
6. Close the first browser and release all owned resources.
7. Start a new independent Solari browser session.
8. Repeat the minimal reproduction workflow.
9. Compare investigation and verification evidence.
10. Produce a truthful outcome.
11. Persist the run locally.
12. Show the result, evidence, replay access, logs, timeline, and history in the local ReproDocket interface.

Version one is not a general browser agent, test case management suite, Jira replacement, hosted SaaS, team account system, billing product, CI platform, or generic AI framework.

## 4. Decision hierarchy

Engineering decisions use this precedence:

1. The Solari challenge requirements.
2. Current project acceptance and Gold Standard requirements.
3. Reliability, security, truthful evidence, and fastest credible delivery.
4. User technology and workflow preferences when they do not weaken the first three.

A prior design choice is not preserved merely because it was previously chosen. If implementation evidence proves another approach materially better, the change must be documented, tested, and evaluated against the same completion gates.

## 5. Locked technology choices

### 5.1 Primary language and runtime

ReproDocket uses TypeScript on Node.js. This deliberately follows Solari's strongest first party JavaScript and TypeScript browser path and avoids introducing an unnecessary C# bridge around the Solari SDK.

The repository commits `package-lock.json`. Dependency resolution for a validated release is therefore reproducible. Dependency versions are selected from the current compatible Solari SDK and proven by live smoke tests rather than copied blindly from an older example.

### 5.2 Local server

Fastify provides the local HTTP service. The production style local application serves the built React UI, API routes, evidence files, and Server Sent Events from one Node process.

The normal application binds only to loopback by default.

### 5.3 Local UI

React with Vite provides the local interface. Electron, Next.js, Docker, and a separate database server are excluded from the initial product because they do not improve the challenge outcome enough to justify their setup and failure surface.

### 5.4 Progress transport

Server Sent Events provide server to browser progress updates. The application does not introduce WebSockets unless bidirectional realtime requirements later prove necessary.

### 5.5 Persistence

The first version uses filesystem based run storage. Each run is self contained and machine readable. A database is not required for startup, installation, migration, or recovery.

Normal run data lives under the current user's local application data directory rather than the Git repository.

### 5.6 Secrets

The normal Windows experience stores the Solari API key using a Windows user scoped protected secret store. The key never belongs in source control, evidence output, screenshots, exception serialization, or public documentation.

Environment variable support may remain available for development and automated environments, but a normal user is not required to create or edit an `.env` file.

## 6. Repository structure

```text
solari-cookbook/
  examples/                       existing upstream cookbook
  LICENSE                         existing upstream license
  README.md                       upstream overview plus ReproDocket entry
  docs/
    superpowers/
      specs/
        2026-08-31-reprodocket-design.md
  reprodocket/
    package.json
    package-lock.json
    tsconfig.json
    vite.config.ts
    eslint.config.js
    src/
      client/
        App.tsx
        main.tsx
        pages/
        components/
        api/
      server/
        server.ts
        routes/
        sse/
        startup/
      core/
        InvestigationEngine.ts
        InvestigationPlanner.ts
        VerificationEngine.ts
        RunCoordinator.ts
        OutcomeClassifier.ts
        models/
      solari/
        SolariBrowserFactory.ts
        SolariInvestigator.ts
        SolariRecorder.ts
        SolariSandboxFixture.ts
        SessionTracker.ts
      evidence/
        EvidenceCollector.ts
        ScreenshotCollector.ts
        ConsoleCollector.ts
        NetworkCollector.ts
        EvidenceHasher.ts
        Redactor.ts
      storage/
        RunStore.ts
        FileRunStore.ts
        RunManifestSchema.ts
      security/
        SecretStore.ts
        WindowsDpapiSecretStore.ts
      shared/
        contracts/
        errors/
    fixtures/
      defective-site/
    tests/
      unit/
      integration/
      ui/
      live/
      e2e/
    scripts/
      bootstrap.ps1
      preflight.ps1
      run.ps1
      validate.ps1
      stop.ps1
      clean.ps1
    docs/
      architecture.md
      validation.md
```

Files and modules should remain small enough that their responsibility is obvious from their public interface. Unrelated refactoring is out of scope, but files that become multi responsibility bottlenecks during this work must be split as part of the affected slice.

## 7. User experience

### 7.1 Zero configuration normal startup

The normal entry point is:

```powershell
.\reprodocket\scripts\run.ps1
```

The script determines the repository root, validates prerequisites, repairs safely repairable missing prerequisites, restores dependencies, builds when required, starts one owned local ReproDocket instance, verifies health, and opens the user's default browser.

The user does not configure ports, create a database, edit frontend configuration, create environment files, locate artifact directories, or start separate frontend and backend terminals.

If the preferred port is occupied by another process, ReproDocket chooses another port. It does not terminate an unrelated process.

If an already running ReproDocket instance can be authoritatively identified as belonging to this repository instance, `run.ps1` opens that instance rather than creating a duplicate.

### 7.2 First run credential flow

If no Solari credential is available, the local UI presents a Solari connection screen. Saving the key performs a real harmless credential verification before reporting success. A failure remains visible with a useful reason and does not store a credential as validated.

### 7.3 Main screens

The MVP exposes three complete screens and no decorative dead surfaces.

1. New Investigation.
2. Active Investigation.
3. Run Detail and History.

Every visible button, tab, menu, link, badge, and action must either complete a real workflow, be truthfully disabled with a current reason, or not appear.

## 8. Investigation request

The initial user request contains:

1. Target URL.
2. Problem description.
3. Optional known reproduction steps.

Only supported URL schemes are accepted. The browser investigation boundary accepts ordinary HTTP and HTTPS targets. File, data, JavaScript, and other executable or local schemes are rejected.

The initial product contains an `InvestigationPlanner` boundary so runtime planning can evolve without contaminating evidence collection and browser lifecycle code. A second hosted AI provider is not a mandatory dependency for the challenge version. If a future planner is added, its output remains auditable and cannot independently create a VERIFIED result without observed evidence.

## 9. Run lifecycle

Lifecycle and outcome are separate authorities.

Lifecycle values:

* CREATED
* PREPARING
* INVESTIGATING
* VERIFYING
* FINALIZING
* COMPLETED
* FAILED
* CANCELLED
* INTERRUPTED

Outcome values:

* VERIFIED
* REPRODUCED
* NOT_REPRODUCED
* INCONCLUSIVE

A completed run may truthfully be NOT_REPRODUCED. A failed run must never be silently mapped to NOT_REPRODUCED. An unavailable dependency, infrastructure failure, blocked authentication path, or ambiguous report produces an appropriate lifecycle failure or INCONCLUSIVE result rather than a false healthy conclusion.

Every lifecycle transition is persisted before dependent work assumes the transition occurred.

## 10. Investigation data flow

The production flow is:

```text
Local UI
  -> Fastify API
  -> RunCoordinator
  -> InvestigationEngine
  -> SolariBrowserFactory
  -> real Solari browser
  -> EvidenceCollector
  -> FileRunStore
  -> VerificationEngine
  -> second real Solari browser
  -> Evidence comparison
  -> OutcomeClassifier
  -> finalized manifest and report
  -> Fastify API and SSE
  -> Local UI
```

The same authoritative run identity follows the workflow from creation through evidence display. No UI surface synthesizes a successful state that is absent from the persisted run authority.

## 11. Horizontal and vertical integration requirements

### 11.1 Vertical integration

Every supported user outcome must be proven through the entire stack, not only at one layer.

For a normal investigation, proof must traverse:

1. User input in the actual local UI.
2. API validation.
3. Coordinator state creation.
4. Solari session creation.
5. Browser action execution.
6. Evidence collection.
7. Persistent storage.
8. Independent verification.
9. Final classification.
10. UI projection of the exact persisted result.
11. Run reload after restart.
12. Resource cleanup and user return path.

A test that exercises only the core engine, API, or Solari adapter cannot satisfy vertical completion.

### 11.2 Horizontal integration

Sibling product surfaces that describe the same run must agree and remain connected to the same authoritative state.

Horizontal checks include:

* Active Investigation status versus persisted lifecycle state.
* Run History status versus Run Detail status.
* Outcome badge versus report outcome.
* Screenshot list versus manifest evidence entries.
* Console and network views versus their persisted artifacts.
* Replay control versus actual replay availability.
* Session cleanup status versus tracked Solari resource state.
* Settings and credential readiness versus the runtime provider path.
* Validation summaries versus the exact current source revision and evidence set.
* README capability claims versus the actually proven feature maturity.

A defect in shared routing, storage, state projection, serialization, or lifecycle infrastructure triggers review of adjacent sibling paths that use the same infrastructure.

### 11.3 Connectivity matrix

Before a milestone can graduate, Codex maintains a connectivity matrix for every visible feature. Each row records:

* Discoverability.
* Entry point.
* Prerequisite truth.
* Action routing.
* Feedback.
* Authoritative state.
* Persistence.
* Reload.
* Recovery.
* Exit or return path.
* Automated regression coverage.
* Human QA requirement if any.

Rows may be COMPLETE, PARTIAL, DISCONNECTED, DEAD, STALE, INTENTIONALLY_DISABLED, DEFERRED, or UNKNOWN. UNKNOWN and PARTIAL visible behavior block a broad completion claim until investigated.

## 12. Solari browser lifecycle

Every investigation browser is a real Solari browser. Recording is enabled when the current SDK supports the required recorded session path.

The first investigation and the independent verification use different Solari session identities. Reusing the original browser, page, context, or session is prohibited for verification.

Resource creation is recorded immediately. Cleanup occurs in `finally` style ownership boundaries and cleanup results are recorded. A test is not considered lifecycle clean if functional assertions pass while an owned Solari browser or sandbox remains active unexpectedly.

## 13. Evidence model

Evidence is captured at meaningful semantic boundaries rather than indiscriminately.

For browser actions, ReproDocket records the fields needed to reconstruct the workflow, including timestamp, action, target description where available, URL before and after, result, and duration.

Evidence sources include:

* Screenshots.
* Console errors and warnings.
* Uncaught page errors.
* Relevant failed or erroneous network requests.
* Action timeline.
* Session identity.
* Replay availability and replay reference.
* Investigation and verification observations.

Default evidence capture excludes secrets, authorization headers, cookies, passwords, payment data, unrestricted request bodies, and unrelated browser storage.

A redaction layer runs before sensitive values can enter durable evidence or UI projection.

## 14. Evidence integrity and provenance

Each durable artifact receives a SHA256 digest. The final run manifest records artifact identity and digest. The UI reads evidence through the run store and verifies artifact identity rather than selecting a convenient file by name or timestamp.

The validation report records the exact source revision, validation profile, run identity, fixture version where applicable, and important runtime versions.

Historical artifacts do not satisfy a fresh validation gate after relevant source changes.

## 15. Outcome classification

VERIFIED requires that the alleged defect was observed during the investigation and independently reproduced in the clean verification session.

REPRODUCED means the first investigation observed the defect but the clean verification session did not establish it again.

NOT_REPRODUCED means the system completed enough of the intended workflow to test the allegation and did not observe the alleged defect.

INCONCLUSIVE means trustworthy determination is not possible because the workflow is blocked, ambiguous, unavailable, externally interrupted, or otherwise lacks sufficient authoritative evidence.

When evidence conflicts, uncertainty wins over fabricated confidence.

## 16. Deterministic fixture system

The full harness uses a controlled defective web application hosted inside a real Solari sandbox. The sandbox exposes the fixture through a Solari preview URL. Real Solari browsers then investigate that public fixture URL.

Fixture production logic remains separate from ReproDocket production behavior. ReproDocket must not branch on fixture identity or contain special answers for seeded defects.

Initial fixture scenarios include at least:

| Scenario | Ground truth | Expected result |
| --- | --- | --- |
| Account save produces blank panel | Deterministic defect | VERIFIED |
| Billing route refresh becomes 404 | Deterministic defect | VERIFIED |
| Missing ZIP validation submits | Deterministic defect | VERIFIED |
| Healthy profile save | Healthy flow | NOT_REPRODUCED |
| Healthy login validation | Healthy flow | NOT_REPRODUCED |
| Deliberately ambiguous report | Insufficient evidence | INCONCLUSIVE |

The fixture set includes positive, negative, and boundary cases so an implementation that labels everything broken or everything healthy cannot pass.

## 17. Validation authorities

### 17.1 Static quality

Required checks include TypeScript compilation, lint, formatting policy where adopted, package integrity, and secret scanning appropriate to the repository.

### 17.2 Unit validation

Unit tests cover pure and isolated behavior such as outcome classification, lifecycle transitions, manifest validation, evidence hashing, redaction, URL validation, state guards, and storage path safety.

### 17.3 Local integration validation

Integration tests cover Fastify routes, SSE delivery, run persistence, interrupted run recovery, evidence serving, local startup state, secret store boundaries, and server shutdown behavior.

### 17.4 Local UI validation

The actual ReproDocket UI is launched and exercised through its normal browser boundary. Tests cover onboarding, new investigation entry, live status, run history, run detail, evidence rendering, replay state, failures, disabled states, restart and reload, and critical readability.

### 17.5 Live Solari validation

Live tests create actual Solari resources and prove session creation, navigation, interaction, screenshot collection, event collection, recording behavior, replay readiness behavior, release, and cleanup.

Mocks cannot satisfy this authority.

### 17.6 Full end to end validation

The full authority performs the whole production shaped path:

1. Start the local ReproDocket application.
2. Create the real Solari sandbox fixture.
3. Start the fixture server.
4. Obtain and verify its public preview URL.
5. Enter a scenario through the local user facing workflow.
6. Create a real Solari investigation browser.
7. Reproduce or reject the seeded condition.
8. Persist investigation evidence.
9. Close the first browser.
10. Create a different real Solari verification browser.
11. Execute verification.
12. Persist verification evidence.
13. Finalize classification and integrity metadata.
14. Verify the result appears correctly in the actual local UI.
15. Restart or reload as required and prove run persistence.
16. Release browsers and kill the owned fixture sandbox.
17. Verify cleanup.
18. Verify evidence provenance and fresh source identity.

## 18. Validation profiles

TARGETED validation is permitted for bounded implementation slices when the claim is narrow. It runs the smallest sufficient real checks for the affected behavior.

FULL validation is mandatory for milestone completion, broad integration claims, hardening completion, final product completion, and challenge submission.

Running `validate.ps1` with no narrowing option means FULL validation.

A failed mandatory stage returns a nonzero exit code and blocks a completion claim.

## 19. Bug finding pass

Functional implementation completion does not immediately graduate the product to final completion.

After the complete vertical slice exists, Codex performs a deliberate product wide bug finding pass covering every visible feature and supported user workflow. The pass is distinct from ordinary development testing and includes:

* Visible feature inventory.
* Dead control search.
* Disconnected path search.
* Stale state search.
* Navigation trap search.
* Persistence and reload search.
* Contradictory label search.
* Error state review.
* Race and duplicate action review.
* Resource lifetime review.
* Secret and sensitive data leakage review.
* Incorrect cross run evidence review.
* Local process ownership review.
* Solari session and sandbox leak review.
* Restart and interruption review.
* Browser back, refresh, and reopen behavior where applicable.
* Public documentation versus runtime truth review.

Every discovered defect is classified by root boundary. Shared infrastructure defects require adjacent consumer review rather than patching only the first visible symptom.

## 20. Hardening pass

Hardening is proportional to the actual risk and supported scope. The project does not add enterprise security theater or speculative infrastructure merely to appear mature.

Mandatory hardening boundaries include:

### 20.1 Local service security

* Bind to loopback by default.
* Reject unsupported target URL schemes.
* Validate mutation request origin or use an equivalent local session protection mechanism.
* Prevent arbitrary artifact path traversal.
* Do not expose secrets through API responses.
* Apply reasonable input length and shape limits.

### 20.2 Secret protection

* Protect stored Solari credentials using the operating system user boundary.
* Redact credentials and sensitive headers from logs and evidence.
* Verify public repository scans do not contain a real credential.

### 20.3 Run and evidence safety

* Use run scoped directories and generated identifiers.
* Prevent one run from overwriting another run's evidence.
* Write manifests and finalization state atomically where practical.
* Preserve incomplete runs as INTERRUPTED rather than deleting them silently.
* Detect malformed or tampered manifests and fail closed.

### 20.4 Duplicate and concurrency behavior

The initial supported scope allows one actively executing investigation pipeline per local application instance unless live testing proves safe parallel execution necessary for the challenge. Additional user requests are not silently discarded. They are rejected with an explicit current reason or queued through a deliberately implemented queue if that feature is completed. No hidden limit is permitted.

### 20.5 Resource ownership

* Track every owned Solari browser and sandbox.
* Track the owned local server process.
* Never kill unrelated processes to free a port.
* Verify process identity before stop operations.
* Reconcile cleanup after cancellation, exceptions, and shutdown.

## 21. Failure injection

The harness deliberately introduces representative failures and verifies the resulting lifecycle, classification, diagnostics, persistence, and cleanup.

Required initial failure cases include:

* Missing credential.
* Invalid credential.
* Solari browser creation failure.
* Navigation timeout.
* Target HTTP failure.
* Browser failure during investigation.
* Local evidence write failure.
* Verification browser unavailable.
* Recording not immediately ready.
* Sandbox creation failure.
* Fixture server startup failure.
* Local preferred port occupied.
* Application interruption during an active run.
* Malformed persisted run.
* Missing evidence artifact.
* Cross run artifact mismatch.

The test expectation is not merely that an exception occurred. Each scenario defines expected user visible behavior, durable state, resource cleanup, and resumable next action where applicable.

## 22. Second validation pass

After the deliberate bug finding and hardening passes, the product is validated again from a fresh current source state.

The second validation is not allowed to reuse stale success evidence from before hardening. Any change to shared lifecycle, evidence, storage, capture, state projection, or validation infrastructure invalidates dependent prior evidence and requires the relevant profile to be rerun.

Final challenge readiness therefore follows this order:

```text
front to back implementation
  -> integration closure
  -> FULL validation
  -> deliberate bug finding audit
  -> bounded repairs
  -> targeted regression for each repair
  -> proportional hardening
  -> fresh FULL validation
  -> human visual and usability review
  -> any escaped defect repairs plus harness coverage repairs
  -> fresh affected validation
  -> final FULL validation
  -> documentation and publication audit
```

## 23. Visual and usability proof

ReproDocket itself is part of the product under test. Backend success does not prove that the local result is understandable or usable.

Milestone evidence includes the real current interface for:

* First run connection state.
* New Investigation.
* Active Investigation.
* VERIFIED result.
* REPRODUCED result where a deterministic scenario can produce it.
* NOT_REPRODUCED result.
* INCONCLUSIVE result.
* History.
* Evidence detail.
* Relevant error states.

Automated checks cover deterministic facts such as overflow, missing critical controls, incorrect outcome state, missing evidence, and broken navigation. Human review remains a separate authority for subjective readability and polish.

Background safe browser automation is preferred. Automation must not move the user's mouse, type into unrelated active applications, or silently escalate to foreground ownership. Foreground interaction is used only where the exact native behavior requires it and must be explicit and minimal.

## 24. Bootstrap and preflight

The small bootstrap entry point remains capable of starting on a normal Windows system where only Windows PowerShell may initially be present. It detects PowerShell 7, installs or routes to it when safely possible, and performs substantive work under PowerShell 7.

Preflight validates actual behavior rather than executable presence. It checks supported runtime versions, command execution, package tooling, disk capacity, write access, dependency restoration, application startup capability, credential readiness, and real Solari connectivity where the selected profile requires it.

WMIC is prohibited. Modern PowerShell, CIM, and appropriate .NET APIs are used.

A failed preflight explains the exact blocking condition and safe remediation.

## 25. Local process ownership

The running local application writes an ownership record containing enough information to distinguish its process from a recycled PID or unrelated process. This includes process identifier, process start identity, port, repository or application identity, and an instance nonce or equivalent.

`stop.ps1` verifies ownership before terminating anything.

## 26. Run storage

Normal user data is stored outside the repository under the user local application data path.

Conceptual layout:

```text
ReproDocket/
  secrets/
  runs/
    <run-id>/
      run.json
      investigation/
        screenshots/
        console.json
        network.json
        timeline.json
        replay.json
      verification/
        screenshots/
        console.json
        network.json
        timeline.json
        replay.json
      report/
        report.html
        report.json
      integrity.json
```

Generated validation artifacts use scoped ignored output and remain bounded. Routine source control does not accumulate screenshots, recordings, browser dumps, or transient logs.

## 27. Reporting

The local report is understandable without terminal history or hidden reasoning. It states:

* Target.
* Reported problem.
* Lifecycle result.
* Investigation result.
* Verification result.
* Final outcome.
* Reproduction steps.
* Key evidence.
* Replay availability.
* Session cleanup result.
* Run duration.
* Relevant limitations or blockers.

Machine readable JSON and human readable HTML are generated from the same authoritative run data.

## 28. Documentation truth

README, ReproDocket documentation, capability wording, UI state, validation output, and challenge submission copy must agree.

A feature is not documented as available merely because a class, route, button, unit test, or partial happy path exists.

Maturity terminology is:

* Planned.
* Foundation.
* In Progress.
* Available (Unhardened).
* Available.

Available requires complete supported workflow, integration, persistence and lifecycle behavior where applicable, failure handling, recovery, resource closure, validation, documentation, and no known category level hardening blocker.

## 29. Git and publication behavior

Implementation occurs on `feature/reprodocket` until an intentional integration decision is made.

Before each commit, Codex inspects repository state, stages only related files, verifies the required current validation, and checks the staged diff for whitespace and accidental artifacts. Generated evidence and secrets remain untracked.

A commit is not created merely to checkpoint a failing slice unless the repository policy explicitly requires otherwise. Normal descriptive commits are preferred.

Publication or merge must verify remote state, branch divergence, repository hygiene, challenge attribution, license preservation, and current evidence.

## 30. Milestones and gates

### M0 Foundation

Required before completion:

* Project structure established.
* Bootstrap and preflight executable.
* Build executable.
* Unit harness executable.
* Validation entry point executable.
* Generated output policy established.
* Failure exit codes proven.

### M1 Local shell

Required before completion:

* One command starts ReproDocket.
* One owned local process serves the complete application.
* Default browser opens automatically.
* First run and normal application surfaces are usable.
* Restart and stop ownership are proven.

### M2 Evidence core

Required before completion:

* Runs persist.
* Runs reload.
* Integrity metadata is produced.
* Interrupted state survives restart.
* Cross run evidence cannot be confused.
* Local UI displays persisted evidence.

### M3 Solari browser

Required before completion:

* Real Solari session creation proven.
* Navigation and interaction proven.
* Screenshot capture proven.
* Event collection proven.
* Recording and replay behavior proven to the current SDK boundary.
* Cleanup proven.

### M4 Fixture infrastructure

Required before completion:

* Real Solari sandbox created.
* Controlled fixture hosted.
* Public preview URL verified externally.
* Fixture lifecycle cleanup proven.

### M5 Investigation

Required before completion:

* Seeded defect workflows execute through the production investigation path.
* Positive and negative cases are distinguished.
* Evidence supports the observed first run result.

### M6 Independent verification

Required before completion:

* Second Solari session identity differs from first.
* Minimal reproduction is replayed cleanly.
* Comparison logic is real.
* VERIFIED cannot occur without evidence from both sessions.

### M7 Complete local results UI

Required before completion:

* Current run progress visible.
* Final outcome visible.
* Screenshots visible.
* Timeline visible.
* Console evidence visible.
* Network evidence visible.
* Replay state visible.
* History and reload visible.
* No dead controls or placeholder surfaces.

### M8 Bug finding and adversarial audit

Required before completion:

* Full visible feature inventory completed.
* Horizontal and vertical connectivity matrix completed.
* Failure injection suite completed.
* Disconnected and stale paths repaired or truthfully classified.
* Any harness gaps discovered by defects are repaired.

### M9 Hardening

Required before completion:

* Security and privacy boundaries reviewed.
* Resource ownership and lifetime reviewed.
* Persistence and interruption reviewed.
* Sensitive logging reviewed.
* Duplicate and state transition behavior reviewed.
* Proportional repairs complete.
* No known ordinary workflow hardening blocker remains inside declared scope.

### M10 Final proof

Required before completion:

* Fresh FULL validation passes after hardening.
* Product UI visual and usability review completed.
* Escaped defects produce both product fixes and corresponding regression coverage where automation could have caught them.
* A final fresh FULL validation passes after any such repairs.
* Exact source revision and evidence provenance recorded.
* Documentation truth audit passes.

### M11 Challenge submission

Required before completion:

* Upstream fork relationship and license preserved.
* Root README clearly surfaces ReproDocket without erasing the cookbook lineage.
* Setup is reproducible from a fresh checkout.
* No secret or private machine information is public.
* Current supported scope is stated truthfully.
* Demo uses the real current product and real Solari path.
* Public post copy follows the challenge instructions and tags the required accounts.

## 31. Completion definition

ReproDocket is front to back complete only when every visible supported feature is connected horizontally and vertically, the normal user can enter and complete the supported workflow, authoritative state reaches persistence and returns correctly to the UI, restart and recovery behavior works where applicable, real Solari resources are exercised, failures are diagnostic and safe, resources are closed, evidence is fresh and traceable, documentation agrees with runtime truth, and the complete current application survives the deliberate bug finding, hardening, and second validation sequence.

A successful build, large passing unit count, one working API request, one screenshot, one fixture result, or one successful Solari session is insufficient by itself.

## 32. Explicit stop conditions

Codex must stop broad feature expansion and address the current blocker when any of the following is true:

* A user visible path is broken or disconnected.
* The harness is incapable of failing for a known seeded defect.
* Evidence identity or provenance is uncertain.
* A secret may have leaked.
* Repository state is unknown and writing would risk unrelated work.
* An owned external resource cannot be distinguished from unrelated state.
* A shared infrastructure defect makes adjacent results untrustworthy.
* A required validation authority reports failure.
* A completion claim would rely on stale evidence.

Unknown or subjective residuals are reported truthfully rather than converted into PASS.

## 33. Final engineering sequence

The intended implementation sequence is:

```text
inspect exact fork state
  -> establish isolated project foundation
  -> build executable harness first
  -> complete zero configuration local shell
  -> complete evidence persistence
  -> complete live Solari browser lifecycle
  -> complete Solari hosted deterministic fixture
  -> complete investigation path
  -> complete independent verification path
  -> complete all local evidence views
  -> run FULL validation
  -> perform product wide bug finding and connectivity audit
  -> repair discovered defects with regressions
  -> harden only demonstrated risk boundaries and declared scope
  -> run fresh FULL validation
  -> perform human visual and usability review
  -> repair escaped defects and strengthen harness where applicable
  -> run fresh affected validation and final FULL validation
  -> reconcile documentation, repository, attribution, and publication state
  -> prepare challenge demonstration and post
```

The project does not skip directly from feature implementation to showcase preparation. Front to back completion, bug discovery, necessary hardening, and fresh revalidation are explicit phases of the product itself.