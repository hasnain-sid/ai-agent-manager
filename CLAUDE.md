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

## Environment and language (CONFIRMED)

- **Host:** Linux under WSL2 is the supported environment, and Claude Code runs inside WSL2.
  Native Windows is offline analysis only, because the Claude Code shell sandbox (R2 C5) is
  unavailable there (DEC-001).
- **Repository location:** the repo must live on the WSL2 filesystem (for example under the WSL2
  home directory), not under `/mnt/c`. `/mnt/c` is slow and does not reliably keep Unix permission
  bits, which hook probes depend on. Text files use LF line endings, enforced by `.gitattributes`
  (DEC-015).
- **Language:** Python (DEC-002).
- **Agent access:** existing Claude Code access for T01-T03 development (DEC-008). Release access
  is open question OQ-001, resolved at T04.

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
   | Demonstrated policy conflict or failed deterministic check; CI fail | `10` |
   | Approval pending | `11` |
   | Required evidence incomplete or unsupported; review required | `12` |

   A rejected model claim never changes the exit code (DEC-006).

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

## Core and the contract freeze (DEC-004, DEC-010)

**Core** is: the evidence envelope (R3); the evidence store schema and its migrations; the step
runner and state machine (R4); the adapter protocol; the add-on and pack loader; the inventory
engine (source parsers stay in the adapter); the manager policy engine; the claim gate; graph
storage and query; the report schema; the CLI surface; and the exit codes.

After the Stage 4 freeze (after T10):

- No breaking changes and no capstone-specific code in core.
- Additive, versioned fields are allowed, and each one requires re-running the contract fixtures.

Outside core:

- **Blast-radius add-on:** graph mapping (`blast.graph`) and comparison (`blast.compare`).
- **AWS add-on:** the CloudTrail importer, the attribution join and the T14 verifier.
- **Evaluation module:** the T17 spend guard and evaluation ledger.

Verifier independence (DEC-012): the verifier may import the T11 evaluator and must never import
the synthesiser; the synthesiser imports neither the evaluator nor the verifier. An import-boundary
test enforces this. A verification that used only the local evaluator is labelled `local-only` and
is not an AC6 pass.

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

Product code does not exist yet. Its layout is decided at T03 and logged in docs/decisions.md
then (DEC-009). Organise it so core (above) is separable from add-ons and the evaluation module.

- **Learning material goes in `learning/Txx/`**, using the layout in learning/README.md.
- **Product code is organised by product structure, never by topic.** No `t03/` packages or
  topic-named modules in product code; a topic's build deliverable lands wherever the product
  structure says it belongs, and `learning/Txx/04-build-task.md` links to it.
