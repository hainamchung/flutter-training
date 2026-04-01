# Buổi 06: State Management Cơ Bản

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

> **Buổi 6/16** · Tuần 3 · Ngày 2 · ⏱ ~150 phút · 📊 Tiến độ: 38%

## 🗺 Lộ trình học

```
Widget ──▶ Layout ──▶ Navigation ──▶ [🔵 BẠN ĐANG Ở ĐÂY: State] ──▶ Architecture
  ✅          ✅          ✅                   🔵                        ⬜
```

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | 🔵 Đang học |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ⬜ Chưa bắt đầu |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ⬜ Chưa bắt đầu |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 🌍 Vai trò trong hệ sinh thái Flutter

setState và InheritedWidget là foundation cho Riverpod/BLoC. Mọi app Flutter đều cần quản lý state — từ form input đến API data. Hiểu đúng state cơ bản giúp bạn chuyển sang bất kỳ state management solution nào một cách vững vàng.

## 💼 Đóng góp vào dự án thực tế

- **Form validation** — registration, login, search — mọi app đều cần form
- **Shared state** — theme, user info, cart dùng chung giữa nhiều screens
- **Provider pattern** — codebase legacy dùng Provider vẫn rất phổ biến
- **State debugging** — hiểu data flow giúp tìm bug nhanh hơn

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Phân biệt **ephemeral state** vs **app state** và biết khi nào dùng cái nào
- Bản chất `setState()` — cách nó hoạt động bên trong và giới hạn của nó
- `InheritedWidget` — cơ chế gốc của Flutter để truyền data xuống widget tree
- Tổng quan về các giải pháp state management (Riverpod, BLoC, GetX)

### ✅ Làm được
- Sử dụng thành thạo **Provider** (`ChangeNotifier`, `ChangeNotifierProvider`, `Consumer`, `context.watch/read`)
- Xây dựng **Form** với validation hoàn chỉnh
- Quản lý state đơn giản trong app multi-screen

### 🚫 Chưa cần biết
- Riverpod chi tiết (Buổi 07)
- BLoC pattern (Buổi 08)
- State management cho complex async flows (Buổi 07-08)

## 🎯 Hôm nay học gì?

| # | Chủ đề | Mô tả | Thời lượng |
|---|--------|-------|------------|
| 1 | State trong Flutter | Ephemeral vs App state, khi nào dùng gì | ~20 phút |
| 2 | InheritedWidget | Cơ chế gốc truyền data trong Widget tree | ~25 phút |
| 3 | Provider | ChangeNotifier + Provider pattern | ~40 phút |
| 4 | Form & Validation | TextFormField, validators, form state | ~25 phút |
| 5 | State Management Patterns | Lifting state, callback vs stream | ~10 phút |

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| **State là gì? Ephemeral vs App** | 🔴 | **FOUNDATION.** Sai phân loại = thiết kế sai |
| Decision Tree chọn state | 🔴 | Phải tự quyết định — AI không biết business context |
| **setState() Deep Dive** | 🔴 | markNeedsBuild, scheduleBuild — hiểu bên trong |
| Giới hạn setState | 🔴 | Biết khi nào KHÔNG đủ → chuyển Provider/Riverpod |
| **Lifting State Up** | 🔴 | Pattern cốt lõi — lifting state lên ancestor chung |
| InheritedWidget | 🟡 | Mechanism gốc, không cần tự viết (dùng Provider) |
| **ChangeNotifier + Provider** | 🔴 | Vẫn gặp codebase cũ, phải hiểu watch/read/select |
| **watch vs read vs select** | 🔴 | **Sai = rebuild toàn bộ UI**, performance chết |
| Forms & Validation | 🟡 | AI viết form tốt nhưng cần hiểu flow |
| ⚠️ GetX warning | 🔴 | Biết tại sao KHÔNG dùng GetX trong production |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD1: Ephemeral vs App State | 🔴 | Phải phân biệt được |
| VD2: InheritedWidget | 🟡 | Hiểu gốc, không cần thuộc |
| VD3: Provider Counter | 🔴 | Starter pattern — tự viết |
| VD4: Provider Shopping Cart | 🔴 | Multi-widget share state — core skill |
| VD5: Form Validation | 🟡 | AI viết form tốt, cần hiểu flow |
| BT1 ⭐ Theme Switcher | 🔴 | Provider cơ bản — tự viết |
| BT2 ⭐⭐ Shopping Cart | 🔴 | State phức tạp — tự thiết kế |
| BT3 ⭐⭐⭐ Registration Form | 🟡 | Form validation — AI assist tốt |

---

## 📋 Cấu trúc buổi học

| Phần | Nội dung | Thời gian | Ghi chú |
|------|----------|-----------|---------|
| 1 | State concepts — Ephemeral vs App state | ~20 phút | Lý thuyết + decision tree |
| 2 | `setState()` review — Cách hoạt động & giới hạn | ~15 phút | Deep dive internal |
| 3 | `InheritedWidget` — Cơ chế gốc | ~30 phút | Hiểu nền tảng trước khi dùng package |
| 4 | **Provider** — ChangeNotifier + Provider | ~40 phút | ⚡ Phần trọng tâm |
| 5 | Forms & Validation | ~25 phút | Gap bổ sung — rất quan trọng cho production |
| 6 | Tổng quan state management landscape | ~20 phút | So sánh các giải pháp |

## 💡 Tại sao buổi này quan trọng?

```
┌─────────────────────────────────────────────────┐
│  Mọi app Flutter đều cần quản lý state.         │
│  Hiểu đúng state management = code sạch,        │
│  dễ test, dễ maintain, dễ scale.                 │
│                                                   │
│  Đây là CẦU NỐI từ "biết Flutter"               │
│  sang "viết Flutter production-quality".          │
└─────────────────────────────────────────────────┘
```

State management là **kỹ năng nền tảng** — dù bạn chọn Provider, Riverpod, hay BLoC sau này, các concept học hôm nay đều áp dụng được.

## 🧰 Chuẩn bị

- **IDE**: VS Code hoặc Android Studio với Flutter plugin
- **Flutter SDK**: >= 3.x
- **Package**: `provider: ^6.0.0` (sẽ thêm vào `pubspec.yaml`)
- **Kiến thức cần có**: Dart, Widget tree, Layout, Navigation (Buổi 1–5)

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả | Kỹ năng |
|-----|--------|------|------|-------|---------|
| BT1 | ⭐ | Flutter app | `flutter run` | Theme Switcher với Provider | Provider cơ bản, `ChangeNotifier` |
| BT2 | ⭐⭐ | Flutter app | `flutter run` | Shopping Cart với Provider | State phức tạp, nhiều widget chia sẻ state |
| BT3 | ⭐⭐⭐ | Flutter app | `flutter run` | Registration Form với multi-field validation | Form, `GlobalKey<FormState>`, custom validators |

## 🔗 Liên kết

- Lý thuyết: [01-ly-thuyet.md](./01-ly-thuyet.md)
- Ví dụ: [02-vi-du.md](./02-vi-du.md)
- Thực hành: [03-thuc-hanh.md](./03-thuc-hanh.md) + 🤖 bài tập AI
- Tài liệu tham khảo: [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md)
