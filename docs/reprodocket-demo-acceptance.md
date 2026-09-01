# ReproDocket Demonstration Acceptance Contract

Date: August 31, 2026
Status: Pre-implementation demo acceptance

## 1. Purpose

The eventual public demonstration must prove the real product path quickly. It is not a separate polished mockup and it cannot substitute for engineering validation.

The preferred demonstration uses the deterministic Solari-hosted fixture so no private website, login, or personal data is needed.

## 2. Primary demo scenario

Preferred primary scenario after final validation:

```text
Account save blank-panel/page-error fixture
```

Why this scenario is preferred:

```text
clear visible before/after defect
simple understandable plan
uncaught page-error evidence
real Solari browser interaction
easy independent repetition in a fresh session
useful screenshot evidence
no real credential or personal target data
```

If final testing shows another fixture case is visually clearer, select the strongest validated deterministic case instead and record the reason.

## 3. Demo preconditions

Before recording/publishing the demo:

```text
exact release-candidate revision known
Full automated validation current for that revision
human visual/usability review current
fixture version known
Solari provider ready
no unresolved owned resource from prior run
local run history either intentionally prepared with safe fixture data or clean
no private tabs/windows/notifications required in frame
public-safe synthetic plan data only
```

Do not record a knowingly stale build because its UI happens to look better.

## 4. One-continuous-run proof

Where practical, the main demo should show one continuous user-facing flow:

```text
open ReproDocket
show Solari connected state
enter/select public fixture target
enter problem description
enter/paste action + expectation plan
start investigation
show real active stages
show first Solari attempt result/evidence
show transition to fresh verification attempt
show final VERIFIED result
show distinct investigation/verification session identities
open at least one screenshot/error/timeline evidence view
show replay state if proven and useful
show cleanup result
navigate to Runs/history and back to the completed run
```

The demo does not need to linger on installation or every evidence table. The README can cover setup.

## 5. Evidence that must be visible or directly inspectable

The main final result should make these truths easy to establish:

```text
the target/problem/plan used
investigation and verification are separate attempts
both attempts reproduced the defined condition for VERIFIED
session identities differ
supporting evidence exists for both attempts
final outcome is not just a console message
cleanup state is recorded
run persists in local history
```

Do not obscure the second session identity merely to simplify the screen. Independence is the defining product claim.

## 6. Solari visibility

The demo must make Solari's role clear without switching to the Solari dashboard as the primary evidence surface.

Acceptable cues include:

```text
provider state says Solari Connected
attempt cards identify Solari browser sessions
fixture/demo documentation states the fixture is hosted in a Solari sandbox
replay action identifies Solari-hosted replay when applicable
```

The local ReproDocket UI remains the center of the demonstration.

## 7. Duration and pacing

Target a concise social-friendly main cut, roughly 45 to 90 seconds if the live session timings allow it.

If Solari replay readiness or browser execution creates unavoidable waiting, use a truthful edit/cut with visible stage continuity rather than fake acceleration or prerecorded UI state presented as live.

A longer technical walkthrough may be linked separately if it materially helps reviewers.

## 8. No fake progress

Do not fabricate progress percentages, terminal output, browser events, or completion timing for the demo.

If an operation is waiting, show a real waiting state. If a long idle interval is removed in editing, the video edit must not create a false event order.

## 9. Public-safe data

Use only fixture data or another explicitly public-safe target.

The plan must not contain real passwords, API keys, tokens, payment values, personal emails, private URLs, or signed provider capabilities.

Inspect every visible screenshot and recording frame before publication.

## 10. Secondary result proof

The README or longer walkthrough should show that ReproDocket is not hardcoded only for VERIFIED.

At minimum, current evidence should exist for:

```text
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
```

The short social demo may focus on VERIFIED, but the repository validation/readme should explain and prove the other outcome classes.

## 11. Failure behavior proof

A public demo does not need to showcase every failure injection, but an engineering reviewer should be able to find validation evidence that the product distinguishes:

```text
invalid/missing plan
ambiguous target element
provider/infrastructure failure
cancellation
replay unavailable
cleanup failure
damaged evidence
```

Do not turn the social video into a test-suite montage.

## 12. Capture quality

Record/capture the local ReproDocket application at readable native resolution.

Required presentation checks:

```text
text legible without zooming the social player excessively
critical outcome not cropped
attempt comparison readable
no clipped status or evidence labels
pointer activity does not distract
no unrelated desktop windows cover the target
no private notification appears
```

Automated UI screenshots remain background-safe where possible. The deliberate public video recording is a separate human-controlled capture, not a justification for foreground-stealing QA automation.

## 13. Screenshot set for README

If screenshots materially improve the README, preferred maximum set:

```text
New Investigation
VERIFIED result with two attempts
Evidence detail
```

A fourth image may show active progress only if it communicates real value.

No screenshot is published before privacy/provenance review.

## 14. Challenge submission proof

Immediately before the public post, re-read the challenge source and verify current requirements.

The post should be able to support these facts from the repository/demo rather than assertion alone:

```text
actual Solari cookbook fork
real Solari use case
public code
real working build demonstrated
AI-assisted development used as requested
required account tags included
```

Do not publish a post until the repository link resolves to the intended public release state.

## 15. Acceptance gate

The demo is acceptable only when:

```text
it was captured from the current validated product
its scenario is independently validated by the fixture truth tests
its final result matches the saved run manifest
it visibly establishes fresh-session verification
all shown data is public-safe
its edits do not falsify timing/order/state
it contains no dead or staged showcase-only controls
its linked repository/docs describe the same scope
```

The demonstration is supporting public evidence. It never replaces Full automated validation or human product acceptance.
