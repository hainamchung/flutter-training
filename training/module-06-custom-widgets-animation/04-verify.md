# Verification — Kiểm tra kết quả Module 6

> Đối chiếu bài làm với [02-concept.md](./02-concept.md) và thực hành trong codebase.

---

## 1. Self-Assessment Checklist

Trả lời **Yes / No** cho từng câu. Nếu **No** → quay lại concept tương ứng trong [02-concept.md](./02-concept.md).

| # | Câu hỏi | Concept | Badge |
|---|---------|---------|-------|
| 1 | Tôi phân biệt được StatelessWidget vs StatefulWidget — biết khi nào dùng cái nào? | Widget Types | 🔴 |
| 2 | Tôi mô tả được widget lifecycle: initState → build → dispose? | Lifecycle Methods | 🔴 |
| 3 | Tôi giải thích được BuildContext là "address" của widget trong tree? | BuildContext | 🔴 |
| 4 | Tôi dùng được flutter_hooks: useState, useEffect, useRef? | flutter_hooks | 🔴 |
| 5 | Tôi hiểu useEffect với dependency array: [], [deps], null? | useEffect | 🔴 |
| 6 | Tôi phân biệt được Keys: ValueKey, UniqueKey, GlobalKey? | Keys | 🟡 |
| 7 | Tôi hiểu InheritedWidget pattern — cách pass data down tree? | InheritedWidget | 🟡 |
| 8 | Tôi dùng được Implicit animations: AnimatedContainer, AnimatedOpacity? | Implicit Animations | 🟡 |
| 9 | Tôi dùng được Explicit animations: AnimationController, Tween? | Explicit Animations | 🟡 |
| 10 | Tôi hiểu AnimationController với vsync và SingleTickerProviderStateMixin? | AnimationController | 🟡 |

> **Target:** 5/5 Yes cho 🔴 MUST-KNOW, tối thiểu 7/10 tổng.

---

## 2. Exercise Verification

### Exercise 1 — Stateless Widget ⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| File location | `lib/ui/component/stat_card/stat_card.dart` | ☐ |
| const constructor | `const StatCard({...})` | ☐ |
| final fields | `final IconData icon`, `final String label`, etc. | ☐ |
| build() method | Returns composed widgets | ☐ |
| Card styling | elevation, borderRadius | ☐ |
| InkWell | Tap feedback | ☐ |
| Sử dụng trong page | Row với multiple StatCards | ☐ |
| Revert | Changes reverted | ☐ |

### Exercise 2 — Stateful Widget ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| File location | `lib/ui/component/step_counter/step_counter.dart` | ☐ |
| StatefulWidget structure | Widget + State classes | ☐ |
| initState() | Initialize `_value` | ☐ |
| didUpdateWidget() | Handle prop changes | ☐ |
| setState() | Trigger rebuild | ☐ |
| Button disabled states | At min/max limits | ☐ |
| Test behavior | Increment, decrement, limits | ☐ |
| Revert | Changes reverted | ☐ |

### Exercise 3 — flutter_hooks Migration ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| HookWidget base | `extends HookWidget` | ☐ |
| useState | `final value = useState(initialValue)` | ☐ |
| useCallback | Memoized handlers | ☐ |
| Value access | `value.value++` | ☐ |
| Code comparison | Lines of code với Stateful version | ☐ |
| Behavior test | Same behavior as Stateful | ☐ |
| Revert | Changes reverted | ☐ |

### Exercise 4 — Implicit Animations ⭐⭐

**Verification points:**

| Checkpoint | Yêu cầu | Trạng thái |
|------------|----------|-------------|
| AnimatedCrossFade | Expand/collapse content | ☐ |
| AnimatedRotation | Arrow rotation | ☐ |
| Duration | Animation timing | ☐ |
| GestureDetector | Tap to toggle | ☐ |
| Bonus: AnimatedContainer | Size/color animation | ☐ |
| Animation smooth | No jank | ☐ |
| Revert | Changes reverted | ☐ |

### Exercise 5 — AI Prompt Dojo ⭐⭐⭐

**Verification points:**

- [ ] AI cung cấp complete widget API design
- [ ] AI đưa ra đúng stateless/stateful decisions
- [ ] AI address composition opportunities
- [ ] AI nêu performance considerations
- [ ] Bạn verify được suggestions
- [ ] Bạn viết đánh giá

---

## 3. Concept Cross-Check

| # | Scenario | Đáp án đúng |
|---|----------|-------------|
| 1 | Widget cần internal state (counter) → dùng gì? | `StatefulWidget` hoặc `useState` |
| 2 | `initState()` gọi khi nào? | Khi State được tạo lần đầu |
| 3 | `dispose()` gọi khi nào? | Khi State bị hủy |
| 4 | `setState()` làm gì? | Schedule rebuild với state mới |
| 5 | `useEffect(fn, [])` chạy khi nào? | Chỉ khi mount |
| 6 | `useEffect(fn, [dep])` chạy khi nào? | Mount + khi dep thay đổi |
| 7 | `useState` vs `useRef` — khác nhau gì? | useState reactive, useRef non-reactive |
| 8 | `AnimatedContainer` cần AnimationController? | ❌ Không — implicit animation |
| 9 | `AnimationController` cần gì để tick? | `vsync` + `SingleTickerProviderStateMixin` |
| 10 | `BuildContext` là gì? | Handle đến widget's location trong tree |

---

## 4. Common Mistakes

| # | Sai lầm | Hậu quả | Fix |
|---|---------|---------|-----|
| 1 | Forgot `super.initState()` | Exception | Always call first |
| 2 | Forgot to dispose controller | Memory leak | Always dispose |
| 3 | Set state in build | Warning | Use `Future.microtask()` |
| 4 | Using context after dispose | Exception | Check `mounted` |
| 5 | useEffect without cleanup | Memory leak | Return cleanup function |
| 6 | Forgot `vsync` | Exception | Add `with SingleTickerProviderStateMixin` |
| 7 | Wrong key type for list | State lost | Use `ValueKey(item.id)` |
| 8 | StatefulWidget khi không cần | Unnecessary complexity | Use StatelessWidget hoặc useState |
| 9 | Forgot `super.dispose()` | Potential leak | Call last in dispose |
| 10 | Mutable props | Unexpected behavior | Use `final` for props |

---

## 5. Module Completion Criteria

### Yêu cầu tối thiểu

| Tiêu chí | Yêu cầu | Trạng thái |
|----------|----------|-------------|
| Self-assessment | ≥ 7/10 Yes (bắt buộc 5/5 🔴) | ☐ |
| Exercise 1 | Stateless widget created | ☐ |
| Exercise 2 | Stateful widget with lifecycle | ☐ |
| Exercise 3 | Hooks migration (bonus) | ☐ |
| Exercise 4 | Implicit animations | ☐ |
| Concept cross-check | ≥ 8/10 đúng | ☐ |

### Kết quả

| Level | Criteria | Action |
|-------|----------|--------|
| 🟢 **Completed** | 7+/10 self-assessment + Ex1 + Ex2 + Ex4 | Chuyển Module 7 |
| 🟡 **Needs Review** | 4-6/10 self-assessment | Đọc lại concepts còn yếu |
| 🔴 **Incomplete** | < 4/10 self-assessment | Đọc lại toàn bộ module |

---

## ✅ Module Complete

Hoàn thành khi:
- Self-assessment: ≥ 7/10 Yes (bắt buộc 5/5 🔴 MUST-KNOW)
- Exercises: Ex1 + Ex2 + Ex4 hoàn thành
- Cross-check: ≥ 8/10 đúng

**Next:** [Module 7 — Base UI Framework](../module-07-base-viewmodel/) — BasePage, ViewModel pattern, shared components.

<!-- AI_VERIFY: generation-complete -->
