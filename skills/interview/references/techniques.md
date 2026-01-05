# Interview Techniques

Patterns and approaches for effective requirements gathering.

## Core Techniques

### The 5 Whys
When answers are surface-level, dig deeper:
- "Why is that important?"
- "Why does that happen?"
- "Why can't we do X instead?"

Stop when you hit a fundamental constraint or business truth.

### Inversion
Ask about the opposite to clarify boundaries:
- "What would make this fail?"
- "What's explicitly NOT included?"
- "What should we definitely avoid?"

### Concrete Examples
Abstract requirements become clearer with examples:
- "Can you walk me through a specific scenario?"
- "Give me an example of when this would happen"
- "What would a user see at this step?"

### Edge Case Probing
Systematically explore boundaries:
- "What if the input is empty?"
- "What about the first-time user?"
- "What happens at scale - 10x, 100x users?"
- "What if two users do this simultaneously?"

### Trade-off Forcing
When everything seems equal priority:
- "If you could only have one of these, which?"
- "What would you cut if you had half the time?"
- "Speed or completeness - which matters more?"

### Stakeholder Perspective
Shift viewpoints to uncover hidden requirements:
- "How would your manager evaluate this?"
- "What would a new hire need to know?"
- "How would an attacker try to abuse this?"

## Handling Common Situations

### Vague Answers
When answers lack specificity:
1. Ask for a concrete example
2. Offer options: "Do you mean A or B?"
3. Probe with "specifically, what does X mean?"
4. Suggest a default and ask if it's right

### Contradictions
When answers conflict with earlier statements:
1. Surface it directly: "Earlier you said X, now Y - which is it?"
2. Offer to resolve: "Should we prioritize X or Y?"
3. Document both as a trade-off if genuinely in tension

### "I Don't Know" Responses
When the user is uncertain:
1. Offer common patterns: "Typically, people do X or Y"
2. Suggest they can decide later: "Let's note this as TBD"
3. Ask who would know: "Who should we ask about this?"
4. Move on and return later

### Scope Creep
When answers expand scope significantly:
1. Acknowledge: "That's interesting - it would mean..."
2. Check intent: "Is this must-have for v1, or later?"
3. Explicitly add to "out of scope" or "future" if not v1

### Low Effort Answers
When answers are too brief:
1. Rephrase and ask for more: "Can you elaborate on...?"
2. Offer specifics: "For example, should it handle X?"
3. Skip if truly not applicable

## Question Delivery

### Batching
- Ask 1-3 related questions at a time
- Don't overwhelm with long lists
- Group by theme for context

### Signposting
Announce phase transitions:
- "Now let's talk about technical design..."
- "Moving on to edge cases..."
- "Almost done - just a few questions about testing..."

### Using AskUserQuestion
Best for:
- Clear categorical choices (2-4 options)
- Priority/preference decisions
- Confirmation of scope boundaries

Not for:
- Open-ended exploration
- Questions needing explanation
- Deeply technical discussions

### Active Listening
Show you're tracking:
- Reference earlier answers: "You mentioned X earlier..."
- Build on context: "Given the constraint about Y..."
- Confirm understanding: "So if I understand correctly..."

## Red Flags to Probe

Watch for and investigate:
- "It should just work" → What does "work" mean specifically?
- "Users will figure it out" → What guidance do they need?
- "We'll handle that later" → Is it actually deferred or forgotten?
- "It's obvious" → Make it explicit anyway
- "Everyone knows" → Document for those who don't
- "It depends" → On what exactly?

## Interview Pacing

### Time Management
- Spend more time on uncertain areas
- Speed through well-understood domains
- Track if interview is running long → condense remaining phases

### Energy Management
- Put complex technical questions mid-interview (not at start or end)
- End with concrete, actionable questions (implementation planning)
- Vary question types to maintain engagement

### Knowing When to Stop
A phase is complete when:
- Key questions answered with specificity
- No new information emerging
- User is giving confident, clear answers
- Follow-ups aren't revealing new dimensions
