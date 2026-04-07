# Exercises — Architecture & Dependency Injection

> 📌 **Recap:**
> - **M1:** `main.dart` bootstrap → `AppInitializer.init()` → `configureInjection()`
> - **M8:** Riverpod providers — alternative to class-based DI

---

## Exercise 1 — ⭐ Trace Actual Bootstrap & DI Flow

### Mục tiêu

Hiểu flow thực tế từ app startup đến DI resolution.

### Actual Flow

```
main.dart
    ↓
AppInitializer.init()
    ├── Env.init()
    ├── configureInjection()  ← DI setup
    └── SystemChrome setup
```

### Hướng dẫn

1. Đọc `base_flutter/lib/main.dart`:
   - Tìm nơi `AppInitializer.init()` được gọi
   - Xác định thứ tự: init → runApp

2. Đọc `base_flutter/lib/app_initializer.dart`:
   - `init()` là async method
   - Gọi `configureInjection()` ở đâu trong flow

3. Đọc `base_flutter/lib/di.dart`:
   - `configureInjection()` được định nghĩa thế nào
   - `@InjectableInit()` decorator làm gì

### Acceptance Criteria

- [ ] Vẽ sequence diagram: `main()` → `AppInitializer.init()` → `configureInjection()` → `getIt<Service>()`
- [ ] Xác định tất cả `@Injectable()` classes trong `lib/`
- [ ] Xác định tất cả `@module` classes
- [ ] Trả lời: Tại sao `configureInjection()` phải là async?

<details>
<summary>🏗️ Architecture Hint</summary>

1. Tìm `AppInitializer` trong codebase → đọc `init()` method
2. Tìm `configureInjection()` → đọc signature và implementation
3. Tìm tất cả `getIt<` trong codebase → tất cả resolution sites

</details>

---

## Exercise 2 — ⭐⭐ Trace @Injectable Codegen Flow

### Mục tiêu

Hiểu cách injectable annotations được process thành code.

### Hướng dẫn

1. Tìm file `di.config.dart` trong `base_flutter/lib/`

2. So sánh:
   - Classes có `@Injectable()` annotation
   - Entries trong `di.config.dart`

3. Trả lời:
   - `di.config.dart` được auto-generate bởi gì?
   - Có được edit thủ công không?
   - Khi nào cần regenerate?

### Acceptance Criteria

- [ ] Đọc `di.config.dart` và hiểu structure
- [ ] Map `@Injectable()` classes với entries trong config
- [ ] Trả lời: Cách thêm dependency mới mà không cần chạy build_runner (tạm thời)?

<details>
<summary>🏗️ Architecture Hint</summary>

```dart
// di.dart - nơi configureInjection được định nghĩa
@InjectableInit()
Future<void> configureInjection() => getIt.init();

// di.config.dart - auto-generated, chứa registration code
// Ví dụ:
getIt.registerLazySingleton<ApiService>(() => ApiService(getIt<HttpClient>()));
```

</details>

---

## Exercise 3 — ⭐⭐ Add New Injectable Service

### Mục tiêu

Tạo `@Injectable()` service mới và verify nó được registered.

### Hướng dẫn

1. Tạo `LoggerService` trong `lib/common/logger_service.dart`:
   ```dart
   import 'package:injectable/injectable.dart';

   @lazySingleton
   class LoggerService {
     void log(String message) => print('[LOG] $message');
   }
   ```

2. Inject vào một service có sẵn (VD: `AppApiService`):
   ```dart
   @lazySingleton
   class AppApiService {
     final LoggerService _logger;
     AppApiService(this._logger);  // Constructor injection
   }
   ```

3. Chạy `dart run build_runner build --delete-conflicting-outputs`

4. Verify trong `di.config.dart` có entry mới

### Acceptance Criteria

- [ ] Tạo `LoggerService` với `@lazySingleton`
- [ ] Inject vào existing service
- [ ] Run build_runner để regenerate `di.config.dart`
- [ ] Verify registration trong generated file
- [ ] Compile thành công

<details>
<summary>🏗️ Architecture Hint</summary>

- `@lazySingleton` — tạo instance khi `getIt<LoggerService>()` được gọi lần đầu
- Constructor injection: `LoggerService(this._logger)`
- Sau khi thêm class, chạy `dart run build_runner build`

</details>

---

## Exercise 4 — ⭐⭐⭐ Use DI Service in ViewModel

### Mục tiêu

Sử dụng DI-injected service trong ViewModel.

### Hướng dẫn

1. Tạo `AnalyticsService`:
   ```dart
   @lazySingleton
   class AnalyticsService {
     void trackEvent(String name, Map<String, dynamic> data) {
       // Implementation
     }
   }
   ```

2. Inject vào ViewModel:
   ```dart
   @injectable
   class ProfileViewModel extends BaseViewModel<ProfileState> {
     final AnalyticsService _analytics;
     ProfileViewModel(this._analytics);
   }
   ```

3. Sử dụng trong action:
   ```dart
   void onPageView() {
     _analytics.trackEvent('profile_view', {});
   }
   ```

### Acceptance Criteria

- [ ] Tạo `AnalyticsService` với `@lazySingleton`
- [ ] Inject vào ViewModel
- [ ] Sử dụng service trong một action
- [ ] Build thành công

---

## Exercise 5 — ⭐⭐⭐ Mock Injectable in Tests

### Mục tiêu

Viết unit test cho `@Injectable()` class với mock dependencies.

### Hướng dẫn

1. Tạo mock:
   ```dart
   class MockLoggerService extends Mock implements LoggerService {}
   ```

2. Register mock trong test:
   ```dart
   setUp(() {
     final mock = MockLoggerService();
     getIt.registerMock<LoggerService>(mock);
   });
   ```

3. Reset sau test:
   ```dart
   tearDown(() {
     getIt.reset();
   });
   ```

### Acceptance Criteria

- [ ] Tạo test file cho service với mock dependency
- [ ] Dùng `getIt.registerMock<T>(mockInstance)` để swap real implementation
- [ ] Reset mock sau test với `getIt.reset()`
- [ ] Tests pass

---

> **Tiếp theo:** [04-verify.md](./04-verify.md) — Kiểm tra kiến thức DI architecture.

---

## ↩️ Revert Changes

Sau khi hoàn thành mỗi bài tập, revert changes để không ảnh hưởng codebase:

```bash
# Revert tất cả thay đổi trong bài tập
git checkout -- lib/

# Hoặc revert cụ thể file
# git checkout -- lib/path/to/modified/file.dart

# Nếu đã chạy codegen (make gen, make ep):
# 1. Revert barrel/file changes
git checkout -- lib/index.dart

# 2. Chạy lại make để clean
make gen
```

> ⚠️ **Quan trọng:** Luôn revert trước khi chuyển bài tập hoặc trước khi `git commit`. Code của bạn chỉ nên ở trong branch feature, không nên modify các base files trực tiếp.



<!-- AI_VERIFY: generation-complete -->
