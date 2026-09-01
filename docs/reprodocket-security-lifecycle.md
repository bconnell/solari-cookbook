# ReproDocket Security and Lifecycle Model

Date: August 31, 2026
Status: Pre-implementation security baseline

This document defines the minimum security, privacy, ownership, and recovery behavior required for ReproDocket version one. It is scoped to the actual product: a loopback-only local application that accepts a public web target and an auditable nonsecret test plan, then drives remote Solari resources.

Security remains proportional. ReproDocket does not need enterprise identity, multitenant authorization, or speculative infrastructure. It does need strong boundaries around provider credentials, arbitrary target input, localhost mutation APIs, persisted evidence, remote resource lifetime, and untrusted text rendered back into the UI or static report.

The exact user-data privacy boundary is defined in [`reprodocket-data-handling.md`](reprodocket-data-handling.md).

## 1. Trust boundaries

ReproDocket crosses these meaningful boundaries:

1. Local browser to ReproDocket's loopback service.
2. User-authored target, problem, and plan into the application.
3. ReproDocket to Solari APIs and browser/sandbox resources.
4. Remote target content and browser events back into local evidence.
5. Persisted local evidence back into the UI and HTML report.
6. Application shutdown/restart back into persisted lifecycle/resource state.

Every externally supplied string is data, not executable product content.

## 2. Local service exposure

Normal bind targets:

```text
127.0.0.1
::1
```

The application does not default to `0.0.0.0`, a LAN interface, or a public interface.

The server rejects unexpected Host values rather than relying only on the bind address. CORS is disabled by default; no wildcard origin is allowed.

State-changing routes require a same-origin request and a per-process random request nonce supplied through the same-origin bootstrap path. The nonce is defense in depth against drive-by localhost mutation; it is not represented as user authentication.

The nonce is ephemeral and does not enter run evidence, public logs, or source.

## 3. Response security

The built local service uses a restrictive policy compatible with the actual bundled UI. Initial target:

```text
Content-Security-Policy: default-src 'self'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'; script-src 'self'; connect-src 'self'; frame-src 'none'; object-src 'none'; base-uri 'none'; form-action 'self'
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Cross-Origin-Resource-Policy: same-origin
```

Do not add `unsafe-eval` to make development tooling easier in the production-style build.

If a proven replay/viewer feature requires a wider policy, widen only the exact needed source/directive and add regression coverage.

## 4. Target URL policy

Version one accepts only public `http:` and `https:` targets.

Reject before Solari resource creation:

```text
file:
data:
javascript:
top-level blob:
unknown/custom executable schemes
URLs containing username/password authority credentials
malformed/empty hosts
localhost and .localhost
.local
IPv4 loopback including 127/8
RFC1918 private IPv4
IPv4 link-local
169.254.169.254 and equivalent known metadata/link-local targets
IPv6 loopback
IPv6 unique-local
IPv6 link-local
```

The initial parser/policy must use proper URL/IP parsing rather than substring tests.

A public domain that resolves to or redirects to a prohibited destination is blocked to the strongest authoritative boundary exposed by the installed browser/provider. If authoritative DNS visibility is unavailable, document that limitation rather than claiming complete SSRF prevention.

The Solari fixture preview is public and does not require a private-network exception.

Version one has no user override for blocked private destinations.

## 5. Navigation policy

The initial requested URL and relevant final main-frame URLs are recorded in sanitized form.

Each explicit absolute `OPEN`, `WAIT_FOR_URL`, and `EXPECT_URL` target is subject to the same scheme/network policy as the initial target.

Main-frame redirects are revalidated where the browser boundary provides the necessary authority. Prohibited redirect destinations abort the relevant action/run rather than being silently accepted.

The product never converts a download or nonweb scheme into local execution.

## 6. User-authored run text

Target URL, problem description, and source plan are untrusted text.

The source plan is **required** and intentionally persisted. Version one supports only nonsecret test values in that plan. The UI warns users that the plan is stored and must not contain real passwords, API keys, session tokens, payment data, or other confidential values.

The product does not claim it can perfectly detect arbitrary secrets inside freeform user-authored text.

React renders run/target text as text nodes/properties rather than raw HTML. Do not use `dangerouslySetInnerHTML` for target-derived or run-derived data.

The static report escapes every untrusted value. Payloads such as:

```text
</script><script>alert(1)</script>
```

must render as inert text.

Tests include markup, event-handler strings, malformed entities, bidi/control characters, and source-like/debug-like text.

## 7. Evidence minimization

Durable target-derived evidence may include:

```text
semantically timed screenshots
selected console warnings/errors
uncaught page errors
request method
sanitized request URL
main-document/relevant response status
request failure reason
timing where available
page URL/title
action and expectation timeline
session identity
replay state
observation results
cleanup state
```

Do not persist by default:

```text
Authorization headers
Cookie/Set-Cookie headers
proxy credentials
Solari API key
unrestricted request/response bodies
browser cookie jars
localStorage/sessionStorage dumps
saved profile contents
arbitrary DOM serialization
```

Action values are not redundantly copied into evidence events unless necessary. The canonical source plan already records intentional user-authored input and is governed by the nonsecret-plan requirement.

Screenshots and replay material can contain target-visible data. ReproDocket does not claim automatic screenshot/replay redaction.

## 8. Redaction

Provider/target-derived structured text passes through a central redactor before durable persistence.

The redactor handles authoritatively known sensitive values and common high-confidence patterns, including:

```text
current Solari API key value
known secret-bearing headers
obvious bearer-token forms
signed provider capability values when classified as sensitive
explicitly classified sensitive URL parameters
```

Redaction is defense in depth. It is not a substitute for the nonsecret source-plan rule and does not promise perfect arbitrary-secret discovery.

The original secret is never included in a redaction error.

A synthetic known key is injected through provider-derived channels in tests and searched across generated JSON, HTML, logs, and validation summaries.

## 9. Solari credential storage

The normal Windows experience uses the current Windows user protection boundary.

Preferred initial implementation: a small PowerShell 7 helper calling supported .NET protected-data APIs with `CurrentUser` scope. Node sends plaintext to the helper over standard input rather than a command-line argument.

Encrypted bytes are stored beneath the ReproDocket local application-data root. Plaintext exists only in process memory for the minimum deliberate operation.

The Solari key is never intentionally stored in:

```text
.env
package scripts
repository configuration
source plan
command-line arguments
process ownership metadata
run manifest
report/evidence
validation artifacts
public source
```

`SOLARI_API_KEY` remains an optional automation/developer source.

Credential source precedence:

```text
explicit process environment
-> protected local store
-> unavailable
```

The UI may identify `Environment`, `Protected local store`, or `Not configured` without revealing key material.

## 10. Credential verification

Saving a credential does not immediately mean provider READY.

ReproDocket performs a harmless current Solari readiness operation appropriate to the installed SDK. If browser creation is the only authoritative verification path, the resulting short-lived browser is registered and closed immediately.

Failed verification leaves provider state unready and displays a sanitized actionable reason.

An environment-sourced credential is not overwritten through the local-store endpoint. Deleting a protected local credential does not mutate the process environment.

## 11. Filesystem boundary

All application-owned persistent paths derive from one known application-data root.

Run IDs are generated internally. User input never selects a filesystem path.

Artifact routes accept validated run identity plus artifact identity from the run manifest. Do not concatenate user-supplied paths.

Reject any resolution escaping the owned root, including encoded traversal and symlink/reparse-point escape where applicable.

Cleanup never targets source repositories, Desktop, Documents, Downloads, arbitrary temp roots, or caller-supplied folders.

## 12. Run ownership and atomic persistence

Each run owns one generated directory. Creation is fail-if-exists.

Critical manifest transitions are durably written before downstream logic assumes them.

Use same-directory temporary write plus flush/fsync where practical and atomic rename/replace semantics supported by the platform. A crash must leave the prior valid state or a detectable incomplete write, not silently accepted truncated JSON.

On startup, incomplete temporary files are diagnosed. The product uses the last valid authoritative state and does not invent completion.

## 13. Manifest validation and schema evolution

Persisted state is untrusted on reload. Validate it against the versioned schema before use.

Malformed or unsupported manifests are never executed as valid run truth.

One damaged historical run must not crash global history. It remains visible through a bounded damaged-run projection with an actionable diagnostic where possible.

Version one does not auto-migrate unknown future schema versions.

## 14. Evidence integrity

Each finalized durable artifact receives a SHA256 digest recorded in integrity metadata.

A finalized artifact is served/displayed only after run ownership and integrity validation.

Missing, substituted, or hash-mismatched evidence yields `EVIDENCE_INVALID` or a documented equivalent rather than being displayed under a valid evidence label.

Active runs may expose explicitly unsealed evidence before finalization; the UI must distinguish it from finalized evidence.

## 15. Report generation

HTML and JSON reports derive from validated authoritative run state.

The HTML report escapes every untrusted value and requires no JavaScript in version one.

Reports reference only artifacts owned by the run.

The report is a projection of the manifest, not a second source of truth.

## 16. Remote resource ledger

Every Solari resource created by ReproDocket is registered immediately.

Minimum ledger information:

```text
resource type
resource ID
owning run ID/validation ID
created time
release requested time
release confirmed time
last known state
cleanup error
```

Use explicit creation-returned identity rather than guessing ownership from an account-wide resource list.

ReproDocket cleans up only resources it can prove it owns.

## 17. Browser lifecycle

Normal attempt lifecycle:

```text
create
-> register ownership
-> create/use page
-> capture evidence
-> stop collectors
-> close browser
-> record close result
-> resolve replay state within bounded policy
```

Every creation path has structured cleanup.

Investigation and verification browsers are independent resources. Verification is created only after the investigation browser close attempt reaches the required boundary.

A run cannot claim cleanup PASS while either required browser remains unresolved.

## 18. Sandbox lifecycle

Harness-owned sandboxes exist only for deterministic fixture validation/demo support.

Lifecycle:

```text
create
-> register ownership
-> connect
-> verify runtime
-> write fixture
-> start fixture server
-> verify command/process readiness
-> obtain preview URL
-> externally verify fixture identity
-> use
-> kill
-> record teardown result
```

The harness explicitly kills the sandbox. Provider idle pause/timeout is not accepted as teardown.

A failed kill may preserve functional test observations, but Full lifecycle result fails until cleanup is truthfully reconciled.

## 19. Local process ownership

The local server writes an application-owned runtime record containing enough identity to reject recycled PIDs:

```text
pid
process start identity
port
bind address
repository/build identity
instance nonce
started time
```

`stop.ps1` verifies current process identity before termination. PID equality alone is insufficient.

If ownership cannot be proven, leave the process alone and report the condition.

## 20. Port selection

The preferred port is a convenience, not ownership.

If occupied by another process, select another loopback port. Never terminate an unrelated process to obtain the preferred port.

If a valid already-running ReproDocket instance belonging to the current application is authoritatively identified, normal start reuses/opens it rather than creating a duplicate.

## 21. Active run concurrency

Version one supports one executing investigation pipeline per ReproDocket process.

This capacity is explicit. A second submission receives `RUN_ALREADY_ACTIVE`; it is not silently discarded or queued.

Historical browsing remains available while a run is active.

Do not add a queue until queue persistence, cancellation, ordering, shutdown, and UI behavior are fully designed and tested.

## 22. Duplicate submission

Client-side button disabling is convenience only.

Server admission is atomic so rapid double submission/racing requests admit exactly one external pipeline.

The accepted request returns the authoritative run identity before the UI navigates to active detail.

## 23. Cancellation

Cancellation is lifecycle behavior, not process killing.

A cancellation request:

1. identifies the owned active run,
2. records request state,
3. stops scheduling new actions,
4. propagates an abort signal through current work,
5. allows bounded action cleanup,
6. closes owned browsers,
7. kills an owned fixture sandbox if one belongs to that harness operation,
8. persists `CANCELLED` only when the lifecycle reaches that truth,
9. records cleanup failures separately.

The UI does not optimistically declare `CANCELLED` when only the request has been acknowledged.

If cleanup cannot be proven, surface the remaining uncertainty.

## 24. Application shutdown

Graceful shutdown stops accepting new investigations first.

An active run follows the same ownership-aware abort/cleanup path as cancellation.

Known remote resources are not intentionally abandoned merely because the local server is stopping.

Forced OS/process death remains possible and is handled by restart recovery.

## 25. Crash/restart recovery

Startup scans only ReproDocket-owned runtime/run state.

A nonterminal run owned by a previous process becomes `INTERRUPTED` unless a future explicitly designed resume mechanism exists. Version one never automatically resumes/replays target mutations after a crash.

Valid partial evidence is preserved.

If a prior ledger suggests a provider resource may remain alive, cleanup is attempted only when ownership and current provider contract make that action sufficiently authoritative. Otherwise record unresolved cleanup rather than guessing against account-wide resources.

## 26. Retry policy

Retries are centralized and operation-aware.

Each retry decision records:

```text
operation
attempt/max attempts
reason
idempotency safety
backoff
```

Automatically retry only known transient, safely repeatable control-plane operations.

Do not blindly retry target mutations after an unknown action outcome.

Do not rapid-spin on 429/capacity failures. Honor authoritative retry guidance where provided.

Provider 502/503/504 may be candidates for bounded retry only for safe operations; current Solari docs/type/observed behavior govern exact policy.

## 27. Timeouts

Every external stage has a bounded diagnostic timeout, including:

```text
provider readiness
Solari resource creation
page/navigation
individual action
expectation wait
replay readiness
sandbox command
fixture preview readiness
cancellation cleanup
local startup health
```

Timeout errors identify the stage/run rather than becoming a generic failure message.

## 28. Logging

Application logs aid diagnosis but are not outcome authority.

Logs may include stable event name, run ID, lifecycle stage, attempt role, and sanitized error classification.

Logs do not intentionally contain the Solari key, secret-bearing headers, unrestricted target bodies/HTML/storage, or signed provider capabilities classified as secrets.

User-authored source plan data belongs in the run manifest and should not be redundantly dumped to general logs.

Log retention is bounded and separate from durable evidence.

## 29. Error taxonomy

Use stable internal codes so UI/tests do not depend on provider prose. Initial set includes at least:

```text
INVALID_REQUEST
INVALID_PLAN_STATEMENT
PLAN_EXPECTATION_REQUIRED
INVALID_TARGET_URL
BLOCKED_TARGET_NETWORK
LOCAL_REQUEST_REJECTED
SOLARI_CREDENTIAL_MISSING
SOLARI_CREDENTIAL_INVALID
SOLARI_UNAVAILABLE
SOLARI_CAPACITY_REACHED
SOLARI_BROWSER_CREATE_FAILED
SOLARI_SANDBOX_CREATE_FAILED
NAVIGATION_FAILED
ACTION_FAILED
AMBIGUOUS_TARGET_ELEMENT
TARGET_ELEMENT_NOT_FOUND
EXPECTATION_INSUFFICIENT
EVIDENCE_WRITE_FAILED
EVIDENCE_INVALID
RUN_NOT_FOUND
RUN_ALREADY_ACTIVE
RUN_DAMAGED
REPLAY_PENDING
REPLAY_UNAVAILABLE
CANCELLATION_FAILED
CLEANUP_FAILED
```

Map raw provider/system exceptions to these at boundaries. Preserve a sanitized diagnostic cause without exposing secrets.

## 30. Security failure behavior

Security-policy failures fail closed at their real boundary.

Examples:

```text
blocked target -> no Solari browser created
foreign-origin mutation -> no state change
invalid nonce -> no state change
artifact mismatch -> no bytes served
manifest integrity failure -> no valid evidence claim
credential verification failure -> provider not READY
```

Do not convert security policy failure into a normal target result.

## 31. Required security/lifecycle validation

Validation includes deterministic tests for:

```text
loopback-only bind
Host/Origin/nonce mutation protection
URL/IP blocked ranges
redirect policy to strongest enforceable boundary
artifact traversal/cross-run rejection
manifest corruption/tampering
static report escaping
React inert target/user content
Solari-key and known provider-secret redaction
plan-persistence warning and nonsecret-plan documentation
credential plaintext absence from protected-store bytes/command line
atomic persistence interruption
run collision prevention
duplicate/racing submission
cancellation at multiple stages
browser cleanup on success/failure
sandbox cleanup on success/failure
stale PID/process ownership refusal
retry bounds
Node handle/process exit
restart to INTERRUPTED
```

The security suite also deliberately mutates critical guards to prove its tests can fail.

## 32. Explicit nonclaims

Version one does not claim:

```text
formal security certification
formal WCAG certification
perfect arbitrary-secret detection
screenshot/replay redaction
safe handling of real secrets inside reproduction plans
authenticated workflow secret injection
universal SSRF prevention beyond observable/enforceable network boundaries
multiuser isolation
enterprise identity
remote-hosted ReproDocket service
```

These are truthful scope boundaries, not hidden missing features.
