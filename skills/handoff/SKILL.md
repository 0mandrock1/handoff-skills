---
name: handoff
description: >
  Behavioural parent of the handoff skill family. Defines the shared handoff
  contract: document schema, durable context vs session state, verified facts vs
  hypotheses, completed vs unresolved work, a single Next action, what to preserve,
  what to drop, and the secret boundary. Trigger: "handoff contract", "handoff
  structure", "what goes into a handoff", "handoff rules". NOT trigger: writing a
  handoff or wrapping up a session — use handoff-prompt; resuming from an existing
  handoff — use handoff-pickup.
metadata:
  version: 1.0.0
  license: MIT
---

# handoff

Behavioural contract only. Owns no persistent state and never reads storage.

## Core contract

1. A handoff lets a successor continue without re-collecting established facts.
2. Separate, explicitly:
   - **Durable context** — stable rules, architectural decisions, invariants.
   - **Session state** — current branch, running job, temp artifacts, uncommitted changes.
   - **Verified facts** — confirmed by a command, file, API, or authoritative source.
   - **Hypotheses** — assumptions still needing verification.
   - **Completed work** — what was actually done *and* verified.
   - **Unresolved work** — what remains, with blocker/dependency where known.
   - **Next action** — exactly one concrete step the successor starts with.
   - For a background/headless job: job ID, PID, working dir, log path, exit-code
     location, task spec path and its Verify section (or honestly "none").
   - For open questions outside code: each one separately, no guessed answers.
3. Do not force the successor to re-research verified facts. Re-verify only mutable
   external/runtime state, contradictory data, stale evidence, or a fact the current
   step's correctness depends on.
4. Keep failed attempts when they narrow the search space or prevent a repeat.
5. Keep tool/runtime limitations that affect continuation.
6. Keep concrete repo/file/path refs; never invent them.

## Preserve

Scope and goal; decisions + rationale when rationale constrains future changes;
verified facts with terse evidence; mutable state to re-verify; completed work and its
verification; failed attempts and why; runtime limits; relevant repos/files/paths;
unresolved work, blockers, constraints; one Next action.

## Drop

Verbatim transcript without unique decisions; repetition and chatter; hypotheses already
refuted; large logs (keep path + conclusion instead); irrelevant profile context;
any secret.

## Secret boundary

Never include credentials, passwords, tokens, cookies, private keys, authorization
headers, secret values, or reversible encodings of them. If a secret mattered, record
only the owner/interface to obtain it (e.g. "env `API_KEY`, from your secret manager"),
never the value.

## Canonical document schema

```markdown
# HANDOFF <scope> <timestamp-or-version>

## Goal
## Durable context
## Session state
## Verified facts
## Hypotheses
## Decisions
## Completed work
## Failed attempts
## Tool / runtime limitations
## Repos / files / paths
## Unresolved work
## Constraints
## Mutable state to re-verify
## Next action
```

Optional when relevant: `## Run / Verify`, `## Open questions outside code`,
`## Session lessons`. Preserve equivalent headings from older handoffs.

Every unverified statement is explicitly marked hypothesis/unknown. Empty sections may
be omitted, except `Goal`, `Verified facts`, `Unresolved work`, `Next action`.
