# Verify — Architecture & Dependency Injection

> Trả lời **không mở source code** trước, sau đó verify bằng source.

---

## Section A — Actual Bootstrap Flow (3 câu)

### A1. AppInitializer Entry Point
**Q:** `AppInitializer.init()` được gọi ở đâu trong `main.dart`? Tại sao phải gọi TRƯỚC `runApp()`?

**Expected:** Được gọi trong `main()` hoặc `_runMyApp()` trước `runApp()`. Phải gọi trước vì:
1. `configureInjection()` phải hoàn tất trước khi services được sử dụng
2. Providers dùng `getIt<T>()` khi created
3. Nếu gọi sau `runApp()`, services chưa ready → runtime error

### A2. AppInitializer.init() Implementation
**Q:** Trong `app_initializer.dart`, `init()` thực hiện những bước gì theo thứ tự?

**Expected:**
1. `Env.init()` — initialize environment
2. `await configureInjection()` — setup DI
3. `await getIt.get<PackageHelper>().init()` — init helpers
4. `SystemChrome.setPreferredOrientations()` — orientation setup

### A3. configureInjection() Location
**Q:** `configureInjection()` được định nghĩa ở đâu? Nó là async hay sync?

**Expected:** Được định nghĩa trong `lib/di.dart`, là **async** function vì `@preResolve` dependencies có thể là async operations.

---

## Section B — @Injectable Patterns (3 câu)

### B1. DI Registration Flow
**Q:** Trace flow từ `@Injectable()` annotation đến khi service được resolve tại runtime.

**Expected:**
```
Source Code
    ↓ (build_runner)
di.config.dart (generated)
    ↓ (configureInjection())
getIt.registerXxx() calls
    ↓ (getIt<Service>())
Runtime resolution
```

### B2. @lazySingleton vs @singleton
**Q:** Khi nào dùng `@lazySingleton` thay vì `@singleton`?

**Expected:** `@lazySingleton` — tạo instance khi `getIt<T>()` được gọi lần đầu (on demand). Phù hợp cho heavy services như API, Database. `@singleton` — tạo ngay khi `configureInjection()` chạy.

### B3. @preResolve Purpose
**Q:** `@preResolve` giải quyết vấn đề gì? Ví dụ trong codebase.

**Expected:** Vấn đề: async initialization (SharedPreferences.getInstance()). `@preResolve` đảm bảo async operation hoàn tất trước khi `configureInjection()` return.

---

## Section C — Architecture Decisions (2 câu)

### C1. DI + Riverpod Hybrid
**Q:** Tại sao dự án dùng BOTH injectable (get_it) AND Riverpod providers?

**Expected:** Injectable cho service layer (API, storage) — constructor injection, codegen. Riverpod cho UI state management — reactive, excellent for rebuilds. Separation of concerns.

### C2. Mocking in Tests
**Q:** Làm thế nào để mock `@Injectable()` class trong unit test?

**Expected:** `getIt.registerMock<T>(mockInstance)` trong `setUp()`. `getIt.reset()` trong `tearDown()` để cleanup.

---

## ➡️ Next Module

Hoàn thành Module 17! Bạn đã nắm vững DI architecture.

→ Tiến sang **[Module 18 — Testing](../module-18-testing/)** để học viết unit tests cho các @Injectable classes.

<!-- AI_VERIFY: generation-complete -->
