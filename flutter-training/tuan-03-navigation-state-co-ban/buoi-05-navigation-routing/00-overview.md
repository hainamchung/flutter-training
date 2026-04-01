# Buổi 05: Navigation & Routing trong Flutter

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 📍 Vị trí trong lộ trình

```
Buổi 5/16 — Tiến độ: ██████████░░░░░░░░░░░ 31%
```

```
Tuần 1          Tuần 2                  Tuần 3                    Tuần 4
Dart cơ bản     Widget Fundamentals     Navigation & State        App hoàn chỉnh
─────────────── ─────────────────────── ────────────────────────── ──────────────
Buổi 1: Dart ✅  Buổi 3: Widget Tree ✅   [Buổi 5: Navigation] 🔵   Buổi 7
Buổi 2: Dart ✅  Buổi 4: Layout ✅        Buổi 6: State 🔵          Buổi 8
                                         nâng cao
```

```
Widget ──▶ Layout ──▶ [BẠN ĐANG Ở ĐÂY: Navigation] ──▶ State
```

---

## 🌍 Vai trò trong hệ sinh thái Flutter

GoRouter là production standard cho deep linking và auth guards. Navigation quyết định cách user di chuyển giữa các màn hình — là xương sống của user experience. Trong app thực tế, navigation cần xử lý deep linking, auth flow, và tab-based layout.

## 💼 Đóng góp vào dự án thực tế

- **Multi-screen apps** — mọi app production đều có nhiều màn hình
- **Deep linking** — user click link ngoài app → mở đúng màn hình
- **Auth guards** — chặn user chưa login vào màn hình private
- **Tab navigation** — bottom tabs + stack navigation trong mỗi tab

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Cách navigation hoạt động trong Flutter (stack-based model)
- Sự khác biệt giữa Navigator 1.0 (imperative) và GoRouter (declarative)
- Deep linking concept và cách GoRouter hỗ trợ

### ✅ Làm được
- Sử dụng Navigator 1.0 để push/pop screens
- Cấu hình GoRouter cho navigation declarative, URL-based
- Truyền data giữa các screen qua constructor, path/query params, và pop result
- Triển khai nested navigation (tab + stack navigation)

### 🚫 Chưa cần biết
- Navigator 2.0 Router API thuần (phức tạp, dùng GoRouter thay thế)
- Custom route transitions nâng cao (Buổi 14 — Animation)
- Route guards phức tạp với authentication middleware (Buổi 09)

## 🎯 Hôm nay học gì?

| # | Chủ đề | Mô tả | Thời lượng |
|---|--------|-------|------------|
| 1 | Navigator 1.0 | Push/Pop, Named Routes cơ bản | ~30 phút |
| 2 | GoRouter | Declarative routing, nested navigation | ~45 phút |
| 3 | Truyền Data giữa Screens | Arguments, return results, deep linking | ~30 phút |
| 4 | Best Practices | Route naming, guard patterns, error pages | ~15 phút |

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| Navigation Concept — Stack-based | 🔴 | Mental model push/pop stack. Phải hiểu |
| Navigator 1.0 — push/pop | 🟡 | Legacy nhưng vẫn gặp, đọc hiểu |
| Named Routes | 🟡 | Biết hạn chế, đang thay bởi GoRouter |
| GoRouter — route tree, redirect | 🟡 | AI setup tốt, nhưng redirect logic cần hiểu |
| ShellRoute | 🟡 | Persistent bottom nav — dự án thực cần |
| **Truyền Data giữa Screens** | 🔴 | **Bắt buộc hiểu.** Args, params, return data |
| Nested Navigation | 🟡 | Bottom nav + tab giữ state — phổ biến |
| Deep Linking | 🟢 | Config-heavy, AI assist rất tốt |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD/BT Navigator 1.0 | 🟡 | Practice cơ bản push/pop |
| VD/BT GoRouter setup | 🟡 | Tự setup 1 lần, sau AI assist |
| BT3 Deep Linking App | 🟢 | Config platform-specific, AI viết tốt |

---

## 📋 Nội dung chi tiết

| Phần | Chủ đề | Thời lượng | Mức độ |
|------|--------|------------|--------|
| 1 | Navigation concept trong Flutter | ~20 phút | 🟢 Cơ bản |
| 2 | Navigator 1.0 (push, pop, named routes) | ~30 phút | 🟢 Cơ bản |
| 3 | Navigator 2.0 / GoRouter | ~40 phút | 🟡 Trung bình |
| 4 | Truyền data giữa screens | ~20 phút | 🟡 Trung bình |
| 5 | Nested navigation | ~20 phút | 🟡 Trung bình |
| 6 | Deep linking | ~20 phút | 🔴 Nâng cao |

**Tổng thời lượng: ~150 phút** (chia thành 2 phiên, nghỉ giữa giờ)

## 📅 Tiến độ chương trình

> Buổi 5/16 — Hoàn thành 31% lộ trình
> █████░░░░░░░░░░░

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

## 🏋️ Bài tập

| Bài | Tên | Độ khó | Loại | Chạy | Mô tả |
|-----|-----|--------|------|------|--------|
| BT1 | Multi-screen app | ⭐ | Flutter app | `flutter run` | App 3 màn hình (Home → Detail → Settings) với Navigator 1.0 |
| BT2 | GoRouter tabs | ⭐⭐ | Flutter app | `flutter run` | App với bottom tabs (Home, Search, Profile), mỗi tab có stack riêng |
| BT3 | Deep linking app | ⭐⭐⭐ | Flutter app | `flutter run` | App hỗ trợ deep link qua GoRouter (e.g., `/product/:id`) |

## 📚 Yêu cầu trước buổi học

- ✅ Đã nắm vững Dart cơ bản (biến, function, class, async/await)
- ✅ Đã hiểu Widget tree, StatelessWidget, StatefulWidget
- ✅ Đã biết Layout system (Row, Column, Stack, Container)
- 📦 Cài đặt sẵn Flutter SDK >= 3.x
- 📦 IDE: VS Code hoặc Android Studio

## 🔗 File liên quan

- [01-ly-thuyet.md](./01-ly-thuyet.md) — Lý thuyết chi tiết
- [02-vi-du.md](./02-vi-du.md) — Ví dụ minh họa
- [03-thuc-hanh.md](./03-thuc-hanh.md) — Bài tập thực hành + 🤖 bài tập AI
- [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md) — Tài liệu tham khảo
