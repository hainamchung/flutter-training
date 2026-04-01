# Buổi 15: Platform Integration — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Platform Integration là lĩnh vực **khác biệt lớn nhất** so với FE development.
> Nhiều concepts ở đây **không có equivalent** trong web — FE dev cần học hoàn toàn mới.

| FE Expectation | Flutter Reality | Bài tập liên quan |
|----------------|-----------------|---------------------|
| Browser sandbox quản lý hardware access | **Platform Channels** gọi native SDK trực tiếp | BT1 |
| `navigator.permissions.query()` simple | Permission flow phức tạp: check → request → handle "permanently denied" | BT1, BT2 |
| `<input type="file">` cho media | `image_picker` + camera permission + file handling | BT2 |
| Web Push API = browser-managed | FCM/APNs setup + token management + notification channels | BT3 |

---

## BT1 ⭐ — Sử dụng Native Feature Plugins 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_native_features` |
| **Thêm packages** | `flutter pub add url_launcher share_plus package_info_plus` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — App info, mở URL, chia sẻ nội dung |

### Yêu cầu

Tạo một Flutter app sử dụng 3 plugin native feature có sẵn:

1. **`url_launcher`** — Mở một URL trong browser
2. **`share_plus`** — Chia sẻ nội dung text
3. **`package_info_plus`** — Hiển thị thông tin phiên bản app

### Giao diện mong đợi

```
┌─────────────────────────────┐
│      App Info & Sharing     │
├─────────────────────────────┤
│                             │
│  📱 Thông tin ứng dụng     │
│  ─────────────────────────  │
│  App Name: MyApp            │
│  Version: 1.0.0             │
│  Build: 1                   │
│  Package: com.example.myapp │
│                             │
│  ─────────────────────────  │
│                             │
│  [🌐 Mở Flutter.dev]       │
│                             │
│  [📤 Chia sẻ app]          │
│                             │
└─────────────────────────────┘
```

### Hướng dẫn

#### Bước 1: Thêm dependencies

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  url_launcher: ^6.3.1
  share_plus: ^10.1.4
  package_info_plus: ^8.1.3
```

#### Bước 2: Cấu hình platform (nếu cần)

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<!-- url_launcher cần khai báo queries cho Android 11+ -->
<queries>
  <intent>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="https" />
  </intent>
</queries>
```

#### Bước 3: Tạo màn hình chính

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'features/app_info_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Native Features Demo',
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
      ),
      home: const AppInfoScreen(),
    );
  }
}
```

#### Bước 4: Implement features

Tạo file `lib/features/app_info_screen.dart` với các tính năng:

- Khi mở app → tự động load thông tin package
- Bấm "Mở Flutter.dev" → mở `https://flutter.dev` trong browser
- Bấm "Chia sẻ app" → mở share sheet với nội dung: "Check out MyApp v1.0.0!"

### Gợi ý code

```dart
// Load package info
final info = await PackageInfo.fromPlatform();

// Mở URL
final uri = Uri.parse('https://flutter.dev');
if (await canLaunchUrl(uri)) {
  await launchUrl(uri, mode: LaunchMode.externalApplication);
}

// Share
Share.share(
  'Check out ${info.appName} v${info.version}!',
  subject: 'Chia sẻ ứng dụng',
);
```

### Tiêu chí hoàn thành

- [ ] Hiển thị đúng thông tin app (name, version, build number, package name)
- [ ] Bấm nút mở được URL trong browser
- [ ] Bấm nút hiện được share sheet
- [ ] Xử lý trường hợp không mở được URL (hiện thông báo lỗi)
- [ ] Code sạch, tách logic ra khỏi UI

---

## BT2 ⭐⭐ — MethodChannel Communication 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_method_channel` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Device info từ native code qua MethodChannel |

### Yêu cầu

Tạo MethodChannel để giao tiếp với native platform:
1. Gọi native code lấy **device name** (tên thiết bị)
2. Gọi native code lấy **OS version**
3. Xử lý error khi native call thất bại

### Giao diện mong đợi

```
┌─────────────────────────────┐
│    Device Info (Native)     │
├─────────────────────────────┤
│                             │
│  📱 Device Information      │
│  ─────────────────────────  │
│  Device: iPhone 15 Pro      │
│  OS: iOS 17.4               │
│  Status: ✅ Connected       │
│                             │
│  [🔄 Refresh]              │
│                             │
│  ─────────────────────────  │
│  📋 Call Log:               │
│  [14:30:01] getDeviceName   │
│     → "iPhone 15 Pro" ✅    │
│  [14:30:01] getOsVersion    │
│     → "iOS 17.4" ✅         │
│                             │
└─────────────────────────────┘
```

### Hướng dẫn

#### Bước 1: Tạo Dart service

```dart
// lib/services/device_channel_service.dart
import 'package:flutter/services.dart';

class DeviceChannelService {
  static const _channel = MethodChannel('com.example.app/device_info');

  Future<String> getDeviceName() async {
    try {
      final String name = await _channel.invokeMethod('getDeviceName');
      return name;
    } on PlatformException catch (e) {
      throw Exception('Native error: ${e.code} - ${e.message}');
    } on MissingPluginException {
      throw Exception('Plugin not registered. Try full app restart.');
    }
  }

  Future<String> getOsVersion() async {
    // TODO: Tương tự getDeviceName
    throw UnimplementedError();
  }
}
```

#### Bước 2: Implement iOS native (Swift)

File: `ios/Runner/AppDelegate.swift`

```swift
// Trong didFinishLaunchingWithOptions:
let controller = window?.rootViewController as! FlutterViewController

let deviceChannel = FlutterMethodChannel(
    name: "com.example.app/device_info",
    binaryMessenger: controller.binaryMessenger
)

deviceChannel.setMethodCallHandler { (call, result) in
    switch call.method {
    case "getDeviceName":
        result(UIDevice.current.name)
    case "getOsVersion":
        result("iOS \(UIDevice.current.systemVersion)")
    default:
        result(FlutterMethodNotImplemented)
    }
}
```

#### Bước 3: Implement Android native (Kotlin)

File: `android/app/src/main/kotlin/.../MainActivity.kt`

```kotlin
// Trong configureFlutterEngine:
MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "com.example.app/device_info")
    .setMethodCallHandler { call, result ->
        when (call.method) {
            "getDeviceName" -> result.success("${Build.MANUFACTURER} ${Build.MODEL}")
            "getOsVersion" -> result.success("Android ${Build.VERSION.RELEASE}")
            else -> result.notImplemented()
        }
    }
```

#### Bước 4: Tạo UI với call log

Tạo màn hình hiển thị kết quả và log các lần gọi native.

### Yêu cầu nâng cao

- Thêm method `getBatteryLevel` (tham khảo VD1 trong file ví dụ)
- Hiển thị loading indicator khi đang gọi native
- Log thời gian mỗi lần call (bao lâu native trả về)
- Thêm nút "Test Error" — gọi một method không tồn tại để xem error handling

### Tiêu chí hoàn thành

- [ ] MethodChannel hoạt động trên **ít nhất 1 platform** (iOS hoặc Android)
- [ ] Lấy được device name từ native
- [ ] Lấy được OS version từ native
- [ ] Handle `PlatformException` với thông báo lỗi rõ ràng
- [ ] Handle `MissingPluginException`
- [ ] Có loading state khi đang gọi native
- [ ] Channel name Dart và Native match nhau

---

## BT3 ⭐⭐⭐ — Photo Picker với Permission Handling 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_photo_picker` |
| **Thêm packages** | `flutter pub add image_picker permission_handler` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Chọn/chụp ảnh với xử lý permission đầy đủ |

### Yêu cầu

Xây dựng tính năng chọn/chụp ảnh đầy đủ:

1. **Xin quyền** camera và gallery (permission_handler)
2. **Chụp ảnh** từ camera hoặc **chọn ảnh** từ gallery (image_picker)
3. **Hiển thị** ảnh đã chọn trong app
4. **Xử lý mọi trường hợp** permission bị từ chối

### Giao diện mong đợi

```
┌─────────────────────────────┐
│       Photo Picker          │
├─────────────────────────────┤
│                             │
│  ┌───────────────────────┐  │
│  │                       │  │
│  │    [Ảnh đã chọn]     │  │
│  │    hoặc placeholder   │  │
│  │                       │  │
│  └───────────────────────┘  │
│                             │
│  [📷 Chụp ảnh] [🖼 Chọn từ gallery]│
│                             │
│  Status: Đã chọn ảnh ✅    │
│                             │
└─────────────────────────────┘
```

### Hướng dẫn

#### Bước 1: Setup dependencies và permissions

```yaml
# pubspec.yaml
dependencies:
  image_picker: ^1.1.2
  permission_handler: ^11.3.1
```

```xml
<!-- ios/Runner/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>Ứng dụng cần camera để chụp ảnh đại diện</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Ứng dụng cần truy cập thư viện ảnh để chọn ảnh</string>
```

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
```

#### Bước 2: Tạo PermissionService

```dart
// lib/services/permission_service.dart

class PermissionService {
  /// Xin quyền với đầy đủ flow
  /// Returns: true nếu được cấp quyền, false nếu bị từ chối
  Future<bool> requestPermission(
    BuildContext context,
    Permission permission, {
    required String rationaleTitle,
    required String rationaleMessage,
  }) async {
    // TODO: Implement theo flow:
    // 1. Check status
    // 2. Nếu denied → show rationale → request
    // 3. Nếu permanently denied → show dialog mở Settings
    // 4. Return true/false
    throw UnimplementedError();
  }
}
```

#### Bước 3: Tạo PhotoPickerService

```dart
// lib/services/photo_picker_service.dart

enum PhotoSource { camera, gallery }

class PhotoPickerService {
  final ImagePicker _picker = ImagePicker();
  final PermissionService _permissionService = PermissionService();

  /// Chọn ảnh với permission check
  Future<File?> pickPhoto(
    BuildContext context,
    PhotoSource source,
  ) async {
    // TODO: Implement:
    // 1. Xin quyền tương ứng (camera hoặc photos)
    // 2. Nếu được cấp → pick/capture ảnh
    // 3. Return File hoặc null
    throw UnimplementedError();
  }
}
```

#### Bước 4: Tạo PhotoPickerScreen

Tạo màn hình với:
- Placeholder khi chưa chọn ảnh (icon camera + text hướng dẫn)
- Hiển thị ảnh đã chọn (Image.file)
- 2 nút: "Chụp ảnh" và "Chọn từ gallery"
- Status text hiển thị trạng thái

### Flow chi tiết cần xử lý

```
User bấm "Chụp ảnh"
    │
    ▼
Check camera permission
    │
    ├── Granted
    │     └── Mở camera → Chụp → Hiển thị ảnh
    │
    ├── Denied
    │     └── Hiện dialog giải thích
    │           └── User đồng ý → Request permission
    │                 ├── Granted → Mở camera
    │                 └── Denied → Hiện thông báo
    │
    └── Permanently Denied
          └── Hiện dialog "Mở Cài đặt"
                ├── User bấm "Mở" → openAppSettings()
                └── User bấm "Để sau" → Không làm gì

User bấm "Chọn từ gallery"
    │
    ▼
Check photos permission
    │
    └── (Tương tự flow trên)
```

### Tiêu chí hoàn thành

- [ ] Chụp được ảnh từ camera
- [ ] Chọn được ảnh từ gallery
- [ ] Hiển thị ảnh đã chọn/chụp trong app
- [ ] Permission flow đầy đủ:
  - [ ] Check trước khi request
  - [ ] Hiện rationale dialog giải thích lý do
  - [ ] Handle denied — cho phép thử lại
  - [ ] Handle permanently denied — hướng dẫn mở Settings
- [ ] Khai báo đúng trong Info.plist (iOS) và AndroidManifest.xml (Android)
- [ ] Graceful degradation: app không crash khi user từ chối
- [ ] Code tách biệt: PermissionService riêng, PhotoPickerService riêng

### Bonus challenges

- Cho phép chọn nhiều ảnh từ gallery (hiển thị dạng grid)
- Thêm option crop ảnh trước khi sử dụng
- Lưu ảnh đã chọn vào local storage (kết hợp kiến thức Tuần 6)

---

## 💬 Câu hỏi thảo luận

### Câu 1: Khi nào nên viết native code?

> Trong thực tế, bạn gặp tình huống nào cần viết MethodChannel/Pigeon gọi native code thay vì dùng plugin có sẵn trên pub.dev?

**Gợi ý suy nghĩ:**
- Plugin trên pub.dev không hỗ trợ tính năng cụ thể bạn cần
- Cần tối ưu performance cho tác vụ nặng (image processing, ML...)
- Tích hợp SDK native của bên thứ ba (payment, analytics...)
- Plugin có sẵn bị outdated hoặc có bug chưa fix
- Cần truy cập API platform mới nhất (iOS 18, Android 15) mà plugin chưa cập nhật

### Câu 2: Plugin có sẵn vs tự viết (Build vs Buy)

> Team bạn cần tích hợp tính năng in-app purchase. Trên pub.dev có plugin `in_app_purchase` (chính thức từ Flutter team) và `purchases_flutter` (từ RevenueCat). Bạn chọn cái nào? Hay tự viết?

**Gợi ý suy nghĩ:**
- Official plugin: miễn phí, nhưng phải tự handle server-side verification
- RevenueCat: dễ dùng hơn, có dashboard, nhưng tốn phí & thêm dependency
- Tự viết: full control, nhưng tốn rất nhiều effort
- Xem xét: maintenance cost, team size, business requirement, time-to-market

### Câu 3: Web compatibility — làm sao handle?

> App của bạn cần chạy trên cả mobile và web. Nhưng nhiều plugin (camera, geolocator, local_auth) không hỗ trợ web hoặc hỗ trợ hạn chế. Bạn sẽ thiết kế architecture như thế nào?

**Gợi ý suy nghĩ:**
- Dùng `kIsWeb` để kiểm tra và cung cấp alternative
- Abstract hóa native features qua interface → platform-specific implementation
- Web alternatives: `dart:html` cho geolocation, `<input type="file">` cho file picker
- Graceful degradation: ẩn tính năng không hỗ trợ trên web
- Conditional imports: `import 'service_web.dart' if (dart.library.io) 'service_mobile.dart'`

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 8:** Focus vào AI gen platform-specific code và review error propagation + permissions.

### AI-BT1: Gen MethodChannel Native Camera (Permission + Error Propagation) ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Platform Channels (MethodChannel, EventChannel), Pigeon, native features, plugin development, permissions handling, platform-specific UI.
- **Task thực tế:** PM muốn custom camera feature — không dùng image_picker plugin (cần native camera UI). Cần MethodChannel setup Flutter ↔ Native + permission handling + error propagation. AI gen scaffolding, bạn review channel name consistency + error handling + permission edge cases.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần implement MethodChannel cho native camera trong Flutter.
Tech stack: Flutter 3.x, permission_handler ^11.x.
Platforms: Android (Kotlin) + iOS (Swift).

Gen các components:
1. Flutter: CameraService — requestPermission() + openCamera() + handleError().
2. Android: MainActivity.kt — MethodChannel handler, launch camera intent.
3. iOS: AppDelegate.swift — FlutterMethodCallHandler, UIImagePickerController.
4. Permission flow: check → rationale dialog → request → permanentlyDenied → Settings.
5. Error propagation: native exception → PlatformException(code, message, details).

Constraints:
- Channel name: 'com.myapp/camera' — SAME on all 3 sides.
- Error codes: CAMERA_UNAVAILABLE, PERMISSION_DENIED, CAPTURE_FAILED.
- Null return = user cancelled (not error).
- Handle MissingPluginException (desktop/web fallback).
- Permission: Info.plist NSCameraUsageDescription + AndroidManifest CAMERA permission.
Output: 3 files + platform config entries.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 3 files (Flutter + Android + iOS) + config entries.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Channel name CONSISTENT cả 3 sides? ('com.myapp/camera') | ☐ |
| 2 | Error codes CONSISTENT? (CAMERA_UNAVAILABLE trên cả Android + iOS) | ☐ |
| 3 | Permission: handle permanentlyDenied → openAppSettings()? | ☐ |
| 4 | Permission: iOS Info.plist NSCameraUsageDescription present? | ☐ |
| 5 | Error propagation: native →  PlatformException → Flutter catch? | ☐ |
| 6 | Null handling: user cancel = null, NOT exception? | ☐ |
| 7 | MissingPluginException handled (desktop/web)? | ☐ |

**4. Customize:**
Thêm EventChannel cho camera preview stream (real-time frames từ native → Flutter). AI gen MethodChannel (one-shot) — tự thêm EventChannel bidrectional stream + lifecycle management.
