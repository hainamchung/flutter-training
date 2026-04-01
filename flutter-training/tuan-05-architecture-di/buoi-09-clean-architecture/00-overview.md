# Buổi 09: Clean Architecture trong Flutter

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

## 🎯 Vị trí trong lộ trình

```
Tuần 1-2: Dart + Widget ✅
Tuần 3: Navigation + State cơ bản ✅
Tuần 4: State Management nâng cao ✅
Tuần 5: 🔵 Architecture & DI ◄── BẠN ĐANG Ở ĐÂY
Tuần 6: Networking & Data
Tuần 7: Performance & Animation
Tuần 8: Platform Integration & Production
```

```
State Management ──▶ [BẠN ĐANG Ở ĐÂY: Architecture] ──▶ Networking ──▶ Production
     (BLoC/Riverpod)        (Clean Architecture)           (API/DB)       (Deploy)
```

**Buổi 9/16** — Tiến độ: ██████████░░░░░░░░ **56%**

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Clean Architecture là xương sống tổ chức code trong mọi dự án Flutter quy mô trung bình trở lên. Khi team có từ 2-3 người trở lên, việc tách rõ domain, data, và presentation layer giúp mỗi người làm việc độc lập mà không conflict. Đây là kiến thức bắt buộc trước khi bước vào networking, storage, và production.

## 💼 Đóng góp vào dự án thực tế

- **Tách layer rõ ràng** — domain/data/presentation giúp dễ test, maintain, và scale team
- **Dependency Rule** — layer trong không phụ thuộc layer ngoài → thay đổi database không ảnh hưởng UI
- **Feature-first folder structure** — team mới onboard hiểu ngay cấu trúc project
- **Repository pattern** — swap data source (API ↔ local DB) mà không đổi business logic
- **Testability** — mock dễ dàng vì mỗi layer có interface riêng

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| **Clean Architecture là gì** | 🔴 | **Kiến thức nền tảng kiến trúc.** AI không thể architect |
| Tại sao cần Clean Arch | 🔴 | Tách concern, testable, maintainable |
| **SOLID Principles** | 🔴 | SRP, DIP — áp dụng hàng ngày |
| **Adapted 3 Layers cho Flutter** | 🔴 | Domain → Data → Presentation. Phải vẽ được |
| **Domain Layer** — Entities, Use Cases | 🔴 | Trái tim app — business logic pure |
| **Repository Interfaces** | 🔴 | Dependency Inversion — interface ở Domain |
| Data Layer — Models, DataSources | 🔴 | DTO ↔ Entity mapping, DataSource abstraction |
| Presentation Layer | 🟡 | Wiring layers — AI scaffold tốt |
| **Dependency Rule** | 🔴 | **Quan trọng nhất:** chỉ hướng vào trong |
| Feature-first Folder Structure | 🟡 | AI tạo folder tốt, cần biết pattern |
| Khi nào over-engineering | 🟡 | CRUD đơn giản không cần full Clean Arch |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| Tất cả VD buổi 09 | 🔴 | Clean Arch phải tự viết để hiểu layer separation |
| BT1 ⭐ Xác định layers | 🔴 | Phân tích code — AI không làm hộ |
| BT2 ⭐⭐ Tái cấu trúc Todo | 🔴 | Refactor spaghetti → Clean Arch |
| BT3 ⭐⭐⭐ Feature Notes | 🟡 | Full implementation — AI scaffold, bạn thiết kế |

---

## 📋 Nội dung buổi học

| Phần | Chủ đề | Thời lượng |
|------|--------|------------|
| 1 | Clean Architecture là gì? Tại sao cần? | ~20 phút |
| 2 | Domain Layer — Entities, Use Cases, Repository interfaces | ~30 phút |
| 3 | Data Layer — Models, Repository impl, Data Sources | ~30 phút |
| 4 | Presentation Layer — Widgets, State, Pages | ~30 phút |
| 5 | Dependency Rule — Quy tắc phụ thuộc | ~15 phút |
| 6 | Cấu trúc folder chuẩn — Feature-first approach | ~15 phút |

**Tổng thời lượng:** ~2 giờ 20 phút (bao gồm thực hành)

---

## 📅 Tiến độ chương trình

> Buổi 9/16 — Hoàn thành 56% lộ trình
> █████████░░░░░░░

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

## 🏗️ Mục tiêu học tập

### ✅ Hiểu được
- Nguyên tắc Clean Architecture và tại sao cần áp dụng trong Flutter
- Phân biệt 3 layer: Domain, Data, Presentation — mỗi layer làm gì
- Dependency Rule — layer trong không bao giờ phụ thuộc layer ngoài
- Khi nào nên và không nên áp dụng Clean Architecture

### ✅ Làm được
- Tổ chức folder structure theo feature-first approach
- Xây dựng một feature hoàn chỉnh theo Clean Architecture từ đầu đến cuối
- Tạo Entity, UseCase, Repository interface ở domain layer
- Implement Repository, DataSource, Model ở data layer

### 🚫 Chưa cần biết
- Micro-services architecture
- Event Sourcing, CQRS patterns
- Domain-Driven Design (DDD) phức tạp

---

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ Cơ bản | Phân tích | Không | Đọc code spaghetti, xác định phần nào thuộc domain/data/presentation |
| BT2 | ⭐⭐ Trung bình | Flutter | `flutter run` | Tái cấu trúc todo app (Buổi 08) theo Clean Architecture |
| BT3 | ⭐⭐⭐ Nâng cao | Flutter | `flutter run` | Xây dựng feature "Notes" từ đầu với Clean Architecture hoàn chỉnh |

---

## 📂 Cấu trúc tài liệu

```
buoi-09-clean-architecture/
├── 00-overview.md          ← Bạn đang ở đây
├── 01-ly-thuyet.md         ← Lý thuyết chi tiết 6 phần
├── 02-vi-du.md             ← 4 ví dụ minh họa từng layer
├── 03-thuc-hanh.md         ← 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI
└── 04-tai-lieu-tham-khao.md ← Tài liệu đọc thêm
```
