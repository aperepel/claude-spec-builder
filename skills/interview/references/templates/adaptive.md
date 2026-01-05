# Adaptive Interview Template

Default template when work type is unclear or mixed. Dynamically adjusts based on responses.

## Interview Depth
- **Questions**: 15-40+ (adapts as interview progresses)
- **Focus phases**: Determined during Phase 0
- **All phases**: Available, depth determined by responses

## Adaptive Behavior

### Phase 0: Classification
Start by understanding the work:

1. Ask: "What are you trying to accomplish?"
2. Based on response, use AskUserQuestion to confirm:
   ```
   question: "What type of work is this?"
   header: "Work type"
   options:
     - label: "New Feature"
       description: "Adding new capability or functionality"
     - label: "Bug Fix"
       description: "Fixing broken behavior"
     - label: "Refactor"
       description: "Improving code without changing behavior"
     - label: "Integration"
       description: "Connecting systems or services"
   ```

3. Based on selection, load appropriate template and adjust depth.

### Dynamic Depth Adjustment

Monitor responses for complexity signals:

**Signals to EXPAND depth**:
- User mentions multiple components/systems
- Answers reveal unexpected complexity
- User says "it depends" frequently
- Multiple stakeholders mentioned
- Security/compliance mentioned
- Scale/performance concerns raised

**Signals to CONDENSE depth**:
- User gives short, confident answers
- Work is well-understood
- Clear boundaries stated
- Single component affected
- User indicates time pressure

### Phase Transitions

After each phase, evaluate:
1. Did we get enough information?
2. Are there signals to dig deeper?
3. Is user engaged or fatigued?

Adjust remaining phases accordingly.

### Question Selection

For each phase:
1. Start with 2-3 core questions
2. If answers reveal depth, add follow-ups
3. If answers are complete, move on
4. Use AskUserQuestion for decision points

### Mid-Interview Pivots

If work type becomes clearer mid-interview:
- Announce: "This sounds like a [type] - let me adjust my questions"
- Load appropriate template
- Skip already-covered areas
- Focus on template-specific concerns

## Fallback Behaviors

**If user is unsure about anything**:
- Offer common patterns as options
- Mark as TBD for later
- Note who might know

**If scope keeps expanding**:
- Call it out: "This is growing - should we focus on a smaller first phase?"
- Offer to split into multiple specs

**If user is terse**:
- Try more specific questions
- Offer examples
- Respect if they want brief

## Spec Output

Use full template but:
- Include all sections that had content
- Mark sparse sections as "TBD" rather than omitting
- Note where interview was condensed
