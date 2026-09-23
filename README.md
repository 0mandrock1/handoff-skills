# handoff-skills

Three [Claude Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
that turn "end of a long session" into a file the next session can actually resume from —
without re-asking what was already established, and without trusting a job's
self-reported success.

| Skill | Role |
|---|---|
| `handoff` | Contract: document schema, facts vs hypotheses, what to keep/drop, secret boundary. Owns no state. |
| `handoff-prompt` | Producer. Writes the handoff, verifies it by read-back, returns a copy-paste pickup prompt. Sole owner of storage. |
| `handoff-pickup` | Consumer. Reads the latest handoff via the producer's interface, re-verifies only mutable state, starts from `Next action`. |

## Why three skills

- **One owner of storage.** Only `handoff-prompt` knows where files live. The consumer
  asks it (`latest()`, `read()`), so moving storage never breaks pickup.
- **Facts ≠ hypotheses.** The schema forces the split; pickup must not promote a guess
  to a fact.
- **Evidence over "RESULT: ok".** For background jobs the handoff keeps PID, log, exit
  code and Verify steps; pickup checks them instead of believing the job.
- **Read-back or it didn't happen.** A write that can't be read back is a failure.

## Install

Copy `skills/*` into your skills directory, e.g. for Claude Code:

```sh
cp -r skills/* ~/.claude/skills/
```

Or grab the prebuilt `.skill` bundles from [Releases](../../releases).

## Storage

Default backend is a local directory, resolved in order:
`$HANDOFF_DIR` → `handoff.dir` in `.claude/handoff.json` → an existing
`handoffs/`/`.handoffs/` → `<workspace>/.handoffs/`.

In an ephemeral sandbox (e.g. a chat container wiped after the session) the producer
refuses to claim persistence and prints the handoff in chat instead.

Swapping in another backend (remote shell, object store, notes app) means rewriting
only the storage section of `handoff-prompt` — the interface stays the same.

## Usage

End of session:

> handoff for project api

Next session, paste the returned pickup prompt, or:

> pick up the latest handoff for api

## Background

Write-up (Ukrainian): <https://articles.mandrock.me/claude/handoff-skills/>

## License

MIT
