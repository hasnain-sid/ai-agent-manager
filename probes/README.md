# Probes

This folder will hold executable probes against Claude Code: small, repeatable experiments that
establish what an installed version actually does, rather than what documentation says it does.

- R2 in `docs/blueprint.md` lists documentation-backed capabilities. A capability is advertised
  only after a probe here has demonstrated it.
- Every probe result is recorded against a pinned Claude Code (and, where used, Agent SDK) version.
  A different version invalidates the result until the probe is re-run.
- The first probe suite is the T01 build deliverable: a successful call, a refusal, compaction,
  interruption, and hook exit 0, exit 2, crash, timeout, missing script and malformed JSON.
- Probes never contain or print secret values; use canary fixtures.

Empty until T01 begins.
