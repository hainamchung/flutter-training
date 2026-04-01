# Buổi 11: Networking & API Integration

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 📍 Vị trí trong lộ trình

```
Tuần 1-2:        Tuần 3-4:           Tuần 5:              Tuần 6:              Tuần 7:          Tuần 8:
Dart & Widget ──▶ Navigation & ──▶ Architecture ──▶ [BẠN ĐANG Ở ĐÂY] ──▶ Local Storage ──▶ Performance
Fundamentals      State Mgmt         DI & Testing     Networking & API       & Persistence     & Deploy
                                                      Integration
```

> **Buổi 11/16** — Tiến độ: ██████████████░░░░░░ **69%**

## 🌍 Vai trò trong hệ sinh thái Flutter

Networking là trái tim của hầu hết mọi Flutter app — từ social media, e-commerce đến enterprise app, tất cả đều cần gọi API. Dio + Retrofit + Repository pattern là bộ combo chuẩn trong production Flutter. Hiểu cách xây dựng network layer robust sẽ quyết định chất lượng và độ ổn định của app.

## 💼 Đóng góp vào dự án thực tế

- **Dio + Interceptors** — logging, auth token injection, error handling tự động cho mọi API call
- **Retrofit** — type-safe API client, giảm boilerplate, dễ maintain khi API thay đổi
- **Repository pattern** — abstract data source → dễ swap giữa API và mock data khi test
- **Auth token management** — refresh token flow là yêu cầu bắt buộc trong mọi app có login
- **Error handling** — xử lý network errors gracefully → UX tốt hơn cho user

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Sự khác biệt giữa `http` và `dio`, biết khi nào dùng cái nào
- Interceptor chain hoạt động thế nào trong Dio
- JSON serialization: manual vs `json_serializable` vs `freezed`
- Auth token flow: Bearer token, refresh token, secure storage

### ✅ Làm được
- Xây dựng hệ thống interceptor cho logging, auth, error handling
- Sử dụng Retrofit để tạo type-safe API client
- Triển khai auth token management với refresh token flow
- Áp dụng error handling pattern cho network layer

### 🚫 Chưa cần biết
- GraphQL (Apollo, Ferry)
- gRPC protocol
- WebSocket advanced (real-time complex)

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| http vs dio | 🟡 | Chọn 1, hiểu trade-offs |
| **Interceptors** | 🔴 | Auth token, logging, retry — **sai = security hole** |
| **Auth Token Interceptor** | 🔴 | Production bắt buộc có. Phải hiểu flow |
| JSON Serialization | 🟡 | json_serializable — AI generate model tốt |
| **Null Safety trong JSON** | 🔴 | API trả null unexpected = crash |
| Retrofit | 🟢 | Type-safe API — AI generate rất tốt |
| **Auth Token Management** | 🔴 | Token storage, refresh — **security critical** |
| **Token Refresh Interceptor** | 🔴 | Race condition, queue management — hiểu sâu |
| **Error Handling Network** | 🔴 | DioException mapping → domain error |
| Either Pattern (fpdart) | 🟡 | Functional error handling — AI viết được |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: Basic HTTP GET | 🟡 | Chạy 1 lần |
| VD2: Dio + Interceptors | 🔴 | **Phải hiểu interceptor chain** |
| VD3: json_serializable | 🟢 | AI generate model |
| VD4-5: Retrofit, Network Layer | 🟡 | Đọc hiểu architecture |
| BT1 ⭐ Fetch Posts | 🟡 | Basic networking |
| BT2 ⭐⭐ Interceptors | 🔴 | Error handling practice |
| BT3 ⭐⭐⭐ Full API Layer | 🟡 | AI scaffold, bạn review |

---

## 📋 Nội dung chi tiết

| Phần | Chủ đề | Thời lượng | Độ khó |
|------|--------|------------|--------|
| 1 | http vs dio — So sánh & lựa chọn | ~20 phút | ⭐ |
| 2 | Interceptors — Middleware cho HTTP | ~25 phút | ⭐⭐ |
| 3 | JSON Serialization — Manual, json_serializable, freezed | ~30 phút | ⭐⭐ |
| 4 | Retrofit — Type-safe HTTP client | ~25 phút | ⭐⭐ |
| 5 | Auth Token Management — Bearer, refresh, secure storage | ~25 phút | ⭐⭐⭐ |
| 6 | Error Handling cho Network | ~15 phút | ⭐⭐ |

**Tổng thời lượng ước tính: ~140 phút** (bao gồm thực hành)

## 📅 Tiến độ chương trình

> Buổi 11/16 — Hoàn thành 69% lộ trình
> ███████████░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ✅ Hoàn thành |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ✅ Hoàn thành |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | 🔵 Đang học |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

## 📝 Bài tập

| Bài | Mô tả | Độ khó | Loại | Chạy | Thời gian |
|-----|--------|--------|------|------|----------|
| BT1 | Fetch data từ API, hiển thị trong ListView | ⭐ | Flutter | `flutter run` | 30 phút |
| BT2 | Thêm interceptor + error handling cho API layer | ⭐⭐ | Flutter | `flutter run` | 45 phút |
| BT3 | Full API layer với Retrofit + auth token management | ⭐⭐⭐ | Flutter | `flutter run` | 60 phút |
| 🤖 AI-BT1 | Gen Dio service + interceptors + 401 refresh → review race condition | ⭐⭐⭐ | Flutter | `flutter run` | 45 phút |

## 🔗 Kiến thức liên quan

```
Clean Architecture (Buổi 9)          DI & Testing (Buổi 10)
        │                                     │
        ▼                                     ▼
   Domain Layer                        Injectable/GetIt
   (Repository interface)              (Inject dependencies)
        │                                     │
        └──────────────┬──────────────────────┘
                       ▼
              ┌─────────────────┐
              │   BUỔI 11:      │
              │   Networking &   │
              │   API Integration│
              └────────┬────────┘
                       │
           ┌───────────┼───────────┐
           ▼           ▼           ▼
      Data Layer   Interceptors  Error
      (Retrofit,    (Auth, Log)  Handling
       Dio)                      (Either)
```

## ⚡ Yêu cầu trước buổi học

- [ ] Đã hoàn thành Buổi 9-10 (Clean Architecture + DI)
- [ ] Hiểu Repository pattern và Data Source
- [ ] Cài đặt sẵn: `dio`, `retrofit`, `json_serializable`, `build_runner`
- [ ] Có tài khoản test API (JSONPlaceholder hoặc reqres.in)
