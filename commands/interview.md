---
description: Conduct an adaptive interview for requirements gathering and spec generation
---

# Interview Command

When the user runs `/interview`, conduct a structured requirements-gathering interview.

## Usage

```
/interview                           # Interactive - ask what to interview about
/interview <topic>                   # Interview about a specific topic
/interview <file-path>               # Use file as context for interview
/interview <topic> --voice           # Enable TTS for questions (requires claude-mlx-tts)
/interview <topic> --template=<type> # Use specific template (feature, bug, refactor, integration)
/interview --blitz <bead-id>         # Blitz mode: rapid clarification for existing bead
```

## Flags

| Flag | Description |
|------|-------------|
| `--voice` | Enable TTS for questions (requires claude-mlx-tts) |
| `--template=<type>` | Use specific template (feature, bug, refactor, integration) |
| `--blitz` | Blitz mode for existing beads (3-10 questions vs 10-40+) |

## Modes

### Standard Mode (default)

Full interview for new specifications:
1. **Parse input**: Detect if argument is a topic string or file path
2. **Load skill**: Use the interview skill from `skills/interview/SKILL.md`
3. **Load context and detect TTS in parallel**:
   <!-- PARALLELIZATION: These are independent operations with no shared state -->
   - **a) Load context**: If file path provided, read file contents; detect work type from input
   - **b) Detect TTS**: Invoke `tts-status` skill (see TTS Detection below)
   - If TTS detection fails, mention it and proceed text-only (graceful degradation)
4. **Check beads availability**: Look for `.beads/` directory
5. **Run interview**: Conduct adaptive 9-phase interview (10-40+ questions based on scope)
6. **Generate spec**: Output structured markdown specification
7. **Optional**: Create beads epic/subtasks if beads is available

### Blitz Mode (`--blitz`)

Rapid clarification for existing beads:
1. **Validate bead ID**: Check format and existence
2. **Load bead context**: Use `references/bead-context.md` for rich context loading
3. **Assess completeness**: Score bead for gaps and missing info
4. **Run blitz interview**: Adaptive 3-10 questions based on work type and completeness
5. **Update bead**: Apply clarifications via `bd update`
6. **Optional**: Append to existing spec if one exists

**Blitz examples:**
```
/interview --blitz bd-a3f8      # Blitz on root bead
/interview --blitz bd-a3f8.2    # Blitz on subtask
```

**Note:** `/blitz <bead-id>` is a shorthand for `/interview --blitz <bead-id>`

## TTS Detection

**CRITICAL: To detect TTS, invoke the skill - do NOT use curl or HTTP requests.**

```
Invoke skill: claude-mlx-tts:tts-status
```

**DO NOT:**
- ❌ Use `curl` to probe HTTP endpoints
- ❌ Guess ports or health check paths
- ❌ Run shell commands to detect TTS

**Detection results:**
- If skill runs and says "running" → TTS available
- If skill runs and says "not running" → invoke `/tts-start`, retry
- If skill fails or doesn't exist → inform user: "TTS not available, continuing without voice"

**Mode-specific behavior:**

| Mode | TTS Behavior |
|------|--------------|
| Standard | Ask user preference, or use `--voice` flag |
| Blitz | Auto-detect TTS, enable if available (no prompt) |

## ⚠️ Visual Output Is Always Required

**TTS is SUPPLEMENTARY, not a replacement for visual output.**

Every message that is voiced MUST also be printed to the terminal. This applies to:
- Questions
- Progress indicators
- Summaries
- Phase transitions

TTS adds voice ON TOP OF visual output. It never replaces it.

See the interview skill for detailed interview logic and phases.
