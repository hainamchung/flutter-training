# Buổi 10: Dependency Injection & Testing

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 📍 Vị trí trong lộ trình

```
Buổi 10/16 — Tiến độ: ████████████░░░░ 63%
```

```
Tuần 1-4          Tuần 5                    Tuần 6-8
[Dart, Widget,  ──▶ [Clean Architecture] ──▶ [Networking] ──▶ [Production]
 Navigation,        [BẠN ĐANG Ở ĐÂY:     
 State Mgmt]         DI & Testing]         
```

---

## 🗓️ Tuần 5: Architecture & DI

| Buổi | Chủ đề | Trạng thái |
|------|--------|------------|
| Buổi 09 | Clean Architecture | ✅ Hoàn thành |
| **Buổi 10** | **Dependency Injection & Testing** | **🔵 Đang học** |

---

## 🌍 Vai trò trong hệ sinh thái Flutter

DI & Testing là nền tảng đảm bảo chất lượng code trong Flutter. Dependency Injection giúp tách biệt dependencies, làm code dễ test và maintain. Testing (unit, widget, integration) là yêu cầu bắt buộc trong mọi CI/CD pipeline ở dự án production. Không có testing, không có confidence khi deploy.

## 💼 Đóng góp vào dự án thực tế

- **get_it + injectable** — DI container chuẩn trong hầu hết Flutter project production
- **Unit test** — test UseCase, Repository đảm bảo business logic đúng trước khi merge
- **Widget test** — đảm bảo UI render đúng, không regression khi refactor
- **CI/CD pipeline** — mọi PR đều chạy test tự động → giảm bug lên production
- **Code generation** — freezed, json_serializable giảm boilerplate, tăng type safety

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| **DI là gì — Concept** | 🔴 | Dependency Injection = testable code. Hiểu WHY |
| Service Locator Pattern | 🟡 | get_it pattern — cần biết |
| get_it + injectable | 🟡 | Setup AI viết tốt, cần hiểu registration flow |
| **Testing Strategy** | 🔴 | Test Pyramid — unit/widget/integration |
| Mocking với mocktail | 🟡 | AI viết mock code tốt, concept cần hiểu |
| **Mock vs Fake vs Stub** | 🔴 | Phải phân biệt — dùng sai = test vô nghĩa |
| Test Coverage | 🟡 | Biết target, hiểu giới hạn |
| Code Generation (freezed) | 🟢 | AI generate + config rất tốt |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1-2: get_it, injectable | 🟡 | Chạy 1 lần hiểu flow |
| VD3: Unit test UseCase | 🔴 | Phải tự viết test — arrange/act/assert |
| VD4: Widget test | 🟡 | pumpWidget, find — AI viết tốt |
| VD5: freezed | 🟢 | AI generate |
| BT1-3 | 🟡 | Setup + test — AI assist tốt, đọc hiểu output |

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Dependency Injection là gì, tại sao cần DI thay vì hard-code dependencies
- Service Locator pattern vs Constructor Injection — ưu nhược điểm
- Test pyramid: unit → widget → integration — khi nào dùng loại nào
- Code generation workflow với build_runner

### ✅ Làm được
- Setup get_it + injectable cho Flutter project
- Viết unit test với mocktail — mock Repository, test UseCase
- Viết widget test với WidgetTester — find, pump, expect
- Sử dụng freezed + json_serializable cho model classes

### 🚫 Chưa cần biết
- E2E testing frameworks (Patrol, Appium)
- Performance testing, load testing
- Test-Driven Development (TDD) methodology đầy đủ

---

## ⏱️ Phân bổ thời gian

| Phần | Nội dung | Thời gian |
|------|----------|-----------|
| 1 | DI concept — tại sao cần, các pattern | ~20 phút |
| 2 | get_it + injectable — setup & sử dụng | ~30 phút |
| 3 | Testing strategy — test pyramid, chiến lược | ~20 phút |
| 4 | Unit test — mocktail, test UseCase/Repository | ~30 phút |
| 5 | Widget test — WidgetTester, find, pump | ~25 phút |
| 6 | Code generation — freezed, json_serializable, build_runner | ~25 phút |
| | **Tổng** | **~150 phút** |

---

## 📅 Tiến độ chương trình

> Buổi 10/16 — Hoàn thành 63% lộ trình
> ██████████░░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ✅ Hoàn thành |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | 🔵 Đang học |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter | `flutter run` | Setup get_it + injectable cho Notes app |
| BT2 | ⭐⭐ | Test | `flutter test` | Viết unit test cho Repository & UseCase |
| BT3 | ⭐⭐⭐ | Test | `flutter test` | Widget test + Integration test cho Notes app |

---

## 🔗 Liên kết nhanh

- [Lý thuyết](01-ly-thuyet.md)
- [Ví dụ](02-vi-du.md)
- [Thực hành](03-thuc-hanh.md) + 🤖 bài tập AI
- [Tài liệu tham khảo](04-tai-lieu-tham-khao.md)
- [Buổi trước: Clean Architecture](../buoi-09-clean-architecture/00-overview.md)
