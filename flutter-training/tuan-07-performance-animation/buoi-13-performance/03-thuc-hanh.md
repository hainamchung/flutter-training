# Buổi 13: Performance Optimization — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới performance — FE developer có **nền tảng tốt** về optimization nhưng cần adapt sang Flutter model.
> Đọc bảng dưới TRƯỚC khi code để mindset đúng.

| FE Performance Habit | Flutter Reality | Bài tập liên quan |
|----------------------|-----------------|---------------------|
| `React.memo()` / `useMemo()` | `const` widget + extract widget — compile-time optimization | BT1 |
| Chrome DevTools Performance tab | Flutter DevTools: Timeline, Widget Rebuild tracker, Memory tab | BT1, BT2 |
| Bundle analyzer cho size | APK Analyzer + `--split-debug-info` + tree shaking | BT2, BT3 |
| `useEffect` cleanup optional | `dispose()` **bắt buộc** — miss = memory leak, cần verify mỗi StatefulWidget | BT3 |

---

## BT1 ⭐: Identify & Fix Rebuilds 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_fix_rebuilds` |
| **Cách chạy** | `flutter run --profile` |
| **Output** | UI trên emulator — Profile screen, dùng DevTools đo rebuild count |

### Mục tiêu
- Dùng DevTools để đếm số lần rebuild
- Tối ưu bằng `const` constructors và widget splitting
- Đo lường cải thiện cụ thể

### Yêu cầu

Bạn được cho sẵn widget "Bad" bên dưới. Hãy:

1. **Chạy app, mở DevTools** → Widget Inspector → bật "Track Widget Rebuilds"
2. **Nhấn nút "Change Theme" 5 lần** → ghi lại rebuild count cho mỗi widget
3. **Tối ưu** bằng cách:
   - Thêm `const` constructor cho tất cả widget có thể
   - Tách thành small widgets
   - Thêm `const` keyword khi sử dụng
4. **Đo lại** rebuild count → so sánh trước/sau

### Code ban đầu (cần tối ưu)

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: ProfileScreen(),
    );
  }
}

class ProfileScreen extends StatefulWidget {
  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  bool _isDark = false;

  @override
  Widget build(BuildContext context) {
    return Theme(
      data: _isDark ? ThemeData.dark() : ThemeData.light(),
      child: Scaffold(
        appBar: AppBar(
          title: Text('Profile'),
          actions: [
            IconButton(
              icon: Icon(Icons.brightness_6),
              onPressed: () => setState(() => _isDark = !_isDark),
            ),
          ],
        ),
        body: SingleChildScrollView(
          child: Column(
            children: [
              // Section 1: Avatar — KHÔNG thay đổi theo theme
              Container(
                padding: const EdgeInsets.all(32),
                child: Column(
                  children: [
                    CircleAvatar(
                      radius: 50,
                      backgroundColor: Colors.blue,
                      child: Icon(Icons.person, size: 50, color: Colors.white),
                    ),
                    const SizedBox(height: 16),
                    Text(
                      'Nguyễn Văn A',
                      style: TextStyle(
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    Text('nguyenvana@email.com'),
                  ],
                ),
              ),

              // Section 2: Stats — KHÔNG thay đổi theo theme
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildStat('Posts', '128'),
                  _buildStat('Followers', '1.2K'),
                  _buildStat('Following', '567'),
                ],
              ),

              const SizedBox(height: 24),

              // Section 3: Menu items — KHÔNG thay đổi theo theme
              ListTile(
                leading: Icon(Icons.settings),
                title: Text('Cài đặt'),
                trailing: Icon(Icons.chevron_right),
              ),
              ListTile(
                leading: Icon(Icons.help),
                title: Text('Trợ giúp'),
                trailing: Icon(Icons.chevron_right),
              ),
              ListTile(
                leading: Icon(Icons.info),
                title: Text('Thông tin'),
                trailing: Icon(Icons.chevron_right),
              ),
              ListTile(
                leading: Icon(Icons.logout),
                title: Text('Đăng xuất'),
                trailing: Icon(Icons.chevron_right),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildStat(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
        ),
        Text(label, style: TextStyle(color: Colors.grey)),
      ],
    );
  }
}
```

### Bảng ghi kết quả

```
Điền vào bảng sau:

Widget                 | Rebuild (trước) | Rebuild (sau) | Giảm
──────────────────────────────────────────────────────────────
ProfileScreen          |                 |               |
Avatar section         |                 |               |
Stats Row              |                 |               |
ListTile (Cài đặt)    |                 |               |
ListTile (Trợ giúp)   |                 |               |
ListTile (Thông tin)   |                 |               |
ListTile (Đăng xuất)  |                 |               |
──────────────────────────────────────────────────────────────
Tổng rebuilds          |                 |               |
```

### Gợi ý hướng giải

<details>
<summary>💡 Gợi ý 1: const constructors</summary>

Thêm `const` constructor cho mọi StatelessWidget:
```dart
class AvatarSection extends StatelessWidget {
  const AvatarSection({super.key}); // ← const constructor
  // ...
}
```
</details>

<details>
<summary>💡 Gợi ý 2: Widget splitting</summary>

Tách thành các widget nhỏ:
- `AvatarSection` — avatar + tên + email
- `StatsRow` — 3 stat columns
- `MenuSection` — 4 ListTiles

Sử dụng với `const`:
```dart
const AvatarSection(),
const StatsRow(),
const MenuSection(),
```
</details>

<details>
<summary>💡 Gợi ý 3: const trong widget tree</summary>

```dart
const SizedBox(height: 16),          // const
const Icon(Icons.person, size: 50),  // const
const Text('Nguyễn Văn A'),          // const
```
</details>

### Tiêu chí hoàn thành

- [ ] Đã ghi lại rebuild count trước khi tối ưu
- [ ] Đã thêm `const` constructors cho tất cả widget tĩnh
- [ ] Đã tách thành ít nhất 3 small widgets
- [ ] Rebuild count giảm ≥ 60%
- [ ] App vẫn hoạt động đúng (theme toggle vẫn work)

---

## BT2 ⭐⭐: Optimize a Janky Contact List 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_contact_list` |
| **Cách chạy** | `flutter run --profile` |
| **Output** | UI trên emulator — So sánh ListView vs ListView.builder với 10,000 items |

### Mục tiêu
- So sánh `ListView` vs `ListView.builder` với 10,000 items
- Dùng DevTools Performance view để đo FPS
- Profile và chứng minh sự khác biệt bằng số liệu

### Yêu cầu

**Phần A: Tạo phiên bản chậm**

1. Tạo `SlowContactList` dùng `ListView` với 10,000 contacts
2. Mỗi contact gồm: avatar (CircleAvatar), tên, số điện thoại, icon trailing
3. Chạy ở **profile mode** (`flutter run --profile`)
4. Dùng DevTools:
   - Performance view: scroll list, ghi lại số frame janky
   - Widget Inspector: đếm số widget được tạo

**Phần B: Tạo phiên bản nhanh**

1. Tạo `FastContactList` dùng `ListView.builder` với cùng 10,000 contacts
2. Thêm `const` constructors cho `ContactTile`
3. Profile tương tự phần A

**Phần C: Thêm optimizations**

1. Thêm `ListView.separated` với divider
2. Thêm `itemExtent` (fixed height) để skip layout calculation
3. Thêm `addAutomaticKeepAlives: false` nếu không cần giữ state
4. Profile lần nữa

### Starter code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ContactApp());

class ContactApp extends StatelessWidget {
  const ContactApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: ContactListComparison(),
    );
  }
}

class ContactListComparison extends StatefulWidget {
  const ContactListComparison({super.key});

  @override
  State<ContactListComparison> createState() => _ContactListComparisonState();
}

class _ContactListComparisonState extends State<ContactListComparison> {
  bool _useFastList = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_useFastList ? 'Fast List ✅' : 'Slow List ❌'),
        actions: [
          Switch(
            value: _useFastList,
            onChanged: (v) => setState(() => _useFastList = v),
          ),
        ],
      ),
      // TODO: Implement SlowContactList và FastContactList
      body: _useFastList
          ? const FastContactList()
          : const SlowContactList(),
    );
  }
}

// TODO: Implement SlowContactList — dùng ListView
class SlowContactList extends StatelessWidget {
  const SlowContactList({super.key});

  @override
  Widget build(BuildContext context) {
    // Dùng ListView với List.generate(10000, ...)
    throw UnimplementedError();
  }
}

// TODO: Implement FastContactList — dùng ListView.builder
class FastContactList extends StatelessWidget {
  const FastContactList({super.key});

  @override
  Widget build(BuildContext context) {
    // Dùng ListView.builder với itemCount: 10000
    throw UnimplementedError();
  }
}

// TODO: Implement ContactTile với const constructor
class ContactTile extends StatelessWidget {
  final int index;

  const ContactTile({super.key, required this.index});

  @override
  Widget build(BuildContext context) {
    // Avatar, tên, SĐT, trailing icon
    throw UnimplementedError();
  }
}
```

### Bảng ghi kết quả profiling

```
Điền sau khi profile ở --profile mode:

Metric                 | ListView     | ListView.builder | Cải thiện
──────────────────────────────────────────────────────────────────
Initial build time     |              |                  |
Widgets created        |              |                  |
Scroll FPS (avg)       |              |                  |
Janky frames (10s)     |              |                  |
Memory usage (peak)    |              |                  |

Optimizations thêm (Phần C):
+ itemExtent            |              |                  |
+ keepAlives: false     |              |                  |
```

### Tiêu chí hoàn thành

- [ ] Có cả 2 phiên bản Slow/Fast chạy được
- [ ] Đã profile ở `--profile` mode (không phải debug)
- [ ] Đã ghi lại FPS cho cả 2 phiên bản
- [ ] `ListView.builder` đạt ≥ 55fps khi scroll
- [ ] Đã thử thêm `itemExtent` và `addAutomaticKeepAlives`
- [ ] Có bảng so sánh số liệu cụ thể

---

## BT3 ⭐⭐⭐: Profile & Fix a Complex Screen 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_product_detail` |
| **Cách chạy** | `flutter run --profile` |
| **Output** | UI trên emulator — Product detail screen, profile & fix bottlenecks |

### Mục tiêu
- Profile một screen phức tạp có nhiều vấn đề performance
- Dùng DevTools tìm top 3 bottlenecks
- Fix từng issue, verify improvement sau mỗi fix
- Viết báo cáo performance analysis

### Yêu cầu

Bạn được cho screen "E-commerce Product Detail" có **TẤT CẢ** các vấn đề performance phổ biến. Hãy:

1. **Chạy ở profile mode**, mở DevTools
2. **Profile screen** — interact (scroll, tap, nhấn nút)
3. **Identify top 3 bottlenecks** — ghi rõ:
   - Issue là gì?
   - Ở đâu trong code?
   - Ảnh hưởng bao nhiêu ms/frame?
4. **Fix từng issue** — sau mỗi fix, profile lại và verify
5. **Viết báo cáo** theo template bên dưới

### Code có vấn đề

```dart
import 'dart:convert';
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: ProductDetailScreen());
  }
}

class ProductDetailScreen extends StatefulWidget {
  const ProductDetailScreen({super.key});

  @override
  State<ProductDetailScreen> createState() => _ProductDetailScreenState();
}

class _ProductDetailScreenState extends State<ProductDetailScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _animController;
  int _selectedColor = 0;
  int _quantity = 1;
  List<Map<String, dynamic>> _reviews = [];
  bool _loadingReviews = false;

  @override
  void initState() {
    super.initState();
    _animController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat(reverse: true);
    _loadReviews();
  }

  // ❌ Issue: Heavy computation trên main thread
  void _loadReviews() {
    setState(() => _loadingReviews = true);

    // Simulate heavy JSON parsing trên main thread
    final reviews = List.generate(2000, (i) {
      var rating = 0.0;
      for (var j = 0; j < 10000; j++) {
        rating += sin(i * j.toDouble()) * cos(j.toDouble());
      }
      return {
        'id': i,
        'user': 'User $i',
        'rating': (rating.abs() % 5).toInt() + 1,
        'comment': 'This is review #$i. ' * 5,
        'date': '2024-${(i % 12) + 1}-${(i % 28) + 1}',
      };
    });

    setState(() {
      _reviews = reviews;
      _loadingReviews = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SingleChildScrollView(
        child: Column(
          children: [
            // ❌ Issue: Animation không có RepaintBoundary
            _buildAnimatedBanner(),

            // ❌ Issue: Rebuild toàn bộ khi chỉ thay đổi color/quantity
            _buildProductInfo(),
            _buildColorSelector(),
            _buildQuantitySelector(),
            _buildAddToCartButton(),
            _buildDescription(),

            // ❌ Issue: Nested ListView trong SingleChildScrollView
            _buildReviewsSection(),
          ],
        ),
      ),
    );
  }

  Widget _buildAnimatedBanner() {
    return AnimatedBuilder(
      animation: _animController,
      builder: (context, child) {
        return Container(
          height: 300,
          decoration: BoxDecoration(
            gradient: LinearGradient(
              colors: [
                Colors.blue.withValues(alpha: _animController.value),
                Colors.purple.withValues(alpha: 1 - _animController.value),
              ],
            ),
          ),
          child: Center(
            child: Text(
              '🔥 SALE 50% OFF',
              style: TextStyle(
                fontSize: 32,
                fontWeight: FontWeight.bold,
                color: Colors.white,
              ),
            ),
          ),
        );
      },
    );
  }

  Widget _buildProductInfo() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Premium Wireless Headphones',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Row(
            children: [
              Icon(Icons.star, color: Colors.amber),
              Icon(Icons.star, color: Colors.amber),
              Icon(Icons.star, color: Colors.amber),
              Icon(Icons.star, color: Colors.amber),
              Icon(Icons.star_half, color: Colors.amber),
              const SizedBox(width: 8),
              Text('4.5 (${_reviews.length} reviews)'),
            ],
          ),
          const SizedBox(height: 8),
          Text(
            '\$299.99',
            style: TextStyle(
              fontSize: 28,
              fontWeight: FontWeight.bold,
              color: Colors.green,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildColorSelector() {
    final colors = [Colors.black, Colors.white, Colors.blue, Colors.red];
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: List.generate(colors.length, (index) {
          return GestureDetector(
            onTap: () => setState(() => _selectedColor = index),
            child: Container(
              margin: const EdgeInsets.all(4),
              width: 40,
              height: 40,
              decoration: BoxDecoration(
                color: colors[index],
                shape: BoxShape.circle,
                border: Border.all(
                  color: _selectedColor == index
                      ? Colors.blue
                      : Colors.transparent,
                  width: 3,
                ),
              ),
            ),
          );
        }),
      ),
    );
  }

  Widget _buildQuantitySelector() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          Text('Số lượng: ', style: TextStyle(fontSize: 16)),
          IconButton(
            onPressed: () {
              if (_quantity > 1) setState(() => _quantity--);
            },
            icon: Icon(Icons.remove_circle_outline),
          ),
          Text('$_quantity', style: TextStyle(fontSize: 20)),
          IconButton(
            onPressed: () => setState(() => _quantity++),
            icon: Icon(Icons.add_circle_outline),
          ),
        ],
      ),
    );
  }

  Widget _buildAddToCartButton() {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: SizedBox(
        width: double.infinity,
        child: ElevatedButton(
          onPressed: () {},
          style: ElevatedButton.styleFrom(
            padding: const EdgeInsets.symmetric(vertical: 16),
          ),
          child: Text(
            'Thêm vào giỏ hàng ($_quantity)',
            style: TextStyle(fontSize: 18),
          ),
        ),
      ),
    );
  }

  Widget _buildDescription() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Mô tả sản phẩm',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Text(
            'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
            'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. '
            'Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris. '
            'Duis aute irure dolor in reprehenderit in voluptate velit esse. '
            'Excepteur sint occaecat cupidatat non proident, sunt in culpa.',
            style: TextStyle(fontSize: 14, height: 1.5),
          ),
        ],
      ),
    );
  }

  // ❌ Issue: ListView bên trong SingleChildScrollView
  Widget _buildReviewsSection() {
    if (_loadingReviews) {
      return Center(child: CircularProgressIndicator());
    }

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Padding(
          padding: const EdgeInsets.all(16),
          child: Text(
            'Reviews (${_reviews.length})',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
          ),
        ),
        // ❌ Tạo TẤT CẢ 2000 review widgets cùng lúc
        // ❌ ListView trong SingleChildScrollView cần shrinkWrap: true → tính layout 2 lần
        SizedBox(
          height: 400,
          child: ListView(
            children: _reviews.map((review) {
              return Card(
                margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                child: Padding(
                  padding: const EdgeInsets.all(12),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Row(
                        children: [
                          CircleAvatar(
                            child: Text(
                              (review['user'] as String)[0],
                            ),
                          ),
                          const SizedBox(width: 8),
                          Text(
                            review['user'] as String,
                            style: TextStyle(fontWeight: FontWeight.bold),
                          ),
                          Spacer(),
                          ...List.generate(
                            review['rating'] as int,
                            (_) => Icon(Icons.star,
                                color: Colors.amber, size: 16),
                          ),
                        ],
                      ),
                      const SizedBox(height: 8),
                      Text(review['comment'] as String),
                      const SizedBox(height: 4),
                      Text(
                        review['date'] as String,
                        style: TextStyle(color: Colors.grey, fontSize: 12),
                      ),
                    ],
                  ),
                ),
              );
            }).toList(),
          ),
        ),
      ],
    );
  }

  // ❌ Issue: Không dispose AnimationController
  // @override
  // void dispose() {
  //   _animController.dispose();
  //   super.dispose();
  // }
}
```

### Checklist phân tích

Tìm và fix các issues sau (gợi ý — có thể có thêm):

```
□ Issue 1: Heavy computation trên main thread (_loadReviews)
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________

□ Issue 2: Animation không có RepaintBoundary
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________

□ Issue 3: ListView tạo tất cả 2000 widgets
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________

□ Issue 4: setState() rebuild toàn bộ screen
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________

□ Issue 5: AnimationController không dispose
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________

□ Issue 6: Không dùng const ở bất kỳ đâu
  Triệu chứng: ________________________________
  DevTools metric: _____________________________
  Fix: _________________________________________
  Cải thiện: ___________________________________
```

### Template báo cáo

```markdown
# Performance Analysis Report — Product Detail Screen

## Môi trường test
- Device: ________________________________
- Flutter version: ________________________
- Mode: profile

## Kết quả trước khi optimize
- Average FPS khi scroll: _____
- Jank frames (30s): _____
- Initial build time: _____ms
- Memory peak: _____MB

## Top 3 Bottlenecks Found

### Bottleneck #1: _______________
- **Vị trí:** Dòng ___, function ___________
- **Nguyên nhân:** ________________________
- **Ảnh hưởng:** __________ms per frame
- **Fix:** _________________________________
- **Kết quả:** FPS từ ___ lên ___

### Bottleneck #2: _______________
- **Vị trí:** ________________________________
- **Nguyên nhân:** ________________________
- **Ảnh hưởng:** ________________________
- **Fix:** _________________________________
- **Kết quả:** ____________________________

### Bottleneck #3: _______________
- **Vị trí:** ________________________________
- **Nguyên nhân:** ________________________
- **Ảnh hưởng:** ________________________
- **Fix:** _________________________________
- **Kết quả:** ____________________________

## Kết quả sau khi optimize
- Average FPS khi scroll: _____
- Jank frames (30s): _____
- Initial build time: _____ms
- Memory peak: _____MB

## Tổng cải thiện
- FPS: ___% improvement
- Jank: ___% reduction
- Build time: ___% reduction
- Memory: ___% reduction
```

### Tiêu chí hoàn thành

- [ ] Đã chạy ở `--profile` mode
- [ ] Đã tìm được ≥ 3 bottlenecks bằng DevTools
- [ ] Đã fix ≥ 3 issues
- [ ] Mỗi fix đều có số liệu before/after
- [ ] App vẫn hoạt động đúng sau khi optimize
- [ ] Đã viết báo cáo theo template
- [ ] FPS sau optimize ≥ 50fps khi scroll reviews

---

## 💬 Câu hỏi thảo luận

### Câu 1: Premature Optimization

> "Premature optimization is the root of all evil" — Donald Knuth

Theo bạn:
- Khi nào nên optimize performance từ đầu (lúc viết code)?
- Khi nào nên viết xong rồi mới optimize?
- Có kỹ thuật nào "miễn phí" nên luôn áp dụng? (Ví dụ: `const` constructors)

**Gợi ý suy nghĩ:**
- `const` constructors → gần như zero cost, luôn dùng
- `ListView.builder` cho list dài → habit tốt, luôn dùng
- `dispose()` resources → bắt buộc, không phải optimization
- Micro-optimization (tối ưu từng ms) → chỉ khi DevTools chỉ ra bottleneck

### Câu 2: React/Vue vs Flutter Rendering

So sánh rendering model:

```
React:  State change → Virtual DOM diff → DOM patch → Browser layout/paint
Flutter: State change → Widget rebuild → Element diff → RenderObject → Skia/Impeller
```

Theo bạn:
- Cách nào cho developer nhiều control hơn? Tại sao?
- Concept nào từ React performance optimization áp dụng được cho Flutter?
- Concept nào **KHÔNG** áp dụng được? (Ví dụ: `useMemo` không có tương đương trực tiếp)

### Câu 3: Isolate vs async/await

Bạn có API call trả về 5MB JSON. Workflow xử lý:

```dart
// Option A: Chỉ async/await
final response = await http.get(url);
final data = json.decode(response.body); // Trên main thread
final users = data.map((j) => User.fromJson(j)).toList(); // Trên main thread

// Option B: async/await + compute
final response = await http.get(url);
final users = await compute(parseUsers, response.body); // Trên isolate
```

Theo bạn:
- Option nào tốt hơn? Tại sao?
- 5MB JSON parse mất khoảng bao lâu trên main thread?
- Có trường hợp nào Option A đủ tốt không?
- `compute()` có overhead gì? (spawn isolate, copy data qua)
- Khi nào overhead của `compute()` lớn hơn benefit?

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 7:** Focus vào AI review performance code và profiling guidance.

### AI-BT1: Performance Audit với AI — Identify Unnecessary Rebuilds ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Rendering pipeline, rebuild optimization (const, splitting, RepaintBoundary), DevTools profiling, memory leaks, isolates.
- **Task thực tế:** QA report "app giật khi scroll contact list 5000 items". Senior yêu cầu performance audit + fix. AI phân tích code → bạn verify optimization suggestions bằng DevTools.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần performance audit cho Flutter screen sau:
[paste widget code — ContactListScreen với 5000 items]

Vấn đề: scroll giật, FPS 15-20 trên Android mid-range.

Phân tích:
1. Identify ALL unnecessary rebuilds (setState scope, thiếu const, non-split widgets).
2. ListView optimization: đang dùng ListView.builder? itemExtent? cacheExtent?
3. RepaintBoundary: đâu cần (expensive paint), đâu KHÔNG cần (overhead)?
4. const constructor: list TẤT CẢ widgets có thể thêm const.
5. Widget splitting: suggest tách StatefulWidget nào, tại sao.
6. Memory: có leak không? (dispose controllers, cancel subscriptions).
7. DevTools steps: record timeline → tìm build phase > 16ms → trace widget.
Output: Annotated code (mark optimization points) + priority-ordered checklist.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* Annotated code + optimization checklist.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | AI suggest const cho ĐÚNG widgets (immutable, no dynamic data)? | ☐ |
| 2 | ListView.builder dùng thay ListView? itemExtent set? | ☐ |
| 3 | RepaintBoundary chỉ cho expensive operations (không everywhere)? | ☐ |
| 4 | Widget splitting suggestions hợp lý? (setState scope giảm?) | ☐ |
| 5 | Memory leak check: dispose, cancel, mounted check? | ☐ |
| 6 | Verify bằng DevTools: record → profile → confirm improvement? | ☐ |

**4. Customize:**
Apply optimizations → chạy `flutter run --profile` → dùng DevTools Performance tab → record scroll → so sánh trước/sau (FPS, build time). AI chỉ suggest code changes — bạn phải verify bằng DevTools profiling.
