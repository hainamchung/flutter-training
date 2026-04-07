# 02-concept.md — Performance Optimization

## CONCEPTS: Flutter Performance Deep Dive

---

## Concept 1: Flutter DevTools Performance 🔴 MUST-KNOW

### DevTools Overview

```dart
// Flutter DevTools provides:

Performance Tab:
├── Timeline View
│   ├── Frame-by-frame rendering analysis
│   ├── GPU thread vs UI thread visualization
│   ├── Jank identification (slow frames >16ms)
│   └── Shader compilation tracking
│
├── CPU Profiler
│   ├── Dart code execution sampling
│   ├── Top-down / Bottom-up views
│   ├── Flame chart visualization
│   └── Function-level timing
│
└── Memory Tab
    ├── Memory allocation timeline
    ├── Heap snapshot comparison
    ├── Memory leak detection
    └── GC events tracking
```

### Timeline Analysis

```dart
// Reading the timeline:
// - Each frame = vertical slice (~16ms for 60fps)
// - UI thread (Dart): Widget builds, animations
// - GPU thread: Rasterization, rendering
// - Platform thread: Native code, plugins

// Jank indicators:
// - Red bars = slow frames (>16ms)
// - Yellow bars = frames approaching limit
// - Blue bars = Shader compilation frames

// Common causes:
// 1. Expensive widget rebuilds
// 2. Large list rendering
// 3. Image decoding on UI thread
// 4. Large animations without optimization
// 5. Heavy computation blocking UI
```

### Widget Rebuild Stats

```dart
// Enable rebuild statistics:
// 1. Run app with: flutter run --observe
// 2. Open DevTools
// 3. Flutter Inspector → More Actions → Show rebuild counts

// What you'll see:
// - Widget rebuild count per widget
// - Per-frame rebuild highlighting
// - Tree visualization of rebuilds

// Optimization targets:
// - Minimize rebuild count
// - Isolate expensive widgets
// - Use const constructors
```

### 💡 FE Perspective

| Flutter DevTools | Chrome DevTools |
|------------------|-----------------|
| Timeline | Performance panel |
| CPU Profiler | JavaScript Profiler |
| Memory Tab | Memory panel |
| Widget Inspector | React DevTools |
| Flutter Inspector | Elements panel |

---

## Concept 2: Widget Rebuild Optimization 🔴 MUST-KNOW

### Widget Build Cycle

```dart
// Flutter widget rebuild cycle:

1. State Change
   ↓
2. setState() / Provider update / Riverpod notifier
   ↓
3. markNeedsBuild() called
   ↓
4. Element marked dirty
   ↓
5. Next frame: build() called
   ↓
6. Widget tree rebuilt
   ↓
7. Render tree updated
   ↓
8. Screen repainted

// Optimization strategies:
// 1. Minimize state changes
// 2. Use selective rebuilds (select() in Riverpod)
// 3. Const constructors
// 4. Keys for list items
// 5. RepaintBoundary
```

### Const Constructors

```dart
// Const constructor benefits:
// 1. Widget created once at compile time
// 2. Never rebuilds when parent rebuilds
// 3. Reduces widget tree complexity
// 4. Improves build performance

// Rule: Use const when widget has no dynamic values
class MyWidget extends StatelessWidget {
  final String title;        // Dynamic - cannot be const
  final int count;           // Dynamic - cannot be const
  final Widget child;        // Dynamic - cannot be const
  
  // Cannot use const constructor
  
  @override
  Widget build(BuildContext context) {
    return Text(title);  // Rebuilds every time
  }
}

class OptimizedWidget extends StatelessWidget {
  final String title;  // Dynamic - const not possible
  
  const OptimizedWidget({super.key, required this.title});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(title),           // Rebuilds
        const SizedBox(height: 16),   // Never rebuilds - const!
        const Icon(Icons.star),        // Never rebuilds - const!
        _StaticContent(),             // Never rebuilds - StatelessWidget
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
      child: const Text('Static'),
    );
  }
}
```

### Selective Rebuilds

```dart
// Riverpod: Use select() for granular rebuilds
final userProvider = StateNotifierProvider<UserNotifier, UserState>((ref) {
  return UserNotifier();
});

// ❌ BAD: Rebuilds when ANY part of user changes
class BadWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Text('Name: ${user.name}');  // Rebuilds on avatar change too
  }
}

// ✅ GOOD: Select only needed field
class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Only rebuilds when name changes
    final name = ref.watch(userProvider.select((s) => s.name));
    return Text('Name: $name');
  }
}
```

---

## Concept 3: Memory Leak Prevention 🔴 MUST-KNOW

### Common Leak Patterns

```dart
// Pattern 1: StreamSubscription not cancelled
class StreamLeakWidget extends StatefulWidget {
  @override
  State<StreamLeakWidget> createState() => _StreamLeakWidgetState();
}

class _StreamLeakWidgetState extends State<StreamLeakWidget> {
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = Stream.periodic(Duration(seconds: 1)).listen((_) {
      // This continues even after widget disposed
    });
  }

  // ❌ MISSING: Always cancel in dispose!
}

// ✅ FIXED:
class StreamFixedWidget extends StatefulWidget {
  @override
  State<StreamFixedWidget> createState() => _StreamFixedWidgetState();
}

class _StreamFixedWidgetState extends State<StreamFixedWidget> {
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = Stream.periodic(Duration(seconds: 1)).listen((_) {
      // handle
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();  // ✅ Clean up!
    super.dispose();
  }
}
```

### Controller Disposal

```dart
// Pattern 2: Controllers not disposed
class ControllerLeakWidget extends StatefulWidget {
  @override
  State<ControllerLeakWidget> createState() => _ControllerLeakWidgetState();
}

class _ControllerLeakWidgetState extends State<ControllerLeakWidget> {
  final _scrollController = ScrollController();
  final _textController = TextEditingController();
  late AnimationController _animationController;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
    
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.dispose();        // ✅
    _textController.dispose();           // ✅
    _animationController.dispose();      // ✅
    super.dispose();
  }
}
```

### Provider Auto-Dispose

```dart
// Pattern 3: Riverpod providers not using autoDispose
// ✅ Use autoDispose when provider is page-scoped
final counterProvider = StateProvider.autoDispose<int>((ref) {
  // This provider is automatically disposed
  // when no longer used
  return 0;
});

// When to use autoDispose:
// - Page-level state
// - Form state
// - Temporary UI state
// - Any state that should reset on page exit

// When NOT to use autoDispose:
// - Global app state (theme, auth)
// - Shared state across pages
// - Cached data
```

### WeakReference for Caches

```dart
// Pattern 4: Caching without cleanup
class CacheLeakWidget extends StatefulWidget {
  @override
  State<CacheLeakWidget> createState() => _CacheLeakWidgetState();
}

class _CacheLeakWidgetState extends State<CacheLeakWidget> {
  final Map<String, Widget> _cache = {};

  @override
  void dispose() {
    _cache.clear();  // ✅ Clean cache
    super.dispose();
  }
}

// ✅ Use WeakReference for caches
class SafeCacheWidget extends StatefulWidget {
  @override
  State<SafeCacheWidget> createState() => _SafeCacheWidgetState();
}

class _SafeCacheWidgetState extends State<SafeCacheWidget> {
  final Map<String, WeakReference<Widget>> _cache = {};

  Widget getOrCreate(String key, Widget Function() builder) {
    final cached = _cache[key]?.target;
    if (cached != null) return cached;
    
    final widget = builder();
    _cache[key] = WeakReference(widget);
    return widget;
  }
}
```

---

## Concept 4: Const Widgets & RepaintBoundary 🟡 SHOULD-KNOW

### RepaintBoundary

```dart
// RepaintBoundary isolates repaints
// Use when:
// 1. Widget repaints frequently (animations, custom painters)
// 2. Widget doesn't need to update when parent updates
// 3. Widget is expensive to repaint

// Example: Animation in a list
class OptimizedListItem extends StatelessWidget {
  final String title;
  
  const OptimizedListItem({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(title),
      trailing: RepaintBoundary(  // ✅ Isolates animation
        child: AnimatedIcon(),
      ),
    );
  }
}

// Example: Custom Paint with expensive painter
class OptimizedPaintWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Header'),           // Never repaints
        RepaintBoundary(               // ✅ Isolates expensive paint
          child: CustomPaint(
            painter: ExpensivePainter(),
          ),
        ),
        const Text('Footer'),           // Never repaints
      ],
    );
  }
}
```

### Opacity Optimization

```dart
// ❌ BAD: Opacity widget causes repaint
Opacity(
  opacity: isVisible ? 1.0 : 0.0,
  child: ExpensiveWidget(),  // Repaints on opacity change
)

// ✅ GOOD: AnimatedOpacity with RepaintBoundary
RepaintBoundary(
  child: AnimatedOpacity(
    opacity: isVisible ? 1.0 : 0.0,
    duration: Duration(milliseconds: 300),
    child: ExpensiveWidget(),
  ),
)

// ✅ BETTER: Visibility instead of Opacity
Visibility(
  visible: isVisible,
  maintainState: true,
  maintainAnimation: true,
  child: ExpensiveWidget(),
)
```

---

## Concept 5: ListView Optimization 🟡 SHOULD-KNOW

### ListView.builder

```dart
// ListView.builder renders only visible items
// ListView with children renders ALL items

// ❌ BAD: Renders all 1000 items immediately
ListView(
  children: List.generate(1000, (i) => ListTile(title: Text('$i'))),
)
// Memory: 1000 widgets
// Build time: O(n)

// ✅ GOOD: Only renders visible items (~10-20)
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return ListTile(title: Text('$index'));
  },
)
// Memory: ~10-20 widgets
// Build time: O(1) per frame
```

### Adding Keys

```dart
// Keys help Flutter track widgets across rebuilds

// ❌ Without key: Widgets identified by index
// When item removed from middle, all following items rebuild

// ✅ With ValueKey: Widgets identified by unique value
ListView.builder(
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(items[index].id),  // ✅ Unique by ID
      title: Text(items[index].title),
    );
  },
)

// Key types:
// ValueKey<T>: Use value (int, String, etc.)
// ObjectKey<T>: Use object identity
// GlobalKey<T>: Globally unique, used for accessing state
```

### itemExtent

```dart
// itemExtent improves ListView performance
// Use when all items have same height

// ✅ With itemExtent: Scroll performance improved
ListView.builder(
  itemCount: 1000,
  itemExtent: 72.0,  // Fixed height
  itemBuilder: (context, index) => ListTile(title: Text('$index')),
)

// Why itemExtent helps:
// 1. Scroll position calculation is O(1) instead of O(n)
// 2. No need to measure items
// 3. JumpToIndex is supported
```

### SeparatedBuilder

```dart
// ListView.separated for dividers
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => const Divider(),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
)
```

---

## Concept 6: Keys Deep Dive 🟡 SHOULD-KNOW

### Key Types

```dart
// 1. ValueKey - Compare by value
ValueKey<int>(1)
ValueKey<String>('item-1')

// 2. ObjectKey - Compare by object identity
ObjectKey(myObject)

// 3. GlobalKey - Globally unique, can access state
final _formKey = GlobalKey<FormState>();
final _pageKey = GlobalKey<PageState>();

// When to use:
// - List reordering: ValueKey<T>(item.id)
// - Form state: GlobalKey<FormState>
// - Stateful widget identity: ValueKey or ObjectKey
```

### GlobalKey Usage

```dart
// GlobalKey allows accessing State from anywhere
class MyFormPage extends StatefulWidget {
  const MyFormPage({super.key});

  @override
  State<MyFormPage> createState() => _MyFormPageState();
}

class _MyFormPageState extends State<MyFormPage> {
  final _formKey = GlobalKey<FormState>();

  String? validate() {
    return _formKey.currentState?.validate();
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,  // Register with GlobalKey
      child: TextFormField(validator: (v) => v?.isEmpty ? 'Required' : null),
    );
  }
}

// Usage elsewhere:
class SomeOtherWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final formState = _formKey.currentState;
    return ElevatedButton(
      onPressed: () => formState?.validate(),
      child: Text('Validate'),
    );
  }
}
```

---

## Concept 7: Isolate & Compute 🟡 SHOULD-KNOW

### When to Use Isolates

```dart
// Isolate = Separate Dart VM instance
// Use for:
// 1. Heavy computation (parsing, encryption, etc.)
// 2. Large data processing
// 3. Image manipulation
// 4. Any CPU-bound work

// Don't use for:
// 1. I/O operations (already async)
// 2. Small computations
// 3. Operations requiring frequent UI updates
```

### compute() Function

```dart
import 'package:flutter/foundation.dart';

// compute() is simplest way to run in isolate
Future<int> computeFactorial(int n) async {
  return compute(_factorialIsolate, n);
}

int _factorialIsolate(int n) {
  if (n <= 1) return 1;
  return n * _factorialIsolate(n - 1);
}

// compute() with complex data:
Future<List<int>> computePrimes(int max) async {
  return compute(_findPrimesIsolate, max);
}

List<int> _findPrimesIsolate(int max) {
  return List.generate(max, (i) => i + 2)
      .where((n) => _isPrime(n))
      .toList();
}

bool _isPrime(int n) {
  if (n < 2) return false;
  for (int i = 2; i <= n / 2; i++) {
    if (n % i == 0) return false;
  }
  return true;
}
```

### Isolate with SendPort

```dart
import 'dart:isolate';

// For complex isolate communication
class IsolateWorker {
  late Isolate _isolate;
  late SendPort _sendPort;
  late ReceivePort _receivePort;

  Future<void> init() async {
    _receivePort = ReceivePort();
    _isolate = await Isolate.spawn(_isolateEntry, _receivePort.sendPort);
    _sendPort = await _receivePort.first as SendPort;
  }

  Future<R> compute<R, P>(R Function(P) function, P param) async {
    final responsePort = ReceivePort();
    _sendPort.send([function, param, responsePort.sendPort]);
    return await responsePort.first as R;
  }

  void dispose() {
    _isolate.kill();
    _receivePort.close();
  }
}

void _isolateEntry(SendPort sendPort) {
  final receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  
  receivePort.listen((message) {
    final function = message[0] as Function;
    final param = message[1] as dynamic;
    final responsePort = message[2] as SendPort;
    
    final result = Function.apply(function, [param]);
    responsePort.send(result);
  });
}
```

---

## Concept 8: Image & Network Optimization 🟢 AI-GENERATE

### Image Caching

```dart
// CachedNetworkImage package
CachedNetworkImage(
  imageUrl: 'https://example.com/image.jpg',
  imageBuilder: (context, imageProvider) => Container(
    decoration: BoxDecoration(
      image: DecorationImage(image: imageProvider),
    ),
  ),
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
  fadeOutDuration: const Duration(milliseconds: 300),
  fadeInDuration: const Duration(milliseconds: 300),
  memCacheWidth: 800,  // Downscale for memory
  memCacheHeight: 600,
  maxWidthDiskCache: 1024,
  maxHeightDiskCache: 1024,
)
```

### Image Sizing

```dart
// Size images appropriately
Image.network(
  imageUrl,
  width: 200,   // Don't load 4K image for 200px display
  height: 200,
  fit: BoxFit.cover,
)
```

---

## Concept 9: BuildContext Misuse 🟢 AI-GENERATE

### Async Gap Problem

```dart
// ❌ BAD: Using BuildContext after async gap
class BadWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        await fetchData();  // Async gap
        // ❌ context may be unmounted
        if (context.mounted) {
          Navigator.of(context).pop();  // May crash
        }
      },
      child: Text('Submit'),
    );
  }
}

// ✅ GOOD: Check mounted before use
class GoodWidget extends StatefulWidget {
  @override
  State<GoodWidget> createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        await fetchData();
        if (mounted) {  // ✅ Check mounted state
          Navigator.of(context).pop();
        }
      },
      child: Text('Submit'),
    );
  }
}
```

### context.mounted

```dart
// Since Flutter 3.7+
// BuildContext has mounted property

// Pattern for StatefulWidget:
if (mounted) {
  setState(() {
    _loading = false;
  });
}

// For async callbacks:
onPressed: () async {
  await operation();
  if (!mounted) return;
  // Safe to use state here
  setState(() {});
}

// For async callback with context:
onPressed: () async {
  await operation();
  if (!mounted) return;
  Navigator.of(context).pop();
}
```

---

## Concept 10: Performance Patterns 🟢 AI-GENERATE

### Lazy Loading

```dart
// Lazy build for large data
class LazyWidget extends StatefulWidget {
  @override
  State<LazyWidget> createState() => _LazyWidgetState();
}

class _LazyWidgetState extends State<LazyWidget> {
  bool _expanded = false;
  Widget? _heavyContent;

  void _loadHeavyContent() {
    setState(() {
      _expanded = true;
      _heavyContent = const _HeavyWidget();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextButton(
          onPressed: _loadHeavyContent,
          child: Text(_expanded ? 'Hide' : 'Show'),
        ),
        if (_expanded && _heavyContent != null)
          _heavyContent!,
      ],
    );
  }
}
```

### Pagination

```dart
// Pagination for large lists
class PaginatedList extends StatefulWidget {
  @override
  State<PaginatedList> createState() => _PaginatedListState();
}

class _PaginatedListState extends State<PaginatedList> {
  final _scrollController = ScrollController();
  List<Item> _items = [];
  bool _hasMore = true;
  bool _loading = false;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      _loadMore();
    }
  }

  Future<void> _loadMore() async {
    if (_loading || !_hasMore) return;
    setState(() => _loading = true);
    
    final newItems = await api.fetchItems(page: _items.length ~/ 20);
    setState(() {
      _items.addAll(newItems);
      _hasMore = newItems.isNotEmpty;
      _loading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: _items.length + (_loading ? 1 : 0),
      itemBuilder: (context, index) {
        if (index >= _items.length) {
          return const Center(child: CircularProgressIndicator());
        }
        return ListTile(title: Text(_items[index].title));
      },
    );
  }
}
```

---

## Practice Tasks

### Task 1: Profile & Identify Issues

```bash
# 1. Run app with DevTools
flutter run --observe

# 2. Open DevTools
# 3. Performance tab → Record
# 4. Interact with app
# 5. Stop and analyze timeline
# 6. Identify:
#    - Slow frames (red bars)
#    - Expensive widget rebuilds
#    - Memory allocations
```

### Task 2: Apply Optimizations

```dart
// Checklist:
// [ ] Use const constructors where possible
// [ ] Add keys to list items
// [ ] Use RepaintBoundary for animations
// [ ] Implement proper dispose()
// [ ] Use ListView.builder for large lists
// [ ] Add itemExtent for fixed-height lists
// [ ] Use compute() for heavy computation
// [ ] Check mounted before async context use
```

---

## Summary

| Concept | Key Takeaway |
|---------|--------------|
| DevTools | Timeline + CPU Profiler + Memory tabs |
| Rebuild Optimization | Const constructors + selective rebuilds |
| Memory Leaks | dispose() + cancel subscriptions |
| RepaintBoundary | Isolate expensive repaints |
| ListView | builder + keys + itemExtent |
| Keys | ValueKey/ObjectKey/GlobalKey by use case |
| Isolates | Heavy computation offload |
| Images | Caching + appropriate sizing |
| BuildContext | Check mounted after async |
| Performance Patterns | Lazy loading + pagination |

<!-- AI_VERIFY: generation-complete -->
