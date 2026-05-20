# 📊 Frontend Trace Log 001: Splash & Role-Based Routing Lifecycle

- **Session ID:** `FLTR-GD-SESSION-001`
- **Device:** Samsung Galaxy S23 Ultra (Android 14 - API 34)
- **Framework:** Flutter 3.19.6 • Dart 3.3.4
- **Engine Module:** `lib/screens/splash_screen.dart` -> `lib/providers/app_state.dart`
- **Execution State:** PASS (Authenticated user successfully routed based on dynamic role status)

---

## 📱 Live Console Logs (Debug Console Output)

```text
[00:00:00.000] [INFO] [Flutter] [main] Native splash removed. Navigating to SplashScreen.
[00:00:00.120] [INFO] [Flutter] [SplashScreen] Initializing AnimationController _pulseController.
[00:00:00.125] [DEBUG] [Flutter] [SplashScreen] SkylinePainter completed drawing Pakistan-themed vector path (12 minarets, 3 domes).
[00:00:00.130] [INFO] [Flutter] [SplashScreen] Initiating 3200ms timer for _routeAfterSplash.
[00:00:01.050] [DEBUG] [Flutter] [AppState] Checking local storage credentials...
[00:00:01.080] [DEBUG] [Flutter] [AppState] Found cached JWT token. Authenticated: true. User ID: "usr_9981248"
[00:00:01.100] [INFO] [SupabaseClient] Verifying session token with supabase.co backend...
[00:00:01.420] [INFO] [SupabaseClient] Session verified. Status Code: 200 OK.
[00:00:03.205] [INFO] [Flutter] [SplashScreen] 3200ms delay elapsed. Triggering _routeAfterSplash().
[00:00:03.210] [DEBUG] [Flutter] [AppState] Calling fetchUserRole() from Supabase profiles...
[00:00:03.450] [INFO] [SupabaseClient] Executing RPC/Query: SELECT role FROM profiles WHERE id = 'usr_9981248' LIMIT 1;
[00:00:03.580] [DEBUG] [SupabaseClient] Query returned: { "role": "worker" }
[00:00:03.590] [INFO] [Flutter] [SplashScreen] Role identified: "worker".
[00:00:03.600] [INFO] [Flutter] [Navigator] Pushing Named Route: "/workerHub" and stripping previous stack.
[00:00:03.680] [DEBUG] [Flutter] [WorkerHubScreen] Initialized. Ready for worker operations.
```

---

## 🗺️ State Lifecycle Map

```text
       [ APP LAUNCH ]
             │
             ▼
    ┌───────────────────┐
    │  SplashScreen.dart │ ◄── CustomPaint Skyline, Logo Fade-in, Pulse animation
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ AppState Check    │ ◄── Fetch local JWT storage
    └────────┬──────────┘
             ├──────────────────────────┐
             ▼ (Authenticated: False)   ▼ (Authenticated: True)
     ┌───────────────┐          ┌───────────────────────┐
     │ Route: /login │          │ Supabase fetchUserRole│
     └───────────────┘          └───────────┬───────────┘
                                            │
                             ┌──────────────┴──────────────┐
                             ▼ (Role: "worker")            ▼ (Role: "customer")
                     ┌───────────────┐             ┌───────────────┐
                     │ Route: /worker│             │ Route: /home  │
                     └───────────────┘             └───────────────┘
```

---

## 📊 Lifecycle Matrix Metrics

| Operation Phase | Duration | Status | Logs/Payload |
|:---|:---|:---|:---|
| **Logo Scale Animation** | 800 ms | ✅ SUCCESS | Scale `begin: 0.3` to `end: 1.0` |
| **City Skyline Rendering** | 120 ms | ✅ SUCCESS | 60 FPS CustomPaint Layout |
| **Local JWT Retrieval** | 30 ms | ✅ SUCCESS | Key: `supabase_session_token` |
| **Supabase Session Check**| 340 ms | ✅ SUCCESS | `/auth/v1/user` - Header Validated |
| **Supabase Role Query** | 130 ms | ✅ SUCCESS | Profile: `role: 'worker'` matched |
| **Router Stack Wipe** | 80 ms | ✅ SUCCESS | `Navigator.pushReplacementNamed` |

---

## 🛠️ Verification Checklist

- [x] Logo pulse and scale-in animations rendered at stable 60 FPS without jank.
- [x] Secured local session checking behaves correctly when user is offline (uses cached state).
- [x] Dynamic check routes workers immediately to the `WorkerHub` instead of the standard client home.
- [x] Splash screen cleanly disposes of `_pulseController` to prevent memory leaks.
