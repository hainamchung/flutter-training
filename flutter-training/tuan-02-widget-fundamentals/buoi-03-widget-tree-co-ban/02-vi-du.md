# Buổi 03: Widget Tree — Ví dụ minh hoạ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> Mỗi ví dụ là một Flutter app hoàn chỉnh. Tạo project bằng `flutter create`, thay code `main.dart`, chạy bằng `flutter run`.

---

## Cách chạy ví dụ

```bash
# Bước 1: Tạo project (chỉ cần 1 lần)
flutter create vidu_widget_tree
cd vidu_widget_tree

# Bước 2: Thay nội dung file lib/main.dart bằng code ví dụ

# Bước 3: Chạy
flutter run
```

---

## VD1: StatelessWidget — Greeting Card Widget 🟡

### Mục đích
- Hiểu cách tạo **StatelessWidget**
- Hiểu **widget composition** — ghép nhiều widget nhỏ thành widget lớn
- Dùng `const` constructor

> **Liên quan tới:** [2. StatelessWidget vs StatefulWidget 🔴](01-ly-thuyet.md#2-statelesswidget-vs-statefulwidget)

### Code — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD1 - StatelessWidget',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const GreetingScreen(),
    );
  }
}

/// Màn hình chính — cũng là StatelessWidget
class GreetingScreen extends StatelessWidget {
  const GreetingScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Greeting Cards'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: const Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            // Tái sử dụng widget với input khác nhau
            GreetingCard(
              name: 'Nguyễn Văn A',
              message: 'Chào mừng đến với Flutter!',
              avatarColor: Colors.blue,
            ),
            SizedBox(height: 12),
            GreetingCard(
              name: 'Trần Thị B',
              message: 'Widget là nền tảng của mọi thứ!',
              avatarColor: Colors.orange,
            ),
            SizedBox(height: 12),
            GreetingCard(
              name: 'Lê Văn C',
              message: 'StatelessWidget = no internal state',
              avatarColor: Colors.green,
            ),
          ],
        ),
      ),
    );
  }
}

/// GreetingCard — StatelessWidget vì chỉ hiển thị data, không thay đổi
class GreetingCard extends StatelessWidget {
  final String name;
  final String message;
  final Color avatarColor;

  // const constructor — cho phép Flutter tối ưu khi rebuild
  const GreetingCard({
    super.key,
    required this.name,
    required this.message,
    required this.avatarColor,
  });

  @override
  Widget build(BuildContext context) {
    // Widget composition: Card chứa ListTile chứa Text, CircleAvatar...
    return Card(
      elevation: 3,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: avatarColor,
          child: Text(
            name[0], // Lấy ký tự đầu làm avatar
            style: const TextStyle(
              color: Colors.white,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        title: Text(
          name,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        subtitle: Text(message),
        trailing: const Icon(Icons.waving_hand, color: Colors.amber),
      ),
    );
  }
}
```

### Giải thích

```
Widget Tree:
MyApp
 └── MaterialApp
      └── GreetingScreen (StatelessWidget)
           └── Scaffold
                ├── AppBar
                │    └── Text
                └── Padding
                     └── Column
                          ├── GreetingCard (name: "A")  ← StatelessWidget
                          │    └── Card → ListTile → ...
                          ├── SizedBox
                          ├── GreetingCard (name: "B")  ← Cùng type, khác input
                          │    └── Card → ListTile → ...
                          ├── SizedBox
                          └── GreetingCard (name: "C")
                               └── Card → ListTile → ...
```

**Điểm chính:**
- `GreetingCard` là StatelessWidget vì nó chỉ hiển thị data nhận từ constructor
- Cùng 1 class, 3 instance khác nhau (khác `name`, `message`, `avatarColor`)
- Dùng `const` constructor → Flutter có thể tối ưu rebuild
- 🔗 **FE tương đương:** Trong React: `function GreetingCard({ name }) { return <div>...</div> }` — cả hai đều là pure function of props, nhưng Flutter Widget immutable còn React component giữ reference.

---

## VD2: StatefulWidget — Counter App với setState 🔴

### Mục đích
- Hiểu cấu trúc **StatefulWidget** (2 class: Widget + State)
- Hiểu cách **setState()** trigger rebuild
- Thấy phạm vi rebuild

> **Liên quan tới:** [2. StatelessWidget vs StatefulWidget 🔴](01-ly-thuyet.md#2-statelesswidget-vs-statefulwidget)

### Code — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD2 - StatefulWidget',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const CounterScreen(),
    );
  }
}

class CounterScreen extends StatefulWidget {
  const CounterScreen({super.key});

  // Bước 1: StatefulWidget tạo State object
  @override
  State<CounterScreen> createState() => _CounterScreenState();
}

class _CounterScreenState extends State<CounterScreen> {
  // Bước 2: State object giữ mutable state
  int _count = 0;

  void _increment() {
    // Bước 3: setState() → trigger rebuild
    setState(() {
      _count++;
    });
    debugPrint('setState called — count = $_count');
    // Sau dòng này, build() sẽ được gọi lại
  }

  void _decrement() {
    setState(() {
      if (_count > 0) _count--;
    });
  }

  void _reset() {
    setState(() {
      _count = 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    // build() được gọi lại mỗi khi setState() chạy
    debugPrint('build() called — đang rebuild UI');

    return Scaffold(
      appBar: AppBar(
        title: const Text('Counter App'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Bạn đã nhấn nút:',
              style: TextStyle(fontSize: 18),
            ),
            const SizedBox(height: 8),
            Text(
              '$_count',
              style: Theme.of(context).textTheme.displayLarge?.copyWith(
                    color: _count > 10 ? Colors.red : Colors.black,
                    fontWeight: FontWeight.bold,
                  ),
            ),
            if (_count > 10)
              const Text(
                '🔥 Bạn nhấn nhiều quá!',
                style: TextStyle(color: Colors.red, fontSize: 16),
              ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Nút giảm
                FloatingActionButton(
                  heroTag: 'decrement',
                  onPressed: _decrement,
                  tooltip: 'Giảm',
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                // Nút reset
                FloatingActionButton(
                  heroTag: 'reset',
                  onPressed: _reset,
                  tooltip: 'Reset',
                  backgroundColor: Colors.grey,
                  child: const Icon(Icons.refresh),
                ),
                const SizedBox(width: 16),
                // Nút tăng
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: _increment,
                  tooltip: 'Tăng',
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### Giải thích

```
Khi user nhấn nút "Tăng":

1. _increment() chạy
2. setState(() { _count++; })
3. Flutter đánh dấu _CounterScreenState là "dirty"
4. build() được gọi lại
5. Widget Tree MỚI được tạo (với _count mới)
6. Element Tree so sánh cũ vs mới
7. Chỉ Text widget hiển thị _count được update trên màn hình
```

**Mở Debug Console** để thấy `build()` được gọi mỗi khi nhấn nút.

- 🔗 **FE tương đương:** Trong React: `const [count, setCount] = useState(0)` — nhưng React chỉ re-render component đó, Flutter rebuild toàn bộ `build()` method.

---

## VD3: Widget Lifecycle — In thứ tự lifecycle methods 🔴

### Mục đích
- **Thấy rõ** thứ tự gọi các lifecycle methods
- Hiểu khi nào mỗi method được gọi
- Hiểu tầm quan trọng của `dispose()`

> **Liên quan tới:** [6. Widget Lifecycle — Vòng đời StatefulWidget 🔴](01-ly-thuyet.md#6-widget-lifecycle--vòng-đời-statefulwidget)

### Code — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD3 - Lifecycle',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
        useMaterial3: true,
      ),
      home: const LifecycleDemo(),
    );
  }
}

/// Màn hình chứa toggle button để show/hide LifecycleWidget
class LifecycleDemo extends StatefulWidget {
  const LifecycleDemo({super.key});

  @override
  State<LifecycleDemo> createState() => _LifecycleDemoState();
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  bool _showWidget = true;
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Lifecycle Demo'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            const Text(
              '📋 Mở Debug Console để xem lifecycle logs',
              style: TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton.icon(
                  onPressed: () {
                    setState(() {
                      _showWidget = !_showWidget;
                    });
                  },
                  icon: Icon(_showWidget ? Icons.visibility_off : Icons.visibility),
                  label: Text(_showWidget ? 'Ẩn Widget' : 'Hiện Widget'),
                ),
                ElevatedButton.icon(
                  onPressed: () {
                    setState(() {
                      _counter++;
                    });
                  },
                  icon: const Icon(Icons.refresh),
                  label: Text('Update Parent (counter: $_counter)'),
                ),
              ],
            ),
            const SizedBox(height: 24),
            const Divider(),
            const SizedBox(height: 16),
            if (_showWidget)
              LifecycleWidget(counter: _counter),
            if (!_showWidget)
              const Text(
                'Widget đã bị ẩn — check console cho dispose()',
                style: TextStyle(
                  color: Colors.grey,
                  fontStyle: FontStyle.italic,
                ),
              ),
          ],
        ),
      ),
    );
  }
}

/// Widget minh hoạ lifecycle — in log tại mỗi lifecycle method
class LifecycleWidget extends StatefulWidget {
  final int counter;

  const LifecycleWidget({super.key, required this.counter});

  @override
  State<LifecycleWidget> createState() {
    debugPrint('🔵 [1] createState() — Tạo State object');
    return _LifecycleWidgetState();
  }
}

class _LifecycleWidgetState extends State<LifecycleWidget> {
  late int _internalCount;

  @override
  void initState() {
    super.initState();
    _internalCount = 0;
    debugPrint('🟢 [2] initState() — Khởi tạo (chạy 1 lần duy nhất)');
    debugPrint('    → Setup controllers, listeners, fetch data ban đầu ở đây');
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    debugPrint('🟡 [3] didChangeDependencies() — InheritedWidget thay đổi');
    debugPrint('    → Theme, MediaQuery thay đổi sẽ trigger method này');
  }

  @override
  void didUpdateWidget(covariant LifecycleWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    debugPrint('🟠 [*] didUpdateWidget() — Parent truyền widget mới');
    debugPrint('    → Old counter: ${oldWidget.counter}, New counter: ${widget.counter}');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('🔵 [4] build() — Xây dựng Widget Tree');
    debugPrint('    → KHÔNG làm việc nặng ở đây! build() có thể gọi nhiều lần');

    return Card(
      color: Colors.green[50],
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Text(
              '🧬 Lifecycle Widget',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
                color: Colors.green[800],
              ),
            ),
            const SizedBox(height: 8),
            Text('Parent counter: ${widget.counter}'),
            Text('Internal count: $_internalCount'),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _internalCount++;
                });
                debugPrint('--- setState() called → build() sẽ chạy lại ---');
              },
              child: const Text('Tăng Internal Count (setState)'),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void deactivate() {
    debugPrint('🟤 [5] deactivate() — Widget bị gỡ khỏi tree');
    super.deactivate();
  }

  @override
  void dispose() {
    debugPrint('🔴 [6] dispose() — Dọn dẹp! Cancel timer, close stream, dispose controller');
    debugPrint('    → Sau dispose, State object bị destroy vĩnh viễn');
    super.dispose();
  }
}
```

### Cách test

1. **Mở app** → Xem Debug Console:
   ```
   🔵 [1] createState()
   🟢 [2] initState()
   🟡 [3] didChangeDependencies()
   🔵 [4] build()
   ```

2. **Nhấn "Tăng Internal Count"** → Console:
   ```
   --- setState() called ---
   🔵 [4] build()         ← Chỉ build() chạy lại
   ```

3. **Nhấn "Update Parent"** → Console:
   ```
   🟠 [*] didUpdateWidget()   ← Parent truyền counter mới
   🔵 [4] build()
   ```

4. **Nhấn "Ẩn Widget"** → Console:
   ```
   🟤 [5] deactivate()
   🔴 [6] dispose()       ← Widget bị destroy
   ```

5. **Nhấn "Hiện Widget"** → Console:
   ```
   🔵 [1] createState()   ← Tạo lại từ đầu!
   🟢 [2] initState()
   🟡 [3] didChangeDependencies()
   🔵 [4] build()
   ```

- 🔗 **FE tương đương:** Lifecycle tương tự `useEffect(() => { /* init */ return () => { /* cleanup */ } }, [])` — nhưng Flutter lifecycle chi tiết hơn với `didUpdateWidget`, `didChangeDependencies` không có trong React hooks.

---

## VD4: BuildContext — Truy cập Theme và MediaQuery 🔴

### Mục đích
- Hiểu cách dùng `BuildContext` với pattern `.of(context)`
- Dùng `Theme.of(context)` để responsive theo theme
- Dùng `MediaQuery.of(context)` để responsive theo màn hình

> **Liên quan tới:** [4. BuildContext — Chìa khoá truy cập Widget Tree 🔴](01-ly-thuyet.md#4-buildcontext--chìa-khoá-truy-cập-widget-tree)

### Code — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD4 - BuildContext',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
        // Custom text theme
        textTheme: const TextTheme(
          headlineMedium: TextStyle(fontWeight: FontWeight.bold),
          bodyLarge: TextStyle(fontSize: 18),
        ),
      ),
      home: const BuildContextDemo(),
    );
  }
}

class BuildContextDemo extends StatelessWidget {
  const BuildContextDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Lấy Theme từ context
    final theme = Theme.of(context);
    final colorScheme = theme.colorScheme;

    // 2. Lấy kích thước màn hình từ context
    final mediaQuery = MediaQuery.of(context);
    final screenWidth = mediaQuery.size.width;
    final screenHeight = mediaQuery.size.height;
    final isLandscape = mediaQuery.orientation == Orientation.landscape;

    return Scaffold(
      appBar: AppBar(
        title: const Text('BuildContext Demo'),
        backgroundColor: colorScheme.inversePrimary,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ===== Section 1: Theme.of(context) =====
            Text(
              '🎨 Theme.of(context)',
              style: theme.textTheme.headlineMedium,
            ),
            const SizedBox(height: 8),
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: colorScheme.primaryContainer,
                borderRadius: BorderRadius.circular(12),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Primary Color',
                    style: TextStyle(color: colorScheme.primary, fontSize: 18),
                  ),
                  Text(
                    'Secondary Color',
                    style: TextStyle(color: colorScheme.secondary, fontSize: 18),
                  ),
                  Text(
                    'Error Color',
                    style: TextStyle(color: colorScheme.error, fontSize: 18),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'headlineMedium style',
                    style: theme.textTheme.headlineMedium,
                  ),
                  Text(
                    'bodyLarge style',
                    style: theme.textTheme.bodyLarge,
                  ),
                ],
              ),
            ),

            const SizedBox(height: 24),

            // ===== Section 2: MediaQuery.of(context) =====
            Text(
              '📱 MediaQuery.of(context)',
              style: theme.textTheme.headlineMedium,
            ),
            const SizedBox(height: 8),
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    _InfoRow('Screen Width', '${screenWidth.toStringAsFixed(0)} px'),
                    _InfoRow('Screen Height', '${screenHeight.toStringAsFixed(0)} px'),
                    _InfoRow('Orientation', isLandscape ? 'Landscape' : 'Portrait'),
                    _InfoRow('Pixel Ratio', '${mediaQuery.devicePixelRatio}x'),
                    _InfoRow('Top Padding (Safe Area)',
                        '${mediaQuery.padding.top.toStringAsFixed(0)} px'),
                    _InfoRow('Bottom Padding',
                        '${mediaQuery.padding.bottom.toStringAsFixed(0)} px'),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 24),

            // ===== Section 3: Responsive Layout =====
            Text(
              '📐 Responsive Layout',
              style: theme.textTheme.headlineMedium,
            ),
            const SizedBox(height: 8),
            Text(
              screenWidth > 600
                  ? '→ Tablet layout (width > 600)'
                  : '→ Phone layout (width ≤ 600)',
              style: theme.textTheme.bodyLarge?.copyWith(
                color: screenWidth > 600 ? Colors.green : Colors.blue,
              ),
            ),
            const SizedBox(height: 12),
            // Container responsive theo screen width
            Container(
              width: screenWidth > 600 ? screenWidth * 0.5 : screenWidth * 0.9,
              height: 80,
              decoration: BoxDecoration(
                color: screenWidth > 600
                    ? Colors.green[100]
                    : Colors.blue[100],
                borderRadius: BorderRadius.circular(12),
                border: Border.all(
                  color: screenWidth > 600 ? Colors.green : Colors.blue,
                ),
              ),
              alignment: Alignment.center,
              child: Text(
                screenWidth > 600
                    ? 'Tôi chiếm 50% width (tablet)'
                    : 'Tôi chiếm 90% width (phone)',
                style: const TextStyle(fontSize: 16),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Helper widget hiển thị thông tin dạng label-value
class _InfoRow extends StatelessWidget {
  final String label;
  final String value;

  const _InfoRow(this.label, this.value);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: const TextStyle(fontWeight: FontWeight.w500)),
          Text(value, style: TextStyle(color: Colors.grey[700])),
        ],
      ),
    );
  }
}
```

### Giải thích

```
.of(context) đi NGƯỢC LÊN tree để tìm ancestor:

        MaterialApp
        ├── Theme (← Theme.of(context) tìm thấy ở đây)
        ├── MediaQuery (← MediaQuery.of(context) tìm thấy ở đây)
        └── Scaffold
             └── Column
                  └── Text   ← context xuất phát từ đây
```

**Điểm chính:**
- `Theme.of(context)` — lấy theme colors, text styles mà không hardcode
- `MediaQuery.of(context)` — responsive theo screen size, orientation
- Pattern `.of(context)` luôn tìm **ngược lên** (ancestor), không tìm xuống (descendant)

---

## VD5: Key Demo — Reorder List có và không có Key 🟡

### Mục đích
- **Thấy rõ sự khác biệt** khi dùng Key vs không dùng Key
- Hiểu tại sao Key quan trọng khi list thay đổi thứ tự
- Hiểu concept **ValueKey**

> **Liên quan tới:** [5. Key trong Flutter — Giúp Flutter nhận diện Widget 🟡](01-ly-thuyet.md#5-key-trong-flutter--giúp-flutter-nhận-diện-widget)

### Code — `lib/main.dart`

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD5 - Key Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
        useMaterial3: true,
      ),
      home: const KeyDemoScreen(),
    );
  }
}

class KeyDemoScreen extends StatefulWidget {
  const KeyDemoScreen({super.key});

  @override
  State<KeyDemoScreen> createState() => _KeyDemoScreenState();
}

class _KeyDemoScreenState extends State<KeyDemoScreen> {
  List<String> _items = ['🍎 Apple', '🍌 Banana', '🍊 Orange'];

  void _shuffleItems() {
    setState(() {
      _items = List.from(_items)..shuffle();
    });
  }

  void _removeFirst() {
    setState(() {
      if (_items.isNotEmpty) {
        _items = List.from(_items)..removeAt(0);
      }
    });
  }

  void _resetItems() {
    setState(() {
      _items = ['🍎 Apple', '🍌 Banana', '🍊 Orange'];
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Key Demo'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // Buttons
            Wrap(
              spacing: 8,
              children: [
                ElevatedButton.icon(
                  onPressed: _shuffleItems,
                  icon: const Icon(Icons.shuffle),
                  label: const Text('Shuffle'),
                ),
                ElevatedButton.icon(
                  onPressed: _removeFirst,
                  icon: const Icon(Icons.remove_circle),
                  label: const Text('Xoá đầu'),
                ),
                ElevatedButton.icon(
                  onPressed: _resetItems,
                  icon: const Icon(Icons.refresh),
                  label: const Text('Reset'),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // ===== KHÔNG CÓ KEY =====
            Text(
              '❌ KHÔNG có Key',
              style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    color: Colors.red,
                  ),
            ),
            const Text(
              'Checkbox state bị gán SAI widget khi shuffle!',
              style: TextStyle(color: Colors.red, fontSize: 12),
            ),
            const SizedBox(height: 8),
            ...List.generate(_items.length, (index) {
              return ColorfulTile(
                // Không có key!
                title: _items[index],
                label: 'NO KEY',
              );
            }),

            const Divider(height: 32),

            // ===== CÓ KEY =====
            Text(
              '✅ CÓ ValueKey',
              style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    color: Colors.green,
                  ),
            ),
            const Text(
              'Checkbox state đi theo đúng widget!',
              style: TextStyle(color: Colors.green, fontSize: 12),
            ),
            const SizedBox(height: 8),
            ...List.generate(_items.length, (index) {
              return ColorfulTile(
                key: ValueKey(_items[index]), // ✅ Có Key!
                title: _items[index],
                label: 'HAS KEY',
              );
            }),
          ],
        ),
      ),
    );
  }
}

/// Tile có checkbox — StatefulWidget giữ trạng thái checked
class ColorfulTile extends StatefulWidget {
  final String title;
  final String label;

  const ColorfulTile({
    super.key,
    required this.title,
    required this.label,
  });

  @override
  State<ColorfulTile> createState() => _ColorfulTileState();
}

class _ColorfulTileState extends State<ColorfulTile> {
  bool _isChecked = false;

  @override
  Widget build(BuildContext context) {
    return Card(
      color: _isChecked ? Colors.green[50] : null,
      child: CheckboxListTile(
        title: Text(
          '${widget.title} [${ widget.label}]',
          style: TextStyle(
            decoration: _isChecked ? TextDecoration.lineThrough : null,
          ),
        ),
        value: _isChecked,
        onChanged: (value) {
          setState(() {
            _isChecked = value ?? false;
          });
        },
      ),
    );
  }
}
```

### Cách test

1. **Check "Apple" ở CẢ HAI list** (phần NO KEY và HAS KEY)
2. **Nhấn "Shuffle"**
3. **Quan sát:**
   - ❌ **NO KEY:** Checkbox checked vẫn ở **vị trí cũ** (pos 0), dù widget đã đổi chỗ → **State bị gán sai widget!**
   - ✅ **HAS KEY:** Checkbox checked **đi theo Apple** dù Apple đổi vị trí → **State gán đúng!**

### Giải thích

```
Không có Key — Flutter match theo TYPE + POSITION:
Trước:  [✅ Apple]  [  Banana]  [  Orange]   (pos 0 checked)
Sau:    [✅ Orange] [  Apple]   [  Banana]   (pos 0 vẫn checked → SAI!)

Có ValueKey — Flutter match theo KEY:
Trước:  [✅ Apple]  [  Banana]  [  Orange]   (Apple checked)
Sau:    [  Orange]  [✅ Apple]  [  Banana]   (Apple vẫn checked → ĐÚNG!)
```

---

> **Tiếp theo:** Chuyển sang [03-thuc-hanh.md](./03-thuc-hanh.md) để tự tay code 3 bài tập.
