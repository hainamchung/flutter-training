# Rubric đánh giá Middle-level Flutter Developer

> Tiêu chuẩn đánh giá năng lực Flutter developer sau khi hoàn thành chương trình đào tạo. Dùng cho self-assessment và reviewer assessment.

---

## 1. Tổng quan

### Mục đích
- Đánh giá khách quan năng lực Flutter developer sau chương trình training
- Xác định developer đã đạt chuẩn middle-level hay chưa
- Định hướng phát triển tiếp theo cho từng cá nhân

### Cấu trúc
- **5 skill dimensions** (chiều năng lực)
- **4 levels** mỗi dimension: Beginner → Competent → Proficient → Expert
- Mỗi level có **observable behaviors** cụ thể, mapping với training modules

### Quy ước ký hiệu
- ⭐ Beginner — biết khái niệm, cần hướng dẫn
- ⭐⭐ Competent — tự làm được task cơ bản
- ⭐⭐⭐ Proficient — giải quyết vấn đề phức tạp, hiểu trade-offs
- ⭐⭐⭐⭐ Expert — thiết kế giải pháp, mentor người khác

---

## 2. Skill Dimensions & Rubric

### Dimension 1: Dart Proficiency

|| Level | Observable Behaviors | Modules |
||-------|---------------------|---------|
|| ⭐ Beginner | Viết được Dart cơ bản: variables, functions, classes. Hiểu null safety syntax nhưng hay quên. Dùng `dynamic` thay vì generic. | [M0](../module-00-dart-primer/) |
|| ⭐⭐ Competent | Sử dụng thành thạo null safety (`?`, `!`, `??`, `late`). Viết được generic classes/functions. Hiểu `Future`, `Stream`, `async/await`. Dùng collection methods (`map`, `where`, `fold`). | [M0](../module-00-dart-primer/) |
|| ⭐⭐⭐ Proficient | Viết extension methods, mixins có mục đích rõ ràng. Code gen với `build_runner`, `freezed`, `json_serializable`. | [M0](../module-00-dart-primer/), [MC](../module-advanced-C-patterns-tooling/) |
|| ⭐⭐⭐⭐ Expert | Thiết kế API library với Dart. Hiểu sâu type system (variance, covariance). Optimize performance ở Dart level. Contribute được vào Dart packages. | — |

### Dimension 2: Flutter Architecture

|| Level | Observable Behaviors | Modules |
||-------|---------------------|---------|
|| ⭐ Beginner | Biết Flutter có widget tree, hiểu `StatelessWidget` vs `StatefulWidget`. Tạo được screen đơn giản. Chưa phân biệt rõ các layers. | [M1](../module-01-app-entrypoint/), [M2](../module-02-architecture-barrel/) |
|| ⭐⭐ Competent | Hiểu và áp dụng clean architecture (presentation, domain, data layers). Sử dụng MVVM pattern đúng cách. Biết khi nào extract widget, khi nào tạo component mới. Navigation với auto_route. | [M2](../module-02-architecture-barrel/), [M7](../module-07-base-viewmodel/), [M9](../module-09-page-structure/) |
|| ⭐⭐⭐ Proficient | Thiết kế module structure cho feature mới. Handle dependency injection đúng pattern. Áp dụng Repository pattern. Quản lý shared state giữa các features. Base classes cho screen/viewmodel. | [M2](../module-02-architecture-barrel/), [M3](../module-03-common-layer/), [M17](../module-17-architecture-di/) |
|| ⭐⭐⭐⭐ Expert | Thiết kế architecture cho app phức tạp (multi-module, mono-repo). Evaluate và chọn architecture pattern phù hợp project. Định nghĩa coding conventions cho team. | — |

### Dimension 3: State Management

|| Level | Observable Behaviors | Modules |
||-------|---------------------|---------|
|| ⭐ Beginner | Hiểu concept state trong Flutter. Dùng `setState` cho UI đơn giản. Biết Riverpod tồn tại nhưng chưa dùng thành thạo. | [M4](../module-04-flutter-ui-basics/) |
|| ⭐⭐ Competent | Sử dụng Riverpod providers (Provider, StateProvider, FutureProvider, StreamProvider). Hiểu `ref.watch` vs `ref.read`. Manage loading/error/data states. Dùng `AsyncValue` đúng cách. | [M8](../module-08-riverpod-state/) |
|| ⭐⭐⭐ Proficient | Thiết kế state architecture cho feature phức tạp. StateNotifier/AsyncNotifier cho business logic. Handle optimistic updates, caching. Riverpod code generation. Hiểu provider lifecycle, auto-dispose. Flutter Hooks integration. | [M10](../module-07-base-viewmodel/), [M11](../module-11-riverpod-state/) |
|| ⭐⭐⭐⭐ Expert | Evaluate trade-offs giữa các state management approaches. Custom provider types. Performance optimization cho re-renders. State persistence strategies. | — |

### Dimension 4: Testing

|| Level | Observable Behaviors | Modules |
||-------|---------------------|---------|
|| ⭐ Beginner | Biết Flutter có testing framework. Viết được unit test đơn giản (1 function, 1 assertion). Chưa biết mock dependencies. | [M18](../module-18-testing/) |
|| ⭐⭐ Competent | Viết unit test cho ViewModel/UseCase. Mock dependencies với `mocktail`. Widget testing cơ bản (find, tap, verify). Hiểu test coverage, chạy được coverage report. | [M18](../module-18-testing/) |
|| ⭐⭐⭐ Proficient | Golden test cho UI components. Integration test cho user flows. Test async code (Future, Stream). Testing Riverpod providers. Setup test fixtures, factories. CI integration cho automated testing. | [M18](../module-18-testing/), [M19](../module-19-cicd/) |
|| ⭐⭐⭐⭐ Expert | Thiết kế test strategy cho project. Performance testing. Custom test matchers. Mutation testing concepts. TDD workflow thành thạo. | — |

### Dimension 5: Production Readiness

|| Level | Observable Behaviors | Modules |
||-------|---------------------|---------|
|| ⭐ Beginner | Build được app debug mode. Biết sự khác biệt debug vs release. Chưa hiểu CI/CD. | [M1](../module-01-app-entrypoint/) |
|| ⭐⭐ Competent | Cấu hình app flavors (dev, staging, production). Handle environment variables. Error handling cơ bản (try-catch, error boundary). Biết dùng Firebase Crashlytics hoặc Sentry. | [M1](../module-01-app-entrypoint/), [M4](../module-04-flutter-ui-basics/), [M13](../module-13-middleware-interceptor-chain/) |
|| ⭐⭐⭐ Proficient | Setup CI/CD pipeline (Codemagic, Bitbucket Pipelines). Automated build, test, deploy. Performance monitoring, profiling. Security best practices (secure storage). Localization setup. | [M14](../module-14-local-storage/), [M17](../module-17-architecture-di/), [M19](../module-19-cicd/), [M22](../module-22-cicd/), [M23](../module-23-performance/) |
|| ⭐⭐⭐⭐ Expert | Design release strategy (phased rollout, feature flags). Crash-free rate monitoring. App size optimization. Platform-specific optimizations (iOS/Android). | [Advanced A](../module-advanced-A-performance-security/) |

---

## 3. Self-assessment Checklist

Mỗi developer tự đánh giá bằng cách chọn level phù hợp nhất cho từng dimension.

```
Họ tên: ___________________
Ngày đánh giá: ___________

| Dimension              | Self-rating | Ghi chú / Minh chứng              |
|------------------------|-------------|------------------------------------|
| Dart Proficiency       | ⭐ / ⭐⭐ / ⭐⭐⭐ / ⭐⭐⭐⭐ |                                    |
| Flutter Architecture   | ⭐ / ⭐⭐ / ⭐⭐⭐ / ⭐⭐⭐⭐ |                                    |
| State Management       | ⭐ / ⭐⭐ / ⭐⭐⭐ / ⭐⭐⭐⭐ |                                    |
| Testing                | ⭐ / ⭐⭐ / ⭐⭐⭐ / ⭐⭐⭐⭐ |                                    |
| Production Readiness   | ⭐ / ⭐⭐ / ⭐⭐⭐ / ⭐⭐⭐⭐ |                                    |
```

**Hướng dẫn**: Chọn level cao nhất mà bạn **tự tin thực hiện được mà không cần tra cứu nhiều**. Kèm minh chứng cụ thể (link PR, bài tập, project).

---

## 4. Reviewer Assessment Form

Dành cho facilitator hoặc senior developer đánh giá.

```
Developer: ___________________
Reviewer: ____________________
Ngày: ________________________

| Dimension              | Rating      | Evidence                           | Feedback                |
|------------------------|-------------|------------------------------------|-------------------------|
| Dart Proficiency       | ⭐⭐        | PR #12: đã dùng generic đúng      | Cần luyện thêm streams  |
| Flutter Architecture   | ⭐⭐        | Capstone: đúng layer separation    | Base class chưa tối ưu  |
| State Management       | ⭐⭐⭐      | M8 exercise: AsyncNotifier tốt    | —                        |
| Testing                | ⭐⭐        | Coverage 65%                       | Thiếu golden test       |
| Production Readiness   | ⭐⭐        | CI pipeline chạy được             | Chưa setup Crashlytics  |

Tổng đánh giá: ☐ Pass  ☐ Conditional Pass  ☐ Not Ready
Ghi chú: ________________________________________________________________
```

### Tiêu chí đánh giá reviewer:
- Dựa trên **code thực tế** (PRs, bài tập, capstone) — không phải phỏng vấn
- So sánh với observable behaviors trong rubric
- Feedback phải **cụ thể** và **actionable**
- Rating nên conservative: chỉ cho level cao khi có bằng chứng rõ ràng

---

## 5. Module Completion → Skill Level Mapping

Bảng dưới cho thấy hoàn thành module nào sẽ unlock level nào cho mỗi dimension.

|| Module | Dart | Architecture | State Mgmt | Testing | Production |
|--------|------|-------------|------------|---------|------------|
| M0 — Dart Primer               | ⭐⭐ |      |      |      |      |
| M1 — App Entrypoint            |      | ⭐   |      |      | ⭐   |
| M2 — Architecture Barrel       |      | ⭐⭐ |      |      |      |
| M3 — Common Layer              |      | ⭐⭐ |      |      |      |
| M4 — Flutter UI Basics          |      |      |      |      | ⭐⭐ |
| M5 — Built-in Widgets          |      | ⭐⭐ |      |      |      |
| M6 — Custom Widgets & Animation |    | ⭐   | ⭐   |      |      |
| M7 — Base ViewModel & Page     |      | ⭐⭐ |      |      |      |
| M8 — Riverpod State            |      |      | ⭐⭐ |      |      |
| M9 — Page Structure            |      | ⭐⭐ |      |      |      |
| M10 — BaseViewModel & BasePage |      |      | ⭐⭐ |      |      |
| M11 — Riverpod State Advanced  |      |      | ⭐⭐⭐ |     |      |
| M12 — Data Layer              |      | ⭐⭐ |      |      |      |
| M13 — Middleware & Interceptors |    |      |      |      | ⭐⭐ |
| M14 — Local Storage            |      |      |      |      | ⭐⭐⭐ |
| M15 — Popup, Dialog & Paging  |      | ⭐⭐ | ⭐⭐ |      |      |
| M16 — Lint & Code Quality     |      |      |      |      | ⭐⭐ |
| M17 — Architecture & DI        |      | ⭐⭐⭐ |     |      | ⭐⭐⭐ |
| M18 — Testing                  |      |      |      | ⭐⭐ |      |
| M19 — CI/CD                    |      |      |      | ⭐⭐ | ⭐⭐ |
| M20 — Native Platforms          |      |      |      |      | ⭐⭐ |
| M21 — Firebase                 |      |      |      |      | ⭐⭐ |
| M22 — CI/CD Pipeline          |      |      |      | ⭐⭐ | ⭐⭐ |
| M23 — Performance              |      |      |      |      | ⭐⭐⭐ |
| MA — Performance & Security     |      |      |      |      | ⭐⭐⭐ |
| MB — Native Features           |      |      |      |      | ⭐⭐ |
| MC — Patterns & Tooling        | ⭐⭐ |      |      |      | ⭐⭐ |
| Capstone                       | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

> **Đọc bảng**: Hoàn thành M8 sẽ đạt ⭐⭐ ở State Management. Hoàn thành Capstone sẽ đạt ⭐⭐⭐ ở tất cả dimensions.

---

## 6. Exit Criteria — Middle-level Flutter Developer

### Điều kiện BẮT BUỘC (tất cả phải đạt)

- [ ] Hoàn thành tối thiểu **24/28 modules** (bao gồm M0, M2, M5, M7, M8, M12, M18)
- [ ] Capstone project **Pass** hoặc **Conditional Pass** (theo [capstone spec](../module-capstone-full/))
- [ ] Tất cả 5 dimensions đạt tối thiểu **⭐⭐ Competent**
- [ ] Ít nhất **2 dimensions** đạt **⭐⭐⭐ Proficient**
- [ ] Reviewer assessment confirm kết quả self-assessment (gap ≤ 1 level)

### Điều kiện ĐỦ (ít nhất 3/5)

- [ ] Test coverage capstone ≥ 70%
- [ ] CI pipeline capstone chạy thành công (build + test)
- [ ] Code review capstone không có critical issues
- [ ] Demo capstone thuyết phục (trả lời được câu hỏi technical)
- [ ] Peer review positive từ ≥ 2 thành viên nhóm

### Kết quả đánh giá

|| Kết quả             | Điều kiện                                          | Hành động tiếp theo                     |
||----------------------|----------------------------------------------------|-----------------------------------------|
|| **Pass** ✅          | Đạt tất cả BẮT BUỘC + ≥ 3 ĐỦ                    | Bắt đầu nhận project Flutter            |
|| **Conditional Pass** 🟡 | Đạt tất cả BẮT BUỘC + 1–2 ĐỦ                 | Hoàn thiện điều kiện ĐỦ trong 1 tuần   |
|| **Not Ready** 🔴    | Chưa đạt 1+ điều kiện BẮT BUỘC                   | Lập kế hoạch bổ sung, re-assess sau 2 tuần |

---

## 7. Hướng phát triển sau middle-level

Sau khi đạt middle-level, developer nên tiếp tục phát triển:

|| Hướng | Focus | Resources |
||-------|-------|-----------|
|| **Senior Flutter** | Architecture design, performance, mentoring | Optional modules A, B, C |
|| **Full-stack Mobile** | Platform channels, native integration | [M20](../module-20-native-platforms/) |
|| **Tech Lead** | Code review, technical decision making | Thực hành qua capstone review |

<!-- AI_VERIFY: generation-complete -->
