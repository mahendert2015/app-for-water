# Telugu Water Reminder App (`telugu_water_reminder`)

A cross-platform **Flutter** app for **your phone and your wife's phone** that:
1. Sends **hourly water reminders** during your configured waking hours (default `7:00 AM – 10:00 PM`).
2. Plays **Telugu voice sounds** (`telugu_water_reminder.wav` for background/locked-screen notifications + `flutter_tts` `te-IN` for rotating live Telugu voice prompts).
3. **Randomly asks for a live video check** (default 35% chance, configurable in Settings) while drinking water — verified **100% on-device** using Google ML Kit Face Detection (liveness blink check + 4-second live tracking) with **zero video saved or uploaded**.

---

## Project Structure

- `lib/main.dart` — App entry point & notification initialization
- `lib/models/app_settings.dart` — Phone profile (`My Phone` / `Wife's Phone`), wake/sleep hours, daily goal, random video check probability
- `lib/models/hydration_log.dart` — Local hydration entry model (`verifiedByVideo` flag)
- `lib/services/telugu_voice_service.dart` — Telugu (`te-IN`) Text-to-Speech phrases
- `lib/services/notification_service.dart` — Hourly exact-alarm notifications with custom Telugu `.wav` sound
- `lib/services/storage_service.dart` — Local `SharedPreferences` persistence
- `lib/services/partner_sync_service.dart` — Optional couple-sync hook (runs 100% offline by default)
- `lib/screens/home_screen.dart` — Daily progress, Telugu test button, and log history
- `lib/screens/video_check_screen.dart` — On-device live camera + ML Kit liveness & drinking verification
- `lib/screens/settings_screen.dart` — Profile & schedule configuration
- `android/app/src/main/res/raw/telugu_water_reminder.wav` — Bundled Telugu notification audio
- `ios/Runner/telugu_water_reminder.wav` — Bundled Telugu notification audio for iOS

---

## Setup & Installation on Both Phones

### 1. Prerequisites
- Install the [Flutter SDK](https://docs.flutter.dev/get-started/install) (3.24+ / Dart 3.4+).
- Install **Android Studio** (for Android APK) and/or **Xcode** (on macOS for iPhone).

### 2. Initialize Platform Wrapper Files
Unzip `telugu_water_reminder.zip`, open a terminal inside `telugu_water_reminder/`, and run:

```bash
flutter create . --org com.example
flutter pub get
```

> `flutter create .` generates the Gradle (`android/build.gradle.kts`, `gradlew`) and Xcode (`Runner.xcodeproj`) wrapper files while keeping `lib/`, `pubspec.yaml`, `AndroidManifest.xml`, `Info.plist`, and the Telugu `.wav` audio intact.

### 3. Android Minimum SDK & Desugaring
In `android/app/build.gradle.kts` (or `build.gradle`), ensure:
- `minSdk = 23` (required by `google_mlkit_face_detection` & `camera`)
- Core library desugaring enabled (required by `flutter_local_notifications`):
  ```kotlin
  compileOptions {
      isCoreLibraryDesugaringEnabled = true
      sourceCompatibility = JavaVersion.VERSION_17
      targetCompatibility = JavaVersion.VERSION_17
  }
  ```
  And in `dependencies`:
  ```kotlin
  coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.4")
  ```

### 4. Build & Install on Both Phones
- **Android (USB debugging enabled on your phone & wife's phone):**
  ```bash
  flutter build apk --release
  ```
  Install `build/app/outputs/flutter-apk/app-release.apk` on both phones.
- **iOS (Mac + Xcode):**
  Ensure `telugu_water_reminder.wav` is added to the Xcode **Runner** target's *Copy Bundle Resources*, then run:
  ```bash
  flutter run --release
  ```

### 5. First-Time Phone Setup
1. Open the app on **your phone** → tap **Settings** → select **My Phone**.
2. Open the app on **your wife's phone** → tap **Settings** → select **Wife's Phone**.
3. On Android, make sure **Telugu (India)** voice data is enabled in *Settings → System → Languages & input → Text-to-speech output* (Google Speech Recognition & Synthesis) for `flutter_tts`. The bundled `.wav` notification sound works even without TTS data.
4. Grant **Notifications**, **Alarms & Reminders**, and **Camera** permissions when prompted.

---

## Optional: Live Partner Sync (No Video Shared)
By default, both phones work independently offline. If you want each phone to display the other's daily glass count:
1. Create a free [Firebase](https://firebase.google.com/) project and add `firebase_core` + `cloud_firestore` to `pubspec.yaml`.
2. Implement `pushLog` and `fetchPartnerTodayCount` in `lib/services/partner_sync_service.dart` using a shared **Couple Sync Code** (only `phoneRole`, `timestamp`, and `verifiedByVideo` are synced — **never** camera frames or video).
