---
name: handoff-prompt
description: >
  Producer and sole owner of persisted handoff documents. At the end of a session
  (or on request) builds a self-contained handoff, resolves the storage location,
  writes it, verifies by read-back, and returns a copy-paste pickup prompt for the
  next session. Trigger: "handoff", "write a handoff", "wrap up the session",
  "pass context to the next chat", "/handoff-prompt". NOT trigger: rules/structure
  only — use handoff; resuming from an existing handoff — use handoff-pickup.
metadata:
  version: 1.0.0
  license: MIT
---

# handoff-prompt

Inherits from: handoff
Adds: storage resolution, persistence, read-back verification, pickup prompt.
Reuses verbatim: document semantics, preserve/drop rules, secret boundary.

Target runtime: any agent with a filesystem that **outlives the session**
(Claude Code, a desktop agent, a remote shell). See "Ephemeral runtimes".

## Step 0 — parent

Read `handoff/SKILL.md` fully before collecting or writing anything.

## Encapsulation

### Owns
Persisted handoff documents. The physical location is private to this skill;
consumers use only the interface below and never construct paths themselves.

### Public interface
- `handoff-prompt.resolve(project) → {storage_root, basis}` — where handoffs for
  `project` live, and why.
- `handoff-prompt.write(project, document) → {handoff_ref, file_path, pickup_prompt}` —
  persist a new handoff, read it back, return a logical ref + pickup prompt.
- `handoff-prompt.latest(project) → {handoff_ref, metadata}` — newest handoff.
- `handoff-prompt.read(handoff_ref) → {document, metadata}`.
- `handoff-prompt.format() → schema` — the parent schema.

### Internal
Storage resolution, path construction, collision handling, read-back, ref↔path mapping.

## Storage resolution (default backend: local directory)

Resolve `storage_root` in this order, first hit wins; record which one in `basis`:

1. Env `HANDOFF_DIR`.
2. `handoff.dir` in `.claude/handoff.json` at the workspace root.
3. An existing `handoffs/` or `.handoffs/` directory already used by this project
   (evidence: prior `HANDOFF-*.md` files, or git tracking).
4. Fallback: `<workspace>/.handoffs/`.

Layout: `<storage_root>/<project>/HANDOFF-<project>-<UTC yyyymmddTHHMMZ>.md`.
Logical ref: `handoff:<project>:<UTC yyyymmddTHHMMZ>`.

Never overwrite: on collision append `-2`, `-3`, … Never delete old handoffs.
Other backends (remote shell, object store, notes app) are fine: keep the same
interface and the same `resolve → write → read-back` guarantees.

## Ephemeral runtimes

If the filesystem is discarded after the session (e.g. a sandboxed chat container),
writing there is **not** persistence. Return `storage_unresolved`, print the full
document and the pickup prompt in chat, and do not claim it was saved.

## Produce

1. Collect only context relevant to continuation.
2. Build the document with the parent schema.
3. Stable facts → `Durable context`; process/git/job state → `Session state`.
4. Hypotheses stay separate from verified facts; evidence is terse.
5. Record failed attempts only when a successor might repeat them.
6. Record tool/runtime limitations hit this session.
7. Include exact repos/files/paths already established — never invented ones.
8. `Unresolved work` lists real blockers; `Next action` is one executable step.
   For a referenced background job keep: branch, job ID, PID, working dir, log,
   exit-code location, task spec and its Verify (or "no Verify section").
   List every open question outside code and every hard constraint.
9. Secret-boundary review before persisting (scan for key/token/password-shaped values).
10. `resolve(project)`, then write a new file.
11. Immediately `read(handoff_ref)` and compare with the intended content.
    Mismatch or failed read-back = failure; never report success.
12. Return the pickup prompt below, inside one fenced code block.

## Pickup prompt shape

```text
Continue work on <scope>.
Use handoff-pickup for the latest handoff: project=<project>, ref=<handoff_ref>.
Restore verified state through the owner interface and start from Next action
immediately. Do not re-ask established facts; re-verify mutable runtime state
only when it matters. Do not take a job's self-reported success on faith.
```

## Tests

See `references/test-prompts.md`.
