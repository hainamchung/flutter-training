# Module 2: Architecture & Barrel Files

## Tổng quan

Module này đi vào **kiến trúc project** của Flutter app: cách tổ chức layers, barrel file pattern (`index.dart`), DI container registrations, và app widget tree. Bạn sẽ đọc `index.dart`, `di.config.dart`, `my_app.dart` — hiểu export/import conventions, layer boundaries, và code generation workflow.

**Cycle:** CODE (đọc architecture files) → EXPLAIN (hiểu patterns) → PRACTICE (trace + thêm file mới).

**Prerequisite:** Hoàn thành [Module 0 — Dart Primer](../module-00-dart-primer/) (pubspec, `make ep`, codegen) và [Module 1 — App Entrypoint](../module-01-app-entrypoint/) (DI setup, `configureInjection()`, boot sequence).

**📍 Session:** Session 2 — Architecture

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được barrel file pattern và tại sao project dùng 1 `index.dart`?
2. Biết cách import đúng convention (3 nhóm, luôn qua `index.dart`)?
3. Liệt kê được 7 layers trong project và dependency direction?
4. Phân biệt `factory` / `lazySingleton` / `factoryAsync` trong DI?

→ Nếu **4/4 Yes** — chuyển thẳng [Module 03 — Common Layer](../module-03-common-layer/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

6 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | Barrel File Pattern | 🔴 MUST-KNOW | Vi phạm = lint error + reject |
| 2 | Import Convention | 🔴 MUST-KNOW | Rule bắt buộc toàn team |
| 3 | Layer Architecture | 🟡 SHOULD-KNOW | Đặt code đúng chỗ |
| 4 | DI Registration Types | 🟡 SHOULD-KNOW | Chọn sai = memory leak / stale state |
| 5 | App Widget Tree | 🟡 SHOULD-KNOW | Hiểu root composition |
| 6 | Code Generation | 🟢 AI-GENERATE | Biết file nào generated để không sửa nhầm |

**Phân bố:** 🔴 33% · 🟡 50% · 🟢 17%

---

## 📂 Files trong Module này

> ⚠️ Module 2 có 00-overview.md. Nội dung code walk được thực hiện trực tiếp trên `base_flutter` source files (xem links bên dưới). Không có file 01-code-walk riêng.

| File | Nội dung | Vai trò |
|------|----------|---------|
| (đọc trực tiếp) | `index.dart`, `di.config.dart`, `my_app.dart` trong `base_flutter` | CODE — quan sát |
| (tài liệu bổ sung) | 6 concepts: barrel pattern, import convention, layers, DI | EXPLAIN — giải thích |
| `base_flutter/` | Thực hành: trace import chain, add file to barrel | PRACTICE — làm tay |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Trace Import Chain | ⭐ |
| 2 | Map Project Layers | ⭐ |
| 3 | Add New File to Barrel | ⭐⭐ |
| 4 | AI Prompt Dojo — Architecture Review | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [index.dart](../../base_flutter/lib/index.dart) — barrel file (~163 exports)
- [di.config.dart](../../base_flutter/lib/di.config.dart) — generated DI registrations
- [my_app.dart](../../base_flutter/lib/ui/my_app.dart) — app widget root
- [common_coding_rules.md](../../base_flutter/docs/technical/common_coding_rules.md) — import/export rules
- [naming_rules.md](../../base_flutter/docs/technical/naming_rules.md) — naming conventions

---

## Unlocks (Session 2+)

Sau khi hoàn thành Module 2, bạn sẽ:

- **Module 3 — Common Layer:** Deep dive layer `common/` — shared utilities, constants, extensions. Kiến thức layer architecture từ M2 giúp hiểu vị trí common layer trong project.
- **Module 5 — Built-in Widgets:** Widget catalog — layout, input, display, list widgets. Barrel pattern và import convention là prerequisite.
- **Module 8 — State Management:** Riverpod providers, `HookConsumerWidget`, `ref.watch()`. Kiến thức DI + widget tree từ M2 là prerequisite.

→ Bắt đầu: Đọc [index.dart](../../base_flutter/lib/index.dart) → [di.config.dart](../../base_flutter/lib/di.config.dart) → [my_app.dart](../../base_flutter/lib/ui/my_app.dart).

<!-- AI_VERIFY: generation-complete -->
