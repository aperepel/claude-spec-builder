# Refactor Interview Template

Use this template for code improvements, tech debt, and restructuring.

## Interview Depth
- **Questions**: 15-25
- **Focus phases**: Technical Design (4), Edge Cases (6), Testing (7)
- **Skip/minimal phases**: Users (2), Metrics (8)

## Phase Adjustments

### Phase 1: Problem & Context
**Standard depth** - pain focus:
- What's wrong with the current code?
- What symptoms are you experiencing? (slow, hard to maintain, etc.)
- How long has this been a problem?
- What triggered the decision to refactor now?
- What's the impact on development velocity?
- Have there been bugs related to this code?

### Phase 2: Users & Stakeholders
**Minimal** - developer focused:
- Who maintains this code?
- Who needs to approve the refactor?
- Any downstream teams affected?

### Phase 3: Scope & Solution
**Standard depth**:
- What's being refactored? (specific components/modules)
- What's the target state?
- What's staying the same?
- Is this a full rewrite or incremental change?
- What's explicitly out of scope?

### Phase 4: Technical Design
**Full depth** - architecture focus:
- What's the current architecture?
- What's the target architecture?
- What patterns should the new code follow?
- What interfaces/APIs change?
- Data migration needed?
- What dependencies are affected?
- Can this be done incrementally?

### Phase 5: Constraints
**Standard depth**:
- Must behavior be preserved exactly?
- Can we break backward compatibility?
- Any performance requirements?
- Timeline constraints?

### Phase 6: Edge Cases
**Full depth** - regression focus:
- What edge cases does current code handle?
- Are there undocumented behaviors we need to preserve?
- What could break during migration?
- How do we handle partial migration state?

### Phase 7: Testing
**Full depth** - critical for refactors:
- How do we prove behavior is unchanged?
- Existing test coverage?
- New tests needed?
- Performance benchmarks?
- Characterization tests before refactoring?

### Phase 8: Metrics
**Minimal**:
- How to measure improvement? (velocity, bugs, performance)

### Phase 9: Implementation
**Standard depth**:
- Can this be done in phases?
- What order makes sense?
- Rollback strategy if it goes wrong?
- Feature flags for gradual migration?

## Spec Output
Use refactor-focused spec:
- Problem Statement (current pain, debt)
- Current vs Target Architecture
- Migration Plan
- Testing Strategy (behavior preservation)
- Risk Assessment
