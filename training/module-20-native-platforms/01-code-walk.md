# Module 20 – Code Walk: Native Platforms Bridge

> **Mục tiêu**: Đọc hiểu platform channel implementation và native configuration — MethodChannel trong AppDelegate, FlutterEngine lifecycle, Info.plist permissions, và build.gradle configuration.

📌 **Recap**: M0 (Dart basics) · M1 (WidgetsFlutterBinding) · M3 (Config/env)

---

## 1. Platform Channels — Flutter ↔ Native Communication

<!-- AI_VERIFY: section-platform-channels -->

### 1.1 Channel Types Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Flutter (Dart)                            │
├─────────────────────────────────────────────────────────────────┤
│  MethodChannel ──── invokeMethod() ────► Native                 │
│                  ◄─── result ────────                           │
│                                                                 │
│  EventChannel ──── receiveBroadcastStream() ──► Native          │
│                ◄─── stream events ───────                       │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 MethodChannel — Actual Source

**Note:** `BadgeHelper` **does not exist** in `base_flutter/`. The project uses Firebase plugins for all native functionality. MethodChannel is set up in iOS native code only.

**Actual Dart side — no custom helper exists.** Flutter calls go through Firebase plugin MethodChannels internally.

```dart
// base_flutter/ does NOT have a Dart-side BadgeHelper class.
// All platform channel communication happens through:
// 1. Firebase plugins (FirebaseMessaging, FirebaseAnalytics)
// 2. Native plugin registries via GeneratedPluginRegistrant

// If you needed to call a custom MethodChannel, the pattern would be:
import 'package:flutter/services.dart';

Future<void> clearBadgeCount() async {
  try {
    await const MethodChannel('jp.flutter.app').invokeMethod('clearBadgeCount');
  } on PlatformException catch (e) {
    // Handle native side errors gracefully
    debugPrint('Failed to clear badge: ${e.message}');
  }
}
```

**Actual iOS Swift side:**

```swift
// ios/Runner/AppDelegate.swift — ACTUAL SOURCE
@main
@objc class AppDelegate: FlutterAppDelegate {
  lazy var flutterEngine = FlutterEngine(name: "shared_engine")

  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    flutterEngine.run()
    FirebaseApp.configure()
    GeneratedPluginRegistrant.register(with: flutterEngine)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

func didInitializeImplicitFlutterEngine(_ engine: FlutterImplicitEngineBridge) {
  GeneratedPluginRegistrant.register(with: engine.pluginRegistry)
  let notificationChannel = FlutterMethodChannel(
    name: "jp.flutter.app",
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
}

func clearBadgeCount() {
  DispatchQueue.main.async {
    UIApplication.shared.applicationIconBadgeNumber = 0
  }
}
```

**Flow (as taught):**
1. Flutter gọi `channel.invokeMethod('clearBadgeCount')`
2. Platform thread gửi message qua binary messenger
3. Native handler nhận, xử lý
4. Native gọi `result(nil)` → Flutter nhận `null`

**Note:** The actual `base_flutter` project **does not implement** a Dart-side MethodChannel helper — this is a **teaching pattern only**. Real-world Flutter apps typically rely on official plugins for native communication.

### 1.3 MethodChannel với Arguments

```dart
// Dart: Gửi arguments
await _channel.invokeMethod('setBadgeCount', {'count': 5});

// Swift: Nhận arguments
if call.method == "setBadgeCount" {
  if let args = call.arguments as? [String: Any],
     let count = args["count"] as? Int {
    UIApplication.shared.applicationIconBadgeNumber = count
    result(nil)
  } else {
    result(FlutterError(code: "INVALID_ARGS", message: "Invalid arguments", details: nil))
  }
}
```

### 1.4 EventChannel — Streaming Pattern

```dart
// Dart side
import 'package:flutter/services.dart';

class BatteryStream {
  static const _eventChannel = EventChannel('jp.flutter.app/battery');

  static Stream<int> get batteryLevelStream {
    return _eventChannel
        .receiveBroadcastStream()
        .map((event) => event as int);
  }
}

// Usage
BatteryStream.batteryLevelStream.listen((level) {
  print('Battery level: $level%');
});
```

```swift
// iOS side
let batteryChannel = FlutterEventChannel(
  name: "jp.flutter.app/battery",
  binaryMessenger: controller.binaryMessenger
)

batteryChannel.setStreamHandler(BatteryStreamHandler())

class BatteryStreamHandler: NSObject, FlutterStreamHandler {
  func onListen(withArguments arguments: Any?, 
                eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    // Start sending battery updates
    Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
      UIDevice.current.isBatteryMonitoringEnabled = true
      let level = Int(UIDevice.current.batteryLevel * 100)
      events(level)  // Send event
    }
    return nil
  }

  func onCancel(withArguments arguments: Any?) -> FlutterError? {
    // Stop sending updates
    return nil
  }
}
```

---

## 2. iOS Configuration — AppDelegate & SceneDelegate

<!-- AI_VERIFY: section-ios-config -->

### 2.1 AppDelegate.swift — Actual Source Code

<!-- AI_VERIFY: base_flutter/ios/Runner/AppDelegate.swift -->

→ [Mở file gốc: `ios/Runner/AppDelegate.swift`](../../base_flutter/ios/Runner/AppDelegate.swift)

```swift
import UIKit
import Flutter
import Firebase
import UserNotifications

@main
@objc class AppDelegate: FlutterAppDelegate {
  lazy var flutterEngine = FlutterEngine(name: "shared_engine")

  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    // Start the Flutter engine
    flutterEngine.run()

    FirebaseApp.configure()
    GeneratedPluginRegistrant.register(with: flutterEngine)

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

// MARK: - MethodChannel Setup
// Được gọi từ Flutter side qua FlutterImplicitEngineBridge
func didInitializeImplicitFlutterEngine(_ engine: FlutterImplicitEngineBridge) {
  GeneratedPluginRegistrant.register(with: engine.pluginRegistry)

  let notificationChannel = FlutterMethodChannel(
    name: "jp.flutter.app",
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
}

func clearBadgeCount() {
  // Reset badge count
  DispatchQueue.main.async {
    UIApplication.shared.applicationIconBadgeNumber = 0
  }
}
```

**Key observations:**
- `FlutterEngine` được tạo là `lazy var` — chỉ tạo khi cần
- Engine name là `"shared_engine"` — cho phép chia sẻ engine với Scenes
- MethodChannel setup nằm **ngoài** `AppDelegate` class — trong global function `didInitializeImplicitFlutterEngine`
- `FlutterImplicitEngineBridge` là internal Flutter type — không phải API public
- `clearBadgeCount()` chỉ set badge về 0 — không clear notifications

### 2.2 SceneDelegate.swift — FlutterEngine Lifecycle

```swift
// ios/Runner/SceneDelegate.swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?
  var flutterEngine: FlutterEngine?

  func scene(
    _ scene: UIScene,
    willConnectTo session: UISceneSession,
    options connectionOptions: UIScene.ConnectionOptions
  ) {
    guard let windowScene = (scene as? UIWindowScene) else { return }

    // Lấy FlutterEngine từ AppDelegate
    guard let appDelegate = UIApplication.shared.delegate as? AppDelegate,
          let engine = appDelegate.flutterEngine else {
      fatalError("AppDelegate FlutterEngine not found")
    }

    self.flutterEngine = engine

    // Tạo FlutterViewController với engine
    let flutterViewController = FlutterViewController(
      engine: engine,
      nibName: nil,
      bundle: nil
    )

    self.window = UIWindow(windowScene: windowScene)
    self.window?.rootViewController = flutterViewController
    self.window?.makeKeyAndVisible()
  }
}
```

### 2.3 Info.plist — Permissions

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <!-- Camera -->
  <key>NSCameraUsageDescription</key>
  <string>We need camera access to take photos for your profile</string>

  <!-- Photo Library -->
  <key>NSPhotoLibraryUsageDescription</key>
  <string>We need photo library access to select profile pictures</string>

  <!-- Location -->
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>We need your location to show nearby content</string>
  <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
  <string>We need your location to send you relevant notifications</string>

  <!-- Push Notifications -->
  <key>UIBackgroundModes</key>
  <array>
    <string>remote-notification</string>
    <string>fetch</string>
  </array>

  <!-- Face ID -->
  <key>NSFaceIDUsageDescription</key>
  <string>We use Face ID for secure authentication</string>
</dict>
</plist>
```

### 2.4 Podfile — iOS Version & Dependencies

```ruby
# ios/Podfile
platform :ios, '14.0'  # Minimum iOS version

# CocoaPods
install! 'cocoapods', :deterministic_uuids => false

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'
      config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
    end
  end
end
```

---

## 3. Android Configuration — MainActivity & Manifest

<!-- AI_VERIFY: section-android-config -->

### 3.1 MainActivity.kt — Actual Source Code

<!-- AI_VERIFY: base_flutter/android/app/src/main/kotlin/jp/flutter/app/MainActivity.kt -->

→ [Mở file gốc: `android/app/src/main/kotlin/jp/flutter/app/MainActivity.kt`](../../base_flutter/android/app/src/main/kotlin/jp/flutter/app/MainActivity.kt)

```kotlin
package jp.flutter.app

import android.os.Bundle
import io.flutter.embedding.android.FlutterActivity

class MainActivity : FlutterActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
//        println(BuildConfig.API_KEY)       // ← Commented out API key access
//        println(BuildConfig.API_SECRET)    // ← Commented out API secret access
    }
}
```

**Key observations:**
- MainActivity **không có** MethodChannel setup — dự án dùng Flutter Firebase plugins cho tất cả native communication
- Plugin Firebase Messaging tự động register MethodChannel qua `GeneratedPluginRegistrant`
- API keys được access qua `BuildConfig` (từ `google-services.json`) — nhưng đã bị **comment out** trong source
- `configureFlutterEngine()` không được override — Flutter embedding tự động xử lý
- Badge clearing dùng iOS-only MethodChannel trong `didInitializeImplicitFlutterEngine()`

### 3.2 AndroidManifest.xml — Permissions

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

  <!-- Internet -->
  <uses-permission android:name="android.permission.INTERNET"/>

  <!-- Camera -->
  <uses-permission android:name="android.permission.CAMERA"/>
  <uses-feature android:name="android.hardware.camera" android:required="false"/>

  <!-- Location -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

  <!-- Storage -->
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

  <!-- Push Notifications -->
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
  <uses-permission android:name="android.permission.VIBRATE"/>
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

  <!-- Biometric -->
  <uses-permission android:name="android.permission.USE_BIOMETRIC"/>
  <uses-permission android:name="android.permission.USE_FINGERPRINT"/>

  <application>
    <!-- Main Activity -->
    <activity
        android:name=".MainActivity"
        android:exported="true"
        android:launchMode="singleTop"
        android:theme="@style/LaunchTheme"
        android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection">
      <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>

    <!-- Firebase Messaging -->
    <service
        android:name=".flutter.firebase.flutterfireos.flutter_local_notifications.FirebaseMessagingPlugin$GmsTask">
      <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
      </intent-filter>
    </service>
  </application>
</manifest>
```

### 3.3 build.gradle — SDK Versions & Dependencies

```groovy
// android/app/build.gradle
android {
  namespace "jp.flutter.app"
  compileSdk 34  // Compile against Android 14

  defaultConfig {
    applicationId "jp.flutter.app"
    minSdk 23     // Minimum: Android 6.0
    targetSdk 34  // Target: Android 14
    versionCode 1
    versionName "1.0.0"

    // MultiDex for large apps
    multiDexEnabled true
  }

  buildTypes {
    debug {
      debuggable true
      minifyEnabled false
    }
    release {
      debuggable false
      minifyEnabled true
      shrinkResources true

      // Signing config (defined in key.properties)
      signingConfig signingConfigs.release

      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }

  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }

  kotlinOptions {
    jvmTarget = '1.8'
  }
}

dependencies {
  implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"

  // Firebase
  implementation platform('com.google.firebase:firebase-bom:32.7.0')
  implementation 'com.google.firebase:firebase-analytics'
  implementation 'com.google.firebase:firebase-crashlytics'

  // MultiDex
  implementation 'androidx.multidex:multidex:2.0.1'
}
```

---

## 4. App Signing — Certificates & Keystores

<!-- AI_VERIFY: section-app-signing -->

### 4.1 iOS Certificates

| Certificate | Purpose | Used For |
|-------------|---------|----------|
| **Development** | Testing on device | Debug builds |
| **Distribution** | App Store / TestFlight | Release builds |
| **Push** | APNs notifications | Both dev & prod |

```
Keychain Access
├── Certificates
│   ├── Apple Development: jp.flutter.app (Team ID)
│   └── Apple Distribution: jp.flutter.app (Team ID)
└── Keys
    └── Apple Push Notifications: jp.flutter.app (Team ID)
```

### 4.2 Android Keystore

```properties
# android/key.properties (NOT committed to git)
storePassword=your_store_password
keyPassword=your_key_password
keyAlias=your_key_alias
storeFile=/path/to/your/keystore.jks
```

```groovy
// android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
  signingConfigs {
    release {
      keyAlias keystoreProperties['keyAlias']
      keyPassword keystoreProperties['keyPassword']
      storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
      storePassword keystoreProperties['storePassword']
    }
  }
}
```

---

## 5. Conditional Imports — Platform-Specific Code

<!-- AI_VERIFY: section-conditional-imports -->

### 5.1 Platform Detection

```dart
// Dart side
import 'dart:io' show Platform;

void main() {
  if (Platform.isIOS) {
    print('Running on iOS');
    // iOS-specific initialization
  } else if (Platform.isAndroid) {
    print('Running on Android');
    // Android-specific initialization
  }
}
```

### 5.2 Platform-Specific Implementation

```dart
// lib/common/platform/platform_info.dart
// Stub implementation
import 'platform_info_stub.dart'
    if (dart.library.io) 'platform_info_io.dart'
    if (dart.library.html) 'platform_info_web.dart';

abstract class PlatformInfo {
  String get platformName;
  int get platformVersion;

  static PlatformInfo get instance => _instance;
  static PlatformInfo _instance = PlatformInfoImpl();
}
```

```dart
// lib/common/platform/platform_info_io.dart
import 'dart:io';
import 'platform_info.dart';

class PlatformInfoImpl implements PlatformInfo {
  @override
  String get platformName => Platform.operatingSystem;

  @override
  int get platformVersion {
    // iOS: Darwin kernel version
    // Android: SDK version
    return Platform.operatingSystemVersion.hashCode;
  }
}
```

### 5.3 Native-Specific Helper Pattern

> ⚠️ **Teaching Pattern — Not in base_flutter:** The following code shows a common pattern for platform-specific helpers. The `PlatformBadgeHelper` class does **not exist** in the actual codebase — it's shown here as a teaching example.

```dart
// lib/common/helper/platform_badge_helper.dart
// Teaching pattern: platform-specific helper using conditional imports
import 'package:flutter/services.dart';
import 'dart:io' show Platform;

class PlatformBadgeHelper {
  static Future<void> clearBadge() async {
    if (Platform.isIOS) {
      const channel = MethodChannel('jp.flutter.app');
      await channel.invokeMethod('clearBadgeCount');
    }
    // Android: No badge API needed or use ShortcutBadger
  }
}
```

**Real-world implementation note:** In the actual codebase, platform-specific code typically uses conditional imports (`lib/xxx/io.dart` and `lib/xxx/html.dart`) rather than runtime `Platform.isIOS` checks for better tree-shaking.

---

## 6. Native Debugging

<!-- AI_VERIFY: section-native-debugging -->

### 6.1 Xcode Debugging

```
Xcode
├── Product → Profile (Instruments)
│   └── Leaks, Allocations, Time Profiler
├── Debug → View Memory Graph
└── Debug → Simulate Location
```

**Breakpoints in Swift/Kotlin:**
- Set breakpoint in `AppDelegate.swift` → MethodChannel handler
- Set breakpoint in `MainActivity.kt` → `configureFlutterEngine`

### 6.2 Android Studio Debugging

```
Android Studio
├── Run → Debug 'app' (Attach to Flutter)
├── View → Tool Windows → Logcat
└── Profile → CPU Profiler
```

**Breakpoints:**
- Set breakpoint in `MainActivity.kt` → `configureFlutterEngine`
- Set breakpoint in native Firebase code

### 6.3 Flutter DevTools — Platform Messages

```
Flutter DevTools
├── Inspector → Widget Tree
├── Network (for HTTP)
└── Timeline → Platform Events
```

> 💡 **Tip:** Use `debugPrint` in Flutter and `print` in native code to trace platform messages.

---

## Tổng kết

```
Native Platform Bridge:
├── MethodChannel (request-response)
│   ├── invokeMethod() → result callback
│   └── PlatformException for errors
├── EventChannel (streaming)
│   ├── receiveBroadcastStream()
│   └── Stream events from native
├── iOS Configuration
│   ├── AppDelegate.swift (FlutterEngine + channels)
│   ├── Info.plist (permissions)
│   └── Podfile (dependencies)
├── Android Configuration
│   ├── MainActivity.kt (FlutterActivity + channels)
│   ├── AndroidManifest.xml (permissions)
│   └── build.gradle (SDK versions)
└── Conditional Imports
    └── Platform.isIOS / Platform.isAndroid
```

→ **Tiếp theo**: [02-concept.md](./02-concept.md) — Đi sâu 6 concepts về native platforms.

<!-- AI_VERIFY: generation-complete -->
