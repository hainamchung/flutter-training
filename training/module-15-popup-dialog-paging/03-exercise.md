# Exercises — Popup, Dialog & Paging Patterns

## PRACTICE — Làm tay trên codebase thật

---

## Bài 1: Trace Popup Flow End-to-End ⭐

**Mục tiêu:** Hiểu popup lifecycle từ ViewModel đến màn hình.

**Hướng dẫn:**

1. Tìm file `login_view_model.dart` trong `base_flutter/lib/`
2. Trace cách `ErrorDialog` được trigger khi login fail
3. Trace cách `ConfirmDialog.logOut()` được gọi
4. Điền vào bảng dưới:

| Popup Type | Được gọi ở method nào | `popupId` | Return value |
|---|---|---|---|
| `ErrorDialog.error()` | | | |
| `ErrorDialog.errorWithRetry()` | | | |
| `ConfirmDialog.logOut()` | | | |
| `CommonSnackBar.success()` | | | N/A |

**Deliverable:** Bảng đã điền đầy đủ (ảnh chụp màn hình optional nhưng khuyến khích).

**Checklist:**
- [ ] Tìm được file login_view_model.dart
- [ ] Trace được ErrorDialog flow
- [ ] Trace được ConfirmDialog flow
- [ ] Hiểu popupId được construct như thế nào

---

## Bài 2: Implement Custom Error Dialog ⭐⭐

**Mục tiêu:** Tạo custom dialog extend `BasePopup`.

**Scenario:** Team muốn hiển thị "Maintenance" dialog khi server trả 503.

**Hướng dẫn:**

1. Tạo file `lib/ui/popup/maintenance_dialog/maintenance_dialog.dart`:

```dart
// ignore_for_file: unused_element_parameter

import 'package:flutter/material.dart';

import '../../../index.dart';

class MaintenanceDialog extends BasePopup {
  const MaintenanceDialog._({
    super.key,
    required super.popupId,
    required this.title,
    required this.message,
    this.onDismissed,
  });

  // Factory constructor
  factory MaintenanceDialog.of({
    Key? key,
    required String title,
    required String message,
    VoidCallback? onDismissed,
  }) {
    return MaintenanceDialog._(
      key: key,
      popupId: 'MaintenanceDialog', // ← static, chỉ 1 instance
      title: title,
      message: message,
      onDismissed: onDismissed,
    );
  }

  final String title;
  final String message;
  final VoidCallback? onDismissed;

  @override
  Widget buildPopup(BuildContext context) {
    return AlertDialog.adaptive(
      title: CommonText(title, style: null),
      content: CommonText(message, style: null),
      actions: [
        TextButton(
          onPressed: () {
            Navigator.of(context).pop();
            onDismissed?.call();
          },
          child: const CommonText('OK', style: null),
        ),
      ],
    );
  }
}
```

2. Trigger dialog từ interceptor khi nhận HTTP 503:

```dart
// Trong RetryOnErrorInterceptor hoặc RefreshTokenInterceptor
if (response.statusCode == 503) {
  final result = await showDialog< void>(
    context: context,
    barrierDismissible: false, // ← KHÔNG dismiss được
    builder: (_) => MaintenanceDialog.of(
      title: 'Maintenance',
      message: 'Server is under maintenance. Please try again later.',
      onDismissed: () {
        // Navigate to home or retry
      },
    ),
  );
  return; // ← interceptor dừng, không retry
}
```

3. Điền vào bảng:

| Decision | Giá trị |
|---|---|
| popupId | |
| `barrierDismissible` | |
| Override `buildPopup()` hay `build()`? | |
| Return value type | |

**Checklist:**
- [ ] Tạo MaintenanceDialog extend BasePopup
- [ ] Dùng factory constructor
- [ ] Dùng `AlertDialog.adaptive`
- [ ] Trigger từ interceptor hoặc ViewModel
- [ ] Chạy `make ep` để export vào barrel

---

## Bài 3: Implement Paging in a Page ⭐⭐

**Mục tiêu:** Thêm paging vào một page có sẵn.

**Scenario:** `home_page.dart` cần load danh sách thông báo với paging.

**Hướng dẫn:**

1. Tạo `GetNotificationsPagingParams`:

```dart
class GetNotificationsPagingParams extends PagingParams {
  GetNotificationsPagingParams({this.isRead = false});
  final bool isRead;
}
```

2. Tạo ViewModel với paging state:

```dart
class HomeViewModel extends BaseViewModel {
  HomeViewModel(super.ref);

  LoadMoreOutput<NotificationData> get notifications => data.notifications;
  bool get isLoading => notifications.isLoading;
  bool get hasError => notifications.hasError;
  bool get isLastPage => notifications.isLastPage;
  List<NotificationData> get notificationList => notifications.data;

  Future<void> loadInitialNotifications() async {
    await runCatching(
      action: () => pagingExecutor.execute(
        isInitialLoad: true,
        params: GetNotificationsPagingParams(),
      ),
    );
  }

  Future<void> loadMore() async {
    if (isLastPage || isLoading) return;
    await runCatching(
      action: () => pagingExecutor.execute(
        isInitialLoad: false,
        params: GetNotificationsPagingParams(),
      ),
    );
  }
}
```

3. Trong UI — hook vào scroll:

```dart
// Trong page sử dụng ScrollController
@override
Widget build(BuildContext context) {
  return NotificationListener<ScrollNotification>(
    onNotification: (notification) {
      if (notification is ScrollEndNotification) {
        final metrics = notification.metrics;
        if (metrics.pixels >= metrics.maxScrollExtent - 200) {
          viewModel.loadMore(); // ← load more khi gần cuối
        }
      }
      return false;
    },
    child: ListView.builder(
      itemCount: viewModel.notificationList.length + (viewModel.isLastPage ? 0 : 1),
      itemBuilder: (context, index) {
        if (index >= viewModel.notificationList.length) {
          return viewModel.isLoading
              ? const Center(child: CircularProgressIndicator())
              : const SizedBox.shrink();
        }
        return NotificationTile(data: viewModel.notificationList[index]);
      },
    ),
  );
}
```

**Checklist:**
- [ ] Tạo PagingParams subclass
- [ ] Implement paging logic trong ViewModel
- [ ] Trigger loadInitialNotifications() ở init
- [ ] Hook scroll controller cho loadMore()
- [ ] Hiển thị loading/error/end-of-list states

---

## Bài 4: Implement RefreshIndicator + Paging ⭐⭐

**Mục tiêu:** Kết hợp pull-to-refresh với paging.

**Hướng dẫn:**

Thêm `RefreshIndicator` vào ListView:

```dart
@override
Widget build(BuildContext context) {
  return RefreshIndicator(
    onRefresh: () async {
      await viewModel.loadInitialNotifications();
    },
    child: NotificationListener<ScrollNotification>(
      onNotification: (notification) {
        if (notification is ScrollEndNotification) {
          final metrics = notification.metrics;
          if (metrics.pixels >= metrics.maxScrollExtent - 200) {
            viewModel.loadMore();
          }
        }
        return false;
      },
      child: ListView.builder(...),
    ),
  );
}
```

**Khi nào dùng `execute(isInitialLoad: true)` vs `false`?**

| Action | isInitialLoad | State effect |
|--------|-------------|-------------|
| Pull-to-refresh | `true` | Reset list, fetch page 1 |
| Filter/Sort change | `true` | Reset list, fetch page 1 |
| Scroll to bottom | `false` | Append next page |
| App start | `true` | Load initial data |

**Checklist:**
- [ ] Wrap ListView với RefreshIndicator
- [ ] Gọi `loadInitialNotifications()` khi pull-to-refresh
- [ ] Vẫn giữ `loadMore()` cho scroll-to-bottom
- [ ] Error state vẫn giữ data cũ (snapshot)

---

## Bài 5: AI Prompt Dojo — Paging Page Design ⭐⭐⭐

**Mục tiêu:** Viết prompt để AI generate toàn bộ paging page.

### ❌ Bad Prompt

```
Tạo paging page Flutter
```

### ✅ Good Prompt

```
Tạo NotificationListPage trong Flutter theo base_flutter conventions:

Context:
- Dùng MVVM pattern: HomeViewModel extends BaseViewModel
- PagingExecutor cho pagination (base_flutter pattern)
- CommonScaffold cho layout
- CommonText, CommonImage cho UI components
- Riverpod providers

Requirements:
1. State:
   - LoadMoreOutput<NotificationData> notifications
   - isLoading, hasError, isLastPage, notificationList
   - Pull-to-refresh (RefreshIndicator)
   - Load more on scroll (NotificationListener)
   - Error state với retry button

2. UI:
   - CommonScaffold với AppBar title "Notifications"
   - Pull-to-refresh ListView
   - Each item: avatar + title + time + read indicator
   - Loading spinner ở cuối list khi loading more
   - Empty state: icon + "No notifications" message
   - Error state: error icon + message + retry button

3. Files to create:
   - lib/modules/home/presentation/home_view_model.dart
   - lib/modules/home/presentation/pages/notification_list_page.dart
   - lib/modules/home/presentation/widgets/notification_tile.dart

4. Output: 3 Dart files, production-ready code
```

### 🎯 Challenge

1. Chạy prompt trên với Claude/Copilot
2. Review output — có đúng base_flutter conventions?
3. Fix để match với existing codebase patterns
4. Trace paging flow: refresh → loadInitial → scroll → loadMore → error → rollback

**Deliverable:**
- Prompt đã dùng
- Output từ AI
- Các thay đổi để match conventions
- Trace flow diagram

---

## 📤 Submit

1. Push code lên branch `feature/m15-popup-paging`
2. Tạo PR với description:

```
## M15: Popup, Dialog & Paging Patterns

- [ ] Bài 1: Popup flow traced
- [ ] Bài 2: MaintenanceDialog implemented
- [ ] Bài 3: Paging in HomeViewModel
- [ ] Bài 4: Refresh + Paging combined
- [ ] Bài 5: AI Prompt Dojo — paging page design
```

---

## ↩️ Revert Changes

Sau khi hoàn thành mỗi bài tập (1-4), revert changes để không ảnh hưởng codebase:

```bash
# Revert tất cả thay đổi trong bài tập
git checkout -- lib/

# Hoặc revert cụ thể file
git checkout -- lib/ui/popup/maintenance_dialog/maintenance_dialog.dart

# Nếu đã add vào barrel (chạy make ep):
# 1. Revert barrel changes
git checkout -- lib/index.dart

# 2. Chạy lại make ep để clean
make ep
```

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
