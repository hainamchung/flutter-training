# Code Walk — Advanced Performance & Security

> 📌 **Recap từ modules trước:**
> - **M1:** App entrypoint, `WidgetsFlutterBinding` — platform flags và initialization ([M1 § Entrypoint](../module-01-app-entrypoint/01-code-walk.md))
> - **M12:** `AppApiService`, Dio — network layer cho SSL pinning ([M12 § API](../module-12-data-layer/01-code-walk.md))
> - **M14:** `AppPreferences`, `FlutterSecureStorage` — secure token storage ([M14 § Storage](../module-14-local-storage/01-code-walk.md))
> - **M23:** Performance — profiling, DevTools, optimization ([M23 § Performance](../module-23-performance/01-code-walk.md))
>
> Nếu chưa nắm vững → quay lại module tương ứng trước.

---

## Walk Order

```
app_preferences.dart (EncryptedSharedPreferences — secure token storage)
    ↓
app_api_service.dart (Dio + SSL pinning — secure network)
    ↓
main.dart (Debug flags, environment config — debug detection)
    ↓
android/app/build.gradle (R8 configuration — obfuscation)
    ↓
Info.plist (iOS security configs — platform-specific)
```

Security patterns (storage → network → detection) → platform-specific configurations.

---

## 1. Secure Storage — app_preferences.dart

<!-- AI_VERIFY: base_flutter/lib/data_source/preference/app_preferences.dart -->

> 💡 **FE Perspective**
> **Flutter:** `FlutterSecureStorage` encrypts data using platform-native encryption: Keychain on iOS, EncryptedSharedPreferences (wrapping KeyStore) on Android.
> **React/Vue tương đương:** `sessionStorage` với `crypto.subtle` encryption, hoặc `localStorage` với client-side encryption.
> **Khác biệt quan trọng:** Flutter secure storage uses platform-native encryption (Keychain/KeyStore), không phải JavaScript-based.

> ⚠️ **Implementation note:** The actual `base_flutter` project uses `EncryptedSharedPreferences` (AndroidX Security) for token storage, not `FlutterSecureStorage` directly. `FlutterSecureStorage` is also present in the codebase but as a secondary/legacy option. The teaching pattern above shows `FlutterSecureStorage` for clarity — see the actual `app_preferences.dart` for the real implementation.

### Structural Overview

```dart
// app_preferences.dart — Secure storage với platform-specific options
class AppPreferences {
  late final FlutterSecureStorage _secureStorage;
  
  AppPreferences() {
    _secureStorage = const FlutterSecureStorage(
      aOptions: AndroidOptions(
        encryptedSharedPreferences: true,  // EncryptedSharedPreferences
      ),
      iOptions: IOSOptions(
        accessibility: KeychainAccessibility.first_unlock,  // Keychain
      ),
    );
  }
  
  Future<void> saveAccessToken(String token) async {
    await _secureStorage.write(key: _keyAccessToken, value: token);
  }
  
  Future<String?> getAccessToken() async {
    return await _secureStorage.read(key: _keyAccessToken);
  }
}
```

### Android Configuration

```dart
// AndroidOptions:
// - encryptedSharedPreferences: true → dùng EncryptedSharedPreferences (AndroidX Security)
//   Đây là best practice cho Android 10+ (API 23+)
// - Các tùy chọn khác:
//   - sharedPreferencesName: tên file preferences
//   - preferencesKeyPrefix: prefix cho keys
```

**Tại sao `encryptedSharedPreferences: true` quan trọng:**
- Mặc định Android SharedPreferences lưu plaintext (có thể đọc bằng root access)
- `EncryptedSharedPreferences` wrap values với AES-256-GCM encryption
- Key material được bảo vệ bởi Android Keystore (hardware-backed trên Android 6.0+)

### iOS Configuration

```dart
// IOSOptions:
// - accessibility: KeychainAccessibility.first_unlock
//   → Keychain item có thể đọc sau khi device unlock lần đầu
//   → Phù hợp cho tokens cần đọc khi app launch
//
// Các tùy chọn khác:
// - accountName: Keychain account name
// - service: Keychain service identifier
```

**Keychain Accessibility Levels:**

| Level | When accessible | Use case |
|-------|-----------------|----------|
| `first_unlock` | Sau device unlock lần đầu | Tokens cần khi app launch |
| `after_first_unlock` | Sau unlock lần đầu, persist reboot | Background refresh tokens |
| `when_unlocked` | Chỉ khi unlocked | Sensitive data (banking apps) |
| `when_passcode_set_this_device_only` | Khi có passcode | Highest security |

---

## 2. Secure Network — app_api_service.dart (SSL Pinning)

<!-- AI_VERIFY: base_flutter/lib/data_source/api/app_api_service.dart -->

> 💡 **FE Perspective**
> **Flutter:** Dio với custom `TrustedCertificates` cho SSL pinning.
> **React/Vue tương đương:** Fetch với `rejectUnauthorized: true` hoặc certificate pinning library.
> **Khác biệt quan trọng:** Web TLS pinning không phổ biến (CORS restrictions). Mobile apps nên dùng pinning.

### SSL Pinning với Dio

### SSL Pinning — Note on Actual Implementation

> ⚠️ **Important correction:** The code blocks above are **teaching patterns only**. The actual `base_flutter` project **does NOT implement custom SSL pinning** via Dio interceptors. Instead, it uses:

**1. Dio with standard interceptors (token injection, retry, connectivity):**

```dart
// ACTUAL: lib/data_source/api/app_api_service.dart — NO custom SSL pinning
// Dio is configured with interceptors for auth, retry, connectivity
// SSL/TLS is handled at the platform level (ATS on iOS, network security config on Android)

class AppApiService {
  AppApiService(
    this._noneAuthAppServerApiClient,
    this._authAppServerApiClient,
    this._uploadFileServerApiClient,
  );
  // Uses 3 separate API clients with different auth interceptors
  // NOT a single Dio instance with SSL pinning interceptor
}
```

**2. iOS App Transport Security (ATS):**

```xml
<!-- ACTUAL: ios/Runner/Info.plist -->
<!-- ATS is enabled by default — all connections must use HTTPS -->
<!-- Custom domains can be whitelisted if needed -->
```

**3. Android Network Security Config:**

```xml
<!-- ACTUAL: android/app/src/main/res/xml/network_security_config.xml -->
<!-- Configures trusted CAs and domain-specific policies -->
```

**Why no custom SSL pinning in base_flutter:**
- Flutter apps on iOS/Android benefit from platform-level TLS enforcement
- Firebase SDK handles its own secure connections
- Certificate rotation is complex on mobile — requires app update
- The project relies on platform security rather than application-layer pinning

**If you need SSL pinning in a production banking/finance app:**

```dart
// Conceptual pattern (NOT implemented in base_flutter):
// Use a custom Dart FFI approach or platform channels
// Most production apps use third-party packages like:
//   - flutter_ssl_pinning
//   - dart_sertificate_pinning
// Or delegate to native code via MethodChannel

// Dio does NOT have a built-in onSSLRequest interceptor callback.
// SSL pinning in Dio requires:
// 1. Custom HttpClientAdapter with BadCertificateCallback
// 2. Or platform-specific implementation via MethodChannel
```

### Certificate Extraction

**Cách lấy certificate fingerprint cho pinning:**

```bash
# Lấy certificate từ server
openssl s_client -connect api.example.com:443 </dev/null 2>/dev/null | \
  openssl x509 -outform DER | \
  openssl base64 -A

# SHA-256 fingerprint
openssl s_client -connect api.example.com:443 </dev/null 2>/dev/null | \
  openssl x509 -fingerprint -sha256 -noout -in /dev/stdin
```

### Multiple Certificate Pins (Backup Pins)

```dart
// Best practice: Pin nhiều certificates (primary + backup)
class SSLPinningInterceptor extends Interceptor {
  // ⚠️ WARNING: Replace these placeholder hashes with actual certificate hashes from your server!
  // Run: openssl s_client -connect api.example.com:443 </dev/null 2>/dev/null | openssl x509 -fingerprint -sha256 -noout
  static const _pinnedCertificates = {
    // Primary certificate — ⚠️ REPLACE WITH ACTUAL HASH
    'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',
    // Backup certificate — ⚠️ REPLACE WITH ACTUAL HASH
    'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=',
  };
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final securityContext = SecurityContext();
    // Add pinned certificates to context
    for (final cert in _pinnedCertificates) {
      securityContext.setTrustedCertificatesBytes(
        base64Decode(cert),
      );
    }
    
    handler.next(options);
  }
}
```

### When to Use SSL Pinning

| Scenario | SSL Pinning? | Reason |
|----------|--------------|--------|
| Banking / Finance | ✅ Strongly recommended | High security requirement |
| Healthcare / PHI | ✅ Recommended | HIPAA compliance |
| E-commerce | 🟡 Optional | Moderate security |
| Internal tools | 🟢 Optional | Lower risk |
| Development / Staging | ❌ Not needed | Certificates change often |

---

## 3. Debug Detection — main.dart (Teaching Pattern)

> ⚠️ **IMPORTANT:** The code blocks below are **TEACHING PATTERNS ONLY**. The actual `base_flutter` project does **NOT** have a `SecurityHelper` class in `main.dart`. These patterns are for learning purposes only.

<!-- TEACHING PATTERN: SecurityHelper class does NOT exist in main.dart — this is a pedagogical example -->
<!-- AI_VERIFY: base_flutter/lib/main.dart -->
<!-- NOTE: main.dart is 37 lines only - NO SecurityHelper class exists -->

> 💡 **FE Perspective**
> **Flutter:** Platform flags để detect debug mode, emulator detection.
> **React/Vue tương đương:** `process.env.NODE_ENV`, `window.__DEV__`, platform detection.
> **Khác biệt quan trọng:** Flutter có nhiều debug indicators (k-mode, debugger attached, etc.).

### Debug Mode Detection

```dart
// main.dart — Debug detection và security checks
import 'package:flutter/foundation.dart' show kDebugMode;
import 'package:flutter/services.dart' show Platform;

class SecurityHelper {
  /// Check if app running in debug mode
  static bool get isDebugMode => kDebugMode;
  
  /// Check if running on emulator/simulator
  static bool get isEmulator {
    if (Platform.isAndroid) {
      return !_checkPlayServices();
    }
    if (Platform.isIOS) {
      return _checkIOSSimulator();
    }
    return false;
  }
  
  /// Check if device is rooted/jailbroken
  static bool get isDeviceSecure {
    if (Platform.isAndroid) {
      return _checkRootedAndroid();
    }
    if (Platform.isIOS) {
      return _checkJailbrokeniOS();
    }
    return true; // Assume secure if can't check
  }
  
  /// Combined security status
  static SecurityStatus get securityStatus {
    if (isDebugMode) return SecurityStatus.unsafe;
    if (isEmulator) return SecurityStatus.warning;
    if (!isDeviceSecure) return SecurityStatus.critical;
    return SecurityStatus.safe;
  }
}

enum SecurityStatus { safe, warning, unsafe, critical }
```

### Android Root Detection

```dart
// Android-specific root detection
Future<bool> _checkRootedAndroid() async {
  // Check for common root-related files
  final rootFiles = [
    '/system/app/Superuser.apk',
    '/sbin/su',
    '/system/bin/su',
    '/system/xbin/su',
    '/data/local/xbin/su',
    '/data/local/bin/su',
    '/system/sd/xbin/su',
    '/system/bin/failsafe/su',
    '/data/local/su',
  ];
  
  for (final path in rootFiles) {
    if (await File(path).exists()) return true;
  }
  
  // Check for Magisk
  final magiskPaths = [
    '/sbin/.magisk',
    '/data/adb/magisk',
  ];
  
  for (final path in magiskPaths) {
    if (await Directory(path).exists()) return true;
  }
  
  return false;
}
```

### iOS Jailbreak Detection

```dart
// iOS-specific jailbreak detection
Future<bool> _checkJailbrokeniOS() async {
  // Check for Cydia
  final cydiaPath = '/Applications/Cydia.app';
  if (await File(cydiaPath).exists()) return true;
  
  // Check for common jailbreak packages
  final jailbreakPaths = [
    '/Library/MobileSubstrate/MobileSubstrate.dylib',
    '/bin/bash',
    '/usr/sbin/sshd',
    '/etc/apt',
    '/private/var/lib/apt/',
  ];
  
  for (final path in jailbreakPaths) {
    if (await File(path).exists() || await Directory(path).exists()) return true;
  }
  
  // Try to write outside sandbox
  try {
    await File('/private/jailbreak_test.txt').writeAsString('test');
    return true; // Successfully wrote outside sandbox
  } catch (_) {
    return false;
  }
}
```

### Using Debug Detection

```dart
// Usage trong app
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Security check on startup
  final status = SecurityHelper.securityStatus;
  if (status == SecurityStatus.critical) {
    // Show warning, block sensitive operations
    runApp(const BlockedApp());
  } else if (status == SecurityStatus.unsafe && !EnvConfig.allowDebugMode) {
    // Block app in production builds
    runApp(const ProductionOnlyApp());
  } else {
    runApp(const MyApp());
  }
}
```

---

## 4. Code Obfuscation — Android R8 Configuration

<!-- AI_VERIFY: base_flutter/android/app/build.gradle -->

> 💡 **FE Perspective**
> **Flutter:** R8 (Android) minifies và obfuscates Dart code trong release builds.
> **React/Vue tương đương:** Webpack `mode: production` với TerserPlugin obfuscation.
> **Khác biệt quan trọng:** Flutter obfuscation affects cả Dart + native code. Webpack chỉ affects JS.

### build.gradle Configuration

```gradle
// android/app/build.gradle
android {
    buildTypes {
        release {
            // R8 minification và obfuscation
            minifyEnabled true
            shrinkResources true  // Loại bỏ unused resources
            
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        
        debug {
            minifyEnabled false
            // Debug builds không obfuscate để dễ debug
        }
    }
}
```

### proguard-rules.pro

```prolog
# proguard-rules.pro — R8/ProGuard rules cho Flutter

# Flutter specific
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.** { *; }
-keep class io.flutter.util.** { *; }
-keep class io.flutter.view.** { *; }
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

# Keep main app classes
-keep class com.example.myapp.** { *; }

# Generics (Freezed, JSON Serializable)
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.example.myapp.model.** { *; }

# Dio
-dontwarn okhttp3.**
-dontwarn okio.**
-keep class okhttp3.** { *; }
-keep interface okhttp3.** { *; }

# Flutter Secure Storage
-keep class com.it_nomads.fluttersecurestorage.** { *; }

# Firebase
-keep class com.google.firebase.** { *; }
-keepattributes EnclosingMethod
-keepattributes InnerClasses

# Obfuscation
-repackageclasses ''
-obfuscationdictionary obfuscation_dictionaries.txt
-classobfuscationdictionary obfuscation_dictionaries.txt
```

### Obfuscation Dictionary

```text
# obfuscation_dictionaries.txt — Words used for obfuscation
# Avoids using common class/method names

a
b
c
d
e
f
g
h
i
j
k
l
m
n
o
p
q
r
s
t
u
v
w
x
y
z

get
set
init
load
save
fetch
post
put
delete

data
model
state
config
helper
service
repository
```

### iOS Obfuscation (Symbol Stripping)

```bash
# Trong Xcode Build Settings:
# Strip Debug Symbols During Copy: YES
# Strip Linked Product: YES (Release only)

# Hoặc via command line:
xcodebuild PRODUCT_NAME=MyApp \
  STRIP_INSTALLED_PRODUCT=YES \
  STRIP_STYLE=Non-GlobalSymbols
```

---

## 5. iOS Security Configuration — Info.plist (Teaching Pattern)

> ⚠️ **IMPORTANT:** The code blocks below are **TEACHING PATTERNS ONLY**. The actual `base_flutter/ios/Runner/Info.plist` does **NOT** contain `NSAppTransportSecurity` block or Vietnamese strings. These patterns are for learning purposes only.

<!-- TEACHING PATTERN: NSAppTransportSecurity does NOT exist in actual Info.plist — this is a pedagogical example -->
<!-- AI_VERIFY: base_flutter/ios/Runner/Info.plist -->
<!-- NOTE: Actual Info.plist does NOT contain NSAppTransportSecurity or NSFaceIDUsageDescription -->

### Key Security Configurations

```xml
<!-- Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
    <!-- Chỉ cho phép HTTPS -->
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <!-- Exceptions nếu cần -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>api.example.com</key>
        <dict>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <false/>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <false/>
        </dict>
    </dict>
</dict>

<!-- Biometric Authentication -->
<key>NSFaceIDUsageDescription</key>
<string>Chúng tôi sử dụng Face ID để bảo mật đăng nhập</string>

<!-- Location (nếu cần) -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Ứng dụng cần vị trí để hiển thị nội dung gần bạn</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Ứng dụng cần vị trí để gửi thông báo khi có cập nhật</string>
```

---

## Summary — Security & Performance Patterns

| Layer | File | Pattern | Purpose |
|-------|------|---------|---------|
| Storage | `app_preferences.dart` | `FlutterSecureStorage` | Encrypted token storage |
| Network | `app_api_service.dart` | SSL Pinning | Prevent MITM attacks |
| Detection | `main.dart` | Platform flags | Detect debug/emulator/root |
| Android | `build.gradle` | R8 minification | Code obfuscation |
| iOS | `Info.plist` | Transport Security | Enforce HTTPS |

### Key Takeaway

Security hardening và performance optimization là **2 concerns riêng biệt** nhưng thường đi cùng nhau trong production apps. Security ngăn chặn attacks; Performance đảm bảo app chạy mượt. Cả 2 đều cần được implement **từ đầu**, không phải sau khi có users.

> 💡 **FE Perspective Summary:**
> | Flutter | Frontend |
> |---------|----------|
> | R8/ProGuard obfuscation | Webpack production minification |
> | SSL Pinning | TLS certificate pinning (rare in web) |
> | `FlutterSecureStorage` | Encrypted sessionStorage |
> | Debug detection | `process.env.NODE_ENV` check |
> | Performance monitoring | `PerformanceObserver` API |

<!-- AI_VERIFY: generation-complete -->
