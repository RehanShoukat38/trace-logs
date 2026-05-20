# Implementation Plan - Enhance Provider Ranking Agent

The goal is to expand the Provider Ranking Agent from 5 to 11 ranking factors, update the composite score formula, and implement more sophisticated decision logic.

## User Review Required

> [!IMPORTANT]
> The `RankingInput` interface is being expanded to include `service_type` and `previously_booked_provider_ids` to support specialization matching and user preference alignment.
> 
> The `CandidateInput` (Provider Data Model) is being significantly expanded with new fields like `specializations`, `current_jobs`, `max_capacity`, etc.

## Proposed Changes

### Provider Ranking Agent

#### [MODIFY] [providerRankingAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/providerRankingAgent.ts)
- Update `CandidateInput` interface with new fields (specializations, capacity, risk metrics, etc.).
- Update `RankingInput` interface to include `service_type` and `previously_booked_provider_ids`.
- Update `ScoreBreakdown` to include all 11 factor scores.
- Update `rankProviders` method:
    - Implement new weights (20%, 15%, 20%, 10%, 10%, 5%, 10%, 5%, 5%, 5%, 5%).
    - Calculate 6 new scores:
        - `review_recency`: Based on days since `last_review_date`.
        - `skill_specialization_match`: Matching `specializations` against `service_type`.
        - `provider_capacity`: `(max_capacity - current_jobs) / max_capacity`.
        - `cancellation_rate_score`: `1 - cancellation_rate`.
        - `user_preference_alignment`: Bonus for previous bookings.
        - `risk_score`: `1 - (risk_incidents / total_jobs)`.
    - Apply new composite score formula.
    - Implement Decision Logic Enhancements:
        - Flag "high risk" if `risk_score < 0.7`.
        - 10% composite score boost if `specialization_score = 1.0`.
        - Exclude from urgent requests if `cancellation_rate > 0.15`.
- Update `generateRankingReason` to mention top 3 contributing factors as requested.

## Verification Plan

### Automated Tests
- Create/Update `testRanking.ts` to include providers with the new data model fields.
- Verify that the ranking reflects the new factors and weights.
- Verify that "high risk" providers are flagged and moved down.
- Verify that specialization boost is applied correctly.
- Verify that providers with high cancellation rates are excluded from urgent requests.

### Manual Verification
- Review the output `RankingOutput` to ensure all 11 scores and the composite score are present.
- Confirm the `ranking_reason` matches the requested format.
