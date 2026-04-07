# Module 10: BaseViewModel & BasePage (MVVM Pattern)

## Tổng quan

Module này đi sâu vào **MVVM Pattern** — cách tổ chức code theo Model-View-ViewModel separation trong Flutter. Bạn sẽ hiểu `BaseViewModel` class structure (StateNotifier subclass, mounted guard, loading counter), `BasePage` class structure (HookConsumerWidget subclass, ViewModel connection), `CommonState` envelope pattern (loading/data/error states), `runCatching` cho typed error handling, ViewModel lifecycle (init → dispose → refresh), và Provider wiring patterns — hiểu cách mọi thành phần kết nối tạo thành reactive MVVM pipeline.

**Cycle:** CODE (đọc base files + login example) → EXPLAIN (hiểu patterns) → PRACTICE (trace + build + analyze).

**Prerequisite:** Hoàn thành [Module 2 — Architecture](../module-02-architecture-barrel/) (DI + getIt), [Module 3 — Common Layer](../module-03-common-layer/) (Result type, Log), [Module 4 — Flutter UI Basics](../module-04-flutter-ui-basics/) (Widget tree, BuildContext, MaterialApp), [Module 6 — Custom Widgets & Animation](../module-06-custom-widgets-animation/) (AppColors.of(context)), và [Module 7 — Base UI Framework](../module-07-base-viewmodel/) (BaseState, CommonState, BaseViewModel, BasePage).

> ⚠️ Nếu bạn đến từ M07 — đây là phiên bản **tổng hợp & nâng cao** với MVVM context đầy đủ. M07 đã cover chi tiết từng file; module này bridge sang **tại sao dùng MVVM** và **khi nào dùng pattern**.

---

## 🔄 Re-Anchor — Ôn lại M2-M7

| Module | Concept cần nhớ | Kết nối M10 |
|--------|-----------------|------------|
| **M2 — Architecture** | DI với `get_it`, Riverpod Provider expose DI | M10 ViewModel dùng `Ref` để read providers (DI access) |
| **M3 — Common Layer** | `Log` utility, `Config` flags | M10 `mounted` guard dùng `Log.e()`, `AppProviderObserver` gated by `Config` |
| **M4 — Flutter UI Basics** | Widget tree, `BuildContext`, MaterialApp | `BasePage.build()` dùng `BuildContext` đầu tiên, `AppColors.of(context)` init |
| **M7-M10** | Navigation patterns: `AppNavigator`, `replaceAll`, `pop` | M10 LoginViewModel navigate sau login success |
| **M6 — Resource** | `AppColors.of(context)` init trong `build()` | M10 `BasePage.build()` gọi `AppColors.of(context)` đầu tiên |
| **M7 — Base UI** | `BaseState`, `CommonState`, `BaseViewModel`, `BasePage` | M10 MVVM context — tại sao tách biệt ViewModel khỏi View |

→ Nếu bất kỳ concept nào chưa rõ → quay lại module tương ứng trước khi tiếp tục.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được **MVVM separation** — Model (data), View (UI), ViewModel (logic) — và tại sao tách biệt?
2. Trace được `CommonState<LoginState>` envelope flow: data + isLoading + appException + doingAction?
3. Giải thích `mounted` guard trong BaseViewModel — setter/getter check, `Log.e()` fallback?
4. Giải thích `runCatching` flow: action → catch → `handleErrorWhen` → retry chain → `exception` setter?
5. Trace được `BasePage.ref.listen` cho `appException` → `ExceptionHandler` và `isLoading` → Overlay?
6. Viết được `StateNotifierProvider.autoDispose` declaration cho ViewModel mới?
7. Phân biệt được **Page-state** (autoDispose) vs **Shared-state** (persistent)?

→ Nếu **7/7 Yes** — chuyển thẳng [Module 11 — Riverpod State](../module-11-riverpod-state/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

9 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | MVVM Pattern Overview | 🔴 MUST-KNOW | Tại sao tách biệt Model-View-ViewModel |
| 2 | BaseState Contract | 🟢 AI-GENERATE | Abstract marker, generic constraint |
| 3 | CommonState Envelope | 🔴 MUST-KNOW | Generic wrapper: data + infra fields |
| 4 | BaseViewModel Lifecycle | 🔴 MUST-KNOW | Mounted guard, loading count, state accessors |
| 5 | runCatching Pattern | 🔴 MUST-KNOW | Centralized error handling, retry chain |
| 6 | BasePage Reactive Binding | 🔴 MUST-KNOW | ref.listen, loading overlay, exception dispatch |
| 7 | Provider Wiring | 🟡 SHOULD-KNOW | StateNotifierProvider.autoDispose pattern |
| 8 | AppProviderObserver | 🟢 AI-GENERATE | Config-gated lifecycle logging |
| 9 | Page vs Shared State | 🟡 SHOULD-KNOW | Separation, lifecycle, communication |

**Phân bố:** 🔴 ~56% · 🟡 ~22% · 🟢 ~22%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Đọc base_state → common_state → base_view_model → base_page → login example → observer | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 9 concepts: MVVM, state contract, envelope, lifecycle, runCatching, page binding, provider, observer, page vs shared | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập trace + action tracking + build ViewModel + multi-action + AI dojo | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + exercise answers + cross-check | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Trace Login Flow End-to-End (MVVM) | ⭐ |
| 2 | Add Action Tracking to Login | ⭐ |
| 3 | Create a ProfileViewModel | ⭐⭐ |
| 4 | Multiple Actions Tracking | ⭐⭐ |
| 5 | AI Prompt Dojo — MVVM Architecture Review | ⭐⭐⭐ |

---

## 🔓 Unlocks

Hoàn thành Module 10 mở khóa:

| Module | Sử dụng gì từ M10 |
|--------|-------------------|
| **M11 — Riverpod State** | Deep-dive Riverpod providers, `ProviderScope`, `autoDispose` lifecycle |
| **M13 — Middleware** | HookConsumerWidget base, hooks integration với ViewModel |
| **M15 — Capstone** | Concrete pages extend `BasePage`, ViewModel patterns trong practice |
| **M18 — Testing** | Unit test ViewModel methods, mock `runCatching` behavior |

---

## 🔗 Liên kết

- [base_state.dart](../../base_flutter/lib/ui/base/base_state.dart) — abstract marker class (3 lines)
- [common_state.dart](../../base_flutter/lib/ui/base/common_state.dart) — generic state envelope (20 lines)
- [base_view_model.dart](../../base_flutter/lib/ui/base/base_view_model.dart) — StateNotifier + runCatching (177 lines)
- [base_page.dart](../../base_flutter/lib/ui/base/base_page.dart) — reactive UI binding (92 lines)
- [login_state.dart](../../base_flutter/lib/ui/page/login/view_model/login_state.dart) — concrete state example (19 lines)
- [login_view_model.dart](../../base_flutter/lib/ui/page/login/view_model/login_view_model.dart) — concrete ViewModel example (58 lines)
- [login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart) — concrete Page example (157 lines)
- [app_provider_observer.dart](../../base_flutter/lib/ui/base/app_provider_observer.dart) — lifecycle debug observer (63 lines)
- [main.dart](../../base_flutter/lib/main.dart) — ProviderScope root setup

<!-- AI_VERIFY: generation-complete -->
