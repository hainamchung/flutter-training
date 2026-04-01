# Buổi 05: Navigation & Routing — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> Tất cả ví dụ đều là Flutter app hoàn chỉnh, có thể chạy ngay.

---

## Ví dụ 1: Navigator 1.0 — Push/Pop giữa 2 screens 🟡

> 📖 **Liên quan:** [Phần 2.1 — Navigator.push() và MaterialPageRoute](01-ly-thuyet.md#21-navigatorpush-và-materialpageroute)

### 🎯 Mục tiêu
Minh họa cách dùng `Navigator.push()` và `Navigator.pop()` cơ bản.

> **Liên quan tới:** [2. Navigator 1.0 🟡](01-ly-thuyet.md#2-navigator-10)

### 📝 Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD1: Push/Pop',
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}

// ─── Home Screen ───────────────────────────────────────────
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Push DetailScreen lên navigation stack
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => const DetailScreen(),
              ),
            );
          },
          child: const Text('Go to Detail'),
        ),
      ),
    );
  }
}

// ─── Detail Screen ─────────────────────────────────────────
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Đây là Detail Screen',
              style: TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                // Pop DetailScreen khỏi stack → quay lại Home
                Navigator.pop(context);
              },
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 🔍 Giải thích

```
Ban đầu:          Sau push:          Sau pop:
┌──────────┐     ┌──────────┐      ┌──────────┐
│   Home    │     │  Detail  │ ←    │   Home   │ ← hiển thị lại
└──────────┘     ├──────────┤      └──────────┘
                 │   Home   │
                 └──────────┘
```

- `Navigator.push()` thêm `DetailScreen` lên stack → Detail hiển thị
- `Navigator.pop()` xóa `DetailScreen` khỏi stack → Home hiển thị lại
- Nút Back trên AppBar (←) cũng gọi `pop()` tự động

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_push_pop
cd vidu_push_pop
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Màn hình Home hiện nút "Go to Detail"
✅ Nhấn nút → chuyển sang Detail Screen với animation slide
✅ Nhấn "Go Back" hoặc nút ← trên AppBar → quay lại Home
```

- 🔗 **FE tương đương:** Tương tự `history.push('/detail')` + `history.back()` — nhưng Flutter có animation transition built-in và có thể return data qua `Navigator.pop(context, result)`.

---

## Ví dụ 2: Named Routes — 3 screens 🟡

### 🎯 Mục tiêu
Minh họa cách sử dụng named routes thay vì tạo `MaterialPageRoute` mỗi lần.

> **Liên quan tới:** [2. Navigator 1.0 🟡](01-ly-thuyet.md#2-navigator-10)

### 📝 Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD2: Named Routes',
      theme: ThemeData(
        colorSchemeSeed: Colors.green,
        useMaterial3: true,
      ),
      // Đăng ký routes
      initialRoute: '/',
      routes: {
        '/': (context) => const HomeScreen(),
        '/detail': (context) => const DetailScreen(),
        '/settings': (context) => const SettingsScreen(),
      },
    );
  }
}

// ─── Home Screen ───────────────────────────────────────────
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => Navigator.pushNamed(context, '/detail'),
              child: const Text('Go to Detail'),
            ),
            const SizedBox(height: 12),
            ElevatedButton(
              onPressed: () => Navigator.pushNamed(context, '/settings'),
              child: const Text('Go to Settings'),
            ),
          ],
        ),
      ),
    );
  }
}

// ─── Detail Screen ─────────────────────────────────────────
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.article, size: 64, color: Colors.green),
            const SizedBox(height: 16),
            const Text('Detail Screen', style: TextStyle(fontSize: 20)),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => Navigator.pushNamed(context, '/settings'),
              child: const Text('Go to Settings'),
            ),
          ],
        ),
      ),
    );
  }
}

// ─── Settings Screen ───────────────────────────────────────
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Settings')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.settings, size: 64, color: Colors.grey),
            const SizedBox(height: 16),
            const Text('Settings Screen', style: TextStyle(fontSize: 20)),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                // Pop tất cả routes và quay về Home
                Navigator.popUntil(context, ModalRoute.withName('/'));
              },
              child: const Text('Back to Home'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 🔍 Giải thích

- **`routes` map**: Đăng ký tên route → Widget builder. Code gọn hơn so với tạo `MaterialPageRoute` mỗi lần.
- **`Navigator.pushNamed()`**: Navigate bằng String name thay vì tạo route object.
- **`Navigator.popUntil()`**: Pop liên tục cho đến khi tìm thấy route thỏa điều kiện — ở đây là route có name `'/'`.

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §2.3 Named Routes](01-ly-thuyet.md#23-named-routes)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_named_routes
cd vidu_named_routes
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Home screen hiện 2 nút: "Go to Detail" và "Go to Settings"
✅ Nhấn "Go to Detail" → chuyển sang Detail Screen
✅ Từ Detail nhấn "Go to Settings" → chuyển sang Settings Screen
✅ Nhấn "Back to Home" → popUntil quay thẳng về Home (bỏ qua Detail)
```

---

## Ví dụ 3: GoRouter cơ bản — 3 routes 🟡

> 📖 **Liên quan:** [Phần 3.3 — GoRouter Setup cơ bản](01-ly-thuyet.md#33-gorouter-setup-cơ-bản) · [Phần 3.4 — Navigation với GoRouter](01-ly-thuyet.md#34-navigation-với-gorouter)

### 🎯 Mục tiêu
Cấu hình GoRouter, định nghĩa routes, navigate bằng `context.go()` và `context.push()`.

> **Liên quan tới:** [3. Navigator 2.0 / GoRouter 🟡](01-ly-thuyet.md#3-navigator-20--gorouter)

### 📝 Code

> **Lưu ý**: Thêm `go_router` vào `pubspec.yaml` trước khi chạy:
> ```yaml
> dependencies:
>   flutter:
>     sdk: flutter
>   go_router: ^14.2.0
> ```

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() => runApp(const MyApp());

// ─── Router Config ─────────────────────────────────────────
final GoRouter _router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/detail/:id',
      name: 'detail',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? 'unknown';
        return DetailScreen(id: id);
      },
    ),
    GoRoute(
      path: '/about',
      name: 'about',
      builder: (context, state) => const AboutScreen(),
    ),
  ],
  // Xử lý route không tồn tại
  errorBuilder: (context, state) => Scaffold(
    appBar: AppBar(title: const Text('404')),
    body: Center(
      child: Text('Page not found: ${state.uri}'),
    ),
  ),
);

// ─── App ───────────────────────────────────────────────────
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'VD3: GoRouter Basic',
      theme: ThemeData(
        colorSchemeSeed: Colors.purple,
        useMaterial3: true,
      ),
      routerConfig: _router,
    );
  }
}

// ─── Home Screen ───────────────────────────────────────────
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Dùng context.push() → có thể pop back
            ElevatedButton(
              onPressed: () => context.push('/detail/1'),
              child: const Text('Push Detail (id: 1)'),
            ),
            const SizedBox(height: 12),
            // Dùng context.go() → replace stack
            ElevatedButton(
              onPressed: () => context.go('/detail/2'),
              child: const Text('Go to Detail (id: 2)'),
            ),
            const SizedBox(height: 12),
            // Navigate bằng name
            ElevatedButton(
              onPressed: () => context.goNamed('about'),
              child: const Text('Go to About'),
            ),
          ],
        ),
      ),
    );
  }
}

// ─── Detail Screen ─────────────────────────────────────────
class DetailScreen extends StatelessWidget {
  final String id;
  const DetailScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Detail #$id')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Đang xem item: $id',
              style: const TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.pop(),
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }
}

// ─── About Screen ──────────────────────────────────────────
class AboutScreen extends StatelessWidget {
  const AboutScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('About')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.info, size: 64, color: Colors.purple),
            SizedBox(height: 16),
            Text('GoRouter Demo App', style: TextStyle(fontSize: 20)),
            Text('Version 1.0.0'),
          ],
        ),
      ),
    );
  }
}
```

### 🔍 Giải thích

| Action | Code | Kết quả |
|--------|------|---------|
| Push | `context.push('/detail/1')` | Thêm Detail lên stack, **có** nút Back |
| Go | `context.go('/detail/2')` | Navigate tới Detail, stack **bị thay thế** |
| GoNamed | `context.goNamed('about')` | Navigate bằng route name (type-safe hơn) |
| Pop | `context.pop()` | Quay lại screen trước (nếu push) |

> **Thử nghiệm**: Nhấn "Push Detail (id: 1)" → có nút Back. Nhấn "Go to Detail (id: 2)" → không có nút Back vì stack bị replace.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_gorouter
cd vidu_gorouter
# Thêm dependency
flutter pub add go_router
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Home screen hiện 3 nút: Push Detail, Go to Detail, Go to About
✅ "Push Detail (id: 1)" → Detail #1 với nút Back (vì push)
✅ "Go to Detail (id: 2)" → Detail #2 KHÔNG có nút Back (vì go replace stack)
✅ "Go to About" → About screen với icon info
```

- 🔗 **FE tương đương:** Khai báo routes giống React Router `<Route path="/users/:id" element={<UserPage/>}/>` — GoRouter mapping gần 1:1 về cấu trúc declarative.

---

## Ví dụ 4: Truyền data & Return result 🔴

### 🎯 Mục tiêu
Truyền object tới detail screen và nhận result khi pop.

> **Liên quan tới:** [4. Truyền Data giữa Screens 🔴](01-ly-thuyet.md#4-truyền-data-giữa-screens)

### 📝 Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

// ─── Data Model ────────────────────────────────────────────
class Product {
  final int id;
  final String name;
  final double price;
  const Product({required this.id, required this.name, required this.price});
}

// ─── Sample Data ───────────────────────────────────────────
const List<Product> sampleProducts = [
  Product(id: 1, name: 'iPhone 15', price: 999),
  Product(id: 2, name: 'MacBook Pro', price: 2499),
  Product(id: 3, name: 'AirPods Pro', price: 249),
];

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'VD4: Data Passing',
      theme: ThemeData(
        colorSchemeSeed: Colors.orange,
        useMaterial3: true,
      ),
      home: const ProductListScreen(),
    );
  }
}

// ─── Product List Screen ───────────────────────────────────
class ProductListScreen extends StatefulWidget {
  const ProductListScreen({super.key});

  @override
  State<ProductListScreen> createState() => _ProductListScreenState();
}

class _ProductListScreenState extends State<ProductListScreen> {
  String _lastAction = 'Chưa có action nào';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: Column(
        children: [
          // Hiển thị result trả về từ DetailScreen
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            color: Colors.orange.shade50,
            child: Text(
              'Last action: $_lastAction',
              style: const TextStyle(fontSize: 16),
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: sampleProducts.length,
              itemBuilder: (context, index) {
                final product = sampleProducts[index];
                return ListTile(
                  leading: const Icon(Icons.shopping_bag),
                  title: Text(product.name),
                  subtitle: Text('\$${product.price}'),
                  trailing: const Icon(Icons.chevron_right),
                  onTap: () async {
                    // Push DetailScreen, truyền product qua constructor
                    // Chờ result khi pop
                    final result = await Navigator.push<String>(
                      context,
                      MaterialPageRoute(
                        builder: (context) => ProductDetailScreen(
                          product: product,
                        ),
                      ),
                    );
                    // Xử lý result
                    if (result != null) {
                      setState(() => _lastAction = result);
                    }
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// ─── Product Detail Screen ─────────────────────────────────
class ProductDetailScreen extends StatelessWidget {
  final Product product;
  const ProductDetailScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(product.name)),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Product info
            Center(
              child: Icon(
                Icons.shopping_bag,
                size: 100,
                color: Colors.orange.shade300,
              ),
            ),
            const SizedBox(height: 24),
            Text(
              product.name,
              style: const TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Text(
              '\$${product.price}',
              style: TextStyle(fontSize: 22, color: Colors.grey.shade600),
            ),
            const Spacer(),

            // Buttons trả result khác nhau khi pop
            SizedBox(
              width: double.infinity,
              child: ElevatedButton.icon(
                onPressed: () {
                  // Pop với result: đã thêm vào giỏ hàng
                  Navigator.pop(
                    context,
                    'Added "${product.name}" to cart',
                  );
                },
                icon: const Icon(Icons.add_shopping_cart),
                label: const Text('Add to Cart'),
              ),
            ),
            const SizedBox(height: 8),
            SizedBox(
              width: double.infinity,
              child: OutlinedButton.icon(
                onPressed: () {
                  // Pop với result: đã thêm vào wishlist
                  Navigator.pop(
                    context,
                    'Added "${product.name}" to wishlist',
                  );
                },
                icon: const Icon(Icons.favorite_border),
                label: const Text('Add to Wishlist'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 🔍 Giải thích

**Flow truyền data:**

```
ProductListScreen                  ProductDetailScreen
      │                                    │
      │── push(ProductDetailScreen(        │
      │       product: product)) ─────────▶│  Nhận product qua constructor
      │                                    │
      │                                    │── User nhấn "Add to Cart"
      │◀── pop(context,                    │
      │     'Added "iPhone" to cart') ─────│  Trả result qua pop
      │                                    │
      │ result = 'Added "iPhone" to cart'  │
```

- **Truyền đi**: `product` được truyền qua **constructor** của `ProductDetailScreen`
- **Trả về**: `String` result được trả qua **`Navigator.pop(context, result)`**
- **Nhận về**: `await Navigator.push<String>(...)` nhận result khi detail screen pop

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §4.1 Qua Constructor Arguments](01-ly-thuyet.md#41-qua-constructor-arguments-navigator-10) · [§4.3 Return Data từ Pop](01-ly-thuyet.md#43-return-data-từ-pop)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_data_passing
cd vidu_data_passing
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Danh sách 3 sản phẩm: iPhone 15, MacBook Pro, AirPods Pro
✅ Tap vào sản phẩm → chuyển sang Detail Screen với tên + giá
✅ Nhấn "Add to Cart" → pop về list, hiện "Last action: Added ... to cart"
✅ Nhấn "Add to Wishlist" → pop về list, hiện "Last action: Added ... to wishlist"
```

- 🔗 **FE tương đương:** `pathParameters` ≈ `useParams()`, `extra` ≈ `location.state` — nhưng extra mất khi serialize (deep link), nên dùng path params cho ID.

---

## Ví dụ 5: StatefulShellRoute — Bottom Tab Navigation với GoRouter (giữ tab state) 🟡

> 📖 **Liên quan:** [Phần 5.3 — GoRouter: StatefulShellRoute](01-ly-thuyet.md#53-gorouter-statefulshellroute)

### 🎯 Mục tiêu
Dùng `StatefulShellRoute.indexedStack` tạo app với bottom navigation bar cố định, **giữ nguyên state mỗi tab** (scroll position, navigation history) khi chuyển tab.

> **Liên quan tới:** [5. Nested Navigation 🟡](01-ly-thuyet.md#5-nested-navigation)

### 📝 Code

> **Lưu ý**: Cần `go_router` package.

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() => runApp(const MyApp());

// ─── Tạo GlobalKey cho mỗi tab navigator ──────────────────
final _rootNavigatorKey = GlobalKey<NavigatorState>();

// ─── Router Config ─────────────────────────────────────────
final GoRouter _router = GoRouter(
  navigatorKey: _rootNavigatorKey,
  initialLocation: '/home',
  routes: [
    // StatefulShellRoute.indexedStack giữ state mỗi tab
    // → Khác với ShellRoute thông thường, mỗi tab có navigator riêng
    // → Khi chuyển tab, scroll position và navigation stack được preserve
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        // navigationShell chứa thông tin tab hiện tại
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // Mỗi StatefulShellBranch là một tab với navigator riêng
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/home',
              name: 'home',
              builder: (context, state) => const HomeTab(),
              // Sub-route: detail screen BÊN TRONG tab Home
              routes: [
                GoRoute(
                  path: 'detail/:id',
                  name: 'home-detail',
                  builder: (context, state) {
                    final id = state.pathParameters['id'] ?? '';
                    return DetailScreen(id: id, fromTab: 'Home');
                  },
                ),
              ],
            ),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/search',
              name: 'search',
              builder: (context, state) => const SearchTab(),
            ),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              name: 'profile',
              builder: (context, state) => const ProfileTab(),
            ),
          ],
        ),
      ],
    ),
  ],
);

// ─── App ───────────────────────────────────────────────────
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'VD5: StatefulShellRoute Tabs',
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      routerConfig: _router,
    );
  }
}

// ─── Scaffold with Bottom Navigation ──────────────────────
class ScaffoldWithNavBar extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const ScaffoldWithNavBar({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell, // ← StatefulNavigationShell tự quản lý content
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          // goBranch chuyển tab và giữ nguyên state tab cũ
          navigationShell.goBranch(
            index,
            // Nếu tap lại tab đang active → quay về initial location
            initialLocation: index == navigationShell.currentIndex,
          );
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'Home',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: 'Search',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outlined),
            selectedIcon: Icon(Icons.person),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}

// ─── Home Tab ──────────────────────────────────────────────
class HomeTab extends StatelessWidget {
  const HomeTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: ListView.builder(
        itemCount: 10,
        itemBuilder: (context, index) {
          return ListTile(
            leading: CircleAvatar(child: Text('${index + 1}')),
            title: Text('Item ${index + 1}'),
            subtitle: const Text('Tap to view detail'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () {
              // Push detail screen (vẫn trong StatefulShellRoute → bottom nav vẫn hiện)
              context.push('/home/detail/${index + 1}');
            },
          );
        },
      ),
    );
  }
}

// ─── Search Tab ────────────────────────────────────────────
class SearchTab extends StatelessWidget {
  const SearchTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Search')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.search, size: 64, color: Colors.teal),
            SizedBox(height: 16),
            Text('Search Tab', style: TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}

// ─── Profile Tab ───────────────────────────────────────────
class ProfileTab extends StatelessWidget {
  const ProfileTab({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 40,
              child: Icon(Icons.person, size: 40),
            ),
            SizedBox(height: 16),
            Text('John Doe', style: TextStyle(fontSize: 22)),
            Text('john@example.com'),
          ],
        ),
      ),
    );
  }
}

// ─── Detail Screen ─────────────────────────────────────────
class DetailScreen extends StatelessWidget {
  final String id;
  final String fromTab;
  const DetailScreen({super.key, required this.id, required this.fromTab});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Detail #$id')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Item $id',
              style: const TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Text(
              'Navigated from: $fromTab tab',
              style: TextStyle(fontSize: 16, color: Colors.grey.shade600),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 🔍 Giải thích

```
┌─────────────────────────────────────────────┐
│  StatefulShellRoute.indexedStack             │
│  ┌───────────┬───────────┬───────────┐      │
│  │ HomeTab   │ SearchTab │ ProfileTab│      │
│  │ (giữ      │ (giữ      │ (giữ      │      │
│  │  state)   │  state)   │  state)   │      │
│  └───────────┴───────────┴───────────┘      │
│  ┌─────────────────────────────────────┐    │
│  │  🏠 Home  🔍 Search  👤 Profile     │    │  ← NavigationBar cố định
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Điểm quan trọng:**

1. **`StatefulShellRoute.indexedStack`** tạo một `IndexedStack` bên dưới — mỗi tab có navigator riêng, **giữ nguyên state** khi chuyển tab (scroll position, sub-navigation history).
2. **`StatefulShellBranch`** — mỗi branch là một tab. Routes bên trong branch dùng navigator riêng.
3. **`navigationShell.goBranch(index)`** — chuyển tab, giữ state tab cũ. Khác với `context.go()` ở `ShellRoute` thông thường (destroy tab cũ).
4. **Sub-route** `/home/detail/:id` nằm trong branch Home → detail screen hiện trong tab Home, bottom nav vẫn hiện.

> ⚠️ **Tại sao không dùng `ShellRoute`?** `ShellRoute` thông thường **destroy** tab content khi chuyển tab → mất scroll position, mất sub-navigation history. `StatefulShellRoute.indexedStack` giữ tất cả tabs alive, giống behavior của `BottomNavigationBar` + `IndexedStack` truyền thống.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_shell_route
cd vidu_shell_route
# Thêm dependency
flutter pub add go_router
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Bottom navigation bar với 3 tab: Home, Search, Profile
✅ Tab Home hiện danh sách 10 items, tap vào → Detail screen (bottom nav vẫn hiện)
✅ Chuyển sang tab Search rồi quay lại Home → scroll position và navigation history được giữ nguyên
✅ Tap lại tab đang active → quay về initial location của tab đó

---

## Ví dụ 6: 🤖 AI Gen → Review — GoRouter Auth Redirect 🟢

> **Mục đích:** Luyện workflow "AI gen config → bạn review → fix issues"

> **Liên quan tới:** [3. Navigator 2.0 / GoRouter 🟡](01-ly-thuyet.md#3-navigator-20--gorouter)

### Bước 1: Prompt cho AI

```text
Tạo GoRouter config cho app Flutter có:
- 3 public routes: /login, /register, /forgot-password
- 3 private routes: /home, /profile, /settings
- Redirect function: nếu chưa login và vào private route → redirect /login
- Dùng isLoggedIn boolean để check auth state
Output: 1 đoạn code GoRouter config hoàn chỉnh.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Redirect loop** | `/login` có nằm trong danh sách bị redirect không? Nếu có → loop! |
| 2 | **Return type** | Redirect function phải return `String?` (path), không gọi `context.go()` |
| 3 | **Null return** | Return `null` khi không cần redirect (cho phép navigation bình thường) |
| 4 | **Login success** | Sau login, có redirect về trang user muốn vào ban đầu không? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Redirect loop — AI quên exclude login route
redirect: (context, state) {
  if (!isLoggedIn) {
    return '/login'; // Cả /login cũng bị redirect → LOOP!
  }
  return null;
},

// ✅ FIX: Check nếu đang ở login thì không redirect
redirect: (context, state) {
  final isOnLoginPage = state.matchedLocation == '/login';
  if (!isLoggedIn && !isOnLoginPage) {
    return '/login';
  }
  if (isLoggedIn && isOnLoginPage) {
    return '/home'; // Đã login rồi thì không cần ở login nữa
  }
  return null;
},
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Nhận ra redirect function cần whitelist public routes
- ✅ Biết redirect trả về `String?`, không navigate trực tiếp
- ✅ Hiểu tầm quan trọng của review checklist khi dùng AI gen config
```
