# Code Walk — Built-in Widgets Deep Dive

> 📌 **Recap từ Module 4:**
> - Widget tree: mọi thứ là widget, composition pattern
> - Layout primitives: Column, Row, Container, SizedBox
> - BuildContext: widget address để access theme, navigation
> - MaterialApp & Scaffold: app shell structure
>
> Nếu chưa nắm vững → quay lại [Module 4](../module-04-flutter-ui-basics/) trước.

---

## Widget Categories Overview

```
Built-in Widgets
├── Layout Widgets      (Column, Row, Stack, Wrap, Expanded, Flexible)
├── Container Widgets   (Container, SizedBox, Padding, ConstrainedBox)
├── Display Widgets    (Text, Image, Icon, Card, Divider)
├── Input Widgets      (TextField, ElevatedButton, GestureDetector, InkWell)
├── List Widgets       (ListView, GridView, SliverAppBar, CustomScrollView)
├── Navigation Widgets (BottomNavigationBar, TabBar, Drawer)
├── Overlay Widgets     (Dialog, SnackBar, BottomSheet)
└── Responsive Widgets (MediaQuery, LayoutBuilder, FittedBox)
```

---

## 1. Layout Widgets — Column, Row, Stack, Wrap

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart#L60-L70 -->
```dart
child: Column(
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    const SizedBox(height: 100),
    CommonText(
      l10n.login,
      style: style(
        fontSize: 30,
        fontWeight: FontWeight.w700,
        color: color.black,
      ),
    ),
    const SizedBox(height: 50),
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/login/login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart)

> 🔎 **Quan sát**
> - `Column` với `crossAxisAlignment: CrossAxisAlignment.start` — align children left
> - `children` list — composition pattern, list of widgets
> - `SizedBox` tạo spacing giữa các widgets
> - **Hỏi:** `crossAxisAlignment` trong Column align theo chiều nào?

> 💡 **FE Perspective**
> **Flutter:** `Column` ≈ CSS `display: flex; flex-direction: column`. `Row` ≈ CSS `display: flex; flex-direction: row`.
> **React/Vue tương đương:** `<div style={{ display: 'flex', flexDirection: 'column' }}>`.
> **Khác biệt quan trọng:** Flutter layout widgets là explicit, không có CSS shorthand.

---

## 2. Flex Widgets — Expanded, Flexible, Spacer

<!-- AI_VERIFY: base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart#L51-L56 -->
```dart
SafeArea(
  top: enabledEdgeToEdge ? false : useSafeArea,
  bottom: enabledEdgeToEdge ? false : useSafeArea,
  left: enabledEdgeToEdge ? false : useSafeArea,
  right: enabledEdgeToEdge ? false : useSafeArea,
  child: shimmerEnabled ? Shimmer(child: body) : body ?? const SizedBox.shrink(),
),
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/component/common_scaffold/common_scaffold.dart](../../base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart)

**Expanded pattern:**

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: Container(color: Colors.red),
    ),
    Expanded(
      flex: 1,
      child: Container(color: Colors.blue),
    ),
  ],
)
// Red = 2/3 width, Blue = 1/3 width
```

> 🔎 **Quan sát**
> - `Expanded` có thể có `flex` parameter (default = 1)
> - `SafeArea` wrap content để tránh system bars
> - `Shimmer` là conditional widget — render shimmer hoặc body
> - **Hỏi:** `SafeArea` có `top`, `bottom`, `left`, `right` — điều khiển gì?

---

## 3. Container Widgets — Container, SizedBox, Padding

<!-- AI_VERIFY: base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart#L47-L50 -->
```dart
final scaffold = Scaffold(
  key: scaffoldKey,
  endDrawer: endDrawer,
  backgroundColor: backgroundColor ?? Colors.white,
  body: IgnorePointer(
```
<!-- END_VERIFY -->

> 🔎 **Quan sát**
> - `Container` với `key`, `backgroundColor`, `decoration`
> - `IgnorePointer` wrap body — prevent interaction khi loading
> - `SafeArea` padding toàn bộ body
> - **Hỏi:** `Container` vs `SizedBox` — khác nhau gì?

---

## 4. Display Widgets — Text, Image, Icon

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart#L49-L57 -->
```dart
CommonText(
  l10n.login,
  style: style(
    fontSize: 30,
    fontWeight: FontWeight.w700,
    color: color.black,
  ),
),
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/login/login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart)

**Text widget:**

```dart
Text(
  'Hello Flutter',
  style: TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.w400,
    color: Colors.black87,
    letterSpacing: 0.5,
    height: 1.5,
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

**Image widget:**

```dart
Image.asset(
  'assets/images/logo.png',
  width: 100,
  height: 100,
  fit: BoxFit.contain,
)

Image.network(
  'https://example.com/image.png',
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator();
  },
)
```

> 🔎 **Quan sát**
> - `CommonText` là wrapper quanh `Text` — thêm styling utilities
> - `l10n.login` là localized string từ slang
> - `color.black` là app color từ `AppColors`
> - **Hỏi:** Tại sao dùng `CommonText` thay vì `Text` trực tiếp?

---

## 5. Input Widgets — TextField, ElevatedButton, GestureDetector

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart#L77-L83 -->
```dart
PrimaryTextField(
  title: l10n.email,
  hintText: l10n.email,
  onChanged: (email) => ref.read(provider.notifier).setEmail(email),
  keyboardType: TextInputType.text,
  suffixIcon: const Icon(Icons.email),
),
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/login/login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart)

**ElevatedButton:**

```dart
ElevatedButton(
  onPressed: isLoginButtonEnabled
      ? () => ref.read(provider.notifier).login()
      : null,
  style: ButtonStyle(
    minimumSize: WidgetStateProperty.all(const Size(double.infinity, 48)),
    backgroundColor: WidgetStateProperty.all(Colors.black),
  ),
  child: Text('Login'),
)
```

**GestureDetector:**

```dart
GestureDetector(
  onTap: () => print('Tapped!'),
  onDoubleTap: () => print('Double tapped!'),
  onLongPress: () => print('Long pressed!'),
  onPanUpdate: (details) => print('Pan: $details'),
  child: Container(
    width: 100,
    height: 100,
    color: Colors.blue,
  ),
)
```

> 🔎 **Quan sát**
> - `PrimaryTextField` là custom widget wrap `TextField`
> - `onChanged` callback — user typing triggers callback
> - `Consumer` watch button enabled state
> - **Hỏi:** `GestureDetector` vs `InkWell` — khác nhau gì?

---

## 6. List Widgets — ListView, GridView, CustomScrollView

> ⚠️ **TEACHING PATTERN:** Code below demonstrates Flutter ListView patterns. These are teaching examples — see actual usage in `base_flutter/lib/ui/page/` for real-world implementation.

**ListView variants:**

```dart
// Fixed items
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
  ],
)

// Builder (lazy loading)
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(
    title: Text(items[index].name),
  ),
)

// Separated (with dividers)
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => Divider(),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
)
```

**GridView:**

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
    childAspectRatio: 1,
  ),
  itemCount: items.length,
  itemBuilder: (context, index) => Card(child: Text(items[index])),
)
```

**CustomScrollView with Slivers:**

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      title: Text('App Bar'),
      floating: true,
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
      ),
    ),
  ],
)
```

> 🔎 **Quan sát**
> - `ListView.builder` lazy loads items — tốt cho large lists
> - `CustomScrollView` kết hợp multiple sliver types
> - `SliverAppBar` collapsible app bar
> - **Hỏi:** Tại sao `ListView.builder` tốt hơn `ListView` với `children`?

---

## 7. Navigation Widgets — BottomNavigationBar, TabBar, Drawer

<!-- AI_VERIFY: base_flutter/lib/ui/page/main/main_page.dart#L61-L75 -->
```dart
return BottomNavigationBar(
  currentIndex: tabsRouter.activeIndex,
  onTap: (index) {
    if (index == tabsRouter.activeIndex) {
      ref.read(appNavigatorProvider).popUntilRootOfCurrentBottomTab();
    }
    tabsRouter.setActiveIndex(index);
  },
  showSelectedLabels: true,
  showUnselectedLabels: true,
  unselectedItemColor: Colors.grey,
  selectedItemColor: color.black,
  type: BottomNavigationBarType.fixed,
  backgroundColor: Colors.white,
  items: BottomTab.values
      .map(
        (tab) => BottomNavigationBarItem(
          label: tab.title,
          icon: tab.icon,
          activeIcon: tab.activeIcon,
        ),
      )
      .toList(),
);
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/main/main_page.dart](../../base_flutter/lib/ui/page/main/main_page.dart)

**BottomNavigationBar:**

```dart
BottomNavigationBar(
  currentIndex: selectedIndex,
  onTap: (index) => setState(() => selectedIndex = index),
  type: BottomNavigationBarType.fixed,  // or .shifting
  items: const [
    BottomNavigationBarItem(
      icon: Icon(Icons.home),
      label: 'Home',
    ),
    BottomNavigationBarItem(
      icon: Icon(Icons.settings),
      label: 'Settings',
    ),
  ],
)
```

**Drawer:**

```dart
Scaffold(
  drawer: Drawer(
    child: ListView(
      children: [
        DrawerHeader(child: Text('Menu')),
        ListTile(
          leading: Icon(Icons.home),
          title: Text('Home'),
          onTap: () => Navigator.pop(context),
        ),
      ],
    ),
  ),
)
```

> 🔎 **Quan sát**
> - `BottomNavigationBar` items từ enum `BottomTab`
> - `tabsRouter.setActiveIndex` switch tabs
> - `popUntilRootOfCurrentBottomTab` reset tab khi tap active tab
> - **Hỏi:** `BottomNavigationBarType.fixed` vs `shifting` — khác nhau gì?

---

## 8. Overlay Widgets — Dialog, SnackBar, BottomSheet

**showDialog:**

```dart
Future<void> showErrorDialog(BuildContext context) async {
  final result = await showDialog<bool>(
    context: context,
    barrierDismissible: false,
    builder: (context) => AlertDialog(
      title: Text('Error'),
      content: Text('Something went wrong'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: Text('Cancel'),
        ),
        ElevatedButton(
          onPressed: () => Navigator.pop(context, true),
          child: Text('Retry'),
        ),
      ],
    ),
  );

  if (result == true) {
    // Retry action
  }
}
```

**showSnackBar:**

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('Item deleted'),
    action: SnackBarAction(
      label: 'Undo',
      onPressed: () => undoDelete(),
    ),
    duration: Duration(seconds: 3),
  ),
)
```

**showModalBottomSheet:**

```dart
Future<void> showFilterSheet(BuildContext context) async {
  final result = await showModalBottomSheet<String>(
    context: context,
    isScrollControlled: true,
    builder: (context) => Container(
      padding: EdgeInsets.all(16),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ListTile(
            title: Text('Option 1'),
            onTap: () => Navigator.pop(context, 'option1'),
          ),
          ListTile(
            title: Text('Option 2'),
            onTap: () => Navigator.pop(context, 'option2'),
          ),
        ],
      ),
    ),
  );

  if (result != null) {
    // Handle selected option
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** Overlay widgets hiển thị trên cùng của screen.
> **React/Vue tương đương:** Modal, Toast, Drawer components.

---

## 9. Responsive Widgets — MediaQuery, LayoutBuilder, FittedBox

**MediaQuery:**

```dart
// Get screen size
final screenSize = MediaQuery.of(context).size;

// Get safe area insets
final padding = MediaQuery.of(context).padding;

// Check device orientation
final isLandscape = MediaQuery.of(context).orientation == Orientation.landscape;

// Get text scale factor
final textScale = MediaQuery.of(context).textScaleFactor;
```

**LayoutBuilder:**

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 600) {
      return WideLayout();  // Tablet/desktop
    } else {
      return NarrowLayout();  // Phone
    }
  },
)
```

**FittedBox:**

```dart
FittedBox(
  fit: BoxFit.contain,
  child: Image.asset('assets/logo.png'),
)
// Scale image to fit within parent while maintaining aspect ratio
```

**AspectRatio:**

```dart
AspectRatio(
  aspectRatio: 16 / 9,
  child: Container(color: Colors.blue),
)
// Fixed aspect ratio container
```

> 🔎 **Quan sát**
> - `MediaQuery` lấy thông tin device từ root
> - `LayoutBuilder` rebuild khi constraints thay đổi
> - `FittedBox` scale child to fit parent
> - **Hỏi:** `MediaQuery` vs `LayoutBuilder` — khác nhau gì?

---

## 10. Advanced Widgets — ClipRRect, CustomPaint, InteractiveViewer

**ClipRRect:**

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: Image.network('https://example.com/image.png'),
)
```

**InteractiveViewer:**

```dart
InteractiveViewer(
  minScale: 0.5,
  maxScale: 4.0,
  child: Image.asset('assets/large_image.png'),
)
// Pinch to zoom, pan to scroll
```

**CustomPaint:**

```dart
CustomPaint(
  painter: MyPainter(),
  size: Size(200, 200),
)

class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2;

    canvas.drawCircle(
      Offset(size.width / 2, size.height / 2),
      50,
      paint,
    );
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

---

## Widget Walk Summary

| Category | Widgets | Use case |
|----------|---------|---------|
| Layout | Column, Row, Stack, Wrap | Arrange children |
| Flex | Expanded, Flexible, Spacer | Control flex behavior |
| Container | Container, SizedBox, Padding, ConstrainedBox | Sizing and spacing |
| Display | Text, Image, Icon, Card | Show content |
| Input | TextField, Button, GestureDetector, InkWell | User interaction |
| List | ListView, GridView, CustomScrollView, SliverAppBar | Scrollable content |
| Navigation | BottomNavigationBar, TabBar, Drawer | Navigation UI |
| Overlay | Dialog, SnackBar, BottomSheet | Transient UI |
| Responsive | MediaQuery, LayoutBuilder, FittedBox | Adaptive layout |

> ⏭️ **Forward:** Custom widgets và animation trong [Module 6 — Custom Widgets & Animation](../module-06-custom-widgets-animation/).

<!-- AI_VERIFY: generation-complete -->
