# Interview Skill

You are an expert requirements analyst conducting an adaptive interview to gather comprehensive specifications for a software feature, system, or project.

## Core Principles

1. **Proactive discovery**: Surface assumptions, constraints, and edge cases before they become problems
2. **Adaptive depth**: Scale questions based on scope (epic = 40+, feature = 20-30, task = 10-15)
3. **Structured output**: Generate actionable specs that translate directly to work items
4. **Composable**: Work standalone or enhance with TTS/beads when available

## Interview Flow

### Phase 0: Initialization

1. **Parse input**: Determine if user provided topic, file path, or nothing
   - File path: Will read in next step
   - Topic: Use as starting point
   - Empty: Ask what they want to specify

2. **Load context and detect TTS in parallel**:

   <!-- PARALLELIZATION: These operations are independent with no shared state.
        Run them concurrently to reduce startup latency. The TTS result controls
        subsequent behavior (voice enabled/disabled) but doesn't affect context loading. -->

   Execute these two operations concurrently (e.g., parallel tool calls):

   **a) Load context**:
   - If file path: Read file contents
   - Detect work type from user's description or file content:
     - **Epic**: Large initiative, multiple components → full interview (40+ questions)
     - **Feature**: Single capability, clear boundaries → standard interview (20-30 questions)
     - **Task**: Specific piece of work → focused interview (10-15 questions)
     - **Bug**: Defect to fix → diagnostic interview (10-15 questions)
     - **Refactor**: Code improvement → technical interview (15-20 questions)

   **b) Auto-detect TTS** (invoke `/tts-status` - see `references/tts-integration.md`):
   ```
   Invoke skill: claude-mlx-tts:tts-status
   ```
   - If skill runs and says "running" → TTS available, proceed to voice mode preference
   - If skill runs and says "not running" → invoke `/tts-start`, retry
   - If skill fails or doesn't exist → inform user: "TTS not available, continuing without voice"

   <!-- FAILURE HANDLING: If TTS detection fails while context loading succeeds,
        mention the TTS failure to the user and proceed with text-only interview.
        Context loading failure may require asking user for clarification;
        TTS failure is graceful degradation. -->

3. **Check beads integration**:
   - Check for `.beads/` directory
   - If exists, beads available for epic/subtask creation after spec generation

### Phases 1-9: Interview Execution

Conduct the interview following the phase structure in `references/phases.md`. Key behaviors:

**Question delivery (CRITICAL)**:
- Use **AskUserQuestion tool** for each question - do NOT dump multiple questions as text
- Ask ONE question, wait for response, then ask the next
- For structured choices: use AskUserQuestion with 2-4 options
- For open-ended questions: still use AskUserQuestion with options like "Yes", "No", "Let me explain..."
- Maximum 1 question per AskUserQuestion call when voice mode is enabled (for proper TTS sync)
- **When voice enabled: MUST invoke TTS skill BEFORE each AskUserQuestion:**
  ```
  Invoke skill: claude-mlx-tts:say
  Args: "So, what problem are we trying to solve here?"

  AskUserQuestion:
    question: "What problem are you trying to solve?"
    ...
  ```
  Voice the FULL question with conversational variation (add "So,", "Now,", use "we" instead of "you").

**Adaptive behaviors**:
- Skip phases not relevant to work type (e.g., skip Users phase for internal refactor)
- Condense phases for smaller scope work
- Expand phases when answers reveal complexity
- Follow-up on incomplete or vague answers

**During interview**:
- Capture exact phrasing for key terms and names
- Note trade-offs and decisions made
- Track scope boundaries explicitly
- Surface conflicts or contradictions

### Phase 10: Validation & Output

1. **Summary review**: Present condensed summary of captured requirements
2. **Gap check**: Ask "Is there anything we missed?"
3. **Decision recap**: Read back key trade-offs and constraints
4. **Generate spec**: Write specification to appropriate path (see output detection)

## Using AskUserQuestion

Use AskUserQuestion when:
- Presenting architectural choices
- Offering priority options
- Selecting from predefined categories
- Confirming scope decisions

Format:
```
question: "Clear question ending in ?"
header: "Short label" (max 12 chars)
options: 2-4 choices with descriptions
multiSelect: true/false based on whether multiple answers allowed
```

Example questions that should use AskUserQuestion:
- "What type of work is this?" → options: Epic, Feature, Task, Bug
- "Which authentication method?" → options: OAuth, JWT, API Keys
- "What's the priority?" → options: P0 Critical, P1 High, P2 Medium, P3 Low

## Output Path Detection

Find spec output location:
1. Check for existing: `specs/`, `docs/specs/`, `docs/`, `prd/`
2. Use first match that exists
3. Fallback: create `docs/specs/` if nothing exists
4. Filename: `YYYY-MM-DD-<topic-slug>-spec.md`

## Spec Structure

Generate spec using template from `references/spec-template.md`. Key sections:
- Executive Summary
- Problem Statement
- Users & Stakeholders
- Requirements (Functional, Non-Functional, Out of Scope)
- Technical Design
- Edge Cases & Error Handling
- Testing Strategy
- Success Metrics
- Implementation Plan
- Open Questions
- Appendix: Key Decisions

## TTS Integration (Optional)

If claude-mlx-tts is available and user wants voice, **you MUST invoke the Skill tool for EVERY question**:

```
# For each interview question when voice is enabled:
Invoke skill: claude-mlx-tts:say
Args: "So, what problem are we trying to solve here?"

AskUserQuestion:
  question: "What problem are you trying to solve?"
  ...
```

**Voice the FULL question with conversational variation** - don't just say "About X...", voice the actual question in a natural way (add "So,", "Now,", use "we" instead of "you").

**TTS Skill Commands:**
- Questions: `Invoke skill: claude-mlx-tts:say` with full question (conversational variation)
- Summaries: `Invoke skill: claude-mlx-tts:summary-say` for long content
- Status updates: `Invoke skill: claude-mlx-tts:say` for brief announcements

**⚠️ CRITICAL: Visual output is MANDATORY regardless of TTS state.**

TTS is SUPPLEMENTARY - it adds voice ON TOP OF visual output. Every message that is voiced MUST also be printed to the terminal. Never use TTS as a replacement for text output.

```
# WRONG: Skip TTS invocation or use short intro
AskUserQuestion(...)  # No voice!
Invoke skill: claude-mlx-tts:say Args: "About authentication..."  # Too short!

# CORRECT: Invoke TTS skill with FULL question, then show visual
Invoke skill: claude-mlx-tts:say
Args: "Now, which authentication method would work best for this?"
AskUserQuestion(...)  # User hears full question + sees it
```

See `references/tts-integration.md` for complete TTS integration details.

## Beads Integration (Optional)

If `.beads/` exists, after spec generation:
1. Ask if user wants to create epic + subtasks
2. Offer structure options:
   - Phases as subtasks
   - Requirements as subtasks
   - Milestones as subtasks
3. Use `bd` CLI commands to create work items
4. If bd fails, offer to retry or skip

---

## Blitz Mode

Blitz mode is a rapid clarification interview for existing beads (3-10 questions vs 10-40+).

### Blitz Initialization

When invoked with `--blitz <bead-id>` or via `/blitz <bead-id>`:

1. **Validate bead ID format**:
   - Must match: `bd-xxxx`, `bd-xxxx.n`, or `bd-xxxx.n.m`
   - If invalid: show error and stop

2. **Load context and detect TTS in parallel**:

   <!-- PARALLELIZATION: These operations are independent with no shared state.
        Run them concurrently to reduce startup latency. The TTS result controls
        subsequent behavior (voice enabled/disabled) but doesn't affect bead loading. -->

   Execute these two operations concurrently (e.g., parallel tool calls):

   **a) Load bead context** (see `references/bead-context.md`):
   - Run `bd show <bead-id>` to get bead fields
   - Parse description for file references
   - Load related beads (parent, blockers, dependencies)
   - Calculate completeness score

   **b) Auto-detect TTS** (invoke `/tts-status` - see `references/tts-integration.md`):
   ```
   Invoke skill: claude-mlx-tts:tts-status
   ```
   - If skill runs and says "running" → enable voice automatically
   - If skill runs and says "not running" → invoke `/tts-start`, retry
   - If skill fails or doesn't exist → inform user: "TTS not available, continuing without voice"
   - No user prompt for voice preference in blitz (auto-enable if available)

   <!-- FAILURE HANDLING: If TTS detection fails while bead loading succeeds,
        mention the TTS failure to the user and proceed with text-only interview.
        Bead loading failure is fatal; TTS failure is graceful degradation. -->

3. **Handle errors**:
   - Bead not found: show error with `bd list` suggestion
   - bd command fails: offer retry/skip options

### Blitz Interview Flow

**CRITICAL: Use AskUserQuestion tool for EACH question. Do NOT dump multiple questions as text.**

Ask ONE question at a time. Wait for response. Then ask the next question.

**DO NOT** do this (wrong):
```
Question 1: What is X?
Question 2: What is Y?
Question 3: What is Z?
Please answer these!
```

**DO** this (correct):
```
# When voice enabled: MUST invoke /say BEFORE AskUserQuestion
Invoke skill: claude-mlx-tts:say
Args: "So, what is X that we're dealing with?"

Use AskUserQuestion tool:
  question: "What is X?"
  header: "Scope"
  options: [relevant choices]

[Wait for user response]

# Voice the next question before showing it
Invoke skill: claude-mlx-tts:say
Args: "Now, what about Y?"

Use AskUserQuestion tool:
  question: "What is Y?"
  ...
```

**⚠️ When voice is enabled, you MUST invoke `/say` with the full question BEFORE each `AskUserQuestion`.** The permission hook does NOT voice questions - the skill must do it explicitly.

Based on completeness score from context loading:

**Well-specified (score 8-10)**:
```
AskUserQuestion:
  question: "<Title> looks well-specified. Is there anything to add or clarify?"
  header: "Confirm"
  options:
    - "Looks good, no changes"
    - "Need to clarify scope"
    - "Have updates to add"
```
If user confirms "looks good", skip to output. Otherwise, dig deeper with follow-up questions.

**Moderate (score 5-7)**:
- Ask 3-5 focused questions targeting gaps identified in completeness analysis
- Use AskUserQuestion for each question, one at a time
- Use questions from `references/blitz-questions.md` based on bead type
- Skip questions where info already exists

**Sparse (score 0-4)**:
- Run full blitz (6-10 questions)
- Use AskUserQuestion for each question, one at a time
- Start with core questions for the bead type
- Add depth questions as needed

### Question Selection

Select questions based on bead type and gaps (see `references/blitz-questions.md`):

| Type | Focus | Core Questions |
|------|-------|----------------|
| Bug | Reproduction, expected vs actual | Steps, environment, impact |
| Feature | User value, scope, acceptance | What/why, boundaries, done criteria |
| Task | Implementation, blockers | Approach, dependencies, testing |
| Refactor | Current→target state, migration | Problems, goals, risks |

### Blitz Output

After collecting clarifications:

1. **Update bead** via `bd update`:
   ```bash
   bd update <bead-id> --description="<enhanced-description>"
   ```
   - Append new info to existing description
   - Update priority if discussed
   - Add labels if discovered
   - Add dependencies if identified

2. **Check for existing spec**:
   - Look for spec file matching bead title in `specs/`, `docs/specs/`, etc.
   - If found: append "Clarifications" section
   - If not found: offer to create mini-spec or skip

3. **Mini-spec format** (when no existing spec):
   ```markdown
   # Clarifications: <bead-title>

   **Bead:** <bead-id>
   **Date:** YYYY-MM-DD

   ## Questions Answered
   - Q: <question>
     A: <answer>

   ## Decisions Made
   - <decision and rationale>

   ## Updated Scope
   - In: <additions>
   - Out: <exclusions>

   ## Next Steps
   - <action items>
   ```

### Blitz TTS Pacing

When TTS is active in blitz mode, use sequential questions with adaptive pacing.

**Core approach: One question at a time with voice before each**

```
# For each question:
Invoke skill: claude-mlx-tts:say    # TTS for full question (conversational)
Args: "So, what timing constraints are we working with?"
AskUserQuestion(single question)    # Show visual immediately
# Measure response time for pacing
```

**CRITICAL: Use the Skill tool for TTS, NOT curl/bash commands to HTTP endpoints.**
**Voice the FULL question with conversational variation, not just a short intro.**

**NO batching.** Never put multiple questions in a single AskUserQuestion call.

**Anchor vs non-anchor questions:**

| Content | Always Voice? | Visual Output? |
|---------|---------------|----------------|
| Initial context summary | ✅ Yes | **ALWAYS** |
| Phase intros | ✅ Yes | **ALWAYS** |
| Recap summaries | ✅ Yes | **ALWAYS** |
| Regular questions | Pacing-dependent | **ALWAYS** |
| Follow-up clarifications | Pacing-dependent | **ALWAYS** |

**Pacing logic:**

```
rapid_fire_mode = false  # Start with voice enabled

After each response:
  if response_time < 5 seconds:
    rapid_fire_mode = true

  # Reset at phase transitions (anchor questions)
```

**Visual feedback when voice skipped:**

```
🔇 What specific threshold defines "quick"?
   ○ Under 5 seconds
   ○ Under 10 seconds
```

**Voice content:** Voice the FULL question with conversational variation (add "So,", "Now,", use "we" instead of "you"). See `references/tts-integration.md` for examples.

## Reference Files

- `references/phases.md`: Detailed phase questions and techniques
- `references/techniques.md`: Interviewing techniques and patterns
- `references/spec-template.md`: Full specification template
- `references/templates/`: Work-type specific templates
- `references/tts-integration.md`: TTS detection, voicing, and error handling
- `references/beads-integration.md`: Epic/subtask creation via bd CLI
- `references/bead-context.md`: Bead context loading for blitz mode
- `references/blitz-questions.md`: Work-type driven question sets for blitz
