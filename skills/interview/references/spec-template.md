# Specification Template

Use this structure for generated specs. Adapt sections based on work type.

---

```markdown
# Feature Specification: <Title>

**Date:** YYYY-MM-DD
**Status:** Draft
**Author:** <author> with Claude Interview
**Interview Duration:** <N> questions across <N> phases

---

## Executive Summary

<2-3 sentences: What is this, what problem does it solve, key approach>

---

## Problem Statement

### Current State
<What exists today, pain points>

### Desired State
<What we want instead>

### Impact of Inaction
<What happens if we don't do this>

---

## Users & Stakeholders

### Primary Users
<Who uses this directly, their characteristics>

### Technical Skill Level
<Assumptions about user capability>

### Discovery
<How users will find/learn about this>

---

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-1 | <requirement> | Must/Should/Could | <how to verify> |

### Non-Functional Requirements

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-1 | <category> | <requirement> | <measurable target> |

### Out of Scope

- ❌ <explicitly excluded item> — <reason>

---

## Technical Design

### Architecture
<High-level structure, components>

### Component Breakdown
<What needs to be built/modified>

### APIs & Interfaces
<External and internal interfaces>

### Data Model
<Schema, storage, data flow>

### Integration Points
<How this connects to existing systems>

---

## Edge Cases & Error Handling

| Scenario | Handling |
|----------|----------|
| <edge case> | <how to handle> |

---

## Testing Strategy

### Testing Approach
<Unit, integration, e2e, manual>

### Must-Pass Scenarios

| ID | Scenario | Priority |
|----|----------|----------|
| T-1 | <test case> | Critical/High/Medium |

### Quality Criteria
<What "done" looks like>

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| <metric> | <target value> | <how to measure> |

### Success Definition
<How we know this succeeded>

### Failure Indicators
<What would make this a failure>

---

## Implementation Plan

### Approach
<Overall strategy>

### Milestones

| Milestone | Scope | Dependencies |
|-----------|-------|--------------|
| M1: <name> | <what's included> | <blockers> |

### Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| <risk> | Low/Medium/High | <impact> | <mitigation> |

---

## Open Questions

| ID | Question | Owner | Status |
|----|----------|-------|--------|
| Q-1 | <question> | <who decides> | Open/Resolved |

---

## Appendix: Key Decisions

### Trade-offs Accepted

| Trade-off | What We Gave Up | What We Gained |
|-----------|-----------------|----------------|
| <decision> | <cost> | <benefit> |

### Constraints
<Hard limits on the solution>

---

## Change Log

| Date | Author | Change |
|------|--------|--------|
| <date> | <author> | Initial draft |
```

---

## Adaptation by Work Type

### Feature Spec
Full template, all sections.

### Task Spec
Lighter version:
- Executive Summary
- Requirements (brief)
- Technical Design (focused)
- Edge Cases
- Testing (key scenarios only)

### Bug Spec
Focused version:
- Problem Statement (reproduction, impact)
- Root Cause Analysis
- Solution Design
- Testing (regression focus)

### Refactor Spec
Technical focus:
- Problem Statement (current pain, debt)
- Technical Design (before/after)
- Migration Plan
- Testing (behavior unchanged)
- Risk Assessment

### Integration Spec
Connection focus:
- Problem Statement
- Systems Involved
- Data Flows
- API Contracts
- Error Handling
- Testing (integration scenarios)
