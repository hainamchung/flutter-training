# Buổi 15: Platform Integration — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 15/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 12 (toàn bộ lý thuyết + bài tập)

## 1. Platform Channels 🟡

### 1.1. Tổng quan

Platform Channels là cơ chế giao tiếp giữa **Dart** và **Native code** (Swift/Objective-C trên iOS, Kotlin/Java trên Android). Đây là cầu nối cho phép Flutter truy cập các API native mà framework chưa hỗ trợ sẵn.

```
┌──────────────────┐                    ┌──────────────────┐
│   Dart (Flutter)  │  ◄── Messages ──► │  Native Platform  │
│                    │   (async, binary) │  (Swift / Kotlin) │
└──────────────────┘                    └──────────────────┘
         │                                       │
         └──── Platform Channel (tên duy nhất) ──┘
```

**Đặc điểm chính:**
- Giao tiếp **bất đồng bộ** (async message passing)
- Dữ liệu được encode/decode qua **codec** (mặc định: `StandardMessageCodec`)
- Mỗi channel có **tên duy nhất** (string) để định danh
- Hỗ trợ các kiểu dữ liệu cơ bản: `null`, `bool`, `int`, `double`, `String`, `List`, `Map`, `Uint8List`

### 1.2. Ba loại Platform Channel

#### MethodChannel — Gọi method (phổ biến nhất)

Dùng để gọi một method cụ thể và nhận kết quả. Giống kiểu **request-response**.

```dart
// Dart side
final channel = MethodChannel('com.example.app/battery');

// Gọi method native và nhận kết quả
final int batteryLevel = await channel.invokeMethod('getBatteryLevel');

// Native cũng có thể gọi method về Dart
channel.setMethodCallHandler((call) async {
  if (call.method == 'onBatteryChanged') {
    final level = call.arguments as int;
    // Xử lý...
  }
  return null;
});
```

```swift
// Swift side (iOS)
let channel = FlutterMethodChannel(
    name: "com.example.app/battery",
    binaryMessenger: controller.binaryMessenger
)

channel.setMethodCallHandler { (call, result) in
    if call.method == "getBatteryLevel" {
        let level = getBatteryLevel() // Hàm native
        result(level)
    } else {
        result(FlutterMethodNotImplemented)
    }
}
```

```kotlin
// Kotlin side (Android)
val channel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "com.example.app/battery")

channel.setMethodCallHandler { call, result ->
    if (call.method == "getBatteryLevel") {
        val level = getBatteryLevel() // Hàm native
        result.success(level)
    } else {
        result.notImplemented()
    }
}
```

#### EventChannel — Stream events liên tục

Dùng khi native cần **gửi dữ liệu liên tục** về Dart (sensor data, location updates, connectivity changes...).

```dart
// Dart side
final eventChannel = EventChannel('com.example.app/sensor');

// Lắng nghe stream
eventChannel.receiveBroadcastStream().listen(
  (event) {
    print('Sensor data: $event');
  },
  onError: (error) {
    print('Error: $error');
  },
);
```

#### BasicMessageChannel — Gửi/nhận message tự do

Dùng để gửi message dạng tự do, không theo pattern method call. Linh hoạt nhất nhưng ít phổ biến.

```dart
// Dart side
final messageChannel = BasicMessageChannel<String>(
  'com.example.app/messages',
  StringCodec(),
);

// Gửi message
final reply = await messageChannel.send('Hello from Dart');

// Nhận message từ native
messageChannel.setMessageHandler((message) async {
  print('Received: $message');
  return 'Got it!';
});
```

### 1.3. So sánh 3 loại Channel

| Đặc điểm | MethodChannel | EventChannel | BasicMessageChannel |
|-----------|---------------|--------------|---------------------|
| Kiểu giao tiếp | Request-Response | Stream (one-way) | Message passing |
| Hướng | Hai chiều | Native → Dart | Hai chiều |
| Use case | Gọi API native | Sensor, location | Custom protocol |
| Phổ biến | ⭐⭐⭐ | ⭐⭐ | ⭐ |

### 1.4. StandardMessageCodec

Đây là codec mặc định, hỗ trợ các kiểu dữ liệu:

| Dart | Android (Java/Kotlin) | iOS (Swift) |
|------|-----------------------|-------------|
| `null` | `null` | `nil` (NSNull) |
| `bool` | `Boolean` | `NSNumber(boolValue:)` |
| `int` | `Int` / `Long` | `NSNumber(intValue:)` |
| `double` | `Double` | `NSNumber(doubleValue:)` |
| `String` | `String` | `String` |
| `Uint8List` | `byte[]` | `FlutterStandardTypedData` |
| `List` | `ArrayList` | `NSArray` |
| `Map` | `HashMap` | `NSDictionary` |

> 🔗 **FE Bridge:** Platform Channels = **Kiểu C — không có FE equivalent**. Đây là native bridge (Dart ↔ Swift/Kotlin) — tương tự concept FFI/native binding. FE gần nhất là WebAssembly FFI hoặc React Native Bridge, nhưng pattern rất khác. FE dev cần học concept mới hoàn toàn.

---

## 2. Pigeon — Type-safe Platform Communication 🟢

### 2.1. Vấn đề với MethodChannel thủ công

```dart
// ❌ Dễ lỗi — string matching, không có type safety
final result = await channel.invokeMethod('getUserInfo', {'userId': 123});
final name = result['name'] as String; // Runtime error nếu sai type
```

**Vấn đề:**
- Tên method là string → dễ typo, không có IDE support
- Arguments là dynamic → không kiểm tra type lúc compile
- Phải tự viết code serialize/deserialize
- Dart side và Native side có thể không đồng bộ

### 2.2. Pigeon giải quyết vấn đề này

**Pigeon** là code generator chính thức của Flutter team. Bạn định nghĩa API bằng Dart → Pigeon tự động generate code cho cả Swift và Kotlin.

```
┌───────────────────┐
│  Dart API Definition │  ← Bạn viết file này
│  (pigeon file)        │
└─────────┬─────────┘
          │  dart run pigeon
          ▼
┌─────────┴──────────┐
│                      │
▼                      ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Dart code │    │Swift code│    │Kotlin code│
│ (generated)│   │(generated)│   │(generated) │
└──────────┘    └──────────┘    └──────────┘
```

### 2.3. Định nghĩa API với Pigeon

```dart
// pigeons/messages.dart — File định nghĩa API

import 'package:pigeon/pigeon.dart';

// Data class — tự động serialize/deserialize
class DeviceInfo {
  String? name;
  String? osVersion;
  int? batteryLevel;
}

class UserRequest {
  String? userId;
}

class UserResponse {
  String? name;
  String? email;
}

// @HostApi = Dart gọi Native
@HostApi()
abstract class DeviceApi {
  DeviceInfo getDeviceInfo();
  String getDeviceName();
}

// @FlutterApi = Native gọi Dart
@FlutterApi()
abstract class NotificationApi {
  void onNotificationReceived(String message);
}
```

### 2.4. Generate code

```bash
# Thêm dependency
flutter pub add pigeon --dev

# Generate code
dart run pigeon \
  --input pigeons/messages.dart \
  --dart_out lib/src/generated/messages.g.dart \
  --swift_out ios/Runner/Messages.g.swift \
  --kotlin_out android/app/src/main/kotlin/com/example/app/Messages.g.kt
```

### 2.5. Sử dụng generated code

```dart
// Dart side — gọi native (type-safe!)
final deviceApi = DeviceApi();
final info = await deviceApi.getDeviceInfo();
print('Device: ${info.name}, OS: ${info.osVersion}');
```

```swift
// Swift side — implement protocol
class DeviceApiImpl: DeviceApi {
    func getDeviceInfo() throws -> DeviceInfo {
        let info = DeviceInfo()
        info.name = UIDevice.current.name
        info.osVersion = UIDevice.current.systemVersion
        info.batteryLevel = Int64(UIDevice.current.batteryLevel * 100)
        return info
    }

    func getDeviceName() throws -> String {
        return UIDevice.current.name
    }
}

// Đăng ký trong AppDelegate
DeviceApiSetup.setUp(
    binaryMessenger: controller.binaryMessenger,
    api: DeviceApiImpl()
)
```

### 2.6. So sánh MethodChannel vs Pigeon

| Đặc điểm | MethodChannel | Pigeon |
|-----------|---------------|--------|
| Type safety | ❌ Dynamic | ✅ Compile-time |
| Boilerplate | Nhiều | Ít (generated) |
| Sync Dart ↔ Native | Thủ công | Tự động |
| IDE support | Hạn chế | Đầy đủ (autocomplete) |
| Learning curve | Thấp | Trung bình |
| Phù hợp | Prototype, đơn giản | Production, complex API |

---

## 3. Native Features — Các tính năng native phổ biến 🟢

### 3.1. Tổng quan packages

Flutter ecosystem có rất nhiều plugin đã wrap sẵn các tính năng native. Bạn **không cần** viết native code cho hầu hết trường hợp.

| Tính năng | Package | Pub.dev |
|-----------|---------|---------|
| Camera | `camera` | camera |
| Chọn ảnh | `image_picker` | image_picker |
| Vị trí GPS | `geolocator` | geolocator |
| Push Notifications | `firebase_messaging` | firebase_messaging |
| Xác thực sinh trắc | `local_auth` | local_auth |
| Chọn file | `file_picker` | file_picker |
| Mở URL / app khác | `url_launcher` | url_launcher |
| Chia sẻ nội dung | `share_plus` | share_plus |
| Thông tin app | `package_info_plus` | package_info_plus |
| Quyền truy cập | `permission_handler` | permission_handler |

### 3.2. Camera (`camera` package)

```dart
// pubspec.yaml
// dependencies:
//   camera: ^0.11.0

import 'package:camera/camera.dart';

class CameraScreen extends StatefulWidget {
  @override
  State<CameraScreen> createState() => _CameraScreenState();
}

class _CameraScreenState extends State<CameraScreen> {
  late CameraController _controller;
  late List<CameraDescription> _cameras;

  @override
  void initState() {
    super.initState();
    _initCamera();
  }

  Future<void> _initCamera() async {
    _cameras = await availableCameras();
    _controller = CameraController(
      _cameras.first,
      ResolutionPreset.high,
    );
    await _controller.initialize();
    if (mounted) setState(() {});
  }

  Future<void> _capturePhoto() async {
    final XFile photo = await _controller.takePicture();
    print('Photo saved to: ${photo.path}');
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!_controller.value.isInitialized) {
      return const Center(child: CircularProgressIndicator());
    }
    return CameraPreview(_controller);
  }
}
```

> 🔗 **FE Bridge:** Image Picker ≈ `<input type="file" accept="image/*">` — chọn/chụp ảnh. Nhưng **khác ở**: mobile = native camera API, xử lý orientation, EXIF, compression. FE chỉ nhận file từ browser. Flutter plugins như `image_picker` abstract platform differences — FE `<input>` chỉ delegate cho browser.

### 3.3. Geolocation (`geolocator` package)

```dart
// pubspec.yaml
// dependencies:
//   geolocator: ^12.0.0

import 'package:geolocator/geolocator.dart';

Future<Position> getCurrentLocation() async {
  // Kiểm tra service
  bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
  if (!serviceEnabled) {
    throw Exception('Location services are disabled');
  }

  // Kiểm tra permission
  LocationPermission permission = await Geolocator.checkPermission();
  if (permission == LocationPermission.denied) {
    permission = await Geolocator.requestPermission();
    if (permission == LocationPermission.denied) {
      throw Exception('Location permission denied');
    }
  }

  if (permission == LocationPermission.deniedForever) {
    throw Exception('Location permission permanently denied');
  }

  // Lấy vị trí
  return await Geolocator.getCurrentPosition(
    desiredAccuracy: LocationAccuracy.high,
  );
}
```

### 3.4. Push Notifications (`firebase_messaging`)

```dart
import 'package:firebase_messaging/firebase_messaging.dart';

class PushNotificationService {
  final FirebaseMessaging _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    // Xin quyền (iOS)
    final settings = await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
    );

    if (settings.authorizationStatus == AuthorizationStatus.authorized) {
      // Lấy FCM token
      final token = await _messaging.getToken();
      print('FCM Token: $token');

      // Foreground messages
      FirebaseMessaging.onMessage.listen((RemoteMessage message) {
        print('Received: ${message.notification?.title}');
      });

      // Background messages
      FirebaseMessaging.onBackgroundMessage(_handleBackgroundMessage);
    }
  }
}

// Phải là top-level function
Future<void> _handleBackgroundMessage(RemoteMessage message) async {
  print('Background message: ${message.messageId}');
}
```

> 🔗 **FE Bridge:** Push Notification ≈ **Web Push API** + Service Worker — cùng concept token registration + remote push. Nhưng **khác ở**: mobile push = **FCM/APNs** (platform-specific setup), web push = browser + VAPID keys. Mobile notification channels, badges, sounds = richer hơn web notifications rất nhiều.

### 3.5. Biometric Auth (`local_auth`)

```dart
import 'package:local_auth/local_auth.dart';

class BiometricService {
  final LocalAuthentication _auth = LocalAuthentication();

  Future<bool> authenticate() async {
    final bool canAuth = await _auth.canCheckBiometrics;
    final bool isDeviceSupported = await _auth.isDeviceSupported();

    if (!canAuth || !isDeviceSupported) return false;

    // Kiểm tra loại biometric có sẵn
    final List<BiometricType> availableBiometrics =
        await _auth.getAvailableBiometrics();

    return await _auth.authenticate(
      localizedReason: 'Xác thực để truy cập ứng dụng',
      options: const AuthenticationOptions(
        stickyAuth: true,
        biometricOnly: false, // Cho phép PIN/pattern fallback
      ),
    );
  }
}
```

### 3.6. Các tiện ích khác

```dart
// url_launcher — Mở URL
import 'package:url_launcher/url_launcher.dart';

Future<void> openUrl(String url) async {
  final uri = Uri.parse(url);
  if (await canLaunchUrl(uri)) {
    await launchUrl(uri, mode: LaunchMode.externalApplication);
  }
}

// share_plus — Chia sẻ nội dung
import 'package:share_plus/share_plus.dart';

void shareContent() {
  Share.share('Check out this app!', subject: 'My App');
}

// package_info_plus — Thông tin app
import 'package:package_info_plus/package_info_plus.dart';

Future<void> getAppInfo() async {
  final info = await PackageInfo.fromPlatform();
  print('App: ${info.appName}');
  print('Version: ${info.version}');
  print('Build: ${info.buildNumber}');
}

// file_picker — Chọn file
import 'package:file_picker/file_picker.dart';

Future<void> pickFile() async {
  FilePickerResult? result = await FilePicker.platform.pickFiles(
    type: FileType.custom,
    allowedExtensions: ['pdf', 'doc', 'docx'],
  );

  if (result != null) {
    final file = result.files.single;
    print('File: ${file.name}, Size: ${file.size}');
  }
}
```

### 3.7. App Lifecycle (WidgetsBindingObserver)

Flutter cung cấp `WidgetsBindingObserver` để theo dõi lifecycle:

```dart
class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.resumed:
        // App active — refresh data, reconnect
        break;
      case AppLifecycleState.inactive:
        // App inactive (incoming call, etc.)
        break;
      case AppLifecycleState.paused:
        // App in background — save state, pause streams
        break;
      case AppLifecycleState.detached:
        // App about to be destroyed
        break;
      case AppLifecycleState.hidden:
        // App hidden but still running
        break;
    }
  }
  
  @override
  Widget build(BuildContext context) => const MyHomePage();
}
```

**Use cases thực tế:**
- `resumed`: Refresh data từ server, reconnect WebSocket
- `paused`: Save draft, pause video/audio, disconnect socket
- `inactive`: Pause game, blur sensitive content

### 3.8. Deep Linking — Platform Setup

> 📖 **Lý thuyết deep linking** đã học ở [Buổi 05 — Navigation & Routing](../../tuan-03-navigation-state-co-ban/buoi-05-navigation-routing/01-ly-thuyet.md). Ở đây focus vào setup platform-specific.

#### Android: `AndroidManifest.xml`
```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW"/>
  <category android:name="android.intent.category.DEFAULT"/>
  <category android:name="android.intent.category.BROWSABLE"/>
  <data android:scheme="https" android:host="myapp.com" android:pathPrefix="/products"/>
</intent-filter>
```

#### iOS: `Info.plist` + Associated Domains
```xml
<!-- Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array><string>myapp</string></array>
  </dict>
</array>
```

```
// Runner.entitlements
com.apple.developer.associated-domains = ["applinks:myapp.com"]
```

> 💡 **Verify**: `adb shell am start -a android.intent.action.VIEW -d "https://myapp.com/products/1"` (Android) hoặc `xcrun simctl openurl booted "https://myapp.com/products/1"` (iOS)

---

> 💼 **Gặp trong dự án:** Implement MethodChannel cho native camera/biometrics, handle platform-specific permission flows, error propagation từ native → Flutter, type-safe communication với Pigeon
> 🤖 **Keywords bắt buộc trong prompt:** `MethodChannel`, `invokeMethod`, `setMethodCallHandler`, `PlatformException`, `Permission.camera.request()`, `Pigeon`, `@HostApi`, `error propagation`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Native feature:** App cần access native camera với custom UI (không dùng plugin) — MethodChannel setup
- **Permissions:** Camera + Gallery + Location permissions — phải handle denied, permanently denied, restricted
- **Error handling:** Native code throw exception → Flutter phải catch + show user-friendly message

**Tại sao cần các keyword trên:**
- **`MethodChannel`** — bidirectional communication Flutter ↔ Native (iOS/Android)
- **`invokeMethod`** — Flutter gọi native method, AI hay thiếu error handling cho MissingPluginException
- **`PlatformException`** — native errors propagate qua channel, AI hay swallow errors
- **`Permission.camera.request()`** — permission_handler, AI hay thiếu permanently denied handling
- **`Pigeon`** — type-safe alternative cho MethodChannel, AI hay mix cả hai

**Prompt mẫu — MethodChannel + Permission:**
```text
Tôi cần implement MethodChannel cho native camera feature trong Flutter.
Tech stack: Flutter 3.x, permission_handler ^11.x.
Requirements:
1. Flutter side: CameraService class — openCamera() → Future<String?> (image path).
2. Android side: Kotlin MethodCallHandler — handle "openCamera" call.
3. iOS side: Swift FlutterMethodCallHandler — handle "openCamera" call.
4. Permission flow:
   a. Check camera permission status.
   b. If undetermined → request.
   c. If denied → show rationale dialog → request again.
   d. If permanentlyDenied → show dialog → open app settings.
5. Error propagation: native exception → PlatformException → custom CameraFailure.
6. Null safety: native return null nếu user cancel → handle gracefully.
Constraints:
- Channel name: 'com.myapp/camera' (consistent both sides).
- Error codes: 'CAMERA_UNAVAILABLE', 'PERMISSION_DENIED', 'CAPTURE_FAILED'.
- Each platform method PHẢI return result (không fire-and-forget).
- Handle MissingPluginException (plugin not registered).
Output: camera_service.dart (Flutter) + MainActivity.kt (Android) + AppDelegate.swift (iOS).
```

**Expected Output:** AI gen 3 files (Flutter + Android + iOS) cho camera MethodChannel.

⚠️ **Giới hạn AI hay mắc:** AI hay quên handle `permanentlyDenied` permission status (user phải go to Settings). AI cũng hay thiếu error propagation từ native → Flutter (error swallowed on native side). AI hay dùng wrong channel name mismatch giữa Flutter và native.

</details>

---

## 4. Plugin Development 🟢

### 4.1. Plugin vs Package

| | Package | Plugin |
|---|---------|--------|
| Chứa | Chỉ Dart code | Dart + Native code (iOS/Android) |
| Ví dụ | `provider`, `bloc` | `camera`, `url_launcher` |
| Tạo bằng | `flutter create --template=package` | `flutter create --template=plugin` |
| Platform code | Không | Có (Swift, Kotlin, C++...) |

### 4.2. Tạo plugin

```bash
flutter create --template=plugin \
  --org com.example \
  --platforms=android,ios \
  my_plugin
```

**Cấu trúc thư mục sinh ra:**

```
my_plugin/
├── lib/
│   ├── my_plugin.dart              # Public API
│   ├── my_plugin_method_channel.dart  # MethodChannel implementation
│   └── my_plugin_platform_interface.dart  # Platform interface
├── android/
│   └── src/main/kotlin/.../MyPlugin.kt  # Android implementation
├── ios/
│   └── Classes/MyPlugin.swift       # iOS implementation
├── example/                         # Example app để test
│   ├── lib/main.dart
│   └── ...
├── test/                            # Unit tests
├── pubspec.yaml
└── README.md
```

### 4.3. Federated Plugin Architecture

Flutter khuyến khích kiến trúc **federated** cho plugin — tách platform interface và implementation.

```
┌──────────────────────────────────┐
│  my_plugin (app-facing package)  │  ← Developer import cái này
│  Dart API that users call        │
└──────────┬───────────────────────┘
           │ depends on
           ▼
┌──────────────────────────────────┐
│  my_plugin_platform_interface    │  ← Abstract contract
│  Platform interface + method     │
│  channel default                 │
└──────────┬───────────────────────┘
           │ implemented by
     ┌─────┼─────┐
     ▼     ▼     ▼
┌────────┐ ┌────────┐ ┌────────┐
│  _ios  │ │_android│ │  _web  │   ← Platform implementations
└────────┘ └────────┘ └────────┘
```

**Ưu điểm:**
- Mỗi platform implementation là package riêng → maintain độc lập
- Ai đó có thể contribute implementation cho platform mới (Linux, Windows...)
- App-facing package không phụ thuộc trực tiếp vào native code

### 4.4. Anatomy of a Plugin — Platform Interface

```dart
// my_plugin_platform_interface.dart
import 'package:plugin_platform_interface/plugin_platform_interface.dart';
import 'my_plugin_method_channel.dart';

abstract class MyPluginPlatform extends PlatformInterface {
  MyPluginPlatform() : super(token: _token);

  static final Object _token = Object();

  static MyPluginPlatform _instance = MethodChannelMyPlugin();

  static MyPluginPlatform get instance => _instance;

  static set instance(MyPluginPlatform instance) {
    PlatformInterface.verifyToken(instance, _token);
    _instance = instance;
  }

  Future<String?> getPlatformVersion() {
    throw UnimplementedError('getPlatformVersion() has not been implemented.');
  }
}
```

```dart
// my_plugin_method_channel.dart
import 'package:flutter/services.dart';
import 'my_plugin_platform_interface.dart';

class MethodChannelMyPlugin extends MyPluginPlatform {
  final methodChannel = const MethodChannel('my_plugin');

  @override
  Future<String?> getPlatformVersion() async {
    return await methodChannel.invokeMethod<String>('getPlatformVersion');
  }
}
```

```dart
// my_plugin.dart — Public API
import 'my_plugin_platform_interface.dart';

class MyPlugin {
  Future<String?> getPlatformVersion() {
    return MyPluginPlatform.instance.getPlatformVersion();
  }
}
```

### 4.5. Publishing to pub.dev (Tổng quan)

```bash
# 1. Kiểm tra trước khi publish
flutter pub publish --dry-run

# 2. Đảm bảo pubspec.yaml đầy đủ
# name, description, version, homepage, repository, issue_tracker

# 3. Publish
flutter pub publish

# 4. Sau khi publish → package có trên pub.dev
```

**Checklist trước khi publish:**
- ✅ README.md rõ ràng với usage examples
- ✅ CHANGELOG.md cập nhật
- ✅ LICENSE file
- ✅ Example app chạy được
- ✅ Unit tests
- ✅ `dart analyze` & `dart format` pass
- ✅ API documentation đầy đủ

---

## 5. Permissions Handling 🟡

### 5.1. Tổng quan

Mobile apps cần **xin quyền** từ người dùng để truy cập các tính năng nhạy cảm (camera, vị trí, micro...). Khác hoàn toàn với web — trên web, browser tự handle permission prompt đơn giản hơn nhiều.

### 5.2. `permission_handler` package

```dart
// pubspec.yaml
// dependencies:
//   permission_handler: ^11.0.0
```

### 5.3. iOS — Info.plist

Trên iOS, **bắt buộc** khai báo lý do sử dụng permission trong `Info.plist`. Nếu thiếu, app sẽ crash.

```xml
<!-- ios/Runner/Info.plist -->
<dict>
  <!-- Camera -->
  <key>NSCameraUsageDescription</key>
  <string>Ứng dụng cần truy cập camera để chụp ảnh đại diện</string>

  <!-- Photo Library -->
  <key>NSPhotoLibraryUsageDescription</key>
  <string>Ứng dụng cần truy cập thư viện ảnh để chọn ảnh</string>

  <!-- Location -->
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>Ứng dụng cần vị trí để hiển thị cửa hàng gần bạn</string>

  <!-- Microphone -->
  <key>NSMicrophoneUsageDescription</key>
  <string>Ứng dụng cần microphone để ghi âm tin nhắn thoại</string>

  <!-- Contacts -->
  <key>NSContactsUsageDescription</key>
  <string>Ứng dụng cần truy cập danh bạ để mời bạn bè</string>
</dict>
```

### 5.4. Android — AndroidManifest.xml

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

  <!-- Internet (không cần runtime permission) -->
  <uses-permission android:name="android.permission.INTERNET" />

  <!-- Camera -->
  <uses-permission android:name="android.permission.CAMERA" />

  <!-- Location -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

  <!-- Storage (Android < 13) -->
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

  <!-- Photos (Android 13+) -->
  <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

  <!-- Notifications (Android 13+) -->
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

  <application ...>
</manifest>
```

### 5.5. Runtime Permission Flow

```
┌─────────────────┐
│ Check permission │
│ status           │
└────────┬────────┘
         │
         ▼
    ┌────────────┐      ┌──────────────┐
    │  Granted?  │─ Yes ──▶ Proceed     │
    └────┬───────┘      └──────────────┘
         │ No
         ▼
┌─────────────────┐
│ Show rationale   │  ← Giải thích tại sao cần quyền
│ (recommended)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Request          │
│ permission       │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌──────────────────┐
│Granted │ │ Denied            │
└───┬────┘ └────────┬─────────┘
    │               │
    ▼          ┌────┴────────────┐
┌────────┐    ▼                  ▼
│Proceed │ ┌──────────┐  ┌─────────────────┐
└────────┘ │ Denied   │  │Permanently denied│
           │ (có thể  │  │(mở Settings)     │
           │ hỏi lại) │  └─────────────────┘
           └──────────┘
```

### 5.6. Code xử lý permission đầy đủ

```dart
import 'package:permission_handler/permission_handler.dart';

class PermissionService {
  /// Xin quyền camera với đầy đủ error handling
  Future<bool> requestCameraPermission(BuildContext context) async {
    // Bước 1: Kiểm tra trạng thái hiện tại
    final status = await Permission.camera.status;

    if (status.isGranted) return true;

    // Bước 2: Nếu bị denied, hiện dialog giải thích trước
    if (status.isDenied) {
      final shouldRequest = await _showPermissionRationale(
        context,
        title: 'Cần quyền Camera',
        message: 'Ứng dụng cần truy cập camera để chụp ảnh đại diện của bạn.',
      );

      if (!shouldRequest) return false;

      // Bước 3: Xin quyền
      final newStatus = await Permission.camera.request();
      return newStatus.isGranted;
    }

    // Bước 4: Nếu permanently denied → hướng dẫn mở Settings
    if (status.isPermanentlyDenied) {
      await _showOpenSettingsDialog(context);
      return false;
    }

    return false;
  }

  Future<bool> _showPermissionRationale(
    BuildContext context, {
    required String title,
    required String message,
  }) async {
    return await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(title),
        content: Text(message),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Từ chối'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Đồng ý'),
          ),
        ],
      ),
    ) ?? false;
  }

  Future<void> _showOpenSettingsDialog(BuildContext context) async {
    final shouldOpen = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Cần cấp quyền'),
        content: const Text(
          'Bạn đã từ chối quyền truy cập. '
          'Vui lòng mở Cài đặt để cấp quyền cho ứng dụng.',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Để sau'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Mở Cài đặt'),
          ),
        ],
      ),
    );

    if (shouldOpen == true) {
      await openAppSettings();
    }
  }
}
```

### 5.7. Best Practices cho Permissions

1. **Giải thích trước khi xin** — Hiện dialog giải thích *tại sao* cần quyền trước khi system prompt xuất hiện
2. **Xin đúng lúc** — Chỉ xin khi user thực sự cần tính năng đó (ví dụ: bấm nút "Chụp ảnh" mới xin quyền camera)
3. **Graceful degradation** — Nếu user từ chối, app vẫn hoạt động được (ẩn hoặc disable tính năng đó)
4. **Xin tối thiểu** — Chỉ xin quyền thực sự cần. Đừng xin hết quyền lúc mở app
5. **Handle permanently denied** — Hướng dẫn user mở Settings khi cần

> 🔗 **FE Bridge:** Permission handling ≈ **Browser Permissions API** (`navigator.permissions.query`) — cùng concept request/check/denied. Nhưng **khác ở**: mobile permissions = **system-level dialog** (1 lần deny = phải vào Settings), web permissions = browser-managed, ít friction hơn. Pattern phức tạp hơn: check → request → handle "permanently denied".

---

> 💼 **Gặp trong dự án:** Permission handling phức tạp (multiple permissions, rationale dialogs, settings redirect), permission status caching, runtime permission changes, iOS Info.plist + Android manifest declarations
> 🤖 **Keywords bắt buộc trong prompt:** `permission_handler`, `Permission.camera.status`, `isGranted`, `isPermanentlyDenied`, `openAppSettings()`, `Info.plist NSCameraUsageDescription`, `AndroidManifest uses-permission`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Multi-permission:** Feature cần Camera + Microphone + Location cùng lúc — request flow phức tạp
- **UX flow:** PM yêu cầu custom permission dialog (không dùng system dialog) với rationale explanation
- **Edge case:** User deny → mở Settings → grant → quay lại app → phải detect permission changed

**Tại sao cần các keyword trên:**
- **`permission_handler`** — cross-platform permission API, AI cần biết đầy đủ status values
- **`isPermanentlyDenied`** — chỉ trên Android, user checked "Don't ask again", PHẢI open Settings
- **`openAppSettings()`** — redirect user to system settings, AI hay quên fallback nếu settings fail
- **`Info.plist`** — iOS yêu cầu usage description string, thiếu = crash on request
- **`AndroidManifest`** — Android cần declare permission, thiếu = silently fail

**Prompt mẫu — Permission Manager:**
```text
Tôi cần PermissionManager class cho Flutter app.
Tech stack: Flutter 3.x, permission_handler ^11.x.
Permissions cần manage: Camera, Photos, Microphone, Location.
Requirements:
1. checkAndRequest(Permission) → PermissionResult enum (granted, denied, permanentlyDenied, restricted).
2. requestMultiple([Camera, Microphone]) → Map<Permission, PermissionResult>.
3. Custom rationale dialog: hiện trước khi request, giải thích tại sao cần.
4. Permanently denied flow: dialog "Bạn cần mở Settings để cấp quyền" → openAppSettings().
5. Resume detection: khi user quay lại từ Settings → re-check status.
6. iOS Info.plist entries + Android Manifest entries cho tất cả permissions.
Constraints:
- PermissionResult là custom enum (không dùng trực tiếp PermissionStatus).
- Rationale dialog customizable per permission (different message for Camera vs Location).
- Log permission request/result cho analytics.
Output: permission_manager.dart + platform config (Info.plist entries + AndroidManifest entries).
```

**Expected Output:** AI gen permission manager + platform configuration.

⚠️ **Giới hạn AI hay mắc:** AI hay thiếu iOS Info.plist `NSCameraUsageDescription` (app crash khi request!). AI cũng hay confuse `isDenied` vs `isPermanentlyDenied` (quan trọng cho UX flow). AI quên handle `isRestricted` (parental controls trên iOS).

</details>

---

## 6. Platform-specific UI 🟢

### 6.1. Kiểm tra Platform

```dart
import 'dart:io' show Platform;
import 'package:flutter/foundation.dart' show kIsWeb;

// Kiểm tra platform
if (kIsWeb) {
  // Web
} else if (Platform.isIOS) {
  // iOS
} else if (Platform.isAndroid) {
  // Android
} else if (Platform.isMacOS) {
  // macOS
} else if (Platform.isWindows) {
  // Windows
} else if (Platform.isLinux) {
  // Linux
}
```

> ⚠️ **Lưu ý:** `dart:io` Platform không khả dụng trên Web. Luôn kiểm tra `kIsWeb` trước khi dùng `Platform.isXxx`.

### 6.2. Adaptive Widgets

```dart
import 'dart:io' show Platform;
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

/// Dialog thích ứng theo platform
Future<bool?> showAdaptiveConfirmDialog(
  BuildContext context, {
  required String title,
  required String content,
}) {
  if (Platform.isIOS) {
    return showCupertinoDialog<bool>(
      context: context,
      builder: (context) => CupertinoAlertDialog(
        title: Text(title),
        content: Text(content),
        actions: [
          CupertinoDialogAction(
            isDestructiveAction: true,
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Hủy'),
          ),
          CupertinoDialogAction(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Đồng ý'),
          ),
        ],
      ),
    );
  }

  return showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      title: Text(title),
      content: Text(content),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: const Text('Hủy'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context, true),
          child: const Text('Đồng ý'),
        ),
      ],
    ),
  );
}
```

### 6.3. Flutter built-in Adaptive Widgets

Flutter cung cấp một số widget tự động switch giữa Material và Cupertino:

```dart
// Adaptive switch
Switch.adaptive(
  value: _isEnabled,
  onChanged: (value) => setState(() => _isEnabled = value),
)

// Adaptive slider
Slider.adaptive(
  value: _value,
  onChanged: (value) => setState(() => _value = value),
)

// Adaptive circular progress
CircularProgressIndicator.adaptive()

// Icon theo platform
Icon(
  Platform.isIOS ? CupertinoIcons.heart : Icons.favorite,
)
```

### 6.4. Pattern: Platform-aware Widget

```dart
/// Base class cho platform-aware widgets
abstract class PlatformWidget extends StatelessWidget {
  const PlatformWidget({super.key});

  Widget buildMaterialWidget(BuildContext context);
  Widget buildCupertinoWidget(BuildContext context);

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return buildCupertinoWidget(context);
    }
    return buildMaterialWidget(context);
  }
}

/// Ví dụ: Adaptive AppBar
class AdaptiveAppBar extends PlatformWidget {
  final String title;
  const AdaptiveAppBar({super.key, required this.title});

  @override
  Widget buildMaterialWidget(BuildContext context) {
    return AppBar(title: Text(title));
  }

  @override
  Widget buildCupertinoWidget(BuildContext context) {
    return CupertinoNavigationBar(middle: Text(title));
  }
}
```

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

1. **Ưu tiên dùng plugin có sẵn** — Trước khi viết native code, kiểm tra pub.dev xem đã có plugin chưa
2. **Dùng Pigeon thay vì MethodChannel thủ công** cho API phức tạp
3. **Luôn handle error** từ platform channel (`PlatformException`, `MissingPluginException`)
4. **Test trên cả iOS và Android** — Behavior có thể khác nhau
5. **Khai báo permission description rõ ràng** — Apple sẽ reject app nếu description không hợp lý
6. **Dùng federated plugin architecture** cho plugin mới

### ❌ Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|-----|-------------|----------|
| `MissingPluginException` | Chưa run `flutter clean` sau khi thêm plugin | `flutter clean && flutter run` |
| App crash khi xin permission iOS | Thiếu key trong Info.plist | Thêm `NSXxxUsageDescription` |
| `PlatformException` | Native code throw error | Wrap trong try-catch |
| Channel name mismatch | Tên channel Dart ≠ Native | Dùng constant hoặc Pigeon |
| Permission denied không handle | Không kiểm tra `isPermanentlyDenied` | Thêm logic mở Settings |
| `setState` after dispose | Gọi setState sau khi widget bị dispose | Check `mounted` before setState |

### Xử lý lỗi Platform Channel

```dart
import 'package:flutter/services.dart';

Future<int> getBatteryLevel() async {
  try {
    final level = await platform.invokeMethod<int>('getBatteryLevel');
    return level ?? 0;
  } on PlatformException catch (e) {
    // Native code trả về error
    print('Platform error: ${e.code} - ${e.message}');
    return -1;
  } on MissingPluginException {
    // Plugin chưa được register (thường do hot restart)
    print('Plugin not registered. Try full restart.');
    return -1;
  }
}
```

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### So sánh kiến trúc

| Khái niệm | React Native | Flutter |
|-----------|-------------|---------|
| Native bridge | React Native Bridge (JSON) | Platform Channels (binary) |
| Type-safe bridge | Turbo Modules (Codegen) | **Pigeon** (Codegen) |
| Native modules | npm packages with native code | **Flutter Plugins** |
| UI framework | Native components | Own rendering engine |

### Điểm khác biệt quan trọng

#### 1. Platform Channels vs React Native Bridge
- **React Native:** Bridge serialize JSON → chậm hơn
- **Flutter:** Binary codec → nhanh hơn
- Cả hai đều async, nhưng Flutter có Pigeon cho type safety

#### 2. Pigeon vs Turbo Modules
- **Turbo Modules:** Facebook's solution cho type-safe native modules trong RN
- **Pigeon:** Flutter team's solution — cùng ý tưởng (define API → generate code)
- Pigeon đơn giản hơn, generate code cho cả 3 platforms

#### 3. Plugins vs npm Native Modules
- **npm native modules:** Cài qua npm, link manual hoặc auto-link
- **Flutter plugins:** Cài qua pub.dev, tự động link
- Flutter plugins ổn định hơn (ít breaking changes khi upgrade)

#### 4. Permissions — Khái niệm hoàn toàn mới! 🆕
- **Web:** Browser tự hiện prompt đơn giản (Allow/Block)
- **Mobile:** Phải code permission flow, handle nhiều trạng thái (denied, permanently denied)
- **iOS:** Bắt buộc khai báo lý do trong Info.plist
- **Android:** Khai báo trong Manifest + runtime permission (Android 6+)
- **Không có tương đương trên web** — đây là điểm bạn cần học mới

#### 5. Không có DOM!
- Web: `window.open()`, `navigator.share()`, `navigator.geolocation`
- Mobile: Mọi thứ qua plugin/native code
- Không có `localStorage` → dùng SharedPreferences, Hive, SQLite
- Không có `fetch` → dùng `http`, `dio`

### Migration mindset

```
Web Concept          →  Mobile Equivalent
─────────────────────────────────────────────
window.open()        →  url_launcher
navigator.share()    →  share_plus
navigator.geolocation→  geolocator
FileReader API       →  file_picker
Notification API     →  firebase_messaging + local_notifications
localStorage         →  SharedPreferences (đã học ở Tuần 6)
npm install          →  flutter pub add
```

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|--------------|
| 1 | Everything through browser APIs | **Platform Channels** giao tiếp trực tiếp native SDK | Mobile = direct hardware access, web = sandboxed |
| 2 | `<input type="file">` cho media | `image_picker` + native camera + permission flow | Mobile media access = multi-step permission + native API |
| 3 | Web Push API + Service Worker | **FCM/APNs** — platform-specific setup + token management | Mobile push = separate infra cho iOS vs Android |
| 4 | Browser handles permissions transparently | **Explicit permission flow**: request → check → handle denial | "Permanently denied" = phải redirect user vào Settings |

---

## 9. Tổng kết

### ✅ Checklist kiến thức buổi 15

Sau buổi học, bạn cần nắm được:

- [ ] **Platform Channels:** Hiểu 3 loại (MethodChannel, EventChannel, BasicMessageChannel) và khi nào dùng loại nào
- [ ] **MethodChannel:** Có thể gọi native method từ Dart và nhận kết quả
- [ ] **Pigeon:** Hiểu ưu điểm, biết cách define API và generate code type-safe
- [ ] **Native features:** Biết cách dùng ít nhất 3–4 plugin phổ biến (url_launcher, camera, geolocator, share_plus...)
- [ ] **Plugin vs Package:** Phân biệt được, biết cấu trúc plugin
- [ ] **Federated architecture:** Hiểu tại sao Flutter dùng kiến trúc này cho plugin
- [ ] **Permissions:** Biết flow xin quyền đầy đủ (check → explain → request → handle denied)
- [ ] **Info.plist & AndroidManifest:** Biết khai báo permission cho từng platform
- [ ] **Platform-specific UI:** Có thể tạo UI thích ứng iOS/Android
- [ ] **Error handling:** Biết handle PlatformException, MissingPluginException

### Chuẩn bị cho buổi sau

> **Buổi 16: CI/CD & Production** — Build, deploy, và release app lên App Store / Google Play

---

### ➡️ Buổi tiếp theo

> **Buổi 16: CI/CD & Production** — Build modes, code signing, GitHub Actions, Fastlane, và release checklist cho production deployment.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Tạo GitHub repository cho project cuối khóa
