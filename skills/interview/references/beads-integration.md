# Beads Integration

Guide for integrating generated specifications with Beads issue tracking.

## Detection Logic

### Check for Beads

Silently detect beads availability during Phase 0 (Initialization):

```bash
# 1. Check if .beads/ directory exists in project root
ls -d .beads/ 2>/dev/null

# 2. Check if bd command is available
bd --version 2>/dev/null
```

**Detection rules:**
- If `.beads/` exists AND `bd` command works: Beads is available
- If `.beads/` exists but `bd` fails: Warn user, offer install instructions
- If `.beads/` does not exist: Beads not configured for this project

**Important:** Detection is silent. Only mention beads to the user if:
1. Beads is available and spec generation completed successfully
2. User explicitly asks about beads integration

---

## Post-Interview Offer

After generating the specification, offer beads integration:

```
AskUserQuestion:
  question: "Would you like to create beads work items from this spec?"
  header: "Beads"
  options:
    - "Yes, create epic with subtasks"
    - "Just create epic (no subtasks)"
    - "No, skip beads"
  multiSelect: false
```

**Behavior by response:**
- **"Yes, create epic with subtasks"**: Proceed to Structure Options
- **"Just create epic"**: Create only the epic, skip subtask creation
- **"No, skip beads"**: End interview, no beads interaction

---

## Structure Options

If user wants subtasks, offer structure choices:

```
AskUserQuestion:
  question: "How should the subtasks be structured?"
  header: "Structure"
  options:
    - "Phases as subtasks - each interview phase becomes a task"
    - "Requirements as subtasks - each FR/NFR becomes a task"
    - "Milestones as subtasks - implementation milestones as tasks"
  multiSelect: false
```

### Parsing Logic by Structure

#### Phases as Subtasks
Extract from interview phases completed:
- Phase 1: Problem & Context analysis
- Phase 2: Users & Stakeholders identification
- Phase 3: Scope & Solution definition
- Phase 4: Technical Design
- Phase 5: Constraints & Trade-offs
- Phase 6: Edge Cases & Error Handling
- Phase 7: Testing & Quality strategy
- Phase 8: Success Metrics definition
- Phase 9: Implementation Planning

Skip phases that were condensed or skipped during interview.

#### Requirements as Subtasks
Parse the generated spec's Requirements section:
- Extract each FR-N (Functional Requirement)
- Extract each NFR-N (Non-Functional Requirement)
- Use the requirement text as subtask title

Example extraction:
```
FR-1: /interview command accepts topic or file path
FR-2: 9-phase interview structure with adaptive depth
NFR-1: Zero bugs at launch
```

#### Milestones as Subtasks
Parse the Implementation Plan section:
- Extract each milestone (M1, M2, etc.)
- Use milestone name and scope as subtask details
- Capture dependencies for later `bd dep add`

Example extraction:
```
M1: Core - Skill files + command + basic interview flow
M2: TTS - Voice integration with /say and /summary-say
M3: Beads - Epic/subtask generation via bd CLI
```

---

## Epic Creation

### Determine Priority

Extract priority from spec or use defaults:

1. **From spec**: Look for priority indicators in:
   - Requirements table (Must/Should/Could)
   - Risk Assessment (High impact = higher priority)
   - Timeline constraints mentioned

2. **Priority mapping:**
   | Spec Indicator | Beads Priority |
   |----------------|----------------|
   | Critical/Urgent | P0 |
   | High/Must-have | P1 |
   | Medium/Should-have | P2 (default) |
   | Low/Could-have | P3 |
   | Backlog/Nice-to-have | P4 |

### Create Epic Command

```bash
bd create --title="<spec-title>" --type=epic --priority=<priority>
```

**Example:**
```bash
bd create --title="claude-spec-builder: Adaptive Interview Plugin" --type=epic --priority=P2
```

### Capture Epic ID

The `bd create` command outputs the created issue ID. Capture it for subtask creation:

```
Created issue: bd-a1b2
```

Store `bd-a1b2` as the parent epic ID.

---

## Subtask Creation

### Basic Subtask Command

For each subtask extracted from the chosen structure:

```bash
bd create --title="<task-title>" --type=task --priority=P2 --parent=<epic-id>
```

**Example with phases:**
```bash
bd create --title="Problem & Context Analysis" --type=task --priority=P2 --parent=bd-a1b2
bd create --title="Users & Stakeholders Identification" --type=task --priority=P2 --parent=bd-a1b2
bd create --title="Technical Design" --type=task --priority=P1 --parent=bd-a1b2
```

**Example with requirements:**
```bash
bd create --title="FR-1: /interview command accepts topic or file path" --type=task --priority=P1 --parent=bd-a1b2
bd create --title="FR-2: 9-phase interview structure" --type=task --priority=P1 --parent=bd-a1b2
bd create --title="NFR-1: Zero bugs at launch" --type=task --priority=P2 --parent=bd-a1b2
```

**Example with milestones:**
```bash
bd create --title="M1: Core - Skill files + command + basic flow" --type=task --priority=P1 --parent=bd-a1b2
bd create --title="M2: TTS - Voice integration" --type=task --priority=P2 --parent=bd-a1b2
bd create --title="M3: Beads - Epic/subtask generation" --type=task --priority=P2 --parent=bd-a1b2
```

### Priority Assignment for Subtasks

Assign priorities based on context:

| Structure Type | Priority Logic |
|----------------|----------------|
| Phases | P2 for all (equal importance) |
| Requirements | Must=P1, Should=P2, Could=P3 |
| Milestones | First milestone=P1, others=P2 |

### Fallback: No Parent Option

If `--parent` option is not available in the bd version:

1. Create subtask without parent flag
2. Add reference in description:

```bash
bd create --title="<task-title>" --type=task --priority=P2
# Then update with description mentioning parent
bd update <subtask-id> --description="Part of epic: <epic-id>"
```

---

## Dependency Setting

### When to Set Dependencies

Set dependencies when using **Milestones as subtasks** and the spec indicates:
- Sequential ordering (M1 before M2)
- Blocking relationships ("depends on", "requires", "after")
- Parallelization notes ("can run in parallel" = no dependency)

### Dependency Commands

```bash
# Add dependency: bd-a1b2.2 depends on bd-a1b2.1
bd dep add bd-a1b2.2 bd-a1b2.1

# Example from milestones:
# M2 depends on M1, M3 depends on M1, M4 depends on M1
bd dep add bd-a1b2.2 bd-a1b2.1  # M2 depends on M1
bd dep add bd-a1b2.3 bd-a1b2.1  # M3 depends on M1
bd dep add bd-a1b2.4 bd-a1b2.1  # M4 depends on M1
```

### Dependency Extraction

Parse from Implementation Plan:

```markdown
| Milestone | Scope | Dependencies |
|-----------|-------|--------------|
| M1: Core | ... | None |
| M2: TTS | ... | M1 |
| M3: Beads | ... | M1 |
| M4: Templates | ... | M1 |
```

Map milestone names to created subtask IDs and set dependencies.

---

## Error Handling

### Command Failure Detection

After any `bd` command, check for errors:
- Non-zero exit code
- Error messages in output
- "not found" or "permission denied" indicators

### Error Recovery Flow

```
AskUserQuestion:
  question: "Beads command failed: <error-message>. How would you like to proceed?"
  header: "Error"
  options:
    - "Retry - try the command again"
    - "Skip beads - continue without creating work items"
  multiSelect: false
```

### Common Errors and Handling

| Error | Likely Cause | Recovery |
|-------|--------------|----------|
| `bd: command not found` | bd not installed | Offer install instructions, skip beads |
| `No .beads directory` | Not initialized | Run `bd init`, retry |
| `Invalid parent ID` | Epic creation failed | Retry epic creation first |
| `Permission denied` | File system issue | Check permissions, skip beads |
| `Sync conflict` | Git sync issue | Run `bd sync`, retry |

### Install Instructions (if bd not found)

```
Beads CLI not found. To install:

  curl -sSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash

Then initialize in your project:

  bd init

After installation, you can manually create work items from the spec.
```

### Critical Rule

**Never fail the entire interview because of beads errors.**

The spec has been generated successfully. Beads integration is a bonus feature. If it fails:
1. Inform the user clearly
2. Offer to retry or skip
3. Ensure the generated spec is preserved regardless

---

## Complete Integration Flow

### Happy Path

```
1. [Phase 0] Detect beads: .beads/ exists, bd works
2. [Phase 1-9] Conduct interview
3. [Phase 10] Generate spec to docs/specs/
4. [Post-Interview] Ask: "Create beads work items?"
   → User: "Yes, create epic with subtasks"
5. Ask: "How should subtasks be structured?"
   → User: "Milestones as subtasks"
6. Create epic:
   $ bd create --title="Feature: User Authentication" --type=epic --priority=P1
   → Created: bd-f3c9
7. Create subtasks:
   $ bd create --title="M1: Core auth flow" --type=task --priority=P1 --parent=bd-f3c9
   → Created: bd-f3c9.1
   $ bd create --title="M2: OAuth integration" --type=task --priority=P2 --parent=bd-f3c9
   → Created: bd-f3c9.2
   $ bd create --title="M3: Security hardening" --type=task --priority=P2 --parent=bd-f3c9
   → Created: bd-f3c9.3
8. Set dependencies:
   $ bd dep add bd-f3c9.2 bd-f3c9.1
   $ bd dep add bd-f3c9.3 bd-f3c9.1
9. Report success:
   "Created epic bd-f3c9 with 3 subtasks. View with: bd show bd-f3c9"
```

### Error Path

```
1. [Phase 0] Detect beads: .beads/ exists, bd works
2. [Phase 1-9] Conduct interview
3. [Phase 10] Generate spec to docs/specs/
4. [Post-Interview] Ask: "Create beads work items?"
   → User: "Yes, create epic with subtasks"
5. Create epic:
   $ bd create --title="Feature: User Auth" --type=epic --priority=P1
   → ERROR: "sync conflict detected"
6. Ask: "Beads command failed. Retry or skip?"
   → User: "Retry"
7. Retry epic creation:
   $ bd sync && bd create --title="Feature: User Auth" --type=epic --priority=P1
   → Created: bd-f3c9
8. Continue with subtasks...
```

---

## Summary Report

After successful beads integration, provide a summary:

### Visual Output (Terminal)

```
## Beads Work Items Created

**Epic:** bd-f3c9 - Feature: User Authentication

**Subtasks:**
- bd-f3c9.1: M1: Core auth flow (P1)
- bd-f3c9.2: M2: OAuth integration (P2) [depends on bd-f3c9.1]
- bd-f3c9.3: M3: Security hardening (P2) [depends on bd-f3c9.1]

**Quick commands:**
- View epic: `bd show bd-f3c9`
- List all: `bd list`
- Start work: `bd update bd-f3c9.1 --status in_progress`
```

### Voice Output (TTS)

**Never read bead IDs aloud.** Use human-friendly titles only:

```
# Good - human-friendly
/say "Created epic: User Authentication, with 3 subtasks. Core auth flow at priority 1, OAuth integration and Security hardening at priority 2."

# Bad - reads technical IDs
/say "Created epic bd-f3c9 with subtasks bd-f3c9.1, bd-f3c9.2, and bd-f3c9.3"
```

For command suggestions in voice:
```
# Good - describe the action
/say "Use the show command to view details, or update command to start work."

# Bad - reads literal commands with IDs
/say "Run bd show bd-f3c9 to view the epic."
```

See `references/tts-integration.md` for complete voice content guidelines.

---

## Reference: bd CLI Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `bd --version` | Check installation | `bd --version` |
| `bd create` | Create issue | `bd create --title="..." --type=epic` |
| `bd show <id>` | View issue details | `bd show bd-a1b2` |
| `bd list` | List all issues | `bd list` |
| `bd update <id>` | Update issue | `bd update bd-a1b2 --status done` |
| `bd dep add` | Add dependency | `bd dep add bd-a1b2.2 bd-a1b2.1` |
| `bd sync` | Sync with git | `bd sync` |

### Create Options

```
bd create [options]
  --title=<string>     Issue title (required)
  --type=<type>        epic, task, bug, feature (default: task)
  --priority=<P0-P4>   Priority level (default: P2)
  --parent=<id>        Parent issue ID for subtasks
  --description=<text> Detailed description
```

### Priority Levels

| Priority | Meaning |
|----------|---------|
| P0 | Critical - drop everything |
| P1 | High - do next |
| P2 | Medium - normal priority |
| P3 | Low - do when possible |
| P4 | Backlog - someday/maybe |
