# Module 21 – Concepts: Firebase Services

> **Mục tiêu**: Nắm vững 7 concepts cốt lõi về Firebase — Core, Analytics, Crashlytics, Auth, Remote Config, Cloud Messaging, Storage.

📌 **Recap**: M1 (WidgetsFlutterBinding) · M3 (Config/env) · M20 (Platform Channels)

---

## Concept 1: Firebase Core — Initialization & Configuration

<!-- AI_VERIFY: concept-firebase-core -->

### 1.1 Initialization Flow

```dart
// ⚠️ Lưu ý: FirebaseRemoteConfigService KHÔNG tồn tại trong project này
Future<void> main() async {
  // 1. Ensure Flutter binding
  WidgetsFlutterBinding.ensureInitialized();

  // 2. Initialize Firebase (bắt buộc)
  await Firebase.initializeApp();

  // 3. Initialize App (services được inject qua DI)
  await AppInitializer.init();

  // 4. Run app
  runApp(ProviderScope(
    observers: [AppProviderObserver()],
    child: MyApp(),
  ));
}
```

> ⚠️ **Sai:** `await FirebaseRemoteConfigService().initialize()` — class này KHÔNG tồn tại
> ✅ **Đúng:** Firebase được khởi tạo qua `Firebase.initializeApp()`. Các service như Analytics, Crashlytics, Messaging được quản lý qua DI (`@LazySingleton`) và accessed qua Riverpod providers.

### 1.2 Actual Firebase Services in Project

|| Service | Class | Pattern | Provider |
|---------|-------|--------|----------|
| Analytics | `AnalyticsHelper` | Riverpod Provider | `analyticsHelperProvider` |
| Crashlytics | `CrashlyticsHelper` | `@LazySingleton` + Riverpod | `crashlyticsHelperProvider` |
| Messaging | `FirebaseMessagingService` | `@LazySingleton` + Riverpod | `firebaseMessagingServiceProvider` |
| Auth | SDK directly | Firebase Auth SDK | `FirebaseAuth.instance` |
| Firestore | `FirebaseFirestoreService` | `@LazySingleton` | (custom service) |

```dart
// Analytics - sử dụng qua Riverpod Provider
// analytics_helper.dart
final analyticsHelperProvider = Provider<AnalyticsHelper>(
  (ref) => AnalyticsHelper(ref: ref),
);

// Crashlytics - @LazySingleton injectable
@LazySingleton()
class CrashlyticsHelper { ... }

// Messaging - @LazySingleton injectable
@LazySingleton()
class FirebaseMessagingService { ... }
```

<!-- AI_VERIFY: based-on main.dart, analytics_helper.dart, crashlytics_helper.dart, firebase_messaging_service.dart -->

### 1.2 Platform Configuration

| Platform | File | Location |
|----------|------|----------|
| iOS | `GoogleService-Info.plist` | `ios/Runner/` |
| Android | `google-services.json` | `android/app/` |

### 1.3 Multiple Firebase Projects

```dart
// Initialize with options for specific platform
await Firebase.initializeApp(
  name: 'my-flutter-app-prod',
  options: FirebaseOptions(
    apiKey: 'AIzaSy...',
    appId: '1:123:ios:abc',
    messagingSenderId: '123456789',
    projectId: 'my-flutter-app',
  ),
);
```

> 💡 **FE Perspective**
> **Flutter:** `Firebase.initializeApp()` = `initializeApp()` in Firebase JS SDK.
> **React/Vue tương đương:** Firebase JS SDK initialization in `index.js` or `main.tsx`.

---

## Concept 2: Firebase Analytics

<!-- AI_VERIFY: concept-firebase-analytics -->

> ⚠️ **Trong project này:** Analytics được quản lý qua `AnalyticsHelper` class với Riverpod Provider pattern, không phải singleton trực tiếp.

### 2.1 Event Tracking

```dart
// ⚠️ Sử dụng AnalyticsHelper từ Riverpod Provider
// Không phải FirebaseAnalytics.instance trực tiếp
final analyticsHelper = ref.read(analyticsHelperProvider);

// Custom event
await analyticsHelper.logEvent(NormalEvent.purchaseComplete);

class NormalEvent {
  final String fullEventName;
  final EventParameter? parameter;

  NormalEvent.purchaseComplete() : this(
    fullEventName: 'purchase_complete',
    parameter: EventParameter(parameters: {
      'value': 99.99,
      'currency': 'USD',
    }),
  );
}
```

### 2.2 Screen Tracking

```dart
// Track screen view
final analyticsHelper = ref.read(analyticsHelperProvider);
await analyticsHelper.logScreenView(ScreenViewEvent.productDetail);
```

### 2.3 User Properties

```dart
// Set user properties
final analyticsHelper = ref.read(analyticsHelperProvider);
await analyticsHelper.setUserProperties({
  'user_type': 'premium',
  'signup_date': '2024-01-01',
});

// Set user ID
await analyticsHelper.setUserId('user_123');
```

### 2.4 Analytics Helper Implementation

```dart
// analytics_helper.dart - Actual implementation pattern
class AnalyticsHelper {
  AnalyticsHelper({
    FirebaseAnalytics? firebaseAnalytics,
    required this.ref,
  }) : _firebaseAnalytics = firebaseAnalytics ?? FirebaseAnalytics.instance;

  final FirebaseAnalytics _firebaseAnalytics;
  final Ref ref;

  Future<Map<String, Object>> get _commonParameters async => {
        ParameterConstants.userId: await ref.read(deviceHelperProvider).deviceId,
      };

  Future<void> logEvent(NormalEvent event) async {
    final parameters = {
      ...await _commonParameters,
      ...event.parameter?.parameters ?? {},
    };
    // ... logging logic
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** `AnalyticsHelper` = wrapper around Firebase Analytics SDK với common parameters.
> **React/Vue tương đương:** Custom analytics service wrapping Firebase SDK.

---

## Concept 3: Firebase Crashlytics

<!-- AI_VERIFY: concept-firebase-crashlytics -->

> ⚠️ **Trong project này:** Crashlytics được quản lý qua `CrashlyticsHelper` class với `@LazySingleton` injectable pattern, accessed qua Riverpod provider.

### 3.1 Exception Recording

```dart
// ⚠️ Sử dụng CrashlyticsHelper từ Riverpod Provider
// main.dart sử dụng runZonedGuarded để bắt uncaught errors
Future<void> main() async => runZonedGuarded(
  _runMyApp,
  (error, stackTrace) => _reportError(error: error, stackTrace: stackTrace),
);

void _reportError({required error, required StackTrace stackTrace}) {
  Log.e(error, stackTrace: stackTrace, name: 'Uncaught exception');
  FirebaseCrashlytics.instance.recordError(error, stackTrace);
}
```

### 3.2 Crashlytics Helper

```dart
// crashlytics_helper.dart - Actual implementation
final crashlyticsHelperProvider = Provider<CrashlyticsHelper>(
  (ref) => getIt.get<CrashlyticsHelper>(),
);

@LazySingleton()
class CrashlyticsHelper {
  Future<void> recordError({
    dynamic exception,
    dynamic reason,
    bool? printDetails,
    StackTrace? stack,
    bool fatal = false,
    Iterable<Object> information = const [],
  }) {
    return FirebaseCrashlytics.instance.recordError(
      exception,
      stack,
      reason: reason,
      information: information,
      printDetails: printDetails,
      fatal: fatal,
    );
  }
}
```

### 3.3 Custom Keys & Logs

```dart
// Record Flutter error (automatic via FlutterError.onError)
FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterError;

// Record non-fatal exception manually
try {
  await api.call();
} catch (e, stackTrace) {
  await FirebaseCrashlytics.instance.recordError(
    e,
    stackTrace,
    reason: 'API call failed',
    extraData: {'endpoint': '/api/user'},
  );
}
```

### 3.4 Custom Keys & Logs

```dart
// Custom keys
await FirebaseCrashlytics.instance.setCustomKey('user_id', '123');
await FirebaseCrashlytics.instance.setCustomKey('plan', 'premium');

// User identifier
await FirebaseCrashlytics.instance.setUserIdentifier('user_123');

// Log messages
await FirebaseCrashlytics.instance.log('User clicked purchase button');

// Breadcrumbs (manual)
await FirebaseCrashlytics.instance.log('BREADCRUMB: Step 1 completed');
```

### 3.4 Crashlytics vs Analytics

| Aspect | Crashlytics | Analytics |
|--------|-------------|-----------|
| Purpose | Exception/crash reporting | User behavior |
| Data | Stack traces, exceptions | Events, metrics |
| Latency | Real-time | Batched |
| Cost | Free (Spark/Blaze) | Free (limited) / Pay |

> 💡 **FE Perspective**
> **Flutter Crashlytics** ≈ **Sentry SDK** in JavaScript.
> **Key difference:** Crashlytics is Firebase-native, Sentry is cross-platform.

---

## Concept 4: Firebase Auth

<!-- AI_VERIFY: concept-firebase-auth -->

### 4.1 Auth State Stream

```dart
class AuthNotifier extends ChangeNotifier {
  final FirebaseAuth _auth = FirebaseAuth.instance;

  AuthNotifier() {
    _auth.authStateChanges().listen((user) {
      _currentUser = user;
      notifyListeners();
    });
  }
}
```

### 4.2 Sign-In Methods

```dart
// Email/Password
await FirebaseAuth.instance.signInWithEmailAndPassword(
  email: 'user@example.com',
  password: 'password123',
);

// Google Sign-In
final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
final GoogleSignInAuthentication googleAuth = await googleUser!.authentication;
final credential = GoogleAuthProvider.credential(
  accessToken: googleAuth.accessToken,
  idToken: googleAuth.idToken,
);
await FirebaseAuth.instance.signInWithCredential(credential);
```

### 4.3 User Management

```dart
// Get current user
final user = FirebaseAuth.instance.currentUser;

// Update profile
await user?.updateDisplayName('John Doe');
await user?.updatePhotoURL('https://example.com/photo.jpg');

// Send verification email
await user?.sendEmailVerification();

// Password reset
await FirebaseAuth.instance.sendPasswordResetEmail(email: 'user@example.com');
```

### 4.4 Token Refresh

```dart
// Get ID token (auto-refreshed)
final idToken = await user?.getIdToken();

// Force refresh
final idToken = await user?.getIdToken(true);

// Listen to token changes
user?.authStateChanges().listen((user) async {
  if (user != null) {
    final token = await user.getIdToken();
    // Update token on server
  }
});
```

> 💡 **FE Perspective**
> **Flutter Auth** ≈ Firebase Auth JS SDK with `onAuthStateChanged()`.
> **React/Vue tương đương:** `useAuthState()` hook or `getAuth()` context.

---

## Concept 5: Firebase Remote Config

<!-- AI_VERIFY: concept-firebase-remote-config -->

### 5.1 Parameter Types

| Type | Method | Default |
|------|--------|---------|
| Boolean | `getBool()` | `setDefaults({'key': false})` |
| Integer | `getInt()` | `setDefaults({'key': 0})` |
| Double | `getDouble()` | `setDefaults({'key': 0.0})` |
| String | `getString()` | `setDefaults({'key': ''})` |

### 5.2 Fetch & Activate Pattern

```dart
class RemoteConfigManager {
  final FirebaseRemoteConfig _config = FirebaseRemoteConfig.instance;

  Future<void> initialize() async {
    await _config.setDefaults({
      'show_new_feature': false,
      'max_retry': 3,
      'maintenance_mode': false,
    });

    await fetchAndActivate();
  }

  Future<bool> fetchAndActivate() async {
    await _config.setConfigSettings(RemoteConfigSettings(
      fetchTimeout: const Duration(minutes: 1),
      minimumFetchInterval: const Duration(hours: 1),
    ));

    await _config.fetch();
    return await _config.activate();
  }
}
```

### 5.3 Conditions & Targeting

``` dart
// Firebase Console conditions:
// - App version: App version == "2.0.0"
// - Platform: Platform == "iOS"
// - User in segment: User property == "premium"

// Usage
if (_config.getBool('show_new_feature')) {
  // Show feature
}
```

> 💡 **FE Perspective**
> **Flutter Remote Config** ≈ Firebase Remote Config JS with `fetchAndActivate()`.
> **React/Vue tương đương:** `remoteConfig().fetchAndActivate()`.

---

## Concept 6: Firebase Cloud Messaging (FCM)

<!-- AI_VERIFY: concept-firebase-messaging -->

> ⚠️ **Trong project này:** Messaging được quản lý qua `FirebaseMessagingService` class với `@LazySingleton` injectable pattern, accessed qua Riverpod provider.

### 6.1 Message Types

| Type | When | Handler |
|------|------|---------|
| **Foreground** | App is open/visible | `onMessage` listener |
| **Background** | App is in background | Background handler |
| **Terminated** | App is closed | `getInitialMessage()` |

### 6.2 Message Structure

```dart
// RemoteMessage
class RemoteMessage {
  String? messageId;      // Unique message ID
  String? collapseKey;     // Collapse key for grouping
  String? senderId;        // Sender ID
  String? category;         // Message category
  String? threadId;        // Thread ID for iOS
  Map<String, String>? data;         // Custom data
  Notification? notification;         // Display notification
  AndroidNotification? android;       // Android-specific
  AppleNotification? apple;           // iOS-specific
}
```

### 6.3 Token Management

```dart
// ⚠️ Sử dụng FirebaseMessagingService từ DI
// firebase_messaging_service.dart
final firebaseMessagingServiceProvider = Provider((ref) => getIt.get<FirebaseMessagingService>());

@LazySingleton()
class FirebaseMessagingService {
  final _messaging = FirebaseMessaging.instance;

  // Get FCM token
  Future<String?> get deviceToken async {
    final deviceToken = await _messaging.getToken();
    return deviceToken;
  }

  // Token refresh stream
  Stream<String> get onTokenRefresh => _messaging.onTokenRefresh;

  // Topic subscription
  Future<void> subscribeToTopic(String topic) async {
    await _messaging.subscribeToTopic(topic);
  }

  Future<void> unsubscribeFromTopic(String topic) async {
    await _messaging.unsubscribeFromTopic(topic);
  }
}
```

### 6.4 Message Streams

```dart
// Get FCM token
final token = await FirebaseMessaging.instance.getToken();

// Token refresh stream
FirebaseMessaging.instance.onTokenRefresh.listen((newToken) {
  // Send new token to server
  _updateTokenOnServer(newToken);
});

// Topic subscription
await FirebaseMessaging.instance.subscribeToTopic('news');
await FirebaseMessaging.instance.unsubscribeFromTopic('news');
```

### 6.4 Background Handler

```dart
// Must be top-level function with @pragma
@pragma('vm:entry-point')
Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // Handle message
}

// Register in main.dart
FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);
```

> 💡 **FE Perspective**
> **Flutter FCM** ≈ Firebase Messaging JS SDK with `onMessage()` callback.
> **React/Vue tương đương:** `onMessage` event handler in `messaging().onMessage()`.

---

## Concept 7: Firebase Storage & Firestore

<!-- AI_VERIFY: concept-firebase-storage-firestore -->

### 7.1 Firebase Storage

```dart
// Upload file
final ref = FirebaseStorage.instance.ref('uploads/${fileName}');
final uploadTask = ref.putFile(file, SettableMetadata(
  contentType: 'image/jpeg',
));

// Track progress
uploadTask.snapshotEvents.listen((TaskSnapshot snapshot) {
  final progress = snapshot.bytesTransferred / snapshot.totalBytes;
  print('Upload progress: $progress%');
});

// Get download URL
final url = await ref.getDownloadURL();

// Delete file
await ref.delete();
```

### 7.2 Cloud Firestore

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

// Add document
await FirebaseFirestore.instance.collection('users').add({
  'name': 'John Doe',
  'email': 'john@example.com',
  'createdAt': FieldValue.serverTimestamp(),
});

// Get document
final doc = await FirebaseFirestore.instance
    .collection('users')
    .doc('user_id')
    .get();

// Real-time updates
FirebaseFirestore.instance
    .collection('users')
    .doc('user_id')
    .snapshots()
    .listen((snapshot) {
      final user = snapshot.data();
      print('User updated: $user');
    });

// Query
final users = await FirebaseFirestore.instance
    .collection('users')
    .where('isActive', isEqualTo: true)
    .orderBy('createdAt', descending: true)
    .limit(10)
    .get();
```

### 7.3 Security Rules

```javascript
// Firestore rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // User can read/write their own document
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Public read, auth write
    match /posts/{postId} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

> 💡 **FE Perspective**
> **Flutter Firestore** ≈ Firebase Firestore JS SDK with `getDoc()`, `setDoc()`, `onSnapshot()`.
> **React/Vue tương đương:** Firestore SDK with React hooks (`useDocument`, `useCollection`).

---

## Quick Reference

| Service | Package | Key API | Project Pattern |
|---------|---------|---------|----------------|
| Core | `firebase_core` | `Firebase.initializeApp()` | - |
| Analytics | `firebase_analytics` | `logEvent()` | `AnalyticsHelper` + Riverpod |
| Crashlytics | `firebase_crashlytics` | `recordError()` | `CrashlyticsHelper` + `@LazySingleton` |
| Auth | `firebase_auth` | `signInWithEmailAndPassword()` | SDK directly |
| Remote Config | `firebase_remote_config` | `fetchAndActivate()` | ⚠️ **Không sử dụng** |
| Messaging | `firebase_messaging` | `getToken()` | `FirebaseMessagingService` + `@LazySingleton` |
| Storage | `firebase_storage` | `putFile()` | SDK directly |
| Firestore | `cloud_firestore` | `add()`, `get()` | `FirebaseFirestoreService` + `@LazySingleton` |

> ⚠️ **Lưu ý:** `FirebaseRemoteConfigService` **KHÔNG tồn tại** trong project này.

### Firebase vs Sentry

| Feature | Firebase Crashlytics | Sentry |
|---------|---------------------|--------|
| Cost | Free (Spark/Blaze) | Free (5k events/mo) / Pay |
| Platform | Firebase-native | Cross-platform |
| Integration | Easy with Firebase | Easy setup |
| Features | Basic crash reporting | Advanced issue tracking |

→ **Tiếp theo**: [03-exercise.md](./03-exercise.md) — Hands-on exercises.

<!-- AI_VERIFY: generation-complete -->
