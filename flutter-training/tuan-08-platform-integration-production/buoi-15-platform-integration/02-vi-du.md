# Buổi 15: Platform Integration — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## VD1: MethodChannel — Lấy Battery Level từ Native 🟡

### Mục đích
Tạo MethodChannel để gọi native code lấy mức pin thiết bị — minh họa cơ chế Dart ↔ Native communication.

> **Liên quan tới:** [1. Platform Channels 🟡](01-ly-thuyet.md#1-platform-channels)

### Dart Side

```dart
// lib/battery_service.dart
import 'package:flutter/services.dart';

class BatteryService {
  // Tên channel phải giống hệt bên native
  static const _channel = MethodChannel('com.example.app/battery');

  /// Lấy mức pin từ native
  Future<int> getBatteryLevel() async {
    try {
      final int level = await _channel.invokeMethod('getBatteryLevel');
      return level;
    } on PlatformException catch (e) {
      throw Exception('Failed to get battery level: ${e.message}');
    }
  }
}
```

```dart
// lib/battery_screen.dart
import 'package:flutter/material.dart';
import 'battery_service.dart';

class BatteryScreen extends StatefulWidget {
  const BatteryScreen({super.key});

  @override
  State<BatteryScreen> createState() => _BatteryScreenState();
}

class _BatteryScreenState extends State<BatteryScreen> {
  final _batteryService = BatteryService();
  String _batteryLevel = 'Chưa kiểm tra';

  Future<void> _checkBattery() async {
    try {
      final level = await _batteryService.getBatteryLevel();
      setState(() => _batteryLevel = '$level%');
    } catch (e) {
      setState(() => _batteryLevel = 'Lỗi: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Battery Level')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.battery_std,
              size: 80,
              color: _getBatteryColor(),
            ),
            const SizedBox(height: 16),
            Text(
              'Mức pin: $_batteryLevel',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _checkBattery,
              child: const Text('Kiểm tra pin'),
            ),
          ],
        ),
      ),
    );
  }

  Color _getBatteryColor() {
    if (_batteryLevel.contains('%')) {
      final level = int.tryParse(_batteryLevel.replaceAll('%', '')) ?? 0;
      if (level > 50) return Colors.green;
      if (level > 20) return Colors.orange;
      return Colors.red;
    }
    return Colors.grey;
  }
}
```

### iOS Side (Swift)

```swift
// ios/Runner/AppDelegate.swift
import Flutter
import UIKit

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {

    let controller = window?.rootViewController as! FlutterViewController

    let batteryChannel = FlutterMethodChannel(
      name: "com.example.app/battery",
      binaryMessenger: controller.binaryMessenger
    )

    batteryChannel.setMethodCallHandler { (call, result) in
      if call.method == "getBatteryLevel" {
        self.receiveBatteryLevel(result: result)
      } else {
        result(FlutterMethodNotImplemented)
      }
    }

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  private func receiveBatteryLevel(result: FlutterResult) {
    let device = UIDevice.current
    device.isBatteryMonitoringEnabled = true

    if device.batteryState == .unknown {
      result(FlutterError(
        code: "UNAVAILABLE",
        message: "Battery level not available",
        details: nil
      ))
    } else {
      result(Int(device.batteryLevel * 100))
    }
  }
}
```

### Android Side (Kotlin)

```kotlin
// android/app/src/main/kotlin/.../MainActivity.kt
package com.example.app

import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import android.os.Build
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {
    private val CHANNEL = "com.example.app/battery"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                if (call.method == "getBatteryLevel") {
                    val batteryLevel = getBatteryLevel()
                    if (batteryLevel != -1) {
                        result.success(batteryLevel)
                    } else {
                        result.error(
                            "UNAVAILABLE",
                            "Battery level not available",
                            null
                        )
                    }
                } else {
                    result.notImplemented()
                }
            }
    }

    private fun getBatteryLevel(): Int {
        val batteryManager = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
        return batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
    }
}
```

### Giải thích flow

```
[Dart] getBatteryLevel()          [Native]
    │                                  │
    │── invokeMethod('getBattery') ──▶│
    │                                  │ Gọi native API
    │                                  │ (UIDevice / BatteryManager)
    │◀── result.success(85) ──────────│
    │                                  │
    ▼
  Hiển thị "85%"
```

- 🔗 **FE tương đương:** **Không có FE equivalent trực tiếp.** Platform Channel = native bridge (Dart ↔ Swift/Kotlin). Gần nhất là WebAssembly FFI hoặc React Native Bridge — nhưng FE developer cần học concept mới.

---

## VD2: Pigeon — Type-safe API Definition 🟢

### Mục đích
Sử dụng Pigeon để tạo type-safe platform channel — so sánh với MethodChannel thủ công.

> **Liên quan tới:** [2. Pigeon — Type-safe Platform Communication 🟢](01-ly-thuyet.md#2-pigeon--type-safe-platform-communication)

### Bước 1: Định nghĩa API

```dart
// pigeons/device_api.dart
import 'package:pigeon/pigeon.dart';

/// Cấu hình output
@ConfigurePigeon(PigeonOptions(
  dartOut: 'lib/src/generated/device_api.g.dart',
  swiftOut: 'ios/Runner/DeviceApi.g.swift',
  kotlinOut: 'android/app/src/main/kotlin/com/example/app/DeviceApi.g.kt',
))

/// Data class — sẽ tự động serialize/deserialize
class DeviceInfoMessage {
  String? deviceName;
  String? osName;
  String? osVersion;
  int? batteryLevel;
  bool? isPhysicalDevice;
}

class AppInfoRequest {
  String? packageName;
}

class AppInfoResponse {
  String? appName;
  String? version;
  String? buildNumber;
}

/// @HostApi = Dart gọi Native
@HostApi()
abstract class DeviceInfoApi {
  /// Lấy thông tin thiết bị
  DeviceInfoMessage getDeviceInfo();

  /// Lấy thông tin app
  AppInfoResponse getAppInfo(AppInfoRequest request);
}

/// @FlutterApi = Native gọi Dart
@FlutterApi()
abstract class DeviceEventApi {
  /// Native gọi khi battery thay đổi
  void onBatteryLevelChanged(int level);
}
```

### Bước 2: Generate code

```bash
# Cài pigeon
flutter pub add pigeon --dev

# Generate
dart run pigeon --input pigeons/device_api.dart
```

### Bước 3: Sử dụng trong Dart (generated code)

```dart
// lib/device_screen.dart
import 'package:flutter/material.dart';
import 'src/generated/device_api.g.dart';

class DeviceScreen extends StatefulWidget {
  const DeviceScreen({super.key});

  @override
  State<DeviceScreen> createState() => _DeviceScreenState();
}

class _DeviceScreenState extends State<DeviceScreen> {
  final _deviceApi = DeviceInfoApi();  // ← Type-safe! IDE autocomplete
  DeviceInfoMessage? _deviceInfo;
  String? _error;

  Future<void> _loadDeviceInfo() async {
    try {
      // ✅ Type-safe call — compiler kiểm tra argument và return type
      final info = await _deviceApi.getDeviceInfo();
      setState(() {
        _deviceInfo = info;
        _error = null;
      });
    } catch (e) {
      setState(() => _error = e.toString());
    }
  }

  @override
  void initState() {
    super.initState();
    _loadDeviceInfo();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Device Info (Pigeon)')),
      body: _error != null
          ? Center(child: Text('Lỗi: $_error'))
          : _deviceInfo == null
              ? const Center(child: CircularProgressIndicator())
              : Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      _InfoRow('Device', _deviceInfo!.deviceName ?? 'N/A'),
                      _InfoRow('OS', _deviceInfo!.osName ?? 'N/A'),
                      _InfoRow('Version', _deviceInfo!.osVersion ?? 'N/A'),
                      _InfoRow('Battery', '${_deviceInfo!.batteryLevel ?? 0}%'),
                      _InfoRow(
                        'Physical Device',
                        _deviceInfo!.isPhysicalDevice == true ? 'Yes' : 'No',
                      ),
                    ],
                  ),
                ),
    );
  }
}

class _InfoRow extends StatelessWidget {
  final String label;
  final String value;

  const _InfoRow(this.label, this.value);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        children: [
          SizedBox(
            width: 120,
            child: Text(
              label,
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(child: Text(value)),
        ],
      ),
    );
  }
}
```

### Bước 4: Implement bên Native (Swift)

```swift
// ios/Runner/DeviceApiImpl.swift
import Flutter

class DeviceInfoApiImpl: DeviceInfoApi {
    func getDeviceInfo() throws -> DeviceInfoMessage {
        let device = UIDevice.current
        device.isBatteryMonitoringEnabled = true

        let info = DeviceInfoMessage(
            deviceName: device.name,
            osName: device.systemName,
            osVersion: device.systemVersion,
            batteryLevel: Int64(device.batteryLevel * 100),
            isPhysicalDevice: true  // Simplified
        )
        return info
    }

    func getAppInfo(request: AppInfoRequest) throws -> AppInfoResponse {
        let bundle = Bundle.main
        return AppInfoResponse(
            appName: bundle.infoDictionary?["CFBundleName"] as? String,
            version: bundle.infoDictionary?["CFBundleShortVersionString"] as? String,
            buildNumber: bundle.infoDictionary?["CFBundleVersion"] as? String
        )
    }
}

// Đăng ký trong AppDelegate:
// DeviceInfoApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: DeviceInfoApiImpl())
```

### So sánh: Trước (MethodChannel) vs Sau (Pigeon)

```dart
// ❌ TRƯỚC — MethodChannel thủ công
final result = await channel.invokeMethod('getDeviceInfo');
final name = result['deviceName'] as String;  // Runtime error nếu sai key/type
final os = result['osVersion'] as String;     // Không có IDE support

// ✅ SAU — Pigeon
final info = await deviceApi.getDeviceInfo();
final name = info.deviceName;  // Compile-time check, autocomplete
final os = info.osVersion;     // IDE biết chính xác type
```

---

## VD3: Camera + Permission — Chụp ảnh với xin quyền 🟡

### Mục đích
Tích hợp camera với permission handling đầy đủ — minh họa cách kết hợp plugin + permission.

> **Liên quan tới:** [3. Native Features — Các tính năng native phổ biến 🟢](01-ly-thuyet.md#3-native-features--các-tính-năng-native-phổ-biến)

### Setup

```yaml
# pubspec.yaml
dependencies:
  camera: ^0.11.0+2
  permission_handler: ^11.3.1
```

```xml
<!-- ios/Runner/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>Ứng dụng cần truy cập camera để chụp ảnh</string>
```

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
```

### Code

```dart
// lib/features/camera/camera_capture_screen.dart
import 'dart:io';
import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

class CameraCaptureScreen extends StatefulWidget {
  const CameraCaptureScreen({super.key});

  @override
  State<CameraCaptureScreen> createState() => _CameraCaptureScreenState();
}

class _CameraCaptureScreenState extends State<CameraCaptureScreen> {
  CameraController? _controller;
  bool _isPermissionGranted = false;
  bool _isInitializing = true;
  String? _capturedPhotoPath;
  String? _errorMessage;

  @override
  void initState() {
    super.initState();
    _initializeCamera();
  }

  /// Flow: Check permission → Request if needed → Init camera
  Future<void> _initializeCamera() async {
    setState(() {
      _isInitializing = true;
      _errorMessage = null;
    });

    // Bước 1: Kiểm tra permission
    final status = await Permission.camera.status;

    if (status.isGranted) {
      _isPermissionGranted = true;
      await _setupCamera();
    } else if (status.isDenied) {
      // Bước 2: Xin permission
      final newStatus = await Permission.camera.request();
      if (newStatus.isGranted) {
        _isPermissionGranted = true;
        await _setupCamera();
      } else {
        setState(() {
          _isInitializing = false;
          _errorMessage = 'Cần quyền camera để sử dụng tính năng này';
        });
      }
    } else if (status.isPermanentlyDenied) {
      setState(() {
        _isInitializing = false;
        _errorMessage = 'Quyền camera bị từ chối vĩnh viễn. '
            'Vui lòng mở Cài đặt để cấp quyền.';
      });
    }
  }

  Future<void> _setupCamera() async {
    try {
      final cameras = await availableCameras();
      if (cameras.isEmpty) {
        setState(() {
          _isInitializing = false;
          _errorMessage = 'Không tìm thấy camera trên thiết bị';
        });
        return;
      }

      // Ưu tiên camera sau
      final backCamera = cameras.firstWhere(
        (c) => c.lensDirection == CameraLensDirection.back,
        orElse: () => cameras.first,
      );

      _controller = CameraController(
        backCamera,
        ResolutionPreset.high,
        enableAudio: false,
      );

      await _controller!.initialize();

      if (mounted) {
        setState(() => _isInitializing = false);
      }
    } catch (e) {
      setState(() {
        _isInitializing = false;
        _errorMessage = 'Không thể khởi tạo camera: $e';
      });
    }
  }

  Future<void> _capturePhoto() async {
    if (_controller == null || !_controller!.value.isInitialized) return;

    try {
      final XFile photo = await _controller!.takePicture();
      setState(() => _capturedPhotoPath = photo.path);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Lỗi chụp ảnh: $e')),
      );
    }
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chụp ảnh')),
      body: _buildBody(),
    );
  }

  Widget _buildBody() {
    // Đang khởi tạo
    if (_isInitializing) {
      return const Center(child: CircularProgressIndicator());
    }

    // Lỗi / không có quyền
    if (_errorMessage != null) {
      return _buildErrorView();
    }

    // Đã chụp ảnh → hiển thị preview
    if (_capturedPhotoPath != null) {
      return _buildPhotoPreview();
    }

    // Camera preview
    return _buildCameraPreview();
  }

  Widget _buildErrorView() {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.camera_alt_outlined, size: 64, color: Colors.grey),
            const SizedBox(height: 16),
            Text(
              _errorMessage!,
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 24),
            if (_errorMessage!.contains('Cài đặt'))
              ElevatedButton(
                onPressed: () => openAppSettings(),
                child: const Text('Mở Cài đặt'),
              )
            else
              ElevatedButton(
                onPressed: _initializeCamera,
                child: const Text('Thử lại'),
              ),
          ],
        ),
      ),
    );
  }

  Widget _buildCameraPreview() {
    return Stack(
      children: [
        // Camera preview full screen
        SizedBox.expand(
          child: FittedBox(
            fit: BoxFit.cover,
            child: SizedBox(
              width: _controller!.value.previewSize!.height,
              height: _controller!.value.previewSize!.width,
              child: CameraPreview(_controller!),
            ),
          ),
        ),
        // Nút chụp ảnh
        Positioned(
          bottom: 40,
          left: 0,
          right: 0,
          child: Center(
            child: GestureDetector(
              onTap: _capturePhoto,
              child: Container(
                width: 72,
                height: 72,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  border: Border.all(color: Colors.white, width: 4),
                  color: Colors.white.withValues(alpha: 0.3),
                ),
                child: const Icon(Icons.camera, color: Colors.white, size: 36),
              ),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildPhotoPreview() {
    return Column(
      children: [
        Expanded(
          child: Image.file(
            File(_capturedPhotoPath!),
            fit: BoxFit.contain,
          ),
        ),
        Padding(
          padding: const EdgeInsets.all(16),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              ElevatedButton.icon(
                onPressed: () {
                  setState(() => _capturedPhotoPath = null);
                },
                icon: const Icon(Icons.refresh),
                label: const Text('Chụp lại'),
              ),
              ElevatedButton.icon(
                onPressed: () {
                  // Xử lý lưu/sử dụng ảnh
                  Navigator.pop(context, _capturedPhotoPath);
                },
                icon: const Icon(Icons.check),
                label: const Text('Sử dụng'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

### Giải thích flow

```
User bấm "Chụp ảnh"
    │
    ▼
Check permission.camera.status
    │
    ├── Granted → Init camera → Show preview → Chụp → Preview ảnh
    │
    ├── Denied → Request permission
    │     ├── User cho phép → Init camera
    │     └── User từ chối → Hiện thông báo + nút "Thử lại"
    │
    └── Permanently Denied → Hiện thông báo + nút "Mở Cài đặt"
```

- 🔗 **FE tương đương:** Permission request ≈ `navigator.permissions.query()` / `navigator.mediaDevices.getUserMedia()` — nhưng mobile permission flow phức tạp hơn: system dialog + "permanently denied" state + redirect Settings.

---

## VD4: Plugin Structure — Cấu trúc một Flutter Plugin 🟢

### Mục đích
Hiểu cấu trúc đầy đủ của một Flutter plugin bằng cách tạo plugin đơn giản lấy thông tin device.

> **Liên quan tới:** [4. Plugin Development 🟢](01-ly-thuyet.md#4-plugin-development)

### Tạo plugin

```bash
flutter create --template=plugin \
  --org com.example \
  --platforms=android,ios \
  device_info_plugin
```

### Cấu trúc sinh ra

```
device_info_plugin/
├── lib/
│   ├── device_info_plugin.dart                    # 1. Public API
│   ├── device_info_plugin_platform_interface.dart  # 2. Platform interface
│   └── device_info_plugin_method_channel.dart      # 3. Default implementation
├── android/
│   └── src/main/kotlin/.../DeviceInfoPlugin.kt    # 4. Android native
├── ios/
│   └── Classes/DeviceInfoPlugin.swift              # 5. iOS native
├── example/                                        # 6. Example app
│   └── lib/main.dart
├── test/
│   └── device_info_plugin_test.dart               # 7. Tests
├── pubspec.yaml
├── README.md
├── CHANGELOG.md
└── LICENSE
```

### File 1: Public API (người dùng import cái này)

```dart
// lib/device_info_plugin.dart
import 'device_info_plugin_platform_interface.dart';

/// Plugin public API
class DeviceInfoPlugin {
  /// Lấy tên thiết bị
  Future<String?> getDeviceName() {
    return DeviceInfoPluginPlatform.instance.getDeviceName();
  }

  /// Lấy phiên bản OS
  Future<String?> getOsVersion() {
    return DeviceInfoPluginPlatform.instance.getOsVersion();
  }
}
```

### File 2: Platform Interface (contract)

```dart
// lib/device_info_plugin_platform_interface.dart
import 'package:plugin_platform_interface/plugin_platform_interface.dart';
import 'device_info_plugin_method_channel.dart';

abstract class DeviceInfoPluginPlatform extends PlatformInterface {
  DeviceInfoPluginPlatform() : super(token: _token);

  static final Object _token = Object();

  // Mặc định dùng MethodChannel implementation
  static DeviceInfoPluginPlatform _instance = MethodChannelDeviceInfoPlugin();

  static DeviceInfoPluginPlatform get instance => _instance;

  static set instance(DeviceInfoPluginPlatform instance) {
    PlatformInterface.verifyToken(instance, _token);
    _instance = instance;
  }

  /// Các method cần implement ở mỗi platform
  Future<String?> getDeviceName() {
    throw UnimplementedError('getDeviceName() has not been implemented.');
  }

  Future<String?> getOsVersion() {
    throw UnimplementedError('getOsVersion() has not been implemented.');
  }
}
```

### File 3: MethodChannel Implementation (default)

```dart
// lib/device_info_plugin_method_channel.dart
import 'package:flutter/services.dart';
import 'device_info_plugin_platform_interface.dart';

class MethodChannelDeviceInfoPlugin extends DeviceInfoPluginPlatform {
  final methodChannel = const MethodChannel('device_info_plugin');

  @override
  Future<String?> getDeviceName() async {
    return await methodChannel.invokeMethod<String>('getDeviceName');
  }

  @override
  Future<String?> getOsVersion() async {
    return await methodChannel.invokeMethod<String>('getOsVersion');
  }
}
```

### File 4: Android Native (Kotlin)

```kotlin
// android/src/main/kotlin/.../DeviceInfoPlugin.kt
package com.example.device_info_plugin

import android.os.Build
import io.flutter.embedding.engine.plugins.FlutterPlugin
import io.flutter.plugin.common.MethodCall
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugin.common.MethodChannel.MethodCallHandler

class DeviceInfoPlugin : FlutterPlugin, MethodCallHandler {
    private lateinit var channel: MethodChannel

    override fun onAttachedToEngine(binding: FlutterPlugin.FlutterPluginBinding) {
        channel = MethodChannel(binding.binaryMessenger, "device_info_plugin")
        channel.setMethodCallHandler(this)
    }

    override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
        when (call.method) {
            "getDeviceName" -> result.success("${Build.MANUFACTURER} ${Build.MODEL}")
            "getOsVersion" -> result.success("Android ${Build.VERSION.RELEASE}")
            else -> result.notImplemented()
        }
    }

    override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {
        channel.setMethodCallHandler(null)
    }
}
```

### File 5: iOS Native (Swift)

```swift
// ios/Classes/DeviceInfoPlugin.swift
import Flutter
import UIKit

public class DeviceInfoPlugin: NSObject, FlutterPlugin {
    public static func register(with registrar: FlutterPluginRegistrar) {
        let channel = FlutterMethodChannel(
            name: "device_info_plugin",
            binaryMessenger: registrar.messenger()
        )
        let instance = DeviceInfoPlugin()
        registrar.addMethodCallDelegate(instance, channel: channel)
    }

    public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
        switch call.method {
        case "getDeviceName":
            result(UIDevice.current.name)
        case "getOsVersion":
            result("iOS \(UIDevice.current.systemVersion)")
        default:
            result(FlutterMethodNotImplemented)
        }
    }
}
```

### File 6: pubspec.yaml của plugin

```yaml
# pubspec.yaml
name: device_info_plugin
description: A Flutter plugin to get device information
version: 0.0.1
homepage: https://example.com

environment:
  sdk: ^3.5.0
  flutter: ">=3.24.0"

dependencies:
  flutter:
    sdk: flutter
  plugin_platform_interface: ^2.1.8

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  plugin:
    platforms:
      android:
        package: com.example.device_info_plugin
        pluginClass: DeviceInfoPlugin
      ios:
        pluginClass: DeviceInfoPlugin
```

### Sử dụng plugin

```dart
// example/lib/main.dart
import 'package:device_info_plugin/device_info_plugin.dart';

final plugin = DeviceInfoPlugin();
final name = await plugin.getDeviceName();   // "Samsung Galaxy S24"
final os = await plugin.getOsVersion();       // "Android 14"
```

---

## VD5: Platform-adaptive UI — Dialog theo Platform 🟢

### Mục đích
Tạo UI tự động thay đổi theo platform: CupertinoAlertDialog trên iOS, AlertDialog trên Android.

> **Liên quan tới:** [6. Platform-specific UI 🟢](01-ly-thuyet.md#6-platform-specific-ui)

### Code

```dart
// lib/widgets/adaptive_dialog.dart
import 'dart:io' show Platform;
import 'package:flutter/cupertino.dart';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';

/// Helper hiển thị dialog theo platform
class AdaptiveDialog {
  /// Confirm dialog — tự động chọn style iOS hoặc Android
  static Future<bool> showConfirm(
    BuildContext context, {
    required String title,
    required String message,
    String confirmText = 'Đồng ý',
    String cancelText = 'Hủy',
    bool isDestructive = false,
  }) async {
    final bool isIOS = !kIsWeb && Platform.isIOS;

    if (isIOS) {
      return await _showCupertinoConfirm(
        context,
        title: title,
        message: message,
        confirmText: confirmText,
        cancelText: cancelText,
        isDestructive: isDestructive,
      );
    }

    return await _showMaterialConfirm(
      context,
      title: title,
      message: message,
      confirmText: confirmText,
      cancelText: cancelText,
      isDestructive: isDestructive,
    );
  }

  static Future<bool> _showCupertinoConfirm(
    BuildContext context, {
    required String title,
    required String message,
    required String confirmText,
    required String cancelText,
    required bool isDestructive,
  }) async {
    return await showCupertinoDialog<bool>(
          context: context,
          builder: (context) => CupertinoAlertDialog(
            title: Text(title),
            content: Text(message),
            actions: [
              CupertinoDialogAction(
                onPressed: () => Navigator.pop(context, false),
                child: Text(cancelText),
              ),
              CupertinoDialogAction(
                isDestructiveAction: isDestructive,
                onPressed: () => Navigator.pop(context, true),
                child: Text(confirmText),
              ),
            ],
          ),
        ) ??
        false;
  }

  static Future<bool> _showMaterialConfirm(
    BuildContext context, {
    required String title,
    required String message,
    required String confirmText,
    required String cancelText,
    required bool isDestructive,
  }) async {
    return await showDialog<bool>(
          context: context,
          builder: (context) => AlertDialog(
            title: Text(title),
            content: Text(message),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context, false),
                child: Text(cancelText),
              ),
              TextButton(
                onPressed: () => Navigator.pop(context, true),
                style: isDestructive
                    ? TextButton.styleFrom(foregroundColor: Colors.red)
                    : null,
                child: Text(confirmText),
              ),
            ],
          ),
        ) ??
        false;
  }
}
```

```dart
// lib/widgets/adaptive_list_tile.dart
import 'dart:io' show Platform;
import 'package:flutter/cupertino.dart';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';

/// ListTile thích ứng theo platform
class AdaptiveListTile extends StatelessWidget {
  final Widget? leading;
  final String title;
  final String? subtitle;
  final Widget? trailing;
  final VoidCallback? onTap;

  const AdaptiveListTile({
    super.key,
    this.leading,
    required this.title,
    this.subtitle,
    this.trailing,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    final bool isIOS = !kIsWeb && Platform.isIOS;

    if (isIOS) {
      return CupertinoListTile(
        leading: leading,
        title: Text(title),
        subtitle: subtitle != null ? Text(subtitle!) : null,
        trailing: trailing ?? const CupertinoListTileChevron(),
        onTap: onTap,
      );
    }

    return ListTile(
      leading: leading,
      title: Text(title),
      subtitle: subtitle != null ? Text(subtitle!) : null,
      trailing: trailing ?? const Icon(Icons.chevron_right),
      onTap: onTap,
    );
  }
}
```

### Demo Screen — Hiển thị cả hai style

```dart
// lib/screens/adaptive_demo_screen.dart
import 'dart:io' show Platform;
import 'package:flutter/cupertino.dart';
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';
import '../widgets/adaptive_dialog.dart';
import '../widgets/adaptive_list_tile.dart';

class AdaptiveDemoScreen extends StatefulWidget {
  const AdaptiveDemoScreen({super.key});

  @override
  State<AdaptiveDemoScreen> createState() => _AdaptiveDemoScreenState();
}

class _AdaptiveDemoScreenState extends State<AdaptiveDemoScreen> {
  bool _darkMode = false;
  bool _notifications = true;
  double _fontSize = 16;

  String get _platformName {
    if (kIsWeb) return 'Web';
    if (Platform.isIOS) return 'iOS';
    if (Platform.isAndroid) return 'Android';
    return 'Unknown';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Platform-Adaptive UI'),
      ),
      body: ListView(
        children: [
          // Header — hiển thị platform hiện tại
          Container(
            padding: const EdgeInsets.all(16),
            color: Theme.of(context).colorScheme.primaryContainer,
            child: Text(
              'Đang chạy trên: $_platformName',
              style: Theme.of(context).textTheme.titleMedium,
              textAlign: TextAlign.center,
            ),
          ),

          const SizedBox(height: 16),
          const Padding(
            padding: EdgeInsets.symmetric(horizontal: 16),
            child: Text('Settings',
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 18)),
          ),

          // Adaptive switches
          SwitchListTile.adaptive(
            title: const Text('Dark Mode'),
            subtitle: const Text('Bật giao diện tối'),
            value: _darkMode,
            onChanged: (v) => setState(() => _darkMode = v),
          ),
          SwitchListTile.adaptive(
            title: const Text('Notifications'),
            subtitle: const Text('Nhận thông báo'),
            value: _notifications,
            onChanged: (v) => setState(() => _notifications = v),
          ),

          // Adaptive slider
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Font size: ${_fontSize.round()}'),
                Slider.adaptive(
                  value: _fontSize,
                  min: 12,
                  max: 24,
                  onChanged: (v) => setState(() => _fontSize = v),
                ),
              ],
            ),
          ),

          const Divider(),

          // Adaptive list tiles
          AdaptiveListTile(
            leading: const Icon(Icons.person),
            title: 'Profile',
            subtitle: 'Quản lý thông tin cá nhân',
            onTap: () {},
          ),
          AdaptiveListTile(
            leading: const Icon(Icons.security),
            title: 'Privacy',
            subtitle: 'Cài đặt quyền riêng tư',
            onTap: () {},
          ),

          const Divider(),

          // Buttons — demo dialog
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () async {
                      final confirmed = await AdaptiveDialog.showConfirm(
                        context,
                        title: 'Xác nhận',
                        message: 'Bạn có chắc muốn lưu thay đổi?',
                      );
                      if (confirmed && mounted) {
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(content: Text('Đã lưu!')),
                        );
                      }
                    },
                    child: const Text('Hiện Confirm Dialog'),
                  ),
                ),
                const SizedBox(height: 8),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.red,
                      foregroundColor: Colors.white,
                    ),
                    onPressed: () async {
                      await AdaptiveDialog.showConfirm(
                        context,
                        title: 'Xóa tài khoản',
                        message: 'Hành động này không thể hoàn tác. Bạn có chắc?',
                        confirmText: 'Xóa',
                        isDestructive: true,
                      );
                    },
                    child: const Text('Hiện Destructive Dialog'),
                  ),
                ),
              ],
            ),
          ),

          // Adaptive progress indicator
          const Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('Loading: '),
                SizedBox(width: 8),
                CircularProgressIndicator.adaptive(),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### Kết quả

| Thành phần | iOS | Android |
|-----------|-----|---------|
| Dialog | CupertinoAlertDialog (rounded, centered actions) | AlertDialog (Material, text buttons) |
| Switch | CupertinoSwitch style | Material Switch style |
| Slider | Cupertino style | Material style |
| Progress | CupertinoActivityIndicator | CircularProgressIndicator |
| ListTile | CupertinoListTile + Chevron | ListTile + arrow icon |

---

## VD6: EventChannel — Streaming Data từ Native → Dart

### Mục đích
Tạo EventChannel để stream dữ liệu liên tục từ native side (sensor data) về Dart — minh họa cơ chế Native → Dart streaming với lifecycle đầy đủ.

> **Liên quan tới:** [1. Platform Channels 🟡](01-ly-thuyet.md#1-platform-channels)

### Dart Side

```dart
// lib/sensor_stream_screen.dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class SensorStreamScreen extends StatefulWidget {
  const SensorStreamScreen({super.key});

  @override
  State<SensorStreamScreen> createState() => _SensorStreamScreenState();
}

class _SensorStreamScreenState extends State<SensorStreamScreen> {
  static const _eventChannel = EventChannel('com.example/sensor_data');

  StreamSubscription? _subscription;
  double _sensorValue = 0;
  bool _isListening = false;

  void _startListening() {
    _subscription = _eventChannel.receiveBroadcastStream().listen(
      (event) {
        setState(() {
          _sensorValue = (event as num).toDouble();
          _isListening = true;
        });
      },
      onError: (error) {
        setState(() => _isListening = false);
        debugPrint('Sensor error: $error');
      },
      cancelOnError: true,
    );
  }

  void _stopListening() {
    _subscription?.cancel();
    _subscription = null;
    setState(() => _isListening = false);
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Sensor Stream')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.sensors,
              size: 64,
              color: _isListening ? Colors.green : Colors.grey,
            ),
            const SizedBox(height: 16),
            Text(
              'Sensor: ${_sensorValue.toStringAsFixed(1)}',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: _isListening ? _stopListening : _startListening,
              child: Text(_isListening ? 'Dừng' : 'Bắt đầu'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### iOS Side (Swift)

```swift
// ios/Runner/AppDelegate.swift
import Flutter
import UIKit

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController
    
    // EventChannel — streaming data từ Native → Dart
    let eventChannel = FlutterEventChannel(
      name: "com.example/sensor_data",
      binaryMessenger: controller.binaryMessenger
    )
    eventChannel.setStreamHandler(SensorStreamHandler())
    
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

// StreamHandler — quản lý lifecycle của stream
class SensorStreamHandler: NSObject, FlutterStreamHandler {
  private var timer: Timer?
  
  // Được gọi khi Dart side listen()
  func onListen(withArguments arguments: Any?, eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
      let sensorValue = Double.random(in: 0...100)
      events(sensorValue) // Gửi data về Dart
    }
    return nil
  }
  
  // Được gọi khi Dart side cancel()
  func onCancel(withArguments arguments: Any?) -> FlutterError? {
    timer?.invalidate()
    timer = nil
    return nil
  }
}
```

### Android Side (Kotlin)

```kotlin
// android/app/src/main/kotlin/.../MainActivity.kt
package com.example.app

import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.EventChannel
import android.os.Handler
import android.os.Looper
import kotlin.random.Random

class MainActivity : FlutterActivity() {
    private val SENSOR_CHANNEL = "com.example/sensor_data"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        // EventChannel — streaming data từ Native → Dart
        EventChannel(flutterEngine.dartExecutor.binaryMessenger, SENSOR_CHANNEL)
            .setStreamHandler(object : EventChannel.StreamHandler {
                private var handler: Handler? = null
                private var runnable: Runnable? = null

                // Được gọi khi Dart side listen()
                override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
                    handler = Handler(Looper.getMainLooper())
                    runnable = object : Runnable {
                        override fun run() {
                            val sensorValue = Random.nextDouble(0.0, 100.0)
                            events?.success(sensorValue) // Gửi data về Dart
                            handler?.postDelayed(this, 1000)
                        }
                    }
                    handler?.post(runnable!!)
                }

                // Được gọi khi Dart side cancel()
                override fun onCancel(arguments: Any?) {
                    runnable?.let { handler?.removeCallbacks(it) }
                    handler = null
                    runnable = null
                }
            })
    }
}
```

### Giải thích flow

```
[Dart] listen()                    [Native]
    │                                  │
    │── subscribe (onListen) ────────▶│
    │                                  │ Bắt đầu Timer / Handler
    │                                  │
    │◀── events(42.5) ────────────────│  ← mỗi 1 giây
    │◀── events(78.3) ────────────────│
    │◀── events(15.9) ────────────────│
    │     ...                          │
    │                                  │
    │── cancel (onCancel) ───────────▶│
    │                                  │ Hủy Timer / Handler
    ▼                                  ▼
```

> 🔄 **React Native ↔ Flutter**: Trong React Native, EventEmitter pattern (`NativeEventEmitter`) tương đương EventChannel. Khác biệt chính: Flutter EventChannel là bidirectional stream handler với lifecycle rõ ràng (`onListen`/`onCancel`), trong khi React Native EventEmitter fire-and-forget.

---

## VD7: 🤖 AI Gen → Review — MethodChannel + Permission 🟢

> **Mục đích:** Luyện workflow "AI gen platform integration code → bạn review channel consistency + error handling + permission flow → fix"

> **Liên quan tới:** [1. Platform Channels 🟡](01-ly-thuyet.md#1-platform-channels)

### Bước 1: Prompt cho AI

```text
Tạo MethodChannel cho "Battery Level" feature.
Flutter: BatteryService → getBatteryLevel() → int.
Android: Kotlin handler.
iOS: Swift handler.
Channel: 'com.myapp/battery'. Error handling + PlatformException.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Channel name** | 'com.myapp/battery' trên cả Flutter, Android, iOS? (mismatch = MissingPluginException!) |
| 2 | **Error propagation** | Native error → result.error(code, message, details)? (không swallow!) |
| 3 | **Method name** | "getBatteryLevel" trên cả 3 sides? (case-sensitive!) |
| 4 | **Return type** | Consistent return type? (int on all sides) |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Channel name mismatch
// Flutter:
const channel = MethodChannel('com.myapp/battery');
// Android:
MethodChannel(flutterEngine.dartExecutor, "myapp/battery") // KHÁC!

// ✅ FIX: Consistent name
// Flutter + Android + iOS: 'com.myapp/battery'
```

```kotlin
// ❌ LỖI 2: Swallow error on native side (Android Kotlin)
override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
  try {
    val level = getBatteryLevel()
    result.success(level)
  } catch (e: Exception) {
    // THIẾU result.error()! Flutter side nhận NOTHING → timeout
  }
}

// ✅ FIX: Always return result (success OR error)
catch (e: Exception) {
  result.error("BATTERY_UNAVAILABLE", e.message, null)
}
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết channel name PHẢI consistent cả 3 sides (case-sensitive)
- ✅ Nhận ra AI hay swallow native errors (phải dùng result.error())
- ✅ Verify method name consistency giữa Flutter và native
