# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Claude Spec Builder is a Claude Code plugin that conducts adaptive interviews for structured requirements gathering. It produces actionable markdown specifications and optionally creates beads (issue tracking) for implementation.

## Architecture

- **Plugin structure**: Uses `.claude-plugin/plugin.json` for plugin metadata
- **Commands**: `commands/interview.md` and `commands/blitz.md` define slash commands
- **Skills**: `skills/interview/SKILL.md` contains the core interview logic
- **References**: `skills/interview/references/` contains templates, phases, and integration guides

## Plugin Development Notes

### Invalid plugin.json Fields (Claude Code 2.1.x)

Do NOT add `minClaudeVersion` to plugin.json - this field is invalid and causes plugin load failures:
```
"Validation errors: Unrecognized key: minClaudeVersion"
```

### Hooks Location

If adding hooks, define them inline in `plugin.json` rather than a separate `hooks/hooks.json` for better reliability:

```json
{
  "name": "claude-spec-builder",
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Session ended.'"
          }
        ]
      }
    ]
  }
}
```

### Command Types

This plugin uses **instructional commands** (not script wrappers):
- Commands describe complex multi-phase behavior for Claude to follow
- NOT suitable for `!` backtick inline execution pattern
- AskUserQuestion interactions require Claude interpretation

## Key Commands

- `/interview [topic]` - Full adaptive interview (10-40+ questions)
- `/interview --voice` - Interview with TTS enabled
- `/blitz <bead-id>` - Rapid clarification for existing bead (3-10 questions)

## TTS Integration

When detecting TTS availability, use the skill invocation pattern:
```
Invoke skill: claude-mlx-tts:tts-status
```

Do NOT use curl or HTTP requests to probe TTS endpoints.
