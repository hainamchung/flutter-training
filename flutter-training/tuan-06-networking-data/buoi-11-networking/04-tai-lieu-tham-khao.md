# Buổi 11: Networking & API Integration — Tài liệu tham khảo

## 📦 Packages chính

### HTTP Clients

| Package | Mô tả | Link |
|---------|--------|------|
| **dio** | HTTP client mạnh mẽ, interceptors, FormData, cancel token | [pub.dev/packages/dio](https://pub.dev/packages/dio) |
| **http** | HTTP client chính thức từ Dart team | [pub.dev/packages/http](https://pub.dev/packages/http) |
| **retrofit** | Type-safe HTTP client với annotation, code generation | [pub.dev/packages/retrofit](https://pub.dev/packages/retrofit) |
| **retrofit_generator** | Code generator cho retrofit | [pub.dev/packages/retrofit_generator](https://pub.dev/packages/retrofit_generator) |

### JSON Serialization

| Package | Mô tả | Link |
|---------|--------|------|
| **json_annotation** | Annotations cho JSON serialization | [pub.dev/packages/json_annotation](https://pub.dev/packages/json_annotation) |
| **json_serializable** | Code generator cho JSON fromJson/toJson | [pub.dev/packages/json_serializable](https://pub.dev/packages/json_serializable) |
| **freezed** | Immutable models + JSON + copyWith + equality | [pub.dev/packages/freezed](https://pub.dev/packages/freezed) |
| **freezed_annotation** | Annotations cho freezed | [pub.dev/packages/freezed_annotation](https://pub.dev/packages/freezed_annotation) |
| **build_runner** | Build system cho code generation | [pub.dev/packages/build_runner](https://pub.dev/packages/build_runner) |

### Security & Auth

| Package | Mô tả | Link |
|---------|--------|------|
| **flutter_secure_storage** | Lưu trữ data an toàn (Keychain/Keystore) | [pub.dev/packages/flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) |

### Error Handling & Utilities

| Package | Mô tả | Link |
|---------|--------|------|
| **fpdart** | Functional programming: Either, Option, Task | [pub.dev/packages/fpdart](https://pub.dev/packages/fpdart) |
| **connectivity_plus** | Kiểm tra kết nối mạng | [pub.dev/packages/connectivity_plus](https://pub.dev/packages/connectivity_plus) |

---

## 📖 Tài liệu chính thức

### Flutter & Dart

- [Flutter — Fetch data from the internet](https://docs.flutter.dev/cookbook/networking/fetch-data) — Hướng dẫn chính thức
- [Flutter — JSON and serialization](https://docs.flutter.dev/data-and-backend/serialization/json) — So sánh manual vs code generation
- [Dart — dart:convert library](https://api.dart.dev/stable/dart-convert/dart-convert-library.html) — jsonEncode/jsonDecode

### Package Documentation

- [dio — Getting Started](https://pub.dev/packages/dio#getting-started) — Hướng dẫn cơ bản dio
- [dio — Interceptors](https://pub.dev/packages/dio#interceptors) — Tài liệu interceptor chi tiết
- [retrofit — README](https://pub.dev/packages/retrofit) — Annotations và cách sử dụng
- [json_serializable — Guide](https://pub.dev/packages/json_serializable#set-up) — Setup build_runner
- [freezed — README](https://pub.dev/packages/freezed) — Tạo immutable models

---

## 🌐 APIs cho thực hành

| API | URL | Mô tả | Auth |
|-----|-----|--------|------|
| **JSONPlaceholder** | [jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com) | Fake REST API miễn phí. Posts, comments, users, todos, albums, photos | Không |
| **reqres.in** | [reqres.in](https://reqres.in) | Fake REST API có hỗ trợ auth (login/register) | Có (simulated) |
| **MockAPI** | [mockapi.io](https://mockapi.io) | Tự tạo mock API, custom endpoints | Tùy chọn |
| **json-server** | [github.com/typicode/json-server](https://github.com/typicode/json-server) | Local mock API từ JSON file | Không |

### JSONPlaceholder Endpoints phổ biến

```
GET    /posts          — Lấy tất cả posts
GET    /posts/1        — Lấy post theo ID
GET    /posts?userId=1 — Filter posts theo userId
POST   /posts          — Tạo post mới
PUT    /posts/1        — Update post
DELETE /posts/1        — Xóa post
GET    /users          — Lấy tất cả users
GET    /comments?postId=1 — Comments của post
```

### reqres.in Endpoints

```
POST   /api/login      — { email, password } → { token }
POST   /api/register   — { email, password } → { id, token }
GET    /api/users       — Lấy users (paginated)
GET    /api/users/2     — Lấy single user
GET    /api/users?page=2 — Pagination
```

---

## 📝 Blog & Articles

### Networking trong Flutter

- [Very Good Engineering — Networking in Flutter](https://verygood.ventures/blog) — Best practices từ Very Good Ventures team
- [ResoCoder — Flutter TDD Clean Architecture](https://resocoder.com/flutter-clean-architecture-tdd/) — Series Clean Architecture bao gồm cả networking layer
- [FilledStacks — Network Setup](https://www.filledstacks.com/) — Cách setup network layer production-grade

### JSON Serialization

- [Flutter docs — JSON Serialization](https://docs.flutter.dev/data-and-backend/serialization/json) — So sánh approaches: manual, json_serializable, freezed
- [Code With Andrea — Dart JSON Serialization](https://codewithandrea.com/articles/parse-json-dart/) — Hướng dẫn chi tiết từng cách parse JSON

### Error Handling

- [ResoCoder — Either type for Error Handling](https://resocoder.com/2019/12/14/flutter-tdd-clean-architecture-course-14-either-type/) — Cách dùng Either (bài viết dùng dartz, API tương tự fpdart)
- [Code With Andrea — Error Handling](https://codewithandrea.com/articles/flutter-exception-handling-try-catch-result-type/) — Result type pattern

---

## 🎥 Video

- [Reso Coder — Flutter TDD Clean Architecture (Full Playlist)](https://www.youtube.com/playlist?list=PLB6lc7nQ1n4iYGE_khpXRdJkJEp9WOech) — Bao gồm networking, repository pattern, error handling
- [Flutter Official — Networking](https://www.youtube.com/watch?v=iEv_6GJJAx4) — Fetch data tutorial từ Flutter team
- [Code With Andrea — REST API Integration](https://www.youtube.com/c/CodeWithAndrea) — Series về API integration patterns

---

## 🛠️ pubspec.yaml Mẫu

```yaml
# Copy vào project để bắt đầu nhanh
dependencies:
  flutter:
    sdk: flutter

  # HTTP Client
  dio: ^5.4.3+1

  # Type-safe API client
  retrofit: ^4.1.0

  # JSON Serialization
  json_annotation: ^4.8.1

  # Immutable models (optional nhưng khuyến khích)
  freezed_annotation: ^2.4.0

  # Functional programming (Either) — fpdart is the actively maintained successor to dartz
  fpdart: ^1.1.0

  # Secure token storage
  flutter_secure_storage: ^9.0.0

  # Network connectivity check
  connectivity_plus: ^6.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Code generators
  build_runner: ^2.4.0
  json_serializable: ^6.7.0
  retrofit_generator: ^8.1.0
  freezed: ^2.4.0
```

---

## 🔑 Lệnh thường dùng

```bash
# Chạy code generation (một lần)
dart run build_runner build --delete-conflicting-outputs

# Chạy code generation (watch mode — auto rebuild khi file thay đổi)
dart run build_runner watch --delete-conflicting-outputs

# Chỉ generate cho 1 file cụ thể (nhanh hơn)
dart run build_runner build --build-filter="lib/models/user.dart"

# Clean generated files
dart run build_runner clean

# Chạy json-server (mock API)
npx json-server --watch db.json --port 3000
```

---

## 🤖 AI Prompt Library — Buổi 11: Networking

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Networking trong Flutter (Dio). Background: 4+ năm React (Axios, fetch, interceptors).
Câu hỏi: Dio interceptors giống Axios interceptors không? Retrofit annotation giống gì trong TS/React? Token refresh pattern có gì khác giữa mobile và web?
Yêu cầu: mapping 1-1 với Axios/fetch concepts, highlight mobile-specific concerns (token secure storage, certificate pinning).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần setup Retrofit API service cho Flutter.
Tech stack: dio ^5.x, retrofit ^4.x, json_serializable.
Endpoints: GET /posts, GET /posts/{id}, POST /posts, PUT /posts/{id}, DELETE /posts/{id}.
Model: PostModel (id, title, body, userId) với fromJson/toJson.
Output: api_service.dart (abstract class, annotations) + post_model.dart.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Dio networking code sau:
[paste code]

Kiểm tra:
1. Interceptor order đúng? (Auth trước Error?)
2. Token refresh handle race condition?
3. Error mapping exhaustive? (all DioExceptionType covered?)
4. Timeout config hợp lý? (connect, receive, send)
5. Retry logic chỉ cho idempotent methods?
6. Security: token storage dùng SecureStorage?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi networking trong Flutter (Dio):
[paste error + stack trace]

Dio config:
[paste dio setup]

Cần: (1) Nguyên nhân, (2) Fix, (3) Best practice cho trường hợp này.
```

---

## 🔗 Liên kết nội bộ

| Buổi | Chủ đề | File |
|------|--------|------|
| Buổi 9 | Clean Architecture — Repository, DataSource pattern | `tuan-05-architecture-di/buoi-09-clean-architecture/` |
| Buổi 10 | DI & Testing — Injectable, GetIt | `tuan-05-architecture-di/buoi-10-di-testing/` |
| **Buổi 11** | **Networking & API Integration (hiện tại)** | **`tuan-06-networking-data/buoi-11-networking/`** |
| Buổi 12 | Local Storage & Persistence (tiếp theo) | `tuan-06-networking-data/buoi-12-local-storage/` |
