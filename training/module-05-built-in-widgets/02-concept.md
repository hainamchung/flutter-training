# Concepts — Built-in Widgets Deep Dive

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md). Cycle: **CODE → EXPLAIN → PRACTICE**.

---

## 1. Layout Widgets 🔴 MUST-KNOW

**WHY:** Layout widgets là nền tảng của mọi UI. Không nắm → không build được layout.

### Column & Row

```dart
Column(                                    // Vertical flex
  mainAxisAlignment: MainAxisAlignment.start,  // align along main axis
  crossAxisAlignment: CrossAxisAlignment.center, // align along cross axis
  mainAxisSize: MainAxisSize.min,              // or max (default)
  children: [widget1, widget2, widget3],
)

Row(                                      // Horizontal flex
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: [widget1, widget2, widget3],
)
```

**Main Axis vs Cross Axis:**

| Widget | Main Axis | Cross Axis |
|--------|-----------|------------|
| `Column` | Vertical (↓) | Horizontal (←→) |
| `Row` | Horizontal (→) | Vertical (↑↓) |

**MainAxisAlignment options:**

| Value | Column behavior | Row behavior |
|-------|-----------------|--------------|
| `start` | Top | Left |
| `center` | Center | Center |
| `end` | Bottom | Right |
| `spaceBetween` | Even, no margin | Even, no margin |
| `spaceEvenly` | Even, include margins | Even, include margins |
| `spaceAround` | Half margin at edges | Half margin at edges |

**CrossAxisAlignment options:**

| Value | Column behavior | Row behavior |
|-------|-----------------|--------------|
| `start` | Left | Top |
| `center` | Center | Center |
| `end` | Right | Bottom |
| `stretch` | Fill width | Fill height |

### Stack

```dart
Stack(
  alignment: Alignment.center,           // Align positioned children
  fit: StackFit.expand,                  // or loose, passthrough
  clipBehavior: Clip.none,               // or hardEdge, antiAlias
  children: [
    // Non-positioned child (at top-left)
    Container(width: 100, height: 100, color: Colors.red),
    
    // Positioned child (relative to Stack)
    Positioned(
      left: 10,
      top: 10,
      child: Text('Overlay'),
    ),
    
    // Centered child
    Positioned.fill(
      child: Center(child: Text('Centered')),
    ),
  ],
)
```

**Stack vs Column/Row:**

| Widget | Children arrangement | Use case |
|--------|---------------------|----------|
| `Column` | Vertical stack | List, form |
| `Row` | Horizontal stack | Toolbar, button row |
| `Stack` | Overlapping | Background + content, overlays |

### Wrap

```dart
Wrap(
  direction: Axis.horizontal,         // or vertical
  alignment: WrapAlignment.start,      // align along main axis
  spacing: 8,                         // gap between children (main axis)
  runSpacing: 8,                      // gap between lines (cross axis)
  children: [
    Chip(label: Text('Flutter')),
    Chip(label: Text('Dart')),
    Chip(label: Text('Mobile')),
  ],
)
```

**Wrap vs Flow:**

- `Wrap` tự động wrap xuống dòng mới khi hết space
- `Flow` yêu cầu manual positioning (more control, less convenience)

> 💡 **FE Perspective**
> **Flutter:** `Column`/`Row` ≈ CSS flexbox. `Stack` ≈ CSS `position: relative/absolute`. `Wrap` ≈ CSS flexbox với `flex-wrap: wrap`.
> **React/Vue tương đương:** `<div style={{ display: 'flex', flexDirection: 'column' }}>` cho Column.
> **Khác biệt quan trọng:** Flutter layout dùng widgets, không có CSS. Flutter layout là declarative widget composition.

---

## 2. Flex Widgets 🔴 MUST-KNOW

**WHY:** Flex widgets control how children share available space trong Column/Row.

### Expanded

```dart
Row(
  children: [
    // Fixed width: 100
    Container(width: 100, color: Colors.red),
    
    // Takes remaining space (flex: 1)
    Expanded(
      flex: 1,
      child: Container(color: Colors.blue),
    ),
    
    // Takes 2x remaining space
    Expanded(
      flex: 2,
      child: Container(color: Colors.green),
    ),
  ],
)
```

**Space calculation:**
- Fixed children: 100px
- Remaining: `totalWidth - 100`
- Expanded flex 1: `1/3 * remaining`
- Expanded flex 2: `2/3 * remaining`

### Flexible

```dart
Row(
  children: [
    Flexible(
      fit: FlexFit.loose,     // or tight
      flex: 1,
      child: Container(color: Colors.red),
    ),
  ],
)
```

**FlexFit.tight vs FlexFit.loose:**

| Fit | Behavior | Use case |
|-----|----------|----------|
| `tight` | Child fills available space | Force equal sizing |
| `loose` | Child only takes needed space | Max size constraint |

### Spacer

```dart
Row(
  children: [
    Icon(Icons.home),
    Spacer(),                    // Fills remaining space
    Icon(Icons.settings),
  ],
)
// Icons at opposite ends
```

**Spacer vs Expanded:**

| Widget | Purpose | Equivalent |
|--------|---------|------------|
| `Spacer()` | Push siblings apart | `Expanded(child: SizedBox())` |
| `Expanded()` | Take remaining space | Fill space + render child |

---

## 3. Container Widgets 🔴 MUST-KNOW

**WHY:** Container widgets control sizing, spacing, and visual styling.

### Container

```dart
Container(
  width: 200,                    // Fixed width (or double.infinity)
  height: 100,                   // Fixed height
  constraints: BoxConstraints(    // Min/max constraints
    minWidth: 100,
    maxWidth: 300,
    minHeight: 50,
    maxHeight: 200,
  ),
  margin: EdgeInsets.all(16),    // Outside spacing
  padding: EdgeInsets.symmetric(horizontal: 12, vertical: 8), // Inside
  decoration: BoxDecoration(      // Visual styling
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
    border: Border.all(color: Colors.black, width: 1),
    boxShadow: [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 4,
        offset: Offset(2, 2),
      ),
    ],
    gradient: LinearGradient(
      colors: [Colors.blue, Colors.red],
    ),
  ),
  transform: Matrix4.rotationZ(0.1),  // Rotate
  alignment: Alignment.center,         // Align child
  child: Text('Content'),
)
```

### SizedBox

```dart
SizedBox(
  width: 100,           // Fixed width
  height: 50,           // Fixed height
  child: Text('Fixed'),
)

// For spacing only (no child)
const SizedBox(height: 16);    // Vertical spacing
const SizedBox(width: 8);      // Horizontal spacing

// Expand to maximum
SizedBox.expand(child: Text('Fill parent'));

// Expand to minimum
SizedBox.shrink(child: Text('Shrink to child'));
```

### Padding

```dart
// Explicit padding wrapper
Padding(
  padding: EdgeInsets.all(16),
  child: Text('Padded'),
)

// Common EdgeInsets patterns
EdgeInsets.all(16)                    // All sides
EdgeInsets.only(left: 16, right: 8)  // Specific sides
EdgeInsets.symmetric(horizontal: 16, vertical: 8)  // Horizontal + vertical
EdgeInsets.fromLTRB(16, 8, 16, 8)   // Left, top, right, bottom
```

### ConstrainedBox

```dart
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: 100,
    maxWidth: 200,
    minHeight: 50,
    maxHeight: 100,
  ),
  child: Container(width: 300, height: 200),  // Will be constrained
)
```

**Container vs ConstrainedBox:**

| Widget | Purpose | Sizing |
|--------|---------|--------|
| `Container` | All-in-one box model | Can set fixed size |
| `ConstrainedBox` | Apply constraints only | Doesn't set size itself |

---

## 4. Display Widgets 🔴 MUST-KNOW

**WHY:** Display widgets show content: text, images, icons, cards.

### Text

```dart
Text(
  'Hello Flutter',
  style: TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.w400,
    color: Colors.black87,
    letterSpacing: 0.5,
    height: 1.5,
    fontFamily: 'Roboto',
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
  softWrap: true,
)
```

**TextStyle hierarchy:**

```dart
Text(
  'Text',
  style: Theme.of(context).textTheme.bodyLarge?.copyWith(
    color: Colors.red,
  ),
)
```

### Image

```dart
// Asset image
Image.asset(
  'assets/images/logo.png',
  width: 100,
  height: 100,
  fit: BoxFit.contain,
)

// Network image
Image.network(
  'https://example.com/image.png',
  width: 100,
  height: 100,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator(
      value: loadingProgress.expectedTotalBytes != null
          ? loadingProgress.cumulativeBytesLoaded /
              loadingProgress.expectedTotalBytes!
          : null,
    );
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)

// File image
Image.file(File('path/to/image.png'))
```

### Icon

```dart
Icon(
  Icons.home,
  size: 24,
  color: Colors.blue,
)

// IconButton (interactive)
IconButton(
  icon: Icon(Icons.menu),
  onPressed: () {},
  tooltip: 'Menu',
)
```

### Card

```dart
Card(
  elevation: 4,                    // Shadow depth
  margin: EdgeInsets.all(8),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(12),
  ),
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Column(
      children: [
        Text('Card Title'),
        Text('Card content'),
      ],
    ),
  ),
)
```

---

## 5. Input Widgets 🟡 SHOULD-KNOW

**WHY:** Input widgets handle user interaction: text input, button presses, gestures.

### TextField

```dart
TextField(
  controller: TextEditingController(),
  focusNode: FocusNode(),
  decoration: InputDecoration(
    labelText: 'Email',
    hintText: 'Enter your email',
    prefixIcon: Icon(Icons.email),
    suffixIcon: IconButton(
      icon: Icon(Icons.clear),
      onPressed: () => controller.clear(),
    ),
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
    ),
    filled: true,
    fillColor: Colors.grey[100],
  ),
  keyboardType: TextInputType.emailAddress,
  textInputAction: TextInputAction.next,
  onChanged: (value) => print('Changed: $value'),
  onSubmitted: (value) => print('Submitted: $value'),
  obscureText: false,
  maxLength: 100,
)
```

### ElevatedButton

```dart
ElevatedButton(
  onPressed: () => print('Pressed'),
  style: ButtonStyle(
    minimumSize: WidgetStateProperty.all(Size(200, 48)),
    backgroundColor: WidgetStateProperty.resolveWith((states) {
      if (states.contains(WidgetState.pressed)) {
        return Colors.blue[700];
      }
      return Colors.blue;
    }),
    foregroundColor: WidgetStateProperty.all(Colors.white),
    elevation: WidgetStateProperty.all(4),
    shape: WidgetStateProperty.all(
      RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
    ),
  ),
  child: Text('Submit'),
)

// TextButton (no background)
TextButton(
  onPressed: () => print('Text button'),
  child: Text('Cancel'),
)

// OutlinedButton
OutlinedButton(
  onPressed: () => print('Outlined'),
  child: Text('Options'),
)
```

### GestureDetector

```dart
GestureDetector(
  onTap: () => print('Single tap'),
  onDoubleTap: () => print('Double tap'),
  onLongPress: () => print('Long press'),
  onPanStart: (details) => print('Pan start: $details'),
  onPanUpdate: (details) => print('Pan update: $details'),
  onPanEnd: (details) => print('Pan end: $details'),
  onScaleStart: (details) => print('Scale start'),
  onScaleUpdate: (details) => print('Scale update: ${details.scale}'),
  onScaleEnd: (details) => print('Scale end'),
  behavior: HitTestBehavior.opaque,  // or translucent, opaque
  child: Container(width: 100, height: 100, color: Colors.blue),
)
```

### InkWell

```dart
InkWell(
  onTap: () => print('Tapped with ripple'),
  onDoubleTap: () => print('Double tap'),
  onLongPress: () => print('Long press'),
  splashColor: Colors.blue.withOpacity(0.3),
  highlightColor: Colors.blue.withOpacity(0.1),
  borderRadius: BorderRadius.circular(8),
  child: Container(
    padding: EdgeInsets.all(16),
    child: Text('Tap me'),
  ),
)
```

**GestureDetector vs InkWell:**

| Widget | Ripple effect | Use case |
|--------|---------------|----------|
| `GestureDetector` | ❌ No | Custom gestures, non-material |
| `InkWell` | ✅ Yes | Material ripple, buttons |

---

## 6. List Widgets 🔴 MUST-KNOW

**WHY:** List widgets render scrollable collections efficiently.

### ListView

```dart
// Fixed items (all created upfront)
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
  ],
)

// Builder (lazy, for large lists)
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(
    title: Text(items[index].title),
    subtitle: Text(items[index].subtitle),
    leading: CircleAvatar(child: Text('${index + 1}')),
    trailing: Icon(Icons.chevron_right),
    onTap: () => onItemTap(items[index]),
  ),
)

// Separated (with dividers)
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => Divider(),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
)

// Custom scroll physics
ListView(
  physics: BouncingScrollPhysics(),  // iOS bounce
  // or ClampingScrollPhysics()      // Android clamp
  children: [...],
)
```

### GridView

```dart
// Fixed cross-axis count
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,           // 2 columns
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
    childAspectRatio: 1,         // 1:1 ratio
  ),
  itemCount: items.length,
  itemBuilder: (context, index) => Card(child: Text(items[index])),
)

// Extent (fixed cross-axis extent)
GridView.builder(
  gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 150,    // Max 150px per item
    mainAxisSpacing: 8,
    crossAxisSpacing: 8,
  ),
  itemBuilder: ...,
)
```

### CustomScrollView + Slivers

```dart
CustomScrollView(
  slivers: [
    // Collapsible app bar
    SliverAppBar(
      expandedHeight: 200,
      floating: false,
      pinned: true,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('App Title'),
        background: Image.network('...', fit: BoxFit.cover),
      ),
    ),
    
    // Sliver list
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: 50,
      ),
    ),
    
    // Sliver grid
    SliverGrid(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => Card(child: Text('Grid $index')),
        childCount: 20,
      ),
    ),
  ],
)
```

**ListView vs CustomScrollView:**

| Widget | Flexibility | Performance |
|--------|-------------|-------------|
| `ListView` | Fixed type | Good for simple lists |
| `CustomScrollView` | Combine slivers | Best for mixed content |

---

## 7. Navigation Widgets 🟡 SHOULD-KNOW

**WHY:** Navigation widgets provide app navigation UI: tabs, drawers, app bars.

### BottomNavigationBar

```dart
Scaffold(
  body: IndexedStack(           // Keep pages alive
    index: _selectedIndex,
    children: [HomePage(), SettingsPage()],
  ),
  bottomNavigationBar: BottomNavigationBar(
    currentIndex: _selectedIndex,
    onTap: (index) => setState(() => _selectedIndex = index),
    type: BottomNavigationBarType.fixed,   // or shifting
    items: const [
      BottomNavigationBarItem(
        icon: Icon(Icons.home),
        activeIcon: Icon(Icons.home_filled),
        label: 'Home',
        backgroundColor: Colors.blue,
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.settings),
        activeIcon: Icon(Icons.settings_filled),
        label: 'Settings',
      ),
    ],
  ),
)
```

**Fixed vs Shifting:**

| Type | Behavior | Animation |
|------|----------|-----------|
| `fixed` | Labels always visible | Cross-fade icons |
| `shifting` | Labels animate in | Scale + color |

### Drawer

```dart
Scaffold(
  drawer: Drawer(
    child: ListView(
      padding: EdgeInsets.zero,
      children: [
        DrawerHeader(
          decoration: BoxDecoration(color: Colors.blue),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              CircleAvatar(radius: 30, child: Icon(Icons.person)),
              SizedBox(height: 8),
              Text('User Name', style: TextStyle(color: Colors.white)),
              Text('user@email.com', style: TextStyle(color: Colors.white70)),
            ],
          ),
        ),
        ListTile(
          leading: Icon(Icons.home),
          title: Text('Home'),
          onTap: () => Navigator.pop(context),
        ),
        ListTile(
          leading: Icon(Icons.settings),
          title: Text('Settings'),
          onTap: () => Navigator.pop(context),
        ),
        Divider(),
        ListTile(
          leading: Icon(Icons.logout),
          title: Text('Logout'),
          onTap: () => Navigator.pop(context),
        ),
      ],
    ),
  ),
)
```

### TabBar

```dart
DefaultTabController(
  length: 3,
  child: Scaffold(
    appBar: AppBar(
      title: Text('Tabs'),
      bottom: TabBar(
        tabs: const [
          Tab(icon: Icon(Icons.home), text: 'Home'),
          Tab(icon: Icon(Icons.search), text: 'Search'),
          Tab(icon: Icon(Icons.settings), text: 'Settings'),
        ],
        isScrollable: true,          // Scroll if many tabs
        indicatorColor: Colors.blue,
        labelColor: Colors.blue,
        unselectedLabelColor: Colors.grey,
      ),
    ),
    body: TabBarView(
      children: [
        HomeTab(),
        SearchTab(),
        SettingsTab(),
      ],
    ),
  ),
)
```

---

## 8. Overlay Widgets 🟡 SHOULD-KNOW

**WHY:** Overlay widgets show transient UI: dialogs, snackbars, bottom sheets.

### AlertDialog

```dart
showDialog(
  context: context,
  barrierDismissible: false,  // Prevent dismiss by tapping outside
  builder: (context) => AlertDialog(
    title: Text('Confirm'),
    content: Text('Are you sure you want to delete?'),
    actions: [
      TextButton(
        onPressed: () => Navigator.pop(context, false),
        child: Text('Cancel'),
      ),
      ElevatedButton(
        onPressed: () => Navigator.pop(context, true),
        child: Text('Delete'),
      ),
    ),
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
  ),
).then((result) {
  if (result == true) {
    // Delete confirmed
  }
});
```

### SnackBar

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('Item deleted'),
    action: SnackBarAction(
      label: 'Undo',
      textColor: Colors.yellow,
      onPressed: () => undoDelete(),
    ),
    behavior: SnackBarBehavior.floating,  // or fixed
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
    duration: Duration(seconds: 3),
    backgroundColor: Colors.black87,
  ),
)
```

### BottomSheet

```dart
showModalBottomSheet(
  context: context,
  isScrollControlled: true,    // Allow scroll over keyboard
  backgroundColor: Colors.white,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
  ),
  builder: (context) => DraggableScrollableSheet(
    initialChildSize: 0.6,
    minChildSize: 0.3,
    maxChildSize: 0.9,
    expand: false,
    builder: (context, scrollController) => ListView.builder(
      controller: scrollController,
      itemCount: items.length,
      itemBuilder: (context, index) => ListTile(
        title: Text(items[index]),
      ),
    ),
  ),
)
```

---

## 9. Responsive Widgets 🟡 SHOULD-KNOW

**WHY:** Responsive widgets adapt UI to different screen sizes and contexts.

### MediaQuery

```dart
class ResponsivePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final screenSize = MediaQuery.of(context).size;
    final isLandscape = MediaQuery.of(context).orientation == Orientation.landscape;
    final padding = MediaQuery.of(context).padding;
    final textScale = MediaQuery.of(context).textScaleFactor;

    return Scaffold(
      body: Column(
        children: [
          // Adaptive layout based on width
          if (screenSize.width > 600)
            WideLayout()
          else
            NarrowLayout(),
          
          // Show padding (notch, safe area)
          Padding(
            padding: EdgeInsets.only(top: padding.top),
            child: Text('Status bar area'),
          ),
        ],
      ),
    );
  }
}
```

### LayoutBuilder

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 600) {
      return WideLayout();
    } else if (constraints.maxWidth > 400) {
      return MediumLayout();
    } else {
      return NarrowLayout();
    }
  },
)
```

### FittedBox

```dart
// Scale child to fit within parent
FittedBox(
  fit: BoxFit.contain,        // or cover, fill, fitWidth, fitHeight
  alignment: Alignment.center,
  child: Image.asset('assets/logo.png'),
)

// Scale text to fit container
FittedBox(
  fit: BoxFit.scaleDown,
  child: Text('Long text that should scale down'),
)
```

### AspectRatio

```dart
AspectRatio(
  aspectRatio: 16 / 9,
  child: Container(
    decoration: BoxDecoration(color: Colors.blue),
    child: Center(child: Text('16:9')),
  ),
)
```

---

## 10. Advanced Widgets 🟢 AI-GENERATE

**WHY:** Advanced widgets provide specialized functionality.

### ClipRRect

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: Image.network('https://example.com/image.png'),
)

// ClipOval equivalent
ClipOval(
  child: Image.network('https://example.com/avatar.png'),
)
```

### InteractiveViewer

```dart
InteractiveViewer(
  minScale: 0.5,
  maxScale: 4.0,
  boundaryMargin: EdgeInsets.all(100),
  child: Image.asset('assets/large_image.png'),
)
```

### CustomPaint

```dart
CustomPaint(
  size: Size(200, 200),
  painter: _CirclePainter(color: Colors.blue),
  child: Center(child: Text('Circle')),
)

class _CirclePainter extends CustomPainter {
  _CirclePainter({required this.color});
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      Offset(size.width / 2, size.height / 2),
      size.width / 2,
      paint,
    );
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return color != oldDelegate.color;
  }
}
```

---

## PITFALLS

| # | Pitfall | Symptom | Fix |
|---|---------|---------|-----|
| 1 | `ListView` với `children` cho large list | Performance issues, all items created | Dùng `ListView.builder` |
| 2 | Nested quá nhiều `Column` trong `Column` | "RenderFlex overflowed" error | Dùng `Expanded`/`Flexible` |
| 3 | `Stack` children không positioned | Children overlap at top-left | Wrap with `Positioned` hoặc dùng `Column`/`Row` |
| 4 | `TextField` không có `controller` | Cannot read/clear text | Provide `TextEditingController` |
| 5 | `showDialog` không `await` | Miss dialog result | `await showDialog(...)` |
| 6 | `MediaQuery` bên ngoài build | Exception | Chỉ dùng trong `build()` |
| 7 | `Image.network` không handle loading/error | Blank screen on error | Provide `loadingBuilder` và `errorBuilder` |

---

## Cheat Sheet

### Layout Quick Reference

```dart
// Column (vertical)
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [...],
)

// Row (horizontal)
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [...],
)

// Stack (overlapping)
Stack(
  children: [
    Container(...),
    Positioned(left: 10, top: 10, child: ...),
  ],
)

// Wrap (flow layout)
Wrap(
  spacing: 8,
  runSpacing: 8,
  children: [...],
)
```

### List Quick Reference

```dart
// Simple list
ListView.builder(
  itemCount: items.length,
  itemBuilder: (ctx, i) => ListTile(title: Text(items[i])),
)

// Grid
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
  ),
  itemBuilder: ...,
)

// Slivers
CustomScrollView(
  slivers: [
    SliverAppBar(flexibleSpace: ...),
    SliverList(delegate: ...),
  ],
)
```

### Input Quick Reference

```dart
TextField(
  decoration: InputDecoration(
    labelText: 'Label',
    border: OutlineInputBorder(),
  ),
)

ElevatedButton(onPressed: () {}, child: Text('Click'))

GestureDetector(
  onTap: () {},
  child: Container(...),
)
```

---

> 📋 Badge summary → xem [00-overview.md](./00-overview.md)

**Tiếp theo:** [03-exercise.md](./03-exercise.md) — thực hành với widgets.

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
