# Buổi 01: Giới thiệu Dart & Flutter

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

> **Buổi 1/16** · Tiến độ tổng thể: █░░░░░░░░░░░░░░░ **6%**
> **Thời lượng:** ~2.5 giờ · **Cập nhật:** 2026-03-31

---

## 🗺️ Vị trí trong lộ trình

```
Lộ trình Flutter Training (16 buổi)
=====================================================

Tuần 1: Dart & Flutter Foundation
  ┌─────────────────────────────────┐
  │ ★ BẠN ĐANG Ở ĐÂY              │
  │ Buổi 01: Giới thiệu Dart &     │
  │          Flutter                │
  │ ► Dart foundation               │
  └─────────────────────────────────┘
  Buổi 02: Dart nâng cao (OOP, Async, Collections)

Tuần 2: Widget & Layout
  Buổi 03-04: Widget Tree cơ bản, Layout System

Tuần 3: Navigation & State cơ bản
  Buổi 05-06: Navigation & Routing, State Management cơ bản

Tuần 4: State Management nâng cao
  Buổi 07-08: Riverpod, BLoC Pattern

Tuần 5: Architecture & DI
  Buổi 09-10: Clean Architecture, DI & Testing

Tuần 6: Networking & Data
  Buổi 11-12: Networking, Local Storage

Tuần 7: Performance & Animation
  Buổi 13-14: Performance Optimization, Animation

Tuần 8: Platform & Production
  Buổi 15-16: Platform Integration, CI/CD & Production
```

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Dart là ngôn ngữ duy nhất cho Flutter — hiểu Dart = viết Flutter nhanh hơn. Mọi widget, state management, và business logic đều được viết bằng Dart. Nắm vững Dart fundamentals ở buổi này là bước đầu tiên để xây dựng bất kỳ Flutter app nào.

## 💼 Đóng góp vào dự án thực tế

- **Setup môi trường phát triển** — `flutter doctor` clean là bước đầu tiên của mọi Flutter developer
- **Đọc hiểu Dart code** — tất cả source code Flutter đều viết bằng Dart
- **Null safety** — tránh null errors ở production, giảm crash rate
- **Project structure** — hiểu `pubspec.yaml`, `lib/`, `test/` để onboard vào dự án nhanh

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| Flutter là gì? — Kiến trúc | 🔴 | Rendering engine, Skia — debug production cần biết |
| Dart là gì? — AOT vs JIT | 🔴 | Hiểu tại sao Hot Reload hoạt động |
| Cài đặt môi trường | 🟢 | AI/docs hướng dẫn tốt, làm 1 lần là xong |
| Variables — var, final, const | 🔴 | const vs final ảnh hưởng performance rebuild |
| Types — int, double, String | 🔴 | Nền tảng bắt buộc |
| **Null Safety — ?, !, ??, late** | 🔴 | **Quan trọng nhất buổi 01.** Sai = crash runtime |
| Functions — named params | 🟡 | Flutter dùng named params cực nhiều |
| Strings, Control Flow | 🟢 | Đã biết từ JS/TS |
| Flutter app đầu tiên | 🟡 | Cần hiểu MaterialApp, runApp() |
| pubspec.yaml | 🟡 | Giống package.json — AI hỗ trợ tốt |
| Best Practices & Lỗi | 🟡 | Đọc 1 lần, nhớ anti-pattern |
| Góc nhìn Frontend | ⚪ | Reference — đọc khi cần |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: Hello World Dart CLI | 🟢 | Chạy hiểu flow, không cần lặp nhiều |
| VD2: Variables & Null Safety | 🔴 | **Phải tự viết** — đặc biệt null safety |
| VD3: Functions | 🟡 | Hiểu named/optional params |
| VD4: Flutter Hello World | 🟡 | Hiểu widget tree cơ bản |
| VD5: Hot Reload Demo | 🟢 | Demo 1 lần là đủ |
| BT1 ⭐ Setup Environment | 🔴 | Bắt buộc — không setup không học được |
| BT2 ⭐ Dart CLI Calculator | 🟡 | Luyện syntax, nên tự viết |
| BT3 ⭐⭐ Profile Card | 🟡 | Bắt đầu hiểu widget, AI scaffold được |

---

## 📋 Nội dung buổi học

| #  | Phần                          | Thời lượng | Mô tả                                          |
|----|-------------------------------|------------|-------------------------------------------------|
| 1  | Giới thiệu Flutter & Dart    | ~30 phút   | Framework là gì, tại sao chọn Flutter, kiến trúc |
| 2  | Cài đặt môi trường            | ~30 phút   | Flutter SDK, VS Code, Emulator, flutter doctor   |
| 3  | Dart Fundamentals             | ~60 phút   | Variables, types, null safety, functions          |
| 4  | Flutter Hello World           | ~30 phút   | flutter create, project structure, hot reload     |

---

## 🎯 Sau buổi học

### ✅ Hiểu được
- Flutter là gì, Dart là gì, tại sao chúng tồn tại
- Kiến trúc 3 lớp của Flutter (Framework → Engine → Embedder)
- Null safety trong Dart hoạt động thế nào
- pubspec.yaml có vai trò gì (tương tự package.json)
- Hot reload khác gì hot restart

### ✅ Làm được
- Cài đặt Flutter SDK, chạy `flutter doctor` thành công
- Viết chương trình Dart CLI đầu tiên với `dart run`
- Tạo Flutter app với `flutter create` và chạy trên emulator
- Sử dụng variables, functions, null safety cơ bản trong Dart

### 🚫 Chưa cần biết
- OOP trong Dart (Buổi 02)
- Widget tree chi tiết, StatefulWidget (Buổi 05-06)
- State management (Buổi 09-10)
- Async/await, Future, Stream (Buổi 03)

---

## 📅 Tiến độ chương trình

> Buổi 1/16 — Hoàn thành 6% lộ trình
> █░░░░░░░░░░░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | 🔵 Đang học |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ⬜ Chưa bắt đầu |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ⬜ Chưa bắt đầu |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ⬜ Chưa bắt đầu |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ⬜ Chưa bắt đầu |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 📂 Danh sách file

| #  | File                        | Nội dung                              | Thứ tự đọc |
|----|-----------------------------|---------------------------------------|-------------|
| 0  | `00-overview.md`            | Tổng quan buổi học (file này)         | 1           |
| 1  | `01-ly-thuyet.md`           | Lý thuyết đầy đủ                     | 2           |
| 2  | `02-vi-du.md`               | 5 ví dụ hoàn chỉnh, chạy được        | 3           |
| 3  | `03-thuc-hanh.md`           | 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI | 4           |
| 4  | `04-tai-lieu-tham-khao.md`  | Tài liệu & link tham khảo           | 5           |

> **Cách đọc:** Đọc lý thuyết (01) → xem ví dụ (02) → làm bài tập (03) → tra cứu thêm (04)

---

## 📝 Tổng quan bài tập

| Bài | Tên               | Độ khó | Loại       | Chạy             | Mô tả                                     |
|-----|--------------------|--------|------------|------------------|--------------------------------------------| 
| BT1 | Setup Environment  | ⭐     | Setup      | `flutter doctor` | Cài Flutter, flutter doctor, chạy app mẫu |
| BT2 | Dart CLI Calculator| ⭐     | Dart CLI   | `dart run`       | Máy tính đơn giản dùng Dart thuần          |
| BT3 | Flutter Profile Card| ⭐⭐  | Flutter UI | `flutter run`    | Card hiển thị thông tin cá nhân            |

---

> **Tiếp theo:** [01-ly-thuyet.md](./01-ly-thuyet.md) — Lý thuyết chi tiết về Dart & Flutter
