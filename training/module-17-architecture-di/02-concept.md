# Concepts — Architecture & Dependency Injection

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md).

---

## 1. Bootstrap Flow — AppInitialization — 🔴 MUST-KNOW

**WHY:** Hiểu chính xác flow khởi tạo app — `main.dart` gọi `AppInitializer.init()`, không gọi `configureInjection()` trực tiếp.

**EXPLAIN:**

```dart
// main.dart - Entry point
Future<void> main() async => runZonedGuarded(
  _runMyApp,
  (error, stackTrace) => _reportError(error: error, stackTrace: stackTrace),
);

Future<void> _runMyApp() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  await AppInitializer.init();  // ← Gọi AppInitializer.init(), KHÔNG gọi configureInjection() trực tiếp
  runApp(ProviderScope(
    observers: [AppProviderObserver()],
    child: MyApp(),
  ));
}
```

```dart
// app_initializer.dart - Initialize all app dependencies
class AppInitializer {
  const AppInitializer._();

  static Future<void> init() async {
    Env.init();
    await configureInjection();  // ← configureInjection() được gọi từ đây
    await getIt.get<PackageHelper>().init();
    await SystemChrome.setPreferredOrientations(...);
    await SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
  }
}
```

```dart
// di.dart - Dependency Injection configuration
final GetIt getIt = GetIt.instance;

@InjectableInit()
Future<void> configureInjection() => getIt.init();  // ← Được gọi bởi AppInitializer
```

```
Bootstrap Flow thực tế:
─────────────────────────────────────────────────────────────
main() → _runMyApp()
    │
    ├─ WidgetsFlutterBinding.ensureInitialized()
    │
    ├─ Firebase.initializeApp()
    │
    └─ AppInitializer.init()
            │
            ├─ Env.init()
            │
            ├─ configureInjection()  ← getIt.init() đăng ký tất cả dependencies
            │       │
            │       ├─ ServiceModule (SharedPreferences)
            │       ├─ @LazySingleton (AppApiService, FirebaseMessagingService...)
            │       └─ @Factory (RouteGuard, PagingExecutor...)
            │
            ├─ PackageHelper.init()
            │
            └─ SystemChrome configuration
                    │
                    └─ runApp()
─────────────────────────────────────────────────────────────
```

> ⚠️ **Sai:** `main.dart` gọi `configureInjection()` trực tiếp
> ✅ **Đúng:** `main.dart` gọi `AppInitializer.init()`, `AppInitializer` gọi `configureInjection()`

> 💡 **FE Perspective**
> **Flutter:** Bootstrap qua `AppInitializer.init()` — encapsulate logic khởi tạo.
> **React equivalent:** `index.tsx` → `<App />` với providers. Logic init trong `useEffect`.
> **Angular:** `main.ts` → `platformBrowserDynamic().bootstrapModule(AppModule)`.

<!-- AI_VERIFY: based-on main.dart line 21, app_initializer.dart line 10, di.dart line 16 -->

---

## 2. Service Locator Pattern vs DI Container — 🔴 MUST-KNOW

**WHY:** Phải hiểu cơ chế nền tảng — `get_it` không phải DI container thuần túy.

**EXPLAIN:**

`get_it` là **Service Locator** — singleton instance cung cấp dependencies khi được yêu cầu. Khác với DI Container (Angular, NestJS) nơi dependencies được inject tự động vào constructor.

```
Service Locator (get_it)          DI Container (Angular)
─────────────────────────         ──────────────────────
App start: register all          App start: bootstrap module tree
Usage: getIt<Service>()           Usage: constructor receives deps

Hoisting dependency up           Container resolves down
```

| Aspect | Service Locator (get_it) | DI Container |
|--------|------------------------|-------------|
| Registration | Manual | Declarative (via modules) |
| Resolution | Explicit `getIt<T>()` | Implicit via constructor |
| Testability | Swap via `registerMock()` | Override module |
| Complexity | Lower | Higher |

> 💡 **FE Perspective**
> **Flutter:** `getIt` service locator — caller chủ động lấy dependency.
> **React equivalent:** Context + `useContext()` hoặc Zustand store — component chủ động get state.
> **Angular:** `@Injectable()` decorator + constructor injection — container tự inject.
> **Khác biệt:** get_it đơn giản hơn DI container nhưng cần manual `getIt<T>()` ở mọi nơi.

---

## 3. @Injectable() & Constructor Injection — 🔴 MUST-KNOW

**WHY:** Mọi service trong codebase dùng pattern này — phải hiểu để debug và extend.

**EXPLAIN:**

Constructor injection — dependencies được declare trong constructor signature, injectable scanner tự động resolve tại runtime.

```dart
@Injectable()
class AppApiService {
  // Dependencies declared in constructor
  // injectable resolves by type at runtime
  AppApiService(this._preferences, this._client);

  final AppPreferences _preferences;
  final Dio _client;
}
```

**Tại sao dùng constructor injection?**

1. **Explicit dependencies** — nhìn constructor là biết class cần gì
2. **Immutability** — dependencies là `final`
3. **Testability** — pass mocks vào constructor trong tests
4. **Code gen compatible** — injectable scanner hoạt động tốt với constructor

> 💡 **FE Perspective**
> **Flutter:** Constructor injection qua `@Injectable()` + `get_it`.
> **React equivalent:** Props drilling vs Context vs Zustand. Constructor injection = explicit props (nhưng được resolve tự động).
> **Angular:** Constructor injection là default — `constructor(private svc: Service)`.
> **Khác biệt:** Flutter cần `getIt<T>()` call. Angular container tự resolve.

---

## 4. @module & @preResolve — 🟡 SHOULD-KNOW

**WHY:** Hiểu để configure async dependencies và environment-specific setup.

**EXPLAIN:**

```dart
@module
abstract class ServiceModule {
  @preResolve
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}
```

`@module` nhóm registrations. `@preResolve` yêu cầu `configureInjection()` **chờ** async operation trước khi proceed.

```
configureInjection()
  │
  ├─ Sync registrations (@Injectable, @singleton, @lazySingleton)
  │
  └─ @preResolve → await SharedPreferences.getInstance()
         │
         └─ getIt<SharedPreferences>() ready
```

**Tại sao @preResolve?**

`SharedPreferences.getInstance()` là async — app cần nó sẵn sàng ngay khi cần. `@preResolve` đảm bảo initialization hoàn tất trước khi `runApp()` được gọi.

> 💡 **FE Perspective**
> **Flutter:** `@preResolve` = `APP_INITIALIZER` trong Angular.
> **React equivalent:** `useEffect(() => { init().then(...) }, [])` — async initialization before render.
> **Khác biệt:** Flutter `@preResolve` chạy sync trong `configureInjection()`. React hooks chạy trong render cycle.

---

## 5. Singleton vs LazySingleton vs Injectable — 🟡 SHOULD-KNOW

**WHY:** Chọn đúng lifetime tránh memory leak và unexpected behavior.

**EXPLAIN:**

| Annotation | Created | Use case |
|-----------|---------|----------|
| `@singleton` | Immediately on `configureInjection()` | Config, constants |
| `@lazySingleton` | On first `getIt<T>()` | Heavy services: API, DB |
| `@injectable` | New instance every `getIt<T>()` | Stateless utilities |

```dart
@singleton     // Only 1 instance, created ASAP
class AppConfig { ... }

@lazySingleton // Only 1 instance, created on demand
class AppApiService { ... }

@injectable    // New instance each time
class Logger { ... }
```

> 💡 **FE Perspective**
> **Flutter:** Lifetime annotations kiểm soát instance count.
> **React equivalent:** Singleton = global state (Redux store). New instance = factory function.
> **Khác biệt:** Flutter lifetime được annotation declare. React lifetime được implementation control.

---

## 6. DI + Riverpod Hybrid Architecture — 🟡 SHOULD-KNOW

**WHY:** Dự án dùng cả injectable cho services VÀ Riverpod cho UI state — hiểu boundary.

**EXPLAIN:**

```
Architecture layers:
────────────────────────────
UI Layer     → Riverpod providers (state management)
Service Layer → @Injectable() classes (get_it)
Data Layer   → @Injectable() classes (API, storage)
────────────────────────────
```

**Pattern trong codebase:**

```dart
// 1. @Injectable() service — gets dependencies via get_it
@Injectable()
class AuthRepository {
  AuthRepository(this._apiService, this._preferences);
  final AppApiService _apiService;
  final AppPreferences _preferences;
}

// 2. Riverpod provider — uses the service
final authRepositoryProvider = Provider<AuthRepository>(
  (ref) => getIt<AuthRepository>(),
);

// 3. UI consumes via Riverpod
final authState = ref.watch(authRepositoryProvider);
```

> 💡 **FE Perspective**
> **Flutter:** Hybrid DI — injectable cho services, Riverpod cho state.
> **React equivalent:** Services = Zustand/Redux stores. State = React Query / Zustand.
> **Khác biệt:** Flutter tách rõ service ( injectable) và UI state (Riverpod). React thường gộp.

---

## 7. Architecture Trade-offs — get_it vs Riverpod DI — 🟢 AI-GENERATE

**WHY:** Cần hiểu khi nào chọn approach nào.

**EXPLAIN:**

| Criteria | get_it + injectable | Riverpod Provider DI |
|---------|--------------------|--------------------|
| Boilerplate | Low (codegen) | Very low |
| UI state | Not designed for | Excellent |
| Service layer | Excellent | Good |
| Testing | `registerMock()` | `overrideWith()` |
| Async init | `@preResolve` | Family providers |
| Type safety | Excellent | Excellent |

**Khi nào dùng gì?**

- **Chỉ Riverpod DI:** Project nhỏ, dùng `Riverpod.overrideWith()` đủ cho mọi mock.
- **get_it + Riverpod:** Project lớn, nhiều services, cần clear separation.

> 💡 **FE Perspective**
> **Flutter:** get_it + Riverpod = separation of concerns (services vs state).
> **React equivalent:** Service classes (Axios instances) = Singleton context. UI state = Zustand/Redux.
> **Khác biệt:** React ít có khái niệm DI container. Thường dùng singleton patterns.

---

## Summary — Badge Table

| # | Concept | Badge | Lý do |
|---|---------|-------|-------|
| 1 | Bootstrap Flow (AppInitialization) | 🔴 MUST-KNOW | `main()` → `AppInitializer.init()` → `configureInjection()` |
| 2 | Service Locator vs DI Container | 🔴 MUST-KNOW | Nền tảng — hiểu get_it là locator không phải container |
| 3 | @Injectable() & Constructor Injection | 🔴 MUST-KNOW | Mọi service dùng pattern này |
| 4 | @module & @preResolve | 🟡 SHOULD-KNOW | Config async deps, ít dùng nhưng quan trọng |
| 5 | Singleton vs LazySingleton vs Injectable | 🟡 SHOULD-KNOW | Chọn đúng lifetime tránh leak |
| 6 | DI + Riverpod Hybrid | 🟡 SHOULD-KNOW | Architecture decision của dự án |
| 7 | Architecture Trade-offs | 🟢 AI-GENERATE | Khi nào chọn approach nào |

---

> **Tiếp theo:** [03-exercise.md](./03-exercise.md) — bài tập áp dụng DI patterns.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
