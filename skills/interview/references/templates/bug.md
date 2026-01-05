# Bug Interview Template

Use this template for defects and issues that need fixing.

## Interview Depth
- **Questions**: 10-15
- **Focus phases**: Problem (1), Edge Cases (6), Testing (7)
- **Skip/minimal phases**: Users (2), Metrics (8), Implementation (9)

## Phase Adjustments

### Phase 1: Problem & Context
**Full depth** - diagnostic focus:
- What's the bug? Describe exact behavior
- Expected vs actual behavior
- How to reproduce (exact steps)
- When did this start happening?
- Is it consistent or intermittent?
- What's the impact? (users affected, severity)
- Any recent changes that might have caused it?
- Error messages or logs available?

### Phase 2: Users & Stakeholders
**Minimal** (2-3 questions):
- Who reported this?
- Who's affected?
- Is there a workaround?

### Phase 3: Scope & Solution
**Condensed**:
- What's the proposed fix?
- Any risk of regression?
- Is this the root cause or just a symptom?

### Phase 4: Technical Design
**Focused on fix**:
- Where in the code is the issue?
- What's the fix approach?
- What other areas might be affected?
- Any architectural concerns?

### Phase 5: Constraints
**Minimal**:
- How urgent is this?
- Any constraints on the fix?

### Phase 6: Edge Cases
**Full depth** for the fix:
- Does the fix handle all variations?
- Could the fix break anything else?
- What about similar code elsewhere?

### Phase 7: Testing
**Full depth** - regression focus:
- How to verify the fix works?
- Regression test needed?
- What should automated test cover?
- Manual testing scenarios?

### Phase 8: Metrics
**Skip or minimal**:
- How to confirm bug is fixed in prod?

### Phase 9: Implementation
**Skip** - usually straightforward for bugs.

## Spec Output
Use condensed bug spec format:
- Problem Statement (reproduction, impact)
- Root Cause Analysis
- Solution Design
- Testing Plan
