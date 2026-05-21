# Karigar — Vercel Deployment Trace Log

> **Deployed:** 2026-05-21 05:02 PKT  
> **Status:** ● Ready (Production)  
> **Live URL:** [https://karigar-pk.vercel.app](https://karigar-pk.vercel.app)

---

## 1. Flutter Web Build

**Command:** `flutter build web`  
**CWD:** `/Users/macbookpro/Desktop/karigar/mobile`  
**Duration:** 36.9s  

### Dependency Resolution

```
Resolving dependencies...
Downloading packages...
> characters 1.4.1 (was 1.4.0)
  cli_util 0.4.2 (0.5.1 available)
  flutter_map 7.0.2 (8.3.0 available)
  geolocator 13.0.4 (14.0.2 available)
  geolocator_android 4.6.2 (5.0.2 available)
  latlong2 0.9.1 (0.10.1 available)
> matcher 0.12.19 (was 0.12.17) (0.12.20 available)
> material_color_utilities 0.13.0 (was 0.11.1)
> meta 1.18.0 (was 1.17.0) (1.18.2 available)
  mgrs_dart 2.0.0 (3.0.0 available)
  path_provider_foundation 2.5.1 (2.6.0 available)
  permission_handler 11.4.0 (12.0.1 available)
  permission_handler_android 12.1.0 (13.0.1 available)
  proj4dart 2.1.0 (3.0.0 available)
> test_api 0.7.11 (was 0.7.7) (0.7.12 available)
  unicode 0.3.1 (1.1.9 available)
  vector_math 2.2.0 (2.3.0 available)
  xml 6.6.1 (7.0.1 available)

Changed 5 dependencies!
16 packages have newer versions incompatible with dependency constraints.
```

### Plugin Warnings

```
The following plugins do not support Swift Package Manager for ios:
  - permission_handler_apple
  - google_maps_flutter_ios
This will become an error in a future version of Flutter.
```

### Compilation Output

```
Compiling lib/main.dart for the Web...                             36.9s
✓ Built build/web

Wasm dry run succeeded. Consider building with the --wasm flag.

Font asset "CupertinoIcons.ttf" was tree-shaken:
  257,628 → 1,472 bytes (99.4% reduction)

Font asset "MaterialIcons-Regular.otf" was tree-shaken:
  1,645,184 → 20,916 bytes (98.7% reduction)
```

> [!TIP]
> Font tree-shaking saved ~1.88 MB by removing unused icon glyphs.

---

## 2. Vercel Deployment

**Command:** `vercel build/web --prod`  
**Source Directory:** `~/Desktop/karigar/mobile/build/web`  
**Files Uploaded:** 52  
**Upload Size:** 8.4 MB  

### Project Setup

| Setting | Value |
|---|---|
| **Scope** | `backabdullah18-6837's projects` |
| **Project Name** | `karigar-pk` |
| **Link to Existing** | No (new project) |
| **Code Directory** | `./` |
| **Framework** | None detected (static) |
| **Build Command** | Skipped (pre-built) |
| **Output Directory** | `.` (root) |

---

## 3. Build Trace (Vercel Server-Side)

```
2026-05-21T00:02:13.878Z  Running build in Washington, D.C., USA (East) – iad1
2026-05-21T00:02:13.879Z  Build machine configuration: 2 cores, 8 GB
2026-05-21T00:02:13.996Z  Retrieving list of deployment files...
2026-05-21T00:02:13.999Z  Previous build caches not available.
2026-05-21T00:02:14.578Z  Downloading 52 deployment files...
2026-05-21T00:02:15.669Z  Running "vercel build"
2026-05-21T00:02:15.692Z  Vercel CLI 54.2.0
2026-05-21T00:02:15.925Z  Build Completed in /vercel/output [160ms]
2026-05-21T00:02:16.002Z  Deploying outputs...
2026-05-21T00:02:17.938Z  Deployment completed
2026-05-21T00:02:18.051Z  Creating build cache...
2026-05-21T00:02:18.065Z  Skipping cache upload because no files were prepared
```

**Total server-side time:** ~4.2 seconds

---

## 4. Deployment Inspection

```
  General

    id        dpl_7umMhYMiVEYSkMw5VrHdqeaC6Bqj
    name      karigar-pk
    target    production
    status    ● Ready
    url       https://karigar-n2d41pslz-backabdullah18-6837s-projects.vercel.app
    created   Thu May 21 2026 05:02:13 GMT+0500 (Pakistan Standard Time)
```

### Aliases

| Alias URL |
|---|
| https://karigar-pk.vercel.app |
| https://karigar-pk-backabdullah18-6837s-projects.vercel.app |
| https://karigar-pk-backabdullah18-6837-backabdullah18-6837s-projects.vercel.app |

### Builds

| Source | Duration |
|---|---|
| `.` (static) | 0ms |

---

## 5. Runtime Logs

```
Streaming logs for deployment dpl_7umMhYMiVEYSkMw5VrHdqeaC6Bqj

No runtime logs — static deployment (Edge Network CDN).
All assets served directly from Vercel's Edge without server-side execution.
```

> [!NOTE]
> Since this is a static Flutter web build (HTML/JS/CSS), there are no server-side runtime logs.
> All requests are served from Vercel's CDN edge nodes. Runtime-level request logs
> are only available on Vercel Pro/Enterprise plans via Log Drains or the dashboard.

---

## 6. Summary

| Metric | Value |
|---|---|
| **Flutter build time** | 36.9s |
| **Upload size** | 8.4 MB (52 files) |
| **Vercel build time** | 160ms |
| **Deploy time** | ~4s |
| **Total end-to-end** | ~45s |
| **Production URL** | https://karigar-pk.vercel.app |
| **Deployment ID** | `dpl_7umMhYMiVEYSkMw5VrHdqeaC6Bqj` |
| **Region** | iad1 (Washington, D.C.) |
| **CDN** | Vercel Edge Network (global) |
| **Errors** | 0 |
| **Warnings** | 2 (SPM plugin compat) |

---

## Quick Re-deploy Commands

```bash
# Full rebuild + deploy
cd /Users/macbookpro/Desktop/karigar/mobile
flutter build web && vercel build/web --prod

# Inspect latest deployment
vercel inspect karigar-pk.vercel.app

# View build logs
vercel inspect karigar-pk.vercel.app --logs

# Stream runtime logs (Pro plan)
vercel logs karigar-pk.vercel.app
```
