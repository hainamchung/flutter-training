# Verification — Kiểm tra kết quả Module 4

> Đối chiếu bài làm với [02-concept.md](./02-concept.md) và thực hành trong codebase.

---

## 1. Self-Assessment Checklist

Trả lời **Yes / No** cho từng câu. Nếu **No** → quay lại concept tương ứng trong [02-concept.md](./02-concept.md).

| # | Câu hỏi | Concept | Badge |
|---|---------|---------|-------|
| 1 | Tôi giải thích được tại sao Flutter dùng widget tree thay vì imperative rendering? | Widget Tree | 🔴 |
| 2 | Tôi mô tả được Widget vs Element vs RenderObject layers? | Widget Tree | 🔴 |
| 3 | Tôi phân biệt được `MaterialApp` vs `CupertinoApp` và khi nào dùng? | MaterialApp | 🟡 |
| 4 | Tôi hiểu `Scaffold` structure: appBar, body, floatingActionButton, drawer? | Scaffold | 🔴 |
| 5 | Tôi dùng được `Column`, `Row`, `Stack`, `Container`, `SizedBox` để build layout? | Layout Widgets | 🔴 |
| 6 | Tôi giải thích được `BuildContext` là "address" của widget trong tree? | BuildContext | 🔴 |
| 7 | Tôi thực hiện được `push`, `pop`, `replace`, `replaceAll` navigation? | Navigation | 🟡 |
| 8 | Tôi phân biệt được `StatelessWidget` vs `StatefulWidget` và khi nào dùng? | Widget Types | 🟡 |

> **Target:** 4/5 Yes cho 🔴 MUST-KNOW, tối thiểu 6/8 tổng.

---

## 2. Exercise Verification

### Exercise 1 — Trace Widget Tree ⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| Widget tree diagram | Từ `runApp` đến page body | ☐ |
| Bảng phân tích | ≥ 5 widgets với type, purpose, props | ☐ |
| Câu hỏi 1 | `ProviderScope` wrap `MyApp` → Riverpod context available toàn app | ☐ |
| Câu hỏi 2 | `TranslationProvider` build trước `DevicePreview` → child được wrap | ☐ |
| Câu hỏi 3 | `CommonScaffold` là `StatelessWidget` nhưng rebuild khi parent rebuild hoặc Riverpod state change | ☐ |

### Exercise 2 — Modify Login Layout ⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| SizedBox changes | 3 giá trị thay đổi (100→60, 50→32, 24→16) | ☐ |
| EdgeInsets change | `EdgeInsets.symmetric(horizontal: 24, vertical: 16)` | ☐ |
| Alignment change | `CrossAxisAlignment.center` | ☐ |
| App runs | Không crash, layout changed visible | ☐ |
| Revert | Changes reverted sau khi verify | ☐ |

### Exercise 3 — Add Widgets to Scaffold ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| ExamplePage created | File tại `lib/ui/page/example/example_page.dart` | ☐ |
| Route added | `ExampleRoute.page` trong `app_router.dart` | ☐ |
| build_runner | Không error | ☐ |
| Navigation works | Login → Example → back hoạt động | ☐ |
| FAB works | Tap FAB → SnackBar hiển thị | ☐ |
| Revert | All changes reverted | ☐ |

### Exercise 4 — AI Prompt Dojo ⭐⭐⭐

**Verification points:**

- [ ] AI tìm được ≥ 2 issues có giá trị
- [ ] Bạn verify mỗi issue
- [ ] AI hiểu Flutter widget tree pattern
- [ ] Bạn viết đánh giá: tốt ở... / miss ở... / sai ở...

---

## 3. Concept Cross-Check

| # | Scenario | Đáp án đúng |
|---|----------|-------------|
| 1 | Widget tree start từ đâu? | `runApp(Widget)` nhận root widget |
| 2 | `MaterialApp` wrap `MyApp` → theme available ở đâu? | Tất cả descendants có access via `Theme.of(context)` |
| 3 | `Column` với `mainAxisAlignment: MainAxisAlignment.center` → children ở đâu? | Centered vertically (along main axis) |
| 4 | `Stack` children overlap hay stack vertically? | Overlap — children painted on top of each other |
| 5 | `BuildContext` là gì? | Handle đến widget's location trong tree |
| 6 | `context.router.push()` dùng context ở đâu để navigate? | Navigator gần nhất trong tree từ context đó |
| 7 | `StatelessWidget` vs `StatefulWidget` — khác nhau chính? | `StatelessWidget` không có internal state, `StatefulWidget` có |
| 8 | `setState()` làm gì? | Schedule rebuild của widget với state mới |

---

## 4. Common Mistakes

| # | Sai lầm | Hậu quả | Fix |
|---|---------|---------|-----|
| 1 | Dùng `var` thay vì `final` cho widget variables | Unnecessary mutation | Luôn dùng `final` |
| 2 | Quên `const` constructor | Larger widget tree, performance issue | Dùng `const` khi possible |
| 3 | Nested太多的 `Column`/`Row` | Deep widget tree, hard to read | Flatten layout |
| 4 | Set state trong `build()` | "setState during build" error | Wrap trong `Future.microtask()` |
| 5 | Dùng `MediaQuery.of(context)` bên ngoài build | Exception | Chỉ dùng trong `build()` hoặc check `mounted` |
| 6 | Push navigation dùng wrong navigator | Tab navigation bị break | Dùng `appNavigatorProvider` thay vì `context.router` |
| 7 | Quên `@RoutePage()` annotation | Code gen không tạo route | Luôn thêm annotation |

---

## 5. Module Completion Criteria

### Yêu cầu tối thiểu

| Tiêu chí | Yêu cầu | Trạng thái |
|----------|----------|-------------|
| Self-assessment | ≥ 6/8 Yes (bắt buộc 4/4 🔴) | ☐ |
| Exercise 1 | Trace widget tree hoàn chỉnh | ☐ |
| Exercise 2 | Modify layout + verify | ☐ |
| Exercise 3 | Add widgets to scaffold (bonus) | ☐ |
| Concept cross-check | ≥ 6/8 đúng | ☐ |

### Kết quả

| Level | Criteria | Action |
|-------|----------|--------|
| 🟢 **Completed** | 6+/8 self-assessment + Ex1 + Ex2 hoàn thành | Chuyển Module 5 |
| 🟡 **Needs Review** | 4-5/8 self-assessment | Đọc lại concepts còn yếu |
| 🔴 **Incomplete** | < 4/8 self-assessment | Đọc lại toàn bộ module |

---

## ✅ Module Complete

Hoàn thành khi:
- Self-assessment: ≥ 6/8 Yes (bắt buộc 4/4 🔴 MUST-KNOW)
- Exercises: Ex1 + Ex2 hoàn thành
- Cross-check: ≥ 6/8 đúng

**Next:** [Module 5 — Built-in Widgets Deep Dive](../module-05-built-in-widgets/) — Material widgets, layout widgets, input widgets, list widgets, navigation widgets.

<!-- AI_VERIFY: generation-complete -->
