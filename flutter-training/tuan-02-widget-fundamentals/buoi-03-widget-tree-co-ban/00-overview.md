# Buổi 03: Widget Tree — Nền tảng của mọi thứ trong Flutter

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

> **"Everything is a widget"** — Nếu bạn chỉ nhớ một câu về Flutter, hãy nhớ câu này.

## 🗺️ Vị trí trong lộ trình

```
Tuần 1                    Tuần 2                     Tuần 3-4
Dart cơ bản ──▶ [🔵 BẠN ĐANG Ở ĐÂY: Widget] ──▶ State Management ──▶ Architecture
```

**Buổi 3/16** — Tiến độ: ██████░░░░░░░░░░ **19%**

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Mọi UI trong Flutter đều là widget tree — hiểu tree = debug layout nhanh. Widget tree là nền tảng kiến trúc của Flutter: mỗi pixel trên màn hình đều là kết quả của một widget tree được build. Nắm vững widget tree giúp bạn đọc hiểu và debug UI của bất kỳ Flutter app nào.

## 💼 Đóng góp vào dự án thực tế

- **Compose UI từ widgets** — xây dựng UI phức tạp từ các widget nhỏ, tái sử dụng
- **Debug layout issues** — hiểu widget tree để xác định widget nào gây lỗi
- **StatefulWidget lifecycle** — quản lý resource (controllers, listeners) đúng cách
- **Widget keys** — tránh bug khi list thay đổi thứ tự items

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| "Everything is a Widget" | 🔴 | Mental model quan trọng nhất Flutter |
| StatelessWidget vs StatefulWidget | 🔴 | **Bắt buộc hiểu sâu** — sai = bug, leak |
| **3 Trees (Widget/Element/Render)** | 🔴 | **Phân biệt senior vs junior** — debug nhanh 10x |
| **BuildContext** | 🔴 | `.of(context)` dùng mọi nơi, sai = crash |
| Key trong Flutter | 🟡 | Cần hiểu khi nào dùng |
| **Widget Lifecycle** | 🔴 | initState, dispose — phải thuộc thứ tự |
| **setState()** | 🔴 | Trigger rebuild — hiểu phạm vi, anti-patterns |
| Common Widgets | 🟡 | AI nhớ hộ, cần biết tổng quan |
| Best Practices | 🟡 | Đọc 1 lần, nhớ anti-pattern |
| Góc nhìn Frontend | 🟢 | So sánh React/Vue — reference |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: Greeting Card | 🟡 | Hiểu StatelessWidget concept |
| VD2: Counter với setState | 🔴 | **Tự viết 100%** — foundation |
| VD3: Widget Lifecycle | 🔴 | In thứ tự lifecycle → hiểu từng bước |
| VD4: BuildContext | 🔴 | Theme/MediaQuery — dùng hàng ngày |
| VD5: Key Demo | 🟡 | Hiểu vấn đề key giải quyết |
| BT1 ⭐ Counter App | 🔴 | StatefulWidget nhập môn — tự viết |
| BT2 ⭐⭐ Todo List UI | 🔴 | Quản lý danh sách + state |
| BT3 ⭐⭐⭐ Custom Widget | 🟡 | Reusable widget, AI assist được |

---

## 📋 Tổng quan buổi học

| Phần | Nội dung | Thời lượng | Độ khó |
|------|----------|------------|--------|
| 1 | Widget là gì? — Concept cốt lõi | ~30 phút | ⭐ |
| 2 | StatelessWidget vs StatefulWidget | ~30 phút | ⭐⭐ |
| 3 | Widget Tree, Element Tree, RenderObject Tree | ~30 phút | ⭐⭐⭐ |
| 4 | BuildContext & Keys | ~20 phút | ⭐⭐ |
| 5 | Widget Lifecycle & setState() | ~30 phút | ⭐⭐ |
| 6 | Common Widgets — Bộ công cụ hàng ngày | ~20 phút | ⭐ |

**Tổng thời lượng:** ~2 giờ 40 phút (bao gồm thực hành)

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Concept **"Everything is a widget"** và tại sao Widget là immutable
- Phân biệt **StatelessWidget** và **StatefulWidget** — biết khi nào dùng cái nào
- Kiến trúc **3 trees** (Widget → Element → RenderObject) và tại sao Flutter nhanh
- **BuildContext** là gì, dùng thế nào, tránh lỗi phổ biến
- **Key** và khi nào cần dùng

### ✅ Làm được
- Tạo **StatelessWidget** và **StatefulWidget** hoàn chỉnh
- Sử dụng `setState()` đúng cách để cập nhật UI
- Nắm vững **lifecycle** của StatefulWidget (initState, dispose)
- Sử dụng thành thạo các **common widgets**: Container, Text, Image, ElevatedButton...

### 🚫 Chưa cần biết
- State management nâng cao: Provider, Riverpod, BLoC (Buổi 06-08)
- Animation và custom painting (Buổi 14)
- Custom RenderObject (nâng cao, hiếm khi cần)

---

## 📅 Tiến độ chương trình

> Buổi 3/16 — Hoàn thành 19% lộ trình
> ███░░░░░░░░░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | 🔵 Đang học |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ⬜ Chưa bắt đầu |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ⬜ Chưa bắt đầu |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ⬜ Chưa bắt đầu |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 🧠 Kiến thức cần có (Prerequisites)

Từ **Tuần 1**, bạn cần nắm:

- ✅ Dart syntax: variables, functions, classes
- ✅ OOP: class, inheritance, abstract class, mixin
- ✅ Async: Future, async/await
- ✅ Collections: List, Map, Set
- ✅ Đã chạy được Flutter Hello World (`flutter create`, `flutter run`)

> ⚠️ Nếu chưa vững Dart OOP, hãy ôn lại Buổi 02 trước khi tiếp tục.

---

## 📝 Bài tập

| Bài | Tên | Độ khó | Loại | Chạy | Mô tả |
|-----|-----|--------|------|------|--------|
| BT1 | Counter App | ⭐ | Flutter app | `flutter run` | Ứng dụng đếm số với nút tăng/giảm — nhập môn StatefulWidget |
| BT2 | Todo List UI | ⭐⭐ | Flutter app | `flutter run` | Giao diện Todo list — thêm/xoá item, quản lý state với List |
| BT3 | Custom Widget Composition | ⭐⭐⭐ | Flutter app | `flutter run` | Tạo widget UserCard tái sử dụng — hiểu widget composition |

---

## 📂 Cấu trúc thư mục

```
buoi-03-widget-tree-co-ban/
├── 00-overview.md          ← Bạn đang ở đây
├── 01-ly-thuyet.md         ← Lý thuyết chi tiết (8 phần)
├── 02-vi-du.md             ← 5 ví dụ minh hoạ có code
├── 03-thuc-hanh.md         ← 3 bài tập từ dễ đến khó + 🤖 bài tập AI
└── 04-tai-lieu-tham-khao.md ← Tài liệu & link tham khảo
```
