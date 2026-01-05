---
description: Rapid clarification interview for an existing bead (3-10 questions)
---

# Blitz Command

When the user runs `/blitz`, conduct a rapid clarification interview on an existing bead.

## Usage

```
/blitz <bead-id>              # Blitz interview for specific bead
/blitz bd-a3f8                # Example with root bead
/blitz bd-a3f8.2              # Example with subtask
```

## Argument Validation

The `<bead-id>` argument is required and must match valid bead ID format:
- `bd-xxxx` (4 hex characters)
- `bd-xxxx.n` (with subtask number)
- `bd-xxxx.n.m` (nested subtask)

**If no argument provided:**
```
Usage: /blitz <bead-id>

Run `bd list` to see available beads, or use /interview for new specs.
```

**If invalid format:**
```
Invalid bead ID format: '<input>'. Expected: bd-xxxx or bd-xxxx.n
```

## Behavior

1. **Validate bead ID format**: Check argument matches expected pattern
2. **Route to interview skill**: Equivalent to `/interview --blitz <bead-id>`
3. **Load bead context**: Use `references/bead-context.md` for context loading
4. **Run blitz interview**: Adaptive 3-10 questions based on completeness
5. **Update bead**: Apply clarifications via `bd update`
6. **Optional**: Append to existing spec if one exists

## TTS Integration

Blitz mode auto-detects TTS availability (no `--voice` flag needed):
- If claude-mlx-tts is installed and server running: voice questions automatically
- If not available: proceed with text-only interview
- No user prompt for voice preference in blitz (unlike full interview)

## Relation to /interview

`/blitz <bead-id>` is equivalent to `/interview --blitz <bead-id>`.

The blitz command exists as a convenient shorthand for the common case of clarifying existing beads. Use `/interview` for new specs or when you want the full interview experience.

See the interview skill for detailed blitz mode logic.
