# Module 1: App Entrypoint & Bootstrap

## Tổng quan

Module này đi vào **điểm khởi đầu** của Flutter app: từ `main()` qua các bước init cho đến khi widget tree hiển thị trên screen. Bạn sẽ đọc `main.dart`, `app_initializer.dart`, `di.dart` — hiểu boot sequence, error boundary, DI setup, và Firebase initialization.

**Cycle:** CODE (đọc entry files) → EXPLAIN (hiểu boot concepts) → PRACTICE (trace + modify init).

**Prerequisite:** Hoàn thành [Module 0 — Dart Primer](../module-00-dart-primer/) (pubspec, make sync, codegen pipeline).

**⏱️ Thời lượng ước tính:** 45–60 phút.

**📍 Session:** Session 1 — Foundation

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Hiểu tại sao `WidgetsFlutterBinding.ensureInitialized()` phải gọi trước async operations?
2. Giải thích được `runZonedGuarded` và Zone error handling?
3. Biết cách `get_it` + `injectable` phối hợp (annotation → codegen → runtime)?
4. Hiểu vai trò `ProviderScope` trong Riverpod?

→ Nếu **4/4 Yes** — chuyển thẳng [Module 02 — Architecture](../module-02-architecture-barrel/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

7 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | Flutter App Entry Point | 🔴 MUST-KNOW | Sai = không hiểu app lifecycle |
| 2 | WidgetsFlutterBinding | 🔴 MUST-KNOW | Thiếu = crash trước runApp |
| 3 | runZonedGuarded | 🟡 SHOULD-KNOW | Global error handling |
| 4 | Init Sequence | 🟡 SHOULD-KNOW | Thứ tự init quan trọng |
| 5 | DI (get_it + injectable) | 🟡 SHOULD-KNOW | Nền tảng dependency management |
| 6 | ProviderScope | 🟢 AI-GENERATE | Riverpod root — deep dive ở M8 |
| 7 | Firebase Integration | 🟢 AI-GENERATE | Firebase.initializeApp() + Crashlytics — deep dive ở MB |

**Phân bố:** 🔴 ~29% · 🟡 ~43% · 🟢 ~28%

---

## 📂 Files trong Module này

> ⚠️ Module 1 có 00-overview.md. Nội dung code walk được thực hiện trực tiếp trên `base_flutter` source files (xem links bên dưới). Không có file 01-code-walk riêng.

| File | Nội dung | Vai trò |
|------|----------|---------|
| (đọc trực tiếp) | `main.dart`, `app_initializer.dart`, `di.dart` trong `base_flutter` | CODE — quan sát |
| (tài liệu bổ sung) | 7 concepts: boot sequence, DI, binding | EXPLAIN — giải thích |
| `base_flutter/` | Thực hành: trace boot flow, trace DI registrations | PRACTICE — làm tay |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Trace the Boot Sequence | ⭐ |
| 2 | Add an Initialization Step | ⭐⭐ |
| 3 | Trace DI Registration | ⭐⭐ |
| 4 | Custom Boot Logger (AI Prompt Dojo) | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [main.dart](../../base_flutter/lib/main.dart) — app entry point
- [app_initializer.dart](../../base_flutter/lib/app_initializer.dart) — system initialization
- [di.dart](../../base_flutter/lib/di.dart) — DI container setup
- [log_instructions.md](../../base_flutter/docs/technical/log_instructions.md) — logging conventions

---

## Unlocks (Session 1+)

Sau khi hoàn thành Module 1, bạn sẽ:

- **Module 2 — Architecture & Barrel:** Hiểu project structure (layers, folder convention, barrel file pattern). Session 2. Kiến thức DI và init flow từ M1 là prerequisite.
- **Module 3 — Common Layer:** Deep dive `common/` — shared utilities, constants, extensions. Session 2. Kiến thức layer architecture từ M2 giúp hiểu vị trí common layer trong project.
- **Module 7 — Base UI Framework:** MyApp widget, theme, routing setup — build trên `ProviderScope` đã học ở M1.
- **Module 8 — State Management:** Deep dive Riverpod — `ProviderScope`, Provider types, observers.

→ Bắt đầu: Đọc [main.dart](../../base_flutter/lib/main.dart) → [app_initializer.dart](../../base_flutter/lib/app_initializer.dart) → [di.dart](../../base_flutter/lib/di.dart).

<!-- AI_VERIFY: generation-complete -->
