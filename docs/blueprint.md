# AI Agent Manager — Final Blueprint (merged)

## Stage 1 — Establish the boundary and run one typed operation (size: medium)

### Scope and operating contract

**Labels used throughout**

- **CONFIRMED** — an input you stated.
- **REC** — a recommendation of this blueprint.
- **PROPOSED** — an interface or definition invented here.
- **UNVERIFIED** — "unverified — verify against current docs"; never an unconditional dependency.
- **Depth:** OWN = implement or debug the essential behaviour yourself and defend it without notes. STEER = specify, review and test generated code. AWARE = explain applicability and recognise when it becomes relevant.
- `agm` commands, workflow syntax, schemas and filenames are **illustrative interfaces for the proposed manager**, not an existing product.

**Confirmed**

- Theme: agents reaching over-privileged credentials. Core work is IAM policy evaluation, permission-graph construction and minimal-policy synthesis with a verifier.
- Claude Code is the first adapter; other agents come later, so the core stays vendor-independent.
- Solo builder, AI-assisted, who must understand the core himself.
- Spending cap ₹10,000 per month. Time is not fixed; scope follows availability.
- Carried from the second source document's clarification record: the manager launches one supervised session; installed add-ons are trusted; the threat model covers accidental mistakes and hostile repository or tool content.
- Host: Linux under WSL2 is the supported environment, and Claude Code runs inside WSL2. Native Windows is offline analysis only (DEC-001).
- Implementation language: Python (DEC-002).
- Development for T01–T03 uses existing Claude Code access (DEC-008).

**Unconfirmed (section D ranks the options)**

- How Claude is accessed for the release (direct API key is REC). Whether the Agent SDK can run on a subscription is open question OQ-001, resolved at T04.
- AWS account, inspection permissions and permission to use any discovered credential.

**Release envelope (REC; host and language CONFIRMED)**

- One local manager process, one active workflow, one supervised Claude Code session, a local SQLite database, explicitly selected workspace roots.
- Python on Linux under WSL2, with Claude Code running inside WSL2 (CONFIRMED, DEC-001, DEC-002). Native Windows supports offline analysis only; Claude Code's shell sandbox is unavailable there ([platform support](https://code.claude.com/docs/en/setup), [sandboxing](https://code.claude.com/docs/en/sandboxing)).
- Containers are optional extra isolation, not a prerequisite and not a complete boundary.
- Discovery never implies permission to use a credential.

**Release tiers**

| Tier | Stages | What it can claim |
|---|---|---|
| First useful release | 1–3 | A supervised local task produced sanitised evidence, a manager-policy candidate and an independently checked result. No AWS claims. |
| Capstone-completing | 4–5 | Scoped AWS reachable authority, graph, classification, observed-versus-reachable comparison, verified policy proposal. |
| Production release | 6–7 | Measured behaviour, packaging, compatibility, security and installation evidence. |
| Optional | — | Later-session enforcement (step 9), GitHub add-on, Codex adapter, Access Analyzer policy generation and paid custom checks, interactive-session attachment, implemented multi-agent review. |

### R1 — Capstone steps and acceptance criteria (CONFIRMED by user, DEC-003)

**Why the theme holds.** In the PocketOS incident a Cursor agent running Claude Opus 4.6 deleted a Railway production volume and its backups during a routine staging task ([ACS](https://ia.acs.org.au/article/2026/gone-in-9-seconds--ai-agent-deletes-company-database.html)). Railway's write-up says the agent found an account-scoped API token stored locally ([Railway](https://blog.railway.com/p/your-ai-wants-to-nuke-your-database)). The "violated every principle" line was the agent's own generated confession, relayed by the founder. The incident involved Cursor and Railway, not Claude Code and AWS, so this capstone is an analogue. The lesson transfers: instructions are not authority boundaries, and reachable credential scope is the blast radius.

**Numbered steps**

1. **Discover credential references.** Read the effective Claude Code configuration and enumerate credential sources inside a declared boundary. Store locations and handles only, never values.
2. **Gather authorization evidence.** Resolve references to principals where permitted. Using a separate inspector identity, collect policies and simulate declared request tuples. Record coverage and what could not be inspected.
3. **Build the evidence graph.** Credential reference → principal → action → resource, each edge `allow`, `deny` or `unknown` with provenance.
4. **Classify effects.** Apply a versioned effect pack: destructive, potentially destructive, non-destructive within scope, or unknown, with recovery prerequisites.
5. **Compare with observed work.** Normalise the supervised session's events, add optional provider-side events with attribution confidence, and compute used, unused-high-blast, unexplained and unknown sets.
6. **Propose candidates.** A manager-policy candidate and a separate AWS policy proposal, both deterministic. Neither is applied.
7. **Verify independently.** Check that approved workload tuples are allowed and prohibited tuples are not, using a verifier that shares no logic with the proposer, plus provider oracles.
8. **Decide and report.** Deterministic manager rules produce the CI decision; emit JSON and Markdown with coverage, unknowns and evidence links.
9. **(Optional) Activate later-session enforcement.** Bind an approval to the candidate digest and activate it only on control points the adapter has passed probes for. No AWS mutation.

**Acceptance criteria** (run against a fixture world: fixture home directory, recorded session, recorded provider responses; live variants need authorised AWS access)

- **AC1 Discovery.** Every seeded in-boundary source is reported exactly once; the out-of-boundary decoy is reported as `out_of_boundary`; a canary-secret scan of all artefacts finds nothing.
- **AC2a References (static; passed in Stage 2).** Every reference is a declared principal reference or `unresolved` with reason `resolution_not_granted` (DEC-007).
- **AC2b Identity separation (passed in Stage 4).** The inspector identity is recorded separately. No discovered credential is used without an explicit grant (DEC-007).
- **AC3 Authorization evidence.** Every in-scope tuple has a decision with its inputs and complete pagination. Local-evaluator versus simulator disagreements are listed; a silent mismatch fails.
- **AC4 Graph.** Every material edge cites evidence or is `unknown`. The seeded over-privileged principal shows `s3:DeleteObject` reachable on an enumerated bucket.
- **AC5 Observation.** Normalised events equal the golden set; injected duplicates collapse; injected gaps are flagged; unattributed provider events are not counted as observed.
- **AC6 Proposal verification.** All approved tuples are allowed and all prohibited tuples are non-allow, by both the independent local check and `SimulateCustomPolicy`. `ValidatePolicy` returns no ERROR or SECURITY_WARNING findings.
- **AC7 Determinism.** Same inputs and pack versions give a byte-identical report once volatile fields are excluded.
- **AC8 Honest incompleteness.** Unsupported and uninspectable categories are listed. A gap in evidence the claim requires prevents exit `0`.
- **AC9 Recovery.** Kill and resume completes without re-executing finished steps. An effect recorded immediately before the crash is reconciled, not duplicated.
- **AC10 Conflict.** Approved deletion plus a deny-deletion restriction produces a failure that names both sides.
- **AC11 Claim gate.** A forged model claim or fabricated evidence ID cannot enter authoritative findings or change the exit code.
- **AC12 Contract freeze.** After the Stage 4 freeze, the remaining capstone and both section F rehearsals run with no breaking change and no capstone-specific code in core (core as defined in the Stage 4 contract-freeze note). Any additive, versioned core field is accompanied by a passing re-run of the contract fixtures (DEC-004).

### R2 — Claude Code capability matrix

Documentation-backed capabilities, not conformance results for an installed binary. Pin the agent, SDK and protocol versions and advertise support only after executable probes. Rows drawn from the first document were verified against Claude Code docs at research time; rows marked † come from the second document and were not re-verified in this merge.

| ID | Surface | Observe | Alter or block | Failure behaviour and gaps |
|---|---|---|---|---|
| C1 | Agent loop, sessions, compaction ([Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)) | Messages, tool calls, results | — | Resuming a conversation does not reconcile external effects. Compaction can drop conversation-only instructions. |
| C2 | PreToolUse hook ([hooks](https://code.claude.com/docs/en/hooks)) | Tool name and full input; subagent calls carry agent identifiers | Can replace the whole input; can deny (exit 2 or a JSON deny decision); a hook deny holds even in bypass mode | **Fails open** for command, HTTP and MCP hooks: a crash, a timeout, a mistyped path or a non-2 exit without JSON lets the call proceed. An SDK callback hook that times out blocks instead. A hook `allow` cannot override a settings deny. Version-sensitive; probe it. |
| C3 | Post-tool and lifecycle events (PostToolUse, SessionStart/End, PreCompact, ConfigChange, InstructionsLoaded, FileChanged) | Results, session boundaries, config and instruction loads | Feedback only; PreCompact can block | Cannot undo an effect. Session-end hooks have a short shared budget, so flushes can be cut off. |
| C4 | Permission rules and modes (permissions page) | Resolved rules and their source | Deny, then ask, then allow; first match | Bash rules match command text, so a path-qualified binary or `sh -c` can slip past. Enforced by Claude Code, not by the model. |
| C5 | Bash sandbox ([sandboxing](https://code.claude.com/docs/en/sandboxing)) | — | OS-level filesystem and network limits for Bash and its children | Covers Bash only. Subprocesses inherit environment secrets unless scrubbed. |
| C6 | Settings layers ([settings](https://code.claude.com/docs/en/settings)) | Managed > command-line > local > project > user | — | Per-key exceptions exist; treat them as adapter data. Hooks merge across layers. |
| C7 | MCP ([MCP](https://code.claude.com/docs/en/mcp), [spec](https://modelcontextprotocol.io/specification/latest)) | Local, project and user scopes | Project servers need interactive approval | A non-interactive run shows no approval prompt. Tool annotations are claims, not controls. MCP does not intercept built-in tools. Record configured, discovered, connected, exposed and callable tools separately. † |
| C8 | Skills and plugins ([skills](https://code.claude.com/docs/en/skills), [plugin security](https://code.claude.com/docs/en/plugins/security)) | Bundled skills, agents, hooks, MCP servers | — | No isolation. A skill's `allowed-tools` is pre-approval, not a restrictive allowlist. † |
| C9 | Subagents ([subagents](https://code.claude.com/docs/en/sub-agents)) | Separate context, scoped tools | — | Same OS environment and credentials as the parent. Plugin subagents ignore some frontmatter fields. Do not assume permission inheritance. |
| C10 | Instructions and memory (memory page) | Concatenated instruction files, bounded auto-memory | — | Context, not enforced configuration. |
| C11 | Structured execution ([headless](https://code.claude.com/docs/en/headless), [Python SDK](https://code.claude.com/docs/en/agent-sdk/python)) | SDK messages or `stream-json` output | Allowed tools, permission mode | Non-interactive runs can load repository hooks and servers. `setting_sources=[]` disables user, project and local settings only in SDK versions newer than `0.1.59`; managed settings stay separate; pre-allowed tools may bypass `can_use_tool`. † Test the effective configuration. |
| C12 | Transcripts (sessions page) | JSONL per session | — | Line schema is not officially documented. Retention-limited. Can contain printed secrets. |
| C13 | Telemetry (agent-sdk observability page) | Token and cost metrics, API and tool events | — | Opt-in. Cost figures are approximations. |
| C14 | Codex (later adapter) | — | — | **UNVERIFIED** — check current Codex docs for App Server events and approvals, hook coverage and whether a hook callback error blocks execution. Not a dependency. |

**Consequence.** Hooks give strong observation and best-effort pre-execution blocking. Real authority limits come from the credential the provider enforces and from manager-owned typed tools.

### R3 — Evidence envelope S

One versioned envelope for events, inventory, provider responses, graph edges and decisions.

- **Identity:** schema version; run, session, step and attempt IDs; source ID; source event ID where available.
- **Causality:** parent or tool-call ID; source sequence where available; ingestion sequence.
- **Provenance:** source kind; adapter, tool and configuration versions; transformation version; evidence references.
- **Content:** sanitised typed payload; status; content digest over canonical JSON; redaction and truncation markers.
- **Coverage:** expected boundary; inspected sources; exclusions; missing records; unsupported features; attribution confidence.
- **Claim:** property established; action, resource and context tuple where applicable; limitations; whether the evidence is synthetic, imported or live.

An ingestion sequence is not execution order. A digest detects change against a known digest; it does not prove source authenticity. Status values are `ok`, `partial`, `unknown`, `stub` and `error`; `unknown` is never treated as deny or allow.

### R4 — Execution and decision contracts W

**Step states:** pending → ready → running → succeeded, failed, cancelled, awaiting approval, or outcome unknown. Reconciliation can resolve outcome unknown; it never silently becomes a retry. A timeout does not prove an external effect failed.

| Class | Meaning | Recovery rule |
|---|---|---|
| Pure | Deterministic transformation of immutable inputs | Recompute |
| Read | External inspection | Bounded retries; each response is new evidence |
| Local commit | Manager-controlled state or artefact update | Atomic transaction keyed by input digest |
| External effect | Mutation outside the manager | Retry only with demonstrated idempotency; otherwise reconcile or stop |
| Model proposal | Untrusted generated candidate | Reuse the saved candidate or regenerate explicitly; validate independently |

Each step declares a timeout, cancellation behaviour, output bound and retry budget.

**Approvals** bind to the canonical action and arguments, subject, resource scope, workflow, configuration, policy and pack versions, relevant evidence digests and executor profile. Any change makes the approval stale. An approval cannot override an explicit deny.

**Exit codes**

| Result | Exit |
|---|---:|
| Required properties demonstrated; CI pass | `0` |
| Internal error or unhandled exception (never a decision) | `1` |
| Usage error, invalid configuration or incompatible contract | `2` |
| Demonstrated policy conflict or failed deterministic check; CI fail | `10` |
| Approval pending | `11` |
| Required evidence incomplete or unsupported; review required | `12` |

Decision codes start at 10 because an uncaught Python exception exits `1` and Click, Typer and argparse exit `2` on usage errors. Both source documents put decisions on those numbers, which lets a crash read as a policy verdict.

A rejected model claim never changes the exit code. Exit `10` is only for a demonstrated conflict or a failed deterministic check; a required property that lacks evidence is review, exit `12` (DEC-006).

### R5 — Provider evidence boundary P (AWS)

**Depth scope (tuple-level analysis).** `s3:ListBucket`, `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` on explicitly enumerated general-purpose buckets and objects in one account. API operations map to IAM actions and resource types, and the mapping is not one-to-one ([S3 action mapping](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-policy-actions.html)).

**Supported profile.** Bucket-owner-enforced ownership, no ACL-dependent access, no KMS-dependent workload, explicit bucket and versioning configuration evidence ([Object Ownership](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html), [KMS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)). Subject is either an IAM user, or a role with no bucket policy on the enumerated buckets. A role subject with a bucket policy is `partial`. Unknown configuration is never assumed to match the profile.

**Breadth probe (action-level only).** Simulate the effect pack's critical action names, such as `rds:DeleteDBInstance`, against the subject with resource `*`. An allow is the account-wide-authority signal the PocketOS case is about. A non-allow establishes nothing about resource-specific grants, so report it as "not account-wide; resource-specific reach not evaluated". Confirm this simulator behaviour with a fixture before relying on it.

**Unsupported, reported as unknown.** Versioned deletion, retention, access points, directory buckets, cross-account evaluation, role chaining, privilege-escalation reachability, RCPs, session policies the manager did not create, VPC endpoint policies.

**Three separate facts.** The inspector's authenticated identity; the subject principal and the policy and context inputs supplied; a simulated or observed outcome for one request. `GetCallerIdentity` identifies the caller and does not enumerate authority ([STS](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html)).

| Mechanism | Establishes, under stated conditions | Does not establish |
|---|---|---|
| IAM evaluation logic ([docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-denyallow.html)) | Reference semantics: default deny, explicit deny wins, boundaries and org policies act as limits, principal type changes some resource-policy semantics | — |
| `SimulatePrincipalPolicy` ([API](https://docs.aws.amazon.com/IAM/latest/APIReference/API_SimulatePrincipalPolicy.html), [limits](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)) | Decision for specified requests without executing them | RCPs. Resource policies it was not given. Resource-policy simulation for roles. Live outcome. Older AWS pages disagree on SCPs with conditions, so keep a fixture that detects which behaviour holds. |
| Context-key discovery ([API](https://docs.aws.amazon.com/IAM/latest/APIReference/API_GetContextKeysForPrincipalPolicy.html)) | Keys referenced by the principal's policies | SCP context keys. An empty list is not complete contextual knowledge. |
| `SimulateCustomPolicy` ([API](https://docs.aws.amazon.com/IAM/latest/APIReference/API_SimulateCustomPolicy.html)) | Decision of a candidate document for given tuples | Effect of other policies in the account |
| Access Analyzer `ValidatePolicy` ([API](https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_ValidatePolicy.html)) | Grammar, best-practice and security findings; no additional charge | Effective access |
| Access Analyzer custom checks (optional) | Automated-reasoning PASS or FAIL about the document | Other policies in the account. Charged per check; price UNVERIFIED. |
| Access Analyzer policy generation (optional) ([limits](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-generation.html)) | A template from CloudTrail activity | Action-level detail for S3 data events; `iam:PassRole`. Not a shortcut around the add-on. |
| CloudTrail ([categories](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html), [record format](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html)) | Assumed-role session name and source identity when set | Which local tool call made a request. S3 object activity without a data-event trail. Missing events do not prove no activity. |
| Last-accessed data ([docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_last-accessed.html)) | Services and some actions a principal attempted | Success versus denial; data-plane actions |

### R6 — Generated-code review checklist (referenced by card field 9)

- **G1 Secret leakage** in logs, exceptions, reprs, snapshots, reports. Detect with canary fixtures and an artefact scan.
- **G2 Fail-open defaults:** an error path returns allow, pass or exit 0. Detect with fault injection.
- **G3 Silent truncation:** ignored pagination or truncated output accepted as complete. Detect with multi-page fixtures.
- **G4 Wildcard and normalisation errors:** `*` versus `?`, case rules, `NotAction` and `NotResource` inversion, path escape. Detect with property tests.
- **G5 Non-determinism:** ordering, timestamps or random IDs inside digests. Detect with the AC7 test.
- **G6 Schema drift:** unknown fields dropped silently. Detect with strict mode and an `unknown_fields` record.
- **G7 Retry on non-idempotent effects.** Detect with journal assertions and a fake ledger.
- **G8 Over-broad exception handling.** Detect with lint rules and review.
- **G9 Executing untrusted content:** `credential_process`, shell from config, repository scripts. Detect with a booby-trapped fixture that writes a marker file.
- **G10 Invented APIs or flags.** Detect with botocore stubs against service models and `--help` checks.

### Executable checkpoint

- **What runs:** a minimal installable package with one typed fixture tool, bounded input and output, sanitisation, a deterministic result checker and the report and exit-code contract.
- **Invoke:** `agm rehearse foundation`, `agm rehearse secrets` (owned by T02, DEC-005)
- **Success:** valid input produces the expected result and an R3 envelope. Malformed output, an out-of-root path and a seeded secret are rejected or sanitised. Stub steps yield exit `12`, not `0`. The `secrets` rehearsal (section G) finds no seeded value in any output.
- **Interview evidence:** a trust-boundary diagram, and a demonstration that repository text cannot authorise a tool.
- **Remaining limits:** fixture execution only. No session capture, durable recovery or AWS claims.

### T01

1. **Topic name, depth level, relative size:** T01 — Harness behaviour and extension surfaces; STEER; medium.
2. **Why the manager needs it:** The adapter must tell observing a tool call apart from controlling it, and must know which declared guardrails can silently fail to fire.
3. **What I must understand:** The model and tool loop; parallel tool batches and subagents; context limits and what compaction removes; permissions versus sandboxing; every row of R2, especially C2 fail-open conditions and C6 precedence.
4. **What I can skip or delegate:** Model training, inference infrastructure, rebuilding the agent loop, UI customisation, cloud-hosted sessions.
5. **Native versus build:** Reuse the native harness (R2). Build only probes and the manager boundary around demonstrated capabilities.
6. **Decisions this unlocks:** SDK launch versus CLI subprocess; hooks versus typed wrappers as the control point; structured streams versus transcript parsing.
7. **Build deliverable:** A probe suite covering a successful call, a refusal, compaction, interruption, and hook exit 0, exit 2, crash, timeout, missing script and malformed JSON, with results recorded against a pinned version.
8. **Understanding check:** What continues the loop? What can compaction remove? Why is a post-tool hook too late to prevent an effect? What happens when a command hook exits 1 with plain stderr? **Draw** one model and tool cycle with every control point.
9. **Review checklist for generated code:** Assumed event completeness; permission and sandbox confusion; one-tool-per-turn assumptions; invented hook behaviour (G10). Detect with negative probes.
10. **Capstone contribution:** 1, 5, 9.
11. **Depends on:** None. External: a working Claude Code install inside WSL2 (DEC-001), using existing Claude Code access (DEC-008).

### T02

1. **Topic name, depth level, relative size:** T02 — Manager authority, secrets and threat model; OWN; large.
2. **Why the manager needs it:** Hostile content must not acquire inspector credentials, change policy or approve its own actions. The manager must not become the next over-privileged token.
3. **What I must understand:** Trust boundaries across repository, agent, manager, inspector and provider; minimum authority; prompt injection; path and symlink escape; executable configuration; confused-deputy failures; sanitisation before persistence; fingerprints instead of secret values; which subprocess inherits which environment.
4. **What I can skip or delegate:** Defending against a hostile machine administrator or a malicious trusted add-on; bespoke cryptography; formal certification.
5. **Native versus build:** Use native restrictions (C4, C5, C6) as recommended configuration. Enforce manager-owned operations in code. The agent process never receives AWS credentials; typed tools hold them.
6. **Decisions this unlocks:** Separate inspector process versus shared environment; sanitised workspace copy versus broad repository access; whether the manager mints the workload's provider session.
7. **Build deliverable:** Threat model with data-flow diagram and ranked threats; authority manifest; a read-only inspector IAM policy; secret-safe launch contract; attack fixtures; the section G `agm rehearse secrets` rehearsal (DEC-005). **Independent exercise:** implement a deny-by-default tool wrapper and debug an environment-variable leak into a child process, without a coding assistant.
8. **Understanding check:** Who can authorise credential use? Can a prompt grant filesystem authority? Which subprocess inherits a secret? Name three ways in-agent enforcement can be bypassed. **Draw** the agent, manager, inspector and provider boundaries.
9. **Review checklist for generated code:** G1, G2, G9. Symlink escape; inherited credentials; secret-bearing exceptions; unsafe YAML loading; model-generated approval records.
10. **Capstone contribution:** 1–9.
11. **Depends on:** T01.

### T03

1. **Topic name, depth level, relative size:** T03 — Typed contracts, bounded tools and the CLI contract; STEER; medium.
2. **Why the manager needs it:** Step boundaries must reject malformed or partial data, and CI consumes only the exit code and the report.
3. **What I must understand:** Strict inputs plus semantic validation; discriminated status unions; structured errors (throttle, auth, not found, invalid, timeout, unknown) with a retryable flag; subprocess lifecycle, cancellation and output limits; side-effect classes from R4; record and replay of provider responses; the R4 exit-code table; stdout for the report and stderr for logs.
4. **What I can skip or delegate:** General shell parsing; a universal command-wrapper framework; CLI framework internals.
5. **Native versus build:** Reuse Pydantic, subprocess and the AWS SDK. Strict mode reduces coercion but does not replace domain checks ([strict mode](https://docs.pydantic.dev/latest/concepts/strict_mode/)). Build thin domain wrappers.
6. **Decisions this unlocks:** Direct API wrapper versus subprocess; deterministic calculation versus model-assisted proposal; recorded responses versus mocks (REC: recorded, for determinism and zero provider spend in CI); the product code layout, logged in docs/decisions.md when decided (DEC-009).
7. **Build deliverable:** One file-reading wrapper and one fake provider wrapper with typed success and error envelopes; the report JSON Schema generated from models; one function mapping report status to exit code, with a test that every status maps to exactly one code.
8. **Understanding check:** Can valid JSON be wrong? When is retry unsafe? What happens after output truncation? Why must an internal error never return a decision code? **Explain** the lifecycle of a cancelled child process.
9. **Review checklist for generated code:** G3, G7, G8, G10. Shell interpolation; unchecked exit status; unrestricted endpoints; permissive defaults on optional fields; `sys.exit` scattered across modules.
10. **Capstone contribution:** 1, 2, 5–8.
11. **Depends on:** T02.

## Stage 2 — Capture a supervised session and discover its declared environment (size: large)

### Executable checkpoint

- **What runs:** a supervised Claude task that reads an innocuous fixture through a declared tool profile; an offline replay of the same event shape; bounded discovery over a fixture home directory.
- **Invoke:** `agm rehearse capture --adapter claude`, `agm rehearse capture --adapter replay`, `agm inventory --boundary boundary.yaml`, `agm rehearse coverage`
- **Success:** Both adapters produce valid R3 records. Duplicate events do not duplicate facts. A missing result creates a coverage gap. Inventory distinguishes references from verified identity. A booby-trapped `credential_process` is not executed. A fake adapter lacking a required capability yields `unsupported`, not a crash. AC1, AC2a and the session half of AC5 pass (DEC-007). The `coverage` rehearsal (section G, owned by T06) shows every omitted input as a gap (DEC-005).
- **Interview evidence:** a captured tool-and-result chain; a demonstration of ambiguous attribution; the decision to make discovery static-first because resolving a profile can execute code.
- **Remaining limits:** No AWS authorization analysis. No claim to observe every subprocess or network request. Without agent access only the replay profile passes; the Claude compatibility gate stays unmet.

### T04

1. **Topic name, depth level, relative size:** T04 — Capability-aware adapters; STEER; medium.
2. **Why the manager needs it:** The core must not assume every agent can block, resume or expose the same events (CONFIRMED: other agents come later).
3. **What I must understand:** Required versus optional capabilities; the difference between what the interface promises and what one adapter supports; tested version ranges; negotiation at run start; launch profiles with explicit setting sources (C11); unsupported-operation errors.
4. **What I can skip or delegate:** A second production adapter; interactive-session attachment; dynamic plugin loading.
5. **Native versus build:** Integrate R2 through one Claude adapter built on the Python Agent SDK, plus one deterministic replay adapter. Build the contract and conformance suite.
6. **Decisions this unlocks:** Reject a workflow, degrade explicitly to observation, or choose another adapter. Resolves open question OQ-001: whether the Agent SDK can run on a subscription, and therefore how the Claude adapter authenticates.
7. **Build deliverable:** Adapter protocol and capability model; Claude adapter with a declared tested range; replay adapter; fake adapter; a conformance suite every adapter must pass. Capability entries carry probe evidence.
8. **Understanding check:** Does an interface guarantee implementation support? When is degradation unsafe? What invalidates a capability result? **Draw** core-to-adapter dependencies and the types that cross.
9. **Review checklist for generated code:** Vendor types leaking into core; fabricated support flags; version comparison on strings; silent fallback from blocking to observation.
10. **Capstone contribution:** 1, 5, 9.
11. **Depends on:** T01, T02, T03.

### T05

1. **Topic name, depth level, relative size:** T05 — Events, provenance and durable evidence storage; OWN; large.
2. **Why the manager needs it:** Lost, duplicated or misattributed records turn a partial trace into a false permission claim.
3. **What I must understand:** R3; sources ranked by contract strength (SDK messages and hook payloads, then `stream-json`, then telemetry, then transcripts); event identity from session, tool-call ID and phase; causal links and subagent nesting; conflicting duplicates; gap detection; atomic ingestion and transaction boundaries; relational tables for events, run state, inventory, approvals and edges; SQLite's single-writer model ([transactions](https://sqlite.org/lang_transaction.html)); migrations; retention and deletion; file-change snapshots.
4. **What I can skip or delegate:** A graph database; distributed event streaming; forensic guarantees against a hostile host; a transcript replay UI.
5. **Native versus build:** Reuse structured agent output (C11) and SQLite. Build normalisation, redaction at ingest and explicit evidence semantics.
6. **Decisions this unlocks:** Which records are authoritative and which are projections; when incomplete evidence blocks acceptance; whether raw transcripts are stored at all (REC: redacted events plus a digest of the raw file).
7. **Build deliverable:** Event store, artefact manifest, migration fixture, deletion operation, injected-fault fixtures. **Independent exercise:** hand-write the canonical digest with an order-invariance test, implement transactional deduplication, and repair a missing tool-result association in a session with an interleaved subagent.
8. **Understanding check:** Is arrival order causal order? Can identical output mean duplicate execution? What must commit together? Why redact before storage and not at render? **Draw** an event and its derived graph edge.
9. **Review checklist for generated code:** G1, G5, G6. Deduplication on content hash; conflicting IDs; reordered events; interrupted migration; file snapshots that miss untracked changes or imply author attribution.
10. **Capstone contribution:** 1–8.
11. **Depends on:** T02, T03, T04.

### T06

1. **Topic name, depth level, relative size:** T06 — Bounded discovery and identity references; STEER; medium.
2. **Why the manager needs it:** The PocketOS agent found a token on disk. A discovered location is neither authorisation to use it nor proof of who used a tool.
3. **What I must understand:** Claude Code settings precedence and MCP scopes (C6, C7); the AWS credential provider chain and the gap between the provider that wins and the profiles an agent could select ([Boto3 credentials](https://docs.aws.amazon.com/boto3/latest/guide/credentials.html)); `credential_process` as code execution; role and source-profile chains; boundary files; coverage denominators (found, searched, skipped with reason).
4. **What I can skip or delegate:** Whole-machine secret hunting; OS keychains; arbitrary token-store parsing; automatic credential-process execution.
5. **Native versus build:** Read supported configuration statically. Reuse botocore's config loader for parsing. Build a reference-only inventory. Live resolution is a separate granted operation, never a fallback chain.
6. **Decisions this unlocks:** Which sources are inspected; which identities may be resolved; which gaps matter to the workflow.
7. **Build deliverable:** Inventory for selected environment names, explicit `.env` files, AWS configuration files, MCP env blocks and Claude Code settings layers; a coverage manifest; a fixture home with several sources and one decoy; the section G `agm rehearse coverage` rehearsal (DEC-005).
8. **Understanding check:** Reference versus principal? Configured versus callable tool? Can inspecting a profile execute code? With an environment key pair and a selected profile both set, which wins? **Explain** one precedence collision.
9. **Review checklist for generated code:** G1, G3, G9. Nested roots; symlinks; duplicate definitions; unreadable files; an expired SSO cache treated as "no credential".
10. **Capstone contribution:** 1, 2.
11. **Depends on:** T02, T03, T05.

## Stage 3 — Ship the first useful end-to-end release (size: large)

### Executable checkpoint — first useful release

- **What runs:** bounded discovery → supervised local file task → normalised evidence → exact-tool manager-policy candidate → independent verification → JSON and Markdown report.
- **Invoke:** `agm rehearse first-release`, `agm rehearse recovery` (owned by T07, DEC-005)
- **Success:** An approved read is preserved. A prohibited write is rejected. A forged model claim cannot enter authoritative findings (AC11). A restarted run reaches the same checked result (AC7, AC9). The `recovery` rehearsal (section G) survives a crash around each state transition and artefact commit.
- **Failure rehearsal:** A fake external ledger records an effect immediately before the manager is interrupted. Recovery reconciles it without duplicating the effect. An unreconcilable variant stops for intervention.
- **Interview evidence:** demonstrate the crash gap and defend state recovery versus effect replay.
- **Remaining limits:** Checks the declared local workload only. No AWS coverage, no future-workload sufficiency, no universal interception.

Ship basic installation, CLI help, a licence and a fixture walkthrough here. T18 finishes release hardening.

### T07

1. **Topic name, depth level, relative size:** T07 — Workflow state, recovery and approvals; OWN; large.
2. **Why the manager needs it:** A restarted workflow must not repeat a possibly completed effect or reuse stale permission.
3. **What I must understand:** Typed DAGs; the R4 state machine and execution classes; durable intent written before execution; attempts; idempotency keys from run, step and input digest; reconciliation; cancellation and late results; R4 approval binding; why a database transaction does not make an API call atomic.
4. **What I can skip or delegate:** Distributed scheduling; general compensation engines; dynamic workflows; multi-worker concurrency.
5. **Native versus build:** Reuse native agent sessions (C1). Build a small durable coordinator for a fixed DAG on SQLite. Adopt a workflow library only when its semantics remove demonstrated complexity.
6. **Decisions this unlocks:** Retry, reconcile, resume, cancel or stop for intervention.
7. **Build deliverable:** Durable runner, workflow-definition loader and failure-injection suite; the section G `agm rehearse recovery` rehearsal (DEC-005). **Independent exercise:** implement transition validation and debug the effect-before-commit crash against a separate fake ledger, without a coding assistant.
8. **Understanding check:** When is retry safe? What does resume restore and what does it never redo? What makes an approval stale? What must be deterministic between steps? **Draw** the ambiguous-outcome path.
9. **Review checklist for generated code:** G7. Kill execution around each commit; concurrent launch; late results after cancellation; duplicate approvals; timestamps inside idempotency keys; timeouts mapped to failure.
10. **Capstone contribution:** 1–9.
11. **Depends on:** T03, T05.

### T08

1. **Topic name, depth level, relative size:** T08 — Manager policy and enforceable decisions; OWN; large.
2. **Why the manager needs it:** Preserving observed work and prohibiting destructive work can conflict, and the manager must expose that conflict instead of resolving it quietly.
3. **What I must understand:** Explicit default deny; deny over approval over allow; rule scope; versioned rule and data packs recorded in every decision; unknown handling per class; enforcement-point coverage; the difference between a manager decision (governs the manager's tools and CI) and provider authorisation (governs AWS); hook and proxy limits (C2).
4. **What I can skip or delegate:** A general IAM evaluator inside the policy engine; a universal policy language; approval UIs.
5. **Native versus build:** Use native control points only where probed. Build a small pure evaluator over typed operations. Typed wrappers remain the strongest manager-owned boundary.
6. **Decisions this unlocks:** Allow, deny, request approval, reject unsupported enforcement, or require a workload or policy disposition.
7. **Build deliverable:** Versioned rule schema, evaluator, truth table, conflict report and an explain command. **Independent exercise:** implement precedence with an exhaustive truth-table test and fix a seeded wildcard or stale-approval bypass.
8. **Understanding check:** Can approval override deny? Does a manager allow imply an AWS allow? Can a proxy govern traffic that bypasses it? Why bind approvals to digests and not run IDs? **Explain** a contradictory workload requirement.
9. **Review checklist for generated code:** G2, G4, G5. Missing attributes; malformed rules; first-match semantics introduced by accident; pack mismatches; rules reading unredacted or model-generated fields.
10. **Capstone contribution:** 6–9.
11. **Depends on:** T02, T06, T07.

### T09

1. **Topic name, depth level, relative size:** T09 — Independent verification and claim gating; STEER; medium.
2. **Why the manager needs it:** A model's statement or a producer's own success flag cannot establish correctness.
3. **What I must understand:** Unit and integration tests; static checks and strict typing on core; independent oracles; property-based tests and shrinking; differential checks; evidence completeness; the typed claim gate (prose cannot become a finding); what a finite passing test set proves.
4. **What I can skip or delegate:** Formal proof of unrestricted policies; mutation-testing frameworks; model grading of deterministic properties.
5. **Native versus build:** Reuse pytest, Hypothesis, a linter and a type checker. Build claim-to-evidence checks. Agent self-review is supplementary only.
6. **Decisions this unlocks:** Accept a demonstrated property, mark it incomplete, or reject it.
7. **Build deliverable:** A separate validator with hand-authored expected cases, generated boundary cases and deliberate mutations that break policy behaviour.
8. **Understanding check:** What makes validation independent? What does passing a finite test set prove? How can valid evidence support the wrong claim? **Explain** one useful property test.
9. **Review checklist for generated code:** Producer and oracle sharing code; tautological assertions; skipped tests; fabricated evidence IDs; broadened claims; generators that never produce the hard cases.
10. **Capstone contribution:** 2, 6–8.
11. **Depends on:** T05, T07, T08.

## Stage 4 — Establish scoped AWS reachable authority (size: large)

### Executable checkpoint

- **What runs:** capstone steps 1–4 using frozen core contracts, the AWS add-on, the effect pack and the workflow definition.
- **Invoke:** `agm rehearse blast-radius --only 1-4 --source fixtures`; with authorised access, `--source aws --scope lab-scope.json`; `agm rehearse aws-unknown` (owned by T11, DEC-005)
- **Success:** AC2b, AC3 and AC4 pass (DEC-007). The local evaluator and the simulator agree on the fixed case matrix or every disagreement is listed. Missing policy or context evidence produces exit `12`. A role subject with a bucket policy is marked `partial`. The breadth probe reports the seeded account-wide destructive action. The `aws-unknown` rehearsal (section G) gives `12` for the inaccessible-policy and role-with-bucket-policy cases.
- **Freeze test (end of Stage 4):** F1, the guardrail-conformance audit, is built and run on the frozen core as the first freeze test. Its four fixture repositories produce the golden decisions with no core change (DEC-005).
- **Interview evidence:** one real evaluator-versus-simulator disagreement and how it surfaces as unknown; the role and resource-policy limitation and why it becomes unknown, not safe.
- **Remaining limits:** R5 exclusions. Single account. Live support is not declared tested until the live gate passes.
- **Contract freeze (DEC-004):** after T10, core is frozen. **Core** means: the evidence envelope (R3), the step runner and state machine (R4), the adapter protocol, the add-on and pack loader, the manager policy engine, the report schema and the exit codes. The freeze forbids breaking changes and capstone-specific code in core. Additive, versioned fields are allowed and require re-running the contract fixtures. The T14 verifier lives in the AWS add-on. The T17 spend guard and evaluation ledger live in a separate evaluation module outside core. A necessary breaking change reopens compatibility tests.

### T10

1. **Topic name, depth level, relative size:** T10 — Add-on contracts and pack lifecycle; STEER; medium.
2. **Why the manager needs it:** Provider logic and classification changes must not create hidden workflow engines or bypass core controls.
3. **What I must understand:** Contract versions and compatibility negotiation; manifests; schema validation at load; data packs (safe to load) versus code add-ons (arbitrary code with the manager's authority); digest pinning in a lockfile; who owns retries, approvals and state (the core, always).
4. **What I can skip or delegate:** A marketplace; remote code loading; malicious-add-on containment (CONFIRMED: installed add-ons are trusted).
5. **Native versus build:** Claude Code plugins package agent behaviour (C8). Manager add-ons are separate: typed operations and packs loaded through Python entry points. Build the loader and compatibility checker.
6. **Decisions this unlocks:** In-process trusted library versus restricted subprocess; declarative pack versus executable add-on.
7. **Build deliverable:** Add-on manifest, compatibility checker, example tool, rule and data packs, frozen contract fixtures, tamper and incompatible-version tests.
8. **Understanding check:** Is an interface a sandbox? Who owns retries? Can a data pack execute code? What does digest pinning prove? **Draw** an add-on call through policy, runner and evidence storage.
9. **Review checklist for generated code:** G9. Imports with undeclared side effects; unsafe deserialisation; hidden credential access; direct state writes; add-on-owned approval paths.
10. **Capstone contribution:** 2–9.
11. **Depends on:** T04, T05, T07, T08, T09.

### T11

1. **Topic name, depth level, relative size:** T11 — IAM evaluation subset and scoped AWS authorization evidence; OWN; large.
2. **Why the manager needs it:** "Principal P can perform A on R" must follow AWS semantics or the report is wrong in the dangerous direction. Offline imported-evidence analysis is impossible without a local evaluator.
3. **What I must understand:** R5. The documented evaluation order; statement matching with `Action`, `NotAction`, `Resource`, `NotResource` and wildcards; condition operators including multi-valued and `IfExists` forms and missing keys; identity, boundary, bucket and org policy layers; user, role and session distinctions; pagination; API failure versus subject denial; inspector denial versus subject denial; simulator versus observed evidence. **Declared subset:** identity policies, permissions boundary, same-account bucket policy, explicit deny, and string, ARN, boolean and null operators. Everything else is `unknown`.
4. **What I can skip or delegate:** Service-specific condition keys beyond the subset; cross-account evaluation; RCP evaluation; other AWS services at tuple depth; GitHub authorization. Collection plumbing and fixtures can be generated.
5. **Native versus build:** Reuse the AWS SDK and the provider's simulation and validation APIs as oracles (R5). Build the subset evaluator, scope validation, action mappings and evidence normalisation. Third-party IAM evaluators may serve as a development-time differential oracle only; check licence and maintenance before use (UNVERIFIED).
6. **Decisions this unlocks:** Which tuples can be evaluated and simulated; what stays unknown; whether a provider-policy proposal is justified at all.
7. **Build deliverable:** AWS add-on covering R5; an evaluator returning a decision with reasons and matched statements; a differential case matrix; a permission manifest for the inspector; recorded-response fixtures; authorised live contract tests; the section G `agm rehearse aws-unknown` rehearsal (DEC-005). **Independent exercise:** hand-write statement matching, debug a seeded `NotAction` plus `Deny` case that produces an unexpected allow, and write the property "adding an explicit deny never enlarges the allowed set".
8. **Understanding check:** Identity policy allows `s3:*`, boundary allows `s3:Get*`: result for `s3:PutObject`? Does inspector denial imply subject denial? Why do role resource policies create a gap? Can simulation prove future success? **Draw** the evaluation flow for a same-account request with a boundary and a bucket policy.
9. **Review checklist for generated code:** G3, G4, G10. Wrong ARN types (bucket versus object); a `Deny` with an unmet condition treated as deny; ignored group policies; discarded session restrictions; errors normalised to deny; policy documents not decoded as the API returns them.
10. **Capstone contribution:** 2, 3, 7.
11. **Depends on:** T06, T09, T10. Live acceptance additionally needs explicitly authorised AWS access, currently unconfirmed.

### T12

1. **Topic name, depth level, relative size:** T12 — Evidence graph and conditional effect classification; STEER; medium.
2. **Why the manager needs it:** The graph must keep potential effects, permission facts and available recovery separate.
3. **What I must understand:** Typed edges with decision, source and provenance digest; recursive queries for reachability; ambiguous attribution; context-sensitive classification (an overwrite or delete is recoverable only under verified versioning or backup conditions); pack coverage and an explicit `unclassified` class treated as high risk by default.
4. **What I can skip or delegate:** A graph database; an exhaustive service taxonomy; guaranteed automated recovery.
5. **Native versus build:** Relational joins plus a graph export. Build a small versioned effect pack over R5 and a critical-action list for the breadth probe.
6. **Decisions this unlocks:** Destructive, potentially destructive, non-destructive within scope or unknown; verified versus unestablished recovery conditions.
7. **Build deliverable:** Credential-reference → principal → action → resource → potential-effect graph with evidence on every material edge; pack schema, lint and changelog; a `who-can` query.
8. **Understanding check:** Can a destructive classification establish access? Is overwrite always recoverable? What invalidates recovery evidence? **Draw** an uncertain principal edge.
9. **Review checklist for generated code:** G4. Name-only classification; missing object state; unsupported versioned deletion treated as recoverable; stale configuration; graph paths that silently drop unknowns.
10. **Capstone contribution:** 3, 4.
11. **Depends on:** T05, T08, T10, T11.

## Stage 5 — Compare observed work, propose and verify (size: large)

### Executable checkpoint — capstone complete

- **What runs:** capstone steps 5–8 on top of Stage 4, giving the full 1–8 path. The supervised session performs S3 work through manager-owned typed tools under a manager-minted subject session.
- **Invoke:** `agm rehearse blast-radius --source fixtures`; with authorised resources, `--source aws --scope lab-scope.json`
- **Success:** AC5, AC6, AC8 and AC10 pass. The positive case gives a checked bounded report and exit `0`. Missing required evidence gives exit `12`. An approved deletion against a deny-deletion restriction gives exit `10`. A raw provider call made outside the typed tools appears as a coverage gap.
- **Interview evidence:** the proposal denying deletion while allowing every approved read; the sentence "verified under the simulator's and validator's stated conditions, not proven least privilege", defended.
- **Remaining limits:** No provider-policy application. No effective-authorisation guarantee. The proposal covers approved observed and declared work only.

### T13

1. **Topic name, depth level, relative size:** T13 — Provider-side observation and attribution; STEER; medium.
2. **Why the manager needs it:** Session events show what the agent attempted through manager tools; provider logs show what AWS saw. The comparison needs both, with honest attribution.
3. **What I must understand:** Typed-tool calls as the primary observation; assumed-role session names and source identity in CloudTrail records; management versus data events; delivery delay; why last-accessed data includes denied attempts; attribution levels `exact` (manager-set session tag), `probable` and `unattributed`; why source IP or user agent alone identifies nothing.
4. **What I can skip or delegate:** CloudTrail Lake; organisation-wide trails; parsing arbitrary shell commands for provider calls.
5. **Native versus build:** AWS provides the logs. Build the attribution join and an optional CloudTrail importer. Policy generation from Access Analyzer stays optional corroboration (R5).
6. **Decisions this unlocks:** Whether the manager must mint the workload session to get exact attribution (REC: yes); whether a data-event trail is worth its cost for the lab.
7. **Build deliverable:** Observation module, attribution confidence model and a fixture with two overlapping sessions on one role.
8. **Understanding check:** What does source identity survive that a session name does not? Why does a last-accessed entry not prove success? When is an event `unattributed`? **Draw** the join between session events and provider events.
9. **Review checklist for generated code:** G3. Time-zone errors in window joins; denied events counted as successful work; region blindness; an absent trail treated as "no activity".
10. **Capstone contribution:** 5.
11. **Depends on:** T05, T06, T11.

### T14

1. **Topic name, depth level, relative size:** T14 — Policy synthesis and independent verification; OWN; large.
2. **Why the manager needs it:** This is the core claim — a smaller credential would have sufficed — and the place where overclaiming is most tempting.
3. **What I must understand:** Synthesis from approved observed tuples plus declared allowances, resource-scoped where the ARN is known and flagged where it is not; never including prohibited or pack-critical actions unless declared; two-sided verification (must-allow and must-not-allow matrices); verifier independence from the synthesiser; what each oracle in R5 establishes; why a manager approval condition has no IAM equivalent.
4. **What I can skip or delegate:** Applying policies; condition-key synthesis beyond resource scoping; SCP generation; model-generated candidates.
5. **Native versus build:** Build deterministic synthesis. Integrate `SimulateCustomPolicy` and `ValidatePolicy`. Paid custom checks and policy generation are optional corroboration. The verifier lives in the AWS add-on, not in core, and still shares no logic with the synthesiser (DEC-004).
6. **Decisions this unlocks:** How to treat unexercised legitimate paths: a declared-allowance file (REC), widening to service-level read, or accepting denials. Each changes what the report may claim.
7. **Build deliverable:** Synthesiser, separate verifier, and a report section listing oracle, conditions, result and what it does not establish. **Independent exercise:** hand-write the minimal policy for the fixture session, run it through custom-policy simulation and validation, and explain every discrepancy against the synthesiser's output.
8. **Understanding check:** What does a full pass prove and not prove? Why must the verifier not reuse the synthesiser's matching code? What does "allows all observed actions" fail to guarantee? **Explain** the exact sentence the report may print after a full pass.
9. **Review checklist for generated code:** Unflagged `Resource: "*"`; verifier calling the synthesiser's evaluator; an implicit deny from a malformed simulation input read as success; validation finding severities ignored.
10. **Capstone contribution:** 6, 7.
11. **Depends on:** T09, T11, T12, T13.

## Stage 6 — Use native context and tools, then measure the result (size: medium)

### Executable checkpoint

- **What runs:** the blast-radius workflow with minimal evidence context, then variants with stale instructions, irrelevant tool schemas and a fabricated model claim; a frozen evaluation suite.
- **Invoke:** `agm rehearse context-and-evaluation`, `agm eval run --suite evals/blast-radius`
- **Success:** Deterministic findings do not change across variants. Unsupported claims are rejected. Missing usage stays unknown. A budget guard refuses a live run whose projected spend exceeds the configured cap. Extra context or agents are kept only where measured benefit justifies them.
- **Freeze test (end of Stage 6):** F2, the bounded source-code migration, is built and run on the frozen core. A correct patch passes, a wrong-but-compiling patch fails behavioural tests and an out-of-scope edit is rejected, with no core change (DEC-005).
- **Interview evidence:** a paired comparison with raw outcomes, failures, cost provenance and limits; a claim scoped to its suite.
- **Remaining limits:** Small synthetic suite. No universal optimisation claim. Implemented multi-agent coordination stays optional.

### T15

1. **Topic name, depth level, relative size:** T15 — Context, memory and tool exposure; STEER; medium.
2. **Why the manager needs it:** Stale memory and excess tool context distort proposals and expose unnecessary capability; instruction files are not guardrails.
3. **What I must understand:** Instruction precedence and concatenation (C10); provenance of loaded instructions; compaction loss; context budgets; tool discovery versus authorisation; MCP schema overhead and trust (C7); configuration lifecycle; the split between workload tools exposed to the agent and manager-control operations that are never exposed.
4. **What I can skip or delegate:** Vector memory; semantic compression; automatic permission-changing tool selection; an MCP server that exposes manager authority.
5. **Native versus build:** Reuse C1, C7, C8 and C10. Add evidence selection and explicit tool profiles only where native behaviour leaves a demonstrated gap.
6. **Decisions this unlocks:** Attach evidence, retrieve it on demand, exclude stale context, or reduce exposed tools.
7. **Build deliverable:** Versioned context manifest, one task-specific tool profile, a loaded-instructions provenance block in the report (an additive, versioned report-schema field; re-run the contract fixtures, DEC-004), and stale-memory and compaction fixtures.
8. **Understanding check:** Does more context always help? Can compaction remove a restriction? Is tool discovery authorisation? Why are MCP tool annotations untrusted? **Explain** a conflicting-memory example.
9. **Review checklist for generated code:** G1. Secrets entering prompts; stale claims treated as facts; auto-enabled servers; profile drift; memory overriding code-enforced authority.
10. **Capstone contribution:** 1, 5, 8.
11. **Depends on:** T04, T06, T09.

### T16

1. **Topic name, depth level, relative size:** T16 — Multi-agent applicability and authority; AWARE; small.
2. **Why the manager needs it:** Extra agents add coordination failures and cost to a mostly deterministic capstone, and someone will propose a planner-and-critic design.
3. **What I must understand:** Independent review, partitioned work and fan-out patterns; context isolation is not authority isolation because subagents share the parent's environment and credentials (C9); shared-state races; why a second agent can share the first one's blind spot.
4. **What I can skip or delegate:** Swarms, recursive delegation, consensus systems, any production multi-agent execution.
5. **Native versus build:** Reuse C9 if a measured use case appears. One agent is the default; build nothing.
6. **Decisions this unlocks:** Whether an isolated read-only reviewer adds value beyond deterministic validation. Current answer: no.
7. **Build deliverable:** A short applicability decision comparing one agent with an optional read-only reviewer.
8. **Understanding check:** When is parallelism useful? Does a subagent get different AWS credentials? Who may approve effects? **Draw** a reviewer with no execution authority.
9. **Review checklist for generated code:** If it became relevant, verify workspace isolation, explicit tool scope, correlation IDs, cancellation, per-agent credential scoping and rejection of agent-issued approvals.
10. **Capstone contribution:** General only.
11. **Depends on:** T07, T09, T15.

### T17

1. **Topic name, depth level, relative size:** T17 — Telemetry, cost and representative evaluation; STEER; medium.
2. **Why the manager needs it:** The ₹10,000 cap is CONFIRMED, and reliability claims need measured outcomes with missing data kept explicit.
3. **What I must understand:** Correlation by session and prompt identifiers; inclusive versus exclusive usage across subagents; cached tokens; measured versus estimated versus unknown; why token counts are not an invoice; workloads with ground-truth labels; recall and precision on seeded authority; false-unknown rate; variance across repeated agent runs; paired baselines; regression thresholds.
4. **What I can skip or delegate:** A hosted observability platform; dashboards; public leaderboard claims; model-as-judge scoring.
5. **Native versus build:** Reuse native usage events (C13). Build a local evaluation ledger and a spend guard in a separate evaluation module outside core (DEC-004).
6. **Decisions this unlocks:** Whether to keep a context profile, an add-on, a model-assisted step or a reviewer; record-once-replay-often versus live runs (REC: replay gates CI; live variance is reported, not gated).
7. **Build deliverable:** Frozen workload suite with labels, raw outcomes, repeated model trials, deterministic baselines, a stored regression baseline and labelled cost figures.
8. **Understanding check:** Can token counts equal an invoice? How do caches distort a comparison? Why keep failures in results? What does five out of five support? **Explain** one paired experiment.
9. **Review checklist for generated code:** Double-counted parent and child usage; missing values becoming zero; price-version mismatch; labels derived from the system under test; errored runs excluded from metrics.
10. **Capstone contribution:** 8.
11. **Depends on:** T05, T09, T15, T16.

## Stage 7 — Package and prove the scoped open-source release (size: medium)

### Executable checkpoint

- **What runs:** clean installation, the first-release walkthrough, all mandatory blast-radius fixtures, authorised live compatibility checks for any advertised live support, and a re-run of both section F rehearsals (F1 built in Stage 4, F2 in Stage 6; DEC-005).
- **Invoke:** `agm rehearse release --profile declared-support.json`
- **Success:** The declared support matrix passes. Every section G check has evidence. AC12 holds. The installed artefact reproduces the golden report. A second developer can reproduce the supported demonstration.
- **Interview evidence:** the release artefact, the compatibility policy, failure recordings and the completed OWN exercises.
- **Remaining limits:** Every R5 exclusion stays visible. Optional enforcement and additional providers are not implied by release naming.

A release without live credentials can honestly support offline imported-evidence analysis. It must not advertise untested live collection or present synthetic fixtures as real AWS behaviour.

### T18

1. **Topic name, depth level, relative size:** T18 — CLI, compatibility and open-source delivery; STEER; medium.
2. **Why the manager needs it:** A correct local prototype is unusable if installation, configuration or CI decisions are ambiguous.
3. **What I must understand:** CLI and exit-code stability (R4); clean installation; dependency locks; separate versioning for CLI, report schema, contracts and packs; a supported-versions matrix including the tested Claude Code range; CI split between replay on every push and manual live jobs using short-lived federated credentials; migration compatibility; licence obligations.
4. **What I can skip or delegate:** A dashboard, hosted service, marketplace, OS packages, broad operating-system support.
5. **Native versus build:** Use standard Python packaging and CI ([packaging guide](https://packaging.python.org/en/latest/tutorials/packaging-projects/)). Build manager-specific diagnostics and support declarations.
6. **Decisions this unlocks:** Supported installation profile, compatibility policy, release contents and licence.
7. **Build deliverable:** Installable package, lockfile, CI with a docs-drift job against the pinned agent version, licence and notices, security-reporting instructions, configuration reference, limits page, migrations and reproducible demonstrations.
8. **Understanding check:** How does CI tell review from pass? What breaks compatibility? What version bump does a new report field need? Does public source alone grant reuse rights? **Explain** an upgrade and rollback.
9. **Review checklist for generated code:** G1. Undeclared dependencies; bundled secrets; unpinned transitive dependencies; misleading exit codes; incompatible pack loading; over-broad CI permissions.
10. **Capstone contribution:** 8, 9.
11. **Depends on:** T09, T10, T14, T17.

## A. Dependency map

- T01 → T02, T04
- T02 → T03, T04, T05, T06, T08
- T03 → T04, T05, T06, T07
- T04 → T05, T10, T15
- T05 → T06, T07, T09, T10, T12, T13, T17
- T06 → T08, T11, T13, T15
- T07 → T08, T09, T10, T16
- T08 → T09, T10, T12
- T09 → T10, T11, T14, T15, T16, T17, T18
- T10 → T11, T12, T18
- T11 → T12, T13, T14
- T12 → T14
- T13 → T14
- T14 → T18
- T15 → T16, T17
- T16 → T17
- T17 → T18

**Path to first useful release:** T01 → T02 → T03 → T04 → T05 → T06 → T07 → T08 → T09.

**Remaining capstone path:** T10 → T11 → T12 → T13 → T14, then T15 → T16 → T17 → T18 for measured behaviour and release.

**Optional branches**

- T08 + T10 + T14 + T18 → later-session enforcement (step 9).
- T04 + T09 + T18 → Codex adapter.
- T06 + T10 + T11 + T18 → separately scoped GitHub authorization add-on.
- T11 + T13 → Access Analyzer policy generation and paid custom checks as corroboration.
- T07 + T15 + T17 → implemented multi-agent comparison.
- Interactive attachment needs a new capture and control profile and is outside the confirmed first release.

## B. The OWN list on one page

| Topic | Why ownership matters | Sentence to defend in an interview | Independent exercise |
|---|---|---|---|
| T02 Authority | A model or repository must not grant itself privileges | "Authority is checked in manager code, and credentials cross only explicitly approved boundaries." | Deny-by-default wrapper; debug an inherited-secret leak |
| T05 Evidence | Wrong provenance invalidates every downstream conclusion | "Every material finding identifies its evidence, its transformations and its unresolved gaps." | Canonical digest; transactional deduplication; repair a causal link |
| T07 Recovery | Crashes can duplicate external effects | "Recovering workflow state does not authorise replaying an ambiguous external operation." | Effect-before-commit crash against a fake ledger |
| T08 Policy | Conflicting requirements cannot both be declared satisfied | "A manager decision governs my tools and CI; IAM governs AWS; a conflict stays explicit until someone records a disposition." | Precedence truth table; fix a wildcard or stale-approval bypass |
| T11 IAM evaluation | A wrong evaluator is wrong in the dangerous direction | "I evaluate a declared IAM subset locally, cross-check it against the simulator, and everything outside the subset is unknown." | Statement matching; `NotAction` bug hunt; deny-monotonicity property |
| T14 Synthesis and verification | The core claim of the project | "The proposal is verified to allow approved work and deny prohibited work under stated oracle conditions, which is not a least-privilege proof." | Hand-written minimal policy checked against both oracles |

Six of eighteen. T11 and T14 are owned because you named IAM evaluation and verified synthesis as the core work. The rest of the AWS add-on (collection, mapping, CloudTrail) stays STEER.

## C. Cut list

| Avoid or defer | Reason here | Broader applicability |
|---|---|---|
| Training, fine-tuning, model architecture | Does not unlock any acceptance criterion | Useful for model-development problems |
| Rebuilding the agent harness | Duplicates the native loop, tools and compaction | Useful when no harness meets a demonstrated need |
| RAG, vector stores, memory platforms | Evidence is structured; bounded retrieval is enough | Useful for large knowledge workloads |
| Agent swarms, planners and critics | Same credentials, more cost and variance, no added guarantee | Useful for separable, independently verifiable work |
| Model-as-judge for authorisation or verification | A non-deterministic judgement cannot be an authority check | Not useful for authorisation generally |
| Instruction files as guardrails | They are context, not enforcement | Not useful as enforcement generally |
| Universal shell-command interpretation | Arbitrary commands are an unreliable authorisation interface | Narrow parsers help specific controlled tools |
| Full IAM semantics, all services, cross-account, escalation paths | Turns the add-on into a separate authorization product | Valuable specialist work |
| Automatic least-privilege generation and policy application | Observed work is incomplete; mutation needs its own authorisation and recovery design | Bounded proposals remain useful |
| Global network interception, syscall tracing | Large, invasive, platform-specific | Useful for dedicated security products |
| Graph database | A graph-shaped report does not need graph storage | Revisit on demonstrated traversal or scale needs |
| Distributed scheduler, message bus, external workflow platform | One active local workflow does not justify it | Revisit for multi-worker requirements |
| Web dashboard and hosted tenancy | Adds authentication, deployment and data isolation | Useful once the CLI proves value |
| GitHub token analysis | A second authorization model before the first is stable, with weaker evidence (probing only) | A later independent add-on |
| Multi-cloud | Doubles provider work | The add-on contract keeps the door open |

## D. Open architectural decisions

### D.1 Confirmed requirements (not reopened)

Blast-radius theme with IAM evaluation, graph and verified synthesis as core work; Claude Code first with a vendor-independent core; solo, AI-assisted, core understood personally; ₹10,000 monthly cap; no fixed time; one supervised session; trusted installed add-ons; threat model of accidental mistakes and hostile repository or tool content. Added by resolution: Linux under WSL2 as the supported host with native Windows offline only (DEC-001); Python (DEC-002); capstone steps 1–9 and AC1–AC12 as amended (DEC-003, DEC-006, DEC-007); existing Claude Code access for T01–T03 development (DEC-008).

### D.2 Unconfirmed inputs, ranked easiest first

| Area | Option | What it enables | Recommendation |
|---|---|---|---|
| Runtime | 1. Existing Windows host, offline mode | Fixtures, schemas, storage, policy, reports | **CONFIRMED** as offline analysis only (DEC-001); no shell-sandbox demonstration |
| Runtime | 2. Linux under WSL2 supervised runner, Claude Code inside WSL2 | First live-agent profile | **CONFIRMED** supported environment (DEC-001) |
| Runtime | 3. Restricted container or VM | Extra process and filesystem separation | Add only if hostile-code execution requires it |
| Agent access | 1. Replay adapter | All deterministic development and failure tests | **Start here**; no inference spend |
| Agent access | 2. Existing Claude Code access | Personal launch and capture experiments | **CONFIRMED for T01–T03 development** (DEC-008); do not assume it is redistributable through a product |
| Agent access | 3. Direct API key | Programmatic integration and usage accounting | **REC for the release**; needs a spend ceiling inside the cap ([SDK authentication](https://code.claude.com/docs/en/agent-sdk/overview)) |

Rows marked CONFIRMED are resolved; see docs/decisions.md.

**Open question OQ-001 (resolve at T04).** Can the Agent SDK run on an existing Claude subscription, or does it require a direct API key? This decides how the Claude adapter authenticates and how inference spend counts against the cap. Not a decision yet.
| AWS | 1. Sanitised fixture and import bundle | Offline analysis and deterministic tests | **Start here** |
| AWS | 2. Designated read and simulation inspector | Live bounded policy evidence | **REC live extension** |
| AWS | 3. Disposable lab workload resources | Correlation and observed-call evidence | Useful; needs cleanup and spend controls |
| Credential use | 1. References only | Discovery without activating anything | **Default** |
| Credential use | 2. Explicit inspector identity | Approved inspection and simulation | **REC** |
| Credential use | 3. Per-source transient use of a discovered credential | Confirmed identity for one source | Optional, explicit grant only |
| Credential use | Excluded: automatic activation of discovered credentials | — | Conflicts with the authority boundary |

### D.3 Conflicts between the two source blueprints and the verdict

| Conflict | First document | Second document | Verdict |
|---|---|---|---|
| First useful release | Includes AWS | Local only | **Local only.** It exercises the full path with no AWS access or spend. |
| Workflow engine | DBOS | Hand-built coordinator | **Hand-built.** T07 is OWN, the DAG is fixed, and the DBOS resume-from-CLI behaviour was unproven. |
| AWS depth | Subset evaluator over all actions | Four S3 actions, simulator only | **Subset evaluator, four S3 actions at tuple depth, plus an action-level breadth probe.** Keeps your stated core without unbounded scope. |
| AWS ownership | OWN | STEER | **OWN the evaluator and the synthesis verifier; STEER the rest.** |
| OWN count | Fourteen | Four | **Six.** Fourteen is not selective; four drops the confirmed core. |
| Claude integration | Headless CLI plus hooks | Agent SDK | **Agent SDK.** Explicit setting sources matter against hostile repositories, and SDK callback timeouts block where command hooks fail open. |
| Observing provider calls | Bash `aws` calls plus CloudTrail | Typed operation mapping | **Manager-owned typed tools**, CloudTrail as optional corroboration. |
| Exit codes | 0–4, 64, 70 | 0, 2, 3, 4 | **0, 1, 2, 10, 11, 12** (R4). Both originals collide with interpreter and CLI-framework codes. |
| Further capstones | GitHub tokens; guardrail audit | Data reconciliation; code migration | **Guardrail audit and code migration.** One same-domain, one different-domain and model-assisted. |
| Licence | Apache-2.0 | MIT provisional | **Apache-2.0** for the patent grant. |

### D.4 Remaining open choices

| Choice | Recommended | Main alternative | Trade-off | Evidence that changes it |
|---|---|---|---|---|
| Storage | SQLite plus sanitised artefacts | PostgreSQL | Zero operations versus concurrency | Multiple writers or remote workers become real |
| Workflow implementation | Small fixed-DAG coordinator | Durable-workflow library | Small surface versus proven semantics | Asynchronous long-running provider jobs or custom failure handling keep growing |
| Manager policy | Small typed declarative rules | Established policy engine | Easy audit versus expressiveness | Rule composition becomes hard to specify or test |
| Runtime enforcement | Advisory, opt-in (step 9) | Enforcement as a headline guarantee | Honest versus attractive | A documented fail-closed mode for every relevant control point |
| AWS subject | IAM user, or role without bucket policy; other roles partial | Broader role and session evaluation | Honest bounds versus coverage | Adequate policy-layer evidence and passing contract tests |
| Workload session | Manager mints it and holds the credentials in typed tools | Agent uses ambient credentials | Exact attribution and containment versus zero workflow change | Users refuse the launcher; then `probable` is the ceiling |
| Provider candidate | Deterministic template | Model-generated proposal | Verifiable versus flexible | Independent checks cover every claim a model proposal makes |
| Add-on execution | Reviewed trusted code | Sandboxed add-ons | Matches the confirmed trust model | Untrusted extensions become a requirement |
| Paid AWS checks | Replay in CI; live checks manual | Live on every push | Spend versus freshness | Verified pricing shows negligible cost at your volume |

## E. Main-capstone walk-through

### Illustrative manager commands (proposed syntax)

```text
agm doctor --profile declared-support.json
agm workflow validate blast-radius.yaml
agm inventory --boundary boundary.yaml
agm session run --adapter claude --profile s3-typed-tools --task workload.json
agm run blast-radius.yaml --inputs lab-scope.json
agm report RUN --format markdown
agm verify RUN --profile strict-capstone
agm resume RUN
```

Optional activation uses a separate reviewed request and never touches AWS:

```text
agm approval show REQUEST
agm approval grant REQUEST --digest EXPECTED_DIGEST
agm policy activate CANDIDATE --approval REQUEST
```

### Compact workflow definition (illustrative)

Typed inputs: `DiscoveryScope` (roots, sources, exclusions), `AuthorizationScope` (subjects, action, resource and context tuples, inspector handle), `EvidenceBundle` (R3 records and coverage manifest), `WorkloadSet` (approved observed operations and separately declared required operations), `Restrictions` (prohibited operations and approval conditions). A credential handle names an authorised resolution operation and contains no secret. `D` is deterministic, `M` is model-assisted. R4 defines retry and recovery per behaviour class.

```yaml
workflow: blast-radius
contract: 1
requires: {adapter: [session.capture, tools.typed], provider: aws.iam.s3-profile}
packs: [effects-aws-s3@^1, manager-rules@^1]
defaults: {evidence: R3, on_timeout: unknown, unknown_required_evidence: review, provider_mutation: forbidden}

steps:
  discover:  {needs: [], uses: inventory.discover, in: discovery, out: CredentialReferences, exec: D, behavior: Read}
  authorize: {needs: [discover], uses: aws.authorization_evidence, in: [CredentialReferences, authorization],
              out: AuthorizationEvidence, exec: D, behavior: Read, authority: explicit_inspector_grant}
  graph:     {needs: [discover, authorize], uses: blast.graph, out: EvidenceGraph, exec: D, behavior: Pure}
  classify:  {needs: [graph], uses: effects.classify, out: ClassifiedGraph, exec: D, behavior: Pure}
  compare:   {needs: [classify], uses: blast.compare, in: [ClassifiedGraph, trace, workload],
              out: WorkloadComparison, exec: D, behavior: Pure}
  propose:   {needs: [compare], uses: policy.candidate, in: [WorkloadComparison, restrictions],
              out: CandidateSet, exec: D, behavior: Pure}
  verify:    {needs: [propose], uses: policy.independent_verify,
              in: [CandidateSet, workload, restrictions, AuthorizationEvidence], out: Verification,
              exec: D, behavior: Read, requires: [evidence_integrity, workload_cases, prohibited_cases, coverage]}
  report:    {needs: [verify], uses: report.render, out: CheckedReport, exec: D, behavior: Local_commit,
              exit_code: from(manager.rules)}
  activate:  {optional: true, needs: [report], uses: manager.activate, in: [CandidateSet, Verification, BoundApproval],
              out: ActivationReceipt, exec: D, behavior: Local_commit,
              requires: [passing_gate, matching_digest, adapter_enforcement_probe]}
```

Read steps keep every observation and do not pretend that retries saw one atomic cloud snapshot.

### One row per capstone step

| Step | Manager component | Execution | Required add-on | Evidence or acceptance check | Principal limitation |
|---|---|---|---|---|---|
| 1 Discover references | Inventory and sanitisation | Deterministic | Claude adapter; source parsers | AC1, AC2a; inspected and excluded source manifest | Declared boundary only; a reference is not use |
| 2 Authorization evidence | Typed tools and evidence store | Deterministic | AWS add-on | AC2b, AC3; exact inputs, responses, pagination, coverage | Simulator and inspection limits in R5 |
| 3 Graph | Relational projection and export | Deterministic | AWS add-on mapping | AC4; every edge cites evidence or is unknown | A path is not proof of exploitable reachability |
| 4 Classify | Pack evaluator | Deterministic | S3 effect and recovery pack | Pack version recorded; recovery prerequisites verified or unknown | Recovery depends on backups and privileges outside view |
| 5 Compare | Event normalisation and comparison | Deterministic | Typed S3 tools; optional CloudTrail importer | AC5; attempt versus success; attribution level | Missing observations do not prove non-use |
| 6 Propose | Candidate generator | Deterministic | Manager-rule generator; AWS policy template | Two separate candidate artefacts with digests | Not globally minimal or future-sufficient |
| 7 Verify | Independent gate | Deterministic, external oracles | Rule tests; provider candidate checks | AC6; approved and prohibited matrices; conflicts kept | Finite checked properties |
| 8 Report and decide | Rule engine and renderer | Deterministic; optional non-authoritative explanation | Report template | AC7, AC8, AC10, AC11; exit code | Bounded by supplied evidence |
| 9 Activate (optional) | Approval and activation | Deterministic | Adapter enforcement binding | Exact approval digest and interception probes | Supported control points only; no AWS mutation |

### What the AWS add-on actually owns

Policy acquisition and version provenance, pagination, principal distinctions, API-to-action-to-resource mapping, context requirements, the subset evaluator, simulator request construction, supported-profile detection, normalisation, candidate construction and validation calls. It does not own workflow persistence, retries, approvals, secret logging or any scheduler.

### Manager policy versus AWS policy proposal

- **Manager policy** controls manager-owned typed operations and probed adapter paths. Example: permit selected reads, require approval for selected writes, deny deletion. It yields the CI decision.
- **AWS policy proposal** is a separate document for provider-side review, validated against the supported tuple matrix and provider checks. Attaching it would be a separate authorised mutation workflow that this release does not contain.
- A run can fail because the current credential is over-privileged even when the proposal verifies cleanly.

### Outcome examples

| Scenario | Analysis result | CI decision |
|---|---|---|
| Approved workload is list and read calls; all required scoped evidence exists; candidate preserves them and denies represented deletions | Complete within declared scope | Pass, `0` |
| Inspector cannot read an applicable policy, context is missing, or a required trace is unavailable | Incomplete; available simulated outcomes kept without promotion | Review, `12` |
| Approved observed work contains deletion while restrictions prohibit deletion | Complete conflict finding naming the operation and the opposing rule | Fail, `10` |
| A model states "no destructive access exists" while evidence covers only selected reads | Claim rejected; report built from verified records. The rejected claim never changes the exit code; if the asserted property is required and lacks evidence, the result is review | Review, `12` (DEC-006) |
| Candidate needs approval for a declared write | Approval request bound to digests | Pending, `11` |

Resolve a conflict by changing the approved workload, changing the restriction, or recording a supported exception, then re-version and re-verify. Never silently drop the conflicting operation.

### What remains difficult

- **Provider limits:** evidence can omit applicable controls and evaluates only supplied conditions; reads do not form one atomic snapshot.
- **Visibility:** network calls and non-AWS tokens are invisible to AWS evidence; provider calls outside the typed tools are gaps.
- **Attribution:** shared identities and subprocesses weaken it; exact attribution needs the manager-minted session.
- **Classification packs:** need reviewed updates as service behaviour changes.
- **Condition-dependent recovery:** depends on actual configuration, available copies and restoration authority.
- **Validation beyond observed work:** a verified proposal says nothing about unseen legitimate paths.
- **Enforcement:** a hook or proxy cannot govern paths that bypass it.

### Smallest still-useful version

Stage 3: "For this declared local workload, the manager captured supported events, reported gaps, produced a manager-policy candidate and checked specified allow and deny properties." Stages 4–5 extend that with scoped AWS evidence. Nothing converts it into universal safety.

## F. Two further capstones

### F1 — Claude Code guardrail-conformance audit

**Job:** for a repository, check that the effective Claude Code configuration implements the manager's required guardrails and report the bypass opportunities in R2.

**When:** built at the end of Stage 4 as the first contract-freeze test; re-run in Stage 7 (DEC-005).

| Step | Manager component | Execution | Required add-on | Evidence or acceptance check | Principal limitation |
|---|---|---|---|---|---|
| 1 Resolve configuration | Config resolver | Deterministic | Claude adapter (reused) | Per-key source layer matches golden file | Cloud-hosted sessions out of scope |
| 2 Hook health | Hook checker | Deterministic | Claude adapter | Missing or non-executable hook path detected | Cannot prove runtime firing without a live probe |
| 3 Rule coverage | Pack evaluator | Deterministic | `claude-guardrails` data pack | Required deny rules present | Bash rules are text-based; reported as partial |
| 4 MCP and plugin trust | Inventory | Deterministic | — | Servers listed with scope, approval state and env-var names | Non-interactive runs skip approval prompts; flagged |
| 5 Decide and report | Rule engine, renderer | Deterministic | Manager rules | Missing guardrail gives `10`; partial gives `12` | Configuration conformance is not runtime enforcement |

**Reused:** T03, T04, T06, T08, T10. **New:** one data pack and the hook checker. **Deterministic acceptance:** four fixture repositories (compliant, missing hook, non-executable hook, project MCP server with a credential env) produce the golden decisions. **Limitation:** static only. **Extensibility evidence:** a second workflow on the same contracts with no new provider and no core change. **Required level:** full implementation; it is small.

### F2 — Bounded source-code migration

**Job:** migrate one deprecated helper call to its replacement in a fixture repository, verify behaviour and export a reviewed patch.

**When:** built at the end of Stage 6; re-run in Stage 7 (DEC-005).

| Step | Manager component | Execution | Required add-on | Evidence or acceptance check | Principal limitation |
|---|---|---|---|---|---|
| 1 Establish baseline | Typed tools and evidence | Deterministic | Repository snapshot and test wrapper | Clean fixture state; baseline tests pass | Tested behaviour of the fixture only |
| 2 Propose migration | Agent adapter and tool profile | Model-assisted | Migration specification | Patch limited to declared files and operation | The proposal can be wrong |
| 3 Check migration | Independent gate | Deterministic | Syntax-tree rule plus tests | Old call absent; behaviour preserved; unrelated edits rejected | Tests do not prove full semantic equivalence |
| 4 Export patch | Local commit and reporting | Deterministic | Patch exporter | Checked diff and evidence manifest | No automatic merge or publication |

**Reused:** agent supervision, context profile, file-change evidence, policy boundaries, recovery, reporting. **New:** migration specification, syntax-tree check, fixture test wrapper. **Deterministic acceptance:** a correct patch passes; a wrong-but-compiling patch fails behavioural tests; an out-of-scope edit is rejected. **Limitation:** one fixture, one migration. **Extensibility evidence:** a model-assisted, non-security job runs on the frozen core. **Required level:** executable rehearsal; a general codemod platform is unnecessary.

## G. Capability checklist

Answer each with yes or no and an evidence reference. Correct handling of an unsupported case satisfies a check; it does not satisfy a request for the unsupported capability.

| Readiness check | Concrete demonstration |
|---|---|
| Does the complete first-release path run? | `agm rehearse first-release`; keep input, receipt, evidence, policy, verification, report |
| Is coverage explicit? | `agm rehearse coverage` (T06, Stage 2); omit a root, an event, a policy page and a source; each gap appears |
| Are references distinct from identities and use? | Fixture with an unused credential reference and a separately identified inspector |
| Does discovery avoid executing untrusted configuration? | Booby-trapped `credential_process`; marker file absent |
| Are secrets kept out of every output? | `agm rehearse secrets` (T02, Stage 1); seeded values in arguments, stderr, provider errors, artefacts and model context are absent |
| Are secret-detection limits documented? | Security document states that seeded tests do not prove arbitrary text secret-free |
| Are unknown authorization results preserved? | `agm rehearse aws-unknown` (T11, Stage 4); inaccessible policy and role-with-bucket-policy cases give `12` |
| Is evaluator disagreement surfaced? | Differential run lists disagreements; an injected unreported mismatch fails CI |
| Does interruption preserve state? | `agm rehearse recovery` (T07, Stage 3); crash around each state transition and artefact commit |
| Are ambiguous effects handled without blind retry? | Fake ledger shows exactly one effect after recovery; the unreconcilable case stops |
| Are cancellation and late results safe? | Cancel a running attempt; late output cannot mark it successful |
| Do timeouts map to unknown? | Throttled replay fixture never yields pass |
| Are stale approvals rejected? | Change arguments, scope, policy, evidence or executor profile after approval |
| Are conflicting requirements reported? | Approved deletion plus deny-deletion gives `10` and names both sides |
| Is unsupported adapter behaviour rejected? | Remove a required capability; validation rejects the workflow instead of downgrading |
| Is an untested agent version flagged? | Version outside the tested range adds a warning to the report |
| Are hook and proxy gaps visible? | Timeout, missing-hook and bypass fixtures; report names the real control boundary |
| Are unsupported model claims rejected? | Fabricated evidence IDs and over-broad claims cannot enter findings |
| Is a least-privilege overclaim impossible? | Template test: the phrase appears only in the limits section |
| Is verification independent? | Mutated producer output is caught by hand-authored expectations |
| Are schema changes deliberate? | Additive field passes; incompatible version fails with a diagnostic |
| Are duplicates and ordering handled? | Reordered, repeated and conflicting events keep causality and gaps |
| Is concurrency bounded? | A second active runner is rejected under the single-run contract |
| Do migrations and deletion work? | Upgrade an older schema; purge a run; document copies the manager cannot erase |
| Are add-on contracts real? | F1 and F2 run on the frozen core; an incompatible or tampered pack is refused |
| Is cost attribution honest and bounded? | Missing usage stays unknown; estimates are labelled; the budget guard refuses an over-cap live run |
| Does clean installation work? | Built artefact in a clean environment passes the release rehearsal |
| Is advertised live support tested? | Authorised live contract-test evidence for the pinned profile; otherwise live support is marked unverified |
| Is optional enforcement labelled accurately? | Activation receipt names the policy digest, adapter version, probed control points and bypass limits |

## H. Scope warnings

Spending is capped at ₹10,000 per month and time is open-ended. That makes endless scope the main risk, not a deadline. Bound providers, actions and claims.

| Scope pressure | Drop or defer first | Guarantee that changes | Preserve |
|---|---|---|---|
| Agent or API spend | Repeated model comparisons, optional explanations, multi-agent review | Fewer quality and cost claims | Deterministic fixtures and one real supervised-session demonstration |
| AWS access unavailable | Live collector acceptance; keep the imported-evidence profile | No claim of tested live collection | Provider semantics, unknown handling, provenance, the local evaluator |
| AWS analysis expands | More services at tuple depth, cross-account, role chains, escalation graphs, policy generation, paid checks | Narrower authorization coverage | Inspector and subject separation, explicit scope, the differential suite |
| Environment complexity | Native multi-OS support, containers, remote workers | Fewer supported environments | One tested runner and an honest isolation boundary |
| Extension complexity | Marketplace, untrusted add-ons, GitHub add-on, second adapter | Trusted-installation model stays | Stable contracts, compatibility checks, core-owned approvals |
| Polish dominates | Dashboard, hosted tenancy, automatic configuration editing | CLI-only release | Installation, documentation, clear reports and exit codes |
| Enforcement proves incomplete | Step 9 | An observation-and-checking product, not prevention | Typed manager-owned controls and explicit bypass reporting |

**AWS authorization analysis as a separate project.** This is the largest risk, and owning the evaluator raises it. Guardrails:

- Freeze the declared subset in a single document.
- Map every unsupported construct to `unknown`; do not write code for it.
- Expand the subset only through the differential suite, one construct at a time, and only when an evaluation workload needs it.
- Stop when another case would require broad IAM semantics, organisation-wide policy state or privilege-escalation reasoning. Return `unsupported`, narrow the declared profile and keep the executable negative case.
- If expansion pressure persists, shrink the subset and lean on the simulator.

**Do not cut:** the six OWN exercises, secret boundaries, explicit conflicts, independent verification, durable recovery, coverage reporting and the vendor-independent adapter contract. Without them the project is a script.
