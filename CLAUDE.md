# AI Agent Manager

## What this project is

AI Agent Manager (proposed CLI name `agm`) is an open-source, vendor-independent manager that
supervises one AI coding-agent session (Claude Code is the first adapter; other agents come later)
and answers the blast-radius question for agents that can reach over-privileged credentials:
which credential references exist inside a declared boundary, what authority they actually carry
(IAM policy evaluation over a declared subset, cross-checked against AWS simulators), what the
supervised session really did, and whether a smaller, independently verified policy would have
sufficed. Its core work is IAM policy evaluation, permission-graph construction and minimal-policy
synthesis with a verifier that shares no logic with the synthesiser. Every result is a
deterministic JSON and Markdown report with explicit coverage and unknowns, plus a CI exit code.
It is built solo and AI-assisted, stage by stage (Stages 1-7) and topic by topic (T01-T18), and the
builder must understand the core personally. The motivating analogue is the PocketOS incident:
instructions are not authority boundaries, and reachable credential scope is the blast radius.

## Source of truth

Source of truth: docs/blueprint.md. Read the relevant topic card and shared references (R1-R6)
before any task.

- R1 capstone steps and acceptance criteria AC1-AC12
- R2 Claude Code capability matrix (C1-C14)
- R3 evidence envelope
- R4 execution and decision contracts (step states, behaviour classes, approvals, exit codes)
- R5 AWS provider evidence boundary
- R6 generated-code review checklist (G1-G10)

Progress lives in docs/progress.md. Architectural decisions are logged in docs/decisions.md.

## Current position

Stage 1, topic T01 (Harness behaviour and extension surfaces; STEER; medium). Nothing built yet.

## Standing rules

1. **Unknown is never treated as allow or deny.** `unknown` stays `unknown` through evaluation,
   graph, report and exit code (R3, R4).
2. **No secret values in any output, log or file.** Store locations, handles and fingerprints,
   never values. This includes code, fixtures (use canaries), commit messages, chat replies and
   reports. Discovery never implies permission to use a credential.
3. **Exit codes follow R4.** Never put a decision on `1` or `2`.

   | Result | Exit |
   |---|---:|
   | Required properties demonstrated; CI pass | `0` |
   | Internal error or unhandled exception (never a decision) | `1` |
   | Usage error, invalid configuration or incompatible contract | `2` |
   | Demonstrated policy conflict or failed required check; CI fail | `10` |
   | Approval pending | `11` |
   | Required evidence incomplete or unsupported; review required | `12` |

4. **Labels keep their meaning.**
   - **CONFIRMED**: an input the user stated.
   - **REC**: a recommendation of the blueprint.
   - **PROPOSED**: an interface or definition invented in the blueprint (all `agm` commands,
     workflow syntax, schemas and filenames are illustrative, not an existing product).
   - **UNVERIFIED**: verify against current docs; never an unconditional dependency.
   Do not promote REC or PROPOSED to CONFIRMED without the user saying so.
5. **Never invent vendor commands, flags, APIs or file paths.** Verify against official docs
   (Claude Code, Agent SDK, AWS, Python libraries) or label the item UNVERIFIED. R2 rows marked
   with a dagger were not re-verified and must be probed before being relied on.

Depth levels: **OWN** = implement or debug the essential behaviour yourself and defend it without
notes. **STEER** = specify, review and test generated code. **AWARE** = explain applicability and
recognise when it becomes relevant.

## OWN topics: hints and review only

The six OWN topics are **T02, T05, T07, T08, T11 and T14**. For their independent exercises
(blueprint section B and card field 7) you must **not** write the solution. Give hints, ask
guiding questions, point to the relevant blueprint section or official docs, and review code the
user wrote. Do not produce the implementation, a near-complete skeleton, or a patch that fixes the
seeded bug. The rest of an OWN topic's build deliverable may be steered normally.

| Topic | Independent exercise |
|---|---|
| T02 Authority, secrets, threat model | Deny-by-default tool wrapper; debug an env-var leak into a child process |
| T05 Events, provenance, storage | Canonical digest with order-invariance test; transactional dedup; repair a missing tool-result link with an interleaved subagent |
| T07 Workflow state, recovery, approvals | Transition validation; debug the effect-before-commit crash against a fake ledger |
| T08 Manager policy | Precedence with exhaustive truth-table test; fix a seeded wildcard or stale-approval bypass |
| T11 IAM evaluation subset | Hand-written statement matching; `NotAction` + `Deny` bug hunt; deny-monotonicity property |
| T14 Synthesis and verification | Hand-written minimal policy for the fixture session, checked by custom-policy simulation and validation |

## Folder map

```text
CLAUDE.md                 this file
docs/
  blueprint.md            roadmap and source of truth (do not edit casually)
  progress.md             topic and stage-checkpoint status
  decisions.md            decision log
learning/
  README.md               standard per-topic layout
  Txx/                    one folder per topic, created when the topic starts
probes/
  README.md               executable probes against Claude Code (results pinned to a version)
```

Product code does not exist yet. Its layout will be decided (and logged in docs/decisions.md)
when Stage 1 build work begins.

- **Learning material goes in `learning/Txx/`**, using the layout in learning/README.md.
- **Product code is organised by product structure, never by topic.** No `t03/` packages or
  topic-named modules in product code; a topic's build deliverable lands wherever the product
  structure says it belongs, and `learning/Txx/04-build-task.md` links to it.
