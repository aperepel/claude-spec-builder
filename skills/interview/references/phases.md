# Interview Phases

Detailed question sets for each phase. Adapt based on work type and scope.

## Phase 1: Problem & Context (5-10 questions)

**Goal**: Understand why this work matters and the current situation.

Core questions:
- What problem are you trying to solve?
- Why is this important now? What's the trigger?
- What happens if we don't do this?
- What have you tried before? What worked/didn't?
- Is there existing code or systems we're building on?
- Are there external dependencies or constraints?

Work-type adaptations:
- **Bug**: Focus on reproduction steps, when it started, impact
- **Refactor**: Focus on pain points, tech debt symptoms, maintenance burden
- **Integration**: Focus on systems involved, data flows, existing touchpoints

## Phase 2: Users & Stakeholders (5-8 questions)

**Goal**: Identify who's affected and their needs.

Core questions:
- Who will use this? (primary, secondary users)
- What's their technical skill level?
- How will they discover this feature?
- Who needs to approve or sign off?
- Are there other stakeholders affected?
- What do users currently do instead?

Work-type adaptations:
- **Internal tool**: Skip discovery, focus on team needs
- **API/Integration**: Focus on consuming systems, not end users
- **Refactor**: Focus on developers maintaining the code

## Phase 3: Scope & Solution (8-12 questions)

**Goal**: Define boundaries and high-level approach.

Core questions:
- What's the core capability we're building?
- What's explicitly OUT of scope?
- What are the must-have vs nice-to-have features?
- How does this fit with existing features?
- What's the minimum viable version?
- What could we add later?
- Are there multiple ways to solve this? Which do you prefer?
- What's the simplest solution that works?

Use AskUserQuestion for:
- Priority ranking of features
- Choosing between solution approaches
- Confirming scope boundaries

## Phase 4: Technical Design (10-15 questions)

**Goal**: Architecture, APIs, data, implementation approach.

Core questions:
- What's the overall architecture?
- What components need to be created/modified?
- What APIs or interfaces are needed?
- What data needs to be stored? Schema implications?
- What existing patterns should we follow?
- Are there performance requirements?
- Security considerations?
- What external services or dependencies?
- How does this integrate with existing code?
- What needs to happen at build/deploy time?

Work-type adaptations:
- **Bug**: Focus on root cause, affected areas
- **Feature**: Full technical design
- **Task**: Focus on specific implementation approach
- **Refactor**: Focus on target architecture, migration path

## Phase 5: Constraints & Trade-offs (5-8 questions)

**Goal**: Understand limitations and conscious decisions.

Core questions:
- What are the timeline constraints?
- What technical constraints exist?
- What resources are available (people, budget)?
- What trade-offs are you willing to make?
- Speed vs quality vs scope - what gives?
- Are there compliance or regulatory requirements?
- What technical debt are you willing to accept?

Use AskUserQuestion for:
- Trade-off preferences
- Priority when constraints conflict

## Phase 6: Edge Cases & Errors (8-12 questions)

**Goal**: Surface what can go wrong and how to handle it.

Core questions:
- What happens when [input] is empty/null/invalid?
- What if the user does [unexpected action]?
- How do we handle network failures?
- What if dependent service is down?
- What about concurrent operations?
- Large data volumes?
- What if the user abandons mid-process?
- Rate limiting or abuse scenarios?
- Backward compatibility concerns?

Techniques:
- "What's the worst thing that could happen?"
- "What if a malicious user tried to..."
- "Imagine this runs for a year - what breaks?"

## Phase 7: Testing & Quality (5-8 questions)

**Goal**: Define how we'll verify correctness.

Core questions:
- How will you know this works?
- What are the critical test scenarios?
- What needs manual testing vs automated?
- What quality gates must pass?
- Performance testing needed?
- Security testing needed?
- How do we test integrations?

Work-type adaptations:
- **Bug**: Focus on regression test for the fix
- **Refactor**: Focus on proving behavior unchanged

## Phase 8: Success & Metrics (3-5 questions)

**Goal**: Define how we measure impact.

Core questions:
- How will you know this succeeded?
- What metrics matter?
- What's the timeline for evaluation?
- What would make this a failure?
- How do we monitor in production?

Work-type adaptations:
- **Internal tool**: Focus on adoption, time savings
- **User-facing**: Focus on usage, engagement, satisfaction

## Phase 9: Implementation Planning (3-5 questions)

**Goal**: Sequence the work.

Core questions:
- What needs to happen first?
- Are there natural milestones?
- What can be parallelized?
- What's the deployment strategy?
- Any feature flags or gradual rollout?
- What documentation is needed?

Output:
- Ordered list of milestones/tasks
- Dependencies between items
- Ready for beads epic creation

## Adaptive Depth Guide

| Work Type | Total Questions | Heavy Phases | Light Phases |
|-----------|-----------------|--------------|--------------|
| Epic | 40-65 | All phases full | None |
| Feature | 20-35 | 3, 4, 6 | 2, 8 |
| Task | 10-20 | 4, 6 | 1, 2, 5, 8 |
| Bug | 10-15 | 1 (diagnosis), 6, 7 | 2, 3, 5, 8 |
| Refactor | 15-25 | 4, 6, 7 | 2, 8 |
