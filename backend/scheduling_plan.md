# Implementation Plan - Scheduling Conflict Manager Agent

Create a new agent to manage provider schedules, prevent double-booking, and handle travel time buffers and workload limits.

## User Review Required

> [!NOTE]
> Travel time calculation will be mocked for now (using a simple distance-based estimation) to simulate the Google Maps Distance Matrix API. Buffers and workload limits will be strictly enforced as per requirements.

## Proposed Changes

### Scheduling Conflict Manager Agent

#### [NEW] [schedulingConflictManagerAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/schedulingConflictManagerAgent.ts)
- Define `SchedulingInput` and `SchedulingOutput` interfaces.
- Implement `SchedulingConflictManagerAgent` class.
- Implement `checkSchedule` method:
    - Time Overlap: Check if `requested_time` overlaps with existing `provider_calendar` entries.
    - Travel Time: Mock calculation based on lat/lng distance.
    - Buffer Management: 15 min pre/post, 30 min between jobs.
    - Workload Check: Max 5 jobs, 8 consecutive hours, break after 4 hours.
    - Resolution Logic:
        - If conflict, suggest the next available slot (at least 30 mins after the last booking).
        - Support waitlisting and alternate slot suggestions.
    - Return detailed `SchedulingOutput`.

## Verification Plan

### Automated Tests
- Create `testScheduling.ts` to test various scenarios:
    - Perfect slot with no conflicts.
    - Time overlap with an existing booking.
    - Tight buffer/travel time violation.
    - Workload limit exceeded (e.g., 6th job of the day).
- Verify that conflict detection and resolution suggestions are accurate.
