# Concepts — Flutter UI Basics & Navigation Flow

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. Widget Tree & runApp 🔴 MUST-KNOW

**WHY:** Widget tree là nền tảng của mọi Flutter app. Hiểu sai → không hiểu cách Flutter render, update, hay optimize UI.

<!-- AI_VERIFY: base_flutter/lib/main.dart#L23-L26 -->
```dart
runApp(ProviderScope(
  observers: [AppProviderObserver()],
  child: MyApp(initialResource: initialResource),
));
```
<!-- END_VERIFY -->
→ Đã đọc trong [01-code-walk § main.dart](./01-code-walk.md#12-_runmyapp--app-bootstrap)

**EXPLAIN:**

**Widget là gì?**

Widget là **immutable description** của UI. Flutter không update widget trực tiếp — thay vào đó, bạn tạo **new widget** với state mới, Flutter so sánh (reconciliation) và update chỉ phần thay đổi.

```
┌─────────────────────────────────────────────────────────────┐
│ Widget = Immutable Configuration                             │
│                                                             │
│ Widget {                                                    │
│   key: ...,                                                 │
│   child: Widget,  // composition — widgets contain widgets    │
│   children: [...],                                           │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
                           ↓ rebuild
┌─────────────────────────────────────────────────────────────┐
│ Element = Live UI Object (runtime)                           │
│                                                             │
│ Element {                                                   │
│   widget: Widget,     // reference to config                 │
│   renderObject: ...,  // actual render (e.g., RenderBox)    │
│   child: Element,     // tree structure                      │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
```

**runApp flow:**

```
main() called
    ↓
runApp(Widget root)
    ↓
WidgetsFlutterBinding.ensureInitialized()
    ↓
WidgetsFlutterBinding.attachRootWidget(root)
    ↓
RenderObjectToWidgetAdapter<RenderBox>(
    child: root widget
).createElement()
    ↓
Root element created → widget tree mounted → screen painted
```

**Widget vs Element vs RenderObject:**

| Layer | Vai trò | Mutable? | Ví dụ |
|-------|---------|---------|-------|
| **Widget** | Configuration | ❌ Immutable | `Container()`, `Text()` |
| **Element** | Widget instance | ✅ Mutable | Element tree |
| **RenderObject** | Paint instructions | ✅ Mutable | `RenderBox`, `RenderFlex` |

> 💡 **FE Perspective**
> **Flutter:** Widget = configuration (như React component), Element = instance (như React fiber node), RenderObject = paint (như DOM node).
> **React/Vue tương đương:** Flutter widget ≈ React component (declarative config), Flutter element ≈ React element instance, Flutter renderObject ≈ DOM node.
> **Khác biệt quan trọng:** Flutter widget là **immutable config**, không phải runtime object. React component có thể là function trả về JSX hoặc class với state.
>
> **Element tree management:**
> - Widget (immutable config) → creates → Element (mutable instance)
> - Element tree is managed by Flutter's rendering pipeline
> - Elements are reused when widget type+key matches (reconciliation)
> - Each Element holds a reference to its Widget and its child Elements

**PRACTICE:** Mở `login_page.dart`. Đếm số widgets trong tree. Tìm widgets có `const` constructor — những widget này sẽ được reuse nếu key giống nhau.

---

## 2. MaterialApp & Scaffold 🔴 MUST-KNOW

**WHY:** Mọi Flutter app đều bắt đầu từ `MaterialApp`. `Scaffold` là standard app layout. Hiểu structure → build pages đúng cách.

<!-- AI_VERIFY: base_flutter/lib/ui/my_app.dart#L26-L45 -->
```dart
MaterialApp.router(
  routerDelegate: appRouter.delegate(...),
  routeInformationParser: appRouter.defaultRouteParser(),
  title: Constant.materialAppTitle,
  themeMode: ThemeMode.light,
  theme: lightTheme,
  darkTheme: darkTheme,
  // ...
)
```
<!-- END_VERIFY -->
→ Đã đọc trong [01-code-walk § my_app.dart](./01-code-walk.md#2-my_appdart--materialapp--router-setup)

**EXPLAIN:**

**MaterialApp properties:**

```dart
MaterialApp(
  title: 'App Name',                    // Android: app name; iOS: not used
  theme: ThemeData(...),                // Light theme
  darkTheme: ThemeData(...),           // Dark theme
  themeMode: ThemeMode.system,         // system/light/dark
  home: HomePage(),                    // Simple navigation (non-router)
  // OR router-based:
  routerDelegate: ...,                 // auto_route delegate
  routeInformationParser: ...,         // URL parser
)
```

**Scaffold — Standard App Layout:**

```dart
Scaffold(
  appBar: AppBar(...),                  // Top app bar
  body: Column([                       // Main content
    Header(),
    Content(),
  ]),
  bottomNavigationBar: ...,             // Bottom tabs (optional)
  floatingActionButton: ...,           // FAB (optional)
  drawer: Drawer(...),                 // Side drawer (optional)
  endDrawer: Drawer(...),              // Right-side drawer
)
```

**MaterialApp vs CupertinoApp:**

| Widget | Design System | Use case |
|--------|-------------|----------|
| `MaterialApp` | Material Design (Google) | Default, cross-platform |
| `CupertinoApp` | iOS Human Interface | iOS-only apps |
| `WidgetsApp` | No styling | Custom design systems |

> 💡 **FE Perspective**
> **Flutter:** `MaterialApp` ≈ `<MaterialUIProvider>` trong React (Material-UI). `Scaffold` ≈ `<main>` layout với standard slots.
> **React/Vue tương đương:** Material-UI `<MuiThemeProvider><CssBaseline><AppBar /><Drawer /><main>...</main></CssBaseline></MuiThemeProvider>`.
> **Khác biệt quan trọng:** Flutter `Scaffold` có built-in slots cho app bar, bottom nav, FAB, drawers. React Material-UI dùng CSS components.

**PRACTICE:** Trong `login_page.dart`, trace widget tree từ `Scaffold` (trong `CommonScaffold`) → `body` → `Stack`. Xác định mỗi widget's role.

---

## 3. Layout Widgets (Column, Row, Container) 🔴 MUST-KNOW

**WHY:** Layout widgets là building blocks của mọi UI. Không nắm → không build được layout.

**EXPLAIN:**

**Column & Row — Flex Layout:**

```dart
Column(                                    // Vertical layout - main axis is vertical
  mainAxisAlignment: MainAxisAlignment.start,  // align along main axis
  crossAxisAlignment: CrossAxisAlignment.center, // align along cross axis
  children: [...],
)

Row(                                       // Horizontal layout - main axis is horizontal
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [...],
)
```

**Main Axis vs Cross Axis:**

```
Column (vertical)
┌─────────────────────┐
│  ← Cross Axis →     │
│    ┌─────────────┐  │
│    │             │  │
│    │   Main Axis │  │
│    │     ↓       │  │
│    └─────────────┘  │
└─────────────────────┘

Row (horizontal)
┌─────────────────────┐
│  ← Main Axis →     │
│  ┌─────┬─────┬────┐ │
│  │     │     │    │ │
│  └─────┴─────┴────┘ │
│         ↑           │
│    Cross Axis       │
└─────────────────────┘
```

**MainAxisAlignment values:**

| Value | Column behavior | Row behavior |
|-------|-----------------|--------------|
| `start` | Children at top | Children at left |
| `center` | Children centered | Children centered |
| `end` | Children at bottom | Children at right |
| `spaceBetween` | Even space, no margin | Even space, no margin |
| `spaceEvenly` | Even space, include margins | Even space, include margins |
| `spaceAround` | Half margin at edges | Half margin at edges |

**Container — Box Model:**

```dart
Container(
  width: 200,                    // Fixed width
  height: 100,                   // Fixed height
  constraints: BoxConstraints(   // Min/max constraints
    minWidth: 100,
    maxWidth: 300,
  ),
  margin: EdgeInsets.all(16),   // Outside spacing
  padding: EdgeInsets.symmetric(horizontal: 12), // Inside spacing
  decoration: BoxDecoration(     // Visual styling
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
    border: Border.all(color: Colors.black),
  ),
  alignment: Alignment.center,   // Align child
  child: Text('Hello'),
)
```

**Flutter Box Model ≈ CSS Box Model:**

```
┌─── margin ────────────────────────────────┐
│  ┌─── border ──────────────────────────┐  │
│  │  ┌─── padding ───────────────────┐  │  │
│  │  │                                 │  │  │
│  │  │         content               │  │  │
│  │  │                                 │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

**SizedBox vs Container:**

| Widget | Purpose | Const constructor? |
|--------|---------|-------------------|
| `SizedBox(width, height)` | Simple sizing | ✅ `const` |
| `Container(width, height)` | Full box model | ❌ Mutable |

```dart
// Preferred for simple spacing
const SizedBox(height: 24);          // ✅ const constructor
const SizedBox(width: 16);

// For styling, use Container
Container(
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(...),
)
```

> 💡 **FE Perspective**
> **Flutter:** `Column` ≈ CSS `display: flex; flex-direction: column`. `Row` ≈ CSS `display: flex`. `Container` ≈ `<div>` với width/height/margin/padding/border.
> **React/Vue tương đương:** `<div style={{ display: 'flex', flexDirection: 'column' }}>` cho Column.
> **Khác biệt quan trăng:** Flutter layout dùng widget composition, không có CSS. Flutter có explicit widgets cho layout concepts (Column, Row, Stack) thay vì CSS properties.

**PRACTICE:** Trong `login_page.dart`, tìm:
1. `Column` với `crossAxisAlignment: CrossAxisAlignment.start` — align items như thế nào?
2. `SizedBox(height: 24)` — tạo spacing bao nhiêu pixels?
3. `Container` với decoration — style gì?

---

## 4. BuildContext — Widget Address 🔴 MUST-KNOW

**WHY:** `BuildContext` dùng ở khắp nơi: navigation, theme, size, navigation. Hiểu sai → crash hoặc wrong behavior.

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart#L49-L54 -->
```dart
return CommonScaffold(
  body: Stack(
    children: [
      CommonImage.asset(...),
      CommonScrollbarWithIosStatusBarTapDetector(
        routeName: LoginRoute.name,
        controller: scrollController,
        child: SingleChildScrollView(...)
```
<!-- END_VERIFY -->

**EXPLAIN:**

**BuildContext là gì?**

`BuildContext` là **handle đến location** của widget trong tree. Nó không phải widget, không phải element — nó là interface để access widget's location.

```dart
abstract class BuildContext {
  // Access theme at this location
  ThemeData get theme;

  // Access media query (screen size) at this location
  MediaQueryData get mediaQuery;

  // Find nearest ancestor of type T
  InheritedWidget ancestorOfExactType<T>();

  // Access inherited widget of type T
  T dependOnInheritedWidgetOfExactType<T>();

  // Navigate from this location
  NavigatorState get navigator;

  // ... other methods
}
```

**Context như Address:**

```
Widget Tree:
┌─────────────────────────────────────────────────┐
│ MaterialApp                                    │
│  └─ MyApp                                     │
│       └─ Scaffold                             │
│            └─ Column                          │ ← context ở đây
│                 ├─ Text("Login")              │
│                 └─ TextField                  │ ← context ở đây
└─────────────────────────────────────────────────┘

context.theme     → tìm ThemeData từ root đến đây
context.mediaQuery → tìm MediaQuery từ root đến đây
context.navigator  → tìm Navigator từ root đến đây
```

**Khi nào dùng BuildContext?**

| Operation | API | Ví dụ |
|-----------|-----|-------|
| Theme | `Theme.of(context)` | `Theme.of(context).textTheme` |
| MediaQuery | `MediaQuery.of(context)` | `MediaQuery.of(context).size` |
| Navigation | `Navigator.of(context)` | `Navigator.of(context).pop()` |
| Show dialog | `showDialog(context: context)` | `showDialog(...)` |
| Find ancestor | `context.findAncestorWidgetOfExactType<T>()` | Custom widget lookup |

**Common mistake:**

```dart
// ❌ WRONG — context used outside build
class MyWidget extends StatelessWidget {
  void _onTap(BuildContext context) {
    // context passed as parameter — OK
    Theme.of(context); // OK
  }

  @override
  Widget build(BuildContext context) {
    // context here is BuildContext of THIS widget
    // Using it after async operation might fail
    Future.microtask(() => Theme.of(context)); // ⚠️ OK but risky
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** `BuildContext` ≈ React's `context` trong `useContext()`. Nó là locator để tìm data từ tree.
> **React/Vue tương đương:** `React.createContext()` + `useContext(context)`. Flutter dùng implicit context lookup (`Theme.of(context)`), React dùng explicit Provider.
> **Khác biệt quan trọng:** Flutter context lookup đi từ widget lên tree đến khi tìm widget. React context Provider wrap tree, context chỉ available bên trong.

**PRACTICE:** Trong `common_scaffold.dart`, tìm `ViewUtil.hideKeyboard(context)`. Trace: context ở đâu trong tree? `hideKeyboard` tìm gì qua context?

---

## 5. Basic Navigation (push/pop) 🟡 SHOULD-KNOW

**WHY:** Navigation là cách user di chuyển giữa screens. Basic push/pop là foundation.

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/view_model/login_view_model.dart#L1-L30 -->
```dart
await _ref.read(appNavigatorProvider).replaceAll([MainRoute()]);
```
<!-- END_VERIFY -->

**EXPLAIN:**

**Route Stack Model:**

```
┌─────────────────────────────────────────────────┐
│ Navigation Stack (LIFO)                          │
│                                                  │
│  ┌─────────────────────────────────────────┐    │
│  │ Top of stack (current screen)           │    │
│  ├─────────────────────────────────────────┤    │
│  │ Screen B                                │    │
│  ├─────────────────────────────────────────┤    │
│  │ Screen A (root)                         │    │
│  └─────────────────────────────────────────┘    │
│                                                  │
│  push(B)      → stack = [A, B]                  │
│  push(C)      → stack = [A, B, C]                │
│  pop()        → stack = [A, B]                  │
│  pop()        → stack = [A]                      │
└─────────────────────────────────────────────────┘
```

**Push — Navigate forward:**

```dart
// Navigate to a new screen
context.router.push(const DetailRoute(itemId: 42));

// With result (receives data when popped)
final result = await context.router.push<String>(DetailRoute());
```

**Pop — Navigate back:**

```dart
// Pop current screen
context.router.maybePop();

// Pop with result
context.router.maybePop('success');

// Force pop (skip guard check)
context.router.pop();
```

**Replace — Replace current screen:**

```dart
// Replace current with new (no back to previous)
context.router.replace(const LoginRoute());
```

**ReplaceAll — Clear stack and push:**

```dart
// After logout — clear stack and go to login
context.router.replaceAll([const LoginRoute()]);
```

**AppNavigator wrapper (recommended):**

```dart
// Instead of context.router directly, use AppNavigator
ref.read(appNavigatorProvider).push(const DetailRoute());
ref.read(appNavigatorProvider).pop();
ref.read(appNavigatorProvider).replaceAll([MainRoute()]);
```

> 💡 **FE Perspective**
> **Flutter:** Route stack = navigation state. `push` thêm screen mới, `pop` quay lại.
> **React/Vue tương đương:** React Router `navigate('/detail')` + `navigate(-1)`. Stack-based navigation = URL stack trong browser.
> **Khác biệt quan trọng:** Flutter stack là explicit object, React router dùng URL. Flutter navigation state management tường minh hơn.

**PRACTICE:** Trace navigation khi login thành công:
1. `LoginPage` → `LoginViewModel.login()`
2. Call `replaceAll([MainRoute()])`
3. Stack trước: `[SplashRoute, LoginRoute]`
4. Stack sau: `[MainRoute]` (Splash + Login removed)

---

## 6. StatelessWidget vs StatefulWidget 🟡 SHOULD-KNOW

**WHY:** Chọn đúng widget type tránh unnecessary complexity và performance issues.

**EXPLAIN:**

**StatelessWidget:**

```dart
class GreetingText extends StatelessWidget {
  const GreetingText({super.key, required this.name});

  final String name;

  @override
  Widget build(BuildContext context) {
    return Text('Hello, $name!');
  }
}
```

- Không có internal state
- Props (constructor params) là only input
- Build output chỉ phụ thuộc props

**StatefulWidget:**

```dart
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;  // Internal state

  void _increment() {
    setState(() {      // Trigger rebuild
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
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

**Khi nào dùng gì?**

| Scenario | Widget type | Lý do |
|----------|-------------|--------|
| Pure display, props only | `StatelessWidget` | Simpler, less code |
| Internal state (counter, toggle) | `StatefulWidget` | Need `setState` |
| State từ Riverpod | `StatelessWidget` | Riverpod manages state |
| Ephemeral UI state | `StatefulWidget` hoặc `useState` | local to widget |

> 💡 **FE Perspective**
> **Flutter:** `StatelessWidget` ≈ React functional component. `StatefulWidget` ≈ React class component với `this.state`.
> **React/Vue tương đương:** React: `const Comp = ({ prop }) => <div>{prop}</div>` vs `class Comp extends React.Component { state = {}; render() { ... } }`.
> **Khác biệt quan trọng:** Flutter `StatefulWidget` có 2 classes (widget + state), React có 1 class. Flutter hooks có thể thay thế `StatefulWidget`.

**PRACTICE:** Trong `login_page.dart`, tìm `Consumer` widget. `Consumer` watch state từ Riverpod — nó là `StatelessWidget` nhưng vẫn rebuild khi state thay đổi.

---

## 7. Material vs Cupertino 🟢 AI-GENERATE

**WHY:** Flutter support cả Material (Android) và Cupertino (iOS) design. Chọn design system phù hợp với platform target.

**EXPLAIN:**

**Material Design Widgets:**

```dart
MaterialApp(
  theme: ThemeData(
    primarySwatch: Colors.blue,
    useMaterial3: true,
  ),
  home: Scaffold(
    appBar: AppBar(title: Text('Material')),
    body: Center(
      child: ElevatedButton(
        onPressed: () {},
        child: Text('Material Button'),
      ),
    ),
  ),
)
```

**Cupertino Design Widgets:**

```dart
CupertinoApp(
  theme: CupertinoThemeData(
    primaryColor: CupertinoColors.activeBlue,
  ),
  home: CupertinoPageScaffold(
    navigationBar: CupertinoNavigationBar(
      middle: Text('Cupertino'),
    ),
    child: Center(
      child: CupertinoButton(
        onPressed: () {},
        child: Text('iOS Button'),
      ),
    ),
  ),
)
```

**Cross-platform considerations:**

| Widget | Material | Cupertino | Cross-platform |
|--------|----------|-----------|----------------|
| Button | `ElevatedButton` | `CupertinoButton` | `TextButton`, `OutlinedButton` |
| Navigation | `AppBar` | `CupertinoNavigationBar` | `SliverAppBar` |
| Text Field | `TextField` | `CupertinoTextField` | `TextField` (adaptive) |
| Switch | `Switch` | `CupertinoSwitch` | — |
| Date Picker | `showDatePicker` | `CupertinoDatePicker` | — |

> 💡 **FE Perspective**
> **Flutter:** Material = Google design language. Cupertino = iOS Human Interface Guidelines. Web thường dùng Material-inspired components.
> **React/Vue tương đương:** Ant Design, Material-UI, Chakra UI cho React. Element Plus, Vuetify cho Vue.
> **Khác biệt quan trọng:** Flutter có **built-in** Material và Cupertino widgets. Web frameworks cần third-party UI libraries.

**PRACTICE:** Trong `main_page.dart`, tìm `BottomNavigationBar`. Đây là Material widget. `CupertinoTabBar` là Cupertino equivalent.

---

## PITFALLS

| # | Pitfall | Symptom | Fix |
|---|---------|---------|-----|
| 1 | Dùng `var` thay vì `final` cho widget | Unnecessary mutation, potential bugs | Luôn dùng `final` cho widget variables |
| 2 | Quên `const` constructor | Performance warning, larger widget tree | Dùng `const` khi possible |
| 3 | Nested太多的 `Column`/`Row` | Deep widget tree, performance issue | Flatten layout, dùng `Expanded`/`Flexible` |
| 4 | Dùng `MediaQuery.of(context)` bên ngoài build | Exception | Chỉ dùng trong `build()` hoặc guard với `context.mounted` |
| 5 | Set state trong build() | "setState during build" error | Wrap trong `Future.microtask()` |

---

## Cheat Sheet

### Widget Tree Basics

```dart
MaterialApp          // Root widget
  └─ Scaffold       // Standard app layout
       ├─ appBar    // Top bar
       ├─ body      // Main content
       ├─ drawer    // Side drawer
       └─ floatingActionButton  // FAB
```

### Layout Widgets

```dart
Column(             // Vertical layout
  children: [...],
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
)

Row(                // Horizontal layout
  children: [...],
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
)

Stack(              // Overlapping layout
  children: [...],
)

Container(          // Box model
  width: 100,
  height: 100,
  margin: EdgeInsets.all(16),
  padding: EdgeInsets.symmetric(horizontal: 12),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
  ),
)
```

### Navigation

```dart
// push
context.router.push(const DetailRoute(itemId: 42));

// pop
context.router.maybePop();

// replace
context.router.replaceAll([const LoginRoute()]);
```

---

> 📋 Badge summary → xem [00-overview.md](./00-overview.md)

**Tiếp theo:** [03-exercise.md](./03-exercise.md) — thực hành widget tree và layout.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
