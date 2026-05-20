# Implementation Plan - Job Complexity Classifier Agent

Create a new agent to classify service request complexity (Basic, Intermediate, Complex) to better match providers.

## User Review Required

> [!NOTE]
> The agent will use a rule-based logic for now to simulate the Gemini classification, as the current architecture uses mock implementations for agent testing. It can be easily updated to a real Gemini call using the provided prompt template.

## Proposed Changes

### Job Complexity Classifier Agent

#### [NEW] [jobComplexityClassifierAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/jobComplexityClassifierAgent.ts)
- Define `ComplexityInput` and `ComplexityOutput` interfaces.
- Implement `JobComplexityClassifierAgent` class.
- Implement `classifyComplexity` method:
    - Analyze `service_type`, `user_description`, and `special_notes`.
    - Detect complexity indicators (scope, keywords, tool needs).
    - Map to `BASIC`, `INTERMEDIATE`, or `COMPLEX` levels.
    - Set `estimated_hours` and `estimated_cost_range`.
    - Set `provider_requirements` (min_experience_jobs, certifications_required, specialization_match).
    - Provide `reasoning` and `next_action`.

## Verification Plan

### Automated Tests
- Create `testComplexity.ts` to test various service requests:
    - Basic: "Fix leaky tap"
    - Intermediate: "Install new AC unit"
    - Complex: "Complete home rewiring"
- Verify that complexity levels and provider requirements are correctly assigned.
