# Module 20 – Concepts: Native Platforms Bridge

> **Mục tiêu**: Nắm vững 6 concepts cốt lõi về native platform integration — Platform Channels, iOS configuration, Android configuration, app signing, native debugging, và conditional imports.

📌 **Recap**: M0 (Dart basics) · M1 (WidgetsFlutterBinding) · M3 (Config/env)

---

## Concept 1: Platform Channels — MethodChannel & EventChannel

<!-- AI_VERIFY: concept-platform-channels -->

> ⚠️ **Lưu ý quan trọng về project này:**
> - Project `base_flutter` sử dụng Firebase plugins cho hầu hết native functionality
> - Platform channel chỉ tồn tại cho các feature cụ thể (ví dụ: `jp.flutter.app` cho badge count)
> - Code examples dưới đây là **general patterns** — thay channel name phù hợp khi implement

### 1.1 MethodChannel — Request/Response

```dart
// Dart side
// ⚠️ Channel name phải khớp với native side
// Trong project này, badge_helper.dart dùng: 'jp.flutter.app'
const channel = MethodChannel('jp.flutter.app');

Future<String?> clearBadge() async {
  try {
    final result = await channel.invokeMethod<String>('clearBadgeCount');
    return result;
  } on PlatformException catch (e) {
    debugPrint('Error: ${e.message}');
    return null;
  }
}
```

```swift
// iOS side (AppDelegate.swift)
// ⚠️ Đây là actual code trong base_flutter
let notificationChannel = FlutterMethodChannel(
  name: "jp.flutter.app",  // ← Channel name thực tế trong project
  binaryMessenger: engine.applicationRegistrar.messenger()
)

notificationChannel.setMethodCallHandler({
  (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
  if call.method == "clearBadgeCount" {
    clearBadgeCount()
    result(nil)
  } else {
    result(FlutterMethodNotImplemented)
  }
})
```

<!-- AI_VERIFY: based-on AppDelegate.swift line 26-39 -->

### 1.2 EventChannel — Streaming Data

```dart
// Dart side
const eventChannel = EventChannel('com.example.app/events');

Stream<String> getEventStream() {
  return eventChannel
      .receiveBroadcastStream()
      .map((event) => event as String);
}
```

```swift
// iOS side
let eventChannel = FlutterEventChannel(
  name: "com.example.app/events",
  binaryMessenger: controller.binaryMessenger
)

eventChannel.setStreamHandler(EventStreamHandler())

class EventStreamHandler: NSObject, FlutterStreamHandler {
  func onListen(withArguments arguments: Any?,
                eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    // Start sending events
    events("event_1")
    events("event_2")
    return nil
  }

  func onCancel(withArguments arguments: Any?) -> FlutterError? {
    // Stop sending events
    return nil
  }
}
```

### 1.3 MethodChannel vs EventChannel

| Aspect | MethodChannel | EventChannel |
|--------|--------------|--------------|
| Pattern | Request-response | Pub-sub/streaming |
| Use case | One-time operations | Continuous updates |
| Example | Get device info, clear badge | Battery level, sensor data |
| Dart API | `invokeMethod()` | `receiveBroadcastStream()` |
| Native API | `setMethodCallHandler` | `setStreamHandler` |

> 💡 **FE Perspective**
> **Flutter:** MethodChannel ≈ `fetch()` (request-response). EventChannel ≈ WebSocket/SSE (streaming).
> **React Native tương đương:** MethodChannel = `NativeModules.method()`. EventChannel = `NativeEventEmitter`.

---

## Concept 2: iOS Configuration — Info.plist & Capabilities

<!-- AI_VERIFY: concept-ios-config -->

### 2.1 Common Info.plist Permissions

| Key | Permission | When Needed |
|-----|-----------|-------------|
| `NSCameraUsageDescription` | Camera access | Photo capture, QR scanning |
| `NSPhotoLibraryUsageDescription` | Photo library | Select profile pictures |
| `NSLocationWhenInUseUsageDescription` | Location (foreground) | Show nearby content |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Location (background) | Geofencing, background tracking |
| `NSFaceIDUsageDescription` | Face ID | Biometric authentication |
| `NSMicrophoneUsageDescription` | Microphone | Voice recording |
| `UIBackgroundModes` | Background modes | Push notifications, background fetch |

### 2.2 Background Modes

```xml
<key>UIBackgroundModes</key>
<array>
  <string>remote-notification</string>  <!-- Push notifications -->
  <string>fetch</string>               <!-- Background fetch -->
  <string>location</string>            <!-- Location updates -->
  <string>processing</string>          <!-- Background processing -->
</array>
```

### 2.3 Podfile Configuration

```ruby
# Minimum iOS version
platform :ios, '14.0'

# Post-install hook for build settings
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      # Set deployment target
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'

      # Enable Bitcode (deprecated)
      config.build_settings['ENABLE_BITCODE'] = 'NO'

      # Swift version
      config.build_settings['SWIFT_VERSION'] = '5.0'
    end
  end
end
```

### 2.4 Build Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `IPHONEOS_DEPLOYMENT_TARGET` | 14.0 | Minimum iOS version |
| `SWIFT_VERSION` | 5.0 | Swift language version |
| `CODE_SIGN_STYLE` | Automatic | Auto-managed signing |
| `ENABLE_BITCODE` | NO | Bitcode (deprecated) |

> 💡 **FE Perspective**
> **Flutter iOS config** ≈ React Native `ios/YourApp/Info.plist` + `ios/Podfile`.
> **Key difference:** Flutter auto-generates `GeneratedPluginRegistrant` via CocoaPods.

---

## Concept 3: Android Configuration — Manifest & build.gradle

<!-- AI_VERIFY: concept-android-config -->

### 3.1 AndroidManifest.xml Permissions

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <!-- Network -->
  <uses-permission android:name="android.permission.INTERNET"/>

  <!-- Camera & Storage -->
  <uses-permission android:name="android.permission.CAMERA"/>
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

  <!-- Location -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

  <!-- Notifications -->
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
  <uses-permission android:name="android.permission.VIBRATE"/>

  <!-- Biometric -->
  <uses-permission android:name="android.permission.USE_BIOMETRIC"/>

  <!-- Boot receiver (for scheduled notifications) -->
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
</manifest>
```

### 3.2 SDK Version Configuration

| Version | Purpose | Recommendation |
|---------|---------|----------------|
| `minSdk` | Minimum Android version | 23 (Android 6.0) for modern features |
| `targetSdk` | Target Android version | Latest stable (34) |
| `compileSdk` | Compile SDK version | Should match targetSdk |

```groovy
android {
  defaultConfig {
    minSdk 23
    targetSdk 34
    // ...
  }

  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

### 3.3 ProGuard/R8 Configuration

```properties
# android/app/proguard-rules.pro

# Keep Flutter classes
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

# Keep Firebase classes
-keep class com.google.firebase.** { *; }

# Keep native methods
-keepclasseswithmembernames class * {
    native <methods>;
}
```

### 3.4 App Components

```xml
<application>
  <activity
      android:name=".MainActivity"
      android:exported="true"
      android:launchMode="singleTop">
    <intent-filter>
      <action android:name="android.intent.action.MAIN"/>
      <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
  </activity>
</application>
```

> 💡 **FE Perspective**
> **Flutter Android config** ≈ React Native `android/app/src/main/AndroidManifest.xml`.
> **Key difference:** Flutter uses `FlutterActivity` as entry point, React Native uses `MainActivity` directly.

---

## Concept 4: App Signing — Certificates & Keystores

<!-- AI_VERIFY: concept-app-signing -->

### 4.1 iOS Signing Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    iOS Signing Process                       │
├──────────────────────────────────────────────────────────────┤
│  1. Create Certificate Signing Request (CSR)                 │
│     └─ Keychain → Certificate Assistant → Save to disk        │
│                                                              │
│  2. Request Certificate from Apple Developer Portal           │
│     └─ Upload CSR → Download .cer file                        │
│                                                              │
│  3. Create Provisioning Profile                              │
│     └─ Select App ID → Select Certificate → Choose devices    │
│                                                              │
│  4. Import to Xcode                                          │
│     └─ Double-click .cer → Select profile in Xcode            │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Android Signing Flow

```
┌──────────────────────────────────────────────────────────────┐
│                   Android Signing Process                    │
├──────────────────────────────────────────────────────────────┤
│  1. Generate keystore (one-time)                             │
│     └─ keytool -genkey -v -keystore my-release-key.jks \     │
│                 -keyalg RSA -keysize 2048 -validity 10000    │
│                                                              │
│  2. Configure build.gradle                                   │
│     └─ Add signingConfig with keystore path & credentials    │
│                                                              │
│  3. Build release APK/AAB                                    │
│     └─ ./gradlew assembleRelease                             │
│                                                              │
│  4. Sign & align                                            │
│     └─ ./gradlew signingConfig signingRelease                │
└──────────────────────────────────────────────────────────────┘
```

### 4.3 Build Variants

| Variant | Purpose | Signing | Minify |
|---------|---------|---------|--------|
| `debug` | Development | Development cert | No |
| `release` | Production | Distribution cert | Yes |
| `profile` | Performance testing | Distribution cert | Partial |

> 💡 **FE Perspective**
> **Flutter signing** ≈ React Native `react-native run-ios --configuration Release`.
> **Key difference:** Flutter uses `key.properties`, React Native uses `xcodebuild -sdk`.

---

## Concept 5: Native Debugging

<!-- AI_VERIFY: concept-native-debugging -->

### 5.1 Debugging iOS in Xcode

| Action | How |
|--------|-----|
| Set breakpoint | Click gutter in Swift file |
| Watch variable | Right-click → Watch |
| Inspect memory | Debug → View Memory Graph |
| Network inspection | Xcode → Network Link Conditioner |

### 5.2 Debugging Android in Android Studio

| Action | How |
|--------|-----|
| Set breakpoint | Click gutter in Kotlin file |
| Logcat | View → Tool Windows → Logcat |
| Network inspection | Network Inspector tab |
| Database inspection | App Inspector |

### 5.3 Platform Message Debugging

```dart
// Debug: Print all method channel calls
// ⚠️ Thay 'jp.flutter.app' bằng channel name bạn đang dùng
const channel = MethodChannel('jp.flutter.app')
  ..setMethodCallHandler((call) {
    print('Method: ${call.method}, Args: ${call.arguments}');
    // Handle method
  });
```

### 5.4 Flutter DevTools — Platform Tab

```
Flutter DevTools
└── Platform → View platform channel messages
    ├── Method calls
    ├── Event streams
    └── Platform exceptions
```

> 💡 **FE Perspective**
> **Flutter native debug** = Xcode/Android Studio debugger for native code.
> **React Native:** Use Safari Web Inspector for iOS, Chrome DevTools for Android.

---

## Concept 6: Conditional Imports — Platform-Specific Code

<!-- AI_VERIFY: concept-conditional-imports -->

### 6.1 Platform Detection

```dart
import 'dart:io' show Platform;

void main() {
  if (Platform.isIOS) {
    // iOS-specific code
    print('iOS: ${Platform.operatingSystem}');
  } else if (Platform.isAndroid) {
    // Android-specific code
    print('Android: ${Platform.operatingSystem}');
  }
}
```

### 6.2 Stub Pattern

```dart
// lib/utils/file_utils.dart
import 'file_utils_stub.dart'
    if (dart.library.io) 'file_utils_io.dart'
    if (dart.library.html) 'file_utils_web.dart';

abstract class FileUtils {
  Future<void> saveFile(String path, List<int> bytes);
  Future<List<int>> readFile(String path);

  static FileUtils get instance => _instance;
  static FileUtils _instance = FileUtilsImpl();
}
```

```dart
// lib/utils/file_utils_io.dart (mobile implementation)
import 'dart:io';
import 'file_utils.dart';

class FileUtilsImpl implements FileUtils {
  @override
  Future<void> saveFile(String path, List<int> bytes) async {
    await File(path).writeAsBytes(bytes);
  }

  @override
  Future<List<int>> readFile(String path) async {
    return await File(path).readAsBytes();
  }
}
```

### 6.3 When to Use Conditional Imports

| Use Case | Solution |
|----------|---------|
| Platform-specific implementations | Conditional imports |
| Platform detection only | `Platform.isIOS` |
| Plugin platform code | Plugin handles it |

### 6.4 Platform-Specific Helper Pattern

```dart
// lib/common/helper/badge_helper.dart
// ⚠️ Đây là actual implementation pattern trong base_flutter
import 'package:flutter/foundation.dart';

class BadgeHelper {
  static Future<void> clearBadge() async {
    if (kIsWeb) {
      // Web doesn't have badge
      return;
    }

    try {
      const channel = MethodChannel('jp.flutter.app');
      await channel.invokeMethod('clearBadgeCount');
    } catch (e) {
      debugPrint('Failed to clear badge: $e');
    }
  }
}
```

> 📝 **Lưu ý:** BadgeHelper là ví dụ về platform-specific implementation trong project. iOS native code (AppDelegate.swift) xử lý `clearBadgeCount` method bằng cách set `UIApplication.shared.applicationIconBadgeNumber = 0`.

<!-- AI_VERIFY: based-on AppDelegate.swift line 26-52 -->

> 💡 **FE Perspective**
> **Flutter conditional imports** ≈ React Native platform-specific modules (`MyComponent.ios.ts`, `MyComponent.android.ts`).
> **Key difference:** Flutter uses compile-time conditional imports, React Native uses runtime detection.

---

## Quick Reference

| Concept | iOS | Android |
|---------|-----|--------|
| Entry point | `AppDelegate.swift` | `MainActivity.kt` |
| Permissions | `Info.plist` | `AndroidManifest.xml` |
| Dependencies | `Podfile` + CocoaPods | `build.gradle` |
| Native code | Swift/Objective-C | Kotlin/Java |
| Signing | Certificates + Profiles | Keystore + key.properties |

### Platform Channel Quick Reference

| Flutter | iOS Native | Android Native |
|---------|-----------|---------------|
| `MethodChannel(name)` | `FlutterMethodChannel(name, binaryMessenger)` | `MethodChannel(binaryMessenger, name)` |
| `invokeMethod(method)` | `call.method` | `call.method` |
| `result.success(value)` | `result(value)` | `result.success(value)` |
| `result.error(code, msg, details)` | `result(FlutterError(code, msg, details))` | `result.error(code, msg, details)` |
| `PlatformException` | `FlutterError` | `PlatformException` |

→ **Tiếp theo**: [03-exercise.md](./03-exercise.md) — Hands-on exercises.

<!-- AI_VERIFY: generation-complete -->
