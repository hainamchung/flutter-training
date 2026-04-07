# Concepts — Custom Widgets, Lifecycle & Animation

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. StatelessWidget vs StatefulWidget 🔴 MUST-KNOW

**WHY:** Chọn đúng widget type tránh unnecessary complexity.

### StatelessWidget

```dart
class WelcomeCard extends StatelessWidget {
  const WelcomeCard({
    super.key,
    required this.title,
    required this.message,
  });

  final String title;
  final String message;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          Text(title, style: Theme.of(context).textTheme.titleLarge),
          Text(message),
        ],
      ),
    );
  }
}
```

**Characteristics:**
- `const` constructor — compile-time constant
- `final` fields — immutable configuration
- `build()` is pure — no side effects
- No internal state
- Can be `const` if all values are const

**When to use:**
- Display-only widgets (labels, icons, cards)
- Props-driven rendering
- No user interaction state
- Composition of other widgets

### StatefulWidget

```dart
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  void _increment() {
    setState(() {    // Trigger rebuild
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

**Characteristics:**
- Two classes: Widget (config) + State (runtime)
- `setState()` triggers rebuild
- Can have internal state
- `initState()` for initialization
- `dispose()` for cleanup

**When to use:**
- Internal mutable state (counter, toggle)
- Controller management (FocusNode, ScrollController)
- User interaction that changes UI
- Form state

### Decision Matrix

| Scenario | Widget Type | Reason |
|----------|-------------|--------|
| Display card with props | `StatelessWidget` | No internal state |
| Toggle button | `StatefulWidget` hoặc `useState` | Internal state |
| Form with validation | `StatefulWidget` hoặc `useState` | Internal state |
| Wrapper around other widgets | `StatelessWidget` | Pure composition |
| Animation controller | `StatefulWidget` hoặc `useEffect` | Controller lifecycle |

> 💡 **FE Perspective**
> **Flutter:** `StatelessWidget` ≈ React functional component. `StatefulWidget` ≈ React class component.
> **React/Vue tương đương:** React: `const Comp = ({ title }) => <div>{title}</div>` (stateless) vs `class Comp extends React.Component { state = {} }` (stateful).
> **Khác biệt quan trọng:** Flutter `StatefulWidget` có 2 classes (widget + state), React có 1 class. Flutter hooks có thể thay thế `StatefulWidget`.

---

## 2. Widget Lifecycle Methods 🔴 MUST-KNOW

**WHY:** Hiểu lifecycle methods để initialize và cleanup resources đúng cách.

### Lifecycle Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ Widget Lifecycle                                                 │
│                                                                  │
│  Widget created                                                 │
│      ↓                                                           │
│  State<Widget> created                                          │
│      ↓                                                           │
│  initState() ──→ Called ONCE when State created                │
│      ↓                                                           │
│  didChangeDependencies() ──→ Called when dependencies change    │
│      ↓                                                           │
│  build() ──→ Build widget tree                                 │
│      ↓                                                           │
│  [User interacts] ──→ setState() triggers rebuild               │
│      ↓                                                           │
│  didUpdateWidget(oldWidget) ──→ Widget config changed         │
│      ↓                                                           │
│  build() ──→ Rebuild                                           │
│      ↓                                                           │
│  [Widget removed from tree]                                     │
│      ↓                                                           │
│  deactivate() ──→ Temporarily removed                           │
│      ↓                                                           │
│  dispose() ──→ Called ONCE when State destroyed                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### initState()

```dart
@override
void initState() {
  super.initState();  // MUST call super.initState() first
  
  // Initialize state
  _controller = TextEditingController();
  _focusNode = FocusNode();
  _subscription = stream.listen((data) { /* ... */ });
  
  // Start animations
  _animationController.forward();
  
  // Load data
  _loadData();
}
```

**Rules:**
- Called **exactly once** when State is created
- Must call `super.initState()` first
- Good for: initializing controllers, starting animations
- Avoid: calling `context` methods (not fully mounted yet)

### didChangeDependencies()

```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  
  // Called when inherited widgets change (Theme, MediaQuery, etc.)
  final theme = Theme.of(context);  // Safe here
  final mediaQuery = MediaQuery.of(context);  // Safe here
}
```

**When called:**
- After `initState()`
- When `BuildContext` changes
- When inherited widget changes (Theme, Locale, etc.)

### build()

```dart
@override
Widget build(BuildContext context) {
  // Build widget tree
  // Use state and props
  // Return widget
}
```

**Rules:**
- Called multiple times during widget lifecycle
- Must be pure (no side effects)
- Should complete quickly
- Do NOT call `setState()` here

### didUpdateWidget()

```dart
@override
void didUpdateWidget(covariant MyWidget oldWidget) {
  super.didUpdateWidget(oldWidget);
  
  // Compare old and new props
  if (oldWidget.value != widget.value) {
    _controller.text = widget.value;
  }
}
```

**When called:**
- Parent widget rebuilt with different props
- Compare `oldWidget` with `widget` to decide actions

### dispose()

```dart
@override
void dispose() {
  // Clean up resources
  _controller.dispose();
  _focusNode.dispose();
  _subscription.cancel();
  _animationController.dispose();
  
  super.dispose();  // MUST call super.dispose() last
}
```

**Rules:**
- Called **exactly once** when State is destroyed
- Must call `super.dispose()` last
- Good for: disposing controllers, canceling subscriptions
- Do NOT: call `setState()` here

### Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgot to call `super.initState()` | Exception | Always call `super.initState()` first |
| Forgot to dispose controller | Memory leak | Always dispose controllers |
| Set state in initState | Warning | Use `Future.microtask()` |
| Set state in dispose | Exception | Don't set state in dispose |
| Using context in initState | Context incomplete | Use `didChangeDependencies()` |

> 💡 **FE Perspective**
> **Flutter:** Widget lifecycle ≈ React component lifecycle.
> **React/Vue tương đương:** React `constructor()` ≈ `initState()`, `useEffect` ≈ `initState` + cleanup, `componentWillUnmount` ≈ `dispose()`.
> **Khác biệt quan trọng:** Flutter có explicit `initState`/`dispose`, React hooks có `useEffect(() => { return cleanup; }, [])`.

---

## 3. BuildContext Deep Dive 🔴 MUST-KNOW

**WHY:** `BuildContext` là key để access theme, navigation, size từ bất kỳ đâu trong widget tree.

### What is BuildContext?

`BuildContext` is a **handle to a widget's location** in the tree. It's not a widget, not an element — it's an interface to access the widget's position.

```dart
abstract class BuildContext {
  // Theme at this location
  ThemeData get theme;
  
  // MediaQuery (screen size, padding, etc.)
  MediaQueryData get mediaQuery;
  
  // Find widget by type
  InheritedWidget ancestorOfExactType<T>();
  
  // Access inherited widget
  T dependOnInheritedWidgetOfExactType<T>();
  
  // Navigator at this location
  NavigatorState get navigator;
}
```

### Common Usage Patterns

```dart
// Theme access
final theme = Theme.of(context);
final textTheme = theme.textTheme;

// MediaQuery access
final size = MediaQuery.of(context).size;
final padding = MediaQuery.of(context).padding;

// Navigation
Navigator.of(context).push(route);
Navigator.of(context).pop();

// Show dialog
showDialog(context: context, builder: ...);

// Find ancestor
final scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
```

### Context Scope

```
MaterialApp
  └─ ThemeProvider
       └─ Scaffold
            └─ MyWidget (context)
                 └─ ChildWidget (child context)
```

- `MyWidget`'s context can access `ThemeProvider` and `Scaffold`
- `ChildWidget`'s context can access `MyWidget`, `Scaffold`, `ThemeProvider`
- Each widget has its own context

### Context and Lifecycle

```dart
// ❌ DANGEROUS - using context after dispose
class DangerousWidget extends StatefulWidget {
  @override
  State<DangerousWidget> createState() => _DangerousWidgetState();
}

class _DangerousWidgetState extends State<DangerousWidget> {
  Future<void> _asyncOperation() async {
    await Future.delayed(Duration(seconds: 1));
    // context might be invalid here!
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(...);
    }
  }
}

// ✅ SAFE - check mounted first
if (mounted) {
  // Safe to use context
}

// ✅ SAFE - use WidgetsBinding.instance.addPostFrameCallback
WidgetsBinding.instance.addPostFrameCallback((_) {
  if (mounted) {
    // Safe to use context
  }
});
```

---

## 4. flutter_hooks 🔴 MUST-KNOW

**WHY:** flutter_hooks đơn giản hóa state management và lifecycle, thay thế StatefulWidget boilerplate.

### HookWidget vs HookConsumerWidget

| Base Class | Hooks | Riverpod | Use case |
|------------|-------|---------|----------|
| `HookWidget` | ✅ | ❌ | Hooks only |
| `ConsumerWidget` | ❌ | ✅ | Riverpod only |
| **`HookConsumerWidget`** | ✅ | ✅ | **Both** |

### useState

```dart
class CounterWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useState — reactive state, triggers rebuild
    final count = useState(0);
    final name = useState('Alice');

    return Column(
      children: [
        Text('Count: ${count.value}'),
        Text('Name: ${name.value}'),
        ElevatedButton(
          onPressed: () => count.value++,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

**What happens:**
1. First build: `useState(0)` creates `ValueNotifier<int>(0)`
2. `count.value++` updates the ValueNotifier
3. StateNotifier notifies listeners
4. Widget rebuilds with new value

### useEffect

```dart
class DataWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useEffect — side effects, lifecycle
    useEffect(
      () {
        // This runs after first build
        final subscription = api.stream.listen((data) {
          // Handle data
        });

        // Return cleanup function
        return () => subscription.cancel();
      },
      [],  // Dependencies - run only on mount
    );

    return Container();
  }
}
```

**Dependency array:**

| Keys | When to run |
|------|-------------|
| `[]` | Only on mount |
| `[dep1, dep2]` | On mount + when deps change |
| `null` | Every build (rare) |

### useRef

```dart
class ControllerWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useRef — non-reactive, persists across rebuilds
    final controller = useRef<TextEditingController>(
      TextEditingController(),
    );

    // Use controller.value for the actual controller
    return TextField(controller: controller.value);
  }
}
```

**useState vs useRef:**

| Hook | Reactivity | Use case |
|------|------------|----------|
| `useState<T>(init)` | ✅ Triggers rebuild | UI state (counter, toggle) |
| `useRef<T>(init)` | ❌ No rebuild | Controllers, caches, flags |

### useMemoized

```dart
class ExpensiveWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Expensive computation
    final computed = useMemoized(() {
      return _expensiveCalculation(ref);
    }, [dependency]);

    return Text('Result: $computed');
  }
}
```

### useCallback

```dart
class CallbackWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // useCallback — memoized callback
    final onPressed = useCallback(() {
      ref.read(counterProvider.notifier).increment();
    }, [ref]);

    return ElevatedButton(onPressed: onPressed, child: Text('Click'));
  }
}
```

### Custom Hooks

```dart
// Create reusable hook
Timer useTimer(Duration interval) {
  final remaining = useState(0);
  final isActive = useState(false);
  
  useEffect(() {
    if (!isActive.value) return;
    
    remaining.value = interval.inSeconds;
    final timer = Timer.periodic(Duration(seconds: 1), (_) {
      if (remaining.value > 0) {
        remaining.value--;
      } else {
        isActive.value = false;
      }
    });
    
    return () => timer.cancel();
  }, [isActive.value]);
  
  return TimerState(remaining.value, isActive.value);
}
```

---

## 5. Keys 🔴 SHOULD-KNOW

**WHY:** Keys preserve widget state when the tree changes.

### Key Types

```dart
// ValueKey - based on value
ValueKey('item_1')           // String
ValueKey(42)                 // int
ValueKey(Item(id: 1))         // Object with ==

// UniqueKey - generated unique
UniqueKey()                  // New UUID each time

// GlobalKey - accessible from anywhere
final _formKey = GlobalKey<FormState>();
GlobalKey<FormState>(debugLabel: 'login_form')
```

### When to Use Keys

| Scenario | Key needed? | Key type |
|----------|-------------|----------|
| List reorder | ✅ Yes | `ValueKey(item.id)` |
| List remove | ✅ Yes | `ValueKey(item.id)` |
| List add | ✅ Yes | `ValueKey(item.id)` |
| Static list | ❌ No | — |
| Stateful widget identity | ✅ Yes | `GlobalKey` |

### Example

```dart
// ❌ WITHOUT KEY - state lost on reorder
ListView(
  children: [
    for (final item in items)
      StatefulItemWidget(key: null, item: item),  // BAD
  ],
)

// ✅ WITH KEY - state preserved
ListView(
  children: [
    for (final item in items)
      StatefulItemWidget(key: ValueKey(item.id), item: item),  // GOOD
  ],
)
```

---

## 6. InheritedWidget Pattern 🟡 SHOULD-KNOW

**WHY:** InheritedWidget là cách Flutter pass data down the tree without explicit props.

### InheritedWidget Structure

```dart
// 1. Create the InheritedWidget
class ThemeInherited extends InheritedWidget {
  const ThemeInherited({
    super.key,
    required this.theme,
    required super.child,
  });

  final ThemeData theme;

  @override
  bool updateShouldNotify(ThemeInherited old) => theme != old.theme;
}

// 2. Create helper to access
extension ThemeInheritedExtension on BuildContext {
  ThemeData get appTheme {
    return context.dependOnInheritedWidgetOfExactType<ThemeInherited>()?.theme
        ?? Theme.of(context);
  }
}

// 3. Use in tree
ThemeInherited(
  theme: customTheme,
  child: ChildWidget(),  // Can access via context.appTheme
)
```

### Real Example — LoadingStateProvider

```dart
class LoadingStateProvider extends InheritedWidget {
  const LoadingStateProvider({
    required this.isLoading,
    required super.child,
    super.key,
  });

  final bool isLoading;

  static bool isLoadingOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<LoadingStateProvider>()
        ?.isLoading ?? false;
  }

  @override
  bool updateShouldNotify(LoadingStateProvider oldWidget) =>
      isLoading != oldWidget.isLoading;
}

// Usage in widget
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final isLoading = LoadingStateProvider.isLoadingOf(context);
    return isLoading ? CircularProgressIndicator() : Content();
  }
}
```

---

## 7. Implicit Animations 🟡 SHOULD-KNOW

**WHY:** Implicit animations provide smooth transitions without AnimationController.

### AnimatedContainer

```dart
class ExpandBox extends StatefulWidget {
  @override
  State<ExpandBox> createState() => _ExpandBoxState();
}

class _ExpandBoxState extends State<ExpandBox> {
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
        decoration: BoxDecoration(
          color: _isExpanded ? Colors.blue : Colors.red,
          borderRadius: BorderRadius.circular(_isExpanded ? 20 : 8),
        ),
      ),
    );
  }
}
```

### AnimatedOpacity

```dart
AnimatedOpacity(
  opacity: isVisible ? 1.0 : 0.0,
  duration: Duration(milliseconds: 500),
  child: Text('Fade me'),
)
```

### AnimatedCrossFade

```dart
AnimatedCrossFade(
  duration: Duration(milliseconds: 300),
  firstChild: WidgetA(),
  secondChild: WidgetB(),
  crossFadeState: showWidgetB
      ? CrossFadeState.showSecond
      : CrossFadeState.showFirst,
)
```

### TweenAnimationBuilder

```dart
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0, end: 1),
  duration: Duration(seconds: 1),
  builder: (context, value, child) {
    return Opacity(opacity: value, child: child);
  },
  child: Text('Animating'),
)
```

---

## 8. Explicit Animations 🟡 SHOULD-KNOW

**WHY:** Explicit animations give full control over animation timing and behavior.

### AnimationController

```dart
class FadeAnimation extends StatefulWidget {
  @override
  State<FadeAnimation> createState() => _FadeAnimationState();
}

class _FadeAnimationState extends State<FadeAnimation>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _fadeAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 500),
      vsync: this,  // Required for tick provider
    );
    _fadeAnimation = Tween<double>(begin: 0, end: 1).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeIn,
      ),
    );
    _controller.forward();
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

### Common Transitions

| Transition | Widget | Animation |
|------------|--------|------------|
| Fade | `FadeTransition` | `opacity` |
| Slide | `SlideTransition` | `position` (Offset) |
| Scale | `ScaleTransition` | `scale` |
| Rotation | `RotationTransition` | `turns` |
| Size | `SizeTransition` | `sizeFactor` |

### SlideTransition Example

```dart
class SlideInPage extends StatefulWidget {
  @override
  State<SlideInPage> createState() => _SlideInPageState();
}

class _SlideInPageState extends State<SlideInPage>
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

## 9. Lottie & Physics Animation 🟢 AI-GENERATE

**WHY:** Lottie provides complex animations from design tools. Physics animations simulate real-world behavior.

### Lottie Animation

```dart
import 'package:lottie/lottie.dart';

// Lottie from asset
Lottie.asset(
  'assets/animations/loading.json',
  width: 200,
  height: 200,
  repeat: true,
  reverse: true,
)

// Lottie from network
Lottie.network(
  'https://example.com/loading.json',
  width: 200,
  height: 200,
)

// Control playback
class LottieWidget extends StatefulWidget {
  @override
  State<LottieWidget> createState() => _LottieWidgetState();
}

class _LottieWidgetState extends State<LottieWidget> {
  late LottieComposition _composition;
  bool _isPlaying = false;

  @override
  Widget build(BuildContext context) {
    return Lottie(
      composition: _composition,
      controller: _controller,
      onLoaded: (composition) {
        _controller.duration = composition.duration;
      },
    );
  }
}
```

### Physics-based Animation

```dart
class PhysicsAnimation extends StatefulWidget {
  @override
  State<PhysicsAnimation> createState() => _PhysicsAnimationState();
}

class _PhysicsAnimationState extends State<PhysicsAnimation>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      // Physics-based duration
      vsync: this,
      // Spring simulation instead of fixed duration
    );
  }

  @override
  void _animateWithSpring() {
    final spring = SpringDescription(
      mass: 1,
      stiffness: 500,
      damping: 25,
    );
    final simulation = SpringSimulation(spring, 0, 1, 0);
    _controller.animateWith(simulation);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

---

## PITFALLS

| # | Pitfall | Symptom | Fix |
|---|---------|---------|-----|
| 1 | Forgot to call `super.initState()` | Exception | Always call first |
| 2 | Forgot to dispose controller | Memory leak | Always dispose |
| 3 | Set state in build | Warning/loop | Use `Future.microtask()` |
| 4 | Using context after dispose | Exception | Check `mounted` |
| 5 | useEffect without cleanup | Memory leak | Return cleanup function |
| 6 | Forgot `vsync` in AnimationController | Exception | Add `with SingleTickerProviderStateMixin` |
| 7 | Key without stable identity | State lost | Use `ValueKey(item.id)` |

---

## Cheat Sheet

### Widget Lifecycle

```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    // Initialize
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // Dependencies changed
  }

  @override
  Widget build(BuildContext context) {
    // Build
  }

  @override
  void didUpdateWidget(covariant MyWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Widget changed
  }

  @override
  void dispose() {
    // Cleanup
    super.dispose();
  }
}
```

### Hooks Quick Reference

```dart
// useState - reactive
final count = useState(0);
count.value++;

// useEffect - side effect
useEffect(() {
  final sub = stream.listen(...);
  return () => sub.cancel();
}, [dependency]);

// useRef - non-reactive
final controller = useRef(TextEditingController());

// useMemoized - cached computation
final result = useMemoized(() => compute(), [dep]);

// useCallback - memoized function
final callback = useCallback(() => doSomething(), [dep]);
```

### Animation Quick Reference

```dart
// Implicit - no controller
AnimatedContainer(
  duration: Duration(milliseconds: 300),
  width: isExpanded ? 200 : 100,
)

// Explicit - with controller
late AnimationController _controller;
late Animation<double> _animation;

@override
void initState() {
  _controller = AnimationController(duration: Duration(milliseconds: 500), vsync: this);
  _animation = Tween<double>(begin: 0, end: 1).animate(CurvedAnimation(parent: _controller, curve: Curves.easeIn));
  _controller.forward();
}

@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

FadeTransition(opacity: _animation, child: Text('Hi'))
```

---

> 📋 Badge summary → xem [00-overview.md](./00-overview.md)

**Tiếp theo:** [03-exercise.md](./03-exercise.md) — thực hành custom widgets và animation.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
