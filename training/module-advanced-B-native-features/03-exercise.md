# Exercise — Advanced Native Features

> 📌 **Prerequisites:**
> - Hoàn thành [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md)
> - [M21 — Firebase](./../module-21-firebase/) — Firebase setup required for Push Notifications (Exercise 4)
> - Có `base_flutter` project đã setup
> - Test trên device/simulator (native features không work trong web)

---

## Exercise Overview

| # | Exercise | Focus | Difficulty | Thời gian |
|---|----------|-------|------------|-----------|
| 1 | Camera & Image Upload | Avatar capture, crop, compress, upload | 🟡 Medium | ~45 min |
| 2 | Location Services | Get location, calculate distance | 🟡 Medium | ~30 min |
| 3 | Biometric Authentication | Face ID/Touch ID login | 🟡 Medium | ~30 min |
| 4 | Push Notifications | FCM setup, foreground/background handling | 🔴 Hard | ~60 min |
| 5 | Deep Linking | Custom scheme + App Links | 🔴 Hard | ~45 min |

**Tổng thời gian:** ~3-4 hours

---

## Exercise 1: Camera & Image Upload 📸

**Mục tiêu:** Implement complete avatar upload pipeline.

### Setup

```bash
# Thêm dependencies vào pubspec.yaml
flutter pub add image_picker image_cropper flutter_image_compress
flutter pub add permission_handler  # For permission handling
```

### Task 1.1: Create Camera Service

Tạo `lib/common/service/camera_service.dart`:

```dart
import 'package:image_picker/image_picker.dart';

class CameraService {
  final ImagePicker _picker = ImagePicker();
  
  // TODO: Implement pickFromGallery()
  // Requirements:
  // - Max width: 1024px
  // - Quality: 85%
  // - Return XFile or null
  
  // TODO: Implement takePhoto()
  // Requirements:
  // - Use rear camera
  // - Max: 1920x1920
  // - Quality: 90%
  // - Return XFile or null
  
  // TODO: Implement pickImageWithPermissionCheck()
  // Requirements:
  // - Check camera permission
  // - Request if not granted
  // - Show dialog if permanently denied
}
```

### Task 1.2: Create Image Processing Service

Tạo `lib/common/service/image_processing_service.dart`:

```dart
// TODO: Implement cropToSquare()
// Use image_cropper package
// Aspect ratio: 1:1

// TODO: Implement compressForUpload()
// Requirements:
// - Quality: 70%
// - Max dimensions: 800x800
// - Format: JPEG

// TODO: Implement uploadAvatar()
// Call API endpoint POST /api/v1/users/me/avatar
// Content-Type: multipart/form-data
```

### Task 1.3: Integrate với Profile Page

Update avatar widget để support tap-to-change:

```dart
class AvatarWidget extends StatelessWidget {
  // TODO: Add GestureDetector → show bottom sheet
  // Bottom sheet options:
  // 1. "Chụp ảnh mới" → takePhoto()
  // 2. "Chọn từ thư viện" → pickFromGallery()
  // 3. "Hủy"
}
```

### Verification

```bash
# Test on device/simulator:
flutter run

# Verify:
# 1. Tapping avatar shows bottom sheet
# 2. Camera capture works
# 3. Gallery picker works
# 4. Image crops to square
# 5. Upload succeeds (check API logs)
```

---

## Exercise 2: Location Services 📍

**Mục tiêu:** Add location-based feature (distance to store).

### Setup

```bash
flutter pub add geolocator
```

### Task 2.1: Create Location Service

Tạo `lib/common/service/location_service.dart`:

```dart
// TODO: Implement isLocationEnabled()
// Check if location services are enabled

// TODO: Implement checkAndRequestPermission()
// Handle all permission states:
// - denied → request
// - deniedForever → open settings

// TODO: Implement getCurrentPosition()
// Return Position or null with error handling

// TODO: Implement getDistanceTo()
// Calculate distance from current location to target
```

### Task 2.2: Add Store Distance Feature

Tạo `lib/ui/widget/store_distance_widget.dart`:

```dart
class StoreDistanceWidget extends ConsumerStatefulWidget {
  final double storeLatitude;
  final double storeLongitude;
  final String storeName;
  
  @override
  ConsumerState<StoreDistanceWidget> createState() => _StoreDistanceWidgetState();
}

class _StoreDistanceWidgetState extends ConsumerState<StoreDistanceWidget> {
  // TODO: Get current location
  // TODO: Calculate distance
  // TODO: Display distance (e.g., "1.2 km away")
  // TODO: Handle permission denied state
}
```

### Verification

```bash
flutter run

# Test:
# 1. Show current distance to store
# 2. Handle location permission denied
# 3. Show loading state while getting location
# 4. Update when location changes
```

---

## Exercise 3: Biometric Authentication 🔐

**Mục tiêu:** Add Face ID / Touch ID login option.

### Setup

```bash
flutter pub add local_auth
```

### Task 3.1: Create Biometric Auth Service

Tạo `lib/common/service/biometric_auth_service.dart`:

```dart
// TODO: Implement checkBiometricStatus()
// Return: notSupported, notEnrolled, available(biometricTypes)

// TODO: Implement authenticate()
// Options:
// - reason: localized reason string
// - biometricOnly: only biometric, no fallback

// TODO: Implement isBiometricAvailable()
// Quick check if biometric can be used
```

### Task 3.2: Add Biometric Login Button

Update login page:

```dart
class LoginPage extends ConsumerWidget {
  // TODO: Add biometric login button
  // Show only if biometric is available
  // Icon: fingerprint (Android) or face (iOS)
  // On tap: authenticate() → if success, auto-fill and submit
}
```

### Task 3.3: Platform Configuration

**Android (AndroidManifest.xml):**

```xml
<uses-permission android:name="android.permission.USE_BIOMETRIC"/>
<uses-permission android:name="android.permission.USE_FINGERPRINT"/>
```

**iOS (Info.plist):**

```xml
<key>NSFaceIDUsageDescription</key>
<string>Đăng nhập nhanh bằng Face ID</string>
```

### Verification

```bash
flutter run

# Test:
# 1. Biometric button shows (or not if unavailable)
# 2. Tap triggers biometric prompt
# 3. Success → auto-login
# 4. Failure → show error
```

---

## Exercise 4: Push Notifications 🔔

**Mục tiêu:** Setup Firebase Cloud Messaging.

### Setup

```bash
# Thêm Firebase dependencies
flutter pub add firebase_core firebase_messaging

# Hoặc nếu không dùng Firebase:
flutter pub add flutter_local_notifications
```

### Task 4.1: Create Notification Service

Tạo `lib/common/service/notification_service.dart`:

```dart
// TODO: Implement initializeFirebaseMessaging()
// Requirements:
// - Request permission (iOS)
// - Get FCM token
// - Listen for token refresh
// - Handle foreground messages
// - Handle background/terminated message tap

// TODO: Implement handleForegroundMessage()
// Show local notification with message data

// TODO: Implement handleMessageOpenedApp()
// Navigate to appropriate screen based on message data
```

### Task 4.2: Handle Background Handler

Tạo `lib/firebase_messaging_handler.dart` (top-level function):

```dart
// ⚠️ PHẢI là top-level function
@pragma('vm:entry-point')
Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // Handle background message
}
```

Update `main.dart`:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  
  // Register background handler
  FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);
  
  runApp(const MyApp());
}
```

### Task 4.3: Local Notifications Setup

```dart
// lib/common/service/local_notification_service.dart

// TODO: Implement initialize()
// Setup FlutterLocalNotificationsPlugin

// TODO: Implement showNotification()
// Show notification with title, body, payload

// TODO: Implement createNotificationChannel() (Android)
// Create channel với high importance
```

### Verification

```bash
# Test:
# 1. Send test notification from Firebase Console
# 2. App in foreground → local notification shown
# 3. App in background → tap → navigate to correct screen
# 4. App terminated → launch from notification → navigate
```

---

## Exercise 5: Deep Linking 🔗

**Mục tiêu:** Configure deep links cho app.

### Setup

```bash
flutter pub add app_links
```

### Task 5.1: Configure Android

Update `android/app/src/main/AndroidManifest.xml`:

```xml
<!-- Custom scheme -->
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="myapp"/>
</intent-filter>

<!-- App Links (HTTPS) -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="https" android:host="myapp.com"/>
</intent-filter>
```

### Task 5.2: Configure iOS

Update `ios/Runner/Runner.entitlements`:

```xml
<key>com.apple.developer.associated-domains</key>
<array>
    <string>applinks:myapp.com</string>
</array>
```

### Task 5.3: Create Deep Link Handler

Tạo `lib/common/helper/deep_link_helper.dart`:

```dart
// TODO: Implement listenToDeepLinks()
// Start listening to app_links.uriLinkStream

// TODO: Implement handleDeepLink()
// Parse URI path → determine screen → navigate

// Supported routes:
// - /profile/:id
// - /order/:id
// - /invite?code=xxx
```

Update `main.dart`:

```dart
void main() {
  // Setup deep link listener
  DeepLinkHelper().listenToDeepLinks();
}
```

### Task 5.4: Update AppNavigator

Add deep link routes:

```dart
// Trong AppNavigator
class AppRoutes {
  // Deep link routes
  static const profileDeepLink = '/profile/:userId';
  static const orderDeepLink = '/order/:orderId';
  static const inviteDeepLink = '/invite';
}
```

### Verification

```bash
# Test on device:
# 1. Open URL: myapp://profile/123 → app opens → navigate to profile 123
# 2. Open URL: https://myapp.com/profile/456 → app opens → navigate to profile 456
# 3. Handle invalid routes gracefully
```

---

## Bonus Challenges ⭐

### Bonus 1: Geofencing (Teaching Exercise)

> ⚠️ **Note:** Geofencing is a **teaching exercise** since `GeofenceService` doesn't exist in base_flutter. You'll create a teaching pattern to understand the concept.

Implement geofence notification:

```dart
// Teaching pattern: Khi user enters/exits a region → trigger notification
// This is a conceptual implementation for learning purposes

// Step 1: Define geofence regions (could be stored in API)
class GeofenceRegion {
  final String id;
  final double latitude;
  final double longitude;
  final double radiusMeters;
}

// Step 2: Create teaching service
class GeofenceService {
  // Simulated geofence events (for learning)
  // Real implementation would use native platform channels

  Stream<GeofenceEvent> get geofenceEventStream {
    // Teaching: This would connect to native geofencing APIs
    // iOS: CLLocationManager startMonitoringForRegion
    // Android: GeofencingApi
    throw UnimplementedError(
      'Real geofencing requires native platform channels. '
      'This is a teaching pattern only.'
    );
  }
}

// Step 3: Example usage (teaching concept)
Future<void> setupGeofencing() async {
  final regions = [
    GeofenceRegion(
      id: 'office',
      latitude: 35.6762,
      longitude: 139.6503,
      radiusMeters: 100,
    ),
  ];

  // Teaching: How to handle geofence events
  // Real implementation would listen to geofenceEventStream
  // and trigger notifications when entering/leaving regions
}
```

**For actual implementation, you'd need:**
1. Native iOS code: `CLLocationManager` + `startMonitoring(for:)`
2. Native Android code: `GeofencingApi.addGeofences()`
3. Platform channel to communicate between Flutter and native

**Packages to explore for real implementation:**
- `flutter_geofence`
- `geofence_service`
- Custom platform channel

### Bonus 2: Notification Actions

Add reply action to notifications:

```dart
// Add action buttons to notification
// - Reply → open reply dialog
// - Mark as read → mark notification as read
```

### Bonus 3: Multiple Image Upload

Extend camera service for multiple image selection:

```dart
// Pick multiple images from gallery
// Compress and upload in batch
// Show progress indicator
```

---

## Submission

1. **Tạo PR** với title: `feat(native): Camera, Location, Biometric, Push, Deep Link`
2. **Kiểm tra:**
   - [ ] Camera capture → crop → compress → upload pipeline works
   - [ ] Location permission → get position → calculate distance
   - [ ] Biometric authentication on supported devices
   - [ ] Push notifications handle all 3 app states
   - [ ] Deep links navigate to correct screens
3. **Demo:** Show each native feature working on device

---

## Hints

### Hint 1: Camera Permission

```
image_picker tự handle permission, nhưng check trước để hiển thị explanation.
permission_handler package giúp check/request permissions.
```

### Hint 2: Biometric Fallback

```
Luôn có fallback khi biometric fails.
Nếu biometricOnly=true và fail → user phải try lại.
Nếu biometricOnly=false → có thể fallback to PIN/password.
```

### Hint 3: Push Notification Testing

```
Để test push notifications:
1. Development: dùng Firebase Console để send test message
2. iOS Simulator: Push không hoạt động, cần real device
3. Background: app phải được terminate (không just backgrounded)
```

### Hint 4: Deep Link Testing

```
Test deep links:
1. Terminal: xcrun simctl openurl booted "myapp://profile/123"
2. Browser: nhập myapp://profile/123
3. Real device: tạo test webpage với link
```

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
