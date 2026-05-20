# Implementation Plan - Provider Optimization Agent

Create a new agent to analyze provider performance, balance workload, forecast demand, and provide actionable growth recommendations.

## User Review Required

> [!NOTE]
> Demand forecasting and earning impact estimates are based on simulated data and heuristic patterns. In a production environment, these would be backed by historical data trends and machine learning models.

## Proposed Changes

### Provider Optimization Agent

#### [NEW] [providerOptimizationAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/providerOptimizationAgent.ts)
- Define `OptimizationInput` and `OptimizationOutput` interfaces.
- Implement `ProviderOptimizationAgent` class.
- Implement `generateReport` method:
    - Workload Balancing: Compare `jobs_this_month` vs `avg_provider_jobs_month` (flag underutilized if < 80%, overworked if > 130%).
    - Earning Fairness: Calculate gap vs peer average and flag status.
    - Demand Forecasting: Predict next 7 and 30 days based on input demand trends and seasonal factors.
    - Slot Optimization: Identify idle hours and suggest discounts or actions.
    - Growth Recommendations: Suggest area expansion (e.g., Bahria Town) and skills development (e.g., AC installation).
    - Competitive Analysis: Position provider among peers and identify strengths/weaknesses.
    - Return detailed `OptimizationOutput`.

## Verification Plan

### Automated Tests
- Create `testOptimization.ts` to test various provider profiles:
    - Underutilized provider with high potential in underserved areas.
    - High-earning, overworked provider (top 10%).
    - Balanced average provider.
- Verify that recommendations and status flags are logically consistent with the peer data.
