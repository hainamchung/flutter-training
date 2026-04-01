# Buổi 03: Widget Tree — Tài liệu tham khảo

> Danh sách tài liệu được chọn lọc, sắp theo mức độ ưu tiên.

---

## 📚 Tài liệu chính thức (Flutter.dev)

| # | Tài liệu | Link | Ghi chú |
|---|----------|------|---------|
| 1 | **Widget catalog** | https://docs.flutter.dev/ui/widgets | Danh mục toàn bộ widget, phân loại theo chức năng |
| 2 | **Introduction to widgets** | https://docs.flutter.dev/development/ui/widgets-intro | Hướng dẫn nhập môn widget — đọc đầu tiên |
| 3 | **StatefulWidget class** | https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html | API docs chi tiết + lifecycle diagram |
| 4 | **StatelessWidget class** | https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html | API docs + khi nào dùng |
| 5 | **BuildContext class** | https://api.flutter.dev/flutter/widgets/BuildContext-class.html | Giải thích BuildContext là gì |
| 6 | **Key class** | https://api.flutter.dev/flutter/foundation/Key-class.html | Tài liệu về Key và các subclass |
| 7 | **Flutter architectural overview** | https://docs.flutter.dev/resources/architectural-overview | Kiến trúc tổng quan: Widget → Element → RenderObject |

---

## 📝 Bài viết chuyên sâu

### Widget Tree & Architecture

| # | Bài viết | Tác giả / Nguồn | Nội dung chính |
|---|---------|-----------------|---------------|
| 1 | **Flutter internals: Widget, Element, RenderObject** | flutter.dev (Inside Flutter) | Giải thích 3 trees chi tiết, cách Flutter diff và rebuild |
| 2 | **Understanding Flutter Layout** | flutter.dev | Cách layout hoạt động, constraints propagation |

### BuildContext

| # | Bài viết | Tác giả / Nguồn | Nội dung chính |
|---|---------|-----------------|---------------|
| 1 | **BuildContext class documentation** | api.flutter.dev | Context là handle cho Element, cách dùng .of(context) |

### Keys

| # | Bài viết | Tác giả / Nguồn | Nội dung chính |
|---|---------|-----------------|---------------|
| 1 | **When to Use Keys** | flutter.dev | Hướng dẫn chính thức khi nào cần Key |
| 2 | **Keys! What are they good for?** | Emily Fortuna (Flutter team) | Giải thích trực quan với ví dụ animation |

---

## 🎬 Video

| # | Video | Kênh | Thời lượng | Nội dung |
|---|-------|------|-----------|---------|
| 1 | **Widgets 101** | Flutter (official) | ~8 phút | Concept cơ bản: Widget, build(), composition |
| 2 | **How to Create Stateless Widgets** | Flutter (official) | ~5 phút | StatelessWidget deep dive |
| 3 | **How Stateful Widgets Are Used** | Flutter (official) | ~11 phút | StatefulWidget + lifecycle |
| 4 | **When to Use Keys** | Flutter (official) | ~8 phút | Demo trực quan Key trong list |
| 5 | **How Flutter renders Widgets** | Flutter (official) | ~10 phút | Widget → Element → RenderObject pipeline |

> 💡 **Tip:** Tìm widget tutorial series "Widget of the Week" trên kênh YouTube Flutter chính thức — mỗi tập giới thiệu 1 widget trong ~1 phút.

---

## 📖 Sách

| # | Sách | Tác giả | Chương liên quan |
|---|------|--------|-----------------|
| 1 | **Flutter in Action** | Eric Windmill | Chapter 4: Flutter UI — Build layouts |
| 2 | **Flutter Complete Reference** | Alberto Miola | Part 2: Widgets fundamentals |
| 3 | **Beginning Flutter: A Hands-On Guide** | Marco L. Napoli | Chapter 5, 6: Widgets & State |

---

## 🔧 Công cụ hữu ích

| # | Công cụ | Mô tả |
|---|--------|-------|
| 1 | **Flutter DevTools — Widget Inspector** | Xem Widget Tree thực tế trong app đang chạy. Dùng `flutter run` → nhấn "d" → mở DevTools |
| 2 | **Flutter Outline (VS Code)** | Panel hiển thị widget tree ngay trong editor. View → Flutter Outline |
| 3 | **DartPad** | https://dartpad.dev — Code Flutter online, không cần setup |

---

## 🗂️ Cheat Sheet nhanh

### Widget Lifecycle Order

```
createState → initState → didChangeDependencies → build
                                                    ↕
                                          didUpdateWidget
                                                    ↓
                                    deactivate → dispose
```

### Khi nào dùng Key?

```
List có thể reorder?        → ValueKey
List item có unique ID?      → ValueKey(item.id)
Cần truy cập State từ ngoài? → GlobalKey
Widget cần luôn unique?      → UniqueKey
```

### Common Widget chọn nhanh

```
Muốn box + padding + decoration? → Container
Muốn khoảng trống?               → SizedBox
Muốn text?                       → Text
Muốn hình?                       → Image.asset / Image.network
Muốn icon?                       → Icon
Muốn nút bấm?                    → ElevatedButton / TextButton
Muốn ô nhập?                     → TextField
Muốn card?                       → Card + ListTile
```

---

## 🤖 AI Prompt Library — Buổi 03: Widget Tree cơ bản

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Widget Lifecycle trong Flutter. Background: 3+ năm React (useEffect, cleanup function, unmount).
Câu hỏi: initState/dispose trong Flutter tương đương gì với useEffect trong React? Khi nào dùng didChangeDependencies thay vì initState? Tại sao initState không được async?
Yêu cầu: giải thích bằng tiếng Việt, mapping 1-1 với React hooks lifecycle, kèm code Flutter 3.x minh họa.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần tạo Flutter StatefulWidget cho màn hình Profile với editable fields.
Tech stack: Flutter 3.x, Dart 3.x.
Constraints:
- 3 TextEditingController (name, email, bio) — tạo trong initState hoặc khai báo trực tiếp.
- 2 FocusNode (email, bio) — để auto-focus field tiếp theo khi nhấn Done.
- TẤT CẢ controllers và FocusNode PHẢI dispose trong dispose().
- Dùng Form widget + GlobalKey<FormState> cho validation.
- const constructor cho label Text, Icon cố định.
Output: 1 file profile_form.dart hoàn chỉnh.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Flutter widget code sau:
[paste code]

Kiểm tra theo thứ tự:
1. Lifecycle: mọi controller/subscription tạo ra đều có dispose tương ứng?
2. setState: có check mounted trước khi gọi? Có gọi sau dispose?
3. const: widget static có dùng const constructor? const prefix cho constructor call?
4. Key: List item có UniqueKey/ValueKey? Không dùng index làm key?
5. Performance: build method có logic nặng? Nên tách ra didChangeDependencies?
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter:
[paste error message đầy đủ]

Code liên quan:
[paste đoạn code gây lỗi]

Context: đang implement StatefulWidget với Timer/Controller, Flutter 3.x.
Cần: (1) Giải thích nguyên nhân bằng tiếng Việt, (2) Cách fix cụ thể, (3) Nếu liên quan đến lifecycle (setState after dispose, use after dispose), giải thích flow lifecycle dẫn đến lỗi.
```

---

## 🔗 Liên kết nội bộ

| Tài liệu | Đường dẫn |
|----------|-----------|
| Buổi trước: Dart nâng cao | `tuan-01-dart-co-ban/buoi-02-dart-nang-cao/` |
| Buổi sau: Layout & Responsive | `tuan-02-widget-fundamentals/buoi-04-layout-responsive/` |
| Reference Architecture | `project-mau/reference-architecture.md` |
| Middle Level Rubric | `tieu-chuan/middle-level-rubric.md` |
