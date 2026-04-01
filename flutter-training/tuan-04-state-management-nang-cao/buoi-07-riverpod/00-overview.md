# Buổi 07: Riverpod Deep Dive

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 📍 Vị trí trong lộ trình

```
Buổi 7/16 — Tiến độ: ████████████░░░░░░░░ 44%
```

```
State cơ bản (Provider) ──▶ [🔵 BẠN ĐANG Ở ĐÂY: Riverpod] ──▶ BLoC ──▶ Architecture
```

## 📅 Tiến độ chương trình

> Buổi 7/16 — Hoàn thành 44% lộ trình
> ███████░░░░░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | 🔵 Đang học |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ⬜ Chưa bắt đầu |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Riverpod là production state management — quản lý auth, cache, API state. Trong dự án Flutter thực tế, Riverpod là giải pháp được recommend bởi cộng đồng vì compile-safe, testable, và không phụ thuộc vào BuildContext.

## 💼 Đóng góp vào dự án thực tế

- **API state management** — loading, error, data states cho mọi API call
- **Authentication flow** — login, logout, token refresh với Riverpod
- **Caching** — cache API responses, user preferences
- **Dependency injection** — providers thay thế service locator
- **Testable code** — override providers trong unit test dễ dàng

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Tại sao Riverpod ra đời và giải quyết vấn đề gì của Provider
- Sự khác biệt giữa các loại Provider (Provider, NotifierProvider, FutureProvider, StreamProvider)
- Khi nào dùng ref.watch, ref.read, ref.listen

### ✅ Làm được
- Sử dụng thành thạo các loại Provider trong Riverpod
- Phân biệt và áp dụng đúng ref.watch, ref.read, ref.listen
- Sử dụng Modifiers: autoDispose, family
- Áp dụng code generation với `@riverpod` annotation
- Viết unit test cho Riverpod providers

### 🚫 Chưa cần biết
- Riverpod + Clean Architecture kết hợp (Buổi 09)
- Riverpod + Networking thực tế với Dio (Buổi 11)
- Custom ProviderObserver cho logging/analytics (nâng cao)

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| Tại sao Riverpod — vấn đề Provider | 🔴 | Phải hiểu WHY, không chỉ HOW |
| **Các loại Provider** | 🔴 | Provider, Notifier, Future, Stream — phải phân biệt |
| ConsumerWidget vs Consumer | 🟡 | Syntax AI nhớ hộ, cần biết khi nào dùng gì |
| Code Gen @riverpod | 🟢 | AI generate rất tốt, hiểu output là đủ |
| **ref.watch / ref.read / ref.listen** | 🔴 | **QUAN TRỌNG NHẤT.** Dùng sai = bug + performance |
| Anti-patterns | 🔴 | ref.watch trong callback — phải nhận ra |
| autoDispose, family | 🟡 | Concept quan trọng, cú pháp AI nhớ hộ |
| Testing với Riverpod | 🟡 | ProviderContainer, override — AI viết test tốt |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD1-3: Basic providers | 🔴 | Tự viết ít nhất 1 cái |
| VD4: family + autoDispose | 🟡 | Hiểu concept |
| VD5: @riverpod code gen | 🟢 | AI generate |
| BT1 ⭐ Todo App Riverpod | 🔴 | Nền tảng — tự viết |
| BT2 ⭐⭐ Weather App | 🟡 | FutureProvider practice |
| BT3 ⭐⭐⭐ Full App + Tests | 🟢 | AI assist test code |

---

## 📋 Nội dung chi tiết

| Phần | Nội dung | Thời lượng |
|------|----------|------------|
| 1 | Tại sao Riverpod? — Vấn đề của Provider & cách Riverpod giải quyết | ~20 phút |
| 2 | Core Providers — Provider, NotifierProvider, FutureProvider, StreamProvider | ~40 phút |
| 3 | ref methods — ref.watch, ref.read, ref.listen | ~20 phút |
| 4 | Modifiers — autoDispose, family | ~20 phút |
| 5 | Code generation — @riverpod annotation | ~20 phút |
| 6 | Testing với Riverpod | ~20 phút |

**Tổng thời lượng: ~140 phút** (2 tiếng 20 phút, chia thành 2 buổi nếu cần)

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter app | `flutter run` | Todo App với Riverpod — NotifierProvider, add/toggle/delete |
| BT2 | ⭐⭐ | Flutter app | `flutter run` | Weather App với FutureProvider — fetch mock weather data theo city |
| BT3 | ⭐⭐⭐ | Flutter app | `flutter run` | Full Riverpod App — autoDispose + family + unit tests |

## 📚 Kiến thức tiên quyết

- [x] Buổi 06: State Management cơ bản (Provider, ChangeNotifier)
- [x] Hiểu `context.read()` / `context.watch()` trong Provider
- [x] Dart async/await, Stream cơ bản
- [x] Widget lifecycle (StatefulWidget, initState, dispose)

## 🗂️ Cấu trúc files

```
buoi-07-riverpod/
├── 00-overview.md          ← Bạn đang ở đây
├── 01-ly-thuyet.md         ← Lý thuyết chi tiết
├── 02-vi-du.md             ← 5 ví dụ minh họa
├── 03-thuc-hanh.md         ← 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI
└── 04-tai-lieu-tham-khao.md ← Tài liệu bổ sung
```
