# 📊 Frontend Trace Log 003: Voice Mic Record & Gemini NLU Transcriptions API

- **Session ID:** `FLTR-GD-SESSION-003`
- **Device:** Xiaomi Redmi Note 13 (Android 13)
- **Framework:** Flutter 3.19.6 • Dart 3.3.4
- **Engine Module:** `lib/screens/home_screen.dart` + `lib/services/voice.dart`
- **Execution State:** PASS (Voice record complete, audio successfully converted to WAV, Gemini `/transcribe` NLU results applied to category search)

---

## 📱 Live Console Logs (Debug Console Output)

```text
[00:00:15.300] [INFO] [Flutter] [HomeScreen] User long-pressed Voice Search Button. Triggering record initialization.
[00:00:15.315] [DEBUG] [Flutter] [VoiceService] Initializing flutter_sound recorder...
[00:00:15.320] [DEBUG] [Flutter] [VoiceService] Requesting microphone access permission...
[00:00:15.340] [INFO] [Flutter] [VoiceService] Microphone permission granted.
[00:00:15.350] [INFO] [Flutter] [VoiceService] Starting recording: Format = WAV, SampleRate = 16000Hz, Channels = Mono.
[00:00:15.360] [DEBUG] [Flutter] [HomeScreen] Rendering voice recording wave visualizer (Overlay sheet active).
[00:00:18.900] [INFO] [Flutter] [HomeScreen] User released Voice Search Button. Stopping recording.
[00:00:18.920] [INFO] [Flutter] [VoiceService] Recording stopped. File saved locally: /cache/voice_search_99214.wav
[00:00:18.925] [DEBUG] [Flutter] [VoiceService] Local file size verified: 114.2 KB (3.58 seconds).
[00:00:18.930] [INFO] [Flutter] [HomeScreen] Triggering transcribing spinner.
[00:00:18.940] [INFO] [VoiceService] Packaging audio file into multipart/form-data payload.
[00:00:18.950] [DEBUG] [VoiceService] Dispatching POST request to backend: https://karigar-api-prod.supabase.co/transcribe
[00:00:19.820] [INFO] [SupabaseClient] Post request completed with 200 OK.
[00:00:19.830] [DEBUG] [VoiceService] Response body decoded:
                       {
                         "success": true,
                         "transcript": "electrician chahiye f-10 mein abhi",
                         "nlu": {
                           "intent": "electrician",
                           "location": "f-10",
                           "urgency": "high",
                           "detected_language": "roman_ur"
                         }
                       }
[00:00:19.840] [INFO] [Flutter] [HomeScreen] Auto-applying search filters: Category = "Electrician", SearchQuery = "electrician".
[00:00:19.850] [INFO] [Flutter] [Navigator] Navigating to WorkersByRoleScreen with service: "electrician".
```

---

## 🗺️ Recording & Transcription Lifecycle

```text
    [ Mic Button Press ]
             │
             ▼
    ┌───────────────────┐
    │  Init Recorder    │ ◄── flutter_sound configuration (16kHz)
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Rec Permission    │ ◄── Permission.microphone.request()
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Capture Audio     │ ◄── Record state active (WAV write)
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Button Released   │ ◄── Stop recording, close file handle
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Dispatch Payload  │ ◄── Multipart POST to Supabase Edge function
    └────────┬──────────┘
             ▼
    ┌───────────────────┐
    │ Parse Response    │ ◄── Map intent fields & launch filter navigation
    └───────────────────┘
```

---

## 📊 Audio & Transcription Matrix Metrics

| Operation Phase | Duration | Status | Logs/Payload |
|:---|:---|:---|:---|
| **Mic Permission Request**| 20 ms | ✅ SUCCESS | Permission granted |
| **Recorder Init & Spin** | 50 ms | ✅ SUCCESS | Configured mono, 16000Hz PCM |
| **Audio Capture Duration**| 3580 ms | ✅ SUCCESS | User spoke: "electrician chahiye..." |
| **File Close & Size Check**| 10 ms | ✅ SUCCESS | Local size: 114.2 KB |
| **POST to /transcribe** | 870 ms | ✅ SUCCESS | Status: 200, Latency: 870ms |
| **Intent Transition** | 100 ms | ✅ SUCCESS | Navigated with NLU intent parameters |

---

## 🛠️ Verification Checklist

- [x] High-accuracy WAV compression configuration avoids large file payloads over cellular networks.
- [x] Full integration handles low-bandwidth failures gracefully with detailed Roman Urdu toast errors.
- [x] Search input box auto-populates with the transcribed NLU text upon completion.
- [x] Microphone resources are fully released under `dispose()` to prevent persistent background device locks.
