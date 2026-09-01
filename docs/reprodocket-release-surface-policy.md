# ReproDocket Release Surface Policy

Date: August 31, 2026
Status: Pre-implementation publication policy

## 1. Purpose

ReproDocket is being built in a public fork, which means development artifacts are visible before the product is ready. This policy separates useful public engineering evidence from internal execution material so the final repository is deliberate rather than an accidental dump of every planning file created during development.

Nothing in this document claims that the product is implemented or release-ready.

## 2. Public release principles

The final repository should let an outside engineer understand, run, inspect, and evaluate ReproDocket quickly.

Keep material that improves one of these goals:

```text
what the product does
how to run it
how Solari is used
what is actually supported
how the important architecture works
how it was validated
what limitations remain
how to reproduce the public demo
where the source/tests are
what belongs to upstream Solari versus ReproDocket
```

Remove or de-emphasize material that mainly helps an implementation agent remember process and does not help a normal reviewer evaluate the finished product.

## 3. Expected final public keep set

Subject to final runtime truth, these are expected to remain useful publicly:

```text
root README with restrained ReproDocket entry
upstream LICENSE unchanged
reprodocket/ source
tests and deterministic fixture
bootstrap/run/stop/validate scripts
reprodocket/package.json and package-lock.json
reprodocket/README.md
concise architecture/design documentation
current validation documentation
current security/privacy limitations
short reviewer walkthrough
safe demo instructions and selected current screenshots if useful
```

The public design may be shortened after implementation so it describes the actual architecture rather than every preimplementation branch decision.

## 4. Development artifacts requiring explicit final review

These are useful during construction but are not automatically final-release material:

```text
AGENTS.md
docs/implementation/00-contract-reconciliation.md
docs/implementation/01-06 detailed executor plans
docs/implementation/README.md
pre-Codex readiness report
planning-time package-version observations
challenge-specific working notes/drafts
```

At final publication, classify each as:

```text
KEEP
CONDENSE/MOVE
REMOVE_FROM_RELEASE
```

Do not remove them prematurely while Codex still depends on them.

If removed from the release candidate, remove them through an ordinary reviewed commit after their requirements have been reflected in permanent source/tests/public docs. Do not rewrite Git history merely to pretend they never existed.

## 5. AGENTS.md policy

`AGENTS.md` is intentionally present during implementation because it gives agentic development tools persistent repository context.

Before release ask:

```text
Does it still contain useful contributor guidance?
Does it accurately match the final product and commands?
Would an outside engineer benefit from seeing it?
Does it contain implementation-only ceremony better kept out of the product surface?
```

If kept, rewrite it as concise durable contributor/executor guidance rather than a historical build transcript.

If removed, verify no required setup or safety rule exists only in that file.

## 6. Detailed plan policy

The numbered plans are not product documentation and must never be linked from the primary README as proof that features exist.

A plan checkbox is not validation evidence.

If the plans remain public after release, clearly label them historical/development records and ensure no stale `will`, `must`, or planned API language conflicts with actual source behavior.

Preferred final presentation is usually to keep a concise architecture/validation record and remove or archive large agent execution plans from the primary release branch once they are no longer needed.

## 7. Challenge-specific material

The final project should continue to make sense after the hiring challenge ends.

Challenge-specific material may include a short acknowledgment that ReproDocket was created from the Solari cookbook challenge and that AI-assisted development was used as requested.

Do not make the product README read like a job application cover letter.

A social-post draft does not need to live in the repository unless retaining it clearly benefits public project history.

## 8. Validation evidence policy

Do not commit large routine validation output simply to prove effort.

Prefer durable concise evidence such as:

```text
exact source revision
validation profile
commands
pass/fail/block status
fixture version
runtime/package versions
known limitations
human-QA status
```

Large screenshots/replays/log dumps belong in source control only when intentionally selected as small public demo evidence and reviewed for privacy/provenance.

Generated local runs remain ignored by default.

## 9. Screenshot policy

README screenshots should be few, current, legible, and useful.

Preferred public set after final human QA:

```text
one New Investigation view if it explains the workflow
one active run view only if progress is meaningful
one strong VERIFIED result view showing investigation versus verification
one evidence-detail close view if it materially helps
```

Do not publish dozens of screenshots from the QA harness.

Every public screenshot must come from the exact current release candidate or be explicitly labeled historical.

Use only the deterministic synthetic fixture or another target approved as public-safe.

## 10. Public validation claims

The README may state concrete validation facts only when generated from the exact current release candidate.

Good shape:

```text
Validated on Windows 11 x64 with Node <actual version>.
Full validation exercised real Solari browser and sandbox resources.
<actual current counts/results if worth publishing>.
```

Avoid perpetual claims such as:

```text
zero bugs
works on every website
production ready
fully secure
WCAG compliant
no false positives
```

unless a specific scoped audit genuinely supports them.

## 11. Root cookbook README

Preserve the upstream title, examples, gotchas, links, contributing guidance, and license note.

Once ReproDocket is actually usable and validated, add a compact clearly separated fork-specific section near the top that links to `reprodocket/README.md`.

Do not rewrite the repository to imply Pinetree Research authored ReproDocket or that ReproDocket is an upstream Solari example.

## 12. Repository About/topics/releases

Repository metadata is reviewed at final publication only after the product exists.

If the fork metadata can be edited without obscuring its fork lineage, a concise description may mention ReproDocket. Do not add unsupported platform/AI/security claims to topics or description.

Do not create a release/tag before the exact release candidate passes final required validation and human acceptance.

## 13. Commit history

Normal development history may show planning and incremental work. Do not destructively rewrite public history merely to make the project look as though it appeared fully formed.

The final merge strategy should produce a readable main branch while preserving traceability and obeying any repository rules.

## 14. Final release-surface audit

Before publication, review every file changed from upstream main and record:

```text
why it exists
whether it is product/test/fixture/script/public-doc/development-only
whether normal reviewers should see it
whether it contains stale capability wording
whether it contains private or machine-specific information
whether links/commands match final source
```

No development-only artifact is retained simply because nobody remembered to review it.
