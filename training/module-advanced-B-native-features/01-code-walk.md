# Code Walk — Advanced Native Features

> 📌 **Recap từ modules trước:**
> - **M0:** Dart basics, async/await — foundation for native APIs ([M0 § Dart](../module-00-dart-primer/01-code-walk.md))
> - **M1:** App entrypoint, platform channels — permission initialization ([M1 § Entrypoint](../module-01-app-entrypoint/01-code-walk.md))
> - **M5:** Built-in Widgets — widget catalog, layout, input ([M5 § Widgets](../module-05-built-in-widgets/01-code-walk.md))
> - **MA:** Security — secure storage, debug detection ([MA § Security](../module-advanced-A-performance-security/01-code-walk.md))
>
> Nếu chưa nắm vững → quay lại module tương ứng trước.

---

## Walk Order

```
image_picker (Camera/Gallery access)
    ↓
geolocator (Location services)
    ↓
local_auth (Biometric authentication)
    ↓
firebase_messaging (Push notifications)
    ↓
app_links (Deep linking)
```

---

## 1. Camera Integration — image_picker

> 💡 **FE Perspective**
> **Flutter:** `image_picker` package wraps platform-specific camera APIs.
> **React/Vue tương đương:** `<input type="file" accept="image/*">` hoặc `navigator.mediaDevices.getUserMedia()`.
> **Khác biệt quan trọng:** Flutter handles platform-specific permissions và image source selection automatically.

### Basic Usage

```dart
import 'package:image_picker/image_picker.dart';

class CameraService {
  final ImagePicker _picker = ImagePicker();
  
  /// Pick image from gallery
  Future<XFile?> pickFromGallery() async {
    return await _picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 1024,    // Resize for upload
      maxHeight: 1024,
      imageQuality: 85,  // Compress
    );
  }
  
  /// Take photo with camera
  Future<XFile?> takePhoto() async {
    return await _picker.pickImage(
      source: ImageSource.camera,
      maxWidth: 1920,    // Higher res for camera
      maxHeight: 1920,
      imageQuality: 90,
      preferredCameraDevice: CameraDevice.rear,
    );
  }
  
  /// Pick video
  Future<XFile?> pickVideo() async {
    return await _picker.pickVideo(
      source: ImageSource.camera,
      maxDuration: const Duration(minutes: 1),
    );
  }
}
```

### Permission Handling

```dart
// image_picker tự handle permissions nhưng có thể check trước
class CameraService {
  Future<bool> checkCameraPermission() async {
    final status = await Permission.camera.status;
    return status.isGranted;
  }
  
  Future<bool> requestCameraPermission() async {
    final status = await Permission.camera.request();
    return status.isGranted;
  }
  
  Future<XFile?> takePhotoWithPermissionCheck() async {
    // Check permission first
    if (!await checkCameraPermission()) {
      final granted = await requestCameraPermission();
      if (!granted) {
        // Show settings dialog
        return null;
      }
    }
    
    return await takePhoto();
  }
}
```

### Image Processing Pipeline

```dart
class ImageProcessingService {
  /// Crop image to square (1:1 ratio)
  Future<File?> cropToSquare(XFile image) async {
    // Sử dụng image_cropper package
    final croppedFile = await ImageCropper().cropImage(
      sourcePath: image.path,
      aspectRatio: const CropAspectRatio(ratioX: 1, ratioY: 1),
      uiSettings: [
        AndroidUiSettings(
          toolbarTitle: 'Crop Avatar',
          toolbarColor: Colors.deepOrange,
          initAspectRatio: CropAspectRatioPreset.square,
          lockAspectRatio: false,
        ),
        IOSUiSettings(
          title: 'Crop Avatar',
          aspectRatioLockEnabled: false,
        ),
      ],
    );
    return croppedFile != null ? File(croppedFile.path) : null;
  }
  
  /// Compress image for upload
  Future<File> compressForUpload(File file) async {
    final result = await FlutterImageCompress.compressAndGetFile(
      file.absolute.path,
      '${file.path}_compressed.jpg',
      quality: 70,      // 70% quality
      minWidth: 800,
      minHeight: 800,
    );
    return result ?? file;
  }
  
  /// Full upload pipeline: pick → crop → compress → upload
  Future<String?> uploadAvatarFull() async {
    final image = await CameraService().pickFromGallery();
    if (image == null) return null;
    
    final cropped = await cropToSquare(image);
    if (cropped == null) return null;
    
    final compressed = await compressForUpload(cropped);
    
    return await ImageUploadService().upload(compressed);
  }
}
```

### Platform Configuration

**iOS (Info.plist):**

```xml
<key>NSCameraUsageDescription</key>
<string>App cần camera để chụp ảnh hồ sơ</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>App cần truy cập ảnh để chọn avatar</string>
<key>NSMicrophoneUsageDescription</key>
<string>App cần microphone để quay video</string>
```

**Android (AndroidManifest.xml):**

```xml
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

---

## 2. Location Services — geolocator

> 💡 **FE Perspective**
> **Flutter:** `geolocator` wraps platform-specific location APIs (CoreLocation, FusedLocationProviderClient).
> **React/Vue tương đương:** `navigator.geolocation.getCurrentPosition()`.
> **Khác biệt quan trọng:** Flutter có thêm background location, geofencing, heading.

### Basic Usage

```dart
import 'package:geolocator/geolocator.dart';

class LocationService {
  /// Check if location services enabled
  Future<bool> isLocationServiceEnabled() async {
    return await Geolocator.isLocationServiceEnabled();
  }
  
  /// Check permission status
  Future<LocationPermission> checkPermission() async {
    return await Geolocator.checkPermission();
  }
  
  /// Request permission
  Future<LocationPermission> requestPermission() async {
    return await Geolocator.requestPermission();
  }
  
  /// Get current position
  Future<Position?> getCurrentPosition() async {
    // Check service
    if (!await isLocationServiceEnabled()) {
      return null;
    }
    
    // Check permission
    var permission = await checkPermission();
    if (permission == LocationPermission.denied) {
      permission = await requestPermission();
      if (permission == LocationPermission.denied) {
        return null;
      }
    }
    
    if (permission == LocationPermission.deniedForever) {
      // User permanently denied → open settings
      await Geolocator.openAppSettings();
      return null;
    }
    
    // Get position
    return await Geolocator.getCurrentPosition(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        timeLimit: Duration(seconds: 10),
      ),
    );
  }
}
```

### Permission Levels

```dart
// Location permission states
enum LocationPermission {
  denied,           // User denied permission
  deniedForever,    // User permanently denied, must open settings
  whileInUse,       // Only when app in foreground
  always,           // Background location access
}
```

### Continuous Location Updates

```dart
class LocationTracker {
  StreamSubscription<Position>? _positionSubscription;
  
  /// Start tracking location
  void startTracking({
    required void Function(Position) onPositionUpdate,
    required void Function(LocationPermission) onPermissionChange,
  }) {
    _positionSubscription = Geolocator.getPositionStream(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 10,  // Update every 10 meters
        timeLimit: Duration(minutes: 5),
      ),
    ).listen(
      (position) => onPositionUpdate(position),
    );
  }
  
  /// Handle permission changes separately via Geolocator.checkPermission() or permission stream
  
  /// Stop tracking
  void stopTracking() {
    _positionSubscription?.cancel();
    _positionSubscription = null;
  }
  
  /// Calculate distance between two positions
  double calculateDistance(
    double startLat, double startLng,
    double endLat, double endLng,
  ) {
    return Geolocator.distanceBetween(
      startLat, startLng,
      endLat, endLng,
    );
  }
}
```

### Background Location

```dart
// Android: Background location service
class BackgroundLocationService {
  Future<void> requestBackgroundPermission() async {
    final permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.always) {
      return;
    }
    
    // Request always permission
    final newPermission = await Geolocator.requestPermission();
    if (newPermission != LocationPermission.always) {
      throw Exception('Background location permission denied');
    }
  }
  
  Stream<Position> get backgroundStream {
    return Geolocator.getPositionStream(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 50,
        // Android: enable background mode
        androidSettings: AndroidSettings(
          intervalDuration: const Duration(minutes: 1),
          distanceFilter: 50,
          foregroundNotificationConfig: const ForegroundNotificationConfig(
            notificationTitle: 'Location Tracking',
            notificationBody: 'App is tracking your location',
            notificationIcon: 'ic_location',
          ),
        ),
      ),
    );
  }
}
```

### Geofencing

> ⚠️ **Important correction:** `Geolocator.getGeofenceEvents()` **does NOT exist** in the `geolocator` package (v10.x). Geofencing in Flutter requires custom platform channel implementation or third-party packages.

**Teaching pattern (how geofencing WOULD work):**

```dart
// Teaching pattern — conceptual geofencing API
// This is NOT a real API. The base_flutter project does NOT use geofencing.

class GeofenceService {
  // Conceptual geofence setup
  static const _geofenceRegion = CircularRegion(
    id: 'office',
    latitude: 35.6762,
    longitude: 139.6503,
    radius: 100, // meters
  );

  // NOTE: This method does NOT exist in geolocator package
  // Real geofencing requires:
  // 1. Custom MethodChannel to iOS CoreLocation / Android FusedLocationClient
  // 2. Or use packages like: geofence_service, location_background, flutter_geofence
  Stream<GeofenceEvent> get geofenceEvents {
    // This is a teaching concept, not actual code
    throw UnimplementedError(
      'Geofencing requires custom platform channel implementation. '
      'The geolocator package does not support geofencing out of the box.'
    );
  }

  void onEnterOffice() {
    LocalPushNotificationHelper.show(
      title: 'Check-in',
      body: 'Bạn đã đến văn phòng',
    );
  }
}
```

**For real geofencing, use platform channels:**

```dart
// Real implementation would use MethodChannel:
class GeofenceService {
  static const _channel = MethodChannel('jp.flutter.app/geofence');

  Future<void> startGeofencing(List<CircularRegion> regions) async {
    await _channel.invokeMethod('startGeofencing', {
      'regions': regions.map((r) => {
        'id': r.id,
        'latitude': r.latitude,
        'longitude': r.longitude,
        'radius': r.radius,
      }).toList(),
    });
  }

  Stream<GeofenceEvent> get geofenceEventStream {
    return _channel.receiveBroadcastStream().map((event) {
      return GeofenceEvent.fromMap(event as Map);
    });
  }
}
```

**Note:** The `base_flutter` project does **NOT** implement geofencing. The `geolocator` package (version 10.1.0) is commented out in `pubspec.yaml`.
```

---

## 3. Biometric Authentication — local_auth

> 💡 **FE Perspective**
> **Flutter:** `local_auth` wraps Face ID, Touch ID (iOS) và Fingerprint, Face Unlock (Android).
> **React/Vue tương đương:** WebAuthn / Web Authentication API.
> **Khác biệt quan trọng:** Flutter biometric uses device hardware (Secure Enclave / StrongBox).

### Basic Usage

```dart
import 'package:local_auth/local_auth.dart';

class BiometricAuthService {
  final LocalAuthentication _auth = LocalAuthentication();
  
  /// Check if device supports biometrics
  Future<bool> isDeviceSupported() async {
    return await _auth.isDeviceSupported();
  }
  
  /// Check available biometric types
  Future<List<BiometricType>> getAvailableBiometrics() async {
    return await _auth.getAvailableBiometrics();
  }
  
  /// Check if can check biometrics
  Future<bool> canCheckBiometrics() async {
    return await _auth.canCheckBiometrics;
  }
  
  /// Authenticate with biometrics
  Future<bool> authenticate({
    required String reason,
    bool biometricOnly = false,
  }) async {
    final canAuth = await isDeviceSupported();
    if (!canAuth) return false;
    
    try {
      return await _auth.authenticate(
        localizedReason: reason,
        options: AuthenticationOptions(
          stickyAuth: true,           // Keep auth until success/cancel
          biometricOnly: biometricOnly, // Only biometric, no fallback to PIN
        ),
      );
    } on PlatformException catch (e) {
      // Handle specific errors
      switch (e.code) {
        case 'NotAvailable':
          return false;
        case 'NotEnrolled':
          // No biometrics enrolled
          return false;
        case 'LockedOut':
          // Too many failed attempts
          return false;
        case 'PermanentlyLockedOut':
          // Must reset device
          return false;
        default:
          return false;
      }
    }
  }
  
  /// Check biometric availability
  Future<BiometricStatus> checkBiometricStatus() async {
    final isSupported = await isDeviceSupported();
    if (!isSupported) {
      return BiometricStatus.notSupported;
    }
    
    final biometrics = await getAvailableBiometrics();
    if (biometrics.isEmpty) {
      return BiometricStatus.notEnrolled;
    }
    
    return BiometricStatus.available(biometrics);
  }
}

enum BiometricStatus {
  notSupported,
  notEnrolled,
  available(List<BiometricType> types),
}
```

### Usage Example

```dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      body: Column(
        children: [
          // Biometric login button
          BiometricButton(
            onPressed: () async {
              final auth = BiometricAuthService();
              final status = await auth.checkBiometricStatus();
              
              if (status is BiometricStatus.available) {
                final success = await auth.authenticate(
                  reason: 'Xác thực để đăng nhập',
                );
                if (success) {
                  ref.read(authProvider.notifier).loginWithBiometric();
                }
              } else {
                // Fallback to password
                _showPasswordLogin();
              }
            },
          ),
        ],
      ),
    );
  }
}

class BiometricButton extends StatelessWidget {
  // ... implementation
}
```

### Platform Configuration

**iOS (Info.plist):**

```xml
<key>NSFaceIDUsageDescription</key>
<string>App sử dụng Face ID để đăng nhập nhanh hơn</string>
```

**Android (AndroidManifest.xml):**

```xml
<uses-permission android:name="android.permission.USE_BIOMETRIC"/>
<uses-permission android:name="android.permission.USE_FINGERPRINT"/>
```

**Android MainActivity.kt:**

```kotlin
// MainActivity.kt
class MainActivity: FlutterActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        GeneratedPluginRegistrant.registerWith(flutterEngine)
    }
}
```

---

## 4. Push Notifications — firebase_messaging

> 💡 **FE Perspective**
> **Flutter:** `firebase_messaging` wraps FCM SDK cho push notifications.
> **React/Vue tương đương:** Web Push API (`PushManager.subscribe`).
> **Khác biệt quan trọng:** Flutter có 3 states: foreground, background, terminated.

### Setup

```dart
// lib/data_source/firebase/messaging/firebase_messaging_service.dart
class FirebaseMessagingService {
  final FirebaseMessaging _messaging = FirebaseMessaging.instance;
  
  /// Initialize messaging
  Future<void> initialize() async {
    // Request permission (iOS)
    final settings = await _messaging.requestPermission(
      alert: true,
      announcement: false,
      badge: true,
      carPlay: false,
      criticalAlert: false,
      provisional: false,
      sound: true,
    );
    
    // Get FCM token
    final token = await _messaging.getToken();
    Log.i('FCM Token: $token');
    
    // Listen for token refresh
    _messaging.onTokenRefresh.listen((token) {
      _sendTokenToServer(token);
    });
    
    // Handle foreground messages
    FirebaseMessaging.onMessage.listen(_handleForegroundMessage);
    
    // Handle background/terminated message tap
    FirebaseMessaging.onMessageOpenedApp.listen(_handleMessageOpenedApp);
    
    // Check for initial message (terminated state)
    final initialMessage = await _messaging.getInitialMessage();
    if (initialMessage != null) {
      _handleInitialMessage(initialMessage);
    }
  }
  
  void _handleForegroundMessage(RemoteMessage message) {
    // App đang mở → show in-app notification
    LocalPushNotificationHelper.show(
      title: message.notification?.title ?? 'Notification',
      body: message.notification?.body ?? '',
      payload: jsonEncode(message.data),
    );
  }
  
  void _handleMessageOpenedApp(RemoteMessage message) {
    // User tap notification → navigate
    _navigateToScreen(message.data);
  }
  
  void _handleInitialMessage(RemoteMessage message) {
    // App launch from notification → navigate
    _navigateToScreen(message.data);
  }
  
  void _navigateToScreen(Map<String, dynamic> data) {
    final screen = data['screen'] as String?;
    final id = data['id'] as String?;
    
    // Use AppNavigator to navigate
    switch (screen) {
      case 'conversation':
        AppNavigator.push(ConversationRoute(id: id));
        break;
      case 'order':
        AppNavigator.push(OrderDetailRoute(id: id));
        break;
    }
  }
}
```

### Local Notifications

```dart
// flutter_local_notifications setup
class LocalPushNotificationHelper {
  static final FlutterLocalNotificationsPlugin _plugin = 
      FlutterLocalNotificationsPlugin();
  
  static Future<void> initialize() async {
    const androidSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings(
      requestAlertPermission: false,  // Use FCM permission
      requestBadgePermission: false,
      requestSoundPermission: false,
    );
    
    await _plugin.initialize(
      const InitializationSettings(
        android: androidSettings,
        iOS: iosSettings,
      ),
      onDidReceiveNotificationResponse: _onNotificationTapped,
    );
    
    // Create Android notification channel
    await _createNotificationChannel();
  }
  
  static Future<void> _createNotificationChannel() async {
    const channel = AndroidNotificationChannel(
      'default_channel',
      'Default Notifications',
      description: 'Thông báo mặc định',
      importance: Importance.high,
    );
    
    await _plugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(channel);
  }
  
  static Future<void> show({
    required String title,
    required String body,
    String? payload,
    String? imageUrl,
  }) async {
    final androidDetails = AndroidNotificationDetails(
      'default_channel',
      'Default Notifications',
      channelDescription: 'Thông báo mặc định',
      importance: Importance.high,
      priority: Priority.high,
      styleInformation: imageUrl != null
          ? BigPictureStyleInformation(
              await _downloadImage(imageUrl),
              largeIcon: const DrawableResourceAndroidBitmap('@mipmap/ic_launcher'),
            )
          : null,
    );
    
    const iosDetails = DarwinNotificationDetails(
      presentAlert: true,
      presentBadge: true,
      presentSound: true,
    );
    
    await _plugin.show(
      DateTime.now().millisecondsSinceEpoch ~/ 1000,
      title,
      body,
      NotificationDetails(
        android: androidDetails,
        iOS: iosDetails,
      ),
      payload: payload,
    );
  }
  
  static void _onNotificationTapped(NotificationResponse response) {
    // Parse payload và navigate
    final payload = response.payload;
    if (payload != null) {
      final data = jsonDecode(payload) as Map<String, dynamic>;
      _navigateFromPayload(data);
    }
  }
}
```

### Notification Actions (Android)

```dart
// Notification với actions
final androidDetails = AndroidNotificationDetails(
  'messages_channel',
  'Messages',
  channelDescription: 'Tin nhắn',
  importance: Importance.high,
  priority: Priority.high,
  actions: [
    const AndroidNotificationAction(
      'reply_action',
      'Reply',
      showsUserInterface: true,
      title: 'Trả lời',
    ),
    const AndroidNotificationAction(
      'mark_read_action',
      'Mark as Read',
    ),
  ],
);
```

---

## 5. Deep Linking — app_links

> 💡 **FE Perspective**
> **Flutter:** `app_links` package handles App Links (Android) và Universal Links (iOS).
> **React/Vue tương đương:** URL routing với `react-router` hoặc `Linking` API (React Native).
> **Khác biệt quan trọng:** Flutter cần configure platform (AndroidManifest, entitlements) và handle URI parsing.

### URL Scheme vs App Links

| Type | Format | Example | Platform |
|------|--------|---------|----------|
| **Custom Scheme** | `myapp://` | `myapp://profile/123` | Both |
| **Universal Links** | `https://` | `https://myapp.com/profile/123` | iOS |
| **App Links** | `https://` | `https://myapp.com/profile/123` | Android |

### Configuration

**Android (AndroidManifest.xml):**

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <!-- App Links -->
    <data android:scheme="https" android:host="myapp.com"/>
</intent-filter>

<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <!-- Custom Scheme -->
    <data android:scheme="myapp"/>
</intent-filter>
```

**iOS (Runner.entitlements):**

```xml
<key>com.apple.developer.associated-domains</key>
<array>
    <string>applinks:myapp.com</string>
    <string>webcredentials:myapp.com</string>
</array>
```

### Deep Link Handler

```dart
class DeepLinkHelper {
  final AppLinks _appLinks = AppLinks();
  StreamSubscription<Uri>? _subscription;
  
  void listenToDeepLinks() {
    _subscription = _appLinks.uriLinkStream.listen((uri) {
      _handleDeepLink(uri);
    });
  }
  
  void _handleDeepLink(Uri uri) {
    // Parse URI và navigate
    // myapp://profile/123 → navigate to profile 123
    // https://myapp.com/profile/123 → same
    
    final path = uri.pathSegments;
    final queryParams = uri.queryParameters;
    
    switch (path.first) {
      case 'profile':
        if (path.length > 1) {
          final userId = path[1];
          AppNavigator.push(ProfileRoute(userId: userId));
        }
        break;
      case 'order':
        final orderId = queryParams['id'];
        if (orderId != null) {
          AppNavigator.push(OrderDetailRoute(orderId: orderId));
        }
        break;
      case 'invite':
        final code = queryParams['code'];
        if (code != null) {
          AppNavigator.push(InviteRoute(code: code));
        }
        break;
    }
  }
  
  void dispose() {
    _subscription?.cancel();
  }
}
```

### Deferred Deep Linking

```dart
// Handle deferred deep link (app not installed, install from link)
class DeferredDeepLinkHandler {
  Future<void> checkInitialLink() async {
    // Get initial link (may be null if app was launched without deep link)
    final initialUri = await _appLinks.getInitialUri();
    
    if (initialUri != null) {
      // Store link for later use
      await _savePendingDeepLink(initialUri);
      _handleDeepLink(initialUri);
    }
  }
  
  Future<void> _savePendingDeepLink(Uri uri) async {
    // Save to secure storage
    await FlutterSecureStorage().write(
      key: 'pending_deep_link',
      value: uri.toString(),
    );
  }
  
  Future<Uri?> getPendingDeepLink() async {
    final link = await FlutterSecureStorage().read(key: 'pending_deep_link');
    if (link != null) {
      await FlutterSecureStorage().delete(key: 'pending_deep_link');
      return Uri.parse(link);
    }
    return null;
  }
}
```

---

## Summary — Native Features Integration

| Feature | Package | Key APIs | Use Case |
|---------|---------|----------|----------|
| Camera | `image_picker` | `pickImage()`, `pickVideo()` | Photo capture, gallery |
| Image Crop | `image_cropper` | `cropImage()` | Avatar editing |
| Location | `geolocator` | `getCurrentPosition()`, `getPositionStream()` | GPS tracking |
| Biometrics | `local_auth` | `authenticate()` | Secure login |
| Push | `firebase_messaging` | `getToken()`, `onMessage` | Notifications |
| Local Notif | `flutter_local_notifications` | `show()` | In-app alerts |
| Deep Link | `app_links` | `uriLinkStream` | URL routing |

> 💡 **FE Perspective Summary:**
> | Flutter | Frontend |
> |---------|----------|
> | `image_picker` | `<input type="file">` |
> | `geolocator` | `navigator.geolocation` |
> | `local_auth` | WebAuthn API |
> | `firebase_messaging` | Web Push API |
> | `app_links` | URL routing |

<!-- AI_VERIFY: generation-complete -->
