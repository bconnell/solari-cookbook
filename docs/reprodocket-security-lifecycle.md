# ReproDocket Security and Lifecycle Model

Date: August 31, 2026
Status: Pre-implementation security baseline

This document defines the minimum security, privacy, ownership, and recovery behavior required for ReproDocket version one. It is intentionally scoped to the actual product: a loopback-only local web application that accepts a target URL and drives remote Solari resources.

Security work must remain proportional. The product does not need enterprise identity, multi-tenant authorization, or speculative infrastructure. It does need strong boundaries around credentials, arbitrary target input, locally served mutation APIs, persisted evidence, remote resource lifecycle, and untrusted text rendered back into the UI or report.

## 1. Trust boundaries

ReproDocket crosses five meaningful trust boundaries:

1. The local user's browser to the ReproDocket loopback service.
2. User supplied target URL and defect text into the application.
3. ReproDocket to the Solari APIs and browser or sandbox resources.
4. Remote target content, console messages, network metadata, and screenshots back into local evidence.
5. Persisted local evidence back into the browser UI and generated HTML report.

Every externally supplied string should be treated as data rather than executable content.

## 2. Local service exposure

The local server binds to loopback only by default.

Allowed bind targets:

```text
127.0.0.1
::1
```

The application must not default to `0.0.0.0`, a LAN interface, or a public interface.

The server must reject unexpected Host values rather than relying only on the bind address. Accepted Host values are the actual selected loopback host and port used for the running instance.

CORS is disabled by default. No wildcard origin is allowed.

Mutation routes require same-origin requests. Requests with an `Origin` header that does not match the current local application origin are rejected.

A per-process random local session nonce is generated at startup. The normal UI receives it through a same-origin bootstrap path and includes it on state-changing API requests. The nonce is not persisted across application restarts and is not written to public logs or evidence.

The nonce is defense in depth against drive-by requests to a localhost service. It is not presented as user authentication.

## 3. HTTP response security

The local service should emit security headers appropriate to a loopback web application, including a restrictive Content Security Policy.

The initial policy should be compatible with the built Vite application without allowing arbitrary remote scripts. The application must not require `unsafe-eval` in the production style build.

Recommended baseline goals:

```text
Content-Security-Policy: default-src 'self'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; script-src 'self'; connect-src 'self'; frame-src 'none'; object-src 'none'; base-uri 'none'; form-action 'self'
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Cross-Origin-Resource-Policy: same-origin
```

If local replay rendering later requires a frame or additional source, the policy must be widened only for that proven use case.

## 4. Target URL policy

Version one accepts only `http:` and `https:` target URLs.

The following are rejected before a Solari session is created:

* `file:`
* `data:`
* `javascript:`
* `blob:` as a top-level target
* custom executable schemes
* URLs containing credentials in the authority component
* empty or malformed host names
* `localhost`
* `.localhost` names
* `.local` names
* loopback IP address literals
* link-local IP address literals
* private RFC1918 IPv4 address literals
* IPv6 loopback, unique-local, and link-local address literals
* known cloud metadata link-local destinations such as `169.254.169.254`

The reason is simple: a target is executed from a cloud browser. ReproDocket must not become a convenient interface for probing internal or metadata services from the provider environment.

If the host is a domain name, target policy must be rechecked after DNS resolution where the SDK and execution environment make that feasible. A redirect chain that lands on a prohibited local, private, or metadata destination must be rejected or aborted rather than treated as a valid target.

The deterministic test fixture uses a public Solari preview URL and therefore does not require an exception to this rule.

Version one does not provide a user override for blocked private network targets.

## 5. Target navigation policy

A run records the initial requested origin and the final navigated URL. Redirects are allowed between ordinary public HTTP and HTTPS destinations.

Navigation to a prohibited scheme or prohibited network destination during an automated action aborts that action and yields a truthful blocked or inconclusive result.

The system must not silently follow a download into a local executable action.

## 6. User supplied text

Problem descriptions, optional reproduction steps, target page titles, console messages, URLs, network error strings, and all target-derived evidence are untrusted text.

React views render them as text rather than raw HTML.

No component may use `dangerouslySetInnerHTML` with target-derived or run-derived content.

The generated standalone HTML report must use an escaping function for every untrusted value. Tests must include script tags, event-handler attributes, HTML entities, Unicode control characters, and strings resembling closing tags.

A report containing target text such as:

```text
</script><script>alert(1)</script>
```

must display that value as text and must not execute it.

## 7. Evidence collection minimization

ReproDocket captures enough evidence to explain the run without intentionally collecting secrets.

Default durable evidence may include:

* screenshots at defined semantic boundaries,
* console warnings and errors,
* uncaught page errors,
* request method,
* sanitized request URL,
* response status,
* request failure reason,
* timing,
* page URL and title,
* action description,
* session identity,
* replay availability,
* product-generated observations.

Default durable evidence must exclude:

* Authorization headers,
* Cookie and Set-Cookie headers,
* proxy credentials,
* Solari API keys,
* password field values,
* credit card values,
* unrestricted request bodies,
* unrestricted response bodies,
* browser cookies,
* localStorage and sessionStorage dumps,
* saved profile contents,
* arbitrary DOM serialization.

Screenshots can still contain sensitive information shown by the target site. The UI and documentation must make that limitation clear. ReproDocket does not claim screenshots are automatically privacy-safe.

## 8. Redaction

All text evidence passes through a central redaction boundary before durable persistence.

The redactor must recognize at least:

* exact current Solari API key value,
* values from known secret-bearing headers,
* obvious bearer token patterns,
* password field values known to the automation layer,
* query parameters explicitly classified as sensitive by the action layer.

Redaction must occur before logs are serialized, not only before the UI displays them.

The original secret must never be included in a redaction failure message.

Tests must deliberately inject a known fake key and verify it is absent from every generated JSON, HTML, log, and validation summary artifact.

## 9. Solari API key storage

The normal Windows path uses the current Windows user boundary.

The preferred zero-extra-dependency implementation is a small PowerShell 7 helper using `.NET` protected data APIs with `CurrentUser` scope. Node communicates the plaintext secret through the child process standard input rather than as a command-line argument.

The helper returns only encrypted bytes for storage and plaintext only through standard output to the owning Node process during a deliberate decrypt operation.

The stored encrypted value resides under the user's local application data directory.

No secret is stored in:

* `.env`,
* package scripts,
* repository config,
* command-line arguments,
* process ownership metadata,
* run manifests,
* validation artifacts.

`SOLARI_API_KEY` remains an optional developer and automation source. When both environment and stored credential exist, the documented precedence must be deterministic and visible in diagnostics without showing the secret value.

Recommended precedence:

```text
explicit process environment -> protected local store -> unavailable
```

The UI may identify the source as `Environment`, `Protected local store`, or `Not configured`.

## 10. Credential verification

Saving a credential does not immediately mark the provider READY.

The application performs a real harmless Solari control-plane or short-lived browser readiness operation appropriate to the current SDK.

A failed verification leaves the status unready and provides a sanitized reason.

If the verification requires creating a browser session, that session must be closed immediately and ownership must be proven in tests.

## 11. Local filesystem boundaries

All application-owned persistent paths are derived from one known application root under local application data.

Run IDs are generated internally. User input cannot select a filesystem path.

Artifact serving routes accept a run identity and an artifact identity from a validated manifest. They do not concatenate arbitrary user supplied path strings.

Any path resolution that escapes the configured run root is rejected.

Symbolic links or reparse points that escape an application-owned evidence root must not be followed for artifact serving or cleanup.

Cleanup never targets Desktop, Documents, Downloads, source repositories, arbitrary temporary directories, or a caller supplied path.

## 12. Run directory ownership

Each run owns exactly one generated directory.

A run may create only beneath its own directory except for intentionally shared immutable application metadata.

One run cannot overwrite another run's manifest or artifacts.

Creation uses fail-if-exists semantics for the generated run directory.

## 13. Atomic persistence

Lifecycle transitions that later logic depends on must be durably written before that logic proceeds.

For critical JSON state, write to a sibling temporary file, flush where practical, then atomically replace or rename to the authoritative filename.

A crash during write must leave either the previous valid manifest or a detectable incomplete temporary file. It must not silently create valid-looking truncated JSON.

On startup, incomplete temporary files are diagnosed and the owning run is marked or recovered according to the last valid durable state.

## 14. Manifest validation

Persisted manifests are untrusted on reload because users, crashes, old versions, or disk faults may alter them.

The application validates manifests against a versioned schema before using them.

Malformed manifests are not executed or treated as valid run truth.

A malformed historical run may remain visible as a damaged run entry with diagnostics, but the application must not crash the entire history view because one run is corrupt.

## 15. Evidence integrity

Every final durable evidence artifact receives a SHA256 digest recorded in the integrity manifest.

The application verifies an artifact's identity before displaying or exporting it from a finalized run.

Digest mismatch yields `EVIDENCE_INVALID` or an equivalent explicit state. It must not silently display the mismatched artifact under a valid evidence label.

Partial active runs may display newly created artifacts before final integrity sealing, but the UI must distinguish active/unsealed evidence from finalized evidence.

## 16. HTML report generation

The HTML report is generated only from validated authoritative run state.

The report generator escapes every untrusted field and references only artifacts owned by the run.

The report must be readable without JavaScript. JavaScript is unnecessary for the first report format and should not be embedded.

This makes the report safer to open and easier to archive.

## 17. Remote resource ownership

Every Solari resource created by ReproDocket is registered immediately in a resource ledger before dependent work begins.

Minimum ledger fields:

```text
resourceType
resourceId
owningRunId
createdAt
releaseRequestedAt
releaseConfirmedAt
lastKnownState
cleanupError
```

Resource identity is not inferred from a list of all account resources when an explicit creation result is available.

ReproDocket cleans up only resources it can prove it owns.

## 18. Browser lifecycle

The normal browser lifecycle is:

```text
create -> register ownership -> use -> stop evidence capture -> close -> confirm local adapter completion
```

Every creation path uses `try/finally` or an equivalent structured cleanup boundary.

The investigation browser and verification browser are independent owned resources.

A successful run cannot claim cleanup PASS until both have reached their required terminal state.

## 19. Sandbox lifecycle

Harness-owned Solari sandboxes exist only to host deterministic fixtures.

The sandbox lifecycle is:

```text
create -> register ownership -> connect -> deploy fixture -> start server -> verify command exit code -> obtain preview URL -> externally probe preview -> use -> kill -> confirm cleanup result
```

The harness explicitly kills the sandbox. It does not rely on the provider's idle pause or timeout behavior.

If `kill()` fails, functional scenario assertions may still be recorded, but the lifecycle gate fails and the validation run is not FULL PASS.

## 20. Local process ownership

The ReproDocket local server writes an ownership record into an application-owned runtime directory.

Minimum identity fields:

```text
pid
processStartTime
port
bindAddress
repositoryOrBuildIdentity
instanceNonce
startedAt
```

`stop.ps1` reads the record and confirms the current process still matches the recorded identity before requesting termination.

PID equality alone is not sufficient because operating systems reuse process IDs.

If ownership cannot be proven, the script reports the condition and leaves the process untouched.

## 21. Port selection

The application begins with a preferred loopback port but treats it as a preference, not ownership.

If another process already owns that port, ReproDocket selects an available loopback port and records it.

It never terminates an unrelated process merely to claim the preferred port.

## 22. Active run concurrency

Version one supports one executing investigation pipeline per ReproDocket process.

This is an explicit supported capacity, not a hidden cap.

When a run is active, a second submission receives an explicit `RUN_ALREADY_ACTIVE` response and the UI explains that the current run must finish or be cancelled before another starts.

Version one does not silently queue requests. A queue should not exist until it is fully implemented and tested.

The restriction applies only to active execution. Historical run browsing remains available while a run is active.

## 23. Duplicate submissions

The Investigate action becomes disabled after the server accepts the request and returns the authoritative run identity.

Client-side disabling is not the only guard. The server enforces the active-run state atomically so rapid double clicks or duplicate requests cannot create multiple pipelines.

Tests must race two submissions and prove only one external pipeline can be admitted.

## 24. Cancellation

Cancellation is part of the supported lifecycle rather than a process kill shortcut.

A cancellation request:

1. identifies the currently owned run,
2. records cancellation requested,
3. stops scheduling new browser actions,
4. allows active action cleanup to settle within a bounded time,
5. closes owned browser sessions,
6. kills an owned fixture sandbox if one belongs to that run,
7. persists the terminal `CANCELLED` lifecycle state,
8. records cleanup failures without converting them to success.

If cancellation cannot finish safely within the timeout, the run is not falsely marked clean. The application reports the remaining owned resource uncertainty.

## 25. Application shutdown

Normal application shutdown first stops accepting new investigations.

If a run is active, shutdown follows the same bounded ownership-aware cleanup path as cancellation.

The local process must not simply exit and abandon known remote resources when it has enough time and authority to release them.

A forced OS termination can still occur. Startup recovery handles that case separately.

## 26. Crash and restart recovery

At startup, ReproDocket scans only its own run and runtime state.

Runs left in nonterminal lifecycle states from a previous process are classified as `INTERRUPTED` unless authoritative recovery logic can prove continued ownership and resume is explicitly supported.

Version one does not automatically resume browser action execution after a process crash.

That conservative behavior avoids replaying external actions unexpectedly.

The application preserves all valid evidence captured before interruption.

If previous resource ledger entries indicate a browser or sandbox may still exist, the application may attempt cleanup only when the recorded identity and current provider contract make ownership sufficiently authoritative. Otherwise it reports unresolved cleanup rather than acting on unrelated resources.

## 27. Retry policy

Retries are centralized rather than scattered throughout adapters.

A retry decision records:

```text
operation
attempt
maximumAttempts
reason
idempotencySafety
backoff
```

Only known transient and safely repeatable operations are retried automatically.

Browser actions that can mutate the target application are not blindly retried after an unknown outcome. An unknown mutation result becomes an explicit uncertainty state so the verifier does not accidentally perform a duplicate destructive action.

## 28. Timeouts

Every external stage has a bounded timeout.

Timeouts are stage-specific and diagnostic. At minimum there are separate budgets for:

* Solari resource creation,
* navigation,
* individual browser interaction,
* replay readiness polling,
* sandbox command execution,
* fixture preview readiness,
* cancellation cleanup,
* application health startup.

A timeout identifies the stage and run rather than surfacing as a generic `operation failed` message.

## 29. Logging

Application logs are diagnostic but not the source of truth for run outcome.

Logs include stable event names, run ID, lifecycle stage, and sanitized error classifications.

Logs do not include the Solari key, cookies, authorization headers, password values, unrestricted target HTML, or unrestricted request/response bodies.

Log retention is bounded and separate from durable run evidence.

## 30. Error taxonomy

The product should use stable internal error codes so UI messages, tests, and diagnostics do not depend on exact exception prose.

Initial codes should include at least:

```text
INVALID_TARGET_URL
BLOCKED_TARGET_NETWORK
SOLARI_CREDENTIAL_MISSING
SOLARI_CREDENTIAL_INVALID
SOLARI_BROWSER_CREATE_FAILED
SOLARI_SANDBOX_CREATE_FAILED
SOLARI_CAPACITY_REACHED
NAVIGATION_TIMEOUT
ACTION_TIMEOUT
ACTION_OUTCOME_UNKNOWN
EVIDENCE_WRITE_FAILED
EVIDENCE_INVALID
RUN_ALREADY_ACTIVE
RUN_NOT_FOUND
RUN_MANIFEST_INVALID
REPLAY_NOT_READY
REPLAY_UNAVAILABLE
RESOURCE_CLEANUP_FAILED
LOCAL_PORT_UNAVAILABLE
LOCAL_REQUEST_REJECTED
```

Public UI copy should remain concise and human-readable while logs and machine-readable reports retain the stable code.

## 31. Public error messages

Errors shown in the UI answer three questions:

1. What could not be completed?
2. What is known about the run?
3. What can the user safely do next?

They do not expose stack traces, local absolute source paths, package cache locations, raw provider response bodies, or credentials.

Detailed developer diagnostics remain local and sanitized.

## 32. Dependency and supply-chain boundary

Runtime dependencies are kept small and purpose-driven.

Every dependency added to `dependencies` must be used by the shipped product. Test-only and build-only packages belong in `devDependencies`.

The lockfile is committed.

Before final publication, validation records `npm audit` or the chosen equivalent and classifies findings by actual reachability and severity rather than blindly claiming zero risk.

An automated dependency update is not accepted until the real build and applicable tests pass again.

## 33. Browser profile boundary

Saved Solari browser profiles are outside ReproDocket version one.

The product must not create, enumerate, mutate, or persist Solari login profiles unless that capability is explicitly added later with a separate privacy and lifecycle review.

Authentication-required target workflows may therefore become INCONCLUSIVE in version one unless the target itself provides a non-sensitive test login through the supported workflow.

## 34. Captcha, proxy, and stealth boundary

The initial deterministic fixture does not require proxy or captcha features.

ReproDocket should use the simplest Solari browser mode that truthfully exercises the target. It must not silently enable paid proxy or captcha behavior merely to make a test pass.

If stealth is later required for a real target, the run records that mode as part of provenance.

## 35. Evidence export boundary

Version one may generate a local HTML and JSON report inside the run directory.

It does not automatically upload evidence to GitHub, cloud storage, or another external service.

Any future upload or issue-creation integration requires explicit user action and a separate external mutation boundary.

## 36. Validation security cases

The security suite must include deterministic tests for:

* blocked file/data/javascript schemes,
* loopback and private IP literal blocking,
* cloud metadata target blocking,
* same-origin mutation enforcement,
* invalid local session nonce,
* path traversal attempts,
* encoded path traversal attempts,
* cross-run artifact access attempts,
* malicious HTML in problem descriptions,
* malicious HTML in console text,
* malicious HTML in page titles,
* fake Solari key redaction,
* authorization header redaction,
* malformed run manifests,
* tampered evidence hashes,
* duplicate investigation requests,
* shutdown during active run,
* cleanup failure reporting,
* stale PID ownership record,
* occupied preferred port,
* unexpected Host header.

Each case must fail in a way that is observable at the real boundary it protects.

## 37. Security completion gate

The security and lifecycle category is not complete until:

1. the loopback service is not unintentionally network-exposed,
2. mutation requests are protected from unrelated web origins,
3. target URL policy rejects prohibited schemes and internal destinations,
4. secrets are protected at rest and redacted before persistence,
5. untrusted target text cannot execute in the UI or HTML report,
6. evidence paths cannot escape their run ownership boundary,
7. duplicate submissions cannot create duplicate pipelines,
8. browser, sandbox, and local process ownership is explicit,
9. cancellation and shutdown close owned resources,
10. interrupted runs remain truthful and recoverable as records,
11. failure injection proves the guards actually reject invalid states, and
12. a fresh full validation after security changes passes before publication.

Any unproven item remains an explicit blocker or limitation rather than being assumed secure.