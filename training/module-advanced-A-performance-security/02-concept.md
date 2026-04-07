# Concepts — Advanced Performance & Security

> 📌 **Module context:** Module này survey 6 concepts chính về security hardening và performance optimization trong Flutter, mapping từ code đã đọc ở [01-code-walk](./01-code-walk.md). Mỗi concept kèm FE bridge cho dev có background React/JavaScript.

---

## Concept 1: Code Obfuscation — R8/ProGuard/Dart Obfuscate 🔴 MUST-KNOW

**WHY:** Obfuscation bảo vệ intellectual property và makes reverse engineering khó hơn. Production apps PHẢI obfuscate.

### How Obfuscation Works

```
┌─────────────────────┐
│  Source Code        │
│  class _A {         │
│    void _m() {...}  │
│  }                  │
└─────────┬───────────┘
          │ R8/ProGuard/Dart Obfuscator
          ▼
┌─────────────────────┐
│  Obfuscated Code    │
│  class a {          │
│    void b() {...}   │
│  }                  │
└─────────────────────┘
```

### Obfuscation Levels

| Level | Effect | Use Case |
|-------|--------|----------|
| **Minification** | Remove whitespace, shorten names | All production builds |
| **Renaming** | `MyClass` → `a`, `myMethod` → `b` | Release builds |
| **Tree-shaking** | Remove unused code | `minifyEnabled: true` |
| **String encryption** | Encrypt hardcoded strings | High-security apps |

### Flutter/Dart Obfuscation Configuration

```dart
// pubspec.yaml — Flutter build configuration
flutter:
  # Obfuscation được enable qua --obfuscate flag
  # dart compile js --obfuscate (web)
  # flutter build apk --obfuscate --split-debug-info=... (mobile)
```

```bash
# Build với obfuscation
flutter build apk --release \
  --obfuscate \
  --split-debug-info=./build/debug-info
```

### Android R8 Configuration

```gradle
// android/app/build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true          // Enable R8
            shrinkResources true       // Remove unused resources
            useProguard true           // Enable ProGuard rules
            
            proguardFiles getDefaultProguardFile(
                'proguard-android-optimize.txt'
            ), 'proguard-rules.pro'
        }
    }
}
```

### What to Keep (Don't Obfuscate)

```prolog
# proguard-rules.pro
# Classes được reflect (Firebase, JSON serializable)
-keepattributes Signature
-keepattributes *Annotation*

# Keep Freezed/JSON serializable models
-keep class com.example.app.model.** { *; }

# Keep Firebase classes
-keep class com.google.firebase.** { *; }

# Keep Native callbacks (platform channels)
-keep class com.example.app.** {
    native <methods>;
}
```

> 💡 **FE Perspective**
> **Flutter:** R8/ProGuard minify và obfuscate cả Dart VM instructions và native code.
> **React/Vue tương đương:** Webpack `mode: production` + TerserPlugin minification.
> **Khác biệt quan trọng:** Flutter obfuscation cần native config (R8 rules). Webpack chỉ cần JS config.

---

## Concept 2: SSL Certificate Pinning 🟡 SHOULD-KNOW

> ⚠️ **Prerequisite:** SSL pinning requires custom platform channel implementation (MethodChannel/FlutterEventChannel) which is covered in [M20 — Native Platforms](../module-20-native-platforms/). Complete M20 before attempting the SSL Pinning exercise in Section 3.

**WHY:** SSL Pinning prevents man-in-the-middle attacks even if attacker has valid certificate. Critical cho apps handling sensitive data.

### How SSL Pinning Works

```
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Client     │          │   Attacker   │          │   Server     │
│  (App)       │          │  (MITM)      │          │  (API)       │
└──────┬───────┘          └──────┬───────┘          └──────┬───────┘
       │  1. Connect             │                          │
       │─────────────────────────│                          │
       │                         │  2. Fake cert            │
       │                         │◄─────────────────────────│
       │  3. Check pin           │                          │
       │  ❌ Pin mismatch!       │                          │
       │  Connection REJECTED    │                          │
       └─────────────────────────┘                          │
```

### Pinning Methods

| Method | Pros | Cons | Recommended |
|--------|------|------|-------------|
| **Certificate Pinning** | Simple | Rotation requires app update | ✅ Best practice |
| **Public Key Pinning** | Rotation possible (same key) | Complex setup | 🟡 Advanced |
| **CA Pinning** | Broad protection | Less specific | 🟢 Not recommended |

### Implementation với Dio

```dart
// SSL Pinning với custom SecurityContext
class SSLPinningService {
  static const _pinnedPublicKeys = [
    // SHA-256 Base64 encoded public key
    'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',
    // Backup pin (for key rotation)
    'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=',
  ];
  
  Dio createPinnedDio() {
    final dio = Dio();
    
    dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final validator = await _createValidator();
        options.validateStatus = (status) => status != null && status < 500;
        handler.next(options);
      },
    ));
    
    return dio;
  }
  
  Future<bool> validateCertificate(X509Certificate cert) async {
    final publicKeyHash = _hashPublicKey(cert.der);
    return _pinnedPublicKeys.contains(publicKeyHash);
  }
  
  String _hashPublicKey(List<int> der) {
    final hash = sha256.convert(der);
    return 'sha256/${base64.encode(hash.bytes)}';
  }
}
```

### Certificate Rotation Strategy

```dart
// Backup pin strategy cho certificate rotation
class CertificateRotationPolicy {
  /// Primary certificate - phải match
  static const _primaryPin = 'sha256/PRIMARY_CERT_HASH=';
  
  /// Backup certificate - cho phép trong transition period
  /// Transition period: 30-60 days sau khi certificate rotated
  static const _backupPin = 'sha256/BACKUP_CERT_HASH=';
  
  /// Emergency fallback - chỉ enable khi có incident
  /// Không commit vào source control
  static const _emergencyPin = 'sha256/EMERGENCY_CERT_HASH=';
  
  bool isValidCertificate(X509Certificate cert) {
    final hash = _getPublicKeyHash(cert);
    return hash == _primaryPin || hash == _backupPin;
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** SSL Pinning được implement ở network layer (Dio). Native APIs handle validation.
> **React/Vue tương đương:** Web TLS Certificate Pinning (hầu như không dùng do CORS).
> **Khác biệt quan trọng:** Mobile apps có full control. Web bị giới hạn bởi browser security model.

> ⚠️ **Reality check:** Custom SSL pinning via Dio interceptors is NOT straightforward. `Dio` does NOT have a built-in `onSSLRequest` callback. True SSL pinning requires:
> 1. Custom `HttpClientAdapter` with `BadCertificateCallback`
> 2. OR platform-specific implementation via MethodChannel (covered in M20)
> 3. OR third-party packages like `flutter_ssl_pinning`
> The base_flutter project relies on platform-level TLS enforcement instead of application-layer pinning.

---

## Concept 3: Secure Storage Best Practices 🟡 SHOULD-KNOW

**WHY:** Sensitive data (tokens, API keys, user info) PHẢI được encrypt. Plain storage = security risk.

### Storage Options

| Storage | Encryption | Use Case | Capacity |
|---------|-------------|----------|----------|
| `FlutterSecureStorage` | AES-256 (Keychain/KeyStore) | Tokens, secrets | Small (< 1MB) |
| `shared_preferences` | None (plaintext) | Settings, flags | Small |
| SQLite (encrypted) | SQLCipher | Large structured data | Large |
| `flutter_secure_storage` + Hive | Hybrid | Complex + secure | Large |

### Secure Storage Patterns

```dart
// Pattern 1: Token Storage
class SecureTokenStorage {
  final FlutterSecureStorage _storage;
  
  // Keys constants
  static const _keyAccessToken = 'access_token';
  static const _keyRefreshToken = 'refresh_token';
  static const _keyTokenExpiry = 'token_expiry';
  
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
    required DateTime expiry,
  }) async {
    await Future.wait([
      _storage.write(key: _keyAccessToken, value: accessToken),
      _storage.write(key: _keyRefreshToken, value: refreshToken),
      _storage.write(key: _keyTokenExpiry, value: expiry.toIso8601String()),
    ]);
  }
  
  Future<TokenPair?> getTokens() async {
    final access = await _storage.read(key: _keyAccessToken);
    final refresh = await _storage.read(key: _keyRefreshToken);
    final expiryStr = await _storage.read(key: _keyTokenExpiry);
    
    if (access == null || refresh == null || expiryStr == null) {
      return null;
    }
    
    return TokenPair(
      accessToken: access,
      refreshToken: refresh,
      expiry: DateTime.parse(expiryStr),
    );
  }
  
  Future<void> clearTokens() async {
    await Future.wait([
      _storage.delete(key: _keyAccessToken),
      _storage.delete(key: _keyRefreshToken),
      _storage.delete(key: _keyTokenExpiry),
    ]);
  }
}
```

```dart
// Pattern 2: Encrypted Hive for complex data
class SecureHiveStorage {
  static Future<Box<dynamic>> openEncryptedBox(String name) async {
    final key = await _getOrCreateEncryptionKey();
    return await Hive.openBox(
      name,
      encryptionCipher: HiveAesCipher(key),
    );
  }
  
  static Future<List<int>> _getOrCreateEncryptionKey() async {
    final secureStorage = FlutterSecureStorage();
    var key = await secureStorage.read(key: 'hive_encryption_key');
    
    if (key == null) {
      key = base64Encode(Hive.generateSecureKey());
      await secureStorage.write(key: 'hive_encryption_key', value: key);
    }
    
    return base64Decode(key);
  }
}
```

### Android KeyStore Configuration

```dart
// Advanced Android KeyStore configuration
final androidOptions = AndroidOptions(
  encryptedSharedPreferences: true,
  // Preferences backup không được enable trong secure storage
  // vì có thể leak sensitive data qua Google Backup
  backupEnabled: false,
  // Mỗi key được bảo vệ bởi individual keys trong Android KeyStore
  keyCipherAlgorithm: KeyCipherAlgorithm.RSA_ECB_PKCS1Padding,
  valueCipherAlgorithm: ValueCipherAlgorithm.AES_GCM_NoPadding,
);
```

### iOS Keychain Configuration

```dart
// Advanced iOS Keychain configuration
final iosOptions = IOSOptions(
  accessibility: KeychainAccessibility.first_unlock_this_device_only,
  // Synchronization: không sync qua iCloud Keychain
  // vì có thể leak sensitive data qua cloud backup
  synchronizable: false,
);
```

> 💡 **FE Perspective**
> **Flutter:** `FlutterSecureStorage` sử dụng platform-native encryption (Keychain/KeyStore).
> **React/Vue tương đương:** `crypto.subtle` API for encryption, `sessionStorage` (in-memory), `localStorage` (plaintext).
> **Khác biệt quan trọng:** Mobile native encryption APIs mạnh hơn web Storage APIs.

---

## Concept 4: API Key Protection 🔴 MUST-KNOW

**WHY:** Hardcoded API keys = security vulnerability. Keys có thể bị extracted từ APK/IPA.

### Protection Methods

| Method | Security | Ease of Use | Recommended |
|--------|----------|-------------|-------------|
| **Environment variables** | 🟢 High | Medium | ✅ Best practice |
| **Remote config** | 🟡 Medium | Easy | 🟡 For dynamic keys |
| **Backend proxy** | 🟢 Highest | Hard | ✅ Best for production |
| **Hardcoded** | 🔴 None | Easy | ❌ Never do this |
| **Version control** | 🔴 None | Easy | ❌ Never do this |

### Environment Variables Pattern

```dart
// lib/config/env_config.dart
class EnvConfig {
  // API keys từ environment (flutter run --dart-define=API_KEY=xxx)
  // hoặc CI/CD secrets
  static String get apiKey => const String.fromEnvironment(
    'API_KEY',
    defaultValue: '',
  );
  
  static String get apiBaseUrl => const String.fromEnvironment(
    'API_BASE_URL',
    defaultValue: 'https://api.example.com',
  );
  
  // Debug mode check
  static bool get allowDebugMode => const bool.fromEnvironment(
    'ALLOW_DEBUG_MODE',
    defaultValue: false,
  );
}
```

### Build Configuration

```bash
# Development
flutter run --dart-define=API_KEY=dev_api_key_xxx \
           --dart-define=API_BASE_URL=https://dev-api.example.com

# Staging
flutter run --dart-define=API_KEY=staging_api_key_xxx \
           --dart-define=API_BASE_URL=https://staging-api.example.com

# Production (CI/CD injects secrets)
flutter build apk --release \
  --dart-define=API_KEY=$PROD_API_KEY \
  --dart-define=API_BASE_URL=$PROD_API_URL
```

### CI/CD Secret Management

```yaml
# GitHub Actions example
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
      
      - name: Decode keystore
        run: echo $ANDROID_KEYSTORE | base64 -d > release.keystore
      
      - name: Build APK
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}
          API_BASE_URL: ${{ secrets.PROD_API_URL }}
        run: |
          flutter build apk --release \
            --dart-define=API_KEY=$API_KEY \
            --dart-define=API_BASE_URL=$API_BASE_URL \
            --keyalias=$KEY_ALIAS \
            --keystore=release.keystore
```

### Backend Proxy (Best Practice for Production)

```dart
// NEVER: Direct API call với exposed key
// ❌ const apiKey = 'sk_live_xxxxx'; 
// ❌ dio.get('https://api.payment.com?api_key=$apiKey');

// INSTEAD: Backend proxy
// ✅ App gọi backend proxy của mình
dio.get('https://my-backend.com/api/payment/init');

// Backend proxy gọi payment API với server-side key
// Payment API key được store trong backend environment variables
```

> 💡 **FE Perspective**
> **Flutter:** `const String.fromEnvironment()` cho compile-time constants, CI/CD secrets injection.
> **React/Vue tương đương:** `.env` files với `process.env.REACT_APP_*`, CI/CD secrets.
> **Khác biệt quan trọng:** Flutter dùng `--dart-define` thay vì `.env`. Keys được compile vào binary.

---

## Concept 5: Debug Mode Detection 🟡 SHOULD-KNOW

**WHY:** Detect debug environment giúp enforce security policies và prevent sensitive operations trong development.

### Detection Flags

```dart
// Debug detection helpers
class DebugDetector {
  /// Flutter debug mode
  /// kDebugMode = true khi flutter run (debug mode)
  /// kDebugMode = false khi flutter build apk/ipa (release mode)
  static bool get isDebugMode => kDebugMode;
  
  /// Dart assertion mode
  /// Assertions enabled = debug mode
  static bool get isAssertMode => kAssertsEnabled;
  
  /// Simulator/Emulator detection
  static bool get isSimulator {
    if (Platform.isIOS) {
      return _checkIOSSimulator();
    }
    if (Platform.isAndroid) {
      return _checkAndroidEmulator();
    }
    return false;
  }
  
  /// Root/Jailbreak detection
  static bool get isDeviceCompromised {
    if (Platform.isAndroid) return _isAndroidRooted();
    if (Platform.isIOS) return _isJailbroken();
    return false;
  }
  
  /// Combined check for production safety
  static bool get isSafeForProduction {
    return !isDebugMode && !isSimulator && !isDeviceCompromised;
  }
}
```

### Security Enforcement

```dart
// Enforcement patterns
class SecurityGate {
  /// Block entire app in unsafe environment
  static Widget wrapIfNeeded(Widget child) {
    if (EnvConfig.requireSecureEnvironment && !DebugDetector.isSafeForProduction) {
      return _SecurityWarningScreen();
    }
    return child;
  }
  
  /// Block specific features in debug mode
  static Future<void> enforceProductionOnly(Future<void> action) async {
    if (EnvConfig.isProduction && DebugDetector.isDebugMode) {
      throw SecurityException('Feature not available in debug mode');
    }
    await action();
  }
  
  /// Log security warnings in debug
  static void logSecurityCheck(String check, bool passed) {
    if (DebugDetector.isDebugMode) {
      debugPrint('[Security] $check: ${passed ? 'PASS' : 'FAIL'}');
    }
  }
}

class _SecurityWarningScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.warning_amber, size: 64, color: Colors.orange),
              const SizedBox(height: 16),
              Text(
                'Security Warning',
                style: Theme.of(context).textTheme.headlineMedium,
              ),
              const SizedBox(height: 8),
              Text(
                'This app cannot run in debug/rooted environment.\n'
                'Please use a release build.',
                textAlign: TextAlign.center,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** Multiple debug indicators: `kDebugMode`, platform checks, emulator detection.
> **React/Vue tương đương:** `process.env.NODE_ENV === 'development'`, `window.__DEV__`.
> **Khác biệt quan trọng:** Flutter có thêm emulator/root detection - web không cần.

---

## Concept 6: Performance Monitoring in Production 🟡 SHOULD-KNOW

**WHY:** Production monitoring giúp identify performance issues before users complain.

### Monitoring Tools

| Tool | Type | Metrics | Integration |
|------|------|---------|-------------|
| **Firebase Performance** | APM | App startup, network, traces | Firebase SDK |
| **Sentry** | Error + Performance | Crashes, slow frames, traces | Sentry SDK |
| **Datadog** | APM | Full-stack monitoring | Datadog SDK |
| **Custom metrics** | DIY | App-specific metrics | Custom implementation |

### Custom Performance Monitoring

```dart
// Custom performance monitoring service
class PerformanceMonitor {
  static final PerformanceMonitor _instance = PerformanceMonitor._();
  factory PerformanceMonitor() => _instance;
  PerformanceMonitor._();
  
  final Map<String, Stopwatch> _timers = {};
  final List<PerformanceMetric> _metrics = [];
  
  /// Start tracking an operation
  void startTrace(String name) {
    _timers[name] = Stopwatch()..start();
  }
  
  /// End tracking and record metric
  void endTrace(String name) {
    final stopwatch = _timers.remove(name);
    if (stopwatch != null) {
      stopwatch.stop();
      _recordMetric(name, stopwatch.elapsedMilliseconds);
    }
  }
  
  void _recordMetric(String name, int durationMs) {
    final metric = PerformanceMetric(
      name: name,
      durationMs: durationMs,
      timestamp: DateTime.now(),
      deviceInfo: _getDeviceInfo(),
    );
    _metrics.add(metric);
    
    // Log in debug mode
    if (kDebugMode) {
      debugPrint('[Performance] $name: ${durationMs}ms');
    }
    
    // Send to backend in release mode
    if (!kDebugMode) {
      _sendToBackend(metric);
    }
  }
  
  DeviceInfo _getDeviceInfo() {
    return DeviceInfo(
      platform: Platform.operatingSystem,
      version: Platform.operatingSystemVersion,
      locale: Platform.localeName,
    );
  }
  
  Future<void> _sendToBackend(PerformanceMetric metric) async {
    // Batch send for efficiency
    // Implement batch buffer + periodic flush
  }
}
```

### Custom Performance Metrics

```dart
// Define custom metrics for app-specific monitoring
class AppMetrics {
  // Navigation metrics
  static const navigationTime = 'app.navigation.time';
  static const navigationCount = 'app.navigation.count';
  
  // API metrics
  static const apiLatency = 'app.api.latency';
  static const apiErrorRate = 'app.api.error_rate';
  
  // UI metrics
  static const frameBuildTime = 'app.ui.frame_build_time';
  static const listScrollFPS = 'app.ui.list_scroll_fps';
  
  // Business metrics
  static const loginTime = 'app.business.login_time';
  static const searchResultsTime = 'app.business.search_time';
}
```

### Memory Leak Detection

```dart
// Memory leak detection helper
class MemoryMonitor {
  static void checkMemoryLeak(Object object, String name) {
    if (kDebugMode) {
      // In debug mode, track object allocations
      // Use DevTools memory timeline for detailed analysis
      debugPrint('[Memory] Object: $name, Retained: ${_getRetainedSize(object)}');
    }
  }
  
  static int _getRetainedSize(Object object) {
    // Approximate size calculation
    // For accurate analysis, use DevTools memory timeline
    return 0; // Placeholder
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** `Performance` class, `Timeline` API, DevTools profiling.
> **React/Vue tương đương:** `PerformanceObserver`, Chrome DevTools Performance tab.
> **Khác biệt quan trọng:** Flutter có thêm frame timing, GPU timing (tracing/skia).

---

## Concept Map — How They Connect

```
Concept 1: Obfuscation      → Protect code from reverse engineering
Concept 2: SSL Pinning      → Protect network communication
Concept 3: Secure Storage   → Protect sensitive data at rest
Concept 4: API Key Protection → Protect secrets from exposure
Concept 5: Debug Detection  → Enforce security policies
Concept 6: Performance      → Monitor app health in production
```

**Security Pyramid:**

```
        ┌─────────────┐
        │   HTTPS     │     ← Basic encryption
        ├─────────────┤
        │ SSL Pinning │     ← Prevent MITM
        ├─────────────┤
        │   Obfuscate │     ← Protect code
        ├─────────────┤
        │Secure Storage│   ← Protect data
        └─────────────┘
```

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
