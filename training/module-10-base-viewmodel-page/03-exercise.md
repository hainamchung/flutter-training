# Exercise — BaseViewModel & BasePage (MVVM Pattern)

> ⭐ = beginner · ⭐⭐ = intermediate · ⭐⭐⭐ = advanced

---

## Exercise 1: Trace Login Flow End-to-End (MVVM) ⭐

**Objective:** Trace complete MVVM flow từ user tap button đến navigation.

**Instructions:**

1. Đọc lại [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart) và [login_view_model.dart](../../base_flutter/lib/ui/page/login/view_model/login_view_model.dart)

2. Trace từng bước:

```
User taps "Login" button
    │
    ▼
Step 1: onPressed callback
    └─ ref.read(provider.notifier).login()
    │
    ▼
Step 2: LoginViewModel.login()
    └─ runCatching(action: _doLogin)
    │
    ▼
Step 3: Inside runCatching
    ├─ showLoading()
    ├─ startAction('login')
    │
    ▼
Step 4: _doLogin action
    ├─ await _ref.read(sharedViewModelProvider).deviceToken
    ├─ await _ref.read(appPreferencesProvider).saveAccessToken(...)
    ├─ await _ref.read(appNavigatorProvider).replaceAll([MainRoute()])
    │
    ▼
Step 5: Success path
    ├─ hideLoading()
    ├─ stopAction('login')
    │
    ▼
Navigation to MainPage
```

3. Trả lời câu hỏi:
   - Tại sao dùng `ref.read` thay vì `ref.watch` trong `onPressed`?
   - Điều gì xảy ra nếu API call thành công nhưng `appNavigatorProvider.replaceAll` fail?

**Deliverable:** Viết trace flow bằng text/markdown với mũi tên → như trên.

---

## Exercise 2: Add Action Tracking to Login ⭐

**Objective:** Thêm button loading state dùng `doingAction`.

**Instructions:**

1. Mở [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart)

2. Tìm `ElevatedButton` cho login

3. Thêm logic disable button khi đang submit:

```dart
Consumer(
  builder: (context, ref, child) {
    // THÊM: watch doingAction('login')
    final isLoginAction = ref.watch(
      provider.select((value) => value.isDoingAction('login')),
    );
    
    return ElevatedButton(
      onPressed: isLoginAction
          ? null  // disable khi đang submit
          : () => ref.read(provider.notifier).login(),
      ...
    );
  },
)
```

4. Verify: Khi tap button → button disabled → API complete → button enabled.

**Deliverable:** Code snippet của Consumer widget mới.

---

## Exercise 3: Create a ProfileViewModel ⭐⭐

**Objective:** Tạo complete ProfileViewModel + ProfileState + ProfilePage.

**Instructions:**

1. **Tạo ProfileState:**

```dart
// lib/ui/page/profile/view_model/profile_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../../base/base_state.dart';

part 'profile_state.freezed.dart';

@freezed
sealed class ProfileState extends BaseState with _$ProfileState {
  const factory ProfileState({
    @Default('') String name,
    @Default('') String email,
    @Default('') String avatarUrl,
    @Default(false) bool isEditing,
  }) = _ProfileState;
}
```

2. **Tạo ProfileViewModel:**

```dart
// lib/ui/page/profile/view_model/profile_view_model.dart
import 'dart:async';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import '../../../base/base_view_model.dart';
import '../../../base/base_state.dart';
import '../../../index.dart';

final profileViewModelProvider =
    StateNotifierProvider.autoDispose<ProfileViewModel, CommonState<ProfileState>>(
  (ref) => ProfileViewModel(ref),
);

class ProfileViewModel extends BaseViewModel<ProfileState> {
  ProfileViewModel(this._ref) : super(const CommonState(data: ProfileState()));

  final Ref _ref;

  void setName(String name) {
    data = data.copyWith(name: name);
  }

  void setEmail(String email) {
    data = data.copyWith(email: email);
  }

  void toggleEditing() {
    data = data.copyWith(isEditing: !data.isEditing);
  }

  FutureOr<void> saveProfile() async {
    await runCatching(
      action: () async {
        await _ref.read(appPreferencesProvider).saveUserName(data.name);
        await _ref.read(appPreferencesProvider).saveUserEmail(data.email);
      },
      actionName: 'saveProfile',
      handleErrorWhen: (_) => false,
    );
  }

  FutureOr<void> loadProfile() async {
    await runCatching(
      action: () async {
        final name = await _ref.read(appPreferencesProvider).getUserName();
        final email = await _ref.read(appPreferencesProvider).getUserEmail();
        data = data.copyWith(name: name, email: email);
      },
      actionName: 'loadProfile',
    );
  }
}
```

3. **Tạo ProfilePage (abstract):**

```dart
// lib/ui/page/profile/profile_page.dart
abstract class ProfilePage extends BasePage<ProfileState,
    AutoDisposeStateNotifierProvider<ProfileViewModel, CommonState<ProfileState>>> {
  const ProfilePage({super.key});

  @override
  AutoDisposeStateNotifierProvider<ProfileViewModel, CommonState<ProfileState>> get provider =>
      profileViewModelProvider;
}
```

4. **Thêm vào barrel file** `lib/ui/page/profile/profile.dart`:
```dart
export 'profile_page.dart';
export 'view_model/profile_state.dart';
export 'view_model/profile_view_model.dart';
```

**Deliverable:**
- File structure:
  ```
  lib/ui/page/profile/
  ├── profile.dart
  ├── profile_page.dart
  └── view_model/
      ├── profile_state.dart
      └── profile_view_model.dart
  ```
- Code cho ProfileViewModel (đủ methods: setName, setEmail, toggleEditing, saveProfile, loadProfile)

---

## Exercise 4: Multiple Actions Tracking ⭐⭐

**Objective:** Hiểu cách track nhiều actions đồng thời.

**Scenario:** ProfilePage có 3 actions: `loadProfile`, `saveProfile`, `refreshProfile`. UI cần hiển thị:
- Loading overlay khi ANY action đang chạy
- Button disabled khi action tương ứng đang chạy

**Instructions:**

1. Trace code trong BasePage:
```dart
ref.listen(provider.select((value) => value.isLoading), (previous, next) {
  // Loading overlay — fired khi isLoading = true (bất kỳ action nào)
});
```

2. Đọc `CommonState` definition:
```dart
@Default(<String, bool>{}) Map<String, bool> doingAction;
```

3. Trả lời:
   - Tại sao `isLoading` trong `CommonState` là single bool thay vì Map?
   - Làm sao disable chỉ "Save" button khi đang save, nhưng "Load" button vẫn enabled?

4. Viết code cho 3 Consumer widgets:

```dart
// 1. Load button — disabled khi loadProfile đang chạy
Consumer(
  builder: (context, ref, child) {
    final isLoading = ref.watch(provider.select((s) => s.isDoingAction('loadProfile')));
    return ElevatedButton(
      onPressed: isLoading ? null : () => ref.read(provider.notifier).loadProfile(),
      child: Text(isLoading ? 'Loading...' : 'Load'),
    );
  },
)

// 2. Save button — disabled khi saveProfile đang chạy
// ... viết code ...

// 3. Refresh button — disabled khi refreshProfile đang chạy
// ... viết code ...
```

**Deliverable:** Code cho cả 3 Consumer widgets với proper action tracking.

---

## Exercise 5: AI Prompt Dojo — MVVM Architecture Review ⭐⭐⭐ (Optional)

> ⚠️ **Yêu cầu:** Bài tập này cần AI assistant (Claude/Copilot) để hoàn thành. Nếu không có access, bạn có thể bỏ qua hoặc làm manual review thay thế.

**Scenario:** Senior dev giao cho bạn review architecture của một feature mới.

**Instructions:**

Sử dụng AI assistant (Claude/Copilot) với prompt:

```
You are a Flutter Architecture Reviewer specializing in MVVM pattern.

Review the following ViewModel code for:
1. MVVM compliance — proper separation of concerns
2. Error handling — runCatching usage
3. State management — CommonState/BaseViewModel patterns
4. Performance — unnecessary rebuilds
5. Testability — can this be unit tested easily?

Code to review:
```dart
class OrderViewModel extends BaseViewModel<OrderState> {
  OrderViewModel(this._ref) : super(const CommonState(data: OrderState()));

  final Ref _ref;

  void setOrderId(String id) {
    data = data.copyWith(orderId: id);
  }

  void setCustomerName(String name) {
    data = data.copyWith(customerName: name);
  }

  Future<void> submitOrder() async {
    try {
      showLoading();
      final response = await _ref.read(apiProvider).submitOrder(data.orderId);
      await _ref.read(navigatorProvider).push(OrderConfirmationRoute());
    } catch (e) {
      hideLoading();
      rethrow;
    }
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

- [ ] Exercise 1: Hoàn thành trace flow
- [ ] Exercise 2: Button disabled khi đang submit
- [ ] Exercise 3: ProfileViewModel created
- [ ] Exercise 4: Multiple action tracking hiểu được
- [ ] Exercise 5: AI review completed (optional — requires AI assistant access)

---

## 🔑 Answer Keys

<details>
<summary>Exercise 1: Trace Flow — Answers</summary>

**Tại sao dùng `ref.read` thay vì `ref.watch` trong `onPressed`?**

- `ref.watch` chỉ dùng trong `build()` method — subscribe vào provider và trigger rebuild khi value thay đổi
- `onPressed` là event handler — chỉ cần GỌI method một lần, không cần subscribe
- `ref.watch` trong callback → crash: "ref.watch was called from outside build()"

**Điều gì xảy ra nếu `appNavigatorProvider.replaceAll` fail?**

- `replaceAll` throw exception
- `await` trong action closure → exception được catch bởi runCatching
- `hideLoading()` gọi trong catch block
- `handleErrorWhen: (_) => false` → `exception` setter được gọi
- `BasePage.ref.listen(provider.select((v) => v.appException))` → fire
- `handleException()` → show error dialog

</details>

<details>
<summary>Exercise 2: Action Tracking — Code</summary>

```dart
Consumer(
  builder: (context, ref, child) {
    final isLoginAction = ref.watch(
      provider.select((value) => value.isDoingAction('login')),
    );
    
    return ElevatedButton(
      onPressed: isLoginAction
          ? null  // disable khi đang submit
          : () {
              // Private method gọi được vì extension định nghĩa TRONG CÙNG FILE.
              // `_logLoginButtonClickEvent()` là private của extension `AnalyticsHelperOnLoginPage`
              // trên cùng file login_page.dart. Extension methods có quyền truy cập private
              // members của class mà nó extend — miễn là extension đó được khai báo trong
              // cùng library/file.
              ref.read(analyticsHelperProvider)._logLoginButtonClickEvent();
              ref.read(provider.notifier).login();
            },
      style: ButtonStyle(
        minimumSize: WidgetStateProperty.all(const Size(double.infinity, 48)),
        backgroundColor: WidgetStateProperty.all(
          Colors.black.withValues(alpha: isLoginAction ? 0.5 : 1),
        ),
      ),
      child: CommonText(
        l10n.login,
        style: style(
          fontSize: 18,
          fontWeight: FontWeight.bold,
          color: Colors.white,
        ),
      ),
    );
  },
)
```

</details>

<details>
<summary>Exercise 3: ProfileViewModel — Key Points</summary>

**ProfileViewModel pattern checklist:**
- [ ] Extends `BaseViewModel<ProfileState>`
- [ ] Provider: `StateNotifierProvider.autoDispose`
- [ ] Constructor: `ProfileViewModel(this._ref)`
- [ ] `setXxx()` methods: `data = data.copyWith(xxx: value)`
- [ ] `runCatching` với `actionName` cho tracking
- [ ] `handleErrorWhen: (_) => false` cho inline errors

</details>

<details>
<summary>Exercise 4: Multiple Actions — Answers</summary>

**Tại sao `isLoading` là single bool thay vì Map?**

- Loading overlay là GLOBAL — hiển thị khi BẤT KỲ action nào đang chạy
- `isLoading = true` khi có ít nhất 1 action đang chạy (loadingCount > 0)
- `doingAction` là granular — track từng action riêng biệt cho button states

**Làm sao disable chỉ "Save" button?**

```dart
// Save button — chỉ disabled khi saveProfile đang chạy
final isSaveAction = ref.watch(
  provider.select((s) => s.isDoingAction('saveProfile')),
);
onPressed: isSaveAction ? null : () => ref.read(provider.notifier).saveProfile()
```

Load button vẫn enabled vì `isDoingAction('loadProfile')` = false.

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
