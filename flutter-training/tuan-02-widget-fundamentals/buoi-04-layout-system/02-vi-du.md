# Buổi 04: Layout System — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Hướng dẫn:** Mỗi ví dụ là một Flutter app hoàn chỉnh. Copy toàn bộ code vào `lib/main.dart` để chạy.

---

## VD1: Constraints Demo — Constraints flow từ parent xuống child 🔴

### Mục đích
Hiểu cách constraints từ parent ảnh hưởng đến kích thước child. Container "muốn" 300x300 nhưng bị parent constraints giới hạn.

> **Liên quan tới:** [1. Constraints Model — Nguyên tắc cốt lõi 🔴](01-ly-thuyet.md#1-constraints-model--nguyên-tắc-cốt-lõi)

### Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ConstraintsDemo());

class ConstraintsDemo extends StatelessWidget {
  const ConstraintsDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Constraints Demo',
      home: Scaffold(
        appBar: AppBar(title: const Text('VD1: Constraints Demo')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // === DEMO 1: Tight constraints ===
              const Text(
                '1. Tight Constraints (SizedBox 150x100)',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              const Text('Container muốn 300x300, nhưng SizedBox giới hạn 150x100:'),
              const SizedBox(height: 8),
              // SizedBox gửi tight constraints: exactly 150x100
              SizedBox(
                width: 150,
                height: 100,
                child: Container(
                  width: 300,   // Muốn 300 nhưng chỉ được 150!
                  height: 300,  // Muốn 300 nhưng chỉ được 100!
                  color: Colors.red.shade300,
                  child: const Center(
                    child: Text(
                      '150x100\n(muốn 300x300)',
                      textAlign: TextAlign.center,
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),

              const SizedBox(height: 32),

              // === DEMO 2: Loose constraints ===
              const Text(
                '2. Loose Constraints (Center)',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              const Text('Center cho phép child tự chọn kích thước (loose):'),
              const SizedBox(height: 8),
              Container(
                width: double.infinity,
                height: 200,
                color: Colors.grey.shade200,
                // Center chuyển constraints thành loose
                child: Center(
                  child: Container(
                    width: 120,   // Chọn tự do: 120
                    height: 80,   // Chọn tự do: 80
                    color: Colors.blue.shade300,
                    child: const Center(
                      child: Text(
                        '120x80\n(tự chọn)',
                        textAlign: TextAlign.center,
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ),
              ),

              const SizedBox(height: 32),

              // === DEMO 3: Container không có child ===
              const Text(
                '3. Container KHÔNG có child → to nhất có thể',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                height: 60,
                color: Colors.green.shade300,
                // Không có child → fill hết chiều rộng parent
                alignment: Alignment.center,
                child: const Text(
                  'Container không child → fill hết width',
                  style: TextStyle(color: Colors.white),
                ),
              ),

              const SizedBox(height: 32),

              // === DEMO 4: ConstrainedBox ===
              const Text(
                '4. ConstrainedBox — thêm constraints bổ sung',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              const Text('Min width 200, min height 60 (dù content nhỏ hơn):'),
              const SizedBox(height: 8),
              ConstrainedBox(
                constraints: const BoxConstraints(
                  minWidth: 200,
                  minHeight: 60,
                ),
                child: Container(
                  color: Colors.orange.shade300,
                  padding: const EdgeInsets.all(8),
                  child: const Text(
                    'Nhỏ',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),

              const SizedBox(height: 32),

              // === DEMO 5: LayoutBuilder để xem constraints ===
              const Text(
                '5. LayoutBuilder — xem constraints thực tế',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                color: Colors.purple.shade100,
                padding: const EdgeInsets.all(16),
                child: LayoutBuilder(
                  builder: (context, constraints) {
                    return Text(
                      'Parent constraints:\n'
                      'minWidth: ${constraints.minWidth}\n'
                      'maxWidth: ${constraints.maxWidth}\n'
                      'minHeight: ${constraints.minHeight}\n'
                      'maxHeight: ${constraints.maxHeight}',
                      style: const TextStyle(
                        fontFamily: 'monospace',
                        fontSize: 14,
                      ),
                    );
                  },
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Giải thích

```
Demo 1: SizedBox (150x100) → gửi TIGHT constraints
         Container muốn 300x300 → bị ép thành 150x100
         → Widget KHÔNG tự quyết định kích thước, phụ thuộc parent!

Demo 2: Center → chuyển constraints thành LOOSE
         Container tự chọn 120x80 → được phép!
         Center đặt Container ở giữa → Parent sets position!

Demo 3: Container không child → cố gắng to nhất
         → Fill hết width mà parent cho phép

Demo 4: ConstrainedBox thêm minWidth=200, minHeight=60
         → Content "Nhỏ" chỉ cần ~30px, nhưng container ít nhất 200x60

Demo 5: LayoutBuilder cho thấy constraints THỰC TẾ mà widget nhận được
```

- 🔗 **FE tương đương:** CSS box model cho child quyết định size rồi parent adjust. Flutter ngược lại — parent constraints quyết định trước, child chọn trong khoảng đó.
- 🔗 **FE tương đương:** LayoutBuilder ≈ CSS Container Queries — responsive dựa trên kích thước parent thay vì viewport.

---

## VD2: Row + Column Layout — Card với image + text 🟡

### Mục đích
Xây dựng card layout kết hợp Row, Column, Padding — pattern phổ biến nhất trong Flutter.

> **Liên quan tới:** [3. Multi-child Layout Widgets 🟡](01-ly-thuyet.md#3-multi-child-layout-widgets)

### Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CardLayoutDemo());

class CardLayoutDemo extends StatelessWidget {
  const CardLayoutDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Card Layout Demo',
      home: Scaffold(
        appBar: AppBar(title: const Text('VD2: Row + Column Layout')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              // === Card 1: Horizontal card (image left, text right) ===
              _buildHorizontalCard(
                title: 'Flutter Development',
                subtitle: 'Learn to build beautiful apps',
                icon: Icons.phone_android,
                color: Colors.blue,
              ),
              const SizedBox(height: 16),

              // === Card 2: Với Rating row ===
              _buildRatingCard(
                title: 'Dart Programming',
                description:
                    'Master the Dart language with hands-on exercises and real-world examples.',
                rating: 4.5,
                reviews: 128,
              ),
              const SizedBox(height: 16),

              // === Card 3: User profile card ===
              _buildProfileCard(
                name: 'Nguyễn Văn A',
                role: 'Flutter Developer',
                email: 'nguyen.a@example.com',
              ),
              const SizedBox(height: 16),

              // === Card 4: Stats card với Row of columns ===
              _buildStatsCard(),
            ],
          ),
        ),
      ),
    );
  }

  /// Card ngang: icon bên trái, text bên phải
  Widget _buildHorizontalCard({
    required String title,
    required String subtitle,
    required IconData icon,
    required Color color,
  }) {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // Icon bên trái — kích thước cố định
            Container(
              width: 60,
              height: 60,
              decoration: BoxDecoration(
                color: color.withValues(alpha: 0.1),
                borderRadius: BorderRadius.circular(12),
              ),
              child: Icon(icon, color: color, size: 32),
            ),
            const SizedBox(width: 16),
            // Text bên phải — Expanded để fill hết width còn lại
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    subtitle,
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.grey.shade600,
                    ),
                  ),
                ],
              ),
            ),
            // Arrow icon bên phải — kích thước cố định
            Icon(Icons.chevron_right, color: Colors.grey.shade400),
          ],
        ),
      ),
    );
  }

  /// Card với rating stars
  Widget _buildRatingCard({
    required String title,
    required String description,
    required double rating,
    required int reviews,
  }) {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              description,
              style: TextStyle(
                fontSize: 14,
                color: Colors.grey.shade600,
              ),
              maxLines: 2,
              overflow: TextOverflow.ellipsis,
            ),
            const SizedBox(height: 12),
            // Row: Stars + rating text + spacer + reviews count
            Row(
              children: [
                // Stars
                ...List.generate(5, (index) {
                  return Icon(
                    index < rating.floor()
                        ? Icons.star
                        : (index < rating ? Icons.star_half : Icons.star_border),
                    color: Colors.amber,
                    size: 20,
                  );
                }),
                const SizedBox(width: 8),
                Text(
                  '$rating',
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                const Spacer(), // Đẩy reviews sang phải
                Text(
                  '$reviews reviews',
                  style: TextStyle(color: Colors.grey.shade600),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  /// Profile card
  Widget _buildProfileCard({
    required String name,
    required String role,
    required String email,
  }) {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // Avatar
            const CircleAvatar(
              radius: 30,
              backgroundColor: Colors.teal,
              child: Icon(Icons.person, color: Colors.white, size: 32),
            ),
            const SizedBox(width: 16),
            // Info — Expanded để text dài tự wrap
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    role,
                    style: TextStyle(color: Colors.grey.shade600),
                  ),
                  const SizedBox(height: 2),
                  Text(
                    email,
                    style: TextStyle(
                      color: Colors.blue.shade600,
                      fontSize: 13,
                    ),
                  ),
                ],
              ),
            ),
            // Actions
            IconButton(
              icon: const Icon(Icons.message_outlined),
              onPressed: () {},
            ),
          ],
        ),
      ),
    );
  }

  /// Stats card — Row of equal columns
  Widget _buildStatsCard() {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            _buildStatItem('Projects', '12', Colors.blue),
            _buildDivider(),
            _buildStatItem('Followers', '1.2K', Colors.green),
            _buildDivider(),
            _buildStatItem('Rating', '4.8', Colors.orange),
          ],
        ),
      ),
    );
  }

  Widget _buildStatItem(String label, String value, Color color) {
    // Expanded để mỗi stat chiếm bằng nhau
    return Expanded(
      child: Column(
        children: [
          Text(
            value,
            style: TextStyle(
              fontSize: 24,
              fontWeight: FontWeight.bold,
              color: color,
            ),
          ),
          const SizedBox(height: 4),
          Text(
            label,
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey.shade600,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildDivider() {
    return Container(
      width: 1,
      height: 40,
      color: Colors.grey.shade300,
    );
  }
}
```

### Giải thích

```
Layout structure của mỗi card:

Card 1 (Horizontal):
┌─────────────────────────────────────────────┐
│  [Icon 60x60] [16px] [  Expanded Text  ] [>]│
│                       │ Title            │   │
│                       │ Subtitle         │   │
└─────────────────────────────────────────────┘
   fixed        gap      fills remaining   fixed

Card 2 (Rating):
┌─────────────────────────────────────────┐
│  Title                                   │
│  Description (maxLines: 2)               │
│  [★★★★☆] [4.5] ←Spacer→ [128 reviews]  │
└─────────────────────────────────────────┘

Card 3 (Profile):
┌──────────────────────────────────────────┐
│  [Avatar] [16px] [  Expanded  ] [✉️]     │
│                  │ Name        │          │
│                  │ Role        │          │
│                  │ Email       │          │
└──────────────────────────────────────────┘

Card 4 (Stats):
┌──────────────────────────────────────────┐
│  [ Expanded ] | [ Expanded ] | [Expanded]│
│     12        │    1.2K      │    4.8    │
│   Projects    │  Followers   │  Rating   │
└──────────────────────────────────────────┘
   flex: 1          flex: 1       flex: 1
```

- 🔗 **FE tương đương:** Tương tự `display: flex` trong CSS — nhưng Flutter không có `gap` property, phải dùng `SizedBox` giữa các children.

---

## VD3: Expanded/Flexible — Flex factor distribution 🔴

### Mục đích
Trực quan hóa cách `flex` factor phân bổ không gian: flex:1 vs flex:2 vs flex:3.

> **Liên quan tới:** [4. Flex System — Expanded, Flexible, Spacer 🔴](01-ly-thuyet.md#4-flex-system--expanded-flexible-spacer)

### Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const FlexDemo());

class FlexDemo extends StatelessWidget {
  const FlexDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flex Demo',
      home: Scaffold(
        appBar: AppBar(title: const Text('VD3: Expanded / Flexible')),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // === DEMO 1: Equal flex ===
              const Text(
                '1. Expanded — flex đều nhau (1:1:1)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              _buildFlexRow([
                _FlexItem(flex: 1, label: 'flex:1', color: Colors.red),
                _FlexItem(flex: 1, label: 'flex:1', color: Colors.green),
                _FlexItem(flex: 1, label: 'flex:1', color: Colors.blue),
              ]),

              const SizedBox(height: 24),

              // === DEMO 2: Different flex ===
              const Text(
                '2. Expanded — flex khác nhau (1:2:3)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 4),
              const Text(
                'Tổng flex = 6. Red = 1/6, Green = 2/6, Blue = 3/6',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 8),
              _buildFlexRow([
                _FlexItem(flex: 1, label: 'flex:1\n(1/6)', color: Colors.red),
                _FlexItem(flex: 2, label: 'flex:2\n(2/6)', color: Colors.green),
                _FlexItem(flex: 3, label: 'flex:3\n(3/6)', color: Colors.blue),
              ]),

              const SizedBox(height: 24),

              // === DEMO 3: Fixed + Expanded ===
              const Text(
                '3. Fixed widget + Expanded',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 4),
              const Text(
                'Icon (48px) + Expanded text + Button (80px)',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 8),
              Container(
                height: 60,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey.shade300),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Row(
                  children: [
                    Container(
                      width: 48,
                      color: Colors.orange.shade200,
                      child: const Center(child: Text('48px')),
                    ),
                    Expanded(
                      child: Container(
                        color: Colors.green.shade200,
                        child: const Center(
                          child: Text('Expanded\n(fills remaining)'),
                        ),
                      ),
                    ),
                    Container(
                      width: 80,
                      color: Colors.blue.shade200,
                      child: const Center(child: Text('80px')),
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 24),

              // === DEMO 4: Expanded vs Flexible ===
              const Text(
                '4. Expanded (tight) vs Flexible (loose)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              const Text('Expanded — bắt buộc fill hết:'),
              const SizedBox(height: 4),
              Container(
                height: 50,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    Expanded(
                      child: Container(
                        color: Colors.red.shade200,
                        child: const Text(' Expanded: fills ALL space'),
                      ),
                    ),
                  ],
                ),
              ),
              const SizedBox(height: 8),
              const Text('Flexible — co lại vừa content:'),
              const SizedBox(height: 4),
              Container(
                height: 50,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey.shade300),
                ),
                child: Row(
                  children: [
                    Flexible(
                      child: Container(
                        color: Colors.blue.shade200,
                        child: const Text(' Flexible: wraps content'),
                      ),
                    ),
                  ],
                ),
              ),

              const SizedBox(height: 24),

              // === DEMO 5: Spacer ===
              const Text(
                '5. Spacer — đẩy widget ra hai bên',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                decoration: BoxDecoration(
                  color: Colors.grey.shade100,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Row(
                  children: [
                    const Text('Left', style: TextStyle(fontWeight: FontWeight.bold)),
                    const Spacer(),
                    const Text('Center', style: TextStyle(fontWeight: FontWeight.bold)),
                    const Spacer(),
                    const Text('Right', style: TextStyle(fontWeight: FontWeight.bold)),
                  ],
                ),
              ),

              const SizedBox(height: 24),

              // === DEMO 6: Spacer vs Expanded ===
              const Text(
                '6. Spacer(flex:1) vs Spacer(flex:2)',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                decoration: BoxDecoration(
                  color: Colors.amber.shade100,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Row(
                  children: [
                    const Text('A'),
                    const Spacer(flex: 1), // 1 phần
                    const Text('B'),
                    const Spacer(flex: 2), // 2 phần — gấp đôi khoảng trống
                    const Text('C'),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildFlexRow(List<_FlexItem> items) {
    return Container(
      height: 60,
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey.shade300),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        children: items.map((item) {
          return Expanded(
            flex: item.flex,
            child: Container(
              color: item.color.withValues(alpha: 0.3),
              child: Center(
                child: Text(
                  item.label,
                  textAlign: TextAlign.center,
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: item.color.shade800,
                  ),
                ),
              ),
            ),
          );
        }).toList(),
      ),
    );
  }
}

class _FlexItem {
  final int flex;
  final String label;
  final MaterialColor color;

  const _FlexItem({
    required this.flex,
    required this.label,
    required this.color,
  });
}
```

### Giải thích

```
Demo 1: flex 1:1:1 → mỗi widget chiếm 1/3
  [████ 33% ████][████ 33% ████][████ 33% ████]

Demo 2: flex 1:2:3 → tổng 6, phân bổ 1/6, 2/6, 3/6
  [██ 17% ][████ 33% ███][██████ 50% ██████]

Demo 3: Fixed + Expanded
  [48px][══════ Expanded fills 300-48-80=172px ══════][80px]
  Bước 1: Layout fixed widgets (48+80=128px)
  Bước 2: Còn lại → Expanded

Demo 4: Expanded bắt buộc fill, Flexible co lại vừa đủ

Demo 5-6: Spacer tạo khoảng trống linh hoạt, flex control tỷ lệ
```

---

## VD4: ListView.builder — Scrollable list với lazy rendering 🟡

### Mục đích
Sử dụng `ListView.builder` cho danh sách dài, kết hợp `ListTile` và custom layout.

> **Liên quan tới:** [5. Scrollable Widgets 🟡](01-ly-thuyet.md#5-scrollable-widgets)

### Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ListViewDemo());

class ListViewDemo extends StatelessWidget {
  const ListViewDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ListView Demo',
      home: const ContactListScreen(),
    );
  }
}

class ContactListScreen extends StatelessWidget {
  const ContactListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Dữ liệu mẫu
    final contacts = List.generate(50, (index) {
      final names = [
        'Nguyễn Văn A', 'Trần Thị B', 'Lê Văn C', 'Phạm Thị D',
        'Hoàng Văn E', 'Đỗ Thị F', 'Bùi Văn G', 'Vũ Thị H',
      ];
      return _Contact(
        name: names[index % names.length],
        email: 'user${index + 1}@example.com',
        phone: '0${900000000 + index}',
        isOnline: index % 3 == 0,
      );
    });

    return Scaffold(
      appBar: AppBar(
        title: const Text('VD4: ListView.builder'),
        actions: [
          Center(
            child: Padding(
              padding: const EdgeInsets.only(right: 16),
              child: Text(
                '${contacts.length} contacts',
                style: const TextStyle(fontSize: 14),
              ),
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          // Search bar (fixed header)
          Padding(
            padding: const EdgeInsets.all(12),
            child: TextField(
              decoration: InputDecoration(
                hintText: 'Tìm kiếm...',
                prefixIcon: const Icon(Icons.search),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
                filled: true,
                fillColor: Colors.grey.shade100,
              ),
            ),
          ),

          // Contact list — Expanded để ListView có bounded height!
          Expanded(
            child: ListView.builder(
              itemCount: contacts.length,
              // Mỗi item CHỈ được build khi visible trên màn hình
              itemBuilder: (context, index) {
                final contact = contacts[index];
                return _ContactTile(contact: contact, index: index);
              },
            ),
          ),
        ],
      ),
    );
  }
}

class _ContactTile extends StatelessWidget {
  final _Contact contact;
  final int index;

  const _ContactTile({required this.contact, required this.index});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Stack(
        children: [
          CircleAvatar(
            backgroundColor: Colors.primaries[index % Colors.primaries.length],
            child: Text(
              contact.name[0],
              style: const TextStyle(
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          // Online indicator
          if (contact.isOnline)
            Positioned(
              right: 0,
              bottom: 0,
              child: Container(
                width: 12,
                height: 12,
                decoration: BoxDecoration(
                  color: Colors.green,
                  shape: BoxShape.circle,
                  border: Border.all(color: Colors.white, width: 2),
                ),
              ),
            ),
        ],
      ),
      title: Text(
        contact.name,
        style: const TextStyle(fontWeight: FontWeight.w600),
      ),
      subtitle: Text(contact.email),
      trailing: IconButton(
        icon: const Icon(Icons.phone_outlined),
        onPressed: () {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Gọi ${contact.phone}...')),
          );
        },
      ),
    );
  }
}

class _Contact {
  final String name;
  final String email;
  final String phone;
  final bool isOnline;

  const _Contact({
    required this.name,
    required this.email,
    required this.phone,
    required this.isOnline,
  });
}
```

### Giải thích

```
Cấu trúc layout:

Scaffold
├── AppBar (fixed)
└── Body
    └── Column
        ├── Padding > TextField (search bar — fixed)
        └── Expanded ← QUAN TRỌNG: cho ListView bounded height!
            └── ListView.builder
                ├── ListTile 0 (visible → được build)
                ├── ListTile 1 (visible)
                ├── ...
                ├── ListTile 49 (chưa visible → CHƯA build)
                └── (lazy rendering: chỉ build khi cần)

Điểm chú ý:
1. ListView PHẢI trong Expanded (tránh unbounded height)
2. ListView.builder cho 50 items — chỉ render ~10 items visible
3. Stack trong leading: Avatar + online indicator chồng lên nhau
4. ListTile: leading (avatar) + title + subtitle + trailing (phone icon)
```

---

## VD5: Common Layout Errors — Lỗi và cách sửa 🔴

### Mục đích
Thấy các lỗi layout phổ biến và cách sửa. Mỗi tab demo **code lỗi** và **code đã sửa**.

> **Liên quan tới:** [7. Common Layout Errors 🔴](01-ly-thuyet.md#7-common-layout-errors)

### Code

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LayoutErrorsDemo());

class LayoutErrorsDemo extends StatelessWidget {
  const LayoutErrorsDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Layout Errors Demo',
      home: DefaultTabController(
        length: 3,
        child: Scaffold(
          appBar: AppBar(
            title: const Text('VD5: Common Layout Errors'),
            bottom: const TabBar(
              tabs: [
                Tab(text: 'Unbounded\nHeight'),
                Tab(text: 'RenderFlex\nOverflow'),
                Tab(text: 'ParentData\nError'),
              ],
            ),
          ),
          body: const TabBarView(
            children: [
              _UnboundedHeightDemo(),
              _RenderFlexOverflowDemo(),
              _ParentDataDemo(),
            ],
          ),
        ),
      ),
    );
  }
}

// ==========================================
// ERROR 1: Unbounded Height
// ==========================================
class _UnboundedHeightDemo extends StatelessWidget {
  const _UnboundedHeightDemo();

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _buildErrorBox(
            title: '❌ LỖI: ListView trong Column (không Expanded)',
            description:
                'ListView cần bounded height, nhưng Column cho nó INFINITY.\n'
                'Error: "Vertical viewport was given unbounded height."',
            code: '''
Column(
  children: [
    Text('Header'),
    ListView(         // ← LỖI!
      children: [...],
    ),
  ],
)''',
          ),
          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA: Wrap ListView bằng Expanded',
            description: 'Expanded giới hạn height cho ListView.',
          ),
          // Code đã sửa — chạy thực tế
          Container(
            height: 200,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Column(
              children: [
                Container(
                  padding: const EdgeInsets.all(8),
                  color: Colors.green.shade50,
                  width: double.infinity,
                  child: const Text(
                    'Header (fixed)',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
                Expanded(
                  // ← SỬA: Expanded cho ListView
                  child: ListView.builder(
                    itemCount: 20,
                    itemBuilder: (context, index) =>
                        ListTile(title: Text('Item $index')),
                  ),
                ),
              ],
            ),
          ),

          const SizedBox(height: 24),

          _buildErrorBox(
            title: '❌ LỖI: Expanded trong Column TRONG Column',
            description:
                'Column cha cho Column con unbounded height → Expanded không biết "remaining space" là bao nhiêu.',
            code: '''
Column(
  children: [
    Column(           // ← maxHeight = INFINITY
      children: [
        Expanded(...) // ← LỖI! INFINITY - ? = ???
      ],
    ),
  ],
)''',
          ),
          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA: Wrap Column con bằng Expanded',
            description: 'Expanded ở Column cha giới hạn height cho Column con.',
          ),
          Container(
            height: 150,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Column(
              children: [
                const Padding(
                  padding: EdgeInsets.all(8),
                  child: Text('Column cha - Header'),
                ),
                Expanded(
                  // ← SỬA: Bọc Column con
                  child: Column(
                    children: [
                      Expanded(
                        child: Container(
                          color: Colors.green.shade100,
                          child: const Center(
                            child: Text('Expanded trong Column con ✅'),
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// ==========================================
// ERROR 2: RenderFlex Overflow
// ==========================================
class _RenderFlexOverflowDemo extends StatelessWidget {
  const _RenderFlexOverflowDemo();

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _buildErrorBox(
            title: '❌ LỖI: Text quá dài trong Row (không Expanded)',
            description:
                'Text không bị giới hạn width → tràn ra ngoài Row.\n'
                'Error: "A RenderFlex overflowed by XX pixels on the right."',
            code: '''
Row(
  children: [
    Icon(Icons.info),
    Text('Đoạn text rất rất dài...'), // ← TRÀN!
  ],
)''',
          ),
          const SizedBox(height: 8),
          // Demo lỗi (có sọc vàng đen)
          Container(
            decoration: BoxDecoration(
              border: Border.all(color: Colors.red.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            clipBehavior: Clip.hardEdge,
            child: Row(
              children: [
                const Icon(Icons.warning, color: Colors.red),
                const SizedBox(width: 8),
                // Text quá dài nhưng KHÔNG overflow vì có clipBehavior
                Container(
                  color: Colors.red.shade50,
                  padding: const EdgeInsets.all(8),
                  child: const Text(
                    'Đoạn text rất rất dài mà không wrap sẽ bị tràn... (demo clip)',
                    style: TextStyle(color: Colors.red),
                  ),
                ),
              ],
            ),
          ),

          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA 1: Dùng Expanded',
            description: 'Expanded giới hạn width cho Text → tự wrap.',
          ),
          Container(
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                const Icon(Icons.check_circle, color: Colors.green),
                const SizedBox(width: 8),
                Expanded(
                  // ← SỬA: Expanded
                  child: Text(
                    'Đoạn text rất rất dài nhưng giờ sẽ tự xuống dòng vì có Expanded bọc bên ngoài!',
                    style: TextStyle(color: Colors.green.shade700),
                  ),
                ),
              ],
            ),
          ),

          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA 2: Dùng Flexible + ellipsis',
            description: 'Flexible + TextOverflow.ellipsis → cắt bớt text.',
          ),
          Container(
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                const Icon(Icons.check_circle, color: Colors.green),
                const SizedBox(width: 8),
                Flexible(
                  // ← Flexible
                  child: Text(
                    'Đoạn text rất rất dài nhưng sẽ bị cắt bớt với dấu ba chấm...',
                    overflow: TextOverflow.ellipsis, // ← ellipsis
                    style: TextStyle(color: Colors.green.shade700),
                  ),
                ),
              ],
            ),
          ),

          const SizedBox(height: 24),
          _buildErrorBox(
            title: '❌ LỖI: Quá nhiều widget trong Row',
            description: 'Nhiều widget cố định kích thước → tổng width > parent width.',
            code: '''
Row(
  children: [
    Container(width: 150, ...),
    Container(width: 150, ...),
    Container(width: 150, ...), // ← Tổng 450 > 390 (screen)
  ],
)''',
          ),
          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA: Dùng Wrap thay Row',
            description: 'Wrap tự xuống dòng khi hết chỗ.',
          ),
          Wrap(
            spacing: 8,
            runSpacing: 8,
            children: List.generate(5, (index) {
              return Container(
                width: 150,
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: Colors.green.shade100,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.green.shade300),
                ),
                child: Text('Item $index', textAlign: TextAlign.center),
              );
            }),
          ),
        ],
      ),
    );
  }
}

// ==========================================
// ERROR 3: ParentData Error
// ==========================================
class _ParentDataDemo extends StatelessWidget {
  const _ParentDataDemo();

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _buildErrorBox(
            title: '❌ LỖI: Expanded KHÔNG trực tiếp trong Row/Column',
            description:
                'Expanded phải là con TRỰC TIẾP của Row/Column/Flex.\n'
                'Error: "Incorrect use of ParentDataWidget."',
            code: '''
Column(
  children: [
    Container(        // ← Container chen giữa!
      child: Expanded(  // ← LỖI: không trực tiếp trong Column
        child: Text('text'),
      ),
    ),
  ],
)''',
          ),
          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA: Đặt Expanded trực tiếp trong Column',
            description: 'Expanded phải là con trực tiếp, Container vào bên trong.',
          ),
          Container(
            height: 150,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Column(
              children: [
                Container(
                  padding: const EdgeInsets.all(8),
                  color: Colors.green.shade50,
                  width: double.infinity,
                  child: const Text('Fixed header'),
                ),
                Expanded(
                  // ← Trực tiếp trong Column ✅
                  child: Container(
                    // Container bên TRONG Expanded
                    color: Colors.green.shade100,
                    child: const Center(
                      child: Text('Expanded > Container ✅'),
                    ),
                  ),
                ),
              ],
            ),
          ),

          const SizedBox(height: 24),

          _buildErrorBox(
            title: '❌ LỖI: Positioned KHÔNG trong Stack',
            description:
                'Positioned chỉ hoạt động trong Stack.\n'
                'Error: "Incorrect use of ParentDataWidget."',
            code: '''
Column(
  children: [
    Positioned(       // ← LỖI: không trong Stack!
      top: 0,
      child: Text('text'),
    ),
  ],
)''',
          ),
          const SizedBox(height: 16),
          _buildFixBox(
            title: '✅ SỬA: Dùng Positioned trong Stack',
            description: 'Positioned chỉ dùng trong Stack.',
          ),
          Container(
            height: 150,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.green.shade300, width: 2),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Stack(
              children: [
                Container(color: Colors.green.shade50),
                Positioned(
                  // ← Trong Stack ✅
                  top: 8,
                  left: 8,
                  child: Container(
                    padding: const EdgeInsets.all(8),
                    decoration: BoxDecoration(
                      color: Colors.green,
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: const Text(
                      'Positioned trong Stack ✅',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
                const Positioned(
                  bottom: 8,
                  right: 8,
                  child: Text('bottom-right'),
                ),
              ],
            ),
          ),

          const SizedBox(height: 24),

          // Bảng tóm tắt quy tắc
          Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.blue.shade50,
              borderRadius: BorderRadius.circular(8),
              border: Border.all(color: Colors.blue.shade200),
            ),
            child: const Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '📋 Quy tắc ParentData:',
                  style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                ),
                SizedBox(height: 8),
                Text('• Expanded → con trực tiếp của Row/Column/Flex'),
                Text('• Flexible → con trực tiếp của Row/Column/Flex'),
                Text('• Spacer → con trực tiếp của Row/Column/Flex'),
                Text('• Positioned → con trực tiếp của Stack'),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// === Helper widgets ===
Widget _buildErrorBox({
  required String title,
  required String description,
  required String code,
}) {
  return Container(
    width: double.infinity,
    padding: const EdgeInsets.all(12),
    decoration: BoxDecoration(
      color: Colors.red.shade50,
      borderRadius: BorderRadius.circular(8),
      border: Border.all(color: Colors.red.shade200),
    ),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          title,
          style: TextStyle(
            fontWeight: FontWeight.bold,
            color: Colors.red.shade700,
          ),
        ),
        const SizedBox(height: 8),
        Text(description),
        const SizedBox(height: 8),
        Container(
          width: double.infinity,
          padding: const EdgeInsets.all(8),
          decoration: BoxDecoration(
            color: Colors.grey.shade900,
            borderRadius: BorderRadius.circular(4),
          ),
          child: Text(
            code,
            style: const TextStyle(
              fontFamily: 'monospace',
              fontSize: 12,
              color: Colors.white70,
            ),
          ),
        ),
      ],
    ),
  );
}

Widget _buildFixBox({
  required String title,
  required String description,
}) {
  return Container(
    width: double.infinity,
    padding: const EdgeInsets.all(12),
    margin: const EdgeInsets.only(bottom: 8),
    decoration: BoxDecoration(
      color: Colors.green.shade50,
      borderRadius: BorderRadius.circular(8),
      border: Border.all(color: Colors.green.shade200),
    ),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          title,
          style: TextStyle(
            fontWeight: FontWeight.bold,
            color: Colors.green.shade700,
          ),
        ),
        const SizedBox(height: 4),
        Text(description),
      ],
    ),
  );
}
```

### Giải thích

```
Tab 1 — Unbounded Height:
  ❌ ListView trong Column → Column cho INFINITY height → LỖI
  ✅ Sửa: Expanded bọc ListView
  ❌ Expanded trong Column con (Column trong Column) → INFINITY
  ✅ Sửa: Expanded bọc Column con

Tab 2 — RenderFlex Overflow:
  ❌ Text dài trong Row → tràn ra ngoài
  ✅ Sửa 1: Expanded → text tự wrap
  ✅ Sửa 2: Flexible + ellipsis → cắt bớt
  ❌ Nhiều fixed widgets > parent width
  ✅ Sửa: Dùng Wrap thay Row

Tab 3 — ParentData Error:
  ❌ Expanded không trực tiếp trong Row/Column
  ✅ Sửa: Đặt Expanded trực tiếp
  ❌ Positioned không trong Stack
  ✅ Sửa: Đặt Positioned trong Stack

Mỗi error demo cả lý thuyết (code lỗi) + thực tế (code sửa chạy được).
```

---

### Ví dụ bonus: CustomScrollView

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      expandedHeight: 200,
      floating: true,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('CustomScrollView Demo'),
        background: Image.network(
          'https://picsum.photos/800/400',
          fit: BoxFit.cover,
        ),
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(
          leading: CircleAvatar(child: Text('${index + 1}')),
          title: Text('Item $index'),
          subtitle: Text('Subtitle for item $index'),
        ),
        childCount: 50,
      ),
    ),
  ],
)
```

> 📖 **Khi nào dùng?** Khi cần header co giãn (collapsing toolbar) + list hiệu năng cao. Sliver = lazy rendering cho scroll performance.
