# Feature Specification: claude-spec-builder

**Date:** 2026-01-04
**Status:** Draft
**Author:** aperepel with Claude Interview
**Interview Duration:** 65 questions across 9 phases

---

## Executive Summary

**claude-spec-builder** is a Claude Code plugin that consolidates fragmented requirements-gathering approaches (doc-coauthoring, brainstorming, Socratic questioning) into a comprehensive, one-stop-shop interview tool. It conducts adaptive interviews (10-40+ questions based on scope), outputs structured markdown specifications, and optionally integrates with beads (epic/subtask generation) and claude-mlx-tts (voice the interview).

---

## Problem Statement

### Current State

Power users of Claude Code currently:
- Cobble together multiple point skills from various marketplaces
- Use one-shot spec generation followed by reactive refinement
- Manually review and edit specs through back-and-forth loops

This approach is **fragmented** (multiple tools), **reactive** (patching after the fact), and **incomplete** (assumptions surface too late).

### Desired State

A single `/interview` command that:
- Proactively surfaces all assumptions, constraints, and edge cases upfront
- Adapts depth based on scope (epic = 40+ questions, task = 10-15)
- Generates actionable specs that translate directly to work items
- Integrates with existing tools (beads, TTS) when available

### Impact of Inaction

- Copy-paste friction between projects
- Unrealized community value
- Continued fragmented tooling experience

---

## Users & Stakeholders

### Primary Users

All Claude Code developers (power users assumed):
- Solo developers/founders building products
- Product managers capturing requirements
- Tech leads designing systems
- Anyone who needs to spec work before coding

### Technical Skill Level

Power users—familiar with Claude Code, plugins, commands, and spec-driven development. No hand-holding required.

### Discovery

Multi-channel distribution:
- Claude Plugin Marketplace
- GitHub visibility (README, topics, awesome-lists)
- Social/content (Twitter/X, blog posts)
- Organic discovery

---

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-1 | `/interview` command accepts topic or file path | Must | Topic detected vs file path, interview starts |
| FR-2 | 9-phase interview structure with adaptive depth | Must | Phases skip/condense based on work type |
| FR-3 | AskUserQuestion for structured choices | Must | 2-4 options presented when applicable |
| FR-4 | Generate markdown spec to detected project path | Must | Spec written to specs/, docs/, or prd/ |
| FR-5 | TTS integration (optional, auto-detected) | Must | Questions voiced via /say, summaries via /summary-say |
| FR-6 | Beads integration (optional, auto-detected) | Must | Epic + subtasks created via bd CLI |
| FR-7 | Interview templates (Feature, Bug, Refactor, Integration, Adaptive) | Should | Templates loaded from references/templates/ |
| FR-8 | Pre-flight check for integrations | Must | Verify TTS/beads work before deep interview |
| FR-9 | End-of-interview validation pass | Must | Summary review + gap check + key decisions recap |
| FR-10 | Scalable interview depth | Must | Epic = full, task = lighter, research = quick |

### Non-Functional Requirements

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-1 | Reliability | Zero bugs at launch | No known issues |
| NFR-2 | Portability | Pure skill files, no Python dependencies | Works on any Claude Code install |
| NFR-3 | Composability | Works standalone AND with other plugins | Graceful degradation |
| NFR-4 | Quality | Specs are concise, structured, actionable | No fluff, translates to beads |

### Out of Scope

- ❌ Multi-user collaboration — no shared interviews or handoffs
- ❌ Interview history/persistence — no saving past interviews
- ❌ Custom question authoring UI — no GUI for adding questions
- ❌ Multiple output formats — markdown only (no YAML/JSON)
- ❌ Configuration files — all decisions made at runtime

---

## Technical Design

### Plugin Structure

```
claude-spec-builder/
├── .claude-plugin/
│   └── manifest.json
├── commands/
│   └── interview.md
├── skills/
│   └── interview/
│       ├── SKILL.md
│       └── references/
│           ├── phases.md
│           ├── techniques.md
│           ├── spec-template.md
│           └── templates/
│               ├── feature.md
│               ├── bug.md
│               ├── refactor.md
│               ├── integration.md
│               └── adaptive.md
├── README.md
└── LICENSE (MIT)
```

### Command Interface

```bash
/interview                           # Interactive, asks what to interview about
/interview user-auth                 # Topic
/interview docs/plans/feature.md     # File (auto-detected)
/interview user-auth --voice         # With optional flags
/interview user-auth --template=bug  # Explicit template
```

### Interview Phases

| Phase | Focus | Questions | Adaptable |
|-------|-------|-----------|-----------|
| 0 | Initialization | Analyze input, detect complexity | Always |
| 1 | Problem & Context | Why this? Why now? | 5-10 |
| 2 | Users & Stakeholders | Who's affected? | 5-8 |
| 3 | Scope & Solution | What's in/out? | 8-12 |
| 4 | Technical Design | Architecture, APIs | 10-15 |
| 5 | Constraints & Trade-offs | Time, resources | 5-8 |
| 6 | Edge Cases & Errors | What can go wrong? | 8-12 |
| 7 | Testing & Quality | How to verify? | 5-8 |
| 8 | Success & Metrics | How to measure? | 3-5 |
| 9 | Implementation Planning | What order? | 3-5 |

### TTS Integration

| Scenario | Command | Rationale |
|----------|---------|-----------|
| Interview questions | `/say` | Short, exact wording needed |
| Phase summaries | `/summary-say` | Long, can be condensed |
| Progress announcements | `/say` | Short status updates |
| Final validation recap | `/summary-say` | Long, summarize key points |

**Behavior:**
1. Auto-detect if claude-mlx-tts is installed
2. Ask user once if they want voice mode
3. If TTS server not running, attempt auto-start, else inform user
4. If TTS unavailable, offer install instructions and continue without

### Beads Integration

**Behavior:**
1. Auto-detect if beads is present (`.beads/` directory)
2. At end of interview, ask if user wants to create epic + subtasks
3. Present structure options (phases as subtasks, requirements as subtasks, milestones as subtasks)
4. Use `bd` CLI commands (never modify `.beads/` directly)
5. If bd fails, offer to fix+retry or skip beads

### Output Path Detection

1. Look for existing directories: `specs/`, `docs/specs/`, `docs/`, `prd/`
2. Match project convention
3. Fallback to `docs/specs/` if nothing exists
4. If not writable, inform user and save in current directory

### Progressive Disclosure

- `SKILL.md` loads at startup (~500 tokens)
- `references/` files load only when skill is active
- Templates load only when selected

---

## Edge Cases & Error Handling

| Scenario | Handling |
|----------|----------|
| Mid-interview abandonment | User's responsibility—no state saved |
| File path doesn't exist | State it, stop |
| Missing optional integration | Offer install instructions, continue |
| Output dir not writable | Inform user, fallback to current dir |
| Low-effort answers | Rephrase, ask for details, offer to skip |
| User contradicts themselves | Surface conflict, offer resolution options |
| Very long spec | Executive summary + details, offer to split |
| Beads creation fails | Offer fix+retry or skip bd |
| Pre-requisites fail | Check integrations early, before deep interview |
| TTS server not running | Attempt auto-start, else let user handle |

---

## Testing Strategy

### Testing Approach

Manual dogfooding on real projects.

### Must-Pass Scenarios

| ID | Scenario | Priority |
|----|----------|----------|
| T-1 | Basic interview flow: topic → questions → spec | Critical |
| T-2 | TTS integration: questions voiced, summaries condensed | Critical |
| T-3 | Beads integration: epic + subtasks created | Critical |
| T-4 | File path input: reads file, uses as context | High |
| T-5 | Template selection: correct phases for work type | Medium |

### Spec Quality Criteria

- ✅ Concise—no bloat
- ✅ Structured—clear sections
- ✅ No fluff—every sentence earns its place
- ✅ Edge cases captured—tribal knowledge preserved
- ✅ Actionable—translates directly to beads work items
- ✅ Parallelizable—can spawn sub-agents for independent tasks

### End-of-Interview Validation

1. Show condensed summary → user confirms
2. Gap check → "anything missing?"
3. Read-back key decisions and trade-offs

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Personal usage | Regular use across projects | Self-tracked |
| GitHub stars | Growing interest | GitHub |
| Install count | Adoption | Marketplace (if tracked) |
| Bug reports | Zero | GitHub issues |

### Success Definition

Multi-dimensional:
1. **Personal utility** — used regularly across own projects
2. **Community adoption** — others install and use it
3. **Output quality** — specs are genuinely better than before

### Failure Indicator

Bugs—if it doesn't work reliably, it's a failure. Reliability is non-negotiable.

### Evaluation Timeline

Ongoing, no fixed checkpoint. Continuous improvement mindset.

---

## Implementation Plan

### Approach

End-to-end vertical slice first—get minimal interview + TTS + beads working, then expand.

### Milestones

```
M1: Core (sequential, foundational)
    │
    ├── M2: TTS ──────┐
    ├── M3: Beads ────┼── All can run in parallel
    └── M4: Templates ┘
         │
         └── Ship when all complete
```

| Milestone | Scope | Dependencies |
|-----------|-------|--------------|
| M1: Core | Skill files + command + basic interview flow | None |
| M2: TTS | Voice integration with /say and /summary-say | M1 |
| M3: Beads | Epic/subtask generation via bd CLI | M1 |
| M4: Templates & Ship | All templates, polish, publish | M1 |

### Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Core interview flow doesn't feel right | Medium | High | Iterate heavily on M1, dogfood early |
| TTS integration complexity | Low | Medium | Already have working TTS plugin |
| Beads integration edge cases | Low | Medium | Use bd CLI, well-tested |

### Parallelization

After M1, TTS/Beads/Templates can all run in parallel (potentially via sub-agents).

### Blocking Dependencies

None identified.

---

## Documentation

Minimal README:
- Install instructions
- Basic usage
- One example

Expand docs later if needed based on user feedback.

---

## Open Questions

| ID | Question | Owner | Status |
|----|----------|-------|--------|
| — | None identified | — | — |

---

## Appendix: Key Decisions

### Trade-offs Accepted

| Trade-off | What We Gave Up | What We Gained |
|-----------|-----------------|----------------|
| Flexibility over rigid structure | Consistent 40+ question interviews | Adaptive depth based on scope |
| User responsibility for state | Resume capability | Simplicity, no state management |
| Minimal docs | Comprehensive documentation | Faster ship, iterate based on feedback |
| No config files | Persistent preferences | Zero setup, all runtime decisions |

### Constraints

- **Timeline:** No hard deadline, quality over speed
- **Technical debt:** Zero—ship clean
- **License:** MIT
- **Implementation:** Pure skill files, no Python

### Integration Philosophy

Composable plugin architecture—works standalone, gets superpowers when combined with:
- **beads:** Spec → actionable epic + subtasks
- **claude-mlx-tts:** Voice the interview experience

---

## Change Log

| Date | Author | Change |
|------|--------|--------|
| 2026-01-04 | aperepel + Claude | Initial draft from 65-question interview |
