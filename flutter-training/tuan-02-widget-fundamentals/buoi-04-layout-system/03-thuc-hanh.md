# Buổi 04: Layout System — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Hướng dẫn:** Tạo Flutter project mới cho mỗi bài tập hoặc dùng chung project và thay `main.dart`. Mỗi bài tập là một Flutter app hoàn chỉnh.

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ CSS Layout**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| CSS Habit | Flutter Reality | Bài tập liên quan |
|-----------|-----------------|---------------------|
| `width: 100%` để fill parent | Dùng `double.infinity` hoặc `Expanded`, KHÔNG có `%` units | BT1, BT2 |
| `margin: auto` để center | Dùng `Center()` widget hoặc `MainAxisAlignment.center` | BT1, BT2 |
| `display: flex; flex-direction: column` | `Column()` — nhưng truyền `mainAxisSize` nếu không muốn expand max | BT2, BT3 |
| Child tự quyết size | Parent gửi constraints → child chọn size trong constraints | Tất cả |
| `overflow: scroll` để scroll | Wrap trong `SingleChildScrollView` — Column/Row **không tự scroll** | BT2, BT3 |

---

## BT1 ⭐: Login Form UI 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_login_form` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Màn hình Login với email/password fields |

### Yêu cầu
Xây dựng màn hình Login cơ bản sử dụng Column, TextField, ElevatedButton, và padding.

### Mockup

```
╔══════════════════════════════════╗
║                                  ║
║         🔐                       ║
║     Welcome Back                 ║
║   Đăng nhập để tiếp tục         ║
║                                  ║
║  ┌────────────────────────────┐  ║
║  │ 📧 Email                   │  ║
║  └────────────────────────────┘  ║
║                                  ║
║  ┌────────────────────────────┐  ║
║  │ 🔒 Mật khẩu               │  ║
║  └────────────────────────────┘  ║
║                                  ║
║           Quên mật khẩu?         ║
║                                  ║
║  ┌────────────────────────────┐  ║
║  │       ĐĂNG NHẬP            │  ║
║  └────────────────────────────┘  ║
║                                  ║
║  ──────── Hoặc ────────         ║
║                                  ║
║  ┌────────────────────────────┐  ║
║  │  G  Đăng nhập với Google   │  ║
║  └────────────────────────────┘  ║
║                                  ║
║    Chưa có tài khoản? Đăng ký   ║
║                                  ║
╚══════════════════════════════════╝
```

### Hướng dẫn từng bước

1. **Scaffold** với `backgroundColor: Colors.white`
2. **SingleChildScrollView** bọc toàn bộ (để scroll khi bàn phím hiện)
3. **Padding** `EdgeInsets.symmetric(horizontal: 24)`
4. **Column** chứa tất cả — `crossAxisAlignment: CrossAxisAlignment.stretch`
5. **SizedBox(height: 80)** — spacing phía trên
6. **Icon** khóa + Text tiêu đề + Text phụ đề — căn giữa
7. **TextField** cho Email — `InputDecoration` với `prefixIcon`, `border: OutlineInputBorder()`
8. **TextField** cho Password — thêm `obscureText: true`
9. **Align** `Alignment.centerRight` cho "Quên mật khẩu?" link
10. **ElevatedButton** "Đăng nhập" — `minimumSize: Size(double.infinity, 50)`
11. **Row** cho divider: `Expanded(child: Divider())` + Text "Hoặc" + `Expanded(child: Divider())`
12. **OutlinedButton** "Đăng nhập với Google"
13. **Row** cho "Chưa có tài khoản?" + TextButton "Đăng ký" — `MainAxisAlignment.center`

### Gợi ý code khung

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Login Form',
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
      ),
      home: const LoginScreen(),
    );
  }
}

class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.symmetric(horizontal: 24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const SizedBox(height: 80),

              // TODO: Icon + Title + Subtitle (căn giữa)

              const SizedBox(height: 40),

              // TODO: Email TextField

              const SizedBox(height: 16),

              // TODO: Password TextField

              const SizedBox(height: 8),

              // TODO: "Quên mật khẩu?" link (Align right)

              const SizedBox(height: 24),

              // TODO: Đăng nhập button (full width)

              const SizedBox(height: 24),

              // TODO: Divider row "Hoặc"

              const SizedBox(height: 24),

              // TODO: Google sign-in button (OutlinedButton)

              const SizedBox(height: 24),

              // TODO: "Chưa có tài khoản? Đăng ký" row
            ],
          ),
        ),
      ),
    );
  }
}
```

### Widgets cần dùng

| Widget | Mục đích |
|--------|----------|
| `Scaffold` | Cấu trúc trang |
| `SafeArea` | Tránh notch/status bar |
| `SingleChildScrollView` | Scroll khi bàn phím hiện |
| `Column` | Sắp xếp dọc |
| `SizedBox` | Spacing |
| `Padding` | Khoảng cách ngoài |
| `TextField` | Input fields |
| `ElevatedButton` | Nút chính |
| `OutlinedButton` | Nút phụ |
| `Align` | Căn "Quên mật khẩu?" sang phải |
| `Row` + `Expanded` | Divider "Hoặc" |

### Tiêu chí hoàn thành

- [ ] Có 2 TextField (Email + Password)
- [ ] Password field có `obscureText: true`
- [ ] Nút Đăng nhập full width
- [ ] Scroll được khi bàn phím hiện lên
- [ ] Spacing hợp lý giữa các element
- [ ] Có link "Quên mật khẩu?" căn phải
- [ ] Có divider "Hoặc" với đường kẻ 2 bên

---

## BT2 ⭐⭐: Dashboard Layout 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_dashboard` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Dashboard với stat cards và activity list |

### Yêu cầu
Xây dựng màn hình Dashboard với AppBar, GridView cho các stat cards, và ListView cho recent items.

### Mockup

```
╔══════════════════════════════════╗
║  Dashboard            🔔  👤     ║
╠══════════════════════════════════╣
║                                  ║
║  Xin chào, Nguyễn Văn A 👋     ║
║                                  ║
║  ┌──────────┐  ┌──────────┐     ║
║  │ 📊 125   │  │ 💰 $2.4K │     ║
║  │ Orders   │  │ Revenue  │     ║
║  └──────────┘  └──────────┘     ║
║  ┌──────────┐  ┌──────────┐     ║
║  │ 👥 1,240 │  │ ⭐ 4.8   │     ║
║  │ Users    │  │ Rating   │     ║
║  └──────────┘  └──────────┘     ║
║                                  ║
║  Recent Activity                 ║
║  ┌────────────────────────────┐  ║
║  │ 🟢 Order #1234 completed  │  ║
║  │   2 phút trước             │  ║
║  ├────────────────────────────┤  ║
║  │ 🔵 New user registered    │  ║
║  │   15 phút trước            │  ║
║  ├────────────────────────────┤  ║
║  │ 🟡 Payment received       │  ║
║  │   1 giờ trước              │  ║
║  ├────────────────────────────┤  ║
║  │ ...                        │  ║
║  └────────────────────────────┘  ║
╚══════════════════════════════════╝
```

### Hướng dẫn từng bước

1. **Scaffold** với **AppBar** — title "Dashboard", actions: notification icon + avatar
2. **SingleChildScrollView** cho body (vì cả GridView + ListView cần scroll cùng nhau)
3. **Padding** cho content
4. **Text** chào mừng — "Xin chào, Nguyễn Văn A 👋"
5. **GridView.count** — `crossAxisCount: 2`, `shrinkWrap: true`, `physics: NeverScrollableScrollPhysics()`
   - 4 stat cards: Orders, Revenue, Users, Rating
   - Mỗi card: **Container** với **Column** (icon, value, label)
6. **Text** "Recent Activity" section header
7. **ListView.builder** — `shrinkWrap: true`, `physics: NeverScrollableScrollPhysics()`
   - 10 activity items
   - Mỗi item: **ListTile** với leading icon, title, subtitle (time ago)

> ⚠️ **Quan trọng:** Vì cả GridView và ListView nằm trong SingleChildScrollView, phải dùng `shrinkWrap: true` + `NeverScrollableScrollPhysics()` để tránh nested scroll conflict. Đây là trade-off cho list ngắn — nếu list dài, dùng `CustomScrollView` + Slivers.

### Gợi ý code khung

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dashboard',
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: const DashboardScreen(),
    );
  }
}

class DashboardScreen extends StatelessWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        actions: [
          IconButton(icon: const Icon(Icons.notifications_outlined), onPressed: () {}),
          // TODO: Avatar icon
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // TODO: Welcome text

            const SizedBox(height: 16),

            // TODO: GridView.count for stat cards
            // Nhớ: shrinkWrap: true, physics: NeverScrollableScrollPhysics()

            const SizedBox(height: 24),

            // TODO: "Recent Activity" header

            const SizedBox(height: 8),

            // TODO: ListView.builder for activity items
            // Nhớ: shrinkWrap: true, physics: NeverScrollableScrollPhysics()
          ],
        ),
      ),
    );
  }

  /// Stat card widget
  Widget _buildStatCard({
    required String title,
    required String value,
    required IconData icon,
    required Color color,
  }) {
    // TODO: Implement stat card
    // Card > Padding > Column > [Icon, Value text, Label text]
    throw UnimplementedError();
  }
}
```

### Widgets cần dùng

| Widget | Mục đích |
|--------|----------|
| `AppBar` | Header với actions |
| `SingleChildScrollView` | Scroll toàn trang |
| `GridView.count` | Grid 2 cột cho stat cards |
| `Card` | Stat card container |
| `Column` | Layout trong card |
| `ListView.builder` | Danh sách activity |
| `ListTile` | Mỗi activity item |
| `CircleAvatar` | Leading icon cho ListTile |

### Tiêu chí hoàn thành

- [ ] AppBar có title + notification + avatar icons
- [ ] 4 stat cards trong GridView 2 cột
- [ ] Mỗi stat card có icon, value (lớn, bold), label
- [ ] Recent Activity section với ít nhất 5 items
- [ ] Toàn trang scroll mượt (không bị nested scroll conflict)
- [ ] `shrinkWrap: true` + `NeverScrollableScrollPhysics()` cho GridView/ListView lồng nhau

---

## BT3 ⭐⭐⭐: Responsive Layout App 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_responsive_app` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Layout thay đổi theo phone/tablet breakpoint |

### Yêu cầu
Xây dựng app thay đổi layout giữa phone và tablet sử dụng LayoutBuilder.

### Mockup — Phone (< 600px width)

```
╔══════════════════════╗
║  My App        ☰     ║
╠══════════════════════╣
║                      ║
║  [Category 1]        ║
║  [Category 2]        ║
║  [Category 3]        ║
║  [Category 4]        ║
║  [Category 5]        ║
║                      ║
║  ← Tap để xem detail ║
║                      ║
╚══════════════════════╝
```

### Mockup — Tablet (>= 600px width)

```
╔═══════════════════════════════════════════════════╗
║  My App                                           ║
╠═══════════════════════════════════════════════════╣
║              │                                    ║
║  Category 1  │  📋 Detail View                    ║
║  Category 2  │                                    ║
║ >Category 3< │  Title: Category 3                 ║
║  Category 4  │                                    ║
║  Category 5  │  Lorem ipsum dolor sit amet...     ║
║              │                                    ║
║              │  [Image placeholder]               ║
║              │                                    ║
║              │  More details about this category  ║
║              │                                    ║
╚═══════════════════════════════════════════════════╝
   1/3 width       2/3 width
```

### Hướng dẫn từng bước

1. **Data model:** Tạo class `Category` với `name`, `icon`, `description`, `color`
2. **StatefulWidget** cho main screen — lưu `selectedIndex`
3. **LayoutBuilder** trong body:
   - `constraints.maxWidth >= 600` → **Tablet layout** (Row: list 1/3 + detail 2/3)
   - `constraints.maxWidth < 600` → **Phone layout** (chỉ list, tap → Navigator.push detail)
4. **List widget:** `ListView.builder` hiển thị categories
5. **Detail widget:** Hiển thị thông tin chi tiết của category đã chọn
6. **Tablet:** Cả list + detail trên cùng màn hình, dùng `Expanded(flex: 1)` + `Expanded(flex: 2)`
7. **Phone:** List screen → tap → push DetailScreen

### Gợi ý code khung

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Responsive App',
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      home: const ResponsiveScreen(),
    );
  }
}

// Data
class Category {
  final String name;
  final IconData icon;
  final String description;
  final Color color;

  const Category({
    required this.name,
    required this.icon,
    required this.description,
    required this.color,
  });
}

final categories = [
  Category(
    name: 'Flutter Basics',
    icon: Icons.widgets,
    description: 'Học các widget cơ bản trong Flutter...',
    color: Colors.blue,
  ),
  // TODO: Thêm 4-5 categories nữa
];

class ResponsiveScreen extends StatefulWidget {
  const ResponsiveScreen({super.key});

  @override
  State<ResponsiveScreen> createState() => _ResponsiveScreenState();
}

class _ResponsiveScreenState extends State<ResponsiveScreen> {
  int _selectedIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My App')),
      body: LayoutBuilder(
        builder: (context, constraints) {
          if (constraints.maxWidth >= 600) {
            // TODO: Tablet layout — Row(list + detail)
            return Row(
              children: [
                Expanded(
                  flex: 1,
                  child: _buildCategoryList(
                    onTap: (index) {
                      setState(() => _selectedIndex = index);
                    },
                  ),
                ),
                const VerticalDivider(width: 1),
                Expanded(
                  flex: 2,
                  child: _buildDetailView(categories[_selectedIndex]),
                ),
              ],
            );
          } else {
            // TODO: Phone layout — chỉ list, tap → push
            return _buildCategoryList(
              onTap: (index) {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => DetailScreen(
                      category: categories[index],
                    ),
                  ),
                );
              },
            );
          }
        },
      ),
    );
  }

  Widget _buildCategoryList({required ValueChanged<int> onTap}) {
    // TODO: ListView.builder showing categories
    throw UnimplementedError();
  }

  Widget _buildDetailView(Category category) {
    // TODO: Detail view with icon, title, description
    throw UnimplementedError();
  }
}

class DetailScreen extends StatelessWidget {
  final Category category;

  const DetailScreen({super.key, required this.category});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(category.name)),
      // TODO: Reuse _buildDetailView or similar
      body: const Placeholder(),
    );
  }
}
```

### Widgets cần dùng

| Widget | Mục đích |
|--------|----------|
| `LayoutBuilder` | Detect phone vs tablet |
| `Row` + `Expanded` | Side-by-side layout (tablet) |
| `ListView.builder` | Category list |
| `Navigator.push` | Navigate (phone) |
| `VerticalDivider` | Phân cách list/detail (tablet) |
| `ListTile` | Category item |

### Tiêu chí hoàn thành

- [ ] **Phone (< 600px):** Hiển thị chỉ list, tap → push sang detail screen
- [ ] **Tablet (≥ 600px):** Hiển thị list (1/3) + detail (2/3) side-by-side
- [ ] Chọn category trong list → detail view cập nhật (tablet)
- [ ] Highlight category đang chọn trong list (tablet)
- [ ] Dùng LayoutBuilder (không hard-code)
- [ ] Detail view có icon, title (lớn), description

### Bonus

- [ ] Thêm animation khi chuyển detail (AnimatedSwitcher)
- [ ] OrientationBuilder: phone landscape → cũng dùng tablet layout
- [ ] Thêm breakpoint thứ 3 cho desktop (≥ 1200px)

---

## 💬 Câu hỏi thảo luận

### Câu 1: Constraints Model vs CSS Box Model

> Flutter dùng "Constraints go down, Sizes go up, Parent sets position". CSS dùng Box Model (content → padding → border → margin). Theo bạn, hệ thống nào dễ dự đoán hơn (predictable)? Tại sao?

**Gợi ý suy nghĩ:**
- Trong CSS, `width: 100%` nghĩa gì? Có bao gồm padding không? (`box-sizing`)
- Trong Flutter, widget có biết vị trí của mình không?
- CSS có "margin collapse" — Flutter có không?
- Khi debug layout, hệ thống nào dễ trace hơn?

---

### Câu 2: ListView vs GridView — Khi nào dùng cái nào?

> Bạn đang build một app hiển thị sản phẩm. Khi nào bạn chọn ListView, khi nào chọn GridView, và khi nào cần CustomScrollView? Cho ví dụ cụ thể.

**Gợi ý suy nghĩ:**
- Sản phẩm dạng list (như Shopee danh mục) vs dạng grid (như Shopee trang chủ)
- Trang chi tiết sản phẩm có header image co giãn → Slivers?
- Performance: `shrinkWrap: true` vs `ListView.builder` — trade-off gì?
- Kết hợp grid + list trong cùng scroll → CustomScrollView

---

### Câu 3: Slivers — Tại sao cần?

> Slivers phức tạp hơn ListView/GridView thông thường. Tại sao Flutter cần Slivers? Khi nào bạn PHẢI dùng Slivers thay vì nested ListView?

**Gợi ý suy nghĩ:**
- SliverAppBar (collapsing toolbar) — có thể làm bằng ListView không?
- Kết hợp SliverGrid + SliverList trong cùng scroll
- Performance: Slivers lazy-render tốt hơn `shrinkWrap: true`
- Ví dụ thực tế: app Twitter/Instagram — feed = SliverList, header = SliverAppBar

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 2:** Focus vào verify layout correctness và debug constraint errors.

### AI-BT1: Gen Layout 3-cột Responsive + Debug RenderFlex Overflow ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Row, Column, Expanded, Flexible, constraints model, layout errors.
- **Task thực tế:** PM yêu cầu "Trang product listing: mỗi row hiển thị 3 sản phẩm, khi màn hình nhỏ chuyển sang 2 cột, có hình ảnh + tên + giá, text dài phải ellipsis không được tràn". Layout responsive cho nhiều kích thước màn hình.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần tạo Flutter layout responsive cho trang product listing.
Context: e-commerce app, trang danh sách sản phẩm hiển thị grid.
Tech stack: Flutter 3.x, Dart 3.x.
Constraints:
- Dùng LayoutBuilder để detect chiều rộng: >= 600px → 3 cột, < 600px → 2 cột.
- Mỗi product card: Image(height: 120, fit: BoxFit.cover) + Text tên (maxLines: 2, overflow: ellipsis) + Text giá (bold).
- Dùng GridView.builder với SliverGridDelegateWithFixedCrossAxisCount.
- KHÔNG dùng hardcode width cho card — phải fill tự động theo grid.
- Padding 12 giữa các card (crossAxisSpacing, mainAxisSpacing).
- Card có border radius 8, elevation 2.
Output: 1 file product_grid.dart với ProductCard widget + ProductGridScreen.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 2 widgets: `ProductCard` (StatelessWidget) + `ProductGridScreen` có LayoutBuilder + GridView.builder.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Text tên sản phẩm có `maxLines` + `overflow: TextOverflow.ellipsis`? | ☐ |
| 2 | Image dùng `BoxFit.cover` hoặc `contain`, KHÔNG để mặc định? | ☐ |
| 3 | Không có hardcode width cho card (dùng GridView delegate tự chia)? | ☐ |
| 4 | LayoutBuilder ở đúng vị trí (không bọc ngoài Scaffold)? | ☐ |
| 5 | childAspectRatio phù hợp (image 120 + text ~60 → ~0.7)? | ☐ |
| 6 | `flutter analyze` không có warning? | ☐ |

**4. Customize:**
Tự thử: cố ý bỏ `overflow: TextOverflow.ellipsis` khỏi Text tên sản phẩm dài → observe RenderFlex overflow error → paste error vào AI để AI giải thích constraint flow → so sánh fix của AI vs fix bằng Expanded/Flexible mà bạn tự biết.
