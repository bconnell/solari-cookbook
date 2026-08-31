# ReproDocket Test Matrix

Date: August 31, 2026
Status: Pre-implementation acceptance matrix

This matrix is the minimum initial validation catalog for ReproDocket version one. Tests may be added as implementation reveals new failure classes. Removing or weakening a listed case requires an explicit design reason and equivalent coverage.

The matrix distinguishes cheap deterministic tests from live Solari tests so normal development does not create unnecessary billable sessions. A final FULL validation still requires the live authorities.

## Result vocabulary

Test result:

```text
PASS
FAIL
BLOCKED
SKIPPED_NOT_APPLICABLE
```

Product run lifecycle:

```text
CREATED
PREPARING
INVESTIGATING
VERIFYING
FINALIZING
COMPLETED
FAILED
CANCELLED
INTERRUPTED
```

Product investigation outcome:

```text
VERIFIED
REPRODUCED
NOT_REPRODUCED
INCONCLUSIVE
```

A test runner result must never be confused with a product investigation outcome.

## A. Static and repository validation

| ID | Test | Expected authority | Live Solari |
| --- | --- | --- | --- |
| A001 | TypeScript compile | No type errors | No |
| A002 | Production UI build | Build exits zero and emits expected static bundle | No |
| A003 | Server build | Server entry compiles for supported Node runtime | No |
| A004 | ESLint | No configured lint errors | No |
| A005 | Format policy | Tracked product files satisfy configured formatter | No |
| A006 | Lockfile tracked | `reprodocket/package-lock.json` is not ignored and is present after dependency install | No |
| A007 | Generated output ignored | dist, coverage, reports, runtime state, local runs, and secrets are excluded from Git | No |
| A008 | Secret pattern scan | No real or fixture-identical secret in tracked files | No |
| A009 | Public private-path scan | Public docs do not contain developer-specific absolute paths | No |
| A010 | Public placeholder scan | Product UI/public docs contain no accidental TODO/FIXME/stub/sample-only claims at release gate | No |
| A011 | Dependency role check | Runtime packages used by product are in dependencies; test/build packages are devDependencies | No |
| A012 | Unsupported import guard | Production code does not import fixture implementation | No |
| A013 | Fixture identity guard | Production code does not branch on known fixture route/scenario IDs to force expected outcomes | No |

## B. Core model and lifecycle unit tests

| ID | Test | Expected authority | Live Solari |
| --- | --- | --- | --- |
| B001 | New run state | New run begins CREATED with no outcome | No |
| B002 | Legal transition CREATED to PREPARING | Accepted | No |
| B003 | Legal transition PREPARING to INVESTIGATING | Accepted | No |
| B004 | Legal transition INVESTIGATING to VERIFYING | Accepted | No |
| B005 | Legal transition VERIFYING to FINALIZING | Accepted | No |
| B006 | Legal transition FINALIZING to COMPLETED | Accepted | No |
| B007 | Failure transition | Any active state can move to FAILED through defined failure path | No |
| B008 | Cancel transition | Active state can move to CANCELLED through defined cancellation path | No |
| B009 | Restart interruption | Nonterminal persisted prior-process run becomes INTERRUPTED | No |
| B010 | Terminal immutability | COMPLETED/FAILED/CANCELLED/INTERRUPTED cannot silently return to active | No |
| B011 | Completed not reproduced | COMPLETED + NOT_REPRODUCED is valid | No |
| B012 | Failed not healthy | FAILED cannot imply NOT_REPRODUCED | No |
| B013 | Verified evidence requirement | VERIFIED rejected without first-run evidence | No |
| B014 | Verified second evidence requirement | VERIFIED rejected without verification-run evidence | No |
| B015 | Verified distinct sessions | VERIFIED rejected when investigation and verification session IDs match | No |
| B016 | Reproduced definition | First run defect + unsuccessful clean confirmation produces REPRODUCED, not VERIFIED | No |
| B017 | Not reproduced definition | Sufficient workflow completion with no defect produces NOT_REPRODUCED | No |
| B018 | Inconclusive definition | Insufficient authoritative observation produces INCONCLUSIVE | No |
| B019 | Evidence conflict | Contradictory evidence fails toward uncertainty rather than VERIFIED | No |
| B020 | Stable error codes | Known failures map to stable machine codes independent of exception prose | No |

## C. Target URL security tests

| ID | Input | Expected | Live Solari |
| --- | --- | --- | --- |
| C001 | `https://example.com` | Accepted | No |
| C002 | `http://example.com` | Accepted | No |
| C003 | `file:///C:/Windows/win.ini` | INVALID_TARGET_URL | No |
| C004 | `javascript:alert(1)` | INVALID_TARGET_URL | No |
| C005 | `data:text/html,...` | INVALID_TARGET_URL | No |
| C006 | `blob:https://example.com/...` | INVALID_TARGET_URL as top-level target | No |
| C007 | `https://user:pass@example.com` | Rejected credentials-in-URL | No |
| C008 | `http://localhost` | BLOCKED_TARGET_NETWORK | No |
| C009 | `http://foo.localhost` | BLOCKED_TARGET_NETWORK | No |
| C010 | `http://device.local` | BLOCKED_TARGET_NETWORK | No |
| C011 | `http://127.0.0.1` | BLOCKED_TARGET_NETWORK | No |
| C012 | `http://127.12.2.3` | BLOCKED_TARGET_NETWORK | No |
| C013 | `http://10.0.0.1` | BLOCKED_TARGET_NETWORK | No |
| C014 | `http://172.16.0.1` | BLOCKED_TARGET_NETWORK | No |
| C015 | `http://172.31.255.255` | BLOCKED_TARGET_NETWORK | No |
| C016 | `http://192.168.1.1` | BLOCKED_TARGET_NETWORK | No |
| C017 | `http://169.254.169.254` | BLOCKED_TARGET_NETWORK | No |
| C018 | `http://[::1]` | BLOCKED_TARGET_NETWORK | No |
| C019 | IPv6 unique-local literal | BLOCKED_TARGET_NETWORK | No |
| C020 | IPv6 link-local literal | BLOCKED_TARGET_NETWORK | No |
| C021 | Malformed host | INVALID_TARGET_URL | No |
| C022 | Whitespace-only URL | INVALID_TARGET_URL | No |
| C023 | Public URL redirect to blocked local address | Navigation aborted or INCONCLUSIVE, never accepted as valid target | Conditional live fixture |
| C024 | DNS resolves to private address | Rejected where resolution check can be authoritatively performed | Conditional live fixture |

## D. Local server security tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| D001 | Bind address | Server listens on loopback, not all interfaces | No |
| D002 | Unexpected Host | Rejected | No |
| D003 | Same-origin GET | Allowed where route is read-only | No |
| D004 | Same-origin mutation | Allowed with valid local session nonce | No |
| D005 | Foreign Origin mutation | LOCAL_REQUEST_REJECTED | No |
| D006 | Missing nonce mutation | LOCAL_REQUEST_REJECTED | No |
| D007 | Wrong nonce mutation | LOCAL_REQUEST_REJECTED | No |
| D008 | Wildcard CORS probe | No wildcard CORS response | No |
| D009 | CSP present | Restrictive production CSP returned | No |
| D010 | MIME sniffing header | `nosniff` present | No |
| D011 | Referrer policy | No-referrer or equally restrictive configured policy | No |
| D012 | Path traversal `../` | Rejected | No |
| D013 | Encoded traversal `%2e%2e` | Rejected | No |
| D014 | Cross-run artifact ID | Cannot fetch artifact not owned by requested run | No |
| D015 | Symlink/reparse escape where supported | Artifact server refuses escape | No |

## E. Secret storage and redaction tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| E001 | Missing credential | Provider state Not configured | No |
| E002 | Environment credential precedence | Environment source selected when present | No |
| E003 | Protected store fallback | Stored encrypted credential selected when env missing | No |
| E004 | Plaintext absent at rest | Protected store file does not contain submitted plaintext | No |
| E005 | PowerShell helper stdin | Secret does not appear in child command line | No |
| E006 | Round trip protect/unprotect | Current user can recover exact test secret | No |
| E007 | Wrong/invalid protected payload | Sanitized storage error, no crash | No |
| E008 | Fake Solari key in console text | Redacted before persistence | No |
| E009 | Bearer header | Header value not persisted | No |
| E010 | Cookie header | Not persisted | No |
| E011 | Set-Cookie header | Not persisted | No |
| E012 | Password action value | Not persisted in timeline | No |
| E013 | Secret in provider exception | Sanitized before log/report/UI | No |
| E014 | Secret scan generated output | Known injected fake secret absent from JSON/HTML/logs | No |

## F. Persistence and evidence integrity tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| F001 | Unique run directory | Two runs never share a directory | No |
| F002 | Fail-if-exists creation | Existing run ID is never overwritten | No |
| F003 | Atomic manifest update | Readers see prior valid or new valid manifest, not truncated JSON | No |
| F004 | Malformed manifest | Run isolated as damaged; history service remains available | No |
| F005 | Unsupported manifest version | Explicit incompatible/damaged state | No |
| F006 | Hash generation | Final artifacts receive SHA256 digests | No |
| F007 | Hash verification | Untampered artifact validates | No |
| F008 | Tampered artifact | EVIDENCE_INVALID | No |
| F009 | Missing finalized artifact | EVIDENCE_INVALID or explicit missing state | No |
| F010 | Cross-run file swap | Digest/ownership validation rejects it | No |
| F011 | Active unsealed evidence | UI can distinguish from finalized evidence | No |
| F012 | Restart reload | Completed run reloads with same authoritative outcome | No |
| F013 | Interrupted reload | Prior active run becomes INTERRUPTED without re-executing actions | No |
| F014 | Damaged one run | Other valid history rows still load | No |
| F015 | Report JSON parity | report.json derives from same authoritative run as UI | No |
| F016 | HTML parity | Key outcome/steps/evidence match report.json | No |
| F017 | Malicious HTML problem text | Escaped in report | No |
| F018 | Malicious page title | Escaped in report | No |
| F019 | Malicious console text | Escaped in report | No |
| F020 | Report works without script | Human-readable static HTML | No |

## G. API and coordinator integration tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| G001 | Create investigation valid request | Returns authoritative run ID | No with adapter double |
| G002 | Invalid request shape | 4xx with stable code | No |
| G003 | Invalid target | Rejected before adapter invocation | No |
| G004 | Active run duplicate | RUN_ALREADY_ACTIVE | No |
| G005 | Racing two creates | Exactly one admitted | No |
| G006 | Get run | Returns persisted authority | No |
| G007 | List history | Stable ordering and truthful status | No |
| G008 | Missing run | RUN_NOT_FOUND | No |
| G009 | Damaged run | Does not crash list endpoint | No |
| G010 | Cancel active run | Cancellation state propagated | No with adapter double |
| G011 | Cancel terminal run | Safe deterministic response; no second cleanup | No |
| G012 | SSE initial state | Subscriber receives current authoritative state | No |
| G013 | SSE transition sequence | Events preserve lifecycle order | No |
| G014 | SSE reconnect | Client can recover state from authoritative GET even if events were missed | No |
| G015 | Evidence route valid artifact | Serves owned artifact with correct MIME | No |
| G016 | Evidence route invalid artifact | Rejected | No |
| G017 | Replay unavailable | API returns explicit replay state, not broken link | No |
| G018 | Local server graceful stop | Stops accepting new work before cleanup | No |

## H. Local UI user-boundary tests

These tests use the real built ReproDocket local application and a browser automation runner. They must not substitute direct API calls for the claimed user workflow.

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| H001 | First launch no key | Connection surface visible and understandable | No |
| H002 | Invalid key submit | Sanitized failure shown; not READY | Optional live credential test |
| H003 | Valid configured state | New Investigation form reachable | No with prepared secret store; live in FULL |
| H004 | Target required | Form prevents empty submit and server agrees | No |
| H005 | Problem required | Form prevents empty submit and server agrees | No |
| H006 | Optional steps omitted | Submission remains valid | No |
| H007 | Submit investigation | Navigates/progresses to authoritative active run | No with adapter double; live in E2E |
| H008 | Double click Investigate | One run only | No |
| H009 | Active progress | Current stage updates without page refresh | No |
| H010 | Browser refresh during active run | Rehydrates current state | No |
| H011 | Browser back/forward | Does not corrupt active run or history | No |
| H012 | Completed VERIFIED detail | Outcome and two evidence authorities visible | No fixture state; live E2E validates truth |
| H013 | Completed NOT_REPRODUCED detail | Correct outcome text, no false error styling | No |
| H014 | Completed INCONCLUSIVE detail | Reason and safe next action visible | No |
| H015 | REPRODUCED detail | Distinct from VERIFIED | No |
| H016 | History ordering | Recent runs visible and selectable | No |
| H017 | History/detail consistency | Same outcome/lifecycle across surfaces | No |
| H018 | Screenshot viewer | Correct run-owned image displayed | No |
| H019 | Console view | Sanitized evidence rendered as text | No |
| H020 | Network view | Sanitized method/url/status/failure rendered | No |
| H021 | Timeline view | Ordered actions/stages visible | No |
| H022 | Replay ready | Working control shown only when available | No state test; live contract later |
| H023 | Replay pending | Pending state, not dead link | No |
| H024 | Replay unavailable | Explicit unavailable state | No |
| H025 | Cancel visible active run | Cancellation request and terminal state visible | No with adapter double; live cancellation later |
| H026 | Error state navigation | User can return to history/new investigation safely | No |
| H027 | Damaged historical run | History remains usable and damage is explicit | No |
| H028 | Narrow viewport | No critical horizontal overflow | No |
| H029 | 100 percent desktop scale baseline | Primary controls/text readable | No + human |
| H030 | 125/150 percent browser zoom smoke | Critical workflow remains usable | No + human |
| H031 | Keyboard navigation | Main form, history, evidence tabs, and cancel path keyboard reachable | No + human spot check |
| H032 | Accessible names | Critical form controls/buttons expose names/roles | No |
| H033 | Focus visibility | Keyboard focus is visually discoverable | No + human |
| H034 | Malicious target text | Displayed as text, not executed | No |
| H035 | No dead controls | Every visible interaction has a real or truthfully disabled behavior | No |

## I. Solari browser live contract tests

These tests use the real installed `@solarisdk/browser` package and a real Solari account.

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| I001 | Client construction | Current SDK accepts configured credential/base URL | Yes |
| I002 | Browser create | Session ID returned | Yes |
| I003 | New page | Page available | Yes |
| I004 | Navigate public fixture/example | Final URL/title observable | Yes |
| I005 | Locator interaction | Real element can be read/clicked/fill as fixture requires | Yes |
| I006 | Screenshot bytes | Nonempty valid PNG/JPEG as configured | Yes |
| I007 | Console capture | Known fixture console error observed when triggered | Yes |
| I008 | Page error capture | Known fixture uncaught error observed when triggered | Yes |
| I009 | Network failure/status capture | Known fixture failing request observed | Yes |
| I010 | Recording create | Session created with recording enabled | Yes |
| I011 | Close browser | Adapter close completes | Yes |
| I012 | Process handle cleanup | Test process exits without leaked client handle | Yes |
| I013 | Replay readiness | Bounded polling reaches Ready or truthful Unavailable/timeout state | Yes |
| I014 | Replay URL | Valid replay URL retrieved when documented path succeeds | Yes |
| I015 | Replay download probe | If installed TS SDK exposes supported download, bytes are retained and identified; otherwise capability remains unsupported without failure | Yes |
| I016 | Distinct session creation | Two sequential launch calls yield different IDs | Yes |
| I017 | 429 handling | Capacity classification does not blindly rapid-retry | Yes only when safely inducible; otherwise deterministic adapter error test |
| I018 | 502/503/504 policy | Bounded retry classifier treats as eligible only for safe operations | Deterministic policy test; live occurrence not required |

## J. Solari sandbox live contract tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| J001 | Sandbox client construction | Current package compiles and connects | Yes |
| J002 | Sandbox create | Owned sandbox ID returned | Yes |
| J003 | Connect | Control channel available | Yes |
| J004 | Write fixture file | File operation succeeds | Yes |
| J005 | Run fixture command | Transport success plus command exitCode 0 | Yes |
| J006 | Background server start | Fixture server remains available after command returns | Yes |
| J007 | Preview URL | Current package preview method returns URL | Yes |
| J008 | External preview probe | URL returns expected fixture identity outside sandbox | Yes |
| J009 | Kill sandbox | Kill completes | Yes |
| J010 | No reliance on idle pause | Harness explicitly requests kill | Yes |
| J011 | Failed command | Nonzero command exit is recognized as failure | Yes or deterministic harmless bad command |
| J012 | Cleanup after setup exception | Owned sandbox still killed | Yes |

## K. Deterministic fixture truth tests

These tests validate the fixture independently from ReproDocket classification so the oracle itself is trustworthy.

| ID | Scenario | Ground truth assertion | Live Solari |
| --- | --- | --- | --- |
| K001 | Account save blank panel | Exact action deterministically causes panel to become blank and emits defined evidence cue | Local fixture + live preview in FULL |
| K002 | Account save reset | New fixture session starts from clean state | Local + live |
| K003 | Billing route refresh | Defined route refresh deterministically returns/lands in 404 state | Local + live |
| K004 | Missing ZIP validation | Form accepts prohibited missing ZIP and emits defined success cue | Local + live |
| K005 | Healthy profile save | Valid save remains visible and returns expected confirmation | Local + live |
| K006 | Healthy login validation | Invalid login is handled by intended validation rather than target crash | Local + live |
| K007 | Ambiguous case | Fixture intentionally withholds enough evidence for deterministic classification | Local + live |
| K008 | Fixture version identity | Served fixture exposes non-secret version used in validation provenance | Local + live |
| K009 | No ReproDocket special route | Fixture truth is independent from product implementation | Source guard |

## L. Full investigation and verification tests

| ID | Scenario | Expected product result | Live Solari |
| --- | --- | --- | --- |
| L001 | Account save defect first + second run | VERIFIED | Yes |
| L002 | Billing refresh defect first + second run | VERIFIED | Yes |
| L003 | Missing ZIP defect first + second run | VERIFIED | Yes |
| L004 | Healthy profile save | NOT_REPRODUCED | Yes |
| L005 | Healthy login validation | NOT_REPRODUCED | Yes |
| L006 | Ambiguous report | INCONCLUSIVE | Yes |
| L007 | First run defect, second run intentionally nonrepeatable fixture variant | REPRODUCED | Yes if fixture can model without production branching |
| L008 | Distinct session IDs | First and second IDs differ for every verification attempt | Yes |
| L009 | First session closed before second | No overlapping reuse unless design later explicitly requires overlap | Yes |
| L010 | Evidence attribution | Investigation artifacts owned by first authority, verification by second | Yes |
| L011 | Final manifest | Contains both authorities and final classification | Yes |
| L012 | Local UI projection | UI shows same final outcome and evidence IDs as manifest | Yes |
| L013 | Restart after completion | Same run still available after local server restart | Yes for run generation, local restart no additional live session |
| L014 | Cleanup | All harness-owned browsers and fixture sandbox closed/killed | Yes |

## M. Failure injection and recovery tests

| ID | Failure | Required result | Live Solari |
| --- | --- | --- | --- |
| M001 | Missing credential | No session created; actionable local message | No |
| M002 | Invalid credential | Not READY; sanitized provider error | Yes once with invalid test value |
| M003 | Browser creation adapter failure | Run FAILED/INCONCLUSIVE by policy; no false healthy outcome | No |
| M004 | Real browser create failure when naturally encountered | Same truthful handling | Opportunistic |
| M005 | Navigation timeout | INCONCLUSIVE or FAILED per stage policy; evidence preserved | Local adapter + live fixture delay test if economical |
| M006 | Target 500 | Target condition recorded; classification depends on report, never infrastructure success | Yes fixture |
| M007 | Browser dies mid-run | Partial evidence preserved; cleanup reconciled | Adapter deterministic; live only if safely inducible |
| M008 | Evidence write failure | EVIDENCE_WRITE_FAILED and no COMPLETED claim | No using controlled unwritable/failing store |
| M009 | Verification browser unavailable | First result may be REPRODUCED/INCONCLUSIVE but never VERIFIED | No + opportunistic live |
| M010 | Replay initially 404/not ready | Bounded poll continues without misclassifying main run | Yes |
| M011 | Replay never ready within budget | Run can complete with REPLAY_UNAVAILABLE if replay is noncritical and evidence sufficient | Yes if naturally reproducible; deterministic otherwise |
| M012 | Sandbox create failure | E2E BLOCKED/FAIL, no browser run against nonexistent fixture | No + opportunistic live |
| M013 | Fixture command nonzero | Fixture setup fails; sandbox cleanup still attempted | Yes |
| M014 | Fixture preview not reachable | No investigation begins; cleanup occurs | Yes or deterministic transport double |
| M015 | Preferred local port occupied | Alternate loopback port chosen; unrelated process untouched | No |
| M016 | Stale PID record | Unrelated/recycled process not killed | No |
| M017 | App termination with active run simulated | Restart marks run INTERRUPTED | No |
| M018 | Graceful shutdown active run | Cancellation cleanup path executes | No + live final smoke |
| M019 | Cancel during navigation | New actions stop; owned resources close; CANCELLED persisted | No + live final smoke |
| M020 | Cancel during verification | No false VERIFIED; resources close | No + live final smoke |
| M021 | Malformed persisted run | Isolated damage, application remains usable | No |
| M022 | Missing screenshot after finalization | Evidence invalid state | No |
| M023 | Cross-run screenshot substitution | Evidence invalid state | No |
| M024 | Foreign-origin POST | Request rejected without state mutation | No |
| M025 | Duplicate submission race | One admitted, one RUN_ALREADY_ACTIVE | No |

## N. Horizontal connectivity tests

| ID | Compared surfaces | Required invariant |
| --- | --- | --- |
| N001 | Active UI vs run manifest | Same lifecycle state |
| N002 | History vs run detail | Same lifecycle and outcome |
| N003 | Outcome badge vs report JSON | Same final outcome |
| N004 | HTML report vs report JSON | Same target, problem, steps, outcomes, key evidence IDs |
| N005 | Screenshot UI vs manifest | Every shown image belongs to manifest; missing entries not invented |
| N006 | Console UI vs persisted console | Same sanitized event set/order within documented normalization |
| N007 | Network UI vs persisted network | Same sanitized event set/order within documented normalization |
| N008 | Replay button vs replay state | Button available only when authoritative replay reference is ready |
| N009 | Cleanup badge vs resource ledger | PASS only when required resources terminal/closed |
| N010 | Credential UI vs credential authority | READY only after configured source validated |
| N011 | Validation summary vs current Git revision | Revision identity matches run metadata |
| N012 | README capability table vs validation state | No capability claimed Available without applicable completion gate |
| N013 | Error UI vs machine code | Human message corresponds to correct stable error code |
| N014 | Cancel UI vs lifecycle | Cancelled display backed by terminal CANCELLED state |
| N015 | History after restart vs prior history | No disappeared valid run without explicit retention action |

## O. Vertical connectivity tests

| ID | User outcome | Required vertical path |
| --- | --- | --- |
| O001 | Configure credential | UI -> API -> secret service -> protected store -> verification -> persisted readiness metadata -> UI |
| O002 | Start investigation | UI -> validation -> coordinator -> run store -> Solari browser -> evidence -> UI progress |
| O003 | Verify defect | First evidence -> first cleanup -> second Solari session -> second evidence -> classifier -> manifest -> UI |
| O004 | View screenshot | Run detail -> artifact ID -> manifest ownership -> integrity check -> artifact response -> browser render |
| O005 | View replay | Run detail -> replay state -> stored reference/data -> safe presentation |
| O006 | Cancel | UI -> API -> coordinator -> cancellation signal -> resource cleanup -> terminal persistence -> UI |
| O007 | Restart and history | process start -> run store scan -> schema/integrity validation -> history API -> UI |
| O008 | Damaged historical evidence | persisted damage -> validator -> damaged run projection -> UI diagnostic without global crash |
| O009 | Full fixture validation | validation script -> sandbox -> fixture -> public preview -> browser A -> browser B -> local store -> local UI -> cleanup -> validation report |

## P. Resource and lifetime tests

| ID | Test | Expected | Live Solari |
| --- | --- | --- | --- |
| P001 | Browser close normal | Resource ledger records requested/completed close | Yes |
| P002 | Browser close on exception | Close still attempted | Yes |
| P003 | Verification browser close | Independent close recorded | Yes |
| P004 | Sandbox kill normal | Kill recorded | Yes |
| P005 | Sandbox kill on fixture failure | Kill still attempted | Yes |
| P006 | Local server stop | Owned server stops | No |
| P007 | Stop script ownership mismatch | Process untouched | No |
| P008 | Repeated start | Existing owned instance reused rather than duplicated | No |
| P009 | Foreign preferred port | Foreign process untouched | No |
| P010 | Open Node handles | Normal validation process terminates cleanly | No + live browser contract |
| P011 | Temporary file cleanup | Only run-owned temporary files removed/renamed | No |
| P012 | Cleanup failure | Visible as failure, not swallowed | No |

## Q. Visual and human QA catalog

Automated assertions prepare evidence but do not replace subjective review.

Required product views for final human review:

| ID | View/state | Required framing |
| --- | --- | --- |
| Q001 | First run connection | Full application context |
| Q002 | New Investigation | Full context + form readability |
| Q003 | Active Investigation | Full context + close view of progress states |
| Q004 | VERIFIED result | Full detail + close evidence/outcome area |
| Q005 | REPRODUCED result | Full detail |
| Q006 | NOT_REPRODUCED result | Full detail |
| Q007 | INCONCLUSIVE result | Full detail + reason/action area |
| Q008 | History | Full context with several run types |
| Q009 | Screenshot evidence | Close view of image viewer and provenance context |
| Q010 | Console evidence | Close readable text |
| Q011 | Network evidence | Close readable rows/details |
| Q012 | Replay state | Ready and unavailable/pending states as applicable |
| Q013 | Error condition | Representative sanitized failure |
| Q014 | Narrow viewport | Full view at supported minimum width |
| Q015 | Browser zoom/readability | Representative 125 or 150 percent inspection |

Human review checks:

* visual hierarchy,
* readable text,
* no clipped or contradictory labels,
* clear distinction between lifecycle failure and defect outcome,
* clear distinction between investigation and verification evidence,
* obvious current run identity,
* evidence controls that look actionable only when they are actionable,
* sensible empty states,
* useful error copy,
* no internal implementation prose leaking into product UI,
* no debug output or raw stack traces,
* no misleading visual success when evidence is incomplete.

Any defect a human finds that deterministic automation could reasonably have detected creates a new regression test before final closure.

## R. Bootstrap and Windows automation tests

| ID | Test | Expected |
| --- | --- | --- |
| R001 | Run from repository root | Correct ReproDocket root found |
| R002 | Run from another current directory | Script resolves its own location correctly |
| R003 | PowerShell 5.1 bootstrap entry | Detects/reroutes to PowerShell 7 without parser failure |
| R004 | PowerShell 7 present | No unnecessary installation |
| R005 | Node supported | Continues |
| R006 | Node missing | Safe automated install path attempted when package manager available |
| R007 | Node unsupported old version | Clear upgrade path, not false PASS |
| R008 | npm missing/broken | Preflight fails diagnostically |
| R009 | Dependencies missing | Restore executes |
| R010 | Lockfile present | `npm ci` used for deterministic restore after lock exists |
| R011 | Insufficient disk reserve | Artifact-heavy/full validation stops before failure |
| R012 | App already owned/running | Opens/reuses it |
| R013 | Preferred port occupied foreign | Alternate port chosen |
| R014 | Browser open failure | Server remains usable and URL printed clearly |
| R015 | `stop.ps1` valid ownership | Stops only owned process |
| R016 | `stop.ps1` stale ownership | Refuses unrelated process |
| R017 | `validate.ps1` mandatory stage fail | Nonzero exit |
| R018 | `validate.ps1` all required stages pass | Zero exit only after all authorities pass |
| R019 | WMIC source scan | No `wmic` or `wmic.exe` in new scripts |
| R020 | No user-folder scratch default | No Desktop/Downloads/Documents scratch output |

## S. Sensitivity and mutation checks for the harness

The validation harness itself must prove it can fail.

| ID | Deliberate regression | Test expected to fail |
| --- | --- | --- |
| S001 | Force classifier to return VERIFIED without second evidence | B014/B015/L008 |
| S002 | Reuse investigation session ID as verification session | B015/L008 |
| S003 | Disable URL scheme guard | C003/C004/C005 |
| S004 | Disable private IP guard | C011-C020 |
| S005 | Remove origin/nonce check | D005-D007 |
| S006 | Render report field unescaped | F017-F019 |
| S007 | Skip redaction | E008-E014 |
| S008 | Serve arbitrary artifact path | D012-D015 |
| S009 | Do not hash final screenshot | F006/F009 |
| S010 | Show screenshot from another run | F010/N005 |
| S011 | Permit two active pipelines | G004/G005/P008 |
| S012 | Mark cancellation COMPLETED | B008/H025/O006 |
| S013 | Swallow cleanup error | P012/L014 |
| S014 | Skip sandbox kill | J009/J010/L014 |
| S015 | Always classify target as broken | K005/K006/L004/L005 |
| S016 | Always classify target healthy | K001/K003/K004/L001-L003 |
| S017 | Fixture-specific hardcoded outcome | A012/A013/K009 |
| S018 | Stale validation revision | N011/final provenance gate |

At least representative guards from each critical class must undergo an actual red-green or mutation proof during implementation. Merely listing these sensitivity checks is not sufficient.

## T. FULL validation acceptance

A FULL validation run is accepted only when all applicable mandatory categories have current results for the exact source revision being claimed:

```text
Static/repository
Core unit
Security unit
Persistence/integrity
Local integration
Built local UI user boundary
Solari browser contract
Solari sandbox contract
Fixture truth
Full investigation/verification scenarios
Failure/recovery
Horizontal connectivity
Vertical connectivity
Resource/lifetime
Bootstrap/runtime
Visual evidence preparation
Security/publication scans
```

Human visual and usability review remains a separate final authority. A machine FULL PASS is not renamed to human QA PASS.

## U. Execution economy

Normal targeted development should run the cheapest authoritative checks first:

```text
changed unit tests
-> affected integration tests
-> affected local UI tests
-> build/lint/typecheck
-> live Solari contract only when its boundary changed
-> full Solari E2E only at milestone and final gates
```

If a cheap deterministic test fails, the pipeline stops before creating unnecessary billable Solari resources.

A final milestone or publication claim still requires the full live path regardless of cost optimization.