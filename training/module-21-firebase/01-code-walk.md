# Module 21 – Code Walk: Firebase Services Integration

> **Mục tiêu**: Đọc hiểu Firebase integration patterns — initialization, Analytics, Auth, Crashlytics, Messaging, Storage.

📌 **Recap**: M1 (WidgetsFlutterBinding) · M3 (Config/env) · M20 (Platform Channels)

---

## 1. Firebase Initialization

<!-- AI_VERIFY: section-firebase-init -->

### 1.1 main.dart — Firebase Core Setup

```dart
// ACTUAL SOURCE: lib/main.dart
import 'dart:async';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:flutter/material.dart';
import 'package:flutter_native_splash/flutter_native_splash.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import 'index.dart';

// ignore: avoid_unnecessary_async_function
Future<void> main() async => runZonedGuarded(
      _runMyApp,
      (error, stackTrace) => _reportError(error: error, stackTrace: stackTrace),
    );

Future<void> _runMyApp() async {
  final widgetsBinding = WidgetsFlutterBinding.ensureInitialized();
  FlutterNativeSplash.preserve(widgetsBinding: widgetsBinding);
  await Firebase.initializeApp();
  await AppInitializer.init();
  final initialResource = _loadInitialResource();
  runApp(ProviderScope(
    observers: [AppProviderObserver()],
    child: MyApp(initialResource: initialResource),
  ));
}

void _reportError({required error, required StackTrace stackTrace}) {
  Log.e(error, stackTrace: stackTrace, name: 'Uncaught exception');
  FirebaseCrashlytics.instance.recordError(error, stackTrace);
}
```

**Key observations:**
- `Firebase.initializeApp()` được gọi **trước** `AppInitializer.init()` — thứ tự QUAN TRỌNG
- `runZonedGuarded()` wrap toàn bộ app — uncaught exceptions được gửi lên Crashlytics
- `FlutterError.onError` KHÔNG được set trong main.dart — được set bởi `CrashlyticsHelper` bên trong AppInitializer
- `FirebaseCrashlytics.instance.recordError()` được gọi trong `_reportError()` — chỉ khi có uncaught exception

### 1.2 iOS Configuration — GoogleService-Info.plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>CLIENT_ID</key>
  <string>123456789-abcdefghijklmnop.apps.googleusercontent.com</string>
  <key>REVERSED_CLIENT_ID</key>
  <string>com.googleusercontent.apps.123456789-abcdefghijklmnop</string>
  <key>API_KEY</key>
  <string>AIzaSy...xyz</string>
  <key>GCM_SENDER_ID</key>
  <string>123456789000</string>
  <key>PLIST_VERSION</key>
  <string>1</string>
  <key>BUNDLE_ID</key>
  <string>jp.flutter.app</string>
  <key>PROJECT_ID</key>
  <string>my-flutter-app</string>
  <key>STORAGE_BUCKET</key>
  <string>my-flutter-app.appspot.com</string>
  <key>IS_ADS_ENABLED</key>
  <false/>
  <key>IS_ANALYTICS_ENABLED</key>
  <false/>
  <key>IS_APPINVITER_ENABLED</key>
  <true/>
  <key>IS_GCM_ENABLED</key>
  <true/>
  <key>IS_SIGNIN_ENABLED</key>
  <true/>
  <key>GOOGLE_APP_ID</key>
  <string>1:123456789000:ios:abcdef123456</string>
</dict>
</plist>
```

### 1.3 Android Configuration — google-services.json

```json
{
  "project_info": {
    "project_number": "123456789000",
    "project_id": "my-flutter-app",
    "storage_bucket": "my-flutter-app.appspot.com"
  },
  "client": [
    {
      "client_info": {
        "mobilesdk_app_id": "1:123456789000:android:abcdef123456",
        "android_client_info": {
          "package_name": "jp.flutter.app"
        }
      },
      "oauth_client": [
        {
          "client_id": "123456789-abcdefghijklmnop.apps.googleusercontent.com",
          "client_type": 3
        }
      ],
      "api_key": [
        {
          "current_key": "AIzaSy...xyz"
        }
      ],
      "services": {
        "appinvite_service": {
          "other_platform_oauth_client": [
            {
              "client_id": "123456789-abcdefghijklmnop.apps.googleusercontent.com",
              "client_type": 3
            }
          ]
        }
      }
    }
  ],
  "configuration_version": "1"
}
```

---

## 2. Firebase Analytics

<!-- AI_VERIFY: section-firebase-analytics -->

### 2.1 Analytics Service — Actual Source

<!-- AI_VERIFY: base_flutter/lib/common/helper/analytics/analytics_helper.dart -->

→ [Mở file gốc: `lib/common/helper/analytics/analytics_helper.dart`](../../base_flutter/lib/common/helper/analytics/analytics_helper.dart)

```dart
// ACTUAL SOURCE: lib/common/helper/analytics/analytics_helper.dart
import 'package:firebase_analytics/firebase_analytics.dart';
import 'package:flutter/foundation.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import '../../../index.dart';

final analyticsHelperProvider = Provider<AnalyticsHelper>(
  (ref) => AnalyticsHelper(ref: ref),
);

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
    if (kDebugMode) {
      Log.d(
        'logEvent: ${event.fullEventName},\nparameters: ${Log.prettyJson(parameters)}',
        color: LogColor.cyan,
        mode: LogMode.logEvent,
      );
    }
    return _firebaseAnalytics.logEvent(
      name: event.fullEventName,
      parameters: parameters,
    );
  }

  Future<void> logScreenView(ScreenViewEvent screenViewEvent) async {
    final parameters = {
      ...await _commonParameters,
      ...screenViewEvent.parameter?.parameters ?? {},
    };
    if (kDebugMode) {
      Log.d(
        'logScreenView: ${screenViewEvent.screenName},\nkey: ${screenViewEvent.fullKey}\nparameters: ${Log.prettyJson(parameters)}',
        color: LogColor.cyan,
        mode: LogMode.logEvent,
      );
    }
    return _firebaseAnalytics.logScreenView(
      screenName: screenViewEvent.screenName.screenName,
      screenClass: screenViewEvent.screenName.screenClass,
      parameters: parameters,
    );
  }

  Future<void> logPurchase({
    String? currency,
    String? coupon,
    double? value,
    List<AnalyticsEventItem>? items,
    double? tax,
    double? shipping,
    String? transactionId,
    String? affiliation,
  }) async {
    if (kDebugMode) {
      Log.d(
        'logPurchase: currency: $currency, coupon: $coupon, value: $value, ...',
        color: LogColor.cyan,
        mode: LogMode.logEvent,
      );
    }
    await _firebaseAnalytics.logPurchase(
      currency: currency,
      coupon: coupon,
      value: value,
      items: items,
      tax: tax,
      shipping: shipping,
      transactionId: transactionId,
      affiliation: affiliation,
    );
  }

  Future<void> setUserId(String userId) {
    return FirebaseAnalytics.instance.setUserId(id: userId);
  }

  Future<void> setUserProperties(Map<String, String?> properties) async {
    await Future.wait(properties.entries.map((e) async {
      await FirebaseAnalytics.instance.setUserProperty(name: e.key, value: e.value);
    }));
  }

  Future<void> reset() {
    return FirebaseAnalytics.instance.resetAnalyticsData();
  }
}
```

**Key observations:**
- Class name: `AnalyticsHelper` — NOT `FirebaseAnalyticsService`
- Provider: `analyticsHelperProvider` (Riverpod) — NOT static singleton
- Uses **event classes** (`NormalEvent`, `ScreenViewEvent`) — NOT raw string event names
- All events automatically include `_commonParameters` (userId from `deviceHelperProvider`)
- Debug logging via `Log.d()` with `LogMode.logEvent` — NOT `print()`
- **`@Riverpod` injection** via `Ref` in constructor — NOT manual singleton

### 2.2 Usage in Pages — Actual Pattern

```dart
// ACTUAL: AnalyticsHelper uses Riverpod, not static singleton
// Usage pattern via ConsumerWidget:

class HomePage extends ConsumerStatefulWidget {
  const HomePage({super.key});

  @override
  ConsumerState<HomePage> createState() => _HomePageState();
}

class _HomePageState extends ConsumerState<HomePage> {
  @override
  void initState() {
    super.initState();
    // Track screen view — uses AnalyticsHelper via Riverpod
    ref.read(analyticsHelperProvider).logScreenView(
      ScreenViewEvent(screenName: ScreenName.homePage),
    );
  }

  void _onButtonTap(String buttonName) {
    // Track custom event — uses NormalEvent classes
    ref.read(analyticsHelperProvider).logEvent(
      NormalEvent.buttonTap(buttonName: buttonName),
    );
  }
}
```

**Key differences from teaching pattern:**

| Aspect | Teaching Pattern | Actual `base_flutter` |
|--------|-----------------|----------------------|
| Class name | `FirebaseAnalyticsService` | `AnalyticsHelper` |
| Instantiation | Static singleton (`factory`) | Riverpod provider |
| Event names | Raw strings (`'button_tap'`) | Event classes (`NormalEvent.buttonTap()`) |
| Common params | Manual per-call | Auto-injected via `_commonParameters` |
| Debug logging | `print()` | `Log.d()` with `LogMode.logEvent` |
| Screen tracking | `setCurrentScreen()` | `logScreenView(ScreenViewEvent)` |

---

## 3. Firebase Crashlytics

<!-- AI_VERIFY: section-firebase-crashlytics -->

### 3.1 Crashlytics Helper — Actual Source

> **NOTE**: `base_flutter` does NOT have a `FirebaseCrashlyticsService` singleton class. Instead, it uses `CrashlyticsHelper` — a simple `@LazySingleton()` injectable class managed by Riverpod.

→ [Mở file gốc: `lib/common/helper/crashlytics_helper.dart`](../../base_flutter/lib/common/helper/crashlytics_helper.dart)

```dart
// ACTUAL SOURCE: lib/common/helper/crashlytics_helper.dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:injectable/injectable.dart';

import '../../../index.dart';

final crashlyticsHelperProvider = Provider<CrashlyticsHelper>(
  (ref) => getIt.get<CrashlyticsHelper>(),
);

@LazySingleton()
class CrashlyticsHelper {
  Future<void> recordError({
    // ignore: avoid_dynamic
    dynamic exception,
    // ignore: avoid_dynamic
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

**Key observations:**
- Class name: `CrashlyticsHelper` — NOT `FirebaseCrashlyticsService`
- Provider: `crashlyticsHelperProvider` (Riverpod) — NOT static singleton
- Injected via `@LazySingleton()` — NOT manually instantiated
- Only exposes `recordError()` — no `initialize()`, `log()`, `setUserIdentifier()`, `recordBreadcrumb()`, etc.
- **`@Riverpod` / `injectable` injection** — managed by `get_it` DI container

### 3.2 How Crashlytics is actually wired in AppInitializer

```dart
// ACTUAL SOURCE: lib/common/app_initializer.dart (excerpt)
// FlutterError.onError is set inside AppInitializer.init()
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

Future<void> init() async {
  // ... other initialization ...

  // Set up Flutter error handler for Crashlytics
  // This is done here, not in main.dart
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterError;

  // ... rest of initialization ...
}
```

**Key observations:**
- `FlutterError.onError` is set inside `AppInitializer.init()`, NOT in main.dart
- `CrashlyticsHelper.recordError()` is available for manual error recording
- The `runZonedGuarded()` in main.dart calls `FirebaseCrashlytics.instance.recordError()` directly for uncaught exceptions

### 3.3 Usage in Exception Handler

> **NOTE**: `base_flutter` does NOT have a `ExceptionHandler` class as shown. Instead, it uses `RemoteException` with `CrashlyticsHelper`.

```dart
// ACTUAL: RemoteException from lib/exception/remote_exception.dart
// CrashlyticsHelper is used via DI to record errors:

class SomeService {
  final CrashlyticsHelper _crashlyticsHelper;

  Future<void> doSomething() async {
    try {
      // ... code ...
    } catch (e, st) {
      await _crashlyticsHelper.recordError(
        exception: e,
        reason: 'Context: doSomething failed',
        stack: st,
      );
      rethrow;
    }
  }
}
```

**Key differences from teaching pattern:**

|| Aspect | Teaching Pattern | Actual `base_flutter` |
|--------|-----------------|----------------------|
| Class name | `FirebaseCrashlyticsService` | `CrashlyticsHelper` |
| Instantiation | Static singleton | Riverpod + `@LazySingleton()` |
| Methods | `initialize()`, `log()`, `setUserIdentifier()`, etc. | Only `recordError()` |
| Usage | Manual initialization | Auto-injected via DI |

---

## 4. Firebase Auth

<!-- AI_VERIFY: section-firebase-auth -->

### 4.1 Auth Exception Types — Actual Source

> **NOTE**: `base_flutter` does NOT have a `FirebaseAuthService` singleton class. Auth is handled via Firebase SDK directly with custom exception types.

→ [Mở file gốc: `lib/exception/app_firebase_auth_exception.dart`](../../base_flutter/lib/exception/app_firebase_auth_exception.dart)

```dart
// ACTUAL SOURCE: lib/exception/app_firebase_auth_exception.dart
import '../index.dart';

class AppFirebaseAuthException extends AppException {
  AppFirebaseAuthException({
    required this.kind,
    super.rootException,
    super.onRetry,
  }) : super();

  final AppFirebaseAuthExceptionKind kind;

  @override
  String get message => switch (kind) {
        AppFirebaseAuthExceptionKind.invalidEmail => l10n.invalidEmail,
        AppFirebaseAuthExceptionKind.userDoesNotExist => l10n.userDoesNotExist,
        AppFirebaseAuthExceptionKind.invalidLoginCredentials => l10n.invalidLoginCredentials,
        AppFirebaseAuthExceptionKind.usernameAlreadyInUse => l10n.usernameAlreadyInUse,
        AppFirebaseAuthExceptionKind.requiresRecentLogin => l10n.requiresRecentLogin,
        AppFirebaseAuthExceptionKind.unknown => l10n.unknownException(errorCode: 'FBA-001'),
      };

  @override
  AppExceptionAction get action => AppExceptionAction.doNothing;
}

enum AppFirebaseAuthExceptionKind {
  invalidEmail,
  invalidLoginCredentials,
  userDoesNotExist,
  usernameAlreadyInUse,
  requiresRecentLogin,
  unknown
}
```

**Key observations:**
- `base_flutter` uses **`FirebaseAuth`** SDK directly (e.g., `FirebaseAuth.instance`)
- No custom `FirebaseAuthService` class exists — auth is typically handled via API interceptors
- Auth exceptions are wrapped in `AppFirebaseAuthException` with localized messages
- **`AppFirebaseAuthExceptionKind`** enum covers: invalidEmail, invalidLoginCredentials, userDoesNotExist, usernameAlreadyInUse, requiresRecentLogin, unknown
- Exception messages are **localized** via `l10n` (flutter_slang)

### 4.2 Auth Usage Pattern

```dart
// In API interceptors or services, Firebase Auth is used directly:
import 'package:firebase_auth/firebase_auth.dart';

// Get current user
final user = FirebaseAuth.instance.currentUser;

// Listen to auth state changes
FirebaseAuth.instance.authStateChanges().listen((user) {
  // Handle auth state change
});

// Sign in with email/password
try {
  await FirebaseAuth.instance.signInWithEmailAndPassword(
    email: email,
    password: password,
  );
} on FirebaseAuthException catch (e) {
  throw AppFirebaseAuthException(
    kind: _mapFirebaseAuthError(e.code),
    rootException: e,
  );
}
```

**Key differences from teaching pattern:**

|| Aspect | Teaching Pattern | Actual `base_flutter` |
|--------|-----------------|----------------------|
| Class name | `FirebaseAuthService` | Does NOT exist |
| Instantiation | Static singleton | Uses `FirebaseAuth.instance` directly |
| Auth state | Custom `AuthStateNotifier` | Via interceptors/API layer |
| Exceptions | Generic try/catch | `AppFirebaseAuthException` with `AppFirebaseAuthExceptionKind` |
| Localization | Not shown | All error messages via `l10n` |

### 4.3 Auth Exception Mapping

```dart
// How base_flutter maps Firebase Auth errors to AppFirebaseAuthExceptionKind
AppFirebaseAuthExceptionKind _mapFirebaseAuthError(String code) {
  return switch (code) {
    'invalid-email' => AppFirebaseAuthExceptionKind.invalidEmail,
    'user-not-found' || 'wrong-password' => AppFirebaseAuthExceptionKind.invalidLoginCredentials,
    'email-already-in-use' => AppFirebaseAuthExceptionKind.usernameAlreadyInUse,
    'requires-recent-login' => AppFirebaseAuthExceptionKind.requiresRecentLogin,
    _ => AppFirebaseAuthExceptionKind.unknown,
  };
}
```

---

## 5. Firebase Cloud Messaging (Push Notifications)

<!-- AI_VERIFY: section-firebase-messaging -->

### 5.1 Messaging Service — Actual Source

→ [Mở file gốc: `lib/data_source/firebase/messaging/firebase_messaging_service.dart`](../../base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart)

```dart
// ACTUAL SOURCE: lib/data_source/firebase/messaging/firebase_messaging_service.dart
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:injectable/injectable.dart';

import '../../../index.dart';

final firebaseMessagingServiceProvider = Provider((ref) => getIt.get<FirebaseMessagingService>());

@LazySingleton()
class FirebaseMessagingService {
  final _messaging = FirebaseMessaging.instance;

  Stream<String> get onTokenRefresh => _messaging.onTokenRefresh;

  Future<String?> get deviceToken async {
    try {
      final deviceToken = await _messaging.getToken();
      return deviceToken;
    } catch (e) {
      Log.e('Error getting device token: $e');
      return null;
    }
  }

  Stream<RemoteMessage> get onMessage => FirebaseMessaging.onMessage;

  Stream<RemoteMessage> get onMessageOpenedApp => FirebaseMessaging.onMessageOpenedApp;

  Future<RemoteMessage?> get initialMessage => _messaging.getInitialMessage();

  Future<void> deleteToken() async {
    await _messaging.deleteToken();
  }

  Future<void> subscribeToTopic(String topic) async {
    Log.d('Subscribing to topic: $topic');
    await _messaging.subscribeToTopic(topic);
  }

  Future<void> unsubscribeFromTopic(String topic) async {
    Log.d('Unsubscribing from topic: $topic');
    await _messaging.unsubscribeFromTopic(topic);
  }
}
```

**Key observations:**
- Class name: `FirebaseMessagingService` — matches teaching, but **injected via `@LazySingleton()`**
- Uses **`Riverpod` + `injectable`** — not a static singleton
- No `requestPermission()` method — permission is requested implicitly by the SDK or handled elsewhere
- No `initialize()` with handlers — foreground/background message handling is done in page code
- Token management: `deviceToken`, `onTokenRefresh`, `deleteToken()`
- Topic management: `subscribeToTopic()`, `unsubscribeToTopic()`
- **`Log.d()`** for debug logging — NOT `print()`

### 5.2 Usage in Pages

```dart
// ACTUAL: FirebaseMessagingService usage via Riverpod
class SplashPage extends ConsumerStatefulWidget {
  const SplashPage({super.key});

  @override
  ConsumerState<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends ConsumerState<SplashPage> {
  @override
  void initState() {
    super.initState();
    _initializeFCM();
  }

  Future<void> _initializeFCM() async {
    final messaging = ref.read(firebaseMessagingServiceProvider);

    // Get device token
    final token = await messaging.deviceToken;
    if (token != null) {
      // Send token to server
      Log.d('FCM token: $token');
    }

    // Listen for token refresh
    messaging.onTokenRefresh.listen((newToken) {
      Log.d('FCM token refreshed: $newToken');
    });

    // Listen for foreground messages
    messaging.onMessage.listen((message) {
      Log.d('Foreground message: ${message.notification?.title}');
    });

    // Listen for when app opened from background
    messaging.onMessageOpenedApp.listen((message) {
      Log.d('App opened from notification: ${message.data}');
    });
  }
}
```

**Key differences from teaching pattern:**

|| Aspect | Teaching Pattern | Actual `base_flutter` |
|--------|-----------------|----------------------|
| Class name | `FirebaseMessagingService` | `FirebaseMessagingService` (matches) |
| Instantiation | Static singleton | Riverpod + `@LazySingleton()` |
| Permission | `requestPermission()` shown | Not shown — SDK handles implicitly |
| Initialization | `initialize()` with handlers | No `initialize()` — handlers in page code |
| Token | `getToken()` | `deviceToken` (getter) |
| Logging | `print()` | `Log.d()` |

---

## 6. Firebase Remote Config

<!-- AI_VERIFY: section-firebase-remote-config -->

### 6.1 Remote Config — Not Implemented in `base_flutter`

> **NOTE**: `base_flutter` does NOT have a `FirebaseRemoteConfigService` class or any Remote Config implementation. This section is provided as a teaching pattern only.

If you need to implement Remote Config in your project, here is a reference pattern:

```dart
// Reference implementation (NOT in base_flutter)
import 'package:firebase_remote_config/firebase_remote_config.dart';
import 'package:flutter/foundation.dart';

class FirebaseRemoteConfigService {
  static final FirebaseRemoteConfig _remoteConfig =
      FirebaseRemoteConfig.instance;

  static final FirebaseRemoteConfigService _instance =
      FirebaseRemoteConfigService._internal();
  factory FirebaseRemoteConfigService() => _instance;
  FirebaseRemoteConfigService._internal();

  // Default values
  static const Map<String, dynamic> _defaults = {
    'show_feature_x': false,
    'max_retry_count': 3,
    'min_app_version': '1.0.0',
    'maintenance_mode': false,
  };

  // Initialize
  Future<void> initialize() async {
    await _remoteConfig.setDefaults(_defaults);

    // Settings
    await _remoteConfig.setConfigSettings(RemoteConfigSettings(
      fetchTimeout: const Duration(minutes: 1),
      minimumFetchInterval: const Duration(hours: 1),
    ));
  }

  // Fetch and activate
  Future<bool> fetchAndActivate() async {
    await _remoteConfig.fetch();
    return await _remoteConfig.activate();
  }

  // Get values
  bool getBool(String key) => _remoteConfig.getBool(key);
  int getInt(String key) => _remoteConfig.getInt(key);
  double getDouble(String key) => _remoteConfig.getDouble(key);
  String getString(String key) => _remoteConfig.getString(key);

  // Check if key exists
  bool containsKey(String key) => _remoteConfig.containsKey(key);

  // Get all keys
  Set<String> getAllKeys() => _remoteConfig.getKeysByPrefix('');
}
```

### 6.2 Usage

```dart
// Check if feature is enabled
final showFeatureX = FirebaseRemoteConfigService().getBool('show_feature_x');

if (showFeatureX) {
  // Show feature
}

// Check minimum app version
final minVersion = FirebaseRemoteConfigService().getString('min_app_version');
if (_compareVersions(minVersion) > 0) {
  // Show update prompt
}
```

---

## 7. Firebase Storage

<!-- AI_VERIFY: section-firebase-storage -->

### 7.1 Storage — Not Implemented in `base_flutter`

> **NOTE**: `base_flutter` does NOT have a `FirebaseStorageService` class or any Firebase Storage implementation. This section is provided as a teaching pattern only.

If you need to implement Firebase Storage in your project, here is a reference pattern:

```dart
// Reference implementation (NOT in base_flutter)
import 'package:firebase_storage/firebase_storage.dart';
import 'package:path/path.dart' as path;

class FirebaseStorageService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  static final FirebaseStorageService _instance =
      FirebaseStorageService._internal();
  factory FirebaseStorageService() => _instance;
  FirebaseStorageService._internal();

  // Upload file
  Future<String> uploadFile({
    required String filePath,
    required String destinationPath,
    void Function(TaskSnapshot)? onProgress,
  }) async {
    final file = path.basename(filePath);
    final ref = _storage.ref().child(destinationPath).child(file);

    final uploadTask = ref.putFile(
      File(filePath),
      SettableMetadata(
        contentType: _getContentType(file),
      ),
    );

    if (onProgress != null) {
      uploadTask.snapshotEvents.listen(onProgress);
    }

    final snapshot = await uploadTask;
    return await snapshot.ref.getDownloadURL();
  }

  // Download URL
  Future<String> getDownloadURL(String path) async {
    return await _storage.ref(path).getDownloadURL();
  }

  // Delete file
  Future<void> deleteFile(String path) async {
    await _storage.ref(path).delete();
  }

  // List files
  Future<ListResult> listFiles(String path) async {
    return await _storage.ref(path).listAll();
  }

  // Get content type
  String _getContentType(String filePath) {
    final ext = path.extension(filePath).toLowerCase();
    switch (ext) {
      case '.jpg':
      case '.jpeg':
        return 'image/jpeg';
      case '.png':
        return 'image/png';
      case '.pdf':
        return 'application/pdf';
      default:
        return 'application/octet-stream';
    }
  }
}
```

> **Teaching Note**: `base_flutter` uses **`FirebaseFirestoreService`** for data storage (NoSQL database), not Firebase Storage for file uploads. If you need file upload/download, implement `FirebaseStorageService` as shown above.

---

## Tổng kết

> **CRITICAL UPDATE**: The following services are NOT implemented in `base_flutter`:
> - `FirebaseAuthService` (singleton) — use `FirebaseAuth.instance` directly
> - `FirebaseRemoteConfigService` — not implemented
> - `FirebaseStorageService` — not implemented
> - `FirebaseCrashlyticsService` (singleton) — use `CrashlyticsHelper` via DI
>
> Actual services that exist in `base_flutter`:
> - `AnalyticsHelper` (Riverpod provider) — for Firebase Analytics
> - `CrashlyticsHelper` (DI injectable) — for Crashlytics error recording
> - `FirebaseMessagingService` (DI injectable) — for FCM push notifications
> - `FirebaseFirestoreService` (DI injectable) — for Firestore NoSQL
> - `AppFirebaseAuthException` — for auth error handling

```
Firebase Services Integration:
├── Firebase.initializeApp()
│   ├── google-services.json (Android)
│   └── GoogleService-Info.plist (iOS)
├── Firebase Analytics
│   ├── AnalyticsHelper (actual — Riverpod provider)
│   ├── logEvent()
│   ├── logScreenView()
│   └── setUserId()
├── Firebase Crashlytics
│   ├── CrashlyticsHelper (actual — DI injectable)
│   ├── recordError()
│   └── FlutterError.onError set in AppInitializer
├── Firebase Auth
│   ├── FirebaseAuth.instance (direct usage)
│   ├── AppFirebaseAuthException (custom exception)
│   └── authStateChanges stream
├── Firebase Cloud Messaging
│   ├── FirebaseMessagingService (actual — DI injectable)
│   ├── deviceToken
│   ├── onTokenRefresh
│   └── subscribeToTopic()
├── Firebase Firestore
│   └── FirebaseFirestoreService (actual — DI injectable)
└── Firebase Remote Config
    └── NOT IMPLEMENTED in base_flutter (teaching only)

⚠️ Firebase Storage is NOT implemented in base_flutter (teaching only)
```

→ **Tiếp theo**: [02-concept.md](./02-concept.md) — Đi sâu 7 concepts về Firebase services.

<!-- AI_VERIFY: generation-complete -->
