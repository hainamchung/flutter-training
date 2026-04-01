# Buổi 14: Animation trong Flutter — Tài liệu tham khảo

## 📚 Tài liệu chính thức

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Introduction to animations | https://docs.flutter.dev/ui/animations | Tổng quan animation trong Flutter |
| Animations tutorial | https://docs.flutter.dev/ui/animations/tutorial | Step-by-step explicit animation |
| Implicit animations | https://docs.flutter.dev/ui/animations/implicit-animations | Hướng dẫn implicit animation |
| Hero animations | https://docs.flutter.dev/ui/animations/hero-animations | Hero transition chi tiết |
| Staggered animations | https://docs.flutter.dev/ui/animations/staggered-animations | Interval + multiple Tweens |
| Animation & motion widgets | https://docs.flutter.dev/ui/widgets/animation | Catalog tất cả animation widgets |

## 📦 Packages

| Package | Link | Mô tả |
|---------|------|--------|
| lottie | https://pub.dev/packages/lottie | Render Lottie animations (After Effects → JSON) |
| rive | https://pub.dev/packages/rive | Rive animations với state machines |
| flutter_animate | https://pub.dev/packages/flutter_animate | Simplified explicit animation API — chain animations dễ dàng |
| animations | https://pub.dev/packages/animations | Material motion transitions (Container Transform, Shared Axis...) |
| shimmer | https://pub.dev/packages/shimmer | Loading shimmer effect |
| animated_text_kit | https://pub.dev/packages/animated_text_kit | Text animation effects (typewriter, fade, scale...) |

### flutter_animate — Đáng chú ý

Giúp viết explicit animation ngắn gọn hơn rất nhiều:

```dart
// Thay vì viết AnimationController + Tween + AnimatedBuilder
// Chỉ cần:
Text('Hello').animate()
  .fadeIn(duration: 600.ms)
  .slideX(begin: -0.2, end: 0)
  .then(delay: 200.ms)
  .shake();
```

### animations (Material Motion)

Transitions theo Material Design spec:

```dart
// Container Transform — card mở rộng thành full page
OpenContainer(
  closedBuilder: (context, openContainer) => ListTile(...),
  openBuilder: (context, closeContainer) => DetailPage(...),
)

// Shared Axis — page transition dọc/ngang
SharedAxisTransition(...)

// Fade Through — switch giữa content
FadeThroughTransition(...)
```

## 📝 Bài viết & Blog

| Bài viết | Link | Nội dung |
|----------|------|----------|
| Flutter Animation Deep Dive | https://medium.com/flutter/animation-deep-dive-39d3ffea111f | Kiến trúc animation bên trong Flutter |
| Animations in Flutter — Cheat Sheet | https://medium.com/flutter-community/flutter-animations-comprehensive-guide-cb93b246ca5d | Tổng hợp tất cả loại animation |
| When to use AnimatedBuilder vs AnimatedWidget | https://blog.codemagic.io/flutter-animated-series-animated-builder/ | So sánh 2 cách build explicit animation |
| CustomPainter cookbook | https://medium.com/flutter-community/flutter-custom-painter-5fd1b2b4d6a2 | Hướng dẫn vẽ với CustomPainter |
| Performance best practices | https://docs.flutter.dev/perf/best-practices | Tối ưu performance tổng thể |

## 🎥 Video

| Video | Link | Thời lượng |
|-------|------|-----------|
| Animations — Flutter Widget of the Week | https://www.youtube.com/watch?v=IVTjpW3W33s | ~3 phút — tổng quan nhanh |
| AnimatedContainer — Widget of the Week | https://www.youtube.com/watch?v=yI-8QHpGIP4 | ~2 phút |
| Hero — Widget of the Week | https://www.youtube.com/watch?v=Be9UH1kXFDw | ~2 phút |
| CustomPaint & CustomPainter | https://www.youtube.com/watch?v=kp14Y4uKpHs | ~2 phút |
| Flutter Animation Tutorial (Full Course) | https://www.youtube.com/watch?v=txLvvlooT20 | ~45 phút — full course |
| Rive Flutter Tutorial | https://www.youtube.com/watch?v=6QZy5sYozVI | ~20 phút — tích hợp Rive |

## 🔧 Công cụ & Tài nguyên

| Tài nguyên | Link | Mô tả |
|------------|------|--------|
| LottieFiles | https://lottiefiles.com/ | Thư viện Lottie animations miễn phí + trả phí |
| Rive App | https://rive.app/ | Editor tạo interactive animations |
| Rive Community | https://rive.app/community/ | Animations miễn phí từ community |
| Flutter Curves Visualizer | https://api.flutter.dev/flutter/animation/Curves-class.html | Xem tất cả built-in curves |
| Easing Functions Cheat Sheet | https://easings.net/ | Visualize các easing functions |
| Flutter DevTools | https://docs.flutter.dev/tools/devtools/overview | Debug performance, repaint |

## 📖 Đọc thêm nâng cao

| Chủ đề | Link | Ghi chú |
|--------|------|---------|
| Physics-based animation | https://docs.flutter.dev/ui/animations/physics-simulation | Spring, gravity simulation |
| Custom implicit animations (AnimatedWidget subclass) | https://docs.flutter.dev/ui/animations/implicit-animations#creating-custom-implicit-animations | Tạo AnimatedFoo tùy chỉnh |
| Render object animations | https://api.flutter.dev/flutter/rendering/RenderObject-class.html | Animation ở tầng render (nâng cao) |
| Flutter GPU / Impeller | https://docs.flutter.dev/perf/impeller | Rendering engine mới — ảnh hưởng animation performance |

## 🗂️ Tham khảo nhanh — Decision Matrix

```
Cần gì?                        → Dùng gì?
─────────────────────────────────────────────
Thay đổi property đơn giản     → AnimatedContainer / AnimatedOpacity
Animate bất kỳ giá trị nào     → TweenAnimationBuilder
Loop / repeat animation        → AnimationController + repeat()
Staggered animation            → AnimationController + Interval
Chuyển element giữa routes     → Hero
Page transition custom         → PageRouteBuilder
Vẽ hình / chart tùy chỉnh     → CustomPainter
Animation phức tạp từ designer → Lottie / Rive
Declarative animation chains   → flutter_animate package
Material Design transitions    → animations package
```

---

## 🤖 AI Prompt Library — Buổi 14: Animation

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Animation trong Flutter. Background: 4+ năm React (CSS transitions, Framer Motion, React Spring).
Câu hỏi: Implicit animation giống CSS transition? AnimationController giống requestAnimationFrame? Hero giống Framer Motion layoutId? Tween giống interpolation?
Yêu cầu: mapping 1-1 với web animation concepts, highlight Flutter-specific (vsync, ticker, 3 trees).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần staggered animation cho onboarding screen trong Flutter.
4 elements: logo (fade 0-0.3), title (slide+fade 0.2-0.5), subtitle (slide+fade 0.4-0.7), button (scale+fade 0.6-1.0).
Single AnimationController, TickerProviderStateMixin.
Constraints: dispose controller, mounted check, UX-friendly curves.
Output: onboarding_screen.dart.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Flutter animation code sau:
[paste code]

Kiểm tra:
1. AnimationController dispose()? Memory leak?
2. vsync: TickerProviderStateMixin vs SingleTickerProviderStateMixin (multiple controllers)?
3. mounted check trước setState (async callback)?
4. Curves: UX-appropriate? (không linear?)
5. Performance: RepaintBoundary cho expensive animations?
6. Lottie/Rive: dispose, cache, visibility-aware?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi animation trong Flutter:
[paste error — ticker, dispose, setState after dispose]

Code:
[paste animation widget]

Cần: (1) Nguyên nhân (lifecycle issue?), (2) Fix, (3) Pattern chuẩn cho animation lifecycle.
```
