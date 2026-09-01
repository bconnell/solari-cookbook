# ReproDocket Deterministic Fixture Specification

Date: August 31, 2026
Status: Pre-implementation fixture contract

## 1. Purpose

The fixture provides independent, deterministic ground truth for ReproDocket's live Solari validation. It is validation infrastructure, not part of normal production behavior. It must expose both defects and healthy controls so the classifier can fail for false positives and false negatives.

The fixture must run inside a real Solari sandbox and be reachable through a Solari preview URL before a real Solari browser consumes it.

Production ReproDocket code must not import the fixture, inspect fixture scenario identifiers, or branch on fixture routes to manufacture outcomes.

## 2. Implementation constraints

Use the smallest reliable runtime already available in the Solari base sandbox. The cookbook currently demonstrates `python3 -m http.server`, so the preferred initial fixture server is a Python standard-library HTTP server uploaded as source files by the harness. Before relying on Python, the live sandbox contract test must prove `python3` exists in the selected template.

Do not install a framework or package manager into the sandbox merely to host the fixture unless the base template no longer provides a suitable standard runtime.

The server binds inside the sandbox on the harness-selected port. The harness obtains the public URL through the current Solari preview API and verifies fixture identity from outside the sandbox.

## 3. Fixture identity

The fixture exposes:

```text
GET /__fixture__/meta
```

Response:

```json
{
  "name": "ReproDocket deterministic fixture",
  "fixtureVersion": "1",
  "status": "ready"
}
```

The exact version may change when fixture behavior materially changes. Full validation records it in run provenance.

The metadata endpoint contains no secret, account identifier, sandbox credential, internal hostname, or private path.

## 4. Reset/isolation model

Every ordinary scenario must be repeatable from a fresh browser state.

The preferred isolation model is browser-local state for deterministic per-browser behavior, with server-global state used only for the deliberately nonrepeatable scenario.

The full harness either creates a fresh fixture sandbox per test group or calls an explicitly harness-owned reset operation between scenarios. A reset path must never be reachable through ReproDocket production logic as a special-case oracle.

If a reset endpoint is used, it is scoped to the validation fixture and authenticated or sufficiently isolated by the ephemeral preview lifecycle. Its use remains in harness code only.

## 5. Home route

`GET /` returns a small index page linking to the fixture scenarios for manual inspection. It is not used as a classifier oracle.

Visible title:

```text
ReproDocket Test Fixture
```

Links:

```text
Account settings
Billing details
Address form
Profile
Login
Ambiguous controls
Nonrepeatable save
```

## 6. Scenario A: account save blank panel

Route:

```text
GET /account
```

Initial visible UI:

```text
Account settings
Email
Save
```

The account content is inside a stable region with an accessible name or heading.

Action sequence:

```text
OPEN "/account"
FILL "Email" WITH "qa.changed@example.com"
CLICK "Save"
EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"
```

Defect behavior:

1. The Save handler clears or hides the account detail panel in a visibly broken way.
2. The handler throws an uncaught JavaScript error with exact message `PROBE_ACCOUNT_SAVE_ERROR`.
3. The page remains open so post-failure evidence can be captured.
4. A fresh browser opening `/account` begins in the clean initial state.

Ground truth: both investigation and fresh verification satisfy the page-error expectation, therefore the full run should be eligible for `VERIFIED` when all other evidence/lifecycle requirements are met.

The fixture truth test independently proves the panel change and page error without calling ReproDocket classification code.

## 7. Scenario B: billing refresh 404

Route:

```text
GET /billing/details
```

The first request in a fresh browser returns HTTP 200, renders a visible `Billing details` heading, and sets a fixture-only cookie or equivalent browser-scoped state.

A subsequent same-browser navigation/reload to the same route returns HTTP 404 with visible text:

```text
Billing page not found
```

Action/expectation plan:

```text
OPEN "/billing/details"
RELOAD
EXPECT_MAIN_STATUS 404
```

A completely fresh browser receives 200 on its first request and 404 on its reload, so both independent attempts reproduce the same defect without sharing browser state.

The server must not use a global request counter for this scenario because that would make the second independent browser behave differently.

Ground truth: deterministic `VERIFIED` candidate.

## 8. Scenario C: missing ZIP accepted

Route:

```text
GET /address
```

Visible fields:

```text
Street
City
ZIP
Submit address
```

The intentionally defective form accepts a submission when Street and City are present but ZIP is blank, then displays:

```text
Address accepted
```

Plan:

```text
OPEN "/address"
FILL "Street" WITH "100 Test Ave"
FILL "City" WITH "Testville"
CLICK "Submit address"
EXPECT_TEXT "Address accepted"
```

The fixture does not silently prefill ZIP.

Ground truth: accepting the invalid form is the defect condition and is deterministic in both fresh browsers, so this is a `VERIFIED` candidate.

## 9. Scenario D: healthy profile negative control

Route:

```text
GET /profile
```

Reported problem used by the harness:

```text
Saving the profile succeeds but no confirmation appears.
```

Actual healthy behavior: clicking Save displays:

```text
Profile saved
```

Plan:

```text
OPEN "/profile"
CLICK "Save"
EXPECT_NO_TEXT "Profile saved"
```

The defect expectation is deliberately false on the healthy fixture. Because the workflow executes completely and the opposite healthy cue is visible, ReproDocket has sufficient evidence to classify the alleged defect as `NOT_REPRODUCED`.

The fixture truth test separately proves `Profile saved` appears.

## 10. Scenario E: healthy login negative control

Route:

```text
GET /login
```

Visible fields:

```text
Email
Password
Sign in
```

Reported problem used by the harness:

```text
Submitting invalid credentials crashes the login form instead of showing validation.
```

Actual healthy behavior: submitting known invalid fixture credentials keeps the page alive and displays:

```text
Invalid credentials
```

Plan:

```text
OPEN "/login"
FILL "Email" WITH "qa@example.com"
FILL "Password" WITH "fixture-only-password"
CLICK "Sign in"
EXPECT_NO_TEXT "Invalid credentials"
```

The fixture-only password is not a real secret. Test data must remain obviously synthetic and must not match the shape of a real project credential.

Ground truth: `NOT_REPRODUCED` candidate with sufficient healthy counterevidence.

## 11. Scenario F: ambiguous target

Route:

```text
GET /ambiguous
```

The page intentionally exposes two simultaneously visible actionable controls with the same accessible name:

```text
Save
Save
```

Neither control has a unique label/role/name combination under ReproDocket's documented accessible resolver policy.

Plan:

```text
OPEN "/ambiguous"
CLICK "Save"
EXPECT_TEXT "Saved"
```

Expected executor behavior: fail the action with `AMBIGUOUS_TARGET_ELEMENT`. It must not choose the first match by DOM order.

Because the intended action cannot be authoritatively executed, the product result is `INCONCLUSIVE`, not `NOT_REPRODUCED`.

Ground truth: ambiguity is intentional and independently tested.

## 12. Scenario G: nonrepeatable defect

Route:

```text
GET /nonrepeatable
```

Purpose: prove `REPRODUCED` remains distinct from `VERIFIED`.

This is the only initial scenario allowed to use fixture-global state across browser sessions.

Initial global scenario state for a fresh fixture instance:

```text
triggerCount = 0
```

On the first valid Save action across the fixture instance:

1. increment `triggerCount`,
2. emit/throw `PROBE_NONREPEATABLE_ERROR`,
3. render a visible broken state while keeping the page open.

On later valid Save actions:

1. do not throw the probe error,
2. render `Saved successfully`.

Plan:

```text
OPEN "/nonrepeatable"
CLICK "Save"
EXPECT_PAGE_ERROR "PROBE_NONREPEATABLE_ERROR"
```

With one fixture instance and two sequential fresh Solari browsers:

```text
investigation -> expectation confirmed
verification -> expectation not observed, healthy save visible
```

Expected final outcome: `REPRODUCED`.

The E2E harness must reset/recreate fixture state before running this scenario so earlier tests cannot consume its one-time failure.

Production ReproDocket receives only the URL and public plan. It is never told that the route is nonrepeatable.

## 13. Main-document status observation

`EXPECT_MAIN_STATUS` evaluates the main document's relevant navigation response, not arbitrary subresource failures.

The fixture must make status behavior unambiguous in its truth tests. A failing image/script/API request cannot satisfy `EXPECT_MAIN_STATUS 404` if the main document returned 200.

## 14. Page-error observation

`EXPECT_PAGE_ERROR` matches an uncaught page error event using the documented exact/normalized message policy. It does not match arbitrary console text that merely contains the same string.

Fixture error messages are stable machine probes such as:

```text
PROBE_ACCOUNT_SAVE_ERROR
PROBE_NONREPEATABLE_ERROR
```

They are intentionally nonsecret and safe to publish.

## 15. Network evidence probes

At least one fixture route should emit a deterministic failing subrequest so the live evidence bridge can prove network capture independently from defect classification.

Suggested endpoint:

```text
GET /api/evidence-probe -> 503
```

A page may trigger this request on an explicit `Generate evidence probe` action used only by live contract tests. The 503 is evidence; it is not itself a reported defect unless a plan expectation defines it.

## 16. Console evidence probes

At least one explicit fixture action should produce stable console warning/error text for live collector tests:

```text
PROBE_CONSOLE_WARNING
PROBE_CONSOLE_ERROR
```

These signals must not automatically satisfy defect observation.

## 17. Accessibility and locator truth

Fixture controls use ordinary semantic HTML: labels for form fields, buttons for actions, anchors for navigation, headings for sections. The fixture must not require privileged CSS selectors for normal scenarios.

Ambiguity exists only in the dedicated ambiguous scenario.

Fixture truth tests verify the accessible names expected by the reproduction plans.

## 18. Visual evidence suitability

Pages are intentionally plain but readable. They must show enough visible state for screenshots to make the before/failure/healthy distinction understandable without inspecting DOM source.

Do not polish the fixture so heavily that fixture design becomes a competing product project. The local ReproDocket UI is the showcase surface.

## 19. Harness lifecycle

The harness sequence is:

```text
create sandbox
register ownership
connect
verify python/runtime
write fixture files
start server in background
check command exit/status
obtain preview URL
GET /__fixture__/meta from outside sandbox
verify expected fixture version
run browser scenarios
kill sandbox in outer finally
record teardown result
```

No test success may bypass the final sandbox cleanup attempt.

## 20. Fixture sensitivity checks

The harness must deliberately prove at least these negative controls:

```text
remove account page error -> account fixture truth test fails
make billing reload return 200 -> billing truth test fails
reject missing ZIP -> missing-ZIP defect truth test fails
remove profile confirmation -> healthy profile truth test fails
make ambiguous controls unique -> ambiguity truth test fails
make nonrepeatable route fail every time -> REPRODUCED scenario truth test fails
```

These are temporary test mutations and are never committed in the broken state.

## 21. Public safety

The fixture contains only synthetic data. It performs no real authentication, payment, account mutation, email delivery, external webhook, or third-party write. It may make only the network requests required to serve/probe the ephemeral fixture itself.

The fixture is safe to use in a public demo and safe for reviewers to inspect in source.
