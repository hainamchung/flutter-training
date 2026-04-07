# Concepts — Popup, Dialog & Paging Patterns

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. BasePopup & popupId Deduplication Pattern 🔴 MUST-KNOW

**WHY:** `BasePopup` là abstract base class chuẩn hoá mọi popup trong app. `popupId` ngăn hiển thị trùng lặp — đặc biệt quan trọng khi user double-click hoặc async callback trigger popup nhiều lần.

<!-- AI_VERIFY: base_flutter/lib/ui/base/base_popup.dart -->
```dart
abstract class BasePopup extends StatelessWidget {
  const BasePopup({required this.popupId, super.key});

  final String popupId;
  Widget buildPopup(BuildContext context);

  @override
  Widget build(BuildContext context) {
    return buildPopup(context);
  }
}
```
<!-- END_VERIFY -->

**EXPLAIN:**

**Template Method pattern:**

```
BasePopup.build()
    │
    └── calls buildPopup() ← overridden by each subclass
```

- `build()` — template method, không override
- `buildPopup()` — abstract, subclass implement UI

**popupId deduplication logic:**

```dart
// BasePopup popupId convention:
// ErrorDialog.error("msg")     → "ErrorDialog.error_msg"
// ConfirmDialog.logOut()       → "ConfirmDialog_Logout"
// CommonSnackBar.success("ok") → "CommonSnackBar.success_ok"
// MaintenanceModeDialog      → "MaintenanceModeDialog" (static)
```

| popupId type | When to use | Example |
|---|---|---|
| **Dynamic** (includes message) | Generic popup reused with different content | `ErrorDialog.error("timeout")` vs `ErrorDialog.error("network")` |
| **Static** | Singleton popup — only one instance globally | `MaintenanceModeDialog` |

**Duplicate prevention:** > **popupId là một dedup KEY** được cung cấp bởi mỗi popup — việc deduplication thực tế phụ thuộc vào cách popups được quản lý bởi caller. `BasePopup` chỉ lưu trữ popupId như một string field, không tự động deduplicate.

> 💡 **FE Perspective**
> **Flutter:** `popupId` = deduplication key cho popup. Ngăn trùng khi double-click.
> **React/Vue tương đương:** React: modal state flag hoặc portal key. Vue: `v-if` + `ref` check.
> **Khác biệt quan trọng:** Flutter dùng string ID — có thể inject bất kỳ đâu. React thường dùng boolean state hoặc modal manager library.

---

## 2. Dialog Patterns — ErrorDialog, ConfirmDialog, MaintenanceModeDialog 🔴 MUST-KNOW

**WHY:** 3 archetypes dialog cover 90% use cases trong app. Factory pattern đảm bảo chỉ tạo được valid combinations.

### ErrorDialog — Error + Retry

```dart
// 2 factory constructors
ErrorDialog.error(message: "Network error")
ErrorDialog.errorWithRetry(message: "Network error", onRetryPressed: () {})

// popupId = "ErrorDialog.error_$message" — dynamic
```

**Retry flow:**
```
User taps Retry
    → Navigator.pop()        ← dismiss dialog
    → onRetryPressed?.call() ← trigger retry
```

### ConfirmDialog — Confirm/Cancel với Bool return

```dart
// Domain-specific factories
ConfirmDialog.deleteAccount(doOnConfirm: () {})
ConfirmDialog.logOut(doOnConfirm: () {})

// popupId = "ConfirmDialog_$message" — dynamic

// Trả về Future<bool?>
final confirmed = await showDialog<bool>(
  context: ctx,
  builder: (_) => ConfirmDialog.logOut(doOnConfirm: () {}),
);
// confirmed = true  → user tap Confirm/OK
// confirmed = false → user tap Cancel/Back
```

**Return value pattern:**

| Dialog | Return | When |
|--------|--------|-------|
| `ErrorDialog.error()` | `void` | Chỉ dismiss |
| `ErrorDialog.errorWithRetry()` | `void` | Dismiss → retry callback |
| `ConfirmDialog.*()` | `bool` | `Navigator.pop(true/false)` |

> 💡 **FE Perspective**
> **Flutter:** `showDialog<T>()` trả `Future<T?>`. `ConfirmDialog` trả `bool`.
> **React/Vue tương đương:** `window.confirm()` hoặc Promise-based modal pattern.
> **Khác biệt quan trọng:** Flutter dialog là widget trên Navigator stack — `Navigator.pop(bool)` để return value. React dùng callback hoặc Promise wrapper.

---

## 3. AlertDialog.adaptive — Platform-Aware Dialog 🟡 SHOULD-KNOW

**WHY:** `AlertDialog.adaptive` tự động render Material dialog trên Android và Cupertino dialog trên iOS — không cần viết platform-specific code.

```dart
// adaptive = auto Material/Cupertino per platform
return AlertDialog.adaptive(
  title: Text('Title'),
  content: Text('Message'),
  actions: [...],
);
```

**Khác biệt với platform-specific dialogs:**

| Widget | Platform | Use case |
|--------|----------|----------|
| `AlertDialog.adaptive` | Auto (Material/Cupertino) | Most dialogs |
| `CupertinoAlertDialog` | iOS only | iOS-specific styling |
| `MaterialBanner` | Material only | Top banner notification |
| `MaintenanceModeDialog` (CommonScaffold) | Neither | Full-screen takeover |

> 💡 **FE Perspective**
> **Flutter:** `AlertDialog.adaptive` = Material/Cupertino tuỳ platform.
> **React/Vue tương đương:** CSS framework có responsive components — e.g., Material-UI Dialog.

---

## 4. CommonSnackBar — Color-Coded Notifications 🟡 SHOULD-KNOW

**WHY:** SnackBar là feedback ngắn gọn không blocking. Factory pattern đảm bảo consistent styling và semantic colors.

```dart
// 3 factories — cùng shape, khác màu
CommonSnackBar.success("Operation completed")
CommonSnackBar.info("New notification")
CommonSnackBar.error("Failed to load data")
```

| Factory | Color | Semantic |
|---------|-------|----------|
| `.success()` | `color.green1` | Action completed |
| `.info()` | `color.grey1` | Information |
| `.error()` | `color.red1` | Error/alert |

**Dismiss mechanism:**

```dart
// KHÔNG dùng Navigator.pop() — SnackBar không nằm trên route stack
ScaffoldMessenger.maybeOf(context)?.hideCurrentSnackBar();
```

> 💡 **FE Perspective**
> **Flutter:** `ScaffoldMessenger.showSnackBar()` với 3 factory methods.
> **React/Vue tương đương:** `react-toastify`: `toast.success()`, `toast.error()`.
> **Khác biệt quan trọng:** Flutter SnackBar gắn với `Scaffold` context. React toast thường global, không cần scaffold.

---

## 5. PagingExecutor — Template Method for Pagination 🔴 MUST-KNOW

**WHY:** Pagination logic phức tạp (snapshot, rollback, error handling) — Template Method pattern tách biệt paging orchestration khỏi API-specific logic.

<!-- AI_VERIFY: base_flutter/lib/data_source/api/paging/base/paging_executor.dart -->
```dart
abstract class PagingExecutor<T, P extends PagingParams> {
  PagingExecutor({...});

  LoadMoreOutput<T> _output;   // current state
  LoadMoreOutput<T> _oldOutput; // snapshot for rollback

  // Abstract — subclass implements API call
  Future<LoadMoreOutput<T>> action({
    required int page,
    required int limit,
    required P? params,
  });

  // Template method — orchestrates paging logic
  Future<LoadMoreOutput<T>> execute({
    required bool isInitialLoad,
    P? params,
  }) async {
    try {
      if (isInitialLoad) {
        _output = LoadMoreOutput<T>(data: <T>[], page: initPage, offset: initOffset);
      }
      final loadMoreOutput = await action(page: page, limit: limit, params: params);
      // ... merge data, calculate page/offset
      _output = newOutput;
      _oldOutput = newOutput;
      return newOutput;
    } catch (e) {
      _output = _oldOutput; // ← rollback
      throw e is AppException ? e : AppUncaughtException(rootException: e);
    }
  }
}
```
<!-- END_VERIFY -->

**EXPLAIN:**

**Template Method — subclass chỉ implement `action()`:**

```
PagingExecutor.execute()
    │
    ├─ (isInitialLoad) → reset state
    │
    ├─ call action() ← subclass implements: API call → LoadMoreOutput
    │
    ├─ success:
    │     ├─ merge data
    │     ├─ _output = _oldOutput = newOutput
    │     └─ return newOutput
    │
    └─ error:
          ├─ _output = _oldOutput  ← ROLLBACK
          └─ throw AppException
```

**Snapshot/Rollback flow:**

```
User xem page 2 (50 items) → tap load page 3
    → execute(isInitialLoad: false)
    → action() = API call page 3
    → SUCCESS: _output = newOutput (page 3 data)
    → User xem page 3 (50 items)

User đang xem page 3 → tap load page 4
    → execute(isInitialLoad: false)
    → action() = API call page 4
    → FAIL (timeout):
        → _output = _oldOutput  ← rollback về page 3
        → throw AppException
    → User vẫn xem page 3 (không mất data)
```

**`isInitialLoad` distinction:**

| Value | Khi nào | Behavior |
|-------|---------|----------|
| `true` | Pull-to-refresh, filter change | Reset `_output` về initial state trước khi fetch |
| `false` | Load more (scroll to bottom) | Giữ `_output` hiện tại, append/merge page mới |

> 💡 **FE Perspective**
> **Flutter:** `PagingExecutor` với snapshot/rollback — error giữ nguyên pages cũ.
> **React/Vue tương đương:** `react-query` `useInfiniteQuery` — cùng behavior: error không mất cached pages.
> **Khác biệt quan trọng:** Flutter dùng class inheritance (Template Method). React dùng declarative query management (react-query), không phải callback injection.

---

## 6. LoadMoreOutput — Paging State Model 🟡 SHOULD-KNOW

**WHY:** `LoadMoreOutput` là Freezed model wrap toàn bộ paging state — data, pagination info, loading, error trong một object.

<!-- AI_VERIFY: base_flutter/lib/model/base/load_more_output.dart -->
```dart
@freezed
sealed class LoadMoreOutput<T> with _$LoadMoreOutput<T> {
  const LoadMoreOutput._();

  const factory LoadMoreOutput({
    required List<T> data,
    Object? otherData,
    @Default(Constant.initialPage) int page,
    @Default(false) bool isRefreshSuccess,
    @Default(0) int offset,
    @Default(false) bool isLastPage,
    @Default(0) int total,
    @Default(false) bool isLoading,
    AppException? exception,
  }) = _LoadMoreOutput;

  int get nextPage => page + 1;
  bool get hasError => exception != null;
}
```
<!-- END_VERIFY -->

**Field responsibilities:**

| Field | Role | Updated by |
|-------|------|-----------|
| `data` | Items for current page | `action()` return |
| `page` | Current page number | `execute()` auto-increment |
| `offset` | Item offset (for offset-based APIs) | `execute()` auto-calculate |
| `isLastPage` | No more data to load | `action()` return |
| `isLoading` | Currently fetching | `execute()` |
| `exception` | Error state | `execute()` on catch |
| `isRefreshSuccess` | Was this a refresh? | `execute()` on initial load |
| `total` | Total items count | `action()` return |

**UI indicators từ state:**

```
isLoading = true         → hiện loading spinner
hasError = true         → hiện retry button
isLastPage = true       → ẩn loading spinner / "No more items"
hasError = false && isLastPage = false → hiện loading spinner (next page)
```

---

## 🔗 Bridges Summary

| Flutter (Base Project) | Frontend | Notes |
|---|---|---|
| `BasePopup` + `popupId` | Modal state flag | Deduplication |
| `AlertDialog.adaptive` | Material-UI `<Dialog>` | Platform-aware |
| `CommonSnackBar.success/error` | `react-toastify` | Notification |
| `Navigator.pop(bool)` | `Promise<boolean>` | Dialog result |
| `PagingExecutor` + `LoadMoreOutput` | `useInfiniteQuery` | Pagination state |
| Snapshot/rollback on error | Stale data on refetch error | Same behavior |

---

## 🎯 Micro-Task: Practice ngay

**Exercise 1: Dialog Decision Tree**

Khi nào dùng dialog nào?

```
User action → thông báo đơn giản?
    ├── SnackBar (không blocking)
    │     ├─ Success → CommonSnackBar.success()
    │     ├─ Info   → CommonSnackBar.info()
    │     └─ Error  → CommonSnackBar.error()

User action → cần phản hồi trước khi tiếp tục?
    ├── OK only          → ErrorDialog.error()
    ├── OK + Retry       → ErrorDialog.errorWithRetry()
    └── Confirm/Cancel   → ConfirmDialog.*()

User action → full-screen, không dismiss được?
    └── MaintenanceModeDialog
```

**Exercise 2: Paging Flow**

1. User pull-to-refresh → `execute(isInitialLoad: true)` → state reset + fetch page 1
2. User scroll to bottom → `execute(isInitialLoad: false)` → giữ data + fetch page 2
3. Page 3 fail → `_output = _oldOutput` → user vẫn xem page 2

---

→ Tiếp theo: [03-exercise.md](./03-exercise.md)

<!-- AI_VERIFY: generation-complete -->
