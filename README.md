# claude-spec-builder

Claude Code plugin for structured requirements gathering via adaptive interviews.

## Install

```bash
claude plugins install aperepel/claude-spec-builder
```

## Quick Start

Choose your command based on what you need:

| Scenario | Command | Questions | Duration |
|----------|---------|-----------|----------|
| **New feature or project** | `/interview <topic>` | 10-40+ | Full interview |
| **Clarify existing bead** | `/blitz <bead-id>` | 3-10 | Quick clarification |
| **Use existing doc as context** | `/interview <file-path>` | 10-40+ | Full interview |
| **Voice-enabled interview** | `/interview <topic> --voice` | 10-40+ | Full interview + TTS |

### Decision Tree

```
Need requirements gathering?
│
├─ Do you have an existing bead to clarify?
│  └─ YES → Use /blitz <bead-id>
│     • Loads full bead context (description, dependencies, files)
│     • Auto-detects TTS if available
│     • 3-10 targeted questions
│     • Updates bead automatically
│
└─ NO → Starting something new?
   └─ Use /interview <topic>
      • Full adaptive interview (10-40+ questions)
      • Generates structured spec
      • Optional: --voice flag for TTS
      • Optional: Auto-creates beads epic if available
```

## The Richest Experience

**For maximum productivity, combine all three tools:**

1. **MLX TTS** ([claude-mlx-tts](https://github.com/aperepel/claude-mlx-tts)) - Voice output for questions
2. **Beads** ([beads](https://github.com/aperepel/beads)) - Task management and tracking
3. **claude-spec-builder** (this plugin) - Requirements gathering

### Full-Stack Workflow

```bash
# 1. New feature interview with voice
/interview user-authentication --voice

# Claude will:
# ✓ Detect MLX TTS and enable voice output
# ✓ Voice each question while displaying it visually
# ✓ Conduct adaptive 10-40+ question interview
# ✓ Generate structured spec
# ✓ Auto-create beads epic with subtasks

# 2. Later: Clarify a specific subtask
bd list                        # Find the bead ID
/blitz bd-a3f8.2              # Quick 3-10 question clarification

# Claude will:
# ✓ Load full bead context (description, files, dependencies)
# ✓ Auto-detect and enable TTS (no --voice flag needed)
# ✓ Ask targeted questions based on completeness
# ✓ Update the bead with new information
```

## Commands Reference

### /interview

Full adaptive interview for new specifications.

```bash
/interview                           # Interactive - ask what to interview about
/interview <topic>                   # Interview about a specific topic
/interview <file-path>               # Use file as context for interview
/interview <topic> --voice           # Enable TTS for questions
/interview <topic> --template=<type> # Use specific template (feature/bug/refactor/integration)
/interview --blitz <bead-id>         # Blitz mode (same as /blitz)
```

**Output:**
- Structured markdown specification
- Auto-creates beads epic + subtasks (if beads available)

### /blitz

Rapid clarification interview for existing beads.

```bash
/blitz <bead-id>              # Blitz interview for specific bead
/blitz bd-a3f8                # Example with root bead
/blitz bd-a3f8.2              # Example with subtask
```

**Output:**
- Updates bead with new information
- Appends to existing spec (if one exists)

**Auto-loaded context:**
- Bead description, type, status, priority
- Referenced files from description
- Parent epic (if hierarchical)
- Dependencies and blockers
- Completeness assessment

## Integration Features

### MLX TTS (Voice Output)

When [claude-mlx-tts](https://github.com/aperepel/claude-mlx-tts) is installed:

| Command | Behavior |
|---------|----------|
| `/interview` | Ask user preference or use `--voice` flag |
| `/blitz` | Auto-detect and enable if available (no prompt) |

**Note:** Voice is supplementary - all voiced questions are also displayed visually.

### Beads (Task Management)

When [beads](https://github.com/aperepel/beads) is installed:

- `/interview` → Auto-creates epic + subtasks from spec
- `/blitz <bead-id>` → Auto-updates bead with interview findings

## Examples

### New Feature (Full Experience)

```bash
# With TTS and beads installed
/interview "add OAuth login" --voice

# Result:
# • Voiced interview with 15-30 questions
# • Generated spec: specs/oauth-login.md
# • Created bead epic: bd-x1y2
# • Created 5 subtasks: bd-x1y2.1 through bd-x1y2.5
```

### Clarify Existing Work

```bash
# Check what needs clarification
bd list

# Blitz on a sparse bead
/blitz bd-x1y2.3

# Result:
# • Loaded context (description, dependencies, files)
# • Asked 8 targeted questions
# • Updated bd-x1y2.3 with new details
# • Appended to specs/oauth-login.md
```

### Without Integrations

Works perfectly standalone - just text-based interviews:

```bash
/interview "refactor auth module"

# Result:
# • Text-based interview
# • Generated spec: specs/refactor-auth-module.md
```

## License

MIT
