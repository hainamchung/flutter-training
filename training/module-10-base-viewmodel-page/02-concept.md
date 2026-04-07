# Concepts — BaseViewModel & BasePage (MVVM Pattern)

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. MVVM Pattern Overview — Tại sao Tách biệt? 🔴 MUST-KNOW

**WHY:** Không có architecture → business logic lẫn với UI code → impossible to test, hard to maintain, team conflicts.

### Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    MVVM PATTERN                              │
│                                                             │
│  ┌─ VIEW (Flutter Widget) ──────────────────────────────┐  │
│  │  • UI rendering                                        │  │
│  │  • User input handling                                 │  │
│  │  • Calls ViewModel methods via ref.read               │  │
│  │  • Rebuilds on state changes via ref.watch            │  │
│  │                                                        │  │
│  │  LoginPage, HomePage, ProfilePage                      │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                           │ ref.watch (observe state)        │
│                           ▼                                  │
│  ┌─ VIEWMODEL (Business Logic) ─────────────────────────┐  │
│  │  • State management (StateNotifier)                    │  │
│  │  • Input validation                                   │  │
│  │  • Async actions (runCatching)                        │  │
│  │  • Navigation coordination                            │  │
│  │  • Access other providers via Ref                    │  │
│  │                                                        │  │
│  │  LoginViewModel, HomeViewModel, ProfileViewModel      │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                           │ state.data (model)               │
│                           ▼                                  │
│  ┌─ MODEL (Data) ───────────────────────────────────────┐  │
│  │  • Pure data classes                                  │  │
│  │  • Immutable (freezed)                               │  │
│  │  • No business logic                                 │  │
│  │                                                        │  │
│  │  LoginState, HomeState, ProfileState                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### So sánh: Có MVVM vs Không có MVVM

```dart
// ❌ Không có MVVM — logic trong Widget
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          onChanged: (email) {
            // ❌ Logic trong Widget — KHÓ TEST
            if (email.contains('@')) {
              // validation logic
            }
          },
        ),
        ElevatedButton(
          onPressed: () async {
            // ❌ Loading/error handling trong Widget — DUPLICATE
            showLoading();
            try {
              await api.login();
              hideLoading();
              Navigator.push(...);
            } catch (e) {
              showError(e);
            }
          },
        ),
      ],
    );
  }
}

// ✅ Có MVVM — logic trong ViewModel
class LoginPage extends BasePage<LoginState, ...> {
  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    return Column(
      children: [
        TextField(
          onChanged: (email) => ref.read(provider.notifier).setEmail(email),
        ),
        ElevatedButton(
          onPressed: () => ref.read(provider.notifier).login(),
        ),
      ],
    );
  }
}

class LoginViewModel extends BaseViewModel<LoginState> {
  // ✅ Logic trong ViewModel — DỄ TEST
  void setEmail(String email) {
    data = data.copyWith(email: email);
  }

  Future<void> login() async {
    await runCatching(action: () async {
      await api.login();
      ref.read(navigatorProvider).replaceAll([MainRoute()]);
    });
  }
}
```

### Benefits của MVVM

| Benefit | Giải thích |
|---------|------------|
| **Testable** | ViewModel không phụ thuộc Flutter widget → unit test được |
| **Separation of Concerns** | UI ≠ Logic ≠ Data |
| **Reusability** | ViewModel có thể share giữa multiple views |
| **Team Scalability** | Frontend dev (View), Backend dev (ViewModel/Model) làm việc song song |

> 💡 **FE Perspective**
> **Flutter:** MVVM pattern — View (Widget), ViewModel (BaseViewModel/StateNotifier), Model (BaseState/freezed data).
> **React/Vue tương đương:** React: Component (View) + Custom Hook (ViewModel) + TypeScript interfaces (Model). Vue: Template (View) + Options API/Composition (ViewModel) + reactive data (Model).
> **Khác biệt quan trọng:** Flutter cần explicit base classes (BaseViewModel, BasePage); React/Vue dùng conventions và composition patterns.

---

## 2. BaseState Contract — Abstract Marker Class 🟢 AI-GENERATE

**WHY:** Không có constraint → bất kỳ type nào cũng truyền vào `BaseViewModel<T>`. Một `int` hay `String` làm state → mất structure, `copyWith` không hoạt động.

<!-- AI_VERIFY: base_flutter/lib/common/base/base_state.dart -->
```dart
abstract class BaseState {
  const BaseState();
}
```
<!-- END_VERIFY -->
<!-- END_VERIFY -->

**Tại sao wrap thay vì extend?**

```dart
// ❌ Không dùng marker — mọi type đều pass
class BaseViewModel<ST> extends StateNotifier<CommonState<ST>> { }
// → BaseViewModel<int> compiles! Nhưng int không có copyWith...

// ✅ Marker constraint — chỉ state classes:
class BaseViewModel<ST extends BaseState> extends StateNotifier<CommonState<ST>> { }
// → BaseViewModel<int> → COMPILE ERROR
```

---

## 3. CommonState — Generic Envelope Pattern 🔴 MUST-KNOW

**WHY:** Nếu mỗi page state tự define `isLoading`, `error`, `doingAction` → duplicate code, inconsistent naming.

### Envelope = wrapper tách concerns

```
┌─ CommonState<LoginState> ──────────────┐
│  data: LoginState(email, password)     │ ← Business data (varies per page)
│  appException: null                     │ ← Error state (shared)
│  isLoading: false                       │ ← Loading state (shared)
│  isFirstLoading: false                  │ ← First-load flag (shared)
│  doingAction: {'login': true}          │ ← Per-action tracking (shared)
└─────────────────────────────────────────┘
```

### Benefits

- **BaseViewModel** chỉ operate trên `CommonState` fields (`isLoading`, `appException`) → generalize logic
- **Business state** (`LoginState`) chỉ chứa domain data → clean, testable
- **UI** `ref.watch(provider.select((s) => s.isLoading))` → consistent across ALL pages

### `doingAction` — Map-based action tracking

```dart
// Set:
state = state.copyWith(doingAction: {...state.doingAction, 'login': true});

// Query:
state.isDoingAction('login') // → true

// UI:
ref.watch(provider.select((s) => s.isDoingAction('login')))
// → disable login button khi đang submit
```

→ Nhiều action đồng thời? `{'login': true, 'refreshProfile': true}` — track independent.

### `isFirstLoading` — skeleton pattern

Lần load đầu tiên → `isFirstLoading = true` → show shimmer/skeleton. Các lần sau → spinner thường. UX tối ưu: user thấy content shape trước khi data arrive.

> 💡 **FE Perspective**
> **Flutter:** `CommonState<T>` envelope tách business data (`data: T`) khỏi infrastructure fields (`isLoading`, `appException`, `doingAction`).
> **React/Vue tương đương:** Redux state envelope `{ data: T, loading: boolean, error: Error }` (React), Pinia store state wrapper (Vue).

---

## 4. BaseViewModel Lifecycle — Mounted Guard + State Accessors 🔴 MUST-KNOW

**WHY:** Async operations chạy sau khi widget dispose → `setState()` crash. Không có base class → mỗi ViewModel tự handle → inconsistent.

### Lifecycle Diagram

```
┌──────────────────────────────────────────────────────────┐
│              BaseViewModel Lifecycle                      │
│                                                          │
│  Widget mount                                            │
│      ↓                                                   │
│  ┌─────────┐    ┌──────────────────────────┐             │
│  │  init()  │ →  │  active (mounted=true)   │             │
│  └─────────┘    │  ├─ showLoading/hide      │             │
│                  │  ├─ runCatching → action  │             │
│                  │  ├─ set data / state      │             │
│                  │  └─ error → exception     │             │
│                  └──────────┬───────────────┘             │
│                             ↓                             │
│                  Widget unmount (navigate away)           │
│                             ↓                             │
│                  ┌──────────────────────┐                 │
│                  │  dispose()            │                 │
│                  │  mounted = false       │                 │
│                  │  state set → Log.e     │                 │
│                  └──────────┬───────────┘                 │
│                             ↓                             │
│                     Garbage Collected                     │
└──────────────────────────────────────────────────────────┘
```

### `mounted` guard — defensive programming

```
Timeline:
t0: User opens LoginPage → LoginViewModel created, mounted = true
t1: login() called → API request starts (async)
t2: User presses Back → LoginPage disposed → mounted = false
t3: API response arrives → set state → 💥 (without guard)
```

### Loading reference counting

```dart
int _loadingCount = 0;

void showLoading() {
  if (_loadingCount <= 0) {
    state = state.copyWith(isLoading: true, ...);
  }
  _loadingCount++;  // luôn increment
}

void hideLoading() {
  if (_loadingCount <= 1) {
    state = state.copyWith(isLoading: false, ...);
  }
  _loadingCount--;  // luôn decrement
}
```

Parallel calls scenario:
```
fetchProfile() → showLoading() → count=1, isLoading=true
fetchNotifications() → showLoading() → count=2
fetchProfile done → hideLoading() → count=1 (isLoading vẫn true!)
fetchNotifications done → hideLoading() → count=0, isLoading=false ✅
```

> 💡 **FE Perspective**
> **Flutter:** `BaseViewModel` dùng loading counter (`showLoading`/`hideLoading`) cho concurrent requests và `mounted` flag chống setState-after-dispose.
> **React/Vue tương đương:** React `useRef` for mounted check + `useReducer` aggregate state. Vue `onBeforeUnmount` flag + `ref()` composable.
> **Khác biệt quan trọng:** Flutter cần manual batching qua loading count, React 18 auto-batches. Vue auto-handles unmount cleanup.

---

## 5. runCatching — Centralized Error Handling Pattern 🔴 MUST-KNOW

**WHY:** Mỗi ViewModel method cần try/catch + loading + error dispatch → massive duplication.

### Mental Model

> 🧠 Think of `runCatching` as `useQuery({ retry: 2, onError, onSettled })` from React Query — but for any async action, not just data fetching.

### Basic Pattern

**Before (không dùng runCatching):**
```dart
// ❌ Mỗi method tự handle — 30+ lines boilerplate:
Future<void> login() async {
  try {
    showLoading();
    startAction('login');
    await _doLogin();
    hideLoading();
    stopAction('login');
  } catch (e) {
    hideLoading();
    stopAction('login');
    final appException = e is AppException ? e : AppUncaughtException(rootException: e);
    exception = appException;
  }
}
```

**After (dùng runCatching):**
```dart
// ✅ Centralized — chỉ khai báo business logic:
Future<void> login() async {
  await runCatching(
    action: () async => await _doLogin(),
    actionName: 'login',
  );
}
```

### 3 Error Handling Strategies

| Strategy | `handleErrorWhen` | `doOnError` | Kết quả |
|----------|-------------------|-------------|---------|
| **Global dialog** | `null` (default true) | `null` | Exception → `ExceptionHandler` → dialog |
| **Inline error** | `(_) => false` | set `onPageError` | Error hiển thị trên form |
| **Mixed** | conditional | set local + partial global | Một số error inline, một số popup |

**Login example — inline strategy:**
```dart
await runCatching(
  action: () async { ... },
  handleErrorWhen: (_) => false,    // suppress global dialog
  doOnError: (e) async {
    data = data.copyWith(onPageError: e.message);
  },
);
```

### Retry Chain — Recursive Pattern

```
runCatching(maxRetries: 2)
  └→ fail → appException.onRetry = () {
       doOnRetry()              // e.g. refresh token
       runCatching(maxRetries: 1)  // recursive
         └→ fail → appException.onRetry = () {
              doOnRetry()
              runCatching(maxRetries: 0)  // last attempt
                └→ fail → no retry, dispatch error
            }
     }
```

→ User tap "Retry" trên error dialog → `appException.onRetry()` → chain execute.

---

## 6. BasePage — Reactive UI Binding 🔴 MUST-KNOW

**WHY:** Mỗi page cần reactive listen exception, loading, analytics. Không có BasePage → mỗi page tự wire → 20-30 lines boilerplate × N pages.

### HookConsumerWidget — best of both

| Base class | Riverpod | Hooks | Use case |
|------------|----------|-------|----------|
| `ConsumerWidget` | ✅ `ref` | ❌ | Simple read/watch |
| `HookWidget` | ❌ | ✅ `useState` | Local state |
| `HookConsumerWidget` | ✅ | ✅ | **BasePage** — cần cả hai |

### `ref.listen` vs `ref.watch` trong BasePage

> 📌 `ref.read` = one-shot (event handlers). `ref.watch` = subscribe + rebuild widget. `ref.listen` = side-effect callback (không rebuild).

→ BasePage dùng `ref.listen` cho exception + loading → **side-effects**. Subclass dùng `ref.watch` trong `buildPage()` cho **reactive UI**.

### Loading Overlay Lifecycle

```
isLoading: false → true  → _showLoadingOverlay() → OverlayEntry inserted
isLoading: true → false  → _hideLoadingOverlay() → OverlayEntry removed
```

`useState<OverlayEntry?>` giữ reference → đảm bảo remove đúng entry.

### Override Points

| Method | Default | Override khi |
|--------|---------|-------------|
| `handleException` | Delegate to `ExceptionHandler` | Custom error UI cho specific page |
| `buildPageLoading` | `CommonProgressIndicator` | Custom loading widget |
| `onVisibilityChanged` | Log screen view | Additional analytics / refresh data |

---

## 7. Provider Wiring — StateNotifierProvider Pattern 🟡 SHOULD-KNOW

**WHY:** Provider declaration kết nối ViewModel vào Riverpod ecosystem. Sai pattern → memory leak, stale state.

```dart
final loginViewModelProvider =
    StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>(
  (ref) => LoginViewModel(ref),
);
```

### Provider Declaration Breakdown

```dart
StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>
//                   ^^^^^^^^^^^  ^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^
//                   modifier     notifier type   state type
  (ref) => LoginViewModel(ref),
//  ^^^    factory — tạo VM instance
```

### autoDispose Lifecycle

```
Page push → provider first read → LoginViewModel created
    ↓
Page visible → ref.watch/listen active → provider alive
    ↓
Page pop → no more listeners → autoDispose triggers → VM.dispose()
    ↓
Next push → fresh LoginViewModel created (clean state)
```

→ **Memory safe:** không leak state từ page trước.

> 💡 **FE Perspective**
> **Flutter:** `.autoDispose` — provider tự dispose khi page navigate away (không còn listener). Fresh instance mỗi lần navigate lại.
> **React/Vue tương đương:** React `useEffect(() => () => cleanup(), [])` — component unmount trigger cleanup. Vue `onUnmounted()` — Pinia store destroyed khi scoped component unmount.
> **Khác biệt quan trọng:** Riverpod `autoDispose` là **declarative modifier** (gắn trên provider definition) — React/Vue cleanup là **imperative code**.

---

## 8. AppProviderObserver — Lifecycle Logging 🟢 AI-GENERATE

**WHY:** Riverpod providers tạo/dispose silently. Debug state issue → cần visibility vào lifecycle events.

### 4 Lifecycle Hooks

| Hook | Event | Ý nghĩa |
|------|-------|---------|
| `didAddProvider` | Provider initialized | Track creation order |
| `didDisposeProvider` | Provider disposed | Confirm cleanup, detect leaks |
| `didUpdateProvider` | State changed | Debug state transitions |
| `providerDidFail` | Provider threw error | Catch initialization errors |

### Debugging Workflow

1. Bật `logOnDidUpdateProvider` = true
2. Reproduce bug
3. Đọc console → thấy state transitions
4. Identify unexpected state change
5. Trace ngược ViewModel method

---

## 9. Page State vs Shared State 🟡 SHOULD-KNOW

**WHY:** Mix global state với page state → memory leak, stale data, unintended side effects.

### 2 Patterns trong base_flutter

| | Shared State | Page State |
|---|---|---|
| **Provider** | `Provider`, `StateProvider` | `StateNotifierProvider.autoDispose` |
| **Lifecycle** | App-wide (create once) | Page-scoped (create/dispose per navigation) |
| **Access** | Mọi page `ref.read/watch` | Chỉ page owner `ref.watch` |
| **Ví dụ** | `currentUserProvider`, `sharedViewModelProvider` | `loginViewModelProvider`, `homeViewModelProvider` |
| **Reset** | Manual (`ref.invalidate` hoặc set new value) | Automatic (autoDispose on navigate away) |

### Communication Pattern — page → shared → other pages

```
LoginPage login success
    → LoginViewModel: _ref.read(currentUserProvider.notifier).state = user
    → currentUserProvider update
    → HomePage ref.watch(currentUserProvider) → rebuild with new user
    → MyProfilePage ref.watch(currentUserProvider) → rebuild with new user
```

> 💡 **FE Perspective**
> **Flutter:** Shared state = `Provider`/`StateProvider` (app-wide, manual reset). Page state = `StateNotifierProvider.autoDispose` (page-scoped, auto cleanup khi navigate away).
> **React/Vue tương đương:** Shared ≈ Redux global store / Pinia global store. Page ≈ `useState`/`useReducer` (component-local).
> **Khác biệt quan trọng:** Flutter page state **tự động** clean khi navigate away (`.autoDispose`) — React component state cũng tự clean khi unmount, nhưng global store cần manual reset.

---

## Cheat Sheet

### MVVM Flow Summary

```
┌─────────────────────────────────────────────────────────────┐
│  VIEW (BasePage subclass)                                    │
│    buildPage() → ref.watch(provider.select(s => s.data.x))  │
│               → ref.read(provider.notifier).action()        │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  VIEWMODEL (BaseViewModel subclass)                         │
│    setX(value) → data = data.copyWith(x: value)            │
│    action() → runCatching(action: async {...})            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  MODEL (BaseState subclass, freezed)                        │
│    @freezed class XxxState extends BaseState              │
│      const factory XxxState({String x}) = _XxxState;      │
└─────────────────────────────────────────────────────────────┘
```

### Provider Types Quick Reference

| Type | Mutable? | Disposal | Use Case |
|------|----------|----------|----------|
| `Provider` | ❌ | App-wide | DI bridges, services |
| `StateProvider` | ✅ | App-wide | Simple mutable state |
| `StateNotifierProvider.autoDispose` | ✅ | Page-scoped | ViewModel + complex logic |

### ref API Rules

| Method | When | Never |
|--------|------|-------|
| `ref.watch` | build() method | callbacks, event handlers |
| `ref.read` | callbacks, event handlers | build() method |
| `ref.listen` | Side effects (dialog, navigation) | Reactive UI |

---

> 📋 Badge summary → xem [00-overview.md](./00-overview.md)

---

**Tiếp theo:** [03-exercise.md](./03-exercise.md) — thực hành 5 bài tập từ trace flow đến AI Prompt Dojo.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
