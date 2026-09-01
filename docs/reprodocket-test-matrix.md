# ReproDocket Test Matrix

Date: August 31, 2026
Status: Pre-implementation acceptance matrix

This is the minimum version-one validation catalog. Implementation may add tests as new failure classes are discovered. Listed coverage may not be silently weakened or removed. A final FULL validation requires the applicable live Solari authorities as well as local deterministic authorities.

## Result vocabulary

Test result: `PASS`, `FAIL`, `BLOCKED`, `SKIPPED_NOT_APPLICABLE`.

Run lifecycle: `CREATED`, `PREPARING`, `INVESTIGATING`, `VERIFYING`, `FINALIZING`, `COMPLETED`, `FAILED`, `CANCELLED`, `INTERRUPTED`.

Run outcome: `VERIFIED`, `REPRODUCED`, `NOT_REPRODUCED`, `INCONCLUSIVE`.

A test-runner result is never interchangeable with a product outcome.

## A. Static, repository, and contract validation

| ID | Test | Required result |
| --- | --- | --- |
| A001 | TypeScript compile | No type errors |
| A002 | Production UI build | Exit zero and expected static bundle |
| A003 | Server build | Server entry compiles for supported Node runtime |
| A004 | ESLint | No configured lint errors |
| A005 | Format policy | Tracked product files satisfy formatter |
| A006 | Lockfile tracking | `reprodocket/package-lock.json` present and not ignored after install |
| A007 | Generated-output hygiene | dist, coverage, reports, runtime state, local runs, secrets ignored |
| A008 | Secret scan | No real credential in tracked files |
| A009 | Private-path scan | Public docs contain no developer-specific absolute paths |
| A010 | Placeholder/public-copy scan | No accidental TODO/FIXME/stub/sample-only production claims at release gate |
| A011 | Dependency roles | Runtime dependencies and dev dependencies correctly classified |
| A012 | Fixture import guard | Production code does not import fixture implementation |
| A013 | Fixture identity guard | Production code does not branch on fixture scenario/route to force outcome |
| A014 | Contract identifier consistency | Production uses current `PlanStatement`, `ParsedPlanStatement`, `parsePlanStatements` names |
| A015 | Documentation consistency | No active doc states that the version-one plan is optional |

## B. Plan parsing, request validation, lifecycle, and classification

| ID | Test | Required result |
| --- | --- | --- |
| B001 | Action grammar | Every supported action parses and preserves quoted case |
| B002 | Expectation grammar | Every supported expectation parses |
| B003 | Unknown statement | `INVALID_PLAN_STATEMENT` before external work |
| B004 | Trailing garbage | Rejected |
| B005 | Empty required quoted value | Rejected |
| B006 | Missing/empty plan | Rejected |
| B007 | Action-only plan | `PLAN_EXPECTATION_REQUIRED` or documented validation equivalent |
| B008 | Expectation-only plan | Rejected for missing executable action |
| B009 | Complete plan | At least one action plus one expectation accepted |
| B010 | New run | `CREATED`, no outcome |
| B011 | CREATED -> PREPARING | Legal |
| B012 | PREPARING -> INVESTIGATING | Legal |
| B013 | INVESTIGATING -> VERIFYING | Legal |
| B014 | VERIFYING -> FINALIZING | Legal |
| B015 | FINALIZING -> COMPLETED | Legal |
| B016 | Failure transition | Defined active states can enter `FAILED` |
| B017 | Cancellation transition | Defined active states can enter `CANCELLED` |
| B018 | Restart interruption | Prior-process nonterminal run becomes `INTERRUPTED` |
| B019 | Terminal immutability | Terminal lifecycle cannot silently become active |
| B020 | Completed negative result | `COMPLETED + NOT_REPRODUCED` valid |
| B021 | Failed is not healthy | `FAILED` does not imply `NOT_REPRODUCED` |
| B022 | VERIFIED first authority | Rejected without investigation reproduction evidence |
| B023 | VERIFIED second authority | Rejected without verification reproduction evidence |
| B024 | VERIFIED session independence | Rejected when session IDs match or are null |
| B025 | REPRODUCED definition | First confirms, clean second does not confirm |
| B026 | NOT_REPRODUCED definition | Sufficient workflow plus tested defect expectation not observed |
| B027 | INCONCLUSIVE definition | Insufficient/ambiguous authority remains uncertain |
| B028 | Conflicting evidence | Does not promote to VERIFIED |
| B029 | Stable error codes | Machine code independent of raw exception prose |

## C. Target URL and navigation safety

| ID | Input/condition | Required result |
| --- | --- | --- |
| C001 | `https://example.com` | Accepted |
| C002 | `http://example.com` | Accepted |
| C003 | `file:///...` | `INVALID_TARGET_URL` |
| C004 | `javascript:...` | `INVALID_TARGET_URL` |
| C005 | `data:...` | `INVALID_TARGET_URL` |
| C006 | top-level `blob:` | Rejected |
| C007 | credentials embedded in URL | Rejected |
| C008 | `localhost` | `BLOCKED_TARGET_NETWORK` |
| C009 | subdomain of `.localhost` | Blocked |
| C010 | `.local` target | Blocked |
| C011 | IPv4 loopback including 127/8 | Blocked |
| C012 | RFC1918 10/8 | Blocked |
| C013 | RFC1918 172.16/12 | Blocked |
| C014 | RFC1918 192.168/16 | Blocked |
| C015 | IPv4 link-local | Blocked |
| C016 | common metadata address `169.254.169.254` | Blocked |
| C017 | IPv6 loopback | Blocked |
| C018 | IPv6 unique-local | Blocked |
| C019 | IPv6 link-local | Blocked |
| C020 | malformed host/URL | Rejected |
| C021 | public redirect to prohibited destination | Abort or truthful INCONCLUSIVE; never accepted silently |
| C022 | DNS resolves privately | Block where authoritative resolution data is available; otherwise limitation documented |
| C023 | absolute `OPEN`/`WAIT_FOR_URL`/`EXPECT_URL` destination | Same policy as initial target |
| C024 | redirect after initial allowed URL | Revalidated at strongest available browser/provider boundary |

## D. Local service and artifact security

| ID | Test | Required result |
| --- | --- | --- |
| D001 | Listener bind | Loopback only |
| D002 | Unexpected Host | Rejected |
| D003 | Same-origin read | Allowed where read-only |
| D004 | Same-origin mutation + valid nonce | Allowed |
| D005 | Foreign Origin mutation | `LOCAL_REQUEST_REJECTED` |
| D006 | Missing nonce mutation | Rejected |
| D007 | Wrong nonce mutation | Rejected |
| D008 | CORS | No wildcard CORS |
| D009 | CSP | Restrictive production policy present |
| D010 | MIME sniffing | `nosniff` present |
| D011 | Referrer policy | Restrictive policy present |
| D012 | `../` artifact traversal | Rejected |
| D013 | encoded traversal | Rejected |
| D014 | cross-run artifact ID | Rejected |
| D015 | symlink/reparse escape where supported | Rejected |
| D016 | malformed/oversized mutation payload | Bounded safe error |
| D017 | unsupported content type | Rejected where mutation endpoint expects JSON |

## E. Secret storage and redaction

| ID | Test | Required result |
| --- | --- | --- |
| E001 | Missing credential | Provider state Not configured |
| E002 | Environment credential | Used when explicitly present |
| E003 | Protected-store fallback | Used when environment value absent |
| E004 | Plaintext at rest | Submitted secret absent from protected store bytes |
| E005 | PowerShell credential helper | Secret absent from child command line |
| E006 | Protect/unprotect | Current user recovers exact fake test secret |
| E007 | Invalid protected payload | Sanitized error, no crash |
| E008 | Fake Solari key in text | Redacted before persistence |
| E009 | Authorization header | Not persisted |
| E010 | Cookie header | Not persisted |
| E011 | Set-Cookie header | Not persisted |
| E012 | Sensitive fill/password value | Not echoed in durable timeline/logs |
| E013 | Secret in provider exception | Sanitized before UI/report/log |
| E014 | Generated-output secret scan | Injected fake secret absent from durable artifacts |

## F. Persistence, evidence, reporting, and integrity

| ID | Test | Required result |
| --- | --- | --- |
| F001 | Unique run directory | No collision/overwrite |
| F002 | Existing run ID | Fail rather than overwrite |
| F003 | Atomic manifest update | Reader sees old valid or new valid state, not truncation |
| F004 | Malformed manifest | Damaged run isolated; history remains available |
| F005 | Unsupported schema | Explicit incompatible/damaged state |
| F006 | Artifact hash | SHA256 generated at finalization |
| F007 | Untampered artifact | Integrity validation succeeds |
| F008 | Tampered artifact | `EVIDENCE_INVALID` |
| F009 | Missing finalized artifact | Invalid/missing state, not valid display |
| F010 | Cross-run file substitution | Ownership/integrity rejects |
| F011 | Active unsealed evidence | Distinguishable from finalized evidence |
| F012 | Completed restart/reload | Same authoritative outcome |
| F013 | Interrupted restart | Becomes `INTERRUPTED` without replaying actions |
| F014 | One damaged run | Other history remains usable |
| F015 | report.json parity | Same authoritative run data as UI |
| F016 | HTML parity | Same core outcome/plan/evidence as JSON |
| F017 | Malicious problem text | Escaped/inert |
| F018 | Malicious page title | Escaped/inert |
| F019 | Malicious console text | Escaped/inert |
| F020 | Static report | Human readable with JavaScript disabled |
| F021 | Artifact lookup | Uses run ID + artifact ID, never user path |
| F022 | Evidence sequence | Stable monotonic ordering within run |

## G. API, coordinator, events, and concurrency

| ID | Test | Required result |
| --- | --- | --- |
| G001 | Valid create request | 202/authoritative run ID after durable creation |
| G002 | Invalid shape | 4xx stable code |
| G003 | Invalid target/plan | Rejected before Solari adapter invocation |
| G004 | Second active run | `RUN_ALREADY_ACTIVE` |
| G005 | Racing creates | Exactly one admitted |
| G006 | Get run | Persisted authority returned |
| G007 | List history | Stable truthful ordering |
| G008 | Missing run | `RUN_NOT_FOUND` |
| G009 | Damaged run | Does not crash history endpoint |
| G010 | Cancel active run | Cancellation request propagated |
| G011 | Cancel terminal run | Deterministic no-op/error; no second cleanup |
| G012 | SSE first event | Current authoritative snapshot |
| G013 | SSE transitions | Ordered run sequence |
| G014 | SSE reconnect | GET rehydrates truth before/with new subscription |
| G015 | Valid artifact | Correct owned bytes and MIME |
| G016 | Invalid artifact | Rejected |
| G017 | Replay unavailable | Explicit state, no dead link |
| G018 | Graceful local stop | Stops admission before cleanup/exit |

## H. Built local UI user-boundary tests

The claimed UI workflow must be driven through the built local application, not substituted by direct internal calls.

| ID | Test | Required result |
| --- | --- | --- |
| H001 | First launch without key | Connection surface understandable |
| H002 | Invalid key | Sanitized failure; not READY |
| H003 | Valid configured state | New Investigation reachable |
| H004 | Target required | Client feedback and server rejection agree |
| H005 | Problem required | Client feedback and server rejection agree |
| H006 | Plan completeness | Omitted/empty, action-only, and expectation-only plans rejected; complete plan accepted |
| H007 | Submit investigation | Navigates to authoritative active run |
| H008 | Rapid double submit | One run only |
| H009 | Active progress | Updates without full page refresh |
| H010 | Browser refresh during run | Rehydrates current state |
| H011 | Browser back/forward | Does not corrupt run/history |
| H012 | VERIFIED detail | Two independent evidence authorities visible |
| H013 | NOT_REPRODUCED detail | Clear negative result, not styled as infrastructure failure |
| H014 | INCONCLUSIVE detail | Reason and next action visible |
| H015 | REPRODUCED detail | Visually/semantically distinct from VERIFIED |
| H016 | History order/select | Recent runs discoverable and selectable |
| H017 | History/detail parity | Same lifecycle/outcome |
| H018 | Screenshot viewer | Correct run-owned image |
| H019 | Console view | Sanitized text |
| H020 | Page-error view | Sanitized text tied to correct attempt |
| H021 | Network view | Sanitized method/URL/status/failure |
| H022 | Timeline | Ordered plan/lifecycle/evidence events |
| H023 | Replay ready | Actionable only when authoritative state ready |
| H024 | Replay pending/unavailable | Truthful non-actionable state |
| H025 | Cancel | Request then terminal server-backed state |
| H026 | Error-state navigation | Safe return to history/new run |
| H027 | Damaged historical run | Explicit damage; app remains usable |
| H028 | Narrow viewport | No critical horizontal overflow |
| H029 | 100% desktop scale | Primary text/controls readable |
| H030 | 125/150% browser zoom | Critical workflow remains usable |
| H031 | Keyboard navigation | Main form/history/evidence/cancel reachable |
| H032 | Accessible names/roles | Critical controls exposed |
| H033 | Focus visibility | Keyboard focus visible |
| H034 | Malicious target/user text | Rendered inertly |
| H035 | No dead controls | Every visible interaction works or is truthfully disabled |

## I. Live Solari browser contract

| ID | Test | Required result |
| --- | --- | --- |
| I001 | Client construction | Installed package accepts current credential path |
| I002 | Browser create | Real session ID |
| I003 | New page | Page usable |
| I004 | Navigate public target | Final URL/title observable |
| I005 | Locator interaction | Real click/fill/read works |
| I006 | Screenshot | Nonempty valid image bytes |
| I007 | Console capture | Known fixture console signal observed |
| I008 | Page error | Known fixture uncaught error observed |
| I009 | Network signal | Known fixture failed/status request observed |
| I010 | Recording | Session created with recording enabled |
| I011 | Browser close | Close completes |
| I012 | Client/process shutdown | No leaked SDK handle prevents process exit |
| I013 | Replay readiness | Bounded poll -> ready or truthful unavailable/timeout |
| I014 | Replay URL | Valid URL when supported/documented path succeeds |
| I015 | Replay download capability | Local bytes only if installed TS SDK exposes proven API |
| I016 | Sequential fresh sessions | Distinct IDs |
| I017 | Capacity/429 policy | No blind rapid retry |
| I018 | 502/503/504 policy | Retry eligible only for safe bounded operation |

## J. Live Solari sandbox contract

| ID | Test | Required result |
| --- | --- | --- |
| J001 | Sandbox client | Installed package compiles/connects |
| J002 | Create | Owned sandbox ID |
| J003 | Control connection | Commands/files available |
| J004 | Write fixture | Succeeds |
| J005 | Run command | Transport succeeds and exit code checked |
| J006 | Background server | Remains reachable after start command returns |
| J007 | Preview URL | Current package returns public URL |
| J008 | External probe | Expected fixture identity reachable outside VM |
| J009 | Kill | Explicit teardown completes |
| J010 | Idle behavior | Harness never substitutes pause/timeout for kill |
| J011 | Nonzero command | Recognized as failure |
| J012 | Setup exception | Owned sandbox still receives teardown attempt |

## K. Deterministic fixture truth

These tests validate the fixture independently from ReproDocket classification.

| ID | Scenario | Ground-truth assertion |
| --- | --- | --- |
| K001 | Account blank panel | Save causes panel failure plus defined page-error cue |
| K002 | Account reset | Fresh browser begins clean |
| K003 | Billing refresh | Initial route works; same-browser refresh produces defined 404 |
| K004 | Missing ZIP | Invalid address is accepted and displays defined cue |
| K005 | Healthy profile | Healthy confirmation appears; alleged defect condition absent |
| K006 | Healthy login | Intended validation appears; alleged crash condition absent |
| K007 | Ambiguous control | Plan cannot choose uniquely; authority insufficient |
| K008 | Nonrepeatable scenario | First clean attempt confirms, second clean attempt does not |
| K009 | Fixture version | Public nonsecret fixture version available for provenance |
| K010 | Product independence | No production route/scenario hardcoding |

## L. Full investigation and verification

| ID | Scenario | Required result |
| --- | --- | --- |
| L001 | Account blank first + second | VERIFIED |
| L002 | Billing refresh first + second | VERIFIED |
| L003 | Missing ZIP first + second | VERIFIED |
| L004 | Healthy profile | NOT_REPRODUCED |
| L005 | Healthy login | NOT_REPRODUCED |
| L006 | Ambiguous target/action | INCONCLUSIVE |
| L007 | Nonrepeatable fixture | REPRODUCED |
| L008 | Session identity | Investigation/verification IDs non-null and different |
| L009 | Session ordering | First closed before second created |
| L010 | Evidence attribution | Artifacts owned by correct attempt |
| L011 | Final manifest | Both attempts + classification present |
| L012 | Local UI parity | UI outcome/evidence IDs match manifest |
| L013 | Restart | Completed run still available without new Solari work |
| L014 | Cleanup | All harness-owned browsers/sandbox reconciled |

## M. Failure injection and recovery

| ID | Failure | Required behavior |
| --- | --- | --- |
| M001 | Missing credential | No external resource; actionable message |
| M002 | Invalid credential | Not READY; sanitized error |
| M003 | Browser create failure | No false healthy result |
| M004 | Page create/navigation failure | Partial state preserved; truthful failure/uncertainty |
| M005 | Navigation timeout | No false negative defect conclusion |
| M006 | Target 500 | Target evidence, not infrastructure success |
| M007 | Browser dies mid-run | Partial evidence preserved; cleanup reconciled |
| M008 | Evidence write failure | `EVIDENCE_WRITE_FAILED`; no COMPLETED |
| M009 | Verification unavailable | Never VERIFIED |
| M010 | Replay initially not ready | Bounded polling without changing defect outcome |
| M011 | Replay never ready within budget | Truthful replay state; main run may complete if replay noncritical |
| M012 | Sandbox create failure | E2E fails/blocks before browser fixture use |
| M013 | Fixture command nonzero | Setup fails; teardown still attempted |
| M014 | Preview unreachable | No investigation against nonexistent fixture |
| M015 | Preferred local port occupied | Alternate port; foreign process untouched |
| M016 | Stale PID | Foreign/recycled process untouched |
| M017 | Process termination with active run | Restart yields INTERRUPTED |
| M018 | Graceful shutdown active run | Admission stops and cleanup executes |
| M019 | Cancel during navigation | New actions stop; CANCELLED persisted |
| M020 | Cancel during verification | Never false VERIFIED |
| M021 | Malformed run | Isolated damage |
| M022 | Missing final artifact | Invalid evidence state |
| M023 | Cross-run substitution | Rejected |
| M024 | Foreign-origin POST | No mutation |
| M025 | Duplicate submission race | One admitted, one explicit rejection |

## N. Horizontal connectivity

| ID | Compared surfaces | Invariant |
| --- | --- | --- |
| N001 | Active UI vs manifest | Same lifecycle |
| N002 | History vs detail | Same lifecycle/outcome |
| N003 | Outcome badge vs report JSON | Same final outcome |
| N004 | HTML vs JSON report | Same core data/evidence IDs |
| N005 | Screenshot UI vs manifest | Every shown image owned by run |
| N006 | Console UI vs artifact | Same sanitized ordered entries |
| N007 | Page-error UI vs artifact | Same sanitized ordered entries |
| N008 | Network UI vs artifact | Same sanitized ordered entries |
| N009 | Replay control vs replay record | Action only when ready |
| N010 | Cleanup badge vs resource ledger | PASS only when required resources resolved |
| N011 | Credential UI vs provider authority | READY only after verified source |
| N012 | Validation summary vs Git revision | Exact current revision |
| N013 | README/capability claims vs validation | No premature Available claim |
| N014 | Error UI vs error code | Correct human projection |
| N015 | Cancel UI vs lifecycle | Terminal display backed by persisted state |
| N016 | History after restart | Valid runs do not disappear |

## O. Vertical connectivity

| ID | User outcome | Required path |
| --- | --- | --- |
| O001 | Configure credential | UI -> API -> protected store -> verification -> provider state -> UI |
| O002 | Start run | UI -> validation -> coordinator -> store -> Solari browser -> evidence -> progress UI |
| O003 | Verify defect | first evidence -> first cleanup -> fresh second browser -> second evidence -> classifier -> manifest -> UI |
| O004 | View screenshot | detail -> artifact ID -> ownership -> hash -> bytes -> render |
| O005 | View replay | detail -> replay record -> local/reference authority -> safe action/state |
| O006 | Cancel | UI -> API -> coordinator -> signal -> cleanup -> persistence -> UI |
| O007 | Restart/history | startup -> store scan -> schema/integrity -> history API -> UI |
| O008 | Damaged history | persisted damage -> validator -> damaged projection -> UI without global crash |
| O009 | Full fixture | validator -> sandbox -> preview -> browser A -> browser B -> store -> UI -> cleanup -> report |

## P. Resource and lifetime

| ID | Test | Required result |
| --- | --- | --- |
| P001 | Browser normal close | Requested/completed recorded |
| P002 | Browser close on exception | Still attempted |
| P003 | Verification browser close | Independent close recorded |
| P004 | Sandbox normal kill | Recorded |
| P005 | Sandbox kill on setup failure | Still attempted |
| P006 | Local server stop | Owned server stops |
| P007 | Ownership mismatch | Foreign process untouched |
| P008 | Repeated start | Existing owned instance reused, not duplicated |
| P009 | Foreign preferred port | Foreign process untouched |
| P010 | Open Node handles | Normal close lets process terminate |
| P011 | Temporary cleanup | Only known generated owned files affected |
| P012 | Cleanup failure | Visible and blocks clean lifecycle claim |
| P013 | Repeated local cycles | No listener/resource-record growth |
| P014 | Bounded repeated live cycles | Unique sessions, clean closure each iteration |

## Q. Visual, accessibility, and human QA catalog

Required current-build views for final human review:

| ID | View/state | Required framing |
| --- | --- | --- |
| Q001 | Connection | Full app context |
| Q002 | New Investigation | Full context + readable plan editor/help |
| Q003 | Active Investigation | Full context + close progress area |
| Q004 | VERIFIED | Detail + close outcome/evidence |
| Q005 | REPRODUCED | Full detail |
| Q006 | NOT_REPRODUCED | Full detail |
| Q007 | INCONCLUSIVE | Detail + reason/next action |
| Q008 | History | Several run types |
| Q009 | Screenshot evidence | Image + provenance context |
| Q010 | Console/page errors | Readable close view |
| Q011 | Network | Readable rows/details |
| Q012 | Replay | Ready and nonready states as applicable |
| Q013 | Sanitized error | Representative failure |
| Q014 | Narrow viewport | Full supported minimum width |
| Q015 | 125/150% zoom | Critical workflow readable/usable |

Human review checks visual hierarchy, readable text, no clipping/contradictions, clear lifecycle versus defect outcome, clear investigation versus verification authority, obvious run identity, truthful actionable controls, useful empty/error states, no implementation/debug prose, and professional polish. Escaped defects that automation could have caught require regression coverage.

## R. Windows bootstrap and fresh-checkout validation

| ID | Test | Required result |
| --- | --- | --- |
| R001 | Run from repository root | Correct app root |
| R002 | Run from another working directory | `$PSScriptRoot` resolution works |
| R003 | Windows PowerShell 5.1 bootstrap entry | Can discover/route to PowerShell 7 without parser failure |
| R004 | PowerShell 7 already present | No unnecessary install |
| R005 | Supported Node | Continue |
| R006 | Missing Node | Safe automated install path where package manager available |
| R007 | Unsupported Node | Clear upgrade/block, not false PASS |
| R008 | npm missing/broken | Diagnostic failure |
| R009 | Dependencies missing | Restore |
| R010 | Lockfile present | `npm ci` deterministic restore |
| R011 | Low disk reserve | Full/evidence-heavy run stops before exhaustion |
| R012 | App already running and owned | Reuse/open it |
| R013 | Preferred port foreign-owned | Alternate loopback port |
| R014 | Default browser open fails | Server stays usable and URL printed |
| R015 | stop valid ownership | Only owned process stopped |
| R016 | stop stale ownership | Foreign process refused |
| R017 | mandatory validation stage fails | Nonzero exit |
| R018 | all mandatory automated authorities pass | Zero only then |
| R019 | WMIC scan | No new `wmic`/`wmic.exe` usage |
| R020 | Scratch-path policy | No Desktop/Downloads/Documents default scratch |

## S. Harness sensitivity and deliberate mutation

At least one real mutation/red-green proof is required from every critical class.

| ID | Deliberate defect | Guard that must fail |
| --- | --- | --- |
| S001 | VERIFIED without second evidence | B023/B024/L008 |
| S002 | Reuse first session ID | B024/L008 |
| S003 | Disable scheme guard | C003-C006 |
| S004 | Disable private-address guard | C008-C019 |
| S005 | Remove origin/nonce guard | D005-D007 |
| S006 | Render report unescaped | F017-F019 |
| S007 | Disable redaction | E008-E014 |
| S008 | Serve arbitrary/cross-run artifact | D012-D015/F010 |
| S009 | Skip final hash | F006-F009 |
| S010 | Display another run's screenshot | N005 |
| S011 | Permit two active pipelines | G004/G005 |
| S012 | Optimistically mark cancel terminal | H025/O006 |
| S013 | Swallow cleanup failure | P012/L014 |
| S014 | Skip sandbox kill | J009/J010/L014 |
| S015 | Always classify broken | K005/K006/L004/L005 |
| S016 | Always classify healthy | K001/K003/K004/L001-L003 |
| S017 | Hardcode fixture scenario result | A012/A013/K010 |
| S018 | Accept stale revision evidence | N012/final provenance gate |
| S019 | Treat plan as optional | B006-B009/H006 |

A green suite that stays green under the defect it is supposed to guard is a harness failure.

## T. FULL validation acceptance

FULL is accepted only when all applicable mandatory categories have current results for the exact source revision:

```text
static/repository/contract
plan/lifecycle/classification
security
persistence/integrity/reporting
local API/integration
built local UI user boundary
real Solari browser contract
real Solari sandbox contract
fixture truth
full investigation + verification
failure/recovery
horizontal connectivity
vertical connectivity
resource/lifetime
Windows bootstrap/runtime
visual evidence preparation
public-source/documentation audit
```

Human visual/usability acceptance is separately recorded. Machine FULL PASS is not human QA PASS.

## U. Validation economy

Run the cheapest authoritative checks first:

```text
changed unit tests
-> affected integration tests
-> affected built-UI tests
-> build/lint/typecheck
-> live Solari contract only when its boundary changed
-> full Solari end-to-end at milestone/final gates
```

A cheap deterministic failure stops the pipeline before unnecessary billable resources are created. Cost control never replaces required live proof for milestone or publication claims.
