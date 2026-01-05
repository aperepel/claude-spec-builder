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
```

## Behavior

1. **Parse input**: Detect if argument is a topic string or file path
2. **Load skill**: Use the interview skill from `skills/interview/SKILL.md`
3. **Detect integrations**: Check for TTS and beads availability
4. **Run interview**: Conduct adaptive 9-phase interview (10-40+ questions based on scope)
5. **Generate spec**: Output structured markdown specification
6. **Optional**: Create beads epic/subtasks if beads is available

See the interview skill for detailed interview logic and phases.
