# Module 4: Flutter UI Basics & Navigation Flow

## Tổng quan

Module này đi vào **nền tảng của mọi Flutter app** — cách Flutter render UI, cấu trúc project, widget tree, và navigation flow cơ bản. Bạn sẽ đọc `main.dart`, `my_app.dart`, `app_navigator.dart`, và các page files — hiểu cách widget tree được xây dựng, layout primitives hoạt động, và navigation di chuyển giữa các màn hình.

**Cycle:** CODE (đọc entry files) → EXPLAIN (hiểu widget concepts) → PRACTICE (trace + modify UI).

**Prerequisite:** Hoàn thành [Module 0 — Dart Primer](../module-00-dart-primer/) (Dart syntax, async/await, OOP).

**⏱️ Thời lượng ước tính:** 60–90 phút.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được tại sao Flutter dùng widget tree thay vì imperative rendering?
2. Phân biệt được `StatelessWidget` vs `StatefulWidget` — khi nào dùng cái nào?
3. Mô tả được layout flow: `MaterialApp` → `Scaffold` → `AppBar` → `body` → children?
4. Hiểu `BuildContext` là gì và tại sao nó quan trọng trong Flutter?
5. Trace được navigation flow: `push` → route stack → `pop`?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 5 — Built-in Widgets](../module-05-built-in-widgets/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

7 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | Widget Tree & runApp | 🔴 MUST-KNOW | Foundation — mọi thứ trong Flutter là widget |
| 2 | MaterialApp & Scaffold | 🔴 MUST-KNOW | App shell — cách structure app được setup |
| 3 | Layout Widgets (Column, Row, Container) | 🔴 MUST-KNOW | Layout primitives — dùng ở mọi page |
| 4 | BuildContext — Widget Address | 🔴 MUST-KNOW | Navigation, theme, size access — dùng ở mọi widget |
| 5 | Basic Navigation (push/pop) | 🟡 SHOULD-KNOW | Route stack — navigation flow cơ bản |
| 6 | StatelessWidget vs StatefulWidget | 🟡 SHOULD-KNOW | Widget types — chọn đúng cho use case |
| 7 | Material vs Cupertino | 🟢 AI-GENERATE | Design systems — iOS/Android styling |

**Phân bố:** 🔴 ~57% · 🟡 ~29% · 🟢 ~14%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Đọc main.dart → my_app.dart → page files | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 7 concepts từ widget tree + layout | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 4 bài tập trace + modify UI | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + tiêu chí pass | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Trace Widget Tree từ main.dart | ⭐ |
| 2 | Modify Login Page Layout | ⭐ |
| 3 | Add New Widget to Scaffold | ⭐⭐ |
| 4 | AI Prompt Dojo — UI Analysis | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [main.dart](../../base_flutter/lib/main.dart) — app entry point, runApp
- [my_app.dart](../../base_flutter/lib/ui/my_app.dart) — MaterialApp setup, router config
- [app_navigator.dart](../../base_flutter/lib/navigation/app_navigator.dart) — navigation wrapper
- [splash_page.dart](../../base_flutter/lib/ui/page/splash/splash_page.dart) — minimal page skeleton
- [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart) — form UI example
- [main_page.dart](../../base_flutter/lib/ui/page/main/main_page.dart) — tab navigation
- [common_scaffold.dart](../../base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart) — shared layout wrapper

---

## Unlocks (Module 5+)

Sau khi hoàn thành Module 4, bạn sẽ:

- **Module 5 — Built-in Widgets:** Hiểu sâu hơn về widget catalog — layout, input, display, list, navigation widgets.
- **Module 6 — Custom Widgets & Animation:** Tạo widget tùy chỉnh, hiểu lifecycle, và animation basics.
- **Module 7 — Base Page & ViewModel:** BasePage, ViewModel pattern — build trên widget concepts đã học.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
