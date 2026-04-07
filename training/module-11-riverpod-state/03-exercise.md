# Exercise — Riverpod & State Management

> ⭐ = beginner · ⭐⭐ = intermediate · ⭐⭐⭐ = advanced

---

## Exercise 1: Provider Lifecycle Trace ⭐

**Objective:** Trace complete provider lifecycle từ creation đến disposal.

**Instructions:**

1. Đọc code trong [app_provider_observer.dart](../../base_flutter/lib/ui/base/app_provider_observer.dart)

2. Trace lifecycle cho `loginViewModelProvider`:

```
Timeline:
t0: App starts → ProviderScope created → observers attached
t1: User navigates to LoginPage
t2: LoginPage.build() → ref.watch(loginViewModelProvider) ← first access
    ↓
    didAddProvider(loginViewModelProvider) fires
    ↓
    LoginViewModel() created
    ↓
t3: User types email → ref.watch(provider.select(s => s.data.email))
    ↓
    didUpdateProvider(loginViewModelProvider) fires (if Config enabled)
t4: User taps back → LoginPage unmounts
    ↓
    didDisposeProvider(loginViewModelProvider) fires
    ↓
    LoginViewModel disposed, state wiped
```

3. Trả lời câu hỏi:
   - Tại sao `didAddProvider` fires khi `ref.watch` được gọi, không phải khi provider được declared?
   - Điều gì xảy ra nếu user navigate away rồi immediately navigate back trước khi provider disposed?

**Deliverable:** Timeline diagram như trên.

---

## Exercise 2: Classify Provider Types ⭐

**Objective:** Phân loại providers theo type.

**Instructions:**

Đọc các scenarios và xác định provider type:

| # | Scenario | Provider Type |
|---|----------|---------------|
| 1 | App-wide navigation service | ? |
| 2 | Selected theme mode (light/dark) | ? |
| 3 | Form state với validation | ? |
| 4 | API configuration từ server | ? |
| 5 | Chat message stream | ? |
| 6 | User's shopping cart | ? |
| 7 | Computed full name: first + last | ? |
| 8 | Page A's local counter | ? |
| 9 | Page B's local counter (independent) | ? |
| 10 | Firebase Analytics service | ? |

**Đáp án:**

| # | Provider Type |
|---|---------------|
| 1 | `Provider` |
| 2 | `StateProvider<ThemeMode>` |
| 3 | `StateNotifierProvider.autoDispose` |
| 4 | `FutureProvider<AppConfig>` |
| 5 | `StreamProvider<Message>` |
| 6 | `StateNotifierProvider` (no autoDispose) |
| 7 | `Provider` (computed) |
| 8 | `StateNotifierProvider.autoDispose` |
| 9 | `StateNotifierProvider.autoDispose` (independent instance) |
| 10 | `Provider` |

---

## Exercise 3: Build SettingsViewModel ⭐⭐

**Objective:** Tạo complete SettingsViewModel với multiple settings.

**Scenario:** SettingsPage cho phép user thay đổi:
- Theme mode (light/dark/system)
- Language (en/vi)
- Notification enabled (bool)
- Auto-play videos (bool)

**Instructions:**

1. **Tạo SettingsState:**

```dart
@freezed
sealed class SettingsState extends BaseState with _$SettingsState {
  const factory SettingsState({
    @Default(ThemeMode.system) ThemeMode themeMode,
    @Default('en') String language,
    @Default(true) bool notificationsEnabled,
    @Default(true) bool autoPlayVideos,
  }) = _SettingsState;
}
```

2. **Tạo SettingsViewModel:**

```dart
final settingsViewModelProvider =
    StateNotifierProvider.autoDispose<SettingsViewModel, CommonState<SettingsState>>(
  (ref) => SettingsViewModel(ref),
);

class SettingsViewModel extends BaseViewModel<SettingsState> {
  SettingsViewModel(this._ref) : super(const CommonState(data: SettingsState()));

  final Ref _ref;

  void setThemeMode(ThemeMode mode) {
    data = data.copyWith(themeMode: mode);
    // TODO: Save to preferences
  }

  void setLanguage(String lang) {
    data = data.copyWith(language: lang);
    // TODO: Save to preferences
  }

  void toggleNotifications() {
    data = data.copyWith(notificationsEnabled: !data.notificationsEnabled);
  }

  void toggleAutoPlay() {
    data = data.copyWith(autoPlayVideos: !data.autoPlayVideos);
  }

  FutureOr<void> loadSettings() async {
    await runCatching(
      action: () async {
        final prefs = _ref.read(appPreferencesProvider);
        final theme = await prefs.getThemeMode();
        final lang = await prefs.getLanguage();
        final notifs = await prefs.getNotificationsEnabled();
        final autoPlay = await prefs.getAutoPlayVideos();
        
        data = data.copyWith(
          themeMode: theme ?? ThemeMode.system,
          language: lang ?? 'en',
          notificationsEnabled: notifs ?? true,
          autoPlayVideos: autoPlay ?? true,
        );
      },
      actionName: 'loadSettings',
    );
  }

  FutureOr<void> saveSettings() async {
    await runCatching(
      action: () async {
        await Future.wait([
          _ref.read(appPreferencesProvider).saveThemeMode(data.themeMode),
          _ref.read(appPreferencesProvider).saveLanguage(data.language),
          _ref.read(appPreferencesProvider).saveNotificationsEnabled(data.notificationsEnabled),
          _ref.read(appPreferencesProvider).saveAutoPlayVideos(data.autoPlayVideos),
        ]);
      },
      actionName: 'saveSettings',
    );
  }
}
```

3. **Viết SettingsPage Consumer:**

```dart
@RoutePage()
class SettingsPage extends BasePage<SettingsState,
    AutoDisposeStateNotifierProvider<SettingsViewModel, CommonState<SettingsState>>> {
  const SettingsPage({super.key});

  @override
  AutoDisposeStateNotifierProvider<SettingsViewModel, CommonState<SettingsState>> get provider =>
      settingsViewModelProvider;

  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    // Load settings on mount
    useEffect(() {
      Future.microtask(() => ref.read(provider.notifier).loadSettings());
      return null;
    }, []);

    final themeMode = ref.watch(provider.select((s) => s.data.themeMode));
    final language = ref.watch(provider.select((s) => s.data.language));
    final notificationsEnabled = ref.watch(provider.select((s) => s.data.notificationsEnabled));
    final autoPlayVideos = ref.watch(provider.select((s) => s.data.autoPlayVideos));

    return CommonScaffold(
      body: ListView(
        children: [
          // Theme selector
          ListTile(
            title: const Text('Theme'),
            trailing: DropdownButton<ThemeMode>(
              value: themeMode,
              onChanged: (mode) => ref.read(provider.notifier).setThemeMode(mode!),
              items: ThemeMode.values.map((m) => DropdownMenuItem(value: m, child: Text(m.name))).toList(),
            ),
          ),
          // Language selector
          ListTile(
            title: const Text('Language'),
            trailing: DropdownButton<String>(
              value: language,
              onChanged: (lang) => ref.read(provider.notifier).setLanguage(lang!),
              items: ['en', 'vi'].map((l) => DropdownMenuItem(value: l, child: Text(l))).toList(),
            ),
          ),
          // Toggle switches
          SwitchListTile(
            title: const Text('Notifications'),
            value: notificationsEnabled,
            onChanged: (_) => ref.read(provider.notifier).toggleNotifications(),
          ),
          SwitchListTile(
            title: const Text('Auto-play Videos'),
            value: autoPlayVideos,
            onChanged: (_) => ref.read(provider.notifier).toggleAutoPlay(),
          ),
          // Save button
          ElevatedButton(
            onPressed: () => ref.read(provider.notifier).saveSettings(),
            child: const Text('Save Settings'),
          ),
        ],
      ),
    );
  }
}
```

**Deliverable:** Complete SettingsState + SettingsViewModel + SettingsPage structure.

---

## Exercise 4: Selector Optimization ⭐⭐

**Objective:** Optimize rebuilds bằng selector pattern.

**Scenario:** ProfilePage có nhiều fields. Không dùng selector → unnecessary rebuilds.

**Instructions:**

1. **Without selector (❌Inefficient):**

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final state = ref.watch(profileViewModelProvider);  // ❌ Rebuild khi BẤT KỲ field đổi

  return Column(
    children: [
      Text('Name: ${state.data.name}'),           // Rebuild khi email đổi
      Text('Email: ${state.data.email}'),         // Rebuild khi name đổi
      Text('Avatar: ${state.data.avatarUrl}'),   // Rebuild khi name đổi
      Switch(
        value: state.data.isEditing,               // Rebuild khi email đổi
        onChanged: (_) => ref.read(provider.notifier).toggleEditing(),
      ),
    ],
  );
}
```

2. **With selector (✅ Efficient):**

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ Mỗi selector chỉ rebuild khi field tương ứng đổi
  final name = ref.watch(profileViewModelProvider.select((s) => s.data.name));
  final email = ref.watch(profileViewModelProvider.select((s) => s.data.email));
  final avatarUrl = ref.watch(profileViewModelProvider.select((s) => s.data.avatarUrl));
  final isEditing = ref.watch(profileViewModelProvider.select((s) => s.data.isEditing));

  return Column(
    children: [
      Text('Name: $name'),           // Rebuild CHỈ khi name đổi
      Text('Email: $email'),         // Rebuild CHỈ khi email đổi
      Text('Avatar: $avatarUrl'),    // Rebuild CHỈ khi avatarUrl đổi
      Switch(
        value: isEditing,            // Rebuild CHỈ khi isEditing đổi
        onChanged: (_) => ref.read(provider.notifier).toggleEditing(),
      ),
    ],
  );
}
```

3. **Task: Tính rebuild counts**

Scenario: User updates email 3 times, sau đó toggle isEditing.

| Pattern | Rebuild count cho Text(name) | Rebuild count cho Switch |
|---------|---------------------------|------------------------|
| Without selector | ? | ? |
| With selector | ? | ? |

**Đáp án:**
| Pattern | Text(name) rebuilds | Switch rebuilds |
|---------|---------------------|-----------------|
| Without selector | 4 (initial + 3 email) | 4 (initial + 3 email + toggle) |
| With selector | 1 (initial only!) | 2 (initial + toggle) |

**Deliverable:** Code với selectors và rebuild count analysis.

---

## Exercise 5: AI Prompt Dojo — Riverpod Architecture Review ⭐⭐⭐

**Scenario:** Team lead yêu cầu review architecture của một feature mới.

**Instructions:**

Sử dụng AI assistant với prompt:

```
You are a Flutter Architecture Reviewer specializing in Riverpod state management.

Review the following code for:
1. Provider type selection — correct choice?
2. ref API usage — read vs watch vs listen vs select?
3. Memory management — autoDispose where needed?
4. Performance — unnecessary rebuilds?
5. Testability — override-able for testing?

Code to review:
```dart
// ❌ BAD CODE — DO NOT USE IN PRODUCTION

// User profile feature

// 1. Wrong provider type
final userNameProvider = StateNotifierProvider<UserNameNotifier, String>((ref) {
  return UserNameNotifier();
});

class UserNameNotifier extends StateNotifier<String> {
  UserNameNotifier() : super('');

  void setName(String name) {
    state = name;
  }
}

// 2. Wrong ref usage in build
class ProfilePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ❌ Reading in build instead of using ref.watch
    final name = ref.read(userNameProvider);
    
    return Column(
      children: [
        Text('Name: $name'),
        TextField(
          onChanged: (value) {
            // ❌ Watch in callback — will crash
            ref.watch(userNameProvider.notifier).setName(value);
          },
        ),
      ],
    );
  }
}
```

Provide:
1. List of issues (minimum 5)
2. Suggested fixes for each issue
3. Refactored code example
4. Unit test skeleton
```

**Deliverable:**
1. Copy prompt trên
2. Paste AI response
3. Mô tả 3 điểm bạn học được từ review

---

## ✅ Checklist — Trước khi chuyển module

- [ ] Exercise 1: Lifecycle trace completed
- [ ] Exercise 2: All 10 providers classified
- [ ] Exercise 3: SettingsViewModel created
- [ ] Exercise 4: Selector optimization hiểu được
- [ ] Exercise 5: AI review completed

---

## 🔑 Answer Keys

<details>
<summary>Exercise 1: Lifecycle — Answers</summary>

**Tại sao didAddProvider fires khi ref.watch được gọi?**

- Provider **lazy-created** — chỉ được instantiate khi first access
- Declaration không trigger creation
- `ref.watch` → first access → container creates provider → `didAddProvider` fires

**Điều gì xảy ra nếu navigate away rồi immediately navigate back?**

- First pop → no listeners → autoDispose triggers → dispose pending
- Second push → if disposal not yet completed → provider still exists
- Result: may get cached instance instead of fresh one
- This is why `autoDispose` waits for actual disposal before cleanup

</details>

<details>
<summary>Exercise 4: Selector — Rebuild Count</summary>

**Scenario: 3 email updates + 1 toggle**

| Pattern | Text(name) rebuilds | Switch rebuilds |
|---------|---------------------|-----------------|
| Without selector | 4 | 4 |
| With selector | 1 | 2 |

**Why?**
- Without selector: entire widget rebuilds on ANY state change
- With selector: only widget listening to specific field rebuilds

</details>

---

**Tiếp theo:** [04-verify.md](./04-verify.md) — kiểm tra kiến thức và hoàn thành module.

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
