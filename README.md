# AppForge Business Ultimate Android Test Wrapper

- WebView wrapper for the standalone HTML asset `app/src/main/assets/index.html`.
- App ID: `com.rmlala.appforgebusiness`
- Version: 1.1.1 (test wrapper)
- Min SDK: 23 / Target SDK: 35
- JavaScript + DOM storage enabled to support standalone app functionality and license/cloud API calls.
- HTTPS-only network policy; external non-file links are handed to the Android system.

## Build

Requires JDK 17, Android SDK platform 35/build tools 35.0.0, Gradle 8.9.

```bash
gradle assembleDebug assembleRelease
```

The release APK uses a temporary test signing key when built in the included GitHub workflow. It is not a production signing key. APKs from this workflow are for testing only.
