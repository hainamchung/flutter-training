# Hướng Dẫn AI-Driven Development Cho Flutter

> Tài liệu hướng dẫn sử dụng AI tools để tăng tốc Flutter development, bao gồm setup, system prompts, workflow, và best practices.

---

## 1. Overview: AI Tools Trong Flutter Development

### 1.1. Các AI Tools Phổ Biến

| Tool | Loại | Ưu điểm | Nhược điểm | Giá |
|------|------|---------|------------|-----|
| **GitHub Copilot** | Code completion + Chat | Tích hợp sâu VS Code, context-aware, agent mode | Cần subscription | $10-19/tháng |
| **Cursor** | AI-first IDE | Codebase-aware, multi-file edit, chat mạnh | IDE riêng (fork VS Code) | Free tier / $20/tháng |
| **ChatGPT** | Chat-based | Giải thích tốt, versatile, GPT-4o mạnh | Không có code context trực tiếp | Free / $20/tháng |
| **Claude** | Chat-based | Reasoning tốt, context window lớn | Không tích hợp IDE trực tiếp | Free / $20/tháng |
| **Gemini** | Chat-based + IDE | Tích hợp Android Studio, hiểu Flutter tốt | Còn mới, chưa ổn định | Free / Paid |

### 1.2. Khi Nào Dùng Tool Nào?

```
Viết code mới / boilerplate     → GitHub Copilot / Cursor
Debug lỗi phức tạp              → ChatGPT / Claude (paste error + code)
Giải thích concept               → ChatGPT / Claude
Refactor code lớn                → Cursor (multi-file edit)
Review code                      → GitHub Copilot Chat / Cursor
Generate tests                   → GitHub Copilot / Cursor
Thiết kế architecture            → ChatGPT / Claude (discussion)
```

---

## 2. Setup AI Tools Cho Flutter Development

### 2.1. GitHub Copilot trong VS Code

**Bước 1: Cài đặt Extensions**
1. Mở VS Code → Extensions (Cmd+Shift+X)
2. Tìm và cài:
   - `GitHub Copilot` (extension chính)
   - `GitHub Copilot Chat` (chat interface)
3. Đăng nhập GitHub account (cần subscription active)

**Bước 2: Cấu hình cho Flutter**

Thêm vào `settings.json`:
```json
{
  "github.copilot.enable": {
    "*": true,
    "dart": true,
    "yaml": true,
    "markdown": true
  },
  "github.copilot.advanced": {
    "indentationMode": {
      "dart": "server"
    }
  }
}
```

**Bước 3: Tạo Copilot Instructions file**

Tạo file `.github/copilot-instructions.md` trong root project:
```markdown
# Copilot Instructions for Flutter Project

## Tech Stack
- Flutter 3.x with Dart 3.x
- State Management: Riverpod (or BLoC)
- Architecture: Clean Architecture (feature-first)
- Networking: Dio + Retrofit
- Local Storage: Hive
- Testing: flutter_test, mockito, bloc_test

## Code Style
- Follow official Dart style guide
- Use trailing commas for better formatting
- Prefer const constructors where possible
- Use named parameters for Widget constructors
- File naming: snake_case
- Class naming: PascalCase

## Architecture Rules
- Domain layer has NO dependencies on other layers
- Data layer implements Repository interfaces from Domain
- Presentation layer uses only UseCases from Domain
- Each feature has its own folder with domain/data/presentation subfolders
```

**Bước 4: Custom Chat Participants**

Sử dụng `@workspace` trong Copilot Chat để hỏi về toàn bộ project:
```
@workspace Giải thích kiến trúc của project này
@workspace Tìm tất cả API endpoints đang dùng
```

### 2.2. Cursor cho Flutter Development

**Bước 1: Cài đặt**
1. Download Cursor từ [cursor.com](https://cursor.com)
2. Import settings từ VS Code (Cursor hỗ trợ auto-import)
3. Cài Flutter/Dart extensions tương tự VS Code

**Bước 2: Cấu hình Cursor Rules**

Tạo file `.cursor/rules/flutter.mdc` trong root project:
```markdown
---
description: Rules for Flutter development
globs: "**/*.dart"
---

You are a Senior Flutter Developer. Follow these rules:

1. Use Clean Architecture with feature-first folder structure
2. State management: Riverpod with code generation
3. Use freezed for immutable data classes
4. Use Dio + Retrofit for networking
5. Always handle errors with Result type pattern
6. Write const constructors where possible
7. Follow Dart effective style guide
8. Add trailing commas for better git diffs
```

**Bước 3: Sử dụng Cursor hiệu quả**

| Tính năng | Phím tắt | Mô tả |
|-----------|----------|-------|
| Inline edit | Cmd+K | Edit code tại chỗ với AI |
| Chat | Cmd+L | Mở chat panel |
| Composer | Cmd+I | Multi-file edit |
| Codebase search | @codebase | Reference toàn bộ project |
| File reference | @file | Reference file cụ thể |

---

## 3. System Prompts Cho Từng Task

### 3.1. Generate Widget Boilerplate

**Prompt Template:**
```
Tạo một Flutter StatelessWidget/StatefulWidget cho [mô tả screen/component].

Yêu cầu:
- Dart 3.x, null safety
- Dùng const constructor nếu có thể
- Responsive layout dùng LayoutBuilder hoặc MediaQuery
- Tách thành private methods cho từng section
- Sử dụng Theme.of(context) cho styling, không hardcode colors/fonts
- Thêm trailing commas
- Named parameters cho constructor

Context:
- State management: [Riverpod/BLoC]
- Navigation: [GoRouter]
- Đây là phần [feature name] trong app [app name]

Output: Chỉ code Dart, có comments giải thích.
```

**Ví dụ cụ thể:**
```
Tạo một StatelessWidget tên TaskListScreen hiển thị danh sách tasks.

Yêu cầu:
- Dùng Riverpod ConsumerWidget
- ListView.builder để hiển thị tasks
- Mỗi item hiển thị: title, description, priority badge, due date
- Pull-to-refresh
- Empty state khi không có task
- FAB để thêm task mới
- Tách thành private methods: _buildTaskItem, _buildEmptyState

Context:
- State management: Riverpod
- Model: Task(id, title, description, priority, dueDate, isCompleted)
- Provider: taskListProvider đã có sẵn
```

### 3.2. Generate BLoC/Riverpod State Management Code

**Prompt cho BLoC:**
```
Tạo BLoC pattern hoàn chỉnh cho feature [tên feature].

Bao gồm:
1. Events: [liệt kê events]
2. States: loading, loaded, error (dùng freezed)
3. BLoC class xử lý logic
4. Sử dụng injectable/get_it cho DI

Quy tắc:
- Dùng flutter_bloc package
- State immutable với freezed
- Xử lý error cases đầy đủ
- Emit loading state trước khi gọi API
- Không có business logic trong UI

Use case: [mô tả input/output]
Repository interface: [mô tả methods]
```

**Prompt cho Riverpod:**
```
Tạo Riverpod providers cho feature [tên feature].

Bao gồm:
1. State class (dùng freezed)
2. StateNotifier hoặc AsyncNotifier
3. Providers cần thiết
4. Repository provider

Quy tắc:
- Dùng riverpod_annotation + code generation
- AsyncValue cho async operations
- Proper error handling
- Tách state logic ra khỏi UI

Models đã có: [liệt kê models]
Repository methods: [liệt kê methods]
```

**Ví dụ cụ thể (Riverpod):**
```
Tạo Riverpod providers cho feature Authentication.

Bao gồm:
1. AuthState: initial, loading, authenticated(User), unauthenticated, error(String)
2. AuthNotifier: login(email, password), logout(), checkAuthStatus()
3. Providers: authNotifierProvider, currentUserProvider, isLoggedInProvider

Repository methods:
- Future<User> login(String email, String password)
- Future<void> logout()
- Future<User?> getCurrentUser()
- Future<String?> getStoredToken()

Dùng riverpod_annotation (@riverpod). Xử lý token storage.
```

### 3.3. Generate API Integration Code

**Prompt Template:**
```
Tạo API integration layer cho [tên endpoint/feature].

Bao gồm:
1. Data models (request/response) dùng freezed + json_serializable
2. Retrofit abstract class cho API service
3. Repository implementation
4. Error handling với custom exceptions

API spec:
- Base URL: [URL]
- Endpoints:
  * [METHOD] [path] - [mô tả]
  * [METHOD] [path] - [mô tả]
- Authentication: Bearer token
- Response format: JSON

Quy tắc:
- Dio với interceptors đã setup sẵn
- Retrofit cho type-safe API calls
- Xử lý HTTP errors: 400, 401, 403, 404, 500
- Parse error message từ server response
- Dùng Result<T> pattern cho return type
```

**Ví dụ cụ thể:**
```
Tạo API integration cho Task CRUD operations.

API spec:
- Base URL: https://api.example.com/v1
- Endpoints:
  * GET /tasks?page=1&limit=20&status=active - Lấy danh sách tasks
  * GET /tasks/:id - Lấy chi tiết 1 task
  * POST /tasks - Tạo task mới (body: {title, description, priority, dueDate})
  * PUT /tasks/:id - Cập nhật task
  * DELETE /tasks/:id - Xóa task
- Auth: Bearer token trong header
- Response: { "data": {...}, "message": "success" }
- Error: { "error": { "code": 400, "message": "..." } }

Tạo: TaskModel, TaskRequest, Retrofit TaskApiService, TaskRepositoryImpl.
```

### 3.4. Generate Test Cases

**Prompt Template:**
```
Viết tests cho [class/function name].

Loại test: [unit test / widget test / integration test]

Class/Function cần test:
[paste code hoặc mô tả]

Yêu cầu:
- Dùng flutter_test
- Mock dependencies với mocktail
- Cover: happy path, error cases, edge cases
- Arrange-Act-Assert pattern
- Group tests theo logic
- Descriptive test names bằng tiếng Anh

Dependencies cần mock: [liệt kê]
Edge cases cần cover: [liệt kê]
```

**Ví dụ cụ thể:**
```
Viết unit tests cho TaskBloc.

TaskBloc xử lý:
- LoadTasksEvent → gọi GetTasksUseCase → emit TasksLoaded hoặc TasksError
- AddTaskEvent → gọi AddTaskUseCase → emit thêm task vào list hoặc error
- DeleteTaskEvent → gọi DeleteTaskUseCase → emit bỏ task khỏi list hoặc error

Dependencies cần mock: GetTasksUseCase, AddTaskUseCase, DeleteTaskUseCase

Test cases cần cover:
1. Initial state là TasksInitial
2. LoadTasksEvent thành công → TasksLoaded với list tasks
3. LoadTasksEvent thất bại → TasksError với message
4. AddTaskEvent thành công → thêm task vào loaded list  
5. AddTaskEvent khi chưa load → error
6. DeleteTaskEvent thành công → bỏ task khỏi list
7. DeleteTaskEvent task không tồn tại → error
```

### 3.5. Debug Flutter Errors

**Prompt Template:**
```
Tôi gặp lỗi Flutter sau, hãy giúp tôi debug:

**Error message:**
[Paste full error/stack trace]

**Code gây lỗi:**
[Paste relevant code]

**Context:**
- Flutter version: [version]
- Đang làm gì khi lỗi xảy ra: [mô tả]
- Đã thử: [liệt kê những gì đã thử]
- OS/Device: [iOS/Android, emulator/real device]

Hãy:
1. Giải thích nguyên nhân lỗi
2. Đưa ra solution cụ thể (code fix)
3. Giải thích tại sao fix này hoạt động
4. Gợi ý cách phòng tránh lỗi tương tự
```

**Ví dụ cụ thể:**
```
Tôi gặp lỗi Flutter sau:

Error message:
"RenderFlex overflowed by 42 pixels on the bottom."
"A RenderFlex overflowed by 42 pixels on the bottom."

Code:
Column(
  children: [
    Image.network(url, height: 300),
    Text(title, style: TextStyle(fontSize: 24)),
    Text(description),
    ElevatedButton(onPressed: onTap, child: Text('Action')),
  ],
)

Context:
- Lỗi xảy ra trên màn hình nhỏ (iPhone SE)
- Ảnh chiếm quá nhiều space
- Đã thử wrap trong Expanded nhưng lỗi khác

Hãy fix và giải thích.
```

---

## 4. Workflow: AI-Driven Development

### 4.1. Quy Trình Chuẩn

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Dùng AI generate boilerplate code                  │
│     ↓                                                       │
│  Step 2: Review code AI generate (QUAN TRỌNG!)              │
│     ↓                                                       │
│  Step 3: Customize theo business logic cụ thể               │
│     ↓                                                       │
│  Step 4: Viết tests (có thể dùng AI hỗ trợ)                │
│     ↓                                                       │
│  Step 5: Run tests, fix issues                              │
│     ↓                                                       │
│  Step 6: Code review bởi người (peer hoặc senior)           │
└─────────────────────────────────────────────────────────────┘
```

### 4.2. Chi Tiết Từng Step

**Step 1: Generate Boilerplate**
- Dùng AI để tạo cấu trúc cơ bản: models, repositories, BLoC/providers, UI skeleton
- Cung cấp context đầy đủ: architecture, existing models, conventions
- Bắt đầu từ domain layer → data layer → presentation layer

**Step 2: Review Code AI Generate (BẮT BUỘC)**

Checklist review:
- [ ] Logic business có đúng không?
- [ ] Có security issues không? (hardcoded secrets, SQL injection, XSS)
- [ ] Có null safety issues không?
- [ ] Error handling có đầy đủ không?
- [ ] Code có tuân theo architecture của project không?
- [ ] Naming conventions có consistent không?
- [ ] Có code thừa/không dùng không?
- [ ] Dependencies có hợp lý không? (không import package không cần thiết)
- [ ] Có performance issues không? (N+1 queries, unnecessary rebuilds)

**Step 3: Customize**
- Thêm business logic cụ thể mà AI không biết
- Điều chỉnh UI theo design mockup
- Integrate với existing code trong project
- Thêm edge cases mà AI có thể bỏ sót

**Step 4: Write Tests**
- Dùng AI generate test skeleton
- Thêm edge cases và real-world scenarios
- Đảm bảo test meaningful, không chỉ test cho có

**Step 5: Run & Fix**
- `flutter test` cho unit + widget tests
- Test thủ công trên emulator/device
- Fix issues phát sinh

**Step 6: Human Code Review**
- Peer review hoặc senior review
- Focus: architecture decisions, edge cases, security
- AI code cần review kỹ hơn bình thường

### 4.3. Workflow Theo Feature

Ví dụ: Implement feature "Task List với filter"

```
1. [AI] Generate TaskModel với freezed
2. [Review] Kiểm tra fields, json keys
3. [AI] Generate TaskRepository interface (domain layer)  
4. [AI] Generate TaskApiService với Retrofit
5. [Review] Kiểm tra endpoints, error handling
6. [AI] Generate TaskRepositoryImpl
7. [AI] Generate GetTasksUseCase
8. [AI] Generate TaskListBloc/Provider
9. [Review] Kiểm tra state transitions, edge cases
10. [AI] Generate TaskListScreen UI skeleton
11. [Customize] Điều chỉnh UI theo Figma design
12. [AI] Generate tests cho Repository, BLoC, Widget
13. [Review + Fix] Chạy tests, fix failures
14. [Human Review] Peer review toàn bộ feature
```

---

## 5. Best Practices: Khi Nào Nên / Không Nên Dùng AI

### 5.1. NÊN Dùng AI ✅

| Use Case | Lý do |
|----------|-------|
| **Boilerplate code** | Models, repositories, API services → lặp đi lặp lại, AI làm nhanh hơn |
| **Test generation** | AI tạo test skeleton tốt, bạn thêm edge cases |
| **Code conversion** | Chuyển JSON thành Dart class, chuyển format |
| **Documentation** | Generate comments, README, API docs |
| **Debug errors** | Paste error message + code → AI phân tích nhanh |
| **Refactoring** | Đổi tên, tách function, extract widget |
| **Learning** | Giải thích concept, so sánh approaches |
| **Regex, algorithms** | AI viết regex, sort, search tốt hơn viết tay |
| **Config files** | pubspec.yaml, build.gradle, CI/CD scripts |

### 5.2. KHÔNG NÊN Dùng AI ❌

| Use Case | Lý do |
|----------|-------|
| **Complex business logic** | AI không hiểu domain cụ thể, dễ sai logic |
| **Security-critical code** | Auth flows, encryption → cần expert review kỹ |
| **Architecture decisions** | AI thiên về solutions phổ biến, không hiểu context project |
| **Performance optimization** | Cần profiling thực tế, AI chỉ gợi ý chung |
| **Copy code sản phẩm** | AI có thể tạo code tương tự copyrighted code |
| **Tin hoàn toàn vào AI** | Luôn review, AI có thể hallucinate API không tồn tại |
| **Thay thế việc hiểu code** | Dùng AI mà không hiểu output = khoản nợ kỹ thuật |

### 5.3. Nguyên Tắc Vàng

1. **AI là trợ lý, không phải thay thế**: Bạn vẫn cần hiểu mọi dòng code
2. **Review 100% AI output**: Không commit code AI mà chưa review
3. **Cung cấp context tốt = output tốt**: Prompt càng chi tiết, code càng đúng
4. **Iterative approach**: Bắt đầu nhỏ, review, mở rộng dần
5. **Verify dependencies**: AI có thể suggest package cũ/không tồn tại
6. **Không paste sensitive data**: API keys, passwords, customer data
7. **Học từ AI output**: Đọc và hiểu code AI generate để nâng cao skill

---

## 6. Prompt Templates Cho Flutter Tasks Phổ Biến

### 6.1. Tạo Data Model

```
Tạo Dart data model cho [tên model] với freezed và json_serializable.

Fields:
- [field1]: [type] - [mô tả]
- [field2]: [type] - [mô tả]
- ...

Yêu cầu:
- Dùng @freezed annotation
- Có factory constructor fromJson/toJson
- Custom json keys nếu server dùng snake_case
- Thêm default values nếu field optional
- Bao gồm copyWith support

Output: file .dart, kèm part directive cho code generation.
```

### 6.2. Tạo Custom Widget

```
Tạo custom Flutter widget: [tên widget].

Mô tả: [widget làm gì, dùng ở đâu]

Props (constructor parameters):
- [param1]: [type] - [required/optional] - [mô tả]
- [param2]: [type] - [required/optional] - [mô tả]

Behavior:
- [Mô tả interaction/animation]

Style:
- [Mô tả visual: colors, spacing, typography]

Constraints:
- Responsive: [yes/no, breakpoints]
- Accessibility: [semantic labels]
- Reusable: [stateless if possible]
```

### 6.3. Tạo GoRouter Configuration

```
Tạo GoRouter configuration cho app có các screens sau:

Routes:
- / → SplashScreen
- /login → LoginScreen
- /home → HomeScreen (cần auth)
- /tasks → TaskListScreen (cần auth)
- /tasks/:id → TaskDetailScreen (cần auth)
- /tasks/create → CreateTaskScreen (cần auth)
- /profile → ProfileScreen (cần auth)

Yêu cầu:
- Redirect to /login nếu chưa auth
- Redirect to /home nếu đã auth mà vào /login
- ShellRoute cho bottom navigation (home, tasks, profile)
- Dùng StatefulShellRoute cho nested navigation
- GoRouterProvider nếu dùng Riverpod
```

### 6.4. Setup Dio Client

```
Tạo Dio client configuration với:

1. Base options: baseUrl, connectTimeout, receiveTimeout
2. Interceptors:
   - AuthInterceptor: thêm Bearer token, auto refresh token khi 401
   - LoggingInterceptor: log request/response trong debug mode
   - ErrorInterceptor: chuyển DioException thành custom AppException
3. Certificate pinning (optional)

Auth flow:
- Token lưu trong secure storage
- Khi nhận 401: gọi refresh token API → retry request gốc
- Nếu refresh cũng fail: logout user

Output: dio_client.dart, auth_interceptor.dart, logging_interceptor.dart
```

### 6.5. Tạo Form Với Validation

```
Tạo Flutter form screen cho [mục đích: login/register/create task/...].

Fields:
- [field1]: [type] - validation: [rules]
- [field2]: [type] - validation: [rules]

Yêu cầu:
- Dùng TextFormField với FormKey
- Validation realtime (onChanged) + submit time
- Show/hide password toggle nếu có password field
- Loading state khi submit
- Error message hiển thị từ server
- Keyboard handling: next focus, done to submit
- Dùng [BLoC/Riverpod] cho form state
```

### 6.6. Database Setup Với Hive

```
Setup Hive local database cho Flutter app.

Models cần lưu:
- [Model1]: [liệt kê fields]
- [Model2]: [liệt kê fields]

Yêu cầu:
- Hive type adapters cho mỗi model
- Box cho mỗi model type
- CRUD operations: create, read, readAll, update, delete
- Initialization code (gọi trong main.dart)
- Repository pattern wrapper
- Handle migration nếu model thay đổi
```

### 6.7. CI/CD Pipeline

```
Tạo GitHub Actions workflow cho Flutter project.

Jobs:
1. analyze: flutter analyze + dart format check
2. test: flutter test với coverage report
3. build-android: build APK (release)
4. build-ios: build IPA (release)

Triggers: push to main, pull request to main
Caching: Flutter SDK, pub cache
Secrets cần: ANDROID_KEYSTORE, IOS_CERTIFICATE (mô tả cách setup)
```

---

## 7. Lỗi Thường Gặp Khi Dùng AI Cho Flutter

### 7.1. AI Suggest Package Không Tồn Tại / Đã Cũ

**Vấn đề:** AI hallucinate tên package hoặc suggest API đã deprecated.

**Giải pháp:**
- Luôn kiểm tra package trên [pub.dev](https://pub.dev) trước khi dùng
- Kiểm tra version compatibility với Flutter SDK hiện tại
- Xem changelog, last updated date

### 7.2. AI Trộn Syntax Các Version Khác Nhau

**Vấn đề:** AI trộn Dart 2 và Dart 3 syntax, hoặc trộn null safety / pre-null safety.

**Giải pháp:**
- Specify Dart/Flutter version trong prompt
- Review null safety annotations kỹ
- Chạy `dart analyze` sau khi paste code

### 7.3. AI Không Hiểu Project Context

**Vấn đề:** AI generate code không match architecture hiện tại.

**Giải pháp:**
- Dùng Copilot instructions file hoặc Cursor rules
- Paste existing code examples trong prompt
- Reference existing files: "Tương tự cách implement trong user_repository.dart"

### 7.4. AI Generate Code Quá Phức Tạp

**Vấn đề:** AI over-engineer solutions đơn giản.

**Giải pháp:**
- Thêm "Keep it simple" trong prompt
- Specify phạm vi: "Chỉ cần basic implementation, không cần advanced features"
- Yêu cầu giải pháp theo YAGNI principle

### 7.5. AI Bỏ Qua Error Handling

**Vấn đề:** AI generate happy path code, bỏ qua error cases.

**Giải pháp:**
- Luôn yêu cầu "handle all error cases"
- Liệt kê error scenarios cụ thể trong prompt
- Review error handling là bước bắt buộc trong checklist

---

## 📚 Tài liệu liên quan

| Tài liệu | Mô tả |
|---|---|
| [README — Tổng quan chương trình](../README.md) | Cài đặt môi trường, lộ trình 16 buổi, hướng dẫn sử dụng |
| [Tiêu chuẩn Middle Developer](../tieu-chuan/middle-level-rubric.md) | Rubric đánh giá năng lực Middle Flutter Developer |
| [Reference Architecture](../project-mau/reference-architecture.md) | Kiến trúc tham chiếu cho dự án Flutter thực tế |
| [Vận hành nhóm học](../van-hanh-nhom/study-group-operations.md) | Quy trình tổ chức buổi học peer-to-peer |

---

*Tài liệu thuộc chương trình Flutter Training. Cập nhật lần cuối: 2026-03-31.*
