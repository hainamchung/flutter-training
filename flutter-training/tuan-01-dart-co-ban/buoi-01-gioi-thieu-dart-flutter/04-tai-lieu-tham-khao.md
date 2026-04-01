# Buổi 01 — Tài liệu tham khảo: Giới thiệu Dart & Flutter

> **Buổi 1/16** · **Cập nhật:** 2026-03-31

---

## 📚 Tài liệu chính thức (Official)

### Dart

| Tài liệu                  | Link                                                   | Ghi chú                            |
|----------------------------|---------------------------------------------------------|--------------------------------------|
| Dart Language Tour          | https://dart.dev/language                               | ⭐ Bắt buộc đọc — tổng quan ngôn ngữ Dart |
| Dart Language Basics        | https://dart.dev/language/basics                         | Biến, types, functions cơ bản        |
| Dart Null Safety            | https://dart.dev/null-safety                             | Giải thích chi tiết null safety      |
| Dart Type System            | https://dart.dev/language/type-system                    | Hệ thống kiểu dữ liệu của Dart      |

### Flutter

| Tài liệu                   | Link                                                   | Ghi chú                            |
|-----------------------------|---------------------------------------------------------|--------------------------------------|
| Flutter Get Started         | https://docs.flutter.dev/get-started/install             | ⭐ Hướng dẫn cài đặt chi tiết       |
| Flutter Architecture        | https://docs.flutter.dev/resources/architectural-overview | Kiến trúc 3 lớp của Flutter          |
| Flutter for Web Developers  | https://docs.flutter.dev/get-started/flutter-for/web-devs | Dành cho người từ Web chuyển sang    |

---

## 📝 Tutorials & Codelabs

| Tài liệu                      | Link                                                  | Ghi chú                           |
|--------------------------------|-------------------------------------------------------|------------------------------------|
| Dart Tutorials                 | https://dart.dev/tutorials                             | Tutorial chính thức từ Dart team   |
| Your First Flutter App         | https://codelabs.developers.google.com/codelabs/flutter-codelab-first | Codelab xây dựng app Flutter đầu tiên |
| Flutter Codelabs               | https://docs.flutter.dev/codelabs                      | Danh sách tất cả Flutter codelabs  |
| Dart Cheatsheet                | https://dart.dev/resources/dart-cheatsheet              | Cheatsheet cú pháp Dart nhanh     |

---

## 🎥 Video

| Video                                  | Kênh      | Link                                                       | Ghi chú                      |
|----------------------------------------|-----------|-------------------------------------------------------------|-------------------------------|
| Flutter in 100 Seconds                 | Fireship  | https://www.youtube.com/watch?v=lHhRhPV--G0                 | ⭐ Tổng quan nhanh trong 100s |
| Dart in 100 Seconds                    | Fireship  | https://www.youtube.com/watch?v=NrO0CJCbYLA                 | Tổng quan Dart trong 100s     |
| Flutter Crash Course for Beginners     | Academind | https://www.youtube.com/watch?v=x0uinJvhNxI                 | Khóa Flutter cơ bản ~5 giờ   |
| Flutter & Dart Complete Guide          | Academind | Tìm trên Udemy (Max Schwarzmüller)                          | Khóa đầy đủ nhất (trả phí)   |
| The Net Ninja — Flutter Tutorial       | Net Ninja | https://www.youtube.com/playlist?list=PL4cUxeGkcC9jLYyp2Aoh6hcWuxFDX6PBJ | Playlist Flutter cơ bản      |

---

## 📦 Packages

> **Chưa cần package bên ngoài ở buổi này.**
>
> Buổi 1 tập trung vào Dart cơ bản và Flutter core — tất cả đều có sẵn trong Flutter SDK.
> Các package bên ngoài (http, provider, go_router, ...) sẽ được giới thiệu ở các buổi sau.
>
> Khi cần tìm package: https://pub.dev

---

## 🔧 Công cụ hữu ích

| Công cụ          | Link                              | Mô tả                                      |
|-------------------|-----------------------------------|----------------------------------------------|
| DartPad           | https://dartpad.dev               | ⭐ Chạy Dart/Flutter trực tiếp trên trình duyệt — không cần cài gì |
| Flutter DevTools  | https://docs.flutter.dev/tools/devtools | Debug, inspect widget tree, performance      |
| pub.dev           | https://pub.dev                   | Registry chính thức cho Dart/Flutter packages |
| Flutter Gallery   | https://gallery.flutter.dev       | Showcase tất cả Material & Cupertino widgets |

---

## 📖 Đọc thêm (không bắt buộc)

| Tài liệu                                | Link                                                    | Ghi chú                          |
|------------------------------------------|---------------------------------------------------------|----------------------------------|
| Why Flutter Uses Dart                    | https://hackernoon.com/why-flutter-uses-dart-dd635a054ebf | Giải thích tại sao Dart, không phải JS |
| Flutter's Rendering Pipeline             | https://docs.flutter.dev/resources/rendering              | Deep dive vào cách Flutter render |
| Effective Dart                           | https://dart.dev/effective-dart                           | Style guide chính thức của Dart   |

---

## 🤖 AI Prompt Library — Buổi 01: Giới thiệu Dart & Flutter

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Null Safety trong Dart. Background: 3+ năm React/Vue với TypeScript strict mode.
Câu hỏi: Sound null safety trong Dart khác gì TypeScript strict mode? Tại sao Dart cấm dùng ! operator là best practice trong khi TypeScript dùng ! (non-null assertion) khá phổ biến?
Yêu cầu: giải thích bằng tiếng Việt, so sánh cụ thể với TypeScript, kèm code Dart 3.x minh họa sự khác biệt.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần tạo Dart CLI app — chương trình quản lý danh sách công việc (todo list) đơn giản.
Tech stack: Dart 3.x, sound null safety, chạy bằng dart run.
Constraints:
- Dùng final/const đúng chỗ, không dùng var khi có thể dùng final.
- Null safety: dùng String? cho trường description (có thể không có mô tả).
- Named parameters cho hàm addTodo({required String title, String? description, bool isDone = false}).
- Arrow syntax cho hàm 1 expression.
- Switch expression (Dart 3) cho việc filter todo theo status.
Output: 1 file todo_app.dart hoàn chỉnh, có main() với demo data.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Dart code sau:
[paste code]

Kiểm tra theo thứ tự:
1. Null safety: có dùng ! operator không? Có dynamic nào không cần thiết?
2. Variables: final/const dùng đúng chỗ chưa? Có var nào nên là final?
3. Functions: named parameters cho params > 2? Arrow syntax cho hàm 1 expression?
4. Type safety: mọi biến có explicit type hoặc type inference rõ ràng?
5. Dart conventions: naming (camelCase cho biến, PascalCase cho class), formatting.
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Dart:
[paste error message đầy đủ]

Code liên quan:
[paste đoạn code gây lỗi]

Context: đang học Dart cơ bản (biến, types, null safety, functions), Flutter 3.x, Dart 3.x.
Cần: (1) Giải thích nguyên nhân bằng tiếng Việt, (2) Cách fix cụ thể, (3) Cách phòng tránh lỗi này trong tương lai.
Nếu lỗi liên quan đến null safety, giải thích tại sao Dart bắt lỗi này và đây là điều tốt.
```

---

## 🗓️ Liên kết nội bộ

| File                      | Nội dung                    |
|---------------------------|-----------------------------|
| [00-overview.md](./00-overview.md)       | Tổng quan buổi học          |
| [01-ly-thuyet.md](./01-ly-thuyet.md)     | Lý thuyết chi tiết          |
| [02-vi-du.md](./02-vi-du.md)             | 5 ví dụ thực hành           |
| [03-thuc-hanh.md](./03-thuc-hanh.md)     | 3 bài tập                   |

---

> **Buổi tiếp theo:** Buổi 02 — OOP trong Dart (class, constructor, inheritance, mixin)
