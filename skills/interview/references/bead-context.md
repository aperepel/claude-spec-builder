# Bead Context Loading

Reference for loading rich context when conducting blitz interviews on existing beads.

## Overview

Before asking blitz questions, load comprehensive context about the bead to:
1. Avoid asking questions already answered in the bead
2. Understand the scope and dependencies
3. Tailor questions to what's missing
4. Have full context for making updates

---

## Context Loading Steps

### Step 1: Load Bead Fields via bd show

Execute `bd show <bead-id>` to retrieve all bead fields:

```bash
bd show bd-a3f8
```

**Fields to extract:**

| Field | Purpose | Use In Interview |
|-------|---------|------------------|
| title | Primary identifier | Reference in questions |
| description | Main content | Scan for completeness, file refs |
| type | bug/feature/task/refactor | Drive question focus |
| status | Current state | Context for questions |
| priority | P0-P4 | Understand urgency |
| assignee | Who's working | Context for blockers |
| labels | Categorization | Additional context |
| created | When opened | Timeline context |
| updated | Last activity | Staleness detection |

**Parsing bd show output:**

The output format is:
```
<id>: <title>
Status: <status>
Priority: <priority>
Type: <type>
Created: <datetime>
Created by: <user>
Updated: <datetime>
[Assignee: <user>]
[Labels: <label1>, <label2>]

[Description:
<multi-line description>]

[Depends on (N):
  -> <dep-id>: <dep-title> [priority - status]]

[Blocks (N):
  <- <blocked-id>: <blocked-title> [priority - status]]
```

Extract each field by parsing these patterns.

---

### Step 2: Scan Description for File References

Parse the description to find referenced files:

**Patterns to detect:**

```
# Explicit file paths
/path/to/file.ts
./relative/path/file.py
src/components/Button.tsx

# Markdown links to files
[link text](path/to/file.md)

# Code blocks with filenames
```typescript:src/utils/helper.ts
code here
```

# Import statements
import { X } from './module'
from package import module
```

**File reference extraction logic:**

1. **Absolute paths**: Match `/[^\s]+\.[a-z]+`
2. **Relative paths**: Match `\.\/[^\s]+\.[a-z]+` or `[a-zA-Z][^\s]+\/[^\s]+\.[a-z]+`
3. **Markdown links**: Extract path from `\[.*?\]\((.*?)\)`
4. **Code block headers**: Extract from triple-backtick language:path pattern
5. **Import paths**: Extract from import/from statements (resolve relative to project root)

**For each detected path:**

```
If file exists at path:
  Read file contents (first 200 lines max)
  Store as linked_file: { path, content, exists: true }
Else:
  Store as linked_file: { path, exists: false }
  Note: "Referenced file not found: <path>"
```

**Important:** Don't fail on missing files. Note them for the interview - they may indicate outdated references.

---

### Step 3: Load Related Beads

Extract and load related beads from the bd show output.

**Parent Epic:**

If bead ID has hierarchical structure (e.g., `bd-a3f8.1`):
```bash
# Extract parent from ID
parent_id = bd-a3f8  # Strip the .N suffix
bd show bd-a3f8
```

Store parent context:
- Parent title and description (brief)
- Parent status (is epic in_progress? blocked?)
- Other children of same parent (sibling tasks)

**Blocking Issues (what blocks this bead):**

From "Depends on" section:
```
Depends on (2):
  -> bd-x1y2: Setup database schema [P1 - in_progress]
  -> bd-z3w4: API authentication [P2 - open]
```

For each dependency:
```bash
bd show <dep-id>
```

Store blocker context:
- Blocker title, status, priority
- Is blocker actually blocking? (status != closed)
- Brief description if available

**Blocked By (what this bead blocks):**

From "Blocks" section:
```
Blocks (1):
  <- bd-m5n6: Frontend integration [P2 - open]
```

Store downstream context:
- What's waiting on this bead
- Urgency indicators (high-priority items blocked)

---

## Completeness Detection

Before asking questions, analyze collected context for completeness.

### Completeness Indicators

| Indicator | Check | Weight |
|-----------|-------|--------|
| Description length | > 100 chars | +1 |
| Has acceptance criteria | Contains "acceptance", "criteria", "done when" | +2 |
| Scope defined | Contains "in scope", "out of scope", "includes", "excludes" | +2 |
| Edge cases listed | Contains "edge case", "error handling", "what if" | +1 |
| Has file references | Found linked files | +1 |
| Dependencies mapped | Has "Depends on" entries | +1 |
| Type-specific content | See below | +2 |

### Type-Specific Completeness

**Bug type:**
- Has reproduction steps? ("steps to reproduce", "to reproduce", numbered steps)
- Has expected vs actual? ("expected:", "actual:", "should", "instead")
- Has environment info? ("version", "browser", "OS", "environment")

**Feature type:**
- Has user story format? ("as a", "I want", "so that")
- Has acceptance criteria? (checklist, "done when")
- Has scope boundaries? ("in scope", "out of scope")

**Task type:**
- Has implementation details? ("implement", "create", "add", "update")
- Has definition of done? ("done when", "complete when", "acceptance")
- Has blockers identified? ("blocked by", "depends on", "waiting for")

**Refactor type:**
- Has current state? ("currently", "existing", "before")
- Has target state? ("should be", "target", "after", "goal")
- Has migration path? ("steps", "approach", "plan")

### Completeness Score

Calculate score (0-10):
```
score = sum(indicator_weights)
```

| Score | Assessment | Interview Approach |
|-------|------------|-------------------|
| 8-10 | Well-specified | Quick confirmation: "This looks complete. Anything to add?" |
| 5-7 | Moderate | Focused questions on missing areas |
| 0-4 | Sparse | Full blitz interview (8-10 questions) |

---

## Context Object Structure

After loading, structure context as:

```
bead_context = {
  # Core bead data
  bead: {
    id: "bd-a3f8.1",
    title: "Implement user login",
    description: "...",
    type: "feature",
    status: "open",
    priority: "P2",
    assignee: null,
    labels: ["auth", "mvp"],
    created: "2026-01-04",
    updated: "2026-01-04"
  },

  # Linked files from description
  linked_files: [
    { path: "src/auth/login.ts", exists: true, content: "..." },
    { path: "docs/auth-spec.md", exists: false }
  ],

  # Parent epic (if hierarchical ID)
  parent: {
    id: "bd-a3f8",
    title: "User Authentication Epic",
    status: "in_progress",
    description_summary: "..."
  },

  # What blocks this bead
  blockers: [
    { id: "bd-x1y2", title: "Database schema", status: "in_progress", priority: "P1" }
  ],

  # What this bead blocks
  blocking: [
    { id: "bd-m5n6", title: "Frontend integration", status: "open", priority: "P2" }
  ],

  # Completeness analysis
  completeness: {
    score: 6,
    assessment: "moderate",
    missing: ["acceptance_criteria", "edge_cases"],
    present: ["description", "scope", "dependencies"]
  }
}
```

---

## Error Handling

### Bead Not Found

```bash
bd show bd-invalid
# Error: Issue 'bd-invalid' not found
```

Response to user:
```
Bead 'bd-invalid' not found. Run `bd list` to see available beads.
```

Stop interview immediately - cannot proceed without valid bead.

### Malformed Bead ID

Valid formats:
- `bd-xxxx` (4 hex chars)
- `bd-xxxx.n` (with subtask number)
- `bd-xxxx.n.m` (nested subtask)

If ID doesn't match pattern:
```
Invalid bead ID format: '<id>'. Expected format: bd-xxxx or bd-xxxx.n
```

### bd Command Fails

If `bd show` fails for reasons other than not found:

```bash
bd show bd-a3f8
# Error: sync conflict / permission denied / etc.
```

Options:
1. Attempt `bd sync` and retry
2. Inform user of the error
3. Offer to continue without full context

```
AskUserQuestion:
  question: "bd command failed: <error>. How should we proceed?"
  header: "BD Error"
  options:
    - "Retry after bd sync": Run bd sync, try again
    - "Continue with limited context": Proceed with what we have
    - "Cancel blitz": Stop the blitz interview
  multiSelect: false
```

### File Read Errors

If a linked file exists but can't be read:
- Log the error
- Continue without that file's content
- Note in context: `{ path: "...", exists: true, readable: false, error: "..." }`

---

## Loading Sequence

Complete context loading sequence:

```
1. Validate bead ID format
   -> If invalid: error and stop

2. Run: bd show <bead-id>
   -> If not found: error and stop
   -> If other error: offer recovery options

3. Parse bd show output
   -> Extract all fields into bead object

4. Check for parent (hierarchical ID)
   -> If has parent: run bd show on parent ID
   -> Store parent context

5. Load blockers
   -> For each "Depends on" entry: run bd show
   -> Store blocker context

6. Load blocked items
   -> For each "Blocks" entry: run bd show
   -> Store blocking context

7. Scan description for file references
   -> For each reference: check if file exists
   -> If exists: read first 200 lines
   -> Store linked_files array

8. Calculate completeness score
   -> Check indicators
   -> Determine interview approach

9. Return bead_context object
```

---

## Performance Considerations

### Parallel Loading

After initial `bd show`, load related beads in parallel:
- Parent bead
- All blockers (batch)
- All blocked items (batch)
- File reads (batch)

### Caching

Within a single blitz session, cache bd show results to avoid re-fetching if the same bead is referenced multiple times.

### Limits

| Resource | Limit | Reason |
|----------|-------|--------|
| File content | 200 lines | Avoid loading massive files |
| Related beads | 10 max | Limit context explosion |
| Description scan | 5000 chars | Reasonable description length |

If limits exceeded, truncate and note:
```
Note: File truncated to first 200 lines
Note: Showing first 10 related beads (5 more exist)
```

---

## Integration with Interview

### Using Context in Questions

The loaded context informs question generation:

**If blockers exist and are open:**
```
"I see this depends on <blocker-title> which is still <status>.
 Is that actively being worked on, or is it blocking your progress?"
```

**If file references exist:**
```
"The description references <file-path>. Is the implementation
 approach described there still current?"
```

**If completeness score is low:**
```
"This bead has minimal detail. Let's clarify the key aspects..."
[proceed with full question set]
```

**If completeness score is high:**
```
"This looks well-specified. A few quick questions to confirm:
 1. Is the scope still accurate?
 2. Any blockers or changes since this was written?
 3. Anything you'd add or clarify?"
```

### Updating Bead After Interview

After blitz completes, use collected context to update bead:

```bash
# Update description with new information
bd update <bead-id> --description="<updated-description>"

# Add labels discovered during interview
bd update <bead-id> --labels="<label1>,<label2>"

# Update priority if discussed
bd update <bead-id> --priority=P1

# Add new dependencies discovered
bd dep add <bead-id> <new-blocker-id>
```

---

## Reference: bd show Output Examples

### Feature with Full Details

```
bd-f3c9.2: OAuth integration
Status: open
Priority: P2
Type: feature
Created: 2026-01-03 14:30
Created by: alice
Updated: 2026-01-04 09:15
Assignee: bob
Labels: auth, external-api

Description:
Integrate OAuth 2.0 for Google and GitHub login.

Acceptance criteria:
- User can sign in with Google
- User can sign in with GitHub
- Existing email users can link OAuth accounts

References:
- See src/auth/oauth.ts for current stub
- Design doc: docs/oauth-design.md

Depends on (1):
  -> bd-f3c9.1: Core auth flow [P1 - in_progress]

Blocks (2):
  <- bd-f3c9.3: SSO enterprise [P3 - open]
  <- bd-f3c9.4: Account linking [P2 - open]
```

### Sparse Bug Report

```
bd-x2y3: Login button broken
Status: open
Priority: P1
Type: bug
Created: 2026-01-04 11:00
Created by: user123
Updated: 2026-01-04 11:00

Description:
Can't log in. Button doesn't work.
```

### Task with Dependencies

```
bd-a1b2.3.1: Write unit tests for parser
Status: open
Priority: P2
Type: task
Created: 2026-01-02 16:45
Created by: dev
Updated: 2026-01-03 10:00

Description:
Create comprehensive unit tests for the new parser module.

Files:
- src/parser/index.ts
- tests/parser/ (create)

Depends on (2):
  -> bd-a1b2.3: Implement parser [P1 - in_progress]
  -> bd-a1b2.1: Define grammar [P1 - closed]
```
