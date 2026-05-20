# Implementation Plan - Google Maps Integration for Provider Search

Enhance the `ProviderSearchAgent` to use the Google Maps Distance Matrix API for accurate, traffic-aware travel times and distances.

## User Review Required

> [!IMPORTANT]
> A valid Google Maps API Key is required for real-time data. A placeholder will be used, and the agent will fallback to Haversine calculation if the key is missing or the API call fails.

## Proposed Changes

### Provider Search Agent

#### [MODIFY] [providerSearchAgent.ts](file:///c:/Users/satti/Desktop/Hackathon%20Project/guardian-ai/backend/functions/src/agents/providerSearchAgent.ts)
- Update `Candidate` interface:
    - Add `distance_text: string`.
    - Add `duration_text: string`.
    - Add `api_source: "google_maps_distance_matrix" | "fallback_calculation"`.
- Implement `DistanceCache`:
    - Simple in-memory cache with 15-minute TTL.
- Implement `getRealDistanceAndTime`:
    - Calls Google Maps Distance Matrix API via `fetch`.
    - Includes `departure_time: "now"` for traffic awareness.
    - Handles API errors and missing keys with haversine fallback.
- Update `searchProviders`:
    - Use haversine for initial broad filtering.
    - Select top 10 candidates by haversine distance.
    - Enrich these top 10 using the Distance Matrix API.
    - Re-sort based on actual travel time/distance.

## Verification Plan

### Automated Tests
- Update `testProviderSearch.ts` to verify:
    - Output includes `distance_text` and `duration_text`.
    - `api_source` reflects whether the API was used (or fell back).
    - Cache prevents redundant API calls for the same coordinates.
