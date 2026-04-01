# Buổi 04: Layout System — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 4/16** · **Thời lượng tự học:** ~1 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 03 (lý thuyết + ít nhất BT1-BT2)

> **Mục tiêu:** Hiểu cách Flutter layout UI từ gốc — Constraints model, các layout widget, Flex system, scrollable widgets, responsive design, và cách xử lý layout errors.

> ⚠️ **Yêu cầu Flutter ≥ 3.27**: API `Color.withValues(alpha:)` chỉ có từ Flutter 3.27+. Nếu dùng version cũ hơn, dùng `Color.withOpacity()` thay thế.

---

## 1. Constraints Model — Nguyên tắc cốt lõi 🔴

> ⚠️ **Đây là phần QUAN TRỌNG NHẤT của buổi học.** Nếu bạn chỉ nhớ một thứ, hãy nhớ phần này.

### 1.1. Ba quy tắc vàng

Flutter layout hoạt động theo **3 quy tắc**:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   1. Constraints go DOWN    (Ràng buộc đi xuống)    │
│   2. Sizes go UP            (Kích thước đi lên)     │
│   3. Parent sets position   (Cha đặt vị trí)        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### Quy trình layout chi tiết:

```
         Parent Widget
         ┌───────────────────────────┐
         │                           │
         │  ① "Con ơi, con có thể    │
         │     rộng 100-300px,       │
         │     cao 50-200px"         │
         │         │                 │
         │         ▼ Constraints     │
         │    ┌──────────┐           │
         │    │  Child    │           │
         │    │  Widget   │           │
         │    └──────────┘           │
         │         │                 │
         │         ▼ Size            │
         │  ② "Con chọn rộng 200px,  │
         │     cao 100px"            │
         │                           │
         │  ③ Parent quyết định      │
         │     đặt child ở (50, 25)  │
         └───────────────────────────┘
```

**Bước 1 — Constraints go DOWN:** Parent gửi constraints (giới hạn min/max width, min/max height) cho child. Child **không thể** vượt quá constraints này.

**Bước 2 — Sizes go UP:** Child quyết định kích thước của mình (trong phạm vi constraints nhận được) rồi báo lại cho parent.

**Bước 3 — Parent sets position:** Parent nhận kích thước child, rồi **parent** quyết định đặt child ở đâu. Child **không biết** và **không quyết định** vị trí của mình.

> 💡 **Quan trọng:** Một widget **không biết** và **không quyết định** vị trí của nó trên màn hình. Vị trí do parent quyết định.

### 1.2. BoxConstraints

Mọi layout trong Flutter đều dựa trên `BoxConstraints`:

```dart
BoxConstraints(
  minWidth: 0.0,     // Chiều rộng tối thiểu
  maxWidth: 300.0,   // Chiều rộng tối đa
  minHeight: 0.0,    // Chiều cao tối thiểu
  maxHeight: 200.0,  // Chiều cao tối đa
)
```

#### Tight Constraints vs Loose Constraints

```
TIGHT Constraints (Ràng buộc chặt)
──────────────────────────────────
minWidth == maxWidth && minHeight == maxHeight
→ Child BẮT BUỘC phải có kích thước chính xác này
→ Child KHÔNG có quyền chọn

Ví dụ: BoxConstraints.tight(Size(100, 50))
→ minWidth = 100, maxWidth = 100
→ minHeight = 50, maxHeight = 50
→ Child PHẢI là 100x50


LOOSE Constraints (Ràng buộc lỏng)
──────────────────────────────────
minWidth = 0 && minHeight = 0
→ Child có thể nhỏ tùy ý (nhưng không vượt max)
→ Child CÓ quyền chọn kích thước

Ví dụ: BoxConstraints.loose(Size(300, 200))
→ minWidth = 0, maxWidth = 300
→ minHeight = 0, maxHeight = 200
→ Child có thể từ 0x0 đến 300x200
```

#### Ví dụ trực quan:

```
Screen (Tight: 390x844)
│
├─ Scaffold (Tight: 390x844)
│  │
│  ├─ AppBar (Tight width: 390, Loose height: 0-56)
│  │  → AppBar chọn height = 56
│  │
│  └─ Body (Loose: 0-390 x 0-788)
│     │
│     └─ Center (Loose: 0-390 x 0-788)
│        │
│        └─ Container(width: 200, height: 100)
│           → Chọn 200x100 (trong phạm vi cho phép)
│           → Center đặt ở giữa: ((390-200)/2, (788-100)/2)
```

### 1.3. Widget có kích thước riêng không?

> **KHÔNG.** Widget **không có kích thước cố định**. Kích thước cuối cùng phụ thuộc vào constraints từ parent.

```dart
// Container này KHÔNG phải luôn 200x200!
Container(
  width: 200,  // Chỉ là "mong muốn" (desired size)
  height: 200,
  color: Colors.red,
)
```

Nếu parent chỉ cho phép maxWidth = 100, Container sẽ là **100x200** (width bị giới hạn).

### 1.4. Quy trình layout đầy đủ

```
                    Screen
                      │
            ┌─────────┴──────────┐
            │  constraints:       │
            │  tight(390 x 844)  │
            ▼                    │
         Scaffold               │
            │                    │
    ┌───────┼───────┐           │
    │       │       │           │
    ▼       ▼       ▼           │
  AppBar  Body   BottomNav      │
    │       │       │           │
    │    Center     │      sizes go UP
    │       │       │           │
    │   Container   │           │
    │   (200x100)   │           │
    │       │       │           │
    └───────┼───────┘           │
            │                    │
            │  "Tôi là 390x844" │
            └────────────────────┘
```

> ⚠️ **FE Trap:** FE dev thường áp CSS mental model — child tự quyết size (`width: 200px`), parent chỉ chứa. Flutter **ngược lại**: parent gửi constraints `(minW, maxW, minH, maxH)` → child chọn size TRONG constraints đó. Quên điều này = 90% lỗi layout Flutter ban đầu.

---

## 2. Single-child Layout Widgets 🟡

Các widget chỉ có **một child**, dùng để điều chỉnh kích thước, vị trí, hoặc trang trí.

### 2.1. Container

Widget "đa năng" nhất — kết hợp decoration, padding, margin, constraints:

```dart
Container(
  width: 200,                    // desired width
  height: 100,                   // desired height
  margin: const EdgeInsets.all(16),    // khoảng cách BÊN NGOÀI
  padding: const EdgeInsets.all(12),   // khoảng cách BÊN TRONG
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
    boxShadow: [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 4,
        offset: Offset(0, 2),
      ),
    ],
  ),
  child: Text('Hello'),
)
```

> ⚠️ **Không dùng `color` và `decoration` cùng lúc.** Nếu cần color, đặt trong `decoration`.

**Container KHÔNG có child:**
- Cố gắng to nhất có thể (fill parent)

**Container CÓ child:**
- Co lại vừa đủ bọc child (+ padding/margin)

### 2.2. Padding

Chỉ thêm padding, không có gì khác:

```dart
Padding(
  padding: const EdgeInsets.symmetric(
    horizontal: 16.0,
    vertical: 8.0,
  ),
  child: Text('Content'),
)
```

> 💡 Nên dùng `Padding` thay vì `Container` khi chỉ cần padding — code rõ ràng hơn, performance tốt hơn.

### 2.3. Center

Đặt child ở giữa:

```dart
Center(
  child: Text('Ở giữa màn hình'),
)
```

`Center` thực chất là `Align(alignment: Alignment.center)`.

### 2.4. Align

Đặt child ở vị trí bất kỳ:

```dart
Align(
  alignment: Alignment.topRight,
  child: Text('Góc trên phải'),
)

// Hoặc dùng tọa độ tùy chỉnh (-1 đến 1)
Align(
  alignment: Alignment(0.5, -0.3), // x: 0.5, y: -0.3
  child: Icon(Icons.star),
)
```

```
Alignment map:
(-1,-1)────(0,-1)────(1,-1)
  │ topLeft  topCenter topRight
  │
(-1,0)─────(0,0)─────(1,0)
  │ centerLeft center centerRight
  │
(-1,1)─────(0,1)─────(1,1)
  bottomLeft bottomCenter bottomRight
```

### 2.5. SizedBox

Cung cấp kích thước cố định hoặc tạo khoảng trống:

```dart
// Kích thước cố định
SizedBox(
  width: 100,
  height: 50,
  child: ElevatedButton(
    onPressed: () {},
    child: Text('Button'),
  ),
)

// Tạo khoảng trống (CÁCH HAY NHẤT)
Column(
  children: [
    Text('Trên'),
    const SizedBox(height: 16),  // Khoảng trống 16px
    Text('Dưới'),
  ],
)

// Fill toàn bộ
SizedBox.expand(
  child: Container(color: Colors.red),
)
```

### 2.6. ConstrainedBox

Thêm constraints bổ sung:

```dart
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: 100,
    maxWidth: 300,
    minHeight: 50,
  ),
  child: Text('Ít nhất 100 rộng, tối đa 300'),
)
```

### 2.7. FractionallySizedBox

Kích thước theo **tỷ lệ phần trăm** của parent:

```dart
FractionallySizedBox(
  widthFactor: 0.8,   // 80% chiều rộng parent
  heightFactor: 0.5,  // 50% chiều cao parent
  child: Container(color: Colors.green),
)
```

### 2.8. AspectRatio

Giữ tỷ lệ khung hình:

```dart
AspectRatio(
  aspectRatio: 16 / 9,  // width / height
  child: Container(color: Colors.blue),
)
```

### Bảng tóm tắt — Khi nào dùng widget nào?

| Widget | Khi nào dùng |
|--------|-------------|
| `Container` | Cần kết hợp nhiều thứ: decoration + padding + size |
| `Padding` | Chỉ cần thêm padding |
| `Center` | Đặt ở giữa |
| `Align` | Đặt ở vị trí cụ thể |
| `SizedBox` | Kích thước cố định hoặc tạo spacing |
| `ConstrainedBox` | Thêm min/max constraints |
| `FractionallySizedBox` | Kích thước theo % parent |
| `AspectRatio` | Giữ tỷ lệ width/height |

> 🔗 **FE Bridge:** `Container` ≈ `<div>` với style — nhưng **khác ở**: Container KHÔNG tự co giãn theo content mặc định. Nếu không có child, Container expand hết constraints của parent. CSS div content-fit by default, Flutter Container constraints-fit by default.

---

## 3. Multi-child Layout Widgets 🟡

Các widget có **nhiều children**, dùng để sắp xếp layout phức tạp.

### 3.1. Row & Column

**Row** — sắp xếp children theo **chiều ngang** (horizontal):
**Column** — sắp xếp children theo **chiều dọc** (vertical):

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Icon(Icons.star),
    Text('Rating'),
    Text('4.5'),
  ],
)

Column(
  mainAxisAlignment: MainAxisAlignment.start,
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: [
    Text('Title'),
    const SizedBox(height: 8),
    Text('Subtitle'),
  ],
)
```

#### Main Axis vs Cross Axis

```
ROW:
main axis (horizontal) →→→→→→→→→→→→
cross axis (vertical) ↓
                      ↓
                      ↓

COLUMN:
main axis (vertical) ↓
                     ↓
                     ↓
cross axis (horizontal) →→→→→→→→→→
```

#### MainAxisAlignment

```
start:         [A][B][C]
end:                        [A][B][C]
center:              [A][B][C]
spaceBetween:  [A]      [B]      [C]
spaceAround:    [A]    [B]    [C]
spaceEvenly:     [A]     [B]     [C]
```

#### CrossAxisAlignment

```
(Trong Row, cross axis là vertical)

start:    [A]        center:      [A]       end:            [A]
          [BB]                  [BB]                     [BB]
          [CCC]                [CCC]                    [CCC]

stretch: [AAAA]
         [BBBB]
         [CCCC]
```

#### mainAxisSize

```dart
Row(
  mainAxisSize: MainAxisSize.min, // Co lại vừa đủ chứa children
  // MainAxisSize.max (default) — chiếm hết không gian có thể
  children: [...],
)
```

> 🔗 **FE Bridge:** `Row`/`Column` ≈ `display: flex` + `flex-direction` — nhưng **khác ở**: Flutter KHÔNG có `gap` property, phải dùng `SizedBox` hoặc `Spacer`. Và khi child vượt quá space, Flutter **throw overflow error** thay vì wrap/scroll như CSS flex.

### 3.2. Stack

Xếp children **chồng lên nhau** (như z-index trong CSS):

```dart
Stack(
  children: [
    // Widget đầu tiên ở DƯỚI CÙNG
    Image.network('https://picsum.photos/800/400'),

    // Widget sau ở TRÊN
    Positioned(
      bottom: 16,
      left: 16,
      child: Text(
        'Overlay Text',
        style: TextStyle(color: Colors.white),
      ),
    ),

    // Badge ở góc
    Positioned(
      top: 8,
      right: 8,
      child: CircleAvatar(
        radius: 12,
        child: Text('3'),
      ),
    ),
  ],
)
```

#### Positioned

Đặt vị trí tuyệt đối trong Stack:

```dart
Positioned(
  top: 10,      // cách top Stack 10px
  left: 20,     // cách left Stack 20px
  // right: ..., bottom: ...
  // width: ..., height: ...
  child: Widget(),
)

// Fill toàn bộ Stack
Positioned.fill(
  child: Container(color: Colors.black54),
)
```

> 🔗 **FE Bridge:** `Stack` + `Positioned` ≈ `position: relative` + `absolute` trong CSS — behavior gần tương đương. Nhưng Flutter Stack mặc định **không có kích thước**, size = child lớn nhất (non-positioned).

### 3.3. Wrap

Tự động **xuống dòng** khi hết chỗ (như `flex-wrap: wrap` trong CSS):

```dart
Wrap(
  spacing: 8,         // khoảng cách ngang giữa children
  runSpacing: 8,      // khoảng cách dọc giữa các dòng
  children: [
    Chip(label: Text('Flutter')),
    Chip(label: Text('Dart')),
    Chip(label: Text('Mobile')),
    Chip(label: Text('Cross-platform')),
    Chip(label: Text('UI')),
  ],
)
```

```
Khi đủ chỗ:  [Flutter] [Dart] [Mobile] [Cross-platform] [UI]

Khi hết chỗ:  [Flutter] [Dart] [Mobile]
              [Cross-platform] [UI]
```

### 3.4. ListView & GridView (tổng quan)

*(Chi tiết trong phần 5 — Scrollable widgets)*

```dart
// ListView — danh sách dọc
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
  ],
)

// GridView — lưới
GridView.count(
  crossAxisCount: 2,
  children: [
    Card(child: Center(child: Text('1'))),
    Card(child: Center(child: Text('2'))),
    Card(child: Center(child: Text('3'))),
    Card(child: Center(child: Text('4'))),
  ],
)
```

---

## 4. Flex System — Expanded, Flexible, Spacer 🔴

### 4.1. Cách phân bổ không gian

Row và Column phân bổ không gian theo 2 bước:

```
Bước 1: Layout các widget KHÔNG có flex (kích thước cố định)
Bước 2: Phân bổ KHÔNG GIAN CÒN LẠI cho các widget có flex

Tổng không gian: 300px
┌──────────────────────────────────────────────────┐
│  [Icon 40px]  [???  flex:1  ???] [Button 80px]   │
└──────────────────────────────────────────────────┘

Bước 1: Icon (40px) + Button (80px) = 120px
Bước 2: Còn lại 300 - 120 = 180px → cho flex widget
```

### 4.2. Expanded

**Bắt buộc** child chiếm hết không gian được phân bổ:

```dart
Row(
  children: [
    // Chiếm 1/3 không gian còn lại
    Expanded(
      flex: 1,  // default = 1
      child: Container(color: Colors.red),
    ),
    // Chiếm 2/3 không gian còn lại
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue),
    ),
  ],
)
```

```
Tổng flex = 1 + 2 = 3
Không gian còn lại = 300px

┌──────────────────────────────────────────────────┐
│  [  RED: 100px  ]  [     BLUE: 200px            ]│
│   (300 * 1/3)       (300 * 2/3)                  │
└──────────────────────────────────────────────────┘
```

#### Ví dụ phức tạp hơn:

```dart
Row(
  children: [
    Icon(Icons.avatar, size: 48),   // 48px cố định
    const SizedBox(width: 12),            // 12px cố định
    Expanded(                        // Chiếm TOÀN BỘ còn lại
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('Nguyễn Văn A'),
          Text('Flutter Developer'),
        ],
      ),
    ),
    IconButton(                      // ~48px cố định
      icon: Icon(Icons.more_vert),
      onPressed: () {},
    ),
  ],
)
```

```
┌─────────────────────────────────────────────────────────┐
│ [Avatar 48] [12] [  Expanded: text fills rest  ] [48]   │
│              gap                                  icon   │
└─────────────────────────────────────────────────────────┘
```

### 4.3. Flexible

Giống Expanded nhưng child **không bắt buộc** phải fill hết:

```dart
Row(
  children: [
    Flexible(
      flex: 1,
      fit: FlexFit.loose,   // child có thể NHỎ HƠN không gian phân bổ
      child: Text('Short'),
    ),
    Flexible(
      flex: 2,
      fit: FlexFit.tight,   // GIỐNG Expanded — bắt buộc fill hết
      child: Container(color: Colors.blue),
    ),
  ],
)
```

```
Expanded = Flexible(fit: FlexFit.tight)  ← Bắt buộc fill hết
Flexible = Flexible(fit: FlexFit.loose)  ← Có thể nhỏ hơn
```

```
Expanded (tight):
┌──────────────────────────────┐
│ [█████ fills entire space ██]│
└──────────────────────────────┘

Flexible (loose):
┌──────────────────────────────┐
│ [██ chỉ lớn bằng content]   │
└──────────────────────────────┘
```

### 4.4. Spacer

Tạo khoảng trống linh hoạt (Expanded + SizedBox.shrink):

```dart
Row(
  children: [
    Text('Left'),
    Spacer(),          // Đẩy các widget ra 2 bên
    Text('Right'),
  ],
)

// Tương đương
Row(
  children: [
    Text('Left'),
    Expanded(child: SizedBox.shrink()),
    Text('Right'),
  ],
)
```

```
[Left]                              [Right]
       ←── Spacer fills gap ──→
```

### 4.5. So sánh Flex system — Flutter vs CSS Flexbox

| CSS Flexbox | Flutter |
|-------------|---------|
| `display: flex` | `Row` / `Column` |
| `flex-direction: row` | `Row` |
| `flex-direction: column` | `Column` |
| `flex: 1` | `Expanded(flex: 1)` |
| `flex-grow: 1; flex-shrink: 0` | `Flexible(flex: 1)` |
| `justify-content` | `mainAxisAlignment` |
| `align-items` | `crossAxisAlignment` |
| `flex-wrap: wrap` | Dùng `Wrap` thay vì Row/Column |
| `gap: 8px` | Không có — dùng `SizedBox` giữa children |

> 🔗 **FE Bridge:** `Expanded` ≈ `flex: 1`, `Flexible` ≈ `flex-grow` + `flex-shrink` — nhưng **khác ở**: `Expanded` PHẢI nằm trong `Row`/`Column`/`Flex`, nếu đặt ngoài → runtime error. CSS flex-grow hoạt động trên mọi flex child.

---

> 💼 **Gặp trong dự án:** Bottom navigation bar (icon + label chia đều), dashboard card grid (2-3 cột, chiều cao tự điều chỉnh), form layout (label cố định + input chiếm phần còn lại)
> 🤖 **Keywords bắt buộc trong prompt:** `Row/Column with Expanded`, `flex ratio`, `mainAxisAlignment`, `crossAxisAlignment`, `IntrinsicHeight` khi cần

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Bottom nav bar:** 4-5 icon + text chia đều chiều ngang, highlight tab active — Expanded(flex: 1) cho mỗi item
- **Dashboard stats:** 2 cột card ở trên, 1 cột list ở dưới chiếm phần còn lại — Column + Row(Expanded) + Expanded(ListView)
- **Form row:** Label 120px cố định + TextField chiếm hết chiều rộng còn lại — Row(SizedBox(width:120) + Expanded(TextField))

**Tại sao cần các keyword trên:**
- **`Row/Column with Expanded`** — AI cần dùng đúng Expanded, không dùng Container(width: MediaQuery...) hardcode
- **`flex ratio`** — chỉ rõ tỉ lệ: "sidebar flex:1, content flex:3" thay vì để AI tự đoán
- **`mainAxisAlignment, crossAxisAlignment`** — nếu không nói rõ, AI hay dùng default (start) cho mọi thứ
- **`IntrinsicHeight`** — khi 2 column trong Row cần cùng chiều cao, AI hay quên widget này

**Prompt mẫu — Dashboard layout:**
```text
Tôi cần tạo Flutter layout cho Dashboard screen.
Context: app quản lý bán hàng, hiển thị stats + danh sách đơn hàng gần đây.
Tech stack: Flutter 3.x, Dart 3.x.
Layout yêu cầu:
- Row trên cùng: 3 stat cards chia đều (Expanded flex:1), height 120, spacing 12.
- Card có: icon + số lớn + label nhỏ, căn giữa, có border radius 12 + shadow nhẹ.
- Dưới cards: Expanded chứa ListView.builder danh sách đơn hàng (OrderTile).
- Padding tổng thể 16, gap giữa stats row và list = 16.
Constraints:
- Dùng Column > Row(Expanded × 3) > Expanded(ListView.builder).
- const constructor cho widget static.
- KHÔNG dùng hardcode width/height (trừ card height 120).
Output: 1 file dashboard_screen.dart hoàn chỉnh.
```

**Expected Output:** AI gen Column chứa Padding > Column > Row(3 × Expanded > Card) + Expanded(ListView.builder), với sample data.

⚠️ **Giới hạn AI hay mắc:** AI hay wrap stat card trong Container với width cố định thay vì Expanded. AI cũng có thể dùng shrinkWrap: true cho ListView thay vì Expanded — gây render toàn bộ items cùng lúc.

</details>

---

## 5. Scrollable Widgets 🟡

### 5.1. SingleChildScrollView

Bọc **một widget** để có thể scroll:

```dart
SingleChildScrollView(
  padding: const EdgeInsets.all(16),
  child: Column(
    children: [
      // Nhiều widget con...
      Text('Content 1'),
      Text('Content 2'),
      // ...
    ],
  ),
)
```

> ⚠️ `SingleChildScrollView` render **TẤT CẢ** children cùng lúc. Chỉ dùng khi biết content không quá dài. Với danh sách dài, dùng `ListView.builder`.

### 5.2. ListView

#### ListView cơ bản (render tất cả cùng lúc)

```dart
ListView(
  padding: const EdgeInsets.all(8),
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
  ],
)
```

#### ListView.builder (QUAN TRỌNG — lazy rendering)

Chỉ render các item **đang hiển thị trên màn hình**:

```dart
ListView.builder(
  itemCount: 1000,           // 1000 items
  itemBuilder: (context, index) {
    // CHỈ được gọi cho item đang visible!
    return ListTile(
      leading: CircleAvatar(child: Text('$index')),
      title: Text('Item $index'),
      subtitle: Text('Description for item $index'),
    );
  },
)
```

```
Màn hình hiển thị items 5-12:

    [Item 3]  ← đã bị destroy
    [Item 4]  ← đã bị destroy
╔═══════════════════════╗
║   [Item 5]   visible  ║
║   [Item 6]   visible  ║
║   [Item 7]   visible  ║
║   [Item 8]   visible  ║
║   [Item 9]   visible  ║
║   [Item 10]  visible  ║
║   [Item 11]  visible  ║
║   [Item 12]  visible  ║
╚═══════════════════════╝
    [Item 13] ← chưa tạo
    [Item 14] ← chưa tạo
```

#### ListView.separated (có divider)

```dart
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => Divider(),
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
)
```

### 5.3. GridView

#### GridView.count (số cột cố định)

```dart
GridView.count(
  crossAxisCount: 2,          // 2 cột
  crossAxisSpacing: 8,        // khoảng cách ngang
  mainAxisSpacing: 8,         // khoảng cách dọc
  childAspectRatio: 1.0,      // tỷ lệ width/height
  children: [
    Card(child: Center(child: Text('1'))),
    Card(child: Center(child: Text('2'))),
    Card(child: Center(child: Text('3'))),
    Card(child: Center(child: Text('4'))),
  ],
)
```

#### GridView.extent (chiều rộng tối đa mỗi item)

```dart
GridView.extent(
  maxCrossAxisExtent: 200,    // mỗi item tối đa 200px rộng
  crossAxisSpacing: 8,
  mainAxisSpacing: 8,
  children: [...],
)
```

#### GridView.builder (lazy rendering cho grid)

```dart
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 3,
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
  ),
  itemCount: 100,
  itemBuilder: (context, index) {
    return Card(
      child: Center(child: Text('Item $index')),
    );
  },
)
```

### 5.4. CustomScrollView + Slivers (Giới thiệu)

`CustomScrollView` cho phép kết hợp nhiều loại scrollable widget trong **một scroll view**:

```dart
CustomScrollView(
  slivers: [
    // AppBar co giãn khi scroll
    SliverAppBar(
      expandedHeight: 200,
      floating: false,
      pinned: true,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('My App'),
        background: Image.network(
          'https://picsum.photos/800/400',
          fit: BoxFit.cover,
        ),
      ),
    ),

    // Grid section
    SliverGrid(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => Card(
          child: Center(child: Text('Grid $index')),
        ),
        childCount: 6,
      ),
    ),

    // List section
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(
          title: Text('List item $index'),
        ),
        childCount: 20,
      ),
    ),
  ],
)
```

```
╔══════════════════════════════╗
║  ┌────────────────────────┐  ║
║  │   SliverAppBar          │  ║  ← Co giãn khi scroll
║  │   (expandedHeight: 200) │  ║
║  └────────────────────────┘  ║
║  ┌──────┐  ┌──────┐         ║
║  │Grid 0│  │Grid 1│         ║  ← SliverGrid
║  └──────┘  └──────┘         ║
║  ┌──────┐  ┌──────┐         ║
║  │Grid 2│  │Grid 3│         ║
║  └──────┘  └──────┘         ║
║  ┌────────────────────────┐  ║
║  │ List item 0             │  ║  ← SliverList
║  │ List item 1             │  ║
║  │ List item 2             │  ║
║  │ ...                     │  ║
║  └────────────────────────┘  ║
╚══════════════════════════════╝

Tất cả scroll CÙNG NHAU trong một CustomScrollView!
```

> 💡 Slivers là cách Flutter tối ưu scroll performance. Bạn sẽ dùng nhiều khi build app thực tế.

> 🔗 **FE Bridge:** `ListView.builder` ≈ virtualized list (react-window/react-virtuoso) — Flutter **mặc định** lazy rendering, FE phải cài thêm library. `ListView.builder` chỉ render items visible trên screen, giống `react-window` nhưng built-in.

---

## 6. Responsive Design 🟡

### 6.1. MediaQuery

Lấy thông tin về màn hình:

```dart
@override
Widget build(BuildContext context) {
  final size = MediaQuery.of(context).size;
  final width = size.width;
  final height = size.height;
  final padding = MediaQuery.of(context).padding; // safe area
  final orientation = MediaQuery.of(context).orientation;

  return Container(
    width: width * 0.8,  // 80% chiều rộng màn hình
    child: Text('Screen: ${width}x$height'),
  );
}
```

### 6.2. LayoutBuilder (QUAN TRỌNG)

Biết **chính xác** constraints mà parent cung cấp:

```dart
LayoutBuilder(
  builder: (context, constraints) {
    // constraints.maxWidth = chiều rộng tối đa parent cho phép

    if (constraints.maxWidth > 600) {
      // Tablet/Desktop — layout 2 cột
      return Row(
        children: [
          Expanded(flex: 1, child: SideMenu()),
          Expanded(flex: 2, child: MainContent()),
        ],
      );
    } else {
      // Phone — layout 1 cột
      return Column(
        children: [
          MainContent(),
        ],
      );
    }
  },
)
```

> 💡 **LayoutBuilder vs MediaQuery:**
> - `MediaQuery` — kích thước **toàn màn hình**
> - `LayoutBuilder` — kích thước **vùng mà parent cho phép** (tốt hơn cho reusable widgets)

### 6.3. OrientationBuilder

Phản ứng khi xoay màn hình:

```dart
OrientationBuilder(
  builder: (context, orientation) {
    return GridView.count(
      crossAxisCount: orientation == Orientation.portrait ? 2 : 4,
      children: [...],
    );
  },
)
```

### 6.4. Breakpoints Pattern

Tạo hệ thống breakpoints như CSS media queries:

```dart
class ScreenSize {
  static bool isPhone(BuildContext context) =>
      MediaQuery.of(context).size.width < 600;

  static bool isTablet(BuildContext context) =>
      MediaQuery.of(context).size.width >= 600 &&
      MediaQuery.of(context).size.width < 1200;

  static bool isDesktop(BuildContext context) =>
      MediaQuery.of(context).size.width >= 1200;
}

// Sử dụng
@override
Widget build(BuildContext context) {
  if (ScreenSize.isPhone(context)) {
    return PhoneLayout();
  } else if (ScreenSize.isTablet(context)) {
    return TabletLayout();
  } else {
    return DesktopLayout();
  }
}
```

> 🔗 **FE Bridge:** `MediaQuery` ≈ CSS `@media` queries, `LayoutBuilder` ≈ CSS Container Queries — nhưng **khác ở**: Flutter không có CSS breakpoint system. Phải tự define breakpoints bằng code, không có declarative responsive như CSS.

---

## 7. Common Layout Errors 🔴

### 7.1. "Unbounded height" — Column trong Column

#### Lỗi:

```
══╡ EXCEPTION CAUGHT BY RENDERING LIBRARY ╞══
RenderFlex children have non-zero flex but incoming height constraints
are unbounded.
```

#### Nguyên nhân:

```dart
// ❌ LỖI: Column trong Column — Column con không biết chiều cao
Column(
  children: [
    Column(         // Column con nhận maxHeight = INFINITY
      children: [   // → Không thể phân bổ flex
        Expanded(child: Container()), // Expanded cần bounded height!
      ],
    ),
  ],
)
```

```
Column cha
│ constraints: maxHeight = 800
│
└─ Column con
   │ constraints: maxHeight = INFINITY ← VẤN ĐỀ!
   │ (vì Column cha cho Column con scroll/grow tự do)
   │
   └─ Expanded
      → "Tôi cần chiếm hết không gian còn lại"
      → "Còn lại = INFINITY - ??? = LỖI!" 💥
```

#### Cách sửa:

```dart
// ✅ Cách 1: Wrap Column con bằng Expanded
Column(
  children: [
    Expanded(              // Giới hạn height cho Column con
      child: Column(
        children: [
          Expanded(child: Container()),
        ],
      ),
    ),
  ],
)

// ✅ Cách 2: Dùng SizedBox giới hạn height
Column(
  children: [
    SizedBox(
      height: 300,         // Giới hạn height rõ ràng
      child: Column(
        children: [
          Expanded(child: Container()),
        ],
      ),
    ),
  ],
)

// ✅ Cách 3: Dùng shrinkWrap (CHỈ cho list/column ngắn!)
Column(
  children: [
    ListView(
      shrinkWrap: true,                      // Co lại vừa đủ content
      physics: NeverScrollableScrollPhysics(), // Tắt scroll riêng
      children: [...],
    ),
  ],
)

// ✅ Cách 4 (Production): Dùng CustomScrollView + SliverList
CustomScrollView(
  slivers: [
    SliverToBoxAdapter(child: Text('Header')),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: 100,
      ),
    ),
  ],
)
```

> ⚠️ **Lưu ý:** `Flexible` (loose fit) có thể tránh lỗi nhưng không thực sự giải quyết vấn đề — child có thể collapse về height 0. Với danh sách dài, **luôn dùng `Expanded` + `ListView.builder`** hoặc **`CustomScrollView` + `SliverList`**.

### 7.2. "RenderFlex overflowed" — Content quá rộng/cao

#### Lỗi:

```
══╡ EXCEPTION CAUGHT BY RENDERING LIBRARY ╞══
A RenderFlex overflowed by 42 pixels on the right.

The overflowing RenderFlex has an orientation of Axis.horizontal.
```

Bạn sẽ thấy **sọc vàng đen** (yellow-black stripes) ở cạnh bị tràn.

#### Nguyên nhân:

```dart
// ❌ LỖI: Nội dung rộng hơn màn hình
Row(
  children: [
    Text('Đây là một đoạn text rất dài mà không có wrap nên sẽ bị tràn ra ngoài màn hình...'),
  ],
)
```

#### Cách sửa:

```dart
// ✅ Cách 1: Dùng Expanded để text tự wrap
Row(
  children: [
    Expanded(
      child: Text('Đây là đoạn text dài sẽ tự xuống dòng...'),
    ),
  ],
)

// ✅ Cách 2: Dùng Flexible
Row(
  children: [
    Flexible(
      child: Text(
        'Text dài...',
        overflow: TextOverflow.ellipsis, // Cắt bớt + dấu ...
      ),
    ),
  ],
)

// ✅ Cách 3: Cho phép scroll
SingleChildScrollView(
  scrollDirection: Axis.horizontal,
  child: Row(
    children: [
      Text('Text rất dài...'),
    ],
  ),
)
```

### 7.3. "Incorrect use of ParentDataWidget"

#### Lỗi:

```
══╡ EXCEPTION CAUGHT BY WIDGETS LIBRARY ╞══
Incorrect use of ParentDataWidget.
Expanded widgets must be placed directly inside Flex widgets.
```

#### Nguyên nhân:

```dart
// ❌ LỖI: Expanded KHÔNG nằm trực tiếp trong Row/Column
Column(
  children: [
    Container(              // Container chen giữa!
      child: Expanded(      // Expanded phải là con TRỰC TIẾP của Row/Column
        child: Text('text'),
      ),
    ),
  ],
)
```

#### Cách sửa:

```dart
// ✅ Expanded phải là con TRỰC TIẾP của Row/Column/Flex
Column(
  children: [
    Expanded(               // Trực tiếp trong Column
      child: Container(
        child: Text('text'),
      ),
    ),
  ],
)
```

#### Quy tắc nhớ:

```
Expanded  → PHẢI là con trực tiếp của Row / Column / Flex
Flexible  → PHẢI là con trực tiếp của Row / Column / Flex
Spacer    → PHẢI là con trực tiếp của Row / Column / Flex
Positioned → PHẢI là con trực tiếp của Stack
```

### 7.4. ListView trong Column (Unbounded Height variant)

#### Lỗi:

```
══╡ EXCEPTION CAUGHT BY RENDERING LIBRARY ╞══
Vertical viewport was given unbounded height.
```

#### Nguyên nhân & sửa:

```dart
// ❌ LỖI
Column(
  children: [
    Text('Header'),
    ListView(           // ListView cần biết height, nhưng Column cho INFINITY
      children: [...],
    ),
  ],
)

// ✅ Cách 1: Expanded
Column(
  children: [
    Text('Header'),
    Expanded(           // Giới hạn height cho ListView
      child: ListView(
        children: [...],
      ),
    ),
  ],
)

// ✅ Cách 2: shrinkWrap (CHỈ cho list ngắn!)
Column(
  children: [
    Text('Header'),
    ListView(
      shrinkWrap: true,     // ListView co lại vừa đủ content
      physics: NeverScrollableScrollPhysics(), // Tắt scroll riêng
      children: [...],
    ),
  ],
)
```

> ⚠️ `shrinkWrap: true` **render tất cả items cùng lúc** — chỉ dùng cho danh sách ngắn! Với list dài, **luôn dùng Expanded + ListView.builder**.

> 🆕 **Concept mới hoàn toàn:** "Unbounded height/width" error không có tương đương CSS. CSS tự handle overflow (scroll/hidden/visible). Flutter **crash** khi widget nhận infinite constraints mà không biết chọn size. Đây là lỗi #1 mà FE dev gặp khi bắt đầu Flutter.

---

> 💼 **Gặp trong dự án:** RenderFlex overflow khi content dài hơn màn hình, unbounded height khi nested scrollable, `Incorrect ParentDataWidget` khi Expanded nằm sai chỗ
> 🤖 **Keywords bắt buộc trong prompt:** `RenderFlex overflow`, `unbounded height`, `Expanded in Column`, `shrinkWrap performance`, `constraint debugging`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Layout overflow:** Text quá dài trong Row → vượt rộng màn hình, báo "overflowed by X pixels"
- **Nested scroll:** ListView trong Column → crash "unbounded height" — junior dev copy StackOverflow add `shrinkWrap: true` nhưng gây performance issue
- **ParentDataWidget:** Expanded nằm trong Container bọc bởi Column — báo lỗi parent data

**Tại sao cần các keyword trên:**
- **`RenderFlex overflow`** — AI cần biết đây là lỗi constraints, không phải lỗi logic
- **`unbounded height`** — AI phải giải thích flow: Column cho child Infinity → ListView cần biết height → crash
- **`Expanded in Column`** — AI cần chỉ rõ: Expanded phải là TRỰC TIẾP child của Row/Column/Flex
- **`constraint debugging`** — DebugPaintSizeEnabled, Widget Inspector — để dev biết cách inspect

**Prompt mẫu — Debug layout error:**
```text
Tôi gặp lỗi Flutter layout:
[paste error: "A RenderFlex overflowed by 42 pixels on the right."]

Code: Row chứa Image(width: 80) + Column(Text title dài + Text subtitle dài).

Context: danh sách sản phẩm, mỗi item = Row(image + info column).
Cần:
1. Giải thích tại sao xảy ra overflow (constraints flow).
2. Fix đúng cách — dùng Expanded/Flexible.
3. Cách phòng tránh: pattern chuẩn cho list item layout.
4. Giải thích Text overflow options: ellipsis, maxLines, softWrap.
```

**Expected Output:** AI giải thích Row → children sum width > parent width → overflow. Fix: wrap Column trong Expanded. Kèm text overflow options.

⚠️ **Giới hạn AI hay mắc:** AI hay suggest wrap Row trong SingleChildScrollView để "fix" overflow — SAI, vì list item không nên horizontal scroll. AI cũng hay suggest shrinkWrap mà không cảnh báo performance issue.

</details>

---

## 8. Best Practices & Lỗi thường gặp 🟡

### ✅ Nên làm

```dart
// 1. Dùng const cho widget không đổi
const SizedBox(height: 16)
const Padding(padding: EdgeInsets.all(8))

// 2. Dùng SizedBox cho spacing (không dùng Container rỗng)
Column(
  children: [
    Text('A'),
    const SizedBox(height: 16),  // ✅ Spacing rõ ràng
    Text('B'),
  ],
)

// 3. Dùng Padding thay Container khi chỉ cần padding
Padding(
  padding: const EdgeInsets.all(16),  // ✅ Rõ ý đồ
  child: Text('content'),
)

// 4. ListView.builder cho danh sách dài
ListView.builder(  // ✅ Lazy rendering
  itemCount: 10000,
  itemBuilder: (context, index) => ListTile(title: Text('$index')),
)

// 5. LayoutBuilder cho responsive widget
LayoutBuilder(  // ✅ Biết chính xác constraints
  builder: (context, constraints) {
    return constraints.maxWidth > 600
        ? TwoColumnLayout()
        : SingleColumnLayout();
  },
)
```

### ❌ Không nên làm

```dart
// 1. Nesting Container không cần thiết
Container(                 // ❌ Container thừa
  child: Container(        // ❌ Lại Container
    padding: const EdgeInsets.all(8),
    child: Text('text'),
  ),
)

// 2. Container rỗng cho spacing
Container(height: 16)     // ❌ Dùng SizedBox thay thế

// 3. ListView thường cho list dài
ListView(                  // ❌ Render tất cả 10000 items cùng lúc!
  children: List.generate(10000, (i) => ListTile(title: Text('$i'))),
)

// 4. shrinkWrap: true cho list dài
ListView(
  shrinkWrap: true,        // ❌ Mất lazy rendering, performance tệ
  children: [...veryLongList],
)

// 5. Hard-coded dimensions cho responsive
Container(
  width: 375,              // ❌ Chỉ đúng trên iPhone SE
  child: Text('content'),
)
```

---

## 9. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset (CSS) | Flutter Mindset | Tại sao khác |
|---|-------------------------|-----------------|---------------|
| 1 | Child quyết định size (`width: 200px`), parent chỉ chứa | Parent truyền constraints → child chọn size trong constraints | Flutter layout = parent-driven, CSS = child-driven |
| 2 | `overflow: scroll/hidden/auto` xử lý tràn | Phải wrap trong `SingleChildScrollView` hoặc `ListView` — không có auto overflow | Flutter không có CSS overflow property |
| 3 | `gap`, `margin`, `padding` là CSS properties | `gap` không tồn tại — dùng `SizedBox`/`Spacer`. Padding/Margin là Widget hoặc property | Mọi thứ trong Flutter đều là Widget hoặc Widget property |
| 4 | Responsive = `@media` queries + CSS Grid | Responsive = `MediaQuery` + `LayoutBuilder` + manual breakpoints | Không có declarative responsive system built-in |
| 5 | Flexbox child auto-wrap khi overflow | Row/Column **throw error** khi overflow — phải dùng `Wrap` widget | Flutter strict về layout constraints |

### Container vs div

```
CSS div:
- Mặc định display: block → chiếm full width, height tùy content
- Thêm bất kỳ CSS property nào qua style
- Nesting div thoải mái

Flutter Container:
- KHÔNG có child → to nhất có thể (full parent)
- CÓ child → co lại vừa đủ child
- Kết hợp sẵn: padding, margin, decoration, constraints
- Nhưng KHÔNG nên lạm dụng (dùng Padding, SizedBox khi đủ)
```

### CSS Flexbox vs Flutter Row/Column

```
CSS:
  .parent {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 8px;
  }
  .child { flex: 1; }

Flutter:
  Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    crossAxisAlignment: CrossAxisAlignment.center,
    children: [
      SizedBox(width: 8),  // Không có gap, phải thêm thủ công
      Expanded(flex: 1, child: ...),
    ],
  )
```

### CSS Grid vs Flutter GridView

```
CSS:
  .grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
  }

Flutter:
  GridView.count(
    crossAxisCount: 3,
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
    children: [...],
  )
```

### Quan điểm khác biệt cốt lõi

| Khía cạnh | CSS (Web) | Flutter |
|-----------|-----------|---------|
| Layout model | Box model (content, padding, border, margin) | Constraints model (constraints down, sizes up) |
| Sizing | CSS properties (width, height, %) | Widget properties + BoxConstraints |
| Positioning | position: absolute/relative/fixed | Stack + Positioned |
| Overflow | overflow: hidden/scroll/auto | ClipRect, SingleChildScrollView |
| Responsive | @media queries | MediaQuery, LayoutBuilder |
| Unbounded constraints error | ⛔ Không có tương đương | Flutter crash khi widget nhận infinite constraints |
| Mọi thứ là... | Selectors + Properties | Widgets (composition) |

### Điều cần "quên" từ CSS

1. **Không có cascading** — mỗi widget tự quản lý style
2. **Flutter không dùng CSS-like stylesheet toàn cục**, thay vào đó sử dụng `ThemeData` làm hệ thống theming tập trung
3. **Không có margin collapse** — margin luôn cộng dồn
4. **Không có float** — dùng Row/Column/Stack
5. **Không có z-index** — thứ tự trong `Stack.children` quyết định

---

## 10. Tổng kết

### ✅ Checklist kiến thức buổi 4

Sau buổi học, bạn cần hiểu và giải thích được:

- [ ] **Constraints model** — "Constraints go down, Sizes go up, Parent sets position"
- [ ] **BoxConstraints** — tight vs loose constraints
- [ ] **Container** — khi nào nó to, khi nào nó nhỏ
- [ ] **Padding, Center, Align, SizedBox** — khi nào dùng widget nào
- [ ] **Row & Column** — mainAxisAlignment, crossAxisAlignment, mainAxisSize
- [ ] **Stack & Positioned** — xếp chồng widget
- [ ] **Expanded vs Flexible** — tight vs loose flex
- [ ] **Spacer** — tạo khoảng trống linh hoạt
- [ ] **ListView.builder** — lazy rendering cho list dài
- [ ] **GridView** — count vs extent vs builder
- [ ] **CustomScrollView + Slivers** — kết hợp nhiều scrollable
- [ ] **MediaQuery vs LayoutBuilder** — khác nhau thế nào
- [ ] **Unbounded height** — tại sao xảy ra, cách sửa
- [ ] **RenderFlex overflow** — tại sao xảy ra, cách sửa
- [ ] **ParentDataWidget error** — Expanded phải là con trực tiếp của Flex

### 🔑 Quy tắc nhớ

```
1. Widget KHÔNG tự quyết định vị trí → Parent quyết định
2. Widget KHÔNG có kích thước cố định → Phụ thuộc constraints
3. Expanded PHẢI trong Row/Column/Flex
4. ListView trong Column → cần Expanded hoặc shrinkWrap
5. SizedBox cho spacing, Padding cho padding, Container cho decoration
6. ListView.builder cho list > 20 items
7. LayoutBuilder > MediaQuery cho reusable widgets
```

### 🔜 Buổi tiếp theo

**Buổi 05** sẽ học về **Navigation & Routing** — điều hướng giữa các màn hình trong Flutter.
