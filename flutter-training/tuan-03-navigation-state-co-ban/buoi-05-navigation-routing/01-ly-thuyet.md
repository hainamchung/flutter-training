# Buổi 05: Navigation & Routing — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 5/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 04 (lý thuyết + ít nhất BT1-BT2)

## 1. Navigation Concept trong Flutter 🔴

### 1.1 Navigation là gì?

Navigation là cách app chuyển đổi giữa các **screen** (hay còn gọi là **page**, **route**). Trong Flutter, mọi screen đều là Widget, và navigation được quản lý theo mô hình **stack** (ngăn xếp).

### 1.2 Stack-based Navigation

Flutter sử dụng **Navigator** — một widget quản lý một stack các **Route** objects:

```
┌─────────────────────┐
│   Settings Screen    │  ← top (đang hiển thị)
├─────────────────────┤
│   Detail Screen      │
├─────────────────────┤
│   Home Screen        │  ← bottom (root)
└─────────────────────┘
     Navigation Stack
```

**Nguyên tắc hoạt động:**

| Hành động | Mô tả | Kết quả |
|-----------|--------|---------|
| **Push** | Thêm screen mới lên đỉnh stack | Hiển thị screen mới |
| **Pop** | Xóa screen ở đỉnh stack | Quay lại screen trước |
| **Replace** | Thay thế screen ở đỉnh stack | Hiển thị screen mới, không thể quay lại |

```
Push (thêm screen):           Pop (xóa screen):

┌──────────┐                  ┌──────────┐
│ Screen C │ ← PUSH           │ Screen C │ ← POP (xóa)
├──────────┤                  ├──────────┤
│ Screen B │                  │ Screen B │ ← hiển thị lại
├──────────┤                  ├──────────┤
│ Screen A │                  │ Screen A │
└──────────┘                  └──────────┘
```

### 1.3 Route vs Screen

- **Route**: Là abstraction đại diện cho entry trong Navigator stack. Trong Flutter, `Route` là class quản lý transition animation, lifecycle.
- **Screen/Page**: Là Widget thực tế được render. Route "chứa" screen.
- **MaterialPageRoute**: Route implementation phổ biến nhất, cung cấp material design transition.

> ⚠️ **FE Trap:** FE dev quen URL-based history (mỗi route = 1 URL, back = history.back()). Flutter Navigator dùng **stack** (push/pop). Không có URL bar, không có browser history. `context.go()` = replace stack, `context.push()` = add to stack. Đây là khác biệt tư duy lớn nhất.

---

## 2. Navigator 1.0 🟡

Navigator 1.0 là API **imperative** (mệnh lệnh) — bạn gọi trực tiếp `push()`, `pop()` để điều khiển navigation.

### 2.1 Navigator.push() và MaterialPageRoute

Cách đơn giản nhất để navigate sang screen mới:

```dart
// Từ HomeScreen, navigate sang DetailScreen
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => const DetailScreen(),
  ),
);
```

**`MaterialPageRoute`** cung cấp:
- Platform-aware transition animation (slide từ phải trên Android, slide lên trên iOS)
- Tự động thêm nút Back trên AppBar

### 2.2 Navigator.pop()

Quay lại screen trước:

```dart
// Trong DetailScreen, quay lại HomeScreen
Navigator.pop(context);
```

Hoặc user nhấn nút **Back** trên AppBar / hardware back button — Flutter tự gọi `pop()`.

### 2.3 Named Routes

Thay vì tạo `MaterialPageRoute` mỗi lần, bạn có thể đăng ký **named routes** trong `MaterialApp`:

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const HomeScreen(),
    '/detail': (context) => const DetailScreen(),
    '/settings': (context) => const SettingsScreen(),
  },
);
```

Navigate bằng tên:

```dart
Navigator.pushNamed(context, '/detail');
```

### 2.4 onGenerateRoute — Dynamic Route Handling

Khi cần xử lý route động (ví dụ: parse path parameters):

```dart
MaterialApp(
  onGenerateRoute: (RouteSettings settings) {
    // settings.name = '/product/123'
    final uri = Uri.parse(settings.name!);

    if (uri.pathSegments.length == 2 && uri.pathSegments[0] == 'product') {
      final id = uri.pathSegments[1];
      return MaterialPageRoute(
        builder: (context) => ProductScreen(productId: id),
      );
    }

    // Fallback: 404 page
    return MaterialPageRoute(
      builder: (context) => const NotFoundScreen(),
    );
  },
);
```

### 2.5 Hạn chế của Navigator 1.0

| Hạn chế | Mô tả |
|---------|--------|
| **Imperative API** | Khó quản lý navigation state phức tạp, không dễ predict stack state |
| **Không sync URL (web)** | Trên Flutter Web, URL không tự cập nhật khi navigate |
| **Deep linking khó** | Phải tự handle parsing URL thành route |
| **Nested navigation phức tạp** | Quản lý nhiều Navigator lồng nhau rất rườm rà |
| **Không type-safe** | Named routes dùng String, dễ typo |

> 🔗 **FE Bridge:** `Navigator.push()` ≈ `history.push()`, `Navigator.pop()` ≈ `history.back()` — nhưng **khác ở**: Flutter push tạo **route mới trên stack** (có animation transition), web chỉ thay đổi URL. Pop cũng có animation ngược lại — web không có built-in page transition.

---

## 3. Navigator 2.0 / GoRouter 🟡

### 3.1 Tại sao cần GoRouter?

**Navigator 2.0** (Router API) được Flutter team giới thiệu để giải quyết hạn chế của 1.0, nhưng API rất verbose và phức tạp. **GoRouter** là package chính thức (maintained by Flutter team) wrap Navigator 2.0 với API đơn giản hơn nhiều.

```
Navigator 1.0          Navigator 2.0 (Raw)     GoRouter
─────────────          ───────────────────      ────────
Imperative             Declarative              Declarative
Đơn giản               Rất phức tạp             Đơn giản
Không URL sync         URL sync                 URL sync
Khó deep link          Deep link support        Deep link ngay
                       ~200 lines boilerplate   ~20 lines setup
```

### 3.2 Cài đặt GoRouter

Thêm vào `pubspec.yaml`:

```yaml
dependencies:
  go_router: ^14.2.0  # kiểm tra phiên bản mới nhất trên pub.dev
```

Chạy:

```bash
flutter pub get
```

### 3.3 GoRouter Setup cơ bản

```dart
import 'package:go_router/go_router.dart';

// 1. Định nghĩa router config
final GoRouter router = GoRouter(
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
        final id = state.pathParameters['id']!;
        return DetailScreen(id: id);
      },
    ),
    GoRoute(
      path: '/settings',
      name: 'settings',
      builder: (context, state) => const SettingsScreen(),
    ),
  ],
);

// 2. Sử dụng trong MaterialApp
MaterialApp.router(
  routerConfig: router,
);
```

### 3.4 Navigation với GoRouter

| Method | Mô tả | Ví dụ |
|--------|--------|-------|
| `context.go('/path')` | Navigate tới path, **replace** stack với path mới | `context.go('/detail/123')` |
| `context.push('/path')` | **Push** screen mới lên stack (có thể pop back) | `context.push('/detail/123')` |
| `context.pop()` | Pop screen hiện tại | `context.pop()` |
| `context.goNamed('name')` | Navigate bằng route name | `context.goNamed('detail', pathParameters: {'id': '123'})` |

**Khác biệt quan trọng giữa `go()` và `push()`:**

```
context.go('/detail/123'):      context.push('/detail/123'):
─ Thay đổi toàn bộ stack        ─ Thêm screen lên stack hiện tại
─ URL bar cập nhật               ─ URL bar cập nhật
─ Không nhất thiết pop được      ─ Luôn pop được về screen trước
─ Giống "navigate" trên web      ─ Giống "push" truyền thống
```

> 🔗 **FE Bridge:** GoRouter ≈ React Router / Vue Router — mapping gần **1:1**: `GoRoute` ≈ `Route`, `path` ≈ `path`, `builder` ≈ `element/component`. Nhưng **khác ở**: GoRouter vẫn quản lý stack underneath, web router chỉ quản lý URL matching. `go()` = replace toàn bộ stack, `push()` = thêm lên stack.

### 3.5 ShellRoute — Persistent UI

`ShellRoute` dùng khi bạn muốn giữ một phần UI **cố định** (ví dụ: bottom navigation bar) trong khi nội dung bên trong thay đổi:

```dart
ShellRoute(
  builder: (context, state, child) {
    return ScaffoldWithNavBar(child: child);
  },
  routes: [
    GoRoute(
      path: '/home',
      builder: (context, state) => const HomeTab(),
    ),
    GoRoute(
      path: '/search',
      builder: (context, state) => const SearchTab(),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileTab(),
    ),
  ],
);
```

```
┌──────────────────────────┐
│                          │
│    child (HomeTab /      │   ← thay đổi theo route
│    SearchTab / ...)      │
│                          │
├──────────────────────────┤
│  🏠 Home  🔍 Search  👤  │   ← ShellRoute giữ cố định
└──────────────────────────┘
```

> 🔗 **FE Bridge:** `ShellRoute` ≈ React Router `<Outlet>` + layout component — giữ UI persistent (bottom nav, sidebar) khi navigate giữa child routes. Nhưng **khác ở**: Flutter `StatefulShellRoute` **giữ state** mỗi tab branch — React `<Outlet>` unmount component khi switch tab trừ khi tự implement caching.

---

> 💼 **Gặp trong dự án:** Setup routing cho app 10+ screens, bottom tab navigation giữ state, auth guard redirect chưa login về login screen, deep link từ push notification
> 🤖 **Keywords bắt buộc trong prompt:** `GoRouter`, `ShellRoute`, `StatefulShellRoute`, `redirect guard`, `go vs push`, `pathParameters`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **App mới:** PM giao setup navigation cho app e-commerce: Home, Search, Cart, Profile (bottom tabs) + Product Detail, Checkout flow (fullscreen, ẩn bottom nav)
- **Auth guard:** Một số screen (Cart, Profile, Checkout) yêu cầu login — nếu chưa login redirect về /login, sau login quay lại screen ban đầu
- **Deep link:** Push notification gửi link `/product/123` — app phải mở đúng product detail

**Tại sao cần các keyword trên:**
- **`GoRouter`** (không phải Navigator 1.0) — AI hay gen code Navigator.push kiểu cũ nếu không nói rõ
- **`ShellRoute / StatefulShellRoute`** — để giữ bottom nav bar cố định, AI hay tạo bottom nav trong mỗi screen (SAI)
- **`redirect guard`** — AI cần gen redirect function kiểm tra auth state, tránh redirect loop
- **`go vs push`** — AI phải dùng `go` cho tab navigation (replace stack), `push` cho detail (thêm vào stack)

**Prompt mẫu — Full GoRouter config:**
```text
Tôi cần setup GoRouter config đầy đủ cho e-commerce app.
Tech stack: Flutter 3.x, go_router ^14.x, flutter_riverpod ^2.x.
Screens: Home, Search, Cart, Profile (bottom tabs), ProductDetail (/product/:id), Checkout flow (3 bước), Login, Register.
Constraints:
- StatefulShellRoute cho 4 bottom tabs (giữ state mỗi tab).
- ProductDetail: push trên tab hiện tại (giữ bottom nav).
- Checkout flow: ngoài ShellRoute (ẩn bottom nav), 3 screens linearly: Shipping → Payment → Confirmation.
- Auth redirect: Cart, Profile, Checkout yêu cầu login → redirect /login?from=[current_path] → sau login go(from).
- Route names: dùng enum RouteName thay vì String literals.
- Error page: 404 route.
Output: 1 file router.dart hoàn chỉnh với tất cả routes + redirect logic.
```

**Expected Output:** AI gen file `router.dart` với `GoRouter` config, `StatefulShellRoute`, route list, redirect function, error builder.

⚠️ **Giới hạn AI hay mắc:** AI hay tạo redirect loop (redirect /login → check auth → redirect /login lại). AI cũng hay quên `StatefulShellRoute` và dùng `ShellRoute` thường (mất state khi chuyển tab).

</details>

---

## 4. Truyền Data giữa Screens 🔴

### 4.1 Qua Constructor Arguments (Navigator 1.0)

Cách trực tiếp nhất — truyền data vào constructor của screen:

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(
      product: myProduct,  // truyền object
    ),
  ),
);
```

```dart
class DetailScreen extends StatelessWidget {
  final Product product;
  const DetailScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(product.name)),
      body: Text(product.description),
    );
  }
}
```

### 4.2 Qua Path / Query Parameters (GoRouter)

**Path parameters** — dùng cho required identifiers:

```dart
// Route definition
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return ProductScreen(productId: id);
  },
),

// Navigate
context.go('/product/42');
```

**Query parameters** — dùng cho optional filters:

```dart
// Navigate với query params
context.go('/search?query=flutter&sort=recent');

// Đọc trong builder
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['query'] ?? '';
    final sort = state.uri.queryParameters['sort'] ?? 'default';
    return SearchScreen(query: query, sort: sort);
  },
),
```

### 4.3 Return Data từ Pop

Screen B có thể trả data về cho screen A khi pop:

**Navigator 1.0:**

```dart
// Screen A: push và chờ kết quả
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(
    builder: (context) => const SelectionScreen(),
  ),
);
// result = 'Option A' hoặc null (nếu user nhấn back)
if (result != null) {
  // xử lý result
}

// Screen B: pop với data
Navigator.pop(context, 'Option A');
```

**GoRouter:**

```dart
// Screen A: push và chờ kết quả
final result = await context.push<String>('/selection');

// Screen B: pop với data
context.pop('Option A');
```

> 🔗 **FE Bridge:** `pathParameters` ≈ `useParams()`, `queryParameters` ≈ `useSearchParams()`, `extra` ≈ `location.state` — mapping gần 1:1. Nhưng `extra` trong GoRouter **mất khi refresh** (mobile ít vấn đề vì không có refresh, nhưng web cần lưu ý).

---

## 5. Nested Navigation 🟡

### 5.1 Khái niệm

Nested navigation xảy ra khi app có **nhiều navigation stack đồng thời**. Trường hợp phổ biến nhất: app có bottom tab bar, mỗi tab có history riêng.

```
┌──────────────────────────────────┐
│  Tab 1 Stack    Tab 2 Stack      │
│  ┌──────────┐  ┌──────────┐     │
│  │ Screen C │  │ Screen B │     │
│  ├──────────┤  ├──────────┤     │
│  │ Screen B │  │ Screen A │     │
│  ├──────────┤  └──────────┘     │
│  │ Screen A │                    │
│  └──────────┘                    │
├──────────────────────────────────┤
│   Tab 1 (●)  │  Tab 2  │ Tab 3  │
└──────────────────────────────────┘
```

### 5.2 Cách cũ: BottomNavigationBar + IndexedStack

```dart
class MainScreen extends StatefulWidget {
  // ...
}

class _MainScreenState extends State<MainScreen> {
  int _currentIndex = 0;

  final List<Widget> _tabs = [
    Navigator(  // Mỗi tab có Navigator riêng
      onGenerateRoute: (settings) => MaterialPageRoute(
        builder: (context) => const HomeTab(),
      ),
    ),
    Navigator(
      onGenerateRoute: (settings) => MaterialPageRoute(
        builder: (context) => const SearchTab(),
      ),
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: IndexedStack(
        index: _currentIndex,
        children: _tabs,
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
        ],
      ),
    );
  }
}
```

> ⚠️ Cách này hoạt động nhưng **phức tạp** khi cần deep link hoặc URL sync.

### 5.3 GoRouter: StatefulShellRoute

GoRouter cung cấp `StatefulShellRoute` — giải pháp tốt hơn cho nested navigation với tabs:

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return ScaffoldWithNavBar(navigationShell: navigationShell);
  },
  branches: [
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeTab(),
          routes: [
            GoRoute(
              path: 'detail/:id', // /home/detail/123
              builder: (context, state) {
                final id = state.pathParameters['id']!;
                return DetailScreen(id: id);
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
          builder: (context, state) => const SearchTab(),
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileTab(),
        ),
      ],
    ),
  ],
),
```

**Ưu điểm của `StatefulShellRoute`:**
- Mỗi branch (tab) giữ **navigation state riêng**
- Deep linking hoạt động đúng — `/home/detail/123` sẽ mở tab Home và push DetailScreen
- URL sync trên web
- Code gọn gàng hơn nhiều so với cách thủ công

### 5.4 Navigation Patterns: Drawer, BottomNav, TabBar

Các app thực tế thường dùng nhiều pattern navigation kết hợp. GoRouter hỗ trợ tất cả thông qua `StatefulShellRoute`.

**BottomNavigationBar** — phổ biến nhất, đã demo ở trên (5.3).

**Drawer Navigation:**

```dart
class ScaffoldWithDrawer extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const ScaffoldWithDrawer({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My App')),
      drawer: Drawer(
        child: ListView(
          children: [
            const DrawerHeader(child: Text('Menu')),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('Home'),
              selected: navigationShell.currentIndex == 0,
              onTap: () {
                navigationShell.goBranch(0);
                Navigator.pop(context); // close drawer
              },
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('Settings'),
              selected: navigationShell.currentIndex == 1,
              onTap: () {
                navigationShell.goBranch(1);
                Navigator.pop(context);
              },
            ),
          ],
        ),
      ),
      body: navigationShell,
    );
  }
}
```

**TabBar + TabBarView:**

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          bottom: TabBar(
            onTap: (index) => navigationShell.goBranch(index),
            tabs: const [
              Tab(icon: Icon(Icons.home), text: 'Home'),
              Tab(icon: Icon(Icons.explore), text: 'Explore'),
              Tab(icon: Icon(Icons.person), text: 'Profile'),
            ],
          ),
        ),
        body: navigationShell,
      ),
    );
  },
  branches: [ /* ... same as BottomNav branches ... */ ],
)
```

> 💡 **Điểm chung**: Cả 3 pattern đều dùng `StatefulShellRoute` + `navigationShell.goBranch(index)` để chuyển tab. Chỉ khác phần UI wrapper (BottomNav / Drawer / TabBar).

> 📖 **Đọc thêm**: [GoRouter StatefulShellRoute docs](https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute-class.html)

---

## 6. Deep Linking 🟢

### 6.1 Deep Linking là gì?

Deep linking cho phép mở app và navigate thẳng đến một screen cụ thể thông qua URL:

```
myapp://product/42        → mở app → hiển thị ProductScreen(id: 42)
https://myapp.com/profile → mở app → hiển thị ProfileScreen
```

### 6.2 Cách hoạt động trên mỗi platform

**iOS — Universal Links:**

```
┌──────────────────────────────────────────┐
│ 1. User tap link: https://myapp.com/x    │
│ 2. iOS kiểm tra apple-app-site-assoc.    │
│ 3. Nếu match → mở app với URL            │
│ 4. App nhận URL → GoRouter parse → route  │
└──────────────────────────────────────────┘
```

Cấu hình trong `ios/Runner/Runner.entitlements`:

```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:myapp.com</string>
</array>
```

**Android — App Links:**

Cấu hình trong `android/app/src/main/AndroidManifest.xml`:

```xml
<intent-filter android:autoVerify="true">
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data
    android:scheme="https"
    android:host="myapp.com"
    android:pathPrefix="/" />
</intent-filter>
```

### 6.3 GoRouter xử lý Deep Links

GoRouter **tự động** parse URL thành route. Bạn chỉ cần định nghĩa routes đúng:

```dart
GoRouter(
  routes: [
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductScreen(productId: id);
      },
    ),
  ],
);
```

Khi app nhận deep link `https://myapp.com/product/42`:
1. OS chuyển URL cho app
2. GoRouter parse `/product/42`
3. Match route `/product/:id` với `id = '42'`
4. Build `ProductScreen(productId: '42')`

### 6.4 Test Deep Linking

```bash
# Android
adb shell am start -a android.intent.action.VIEW \
  -d "https://myapp.com/product/42" com.example.myapp

# iOS (Simulator)
xcrun simctl openurl booted "https://myapp.com/product/42"
```

> 🔗 **FE Bridge:** Deep linking trên web = mặc định (mỗi URL là deep link). Trên mobile = **phải config riêng** (AndroidManifest, Info.plist, apple-app-site-association). Đây là concept FE dev thường skip vì web "miễn phí" — mobile thì không.

---

> 💼 **Gặp trong dự án:** Push notification mở đúng screen, marketing link redirect vào app, share product link giữa users, Universal Links / App Links setup
> 🤖 **Keywords bắt buộc trong prompt:** `deep linking`, `Universal Links iOS`, `App Links Android`, `GoRouter path parameters`, `onGenerateRoute fallback`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Push notification:** Firebase Cloud Messaging gửi payload `{"url": "/order/456"}` → app mở OrderDetail
- **Marketing:** QR code in trên packaging chứa `https://myapp.com/product/789` → app nếu đã cài thì mở native, chưa cài thì mở web
- **Share link:** User share link sản phẩm qua Zalo/Messenger → người nhận click → mở app đúng product

**Tại sao cần các keyword trên:**
- **`Universal Links iOS`** — cần file `apple-app-site-association` trên server, AI hay quên
- **`App Links Android`** — cần `assetlinks.json` + `autoVerify` trong AndroidManifest, AI có thể gen sai format
- **`GoRouter path parameters`** — `/product/:id` phải parse id đúng type (int/String)
- **`onGenerateRoute fallback`** — khi deep link URL không match route nào, cần error page

**Prompt mẫu — Deep link setup:**
```text
Tôi cần setup deep linking cho Flutter app dùng GoRouter.
Context: app e-commerce, domain myapp.com, scheme myapp://.
Tech stack: Flutter 3.x, go_router ^14.x.
Cần config cho cả:
1. iOS Universal Links (apple-app-site-association file).
2. Android App Links (assetlinks.json + AndroidManifest intent-filter).
3. GoRouter routes handle: /product/:id, /order/:id, /promo/:code.
4. Fallback: URL không hợp lệ → redirect Home + show snackbar lỗi.
5. Test commands cho emulator (adb shell am start, xcrun simctl openurl).
Output: GoRouter config + iOS/Android config files + test script.
```

**Expected Output:** AI gen router config + `apple-app-site-association` JSON + `AndroidManifest.xml` snippet + `assetlinks.json` + test commands.

⚠️ **Giới hạn AI hay mắc:** AI hay gen `apple-app-site-association` sai format (phải là JSON không có extension .json). AI cũng hay quên `autoVerify="true"` trong AndroidManifest — thiếu cái này App Links không hoạt động.

</details>

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

| Practice | Mô tả |
|----------|--------|
| **Dùng GoRouter cho project mới** | Navigator 1.0 chỉ phù hợp cho app đơn giản hoặc prototype |
| **Centralize route definitions** | Định nghĩa tất cả routes tại một file (e.g., `router.dart`) |
| **Dùng named routes** | `context.goNamed('detail')` thay vì `context.go('/detail')` — tránh typo, dễ refactor |
| **Type-safe route params** | Validate và parse params ngay trong route builder |
| **Handle 404** | Luôn có `errorBuilder` trong GoRouter |
| **Redirect cho auth** | Dùng `redirect` trong GoRouter để protect routes cần đăng nhập |

#### Auth Guard chi tiết với GoRouter.redirect

Redirect cho phép bảo vệ route — chuyển user chưa đăng nhập về trang login, và ngăn user đã login truy cập lại trang login:

```dart
// auth_provider.dart — quản lý trạng thái đăng nhập
class AuthService extends ChangeNotifier {
  bool _isLoggedIn = false;
  bool get isLoggedIn => _isLoggedIn;

  void login() { _isLoggedIn = true; notifyListeners(); }
  void logout() { _isLoggedIn = false; notifyListeners(); }
}

final authService = AuthService();

// router.dart
final GoRouter router = GoRouter(
  // refreshListenable: re-evaluate redirect khi auth state thay đổi
  refreshListenable: authService,

  redirect: (context, state) {
    final isLoggedIn = authService.isLoggedIn;
    final isLoginPage = state.matchedLocation == '/login';

    // Chưa đăng nhập + không ở trang login → redirect về login
    if (!isLoggedIn && !isLoginPage) return '/login';

    // Đã đăng nhập + đang ở trang login → redirect về home
    if (isLoggedIn && isLoginPage) return '/';

    // Không cần redirect
    return null;
  },

  errorBuilder: (context, state) => const NotFoundScreen(),
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
    ),
  ],
);
```

> 💡 **`refreshListenable`** rất quan trọng — nếu không có, redirect chỉ chạy khi navigate. Với `refreshListenable`, redirect tự chạy lại mỗi khi `authService` gọi `notifyListeners()` (ví dụ sau logout).

> 📖 **Đọc thêm**: [GoRouter Redirection](https://pub.dev/documentation/go_router/latest/topics/Redirection-topic.html)
```

### ❌ Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|-----|-------------|----------|
| `Navigator operation requested with a context that does not include a Navigator` | Dùng context của `MaterialApp` thay vì context bên trong | Dùng `Builder` hoặc context từ widget con |
| `context.go()` không pop được | `go()` replace stack, không push | Dùng `context.push()` nếu muốn pop back |
| Bottom nav không giữ state khi chuyển tab | Dùng `ShellRoute` thay vì `StatefulShellRoute` | Chuyển sang `StatefulShellRoute.indexedStack` |
| Deep link không hoạt động | Chưa cấu hình platform-specific (entitlements / manifest) | Kiểm tra lại iOS/Android config |
| Duplicate `GlobalKey` khi nested nav | Nhiều `Navigator` dùng chung key | Mỗi Navigator cần key riêng |

---

> 💼 **Gặp trong dự án:** Route guard redirect loop, back button xử lý sai trên Android, mất state khi chuyển tab, route name collision khi team lớn
> 🤖 **Keywords bắt buộc trong prompt:** `redirect guard no loop`, `WillPopScope/PopScope`, `route naming convention`, `navigation state restoration`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Redirect loop:** Login screen redirect về /home → /home check auth failed → redirect /login → loop vô hạn
- **Android back button:** User nhấn back trên checkout flow → confirm "hủy đơn hàng?" thay vì thoát ngay
- **Tab state:** User đang xem ProductDetail trong tab Search → chuyển sang Home → quay lại Search → mất ProductDetail

**Tại sao cần các keyword trên:**
- **`redirect guard no loop`** — AI phải biết exclude /login khỏi redirect check
- **`WillPopScope/PopScope`** — xử lý back button trên Android, Flutter 3.16+ dùng PopScope thay WillPopScope
- **`route naming convention`** — team lớn cần enum hoặc constants để tránh collision
- **`navigation state restoration`** — StatefulShellRoute giữ state, nhưng deep state cần manual restore

**Prompt mẫu — Auth redirect guard an toàn:**
```text
Tôi cần implement auth redirect guard cho GoRouter KHÔNG bị redirect loop.
Context: app có public routes (Login, Register, Home) và private routes (Profile, Cart, Checkout).
Tech stack: Flutter 3.x, go_router ^14.x, riverpod ^2.x.
Constraints:
- Redirect function kiểm tra auth state từ Riverpod provider.
- Public routes (Login, Register, ForgotPassword) KHÔNG redirect — tránh loop.
- Private routes: nếu chưa login → redirect /login?redirect=[current_path].
- Sau login thành công: check redirect param → go(redirect) hoặc go(/).
- Nếu token expired giữa chừng → redirect /login, clear auth state.
Output: redirect function + GoRouter config + AuthNotifier.
```

**Expected Output:** AI gen `redirect` function với whitelist check, `AuthNotifier` class, và GoRouter config kết nối cả hai.

⚠️ **Giới hạn AI hay mắc:** AI hay quên whitelist login route trong redirect → gây loop. AI cũng hay dùng `context.go('/login')` trong redirect thay vì return path String (GoRouter redirect trả về String?, không navigate).

</details>

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Flutter Mindset | Tại sao khác |
|---|-------------------|-----------------|--------------|
| 1 | Mỗi route = 1 URL, navigation = thay đổi URL | Navigation = push/pop trên stack, có thể không có URL | Mobile không có URL bar — stack-based thay vì URL-based |
| 2 | Back button = browser history | Back button = pop từ Navigator stack (có thể customize) | Mobile back = pop stack, không phải history.back() |
| 3 | Route state sống trong URL (query params) | Route state sống trong stack entry hoặc `extra` object | Mobile state persistence khác web fundamentally |
| 4 | Page transition = CSS animation (optional) | Page transition = built-in MaterialPageRoute animation | Flutter có transition mặc định, web thì không |
| 5 | Tab switch = URL change, component remount | Tab switch = Navigator branch switch, state có thể persist | StatefulShellRoute giữ state mỗi tab branch |

Nếu bạn đến từ React hoặc Vue, bảng so sánh này giúp map kiến thức cũ sang Flutter:

| Concept | React / Vue | Flutter |
|---------|-------------|---------|
| Route definition | `<Route path="/detail" component={Detail} />` | `GoRoute(path: '/detail', builder: ...)` |
| Navigation | `navigate('/detail')` / `router.push('/detail')` | `context.go('/detail')` hoặc `context.push('/detail')` |
| Go back | `navigate(-1)` / `router.back()` | `context.pop()` / `Navigator.pop(context)` |
| URL parameters | `useParams()` / `$route.params` | `state.pathParameters['id']` |
| Query string | `useSearchParams()` / `$route.query` | `state.uri.queryParameters['q']` |
| Nested routes | Nested `<Route>` / `children: []` | `routes: []` trong GoRoute (sub-routes) |
| Layout wrapper | `<Outlet />` trong Layout | `ShellRoute` với `child` parameter |
| Tab navigation | Không built-in (thường custom) | `StatefulShellRoute` + `BottomNavigationBar` |
| Deep linking | Mặc định trên web (URL-based) | Cần config platform (iOS/Android) |
| Route guard | Route middleware / `beforeEach` | `redirect` trong GoRouter |
| History model | **History stack** (browser-managed) | **Navigator stack** (app-managed) |

**Điểm khác biệt quan trọng:**

1. **Web vs Mobile mindset**: Trên web, URL là primary — user có thể type URL. Trên mobile, URL ẩn — navigation chủ yếu từ user action trong app. GoRouter bridge cả hai.

2. **`go()` vs `push()`**: Giống `navigate()` (replace history) vs `history.push()` trong React Router. Dùng `go()` cho top-level navigation, `push()` cho drill-down.

3. **Persistent tabs**: Trên web, tabs thường là riêng page. Trên mobile, tabs cần giữ scroll position, state — đó là lý do có `StatefulShellRoute`.

---

## 9. Tổng kết

### ✅ Checklist kiến thức buổi 5

Sau buổi này, hãy tự kiểm tra:

- [ ] Hiểu stack-based navigation model (push/pop)
- [ ] Biết dùng `Navigator.push()`, `Navigator.pop()`, named routes
- [ ] Biết hạn chế của Navigator 1.0
- [ ] Setup được GoRouter trong project
- [ ] Phân biệt `context.go()` và `context.push()`
- [ ] Truyền data qua constructor, path params, query params
- [ ] Return data từ screen khi pop
- [ ] Dùng `ShellRoute` cho bottom navigation
- [ ] Dùng `StatefulShellRoute` cho nested navigation với state
- [ ] Hiểu deep linking concept và cách GoRouter hỗ trợ
- [ ] Biết cấu hình cơ bản deep link cho iOS/Android

### 🔜 Buổi tiếp theo

**Buổi 06: State Management cơ bản** — Sau khi biết navigate giữa screens, chúng ta sẽ học cách quản lý và chia sẻ state (dữ liệu) giữa các widget.
