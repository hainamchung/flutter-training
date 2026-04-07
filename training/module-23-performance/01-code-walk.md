# 01-code-walk.md — Performance Optimization

## CODE Walk: Performance Profiling & Optimization

Bạn sẽ đọc và trace qua các performance patterns và optimization techniques trong Flutter.

---

## 1. Performance Profiling Tools

### 1.1 Flutter DevTools Overview

Flutter DevTools cung cấp nhiều tabs cho performance analysis:

```
Performance Tab:
├── Timeline view - frame-by-frame analysis
├── CPU Profiler - Dart code execution time
├── Memory tab - memory allocation tracking
└── Widget Rebuild Stats - rebuild frequency

Flutter Inspector Tab:
├── Select widget mode
├── Widget tree view
├── Render tree view
└── Rebuilt widgets highlighting
```

### 1.2 Opening DevTools

```bash
# Method 1: Flutter run with Observatory
flutter run

# Method 2: Attach to running app
flutter attach

# Method 3: Open DevTools separately
flutter devtools
```

### 1.3 Performance Timeline

**Questions to consider:**
- What causes jank (frame drops)?
- How to identify expensive widget rebuilds?
- What's the difference between GPU and UI thread?

---

## 2. Widget Rebuild Optimization

### 2.1 Rebuild Statistics

Enable rebuild count trong `main.dart`:

```dart
// lib/main.dart
import 'package:flutter/rendering.dart';

void main() {
  // Enable paint rebuild statistics (traces what caused each paint)
  // ⚠️ Verify: debugPrintMarkNeedsPaint API may have changed in recent Flutter versions
  // Check Flutter docs for current API if this method is deprecated
  debugPrintMarkNeedsPaint(true);

  runApp(const MyApp());
}
```

> **Note**: `debugPrintMarkNeedsPaint(true)` prints a stack trace whenever a widget needs to be painted, helping identify what triggered the repaint. This is different from widget rebuild statistics — it focuses on paint operations.

Run app và xem console output để track rebuilds.

### 2.2 Tracing Widget Builds

```dart
// Example: Using debugPrint
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    debugPrint('MyWidget built');
    return Container();
  }
}
```

### 2.3 DevTools Widget Inspector

```
1. Open DevTools (flutter run --observe)
2. Go to "Flutter Inspector" tab
3. Enable "Show widget rebuilds" 
4. Interact with app
5. Watch widget count increase
```

---

## 3. Memory Leak Patterns

### 3.1 StreamSubscription Cleanup

```dart
// ❌ BAD: StreamSubscription not cancelled
class BadWidget extends StatefulWidget {
  @override
  State<BadWidget> createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = someStream.listen((data) {
      // handle data
    });
  }

  // ❌ MISSING: _subscription.cancel()
}
```

```dart
// ✅ GOOD: Proper cleanup
class GoodWidget extends StatefulWidget {
  @override
  State<GoodWidget> createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = someStream.listen((data) {
      // handle data
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
}
```

### 3.2 AnimationController Disposal

```dart
// ❌ BAD: AnimationController not disposed
class BadAnimationWidget extends StatefulWidget {
  @override
  State<BadAnimationWidget> createState() => _BadAnimationWidgetState();
}

class _BadAnimationWidgetState extends State<BadAnimationWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  // ❌ MISSING: _controller.dispose()
}
```

```dart
// ✅ GOOD: Proper disposal
class GoodAnimationWidget extends StatefulWidget {
  @override
  State<GoodAnimationWidget> createState() => _GoodAnimationWidgetState();
}

class _GoodAnimationWidgetState extends State<GoodAnimationWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 3.3 TextEditingController Disposal

```dart
// ✅ GOOD: TextEditingController cleanup
class FormWidget extends StatefulWidget {
  @override
  State<FormWidget> createState() => _FormWidgetState();
}

class _FormWidgetState extends State<FormWidget> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }
}
```

---

## 4. ListView Optimization

### 4.1 ListView.builder vs ListView

```dart
// ❌ BAD: ListView with children - renders ALL items
ListView(
  children: [
    for (final item in items) MyListItem(item: item),
  ],
)

// ✅ GOOD: ListView.builder - renders only visible items
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return MyListItem(item: items[index]);
  },
)
```

### 4.2 Adding Keys

```dart
// With ValueKey - helps Flutter track items
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return MyListItem(
      key: ValueKey(items[index].id),
      item: items[index],
    );
  },
)
```

### 4.3 itemExtent for Fixed Height

```dart
// itemExtent improves scrolling performance
ListView.builder(
  itemCount: 1000,
  itemExtent: 80.0, // Fixed height for all items
  itemBuilder: (context, index) {
    return ListTile(title: Text('Item $index'));
  },
)
```

---

## 5. Const Constructors

### 5.1 Impact of Const

```dart
// ❌ BAD: Non-const widgets rebuild on parent rebuild
class BadWidget extends StatelessWidget {
  final String title;

  BadWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(this.title),           // Rebuilds every time
        Container(
          color: Colors.blue,       // Rebuilds every time
          child: Text('Static'),    // Rebuilds every time
        ),
      ],
    );
  }
}
```

```dart
// ✅ GOOD: Const widgets don't rebuild unnecessarily
class GoodWidget extends StatelessWidget {
  final String title;

  const GoodWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(this.title),           // Only this rebuilds
        const SizedBox(height: 16),  // Never rebuilds
        const _StaticContent(),     // Never rebuilds
      ],
    );
  }
}

class _StaticContent extends StatelessWidget {
  const _StaticContent();

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.blue,
      child: const Text('Static content'),
    );
  }
}
```

---

## 6. RepaintBoundary

### 6.1 Preventing Repaints

```dart
// ❌ BAD: Expensive widget causes parent repaint
class BadExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Header'),
        CustomPaint(  // Expensive painting
          painter: ExpensivePainter(),
        ),
        const Text('Footer'),
      ],
    );
  }
}
```

```dart
// ✅ GOOD: RepaintBoundary isolates repaints
class GoodExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Header'),
        RepaintBoundary(
          child: CustomPaint(
            painter: ExpensivePainter(),
          ),
        ),
        const Text('Footer'),
      ],
    );
  }
}
```

---

## 7. Isolate for Heavy Computation

### 7.1 Using compute()

```dart
import 'package:flutter/foundation.dart';

// ❌ BAD: Heavy computation blocks UI thread
class BadWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final result = _heavyComputation(1000000000);  // Blocks UI!
    return Text('Result: $result');
  }
}

// ✅ GOOD: Heavy computation in isolate
class GoodWidget extends StatefulWidget {
  @override
  State<GoodWidget> createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  int _result = 0;
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _runInBackground();
  }

  Future<void> _runInBackground() async {
    final result = await compute(_heavyComputation, 1000000000);
    if (mounted) {
      setState(() {
        _result = result;
        _loading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return _loading
        ? const CircularProgressIndicator()
        : Text('Result: $_result');
  }
}

int _heavyComputation(int n) {
  // Heavy CPU-bound computation
  int sum = 0;
  for (int i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
}
```

### 7.2 Using Isolate Directly

```dart
import 'dart:isolate';

Future<int> heavyComputationInIsolate(int n) async {
  final receivePort = ReceivePort();
  
  await Isolate.spawn(
    _isolateEntry,
    [receivePort.sendPort, n],
  );
  
  final result = await receivePort.first as int;
  return result;
}

void _isolateEntry(List<dynamic> args) {
  final sendPort = args[0] as SendPort;
  final n = args[1] as int;
  
  int sum = 0;
  for (int i = 0; i < n; i++) {
    sum += i;
  }
  
  sendPort.send(sum);
}
```

---

## 8. CachedNetworkImage

```dart
// ❌ BAD: NetworkImage rebuilds on scroll
class BadImageList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return Image.network('https://example.com/image$index.jpg');
      },
    );
  }
}

// ✅ GOOD: CachedNetworkImage caches images
class GoodImageList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return CachedNetworkImage(
          imageUrl: 'https://example.com/image$index.jpg',
          placeholder: (context, url) => const CircularProgressIndicator(),
          errorWidget: (context, url, error) => const Icon(Icons.error),
        );
      },
    );
  }
}
```

---

## Summary

Qua bước code walk này, bạn đã:

1. **DevTools:** Hiểu cách use performance profiling tools
2. **Memory:** Hiểu proper disposal patterns
3. **Rebuilds:** Hiểu const constructors và keys
4. **Lists:** Hiểu ListView.builder optimization
5. **Isolates:** Hiểu offloading heavy computation

→ Tiếp theo: [02-concept.md](./02-concept.md) — giải thích chi tiết từng concept

<!-- AI_VERIFY: generation-complete -->
