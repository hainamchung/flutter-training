# Code Walk — Custom Widgets, Lifecycle & Animation

> 📌 **Recap từ Module 5:**
> - Built-in widgets: Column, Row, Stack, Container, TextField, ElevatedButton
> - List widgets: ListView, GridView, CustomScrollView
> - Navigation widgets: BottomNavigationBar, Drawer, TabBar
> - Overlay widgets: Dialog, SnackBar, BottomSheet
>
> Nếu chưa nắm vững → quay lại [Module 5](../module-05-built-in-widgets/) trước.

---

## Walk Order

```
Custom Widget Examples
    ↓
StatelessWidget pattern (IconButton wrappers)
    ↓
StatefulWidget pattern (PrimaryTextField)
    ↓
flutter_hooks pattern (BasePage, pages)
    ↓
Custom hooks examples
    ↓
Animation examples
```

---

## 1. StatelessWidget Pattern

**StatelessWidget structure:**

```dart
class MyStatelessWidget extends StatelessWidget {
  const MyStatelessWidget({
    super.key,
    required this.title,
    this.onTap,
  });

  final String title;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: EdgeInsets.all(16),
        child: Text(title),
      ),
    );
  }
}
```

**Key characteristics:**
- `const` constructor — enable compile-time constants
- `final` fields — immutable, set at construction
- `build()` method — pure function, no side effects
- No internal state

> 💡 **FE Perspective**
> **Flutter:** StatelessWidget ≈ React functional component với props.
> **React/Vue tương đương:** `const Component = ({ title, onTap }) => <div onClick={onTap}>{title}</div>`.
> **Khác biệt quan trọng:** Flutter StatelessWidget là class-based, React là function-based.

---

## 2. StatefulWidget Pattern — PrimaryTextField

> ⚠️ **TEACHING PATTERN:** The code below is a simplified version for learning. The actual `PrimaryTextField` in `base_flutter` has additional parameters and logic. Use this as a reference pattern, not a direct copy.

<!-- AI_VERIFY: base_flutter/lib/ui/component/primary_text_field/primary_text_field.dart#L1-L50 -->
```dart
class PrimaryTextField extends StatefulWidget {
  const PrimaryTextField({
    required this.title,
    required this.hintText,
    this.onEyeIconPressed,
    this.suffixIcon,
    this.controller,
    this.onChanged,
    this.keyboardType = TextInputType.text,
    super.key,
  });

  final String title;
  final String hintText;
  final Widget? suffixIcon;
  final void Function(String)? onChanged;
  final TextInputType keyboardType;
  final TextEditingController? controller;
  final void Function(bool)? onEyeIconPressed;

  @override
  State<PrimaryTextField> createState() => _PrimaryTextFieldState();
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/component/primary_text_field/primary_text_field.dart](../../base_flutter/lib/ui/component/primary_text_field/primary_text_field.dart)

> 🔎 **Quan sát**
> - `StatefulWidget` — có internal state (obscureText)
> - Constructor params là `final` — immutable configuration
> - `super.key` — pass key to parent
> - **Hỏi:** Tại sao `controller` là nullable (`TextEditingController?`)?

---

### State class — _PrimaryTextFieldState

<!-- AI_VERIFY: base_flutter/lib/ui/component/primary_text_field/primary_text_field.dart#L29-L50 -->
```dart
class _PrimaryTextFieldState extends State<PrimaryTextField>
    with WidgetsBindingObserver, RefocusOnResumeMixin<PrimaryTextField> {
  late FocusNode _focusNode;
  bool _obscureText = true;

  @override
  bool get canManageFocus => true;

  @override
  FocusNode? get focusNode => _focusNode;

  @override
  void initState() {
    super.initState();
    _focusNode = FocusNode();
    _obscureText = widget.keyboardType == TextInputType.visiblePassword;
  }

  @override
  void dispose() {
    _focusNode.dispose();
    super.dispose();
  }

  bool get _isPassword => widget.keyboardType == TextInputType.visiblePassword;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Align(
          alignment: Alignment.centerLeft,
          child: CommonText(widget.title, ...),
        ),
        TextField(
          focusNode: _focusNode,
          onChanged: widget.onChanged,
          controller: widget.controller,
          style: style(...),
          decoration: InputDecoration(...),
          keyboardType: widget.keyboardType,
          obscureText: _isPassword ? _obscureText : false,
        ),
      ],
    );
  }
}
```
<!-- END_VERIFY -->

> 🔎 **Quan sát**
> - `initState()` — initialize `_focusNode`, set `_obscureText`
> - `dispose()` — clean up `_focusNode`
> - `with WidgetsBindingObserver` — listen to app lifecycle
> - `with RefocusOnResumeMixin` — auto-refocus on app resume
> - `setState()` — not used (state managed by parent via controller)
> - **Hỏi:** Tại sao `_focusNode` cần dispose? Điều gì xảy ra nếu quên?

---

## 3. flutter_hooks Pattern — useEffect

<!-- AI_VERIFY: base_flutter/lib/ui/page/splash/splash_page.dart#L20-L35 -->
```dart
@override
Widget buildPage(BuildContext context, WidgetRef ref) {
  useEffect(
    () {
      Future.microtask(() {
        ref.read(provider.notifier).init();
      });

      return null;
    },
    [],
  );

  return const CommonScaffold(
    body: SizedBox(),
  );
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/splash/splash_page.dart](../../base_flutter/lib/ui/page/splash/splash_page.dart)

> 🔎 **Quan sát**
> - `useEffect()` — side effect hook, runs after build
> - `[]` dependency array — run only on mount
> - `Future.microtask()` — schedule callback after build completes
> - `return null` — no cleanup function needed
> - **Hỏi:** Tại sao dùng `Future.microtask()` thay vì gọi `init()` trực tiếp?

> 💡 **FE Perspective**
> **Flutter:** `useEffect(fn, [])` ≈ React `useEffect(fn, [])`.
> **React/Vue tương đương:** `useEffect(() => { init(); }, [])` trong React.
> **Khác biệt quan trọng:** Flutter hooks cần `HookConsumerWidget` base class, React hooks tự động available trong function components.

---

## 4. flutter_hooks Pattern — useState

**useState pattern:**

```dart
class CounterWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useState — reactive state, triggers rebuild
    final count = useState(0);

    return Column(
      children: [
        Text('Count: ${count.value}'),
        ElevatedButton(
          onPressed: () => count.value++,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

**useRef pattern:**

```dart
class ControllerWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useRef — non-reactive, persists across rebuilds
    final controller = useRef(TextEditingController());

    return TextField(controller: controller.value);
  }
}
```

**useState vs useRef:**

| Hook | Reactivity | Use case |
|------|------------|----------|
| `useState<T>(init)` | ✅ Triggers rebuild | UI state (toggle, counter) |
| `useRef<T>(init)` | ❌ No rebuild | Controllers, caches, flags |

> 💡 **FE Perspective**
> **Flutter:** `useState` ≈ React `useState`. `useRef` ≈ React `useRef`.
> **React/Vue tương đương:** `const [count, setCount] = useState(0)` vs `const countRef = useRef(0)`.

---

## 5. Custom Hooks — useBackBlocker

<!-- AI_VERIFY: base_flutter/lib/common/hook/use_back_blocker.dart#L1-L20 -->
```dart
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';

import '../../index.dart';

BackBlockerResult useBackBlocker(AppNavigator nav) {
  final isAllowedToPop = useState(false);

  void handleNavigation(VoidCallback? action) {
    isAllowedToPop.value = true;
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (action != null) {
        action();
      } else {
        nav.pop();
      }
    });
  }

  return (isAllowed: isAllowedToPop.value, handleNavigation: handleNavigation);
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/common/hook/use_back_blocker.dart](../../base_flutter/lib/common/hook/use_back_blocker.dart)

> 🔎 **Quan sát**
> - Custom hook = function gọi built-in hooks
> - `useState` — reactive state cho pop permission
> - `addPostFrameCallback` — defer action đến frame sau
> - Return record type — `BackBlockerResult`
> - **Hỏi:** Tại sao cần `addPostFrameCallback`? Không dùng thì sao?

---

## 6. Custom Hooks — useFocusNodeRefocusOnResume

<!-- AI_VERIFY: base_flutter/lib/common/hook/use_focus_node_refocus_on_resume.dart#L1-L30 -->
```dart
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';

FocusNode useFocusNodeRefocusOnResume(BuildContext context) {
  final focusNode = useFocusNode();
  final controller = useRef(RefocusOnResumeController()).value;

  useOnAppLifecycleStateChange((previous, current) {
    if (context.mounted) {
      controller.handleLifecycleStateChange(
        state: current, context: context, node: focusNode,
      );
    }
  });

  return focusNode;
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/common/hook/use_focus_node_refocus_on_resume.dart](../../base_flutter/lib/common/hook/use_focus_node_refocus_on_resume.dart)

> 🔎 **Quan sát**
> - Multiple hooks composition
> - `useFocusNode()` — auto-dispose FocusNode
> - `useRef()` — persistent controller across rebuilds
> - `useOnAppLifecycleStateChange()` — lifecycle observer
> - `context.mounted` check — prevent use after dispose
> - **Hỏi:** Tại sao dùng `useRef` cho controller thay vì `useState`?

---

## 7. Animation Pattern — Implicit (AnimatedContainer)

**Implicit animation — no AnimationController needed:**

```dart
class AnimatedBox extends StatefulWidget {
  @override
  State<AnimatedBox> createState() => _AnimatedBoxState();
}

class _AnimatedBoxState extends State<AnimatedBox> {
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => setState(() => _isExpanded = !_isExpanded),
      child: AnimatedContainer(
        duration: Duration(milliseconds: 300),
        curve: Curves.easeInOut,
        width: _isExpanded ? 200 : 100,
        height: _isExpanded ? 200 : 100,
        color: _isExpanded ? Colors.blue : Colors.red,
        child: Center(child: Text('Tap me')),
      ),
    );
  }
}
```

---

## 8. Animation Pattern — Explicit (AnimationController)

**Explicit animation — AnimationController needed:**

```dart
class FadeInWidget extends StatefulWidget {
  @override
  State<FadeInWidget> createState() => _FadeInWidgetState();
}

class _FadeInWidgetState extends State<FadeInWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _fadeAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 500),
      vsync: this,  // SingleTickerProviderStateMixin
    );
    _fadeAnimation = Tween<double>(begin: 0, end: 1).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeIn),
    );
    _controller.forward();  // Start animation
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _fadeAnimation,
      child: Text('Fade In'),
    );
  }
}
```

---

## 9. SlideTransition Pattern

```dart
class SlideInWidget extends StatefulWidget {
  @override
  State<SlideInWidget> createState() => _SlideInWidgetState();
}

class _SlideInWidgetState extends State<SlideInWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Offset> _slideAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 300),
      vsync: this,
    );
    _slideAnimation = Tween<Offset>(
      begin: Offset(1, 0),  // Start from right
      end: Offset.zero,       // End at original position
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeOutCubic,
    ));
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      position: _slideAnimation,
      child: Container(width: 100, height: 100, color: Colors.blue),
    );
  }
}
```

---

## Widget Walk Summary

| Pattern | Use case | Complexity |
|---------|----------|------------|
| StatelessWidget | Pure display, props only | Simple |
| StatefulWidget | Internal state, controllers | Medium |
| Hooks (useState/useEffect) | Local state, side effects | Simple |
| Custom hooks | Reusable stateful logic | Medium |
| Implicit animation | Simple property animations | Simple |
| Explicit animation | Complex, controlled animations | Complex |

> ⏭️ **Forward:** Advanced architecture patterns trong [Module 17 — Architecture & DI](../module-17-architecture-di/).

<!-- AI_VERIFY: generation-complete -->
