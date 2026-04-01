# Buổi 12: Local Storage & Data Persistence

> 🟢 **Mức độ ưu tiên: AI-OK — AI hỗ trợ tốt**
> API/config đơn giản, AI generate tốt. Tập trung đọc hiểu + review + verify.

## 🗺️ Lộ trình học

```
Tuần 1-5: ✅ Dart → Widgets → Navigation → State Management → Architecture & DI
Tuần 6:   🔵 Networking ──▶ [BẠN ĐANG Ở ĐÂY: Local Storage] ──▶ ...
Tuần 7-8: ⬜ Performance ──▶ Production
```

**Tiến độ: Buổi 12/16 — 75% hoàn thành** 🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢⬜⬜⬜⬜

```
Networking ──▶ [📦 BẠN ĐANG Ở ĐÂY: Local Storage] ──▶ Performance ──▶ Production
     │                      │
     │                      ├── SharedPreferences (key-value)
     │                      ├── Hive (NoSQL)
     │                      ├── Drift/SQLite (relational)
     │                      ├── Caching strategy
     │                      ├── Offline-first pattern
     │                      └── Secure storage
     │
     └── Buổi 11: API calls, Dio, error handling
```

## 🌍 Vai trò trong hệ sinh thái Flutter

Local storage là phần không thể thiếu trong mọi Flutter app production. Từ lưu user preferences, cache API response, đến hỗ trợ offline mode — tất cả đều cần data persistence. SharedPreferences + Hive + Drift tạo thành bộ giải pháp hoàn chỉnh cho mọi nhu cầu lưu trữ local.

## 💼 Đóng góp vào dự án thực tế

- **SharedPreferences** — lưu settings, theme, language preference trong mọi app
- **Hive/Drift** — cache API data → app vẫn hoạt động khi mất mạng
- **Offline-first pattern** — bắt buộc cho app dùng ở vùng mạng yếu (field work, travel)
- **Secure storage** — lưu token, credentials an toàn trên device
- **Cache strategy** — giảm API calls → tiết kiệm bandwidth, tăng tốc app

---

## 📅 Tiến độ chương trình

> Buổi 12/16 — Hoàn thành 75% lộ trình
> ████████████░░░░

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

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Phân biệt các giải pháp lưu trữ: SharedPreferences, Hive, Drift (SQLite) — khi nào dùng cái nào
- Cache strategy: cache-first, network-first, stale-while-revalidate
- Offline-first architecture concept và sync patterns

### ✅ Làm được
- Triển khai CRUD với Hive và Drift cho các use case phù hợp
- Xây dựng cache layer với Repository pattern
- Bảo mật dữ liệu nhạy cảm với flutter_secure_storage
- Thiết kế offline-first flow với queue writes

### 🚫 Chưa cần biết
- Cloud Firestore, Firebase Realtime Database
- Realm database
- Custom database engines, SQLCipher

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| SharedPreferences | 🟡 | Simple key-value, API đơn giản |
| **Khi nào dùng/không dùng SharedPrefs** | 🔴 | **Lưu sensitive data = security bug** |
| Hive — NoSQL local DB | 🟡 | CRUD API AI viết rất tốt |
| Drift (SQLite) | 🟢 | Relational DB — AI setup tốt |
| Caching Strategy | 🟡 | TTL, cache invalidation — concept quan trọng |
| Offline-First Pattern | 🟡 | Sync engine, pending ops — production app |
| Sync, Conflict Resolution | 🟡 | Thiết kế phức tạp, cần hiểu concept |
| **Secure Storage** | 🔴 | **flutter_secure_storage cho token/credentials. PHẢI biết** |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: SharedPreferences | 🟢 | Simple API, AI viết |
| VD2: Hive CRUD | 🟢 | AI viết CRUD rất tốt |
| VD5: Secure Storage | 🔴 | Security — phải hiểu |
| BT1-2 | 🟢 | AI assist tốt |
| BT3 ⭐⭐⭐ Offline-First | 🟡 | Design pattern — cần hiểu concept |

---

## 📋 Nội dung chi tiết

| Phần | Nội dung | Thời lượng |
|------|----------|------------|
| 1 | SharedPreferences — key-value storage | ~20 phút |
| 2 | Hive — NoSQL database | ~30 phút |
| 3 | Drift (SQLite) — relational database | ~30 phút |
| 4 | Caching strategy & Repository pattern | ~25 phút |
| 5 | Offline-first pattern | ~25 phút |
| 6 | Secure storage | ~10 phút |

**Tổng thời lượng: ~140 phút** (2 tiếng 20 phút, bao gồm thực hành)

---

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter | `flutter run` | Save app settings (dark mode, language, font size) với SharedPreferences |
| BT2 | ⭐⭐ | Flutter | `flutter run` | Todo app với Hive — full CRUD, persist across restart |
| BT3 | ⭐⭐⭐ | Flutter | `flutter run` | Offline-first Notes app: Drift + Dio + cache-first + queue writes |
| 🤖 AI-BT1 | ⭐⭐⭐ | Flutter | `flutter run` | Gen caching layer (cache-first + TTL + offline queue) → review invalidation |

---

## 🔗 Liên kết kiến thức

```
Buổi 9: Clean Architecture     ──▶  Repository pattern (cache layer)
Buổi 10: DI & Testing          ──▶  Inject storage dependencies
Buổi 11: Networking & API      ──▶  API calls + local cache = offline-first
Buổi 12: Local Storage (NOW)   ──▶  Complete data layer
```

---

## ⚡ Điều kiện tiên quyết

- ✅ Hoàn thành Buổi 11 (Networking)
- ✅ Hiểu Repository pattern (Buổi 9)
- ✅ Biết async/await, Future, Stream (Buổi 1-2)
- ✅ Có kiến thức React/Vue: localStorage, IndexedDB
