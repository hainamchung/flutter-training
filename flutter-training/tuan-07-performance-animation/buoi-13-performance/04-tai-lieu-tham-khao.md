# Buổi 13: Performance Optimization — Tài liệu tham khảo

## 📚 Tài liệu chính thức (Official)

### Flutter Performance
| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Flutter Performance Overview | https://docs.flutter.dev/perf | Tổng quan performance best practices |
| Flutter Performance Best Practices | https://docs.flutter.dev/perf/best-practices | Checklist tối ưu performance |
| Rendering Pipeline | https://docs.flutter.dev/resources/architectural-overview#rendering-and-layout | Build → Layout → Paint |
| Flutter Performance Profiling | https://docs.flutter.dev/perf/ui-performance | Hướng dẫn profile app |

### Flutter DevTools
| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| DevTools Overview | https://docs.flutter.dev/tools/devtools/overview | Tổng quan tất cả tools |
| Performance View | https://docs.flutter.dev/tools/devtools/performance | Frame chart, timeline, jank detection |
| CPU Profiler | https://docs.flutter.dev/tools/devtools/cpu-profiler | Flame chart, call stack analysis |
| Memory View | https://docs.flutter.dev/tools/devtools/memory | Snapshots, leak detection |
| Widget Inspector | https://docs.flutter.dev/tools/devtools/inspector | Widget tree, rebuild tracking |

### Dart Isolates
| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Dart Concurrency | https://dart.dev/language/concurrency | Event loop, isolates overview |
| Isolates Documentation | https://dart.dev/language/isolates | Chi tiết Isolate API |
| Background Parsing (Cookbook) | https://docs.flutter.dev/cookbook/networking/background-parsing | compute() thực tế |

---

## 📦 Packages hữu ích

### Performance Testing & Analysis
| Package | Pub.dev | Mục đích |
|---------|---------|----------|
| `flutter_test` | Built-in | Performance testing, widget testing |
| `leak_tracker` | https://pub.dev/packages/leak_tracker | Phát hiện memory leaks trong tests |
| `leak_tracker_flutter_testing` | https://pub.dev/packages/leak_tracker_flutter_testing | Leak tracking cho Flutter widget tests |
| `benchmark_harness` | https://pub.dev/packages/benchmark_harness | Micro-benchmarking Dart code |

### Performance Optimization
| Package | Pub.dev | Mục đích |
|---------|---------|----------|
| `cached_network_image` | https://pub.dev/packages/cached_network_image | Cache network images, tránh re-download |
| `flutter_cache_manager` | https://pub.dev/packages/flutter_cache_manager | Generic cache manager |
| `visibility_detector` | https://pub.dev/packages/visibility_detector | Detect widget visibility để lazy load |

### Profiling & Monitoring
| Package | Pub.dev | Mục đích |
|---------|---------|----------|
| `firebase_performance` | https://pub.dev/packages/firebase_performance | Production performance monitoring |
| `sentry_flutter` | https://pub.dev/packages/sentry_flutter | Error + performance tracking |

---

## 📝 Blogs & Articles

### Flutter Team / Official
| Bài viết | Link | Nội dung |
|----------|------|----------|
| Performance Best Practices | https://docs.flutter.dev/perf/best-practices | Official checklist |
| Reducing Widget Rebuilds | https://docs.flutter.dev/perf/best-practices#controlling-build-cost | const, splitting, builders |
| Impeller Rendering Engine | https://docs.flutter.dev/perf/impeller | Engine mới thay thế Skia |

### Community
| Bài viết | Tác giả | Nội dung |
|----------|---------|----------|
| Flutter Performance Tips | Very Good Ventures | Tổng hợp tips thực tế |
| Understanding Flutter's rendering pipeline | Medium / Flutter Community | Deep dive 3 phases |
| Memory management & leak detection | Flutter documentation | Hướng dẫn debug memory leaks |

---

## 🎥 Videos

### Official Flutter Channel
| Video | Nội dung |
|-------|----------|
| "How Flutter renders Widgets" — Flutter team | Rendering pipeline chi tiết |
| "Flutter Performance Tips" — Flutter team | Tips & tricks từ Flutter team |
| "Isolates and compute() in Flutter" — Flutter team | Khi nào và cách dùng Isolates |
| "DevTools Deep Dive" — Flutter team | Walkthrough đầy đủ DevTools |

### Performance Deep Dives
| Video | Nội dung |
|-------|----------|
| "60fps Flutter Apps" — Google I/O talks | Đạt 60fps, profile techniques |
| "Shader Compilation & Impeller" — Flutter Forward | Impeller vs Skia |
| "Flutter Memory Management" — talks | Quản lý memory, tránh leaks |

---

## 🔧 Công cụ

| Công cụ | Mục đích | Cách dùng |
|---------|----------|-----------|
| Flutter DevTools | Profiling tổng hợp | `flutter run` → nhấn 'd' |
| Performance Overlay | Real-time FPS overlay | `showPerformanceOverlay: true` |
| `flutter run --profile` | Profile mode build | Luôn dùng khi profiling |
| `flutter analyze` | Static analysis | Tìm linting issues |
| `dart fix --apply` | Auto-fix lint issues | Fix recommended changes |
| Observatory (Dart VM) | Low-level VM profiling | Cho advanced debugging |

---

## 📖 Đọc thêm theo chủ đề

### Nếu muốn tìm hiểu sâu hơn:

**Rendering Pipeline:**
- "Inside Flutter" — https://docs.flutter.dev/resources/inside-flutter
- Flutter architectural overview — https://docs.flutter.dev/resources/architectural-overview

**Isolates nâng cao:**
- Dart Isolates documentation — https://dart.dev/language/isolates
- `Isolate.run()` (Dart 2.19+) — simplified API

**Memory Management:**
- Dart memory management — https://dart.dev/language/memory
- LeakTracking guide — xem package `leak_tracker` README

**Production Monitoring:**
- Firebase Performance Monitoring — https://firebase.google.com/docs/perf-mon/flutter/get-started
- Sentry performance monitoring — https://docs.sentry.io/platforms/flutter/performance/

---

## 🤖 AI Prompt Library — Buổi 13: Performance Optimization

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Performance Optimization trong Flutter. Background: 4+ năm React (React.memo, useMemo, useCallback, virtual DOM).
Câu hỏi: Flutter rendering pipeline (build/layout/paint) tương đương React lifecycle phases nào? const widget giống React.memo? RepaintBoundary giống gì? shouldRebuild vs React.memo?
Yêu cầu: mapping 1-1 với React optimization concepts, highlight khác biệt Flutter (3 trees, RenderObject).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần optimize Flutter screen có performance issues.
Current code: [paste code]
Symptoms: jank khi scroll, FPS < 30, build time > 16ms.
Yêu cầu: annotate code với optimization points, priority-ordered fixes.
Constraints: keep functionality unchanged, quantify expected improvement.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Flutter code sau cho performance:
[paste code]

Kiểm tra:
1. Unnecessary rebuilds: setState scope, thiếu const, non-split widgets?
2. ListView: dùng .builder? itemExtent set? cacheExtent?
3. RepaintBoundary: positions hợp lý? Không overuse?
4. Memory: dispose, cancel subscriptions, mounted check?
5. Images: cached? cacheWidth/cacheHeight set? Giải phóng khi off-screen?
6. Isolates: heavy computation offloaded? compute() vs Isolate.spawn?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp performance issue trong Flutter:
[mô tả: jank, freeze, memory leak, high CPU]

Code:
[paste relevant widget/screen]

DevTools data:
[paste: build time, frame rendering, memory graph nếu có]

Cần: (1) Root cause analysis, (2) Fix priority (P0/P1/P2), (3) DevTools steps để verify fix.
```

---

## 🗂️ Tổng hợp cho buổi học

```
Ưu tiên đọc (theo thứ tự):
1. flutter.dev/perf/best-practices      — Checklist chính
2. DevTools Performance view docs        — Biết dùng DevTools
3. Background parsing cookbook            — compute() thực tế
4. flutter.dev/perf/ui-performance      — Profile workflow

Ưu tiên thực hành:
1. Chạy flutter run --profile            — Quen profile mode
2. Mở DevTools → Performance view        — Đọc frame chart
3. Thử compute() với JSON parsing        — Cảm nhận sự khác biệt
4. Thử RepaintBoundary                   — Xem paint reduction
```
