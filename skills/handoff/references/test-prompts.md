# Trigger tests — handoff

## Positive
- "Show me the handoff contract."
- "What should a standard handoff document look like?"
- "What goes into a handoff and what gets dropped?"

## Negative
- "Write a handoff for this session and save it." → `handoff-prompt`
- "Pick up the latest handoff for project api." → `handoff-pickup`

## Adversarial
- "handoff, but don't write anything — just the format rules." → `handoff`
