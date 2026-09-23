---
name: handoff-pickup
description: >
  Consumer that resumes work from a persisted handoff. Finds latest (or a given ref)
  through handoff-prompt's interface, reads it, restores verified state, re-verifies
  only mutable runtime facts, and continues from Next action without re-asking known
  context. Trigger: "pick up the handoff", "continue from the handoff", "resume",
  "latest handoff", "/handoff-pickup", or a pickup prompt naming a handoff ref.
  NOT trigger: writing a new handoff — use handoff-prompt; rules only — use handoff.
metadata:
  version: 1.0.0
  license: MIT
---

# handoff-pickup

Inherits from: handoff
Adds: retrieval through the owner interface, continuation workflow.
Depends on: handoff-prompt — `latest()`, `read()`, `format()`.

## Step 0 — parent and owner

1. Read `handoff/SKILL.md` fully.
2. Read `handoff-prompt/SKILL.md` Public interface.
3. Never construct storage paths yourself; let the owner resolve refs.

## Resume

1. Ref given (including a legacy absolute path) → `handoff-prompt.read(ref)`.
   Otherwise `handoff-prompt.latest(project)` then `read`.
2. Validate against `handoff-prompt.format()`. Accept equivalent legacy headings.
   Missing required sections are explicit gaps, not permission to invent.
3. Restore `Durable context`, `Decisions`, `Completed work`, constraints and refs as
   established context.
4. Keep `Hypotheses` as hypotheses. Never silently promote them to facts.
5. Re-verify only `Mutable state to re-verify` entries, or other runtime state whose
   current value changes the next action. For a referenced job: inspect the live
   process, its log and exit code — a self-reported "success" line is not proof.
   Run the task spec's Verify checks; if there are none, say so precisely.
   A live process with an empty log may just be buffering; never kill an
   unrelated live process. Do not run verification that races a still-running
   job in the same working tree — wait for it.
6. Do not re-research verified facts just because the session is new.
7. Do not ask the user to repeat context the handoff already holds.
8. Treat each open question outside code as unanswered; ask before changes that
   depend on it. Hard constraints stay blockers until the user lifts them.
   Ask one sharp question only if Next action cannot proceed without it.
9. Start on `Next action` immediately — do not stop at a summary.

## Failure modes

- `latest()` finds nothing → report no handoff for that project; do not go
  searching arbitrary paths behind the owner's back.
- `read()` fails → report the owner failure; do not bypass it.
- Document contains a secret-looking value → do not repeat it; treat as a producer
  defect and continue with redacted context.
- Runtime evidence changed → update working state; keep the old value as history,
  not as "wrong".

## Interface

Stateless consumer.
`handoff-pickup.resume(project, ref?) → {verified_state, hypotheses, unresolved_work, next_action}`

## Tests

See `references/test-prompts.md`.
