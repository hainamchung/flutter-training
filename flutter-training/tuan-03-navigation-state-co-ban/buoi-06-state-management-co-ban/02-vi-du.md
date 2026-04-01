# Buổi 06: State Management Cơ Bản — Ví dụ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## VD1: Ephemeral State vs App State 🔴

> **Mục tiêu**: Thấy rõ khi nào `setState()` đủ tốt, khi nào cần chia sẻ state.

> **Liên quan tới:** [1. State trong Flutter là gì? 🔴](01-ly-thuyet.md#1-state-trong-flutter-là-gì)

### Phần A — Ephemeral State (setState là đủ)

Một counter đơn giản — chỉ 1 widget cần biết giá trị count.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: EphemeralDemo()));

class EphemeralDemo extends StatefulWidget {
  const EphemeralDemo({super.key});

  @override
  State<EphemeralDemo> createState() => _EphemeralDemoState();
}

class _EphemeralDemoState extends State<EphemeralDemo> {
  // Ephemeral state — chỉ widget này cần
  int _count = 0;
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Ephemeral State Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count: $_count',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 16),
            // Ephemeral state: UI toggle
            GestureDetector(
              onTap: () => setState(() => _isExpanded = !_isExpanded),
              child: AnimatedContainer(
                duration: const Duration(milliseconds: 300),
                width: _isExpanded ? 200 : 100,
                height: _isExpanded ? 200 : 100,
                color: _isExpanded ? Colors.blue : Colors.grey,
                child: Center(
                  child: Text(
                    _isExpanded ? 'Expanded' : 'Tap me',
                    style: const TextStyle(color: Colors.white),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _count++),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**Giải thích**: `_count` và `_isExpanded` là **ephemeral state** — chỉ widget `EphemeralDemo` cần biết. `setState()` là hoàn toàn phù hợp ở đây.

- 🔗 **FE tương đương:** Trong React: `const [count, setCount] = useState(0)` — nhưng React `setCount` chỉ re-render component đó, Flutter `setState` rebuild toàn bộ subtree.

### Phần B — Khi setState không đủ

Giả sử bạn có 2 screen cần cùng 1 giá trị count:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: HomeScreen()));

// ❌ VẤN ĐỀ: 2 screen cần cùng 1 count, nhưng mỗi widget có state riêng
class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home — Count: $_count')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count: $_count',
                style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => setState(() => _count++),
              child: const Text('Increment'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                // ❌ Phải truyền count qua constructor + nhận callback
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => DetailScreen(
                      count: _count,
                      onIncrement: () => setState(() => _count++),
                    ),
                  ),
                );
              },
              child: const Text('Go to Detail'),
            ),
          ],
        ),
      ),
    );
  }
}

// ❌ Phải nhận count + callback qua constructor — PROP DRILLING
class DetailScreen extends StatelessWidget {
  final int count;
  final VoidCallback onIncrement;

  const DetailScreen({
    super.key,
    required this.count,
    required this.onIncrement,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // ❌ count này là snapshot, không cập nhật real-time
            Text('Count: $count',
                style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: onIncrement,
              child: const Text('Increment from Detail'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Vấn đề**: Count ở `DetailScreen` là snapshot — không cập nhật real-time khi increment. Phải truyền callback qua constructor. Khi app phức tạp hơn → prop drilling hell.

→ **Giải pháp**: Dùng InheritedWidget hoặc Provider (xem VD3, VD4).

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §1.2 Hai loại State](01-ly-thuyet.md#12-hai-loại-state) · [§1.3 Decision Tree](01-ly-thuyet.md#13-decision-tree-chọn-cái-nào)

### ▶️ Chạy ví dụ

```bash
# Phần A — Ephemeral State:
flutter create vidu_ephemeral_state
cd vidu_ephemeral_state
# Thay nội dung lib/main.dart bằng code Phần A, rồi:
flutter run

# Phần B — Prop Drilling:
# Thay nội dung lib/main.dart bằng code Phần B, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
Phần A:
✅ Counter tăng khi nhấn FAB (+)
✅ Tap vào box → toggle expand/collapse với animation

Phần B:
✅ Home hiện count, nhấn "Increment" → count tăng
✅ Nhấn "Go to Detail" → Detail hiện count (snapshot)
✅ Nhấn "Increment from Detail" → count tăng ở Home nhưng Detail KHÔNG cập nhật real-time
```

---

## VD2: InheritedWidget — Custom Counter 🟡

> 📖 **Liên quan:** [Phần 3.2 — Cách hoạt động (InheritedWidget)](01-ly-thuyet.md#32-cách-hoạt-động)

> **Mục tiêu**: Hiểu cách InheritedWidget hoạt động dưới hood.

> **Liên quan tới:** [3. InheritedWidget — Cơ chế gốc của Flutter 🟡](01-ly-thuyet.md#3-inheritedwidget--cơ-chế-gốc-của-flutter)

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: InheritedWidgetDemo()));

// --- 1. Tạo InheritedWidget ---
class CounterInherited extends InheritedWidget {
  final int count;
  final VoidCallback increment;
  final VoidCallback decrement;

  const CounterInherited({
    super.key,
    required this.count,
    required this.increment,
    required this.decrement,
    required super.child,
  });

  // Static method of() — pattern chuẩn để truy cập
  static CounterInherited of(BuildContext context) {
    final widget =
        context.dependOnInheritedWidgetOfExactType<CounterInherited>();
    assert(widget != null, 'CounterInherited not found!');
    return widget!;
  }

  @override
  bool updateShouldNotify(CounterInherited oldWidget) {
    return count != oldWidget.count;
  }
}

// --- 2. StatefulWidget bọc InheritedWidget (quản lý state thực sự) ---
class InheritedWidgetDemo extends StatefulWidget {
  const InheritedWidgetDemo({super.key});

  @override
  State<InheritedWidgetDemo> createState() => _InheritedWidgetDemoState();
}

class _InheritedWidgetDemoState extends State<InheritedWidgetDemo> {
  int _count = 0;

  void _increment() => setState(() => _count++);
  void _decrement() => setState(() => _count--);

  @override
  Widget build(BuildContext context) {
    // InheritedWidget wrap toàn bộ subtree
    return CounterInherited(
      count: _count,
      increment: _increment,
      decrement: _decrement,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('InheritedWidget Demo'),
          // Widget ở đây cũng truy cập được count!
          actions: const [CounterBadge()],
        ),
        body: const Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CounterDisplay(),
            SizedBox(height: 24),
            CounterControls(),
          ],
        ),
      ),
    );
  }
}

// --- 3. Widget con đọc state qua of() ---
class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // O(1) lookup!
    final counter = CounterInherited.of(context);
    return Center(
      child: Text(
        '${counter.count}',
        style: Theme.of(context).textTheme.displayLarge,
      ),
    );
  }
}

class CounterControls extends StatelessWidget {
  const CounterControls({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = CounterInherited.of(context);
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        IconButton(
          icon: const Icon(Icons.remove, size: 32),
          onPressed: counter.decrement,
        ),
        const SizedBox(width: 24),
        IconButton(
          icon: const Icon(Icons.add, size: 32),
          onPressed: counter.increment,
        ),
      ],
    );
  }
}

class CounterBadge extends StatelessWidget {
  const CounterBadge({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = CounterInherited.of(context);
    return Center(
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 16),
        child: CircleAvatar(
          radius: 14,
          child: Text('${counter.count}', style: const TextStyle(fontSize: 12)),
        ),
      ),
    );
  }
}
```

**Giải thích**:
- `CounterInherited` chỉ **giữ data** và cung cấp `of()` method
- `_InheritedWidgetDemoState` **quản lý state thực sự** bằng `setState()`
- 3 widget con (`CounterDisplay`, `CounterControls`, `CounterBadge`) đều truy cập count qua `CounterInherited.of(context)` — **không cần truyền qua constructor**
- Nhược điểm: nhiều boilerplate (phải tạo class riêng, StatefulWidget bọc ngoài...)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_inherited_widget
cd vidu_inherited_widget
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Số lớn ở giữa màn hình hiển thị count
✅ Nhấn +/− → count thay đổi, cả display lẫn badge trên AppBar đều cập nhật
✅ Tất cả widget con truy cập count qua CounterInherited.of(context) — không truyền qua constructor
```

---

## VD3: Provider — Counter đơn giản 🔴

> 📖 **Liên quan:** [Phần 4.2 — ChangeNotifier](01-ly-thuyet.md#42-changenotifier--observable-object) · [Phần 4.4 — Đọc state: watch, read, select](01-ly-thuyet.md#44-đọc-state--watch-read-select)

> **Mục tiêu**: So sánh với VD2 — cùng chức năng nhưng ít boilerplate hơn nhiều.

> **Liên quan tới:** [4. ChangeNotifier + Provider 🔴](01-ly-thuyet.md#4-changenotifier--provider)

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// --- 1. ChangeNotifier — State & Logic ---
class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // Thông báo tất cả listeners
  }

  void decrement() {
    _count--;
    notifyListeners();
  }

  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// --- 2. Main — Cung cấp Provider ---
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: const MaterialApp(home: CounterScreen()),
    ),
  );
}

// --- 3. Screen dùng state ---
class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Provider Counter'),
        actions: [
          // context.watch tự rebuild khi count đổi
          Center(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: CircleAvatar(
                radius: 14,
                child: Text(
                  '${context.watch<CounterModel>().count}',
                  style: const TextStyle(fontSize: 12),
                ),
              ),
            ),
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Consumer — giới hạn rebuild chỉ trong builder
            Consumer<CounterModel>(
              builder: (context, counter, child) {
                return Text(
                  '${counter.count}',
                  style: Theme.of(context).textTheme.displayLarge,
                );
              },
            ),
            const SizedBox(height: 24),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  icon: const Icon(Icons.remove, size: 32),
                  // context.read — chỉ đọc 1 lần, không rebuild
                  onPressed: () => context.read<CounterModel>().decrement(),
                ),
                const SizedBox(width: 16),
                IconButton(
                  icon: const Icon(Icons.refresh, size: 32),
                  onPressed: () => context.read<CounterModel>().reset(),
                ),
                const SizedBox(width: 16),
                IconButton(
                  icon: const Icon(Icons.add, size: 32),
                  onPressed: () => context.read<CounterModel>().increment(),
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

**So sánh VD2 vs VD3**:
| | InheritedWidget (VD2) | Provider (VD3) |
|---|---|---|
| Code lines | ~100+ | ~70 |
| State class | InheritedWidget + StatefulWidget | ChangeNotifier (1 class) |
| Notify mechanism | setState ở parent | `notifyListeners()` tự động |
| Boilerplate | `updateShouldNotify`, `of()` method | Gần zero |
| Đọc state | `CounterInherited.of(context)` | `context.watch<CounterModel>()` |

- 🔗 **FE tương đương:** Tương tự `React.createContext` + `useContext` — nhưng Provider là widget trong tree, không phải API tách biệt. `notifyListeners()` phải gọi thủ công.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_provider_counter
cd vidu_provider_counter
# Thêm dependency
flutter pub add provider
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Số lớn ở giữa hiển thị count (Consumer rebuild chỉ phần này)
✅ Badge trên AppBar hiển thị count (context.watch tự rebuild)
✅ Nhấn +/−/reset → count thay đổi đồng bộ ở cả 2 vị trí
```

---

## VD4: Provider — Shopping Cart 🔴

> **Mục tiêu**: State phức tạp hơn — quản lý danh sách items, tính tổng.

> **Liên quan tới:** [4. ChangeNotifier + Provider 🔴](01-ly-thuyet.md#4-changenotifier--provider)

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// --- Models ---
class Product {
  final String id;
  final String name;
  final double price;
  final String emoji;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.emoji,
  });
}

// --- Data giả ---
const sampleProducts = [
  Product(id: '1', name: 'Cà phê', price: 35000, emoji: '☕'),
  Product(id: '2', name: 'Bánh mì', price: 25000, emoji: '🥖'),
  Product(id: '3', name: 'Phở', price: 55000, emoji: '🍜'),
  Product(id: '4', name: 'Trà sữa', price: 45000, emoji: '🧋'),
  Product(id: '5', name: 'Bánh cuốn', price: 40000, emoji: '🥟'),
];

// --- CartModel — ChangeNotifier ---
class CartModel extends ChangeNotifier {
  final Map<String, int> _items = {}; // productId → quantity

  Map<String, int> get items => Map.unmodifiable(_items);

  int get totalItems => _items.values.fold(0, (sum, qty) => sum + qty);

  double get totalPrice {
    double total = 0;
    _items.forEach((productId, qty) {
      final product = sampleProducts.firstWhere((p) => p.id == productId);
      total += product.price * qty;
    });
    return total;
  }

  int getQuantity(String productId) => _items[productId] ?? 0;

  void addItem(String productId) {
    _items[productId] = (_items[productId] ?? 0) + 1;
    notifyListeners();
  }

  void removeItem(String productId) {
    if (_items.containsKey(productId)) {
      if (_items[productId]! > 1) {
        _items[productId] = _items[productId]! - 1;
      } else {
        _items.remove(productId);
      }
      notifyListeners();
    }
  }

  void clearCart() {
    _items.clear();
    notifyListeners();
  }
}

// --- Main ---
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CartModel(),
      child: const MaterialApp(home: ProductListScreen()),
    ),
  );
}

// --- Product List Screen ---
class ProductListScreen extends StatelessWidget {
  const ProductListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Menu'),
        actions: [
          // Badge hiển thị số items trong cart
          Stack(
            alignment: Alignment.center,
            children: [
              IconButton(
                icon: const Icon(Icons.shopping_cart),
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (_) => const CartScreen()),
                  );
                },
              ),
              Positioned(
                right: 4,
                top: 4,
                child: Consumer<CartModel>(
                  builder: (context, cart, child) {
                    if (cart.totalItems == 0) return const SizedBox.shrink();
                    return CircleAvatar(
                      radius: 10,
                      backgroundColor: Colors.red,
                      child: Text(
                        '${cart.totalItems}',
                        style: const TextStyle(
                          fontSize: 10,
                          color: Colors.white,
                        ),
                      ),
                    );
                  },
                ),
              ),
            ],
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: sampleProducts.length,
        itemBuilder: (context, index) {
          final product = sampleProducts[index];
          return ProductTile(product: product);
        },
      ),
    );
  }
}

class ProductTile extends StatelessWidget {
  final Product product;

  const ProductTile({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    // select — chỉ rebuild khi quantity CỦA PRODUCT NÀY thay đổi
    final quantity = context.select<CartModel, int>(
      (cart) => cart.getQuantity(product.id),
    );

    return ListTile(
      leading: Text(product.emoji, style: const TextStyle(fontSize: 32)),
      title: Text(product.name),
      subtitle: Text('${product.price.toStringAsFixed(0)}đ'),
      trailing: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          if (quantity > 0) ...[
            IconButton(
              icon: const Icon(Icons.remove_circle_outline),
              onPressed: () =>
                  context.read<CartModel>().removeItem(product.id),
            ),
            Text('$quantity', style: const TextStyle(fontSize: 16)),
          ],
          IconButton(
            icon: const Icon(Icons.add_circle, color: Colors.blue),
            onPressed: () => context.read<CartModel>().addItem(product.id),
          ),
        ],
      ),
    );
  }
}

// --- Cart Screen ---
class CartScreen extends StatelessWidget {
  const CartScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Giỏ hàng'),
        actions: [
          TextButton(
            onPressed: () => context.read<CartModel>().clearCart(),
            child: const Text('Xóa tất cả',
                style: TextStyle(color: Colors.white)),
          ),
        ],
      ),
      body: Consumer<CartModel>(
        builder: (context, cart, child) {
          if (cart.items.isEmpty) {
            return const Center(
              child: Text('Giỏ hàng trống 🛒',
                  style: TextStyle(fontSize: 18)),
            );
          }

          final cartEntries = cart.items.entries.toList();

          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cartEntries.length,
                  itemBuilder: (context, index) {
                    final entry = cartEntries[index];
                    final product = sampleProducts
                        .firstWhere((p) => p.id == entry.key);
                    return ListTile(
                      leading: Text(product.emoji,
                          style: const TextStyle(fontSize: 24)),
                      title: Text(product.name),
                      subtitle: Text(
                          '${product.price.toStringAsFixed(0)}đ × ${entry.value}'),
                      trailing: Text(
                        '${(product.price * entry.value).toStringAsFixed(0)}đ',
                        style: const TextStyle(
                            fontWeight: FontWeight.bold, fontSize: 16),
                      ),
                    );
                  },
                ),
              ),
              // Tổng cộng
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.grey[100],
                  border: const Border(
                      top: BorderSide(color: Colors.grey)),
                ),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Tổng cộng:',
                        style: TextStyle(
                            fontSize: 18, fontWeight: FontWeight.bold)),
                    Text(
                      '${cart.totalPrice.toStringAsFixed(0)}đ',
                      style: const TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                        color: Colors.blue,
                      ),
                    ),
                  ],
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

**Điểm nổi bật**:
- `CartModel` chứa **toàn bộ logic**: add, remove, calculate total
- `context.select()` ở `ProductTile` → chỉ rebuild khi quantity của product đó thay đổi (tối ưu performance)
- `Consumer<CartModel>` ở cart badge → chỉ rebuild badge, không rebuild cả AppBar
- `context.read()` trong `onPressed` → chỉ gọi method, không subscribe
- Cart state **chia sẻ** giữa `ProductListScreen` và `CartScreen` — không cần truyền qua constructor

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §4.2 ChangeNotifier](01-ly-thuyet.md#42-changenotifier--observable-object) · [§4.4 Đọc state: watch, read, select](01-ly-thuyet.md#44-đọc-state--watch-read-select) · [§4.5 Consumer Widget](01-ly-thuyet.md#45-consumer-widget)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_provider_cart
cd vidu_provider_cart
# Thêm dependency
flutter pub add provider
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Danh sách 5 món ăn với emoji, tên, giá
✅ Nhấn (+) → thêm vào giỏ, badge trên icon cart cập nhật số lượng
✅ Nhấn (−) → giảm số lượng, hết thì xóa khỏi giỏ
✅ Nhấn icon cart → CartScreen hiện danh sách items với tổng tiền
✅ Nhấn "Xóa tất cả" → giỏ hàng trống
```

---

## VD5: Form với Validation — Login Form 🟡

> 📖 **Liên quan:** [Phần 5.2 — Các thành phần chính (Form)](01-ly-thuyet.md#52-các-thành-phần-chính)

> **Mục tiêu**: Tạo form login hoàn chỉnh với email + password validation.

> **Liên quan tới:** [5. Forms & Validation 🟡](01-ly-thuyet.md#5-forms--validation)

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: LoginScreen()));

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  // 1. GlobalKey để truy cập FormState
  final _formKey = GlobalKey<FormState>();

  // 2. Controllers để đọc/ghi giá trị
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  // 3. State cho UI
  bool _obscurePassword = true;
  bool _isLoading = false;

  @override
  void dispose() {
    // Luôn dispose controllers!
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  // Xử lý submit
  void _handleLogin() {
    // validate() chạy TẤT CẢ validators
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();

      setState(() => _isLoading = true);

      // Giả lập API call
      Future.delayed(const Duration(seconds: 2), () {
        if (mounted) {
          setState(() => _isLoading = false);
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('Login thành công: ${_emailController.text}'),
              backgroundColor: Colors.green,
            ),
          );
        }
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24),
        child: Form(
          key: _formKey,
          // Validate khi user tương tác (sau lần submit đầu)
          autovalidateMode: AutovalidateMode.onUserInteraction,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const SizedBox(height: 32),
              // Icon
              const Icon(Icons.lock_outline, size: 64, color: Colors.blue),
              const SizedBox(height: 32),

              // --- Email Field ---
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'you@example.com',
                  prefixIcon: Icon(Icons.email_outlined),
                  border: OutlineInputBorder(),
                ),
                validator: _validateEmail,
              ),
              const SizedBox(height: 16),

              // --- Password Field ---
              TextFormField(
                controller: _passwordController,
                obscureText: _obscurePassword,
                textInputAction: TextInputAction.done,
                decoration: InputDecoration(
                  labelText: 'Mật khẩu',
                  hintText: 'Nhập mật khẩu',
                  prefixIcon: const Icon(Icons.lock_outlined),
                  border: const OutlineInputBorder(),
                  suffixIcon: IconButton(
                    icon: Icon(_obscurePassword
                        ? Icons.visibility_off
                        : Icons.visibility),
                    onPressed: () {
                      setState(
                          () => _obscurePassword = !_obscurePassword);
                    },
                  ),
                ),
                validator: _validatePassword,
                onFieldSubmitted: (_) => _handleLogin(),
              ),
              const SizedBox(height: 8),

              // Forgot password
              Align(
                alignment: Alignment.centerRight,
                child: TextButton(
                  onPressed: () {},
                  child: const Text('Quên mật khẩu?'),
                ),
              ),
              const SizedBox(height: 16),

              // --- Submit Button ---
              SizedBox(
                height: 48,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _handleLogin,
                  child: _isLoading
                      ? const SizedBox(
                          height: 20,
                          width: 20,
                          child: CircularProgressIndicator(
                            strokeWidth: 2,
                            color: Colors.white,
                          ),
                        )
                      : const Text('Đăng nhập',
                          style: TextStyle(fontSize: 16)),
                ),
              ),
              const SizedBox(height: 16),

              // --- Reset Button ---
              TextButton(
                onPressed: () {
                  _formKey.currentState!.reset();
                  _emailController.clear();
                  _passwordController.clear();
                },
                child: const Text('Reset form'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  // --- Validator functions ---
  String? _validateEmail(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Vui lòng nhập email';
    }
    // Regex cơ bản cho email
    if (!RegExp(r'^[^@\s]+@[^@\s]+\.[^@\s]+$').hasMatch(value.trim())) {
      return 'Email không hợp lệ';
    }
    return null; // Hợp lệ
  }

  String? _validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return 'Vui lòng nhập mật khẩu';
    }
    if (value.length < 6) {
      return 'Mật khẩu phải có ít nhất 6 ký tự';
    }
    if (!RegExp(r'[A-Z]').hasMatch(value)) {
      return 'Mật khẩu phải có ít nhất 1 chữ hoa';
    }
    if (!RegExp(r'[0-9]').hasMatch(value)) {
      return 'Mật khẩu phải có ít nhất 1 số';
    }
    return null;
  }
}
```

**Điểm nổi bật**:
- `GlobalKey<FormState>` cho phép gọi `validate()`, `save()`, `reset()` trên toàn bộ form
- `validator` trả về `null` = hợp lệ, `String` = error message (hiển thị tự động bên dưới field)
- `autovalidateMode: AutovalidateMode.onUserInteraction` — validate real-time sau khi user bắt đầu gõ
- `TextEditingController` để đọc giá trị + clear khi reset
- `mounted` check trước `setState` trong async callback
- Toggle `obscureText` cho password field
- Tách validator thành method riêng → dễ đọc, dễ test

- 🔗 **FE tương đương:** Tương tự React Hook Form `register` + `handleSubmit` — nhưng Flutter dùng `GlobalKey<FormState>` thay vì hook pattern.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_login_form
cd vidu_login_form
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Form login với 2 field: Email và Mật khẩu
✅ Nhập email sai format → hiện lỗi "Email không hợp lệ" bên dưới field
✅ Mật khẩu < 6 ký tự hoặc thiếu chữ hoa/số → hiện lỗi tương ứng
✅ Nhấn "Đăng nhập" với data hợp lệ → loading 2 giây → SnackBar "Login thành công"
✅ Nhấn "Reset form" → xóa tất cả input và error
```

---

## VD6: 🤖 AI Gen → Review — Provider Shopping Cart 🟢

> **Mục đích:** Luyện workflow "AI gen Provider code → bạn review watch/read usage → fix issues"

> **Liên quan tới:** [4. ChangeNotifier + Provider 🔴](01-ly-thuyet.md#4-changenotifier--provider)

### Bước 1: Prompt cho AI

```text
Tạo Shopping Cart dùng Provider trong Flutter.
Features: addItem, removeItem, totalPrice getter, itemCount getter.
Dùng ChangeNotifierProvider wrap MaterialApp.
1 screen CartPage hiển thị list items + total price + clear button.
Output: cart_model.dart + main.dart + cart_page.dart.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **watch vs read** | `context.watch` trong build? `context.read` trong onPressed? (ngược lại = BUG) |
| 2 | **notifyListeners** | Có gọi sau mỗi state mutation (add/remove/clear)? Thiếu = UI không update |
| 3 | **Consumer scope** | Consumer wrap toàn bộ Scaffold hay chỉ phần cần rebuild? |
| 4 | **Immutable state** | CartModel expose List trực tiếp hay qua getter `UnmodifiableListView`? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Dùng context.watch trong onPressed
ElevatedButton(
  onPressed: () {
    context.watch<CartModel>().addItem(item); // WRONG — watch trong callback!
  },
)

// ✅ FIX: Dùng context.read trong callback
ElevatedButton(
  onPressed: () {
    context.read<CartModel>().addItem(item); // CORRECT — read trong callback
  },
)
```

```dart
// ❌ LỖI 2: Expose mutable list
class CartModel extends ChangeNotifier {
  final List<Item> items = []; // WRONG — bên ngoài có thể modify trực tiếp
}

// ✅ FIX: Expose UnmodifiableListView
class CartModel extends ChangeNotifier {
  final List<Item> _items = [];
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);
}
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Phân biệt rõ khi nào `watch` vs `read` — tránh bug UI không update
- ✅ Biết Consumer wrap đúng scope để tối ưu rebuild
- ✅ Hiểu tại sao expose UnmodifiableListView thay vì List trực tiếp
