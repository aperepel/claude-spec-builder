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

**CRITICAL: To detect TTS, invoke the skill - do NOT use curl or HTTP requests.**

```
Invoke skill: claude-mlx-tts:tts-status
```

**DO NOT:**
- ❌ Use `curl` to probe HTTP endpoints
- ❌ Guess ports or health check paths
- ❌ Run shell commands to detect TTS

**Behavior:**
- If skill runs and says "running" → enable voice, use `/say` for questions
- If skill runs and says "not running" → invoke `/tts-start`, retry
- If skill fails or doesn't exist → inform user: "TTS not available, continuing without voice"
- No user prompt for voice preference in blitz (auto-enable if available)

**⚠️ Visual output is MANDATORY regardless of TTS state.**

TTS is SUPPLEMENTARY - it adds voice ON TOP OF visual output. Every message that is voiced MUST also be printed to the terminal. TTS never replaces text output.

## Relation to /interview

`/blitz <bead-id>` is equivalent to `/interview --blitz <bead-id>`.

The blitz command exists as a convenient shorthand for the common case of clarifying existing beads. Use `/interview` for new specs or when you want the full interview experience.

See the interview skill for detailed blitz mode logic.
