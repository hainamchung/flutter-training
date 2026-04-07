# Module 15 — Popup, Dialog & Paging Patterns

**⏱️ Thời lượng ước tính:** 60–75 phút.

## 🎯 Mục tiêu

Survey **popup/dialog system** và **paging executor pattern** trong base project. Hiểu cách base project chuẩn hóa UI feedback (dialogs, snackbars) và data pagination thông qua Template Method pattern.

---

## 📌 Prerequisites — Modules Required

| Module | Concept cần nắm | Relevance cho M15 |
|--------|-----------------|-------------------|
| **M10** | `BaseViewModel`, `runCatching`, exception handling | Popup lifecycle và error handling |
| **M12** | `RestApiClient`, Dio, API pagination response | Data source cho paging |
| **M13** | `PagingExecutor`, `LoadMoreOutput`, error boundary | Paging template, state model |

> **Note:** `BasePopup`, `CommonScaffold`, `CommonText` được giới thiệu trong code walk của M15 dựa trên foundation từ M7–M10. **M7, M8, M9 không bắt buộc** nhưng giúp hiểu nhanh hơn nếu đã học:
> - M7: `BaseViewModel`, `BasePage` — page lifecycle, state management
> - M8: Riverpod providers, `ref.watch` — state management cho dialogs
> - M9: `CommonScaffold`, `CommonText`, `useEffect` — UI patterns, hooks

---

## 🔄 Re-Anchor — Ôn lại M10, M12, M13

| Module | Concept cần nhớ | Kết nối M15 |
|--------|-----------------|-------------|
| **M10 — BaseViewModel Page** | `BaseViewModel`, `runCatching`, exception handling | Popup lifecycle và error handling |
| **M12 — Data Layer** | `RestApiClient`, API pagination | Data source cho paging executor |
| **M13 — Middleware** | `PagingExecutor`, `LoadMoreOutput` | Paging template method pattern |

→ Nếu bất kỳ concept nào chưa rõ → quay lại module tương ứng trước khi tiếp tục.

---

## 📂 Anchor Files

| File | Path | Lines | Role |
|------|------|-------|------|
| BasePopup | `lib/ui/base/base_popup.dart` | ~15 | Abstract popup with popupId dedup |
| ErrorDialog | `lib/ui/popup/error_dialog/error_dialog.dart` | ~75 | Error + Retry dialog |
| ConfirmDialog | `lib/ui/popup/confirm_dialog/confirm_dialog.dart` | ~68 | Confirm/Cancel with bool return |
| CommonSnackBar | `lib/ui/popup/common_snack_bar/common_snack_bar.dart` | ~107 | Color-coded notifications |
| MaintenanceModeDialog | `lib/ui/popup/maintenance_mode_dialog/maintenance_mode_dialog.dart` | ~55 | Full-screen blocking dialog |
| PagingExecutor | `lib/data_source/api/paging/base/paging_executor.dart` | ~67 | Abstract paging template |
| LoadMoreOutput | `lib/model/base/load_more_output.dart` | ~27 | Paging state model (Freezed) |
| GetNotificationsPagingExecutor | `lib/data_source/api/paging/get_notifications_paging_executor.dart` | ~50 | Concrete paging example |

---

## 🔑 Core Concepts

| # | Concept | Summary |
|---|---------|---------|
| 1 | BasePopup & popupId dedup | Abstract class + string ID prevent duplicate popups |
| 2 | Dialog patterns | Error, Confirm, Maintenance — 3 archetypes, factory constructors |
| 3 | SnackBar pattern | Color-coded (success/info/error), ScaffoldMessenger dismiss |
| 4 | AlertDialog.adaptive | Auto Material/Cupertino per platform |
| 5 | PagingExecutor template method | Abstract `action()`, base `execute()` with snapshot/rollback |
| 6 | LoadMoreOutput state | Freezed model: page, offset, isLastPage, isLoading, exception |

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## 💡 FE Perspective

| Flutter (Base Project) | React / FE Equivalent |
|------------------------|-----------------------|
| `BasePopup` + `popupId` | Modal state flag / React portal key |
| `AlertDialog.adaptive` | Material-UI `<Dialog>` with responsive breakpoints |
| `CommonSnackBar.success/error` | `react-toastify`: `toast.success()` / `toast.error()` |
| `Navigator.pop(result)` | Promise-based modal: `const ok = await showConfirm()` |
| `PagingExecutor` + `LoadMoreOutput` | `react-query` `useInfiniteQuery` / `useSWRInfinite` |
| Snapshot/rollback on error | react-query keeping stale data on refetch error |

---

## 📖 Files trong module này

| # | File | Nội dung | Lines |
|---|------|----------|-------|
| 1 | [01-code-walk.md](./01-code-walk.md) | Structural overview: popup hierarchy + paging executor | 200–300 |
| 2 | [02-concept.md](./02-concept.md) | 6 concepts: dedup, dialog types, snackbar, adaptive, template method, state | 150–230 |
| 3 | [03-exercise.md](./03-exercise.md) | 5 exercises: ⭐ trace flow, ⭐⭐ custom dialog, ⭐⭐ paging impl, ⭐⭐⭐ AI Dojo, ⭐⭐⭐ paging page | 120–180 |
| 4 | [04-verify.md](./04-verify.md) | 12 câu verify (Popup + Paging + Design Patterns) | 70–100 |

---

## ✅ Completion Criteria

- [ ] Đọc hiểu toàn bộ popup hierarchy (BasePopup → 4 concrete classes)
- [ ] Hiểu PagingExecutor template method + snapshot/rollback
- [ ] Hoàn thành ≥ 3/4 exercises
- [ ] Verify score ≥ 9/12

> **Forward ref:** Module 16 (Lint & Quality) không liên quan trực tiếp đến dialog/paging. Các module nâng cao (MA Performance & Security, MB Native Features) sẽ cover thêm optimization và native integration patterns.

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích `BasePopup.popupId` deduplication — tại sao `ErrorDialog` dùng dynamic ID còn `MaintenanceModeDialog` dùng static?
2. Phân biệt khi nào dùng `AlertDialog` vs `SnackBar` vs full-screen dialog?
3. Mô tả `PagingExecutor.execute()` flow — snapshot/rollback hoạt động thế nào khi page 3 fail?
4. Implement được custom dialog mới extend `BasePopup` với factory pattern?
5. Trace data flow: scroll to bottom → `PagingExecutor.execute(isInitialLoad: false)` → API → `LoadMoreOutput` → UI update?

→ Nếu **5/5 Yes** — chuyển thẳng **[Module 16 — Lint & Quality](../module-16-lint-quality/)**.
→ Nếu có bất kỳ **No** — hoàn thành module này.

## Unlocks (Module 16+)

Sau khi hoàn thành Module 15, bạn sẽ:

- **Module 16 — Lint & Quality:** Lint rules, code quality automation, và analysis tooling.
- **Module 17 — Architecture & DI:** Dependency Injection với get_it và injectable — đặt nền móng cho toàn bộ architecture.

<!-- AI_VERIFY: generation-complete -->
