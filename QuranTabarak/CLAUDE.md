# CLAUDE.md — دليل البناء لـ Claude Code

> الهدف: بناء ملف **APK** من هذا المشروع وتشغيله على هاتف أندرويد.
> هذا تطبيق WebView بسيط يحمّل ملف HTML أوفلاين بالكامل من مجلد assets.
> التطبيق لا يطلب صلاحية إنترنت — يعمل بدون نت ١٠٠٪.

---

## What this project is

A minimal native Android app (Java + WebView) that loads a single bundled,
fully offline HTML file from `app/src/main/assets/index.html`.

- Build system: Gradle, Android Gradle Plugin (AGP) **8.5.2**
- Gradle version target: **8.7** (8.7–8.9 are fine)
- `compileSdk 34`, `minSdk 24`, `targetSdk 34`
- Language level: Java 17
- One dependency: `androidx.appcompat:appcompat:1.7.0`
- No INTERNET permission (offline by design)

## Primary goal

Produce an installable **debug APK** at:
`app/build/outputs/apk/debug/app-debug.apk`

The debug APK is auto-signed with the debug keystore and installs directly
on any device (enable "install from unknown sources").

---

## Build steps (do these in order)

1. **JDK** — ensure JDK 17+ is available (`java -version`). If missing, install
   a JDK 17 (e.g. Temurin) and set `JAVA_HOME`.

2. **Android SDK** — if no SDK is present:
   - Download Android **command-line tools** from Google.
   - Place under `$ANDROID_HOME/cmdline-tools/latest/`.
   - Export `ANDROID_HOME` (and `ANDROID_SDK_ROOT` to the same path).
   - Install required packages and accept licenses:
     ```
     yes | sdkmanager --licenses
     sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
     ```
   - Create `local.properties` in the project root:
     ```
     sdk.dir=/absolute/path/to/Android/sdk
     ```
   (If Android Studio is installed, the SDK already exists — just point
   `local.properties` / `ANDROID_HOME` at it and skip the download.)

3. **Gradle wrapper** — the wrapper JAR is intentionally NOT bundled.
   Generate it once using a system Gradle (install Gradle 8.7+ if needed via
   SDKMAN/brew/apt), then build with the wrapper:
   ```
   gradle wrapper --gradle-version 8.7
   ./gradlew assembleDebug
   ```
   Alternatively, skip the wrapper and build directly with a system Gradle 8.7+:
   ```
   gradle assembleDebug
   ```

4. **Locate the APK** and report its path to the user:
   `app/build/outputs/apk/debug/app-debug.apk`

---

## Notes & gotchas

- Accept SDK licenses non-interactively with `yes | sdkmanager --licenses`.
- If AGP/Gradle complain about version mismatch, keep Gradle in the 8.7–8.9
  range for AGP 8.5.2.
- The app entry point is `MainActivity.java`; it just loads
  `file:///android_asset/index.html` into a WebView.
- To update app content later: replace `app/src/main/assets/index.html` with a
  new HTML version and rebuild. App version is in `app/build.gradle`
  (`versionCode` / `versionName`).
- The in-app audio (pronunciation) uses the browser Web Speech API and may be
  silent inside a bare WebView on some devices. If the user needs guaranteed
  audio, wire Android's native TextToSpeech via a JavaScript interface.

## Optional: signed release APK

For Play Store distribution, generate a keystore and a release `signingConfig`,
then run `./gradlew assembleRelease`. Not needed for personal/offline use.
