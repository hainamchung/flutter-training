# Verify — BaseViewModel & BasePage (MVVM Pattern)

> Self-assessment checklist và exercise verification.

---

## 📋 Self-Assessment Checklist

Trả lời từng câu hỏi. Nếu câu nào không trả lời được → quay lại [02-concept.md](./02-concept.md).

### MVVM Pattern

- [ ] **1.** Giải thích được 3 layers trong MVVM: Model (data), View (UI widget), ViewModel (logic + state).
- [ ] **2.** Nêu ít nhất 3 benefits của MVVM: testability, separation of concerns, reusability, team scalability.
- [ ] **3.** Giải thích được tại sao logic KHÔNG nên đặt trong Widget (View).

### BaseState & CommonState

- [ ] **4.** Giải thích được `BaseState` là abstract marker class — không có implementation, chỉ là constraint.
- [ ] **5.** Liệt kê được 5 fields trong `CommonState<T>`: `data`, `appException`, `isLoading`, `isFirstLoading`, `doingAction`.
- [ ] **6.** Giải thích được tại sao wrap (CommonState) thay vì extend (LoginState extends LoadingState).

### BaseViewModel Lifecycle

- [ ] **7.** Trace được mounted guard flow: `mounted = true` → async call → unmount → `mounted = false` → set state → Log.e.
- [ ] **8.** Giải thích được tại sao dùng loading counter (`_loadingCount`) thay vì bool.
- [ ] **9.** Nêu được điểm khác biệt giữa `state = x` và `data = x` trong BaseViewModel.

### runCatching

- [ ] **10.** Viết được `runCatching` basic usage với `action` và `actionName`.
- [ ] **11.** Phân biệt được 3 strategies: global dialog, inline error, mixed.
- [ ] **12.** Giải thích được retry chain flow: `maxRetries: 2` → fail → recursive call với `maxRetries: 1`.

### BasePage

- [ ] **13.** Giải thích được tại sao `BasePage` extends `HookConsumerWidget` (cần cả hooks + Riverpod ref).
- [ ] **14.** Phân biệt được `ref.listen` (side-effect, callback) vs `ref.watch` (subscribe, rebuild).
- [ ] **15.** Trace được loading overlay flow: `isLoading: true` → `_showLoadingOverlay()` → `OverlayEntry` inserted.

### Provider & State

- [ ] **16.** Viết được provider declaration cho ViewModel mới: `StateNotifierProvider.autoDispose<MyVM, CommonState<MyState>>`.
- [ ] **17.** Giải thích được autoDispose lifecycle: create → alive → no listeners → dispose.
- [ ] **18.** Phân biệt được Page state (autoDispose, fresh on re-enter) vs Shared state (app-wide, persistent).

### Cross-Check

- [ ] **19.** Trace được complete Login flow: button tap → ViewModel.login() → runCatching → API → Navigator → MainPage.
- [ ] **20.** Giải thích được tại sao `loginViewModelProvider` dùng `.autoDispose` (page-scoped) thay vì không modifier (shared).

---

## ✅ Exercise Verification

### Exercise 1: Trace Login Flow

**Yêu cầu:** Hoàn thành trace flow với 5+ steps.

**Checklist:**
- [ ] User tap → `onPressed` callback
- [ ] `ref.read(provider.notifier).login()`
- [ ] `runCatching` → `showLoading()` + `startAction('login')`
- [ ] Async actions: `deviceToken`, `saveAccessToken`, `replaceAll`
- [ ] Success: `hideLoading()` + `stopAction('login')` + navigation

**Đáp án mẫu:** Xem [03-exercise.md](./03-exercise.md#exercise-1-trace-login-flow-end-to-end-mvvm-)

### Exercise 2: Action Tracking

**Yêu cầu:** Consumer widget disable button khi đang submit.

**Checklist:**
- [ ] `ref.watch(provider.select((value) => value.isDoingAction('login')))`
- [ ] `onPressed: isLoginAction ? null : callback`
- [ ] Button style change khi disabled (opacity, color)

**Đáp án mẫu:** Xem [03-exercise.md](./03-exercise.md#exercise-2-add-action-tracking-to-login-)

### Exercise 3: ProfileViewModel

**Yêu cầu:** Tạo ProfileState + ProfileViewModel + ProfilePage structure.

**Checklist:**
- [ ] `ProfileState extends BaseState` với `@freezed`
- [ ] `ProfileViewModel extends BaseViewModel<ProfileState>`
- [ ] Provider: `StateNotifierProvider.autoDispose`
- [ ] Methods: `setName`, `setEmail`, `toggleEditing`, `saveProfile`, `loadProfile`
- [ ] `runCatching` với `actionName` cho mỗi async action

### Exercise 4: Multiple Actions

**Yêu cầu:** Hiểu separate action tracking vs global loading.

**Checklist:**
- [ ] Giải thích được: `isLoading` = global (any action), `doingAction['x']` = granular (specific action)
- [ ] Viết được 3 Consumer widgets với separate action tracking

### Exercise 5: AI Review

**Yêu cầu:** Complete AI Prompt Dojo với review output.

**Checklist:**
- [ ] Prompt được execute với AI assistant
- [ ] Có ít nhất 5 issues được identified
- [ ] Có refactored code example
- [ ] Có unit test skeleton

---

## 🎯 Completion Criteria

Để hoàn thành module này:

| Criteria | Điều kiện |
|----------|-----------|
| **Core Understanding** | Trả lời được 16/20 câu trong checklist |
| **Practical Skill** | Hoàn thành Exercise 1-4 |
| **Advanced Exploration** | Hoàn thành Exercise 5 |
| **Code Reading** | Đọc và hiểu tất cả files trong [🔗 Liên kết](../module-07-base-viewmodel/00-overview.md#-liên-kết) |

---

## 🔄 Next Module Prep

Sau khi hoàn thành Module 10:

### Đọc trước

1. [Module 11 — Riverpod State](../module-11-riverpod-state/):
   - Provider types taxonomy (Provider, StateProvider, StateNotifierProvider, FutureProvider)
   - ref API: read vs watch vs listen
   - autoDispose và family modifiers

2. [Module 12 — Data Layer](../module-12-data-layer/):
   - HookConsumerWidget deep-dive
   - Built-in hooks: useState, useEffect, useCallback
   - Custom hooks composition

### Chuẩn bị questions cho mentor

1. Khi nào dùng `StateProvider` vs `StateNotifierProvider`?
2. Tại sao `ref.watch` không được dùng trong callbacks?
3. Làm sao để test ViewModel mà không cần widget?

---

## 📖 Glossary Terms

| Term | Definition |
|------|------------|
| **MVVM** | Model-View-ViewModel — architectural pattern tách biệt UI, logic, và data |
| **StateNotifier** | Riverpod class quản lý state, tự notify listeners khi state thay đổi |
| **CommonState** | Generic wrapper chứa business data + infrastructure fields (loading, error) |
| **autoDispose** | Riverpod modifier tự dispose provider khi không còn listeners |
| **mounted guard** | Pattern kiểm tra widget lifecycle trước khi set state |
| **runCatching** | Centralized error handling wrapper với retry support |

---

## 🏆 Module Achievement

Sau khi hoàn thành checklist + exercises:

```
┌─────────────────────────────────────────────────────┐
│           MODULE 10 COMPLETE ✅                     │
│                                                     │
│  Concepts Mastered:                                 │
│  ├── MVVM Pattern                                │
│  ├── BaseState & CommonState                     │
│  ├── BaseViewModel Lifecycle                     │
│  ├── runCatching Pattern                         │
│  ├── BasePage Reactive Binding                   │
│  ├── Provider Wiring                             │
│  └── Page vs Shared State                       │
│                                                     │
│  Next: Module 11 — Riverpod State Management     │
└─────────────────────────────────────────────────────┘
```

---

**Tiếp theo:** [Module 11 — Riverpod State Management](../module-11-riverpod-state/)

<!-- AI_VERIFY: generation-complete -->
