# Code Walk — Riverpod & State Management

> 📌 **Module này là canonical source cho Riverpod v1 concepts.** Bạn sẽ đọc provider examples từ simple đến complex, hiểu ref API rules, và thấy decision tree để chọn đúng provider type.

---

## Walk Order

```
ProviderScope (root container)
    ↓
Provider (read-only DI bridge)
    ↓
StateProvider (simple mutable)
    ↓
StateNotifierProvider.autoDispose (ViewModel pattern)
    ↓
FutureProvider (async data)
    ↓
StreamProvider (real-time data)
    ↓
ref API (read/watch/listen/select)
    ↓
ProviderObserver (lifecycle)
    ↓
Testing Overrides (overrideWith)
```

Bắt đầu từ **container setup** → **simple providers** → **complex providers** → **ref API** → **lifecycle** → **testing**.

---

## 1. ProviderScope — Root Container Setup

<!-- AI_VERIFY: base_flutter/lib/main.dart -->
```dart
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
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/main.dart`](../../base_flutter/lib/main.dart)

### 🔎 Quan sát

| Aspect | Detail |
|--------|--------|
| `ProviderScope` | Widget tạo `ProviderContainer` — in-memory store cho tất cả providers |
| `observers` | List `ProviderObserver` — hook vào lifecycle events |
| `overrides` | Dùng trong test: `ProviderScope(overrides: [...])` |

### 💡 FE Perspective

**Flutter:** `ProviderScope` ≈ Redux `<Provider store={store}>` / Pinia `createPinia()`.

**React/Vue tương đương:** React Redux `<Provider>`, Vue Pinia `app.use(pinia)`.

---

## 2. Provider — Read-only DI Bridge

<!-- AI_VERIFY: base_flutter/lib/ui/shared/shared_view_model.dart -->
```dart
final sharedViewModelProvider = Provider((_ref) => SharedViewModel(_ref));

class SharedViewModel {
  SharedViewModel(this._ref);
  final Ref _ref;

  Future<String> get deviceToken async {
    try {
      final deviceToken = await _ref.read(firebaseMessagingServiceProvider).deviceToken;
      return deviceToken ?? '';
    } catch (e) {
      Log.e('Error getting device token: $e');
      return '';
    }
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/shared/shared_view_model.dart`](../../base_flutter/lib/ui/shared/shared_view_model.dart)

### 🔎 Quan sát

| Aspect | Detail |
|--------|--------|
| `Provider<T>` | Read-only — không tự update, chỉ return value |
| `(ref) => T` | Factory function — tạo instance khi first read |
| Không autoDispose | Global — sống suốt app lifecycle |

### Decision: Khi nào dùng Provider?

- Infrastructure services (DI bridge): `Provider((ref) => getIt.get<T>())`
- Computed values: `Provider((ref) => ref.watch(a) + ref.watch(b))`
- Read-only utility classes: `SharedViewModel`

---

## 3. StateProvider — Simple Mutable State

<!-- AI_VERIFY: base_flutter/lib/ui/shared/shared_providers.dart -->
```dart
final currentUserProvider = StateProvider<UserData>((ref) => const UserData());
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/shared/shared_providers.dart`](../../base_flutter/lib/ui/shared/shared_providers.dart)

### 🔎 Quan sát

| Aspect | Detail |
|--------|--------|
| `StateProvider<T>` | Mutable state — read/write trực tiếp |
| `.notifier` | Access notifier: `_ref.read(currentUserProvider.notifier)` |
| `.state` | Get/set value: `_ref.read(currentUserProvider).name` |

### Usage Patterns

```dart
// Read:
final user = ref.watch(currentUserProvider);

// Write:
ref.read(currentUserProvider.notifier).state = newUserData;

// Update specific field:
ref.read(currentUserProvider.notifier).update((state) {
  return state.copyWith(name: 'New Name');
});
```

### Decision: Khi nào dùng StateProvider?

- Simple mutable state: flags, selected items, theme mode
- NO business logic: nếu cần logic → StateNotifierProvider

---

## 4. StateNotifierProvider.autoDispose — ViewModel Pattern

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/view_model/login_view_model.dart -->
```dart
final loginViewModelProvider =
    StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>(
  (ref) => LoginViewModel(ref),
);

class LoginViewModel extends BaseViewModel<LoginState> {
  LoginViewModel(this._ref) : super(const CommonState(data: LoginState()));
  final Ref _ref;

  void setEmail(String email) {
    data = data.copyWith(email: email, onPageError: '');
  }

  FutureOr<void> login() async {
    await runCatching(
      action: () async { ... },
      handleErrorWhen: (_) => false,
      doOnError: (e) async { ... },
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/page/login/view_model/login_view_model.dart`](../../base_flutter/lib/ui/page/login/view_model/login_view_model.dart)

### 🔎 Quan sát

```dart
StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>
//              ^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^
//              modifier            notifier type   state type
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

### Decision: Khi nào dùng StateNotifierProvider.autoDispose?

- Page-level ViewModel: state + business logic
- Complex state: multiple fields, computed values
- Actions: async methods với error handling
- Page-scoped: auto-cleanup khi navigate away

### 💡 FE Perspective

**Flutter:** `StateNotifierProvider.autoDispose` ≈ React `useReducer` + cleanup trong `useEffect`.

**React/Vue tương đương:** React Redux `useSelector` + dispatch. Vue Pinia `storeTo`.

---

## 5. FutureProvider — Async Data

### Pattern

```dart
// One-shot async data — không cần setState:
final configProvider = FutureProvider<AppConfig>((ref) async {
  return await loadConfig();
});

// Watch async value:
final config = ref.watch(configProvider);

// Handle states:
config.when(
  data: (data) => Text(data.apiUrl),
  loading: () => CircularProgressIndicator(),
  error: (e, s) => Text('Error: $e'),
);
```

### vs StateNotifierProvider

| | FutureProvider | StateNotifierProvider |
|---|---|---|
| **Purpose** | One-shot async fetch | Complex state + actions |
| **State** | AsyncValue (data/loading/error) | Custom state class |
| **Updates** | Automatic on future completion | Manual via notifier |
| **Use case** | Initial load, config fetch | Form state, user interactions |

---

## 6. StreamProvider — Real-time Data

### Pattern

```dart
// Real-time data stream:
final messageStreamProvider = StreamProvider<Message>((ref) {
  return messageRepository.watchMessages();
});

// Widget:
ref.watch(messageStreamProvider).when(
  data: (messages) => ListView.builder(
    itemCount: messages.length,
    itemBuilder: (ctx, i) => MessageTile(message: messages[i]),
  ),
  loading: () => CircularProgressIndicator(),
  error: (e, s) => Text('Error: $e'),
);
```

### Use Cases

- WebSocket connections
- Firestore real-time updates
- Location tracking
- Chat message streams

---

## 7. ref API — read vs watch vs listen vs select

### 7a. ref.watch — Reactive Subscription

```dart
// Widget rebuild khi value thay đổi:
Widget build(BuildContext context, WidgetRef ref) {
  final email = ref.watch(
    loginViewModelProvider.select((s) => s.data.email),
  );
  return Text(email);  // rebuild khi email thay đổi
}
```

**RULES:**
- ✅ Chỉ dùng trong `build()` method
- ❌ Không dùng trong callbacks/event handlers
- ✅ Dùng `.select()` để granular rebuild

### 7b. ref.read — One-shot Access

```dart
// Lấy value một lần, không subscribe:
onPressed: () {
  final vm = ref.read(loginViewModelProvider.notifier);
  vm.login();  // call action
}
```

**RULES:**
- ✅ Dùng trong callbacks, event handlers
- ✅ Dùng trong init methods
- ❌ Không dùng trong `build()` (sẽ không rebuild khi value thay đổi)

### 7c. ref.listen — Side-effect Callback

```dart
// Callback khi value thay đổi, KHÔNG rebuild:
ref.listen(
  loginViewModelProvider.select((s) => s.appException),
  (previous, next) {
    if (previous != next && next != null) {
      handleException(next);  // side-effect
    }
  },
);
```

**RULES:**
- ✅ Dùng cho side-effects: dialog, navigation, analytics
- ✅ Dùng ở mọi nơi (build, callbacks, init)
- ❌ KHÔNG dùng cho reactive UI (dùng ref.watch thay thế)

### 7d. ref.select — Granular Subscription

```dart
// Chỉ rebuild khi email thay đổi:
final email = ref.watch(
  loginViewModelProvider.select((s) => s.data.email),
);
// Password thay đổi → KHÔNG rebuild Text(email)
```

**Without select:**
```dart
// ❌ Rebuild khi BẤT KỲ field thay đổi:
final state = ref.watch(loginViewModelProvider);
```

### 💡 FE Perspective

**Flutter:** 4 ref methods với strict rules.

**React/Vue tương đương:**

| Flutter | React | Vue |
|---------|-------|-----|
| `ref.watch` | `useSelector` | `computed` |
| `ref.read` | `store.getState()` / direct | `store.xxx` |
| `ref.listen` | `useEffect` | `watch` |
| `ref.select` | `createSelector` | auto-granular |

---

## 8. ProviderObserver — Lifecycle Debug

<!-- AI_VERIFY: base_flutter/lib/ui/base/app_provider_observer.dart -->
```dart
class AppProviderObserver extends ProviderObserver {
  @override
  void didAddProvider(ProviderBase<Object?> provider, Object? value, ProviderContainer container) {
    if (Config.logOnDidAddProvider) {
      Log.d('didAddProvider: $provider');
    }
  }

  @override
  void didDisposeProvider(ProviderBase<Object?> provider, ProviderContainer container) {
    if (Config.logOnDidDisposeProvider) {
      Log.d('didDisposeProvider: $provider');
    }
  }

  @override
  void didUpdateProvider(ProviderBase<Object?> provider, Object? previousValue, Object? newValue, ProviderContainer container) {
    if (Config.logOnDidUpdateProvider) {
      Log.d('didUpdateProvider: $provider');
    }
  }

  @override
  void providerDidFail(ProviderBase<Object?> provider, Object error, StackTrace stackTrace, ProviderContainer container) {
    if (Config.logOnProviderDidFail) {
      Log.e('providerDidFail: $provider, error: $error');
    }
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/base/app_provider_observer.dart`](../../base_flutter/lib/ui/base/app_provider_observer.dart)

### 🔎 Quan sát

| Hook | Event | Debug Use Case |
|------|-------|----------------|
| `didAddProvider` | Provider created | Verify initialization order |
| `didDisposeProvider` | Provider disposed | Detect memory leaks |
| `didUpdateProvider` | State changed | Trace state transitions |
| `providerDidFail` | Provider threw | Catch initialization errors |

### Config-gated — Zero Production Overhead

```dart
// develop.json:
{ "logOnDidUpdateProvider": true }

// production.json:
{ "logOnDidUpdateProvider": false }
```

---

## 9. Testing Overrides — overrideWith

### Pattern

```dart
// Widget test với mock providers:
testWidgets('LoginPage renders correctly', (tester) async {
  final mockNavigator = MockAppNavigator();
  
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        appNavigatorProvider.overrideWithValue(mockNavigator),
        appPreferencesProvider.overrideWithValue(MockAppPreferences()),
      ],
      child: LoginPage(),
    ),
  );
  
  // Interact với widget...
  await tester.enterText(find.byType(TextField).first, 'test@example.com');
  await tester.tap(find.byType(ElevatedButton));
  await tester.pump();
  
  // Verify...
  verify(mockNavigator.replaceAll(any)).called(1);
});
```

### Override Methods

| Method | Use Case |
|--------|----------|
| `overrideWithValue` | Mock với fake instance |
| `overrideWith` | Mock với factory function |
| `override` | Complex override (all providers) |

---

## 10. Provider Decision Tree — Full Map

### Decision Flowchart

```
Cần mutable state?
├── NO
│   ├── Cần async data?
│   │   ├── ONE-SHOT → FutureProvider
│   │   └── REAL-TIME → StreamProvider
│   └── READ-ONLY
│       ├── DI service? → Provider((ref) => getIt.get<T>())
│       └── Computed? → Provider((ref) => ref.watch(a) + ref.watch(b))
│
└── YES
    ├── Đơn giản (primitive)? → StateProvider
    └── Phức tạp (logic)?
        ├── PAGE-SCOPED → StateNotifierProvider.autoDispose
        └── APP-WIDE → StateNotifierProvider (no autoDispose)
```

---

## ⏭️ Next Steps

Concepts rút ra từ code walk → [02-concept.md](./02-concept.md)

Tóm tắt concepts sẽ cover:
1. ProviderScope & Container
2. Provider Types Taxonomy
3. ref API (read/watch/listen/select)
4. autoDispose & family
5. DI Bridge Pattern
6. ProviderContainer
7. ProviderObserver
8. FutureProvider & StreamProvider
9. Testing Overrides
10. State Management Decision Tree

Forward ref: [Module 12 — Data Layer](../module-12-data-layer/) sẽ combine Riverpod với Flutter Hooks.

<!-- AI_VERIFY: generation-complete -->
