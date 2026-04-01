# Rubric Đánh Giá Middle Mobile Developer (Flutter)

> Bộ tiêu chuẩn đánh giá năng lực Flutter Developer ở trình độ Middle, bao gồm rubric chi tiết, form tự đánh giá, peer review, và checklist phỏng vấn.

---

## 1. Tổng Quan Thang Điểm

```
┌──────────────────────────────────────────────────────────────┐
│  1 - 3  │  Junior     │  Biết cơ bản, cần hướng dẫn        │
│  4 - 7  │  Middle     │  Tự chủ, giải quyết vấn đề độc lập │
│  8 - 10 │  Senior     │  Chuyên sâu, mentor được người khác │
└──────────────────────────────────────────────────────────────┘
```

### Điều kiện đạt Middle Level

| Tiêu chí | Yêu cầu |
|----------|---------|
| Điểm tối thiểu mỗi category | **4+** (không có category nào dưới 4) |
| Điểm trung bình tổng | **6.0+** trở lên |
| Capstone project | Hoàn thành với điểm **≥ 70/100** |

---

## 2. Rubric Đánh Giá Chi Tiết

### 2.1. Dart Fundamentals

| Điểm | Mô tả |
|------|-------|
| **1-2** | Biết cú pháp cơ bản: variables, functions, if/else, loops. Chưa hiểu rõ type system. |
| **3** | Hiểu classes, inheritance, mixins ở mức cơ bản. Biết dùng `List`, `Map`, `Set`. Bắt đầu hiểu null safety. |
| **4-5** | Thành thạo null safety (`?`, `!`, `late`, `required`). Sử dụng tốt generics, extensions, enums. Hiểu `Future`, `Stream`, `async/await`. Viết code clean, tuân thủ Dart conventions. |
| **6-7** | Hiểu sâu `Isolate`, `compute()` cho heavy tasks. Biết dùng code generation (`build_runner`, `freezed`, `json_serializable`). Hiểu memory management, garbage collection cơ bản. Viết code hiệu quả với functional programming patterns. |
| **8-10** | Đọc hiểu Dart source code. Tự viết custom lint rules. Hiểu Dart VM internals. Tối ưu hóa performance ở mức low-level. |

### 2.2. Flutter UI

| Điểm | Mô tả |
|------|-------|
| **1-2** | Biết dùng basic widgets: `Text`, `Container`, `Column`, `Row`. Copy-paste layout từ ví dụ. |
| **3** | Xây dựng được UI đơn giản với `ListView`, `GridView`, `Stack`. Biết dùng `Navigator` cơ bản. Hiểu widget tree, `BuildContext`. |
| **4-5** | Tự build được UI phức tạp, responsive trên nhiều screen size. Thành thạo `CustomScrollView`, `Sliver` widgets. Sử dụng tốt `Theme`, `MediaQuery`. Biết tạo custom widgets tái sử dụng. Implement animations cơ bản (`AnimatedContainer`, `Hero`). |
| **6-7** | Tạo custom animations phức tạp (`AnimationController`, `Tween`, `CustomPainter`). Build adaptive UI cho cả mobile và tablet. Implement complex layouts hiệu quả. Hiểu rendering pipeline, khi nào widget rebuild. Biết dùng `RepaintBoundary`, `const` constructors để optimize. |
| **8-10** | Tự viết custom `RenderObject`. Hiểu sâu Flutter rendering engine. Tạo được design system/component library cho team. |

### 2.3. State Management

| Điểm | Mô tả |
|------|-------|
| **1-2** | Chỉ dùng `setState()`. Chưa hiểu khái niệm state management. |
| **3** | Biết dùng `InheritedWidget`, `Provider` cơ bản. Hiểu sự khác nhau giữa local state và app state. |
| **4-5** | Thành thạo **ít nhất 1** state management solution (BLoC hoặc Riverpod). Biết khi nào dùng local state vs global state. Implement được BLoC/Cubit pattern hoặc Riverpod providers đúng cách. Hiểu reactive programming cơ bản. |
| **6-7** | Thành thạo **cả BLoC và Riverpod**, biết ưu nhược điểm từng cái. Design được state architecture cho medium-sized app. Xử lý tốt complex state: loading, error, pagination, caching. Biết dùng `Selector`, `Consumer` để optimize rebuild. |
| **8-10** | Tự thiết kế state management solution. Hiểu sâu reactive streams. Handle được complex real-time data flows. |

### 2.4. Architecture

| Điểm | Mô tả |
|------|-------|
| **1-2** | Code tất cả trong 1 file/widget. Không có tổ chức folder rõ ràng. |
| **3** | Biết tách UI và logic. Có folder structure cơ bản. Hiểu khái niệm MVC hoặc MVVM. |
| **4-5** | Áp dụng được **Clean Architecture** hoặc tương đương. Tách rõ layers: Domain, Data, Presentation. Sử dụng Repository pattern, UseCase pattern. Dependency injection với `get_it` hoặc Riverpod. Hiểu SOLID principles và áp dụng thực tế. |
| **6-7** | Design được architecture cho cả project từ đầu. Biết trade-offs giữa các architecture patterns. Implement feature-first folder structure hiệu quả. Tạo được shared modules, packages riêng. Viết code dễ test, dễ maintain, dễ scale. |
| **8-10** | Thiết kế architecture cho large-scale app, multi-module. Mentoring team về architecture decisions. |

### 2.5. Testing

| Điểm | Mô tả |
|------|-------|
| **1-2** | Không viết test. Test thủ công bằng cách chạy app. |
| **3** | Biết viết unit test cơ bản với `flutter_test`. Hiểu khái niệm test nhưng chưa thực hành nhiều. |
| **4-5** | Viết được unit tests cho business logic, repositories. Viết được widget tests cho UI components. Sử dụng `mockito` hoặc `mocktail` để mock dependencies. Test coverage cho core logic ≥ 60%. Hiểu test pyramid: unit > widget > integration. |
| **6-7** | Viết integration tests với `integration_test` package. Implement golden tests cho UI consistency. Sử dụng BLoC test hoặc Riverpod test utilities. Biết TDD workflow: Red → Green → Refactor. Test coverage ≥ 80% cho core features. |
| **8-10** | Setup CI pipeline với automated testing. Viết custom test utilities cho team. Performance testing, stress testing. |

### 2.6. Networking

| Điểm | Mô tả |
|------|-------|
| **1-2** | Dùng `http` package gọi API đơn giản. Copy-paste code networking. |
| **3** | Hiểu HTTP methods, status codes, headers. Parse JSON thủ công với `dart:convert`. |
| **4-5** | Sử dụng thành thạo **Dio** với interceptors (auth, logging, error handling). Dùng **Retrofit** hoặc tương đương cho type-safe API calls. Implement proper error handling: network errors, timeout, server errors. Hiểu authentication flow: token-based auth, refresh token. Biết serialize/deserialize với `json_serializable` hoặc `freezed`. |
| **6-7** | Implement caching strategy (ETag, Cache-Control). Handle offline-first architecture. WebSocket integration. Multipart upload, download with progress. Implement retry logic, circuit breaker pattern. |
| **8-10** | Tự build networking layer cho team. GraphQL integration. gRPC. Optimize network cho low-bandwidth. |

### 2.7. Performance

| Điểm | Mô tả |
|------|-------|
| **1-2** | Không quan tâm đến performance. App chạy được là đủ. |
| **3** | Biết dùng `const` constructors. Tránh rebuild không cần thiết ở mức cơ bản. |
| **4-5** | Sử dụng **Flutter DevTools** để profile app. Biết dùng `const`, `RepaintBoundary`, `ListView.builder` để optimize. Hiểu widget rebuild mechanism, biết cách giảm unnecessary rebuilds. Tối ưu image loading (`cached_network_image`). Handle large lists hiệu quả. |
| **6-7** | Profile và fix jank (frame drops < 16ms). Memory profiling, detect và fix memory leaks. Lazy loading, pagination cho large datasets. App size optimization (tree shaking, deferred components). Biết dùng `Isolate` cho heavy computations. |
| **8-10** | Tối ưu rendering pipeline. Custom performance monitoring. Benchmark và compare architectures. |

### 2.8. Animation

| Tiêu chí | Mức Middle |
|---|---|
| Implicit Animation | Sử dụng thành thạo `AnimatedContainer`, `AnimatedOpacity`, `AnimatedCrossFade` |
| Explicit Animation | Hiểu `AnimationController`, `Tween`, `CurvedAnimation` |
| Hero Animation | Implement shared element transitions giữa các màn hình |
| Lottie/Rive | Tích hợp animation từ file Lottie hoặc Rive |

### 2.9. Platform Integration

| Tiêu chí | Mức Middle |
|---|---|
| Method Channel | Implement hai chiều Dart ↔ Native (iOS/Android) |
| Platform-specific UI | Sử dụng `Platform.isIOS` / `Platform.isAndroid` hoặc `.adaptive` constructors |
| Permissions | Xử lý runtime permissions (camera, location, storage) |
| Device APIs | Tích hợp camera, file picker, local notifications |

### 2.10. CI/CD & Production

| Tiêu chí | Mức Middle |
|---|---|
| Build Flavors | Cấu hình dev/staging/production environments |
| GitHub Actions | Viết workflow CI: test → build → deploy |
| App Distribution | Hiểu quy trình release Play Store / App Store |
| Code Signing | Cấu hình signing certificates (iOS) và keystore (Android) |

---

## 3. Bảng Tổng Hợp Điểm

| # | Category | Trọng số | Điểm (1-10) | Điểm × Trọng số |
|---|----------|---------|-------------|-----------------|
| 1 | Dart Fundamentals | 13% | ___ | ___ |
| 2 | Flutter UI | 17% | ___ | ___ |
| 3 | State Management | 13% | ___ | ___ |
| 4 | Architecture | 13% | ___ | ___ |
| 5 | Testing | 8% | ___ | ___ |
| 6 | Networking | 13% | ___ | ___ |
| 7 | Performance | 8% | ___ | ___ |
| 8 | Animation | 5% | ___ | ___ |
| 9 | Platform Integration | 5% | ___ | ___ |
| 10 | CI/CD & Production | 5% | ___ | ___ |
| | **TỔNG** | **100%** | | **___** |

**Kết quả:**
- [ ] Tất cả categories ≥ 4 điểm
- [ ] Điểm trung bình có trọng số ≥ 6.0
- [ ] **→ Đạt / Chưa đạt Middle Level**

---

## 4. Self-Assessment Form Template

> Mỗi member tự đánh giá **2 tuần/lần** (tuần 2, 4, 6, 8).

```markdown
# Self-Assessment Form

📅 **Kỳ đánh giá:** Tuần [2/4/6/8] - [YYYY-MM-DD]
👤 **Họ tên:** _______________

## A. Tự Đánh Giá Theo Category

| # | Category | Điểm (1-10) | Bằng chứng / Giải thích |
|---|----------|-------------|------------------------|
| 1 | Dart Fundamentals | ___ | [Ví dụ: Đã hoàn thành exercise về generics và null safety] |
| 2 | Flutter UI | ___ | [Ví dụ: Build được responsive layout cho task list screen] |
| 3 | State Management | ___ | [Ví dụ: Implement BLoC cho auth flow thành công] |
| 4 | Architecture | ___ | [Ví dụ: Setup Clean Architecture cho capstone project] |
| 5 | Testing | ___ | [Ví dụ: Viết 15 unit tests cho repository layer] |
| 6 | Networking | ___ | [Ví dụ: Setup Dio + Retrofit với error handling] |
| 7 | Performance | ___ | [Ví dụ: Dùng DevTools profile 1 screen, fix 2 jank issues] |
| 8 | Animation | ___ | [Ví dụ: Implement Hero animation và AnimatedContainer transitions] |
| 9 | Platform Integration | ___ | [Ví dụ: Implement Method Channel cho native feature] |
| 10 | CI/CD & Production | ___ | [Ví dụ: Setup GitHub Actions CI pipeline với build flavors] |

**Điểm trung bình:** ___

## B. Reflection

### Tuần vừa qua tôi đã học được gì?
> [Viết 3-5 điểm chính]

1. ___
2. ___
3. ___

### Điều gì tôi thấy khó nhất?
> [Mô tả cụ thể vấn đề gặp phải]

___

### Tôi đã giải quyết bằng cách nào?
> [Mô tả approach, resource đã dùng]

___

### Mục tiêu 2 tuần tiếp theo
> [Đặt 2-3 mục tiêu cụ thể, measurable]

1. ___
2. ___
3. ___

## C. Tham Gia Nhóm

| Tiêu chí | Tự đánh giá (1-5) |
|----------|-------------------|
| Tham dự đầy đủ buổi sync-up | ___ |
| Hoàn thành action items đúng hạn | ___ |
| Chủ động đặt câu hỏi, thảo luận | ___ |
| Hỗ trợ member khác khi cần | ___ |
| Daily check-in đều đặn | ___ |

## D. Cần Hỗ Trợ

- [ ] Cần mentor/buddy hỗ trợ thêm về: ___
- [ ] Cần thêm thời gian cho topic: ___
- [ ] Cần thay đổi cách học: ___
- [ ] Khác: ___
```

### Lịch tự đánh giá

| Kỳ | Thời điểm | Trọng tâm |
|----|-----------|-----------|
| Kỳ 1 | Cuối tuần 2 | Dart Fundamentals, Flutter UI cơ bản |
| Kỳ 2 | Cuối tuần 4 | State Management, Architecture |
| Kỳ 3 | Cuối tuần 6 | Networking, Testing |
| Kỳ 4 | Cuối tuần 8 | Tổng hợp tất cả, Performance, Animation, Platform Integration, CI/CD, Capstone |

---

## 5. Peer Review Template

> Mỗi member review **2 peers** vào cuối tuần 4 và tuần 8.

```markdown
# Peer Review Form

📅 **Ngày review:** [YYYY-MM-DD]
👤 **Người review:** _______________
👥 **Người được review:** _______________

## A. Đánh Giá Kỹ Năng

| # | Category | Điểm (1-10) | Nhận xét cụ thể |
|---|----------|-------------|-----------------|
| 1 | Dart Fundamentals | ___ | [Code Dart có clean, idiomatic không?] |
| 2 | Flutter UI | ___ | [UI code có maintainable, responsive không?] |
| 3 | State Management | ___ | [Dùng state management hợp lý không?] |
| 4 | Architecture | ___ | [Project structure có rõ ràng, scalable không?] |
| 5 | Testing | ___ | [Có viết test? Test có meaningful không?] |
| 6 | Networking | ___ | [API integration có proper error handling không?] |
| 7 | Performance | ___ | [Code có optimize cho performance không?] || 8 | Animation | ___ | [Animation có smooth, đúng UX pattern không?] |
| 9 | Platform Integration | ___ | [Xử lý platform-specific features tốt không?] |
| 10 | CI/CD & Production | ___ | [CI/CD setup có đúng quy trình, tự động hóa không?] |
## B. Code Review (dựa trên code trong repo)

### Điểm mạnh
1. ___
2. ___
3. ___

### Cần cải thiện
1. ___
2. ___
3. ___

### Code sample đáng chú ý
> [Trích dẫn 1-2 đoạn code tốt hoặc cần cải thiện, kèm giải thích]

## C. Soft Skills

| Tiêu chí | Điểm (1-5) | Nhận xét |
|----------|-----------|---------|
| Khả năng trình bày (Presenter) | ___ | ___ |
| Đóng góp trong thảo luận | ___ | ___ |
| Sẵn sàng giúp đỡ người khác | ___ | ___ |
| Tiếp nhận feedback tích cực | ___ | ___ |
| Hoàn thành đúng cam kết | ___ | ___ |

## D. Tổng Kết

### Điều tôi học được từ peer này:
___

### Lời khuyên cho peer:
___

### Overall impression (1 câu):
___
```

### Nguyên tắc Peer Review

1. **Constructive**: Góp ý xây dựng, tập trung vào code/hành vi, không cá nhân hóa
2. **Specific**: Đưa ví dụ cụ thể, tránh nhận xét chung chung
3. **Balanced**: Khen điểm tốt trước, sau đó mới nêu cần cải thiện
4. **Actionable**: Mỗi feedback kèm gợi ý cách cải thiện
5. **Respectful**: Tôn trọng, sử dụng ngôn ngữ chuyên nghiệp

---

## 6. Checklist Câu Hỏi Phỏng Vấn Flutter - Trình Độ Middle

### 6.1. Dart Fundamentals (5-7 câu)

**Lý thuyết:**
- [ ] Giải thích sự khác nhau giữa `final`, `const`, và `late` trong Dart?
- [ ] Null safety hoạt động như thế nào? Phân biệt `String?`, `String`, toán tử `!`, `??`, `?.`?
- [ ] Giải thích `Future` và `Stream` khác nhau như thế nào? Khi nào dùng cái nào?
- [ ] `async*` và `yield` hoạt động ra sao? Cho ví dụ use case.
- [ ] Extension methods là gì? Khi nào nên dùng?

**Thực hành:**
- [ ] Viết 1 function xử lý danh sách bất đồng bộ sử dụng `Future.wait` vs `Stream`
- [ ] Viết 1 generic class `Result<T>` để xử lý success/failure

### 6.2. Flutter UI (5-7 câu)

**Lý thuyết:**
- [ ] Giải thích Widget tree, Element tree, và RenderObject tree?
- [ ] Sự khác nhau giữa `StatelessWidget` và `StatefulWidget`? Lifecycle của `StatefulWidget`?
- [ ] `BuildContext` là gì? Vì sao không nên dùng `context` sau `async` gap?
- [ ] Giải thích cách `Key` hoạt động trong Flutter? Khi nào cần dùng `Key`?
- [ ] `Sliver` widgets khác gì so với regular widgets?

**Thực hành:**
- [ ] Build 1 responsive layout widget hiển thị khác nhau trên phone vs tablet
- [ ] Tạo 1 custom animated button với `AnimationController`

### 6.3. State Management (5-7 câu)

**Lý thuyết:**
- [ ] So sánh `setState`, `Provider`, `BLoC`, `Riverpod` - ưu nhược điểm từng cái?
- [ ] BLoC pattern hoạt động như thế nào? Giải thích data flow: Event → BLoC → State?
- [ ] Riverpod khác gì `Provider` package? Tại sao Riverpod được coi là thế hệ tiếp theo?
- [ ] Khi nào nên dùng local state (`setState`) vs global state management?
- [ ] Giải thích cách `Selector` / `select` giúp optimize rebuild?

**Thực hành:**
- [ ] Implement 1 authentication flow dùng BLoC: login → loading → success/error
- [ ] Tạo 1 Riverpod provider cho danh sách task với filter và search

### 6.4. Architecture (4-5 câu)

**Lý thuyết:**
- [ ] Giải thích Clean Architecture và 3 layers chính (Domain, Data, Presentation)?
- [ ] Repository pattern giải quyết vấn đề gì? Cách implement?
- [ ] UseCase pattern là gì? Khi nào cần, khi nào thừa?
- [ ] Dependency injection hoạt động thế nào? So sánh `get_it` vs Riverpod DI?
- [ ] SOLID principles áp dụng trong Flutter như thế nào? Cho ví dụ cụ thể.

**Thực hành:**
- [ ] Thiết kế folder structure cho 1 feature mới theo Clean Architecture
- [ ] Implement Repository pattern cho 1 API endpoint + local cache

### 6.5. Testing (4-5 câu)

**Lý thuyết:**
- [ ] Giải thích test pyramid: unit test, widget test, integration test?
- [ ] Mocking là gì? Khi nào dùng `mockito` vs `mocktail`?
- [ ] Widget testing khác gì với unit testing? Dùng `WidgetTester` như thế nào?
- [ ] Golden test là gì? Khi nào nên dùng?

**Thực hành:**
- [ ] Viết unit test cho 1 BLoC/Cubit xử lý login
- [ ] Viết widget test cho 1 form có validation

### 6.6. Networking (4-5 câu)

**Lý thuyết:**
- [ ] So sánh `http` package vs `Dio` - khi nào dùng cái nào?
- [ ] Interceptor trong Dio hoạt động thế nào? Liệt kê các use cases phổ biến?
- [ ] Giải thích cách implement refresh token flow?
- [ ] Error handling cho API calls: best practices?

**Thực hành:**
- [ ] Setup Dio với auth interceptor, logging interceptor
- [ ] Implement 1 API service với `Retrofit`, xử lý error cases

### 6.7. Performance (3-4 câu)

**Lý thuyết:**
- [ ] Cách dùng Flutter DevTools để profile app?
- [ ] `const` constructor giúp tối ưu performance như thế nào?
- [ ] Giải thích khi nào widget rebuild? Cách giảm unnecessary rebuilds?
- [ ] `ListView.builder` vs `ListView` - vì sao nên dùng `.builder` cho list dài?

**Thực hành:**
- [ ] Cho 1 đoạn code không tối ưu, hãy refactor để cải thiện performance
- [ ] Sử dụng DevTools để identify và fix 1 performance issue

---

### Thang Điểm Phỏng Vấn

| Kết quả | Mô tả |
|---------|-------|
| **Strong Middle (≥ 80%)** | Trả lời tốt hầu hết câu hỏi, thực hành thành thạo |
| **Middle (60-79%)** | Trả lời được phần lớn, thực hành ổn, có vài điểm cần cải thiện |
| **Borderline (50-59%)** | Nắm lý thuyết nhưng thực hành còn yếu, cần thêm thời gian |
| **Chưa đạt (< 50%)** | Cần học thêm, đề xuất lộ trình ôn tập cụ thể |

---

## Kỹ năng AI-Augmented Development

| Kỹ năng | Chưa đạt | Đạt | Tốt |
|---|---|---|---|
| **Task-to-Prompt** | Prompt chung chung, output không dùng được | Có context đủ, output cần sửa ≤30% | Prompt có đủ tech stack + constraints, output ≥80% production-ready |
| **Review AI code** | Dùng thẳng output AI không kiểm tra | Phát hiện lỗi syntax/logic cơ bản | Phát hiện architecture violations, memory leaks, edge cases bị bỏ sót |
| **AI Workflow** | Copy-paste rồi commit | Gen → review → fix → commit | Gen → review checklist → customize → test → commit |
| **Biết giới hạn AI** | Tin hoàn toàn vào AI | Verify API version với docs chính thức | Biết chính xác AI hay sai ở đâu với từng loại task |
| **Customize output** | Dùng nguyên code AI gen | Sửa theo yêu cầu sau khi gen | Chủ động dùng AI gen scaffold, tự implement business logic riêng |

---

## 📚 Tài liệu liên quan

| Tài liệu | Mô tả |
|---|---|
| [README — Tổng quan chương trình](../README.md) | Cài đặt môi trường, lộ trình 16 buổi, hướng dẫn sử dụng |
| [AI-Driven Development](../ai-toolkit/ai-driven-development.md) | Hướng dẫn sử dụng AI tools trong phát triển Flutter |
| [Reference Architecture](../project-mau/reference-architecture.md) | Kiến trúc tham chiếu cho dự án Flutter thực tế |
| [Vận hành nhóm học](../van-hanh-nhom/study-group-operations.md) | Quy trình tổ chức buổi học peer-to-peer |

---

*Tài liệu thuộc chương trình Flutter Training. Cập nhật lần cuối: 2026-03-31.*
