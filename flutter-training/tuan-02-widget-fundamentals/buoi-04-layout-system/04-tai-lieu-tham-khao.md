# Buổi 04: Layout System — Tài liệu tham khảo

---

## 📚 Tài liệu chính thức (Flutter.dev)

### Bắt buộc đọc

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| **Understanding constraints** | https://docs.flutter.dev/ui/layout/constraints | 🔴 **ĐỌC ĐẦU TIÊN** — giải thích Constraints model cực kỳ rõ ràng với ví dụ interactive |
| **Layouts in Flutter** | https://docs.flutter.dev/ui/layout | Tổng quan layout system |
| **Layout tutorial** | https://docs.flutter.dev/ui/layout/tutorial | Step-by-step build layout |
| **Box constraints** | https://api.flutter.dev/flutter/rendering/BoxConstraints-class.html | API reference cho BoxConstraints |

### Nên đọc

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| **Building layouts** | https://docs.flutter.dev/ui/layout/building-adaptive-apps | Responsive & adaptive apps |
| **Slivers** | https://docs.flutter.dev/ui/layout/scrolling/slivers | CustomScrollView + Slivers |
| **Widget catalog — Layout** | https://docs.flutter.dev/ui/widgets/layout | Catalog tất cả layout widgets |
| **Dealing with overflow** | https://docs.flutter.dev/testing/common-errors | Common Flutter errors |

---

## 📖 Bài viết & Blog

### Constraints & Layout cơ bản

| Bài viết | Tác giả | Ghi chú |
|----------|---------|---------|
| **Flutter Layout Cheat Sheet** | Tomek Polański | Cheat sheet trực quan cho layout widgets — rất nhiều ảnh minh họa |
| **Understanding Flutter Layout (Box Constraints)** | Marcelo Glasberg | Giải thích constraints chi tiết với ví dụ |
| **A Deep Dive Into Flutter Layout** | Andrea Bizzotto | Series bài viết sâu về layout |

### Responsive Design

| Bài viết | Tác giả | Ghi chú |
|----------|---------|---------|
| **Creating Responsive Apps in Flutter** | Flutter team | Hướng dẫn responsive UI |
| **MediaQuery vs LayoutBuilder** | Various | So sánh 2 approach responsive |

### Advanced Layout

| Bài viết | Tác giả | Ghi chú |
|----------|---------|---------|
| **Slivers Explained** | Emily Fortuna | Giải thích Slivers dễ hiểu |
| **Flutter Slivers Overview** | Flutter team | Tổng quan Slivers (medium.com/flutter) |

---

## 🎥 Video

### Chính thức (Flutter YouTube)

| Video | Nội dung | Thời lượng |
|-------|----------|------------|
| **Constraints** (Decoding Flutter) | Giải thích Constraints model bằng animation | ~10 phút |
| **Row & Column** (Flutter Widget of the Week) | Tổng quan Row/Column | ~2 phút |
| **Expanded** (Flutter Widget of the Week) | Cách Expanded hoạt động | ~1 phút |
| **Flexible** (Flutter Widget of the Week) | Flexible vs Expanded | ~1 phút |
| **SliverAppBar** (Flutter Widget of the Week) | SliverAppBar demo | ~1 phút |
| **ListView & GridView** (Flutter in Focus) | So sánh list widgets | ~8 phút |
| **Slivers** (The Boring Flutter Show) | Deep dive Slivers | ~45 phút |

### Kênh Flutter YouTube chính thức
- https://www.youtube.com/@flutterdev

### Community Videos

| Video | Kênh | Ghi chú |
|-------|------|---------|
| **Flutter Layout Deep Dive** | Robert Brunhage | Layout từ cơ bản đến nâng cao |
| **Flutter Responsive Design Guide** | The Flutter Way | Responsive layout tutorial |
| **Understanding Flutter Constraints** | Fun with Flutter | Constraints giải thích trực quan |

---

## 🛠️ Công cụ hỗ trợ

| Công cụ | Mục đích |
|---------|----------|
| **Flutter Inspector** (DevTools) | Xem widget tree, constraints, size trong thời gian thực |
| **Layout Explorer** (DevTools) | Visualize Flex layout (Row/Column/Flex) |
| **Widget Inspector** overlay | `debugPaintSizeEnabled = true` — hiển thị boundaries trên app |

### Cách sử dụng DevTools Layout Explorer

1. Chạy app ở debug mode
2. Mở Flutter DevTools (`flutter pub global activate devtools`)
3. Tab **Inspector** → chọn widget → xem Constraints, Size
4. Tab **Layout Explorer** → trực quan Flex distribution

---

## 📋 Widget Reference nhanh

### Single-child Layout Widgets

| Widget | Chức năng | API Reference |
|--------|-----------|---------------|
| `Container` | All-in-one: decoration, padding, margin, size | https://api.flutter.dev/flutter/widgets/Container-class.html |
| `Padding` | Chỉ thêm padding | https://api.flutter.dev/flutter/widgets/Padding-class.html |
| `Center` | Đặt child ở giữa | https://api.flutter.dev/flutter/widgets/Center-class.html |
| `Align` | Đặt child ở vị trí bất kỳ | https://api.flutter.dev/flutter/widgets/Align-class.html |
| `SizedBox` | Kích thước cố định / spacing | https://api.flutter.dev/flutter/widgets/SizedBox-class.html |
| `ConstrainedBox` | Thêm constraints | https://api.flutter.dev/flutter/widgets/ConstrainedBox-class.html |
| `FractionallySizedBox` | Size theo % parent | https://api.flutter.dev/flutter/widgets/FractionallySizedBox-class.html |
| `AspectRatio` | Giữ tỷ lệ w/h | https://api.flutter.dev/flutter/widgets/AspectRatio-class.html |

### Multi-child Layout Widgets

| Widget | Chức năng | API Reference |
|--------|-----------|---------------|
| `Row` | Sắp xếp ngang | https://api.flutter.dev/flutter/widgets/Row-class.html |
| `Column` | Sắp xếp dọc | https://api.flutter.dev/flutter/widgets/Column-class.html |
| `Stack` | Xếp chồng | https://api.flutter.dev/flutter/widgets/Stack-class.html |
| `Wrap` | Tự xuống dòng | https://api.flutter.dev/flutter/widgets/Wrap-class.html |
| `ListView` | Danh sách scroll | https://api.flutter.dev/flutter/widgets/ListView-class.html |
| `GridView` | Lưới | https://api.flutter.dev/flutter/widgets/GridView-class.html |

### Flex Widgets

| Widget | Chức năng | API Reference |
|--------|-----------|---------------|
| `Expanded` | Fill hết flex space | https://api.flutter.dev/flutter/widgets/Expanded-class.html |
| `Flexible` | Linh hoạt (có thể nhỏ hơn) | https://api.flutter.dev/flutter/widgets/Flexible-class.html |
| `Spacer` | Khoảng trống linh hoạt | https://api.flutter.dev/flutter/widgets/Spacer-class.html |

### Scrollable Widgets

| Widget | Chức năng | API Reference |
|--------|-----------|---------------|
| `SingleChildScrollView` | Scroll một child | https://api.flutter.dev/flutter/widgets/SingleChildScrollView-class.html |
| `CustomScrollView` | Kết hợp nhiều slivers | https://api.flutter.dev/flutter/widgets/CustomScrollView-class.html |
| `SliverList` | List trong CustomScrollView | https://api.flutter.dev/flutter/widgets/SliverList-class.html |
| `SliverGrid` | Grid trong CustomScrollView | https://api.flutter.dev/flutter/widgets/SliverGrid-class.html |
| `SliverAppBar` | Collapsing app bar | https://api.flutter.dev/flutter/material/SliverAppBar-class.html |

### Responsive Widgets

| Widget | Chức năng | API Reference |
|--------|-----------|---------------|
| `MediaQuery` | Thông tin màn hình | https://api.flutter.dev/flutter/widgets/MediaQuery-class.html |
| `LayoutBuilder` | Biết constraints parent | https://api.flutter.dev/flutter/widgets/LayoutBuilder-class.html |
| `OrientationBuilder` | Detect xoay màn hình | https://api.flutter.dev/flutter/widgets/OrientationBuilder-class.html |

---

## 🤖 AI Prompt Library — Buổi 04: Layout System

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Flutter Layout Constraints model. Background: 4+ năm CSS Flexbox/Grid.
Câu hỏi: "Constraints go down. Sizes go up. Parent sets position." — giải thích flow này cụ thể. So sánh với CSS box model, tại sao Flutter KHÔNG có concept "margin collapse" hay "block formatting context"?
Yêu cầu: giải thích bằng tiếng Việt, vẽ sơ đồ text constraints flow, kèm code minh họa tight vs loose constraints.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần tạo Flutter layout cho Settings screen (dạng grouped list).
Tech stack: Flutter 3.x, Dart 3.x.
Constraints:
- ListView chứa các group: "Account", "Notifications", "Privacy".
- Mỗi group: header text bold + list ListTile (icon + title + trailing widget).
- Trailing: Switch, Icon(chevron_right), hoặc Text value.
- Dùng Column cho mỗi group, ListView cho scroll tổng thể.
- Padding 16 horizontal, divider giữa các items trong group.
- KHÔNG dùng shrinkWrap cho ListView chính.
Output: 1 file settings_screen.dart, có dummy data built-in.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Flutter layout code sau:
[paste code]

Kiểm tra theo thứ tự:
1. Constraints: có Expanded/Flexible quá sâu (không phải child trực tiếp của Row/Column)?
2. Overflow: Text dài có maxLines + overflow? Image có BoxFit?
3. Scroll: ListView có shrinkWrap: true? Nếu có, có cần thiết không?
4. Responsive: có hardcode width/height? Nên dùng MediaQuery/LayoutBuilder?
5. Performance: const cho static widget? ListView.builder thay ListView(children)?
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter layout:
[paste error message đầy đủ, bao gồm "... was given unbounded height" hoặc "RenderFlex overflowed"]

Code liên quan:
[paste widget tree gây lỗi]

Context: đang build layout dùng Row/Column/ListView, Flutter 3.x.
Cần: (1) Giải thích constraints flow dẫn đến lỗi (parent cho gì → child muốn gì → conflict ở đâu), (2) Vẽ sơ đồ text constraints flow, (3) Fix cụ thể, (4) Pattern chuẩn để tránh lỗi này.
```

---

## 🔗 Series đọc tiếp

| Buổi | Chủ đề | Liên quan |
|------|--------|-----------|
| Buổi 03 | Widget Tree cơ bản | Nền tảng — hiểu build(), key |
| **Buổi 04** | **Layout System** | **← Bạn đang ở đây** |
| Buổi 05 | Input & Forms | TextField, Form validation |
| Buổi 06 | Navigation | Navigator, routes |
