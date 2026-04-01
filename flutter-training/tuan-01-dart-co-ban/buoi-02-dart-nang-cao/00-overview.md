# Buổi 02: Dart nâng cao — OOP, Async, Collections

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

> **Buổi 2/16** · Tiến độ tổng thể: ██░░░░░░░░░░░░░░ **13%**
> **Thời lượng:** ~2.5 giờ · **Cập nhật:** 2026-03-31

---

## 🗺️ Vị trí trong lộ trình

```
Lộ trình Flutter Training (16 buổi)
=====================================================

Tuần 1: Dart & Flutter Foundation
  Buổi 01: Giới thiệu Dart & Flutter ✅
  ┌─────────────────────────────────┐
  │ ★ BẠN ĐANG Ở ĐÂY              │
  │ Buổi 02: Dart nâng cao —       │
  │   OOP, Async, Collections      │
  │ ► Dart Foundation ──▶ Dart     │
  │   nâng cao                     │
  └─────────────────────────────────┘

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

```
Hành trình Dart của bạn:

  Buổi 01                      Buổi 02
  ┌──────────────────┐         ┌──────────────────────────┐
  │ Dart Foundation  │────────▶│ ★ Dart Nâng Cao          │
  │ • Variables       │         │ • OOP (class, mixin,     │
  │ • Types           │         │   sealed class)          │
  │ • Null Safety     │         │ • Collections deep dive  │
  │ • Functions       │         │ • Async (Future, Stream) │
  │ • Control flow    │         │ • Dart 3 features        │
  └──────────────────┘         └──────────────────────────┘
       ✅ Hoàn thành                  ★ BẠN ĐANG Ở ĐÂY
```

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Async/await, collections, và error handling là những pattern được dùng hàng ngày trong mọi Flutter app. Từ gọi API, xử lý user input, đến quản lý state — tất cả đều dựa trên nền tảng Dart nâng cao. Đây là bước chuyển từ "biết Dart" sang "viết Dart hiệu quả".

## 💼 Đóng góp vào dự án thực tế

- **OOP patterns** — class hierarchy, mixin, abstract class dùng trong mọi Flutter project
- **Async/await** — gọi API, đọc file, xử lý database đều cần async
- **Error handling** — try/catch + custom exceptions cho code robust
- **Collections** — map, where, fold xử lý data lists hàng ngày
- **Sealed classes** — Result pattern cho API response handling

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| OOP — Class, Constructors, Inheritance | 🔴 | Mọi Widget/State/Bloc đều là class |
| Abstract Class | 🔴 | Repository, Use Case — dùng khắp nơi |
| Mixins | 🟡 | TickerProviderStateMixin — cần biết concept |
| Sealed Class (Dart 3) | 🟡 | Pattern matching cho State/Error, AI viết được |
| Enum nâng cao | 🟡 | Enhanced enum hữu ích, AI viết tốt |
| Collections deep dive | 🟡 | map/where/fold quan trọng, syntax AI nhớ hộ |
| **Future\<T\> & async/await** | 🔴 | **CỰC KỲ QUAN TRỌNG** — lõi mọi API call |
| **Stream\<T\>** | 🔴 | Real-time, BLoC, Firebase — đều dùng Stream |
| **Event Loop** | 🔴 | Hiểu tại sao UI freeze, microtask vs event |
| Extension Methods | 🟢 | Tiện lợi, AI viết rất tốt |
| Records & Pattern Matching | 🟡 | Dart 3 feature, dùng với sealed class |
| Error Handling | 🔴 | try/catch, Custom Exception — phải hiểu sâu |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: Class Hierarchy | 🔴 | Tự viết để nắm OOP Dart |
| VD2: Sealed Class | 🟡 | Hiểu pattern, AI assist được |
| VD3: Future — Async API Call | 🔴 | **Bắt buộc tự viết** |
| VD4: Stream — Countdown | 🔴 | Phải hiểu Stream behavior |
| VD5: Extension Methods | 🟢 | Đọc hiểu là đủ |
| VD6: Collection Operators | 🟢 | AI viết tốt |
| BT1 ⭐ Class Hierarchy | 🔴 | OOP foundation — tự viết |
| BT2 ⭐⭐ Async Chain | 🔴 | Tự xử lý async flow |
| BT3 ⭐⭐⭐ Result\<T\> | 🟡 | Error handling pattern, AI assist |

---

## 📋 Nội dung buổi học

| #  | Phần                                    | Thời lượng | Mô tả                                                       |
|----|-----------------------------------------|------------|--------------------------------------------------------------|
| 1  | OOP trong Dart                          | ~45 phút   | Class, constructor, inheritance, abstract, mixin, sealed class |
| 2  | Collections & Extension methods         | ~30 phút   | List/Map/Set deep dive, collection operators, extensions      |
| 3  | Async Programming                       | ~45 phút   | Future, Stream, async/await, Event loop, microtask queue      |
| 4  | Dart 3 Features & Error Handling        | ~30 phút   | Records, pattern matching, try/catch, custom exceptions       |

---

## 🎯 Sau buổi học

### ✅ Hiểu được
- Class, constructor (default, named, factory, const) hoạt động thế nào
- Sự khác biệt giữa `extends`, `implements`, `with` (mixin)
- `sealed class` trong Dart 3 giải quyết vấn đề gì
- Future vs Stream — khi nào dùng cái nào
- Event loop của Dart xử lý async thế nào (microtask queue vs event queue)
- Records, pattern matching, destructuring trong Dart 3
- Extension methods mở rộng class có sẵn mà không cần kế thừa

### ✅ Làm được
- Xây dựng class hierarchy hoàn chỉnh (Person → Student, Teacher)
- Viết `sealed class Result<T>` với exhaustive switch
- Gọi API giả lập (simulated) với `Future.delayed` + `async/await`
- Tạo countdown timer bằng `Stream`
- Viết extension methods cho `String`
- Dùng `map`, `where`, `fold`, spread operator, collection if/for

### 🚫 Chưa cần biết
- Widget tree, StatefulWidget (Buổi 05-06)
- State management (Buổi 09-10)
- HTTP requests thật (Buổi 11)
- Isolates / compute (nâng cao hơn)

---

## 📝 Bài tập

| BT  | Tên                              | Độ khó   | Loại     | Chạy       | Thời gian | Mô tả                                      |
|-----|----------------------------------|----------|----------|------------|-----------|---------------------------------------------|
| BT1 | Class Hierarchy                  | ⭐       | Dart CLI | `dart run` | ~30 phút  | Person → Student, Teacher. Dart CLI.        |
| BT2 | Async Chain                      | ⭐⭐     | Dart CLI | `dart run` | ~40 phút  | Fetch user → orders → total. Future chain.  |
| BT3 | Result&lt;T&gt; Pattern          | ⭐⭐⭐   | Dart CLI | `dart run` | ~45 phút  | Sealed class Result + exhaustive switch.     |

---

## 📅 Tiến độ chương trình

> Buổi 2/16 — Hoàn thành 13% lộ trình
> ██░░░░░░░░░░░░░░

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

## 🔗 Liên kết nhanh

| File                                         | Nội dung                        |
|----------------------------------------------|---------------------------------|
| [01-ly-thuyet.md](./01-ly-thuyet.md)         | Lý thuyết chi tiết              |
| [02-vi-du.md](./02-vi-du.md)                 | 6 ví dụ hoàn chỉnh             |
| [03-thuc-hanh.md](./03-thuc-hanh.md)         | 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI |
| [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md) | Tài liệu tham khảo  |
