# TTS Integration Reference

Integration guide for optional text-to-speech support using the claude-mlx-tts plugin.

## Overview

TTS integration provides voice narration for interview questions and summaries, creating a more conversational interview experience. This is entirely optional - the interview works fully without TTS.

## ⚠️ CRITICAL: Visual Output Is Mandatory

**TTS is SUPPLEMENTARY, not a replacement for visual output.**

Every message that is voiced MUST also be printed to the terminal. Users must be able to:
- Read questions while hearing them
- Refer back to what was asked
- Work in environments where audio isn't possible

**NEVER do this:**
```
# WRONG: Voice only, no visual output
Invoke skill: claude-mlx-tts:say
Args: "What problem are you solving?"
# User hears the question but sees nothing
```

**ALWAYS do this:**
```
# CORRECT: Voice AND visual output
Invoke skill: claude-mlx-tts:say
Args: "What problem are you solving?"

# Then ALSO output the question text visually:
"What problem are you solving?"

# Then use AskUserQuestion for the structured response
```

This applies to ALL interview output: questions, progress indicators, summaries, phase transitions - everything.

## /say vs /summary-say: Decision Guide

This is the **primary decision framework** for choosing between `/say` (verbatim) and `/summary-say` (condensed). Apply these rules consistently.

### Decision Checklist

For any content you're about to voice, ask:

1. **Is exact wording critical?** (e.g., interview question, error message, brief status)
   - YES → Use `/say`
   - NO → Continue to step 2

2. **Is the content 50+ words?**
   - YES → Use `/summary-say`
   - NO → Continue to step 3

3. **Is this a recap, summary, or transition?** (e.g., phase summary, final validation, beads announcement)
   - YES → Use `/summary-say`
   - NO → Use `/say`

### Content Type Rules

| Content Type | Command | Rationale |
|--------------|---------|-----------|
| **Interview questions** | `/say` | Exact wording matters for clarity |
| **Compound/multi-part questions** | `/summary-say` | Condense to key points, user reads full text |
| **Phase transition summaries** | `/summary-say` | Always condense - prevents attention loss |
| **Final validation recap** | `/summary-say` | Focus on key decisions/trade-offs only |
| **Beads announcements** | `/summary-say` | Summarize epic/subtask structure |
| **Short status updates** | `/say` | Brief, exact announcements |
| **Error messages** | `/say` | Exact communication needed |
| **Phase intro (1-2 sentences)** | `/say` | Short enough for verbatim |

### Good vs Bad Examples

**Phase Transition Summary:**
```
# ❌ WRONG: Using /say for long recap (causes attention loss)
/say "Phase 1 complete. We've established that the problem is inconsistent
data validation across three microservices, triggered by the lack of a
shared schema. This impacts data integrity and causes silent failures
in downstream processing. The existing system validates at each service
boundary but uses different rules. Moving to user analysis..."

# ✅ CORRECT: Using /summary-say (condenses key points)
/summary-say "Phase 1 complete. We've established that the problem is
inconsistent data validation across three microservices, triggered by
the lack of a shared schema. This impacts data integrity and causes
silent failures in downstream processing. The existing system validates
at each service boundary but uses different rules. Moving to user analysis..."
# Result: Voice says something like "Phase 1 done. Core issue is inconsistent
# validation across services causing silent failures. Moving to users."
```

**Final Validation Recap:**
```
# ❌ WRONG: Voicing all requirements verbatim
/say "[Reading 15 functional requirements, 8 non-functional requirements,
5 constraints, and 3 out-of-scope items verbatim...]"

# ✅ CORRECT: Focus on key decisions only
/summary-say "Here's the full recap: [all requirements]. Key decisions
we made: choosing JWT over sessions for statelessness, accepting eventual
consistency for performance, and deferring mobile support to phase 2."
# Result: Voice highlights just the trade-offs, user reads full list
```

**Beads Announcement:**
```
# ❌ WRONG: Skipping voice entirely or reading IDs
/say "Created bd-f3c9 with bd-f3c9.1, bd-f3c9.2, bd-f3c9.3"

# ✅ CORRECT: Summarize the structure (no IDs in voice)
/summary-say "Created epic User Authentication with 3 subtasks:
Core auth flow at priority 1, OAuth integration at priority 2,
and Security hardening at priority 2. The OAuth subtask depends
on the core auth flow being completed first."
# Result: Voice gives human-friendly overview of what was created
```

**Interview Question:**
```
# ❌ WRONG: Using /summary-say for a question (loses exact wording)
/summary-say "What authentication method should we use?"

# ✅ CORRECT: Using /say for exact question (or short intro + visual)
Bash: say.sh "About authentication..." &
AskUserQuestion: "Which authentication method should we use?"
```

### Anchor Points: Always Voice

These content types are **anchor points** that always get voiced, even in rapid-fire mode:
- Phase transition summaries (via `/summary-say`)
- Final validation recap (via `/summary-say`, key decisions focus)
- Beads creation announcements (via `/summary-say`)
- Interview start/end announcements (via `/say`)

Regular questions follow pacing rules and may be skipped in rapid-fire mode, but anchors never skip.

### Fallback Behavior

If `/summary-say` fails (skill error, timeout, etc.):

1. **Do NOT fall back to `/say`** (would read verbatim, causing the problem we're avoiding)
2. **Voice a brief notice instead:**
   ```
   /say "See the summary below."
   ```
3. **Display the full text visually** (always required anyway)
4. **Continue the interview** - don't retry or block

## TTS Detection Logic

### How to Detect TTS Availability

**Use the Skill tool to invoke `/tts-status`. That's it.**

```
Invoke skill: claude-mlx-tts:tts-status
```

**DO NOT**:
- ❌ Use `curl` to probe HTTP endpoints
- ❌ Guess ports or health check paths
- ❌ Run shell commands to detect TTS
- ❌ Check for running processes
- ❌ Try to introspect available_skills manually

**Interpretation of results:**

| Result | Meaning | Next Step |
|--------|---------|-----------|
| Skill runs, says "running" | TTS ready | Use `/say` and `/summary-say` |
| Skill runs, says "not running" | Plugin installed, server stopped | Invoke `/tts-start`, then retry |
| Skill runs, says "not initialized" | Plugin installed, needs setup | Tell user to run `/tts-init` |
| Skill doesn't exist / error | Plugin not installed | Inform user: "TTS not available, continuing without voice" |

### Complete Detection Flow

```
1. Invoke skill: claude-mlx-tts:tts-status

2. If skill executes successfully:
   - Parse output for "running", "not running", or "not initialized"
   - If "running" → TTS is available, proceed with voice
   - If "not running" → invoke claude-mlx-tts:tts-start, then retry
   - If "not initialized" → inform user, offer to continue without voice

3. If skill fails or doesn't exist:
   - TTS plugin is not installed
   - Inform user: "TTS not available, continuing without voice"
```

### Auto-Start Server If Needed

If status shows "not running":

```
Invoke skill: claude-mlx-tts:tts-start
```

Wait for confirmation, then invoke `/tts-status` again to verify.

If start fails (e.g., model not downloaded), inform the user:
> "TTS server couldn't start. You may need to run /tts-init first to install dependencies. Would you like to continue without voice?"

## Voice Mode Activation

### User Preference Flow

At interview initialization (Phase 0), after detecting TTS availability:

1. **Check for --voice flag**: If the user invoked with `--voice` flag, enable voice mode automatically.

2. **Ask user preference**: If no flag and TTS is available, use AskUserQuestion:

```
question: "I can read questions aloud using text-to-speech. Would you like voice mode?"
header: "Voice Mode"
options:
  - "Yes, enable voice": Enable TTS for questions and summaries
  - "No, text only": Proceed with text-only interview
multiSelect: false
```

3. **Remember choice**: Store the voice mode preference for the entire interview duration. Do not ask again.

### State Tracking

Maintain voice mode state:
```
voice_mode_enabled: boolean
tts_verified: boolean (set after pre-flight check)
tts_available: boolean (plugin installed and server running)
```

## Question Voicing

### Using /say for Interview Questions

For each interview question, invoke the say skill with the exact question text:

```
Invoke skill: claude-mlx-tts:say
Args: "<exact question text>"
```

**Important guidelines**:

1. **Visual output is MANDATORY**: After invoking TTS, you MUST also display the question text visually. TTS adds voice; it does not replace text output.

2. **Exact wording**: Pass the complete question as you want it spoken. The TTS will read it verbatim.

3. **Timing**: Invoke TTS before displaying the question text. This ensures voice starts playing as the user reads.

4. **No batching with voice**: When voice mode is enabled, ask ONE question at a time:
   - Never batch multiple questions into a single AskUserQuestion call
   - Voice a short intro, then show the single question
   - This ensures voice and visual stay synchronized
   - See "Adaptive Pacing" section for details

5. **AskUserQuestion integration**: When using AskUserQuestion tool, invoke TTS with the question text before presenting the structured choice. The AskUserQuestion provides the visual output.

### Example Question Voicing

For a regular question (short intro, not full text):
```
# Voice short intro in background
Bash: say.sh "About the core problem..." &

# Immediately show full question
AskUserQuestion:
  question: "What problem are you trying to solve?"
  header: "Problem"
  options: [...]
```

For phase transition (anchor - always voiced):
```
# Voice phase intro in background
Bash: say.sh "Now let's talk about users and stakeholders." &

# Show full question
AskUserQuestion:
  question: "Who will be the primary users of this feature?"
  header: "Users"
  options: [...]
```

For rapid-fire mode (voice skipped):
```
# No TTS call - user is responding quickly

# Show question with indicator
🔇 Who will be the primary users of this feature?
   ○ Internal team
   ○ End customers
   ○ Both
```

## Summary Voicing

### Using /summary-say for Long Content

**See the Decision Guide above for the primary rules.** This section provides implementation details.

For longer text that would be tedious to hear verbatim, use summary-say:

```
Invoke skill: claude-mlx-tts:summary-say
Args: "<long text to summarize and speak>"
```

**Key principle**: `/summary-say` condenses text to ~30% of original length, highlighting key points. The full text is always shown visually - voice provides the digest.

**Mandatory uses of /summary-say** (never use /say for these):

1. **Phase transition summaries**: Always condense what was learned
   ```
   # Pass the full summary - it will be condensed automatically
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Phase 1 complete. We've established that the problem is [X], triggered by [Y], with impact on [Z]. The existing system does [A] but lacks [B]. Moving to user analysis..."
   ```

2. **Final validation recap**: Focus voice on key decisions/trade-offs
   ```
   # Voice will extract decisions, user reads full requirements visually
   Invoke skill: claude-mlx-tts:summary-say
   Args: "[Full recap including requirements, constraints, decisions...]"
   ```

3. **Beads announcements**: Summarize the structure created
   ```
   # Voice gives overview, visual shows IDs for reference
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Created epic [title] with [N] subtasks: [list with priorities and dependencies]"
   ```

4. **Spec summary**: After generating the spec file
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Your specification has been saved. It covers [key sections] with [X] functional requirements and [Y] non-functional requirements. Key decisions include [major trade-offs]."
   ```

5. **Gap identification**: When surfacing missing information
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Looking at what we've covered, there are a few areas that might need more detail: [list of gaps]. Would you like to address any of these?"
   ```

### Quick Reference: /say vs /summary-say

| Content Type | Command | Key Rule |
|--------------|---------|----------|
| Single interview question | `/say` | Exact wording critical |
| Compound/multi-part question | `/summary-say` | Condense, user reads full |
| Phase intro (1-2 sentences) | `/say` | Short enough for verbatim |
| **Phase summary** | **`/summary-say`** | **ALWAYS** - prevents attention loss |
| **Final recap** | **`/summary-say`** | **ALWAYS** - focus on decisions |
| **Beads announcement** | **`/summary-say`** | **ALWAYS** - summarize structure |
| Short status update | `/say` | Brief, exact |
| Error message | `/say` | Exact communication |
| 50+ words of any type | `/summary-say` | Length threshold applies |

## Adaptive Pacing

Adapt voicing based on user response timing. The goal: skip voice for users in rapid-fire mode, but always voice anchor questions.

### Core Approach: Sequential Questions

**CRITICAL: Ask ONE question at a time with voice before each.**

```
# For each question:
say.sh "About timing..." &          # Fire-and-forget TTS
AskUserQuestion(single question)    # Show visual immediately
# Voice trails visual by ~100-200ms (natural, like a narrator)
# Measure response time
# Apply pacing logic for next question
```

**NO batching.** Never put multiple questions in a single AskUserQuestion call when voice is enabled.

### Anchor vs Non-Anchor Questions

| Question Type | Always Voice? | Rationale |
|---------------|---------------|-----------|
| Phase intro/transition | ✅ Yes | Orient user, reset pacing state |
| Recap summaries | ✅ Yes | Important checkpoint |
| Regular questions | Pacing-dependent | Skip if rapid-fire |
| Follow-up clarifications | Pacing-dependent | Skip if rapid-fire |

### Response Timing

**Quick response threshold**: Under 5 seconds from AskUserQuestion invocation.

```
question_start_time = now()
answer = AskUserQuestion(...)
response_time = now() - question_start_time

if response_time < 5.0:
    rapid_fire_mode = true
```

### Pacing State Machine

```
At interview/phase start:
  rapid_fire_mode = false  # Always voice at start

After each user response:
  response_time = time_since_question_shown

  if response_time < 5 seconds:
    rapid_fire_mode = true
    # Skip voice for next non-anchor question

  # Rapid-fire resets at phase transitions
  # (handled by always voicing phase intros)
```

### Visual Feedback

When voice is skipped due to rapid-fire mode, show indicator:

```
🔇 What specific timing threshold defines "quick"?
   ○ Under 5 seconds
   ○ Under 10 seconds
   ○ Adaptive
```

The 🔇 appears before the question text when TTS would have played but was skipped.

### Voice Content

Keep voice intros **short** (~1-2 seconds):

| Instead of (too long) | Say this |
|----------------------|----------|
| "What time threshold defines 'quick response'? For example, under 5 seconds, under 10 seconds, or should it adapt based on question complexity?" | "About timing thresholds..." |
| "How should the pacing state reset? When should it go back to voicing questions?" | "About reset behavior..." |

User reads full question visually. Voice provides context/framing only.

## Error Handling

### TTS Unavailable at Start

If TTS plugin is not installed when voice mode is requested:

```
"Voice mode requires the claude-mlx-tts plugin, which isn't currently installed.

To install it, you can run:
  /tts-init

This will download the required model (approximately 4GB) and set up text-to-speech.

For now, I'll continue with the text-only interview. You can enable voice mode next time after installing the plugin."
```

Then proceed with the interview normally.

### TTS Fails Mid-Interview

If a TTS invocation fails during the interview:

1. **Log the failure silently** (don't interrupt the flow)

2. **After 2-3 consecutive failures**, inform the user:
   ```
   "Voice mode seems to be having issues. I'll continue the interview in text-only mode. The questions and content remain the same."
   ```

3. **Disable TTS for remainder** of the interview (set voice_mode_enabled = false)

4. **Continue immediately** with the current question in text

### Server Stops Mid-Interview

If TTS was working but server becomes unavailable:

```
"The TTS server appears to have stopped. I'll continue with text-only for the remaining questions. You can restart the server with /tts-start if you'd like to re-enable voice."
```

### Recovery After Failure

Do not attempt to auto-restart TTS after failure. The user can manually:
- Run `/tts-status` to check state
- Run `/tts-start` to restart
- The interview will continue without voice

## Pre-flight Check

### Purpose

Before committing to voice mode, verify TTS actually works with a brief test.

### Implementation

After user enables voice mode (or --voice flag):

1. **Invoke test message**:
   ```
   Invoke skill: claude-mlx-tts:say
   Args: "Starting interview."
   ```

2. **Check result**:
   - If successful: Set `tts_verified = true`, proceed with voice mode
   - If failed: Offer fallback

3. **Fallback prompt** (if pre-flight fails):
   ```
   question: "Voice mode test failed. How would you like to proceed?"
   header: "TTS Issue"
   options:
     - "Continue without voice": Proceed with text-only interview
     - "Try to fix TTS": Pause interview to troubleshoot
   multiSelect: false
   ```

4. **If user chooses to fix**: Provide troubleshooting steps
   ```
   "To troubleshoot TTS:
   1. Check status: /tts-status
   2. If not initialized: /tts-init (downloads ~4GB model)
   3. If not running: /tts-start
   4. Once ready, let me know and we can restart the interview with voice."
   ```

### Skip Pre-flight When

- User explicitly chose text-only mode
- TTS plugin not installed
- --voice flag not provided and user declined voice mode

## Integration with Interview Flow

### Phase 0: Initialization

```
1. Check TTS plugin availability
2. If available, check/start server
3. Ask user about voice mode (or check --voice flag)
4. If voice mode enabled, run pre-flight check
5. If pre-flight passes, voice mode is active
6. Announce: "Let's begin the interview." (via TTS if enabled)
```

### During Interview (Phases 1-9)

**CRITICAL: Visual output is MANDATORY. TTS is supplementary.**

```
For each question or question batch:
  If voice_mode_enabled and tts_verified:
    Invoke /say with question text
  ALWAYS display question text visually (regardless of TTS state)
  Use AskUserQuestion tool for structured response
  Wait for user response

For phase transitions:
  If voice_mode_enabled and tts_verified:
    Invoke /summary-say with phase summary
  ALWAYS display summary text visually (regardless of TTS state)
```

**Remember**: TTS adds voice ON TOP OF visual output. It never replaces it.

### Phase 10: Validation & Output

```
1. If voice_mode_enabled: /summary-say with full requirements recap
2. Display recap text
3. Ask confirmation questions (voice with /say if enabled)
4. Generate spec
5. If voice_mode_enabled: /summary-say with spec overview
6. Announce completion
```

## Command Line Integration

### The --voice Flag

When interview skill is invoked with `--voice`:

```
/interview --voice [topic]
```

Behavior:
- Skip the voice mode preference question
- Immediately check TTS availability
- Auto-start server if needed
- Run pre-flight check
- If any step fails, inform user and offer text-only fallback

### Examples

```
/interview --voice            # Voice mode, ask for topic
/interview --voice auth flow  # Voice mode for "auth flow" topic
/interview                    # Ask about voice mode preference
```

## State Summary

Variables to track during interview:

| Variable | Type | Description |
|----------|------|-------------|
| `tts_plugin_available` | boolean | claude-mlx-tts plugin is installed |
| `tts_server_running` | boolean | TTS server is active and responding |
| `voice_mode_requested` | boolean | User asked for voice (flag or prompt) |
| `voice_mode_enabled` | boolean | Voice mode is active for this interview |
| `tts_verified` | boolean | Pre-flight check passed |
| `tts_failure_count` | number | Consecutive TTS failures (disable after 2-3) |

## Voice Content Guidelines

### Technical IDs: Use Human-Friendly Titles

**Never read technical IDs aloud.** Bead IDs like `bd-f3c9` or `bd-a1b2.3` are for visual reference only. When voicing announcements, use human-friendly titles instead.

| Visual Output (shows ID) | Voice Output (no ID) |
|--------------------------|----------------------|
| Created epic: bd-f3c9 - User Authentication | "Created epic: User Authentication" |
| Created: bd-f3c9.1 - Core auth flow | "Created subtask: Core auth flow" |
| View with: `bd show bd-f3c9` | "You can view the epic details with the show command" |
| Depends on bd-f3c9.1 | "Depends on Core auth flow" |

### Examples

**Beads creation announcement:**
```
# Visual (includes IDs for copy/paste)
Created epic bd-f3c9 with 3 subtasks:
- bd-f3c9.1: Core auth flow (P1)
- bd-f3c9.2: OAuth integration (P2)
- bd-f3c9.3: Security hardening (P2)

# Voice (human-friendly)
/say "Created epic User Authentication with 3 subtasks: Core auth flow, OAuth integration, and Security hardening."
```

**Blitz mode bead loading:**
```
# Visual
Loading bead bd-a3f8.2: Implement caching layer

# Voice
/say "Loading: Implement caching layer"
```

**Dependency announcements:**
```
# Visual
bd-f3c9.2 depends on bd-f3c9.1

# Voice
/say "OAuth integration depends on Core auth flow"
```

### When to Include IDs

The only time to voice technical identifiers:
- **Never for bead IDs** - always use titles
- **Version numbers** - "Version 2.3" is fine to voice
- **Priority levels** - "Priority 1" or "P1" is acceptable (brief and meaningful)

### Implementation Notes

When announcing beads operations:
1. Store the title alongside the ID during creation
2. Build voice content using titles, not IDs
3. Visual output shows full `ID: Title` format for reference
4. Voice output uses only the title portion

---

## Best Practices

1. **Never block on TTS**: If TTS fails, continue immediately with text
2. **Keep TTS text natural**: Write questions as you'd speak them
3. **Pause between batched questions**: Give listener time to process
4. **Summarize, don't recite**: Use `/summary-say` for phase summaries, recaps, and beads announcements - NEVER `/say`
5. **Fail gracefully**: Users should barely notice if TTS stops working
6. **Don't over-voice**: Status messages and minor updates don't need TTS
7. **Match tone**: Keep TTS text conversational, not robotic
8. **No technical IDs in voice**: Use human-friendly titles, never bead IDs
9. **50-word threshold**: If content exceeds 50 words, use `/summary-say` regardless of type
10. **Anchor points always voice**: Phase summaries, final recap, and beads announcements are anchors - voice them even in rapid-fire mode
11. **Fallback gracefully**: If `/summary-say` fails, say "See the summary below" - never fall back to `/say` for long content
