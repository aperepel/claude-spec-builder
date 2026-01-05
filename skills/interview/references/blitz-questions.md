# Blitz Interview Questions

Work-type driven question sets for rapid bead clarification (3-10 questions).

## Overview

Blitz interviews are adaptive:
- **Well-specified beads (score 8-10)**: 1-3 confirmation questions
- **Moderate beads (score 5-7)**: 3-5 focused questions on gaps
- **Sparse beads (score 0-4)**: 6-10 comprehensive questions

Question selection depends on:
1. Bead type (bug, feature, task, refactor)
2. Completeness score from context analysis
3. What's missing vs. what's already documented

---

## Bug Questions

### Core Questions (always ask if missing)

1. **Reproduction**
   ```
   "Can you walk me through the exact steps to reproduce this bug?"
   ```

2. **Expected vs Actual**
   ```
   "What should happen, and what's happening instead?"
   ```

3. **Environment**
   ```
   AskUserQuestion:
     question: "What environment does this occur in?"
     header: "Environment"
     options:
       - "Production only"
       - "Development/staging"
       - "All environments"
       - "Specific browser/OS"
     multiSelect: false
   ```

### Depth Questions (ask based on gaps)

4. **Frequency**
   ```
   AskUserQuestion:
     question: "How often does this occur?"
     header: "Frequency"
     options:
       - "Every time (100%)"
       - "Frequently (>50%)"
       - "Sometimes (<50%)"
       - "Rarely/hard to reproduce"
     multiSelect: false
   ```

5. **Impact**
   ```
   "Who's affected and what's the business impact? Any workarounds being used?"
   ```

6. **Recent Changes**
   ```
   "Did this start after a recent change or deployment? Any correlation with other events?"
   ```

7. **Error Details**
   ```
   "Are there any error messages, stack traces, or logs available?"
   ```

### Confirmation (for well-specified bugs)

```
"This bug report looks complete. Quick confirmation:
1. Reproduction steps are accurate?
2. Priority reflects the actual impact?
3. Anything changed since this was filed?"
```

---

## Feature Questions

### Core Questions (always ask if missing)

1. **User Value**
   ```
   "What specific problem does this solve for users? What can they do that they couldn't before?"
   ```

2. **Scope Boundaries**
   ```
   "What's explicitly IN scope and what's OUT of scope for this feature?"
   ```

3. **Acceptance Criteria**
   ```
   "How will we know this feature is complete? What are the must-have criteria?"
   ```

### Depth Questions (ask based on gaps)

4. **User Segments**
   ```
   AskUserQuestion:
     question: "Who is the primary user for this feature?"
     header: "Users"
     options:
       - "All users"
       - "Power users/admins"
       - "New users"
       - "Specific segment"
     multiSelect: false
   ```

5. **Dependencies**
   ```
   "What needs to exist before this can be built? Any blockers or prerequisites?"
   ```

6. **Edge Cases**
   ```
   "What edge cases or error scenarios should we handle? What happens when things go wrong?"
   ```

7. **Rollout Strategy**
   ```
   AskUserQuestion:
     question: "How should this feature be rolled out?"
     header: "Rollout"
     options:
       - "Full release to all users"
       - "Gradual/phased rollout"
       - "Feature flag (optional)"
       - "Beta/early access first"
     multiSelect: false
   ```

8. **Success Metrics**
   ```
   "How will we measure if this feature is successful? What metrics matter?"
   ```

### Confirmation (for well-specified features)

```
"This feature looks well-defined. Quick check:
1. Scope boundaries are still accurate?
2. Any dependencies or blockers since this was written?
3. Acceptance criteria still reflect what 'done' means?"
```

---

## Task Questions

### Core Questions (always ask if missing)

1. **Implementation Approach**
   ```
   "What's the planned implementation approach? Any specific patterns or techniques?"
   ```

2. **Definition of Done**
   ```
   "What constitutes 'done' for this task? Tests, docs, review?"
   ```

3. **Blockers**
   ```
   "Are there any current blockers or dependencies? Waiting on anything?"
   ```

### Depth Questions (ask based on gaps)

4. **Technical Decisions**
   ```
   AskUserQuestion:
     question: "Are there technical decisions that need to be made?"
     header: "Decisions"
     options:
       - "Yes, need guidance on approach"
       - "Some decisions, but have preferences"
       - "No, approach is clear"
     multiSelect: false
   ```

5. **Testing Strategy**
   ```
   "What testing is needed? Unit tests, integration tests, manual verification?"
   ```

6. **Files/Components**
   ```
   "Which files or components will this touch? Any areas of concern?"
   ```

7. **Risks**
   ```
   "Any risks or areas where things might go wrong? How do we mitigate them?"
   ```

### Confirmation (for well-specified tasks)

```
"This task looks ready to work. Quick confirmation:
1. Implementation approach is clear?
2. No new blockers since this was created?
3. Definition of done is still accurate?"
```

---

## Refactor Questions

### Core Questions (always ask if missing)

1. **Current State**
   ```
   "What's wrong with the current implementation? What problems are we solving?"
   ```

2. **Target State**
   ```
   "What should it look like after the refactor? What's the goal architecture?"
   ```

3. **Migration Path**
   ```
   "What's the migration strategy? Big bang or incremental? How do we get from A to B?"
   ```

### Depth Questions (ask based on gaps)

4. **Risk Areas**
   ```
   "What are the highest-risk areas? Where are bugs most likely to be introduced?"
   ```

5. **Testing Coverage**
   ```
   AskUserQuestion:
     question: "What's the current test coverage for the code being refactored?"
     header: "Test coverage"
     options:
       - "Good coverage (>80%)"
       - "Moderate coverage (40-80%)"
       - "Low coverage (<40%)"
       - "No tests"
     multiSelect: false
   ```

6. **Rollback Plan**
   ```
   "If something goes wrong, how do we rollback? Feature flags, quick revert?"
   ```

7. **Performance Impact**
   ```
   "Any expected performance changes? Better, worse, or neutral?"
   ```

8. **Breaking Changes**
   ```
   "Are there any breaking changes for consumers of this code? API changes, behavior changes?"
   ```

### Confirmation (for well-specified refactors)

```
"This refactor plan looks solid. Quick check:
1. Target architecture is still the goal?
2. Migration strategy is feasible?
3. Any new risks or concerns since planning?"
```

---

## Question Selection Algorithm

```
Given: bead_context with type and completeness score

1. Determine question set based on bead.type:
   - bug → Bug Questions
   - feature → Feature Questions
   - task → Task Questions
   - refactor → Refactor Questions

2. Based on completeness.score:
   - 8-10: Use Confirmation question only
   - 5-7: Core questions + 1-2 depth questions for gaps
   - 0-4: Core questions + all relevant depth questions

3. Skip questions where completeness.present includes the topic:
   - If "reproduction_steps" in present → skip reproduction question
   - If "acceptance_criteria" in present → skip acceptance question
   - etc.

4. Order questions:
   - Most critical gaps first
   - Confirmation-style questions last
   - Group related questions together

5. Limit total questions:
   - Score 8-10: max 3 questions
   - Score 5-7: max 5 questions
   - Score 0-4: max 10 questions
```

---

## Adaptive Phrasing

### When Context Exists

If the bead has relevant content, reference it:

```
# Instead of:
"What problem does this solve?"

# Say:
"The description mentions [X]. Is that still the core problem, or has it evolved?"
```

### When Dependencies Exist

Reference blocking issues:

```
"I see this depends on <blocker-title> which is <status>.
Is that still accurate, or has the situation changed?"
```

### When Files Are Referenced

Acknowledge linked files:

```
"The description references <file-path>.
Is the approach described there still current?"
```

---

## TTS Considerations

For blitz interviews with TTS enabled:

1. **Always voice**: Core questions, phase transitions
2. **Optionally voice**: Depth questions (based on user response speed)
3. **Never voice**: Internal processing, completeness analysis

Keep questions concise for TTS - long questions are harder to follow aurally.

```
# Too long for TTS:
"Given that you mentioned the feature should support multiple authentication
providers including OAuth, SAML, and potentially custom integrations,
what are the specific requirements for each provider type?"

# Better for TTS:
"You mentioned multiple auth providers. Which ones are must-have versus nice-to-have?"
```

---

## Completeness Indicators by Type

### Bug Completeness

| Indicator | Weight | Detection |
|-----------|--------|-----------|
| Has repro steps | +2 | "steps", "reproduce", numbered list |
| Has expected/actual | +2 | "expected", "actual", "should" |
| Has environment | +1 | "browser", "version", "OS" |
| Has error details | +1 | "error", "exception", "stack" |
| Has impact | +1 | "users affected", "critical", "blocking" |

### Feature Completeness

| Indicator | Weight | Detection |
|-----------|--------|-----------|
| Has user value | +2 | "user can", "enables", "allows" |
| Has scope | +2 | "in scope", "out of scope", "includes" |
| Has acceptance criteria | +2 | "done when", "criteria", "requirements" |
| Has edge cases | +1 | "edge case", "error", "what if" |
| Has dependencies | +1 | "depends on", "requires", "blocked by" |

### Task Completeness

| Indicator | Weight | Detection |
|-----------|--------|-----------|
| Has implementation | +2 | "implement", "create", "approach" |
| Has done criteria | +2 | "done when", "complete when", "definition" |
| Has testing | +1 | "test", "verify", "validate" |
| Has files/scope | +1 | file paths, "touches", "modifies" |
| Has blockers noted | +1 | "blocked by", "waiting", "depends on" |

### Refactor Completeness

| Indicator | Weight | Detection |
|-----------|--------|-----------|
| Has current state | +2 | "currently", "existing", "before" |
| Has target state | +2 | "target", "goal", "after", "should be" |
| Has migration plan | +2 | "steps", "approach", "migrate" |
| Has risks | +1 | "risk", "concern", "careful" |
| Has rollback | +1 | "rollback", "revert", "feature flag" |
