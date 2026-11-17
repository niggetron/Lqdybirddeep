# Lqdybirddeep — automatic Ladybird Android builds (arm64)

This repository builds the [Ladybird](https://github.com/LadybirdBrowser/ladybird) browser's
**Android app** from upstream sources, signs it, and publishes the APK to the
[Releases](../../releases/latest) tab.

- **Architecture:** `arm64-v8a` only (modern 64-bit phones/tablets)
- **Minimum Android:** 11 (API 30)
- **Signing:** a stable keystore stored in this repo's Actions secrets, so newer builds install
  over older ones without uninstalling

## When does it build?

| Trigger | What it builds |
| --- | --- |
| Push to `main` | Newest upstream Ladybird release tag, new version name |
| Manual run (`Run workflow`) | Optional `ladybird_ref` input: a tag, branch or commit |
| Weekly schedule (Mon 03:00 UTC) | Newest upstream Ladybird release tag |

Each build gets an updated Android `versionName`:

```
<upstream Ladybird version>+b<GitHub run number>     e.g. 2026.47.0+b12
```

and the matching release tag/APK:

```
release:  ladybird-android-2026.47.0-b12
asset:    Ladybird-2026.47.0-arm64-v8a-b12.apk
```

## What the workflow does

1. Picks the newest `YYYY.MM.P` tag from `LadybirdBrowser/ladybird` (or the ref you pass).
2. Installs the toolchain: Clang, Ninja, ccache, JDK 17, Rust (version pinned by Ladybird),
   Android SDK 35, NDK `29.0.13599879`, CMake 3.31.
3. Bootstraps vcpkg at Ladybird's pinned baseline and builds its ports for `arm64-android`.
4. Patches `UI/Android/build.gradle.kts` for an arm64-only release build and injects the
   version name/code.
5. Runs `./gradlew assembleRelease`.
6. Zip-aligns and signs the APK with `apksigner`, emits a `.sha256`.
7. Creates a GitHub Release with the APK attached and marks it as latest.

Builds are long (a full Ladybird/WebKit-class C++ build plus Skia/ANGLE/FFmpeg for Android),
so the workflow caches ccache and the vcpkg binary cache between runs. The first run of a new
upstream version is the slow one; later runs are much faster.

## Installing

Download the APK from the latest release and open it on the phone. Enable "install unknown
apps" for the app you open it from if Android asks.

## Notes

- The app is an early-stage port of Ladybird to Android; expect rough edges from upstream.
- No proprietary Google components are included.
- Build logs are uploaded as artifacts when a build fails.
