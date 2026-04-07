# Module 16: Lint & Code Quality

## Tổng quan

Module này đi sâu vào **code quality tools** — flutter_lints configuration, custom_lint rules, super_lint package, và code quality guidelines. Bạn sẽ đọc `analysis_options.yaml`, custom lint rules, và understand cách maintain consistent code quality across codebase.

**Cycle:** CODE (đọc lint config) → EXPLAIN (hiểu rules) → PRACTICE (config + create lints).

**Prerequisite:** Hoàn thành [Module 1 — App Entrypoint](../module-01-app-entrypoint/) (project structure), [Module 2 — Architecture](../module-02-architecture-barrel/) (code organization).

**⏱️ Thời lượng ước tính:** 60–90 phút.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Config được `analysis_options.yaml` với enable/disable specific lint rules?
2. Giải thích được sự khác nhau giữa `error`, `warning`, và `info` severity levels?
3. Tạo và enable custom lint rule trong `super_lint` package?
4. Dùng được `flutter analyze` và hiểu output?
5. Restart được Analysis Server khi cần?
6. Hiểu được tại sao lint rules improve maintainability?

→ Nếu **6/6 Yes** — chuyển thẳng [Module 17 — Architecture & DI](../module-17-architecture-di/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

10 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | analysis_options.yaml Structure | 🔴 MUST-KNOW | Config file, rule categories |
| 2 | flutter_lints Configuration | 🔴 MUST-KNOW | Enable/disable rules |
| 3 | Key Lint Rules | 🔴 MUST-KNOW | prefer_const, avoid_print, etc. |
| 4 | custom_lint Setup | 🟡 SHOULD-KNOW | Custom rules in project |
| 5 | super_lint Package | 🟡 SHOULD-KNOW | Create custom lints |
| 6 | Linter Rule Categories | 🟡 SHOULD-KNOW | Style, best practice, pub |
| 7 | Analysis Server Management | 🟡 SHOULD-KNOW | Restart when needed |
| 8 | IDE Integration | 🟡 SHOULD-KNOW | VS Code/IntelliJ warnings |
| 9 | Creating Custom Lints | 🟢 AI-GENERATE | Write lint rules |
| 10 | Effective Dart Guidelines | 🟢 AI-GENERATE | Style guide compliance |

**Phân bố:** 🔴 ~30% · 🟡 ~50% · 🟢 ~20%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Đọc analysis_options.yaml → lints config | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 10 concepts từ lint rules | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập config + create lints | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + cross-check | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Analyze existing lint config | ⭐ |
| 2 | Enable/disable specific rules | ⭐ |
| 3 | Create custom lint rule | ⭐⭐ |
| 4 | Set up super_lint package | ⭐⭐ |
| 5 | AI Prompt Dojo — Code Quality Review | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [analysis_options.yaml](../../base_flutter/analysis_options.yaml) — lint configuration (233 lines)
- [pubspec.yaml](../../base_flutter/pubspec.yaml) — dev dependencies (lints, custom_lint)
- [super_lint/lib/](../../base_flutter/super_lint/lib/) — custom lint rules
- [Makefile](../../base_flutter/makefile) — `make analyze` command

---

## Unlocks (Module 17+)

Sau khi hoàn thành Module 16, bạn sẽ:

- **Module 17 — Architecture & DI:** Tổ chức codebase với DI patterns, shared components.
- **Module 18 — Testing:** Maintain code quality với comprehensive tests.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
