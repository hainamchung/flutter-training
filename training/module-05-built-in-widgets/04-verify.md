# Verification — Kiểm tra kết quả Module 5

> Đối chiếu bài làm với [02-concept.md](./02-concept.md) và thực hành trong codebase.

---

## 1. Self-Assessment Checklist

Trả lời **Yes / No** cho từng câu. Nếu **No** → quay lại concept tương ứng trong [02-concept.md](./02-concept.md).

| # | Câu hỏi | Concept | Badge |
|---|---------|---------|-------|
| 1 | Tôi phân biệt được Column, Row, Stack, Wrap — biết khi nào dùng cái nào? | Layout Widgets | 🔴 |
| 2 | Tôi dùng được Expanded, Flexible, Spacer để control flex behavior? | Flex Widgets | 🔴 |
| 3 | Tôi hiểu Container vs SizedBox vs ConstrainedBox — sizing behavior? | Container Widgets | 🔴 |
| 4 | Tôi dùng được Text, Image, Icon, Card để hiển thị content? | Display Widgets | 🔴 |
| 5 | Tôi phân biệt GestureDetector vs InkWell cho touch handling? | Input Widgets | 🟡 |
| 6 | Tôi dùng được ListView.builder vs GridView vs CustomScrollView? | List Widgets | 🔴 |
| 7 | Tôi hiểu BottomNavigationBar vs Drawer vs TabBar navigation patterns? | Navigation Widgets | 🟡 |
| 8 | Tôi dùng được showDialog, showSnackBar, showModalBottomSheet? | Overlay Widgets | 🟡 |
| 9 | Tôi phân biệt MediaQuery vs LayoutBuilder cho responsive design? | Responsive Widgets | 🟡 |
| 10 | Tôi hiểu FittedBox, AspectRatio, ClipRRect cho advanced sizing? | Advanced Widgets | 🟢 |

> **Target:** 5/5 Yes cho 🔴 MUST-KNOW, tối thiểu 8/10 tổng.

---

## 2. Exercise Verification

### Exercise 1 — Widget Catalog Explorer ⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| Widget identification | ≥ 10 unique widgets | ☐ |
| Usage documentation | Type, parent, purpose, key props | ☐ |
| Pattern recognition | Layout vs Display vs Input classification | ☐ |
| Câu hỏi | 3 câu trả lời | ☐ |

### Exercise 2 — Build Dashboard Layout ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| Stats row | Row + 3 Expanded cards | ☐ |
| Featured card | Stack + Positioned overlay | ☐ |
| Action buttons | ElevatedButton + OutlinedButton | ☐ |
| Recent list | ListView.separated | ☐ |
| build_runner | Không error | ☐ |
| Revert | All changes reverted | ☐ |

### Exercise 3 — ListView with Actions ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| Swipe-to-delete | Dismissible với direction.endToStart | ☐ |
| Pull-to-refresh | RefreshIndicator | ☐ |
| Add dialog | AlertDialog với TextField | ☐ |
| Undo action | SnackBar action | ☐ |
| Empty state | Center + Text when list empty | ☐ |
| Revert | All changes reverted | ☐ |

### Exercise 4 — Modal Bottom Sheet ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| DraggableScrollableSheet | initialChildSize, minChildSize, maxChildSize | ☐ |
| Sort options | ChoiceChip với single select | ☐ |
| Filter options | FilterChip với multi select | ☐ |
| Reset button | Clear all selections | ☐ |
| Apply button | Return filter result | ☐ |
| Drag behavior | Expand/collapse khi drag | ☐ |
| Revert | All changes reverted | ☐ |

### Exercise 5 — AI Prompt Dojo ⭐⭐⭐

**Verification points:**

- [ ] AI cung cấp complete widget tree
- [ ] AI đề xuất appropriate widgets
- [ ] AI address performance
- [ ] Bạn verify được suggestions
- [ ] Bạn viết đánh giá

---

## 3. Concept Cross-Check

| # | Scenario | Đáp án đúng |
|---|----------|-------------|
| 1 | Column với 3 children, muốn children equal width → dùng gì? | Wrap children trong `Expanded` |
| 2 | Row muốn last child ở far right → dùng gì? | `Spacer()` hoặc `Expanded` |
| 3 | Stack children overlap → default position ở đâu? | Top-left corner (Alignment.topLeft) |
| 4 | ListView với 1000 items → dùng ListView gì? | `ListView.builder` (lazy loading) |
| 5 | GridView với 2 columns, equal width → dùng gì? | `SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 2)` |
| 6 | GestureDetector vs InkWell → khác nhau chính? | InkWell có ripple effect, GestureDetector không |
| 7 | showDialog barrierDismissible: false → ý nghĩa? | User không thể tap outside để dismiss |
| 8 | MediaQuery.of(context) vs LayoutBuilder → khác nhau? | MediaQuery: device info từ root; LayoutBuilder: constraints từ parent |
| 9 | FittedBox fit: BoxFit.contain → behavior? | Scale child to fit, maintain aspect ratio |
| 10 | AspectRatio aspectRatio: 16/9 → ý nghĩa? | Width:height = 16:9 |

---

## 4. Common Mistakes

| # | Sai lầm | Hậu quả | Fix |
|---|---------|---------|-----|
| 1 | Dùng `ListView` với `children` cho large list | Performance, all items created upfront | Dùng `ListView.builder` |
| 2 | Quên `shrinkWrap: true` khi nested ListView | Infinite height error | Thêm `shrinkWrap: true` |
| 3 | Stack children không Positioned | Children overlap at top-left | Wrap with `Positioned` hoặc dùng `Column`/`Row` |
| 4 | Dùng `var` thay vì `final` cho widget variables | Unnecessary mutation | Luôn dùng `final` |
| 5 | Quên `const` constructor | Larger widget tree | Dùng `const` khi possible |
| 6 | `showDialog` không `await` | Miss dialog result | `await showDialog(...)` |
| 7 | `MediaQuery` bên ngoài `build()` | Exception | Chỉ dùng trong `build()` |
| 8 | Container với both `width` và `constraints` | Constraint override width | Chỉ dùng 1 sizing method |

---

## 5. Module Completion Criteria

### Yêu cầu tối thiểu

| Tiêu chí | Yêu cầu | Trạng thái |
|----------|----------|-------------|
| Self-assessment | ≥ 8/10 Yes (bắt buộc 5/5 🔴) | ☐ |
| Exercise 1 | Widget catalog documented | ☐ |
| Exercise 2 | Dashboard layout builds | ☐ |
| Exercise 3 | ListView with actions works | ☐ |
| Exercise 4 | Bottom sheet functional (bonus) | ☐ |
| Concept cross-check | ≥ 8/10 đúng | ☐ |

### Kết quả

| Level | Criteria | Action |
|-------|----------|--------|
| 🟢 **Completed** | 8+/10 self-assessment + Ex1 + Ex2 + Ex3 | Chuyển Module 6 |
| 🟡 **Needs Review** | 5-7/10 self-assessment | Đọc lại concepts còn yếu |
| 🔴 **Incomplete** | < 5/10 self-assessment | Đọc lại toàn bộ module |

---

## ✅ Module Complete

Hoàn thành khi:
- Self-assessment: ≥ 8/10 Yes (bắt buộc 5/5 🔴 MUST-KNOW)
- Exercises: Ex1 + Ex2 + Ex3 hoàn thành
- Cross-check: ≥ 8/10 đúng

**Next:** [Module 6 — Custom Widgets & Animation](../module-06-custom-widgets-animation/) — StatelessWidget, StatefulWidget, lifecycle, flutter_hooks, animation basics.

<!-- AI_VERIFY: generation-complete -->
