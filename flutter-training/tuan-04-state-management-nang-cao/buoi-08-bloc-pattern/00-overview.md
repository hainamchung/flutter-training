# Buổi 08: BLoC Pattern

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 📍 Vị trí trong lộ trình

> **Buổi 8/16** · Tuần 4: State Management Nâng Cao · **50% hoàn thành — HALFWAY! 🎉**

```
Tuần 1        Tuần 2           Tuần 3              Tuần 4
Dart cơ bản → Widget cơ bản → Navigation/State → State Nâng Cao
  ✅              ✅               ✅                 🔵
```

```
Riverpod ──▶ [🔵 BẠN ĐANG Ở ĐÂY: BLoC] ──▶ Architecture ──▶ Production
   ✅                  📍                        ⬜               ⬜
```

---

## 🌍 Vai trò trong hệ sinh thái Flutter

BLoC là enterprise-grade state management — event-driven, highly testable. Được dùng rộng rãi trong các dự án Flutter lớn nhờ separation of concerns rõ ràng và khả năng test cao. Nhiều công ty chọn BLoC làm standard cho Flutter team của họ.

## 💼 Đóng góp vào dự án thực tế

- **Event-driven architecture** — tách biệt UI events và business logic
- **Complex state flows** — auth flow, payment flow với multiple states
- **Team scalability** — BLoC pattern giúp nhiều developer làm việc song song
- **Testing** — bloc_test cho unit test nhanh và đáng tin cậy

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- BLoC pattern — Business Logic Component, luồng dữ liệu một chiều
- Phân biệt **Cubit** (đơn giản) vs **Bloc** (event-driven) và khi nào dùng cái nào
- So sánh **Riverpod vs BLoC** — trade-offs và tiêu chí lựa chọn

### ✅ Làm được
- Sử dụng thành thạo `BlocProvider`, `BlocBuilder`, `BlocListener`, `BlocConsumer`
- Viết Cubit và Bloc hoàn chỉnh với events và states
- Viết unit test cho BLoC với `bloc_test`

### 🚫 Chưa cần biết
- BLoC + Clean Architecture kết hợp (Buổi 09)
- Multi-BLoC communication patterns phức tạp
- HydratedBloc cho persistent state (nâng cao)

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| **BLoC — Unidirectional data flow** | 🔴 | Event → Bloc → State. Mental model bắt buộc |
| **Events, States, Bloc Class** | 🔴 | Cấu trúc core — phải biết thiết kế Event/State |
| Equatable | 🟡 | AI viết tốt, cần hiểu tại sao cần |
| **Cubit vs Bloc** | 🔴 | Decision: khi nào Cubit, khi nào full Bloc |
| flutter_bloc Widgets | 🟡 | BlocProvider, BlocBuilder — syntax AI viết tốt |
| **context.read vs context.watch** | 🔴 | Sai = rebuild thừa |
| Testing BLoC | 🟡 | bloc_test API — AI viết test rất tốt |
| Riverpod vs BLoC | 🟡 | Trade-offs — đọc hiểu |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD1: Counter Cubit | 🔴 | Hiểu Cubit flow — tự viết |
| VD2: Counter Bloc | 🔴 | Hiểu Event→State — tự viết |
| VD3: BlocBuilder + Listener | 🟡 | Side effects pattern |
| VD4: Complete Todo Bloc | 🟡 | Full CRUD — tự viết hoặc AI assist |
| VD5: Unit Testing | 🟢 | AI viết test tốt |
| BT1 ⭐ Counter Cubit | 🔴 | Foundation — tự viết |
| BT2 ⭐⭐ Todo Bloc | 🟡 | Full CRUD pattern |
| BT3 ⭐⭐⭐ Auth Flow Bloc | 🟡 | Complex flow — AI assist, bạn design state |

---

## 📋 Nội dung chi tiết

| Phần | Nội dung | Thời lượng |
|------|----------|------------|
| 1 | BLoC là gì? Concept & unidirectional data flow | ~20 phút |
| 2 | Events, States, Bloc class — kiến trúc core | ~30 phút |
| 3 | Cubit vs Bloc — khi nào dùng cái nào | ~20 phút |
| 4 | flutter_bloc widgets — Provider, Builder, Listener, Consumer | ~30 phút |
| 5 | Testing BLoC — bloc_test package | ~20 phút |
| 6 | So sánh Riverpod vs BLoC — trade-offs & lựa chọn | ~20 phút |

**Tổng thời lượng: ~140 phút** (2h20, chia 2 buổi hoặc 1 buổi intensive)

---

## 📚 Tài liệu cần đọc trước

- [ ] [bloclibrary.dev](https://bloclibrary.dev) — trang chủ BLoC
- [ ] Ôn lại buổi 06 (setState, Provider) và buổi 07 (Riverpod)
- [ ] Hiểu concept `Stream` trong Dart (buổi 02)

---

## 🛠 Cài đặt cần thiết

```yaml
# pubspec.yaml
dependencies:
  flutter_bloc: ^8.1.5
  equatable: ^2.0.5

dev_dependencies:
  bloc_test: ^9.1.1
  mocktail: ^1.0.4
```

---

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter app | `flutter run` | Counter app với Cubit (increment, decrement, reset) |
| BT2 | ⭐⭐ | Flutter app | `flutter run` | Todo app với full Bloc pattern (events, states, bloc) |
| BT3 | ⭐⭐⭐ | Flutter app | `flutter run` | Authentication flow với Bloc + BlocListener navigation |
| AI-BT1 | ⭐⭐⭐ | Flutter app | `flutter run` | 🤖 Convert Riverpod Weather → BLoC (AI gen + review) |

---

## 📅 Tiến độ chương trình

> Buổi 8/16 — Hoàn thành 50% lộ trình
> ████████░░░░░░░░

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
