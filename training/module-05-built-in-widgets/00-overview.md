# Module 5: Built-in Widgets Deep Dive

## Tổng quan

Module này đi sâu vào **Flutter widget catalog** — tất cả built-in widgets được Flutter cung cấp sẵn. Bạn sẽ học cách dùng layout widgets để compose UI, input widgets để tương tác với user, display widgets để hiển thị dữ liệu, list widgets để render collections, và navigation widgets để di chuyển giữa screens.

**Cycle:** CODE (quan sát widgets trong codebase) → EXPLAIN (hiểu từng widget type) → PRACTICE (build UI với widgets).

**Prerequisite:** Hoàn thành [Module 4 — Flutter UI Basics](../module-04-flutter-ui-basics/) (widget tree, layout primitives, build context).

**⏱️ Thời lượng ước tính:** 90–120 phút.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Phân biệt được tất cả layout widgets: Column, Row, Stack, Wrap, Expanded, Flexible, Container, SizedBox?
2. Giải thích được khi nào dùng ListView vs GridView vs CustomScrollView?
3. Mô tả được cách dùng GestureDetector vs InkWell cho touch handling?
4. Hiểu Material vs Cupertino widgets — biết khi nào dùng cái nào?
5. Trace được MediaQuery vs LayoutBuilder — khi nào responsive?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 6 — Custom Widgets & Animation](../module-06-custom-widgets-animation/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

10 concept groups, phân loại theo mức độ cần nắm:

| # | Concept Group | Badge | Widgets |
|---|---------------|-------|---------|
| 1 | Layout Widgets | 🔴 MUST-KNOW | Column, Row, Stack, Wrap |
| 2 | Flex Widgets | 🔴 MUST-KNOW | Expanded, Flexible, Spacer |
| 3 | Box Widgets | 🔴 MUST-KNOW | Container, SizedBox, ConstrainedBox |
| 4 | Display Widgets | 🔴 MUST-KNOW | Text, Image, Icon, Card |
| 5 | Input Widgets | 🟡 SHOULD-KNOW | TextField, ElevatedButton, GestureDetector |
| 6 | List Widgets | 🔴 MUST-KNOW | ListView, GridView, SliverAppBar |
| 7 | Navigation Widgets | 🟡 SHOULD-KNOW | BottomNavigationBar, TabBar, Drawer |
| 8 | Overlay Widgets | 🟡 SHOULD-KNOW | Dialog, SnackBar, BottomSheet |
| 9 | Responsive Widgets | 🟡 SHOULD-KNOW | MediaQuery, LayoutBuilder, FittedBox |
| 10 | Advanced Widgets | 🟢 AI-GENERATE | CustomPaint, ClipRRect, InteractiveViewer |

**Phân bố:** 🔴 ~50% · 🟡 ~35% · 🟢 ~15%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Widgets trong codebase | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 10 widget groups với examples | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập build UI | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Widget Catalog Explorer | ⭐ |
| 2 | Build a Dashboard Layout | ⭐⭐ |
| 3 | Implement ListView with Actions | ⭐⭐ |
| 4 | Create Modal Bottom Sheet | ⭐⭐ |
| 5 | AI Prompt Dojo — Widget Selection | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart) — Column, TextField, ElevatedButton, Consumer
- [main_page.dart](../../base_flutter/lib/ui/page/main/main_page.dart) — BottomNavigationBar, AutoTabsScaffold
- [primary_text_field.dart](../../base_flutter/lib/ui/component/primary_text_field/primary_text_field.dart) — TextField styling
- [common_scaffold.dart](../../base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart) — Scaffold composition
- [pubspec.yaml](../../base_flutter/pubspec.yaml) — assets config cho Image

---

## Unlocks (Module 6+)

Sau khi hoàn thành Module 5, bạn sẽ:

- **Module 6 — Custom Widgets:** Tạo widget tùy chỉnh dựa trên built-in widgets đã học.
- **Module 7 — Base Page & ViewModel:** Hiểu BasePage và shared components — composition của built-in widgets.
- **Module 9 — Page Structure:** Build pages với combination của tất cả widget types.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
