# Buổi 14: Animation trong Flutter

> 🟢 **Mức độ ưu tiên: AI-OK — AI hỗ trợ tốt**
> API/config đơn giản, AI generate tốt. Tập trung đọc hiểu + review + verify.

## 🎯 Vị trí trong lộ trình

```
Buổi 14/16 — Tiến độ: ████████████████░░ 88%
```

```
Performance ──▶ [📍 BẠN ĐANG Ở ĐÂY: Animation] ──▶ Platform Integration ──▶ CI/CD & Deployment
```

## 🌍 Vai trò trong hệ sinh thái Flutter

Animation là yếu tố tạo nên sự khác biệt giữa app "dùng được" và app "muốn dùng". Flutter có animation framework mạnh mẽ built-in, từ implicit animations đơn giản đến explicit animations phức tạp. Hero transitions, staggered animations, và custom painters giúp app đạt chất lượng production-grade mà user kỳ vọng.

## 💼 Đóng góp vào dự án thực tế

- **Implicit animations** — micro-interactions (button hover, card expand) tạo feel "polished"
- **Hero transitions** — smooth navigation giữa list → detail, tăng spatial awareness
- **AnimationController** — custom animations cho onboarding, loading states, empty states
- **Lottie integration** — nhận file animation từ designer, tích hợp vào app dễ dàng
- **Performance-aware animation** — đạt 60fps trong mọi animation → professional UX

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| Implicit Animations | 🟡 | AnimatedContainer, AnimatedOpacity — dùng nhiều |
| Duration & Curves | 🟡 | Easing curves — UX quan trọng |
| Khi nào Implicit vs Explicit | 🟡 | Decision cần hiểu |
| Explicit Animations | 🟡 | AnimationController, Tween — control phức tạp |
| **AnimationController lifecycle** | 🟡 | vsync, TickerProvider — **phải dispose!** |
| Staggered Animations | 🟢 | AI viết rất tốt |
| Hero Transitions | 🟡 | Shared element — dùng nhiều, API đơn giản |
| CustomPainter | 🟢 | Canvas API — AI viết painting tốt |
| Rive / Lottie | 🟢 | Integration — AI setup tốt |
| Animation Performance | 🟡 | 60fps target, avoid layout trong animation |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD Implicit animation | 🟡 | Hiểu API AnimatedX |
| VD Explicit animation | 🟡 | AnimationController — dispose! |
| VD Hero, Staggered | 🟢 | AI viết tốt, đọc hiểu |
| VD CustomPainter | 🟢 | AI viết painting code tốt |
| BT1-3 | 🟢 | AI assist tốt, focus đọc + review |

---

## 📋 Tổng quan buổi học

| Phần | Chủ đề | Thời lượng |
|------|--------|------------|
| 1 | Implicit Animations | ~25 phút |
| 2 | Explicit Animations | ~30 phút |
| 3 | Hero Transitions | ~15 phút |
| 4 | CustomPainter + Animation | ~25 phút |
| 5 | Rive / Lottie Integration | ~15 phút |
| 6 | Animation Performance Tips | ~10 phút |

**Tổng thời lượng:** ~120 phút (2 tiếng)

## 📅 Tiến độ chương trình

> Buổi 14/16 — Hoàn thành 88% lộ trình
> ██████████████░░

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
- Phân biệt implicit và explicit animation, biết khi nào dùng loại nào
- Animation framework trong Flutter: Ticker, AnimationController, Tween, Curve
- Hero animation hoạt động thế nào giữa các routes
- CustomPainter API cơ bản và cách kết hợp với animation

### ✅ Làm được
- Sử dụng AnimatedContainer, AnimatedOpacity, TweenAnimationBuilder
- Xây dựng explicit animation với AnimationController, Tween, CurvedAnimation
- Triển khai Hero transitions giữa các routes
- Tích hợp Lottie animations từ designers vào app

### 🚫 Chưa cần biết
- Rive advanced (state machines, nested artboards)
- Lottie advanced (dynamic properties, markers)
- 3D animations, Flutter 3D rendering

## 📝 Bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | Flutter | `flutter run` | Implicit animate — card mở rộng/thu gọn khi tap |
| BT2 | ⭐⭐ | Flutter | `flutter run` | Custom explicit animation — animated button với bounce effect |
| BT3 | ⭐⭐⭐ | Flutter | `flutter run` | Complex page transition + Hero + staggered animation |
| 🤖 AI-BT1 | ⭐⭐⭐ | Flutter | `flutter run` | Gen Hero + staggered animation → review dispose + curves |

## 📌 Yêu cầu trước buổi học

- Đã hoàn thành **Buổi 13: Performance Optimization**
- Hiểu StatefulWidget lifecycle (initState, dispose)
- Cài sẵn: `flutter pub add lottie` (sẽ dùng ở phần 5)
- Tải sẵn 1 file Lottie JSON từ [LottieFiles.com](https://lottiefiles.com/)

## 💡 Góc nhìn React/Vue

| Flutter | React/Vue tương đương |
|---------|----------------------|
| Implicit Animation | CSS `transition` property |
| Explicit Animation | `requestAnimationFrame` / GSAP |
| Hero | Shared layout animation (Framer Motion) |
| CustomPainter | Canvas API / SVG |
| Lottie | Lottie (giống hệt!) |
