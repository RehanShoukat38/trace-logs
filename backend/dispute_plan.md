# Implementation Plan - Dispute Resolution Agent

Create a new agent to handle service disputes, cancellations, refunds, and provider penalty management.

## User Review Required

> [!NOTE]
> The agent implements complex business logic for refunds and penalties. For the mock implementation, we will simulate the verification of no-shows and evidence. Damage claims will always trigger an escalation to human review as per the requirements.

## Proposed Changes

### Dispute Resolution Agent

#### [NEW] [disputeResolutionAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/disputeResolutionAgent.ts)
- Define `DisputeInput` and `DisputeOutput` interfaces.
- Implement `DisputeResolutionAgent` class.
- Implement `resolveDispute` method:
    - Dispute Type Handling:
        - `no_show`: Verification + 100% refund + 500 PKR + penalty.
        - `cancellation`: Tiered refunds (100%/75%/50%/25%) based on time remaining.
        - `quality_complaint`: Partial refund (25-75%) + redo offer.
        - `price_disagreement`: Comparison vs quote + refund if diff > 10%.
        - `overrun`: Cap at 130% of estimate.
        - `damage_claim`: Immediate escalation.
    - Provider Penalty System: Track warnings and update status (probation/suspension/blacklist).
    - Escalation Triggers: Handle high-value disputes (>10k PKR) and sensitive issues.
    - Refund Calculation: Structured breakdown of refund and compensation.
    - Return detailed `DisputeOutput`.

## Verification Plan

### Automated Tests
- Create `testDispute.ts` to test various scenarios:
    - User cancels 15 hours before (should get 75% refund).
    - Provider no-show (should get 100% refund + 500 compensation).
    - Damage claim (should trigger escalation).
    - Price disagreement of 20% (should refund diff + 20% compensation).
- Verify that refunds, penalties, and next actions match the business rules.
