# 🤖 Guardian AI: 12-Agent Ecosystem Architectural Specification

===============================================================================
  OPERATIONAL SPECIFICATIONS • APIS • INPUT/OUTPUT SCHEMAS • CORE ALGORITHMS
===============================================================================

[ignoring loop detection]

This document serves as the absolute technical reference for each of the **12 specialized agents** operating within the Guardian AI booking orchestration framework.

---

## 🗺️ Master Agent Lifecycle Map

```text
       [RAW USER INPUT]
              │
              ▼
   ┌───────────────────────┐
   │  1. NLU Intent Agent  │ ◄── Parse text, languages, and coordinates
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │ 2. Job Complexity Agt │ ◄── Determine task level & estimate durations
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │ 3. Geo Provider Search│ ◄── Radius filter & Google Maps travel calculations
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │ 4. Multi-Criteria Rank│ ◄── Composite performance weighting & scoring
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │5. Dynamic Pricing Agt │ ◄── Multiplier markups, platform fees & splits
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │6. Booking Execution   │ ◄── Atomic Firestore transaction lockups
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │ 7. Push Notification  │ ◄── Direct-to-app dashboard dispatches in Urdu/English
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │8. Follow-up Scheduler │ ◄── Multi-phase cron watchdogs & reminders
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │ 9. Quality Tracker    │ ◄── GPS location verification & en-route timings
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │10. Response Handler   │ ◄── Provider confirmations & instant fallbacks
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │11. Dispute Resolution │ ◄── Automated billing arbitration & customer refunds
   └──────────┬────────────┘
              ▼
   ┌───────────────────────┐
   │12. Workload Optimizer │ ◄── Underutilization checks & Islamabad area tips
   └───────────────────────┘
```

---

## 📄 1. NLU Intent Agent (`nluAgent.ts`)
Processes raw user requests, determines the primary intent and service type, auto-detects language, and identifies user coordinates.

*   **Logic & Algorithms:** Bypasses LLM clarification loops if confidence score is $\ge 0.70$. Uses structured Gemini 2.5 Flash parsing to extract Pydantic entities.
*   **Input Schema:**
    ```typescript
    interface NLUInput {
        text: string;           // E.g., "electrician chahiye f-10 mein abhi"
        customerId: string;
    }
    ```
*   **Output Schema:**
    ```typescript
    interface NLUOutput {
        serviceType: "electrician" | "plumber" | "carpenter" | "painter";
        urgency: "high" | "medium" | "low";
        locationDistrict: string;   // E.g., "F-10"
        detectedLanguage: "roman_ur" | "script_ur" | "en";
        confidence: number;         // 0.0 to 1.0
        latitude?: number;          // 33.6844
        longitude?: number;         // 73.0479
    }
    ```
*   **APIs:** Integrates with **Google Places API** for landmark resolution and **Google Geocoding API** for coordinate translations. Fallback sets default district centers.

---

## 📄 2. Job Complexity Classifier Agent (`jobComplexityClassifierAgent.ts`)
Classifies the technical level of the request to establish duration estimates and tool requirements.

*   **Logic & Algorithms:** Assesses parsed intent parameters against complexity matrices. Classifies tasks into `basic`, `intermediate`, or `expert` levels.
*   **Input Schema:** Accepts `NLUOutput` results.
*   **Output Schema:**
    ```typescript
    interface ComplexityOutput {
        complexityLevel: "basic" | "intermediate" | "expert";
        complexityScore: number;       // 0.0 to 1.0
        estimatedHours: number;        // E.g., 2
        requiredTools: string[];       // E.g., ["tester", "pliers"]
    }
    ```
*   **APIs:** Internal rule engine mapped to service-complexity schemas.

---

## 📄 3. Geospatial Provider Search Agent (`providerSearchAgent.ts`)
Identifies geographically close, qualified providers who are currently active and available.

*   **Logic & Algorithms:** Conducts a radial database search matching service type and real-time active status (`available_now`). Calculates travel metrics.
*   **Input Schema:**
    ```typescript
    interface SearchInput {
        serviceType: string;
        latitude: number;
        longitude: number;
        radiusKm: number;          // Defaults to 10
    }
    ```
*   **Output Schema:**
    ```typescript
    interface SearchOutput {
        candidates: Array<{
            providerId: string;
            name: string;
            rating: number;
            baseRate: number;
            distanceKm: number;
            coordinates: { lat: number; lng: number };
        }>;
    }
    ```
*   **APIs:** **Google Maps Distance Matrix API**. Bypasses automatically to internal **Haversine Distance Calculator** if external API requests time out.

---

## 📄 4. Multi-Criteria Provider Ranking Agent (`providerRankingAgent.ts`)
Ranks discoverable providers dynamically to match the absolute best option for the specific user request.

*   **Logic & Algorithms:** Evaluates candidate pools using a multi-criteria utility weighting function:
    $$\text{Score} = (Rating \times 0.3) + \left(\left(1 - \frac{Dist}{MaxDist}\right) \times 0.3\right) + (RespRate \times 0.2) + (CompRate \times 0.2) - Penalties$$
*   **Input Schema:** Mapped lists of `SearchOutput` candidates.
*   **Output Schema:**
    ```typescript
    interface RankingOutput {
        rankedProviders: Array<{
            providerId: string;
            compositeScore: number;
            rank: number;
            selectionReason: string;
        }>;
    }
    ```
*   **APIs:** Internal performance weighting algorithms.

---

## 📄 5. Dynamic Pricing Agent (`dynamicPricingAgent.ts`)
Determines customer invoice rates and splits the earnings fairly between the service provider and the platform.

*   **Logic & Algorithms:** Calculates variable coefficients based on 8 key parameters (Base, Complexity, Distance, Urgency surcharges, Peak-hour surge, Slot premiums, Visits, and Loyalty discounts). Enforces the **85/15 Earnings Fairness Rule** (85% to provider, 15% platform commission).
*   **Input Schema:**
    ```typescript
    interface PricingInput {
        providerId: string;
        complexityLevel: "basic" | "intermediate" | "expert";
        distanceKm: number;
        urgency: "high" | "medium" | "low";
        demandSurge: "normal" | "high";
    }
    ```
*   **Output Schema:**
    ```typescript
    interface PricingOutput {
        baseCost: number;
        urgencyFee: number;
        surgeFee: number;
        totalInvoice: number;
        providerEarningsShare: number;   // 85% of Invoice
        platformCommissionShare: number; // 15% of Invoice
    }
    ```

---

## 📄 6. Booking Execution Agent (`bookingExecutionAgent.ts`)
Creates, processes, and registers the booking record directly inside the database.

*   **Logic & Algorithms:** Runs an **Atomic Firestore Transaction** lock. Locks out simultaneous customer scheduling conflicts, reserves slots, and registers watchdog timestamps.
*   **Input Schema:** Coordinated customer, provider, pricing, and scheduling datasets.
*   **Output Schema:**
    ```typescript
    interface BookingOutput {
        bookingId: string;         // BK-SM-XXXX
        status: "pending_provider_confirmation" | "confirmed";
        scheduledTime: string;
        timeoutDeadline: string;   // 3-minute confirmation window timestamp
    }
    ```
*   **APIs:** **Firebase Admin SDK** (Firestore transactions).

---

## 📄 7. Multilingual Customer Notification Agent (`notificationAgent.ts`)
Dispatches pushes and notifications in localized Urdu styles and script formats.

*   **Logic & Algorithms:** Auto-targets messaging templates. Converts notification scripts into Roman Urdu (`roman_ur`) or Script Urdu (`script_ur`) based on NLU selections. Bypasses WhatsApp delivery errors by routing directly to local in-app notifications.
*   **Input Schema:** Output templates mapping booking IDs and language configs.
*   **Output Schema:**
    ```typescript
    interface NotificationResult {
        notificationId: string;
        channel: "push_notification" | "provider_dashboard";
        dispatched: boolean;
        messageBody: string;
    }
    ```

---

## 📄 8. Follow-Up Lifecycle Scheduler Agent (`followupSchedulerAgent.ts`)
Controls cron timers to coordinate safety, reviews, and monitoring checkups post-service.

*   **Logic & Algorithms:** Establishes a 6-stage lifecycle task matrix. Runs watchdog monitors at specific relative intervals (e.g., 30-minute arrival warnings, 2-hour completion timers, and 6-hour review alerts).
*   **Input Schema:** Booking confirmation events and timestamps.
*   **Output Schema:**
    ```typescript
    interface SchedulerOutput {
        scheduledTasks: Array<{
            taskId: string;
            taskType: "user_reminder" | "completion_check" | "review_request";
            executionTime: string;
        }>;
    }
    ```

---

## 📄 9. Service Quality Tracker Agent (`serviceQualityTrackerAgent.ts`)
Monitors active service phases and geo-location tracking for safety and punctuality.

*   **Logic & Algorithms:** Subscribes to provider coordinate streams, assesses lateness alerts, and updates live Firestore phases (`en_route`, `arrived`, `in_progress`, `completed`).
*   **Input Schema:** Real-time provider GPS packets and arrival timestamps.
*   **Output Schema:**
    ```typescript
    interface QualityTrackingState {
        trackingId: string;
        currentPhase: "en_route" | "arrived" | "in_progress" | "completed";
        isDelayed: boolean;
        estimatedArrivalDelayMinutes: number;
    }
    ```

---

## 📄 10. Provider Response Handler Agent (`providerResponseHandlerAgent.ts`)
Listens for provider clicks and triggers intelligent safety state transitions.

*   **Logic & Algorithms:** Oversees provider acceptances, declines, and timeouts:
    *   *Accept:* Lock other provider offers and transition state immediately to `confirmed`.
    *   *Decline:* Pull next highest-ranked provider candidate (`PRV-XXXX`) and dispatch offers immediately.
    *   *Timeout:* Watchdog triggers fallback. In Demo Mode, auto-confirms the reservation to prevent user friction. In Production Mode, transitions to next available fallback.
*   **Input Schema:** Provider action packets (`accept`, `decline`, `timeout`).
*   **Output Schema:**
    ```typescript
    interface ResponseHandlerState {
        resolvedStatus: "confirmed" | "declined" | "auto_confirmed_demo";
        activeProviderId: string;
        fallbackTriggered: boolean;
    }
    ```

---

## 📄 11. Dispute Auto-Resolution Agent (`disputeResolutionAgent.ts`)
Provides automated arbitration for cost issues, customer cancellations, and performance disputes.

*   **Logic & Algorithms:** Applies structured financial restitution algorithms. If a provider overcharges the client by $\ge 10\%$, it auto-resolves the dispute: issues an instant refund of the price discrepancy, processes a 20% loyalty compensation credit, and applies warning flags.
*   **Input Schema:**
    ```typescript
    interface DisputeInput {
        bookingId: string;
        claimedAmount: number;
        authorizedAmount: number;
        disputeReason: "overcharged" | "no_show" | "poor_quality";
    }
    ```
*   **Output Schema:**
    ```typescript
    interface DisputeResolution {
        disputeId: string;
        resolutionStatus: "auto_resolved" | "pending_manual_review";
        actionsTaken: string[];
        refundAmountIssued: number;
        providerPenaltiesApplied: { WarningCountIncrement: number; rankingPenaltyApplied: boolean };
    }
    ```

---

## 📄 12. Workload & Performance Optimization Agent (`providerOptimizationAgent.ts`)
Analyzes workload averages and delivers actionable, localized strategies to boost provider earnings.

*   **Logic & Algorithms:** Runs periodic batch analysis routines. Analyzes regional averages, spots underutilization patterns, and suggests local optimization targets (e.g., expanding boundaries to G-13 or DHA, or shifting slot hours).
*   **Input Schema:** Historical provider booking completions and earnings logs.
*   **Output Schema:**
    ```typescript
    interface ProviderOptimizationReport {
        providerId: string;
        utilizationScore: "underutilized" | "optimal" | "overloaded";
        averageEarningsGap: number;
        recommendations: Array<{
            priority: "high" | "medium" | "low";
            actionableStrategy: string;
            expectedEarningsBoost: number;
            concreteSteps: string[];
        }>;
    }
    ```

===============================================================================
  🛡️ END OF ECOSYSTEM SPECIFICATION • MULTI-AGENT PIPELINE FULLY COMPILING
===============================================================================
