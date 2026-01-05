# TTS Integration Reference

Integration guide for optional text-to-speech support using the claude-mlx-tts plugin.

## Overview

TTS integration provides voice narration for interview questions and summaries, creating a more conversational interview experience. This is entirely optional - the interview works fully without TTS.

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
| Skill doesn't exist / error | Plugin not installed | Proceed without TTS (silently) |

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
   - Proceed silently without TTS (don't mention it to user)
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

1. **Exact wording**: Pass the complete question as you want it spoken. The TTS will read it verbatim.

2. **Timing**: Invoke TTS before displaying the question text. This ensures voice starts playing as the user reads.

3. **Question batching**: When asking 2-3 questions together:
   - Combine into a single TTS invocation with natural pauses
   - Example: "First question: What problem are you solving? ... And second: Why is this important now?"
   - Or invoke TTS once per question with a brief pause between

4. **AskUserQuestion integration**: When using AskUserQuestion tool, invoke TTS with the question text before presenting the structured choice.

### Example Question Voicing

For a simple question:
```
Invoke skill: claude-mlx-tts:say
Args: "What problem are you trying to solve?"
```

For phase transition with question:
```
Invoke skill: claude-mlx-tts:say
Args: "Now let's talk about users and stakeholders. Who will be the primary users of this feature?"
```

## Summary Voicing

### Using /summary-say for Long Content

For longer text that would be tedious to hear verbatim, use summary-say:

```
Invoke skill: claude-mlx-tts:summary-say
Args: "<long text to summarize and speak>"
```

**Use /summary-say for**:

1. **Phase transitions**: When summarizing what was learned before moving on
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Phase 1 complete. We've established that the problem is [X], triggered by [Y], with impact on [Z]. The existing system does [A] but lacks [B]. Moving to user analysis..."
   ```

2. **Final validation recap**: Before asking for confirmation
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "[Full recap of all captured requirements across all phases...]"
   ```

3. **Spec summary**: After generating the spec file
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Your specification has been saved. It covers [key sections] with [X] functional requirements and [Y] non-functional requirements. Key decisions include [major trade-offs]."
   ```

4. **Gap identification**: When surfacing missing information
   ```
   Invoke skill: claude-mlx-tts:summary-say
   Args: "Looking at what we've covered, there are a few areas that might need more detail: [list of gaps]. Would you like to address any of these?"
   ```

### When to Use /say vs /summary-say

| Content Type | Use | Reason |
|--------------|-----|--------|
| Single question | /say | Exact wording matters |
| 2-3 questions | /say | Still brief enough for verbatim |
| Phase intro (1-2 sentences) | /say | Short, exact phrasing |
| Phase summary (paragraph+) | /summary-say | Condense for listening |
| Status updates | /say | Brief announcements |
| Requirement recaps | /summary-say | Long content, condense key points |
| Error messages | /say | Exact communication needed |
| Spec overview | /summary-say | Summarize key sections |

## Adaptive Pacing

TTS calls are blocking (~3s per question). Adapt voicing based on user response patterns.

### Anchor vs Non-Anchor Questions

| Question Type | Always Voice? | Skip When Fast? |
|---------------|---------------|-----------------|
| Phase intro/transition | Yes | No |
| AskUserQuestion (choices) | Yes | No |
| Core interview questions | Yes | No |
| Follow-up clarifications | No | Yes |
| Quick confirmations | No | Yes |
| Final recap/summary | Yes | No |

### Detecting User Pace

Infer user pace from response patterns:

**Fast-paced user** (reduce voicing):
- Short, terse answers: "yes", "correct", "that's right"
- User answers before you finish context
- Quick back-and-forth pattern
- User explicitly says "faster" or "skip ahead"

**Deliberate-paced user** (voice everything):
- Detailed, thoughtful responses
- User asks clarifying questions
- Takes time between responses
- Explicitly appreciates voice feedback

### Pacing Logic

```
At interview start:
  voice_all = true  # Default to voicing

After each user response:
  if response is terse AND came quickly:
    fast_response_count += 1
  else:
    fast_response_count = 0

  if fast_response_count >= 2:
    # User is in rapid-fire mode
    voice_all = false
    # Only voice anchor questions (phase intros, AskUserQuestion, recaps)

  if user explicitly requests more/less voicing:
    # Respect explicit preference
    adjust voice_all accordingly
```

### Graceful Adaptation

When reducing voicing:
- Don't announce "skipping voice for speed"
- Just naturally continue with text
- Resume voicing for anchor questions

When user pace slows down:
- Gradually reintroduce voicing
- Reset fast_response_count when user gives detailed answer

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

```
For each question or question batch:
  If voice_mode_enabled and tts_verified:
    Invoke /say with question text
  Display question text (always, regardless of TTS)
  Wait for user response

For phase transitions:
  If voice_mode_enabled and tts_verified:
    Invoke /summary-say with phase summary
  Display summary text
```

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

## Best Practices

1. **Never block on TTS**: If TTS fails, continue immediately with text
2. **Keep TTS text natural**: Write questions as you'd speak them
3. **Pause between batched questions**: Give listener time to process
4. **Summarize, don't recite**: Use /summary-say for anything over 2-3 sentences
5. **Fail gracefully**: Users should barely notice if TTS stops working
6. **Don't over-voice**: Status messages and minor updates don't need TTS
7. **Match tone**: Keep TTS text conversational, not robotic
