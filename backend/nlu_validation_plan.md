# Implementation Plan - Google Places Integration for NLU Agent

Enhance the `NLUAgent` to validate extracted locations and retrieve precise coordinates using the Google Places and Geocoding APIs.

## User Review Required

> [!IMPORTANT]
> A valid Google Maps API Key is required for location validation. If the key is missing or validation fails, the agent will flag that clarification is needed.

## Proposed Changes

### NLU Agent

#### [MODIFY] [nluAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/nluAgent.ts)
- Update `NLUOutput` interface:
    - Add `location_name`, `location_lat`, `location_lng`, `location_verified`, and `api_used` to `extracted_intent`.
- Implement `validateAndGeocodeLocation`:
    - Calls Google Places Autocomplete API.
    - If found, calls Place Details API for coordinates.
    - Fallback to Geocoding API if no autocomplete matches.
- Update `processRequest`:
    - After extracting `location_district`, call `validateAndGeocodeLocation`.
    - Populate new intent fields.
    - If location cannot be verified, set `clarification_needed = true` and `next_action = "request_clarification"`.

## Verification Plan

### Automated Tests
- Create `testNLUValidation.ts` to verify:
    - Input like "Bahria Town" gets validated and returns coordinates.
    - `location_verified` is true on success.
    - `api_used` correctly reports the source.
    - Invalid locations trigger `clarification_needed`.
