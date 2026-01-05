# Integration Interview Template

Use this template for connecting systems, APIs, and external services.

## Interview Depth
- **Questions**: 20-30
- **Focus phases**: Technical Design (4), Edge Cases (6)
- **Adapt phases**: Users (2) focuses on consuming systems

## Phase Adjustments

### Phase 1: Problem & Context
**Standard depth**:
- What systems are being integrated?
- Why is this integration needed?
- What's the data/functionality being exchanged?
- Is there an existing integration to replace/enhance?
- What's the business value?

### Phase 2: Users & Stakeholders
**System focused**:
- What systems will consume this integration?
- Who owns each system?
- Who needs to approve the integration?
- Any third-party vendors involved?
- SLAs or contracts to consider?

### Phase 3: Scope & Solution
**Standard depth**:
- What data/operations are in scope?
- What's the integration pattern? (sync, async, event-driven)
- Push or pull?
- Real-time or batch?
- What's explicitly not included?

### Phase 4: Technical Design
**Full depth** - API focus:
- What's the API contract? (endpoints, methods, payloads)
- Authentication/authorization mechanism?
- Data format? (JSON, XML, protobuf)
- Rate limiting requirements?
- Versioning strategy?
- How to handle schema evolution?
- Idempotency requirements?
- What infrastructure changes needed?
- Monitoring/observability approach?

### Phase 5: Constraints
**Standard depth**:
- Latency requirements?
- Throughput requirements?
- Availability requirements?
- Compliance/security constraints?
- Network/firewall considerations?

### Phase 6: Edge Cases
**Full depth** - reliability focus:
- What if the remote system is down?
- What if requests timeout?
- How to handle partial failures?
- What about duplicate messages?
- Data consistency across systems?
- What if data formats change?
- Rate limit exceeded handling?
- Circuit breaker patterns?

### Phase 7: Testing
**Full depth** - integration focus:
- How to test without live external system?
- Mock/stub strategy?
- Contract testing?
- End-to-end testing approach?
- Performance/load testing?

### Phase 8: Metrics
**Standard depth**:
- What to monitor? (latency, errors, throughput)
- Alerting thresholds?
- SLA tracking?

### Phase 9: Implementation
**Standard depth**:
- Implementation phases?
- Parallel running with old integration?
- Cutover strategy?
- Rollback plan?

## Spec Output
Use integration-focused spec:
- Problem Statement
- Systems Overview
- API Contract
- Data Flows
- Error Handling & Resilience
- Testing Strategy
- Monitoring Plan
