# Exercise — Advanced Performance & Security

> 📌 **Prerequisites:**
> - Hoàn thành [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md)
> - Có `base_flutter` project đã setup

---

## Exercise Overview

| # | Exercise | Focus | Difficulty | Thời gian |
|---|----------|-------|------------|-----------|
| 1 | Security Audit | Review security patterns trong codebase | 🟡 Medium | ~45 min |
| 2 | SSL Pinning Implementation | Implement certificate pinning | 🔴 Hard | ~60 min |
| 3 | Performance Monitoring Setup | Add custom performance tracking | 🟡 Medium | ~45 min |

**Tổng thời gian:** ~2-3 hours

---

## Exercise 1: Security Audit 🔍

**Mục tiêu:** Review codebase để identify security vulnerabilities và suggest improvements.

### Task 1.1: Audit Secure Storage Usage

Tìm và review tất cả usages của `FlutterSecureStorage` trong codebase:

```bash
# Search for secure storage usage
grep -r "FlutterSecureStorage" lib/ --include="*.dart"
grep -r "SharedPreferences" lib/ --include="*.dart"
```

**Kiểm tra:**
1. Sensitive data có đang lưu trong `SharedPreferences` không?
2. Storage keys có hardcoded không?
3. Có data cần encrypted nhưng không dùng secure storage?

**Report template:**

```markdown
## Security Audit Report

### 1. Secure Storage Usage

| File | Storage Type | Data Stored | Risk Level | Recommendation |
|------|--------------|-------------|------------|----------------|
| ... | ... | ... | ... | ... |
```

### Task 1.2: Audit API Key Storage

Review cách API keys được handle:

```bash
# Search for API key patterns
grep -rn "api.*key" lib/ --include="*.dart" -i
grep -rn "API.*KEY" lib/ --include="*.dart"
grep -rn "const.*=" lib/config/ --include="*.dart"
```

**Kiểm tra:**
1. Có API key hardcoded trong source code không?
2. Có key trong git history không? (kiểm tra `git log`)
3. Environment variables được setup đúng chưa?

### Task 1.3: Audit Network Security

Review network layer configuration:

```bash
# Search for network configuration
grep -rn "Dio\|http\|HttpClient" lib/ --include="*.dart"
```

**Kiểm tra:**
1. SSL pinning có được implement không?
2. Certificate validation có bị bypass không?
3. Có sensitive data log ra console không?

### Deliverable

Tạo file `security-audit.md` trong thư mục exercise với nội dung:
- List các vulnerabilities found
- Risk level (Critical/High/Medium/Low)
- Recommended fix cho từng issue
- Priority order để fix

---

## Exercise 2: SSL Pinning Implementation 🔐

**Mục tiêu:** Implement SSL certificate pinning cho API client.

### Setup

```bash
# Tạo thư mục exercise
mkdir -p exercises/ssl-pinning
cd exercises/ssl-pinning
```

### Task 2.1: Extract Server Certificate

```bash
# Lấy certificate fingerprint từ server
# Thay thế api.example.com bằng server thực tế của bạn
openssl s_client -connect api.example.com:443 </dev/null 2>/dev/null | \
  openssl x509 -noout -fingerprint -sha256
```

### Task 2.2: Create SSL Pinning Service

Tạo file `lib/common/service/ssl_pinning_service.dart`:

```dart
// TODO: Implement SSL Pinning Service
// Requirements:
// 1. Create Dio client với SSL pinning
// 2. Support multiple backup pins
// 3. Handle certificate validation errors gracefully
// 4. Log pinning failures (in debug mode)

import 'dart:io';

class SSLPinningService {
  // TODO: Add your pinned certificate hashes
  static const _pinnedCertificates = <String>[
    // Primary certificate
    'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',
    // Backup certificate (optional)
    // 'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=',
  ];
  
  // TODO: Implement createPinnedHttpClient()
  // Return HttpClient với certificate validation
  
  // TODO: Implement validateCertificate()
  // Check if server certificate matches pinned certificates
  
  // TODO: Implement onCertificateError()
  // Handle certificate validation errors
}
```

### Task 2.3: Integrate với AppApiService

Update `lib/data_source/api/app_api_service.dart`:

```dart
// Trước:
class AppApiService {
  AppApiService() {
    _dio = Dio(BaseOptions(...));
  }
}

// Sau:
class AppApiService {
  final SSLPinningService _sslService = SSLPinningService();
  
  AppApiService() {
    _dio = Dio(BaseOptions(...));
    
    // Thêm SSL pinning interceptor
    _dio.httpClientAdapter = _sslService.createPinnedHttpClientAdapter();
  }
}
```

### Task 2.4: Handle Certificate Errors

Implement graceful error handling:

```dart
// Handle SSL errors
void handleSSLError(DioException e) {
  if (e.type == DioExceptionType.badCertificate) {
    // Log sự cố
    Log.e('SSL Certificate validation failed: ${e.message}');
    
    // Alert security team (trong production)
    if (!kDebugMode) {
      _reportSecurityIncident(e);
    }
    
    throw SecurityException('SSL Certificate validation failed');
  }
}
```

### Verification

```bash
# Test với valid certificate
flutter test test/ssl_pinning_test.dart

# Manual test: intercept traffic với Charles Proxy
# Certificate pinning sẽ fail nếu intercept được
```

### Deliverable

- `lib/common/service/ssl_pinning_service.dart` - SSL pinning implementation
- Updated `lib/data_source/api/app_api_service.dart`
- `test/ssl_pinning_test.dart` - Unit tests for SSL pinning validation

### Additional Test Files to Create

```
test/
├── ssl_pinning_test.dart          # Test certificate validation
├── performance_monitor_test.dart   # Test performance tracking
└── security_helper_test.dart     # Test debug detection
```

---

## Exercise 3: Performance Monitoring Setup 📊

**Mục tiêu:** Add custom performance monitoring để track app health.

### Task 3.1: Create Performance Monitor Service

Tạo file `lib/common/service/performance_monitor.dart`:

```dart
// Performance monitoring service
class PerformanceMonitor {
  // Singleton pattern
  static final PerformanceMonitor _instance = PerformanceMonitor._internal();
  factory PerformanceMonitor() => _instance;
  PerformanceMonitor._internal();
  
  final Map<String, Stopwatch> _activeTraces = {};
  final List<PerformanceMetric> _metrics = [];
  
  // TODO: Implement startTrace(name)
  // Bắt đầu tracking một operation
  
  // TODO: Implement endTrace(name)
  // Kết thúc tracking và record metric
  
  // TODO: Implement customMetric(name, value)
  // Record custom metric value
  
  // TODO: Implement getMetrics()
  // Return all recorded metrics
  
  // TODO: Implement clearMetrics()
  // Clear metrics buffer
  
  // TODO: Implement reportMetrics()
  // Send metrics to backend (production)
}
```

### Task 3.2: Add Navigation Tracing

Tạo wrapper để track navigation performance:

```dart
// Navigation performance tracking
class PerformanceNavigatorObserver extends RouteObserver<PageRoute> {
  final PerformanceMonitor _monitor = PerformanceMonitor();
  
  @override
  void didPush(Route route, Route? previousRoute) {
    final routeName = route.settings.name ?? 'unknown';
    _monitor.startTrace('navigation.push.$routeName');
  }
  
  @override
  void didPop(Route route, Route? previousRoute) {
    final routeName = route.settings.name ?? 'unknown';
    _monitor.endTrace('navigation.pop.$routeName');
  }
}
```

### Task 3.3: Add API Call Tracing

Tạo interceptor để track API performance:

```dart
// API performance interceptor
class PerformanceInterceptor extends Interceptor {
  final PerformanceMonitor _monitor = PerformanceMonitor();
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    _monitor.startTrace('api.${options.method}.${options.path}');
    handler.next(options);
  }
  
  @override
  void onResponse(Response response, RequestInterceptorHandler handler) {
    _monitor.endTrace('api.${response.requestOptions.method}.${response.requestOptions.path}');
    handler.next(response);
  }
  
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    final path = err.requestOptions.path;
    _monitor.endTrace('api.error.${err.requestOptions.method}.$path');
    handler.next(err);
  }
}
```

### Task 3.4: Integrate vào App

Update `lib/main.dart`:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      navigatorObservers: [
        PerformanceNavigatorObserver(), // Add performance tracking
      ],
      builder: (context, child) {
        // Wrap với performance monitoring
        return child ?? const SizedBox();
      },
    );
  }
}
```

Update `lib/data_source/api/app_api_service.dart`:

```dart
class AppApiService {
  AppApiService() {
    _dio = Dio(BaseOptions(...));
    _dio.interceptors.add(PerformanceInterceptor()); // Add API tracing
  }
}
```

### Verification

```bash
# Chạy app trong debug mode
flutter run

# Kiểm tra console output
# Bạn sẽ thấy logs như:
# [Performance] api.GET./users/me: 245ms
# [Performance] navigation.push./profile: 120ms
```

### Deliverable

- `lib/common/service/performance_monitor.dart` - Performance monitoring service
- `lib/common/observer/performance_navigator_observer.dart` - Navigation tracking
- `lib/common/interceptor/performance_interceptor.dart` - API tracking
- Updated `lib/main.dart` và `lib/data_source/api/app_api_service.dart`

---

## Bonus Challenges ⭐

### Bonus 1: Memory Leak Detection

Implement memory leak detection helper:

```dart
// Track object allocations để phát hiện leaks
class MemoryLeakDetector {
  static void track(Object object, String name) {
    if (kDebugMode) {
      // Log object allocation
      // Sử dụng DevTools timeline cho detailed analysis
    }
  }
  
  static void untrack(Object object, String name) {
    if (kDebugMode) {
      // Log object deallocation
    }
  }
}
```

### Bonus 2: Obfuscation Dictionary

Tạo custom obfuscation dictionary cho project:

```bash
# Tạo file obfuscation_dictionaries.txt
# Thêm words không nên dùng cho obfuscation
# VD: Tên class quan trọng, brand names, etc.
```

---

## Submission

Sau khi hoàn thành các exercises:

1. **Tạo PR** với title: `feat(security): Security hardening và Performance monitoring`
2. **Kiểm tra:**
   - [ ] Security audit report hoàn chỉnh
   - [ ] SSL pinning hoạt động (test với proxy)
   - [ ] Performance monitoring log ra console trong debug mode
   - [ ] Không có security issues mới được introduced
3. **Demo:** Trình bày cách identify security vulnerabilities trong codebase

---

## Hints

### Hint 1: SSL Pinning

```
Nếu bạn không có server thật để test:
1. Sử dụng self-signed certificate
2. Thêm certificate vào trusted store
3. Pin certificate đó
```

### Hint 2: Performance Monitor

```
PerformanceMonitor nên là singleton để dễ access từ mọi nơi.
Sử dụng factory pattern hoặc static instance.
```

### Hint 3: Security Audit

```
Sử dụng `flutter analyze` để tìm potential issues.
Kiểm tra các patterns sau:
- `print()` hoặc `debugPrint()` với sensitive data
- Hardcoded strings (không phải localization)
- TODO comments đánh dấu security concerns
```

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
