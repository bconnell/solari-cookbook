# ReproDocket Local UI Specification

Date: August 31, 2026
Status: Pre-implementation product UI contract

## 1. UI goal

ReproDocket should look and behave like a focused developer tool: compact, legible, evidence-oriented, and calm. The interface exists to answer three questions quickly:

1. What is being tested?
2. What happened in each independent browser attempt?
3. What evidence supports the final result?

The UI must not imitate Solari so closely that ReproDocket could be mistaken for an official Solari product. Solari attribution is clear, but ReproDocket retains its own identity.

Version one does not add theme selection, dashboards, team navigation, account profiles, billing, notification centers, or settings pages that do not directly support the workflow.

## 2. Visual direction

Use a restrained developer-tool visual system.

Recommended characteristics:

```text
neutral dark application shell
high-contrast light content text
subtle elevated panels rather than heavy shadows
one restrained accent for primary actions/focus
semantic status treatments that do not depend on color alone
monospace only for IDs, code, plan statements, URLs, and machine evidence
system UI font stack for normal copy
comfortable density, not oversized marketing spacing
```

Do not hard-code Solari brand colors or logos unless their public usage is clearly permitted. Text attribution such as `Powered by Solari` is sufficient for version one.

No decorative animation is required. Status transitions may use subtle motion only when it improves comprehension and respects reduced-motion preferences.

## 3. Application frame

Desktop baseline target: 1280x720 and larger.

Supported compact target for validation: 768 CSS pixels wide without critical horizontal overflow. Evidence tables may use intentional internal scrolling if necessary, but primary navigation and outcome information remain visible.

Application shell:

```text
+---------------------------------------------------------------+
| ReproDocket                         Solari: Connected          |
+----------------------+----------------------------------------+
| New Investigation    |                                        |
| Runs                  |             Current page               |
|                      |                                        |
| recent run summary   |                                        |
| recent run summary   |                                        |
+----------------------+----------------------------------------+
```

A narrow viewport may collapse the left rail into a compact navigation control, but it must not create a second disconnected navigation implementation.

## 4. Persistent header

Required elements:

```text
ReproDocket product name
current Solari readiness state
```

Optional after implementation proves useful:

```text
current application version
```

Do not show raw API-key fragments, account identifiers, concurrency data, or provider internals in the persistent header.

Provider states:

```text
Not configured
Checking
Connected
Unavailable
Configuration error
```

`Connected` is shown only after the active credential source has been validated through the provider readiness path.

## 5. Navigation

Primary destinations:

```text
New Investigation
Runs
```

A selected run is reachable from Runs/history and may have a direct `/runs/:id` route.

Do not add a Settings destination solely for credential entry. If no credential exists, provider setup is presented as a focused onboarding state. A small provider action can later expose `Change key` only if it is fully implemented and useful.

Browser Back/Forward navigation must not create, cancel, or duplicate a run.

## 6. First-run provider connection

When no supported credential source is available, the first useful screen is:

```text
Connect Solari

ReproDocket uses Solari cloud browsers to run investigations.

Solari API key
[________________________________________]

[Save and verify]

Your key is protected for the current Windows user and is not stored in run evidence.
```

Behavior:

* input uses password-style obscuring by default;
* a reveal control is optional only if accessible and correctly implemented;
* Save and verify enters a visible Checking state;
* double submission is prevented locally and server-side;
* success moves to `Connected` and opens New Investigation;
* failure keeps the user on the screen with a sanitized actionable message;
* no raw provider response or stack trace is displayed;
* an environment-provided credential is not copied into the protected store unless the user explicitly replaces it through the UI.

## 7. New Investigation screen

Primary fields:

```text
Target URL
Problem description
Reproduction and observation plan
```

Suggested layout:

```text
New investigation

Target URL
[ https://example.com                                      ]

Problem description
[ Describe the reported behavior...                       ]
[                                                          ]

Reproduction and observation plan
+----------------------------------------------------------+
| OPEN "/account"                                         |
| FILL "Email" WITH "qa@example.com"                     |
| CLICK "Save"                                            |
| EXPECT_PAGE_ERROR "PROBE_ACCOUNT_SAVE_ERROR"           |
+----------------------------------------------------------+

[ Plan syntax ]                             [ Investigate ]
```

The plan field is required. UI copy must not imply it is optional.

Validation messages are adjacent to/associated with their field. Submission with a missing action, missing expectation, invalid statement, blocked target, or oversized input must explain the exact problem before external work begins.

## 8. Plan syntax help

Plan syntax help is a real disclosure/dialog/panel, not a dead question-mark icon.

It shows the implemented grammar grouped into Actions and Expectations.

Actions:

```text
OPEN "/path"
CLICK "Button name"
FILL "Field label" WITH "value"
SELECT "Field label" VALUE "option"
CHECK "Checkbox label"
UNCHECK "Checkbox label"
PRESS "Enter"
WAIT_FOR_TEXT "text"
WAIT_FOR_URL "/path"
RELOAD
BACK
FORWARD
```

Expectations:

```text
EXPECT_TEXT "text"
EXPECT_NO_TEXT "text"
EXPECT_URL "/path"
EXPECT_PAGE_ERROR "message"
EXPECT_MAIN_STATUS 404
```

Help states plainly that ReproDocket uses accessible names/labels and rejects ambiguous matches rather than guessing.

Do not advertise arbitrary selectors or JavaScript.

## 9. Active Investigation screen

The active view prioritizes progress and ownership, not a fake streaming terminal.

Top section:

```text
Investigating
Target: example.com
Run: <short display ID>
Started: <local time>
[Cancel]
```

Progress stages:

```text
Preparing
Investigation browser
Investigation evidence
Closing investigation browser
Verification browser
Verification evidence
Finalizing report
Cleanup
```

Each stage may be `waiting`, `active`, `complete`, `failed`, or `cancelled` as supported by authoritative state.

Do not fabricate fine-grained progress percentages unless the denominator is real.

A concise live timeline may appear below. Raw internal class names and exception stacks do not.

Cancel remains visible only while cancellation is supported. After click, the label/state becomes `Cancellation requested` until the server reports a terminal lifecycle.

## 10. Run Detail information hierarchy

The first viewport of a completed run should answer the result before presenting raw logs.

Recommended hierarchy:

```text
[ VERIFIED ]
Reported defect reproduced twice in independent Solari sessions.

Target          example.com/account
Lifecycle       Completed
Cleanup         Passed

Investigation             Verification
Reproduced                 Reproduced
session ...                 session ...

Evidence tabs...
```

Outcome language:

### VERIFIED

Primary copy:

```text
Verified
The reported condition was reproduced in the investigation and again in a fresh Solari browser session.
```

### REPRODUCED

```text
Reproduced once
The reported condition was observed in the investigation but was not confirmed in the fresh verification session.
```

### NOT_REPRODUCED

```text
Not reproduced
The workflow was completed with enough evidence to test the reported condition, but that condition was not observed.
```

### INCONCLUSIVE

```text
Inconclusive
ReproDocket could not gather enough authoritative evidence to determine whether the reported condition occurred.
```

Where available, show the specific reason below the summary.

Do not use a green success check for `NOT_REPRODUCED` in a way that could imply the target is bug-free. It means only the submitted condition was not reproduced.

## 11. Investigation versus verification panels

Both attempts are peers visually. Do not make the verification look like a tiny footnote.

Each attempt shows:

```text
role
attempt outcome
session ID or shortened display form with copy action if safe
start/end time
completed action count
expectation results
replay state
evidence count
attempt error if any
```

A `VERIFIED` run makes it visually obvious that two distinct sessions exist.

## 12. Evidence navigation

Evidence categories:

```text
Screenshots
Console
Page errors
Network
Timeline
Replay
Report
```

Hide a category only if its absence cannot be confused with a loading failure. Otherwise show an explicit empty state, such as `No page errors captured`.

The UI must identify whether evidence belongs to Investigation or Verification.

No evidence tab may query by raw filesystem path.

## 13. Screenshot evidence

Screenshot cards show:

```text
attempt
semantic label
capture time
integrity state
image
```

The main image is large enough to inspect the affected page. A thumbnail-only evidence view is insufficient.

Clicking/opening a screenshot may provide a larger lightbox/detail view if fully implemented. Browser native open-in-new-tab is acceptable if it preserves run-owned artifact validation.

Do not crop away context needed to understand the screenshot. Semantic capture labels such as `Before Save`, `After Save`, `Failure state`, and `Verification after Save` are preferred over `Screenshot 1`.

## 14. Console and page-error evidence

Console is a readable chronological list/table with:

```text
time
level
message
attempt
```

Page errors are shown separately or clearly distinguished from console errors because `EXPECT_PAGE_ERROR` relies on that authority.

Long messages wrap or expand without forcing the entire application to overflow horizontally.

No ANSI/control characters may be allowed to create misleading display behavior.

## 15. Network evidence

Initial columns:

```text
method
status/failure
URL
time/duration if available
attempt
```

Only the minimized/sanitized network contract is shown. No request/response body viewer is introduced in version one.

The main-document navigation response is visibly distinguishable where `EXPECT_MAIN_STATUS` depends on it.

## 16. Timeline

Timeline merges lifecycle, action, expectation, evidence, replay, cleanup, and outcome events in authoritative sequence order.

Each line should read as product language, for example:

```text
19:14:03  Investigation browser created
19:14:04  Opened /account
19:14:05  Filled Email
19:14:06  Clicked Save
19:14:06  Page error matched reported condition
19:14:07  Investigation browser closed
19:14:08  Verification browser created
```

Do not expose internal event enum names as the primary visible copy.

## 17. Replay UI

Replay states are explicit:

```text
Preparing replay
Open replay
Replay stored locally
Replay unavailable
```

If only a Solari replay URL is supported, `Open replay` opens the validated provider URL and makes it clear that the replay is hosted by Solari.

If local replay bytes are proven and a local viewer is not implemented, do not show a nonfunctional `Play` button. Offer only behavior that actually exists.

Signed replay/control URLs are treated as sensitive capabilities in logs/reports where required by the SDK contract. Do not display them as raw persistent text when a safe action can represent them instead.

## 18. Report UI

A `View report` action opens the run-owned static HTML report through the validated artifact/report route.

If a `Download report` action is added, it must generate or serve a real file and have complete tests. Otherwise omit it.

The report is not the authority over the manifest. It is a human-readable projection of the same run state.

## 19. Runs/history

History rows/cards display only useful summary fields:

```text
outcome/lifecycle
target host
problem summary
created time
cleanup warning if material
```

Do not show `VERIFIED` for an active run.

Damaged historical state remains represented as a diagnostic row rather than disappearing.

Selecting a run opens the same Run Detail route used after a current run completes.

## 20. Empty states

Required meaningful empty states include:

```text
no runs yet
no screenshots captured
no console evidence captured
no page errors captured
no network failures/status evidence captured
replay pending/unavailable
no error for this attempt
```

Empty state text must distinguish `nothing happened` from `evidence not yet loaded` and from `evidence unavailable/damaged`.

## 21. Error presentation

Errors have:

```text
short human title
plain-language message
stable next action when one exists
optional machine error code in secondary detail
```

Do not show raw stack traces in normal UI.

Representative mappings:

```text
SOLARI_CREDENTIAL_MISSING -> Connect Solari to run an investigation.
SOLARI_CAPACITY_REACHED -> Solari session capacity is currently in use. Close another owned session or try again later.
AMBIGUOUS_TARGET_ELEMENT -> More than one visible element matched this step. Make the plan target unique.
EVIDENCE_INVALID -> This saved evidence no longer matches its recorded integrity data.
RUN_ALREADY_ACTIVE -> One investigation is already running. You can still view previous runs.
```

Provider wording is revised against real observed errors during implementation.

## 22. Accessibility baseline

Use native semantic controls wherever possible.

Required:

```text
labels associated with fields
buttons/links use correct roles
visible keyboard focus
logical tab order
status changes announced through appropriate live-region semantics without excessive chatter
error messages associated with fields
non-color text/icon/status labels
reduced-motion respect
adequate contrast
zoom to 150% without losing the critical workflow
```

Do not claim formal WCAG compliance unless separately audited.

## 23. Status semantics

Color is secondary to text.

Recommended semantic treatment:

```text
VERIFIED             strong positive confirmation treatment
REPRODUCED           amber/caution treatment
NOT_REPRODUCED       neutral/informational treatment
INCONCLUSIVE         muted warning/uncertain treatment
FAILED lifecycle     error treatment
CANCELLED            neutral stopped treatment
INTERRUPTED          warning treatment
cleanup FAIL/PARTIAL visible warning independent of defect outcome
```

A run can be `VERIFIED` while cleanup has a problem. The UI must show both truths rather than hiding cleanup beneath the positive outcome.

## 24. Loading and stale state

Use skeleton/minimal loading indicators only while data is actually loading.

SSE events accelerate updates but the client rehydrates authoritative state through GET after reconnect/refresh.

Do not leave old run data visible under a newly selected run ID while the new run is loading. Either clear the detail region or clearly mark it loading for the requested ID.

## 25. Local times and IDs

Store timestamps in UTC ISO 8601. Display them in the user's local browser timezone with a way to inspect the exact timestamp when useful.

Display shortened IDs for readability but preserve an accessible copy action only if implemented. Do not expose signed capability URLs as identifiers.

## 26. Responsive behavior

At 768 CSS px:

```text
primary navigation remains reachable
New Investigation fields use full width
outcome remains above evidence
attempt panels stack vertically
wide evidence tables scroll inside their panel or convert to cards
critical actions remain visible
```

At desktop widths, attempt panels can sit side by side for comparison.

No horizontal page scroll is accepted for the primary workflow at the supported compact width.

## 27. Visual test captures

Automated screenshot preparation must capture at least:

```text
connection onboarding
new investigation with syntax help
active investigation
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
history with mixed outcomes
screenshot evidence close view
console/page-error evidence close view
network evidence close view
replay ready/nonready state
representative sanitized error
768 px layout
125% or 150% zoom usability view where browser harness supports faithful capture
```

Captures come from the exact built application and current run data. No mocked showcase page may substitute for the actual UI claim.

## 28. Public polish boundary

Before release, inspect the actual rendered UI for:

```text
raw enum/class names
placeholder/sample/demo labels accidentally presented as product capability
truncated text
misaligned controls
inconsistent capitalization
unclear status hierarchy
dead actions
unhelpful empty states
excessive technical jargon
fake progress
stack traces
source paths
credential fragments
wrong run evidence
stale status after navigation/reconnect
```

Any objective issue receives a regression where practical before final human review.

## 29. Non-goals

Version one intentionally does not include:

```text
theme switcher
user accounts
cloud history sync
multi-user collaboration
project organization
notifications
analytics dashboard
provider region selector
manual concurrency controls
arbitrary browser console
DOM inspector
request/response body inspector
custom CSS selector editor
plugin system
```

Absence of these features is intentional scope, not unfinished UI.
