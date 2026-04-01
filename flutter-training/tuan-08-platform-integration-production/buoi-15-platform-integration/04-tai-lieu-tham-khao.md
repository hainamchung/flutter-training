# Buổi 15: Platform Integration — Tài liệu tham khảo

## 📖 Tài liệu chính thức

### Flutter Documentation

| Chủ đề | Link |
|--------|------|
| Platform Integration tổng quan | https://docs.flutter.dev/platform-integration/platform-channels |
| Writing platform-specific code | https://docs.flutter.dev/platform-integration/platform-channels |
| Pigeon documentation | https://pub.dev/packages/pigeon |
| Developing packages & plugins | https://docs.flutter.dev/packages-and-plugins/developing-packages |
| Federated plugins | https://docs.flutter.dev/packages-and-plugins/developing-packages#federated-plugins |
| Platform-adaptive UI | https://docs.flutter.dev/platform-integration/platform-adaptations |
| Android setup | https://docs.flutter.dev/get-started/install/macos/mobile-android |
| iOS setup | https://docs.flutter.dev/get-started/install/macos/mobile-ios |

### Dart Documentation

| Chủ đề | Link |
|--------|------|
| dart:ffi (Foreign Function Interface) | https://dart.dev/interop/c-interop |
| Async programming | https://dart.dev/libraries/async/async-await |

---

## 📦 Packages quan trọng

### Core — Platform Communication

| Package | Mô tả | Link |
|---------|--------|------|
| `pigeon` | Type-safe platform channel code generator | https://pub.dev/packages/pigeon |
| `plugin_platform_interface` | Base class cho federated plugin interface | https://pub.dev/packages/plugin_platform_interface |

### Native Features — Phổ biến nhất

| Package | Tính năng | Link |
|---------|-----------|------|
| `url_launcher` | Mở URL, email, phone, SMS | https://pub.dev/packages/url_launcher |
| `share_plus` | Chia sẻ text, file qua share sheet | https://pub.dev/packages/share_plus |
| `camera` | Truy cập camera, chụp ảnh, quay video | https://pub.dev/packages/camera |
| `image_picker` | Chọn ảnh từ gallery hoặc camera | https://pub.dev/packages/image_picker |
| `geolocator` | GPS, vị trí hiện tại, khoảng cách | https://pub.dev/packages/geolocator |
| `permission_handler` | Quản lý runtime permissions | https://pub.dev/packages/permission_handler |
| `local_auth` | Xác thực sinh trắc (Face ID, Touch ID, Fingerprint) | https://pub.dev/packages/local_auth |
| `firebase_messaging` | Push notifications qua FCM | https://pub.dev/packages/firebase_messaging |
| `package_info_plus` | Thông tin app (version, build, package name) | https://pub.dev/packages/package_info_plus |
| `file_picker` | Chọn file từ hệ thống | https://pub.dev/packages/file_picker |

### Native Features — Bổ sung

| Package | Tính năng | Link |
|---------|-----------|------|
| `connectivity_plus` | Kiểm tra kết nối mạng | https://pub.dev/packages/connectivity_plus |
| `device_info_plus` | Thông tin thiết bị chi tiết | https://pub.dev/packages/device_info_plus |
| `sensors_plus` | Accelerometer, gyroscope | https://pub.dev/packages/sensors_plus |
| `battery_plus` | Thông tin pin | https://pub.dev/packages/battery_plus |
| `flutter_local_notifications` | Local notifications | https://pub.dev/packages/flutter_local_notifications |
| `path_provider` | Đường dẫn thư mục hệ thống | https://pub.dev/packages/path_provider |
| `flutter_secure_storage` | Lưu trữ bảo mật (Keychain/Keystore) | https://pub.dev/packages/flutter_secure_storage |

---

## 📝 Bài viết & Blog

### Platform Channels

| Tiêu đề | Mô tả |
|---------|--------|
| [Flutter Platform Channels](https://medium.com/flutter/flutter-platform-channels-ce7f540a104e) | Bài viết chính thức từ Flutter team giải thích cơ chế hoạt động |
| [Writing custom platform-specific code](https://docs.flutter.dev/platform-integration/platform-channels) | Hướng dẫn step-by-step từ official docs |

### Pigeon

| Tiêu đề | Mô tả |
|---------|--------|
| [Pigeon — pub.dev](https://pub.dev/packages/pigeon) | Documentation chính thức, có example đầy đủ |
| [Using Pigeon for platform channels](https://medium.com/flutter/pigeon-flutter-platform-channel-code-generator-35b0abc701c9) | Tutorial từ Flutter team |

### Plugin Development

| Tiêu đề | Mô tả |
|---------|--------|
| [Developing packages & plugins](https://docs.flutter.dev/packages-and-plugins/developing-packages) | Official guide tạo plugin từ đầu |
| [Flutter Favorite Plugins](https://pub.dev/flutter/favorites) | Danh sách plugin được Flutter team recommend |

### Permissions

| Tiêu đề | Mô tả |
|---------|--------|
| [permission_handler — Usage guide](https://pub.dev/packages/permission_handler) | Documentation đầy đủ kèm các platform-specific notes |
| [iOS Info.plist keys reference](https://developer.apple.com/documentation/bundleresources/information-property-list) | Apple documentation cho các privacy keys |
| [Android permissions overview](https://developer.android.com/guide/topics/permissions/overview) | Google documentation cho Android permission model |

---

## 🎬 Video hướng dẫn

### Từ Flutter Team

| Video | Nội dung |
|-------|----------|
| [How to use MethodChannel in Flutter](https://www.youtube.com/watch?v=ECkQoGAzQOI) | Official tutorial về Platform Channels |
| [Package of the Week: pigeon](https://www.youtube.com/watch?v=iYBPKFEm4kE) | Giới thiệu Pigeon từ Flutter series |
| [How to write a Flutter plugin](https://www.youtube.com/watch?v=BXMFlmkqNYs) | Step-by-step tạo Flutter plugin |

### Từ cộng đồng

| Video | Nội dung |
|-------|----------|
| Flutter Platform Channels Deep Dive | Giải thích chi tiết cơ chế binary messaging |
| Camera Integration Tutorial | Hướng dẫn tích hợp camera đầy đủ |
| Permission Handling Best Practices | Cách xử lý permission chuyên nghiệp |

---

## 🔧 Tools & Resources

### Development

| Tool | Mô tả |
|------|--------|
| [pub.dev](https://pub.dev) | Package repository chính thức |
| [Flutter DevTools](https://docs.flutter.dev/tools/devtools/overview) | Debug tools bao gồm performance overlay |
| [Xcode](https://developer.apple.com/xcode/) | IDE cho iOS development, cần thiết khi viết Swift code |
| [Android Studio](https://developer.android.com/studio) | IDE cho Android development, Kotlin support |

### Testing

| Tool | Mô tả |
|------|--------|
| [flutter_test](https://api.flutter.dev/flutter/flutter_test/flutter_test-library.html) | Unit & widget testing |
| [integration_test](https://docs.flutter.dev/testing/integration-tests) | End-to-end testing |

---

## 📊 So sánh nhanh với React Native / Web

| Concept | Web | React Native | Flutter |
|---------|-----|-------------|---------|
| Native bridge | N/A | JS Bridge → Turbo Modules | Platform Channels → Pigeon |
| Native modules | N/A | npm native modules | Flutter Plugins |
| Permissions | Browser prompt | `react-native-permissions` | `permission_handler` |
| Camera | `getUserMedia()` | `react-native-camera` | `camera` / `image_picker` |
| Geolocation | `navigator.geolocation` | `@react-native-community/geolocation` | `geolocator` |
| Share | `navigator.share()` | `react-native-share` | `share_plus` |
| Open URL | `window.open()` | `Linking.openURL()` | `url_launcher` |

---

## 📋 Quick Reference Card

### MethodChannel — Cheat Sheet

```dart
// Dart → Native (gọi method)
final result = await channel.invokeMethod<String>('methodName', arguments);

// Native → Dart (nhận method call)
channel.setMethodCallHandler((call) async {
  switch (call.method) {
    case 'onEvent': return handleEvent(call.arguments);
    default: throw MissingPluginException();
  }
});

// Error handling
try {
  await channel.invokeMethod('method');
} on PlatformException catch (e) {
  // Native trả error: e.code, e.message, e.details
} on MissingPluginException {
  // Method chưa implement bên native
}
```

### Permission — Cheat Sheet

```dart
// Check
final status = await Permission.camera.status;
// status: .granted, .denied, .restricted, .limited, .permanentlyDenied

// Request
final result = await Permission.camera.request();

// Request nhiều quyền cùng lúc
final statuses = await [
  Permission.camera,
  Permission.photos,
].request();

// Mở Settings app
await openAppSettings();
```

### Platform Check — Cheat Sheet

```dart
import 'dart:io' show Platform;
import 'package:flutter/foundation.dart' show kIsWeb;

// ⚠️ Luôn check kIsWeb TRƯỚC Platform.isXxx
if (kIsWeb) { /* web */ }
else if (Platform.isIOS) { /* iOS */ }
else if (Platform.isAndroid) { /* Android */ }
```

---

## 🤖 AI Prompt Library — Buổi 15: Platform Integration

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Platform Integration trong Flutter. Background: 4+ năm React (React Native bridge, native modules).
Câu hỏi: MethodChannel giống React Native NativeModules? EventChannel giống NativeEventEmitter? Pigeon giống Turbo Modules? Permission handling có gì khác giữa Flutter và React Native?
Yêu cầu: mapping 1-1, highlight Flutter-specific patterns (channel name consistency, PlatformException).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần MethodChannel setup cho "Device Info" feature.
Flutter: DeviceInfoService → getDeviceName(), getBatteryLevel().
Android: Kotlin MethodCallHandler.
iOS: Swift FlutterMethodCallHandler.
Channel: 'com.myapp/device_info'.
Output: 3 files + error handling.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Platform Integration code sau:
[paste Flutter + native code]

Kiểm tra:
1. Channel name consistent? (Flutter = Android = iOS)
2. Error codes consistent? (same strings both sides)
3. PlatformException caught on Flutter side?
4. Permission: all statuses handled? (denied, permanentlyDenied, restricted)
5. Info.plist + AndroidManifest entries present?
6. Null safety: native null return handled?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Platform Integration trong Flutter:
[paste error: MissingPluginException, PlatformException, etc.]

Flutter code:
[paste]
Native code (Android/iOS):
[paste]

Cần: (1) Root cause (channel mismatch? handler not registered?), (2) Fix, (3) Testing steps.
```
```
