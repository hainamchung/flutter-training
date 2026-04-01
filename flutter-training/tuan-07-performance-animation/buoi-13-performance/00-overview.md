# Buổi 13: Performance Optimization

> 🔴 **Tự code tay — Hiểu sâu**
> Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để hỏi khi không hiểu.

## 🎯 Vị trí trong lộ trình

```
Buổi 13/16 — Tiến độ: ████████████████░░░░ 81%
```

```
Data Layer ──▶ [🔵 BẠN ĐANG Ở ĐÂY: Performance] ──▶ Animation ──▶ Production
     │                    │
     │              Làm app NHANH
     │              và MƯỢT hơn
     │
  Tuần 1-6: Xây app hoàn chỉnh
  (Dart → Widget → Navigation → State → Architecture → Networking/Storage)
```

## 🌍 Vai trò trong hệ sinh thái Flutter

Performance optimization quyết định trải nghiệm người dùng cuối cùng. Một app đẹp nhưng giật lag sẽ bị uninstall ngay. Flutter DevTools, const widgets, lazy loading, và isolates là những công cụ giúp app đạt 60fps mượt mà. Đây là kiến thức phân biệt junior và mid-level Flutter developer.

## 💼 Đóng góp vào dự án thực tế

- **Profiling với DevTools** — identify bottleneck trước khi optimize → không waste effort
- **Const constructors + widget splitting** — giảm unnecessary rebuilds → smooth scrolling
- **ListView.builder** — render hiệu quả danh sách hàng nghìn items
- **Memory leak detection** — tránh app crash sau thời gian dài sử dụng
- **Isolates** — offload heavy computation (image processing, parsing) → UI không bị block

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| **Rendering Pipeline** | 🔴 | Build → Layout → Paint. **AI không biết app bạn lag ở đâu** |
| Frame Budget (16.67ms) | 🔴 | Vượt budget = jank. Phải hiểu |
| FPS & Jank Detection | 🔴 | Performance Overlay — công cụ debug #1 |
| **Rebuild Optimization** | 🔴 | **Quan trọng nhất buổi 13** |
| **const Constructors** | 🔴 | Vũ khí #1 chống rebuild thừa. Thành thói quen |
| **Widget Splitting** | 🔴 | Granular widgets = ít rebuild. AI không biết tách ở đâu |
| RepaintBoundary | 🟡 | Isolate paint area — animation heavy |
| **ListView.builder (lazy)** | 🔴 | 10K items + ListView thường = OOM |
| BLoC buildWhen, Selector | 🟡 | Fine-tune rebuild — optimization layer |
| **DevTools** | 🔴 | **PHẢI biết.** Widget Inspector, Performance, Memory |
| **Memory Leaks** | 🔴 | Dispose controller, cancel subscription — sai = crash |
| **Checklist dispose()** | 🔴 | AnimController, StreamSub, TextController — PHẢI dispose |
| Isolates / compute() | 🟡 | Heavy computation = freeze UI |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD1: const Optimization | 🔴 | Before/After — phải thấy sự khác biệt |
| VD2: ListView.builder vs ListView | 🔴 | 10K items — tự chạy so sánh |
| VD3: RepaintBoundary | 🟡 | Hiểu isolation concept |
| VD4: DevTools Profiling | 🔴 | **Tự mở DevTools, đọc flame chart** |
| VD5: Isolate compute() | 🟡 | Heavy task off main thread |
| BT1 ⭐ Identify Rebuilds | 🔴 | Debug skill — AI không làm hộ |
| BT2 ⭐⭐ Optimize Janky List | 🔴 | Real-world optimization |
| BT3 ⭐⭐⭐ Profile Complex Screen | 🟡 | Advanced DevTools usage |

---

## 📋 Tổng quan buổi học

| Phần | Chủ đề | Thời lượng | Độ khó |
|------|--------|------------|--------|
| 1 | Flutter Rendering Pipeline | ~25 phút | ⭐⭐ |
| 2 | FPS & Jank Detection | ~20 phút | ⭐⭐ |
| 3 | Rebuild Optimization | ~30 phút | ⭐⭐⭐ |
| 4 | Flutter DevTools | ~25 phút | ⭐⭐ |
| 5 | Memory Leaks | ~20 phút | ⭐⭐⭐ |
| 6 | Isolates | ~20 phút | ⭐⭐⭐ |

**Tổng thời lượng:** ~140 phút (bao gồm thực hành)

## 📅 Tiến độ chương trình

> Buổi 13/16 — Hoàn thành 81% lộ trình
> █████████████░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ✅ Hoàn thành |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ✅ Hoàn thành |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ✅ Hoàn thành |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | 🔵 Đang học |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Flutter rendering pipeline (Build → Layout → Paint) và frame budget 16ms
- Jank là gì, tại sao dropped frames ảnh hưởng UX
- Memory leak patterns phổ biến trong Flutter
- Khi nào cần Isolates vs khi nào không cần

### ✅ Làm được
- Phát hiện jank và dropped frames bằng DevTools
- Tối ưu rebuilds với const constructors, widget splitting, RepaintBoundary
- Sử dụng Flutter DevTools thành thạo để profile app
- Áp dụng Isolates để offload heavy computation khỏi main thread

### 🚫 Chưa cần biết
- Custom rendering engine
- Skia/Impeller internals
- Native performance profiling (Xcode Instruments, Android Studio Profiler)

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter | `flutter run --profile` | Identify rebuilds trong widget chưa tối ưu, optimize với const + splitting |
| BT2 | ⭐⭐ | Flutter | `flutter run --profile` | Optimize danh sách 10,000 contacts — ListView vs ListView.builder |
| BT3 | ⭐⭐⭐ | Flutter | `flutter run --profile` | Profile & fix một complex screen với nhiều bottlenecks |
| 🤖 AI-BT1 | ⭐⭐⭐ | Flutter | `flutter run --profile` | Performance audit (unnecessary rebuilds + DevTools verification) |

## 💡 Tại sao Performance quan trọng?

```
User mở app → Thấy giật lag → Đánh giá 1 sao → Uninstall
User mở app → Mượt mà 60fps → "App đẹp quá!" → Đánh giá 5 sao
```

> **Từ React/Vue:** Bạn đã quen với React DevTools, Lighthouse scores, virtual DOM.
> Flutter có rendering engine riêng (Skia/Impeller) và bộ DevTools mạnh mẽ không kém.
> Buổi này sẽ giúp bạn chuyển đổi tư duy performance từ web sang mobile.

## 📂 Cấu trúc files

```
buoi-13-performance/
├── 00-overview.md          ← Bạn đang ở đây
├── 01-ly-thuyet.md         ← Lý thuyết chi tiết
├── 02-vi-du.md             ← 5 ví dụ minh họa
├── 03-thuc-hanh.md         ← 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI
└── 04-tai-lieu-tham-khao.md ← Tài liệu tham khảo
```
