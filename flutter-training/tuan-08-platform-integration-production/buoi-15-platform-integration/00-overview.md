# Buổi 15: Platform Integration

> 🟢 **Mức độ ưu tiên: AI-OK — AI hỗ trợ tốt**
> API/config đơn giản, AI generate tốt. Tập trung đọc hiểu + review + verify.

> **Tiến độ:** Buổi 15/16 — 94% chương trình đào tạo

## 🗺️ Vị trí trong lộ trình

```
Animation ──▶ [🔵 BẠN ĐANG Ở ĐÂY: Platform Integration] ──▶ CI/CD & Production
```

## 🌍 Vai trò trong hệ sinh thái Flutter

Platform Integration là cầu nối giữa Flutter và native platform (iOS/Android). Khi app cần truy cập camera, location, notifications, biometrics — đều phải giao tiếp với native APIs. Hiểu Platform Channels và cách dùng plugins là kỹ năng bắt buộc để xây dựng app Flutter hoàn chỉnh cho production.

## 💼 Đóng góp vào dự án thực tế

- **Permissions handling** — xin quyền camera, location, notifications đúng cách trên cả iOS/Android
- **Native plugins** — camera, image picker, share, biometrics dùng trong hầu hết app production
- **Platform Channels** — giao tiếp custom với native code khi không có plugin sẵn
- **Platform-adaptive UI** — Cupertino trên iOS, Material trên Android → native feel
- **Plugin development** — tạo reusable plugin cho team hoặc cộng đồng

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| Platform Channels | 🟡 | MethodChannel concept — debug native issues |
| Pigeon | 🟢 | Type-safe codegen — AI viết rất tốt |
| Native Features (Camera, Geo, Push) | 🟢 | Plugin setup — AI + docs đủ |
| App Lifecycle (WidgetsBindingObserver) | 🟡 | Background/foreground — cần biết |
| Deep Linking platform setup | 🟢 | Config-heavy, AI assist |
| Plugin Development | 🟢 | Hiếm khi tự viết plugin |
| **Permissions Handling** | 🟡 | Runtime permission flow — UX quan trọng |
| Permission denied/permanently denied | 🟡 | Phải xử lý đúng flow |
| Platform-specific UI | 🟢 | Adaptive widgets — AI viết tốt |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD Platform channels | 🟡 | Hiểu mechanism |
| VD Plugin setup | 🟢 | Config — AI assist |
| VD Permissions | 🟡 | Flow cần hiểu |
| BT1-2 | 🟢 | Plugin config — AI assist tốt |
| BT3 Permission handling | 🟡 | Phải hiểu denied flow |

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Cơ chế giao tiếp giữa Dart và Native (iOS/Android) qua Platform Channels
- MethodChannel vs EventChannel vs BasicMessageChannel — khi nào dùng cái nào
- Federated plugin architecture trong Flutter
- Pigeon code generation cho type-safe platform communication

### ✅ Làm được
- Tích hợp các tính năng native phổ biến (camera, location, notifications)
- Xử lý permissions đúng cách trên cả iOS và Android
- Tạo MethodChannel giao tiếp custom với native code
- Xây dựng UI thích ứng theo từng platform (Cupertino vs Material)

### 🚫 Chưa cần biết
- Flutter embedding trong native app
- Custom Flutter engine builds
- Native module development chuyên sâu (Swift/Kotlin advanced)

## 📋 Nội dung chi tiết

| Phần | Chủ đề | Thời lượng |
|------|--------|------------|
| 1 | Platform Channels (MethodChannel, EventChannel, BasicMessageChannel) | ~30 phút |
| 2 | Pigeon — Type-safe code generation | ~25 phút |
| 3 | Native Features (Camera, Location, Notifications, Biometrics...) | ~25 phút |
| 4 | Plugin Development & Federated Architecture | ~25 phút |
| 5 | Permissions Handling | ~15 phút |
| 6 | Platform-specific UI (Cupertino vs Material) | ~20 phút |

## 📅 Tiến độ chương trình

> Buổi 15/16 — Hoàn thành 94% lộ trình
> ███████████████░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ✅ Hoàn thành |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ✅ Hoàn thành |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ✅ Hoàn thành |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ✅ Hoàn thành |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | 🔵 Đang học |

## 🏋️ Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter | `flutter run` | Sử dụng các plugin có sẵn (url_launcher, share_plus, package_info_plus) |
| BT2 | ⭐⭐ | Flutter | `flutter run` | Tạo MethodChannel giao tiếp với native — lấy device name & OS version |
| BT3 | ⭐⭐⭐ | Flutter | `flutter run` | Xây dựng tính năng chọn ảnh: xin quyền → chụp/chọn ảnh → hiển thị trong app |
| 🤖 AI-BT1 | ⭐⭐⭐ | Flutter | `flutter run` | Gen MethodChannel camera (permission + error propagation) → review |

## ⏱️ Thời lượng

- **Lý thuyết + Demo:** ~140 phút
- **Thực hành:** ~60 phút
- **Tổng:** ~200 phút (có thể chia thành 2 buổi)

## 📚 Kiến thức tiên quyết

- Hoàn thành Tuần 1–7 (Dart, Widget, Navigation, State, Architecture, Networking, Performance, Animation)
- Đã xây dựng app Flutter hoàn chỉnh với architecture, networking, local storage
- Quen thuộc với async/await, Future, Stream trong Dart
