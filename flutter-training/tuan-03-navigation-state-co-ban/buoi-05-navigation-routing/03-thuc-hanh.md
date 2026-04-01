# Buổi 05: Navigation & Routing — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ React Router / Vue Router**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| Web Router Habit | Flutter Reality | Bài tập liên quan |
|------------------|-----------------|---------------------|
| `navigate('/path')` luôn replace | `context.go()` = replace stack, `context.push()` = add to stack — chọn sai → navigation lỗi | BT1, BT2 |
| Route params qua URL luôn persist | `extra` data **mất khi app restart** — dùng path/query params cho data quan trọng | BT1 |
| Tab = URL change, component unmount | `StatefulShellRoute` giữ state mỗi tab — phải config đúng để có behavior mong muốn | BT2 |
| Back button = `history.back()` tự động | `Navigator.pop()` cần return data nếu muốn — phải await `push()` để nhận result | BT1, BT3 |

---

## BT1 ⭐: Multi-screen App với Navigator 1.0 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_multi_screen_app` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app 3 màn hình với navigation push/pop |

### 📋 Yêu cầu

Xây dựng Flutter app có **3 màn hình** sử dụng Navigator 1.0:

```
HomeScreen ──push──▶ DetailScreen ──push──▶ SettingsScreen
    ◀──pop───            ◀──pop───
```

**Chức năng cụ thể:**

1. **HomeScreen**:
   - Hiển thị danh sách 5 items (dùng `ListView`)
   - Mỗi item có `title` và `subtitle`
   - Tap item → push `DetailScreen`, truyền item data qua constructor

2. **DetailScreen**:
   - Hiển thị thông tin chi tiết của item được truyền vào
   - Có nút "Settings" → push `SettingsScreen`
   - Có nút "Add to Favorites" → pop về HomeScreen với result `'Added <item_name> to favorites'`

3. **SettingsScreen**:
   - Hiển thị vài settings giả (Switch, Slider)
   - Nút "Back to Home" → `Navigator.popUntil()` về HomeScreen

4. **HomeScreen** hiển thị result trả về từ DetailScreen ở một `SnackBar` hoặc `Text` widget

### 🧩 Gợi ý cấu trúc

```dart
// Data model
class Item {
  final int id;
  final String title;
  final String description;
}

// Screens
class HomeScreen extends StatefulWidget { ... }
class DetailScreen extends StatelessWidget {
  final Item item;
  ...
}
class SettingsScreen extends StatelessWidget { ... }
```

### ✅ Tiêu chí hoàn thành

- [ ] App chạy không lỗi
- [ ] Navigate được qua 3 screens (push/pop)
- [ ] Data truyền đúng từ Home → Detail (qua constructor)
- [ ] Result trả về đúng từ Detail → Home (qua `Navigator.pop(context, result)`)
- [ ] `popUntil` từ Settings về Home hoạt động
- [ ] Back button (AppBar + hardware) hoạt động đúng

### 💡 Hint

<details>
<summary>Hint 1: Truyền data qua constructor</summary>

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(item: selectedItem),
  ),
);
```
</details>

<details>
<summary>Hint 2: Nhận result từ pop</summary>

```dart
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (context) => const DetailScreen(item: item)),
);
if (result != null && mounted) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text(result)),
  );
}
```
</details>

<details>
<summary>Hint 3: popUntil về root</summary>

```dart
Navigator.popUntil(context, (route) => route.isFirst);
```
</details>

---

## BT2 ⭐⭐: GoRouter App với Bottom Tabs 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_gorouter_tabs` |
| **Setup** | `flutter pub add go_router` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app với bottom tab navigation dùng GoRouter |

### 📋 Yêu cầu

Xây dựng Flutter app sử dụng **GoRouter** với bottom tab navigation:

```
┌──────────────────────────────────┐
│                                  │
│  Tab content (Home/Search/       │
│  Profile) hoặc sub-screens       │
│                                  │
├──────────────────────────────────┤
│  🏠 Home  │  🔍 Search  │  👤    │
└──────────────────────────────────┘
```

**Chức năng cụ thể:**

1. **3 tabs**: Home, Search, Profile — mỗi tab có nội dung riêng

2. **Home tab**:
   - Hiển thị grid 6 categories (dùng `GridView`)
   - Tap category → push detail screen (vẫn trong tab Home, bottom nav vẫn hiện)
   - Detail screen hiển thị tên category và danh sách items

3. **Search tab**:
   - Có `TextField` để nhập search query
   - Hiển thị kết quả search (giả lập — filter từ list)
   - Tap item → push detail screen

4. **Profile tab**:
   - Hiển thị thông tin user (avatar, name, email)
   - Có nút "Edit Profile" → push edit screen
   - Có nút "Settings" → push settings screen

5. **Mỗi tab giữ navigation state riêng** — chuyển tab rồi quay lại, stack vẫn giữ nguyên

### 🧩 Gợi ý cấu trúc

```
lib/
├── main.dart           ← App entry + GoRouter config
├── router.dart         ← Route definitions (tách riêng)
├── models/
│   └── category.dart
├── screens/
│   ├── home_tab.dart
│   ├── search_tab.dart
│   ├── profile_tab.dart
│   ├── category_detail_screen.dart
│   ├── edit_profile_screen.dart
│   └── settings_screen.dart
└── widgets/
    └── scaffold_with_nav_bar.dart
```

**Router config gợi ý:**

```dart
final router = GoRouter(
  initialLocation: '/home',
  routes: [
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // Branch 0: Home
        StatefulShellBranch(routes: [
          GoRoute(
            path: '/home',
            builder: ...,
            routes: [
              GoRoute(path: 'category/:id', builder: ...),
            ],
          ),
        ]),
        // Branch 1: Search
        StatefulShellBranch(routes: [
          GoRoute(path: '/search', builder: ...),
        ]),
        // Branch 2: Profile
        StatefulShellBranch(routes: [
          GoRoute(
            path: '/profile',
            builder: ...,
            routes: [
              GoRoute(path: 'edit', builder: ...),
              GoRoute(path: 'settings', builder: ...),
            ],
          ),
        ]),
      ],
    ),
  ],
);
```

### ✅ Tiêu chí hoàn thành

- [ ] App chạy không lỗi với GoRouter
- [ ] 3 tabs hoạt động, bottom nav bar cố định
- [ ] Navigate trong mỗi tab (push/pop) không ảnh hưởng bottom nav
- [ ] Chuyển tab giữ navigation state (dùng `StatefulShellRoute`)
- [ ] Path parameters hoạt động (`/home/category/:id`)
- [ ] Mỗi screen có AppBar phù hợp

### 💡 Hint

<details>
<summary>Hint 1: ScaffoldWithNavBar dùng StatefulNavigationShell</summary>

```dart
class ScaffoldWithNavBar extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const ScaffoldWithNavBar({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,  // ← hiển thị nội dung tab hiện tại
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          // goBranch chuyển tab mà giữ state
          navigationShell.goBranch(index, initialLocation: index == navigationShell.currentIndex);
        },
        destinations: const [...],
      ),
    );
  }
}
```
</details>

<details>
<summary>Hint 2: Push trong tab (giữ bottom nav)</summary>

```dart
// Từ HomeTab, push category detail
// Vì '/home/category/:id' là sub-route của '/home'
// → vẫn nằm trong ShellRoute → bottom nav vẫn hiện
context.push('/home/category/${category.id}');
```
</details>

---

## BT3 ⭐⭐⭐: Deep Linking App 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_deep_linking` |
| **Setup** | `flutter pub add go_router` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app e-commerce đơn giản hỗ trợ deep linking |

### 📋 Yêu cầu

Xây dựng Flutter app **e-commerce đơn giản** hỗ trợ deep linking qua GoRouter:

**Deep link URLs cần hỗ trợ:**

| URL | Screen | Mô tả |
|-----|--------|--------|
| `/` | HomeScreen | Trang chủ, danh sách products |
| `/product/:id` | ProductDetailScreen | Chi tiết product |
| `/category/:slug` | CategoryScreen | Products theo category |
| `/cart` | CartScreen | Giỏ hàng |
| `/profile` | ProfileScreen | Thông tin user |

**Chức năng cụ thể:**

1. **HomeScreen**: Grid hiển thị products (ít nhất 8 items). Mỗi item có ảnh placeholder, tên, giá.

2. **ProductDetailScreen** (`/product/:id`):
   - Nhận `id` từ path parameter
   - Hiển thị chi tiết product (tên, giá, mô tả)
   - Nút "Add to Cart"
   - Nút "Related products" → navigate tới category của product

3. **CategoryScreen** (`/category/:slug`):
   - Nhận `slug` từ path parameter (e.g., `electronics`, `clothing`)
   - Filter và hiển thị products thuộc category
   - Tap product → push ProductDetailScreen

4. **CartScreen** (`/cart`):
   - Hiển thị danh sách items trong cart (dùng in-memory list)
   - Tap item → push ProductDetailScreen

5. **Deep linking**:
   - App có thể mở trực tiếp bất kỳ screen nào từ URL
   - Nếu product ID không tồn tại → hiển thị error screen
   - URL bar (trên web) cập nhật khi navigate

6. **Route redirect**:
   - Nếu cart rỗng, `/cart` redirect về `/` với message

### 🧩 Gợi ý cấu trúc

```
lib/
├── main.dart
├── router.dart
├── models/
│   ├── product.dart
│   └── category.dart
├── data/
│   └── sample_data.dart      ← fake data
├── screens/
│   ├── home_screen.dart
│   ├── product_detail_screen.dart
│   ├── category_screen.dart
│   ├── cart_screen.dart
│   ├── profile_screen.dart
│   └── not_found_screen.dart
└── widgets/
    └── product_card.dart
```

**Router config gợi ý:**

```dart
final router = GoRouter(
  initialLocation: '/',
  errorBuilder: (context, state) => NotFoundScreen(uri: state.uri.toString()),
  redirect: (context, state) {
    // Redirect logic nếu cần (e.g., cart rỗng)
    return null;
  },
  routes: [
    ShellRoute(
      builder: (context, state, child) => AppShell(child: child),
      routes: [
        GoRoute(path: '/', name: 'home', builder: ...),
        GoRoute(path: '/product/:id', name: 'product-detail', builder: ...),
        GoRoute(path: '/category/:slug', name: 'category', builder: ...),
        GoRoute(path: '/cart', name: 'cart', builder: ...),
        GoRoute(path: '/profile', name: 'profile', builder: ...),
      ],
    ),
  ],
);
```

### ✅ Tiêu chí hoàn thành

- [ ] App chạy không lỗi
- [ ] Tất cả 5 screens hoạt động với đúng URL path
- [ ] Path parameters parse đúng (`/product/42` → id=42)
- [ ] Navigate giữa screens mượt mà
- [ ] Error handling: product/category không tồn tại → hiển thị NotFoundScreen
- [ ] `errorBuilder` trong GoRouter xử lý route không hợp lệ
- [ ] **Test deep link** (chạy trên web hoặc dùng lệnh adb/simctl):
  - Mở `http://localhost:xxxx/#/product/3` → ProductDetailScreen
  - Mở `http://localhost:xxxx/#/category/electronics` → CategoryScreen

### 💡 Hint

<details>
<summary>Hint 1: Parse path param và tìm product</summary>

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = int.tryParse(state.pathParameters['id'] ?? '');
    final product = sampleProducts.where((p) => p.id == id).firstOrNull;
    if (product == null) {
      return const NotFoundScreen(message: 'Product not found');
    }
    return ProductDetailScreen(product: product);
  },
),
```
</details>

<details>
<summary>Hint 2: Test deep link trên web</summary>

```bash
# Chạy app trên web
flutter run -d chrome

# Mở trực tiếp URL trong browser:
# http://localhost:xxxx/#/product/3
# http://localhost:xxxx/#/category/electronics
```
</details>

<details>
<summary>Hint 3: Test deep link trên Android emulator</summary>

```bash
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://myapp.com/product/3" \
  com.example.myapp
```
</details>

---

## 💬 Câu hỏi thảo luận

### Câu 1: Navigator 1.0 vs GoRouter — Khi nào dùng gì?

Hãy thảo luận các trường hợp sau và quyết định nên dùng Navigator 1.0 hay GoRouter:

| Scenario | Lựa chọn | Lý do |
|----------|----------|-------|
| Prototype app 2-3 screens | ? | ? |
| Production app với bottom tabs | ? | ? |
| Flutter Web app | ? | ? |
| App cần deep linking | ? | ? |
| Widget nội bộ cần modal/dialog | ? | ? |

**Gợi ý suy nghĩ:**
- Navigator 1.0 vẫn hoàn toàn hợp lệ cho `showDialog`, `showModalBottomSheet`
- GoRouter không thay thế 100% Navigator 1.0, mà **bổ sung** cho page-level navigation
- Cân nhắc: team size, app complexity, platform targets (mobile only vs web)

### Câu 2: Khi nào cần Deep Linking?

Thảo luận các tình huống thực tế mà deep linking là **cần thiết** vs **nice-to-have** vs **không cần**:

- App e-commerce (chia sẻ link sản phẩm)
- App chat nội bộ công ty
- App banking
- App tin tức / blog
- App game

**Suy nghĩ thêm:**
- Deep linking có rủi ro bảo mật gì? (e.g., link tới trang nhạy cảm)
- Làm sao handle deep link khi user chưa login?

### Câu 3: Nested Navigation Patterns

Xem xét app có cấu trúc sau:

```
App
├── Auth Flow (Login → Register → Forgot Password)
├── Main Flow
│   ├── Tab: Home
│   │   ├── Feed
│   │   ├── Post Detail
│   │   └── User Profile (from post)
│   ├── Tab: Search
│   │   ├── Search Results
│   │   └── Post Detail (from search)
│   └── Tab: Profile
│       ├── My Profile
│       ├── Edit Profile
│       └── Settings
└── Fullscreen Flow (e.g., Create Post → Preview → Publish)
```

Thảo luận:
1. Phần nào dùng `StatefulShellRoute`? Phần nào dùng route thường?
2. "Post Detail" xuất hiện ở cả tab Home và Search — dùng chung 1 route hay tách riêng?
3. "Fullscreen Flow" (Create Post) nên ở ngoài `ShellRoute` (ẩn bottom nav) hay trong?
4. Auth flow nên handle bằng `redirect` hay route riêng?

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 3:** Focus vào gen config phức tạp và review đúng pattern.

### AI-BT1: Gen GoRouter Config + Auth Guard cho App 5 màn hình ⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** GoRouter config, ShellRoute, redirect guard, path parameters.
- **Task thực tế:** PM giao task "Setup navigation cho app quản lý task: 5 màn hình chính + login, có bottom tab, một số màn yêu cầu auth". Cần routing config hoàn chỉnh, team backend đã có API, cần frontend match route structure.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần setup GoRouter config đầy đủ cho task management app.
Tech stack: Flutter 3.x, go_router ^14.x, riverpod ^2.x.
5 screens chính:
1. /tasks — danh sách tasks (bottom tab 1)
2. /tasks/:id — task detail (push trên tab)
3. /calendar — lịch (bottom tab 2)
4. /profile — cá nhân (bottom tab 3, yêu cầu auth)
5. /settings — cài đặt (push từ profile)
+ /login và /register (public, không có bottom tab)
Constraints:
- StatefulShellRoute cho 3 bottom tabs.
- Auth redirect: /profile và /settings yêu cầu login.
- Redirect function PHẢI exclude /login, /register khỏi check (tránh loop).
- Route names dùng enum TaskAppRoute (không String literal).
- Task detail nhận id parameter, validate là int.
- Error route cho path không hợp lệ.
Output: 1 file router.dart hoàn chỉnh.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file `router.dart` với `GoRouter`, `StatefulShellRoute`, 3 branch (Tasks, Calendar, Profile), route list, `redirect` function, `errorBuilder`.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Redirect function có exclude `/login` và `/register`? (tránh loop) | ☐ |
| 2 | `/profile` và `/settings` có trong danh sách private routes? | ☐ |
| 3 | `StatefulShellRoute` dùng đúng (không phải `ShellRoute` thường)? | ☐ |
| 4 | Task detail parse `:id` có dùng `int.tryParse` (không crash nếu id sai)? | ☐ |
| 5 | Route names dùng enum, không có String literal trùng lặp? | ☐ |
| 6 | Error route có xử lý 404? | ☐ |
| 7 | `flutter analyze` không warning? | ☐ |

**4. Customize:**
Tự thêm: transition animation custom (slide from right cho push, fade cho tab switch). AI chưa làm phần này. Implement `CustomTransitionPage` trong route builder và áp dụng cho task detail route.
