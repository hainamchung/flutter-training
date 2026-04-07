# Module 6: Custom Widgets, Lifecycle & Animation

## Tổng quan

Module này đi sâu vào **custom widget creation**, **widget lifecycle**, và **animation basics**. Bạn sẽ học cách tạo reusable widgets (StatelessWidget, StatefulWidget), hiểu lifecycle methods (initState, dispose, didUpdateWidget), sử dụng flutter_hooks để simplify state management, và implement animations (implicit, explicit, Lottie).

**Cycle:** CODE (quan sát custom widgets) → EXPLAIN (hiểu lifecycle + hooks) → PRACTICE (build custom widgets + animations).

**Prerequisite:** Hoàn thành [Module 5 — Built-in Widgets](../module-05-built-in-widgets/) (built-in widgets, layout, input).

**⏱️ Thời lượng ước tính:** 90–120 phút.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được widget lifecycle: initState → build → dispose?
2. Phân biệt được StatelessWidget vs StatefulWidget — khi nào dùng cái nào?
3. Hiểu flutter_hooks: useState, useEffect, useRef — và tại sao dùng thay vì StatefulWidget?
4. Mô tả được implicit vs explicit animation — khi nào dùng cái nào?
5. Biết cách dùng AnimationController với Tween?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 10 — Base ViewModel & Page](../module-07-base-viewmodel/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

9 concepts, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | StatelessWidget vs StatefulWidget | 🔴 MUST-KNOW | Widget types — foundation |
| 2 | Widget Lifecycle Methods | 🔴 MUST-KNOW | initState, dispose, didUpdateWidget |
| 3 | BuildContext Deep Dive | 🔴 MUST-KNOW | Widget location, data access |
| 4 | flutter_hooks (useState, useEffect) | 🔴 MUST-KNOW | Hooks pattern — simplify state |
| 5 | Keys (ValueKey, GlobalKey) | 🟡 SHOULD-KNOW | Widget identity, state preservation |
| 6 | InheritedWidget Pattern | 🟡 SHOULD-KNOW | Data passing down tree |
| 7 | Implicit Animations | 🟡 SHOULD-KNOW | AnimatedContainer, AnimatedOpacity |
| 8 | Explicit Animations | 🟡 SHOULD-KNOW | AnimationController, Tween |
| 9 | Lottie & Physics Animation | 🟢 AI-GENERATE | Advanced animation |

**Phân bố:** 🔴 ~44% · 🟡 ~44% · 🟢 ~12%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Custom widgets trong codebase | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 9 concepts từ widget lifecycle + hooks + animation | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập custom widgets + animation | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Create Custom Stateless Widget | ⭐ |
| 2 | Create Custom Stateful Widget | ⭐⭐ |
| 3 | Migrate to flutter_hooks | ⭐⭐ |
| 4 | Implement Implicit Animations | ⭐⭐ |
| 5 | AI Prompt Dojo — Widget Architecture | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [primary_text_field.dart](../../base_flutter/lib/ui/component/primary_text_field/primary_text_field.dart) — StatefulWidget example
- [base_page.dart](../../base_flutter/lib/ui/base/base_page.dart) — HookConsumerWidget usage
- [splash_page.dart](../../base_flutter/lib/ui/page/splash/splash_page.dart) — useEffect pattern
- [use_back_blocker.dart](../../base_flutter/lib/common/hook/use_back_blocker.dart) — Custom hooks example
- [use_focus_node_refocus_on_resume.dart](../../base_flutter/lib/common/hook/use_focus_node_refocus_on_resume.dart) — Custom hooks with lifecycle

---

## Unlocks (Module 7+)

Sau khi hoàn thành Module 6, bạn sẽ:

- **Module 7 — Base UI Framework:** Hiểu BasePage + ViewModel pattern — custom widgets composition.
- **Module 10 — Hooks Deep Dive:** flutter_hooks advanced patterns, custom hooks.
- **Module 17 — Architecture & DI:** Advanced architecture patterns, dependency injection deep dive.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
