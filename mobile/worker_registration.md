# 📊 Frontend Trace Log 007: Worker Registration & Onboarding Lifecycle

- **Session ID:** `FLTR-GD-SESSION-007`
- **Device:** Infinix Note 30 (Android 13)
- **Framework:** Flutter 3.19.6 • Dart 3.3.4
- **Engine Module:** `lib/screens/worker_profile_setup_screen.dart` -> `lib/providers/app_state.dart`
- **Execution State:** PASS (Worker registration form compiled, profile validated, GPS attached, record inserted into unified Supabase `workers` table)

---

## 📱 Live Console Logs (Debug Console Output)

```text
[00:02:15.100] [INFO] [Flutter] [WorkerProfileSetupScreen] Worker tapped Submit Registration Button. Initiating field validations.
[00:02:15.110] [DEBUG] [Flutter] [WorkerProfileSetup] Name validation: PASS ("Javed Iqbal").
[00:02:15.115] [DEBUG] [Flutter] [WorkerProfileSetup] Phone validation: PASS ("+923159988776").
[00:02:15.120] [DEBUG] [Flutter] [WorkerProfileSetup] Service category resolved: PASS ("electrician").
[00:02:15.125] [DEBUG] [Flutter] [WorkerProfileSetup] Base hourly rate parsed: PASS (PKR 850 / hr).
[00:02:15.130] [INFO] [Flutter] [AppState] Resolving GPS location for worker registry binding...
[00:02:15.220] [DEBUG] [Flutter] [AppState] GPS acquired: Lat: 33.6844, Lng: 73.0479 (Accuracy: 5.1m).
[00:02:15.230] [INFO] [Flutter] [WorkerProfileSetup] Verification file validation: PASS (CNIC Front and Back images loaded).
[00:02:15.240] [INFO] [Flutter] [AppState] Submitting unified registration package: submitWorkerRegistration().
[00:02:15.250] [DEBUG] [SupabaseClient] Commencing database INSERT into unified table:
                       INSERT INTO workers (
                         user_id, name, phone, service_type, base_rate,
                         last_lat, last_lng, is_available, is_verified,
                         cnic_front_url, cnic_back_url
                       ) VALUES (
                         'usr_3821948', 'Javed Iqbal', '+923159988776', 'electrician', 850,
                         33.6844, 73.0479, true, false,
                         'cnic/usr_3821948_front.jpg', 'cnic/usr_3821948_back.jpg'
                       );
[00:02:15.540] [INFO] [SupabaseClient] Database transaction completed successfully (1 row created).
[00:02:15.550] [INFO] [Flutter] [AppState] Updating local registration status state: registered=true, role='worker'.
[00:02:15.560] [INFO] [Flutter] [Navigator] Registration complete. Redirecting to WorkerHubScreen.
[00:02:15.680] [DEBUG] [Flutter] [WorkerHubScreen] Hub active. Status: "online_available" (GPS background broadcast loop started).
```

---

## 🗺️ Worker Registration Lifecycle Flow

```text
    [ Form Validation ]
             │
             ▼
    ┌───────────────────┐
    │  Input Verification│ ◄── Name, phone, rate, and category validation
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Resolve Coordinates│ ◄── Pin precise home-base registration GPS coords
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ submitWorkerReg() │ ◄── Dispatch unified registration transaction
    └────────┬──────────┘
             ├──────────────────────────┐
             ▼ (Success: 200 Created)   ▼ (Error/Timeout)
     ┌───────────────────────┐          ┌───────────────────────┐
     │ Update Local User Role│          │ Release Locked DB /   │
     │ Cache to 'worker'     │          │ Pop Validation Toast  │
     └──────────┬────────────┘          └───────────────────────┘
                ▼
     ┌───────────────────────┐
     │ Navigate: /workerHub  │ ◄── Transition UI view and start status loops
     └───────────────────────┘
```

---

## 📊 Registration Performance Metrics

| Operation Phase | Duration | Status | Logs/Payload |
|:---|:---|:---|:---|
| **Form Fields Validation**| 10 ms | ✅ SUCCESS | All client fields checked |
| **GPS Fix Acquisition** | 90 ms | ✅ SUCCESS | Lat: 33.6844, Lng: 73.0479 |
| **Asset URL Packaging** | 30 ms | ✅ SUCCESS | CNIC storage links generated |
| **Supabase DB Insert** | 290 ms | ✅ SUCCESS | Inserted cleanly, Latency: 290ms |
| **Local Cache Updates** | 5 ms | ✅ SUCCESS | Role set: "worker" |
| **Hub Transition UI** | 120 ms | ✅ SUCCESS | Redirected to primary Worker Hub |

---

## 🛠️ Verification Checklist

- [x] Input validations prevent empty, out-of-range, or formatting anomalies in worker phone rates or entries.
- [x] Correctly binds worker location at submit time to ensure they are immediately discoverable on the home maps.
- [x] All CNIC files upload securely to Supabase Storage before database record creation occurs.
- [x] Implements robust retry options to prevent data loss if a transient network disconnect happens mid-submission.
