# Exercises — Thực hành Custom Widgets & Animation

> ⚠️ Tất cả bài tập thực hiện **trên codebase `base_flutter`** — không tạo project mới.
> Prerequisite: Đã đọc xong [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md).

---

## ⭐ Exercise 1: Create Custom Stateless Widget

**Mục tiêu:** Tạo reusable stateless widget — stat card component.

### Hướng dẫn

1. Tạo file `lib/ui/component/stat_card/stat_card.dart`:

```dart
import 'package:flutter/material.dart';

import '../../../index.dart';

class StatCard extends StatelessWidget {
  const StatCard({
    super.key,
    required this.icon,
    required this.label,
    required this.value,
    this.iconColor = Colors.blue,
    this.onTap,
  });

  final IconData icon;
  final String label;
  final String value;
  final Color iconColor;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(
                icon,
                size: 32,
                color: iconColor,
              ),
              const SizedBox(height: 8),
              Text(
                value,
                style: style(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                  color: color.black,
                ),
              ),
              Text(
                label,
                style: style(
                  fontSize: 12,
                  color: Colors.grey[600],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

2. Sử dụng trong một page:

```dart
// Trong một page, thêm:
Row(
  children: [
    Expanded(
      child: StatCard(
        icon: Icons.people,
        label: 'Users',
        value: '1,234',
        iconColor: Colors.blue,
        onTap: () => print('Users tapped'),
      ),
    ),
    const SizedBox(width: 8),
    Expanded(
      child: StatCard(
        icon: Icons.shopping_cart,
        label: 'Orders',
        value: '567',
        iconColor: Colors.green,
      ),
    ),
  ],
)
```

3. Chạy app → verify rendering.

### Câu hỏi

- `StatCard` là StatelessWidget — tại sao `onTap` được truyền nhưng không cần setState?
- `const StatCard(...)` có được không? Tại sao/không?
- `mainAxisSize: MainAxisSize.min` có tác dụng gì?

### ✅ Checklist hoàn thành

- [ ] Tạo StatCard với icon, label, value
- [ ] Card styling với elevation và padding
- [ ] InkWell cho tap feedback
- [ ] Sử dụng trong một page
- [ ] Chạy app verify
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes** sau khi verify

---

## ⭐⭐ Exercise 2: Create Custom Stateful Widget

**Mục tiêu:** Tạo custom stepper widget với internal state.

### Hướng dẫn

1. Tạo file `lib/ui/component/step_counter/step_counter.dart`:

```dart
import 'package:flutter/material.dart';

import '../../../index.dart';

class StepCounter extends StatefulWidget {
  const StepCounter({
    super.key,
    this.initialValue = 0,
    this.minValue = 0,
    this.maxValue = 100,
    this.step = 1,
    this.onChanged,
  });

  final int initialValue;
  final int minValue;
  final int maxValue;
  final int step;
  final ValueChanged<int>? onChanged;

  @override
  State<StepCounter> createState() => _StepCounterState();
}

class _StepCounterState extends State<StepCounter> {
  late int _value;

  @override
  void initState() {
    super.initState();
    _value = widget.initialValue;
  }

  @override
  void didUpdateWidget(covariant StepCounter oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.initialValue != widget.initialValue) {
      _value = widget.initialValue;
    }
  }

  void _increment() {
    if (_value + widget.step <= widget.maxValue) {
      setState(() {
        _value += widget.step;
      });
      widget.onChanged?.call(_value);
    }
  }

  void _decrement() {
    if (_value - widget.step >= widget.minValue) {
      setState(() {
        _value -= widget.step;
      });
      widget.onChanged?.call(_value);
    }
  }

  @override
  Widget build(BuildContext context) {
    final canDecrement = _value - widget.step >= widget.minValue;
    final canIncrement = _value + widget.step <= widget.maxValue;

    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            onPressed: canDecrement ? _decrement : null,
            icon: Icon(Icons.remove),
            color: canDecrement ? Colors.black : Colors.grey,
          ),
          Container(
            constraints: const BoxConstraints(minWidth: 48),
            alignment: Alignment.center,
            child: Text(
              '$_value',
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          IconButton(
            onPressed: canIncrement ? _increment : null,
            icon: Icon(Icons.add),
            color: canIncrement ? Colors.black : Colors.grey,
          ),
        ],
      ),
    );
  }
}
```

2. Sử dụng trong một page:

```dart
// Trong một page:
StepCounter(
  initialValue: 5,
  minValue: 0,
  maxValue: 10,
  step: 1,
  onChanged: (value) {
    print('Step changed: $value');
  },
)
```

3. Test increment/decrement behavior.

### Câu hỏi

- `initState()` vs `didUpdateWidget()` — khác nhau gì?
- Tại sao `_value` là `late`? Nếu không có `late` thì sao?
- `canDecrement` và `canIncrement` được tính như thế nào?

### ✅ Checklist hoàn thành

- [ ] Tạo StepCounter với min/max/step
- [ ] `initState()` initialize value
- [ ] `didUpdateWidget()` handle prop changes
- [ ] `setState()` triggers rebuild
- [ ] Disable buttons at limits
- [ ] Test: increment, decrement, limits
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes**

---

## ⭐⭐ Exercise 3: Migrate to flutter_hooks

**Mục tiêu:** Convert StatefulWidget sang Hooks pattern.

### Hướng dẫn

1. Convert `StepCounter` từ Exercise 2 sang hooks:

> ⚠️ **Pattern note:** `HookWidget` cho phép sử dụng hooks (`useState`, `useCallback`) nhưng **không truy cập được Riverpod providers** (không có `ref`). Nếu cần dùng Riverpod trong hooks, dùng `HookConsumerWidget` thay thế:
> ```dart
> class StepCounterHook extends HookConsumerWidget {
>   @override
>   Widget build(BuildContext context, WidgetRef ref) { ... }
> }
> ```
> Exercise này dùng `HookConsumerWidget` vì kết hợp hooks pattern với Riverpod.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';

import '../../../index.dart';

class StepCounterHook extends HookConsumerWidget {
  const StepCounterHook({
    super.key,
    this.initialValue = 0,
    this.minValue = 0,
    this.maxValue = 100,
    this.step = 1,
    this.onChanged,
  });

  final int initialValue;
  final int minValue;
  final int maxValue;
  final int step;
  final ValueChanged<int>? onChanged;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useState — reactive state
    final value = useState(initialValue);

    // useCallback — memoized handlers
    final increment = useCallback(() {
      if (value.value + step <= maxValue) {
        value.value += step;
        onChanged?.call(value.value);
      }
    }, [value.value, step, maxValue]);

    final decrement = useCallback(() {
      if (value.value - step >= minValue) {
        value.value -= step;
        onChanged?.call(value.value);
      }
    }, [value.value, step, minValue]);

    final canDecrement = value.value - step >= minValue;
    final canIncrement = value.value + step <= maxValue;

    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            onPressed: canDecrement ? decrement : null,
            icon: Icon(Icons.remove),
            color: canDecrement ? Colors.black : Colors.grey,
          ),
          Container(
            constraints: const BoxConstraints(minWidth: 48),
            alignment: Alignment.center,
            child: Text(
              '${value.value}',
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          IconButton(
            onPressed: canIncrement ? increment : null,
            icon: Icon(Icons.add),
            color: canIncrement ? Colors.black : Colors.grey,
          ),
        ],
      ),
    );
  }
}
```

2. So sánh với StatefulWidget version:
   - Lines of code
   - Complexity
   - Readability

### Câu hỏi

- `useState(initialValue)` vs `useState(widget.initialValue)` — khác nhau gì?
- `useCallback` có cần thiết không? Nếu không dùng thì sao?
- `value.value` vs `_value` — cách access khác gì?

### ✅ Checklist hoàn thành

- [ ] Convert StepCounter sang hooks
- [ ] useState cho value
- [ ] useCallback cho handlers
- [ ] So sánh với StatefulWidget version
- [ ] Test behavior tương đương
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes**

---

## ⭐⭐ Exercise 4: Implement Implicit Animations

**Mục tiêu:** Tạo animated expand/collapse card.

### Hướng dẫn

1. Tạo file `lib/ui/component/animated_expand/animated_expand.dart`:

```dart
import 'package:flutter/material.dart';

import '../../../index.dart';

class AnimatedExpandCard extends StatefulWidget {
  const AnimatedExpandCard({
    super.key,
    required this.title,
    required this.content,
    this.initiallyExpanded = false,
  });

  final String title;
  final Widget content;
  final bool initiallyExpanded;

  @override
  State<AnimatedExpandCard> createState() => _AnimatedExpandCardState();
}

class _AnimatedExpandCardState extends State<AnimatedExpandCard> {
  late bool _isExpanded;

  @override
  void initState() {
    super.initState();
    _isExpanded = widget.initiallyExpanded;
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Header - always visible
          InkWell(
            onTap: () {
              setState(() {
                _isExpanded = !_isExpanded;
              });
            },
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    widget.title,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  AnimatedRotation(
                    duration: Duration(milliseconds: 200),
                    turns: _isExpanded ? 0.5 : 0,
                    child: const Icon(Icons.keyboard_arrow_down),
                  ),
                ],
              ),
            ),
          ),
          // Content - animated
          AnimatedCrossFade(
            duration: Duration(milliseconds: 300),
            firstChild: SizedBox(width: double.infinity),
            secondChild: Container(
              width: double.infinity,
              padding: const EdgeInsets.fromLTRB(16, 0, 16, 16),
              child: widget.content,
            ),
            crossFadeState: _isExpanded
                ? CrossFadeState.showSecond
                : CrossFadeState.showFirst,
          ),
        ],
      ),
    );
  }
}
```

2. Tạo demo page:

```dart
// Trong một page:
AnimatedExpandCard(
  title: 'Details',
  content: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text('Item 1: Description here'),
      SizedBox(height: 8),
      Text('Item 2: Description here'),
    ],
  ),
  initiallyExpanded: true,
)
```

3. Test expand/collapse animation.

### Bonus: AnimatedContainer version

```dart
class AnimatedColorBox extends StatefulWidget {
  @override
  State<AnimatedColorBox> createState() => _AnimatedColorBoxState();
}

class _AnimatedColorBoxState extends State<AnimatedColorBox> {
  bool _isSelected = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => setState(() => _isSelected = !_isSelected),
      child: AnimatedContainer(
        duration: Duration(milliseconds: 300),
        curve: Curves.easeInOut,
        width: _isSelected ? 200 : 100,
        height: _isSelected ? 200 : 100,
        decoration: BoxDecoration(
          color: _isSelected ? Colors.blue : Colors.red,
          borderRadius: BorderRadius.circular(_isSelected ? 20 : 8),
        ),
        child: Center(
          child: Text(
            _isSelected ? 'Selected' : 'Tap me',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}
```

### Câu hỏi

- `AnimatedCrossFade` vs `AnimatedContainer` — khác nhau gì?
- `AnimatedRotation` với `turns: 0.5` — rotate bao nhiêu độ?
- Animation `curve: Curves.easeInOut` — behavior gì?

### ✅ Checklist hoàn thành

- [ ] Tạo AnimatedExpandCard với AnimatedCrossFade
- [ ] AnimatedRotation cho arrow
- [ ] GestureDetector cho tap
- [ ] Bonus: AnimatedColorBox
- [ ] Test animations
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes**

---

## ⭐⭐⭐ Exercise 5: AI Prompt Dojo — Widget Architecture

### 🤖 AI Dojo — Custom Widget Architecture Review

**Mục tiêu:** Dùng AI để review và design custom widget architecture.

**Bước thực hiện:**

1. Gửi prompt sau cho AI:

```
Bạn là Flutter widget architect. Review và design custom widget architecture cho scenario sau:

**Scenario:** E-commerce app cần custom widgets:
1. ProductCard — hiển thị image, title, price, add-to-cart button
2. QuantitySelector — stepper với +/- buttons, disabled states
3. PriceDisplay — formatted price với discount strikethrough
4. RatingStars — star rating display (readonly)

Yêu cầu:
1. Design widget API (props, callbacks)
2. Quyết định Stateless vs Stateful cho mỗi widget
3. Nêu composition opportunities (reuse giữa các widgets)
4. Đề xuất file structure
5. Nêu potential performance considerations

Code structure reference:
[PASTE any relevant base_flutter widgets nếu cần]
```

2. Với mỗi AI suggestion:
   - Evaluate xem stateless/stateful decision có đúng không
   - Check xem props/callbacks design có reasonable không
   - Evaluate composition opportunities

3. Hỏi follow-up: "Với design tốt nhất, viết complete code cho ProductCard."

**✅ Tiêu chí đánh giá:**

- [ ] AI cung cấp complete widget API design
- [ ] AI đưa ra đúng stateless/stateful decisions
- [ ] AI address composition opportunities
- [ ] AI nêu performance considerations
- [ ] Bạn verify được ≥ 2 suggestions của AI
- [ ] Bạn viết 2-3 câu đánh giá: "AI tốt ở..., miss ở..., sai ở..."

---

## Exercise Summary

| # | Bài tập | Độ khó | Concept chính | Output |
|---|---------|--------|--------------|--------|
| 1 | Create Stateless Widget | ⭐ | StatelessWidget pattern | Working StatCard |
| 2 | Create Stateful Widget | ⭐⭐ | StatefulWidget + lifecycle | Working StepCounter |
| 3 | Migrate to hooks | ⭐⭐ | flutter_hooks pattern | Hooks version |
| 4 | Implicit Animations | ⭐⭐ | AnimatedContainer, AnimatedCrossFade | Animated widgets |
| 5 | AI Dojo — Widget Architecture | ⭐⭐⭐ | Widget architecture | AI design + evaluation |

**Tiếp theo:** [04-verify.md](./04-verify.md) — checklist tự đánh giá.

---

## ↩️ Revert Changes

Sau khi hoàn thành mỗi bài tập, revert changes để không ảnh hưởng codebase:

```bash
# Revert tất cả thay đổi trong bài tập
git checkout -- lib/

# Hoặc revert cụ thể file
# git checkout -- lib/path/to/modified/file.dart

# Nếu đã chạy codegen (make gen, make ep):
# 1. Revert barrel/file changes
git checkout -- lib/index.dart

# 2. Chạy lại make để clean
make gen
```

> ⚠️ **Quan trọng:** Luôn revert trước khi chuyển bài tập hoặc trước khi `git commit`. Code của bạn chỉ nên ở trong branch feature, không nên modify các base files trực tiếp.



<!-- AI_VERIFY: generation-complete -->
