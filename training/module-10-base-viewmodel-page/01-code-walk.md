# Code Walk — BaseViewModel & BasePage (MVVM Pattern)

> 📌 **Module này là canonical source cho MVVM concepts.** Bạn sẽ đọc toàn bộ chain từ BaseState → CommonState → BaseViewModel → BasePage → LoginViewModel → LoginPage → Observer. Hiểu cách mọi thành phần kết nối tạo thành reactive MVVM pipeline.

<details>
<summary>📋 Recap: M2-M7 concepts (click to expand)</summary>

> - **M2:** DI với `get_it`, Riverpod Provider expose DI — M10 ViewModel dùng `Ref` để read providers
> - **M3:** `Log` utility, `Config` flags — M10 `mounted` guard dùng `Log.e()`
> - **M4:** `AppException` hierarchy — M10 `runCatching` wrap exceptions, build retry chain
> - **M5:** `AppNavigator`, `replaceAll` — M10 LoginViewModel navigate sau login success
> - **M6:** `AppColors.of(context)` init — M10 `BasePage.build()` gọi đầu tiên
> - **M7:** `BaseState`, `CommonState`, `BaseViewModel`, `BasePage` — từng file detail

Nếu chưa nắm vững → quay lại module tương ứng trước khi tiếp tục.
</details>

---

## Walk Order

```
base_state.dart (abstract marker)
    ↓
common_state.dart (generic envelope)
    ↓
base_view_model.dart (StateNotifier + lifecycle + runCatching)
    ↓
base_page.dart (HookConsumerWidget + ref.listen)
    ↓
login_state.dart (concrete state)
    ↓
login_view_model.dart (StateNotifierProvider.autoDispose)
    ↓
login_page.dart (BasePage subclass)
    ↓
app_provider_observer.dart (lifecycle debug)
```

Bắt đầu từ **abstract contract** (BaseState) → **generic wrapper** (CommonState) → **ViewModel logic** (BaseViewModel) → **View binding** (BasePage) → **concrete example** (Login flow) → **debug tool** (Observer).

---

## 1. base_state.dart — Abstract State Contract

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_state.dart -->
```dart
abstract class BaseState {
  const BaseState();
}
```
<!-- END_VERIFY -->

### 🔎 Quan sát

| Aspect | Detail |
|--------|--------|
| **Lines** | 3 — minimal abstract class |
| **Marker pattern** | Không có method/field — chỉ đánh dấu "đây là state class" |
| **`abstract`** | Không instantiate trực tiếp — compiler báo lỗi nếu cố `BaseState()` |
| **`const` constructor** | Cho phép state subclass có `const factory` → compile-time constants |

→ [Mở file gốc: `lib/ui/base/base_state.dart`](../../base_flutter/lib/ui/base/base_state.dart)

### 💡 FE Perspective

**Flutter:** `BaseState` abstract class là marker type cho generic constraint — mọi page state (LoginState, ProfileState) phải extend nó để dùng với `CommonState<T>` và `BaseViewModel<T>`.

**React/Vue tương đương:** TypeScript `interface BaseState {}` marker (React), base state type mà mọi store extend trong Pinia (Vue).

---

## 2. common_state.dart — Generic Envelope Pattern

<!-- AI_VERIFY: base_flutter/lib/ui/base/common_state.dart -->
```dart
@freezed
sealed class CommonState<T extends BaseState> with _$CommonState<T> {
  const CommonState._();
  const factory CommonState({
    required T data,
    AppException? appException,
    @Default(false) bool isLoading,
    @Default(false) bool isFirstLoading,
    @Default(<String, bool>{}) Map<String, bool> doingAction,
  }) = _CommonState<T>;

  bool isDoingAction(String actionName) => doingAction[actionName] == true;
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/base/common_state.dart`](../../base_flutter/lib/ui/base/common_state.dart)

### 🔎 Quan sát

**Envelope = wrapper tách concerns:**

```
┌─ CommonState<LoginState> ──────────────┐
│  data: LoginState(email, password)     │ ← Business data (varies per page)
│  appException: null                     │ ← Error state (shared)
│  isLoading: false                       │ ← Loading state (shared)
│  isFirstLoading: false                  │ ← First-load flag (shared)
│  doingAction: {'login': true}          │ ← Per-action tracking (shared)
└─────────────────────────────────────────┘
```

**Tại sao wrap thay vì extend?**

```dart
// ❌ Extend approach — mỗi state tự define:
@freezed class LoginState {
  String email;
  String password;
  bool isLoading;        // duplicate!
  AppException? error;   // naming inconsistent
}

// ✅ Wrap approach — tách rõ ràng:
CommonState<LoginState>(
  data: LoginState(email: '', password: ''),  // business
  isLoading: false,                            // infra
)
```

**`doingAction` — Map-based action tracking:**

```dart
// Set:
state = state.copyWith(doingAction: {...state.doingAction, 'login': true});

// Query:
state.isDoingAction('login') // → true

// UI:
ref.watch(provider.select((s) => s.isDoingAction('login')))
// → disable login button khi đang submit
```

### 💡 FE Perspective

**Flutter:** `CommonState<T>` envelope tách business data (`data: T`) khỏi infrastructure fields (`isLoading`, `appException`, `doingAction`).

**React/Vue tương đương:** Redux state envelope `{ data: T, loading: boolean, error: Error }` (React), Pinia store state wrapper (Vue).

---

## 3. base_view_model.dart — StateNotifier + Lifecycle

### 3a. Class Structure

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_view_model.dart -->
```dart
abstract class BaseViewModel<ST extends BaseState>
    extends StateNotifier<CommonState<ST>> {
  BaseViewModel(CommonState<ST> initialState) : super(initialState) {
    this.initialState = initialState;
  }
  late final CommonState<ST> initialState;

  @override
  set state(CommonState<ST> value) {
    if (mounted) { super.state = value; }
    else { Log.e('Cannot set state when widget is not mounted'); }
  }

  set data(ST data) {
    if (mounted) { state = state.copyWith(data: data); }
    else { Log.e('Cannot set data when widget is not mounted'); }
  }

  ST get data => state.data;
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/base/base_view_model.dart`](../../base_flutter/lib/ui/base/base_view_model.dart)

### 🔎 Quan sát

**MVVM — ViewModel extends StateNotifier:**

```
┌─ View (Widget) ─────────────────────────────┐
│  BasePage extends HookConsumerWidget        │
│  buildPage() → ref.watch(provider)         │
│  ref.watch(provider.select((s) => s.data)) │
└────────────────────┬───────────────────────┘
                     │ watch
                     ▼
┌─ ViewModel ─────────────────────────────────┐
│  BaseViewModel extends StateNotifier        │
│  ├─ CommonState<ST>                        │
│  ├─ data setter/getter                      │
│  ├─ showLoading() / hideLoading()          │
│  └─ runCatching()                          │
└────────────────────┬────────────────────────┘
                     │ update state
                     ▼
┌─ Model (Data) ────────────────────────────┐
│  LoginState extends BaseState              │
│  email, password, onPageError              │
└────────────────────────────────────────────┘
```

**`mounted` guard — defensive programming:**

```
Timeline:
t0: User opens LoginPage → LoginViewModel created, mounted = true
t1: login() called → API request starts (async)
t2: User presses Back → LoginPage disposed → mounted = false
t3: API response arrives → set state → 💥 (without guard)
```

Với guard:
```dart
set state(CommonState<ST> value) {
  if (mounted) { super.state = value; }    // ✅ safe update
  else { Log.e('...'); }                    // ⚠️ log, don't crash
}
```

> ⚠️ **Flutter không auto-cancel async khi widget unmount** (khác React useEffect cleanup). Trong BaseViewModel, `dispose()` được gọi — nhưng pending Futures vẫn complete. Dùng `CancelToken` (Dio) hoặc check `mounted` trước setState.

### 💡 FE Perspective

**Flutter:** `BaseViewModel` extends `StateNotifier<CommonState<ST>>` — ViewModel là state container với built-in lifecycle management.

**React/Vue tương đương:** React `useReducer` hook với custom wrapper class, Vue `defineStore` với composition API.

---

### 3b. Loading Counter Pattern

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_view_model.dart -->
```dart
int _loadingCount = 0;
bool firstLoadingShown = false;

void showLoading() {
  if (_loadingCount <= 0) {
    state = state.copyWith(
      isLoading: true,
      isFirstLoading: !firstLoadingShown && _loadingCount == 0,
    );
    firstLoadingShown = true;
  }
  _loadingCount++;
}

void hideLoading() {
  if (!mounted) { return; }
  if (_loadingCount <= 1) {
    state = state.copyWith(
      isLoading: false,
      isFirstLoading: false,
    );
  }
  _loadingCount--;
}
```
<!-- END_VERIFY -->

### 🔎 Quan sát

**Tại sao counter thay vì bool?**

Parallel calls scenario:
```
showLoading() call 1: count=0 → enters if block → isLoading=true → count becomes 1
showLoading() call 2: count=1 → skips if block (not <= 0) → count becomes 2
hideLoading() call 1: count=2 → skips if block (not <= 1) → count becomes 1
hideLoading() call 2: count=1 → enters if block → isLoading=false → count becomes 0 ✅
```

Bool approach fails: `showLoading()` → `isLoading=true`, `hideLoading()` → `isLoading=false` — nhưng notification call còn đang chạy!

**Key insight:** `_loadingCount++` runs AFTER the isLoading check, so multiple concurrent `showLoading()` calls won't redundantly set `isLoading=true`.

---

### 3c. runCatching — Centralized Error Handling

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_view_model.dart -->
```dart
Future<void> runCatching({
  required Future<void> Function() action,
  Future<void> Function()? doOnRetry,
  Future<void> Function(AppException)? doOnError,
  Future<void> Function()? doOnSuccessOrError,
  Future<void> Function()? doOnCompleted,
  bool handleLoading = true,
  FutureOr<bool> Function(AppException)? handleRetryWhen,
  FutureOr<bool> Function(AppException)? handleErrorWhen,
  int? maxRetries = 2,
  String? actionName,
}) async {
  assert(maxRetries == null || maxRetries >= 0, 'maxRetries must be positive');
  try {
    if (handleLoading) {
      showLoading();
      if (actionName != null) {
        startAction(actionName);
      }
    }
    await action.call();
    if (handleLoading) {
      hideLoading();
      if (actionName != null) {
        stopAction(actionName);
      }
    }
    await doOnSuccessOrError?.call();
  } on Object catch (e) {
    final appException = e is AppException ? e : AppUncaughtException(rootException: e);
    if (handleLoading) {
      hideLoading();
      if (actionName != null) {
        stopAction(actionName);
      }
    }
    await doOnSuccessOrError?.call();
    await doOnError?.call(appException);
    if (await handleErrorWhen?.call(appException) != false ||
        appException.isForcedErrorToHandle) {
      final shouldRetryAutomatically = await handleRetryWhen?.call(appException) != false &&
          (maxRetries == null || maxRetries - 1 >= 0);
      if (shouldRetryAutomatically || doOnRetry != null) {
        appException.onRetry = () async {
          if (doOnRetry != null) { await doOnRetry.call(); }
          if (shouldRetryAutomatically) {
            await runCatching(
              action: action,
              doOnCompleted: doOnCompleted,
              doOnSuccessOrError: doOnSuccessOrError,
              doOnError: doOnError,
              doOnRetry: doOnRetry,
              handleErrorWhen: handleErrorWhen,
              handleLoading: handleLoading,
              handleRetryWhen: handleRetryWhen,
              maxRetries: maxRetries?.minus(1),
            );
          }
        };
      }
      exception = appException;
    }
  } finally {
    await doOnCompleted?.call();
  }
}
```
<!-- END_VERIFY -->

### 🔎 Quan sát

**Basic pattern:**

```dart
// ❌ Without runCatching — 30+ lines boilerplate per method:
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

// ✅ With runCatching — chỉ business logic:
Future<void> login() async {
  await runCatching(
    action: () async => await _doLogin(),
    actionName: 'login',
  );
}
```

**3 error handling strategies qua `handleErrorWhen`:**

| Strategy | `handleErrorWhen` | `doOnError` | Kết quả |
|----------|-------------------|-------------|---------|
| **Global dialog** | `null` (default true) | `null` | Exception → `ExceptionHandler` → dialog |
| **Inline error** | `(_) => false` | set `onPageError` | Error hiển thị trên form, không popup |
| **Mixed** | conditional | set local + partial global | Một số error inline, một số popup |

**Retry chain — recursive pattern:**

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

### 💡 FE Perspective

**Flutter:** `runCatching` ≈ React Query's `useMutation({ retry, onError, onSettled })` — centralized error handling cho async actions.

**React/Vue tương đương:** React Query `useMutation`, SWR `mutate`, Pinia store action wrapper.

---

## 4. base_page.dart — Reactive UI Binding

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_page.dart -->
```dart
abstract class BasePage<ST extends BaseState, P extends ProviderListenable<CommonState<ST>>>
    extends HookConsumerWidget {
  const BasePage({super.key});

  P get provider;
  ScreenViewEvent get screenViewEvent;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    AppColors.of(context);
    final loadingOverlayEntry = useState<OverlayEntry?>(null);

    ref.listen(provider.select((value) => value.appException), (previous, next) {
      if (previous != next && next != null) {
        handleException(next, ref);
      }
    });

    ref.listen(provider.select((value) => value.isLoading), (previous, next) {
      if (next == true && loadingOverlayEntry.value == null) {
        _showLoadingOverlay(context: context, loadingOverlayEntry: loadingOverlayEntry);
      } else if (previous == true && next == false && loadingOverlayEntry.value != null) {
        _hideLoadingOverlay(loadingOverlayEntry);
      }
    });

    return FocusDetector(
      key: Key(screenViewEvent.fullKey),
      onVisibilityGained: () => onVisibilityChanged(ref),
      child: buildPage(context, ref),
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/base/base_page.dart`](../../base_flutter/lib/ui/base/base_page.dart)

### 🔎 Quan sát

**HookConsumerWidget — best of both:**

| Base class | Riverpod | Hooks | Use case |
|------------|----------|-------|----------|
| `ConsumerWidget` | ✅ `ref` | ❌ | Simple read/watch |
| `HookWidget` | ❌ | ✅ `useState` | Local state |
| `HookConsumerWidget` | ✅ | ✅ | **BasePage** — cần cả hai |

**`ref.listen` vs `ref.watch` trong BasePage:**

| API | Behavior | Use case trong BasePage |
|-----|----------|------------------------|
| `ref.listen(provider.select(...))` | **Side-effect** — callback fires khi value thay đổi, **không** trigger rebuild | Exception dialog, loading overlay |
| `ref.watch(provider)` | **Rebuild** — widget rebuild khi state thay đổi | Data display trong `buildPage()` |
| `ref.read(provider)` | **One-shot** — lấy value hiện tại, **không** subscribe | Event handlers, callbacks |

**BasePage consumption summary:**

```
┌─ BasePage.build() ────────────────────────────────────┐
│                                                        │
│  ref.listen(select: appException) → handleException()  │
│  ref.listen(select: isLoading)    → show/hide overlay  │
│                                                        │
│  buildPage(context, ref)  ← concrete page override     │
│    └─ ref.watch(provider.select(...)) → rebuild UI     │
│    └─ ref.read(provider.notifier)     → call VM method │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 💡 FE Perspective

**Flutter:** `BasePage` extends `HookConsumerWidget` — kết hợp Flutter Hooks (`useState`) và Riverpod `ref` API.

**React/Vue tương đương:** React class component với HOC wrapper (connect Redux) + hooks bên trong.

---

## 5. Login Flow — Complete MVVM Example

### 5a. login_state.dart — Concrete Model

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/view_model/login_state.dart -->
```dart
@freezed
sealed class LoginState extends BaseState with _$LoginState {
  const factory LoginState({
    @Default('') String email,
    @Default('') String password,
    @Default('') String onPageError,
  }) = _LoginState;

  const LoginState._();

  bool get isLoginButtonEnabled =>
      email.isNotEmpty && password.isNotEmpty;
}
```
<!-- END_VERIFY -->

### 🔎 Quan sát

**LoginState extends BaseState:**

```
┌─ BaseState (abstract marker) ──────────────────────┐
│                                                   │
│  ┌─ LoginState (concrete) ──────────────────────┐│
│  │  email: String                                 ││
│  │  password: String                             ││
│  │  onPageError: String                          ││
│  │  isLoginButtonEnabled: bool (computed)        ││
│  └────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────┘
```

`isLoginButtonEnabled` computed getter — business logic trong Model layer. Chỉ check `email.isNotEmpty && password.isNotEmpty` — KHÔNG check `onPageError` vì error được clear khi user type.

### 5b. login_view_model.dart — Concrete ViewModel

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

  void setPassword(String password) {
    data = data.copyWith(password: password, onPageError: '');
  }

  FutureOr<void> login() async {
    await runCatching(
      action: () async {
        final email = data.email.trim();
        final deviceToken = await _ref.read(sharedViewModelProvider).deviceToken;
        Log.d('deviceToken: $deviceToken'.hardcoded);

        await Future.wait([
          _ref.read(appPreferencesProvider).saveAccessToken('mock_access_token'),
          _ref.read(appPreferencesProvider).saveRefreshToken('mock_refresh_token'),
          _ref.read(appPreferencesProvider).saveIsLoggedIn(true),
        ]);

        await _ref.read(appNavigatorProvider).replaceAll([MainRoute()]);
      },
      handleErrorWhen: (_) => false,    // suppress global dialog
      doOnError: (e) async {
        data = data.copyWith(onPageError: e.message);  // inline error
      },
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/page/login/view_model/login_view_model.dart`](../../base_flutter/lib/ui/page/login/view_model/login_view_model.dart)

### 🔎 Quan sát

**Provider declaration:**

```dart
StateNotifierProvider.autoDispose<LoginViewModel, CommonState<LoginState>>
//              ^^^^^^^^^^^^^^^^^^^^^^^    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//              modifier + type            generic state type
  (ref) => LoginViewModel(ref),
//  ^^^    factory — tạo VM instance
```

**autoDispose lifecycle:**

```
Page push → provider first read → LoginViewModel created
    ↓
Page visible → ref.watch/listen active → provider alive
    ↓
Page pop → no more listeners → autoDispose triggers → VM.dispose()
    ↓
Next push → fresh LoginViewModel created (clean state)
```

### 5c. login_page.dart — Concrete View

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart -->
```dart
@RoutePage()
class LoginPage extends BasePage<LoginState,
    AutoDisposeStateNotifierProvider<LoginViewModel, CommonState<LoginState>>> {
  const LoginPage({super.key});

  @override
  ScreenViewEvent get screenViewEvent => ScreenViewEvent(screenName: ScreenName.loginPage);

  @override
  AutoDisposeStateNotifierProvider<LoginViewModel, CommonState<LoginState>> get provider =>
      loginViewModelProvider;

  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    final scrollController = useScrollController();

    return CommonScaffold(
      body: Stack(
        children: [
          // Background
          CommonImage.asset(path: image.imageBackground, ...),
          // Form
          CommonScrollbarWithIosStatusBarTapDetector(
            routeName: LoginRoute.name,
            controller: scrollController,
            child: SingleChildScrollView(
              controller: scrollController,
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const SizedBox(height: 100),
                  CommonText(l10n.login, ...),
                  const SizedBox(height: 50),
                  // Email field
                  PrimaryTextField(
                    title: l10n.email,
                    onChanged: (email) => ref.read(provider.notifier).setEmail(email),
                    ...
                  ),
                  const SizedBox(height: 24),
                  // Password field
                  PrimaryTextField(
                    title: l10n.password,
                    onChanged: (password) => ref.read(provider.notifier).setPassword(password),
                    ...
                  ),
                  // Error message
                  Consumer(
                    builder: (context, ref, child) {
                      final onPageError = ref.watch(
                        provider.select((value) => value.data.onPageError),
                      );
                      return Visibility(
                        visible: onPageError.isNotEmpty,
                        child: CommonText(onPageError, ...),
                      );
                    },
                  ),
                  // Login button
                  Consumer(
                    builder: (context, ref, child) {
                      final isLoginButtonEnabled = ref.watch(
                        provider.select((value) => value.data.isLoginButtonEnabled),
                      );
                      return ElevatedButton(
                        onPressed: isLoginButtonEnabled
                            ? () => ref.read(provider.notifier).login()
                            : null,
                        ...
                      );
                    },
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/page/login/login_page.dart`](../../base_flutter/lib/ui/page/login/login_page.dart)

### 🔎 Quan sát

**Complete MVVM flow:**

```
┌─ VIEW (LoginPage extends BasePage) ─────────────────────────┐
│                                                           │
│  ref.watch(provider.select((s) => s.data.email))        │
│    → rebuild TextField when email changes                  │
│                                                           │
│  ref.watch(provider.select((s) => s.data.isLoginButtonEnabled))│
│    → rebuild button enabled/disabled state                  │
│                                                           │
│  ref.read(provider.notifier).setEmail(email)              │
│    → call ViewModel method (one-shot, no rebuild)         │
│                                                           │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─ VIEWMODEL (LoginViewModel extends BaseViewModel) ─────────┐
│                                                            │
│  setEmail(email)                                          │
│    → data = data.copyWith(email: email)                   │
│                                                            │
│  login()                                                  │
│    → runCatching(action: _doLogin)                         │
│    → on error: data = data.copyWith(onPageError: msg)     │
│                                                            │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ▼
┌─ MODEL (LoginState extends BaseState) ────────────────────┐
│                                                           │
│  email: String                                           │
│  password: String                                        │
│  onPageError: String                                    │
│  isLoginButtonEnabled: bool (computed)                   │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## 6. app_provider_observer.dart — Lifecycle Debug

<!-- AI_VERIFY: base_flutter/lib/ui/base/app_provider_observer.dart -->
```dart
class AppProviderObserver extends ProviderObserver {
  AppProviderObserver({
    this.logOnDidAddProvider = Config.logOnDidAddProvider,
    this.logOnDidDisposeProvider = Config.logOnDidDisposeProvider,
    this.logOnDidUpdateProvider = Config.logOnDidUpdateProvider,
    this.logOnProviderDidFail = Config.logOnProviderDidFail,
  });

  final bool logOnDidAddProvider;
  final bool logOnDidDisposeProvider;
  final bool logOnDidUpdateProvider;
  final bool logOnProviderDidFail;

  @override
  void didAddProvider(ProviderBase<Object?> provider, Object? value, ProviderContainer container) {
    if (logOnDidAddProvider) {
      Log.d('didAddProvider: $provider, value: $value');
    }
  }

  @override
  void didDisposeProvider(ProviderBase<Object?> provider, ProviderContainer container) {
    if (logOnDidDisposeProvider) {
      Log.d('didDisposeProvider: $provider');
    }
  }

  @override
  void didUpdateProvider(ProviderBase<Object?> provider, Object? previousValue, Object? newValue, ProviderContainer container) {
    if (logOnDidUpdateProvider) {
      Log.d('didUpdateProvider: $provider');
    }
  }

  @override
  void providerDidFail(ProviderBase<Object?> provider, Object error, StackTrace stackTrace, ProviderContainer container) {
    if (logOnProviderDidFail) {
      Log.e('providerDidFail: $provider, error: $error');
    }
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: `lib/ui/base/app_provider_observer.dart`](../../base_flutter/lib/ui/base/app_provider_observer.dart)

### 🔎 Quan sát

**4 lifecycle hooks:**

| Hook | Event | Ý nghĩa |
|------|-------|---------|
| `didAddProvider` | Provider initialized | Track creation order, detect unexpected init |
| `didDisposeProvider` | Provider disposed | Confirm cleanup, detect leaks |
| `didUpdateProvider` | State changed | Debug state transitions, trace bugs |
| `providerDidFail` | Provider threw error | Catch unhandled errors, crash reporting |

**Config-gated — production không log:**

```dart
// develop.json:
{ "logOnDidUpdateProvider": true }

// production.json:
{ "logOnDidUpdateProvider": false }
```

→ Zero overhead trong production. Full visibility trong debug.

---

## ⏭️ Next Steps

Concepts rút ra từ code walk → [02-concept.md](./02-concept.md)

Tóm tắt concepts sẽ cover:
1. MVVM Pattern Overview
2. BaseState Contract
3. CommonState Envelope
4. BaseViewModel Lifecycle
5. runCatching Pattern
6. BasePage Reactive Binding
7. Provider Wiring
8. AppProviderObserver
9. Page vs Shared State

Forward ref: [Module 11 — Riverpod State](../module-11-riverpod-state/) sẽ deep-dive vào provider types, ref API, và advanced patterns.

<!-- AI_VERIFY: generation-complete -->
