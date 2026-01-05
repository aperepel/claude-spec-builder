# claude-spec-builder

Claude Code plugin for structured requirements gathering via adaptive interviews.

## Install

```bash
claude plugins install aperepel/claude-spec-builder
```

## Usage

```bash
/interview                    # Start interactive interview
/interview user-auth          # Interview about a topic
/interview docs/plan.md       # Use file as context
```

## What it does

Conducts adaptive interviews (10-40+ questions based on scope) and outputs structured markdown specifications. Works standalone or integrates with:

- **beads** - Generate epic + subtasks from spec
- **claude-mlx-tts** - Voice the interview questions

## License

MIT
