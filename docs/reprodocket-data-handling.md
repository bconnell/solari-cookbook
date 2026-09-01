# ReproDocket Data Handling and Privacy Contract

Date: August 31, 2026
Status: Pre-implementation authority

## 1. Scope

This document clarifies exactly what ReproDocket stores, minimizes, protects, and cannot automatically make private. It is authoritative for version-one data handling where broader design language could otherwise be read too strongly.

ReproDocket is a local developer tool, not a privacy vault. Its central purpose requires preserving enough information to reproduce and audit a browser run.

## 2. Three data classes

### Application secrets

Examples:

```text
Solari API key
future provider credentials
local request nonce
signed provider capability URLs when the SDK treats them as secrets
```

These are protected or ephemeral and must not enter normal durable run evidence.

### User-authored run data

Examples:

```text
target URL
problem description
reproduction/observation plan
values intentionally written inside plan statements
```

These are intentionally persisted as part of the run record because they are necessary to explain and reproduce what was executed.

Version one therefore requires nonsecret plan data. The product must tell users not to put real passwords, API keys, session tokens, payment data, or other confidential values into a plan.

### Target-derived evidence

Examples:

```text
screenshots
page titles
console warnings/errors
uncaught page errors
sanitized network URL/status/failure metadata
replay material or replay reference
```

These can contain information supplied by the target application. ReproDocket minimizes structured collection but cannot guarantee a screenshot or page-generated string contains no sensitive information.

## 3. Version-one authenticated-workflow boundary

Version one is designed for public or otherwise nonsecret test workflows.

It does not provide a secret-reference syntax such as `FILL_SECRET`, credential vault injection, profile import, cookie import, or automatic login-secret masking.

Do not claim support for production-authenticated workflows that require entering real credentials into the persisted plan.

A future authenticated-workflow feature requires a separate design with at least:

```text
secret references rather than literal values
protected local secret storage
runtime-only substitution
screenshot/privacy implications
recording implications
redaction tests
a clear export/report policy
```

That feature is outside version one.

## 4. Plan persistence

The canonical manifest stores the submitted `sourcePlan` and parsed plan because auditability requires knowing what was actually executed.

Therefore a literal value in a statement such as:

```text
FILL "Email" WITH "qa@example.com"
```

is durable run data.

The UI must show a concise warning near the plan editor:

```text
Plans are saved with the run. Use test data only. Do not enter passwords, API keys, tokens, payment data, or other secrets.
```

The public README repeats this limitation.

## 5. Solari API key

The Solari key is never part of the submitted plan.

Normal Windows storage uses the current-user protected secret boundary. Environment-variable input remains an automation/developer option.

The key must not appear in:

```text
run manifests
source plans
parsed plans
console/network evidence
reports
runtime ownership files
validation reports
public demo material
Git history
child-process command lines
```

Known current key values are included in redaction inputs before provider-derived text is persisted.

## 6. Local request nonce

The loopback mutation nonce is generated per process, kept in memory or equivalent ephemeral runtime state, and not included in durable runs/reports. It is not a user credential and is not described as authentication.

## 7. Network evidence

Version one stores only minimized network metadata needed for evidence:

```text
method
sanitized URL
main-document status where applicable
selected response status
failure text
timing where available
```

Do not persist request/response bodies, Authorization headers, Cookie/Set-Cookie headers, or arbitrary header maps by default.

Sensitive query-parameter handling must be conservative. If a URL contains obvious secret-bearing query keys or a provider signed capability, store a sanitized/redacted representation rather than the raw durable URL when doing so does not break evidence meaning.

## 8. Console and page-error evidence

Persist only selected warning/error messages needed by the run. Apply the central redactor before writing durable data.

Redaction protects known application/provider secrets and common obvious token patterns. It is defense in depth, not proof that arbitrary target text contains no secret.

## 9. Screenshots

Screenshots are evidence of visible browser state and therefore may contain any data visibly rendered by the target.

ReproDocket does not claim automatic screenshot redaction in version one.

The UI and README must state this. Public demo screenshots use only the synthetic fixture or another explicitly public-safe target.

## 10. Replays

Recorded browser sessions may contain more target state than ReproDocket's minimized structured evidence.

Replay URLs or underlying signed session capabilities are handled according to the current Solari SDK/API security contract. If a replay URL is a signed capability, do not copy it into public reports or logs unnecessarily.

Locally retained replay bytes are stored only when the TypeScript SDK exposes a supported download path and that behavior is deliberately implemented. They remain local run data and are not committed by default.

Public demo/release material does not version raw replay data unless it has been specifically reviewed as safe and useful.

## 11. Static HTML report

The report includes user-authored target/problem/plan data because those explain the run. It includes only sanitized target-derived structured evidence and safe links/actions according to the replay contract.

Every untrusted value is HTML-escaped.

Because the report contains the plan and target evidence, users must treat it as potentially sensitive even though it contains no intentional ReproDocket provider credential.

## 12. Local storage location

Normal run data lives beneath ReproDocket's application-data root, outside the Git repository.

The product does not automatically upload local run history to a ReproDocket service because no such service exists in version one.

Solari necessarily receives browser/sandbox control data needed to execute the remote sessions. Public docs should link to current Solari documentation rather than inventing claims about Solari's data-retention/privacy behavior.

## 13. Retention and deletion

Version one may keep local run history until the user intentionally deletes application data or until a fully implemented run-delete/retention feature exists.

Do not add a Delete button unless deletion semantics, confirmation, evidence ownership, error recovery, and tests are implemented.

No automatic cleanup may delete unrelated user files or source.

If automatic retention is later introduced, it requires explicit policy, visibility, and tests.

## 14. Public source and demo policy

Tracked source may contain synthetic fixture values only.

Public demo material must not contain:

```text
real API keys
real passwords/tokens
private URLs
personal browser state
private target screenshots
signed provider capability URLs
absolute developer machine paths
account-specific provider data not needed for the demo
```

Fixture values should be obviously synthetic rather than merely fake-looking copies of real secrets.

## 15. Required tests

At minimum, validation proves:

```text
Solari key absent from durable run/report/log output
protected key file does not contain submitted plaintext
child-process command line does not contain key
Authorization/Cookie/Set-Cookie not persisted
HTML report escapes user-authored plan/problem text
UI renders user/target content inertly
plan editor warns that plan text is persisted
public-source scan finds no real secret
public fixture/demo contains only synthetic data
signed replay/capability handling matches the current Solari contract
```

Do not add a test claiming arbitrary user secrets are automatically detected and removed from the source plan. That is not a supported version-one capability.

## 16. Clarification of broader design wording

Where another pre-implementation document broadly says `secrets are excluded from run manifests/evidence`, interpret `secrets` as ReproDocket-owned/provider credentials and known secret-bearing fields that the product can authoritatively identify.

Arbitrary user-entered plan text is intentionally persisted and must therefore be nonsecret test data in version one.

This clarification narrows an overbroad privacy claim; it does not weaken Solari-key protection, header minimization, report escaping, artifact ownership, or central redaction requirements.
