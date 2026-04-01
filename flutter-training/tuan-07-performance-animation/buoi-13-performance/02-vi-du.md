# Buổi 13: Performance Optimization — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## VD1: const Constructor Optimization — Before/After 🔴

### Mục đích
So sánh rebuild count giữa widget CÓ và KHÔNG dùng `const`.

> **Liên quan tới:** [3. Rebuild Optimization 🔴](01-ly-thuyet.md#3-rebuild-optimization)

### Code TRƯỚC khi optimize (không const):

```dart
import 'package:flutter/material.dart';

class RebuildDemoPage extends StatefulWidget {
  const RebuildDemoPage({super.key});

  @override
  State<RebuildDemoPage> createState() => _RebuildDemoPageState();
}

class _RebuildDemoPageState extends State<RebuildDemoPage> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🔄 RebuildDemoPage build()');
    return Scaffold(
      appBar: AppBar(title: Text('Rebuild Demo')), // ❌ Không const
      body: Padding(
        padding: EdgeInsets.all(16), // ❌ Không const
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ❌ Tất cả rebuild mỗi lần nhấn nút!
            HeaderWidget(),
            SizedBox(height: 16), // ❌ Không const
            DescriptionWidget(),
            SizedBox(height: 16), // ❌ Không const
            IconRow(),
            SizedBox(height: 32), // ❌ Không const
            Text(
              'Counter: $_counter',
              style: TextStyle(fontSize: 24), // ❌ Không const
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _counter++),
        child: Icon(Icons.add), // ❌ Không const
      ),
    );
  }
}

// Widget con — không const
class HeaderWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('  🔄 HeaderWidget build()');
    return Text(
      'Performance Demo',
      style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
    );
  }
}

class DescriptionWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('  🔄 DescriptionWidget build()');
    return Text(
      'Nhấn nút + và quan sát console để thấy widget nào rebuild.',
      style: TextStyle(fontSize: 16, color: Colors.grey),
    );
  }
}

class IconRow extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('  🔄 IconRow build()');
    return Row(
      children: [
        Icon(Icons.star, color: Colors.amber, size: 32),
        Icon(Icons.star, color: Colors.amber, size: 32),
        Icon(Icons.star, color: Colors.amber, size: 32),
      ],
    );
  }
}
```

**Console output khi nhấn nút 3 lần:**
```
🔄 RebuildDemoPage build()
  🔄 HeaderWidget build()        ← Không cần rebuild!
  🔄 DescriptionWidget build()   ← Không cần rebuild!
  🔄 IconRow build()             ← Không cần rebuild!
🔄 RebuildDemoPage build()
  🔄 HeaderWidget build()
  🔄 DescriptionWidget build()
  🔄 IconRow build()
🔄 RebuildDemoPage build()
  🔄 HeaderWidget build()
  🔄 DescriptionWidget build()
  🔄 IconRow build()

→ Tổng: 12 lần build không cần thiết cho 3 lần nhấn nút!
```

### Code SAU khi optimize (dùng const):

```dart
import 'package:flutter/material.dart';

class RebuildDemoOptimizedPage extends StatefulWidget {
  const RebuildDemoOptimizedPage({super.key});

  @override
  State<RebuildDemoOptimizedPage> createState() =>
      _RebuildDemoOptimizedPageState();
}

class _RebuildDemoOptimizedPageState
    extends State<RebuildDemoOptimizedPage> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🔄 OptimizedPage build()');
    return Scaffold(
      appBar: AppBar(title: const Text('Rebuild Demo - Optimized')), // ✅
      body: Padding(
        padding: const EdgeInsets.all(16), // ✅
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const HeaderWidget(),            // ✅ KHÔNG rebuild
            const SizedBox(height: 16),      // ✅
            const DescriptionWidget(),       // ✅ KHÔNG rebuild
            const SizedBox(height: 16),      // ✅
            const IconRow(),                 // ✅ KHÔNG rebuild
            const SizedBox(height: 32),      // ✅
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24), // ✅
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _counter++),
        child: const Icon(Icons.add), // ✅
      ),
    );
  }
}

// Widget con — có const constructor
class HeaderWidget extends StatelessWidget {
  const HeaderWidget({super.key}); // ✅ const constructor

  @override
  Widget build(BuildContext context) {
    print('  🔄 HeaderWidget build()');
    return const Text(
      'Performance Demo',
      style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
    );
  }
}

class DescriptionWidget extends StatelessWidget {
  const DescriptionWidget({super.key}); // ✅ const constructor

  @override
  Widget build(BuildContext context) {
    print('  🔄 DescriptionWidget build()');
    return const Text(
      'Nhấn nút + và quan sát console để thấy widget nào rebuild.',
      style: TextStyle(fontSize: 16, color: Colors.grey),
    );
  }
}

class IconRow extends StatelessWidget {
  const IconRow({super.key}); // ✅ const constructor

  @override
  Widget build(BuildContext context) {
    print('  🔄 IconRow build()');
    return const Row(
      children: [
        Icon(Icons.star, color: Colors.amber, size: 32),
        Icon(Icons.star, color: Colors.amber, size: 32),
        Icon(Icons.star, color: Colors.amber, size: 32),
      ],
    );
  }
}
```

**Console output khi nhấn nút 3 lần:**
```
🔄 OptimizedPage build()
  (HeaderWidget, DescriptionWidget, IconRow: KHÔNG rebuild!)
🔄 OptimizedPage build()
🔄 OptimizedPage build()

→ Tổng: 0 lần build không cần thiết! Giảm 12 → 0 rebuilds.
```

### Kết quả so sánh

| Metric | Trước | Sau | Cải thiện |
|--------|-------|-----|-----------|
| Rebuild count (3 taps) | 12 child builds | 0 child builds | -100% |
| Widget instances tạo mới | Mỗi lần build | Reuse compile-time instance | Memory giảm |

- 🔗 **FE tương đương:** `const MyWidget()` ≈ `React.memo(MyComponent)` — cả hai skip re-render khi input không đổi. Flutter `const` mạnh hơn vì là compile-time guarantee.

---

## VD2: ListView.builder vs ListView — 10,000 Items 🔴

### Mục đích
So sánh performance khi render danh sách lớn.

> **Liên quan tới:** [3. Rebuild Optimization 🔴](01-ly-thuyet.md#3-rebuild-optimization)

### ListView thường (CHẬM):

```dart
import 'package:flutter/material.dart';

class ListViewNormalPage extends StatelessWidget {
  const ListViewNormalPage({super.key});

  @override
  Widget build(BuildContext context) {
    final stopwatch = Stopwatch()..start();

    final widget = Scaffold(
      appBar: AppBar(title: const Text('ListView - 10,000 items')),
      body: ListView(
        // ❌ Tạo TẤT CẢ 10,000 items ngay lập tức!
        children: List.generate(10000, (index) {
          return ContactTile(
            name: 'Contact $index',
            email: 'contact$index@example.com',
            avatarColor: Colors.primaries[index % Colors.primaries.length],
          );
        }),
      ),
    );

    stopwatch.stop();
    print('⏱️ ListView build time: ${stopwatch.elapsedMilliseconds}ms');
    // → Thường 200-500ms+ trên device thật!

    return widget;
  }
}

class ContactTile extends StatelessWidget {
  final String name;
  final String email;
  final Color avatarColor;

  const ContactTile({
    super.key,
    required this.name,
    required this.email,
    required this.avatarColor,
  });

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: CircleAvatar(
        backgroundColor: avatarColor,
        child: Text(
          name[0],
          style: const TextStyle(color: Colors.white),
        ),
      ),
      title: Text(name),
      subtitle: Text(email),
      trailing: const Icon(Icons.chevron_right),
    );
  }
}
```

### ListView.builder (NHANH):

```dart
import 'package:flutter/material.dart';

class ListViewBuilderPage extends StatelessWidget {
  const ListViewBuilderPage({super.key});

  @override
  Widget build(BuildContext context) {
    final stopwatch = Stopwatch()..start();

    final widget = Scaffold(
      appBar: AppBar(title: const Text('ListView.builder - 10,000 items')),
      body: ListView.builder(
        itemCount: 10000,
        // ✅ Chỉ build items visible + buffer
        itemBuilder: (context, index) {
          return ContactTile(
            name: 'Contact $index',
            email: 'contact$index@example.com',
            avatarColor: Colors.primaries[index % Colors.primaries.length],
          );
        },
      ),
    );

    stopwatch.stop();
    print('⏱️ ListView.builder build time: ${stopwatch.elapsedMilliseconds}ms');
    // → Thường <5ms!

    return widget;
  }
}
```

### App so sánh cả hai:

```dart
import 'package:flutter/material.dart';

class ListComparisonPage extends StatelessWidget {
  const ListComparisonPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ListView Comparison')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              '10,000 Contacts',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                final sw = Stopwatch()..start();
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => const ListViewNormalPage(),
                  ),
                );
                sw.stop();
                debugPrint('Navigate + build: ${sw.elapsedMilliseconds}ms');
              },
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red.shade100,
                padding: const EdgeInsets.symmetric(
                  horizontal: 32,
                  vertical: 16,
                ),
              ),
              child: const Text('❌ ListView (chậm)'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                final sw = Stopwatch()..start();
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => const ListViewBuilderPage(),
                  ),
                );
                sw.stop();
                debugPrint('Navigate + build: ${sw.elapsedMilliseconds}ms');
              },
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.green.shade100,
                padding: const EdgeInsets.symmetric(
                  horizontal: 32,
                  vertical: 16,
                ),
              ),
              child: const Text('✅ ListView.builder (nhanh)'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Kết quả so sánh

| Metric | ListView | ListView.builder |
|--------|----------|-----------------|
| Initial build time | 200-500ms | <5ms |
| Memory usage | ~50-100MB (10K widgets) | ~2-5MB (~20 widgets) |
| Scroll performance | Mượt (đã build hết) | Mượt (build on-demand) |
| First frame jank | CÓ (build lâu) | KHÔNG |

---

## VD3: RepaintBoundary — Cách ly Animation 🟡

### Mục đích
Dùng `RepaintBoundary` để tránh animation repaint toàn bộ screen.

> **Liên quan tới:** [3. Rebuild Optimization 🔴](01-ly-thuyet.md#3-rebuild-optimization)

### Không có RepaintBoundary (CHẬM):

```dart
import 'package:flutter/material.dart';

class WithoutRepaintBoundaryPage extends StatefulWidget {
  const WithoutRepaintBoundaryPage({super.key});

  @override
  State<WithoutRepaintBoundaryPage> createState() =>
      _WithoutRepaintBoundaryPageState();
}

class _WithoutRepaintBoundaryPageState
    extends State<WithoutRepaintBoundaryPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Without RepaintBoundary')),
      body: Stack(
        children: [
          // ❌ Widget phức tạp bị repaint mỗi frame!
          const ExpensivePaintWidget(label: 'HEADER — Expensive paint'),
          // Animation chạy 60fps → trigger repaint toàn bộ Stack
          Positioned(
            bottom: 50,
            left: 0,
            right: 0,
            child: AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Center(
                  child: Transform.scale(
                    scale: 0.5 + (_controller.value * 0.5),
                    child: Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.blue,
                        borderRadius: BorderRadius.circular(16),
                      ),
                      child: const Center(
                        child: Icon(Icons.animation, color: Colors.white),
                      ),
                    ),
                  ),
                );
              },
            ),
          ),
          const Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            child: ExpensivePaintWidget(label: 'FOOTER — Expensive paint'),
          ),
        ],
      ),
    );
  }
}

// Widget giả lập paint phức tạp
class ExpensivePaintWidget extends StatelessWidget {
  final String label;

  const ExpensivePaintWidget({super.key, required this.label});

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: const Size(double.infinity, 200),
      painter: _ExpensivePainter(label: label),
    );
  }
}

class _ExpensivePainter extends CustomPainter {
  final String label;
  int _paintCount = 0;

  _ExpensivePainter({required this.label});

  @override
  void paint(Canvas canvas, Size size) {
    _paintCount++;
    debugPrint('🎨 PAINT $label (#$_paintCount)');

    // Simulate expensive painting: vẽ 500 circles
    final paint = Paint()..style = PaintingStyle.fill;
    for (int i = 0; i < 500; i++) {
      paint.color = Color.fromARGB(
        50,
        (i * 7) % 255,
        (i * 13) % 255,
        (i * 17) % 255,
      );
      canvas.drawCircle(
        Offset(
          (i * 37 % size.width.toInt()).toDouble(),
          (i * 23 % size.height.toInt()).toDouble(),
        ),
        3,
        paint,
      );
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

### Có RepaintBoundary (NHANH):

```dart
import 'package:flutter/material.dart';

class WithRepaintBoundaryPage extends StatefulWidget {
  const WithRepaintBoundaryPage({super.key});

  @override
  State<WithRepaintBoundaryPage> createState() =>
      _WithRepaintBoundaryPageState();
}

class _WithRepaintBoundaryPageState extends State<WithRepaintBoundaryPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('With RepaintBoundary')),
      body: Stack(
        children: [
          // ✅ RepaintBoundary cách ly phần static
          const RepaintBoundary(
            child: ExpensivePaintWidget(label: 'HEADER — Protected'),
          ),
          // ✅ RepaintBoundary cho animation — layer riêng
          Positioned(
            bottom: 50,
            left: 0,
            right: 0,
            child: RepaintBoundary(
              child: AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  return Center(
                    child: Transform.scale(
                      scale: 0.5 + (_controller.value * 0.5),
                      child: child,
                    ),
                  );
                },
                child: Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: const Center(
                    child: Icon(Icons.animation, color: Colors.white),
                  ),
                ),
              ),
            ),
          ),
          const Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            child: RepaintBoundary(
              child: ExpensivePaintWidget(label: 'FOOTER — Protected'),
            ),
          ),
        ],
      ),
    );
  }
}
```

### Kết quả so sánh

```
Không có RepaintBoundary:
Console output (1 giây animation):
🎨 PAINT HEADER — Expensive paint (#1)
🎨 PAINT FOOTER — Expensive paint (#1)
🎨 PAINT HEADER — Expensive paint (#2)   ← Repaint lại!
🎨 PAINT FOOTER — Expensive paint (#2)   ← Repaint lại!
... (60 lần/giây!)

Có RepaintBoundary:
Console output (1 giây animation):
🎨 PAINT HEADER — Protected (#1)     ← Paint 1 lần duy nhất
🎨 PAINT FOOTER — Protected (#1)     ← Paint 1 lần duy nhất
(Animation chạy 60fps nhưng Header/Footer KHÔNG bị repaint)
```

| Metric | Không RepaintBoundary | Có RepaintBoundary |
|--------|----------------------|-------------------|
| Header paint count (1s) | ~60 lần | 1 lần |
| Footer paint count (1s) | ~60 lần | 1 lần |
| GPU utilization | Cao | Thấp |
| Jank risk | Cao | Thấp |

---

## VD4: DevTools Profiling — Hướng dẫn từng bước 🔴

### Mục đích
Hướng dẫn step-by-step cách dùng DevTools để identify và fix jank.

> **Liên quan tới:** [4. Flutter DevTools 🔴](01-ly-thuyet.md#4-flutter-devtools)

### Bước 1: Tạo app mẫu có jank

```dart
import 'package:flutter/material.dart';
import 'dart:math';

class JankyDemoApp extends StatelessWidget {
  const JankyDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // showPerformanceOverlay: true, // Bật nếu muốn overlay
      home: const JankyListPage(),
    );
  }
}

class JankyListPage extends StatefulWidget {
  const JankyListPage({super.key});

  @override
  State<JankyListPage> createState() => _JankyListPageState();
}

class _JankyListPageState extends State<JankyListPage> {
  final _items = List.generate(1000, (i) => 'Item $i');
  String _searchQuery = '';

  @override
  Widget build(BuildContext context) {
    // ❌ BUG 1: Filter trong build() — chạy mỗi frame!
    final filteredItems = _items.where((item) {
      // Giả lập expensive filter
      _simulateExpensiveWork();
      return item.toLowerCase().contains(_searchQuery.toLowerCase());
    }).toList();

    return Scaffold(
      appBar: AppBar(title: const Text('Janky Demo')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              onChanged: (value) {
                setState(() {
                  _searchQuery = value;
                });
              },
              decoration: const InputDecoration(
                hintText: 'Search...',
                prefixIcon: Icon(Icons.search),
              ),
            ),
          ),
          Expanded(
            // ❌ BUG 2: Không dùng ListView.builder
            child: ListView(
              children: filteredItems.map((item) {
                return _buildExpensiveItem(item);
              }).toList(),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildExpensiveItem(String item) {
    // ❌ BUG 3: Heavy computation trong item build
    final hash = item.hashCode;
    final color = Color.fromARGB(
      255,
      (sin(hash) * 128 + 128).toInt(),
      (cos(hash) * 128 + 128).toInt(),
      (sin(hash * 2) * 128 + 128).toInt(),
    );

    return Card(
      color: color.withValues(alpha: 0.3),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: color,
          child: Text(item.substring(0, 1)),
        ),
        title: Text(item),
        subtitle: Text('Hash: ${hash.toRadixString(16)}'),
      ),
    );
  }

  void _simulateExpensiveWork() {
    // Giả lập tính toán nặng
    var sum = 0.0;
    for (var i = 0; i < 1000; i++) {
      sum += sin(i.toDouble()) * cos(i.toDouble());
    }
  }
}
```

### Bước 2: Chạy app ở Profile mode

```bash
# Terminal
flutter run --profile

# Khi app chạy, copy DevTools URL từ output:
# An Observatory debugger and target is available at: http://127.0.0.1:XXXX
# The Flutter DevTools debugger and profiler is available at: http://127.0.0.1:YYYY
```

### Bước 3: Mở DevTools và Profile

```
1. Mở DevTools URL trong browser

2. Tab "Performance":
   ┌──────────────────────────────────────┐
   │ [Record] [Stop] [Clear]              │
   │                                       │
   │ Click "Record" → thao tác với app     │
   │ → gõ text vào search → scroll list    │
   │ → Click "Stop"                        │
   └──────────────────────────────────────┘

3. Xem Frame Chart:
   ┌──────────────────────────────────────┐
   │ ██ ██ ██ ████████ ██ ████████ ██    │
   │ ── ── ── ──────── ── ──────── ──    │
   │            ↑             ↑           │
   │      Jank frame!    Jank frame!      │
   │                                       │
   │ Click vào frame đỏ để xem chi tiết   │
   └──────────────────────────────────────┘

4. Xem Timeline cho frame bị jank:
   ┌──────────────────────────────────────┐
   │ Build: ██████████████████ 45ms       │
   │   └─ _JankyListPageState.build()     │
   │      └─ _simulateExpensiveWork()     │
   │                                       │
   │ Layout: ██ 3ms                        │
   │ Paint:  █ 2ms                         │
   │                                       │
   │ → Build phase quá lâu!               │
   │ → _simulateExpensiveWork là bottleneck│
   └──────────────────────────────────────┘
```

### Bước 4: Xem Widget Inspector

```
5. Tab "Inspector":
   ┌──────────────────────────────────────┐
   │ Toggle "Track Widget Rebuilds"        │
   │                                       │
   │ Widget Tree:                          │
   │ ├─ JankyListPage                      │
   │ │  ├─ TextField          ⟳ 5         │
   │ │  └─ ListView           ⟳ 5         │
   │ │     ├─ Card            ⟳ 5 × 1000!│
   │ │     ├─ Card            ⟳ 5         │
   │ │     └─ ...                          │
   │                                       │
   │ → 5000 rebuilds! Quá nhiều!          │
   └──────────────────────────────────────┘
```

### Bước 5: Fix từng bottleneck

```dart
import 'package:flutter/material.dart';
import 'dart:math';

class OptimizedListPage extends StatefulWidget {
  const OptimizedListPage({super.key});

  @override
  State<OptimizedListPage> createState() => _OptimizedListPageState();
}

class _OptimizedListPageState extends State<OptimizedListPage> {
  final _allItems = List.generate(1000, (i) => 'Item $i');
  List<String> _filteredItems = [];
  String _searchQuery = '';

  @override
  void initState() {
    super.initState();
    _filteredItems = _allItems; // ✅ FIX 1: Cache filtered list
  }

  void _onSearchChanged(String query) {
    setState(() {
      _searchQuery = query;
      // ✅ FIX 1: Filter một lần khi query đổi, không mỗi frame
      _filteredItems = _allItems
          .where((item) => item.toLowerCase().contains(query.toLowerCase()))
          .toList();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Optimized Demo')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              onChanged: _onSearchChanged,
              decoration: const InputDecoration(
                hintText: 'Search...',
                prefixIcon: Icon(Icons.search),
              ),
            ),
          ),
          Expanded(
            // ✅ FIX 2: Dùng ListView.builder
            child: ListView.builder(
              itemCount: _filteredItems.length,
              itemBuilder: (context, index) {
                return _OptimizedItem(item: _filteredItems[index]);
              },
            ),
          ),
        ],
      ),
    );
  }
}

// ✅ FIX 3: Tách thành widget riêng, tính toán ít hơn
class _OptimizedItem extends StatelessWidget {
  final String item;

  const _OptimizedItem({required this.item});

  @override
  Widget build(BuildContext context) {
    final colorIndex = item.hashCode % Colors.primaries.length;
    final color = Colors.primaries[colorIndex.abs()];

    return Card(
      color: color.withValues(alpha: 0.3),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: color,
          child: Text(item.substring(0, 1)),
        ),
        title: Text(item),
      ),
    );
  }
}
```

### Kết quả sau khi fix

| Metric | Trước | Sau |
|--------|-------|-----|
| Build time per frame | 45ms+ (jank!) | <5ms |
| Widgets built per search | 1000+ | ~15 (visible) |
| Filter computation | Mỗi build() | 1 lần per query |
| Jank frames | Nhiều | 0 |

- 🔗 **FE tương đương:** Flutter Performance Overlay ≈ Chrome DevTools FPS meter — nhưng Flutter hiển thị 2 graphs riêng cho UI thread và Raster thread.

---

## VD5: Isolate compute() — Parse Large JSON 🟡

### Mục đích
Dùng `compute()` để parse JSON lớn trên background isolate, giữ UI mượt.

> **Liên quan tới:** [6. Isolates 🟡](01-ly-thuyet.md#6-isolates)

### Không dùng Isolate (BLOCK UI):

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class WithoutIsolatePage extends StatefulWidget {
  const WithoutIsolatePage({super.key});

  @override
  State<WithoutIsolatePage> createState() => _WithoutIsolatePageState();
}

class _WithoutIsolatePageState extends State<WithoutIsolatePage>
    with SingleTickerProviderStateMixin {
  List<User> _users = [];
  bool _loading = false;
  late final AnimationController _spinController;

  @override
  void initState() {
    super.initState();
    _spinController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat(); // Spinner quay liên tục để test jank
  }

  @override
  void dispose() {
    _spinController.dispose();
    super.dispose();
  }

  Future<void> _loadData() async {
    setState(() => _loading = true);

    // Giả lập API trả về JSON lớn
    final jsonString = _generateLargeJson(50000);

    // ❌ Parse trên main thread — spinner sẽ ĐỨNG!
    final stopwatch = Stopwatch()..start();
    final List<dynamic> decoded = jsonDecode(jsonString);
    final users = decoded.map((json) => User.fromJson(json)).toList();
    stopwatch.stop();

    debugPrint('⏱️ Parse on main thread: ${stopwatch.elapsedMilliseconds}ms');

    setState(() {
      _users = users;
      _loading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Without Isolate')),
      body: Column(
        children: [
          // Spinner indicator — sẽ đứng khi parse JSON
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                RotationTransition(
                  turns: _spinController,
                  child: const Icon(Icons.sync, size: 40),
                ),
                const SizedBox(width: 16),
                const Text('Spinner (đứng = jank!)'),
              ],
            ),
          ),
          ElevatedButton(
            onPressed: _loading ? null : _loadData,
            child: Text(_loading ? 'Đang load...' : 'Load 50,000 users'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: _users.length,
              itemBuilder: (context, index) {
                final user = _users[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text(user.email),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### Dùng compute() (UI mượt):

```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

// ✅ Top-level function (required cho compute)
List<User> _parseUsersInBackground(String jsonString) {
  final List<dynamic> decoded = jsonDecode(jsonString);
  return decoded.map((json) => User.fromJson(json)).toList();
}

class WithIsolatePage extends StatefulWidget {
  const WithIsolatePage({super.key});

  @override
  State<WithIsolatePage> createState() => _WithIsolatePageState();
}

class _WithIsolatePageState extends State<WithIsolatePage>
    with SingleTickerProviderStateMixin {
  List<User> _users = [];
  bool _loading = false;
  late final AnimationController _spinController;

  @override
  void initState() {
    super.initState();
    _spinController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat();
  }

  @override
  void dispose() {
    _spinController.dispose();
    super.dispose();
  }

  Future<void> _loadData() async {
    setState(() => _loading = true);

    final jsonString = _generateLargeJson(50000);

    // ✅ Parse trên isolate riêng — spinner VẪN quay mượt!
    final stopwatch = Stopwatch()..start();
    final users = await compute(_parseUsersInBackground, jsonString);
    stopwatch.stop();

    debugPrint('⏱️ Parse on isolate: ${stopwatch.elapsedMilliseconds}ms');
    debugPrint('   (UI vẫn mượt trong lúc parse!)');

    setState(() {
      _users = users;
      _loading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('With Isolate (compute)')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                RotationTransition(
                  turns: _spinController,
                  child: const Icon(Icons.sync, size: 40, color: Colors.green),
                ),
                const SizedBox(width: 16),
                const Text('Spinner (quay mượt = no jank!)'),
              ],
            ),
          ),
          ElevatedButton(
            onPressed: _loading ? null : _loadData,
            child: Text(_loading ? 'Đang load...' : 'Load 50,000 users'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: _users.length,
              itemBuilder: (context, index) {
                final user = _users[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text(user.email),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// Shared model & helper
class User {
  final String name;
  final String email;

  const User({required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(name: json['name'] as String, email: json['email'] as String);
  }
}

String _generateLargeJson(int count) {
  final buffer = StringBuffer('[');
  for (int i = 0; i < count; i++) {
    if (i > 0) buffer.write(',');
    buffer.write('{"name":"User $i","email":"user$i@example.com"}');
  }
  buffer.write(']');
  return buffer.toString();
}
```

### Kết quả so sánh

| Metric | Không Isolate | Có compute() |
|--------|-------------|-------------|
| UI jank khi parse | Spinner ĐỨNG 300-500ms | Spinner quay mượt |
| Parse time | ~400ms (block main thread) | ~400ms (background) |
| User experience | Thấy lag, app "chết" | Mượt mà, responsive |
| Dropped frames | 20-30 frames | 0 frames |

### Lưu ý khi dùng compute()

```dart
// ✅ DO: Top-level function
List<User> parseUsers(String json) { ... }

// ✅ DO: Static method
class UserParser {
  static List<User> parse(String json) { ... }
}

// ❌ DON'T: Instance method (closure captures `this`)
class _MyState extends State<MyWidget> {
  List<User> parseUsers(String json) { ... }  // ❌ Không dùng được!
}

// ❌ DON'T: Anonymous function captures scope
final result = await compute((json) {
  return _myInstanceMethod(json);  // ❌ Capture `this`!
}, jsonString);
```

---

## VD6: 🤖 AI Gen → Review — Performance Optimization 🟢

> **Mục đích:** Luyện workflow "AI audit performance → bạn verify bằng DevTools → fix"

> **Liên quan tới:** [3. Rebuild Optimization 🔴](01-ly-thuyet.md#3-rebuild-optimization)

### Bước 1: Prompt cho AI

```text
Phân tích performance code Flutter sau và identify optimization opportunities:
- StatefulWidget với setState() rebuild toàn bộ screen
- ListView(children: [...]) thay vì ListView.builder
- Image.network không cache, không set dimensions
Suggest fixes theo priority: P0 (jank fix) → P1 (rebuild reduce) → P2 (nice to have).
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **const suggestions** | AI suggest const cho đúng widgets? (chỉ immutable, không dynamic) |
| 2 | **ListView.builder** | Suggest + itemExtent + cacheExtent? |
| 3 | **RepaintBoundary** | Chỉ cho expensive operations? Không overuse? |
| 4 | **Priority** | P0=jank cho user, P1=rebuild, P2=nice-to-have? Hợp lý? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: RepaintBoundary everywhere
Widget build(BuildContext context) {
  return RepaintBoundary(  // KHÔNG CẦN cho simple Text!
    child: Text('Hello'),
  );
}

// ✅ FIX: Chỉ dùng cho expensive paint
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: CustomPaint(  // Complex canvas drawing = cần boundary
      painter: ChartPainter(data),
    ),
  );
}
```

```dart
// ❌ LỖI 2: AI suggest const nhưng widget có dynamic data
const Text(userName),  // userName là variable → KHÔNG THỂ const!

// ✅ FIX: const chỉ cho literal values
const Text('Static Label'),  // Literal → const OK
Text(userName),              // Dynamic → không const
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết RepaintBoundary chỉ dùng cho expensive paint (không phải mọi widget)
- ✅ Phân biệt khi nào const có ý nghĩa vs không thể dùng
- ✅ Verify bằng DevTools (không chỉ tin suggestion của AI)
