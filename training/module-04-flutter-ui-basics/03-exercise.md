# Exercises — Thực hành Flutter UI Basics & Navigation Flow

> ⚠️ Tất cả bài tập thực hiện **trên codebase `base_flutter`** — không tạo project mới.
> Prerequisite: Đã đọc xong [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md).

---

## ⭐ Exercise 1: Trace Widget Tree từ main.dart

**Mục tiêu:** Hiểu cách widget tree được xây dựng từ entry point đến page.

### Hướng dẫn

1. Mở [main.dart](../../base_flutter/lib/main.dart).
2. Trace widget tree từ `runApp` → `MyApp` → `MaterialApp.router`.
3. Mở [my_app.dart](../../base_flutter/lib/ui/my_app.dart).
4. Trace tiếp: `MaterialApp.router` → `TranslationProvider` → `DevicePreview` → `MyApp child`.
5. Mở [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart).
6. Trace body: `CommonScaffold` → `Scaffold` → `body` → `Stack` → `Column`.

### Template

Vẽ widget tree diagram:

```
runApp(
  └─ ProviderScope(
       └─ MyApp(
            └─ MaterialApp.router(
                 └─ TranslationProvider(
                      └─ DevicePreview(
                           └─ Router(
                                └─ ...
```

Điền bảng phân tích:

| Widget | Type | Purpose | Props quan trọng |
|--------|------|---------|-----------------|
| `MaterialApp.router` | MaterialApp | ? | ? |
| `Scaffold` | ? | ? | ? |
| `Stack` | ? | ? | ? |
| `Column` | ? | ? | ? |

### Câu hỏi

- Tại sao widget tree bắt đầu từ `ProviderScope` thay vì `MyApp`?
- `TranslationProvider` wrap `DevicePreview` → widget nào được build trước?
- `CommonScaffold` là StatelessWidget nhưng vẫn rebuild khi state thay đổi → tại sao?

### ✅ Checklist hoàn thành

- [ ] Vẽ widget tree diagram từ `runApp` đến page body
- [ ] Điền bảng phân tích ≥ 5 widgets
- [ ] Trả lời 3 câu hỏi
- [ ] Hiểu composition pattern trong widget tree

---

## ⭐ Exercise 2: Modify Login Page Layout

**Mục tiêu:** Thực hành layout bằng cách modify Login page spacing và alignment.

### Hướng dẫn

1. Mở [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart).
2. Tìm `Column` chứa form fields.
3. Thực hiện các thay đổi sau:

**Thay đổi 1: Giảm spacing**

```dart
// Tìm
const SizedBox(height: 100),  // Header spacing
const SizedBox(height: 50),   // Title to first field
const SizedBox(height: 24),   // Between fields

// Thay bằng
const SizedBox(height: 60),
const SizedBox(height: 32),
const SizedBox(height: 16),
```

**Thay đổi 2: Thêm padding mới**

```dart
// Tìm SingleChildScrollView padding
padding: const EdgeInsets.all(16),

// Thay bằng
padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
```

**Thay đổi 3: Center form content**

```dart
// Tìm
child: Column(
  crossAxisAlignment: CrossAxisAlignment.start,  // align left

// Thay bằng
child: Column(
  crossAxisAlignment: CrossAxisAlignment.center,  // align center
```

4. Chạy app → verify changes.

### Câu hỏi

- `crossAxisAlignment: CrossAxisAlignment.center` → center theo chiều nào trong `Column`?
- `EdgeInsets.symmetric(horizontal: 24)` → tạo bao nhiêu padding left và right?
- Nếu muốn chỉ center button nhưng text fields vẫn align left → cần thay đổi gì?

### ✅ Checklist hoàn thành

- [ ] Thay đổi spacing (3 SizedBox values)
- [ ] Thay đổi padding (EdgeInsets.symmetric)
- [ ] Thay đổi alignment (CrossAxisAlignment.center)
- [ ] Chạy app verify changes
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes** sau khi verify

---

## ⭐⭐ Exercise 3: Add New Widget to Scaffold

**Mục tiêu:** Thêm widgets vào page — AppBar, FAB, Drawer.

### Hướng dẫn

1. Tạo một page mới `lib/ui/page/example/example_page.dart`:

```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

import '../../../../index.dart';

@RoutePage()
class ExamplePage extends StatelessWidget {
  const ExamplePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Example Page'),
        backgroundColor: Colors.blue,
      ),
      body: const Center(
        child: Text('Hello from Example Page!'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('FAB tapped!')),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

2. Thêm route vào [app_router.dart](../../base_flutter/lib/navigation/routes/app_router.dart) (tạm thời):

```dart
// Thêm vào routes getter
AutoRoute(page: ExampleRoute.page),
```

3. Chạy `dart run build_runner build --delete-conflicting-outputs`.

4. Thêm navigation button vào `login_page.dart` để test:

```dart
// Trong LoginPage Column, thêm
TextButton(
  onPressed: () {
    context.router.push(const ExampleRoute());
  },
  child: Text('Go to Example'),
),
```

5. Chạy app → test navigation → verify FAB → back button.

### Câu hỏi

- `AppBar` trong `Scaffold` khác gì `PreferredSizeWidget`? `CommonAppBar` trong `common_scaffold.dart` hoạt động thế nào?
- `FloatingActionButton` position thế nào? Làm sao đổi sang bottom-right corner?
- `ScaffoldMessenger.of(context).showSnackBar(...)` khác gì `AppNavigator.showSnackBar()`?

### ✅ Checklist hoàn thành

- [ ] Tạo ExamplePage với AppBar, body, FAB
- [ ] Thêm route vào app_router.dart
- [ ] Chạy build_runner thành công
- [ ] Thêm navigation button vào LoginPage
- [ ] Test: navigate → FAB → SnackBar → back
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert all changes** sau khi verify

---

## ⭐⭐⭐ Exercise 4: AI Prompt Dojo — UI Analysis

### 🤖 AI Dojo — Code Analysis cho Flutter UI

**Mục tiêu:** Dùng AI như code analyzer — phát hiện issues trong UI code.

**Bước thực hiện:**

1. Copy nội dung [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart) vào clipboard.

2. Gửi prompt sau cho AI:

```
Bạn là Flutter UI code reviewer. Phân tích login_page.dart và tìm issues về:
- Layout performance: có unnecessary rebuilds không?
- Widget tree depth: có nested quá sâu không?
- const constructors: có widget nào nên là const nhưng không?
- Accessibility: có contentDescription cho images/icons không?
- Code organization: có logic nên extract thành widget riêng không?

Chỉ báo issues CÓ THẬT trong code, không suggest rewrite toàn bộ.

Code:
[PASTE login_page.dart]
```

3. Với mỗi issue AI tìm được:
   - Verify bằng cách đọc code thực tế
   - Issue có thật không?
   - Severity: critical / medium / low?

4. Hỏi follow-up: "Với issue nghiêm trọng nhất, hãy viết fix code cụ thể."

**✅ Tiêu chí đánh giá:**

- [ ] AI tìm được ≥ 2 issues có giá trị
- [ ] Bạn verify mỗi issue — phân biệt issue thật vs hallucination
- [ ] AI hiểu Flutter widget tree pattern
- [ ] Bạn viết 2-3 câu đánh giá: "AI tốt ở..., miss ở..., sai ở..."

---

## Exercise Summary

| # | Bài tập | Độ khó | Concept chính | Output |
|---|---------|--------|--------------|--------|
| 1 | Trace Widget Tree | ⭐ | Widget tree, composition | Diagram + bảng phân tích |
| 2 | Modify Login Layout | ⭐ | Column, SizedBox, padding | Modified + verified |
| 3 | Add Widgets to Scaffold | ⭐⭐ | AppBar, FAB, Scaffold | Working new page |
| 4 | AI Dojo — UI Analysis | ⭐⭐⭐ | Code review skills | AI evaluation |

**Tiếp theo:** [04-verify.md](./04-verify.md) — checklist tự đánh giá.

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
