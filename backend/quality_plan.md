# Implementation Plan - Service Quality Tracker Agent

Create a new agent to track the service lifecycle from en-route to completion, collect evidence, and manage provider quality metrics and feedback.

## User Review Required

> [!NOTE]
> The agent handles multiple stages of the service lifecycle. For the purpose of the mock implementation, we will simulate the calculation of lateness and rating updates. Sentiment analysis will be keyword-based (rudimentary) as a placeholder for a real LLM-based sentiment analysis.

## Proposed Changes

### Service Quality Tracker Agent

#### [NEW] [serviceQualityTrackerAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/serviceQualityTrackerAgent.ts)
- Define `QualityTrackerInput` and `QualityTrackerOutput` interfaces.
- Implement `ServiceQualityTrackerAgent` class.
- Implement `processUpdate` method:
    - Status Tracking: Update tracking data based on `en_route`, `arrived`, `in_progress`, or `completed`.
    - Lateness Calculation: Compare `actual_arrival_time` vs `scheduled_time`.
    - Quality Scoring: Calculate `quality_score` from category ratings (Timeliness, Quality, Professionalism, Cleanliness).
    - Rating Update: Apply the formula `new_rating = (old_rating * total_jobs + new_rating) / (total_jobs + 1)`.
    - Red Flag Detection: Monitor for lateness > 30m, ratings < 3.0, and negative keywords (e.g., "rude", "incomplete").
    - Evidence Verification: Check completion checklist and photo evidence.
    - Return detailed `QualityTrackerOutput`.

## Verification Plan

### Automated Tests
- Create `testQualityTracker.ts` to test various scenarios:
    - Provider arrives 40 mins late (should trigger red flag).
    - Provider completes job with 5-star feedback (should update rating).
    - Job completed with 2-star feedback and "rude" keyword (should trigger escalation).
- Verify that the provider updates and next actions are correct.
