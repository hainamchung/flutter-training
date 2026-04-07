# 04-verify.md — Performance Optimization

## VERIFY: Self-Assessment Checklist

---

## Before You Start

Đảm bảo đã hoàn thành:

- [ ] Đọc `01-code-walk.md` và trace performance patterns
- [ ] Đọc `02-concept.md` và hiểu 10 concepts
- [ ] Hoàn thành ít nhất Exercise 1 và 2
- [ ] Profile app với DevTools

---

## Self-Assessment Questions

### Section 1: Performance Profiling

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 1.1 | Slow frame được identify bằng cách nào? | ___ |
| 1.2 | Timeline view hiển thị những gì? | ___ |
| 1.3 | Widget rebuild stats enable ở đâu? | ___ |
| 1.4 | CPU Profiler dùng để làm gì? | ___ |

### Section 2: Widget Rebuild Optimization

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 2.1 | Const constructor giúp gì cho performance? | ___ |
| 2.2 | Keys trong ListView dùng để làm gì? | ___ |
| 2.3 | RepaintBoundary có tác dụng gì? | ___ |
| 2.4 | Select() trong Riverpod dùng để làm gì? | ___ |

### Section 3: Memory Leak Prevention

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 3.1 | StreamSubscription cần dispose như thế nào? | ___ |
| 3.2 | AnimationController cần dispose khi nào? | ___ |
| 3.3 | TextEditingController cần dispose khi nào? | ___ |
| 3.4 | Provider.autoDispose khác gì Provider thường? | ___ |

### Section 4: ListView Optimization

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 4.1 | ListView.builder khác ListView.children như thế nào? | ___ |
| 4.2 | itemExtent có tác dụng gì? | ___ |
| 4.3 | ValueKey vs ObjectKey khác nhau thế nào? | ___ |
| 4.4 | GlobalKey dùng để làm gì? | ___ |

### Section 5: Isolate & Async

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 5.1 | Isolate dùng để làm gì? | ___ |
| 5.2 | compute() function hoạt động như thế nào? | ___ |
| 5.3 | context.mounted kiểm tra gì? | ___ |
| 5.4 | Khi nào cần dùng isolate thay vì async/await? | ___ |

---

## Answer Key

### Section 1: Performance Profiling

| # | Answer |
|---|--------|
| 1.1 | Red bars in timeline (>16ms per frame) |
| 1.2 | Frame rendering time, UI/GPU thread activity, jank |
| 1.3 | Flutter Inspector → More Actions → Show widget rebuild counts |
| 1.4 | Sample Dart code execution, show function timing |

### Section 2: Widget Rebuild Optimization

| # | Answer |
|---|--------|
| 2.1 | Widget không rebuild khi parent rebuild, improve build time |
| 2.2 | Giúp Flutter track widgets across rebuilds, prevent unnecessary rebuilds |
| 2.3 | Isolate repaints, prevent expensive widgets from causing parent repaints |
| 2.4 | Granular rebuilds - chỉ rebuild khi selected value thay đổi |

### Section 3: Memory Leak Prevention

| # | Answer |
|---|--------|
| 3.1 | `_subscription?.cancel()` trong `dispose()` |
| 3.2 | `_controller.dispose()` trong `dispose()` |
| 3.3 | `_textController.dispose()` trong `dispose()` |
| 3.4 | autoDispose tự động dispose khi không còn reference |

### Section 4: ListView Optimization

| # | Answer |
|---|--------|
| 4.1 | builder: render only visible items; children: render all items |
| 4.2 | Improve scroll performance by specifying fixed item height |
| 4.3 | ValueKey: compare by value; ObjectKey: compare by object identity |
| 4.4 | Globally unique key, can access widget state from anywhere |

### Section 5: Isolate & Async

| # | Answer |
|---|--------|
| 5.1 | Run heavy computation in separate Dart VM to avoid blocking UI |
| 5.2 | Spawns isolate, runs function, returns result |
| 5.3 | Kiểm tra widget còn mounted không sau async operation |
| 5.4 | Khi có heavy CPU-bound computation (>1ms), không phải I/O |

---

## Badge Targets

### 🔴 MUST-KNOW (Phải trả lời đúng 100%)

- [ ] 1.1: Identify slow frames in timeline
- [ ] 2.1: Const constructor benefits
- [ ] 3.1: StreamSubscription disposal pattern
- [ ] 4.1: ListView.builder vs ListView

**Target: 4/4 correct** → Foundation concepts

### 🟡 SHOULD-KNOW (Nên trả lời đúng ≥80%)

- [ ] Section 1: 3+/4 correct
- [ ] Section 2: 3+/4 correct
- [ ] Section 3: 3+/4 correct
- [ ] Section 4: 3+/4 correct
- [ ] Section 5: 3+/4 correct

**Target: ≥15/20 correct**

### 🟢 AI-GENERATE (Understand when to use)

- [ ] Understand when to use Isolate vs async/await
- [ ] Understand RepaintBoundary use cases
- [ ] Understand image caching strategies

---

## Practical Verification

### Verify DevTools Usage

```bash
# 1. Start app with DevTools
flutter run --observe

# 2. Open DevTools
# 3. Profile performance during interaction
# 4. Identify at least one optimization opportunity
```

### Verify Code Optimizations

```bash
# Check for:
# - const constructors: grep -r "const " lib/ | wc -l
# - ListView.builder: grep -r "ListView.builder" lib/
# - dispose patterns: grep -A5 "void dispose" lib/

# Before optimization: profile build count
# After optimization: profile build count again
# Verify: build count decreased
```

### Verify Understanding

Explain to a colleague:

1. **30-second explanation:** Làm thế nào để profile Flutter app?
2. **60-second explanation:** Memory leak prevention patterns?
3. **90-second explanation:** Widget rebuild optimization techniques?

---

## Cross-Check with Module 23

Kiến thức từ Module 23 được sử dụng trong Capstone:

| M23 Concept | Capstone Usage |
|-------------|----------------|
| DevTools profiling | Profile capstone app performance |
| ListView optimization | Optimize any list in app |
| Memory leaks | Ensure no leaks before submission |
| Const constructors | Apply throughout codebase |

---

## Sign-Off

Sau khi hoàn thành checklist và đạt badge targets:

```markdown
## M23 Completion Status

- [ ] All 🔴 MUST-KNOW answered correctly
- [ ] ≥80% 🟡 SHOULD-KNOW answered correctly
- [ ] DevTools profiling verified
- [ ] At least 5 optimizations applied

**Verdict:** [ ] PASS / [ ] NEEDS REVIEW
**Date:** ___
**Notes:** ___
```

---

## If You Need Review

Nếu chưa đạt targets:

1. **Đọc lại** `02-concept.md` - tập trung vào 🔴 concepts
2. **Hoàn thành** Exercise 3 và 4
3. **Practice** DevTools trên app thật
4. **Hỏi** team member hoặc mentor

---

## Module 23 Completion

Bạn đã hoàn thành Module 23:

| Module | Status |
|--------|--------|
| M22 — CI/CD Pipeline | ✅ |
| M23 — Performance Optimization | ✅ |

### Capstone Introduction

Với kiến thức từ Module 23, bạn đã sẵn sàng cho **Capstone Project**:

- **Build & Release:** Build production-ready app
- **CI/CD:** Setup automated pipeline
- **Performance:** Optimize app trước submission

→ [Capstone Project](../module-capstone-full/)

<!-- AI_VERIFY: generation-complete -->
