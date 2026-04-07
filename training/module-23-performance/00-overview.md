# Module 23: Performance Optimization (Advanced)

## Tổng quan

Module này đi sâu vào **Flutter performance optimization** — từ performance profiling với DevTools, memory leak prevention, widget rebuild optimization, đến async performance với Isolates. Bạn sẽ học cách identify performance issues, understand rendering pipeline, và apply optimization techniques để đạt được smooth 60fps performance.

**Cycle:** CODE (đọc profiling tools) → EXPLAIN (hiểu concepts) → PRACTICE (optimize) → VERIFY.

**Prerequisite:** Hoàn thành [Module 22 — CI/CD Pipeline (Advanced)](../module-22-cicd/) (build system, CI).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Use được Flutter DevTools để profile performance và identify bottlenecks?
2. Explain được widget rebuild cycle và cách minimize unnecessary rebuilds?
3. Implement được proper dispose pattern để prevent memory leaks?
4. Optimize được ListView với ListView.builder và keys?
5. Use được RepaintBoundary và const constructors cho rendering optimization?
6. Phân biệt được keys: ValueKey, ObjectKey, GlobalKey — khi nào dùng cái nào?
7. Explain được Isolates và khi nào cần heavy computation offload?

→ Nếu **7/7 Yes** — hoàn thành Session 8 và chuyển thẳng [Capstone Project](../module-capstone-full/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

10 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | Flutter DevTools Performance | 🔴 MUST-KNOW | Timeline, CPU profiler, memory |
| 2 | Widget Rebuild Optimization | 🔴 MUST-KNOW | Keys, selective rebuilds, const |
| 3 | Memory Leak Prevention | 🔴 MUST-KNOW | dispose(), StreamSubscription, controllers |
| 4 | Const Widgets & RepaintBoundary | 🟡 SHOULD-KNOW | Minimize rebuilds, painting optimization |
| 5 | ListView Optimization | 🟡 SHOULD-KNOW | ListView.builder, itemExtent |
| 6 | Keys Deep Dive | 🟡 SHOULD-KNOW | ValueKey, ObjectKey, GlobalKey |
| 7 | Isolate & Compute | 🟡 SHOULD-KNOW | Heavy computation offload |
| 8 | Image & Network Optimization | 🟢 AI-GENERATE | Caching, sizing, compression |
| 9 | BuildContext Misuse | 🟢 AI-GENERATE | async gap, mounted check |
| 10 | Performance Patterns | 🟢 AI-GENERATE | Lazy loading, pagination |

**Phân bố:** 🔴 ~30% · 🟡 ~40% · 🟢 ~30%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Đọc DevTools usage, performance examples | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 10 concepts: profiling, optimization, patterns | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập: profile, optimize, benchmark | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + cross-check | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Profile App with DevTools | ⭐ |
| 2 | Optimize Widget Rebuilds | ⭐ |
| 3 | Fix Memory Leaks | ⭐⭐ |
| 4 | Implement Isolate Computation | ⭐⭐ |
| 5 | AI Prompt Dojo — Performance Review | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [Flutter DevTools](https://docs.flutter.dev/development/tools/devtools/overview) — Performance tools
- [Flutter Observatory](https://docs.flutter.dev/development/tools/observatory) — Dart VM profiling
- [Performance Best Practices](https://docs.flutter.dev/perf-best-practices) — Official guidelines

---

## 💡 FE Perspective

| Flutter | FE Equivalent |
|---------|---------------|
| Flutter DevTools | Chrome DevTools Performance tab |
| Timeline | Chrome DevTools Performance panel |
| Memory Profiler | Chrome DevTools Memory tab |
| Widget Inspector | React DevTools Component Tree |
| RepaintBoundary | `will-change: transform` in CSS |
| Isolate | Web Worker |
| Const constructor | React memoization |
| Keys | React keys (similar concept) |

---

## Unlocks (Post-M23)

Sau khi hoàn thành Module 23, bạn sẽ:

- **Capstone Project:** Optimize performance của capstone app trước khi submit.
- **Production Ready:** Hiểu cách maintain 60fps performance trong production.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
