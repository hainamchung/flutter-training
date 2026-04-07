# Concepts — Riverpod & State Management

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. ProviderScope & Container — Root Setup 🔴 MUST-KNOW

**WHY:** Không có `ProviderScope` → mọi `ref.read/watch/listen` crash. Đặt sai vị trí → provider resolve trước DI init → crash.

### Architecture

```
ProviderScope (root widget)
└── ProviderContainer (hidden state store)
    ├── appNavigatorProvider → AppNavigator instance
    ├── currentUserProvider → UserData value
    ├── loginViewModelProvider → LoginViewModel + CommonState
    └── ... mọi provider khác
```

### Key Points

| Aspect | Detail |
|--------|--------|
| **ProviderScope** | Widget tạo `ProviderContainer` — in-memory store cho tất cả providers |
| **Single root** | Chỉ 1 ProviderScope (root) — nested scopes cho override (testing) |
| **observers** | List `ProviderObserver` — hook vào lifecycle events |
| **overrides** | Dùng trong test: `ProviderScope(overrides: [providerX.overrideWithValue(...)])` |

### ProviderContainer — Invisible State Store

Container **lazy-create** providers — chỉ resolve khi lần đầu `ref.read/watch` gọi tới.

> 💡 **FE Perspective**
> **Flutter:** `ProviderScope` — root widget tạo `ProviderContainer` in-memory, hỗ trợ `overrides` array cho testing, `observers` cho lifecycle debug.
> **React/Vue tương đương:** React Redux `<Provider store={store}>` / Vue `app.use(createPinia())`.
> **Khác biệt quan trọng:** Riverpod override ở **provider level** (granular) — React/Vue override ở **store level** (toàn bộ).

---

## 2. Provider Types Taxonomy 🔴 MUST-KNOW

**WHY:** Chọn sai provider type → overcomplicate code hoặc thiếu reactivity.

### Complete Taxonomy

| Type | Mutable? | Disposal | Dùng cho | Ví dụ |
|------|----------|----------|----------|----------------|
| `Provider` | ❌ Read-only | App-wide | DI bridges, services, computed values | `appNavigatorProvider`, `sharedViewModelProvider` |
| `StateProvider` | ✅ Simple | App-wide | Primitive state, flags | `currentUserProvider` |
| `StateNotifierProvider` | ✅ Complex | Optional | ViewModel + business logic | `loginViewModelProvider` |
| `FutureProvider` | ❌ Async read | Optional | One-shot async data | Config load, initial fetch |
| `StreamProvider` | ❌ Async stream | Optional | Real-time data (WebSocket, Firestore) | Message streams |

### Decision Tree — When to Use Each

```
Cần mutable state?
├── NO
│   ├── Async data?
│   │   ├── ONE-SHOT fetch → FutureProvider
│   │   └── REAL-TIME stream → StreamProvider
│   └── READ-ONLY
│       ├── DI service? → Provider((ref) => getIt.get<T>())
│       └── Computed? → Provider((ref) => ref.watch(a) + ref.watch(b))
│
└── YES
    ├── Đơn giản (string, int, bool)?
    │   └── StateProvider
    └── Phức tạp (multiple fields, logic)?
        ├── PAGE-SCOPED → StateNotifierProvider.autoDispose
        └── APP-WIDE → StateNotifierProvider (no autoDispose)
```

### Quick Decision Guide

| Scenario | Provider Type |
|----------|---------------|
| Navigation service | `Provider` |
| Theme mode | `StateProvider<ThemeMode>` |
| User login state | `StateProvider<bool>` hoặc `StateNotifierProvider` |
| Form state với validation | `StateNotifierProvider.autoDispose` |
| API config | `FutureProvider<AppConfig>` |
| Chat messages | `StreamProvider<Message>` |

> 💡 **FE Perspective**
> **Flutter:** 4+ provider types phân biệt — `Provider` (read-only DI), `StateProvider` (simple mutable), `StateNotifierProvider` (complex logic), `FutureProvider` (async), `StreamProvider` (real-time).
> **React/Vue tương đương:** `Provider` ≈ `useContext`/`provide-inject`. `StateProvider` ≈ `useState`/`ref()`. `StateNotifierProvider` ≈ `useReducer`/Pinia `defineStore`. `FutureProvider` ≈ React Query `useQuery`.

---

## 3. ref API — read vs watch vs listen vs select 🔴 MUST-KNOW

**WHY:** Dùng sai ref method → missing updates, unnecessary rebuilds, hoặc crash.

### 4 Methods Overview

| Method | Behavior | Khi nào dùng | Never dùng trong |
|--------|----------|---------------|------------------|
| `ref.read(p)` | One-shot, **không** subscribe | Event handlers, callbacks | `build()` method |
| `ref.watch(p)` | Subscribe + **rebuild** widget | `build()` method — display data | Callbacks |
| `ref.listen(p, cb)` | Subscribe + **callback** | Side effects: dialog, navigation, analytics | Reactive UI |
| `p.select(fn)` | Subscribe + **granular** rebuild | Chỉ rebuild khi selected field đổi | — |

### Rules — QUAN TRỌNG

```dart
// ✅ ĐÚNG — ref.watch trong build():
Widget build(context, ref) {
  final email = ref.watch(provider.select((s) => s.data.email));
  return Text(email);
}

// ❌ SAI — ref.watch trong callback:
onPressed: () {
  final email = ref.watch(provider); // CRASH! — watch ngoài build
}

// ✅ ĐÚNG — ref.read trong callback:
onPressed: () {
  ref.read(provider.notifier).login(); // OK — one-shot read
}
```

### 🚨 Common Trap cho FE Devs

```dart
// ❌ Gọi ref.watch trong button callback — CRASH:
ElevatedButton(
  onPressed: () {
    final vm = ref.watch(loginViewModelProvider.notifier);
    //              ^^^ Throws: "ref.watch was called from outside build()"
    vm.login();
  },
)

// ✅ Fix: dùng ref.read trong callback:
ElevatedButton(
  onPressed: () {
    ref.read(loginViewModelProvider.notifier).login();
  },
)
```

### ref.listen vs ref.watch

| Method | Rebuilt Widget? | Return Value | Use Case |
|--------|-----------------|--------------|----------|
| `ref.watch(p)` | ✅ Yes | Current value | Reactive UI display |
| `ref.listen(p, cb)` | ❌ No | `void` | Side effects only |

### Selector Pattern — Granular Rebuilds

```dart
// ❌ Without selector — rebuild khi BẤT KỲ field đổi:
final state = ref.watch(loginViewModelProvider);
final email = state.data.email;
final password = state.data.password;

// ✅ With select — chỉ rebuild khi email đổi:
final email = ref.watch(loginViewModelProvider.select((s) => s.data.email));
// Password đổi → KHÔNG rebuild
```

### Performance Impact

| Pattern | User gõ 10 ký tự | Build count |
|---------|-------------------|-------------|
| `ref.watch(provider)` | 10 rebuilds | ❌ 10 (unnecessary) |
| `ref.watch(provider.select((s) => s.data.password))` | 0 rebuilds | ✅ 0 |

> 💡 **FE Perspective**
> **Flutter:** 4 ref methods với strict rules — `ref.read` (one-shot), `ref.watch` (rebuild), `ref.listen` (side-effect), `ref.select` (granular).
> **React/Vue tương đương:** `ref.read` ≈ `store.getState()`. `ref.watch` ≈ `useSelector()`. `ref.listen` ≈ `useEffect` with dependency. `ref.select` ≈ `createSelector`.
> **Khác biệt quan trọng:** Flutter **crash** nếu dùng `ref.watch` ngoài `build()` — React `useSelector` warning-only.

---

## 4. autoDispose & family Modifiers 🟡 SHOULD-KNOW

**WHY:** Thiếu `autoDispose` → memory leak khi navigate away. `family` cho phép parameterized providers.

### autoDispose — Page-scoped VM

```dart
final loginViewModelProvider =
    StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>(
  (ref) => LoginViewModel(ref),
);
```

### Lifecycle

```
Navigate to LoginPage → ref.watch/loginViewModelProvider lần đầu
    → Provider created → LoginViewModel() → state = CommonState(data: LoginState())

Navigate away → no more listeners
    → autoDispose triggers → LoginViewModel.dispose()
    → state wiped clean

Navigate back → fresh instance created (no stale state)
```

### When to Use autoDispose

| | autoDispose | No modifier |
|---|-------------|-------------|
| **Page-level VM** | ✅ Luôn | ❌ |
| **Global shared state** | ❌ | ✅ |
| **SharedViewModel** | ❌ | ✅ |

### family Modifier — Parameterized Providers

```dart
// Mỗi userId tạo provider instance riêng:
final userDetailProvider = FutureProvider.autoDispose.family<User, String>((ref, userId) async {
  return await api.getUser(userId);
});

// Trong widget:
final user = ref.watch(userDetailProvider('123'));
```

> 💡 **FE Perspective**
> **Flutter:** `.autoDispose` — provider tự dispose khi không còn listener. `.family` — parameterized provider.
> **React/Vue tương đương:** `autoDispose` ≈ `useEffect` cleanup. `family` ≈ factory hook `useItemStore(id)`.

---

## 5. DI Bridge Pattern 🟡 SHOULD-KNOW

**WHY:** base_flutter dùng **2 DI systems** — getIt (infrastructure) + Riverpod (state). Bridge pattern thống nhất access qua `ref` API.

### Pattern

```dart
// ❌ Direct getIt — hard dependency:
final navigator = getIt.get<AppNavigator>();

// ✅ Bridge qua Provider — injectable, override via ProviderScope:
_ref.read(appNavigatorProvider).replaceAll([...]);
```

### Testing Override

```dart
// In test:
ProviderScope(
  overrides: [
    apiClientProvider.overrideWithValue(MockApiClient()),
    appNavigatorProvider.overrideWithValue(MockNavigator()),
  ],
  child: MyApp(),
);
```

### Rules

- **Infrastructure services** (Navigator, Preferences, Firebase): getIt register + Provider wrap
- **ViewModels**: Riverpod native (`StateNotifierProvider`) — không cần bridge
- **Shared state**: Riverpod native (`StateProvider`, `Provider`)

---

## 6. ProviderContainer — Manual Container Management 🟡 SHOULD-KNOW

**WHY:** Dùng ProviderContainer để manually manage providers trong tests, background tasks, hoặc advanced patterns.

### Usage

```dart
// Create container với overrides:
final container = ProviderContainer(
  overrides: [
    userRepositoryProvider.overrideWithValue(MockUserRepository()),
  ],
);

// Read from container (without widget):
final user = container.read(currentUserProvider);

// Clean up:
container.dispose();
```

### Common Use Cases

| Use Case | Pattern |
|----------|---------|
| Unit test | `ProviderContainer` + `overrides` |
| Background task | `ProviderContainer` + `dispose()` |
| Custom provider tree | Nested `ProviderScope` |

---

## 7. ProviderObserver — Lifecycle Logging 🟡 SHOULD-KNOW

**WHY:** Riverpod providers tạo/dispose silently. Debug state issue → cần visibility vào lifecycle events.

### 4 Lifecycle Hooks

| Hook | Event | Debug Use Case |
|------|-------|----------------|
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

## 8. FutureProvider & StreamProvider 🟡 SHOULD-KNOW

### FutureProvider — One-shot Async

```dart
final configProvider = FutureProvider<AppConfig>((ref) async {
  return await loadConfig();
});

// Widget:
ref.watch(configProvider).when(
  data: (config) => Text(config.apiUrl),
  loading: () => CircularProgressIndicator(),
  error: (e, s) => Text('Error: $e'),
);
```

### StreamProvider — Real-time Data

```dart
final messageStreamProvider = StreamProvider<Message>((ref) {
  return messageRepository.watchMessages();
});

// Widget:
ref.watch(messageStreamProvider).when(
  data: (messages) => ListView.builder(...),
  loading: () => CircularProgressIndicator(),
  error: (e, s) => Text('Error: $e'),
);
```

### When to Use Each

| Scenario | Provider Type |
|----------|---------------|
| Initial app config | `FutureProvider` |
| User profile fetch | `FutureProvider` |
| WebSocket messages | `StreamProvider` |
| Firestore collection | `StreamProvider` |

---

## 9. Testing Overrides 🟡 SHOULD-KNOW

**WHY:** Override providers để inject mocks trong tests.

### overrideWithValue

```dart
await tester.pumpWidget(
  ProviderScope(
    overrides: [
      appNavigatorProvider.overrideWithValue(mockNavigator),
      appPreferencesProvider.overrideWithValue(MockPreferences()),
    ],
    child: LoginPage(),
  ),
);
```

### overrideWith (Factory)

```dart
appNavigatorProvider.overrideWith(
  (ref) => MockAppNavigator(),
)
```

---

## 10. State Management Decision Tree — Complete 🔴 MUST-KNOW

### Visual Flowchart

```
START: Bạn cần quản lý state?
│
├─ NO → Cần DI access cho service?
│   │   └─ YES → Provider((ref) => getIt.get<T>())
│   │
│   └─ NO → Cần computed value?
│       └─ YES → Provider((ref) => ref.watch(a) + ref.watch(b))
│
├─ YES → Cần async data?
│   ├─ ONE-SHOT fetch
│   │   └─ FutureProvider
│   │
│   └─ REAL-TIME stream
│       └─ StreamProvider
│
└─ YES → Cần mutable state?
    ├─ Simple (primitive)?
    │   └─ StateProvider
    │
    └─ Complex (logic/actions)?
        ├─ Page-scoped?
        │   └─ StateNotifierProvider.autoDispose
        │
        └─ App-wide?
            └─ StateNotifierProvider
```

### Quick Reference Table

| Bạn cần... | Dùng provider này |
|------------|---------------------|
| Navigation, Preferences, Firebase | `Provider((ref) => getIt.get<T>())` |
| Theme mode, selected tab | `StateProvider<T>` |
| Form validation, API calls | `StateNotifierProvider.autoDispose` |
| User session, global cart | `StateNotifierProvider` (no autoDispose) |
| API fetch, config load | `FutureProvider` |
| Chat messages, live updates | `StreamProvider` |
| Provider với parameter | `.family` modifier |

---

## Cheat Sheet

### Provider Types Quick Reference

```
┌─────────────────────────────────────────────────────────┐
│  PROVIDER TAXONOMY                                       │
│                                                          │
│  Read-only (Immutable)                                   │
│  ├── Provider          → DI, computed values           │
│  ├── FutureProvider     → async data (one-shot)         │
│  └── StreamProvider     → async data (real-time)        │
│                                                          │
│  Mutable (Changeable)                                    │
│  ├── StateProvider      → simple state (primitive)     │
│  └── StateNotifierProvider                           │
│      ├── .autoDispose    → page-scoped                 │
│      └── no modifier     → app-wide                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### ref API Quick Reference

```
┌─────────────────────────────────────────────────────────┐
│  ref API                                                 │
│                                                          │
│  ref.watch(provider)    → Subscribe + REBUILD widget    │
│    ✅ Trong build() method                              │
│    ❌ Trong callbacks                                    │
│                                                          │
│  ref.read(provider)     → One-shot, NO subscribe         │
│    ✅ Trong callbacks, event handlers                    │
│    ❌ Trong build() (sẽ không trigger rebuild)          │
│                                                          │
│  ref.listen(provider, callback) → Subscribe + CALLBACK   │
│    ✅ Side effects (dialog, navigation, analytics)       │
│    ✅ Ở mọi nơi (build, callbacks, init)               │
│                                                          │
│  provider.select(fn)  → Granular subscription          │
│    ✅ Kết hợp với ref.watch/listen                     │
│    ✅ Chỉ rebuild khi selected value đổi                │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

> 📋 Badge summary → xem [00-overview.md](./00-overview.md)

---

**Tiếp theo:** [03-exercise.md](./03-exercise.md) — thực hành 5 bài tập.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
