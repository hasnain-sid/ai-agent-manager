# Progress

Status values: `not started` / `learning` / `building` / `done`.
"Stage rehearsal passed" records whether the executable checkpoint of the topic's stage has
passed (stage in brackets).

## Topics

| Topic | Depth | Status | Stage rehearsal passed |
|---|---|---|---|
| T01 Harness behaviour and extension surfaces | STEER (medium) | not started | no (S1) |
| T02 Manager authority, secrets and threat model | OWN (large) | not started | no (S1) |
| T03 Typed contracts, bounded tools and the CLI contract | STEER (medium) | not started | no (S1) |
| T04 Capability-aware adapters | STEER (medium) | not started | no (S2) |
| T05 Events, provenance and durable evidence storage | OWN (large) | not started | no (S2) |
| T06 Bounded discovery and identity references | STEER (medium) | not started | no (S2) |
| T07 Workflow state, recovery and approvals | OWN (large) | not started | no (S3) |
| T08 Manager policy and enforceable decisions | OWN (large) | not started | no (S3) |
| T09 Independent verification and claim gating | STEER (medium) | not started | no (S3) |
| T10 Add-on contracts and pack lifecycle | STEER (medium) | not started | no (S4) |
| T11 IAM evaluation subset and scoped AWS authorization evidence | OWN (large) | not started | no (S4) |
| T12 Evidence graph and conditional effect classification | STEER (medium) | not started | no (S4) |
| T13 Provider-side observation and attribution | STEER (medium) | not started | no (S5) |
| T14 Policy synthesis and independent verification | OWN (large) | not started | no (S5) |
| T15 Context, memory and tool exposure | STEER (medium) | not started | no (S6) |
| T16 Multi-agent applicability and authority | AWARE (small) | not started | no (S6) |
| T17 Telemetry, cost and representative evaluation | STEER (medium) | not started | no (S6) |
| T18 CLI, compatibility and open-source delivery | STEER (medium) | not started | no (S7) |

## Stage checkpoints

`agm` invocations are PROPOSED (illustrative) per the blueprint.

| Stage | Checkpoint | Size | Topics | Invoke | Gates | Passed |
|---|---|---|---|---|---|---|
| 1 | Establish the boundary and run one typed operation | medium | T01-T03 | `agm rehearse foundation`, `agm rehearse secrets` (T02) | Valid input gives expected result + R3 envelope; malformed output, out-of-root path and seeded secret rejected or sanitised; stub steps exit `12`; `secrets` (Stage 1 scope: arguments, stderr, exceptions, artefacts) finds no seeded value | no |
| 2 | Capture a supervised session and discover its declared environment | large | T04-T06 | `agm rehearse capture --adapter claude`, `agm rehearse capture --adapter replay`, `agm inventory --boundary boundary.yaml`, `agm rehearse coverage` (T06), `agm rehearse secrets` (T02) | AC1, AC2a, session half of AC5; `coverage` (omitted root, event, source) shows each gap; `secrets` adds model context and session events; booby-trapped `credential_process` not executed; missing capability gives `unsupported` | no |
| 3 | Ship the first useful end-to-end release | large | T07-T09 | `agm rehearse first-release`, `agm rehearse recovery` (T07) | AC7, AC9, AC11; `recovery` survives a crash around each transition and commit; approved read preserved, prohibited write rejected; fake-ledger crash reconciled | no |
| 4 | Establish scoped AWS reachable authority | large | T10-T12 | `agm rehearse blast-radius --only 1-4 --source fixtures`, `agm rehearse aws-unknown` (T11), `agm rehearse secrets`, `agm rehearse coverage`, `agm rehearse guardrail-audit` (F1, T10) | AC2b, AC3, AC4; `secrets` adds provider errors; `coverage` adds an omitted policy page; evaluator/simulator disagreements listed; missing evidence exits `12`; `aws-unknown` gives `12`; contract freeze after T10; F1 built and run as first freeze test | no |
| 5 | Compare observed work, propose and verify (capstone complete) | large | T13-T14 | `agm rehearse blast-radius --source fixtures` | AC5, AC6, AC8, AC10; exits `0` / `12` / `10` on the defined cases | no |
| 6 | Use native context and tools, then measure the result | medium | T15-T17 | `agm rehearse context-and-evaluation`, `agm eval run --suite evals/blast-radius`, `agm rehearse code-migration` (F2, T15) | Findings stable across context variants; unsupported claims rejected; budget guard refuses over-cap run; F2 built and run on the frozen core, model step replayed | no |
| 7 | Package and prove the scoped open-source release | medium | T18 | `agm rehearse release --profile declared-support.json` | Declared support matrix passes; section G evidence; `guardrail-audit` and `code-migration` re-run; AC12; installed artefact reproduces golden report | no |
