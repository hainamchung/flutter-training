# Code Walk — Architecture & Dependency Injection

> 📌 **Recap:** M1: `main.dart`, bootstrap flow | M3: Config, environment | M8: Riverpod providers

---

## Walk Overview

Survey **3 lớp** DI system — service locator setup, injectable code generation, và runtime resolution.

```
get_it (service locator)
  ├── di.dart           — setup + injectable init
  ├── di.config.dart    — generated registrations
  └── usage sites       — getIt<Service>()
```

---

## Part A — Service Locator Setup

### 1. di.dart — get_it Configuration

<!-- AI_VERIFY: base_flutter/lib/di.dart -->
```
📁 lib/di.dart
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/di.dart`](../../base_flutter/lib/di.dart)

```dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';

// ignore: prefer_importing_index_file
import 'di.config.dart';

@module
abstract class ServiceModule {
  @preResolve
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}

final GetIt getIt = GetIt.instance;

@InjectableInit()
Future<void> configureInjection() => getIt.init();
```

**Structural analysis:**

| Element | Purpose |
|---------|---------|
| `final GetIt getIt = GetIt.instance` | Singleton service locator instance |
| `@module abstract class` | injectable module — groups registrations |
| `@preResolve` | Async initialization — waits for SharedPreferences before app starts |
| `@InjectableInit()` | Code generator annotation — creates `di.config.dart` |
| `configureInjection()` | Called in `main.dart` before `runApp()` |

---

### 2. di.config.dart — Generated Registrations

<!-- AI_VERIFY: base_flutter/lib/di.config.dart -->
```
📁 lib/di.config.dart (AUTO-GENERATED)
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/di.config.dart`](../../base_flutter/lib/di.config.dart)

File này **DO NOT EDIT** — tự động generate bởi injectable. Code generator scan tất cả `@Injectable()` classes và tạo registrations qua `GetItHelper` extension.

**Actual generated content:**

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND
// **************************************************************************
// InjectableConfigGenerator
// **************************************************************************

part of 'di.dart';

extension GetItInjectableX on GetIt {
  Future<GetIt> init({
    String? environment,
    EnvironmentFilter? environmentFilter,
  }) async {
    final gh = GetItHelper(this, environment, environmentFilter);
    final serviceModule = _$ServiceModule();

    // SharedPreferences — async, pre-resolved
    await gh.factoryAsync<SharedPreferences>(
      () => serviceModule.prefs,
      preResolve: true,
    );

    // Firebase services — lazy singleton
    gh.lazySingleton<FirebaseFirestoreService>(
        () => FirebaseFirestoreService());
    gh.lazySingleton<FirebaseMessagingService>(
        () => FirebaseMessagingService());

    // API clients — lazy singleton (dependency order matters)
    gh.lazySingleton<UploadFileServerApiClient>(
        () => UploadFileServerApiClient());
    gh.lazySingleton<RawApiClient>(() => RawApiClient());
    gh.lazySingleton<RefreshTokenApiClient>(
        () => RefreshTokenApiClient());
    gh.lazySingleton<NoneAuthAppServerApiClient>(
        () => NoneAuthAppServerApiClient());
    gh.lazySingleton<RandomUserApiClient>(
        () => RandomUserApiClient());
    gh.lazySingleton<AuthAppServerApiClient>(
        () => AuthAppServerApiClient());

    // AppRouter — lazy singleton
    gh.lazySingleton<AppRouter>(() => AppRouter());

    // Helpers — lazy singleton
    gh.lazySingleton<ConnectivityHelper>(() => ConnectivityHelper());
    gh.lazySingleton<PackageHelper>(() => PackageHelper());
    gh.lazySingleton<PermissionHelper>(() => PermissionHelper());
    gh.lazySingleton<DeviceHelper>(() => DeviceHelper());
    gh.lazySingleton<CrashlyticsHelper>(() => CrashlyticsHelper());

    // Preferences — depends on SharedPreferences
    gh.lazySingleton<AppPreferences>(
        () => AppPreferences(gh<SharedPreferences>()));

    // Push notification helper — depends on PackageHelper
    gh.lazySingleton<LocalPushNotificationHelper>(
        () => LocalPushNotificationHelper(gh<PackageHelper>()));

    // Navigator — depends on AppRouter
    gh.lazySingleton<AppNavigator>(
        () => AppNavigator(gh<AppRouter>()));

    // Database — depends on AppPreferences
    gh.lazySingleton<AppDatabase>(
        () => AppDatabase(gh<AppPreferences>()));

    // AppApiService — depends on 3 API clients
    gh.lazySingleton<AppApiService>(() => AppApiService(
          gh<NoneAuthAppServerApiClient>(),
          gh<AuthAppServerApiClient>(),
          gh<UploadFileServerApiClient>(),
        ));

    // DeepLinkHelper — depends on AppNavigator + AppPreferences
    gh.lazySingleton<DeepLinkHelper>(() => DeepLinkHelper(
          gh<AppNavigator>(),
          gh<AppPreferences>(),
        ));

    return this;
  }
}

class _$ServiceModule extends ServiceModule {}
```

**Key structural observations:**
- Dependency order matters: `AuthAppServerApiClient` được đăng ký sau `RefreshTokenApiClient` vì nó phụ thuộc vào refresh token logic
- `preResolve: true` đảm bảo SharedPreferences được init **trước** khi bất kỳ service nào access nó
- Mỗi class chỉ được register **một lần** — injectable dedupes registrations

---

## Part B — Injectable Classes

### 3. @Injectable() Service Registration

<!-- AI_VERIFY: base_flutter/lib/data_source/api/app_api_service.dart -->
```
📁 lib/data_source/api/app_api_service.dart
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/data_source/api/app_api_service.dart`](../../base_flutter/lib/data_source/api/app_api_service.dart)

```dart
import 'package:dio/dio.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:injectable/injectable.dart';

import '../../../../index.dart';

final appApiServiceProvider = Provider<AppApiService>(
  (ref) => getIt.get<AppApiService>(),
);

@LazySingleton()
class AppApiService {
  AppApiService(
    this._noneAuthAppServerApiClient,
    this._authAppServerApiClient,
    this._uploadFileServerApiClient,
  );

  final NoneAuthAppServerApiClient _noneAuthAppServerApiClient;
  final AuthAppServerApiClient _authAppServerApiClient;
  final UploadFileServerApiClient _uploadFileServerApiClient;

  // Login — uses non-auth client
  Future<TokenAndRefreshTokenData> login({
    required String email,
    required String password,
  }) async {
    final result = await _noneAuthAppServerApiClient
        .request<TokenAndRefreshTokenData, DataResponse<TokenAndRefreshTokenData>>(
      method: RestMethod.post,
      path: 'v1/login',
      body: {'email': email.trim(), 'password': password},
      decoder: (json) =>
          TokenAndRefreshTokenData.fromJson(json.safeCast<Map<String, dynamic>>() ?? {}),
    );
    return result?.data ?? const TokenAndRefreshTokenData();
  }

  // Authenticated API calls — uses auth client
  Future<UserData> getMe() async {
    final response = await _authAppServerApiClient
        .request<UserData, DataResponse<UserData>>(
      method: RestMethod.get,
      path: 'v1/me',
      decoder: (json) => UserData.fromJson(json.safeCast<Map<String, dynamic>>() ?? {}),
    );
    return response?.data ?? const UserData();
  }

  Future<void> logout() async {
    await _authAppServerApiClient.request(method: RestMethod.post, path: 'v1/logout');
  }
}
```

**Key observations:**
- Constructor nhận **3 API clients** — chia theo authentication context:
  - `_noneAuthAppServerApiClient` cho login/forgot-password (không cần token)
  - `_authAppServerApiClient` cho tất cả authenticated calls (tự động inject token)
  - `_uploadFileServerApiClient` cho file uploads (khác endpoint)
- **Mỗi client tự quản lý interceptors riêng** — token injection không phải responsibility của `AppApiService`
- **`@LazySingleton()`** — service được tạo lần đầu tiên `getIt<AppApiService>()` được gọi, sau đó reused

---

### 4. @Injectable() + Environment

<!-- AI_VERIFY: base_flutter/lib/data_source/preference/app_preferences.dart -->
```
📁 lib/data_source/preference/app_preferences.dart
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/data_source/preference/app_preferences.dart`](../../base_flutter/lib/data_source/preference/app_preferences.dart)

```dart
import 'package:encrypted_shared_preferences/encrypted_shared_preferences.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:injectable/injectable.dart';

import '../../../index.dart';

final appPreferencesProvider = Provider((ref) => getIt.get<AppPreferences>());

@LazySingleton()
class AppPreferences {
  AppPreferences(this._sharedPreference)
      : _encryptedSharedPreferences =
            EncryptedSharedPreferences(prefs: _sharedPreference),
        _secureStorage = const FlutterSecureStorage(
          aOptions: AndroidOptions(encryptedSharedPreferences: true),
          iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
        );

  final SharedPreferences _sharedPreference;
  final EncryptedSharedPreferences _encryptedSharedPreferences;
  final FlutterSecureStorage _secureStorage;

  // Keys
  static const keyAccessToken = 'accessToken';
  static const keyRefreshToken = 'refreshToken';
  static const keyUserId = 'userId';
  static const keyIsLoggedIn = 'isLoggedIn';

  // Encrypted storage — access/refresh tokens (sensitive)
  Future<void> saveAccessToken(String token) async {
    await _encryptedSharedPreferences.setString(keyAccessToken, token);
  }

  Future<String> get accessToken {
    return _encryptedSharedPreferences.getString(keyAccessToken);
  }

  Future<void> saveRefreshToken(String token) async {
    await _encryptedSharedPreferences.setString(keyRefreshToken, token);
  }

  Future<String> get refreshToken {
    return _encryptedSharedPreferences.getString(keyRefreshToken);
  }

  // Plain storage — simple flags and IDs (non-sensitive)
  bool get isLoggedIn {
    return _sharedPreference.getBool(keyIsLoggedIn) ?? false;
  }

  Future<void> saveIsLoggedIn(bool isLoggedIn) async {
    await _sharedPreference.setBool(keyIsLoggedIn, isLoggedIn);
  }

  int get userId {
    return _sharedPreference.getInt(keyUserId) ?? -1;
  }

  Future<void> saveUserId(int userId) => _sharedPreference.setInt(keyUserId, userId);

  // Logout — clear all user data
  Future<void> clearCurrentUserData() async {
    await Future.wait([
      _sharedPreference.remove(keyAccessToken),
      _sharedPreference.remove(keyRefreshToken),
      _sharedPreference.remove(keyUserId),
      _sharedPreference.remove(keyIsLoggedIn),
    ]);
  }
}
```

**Security layers — 3-tier storage:**

| Tier | Storage | Data | Encryption |
|------|---------|------|------------|
| Sensitive | `EncryptedSharedPreferences` | Access/refresh tokens | Android encrypted storage |
| Sensitive | `FlutterSecureStorage` | Biometric-protected secrets | Android KeyStore / iOS Keychain |
| Non-sensitive | `SharedPreferences` | Flags, IDs | Plain storage |

**Key observations:**
- Token storage dùng `EncryptedSharedPreferences` — mã hóa AES, key lưu trong Android KeyStore
- iOS dùng `KeychainAccessibility.first_unlock` — token chỉ accessible sau khi device unlock
- **`_secureStorage` field không sử dụng** trong code hiện tại — dành cho future biometric-gated features
- `@LazySingleton()` đảm bảo chỉ một instance tồn tại xuyên suốt app lifecycle

**`@lazySingleton` vs `@singleton`:**

| Annotation | Behavior | Use case |
|-----------|---------|---------|
| `@singleton` | Created immediately on `configureInjection()` | Config, constants |
| `@lazySingleton` | Created on first `getIt<T>()` call | Heavy services (API, DB) |
| `@injectable` | New instance every `getIt<T>()` | Stateless services |

---

## Part C — Usage in Codebase

### 5. Injection in Riverpod Provider

<!-- AI_VERIFY: base_flutter/lib/ui/base/loading_state_provider.dart -->
```
📁 lib/ui/base/loading_state_provider.dart
```
<!-- END_VERIFY -->

> ⚠️ **Naming note:** Despite the `loading_state_provider.dart` filename, this is NOT a Riverpod Provider. It is an `InheritedWidget` pattern (`LoadingStateProvider`) that other widgets can access via `LoadingStateProvider.of(context)`. This is legacy naming from the codebase — the pattern is an InheritedWidget, not a Riverpod Provider.

→ [Mở file gốc: `lib/ui/base/loading_state_provider.dart`](../../base_flutter/lib/ui/base/loading_state_provider.dart)

```dart
// NOT a Riverpod provider — this is a Flutter InheritedWidget pattern
// for passing loading state down the widget tree without Riverpod

import 'package:flutter/material.dart';

class LoadingStateProvider extends InheritedWidget {
  const LoadingStateProvider({
    required this.isLoading,
    required super.child,
    super.key,
  });

  final bool isLoading;

  static LoadingStateProvider? maybeOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<LoadingStateProvider>();
  }

  static bool isLoadingOf(BuildContext context) {
    return maybeOf(context)?.isLoading ?? false;
  }

  @override
  bool updateShouldNotify(LoadingStateProvider oldWidget) {
    return isLoading != oldWidget.isLoading;
  }
}

// Usage in a page:
class HomePage extends StatefulWidget {
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return LoadingStateProvider(
      isLoading: _isLoading,
      child: Scaffold(
        body: _isLoading
            ? const Center(child: CircularProgressIndicator())
            : const HomeContent(),
        // In any child, check loading state:
        // LoadingStateProvider.isLoadingOf(context)
      ),
    );
  }
}
```

**Pattern comparison:**

| Pattern | Use Case | Complexity |
|---------|---------|-----------|
| `InheritedWidget` (this file) | Simple UI-only loading overlay | Low |
| `StateNotifier` + Riverpod | Shared loading state across pages | Medium |
| `getIt` + injectable (M17 pattern) | Service layer dependencies | N/A |

**Key observations:**
- `InheritedWidget` là Flutter pattern cơ bản nhất — không cần external packages
- `updateShouldNotify()` so sánh giá trị mới với cũ để quyết định có rebuild không
- `dependOnInheritedWidgetOfExactType()` đăng ký widget hiện tại làm dependent — Flutter tự động gọi `build()` khi state thay đổi
- **Không dùng cho service layer** — chỉ dùng cho UI state propagation đơn giản

---

### 6. AppInitializer — Bootstrap Coordinator

<!-- AI_VERIFY: base_flutter/lib/app_initializer.dart -->
```
📁 lib/app_initializer.dart
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/app_initializer.dart`](../../base_flutter/lib/app_initializer.dart)

```dart
import 'package:flutter/services.dart';
import 'index.dart';

class AppInitializer {
  const AppInitializer._();

  static Future<void> init() async {
    Env.init();                                   // Load .env
    await configureInjection();                    // ← DI setup — được gọi từ đây
    await getIt.get<PackageHelper>().init();      // Package info init
    await SystemChrome.setPreferredOrientations(  // Device orientation
      getIt.get<DeviceHelper>().deviceType == DeviceType.phone
          ? Constant.mobileOrientation
          : Constant.tabletOrientation,
    );
    // Edge to Edge
    await SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
  }
}
```

**Why AppInitializer exists:**
- Giữ `main.dart` clean — bootstrap logic được tách riêng
- Tất cả async initialization ở một chỗ
- Dễ test — có thể mock `AppInitializer.init()`

```dart
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
  await AppInitializer.init();         // ← configureInjection() called INSIDE here
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

**Bootstrap flow — chi tiết:**

```
main()
  └── runZonedGuarded()              // Global error handler
        └── _runMyApp()              // Async entry
              ├── WidgetsFlutterBinding.ensureInitialized()  // Flutter ready
              ├── FlutterNativeSplash.preserve()            // Splash screen
              ├── Firebase.initializeApp()                   // Firebase init
              ├── AppInitializer.init()                      // ← KEY: DI setup
              │     ├── Env.init()                           // Load .env config
              │     ├── configureInjection()                 // get_it registrations
              │     ├── PackageHelper.init()                 // Package info
              │     └── SystemChrome.setPreferredOrientations()
              └── runApp(ProviderScope(...))  // Start Flutter
```

**Critical timing:**
- `AppInitializer.init()` gọi `configureInjection()` bên TRONG — DI setup xảy ra ở step 4, trước `runApp()`
- **ĐÚNG:** `configureInjection()` phải xảy ra sau tất cả sync init nhưng trước `runApp()`
- Lý do: `getIt` được sử dụng trong `ProviderScope` observers và `appPreferencesProvider`

---

## File Map — Quick Reference

| File | Lines | Role | Pattern |
|------|-------|------|---------|
| `lib/di.dart` | ~16 | Service locator setup + injectable init | Module + InjectableInit |
| `lib/di.config.dart` | 127 | Generated registrations | DO NOT EDIT |
| `lib/app_initializer.dart` | ~20 | Bootstrap coordinator — gọi `configureInjection()` | Sequential async init |
| `lib/main.dart` | ~37 | App entry point, `runZonedGuarded` | Error boundary + Firebase + AppInitializer |
| `@LazySingleton()` classes | varies | Business services (API, storage, Firebase) | Constructor injection |
| `@module` class | ~12 | Async dependency (SharedPreferences) | @preResolve pattern |

---

## Architecture Question

**Q: Tại sao dự án dùng `get_it` (service locator) thay vì Riverpod Provider cho DI?**

Riverpod providers QUAN TRỌNG cho UI state management. Nhưng injectable đặc biệt phù hợp cho **service layer dependencies** (API, storage, config) vì:
1. Constructor injection — dependencies visible in constructor signature
2. Codegen tự động — không cần viết manual provider factories
3. `@preResolve` cho async initialization (SharedPreferences)
4. Lazy singleton cho heavy services

Trong thực tế, nhiều project **chỉ dùng Riverpod** cho DI hoàn toàn. Dự án này dùng **cả hai**: injectable cho services, Riverpod cho UI state.

---

> **Tiếp theo:** [02-concept.md](./02-concept.md) — phân tích 6 concepts đằng sau DI architecture.

<!-- AI_VERIFY: generation-complete -->
