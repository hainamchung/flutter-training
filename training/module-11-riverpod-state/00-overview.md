# Module 11: Riverpod & State Management

## Tổng quan

Module này deep-dive vào **Riverpod** — hệ thống state management mạnh mẽ trong Flutter. Bạn sẽ hiểu Provider taxonomy (Provider, StateProvider, StateNotifierProvider, FutureProvider, StreamProvider), ref API rules (read vs watch vs listen vs select), autoDispose và family modifiers, ProviderContainer và ProviderObserver, DI bridge patterns (getIt → Riverpod), và decision tree để chọn đúng provider type cho mọi use case.

**Cycle:** CODE (đọc provider examples) → EXPLAIN (hiểu taxonomy + rules) → PRACTICE (trace + classify + build + optimize).

**Prerequisite:** Hoàn thành [Module 7 — Base ViewModel](../module-07-base-viewmodel/) (StateNotifierProvider, autoDispose), [Module 10 — BaseViewModel Page](../module-10-base-viewmodel-page/) (BaseViewModel + BasePage patterns), và [Module 12 — Data Layer](../module-12-data-layer/) (API, repositories — exercises có thể involve real API calls).

> ⚠️ **Riverpod Version Note:** Codebase dùng Riverpod **v1 API** (`StateNotifier` + `StateNotifierProvider`). Module này giải thích v1 patterns — concepts áp dụng cho cả v2/v3 với API changes nhỏ.

---

## 🔄 Re-Anchor — Ôn lại M7, M10

| Module | Concept cần nhớ | Kết nối M11 |
|--------|-----------------|------------|
| **M7 — Base ViewModel** | `BaseViewModel extends StateNotifier`, `CommonState` envelope, `runCatching` | M11 phân tích tại sao dùng `StateNotifierProvider.autoDispose` |
| **M10 — BaseViewModel Page** | Provider wiring, BasePage ref patterns | M11 deep-dive ref API rules và provider types |

→ Nếu bất kỳ concept nào chưa rõ → quay lại module tương ứng trước khi tiếp tục.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Vẽ được decision tree chọn provider type trong 30 giây cho bất kỳ use case?
2. Giải thích được 4 cách dùng `ref`: read (one-shot), watch (rebuild), listen (side-effect), select (granular)?
3. Phân biệt được `autoDispose` vs family modifiers — khi nào dùng?
4. Viết được DI bridge pattern: `Provider((ref) => getIt.get<T>())`?
5. Giải thích được `ProviderObserver` hooks: didAdd, didUpdate, didDispose, didFail?
6. Dùng được `ProviderContainer` để manually manage providers trong tests?

→ Nếu **6/6 Yes** — chuyển thẳng [Module 12 — Data Layer](../module-12-data-layer/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

10 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | ProviderScope & Container | 🔴 MUST-KNOW | Root setup, overrides for testing |
| 2 | Provider Types Taxonomy | 🔴 MUST-KNOW | Chọn đúng type — decision tree |
| 3 | ref API (read/watch/listen/select) | 🔴 MUST-KNOW | Rules nghiêm ngặt, dùng sai = bug |
| 4 | autoDispose & family | 🟡 SHOULD-KNOW | Page-scoped lifecycle, parameterized providers |
| 5 | DI Bridge Pattern | 🟡 SHOULD-KNOW | getIt → Provider wrapper, testability |
| 6 | ProviderContainer | 🟡 SHOULD-KNOW | Manual container management |
| 7 | ProviderObserver | 🟡 SHOULD-KNOW | Lifecycle logging, debugging |
| 8 | FutureProvider & StreamProvider | 🟡 SHOULD-KNOW | Async data patterns |
| 9 | Testing Overrides | 🟡 SHOULD-KNOW | overrideWith for unit tests |
| 10 | State Management Decision Tree | 🔴 MUST-KNOW | Khi nào dùng gì |

**Phân bố:** 🔴 ~40% · 🟡 ~60% · 🟢 0%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Provider examples: Provider, StateProvider, StateNotifierProvider, FutureProvider, StreamProvider | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 10 concepts: taxonomy, ref API, lifecycle, DI bridge, testing | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập: trace lifecycle, classify types, build providers, AI dojo | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + exercise answers + cross-check | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Provider Lifecycle Trace | ⭐ |
| 2 | Classify Provider Types | ⭐ |
| 3 | Build SettingsViewModel | ⭐⭐ |
| 4 | Selector Optimization | ⭐⭐ |
| 5 | AI Prompt Dojo — Riverpod Architecture Review | ⭐⭐⭐ |

---

## 🔓 Unlocks

Hoàn thành Module 11 mở khóa:

| Module | Sử dụng gì từ M11 |
|--------|---------------------|
| **M12 — Data Layer** | Dio integration, API client, decoders |
| **M18 — Testing** | Override providers trong `ProviderScope`, mock DI services |

---

## 🔗 Liên kết

- [main.dart](../../base_flutter/lib/main.dart) — ProviderScope root setup
- [shared_providers.dart](../../base_flutter/lib/ui/shared/shared_providers.dart) — `currentUserProvider` (StateProvider)
- [shared_view_model.dart](../../base_flutter/lib/ui/shared/shared_view_model.dart) — `sharedViewModelProvider` (Provider)
- [login_view_model.dart](../../base_flutter/lib/ui/page/login/view_model/login_view_model.dart) — `StateNotifierProvider.autoDispose`
- [base_page.dart](../../base_flutter/lib/ui/base/base_page.dart) — ref.listen + ref.watch consumption
- [app_provider_observer.dart](../../base_flutter/lib/ui/base/app_provider_observer.dart) — lifecycle debug observer

<!-- AI_VERIFY: generation-complete -->
