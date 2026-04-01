# Buổi 11: Networking & API Integration — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới networking — FE developer sẽ thấy **concepts quen thuộc** nhưng **tooling khác**.
> Đọc bảng dưới TRƯỚC khi code để tránh nhầm lẫn.

| FE Networking Habit | Flutter Reality | Bài tập liên quan |
|---------------------|-----------------|---------------------|
| `axios.get()` / `fetch()` trả về JSON tự động | `Dio` response cần `.data` + **parse thủ công** qua `fromJson()` | BT1, BT2 |
| Zod schema validate runtime | `json_serializable` / `freezed` generate code → chạy `build_runner` trước | BT1, BT2 |
| `axios.interceptors.use()` | `Dio interceptors..add()` — cùng concept, khác API | BT2, BT3 |

---

## BT1 ⭐: Fetch & Display Posts từ JSONPlaceholder 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_fetch_posts` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — danh sách posts từ JSONPlaceholder API |
| **Dependencies** | `flutter pub add dio` |

### Yêu cầu

Xây dựng Flutter app hiển thị danh sách posts từ JSONPlaceholder API.

**Chức năng:**
1. Fetch danh sách posts từ `https://jsonplaceholder.typicode.com/posts`
2. Hiển thị trong `ListView` với title và body (truncated)
3. Tap vào post → hiển thị chi tiết (full body + userId)
4. Pull-to-refresh để reload
5. Loading indicator khi đang fetch
6. Error state với nút Retry

**Yêu cầu kỹ thuật:**
- Dùng `dio` package (không dùng `http`)
- Tạo `Post` model với `fromJson` (viết tay hoặc `json_serializable`)
- Tạo `PostService` class tách biệt (không gọi API trực tiếp trong widget)

### Hướng dẫn từng bước

**Bước 1:** Tạo project và thêm dependency

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  dio: ^5.4.3+1
```

**Bước 2:** Tạo model

```dart
// lib/models/post.dart
class Post {
  final int id;
  final int userId;
  final String title;
  final String body;

  const Post({
    required this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}
```

**Bước 3:** Tạo service

```dart
// lib/services/post_service.dart
import 'package:dio/dio.dart';
import '../models/post.dart';

class PostService {
  final Dio _dio;

  PostService()
      : _dio = Dio(BaseOptions(
          baseUrl: 'https://jsonplaceholder.typicode.com',
          connectTimeout: const Duration(seconds: 10),
          receiveTimeout: const Duration(seconds: 10),
        ));

  Future<List<Post>> getPosts() async {
    final response = await _dio.get('/posts');
    final list = response.data as List<dynamic>;
    return list
        .map((json) => Post.fromJson(json as Map<String, dynamic>))
        .toList();
  }

  Future<Post> getPostById(int id) async {
    final response = await _dio.get('/posts/$id');
    return Post.fromJson(response.data as Map<String, dynamic>);
  }
}
```

**Bước 4:** Tạo UI — tự hoàn thành phần `PostListScreen` và `PostDetailScreen`

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

- Dùng `FutureBuilder` hoặc quản lý state trong `StatefulWidget`
- `RefreshIndicator` cho pull-to-refresh
- `ListTile` với `maxLines: 2` cho body truncated
- Navigator.push cho detail screen

</details>

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| Fetch data thành công | 2đ |
| Model đúng cấu trúc | 1đ |
| Service tách biệt khỏi UI | 1đ |
| Loading state | 1đ |
| Error state + Retry | 2đ |
| Pull-to-refresh | 1đ |
| Detail screen | 1đ |
| Code clean, đặt tên rõ ràng | 1đ |
| **Tổng** | **10đ** |

---

## BT2 ⭐⭐: Interceptors & Error Handling 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_interceptors` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — posts app với logging, error handling chuyên nghiệp |
| **Dependencies** | `flutter pub add dio fpdart` |

### Yêu cầu

Mở rộng BT1, thêm interceptor system và error handling chuyên nghiệp.

**Chức năng mới:**
1. **Logging Interceptor** — Log mọi request/response ra console
2. **Error Handling Interceptor** — Map `DioException` sang custom `Failure` class
3. **Hiển thị error messages** thân thiện trong UI (không hiện technical error)
4. **Timeout handling** — Thông báo rõ ràng khi timeout

**Yêu cầu kỹ thuật:**

### Bước 1: Tạo Failure classes

```dart
// lib/core/error/failures.dart
abstract class Failure {
  final String message;
  final String? technicalMessage; // Cho logging, không hiện UI

  const Failure(this.message, {this.technicalMessage});

  @override
  String toString() => 'Failure: $message';
}

class ServerFailure extends Failure {
  final int? statusCode;
  const ServerFailure(
    super.message, {
    this.statusCode,
    super.technicalMessage,
  });
}

class NetworkFailure extends Failure {
  const NetworkFailure(super.message, {super.technicalMessage});
}

class TimeoutFailure extends Failure {
  const TimeoutFailure(super.message, {super.technicalMessage});
}
```

### Bước 2: Tạo Logging Interceptor

```dart
// lib/core/network/logging_interceptor.dart
// Tham khảo VD2 trong 02-vi-du.md
// Tự implement — log method, URL, headers (ẩn auth), body, status code
```

### Bước 3: Tạo Error Mapping Interceptor

```dart
// lib/core/network/error_interceptor.dart
class ErrorMappingInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // TODO: Map DioException → Failure
    // Gợi ý: tạo extension method hoặc static mapper
    handler.next(err);
  }
}
```

### Bước 4: Cập nhật Service để trả Either

```dart
// lib/services/post_service.dart
import 'package:fpdart/fpdart.dart';

class PostService {
  // ...

  Future<Either<Failure, List<Post>>> getPosts() async {
    try {
      final response = await _dio.get('/posts');
      // ... parse và return Right
    } on DioException catch (e) {
      return Left(/* map error */);
    }
  }
}
```

### Bước 5: Cập nhật UI

Xử lý `Either` result trong UI:

```dart
final result = await postService.getPosts();
result.fold(
  (failure) {
    // Hiển thị failure.message (user-friendly)
    // Log failure.technicalMessage (debug)
  },
  (posts) {
    // Hiển thị danh sách posts
  },
);
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

1. Tạo Failure classes phân loại lỗi (ServerFailure, NetworkFailure, TimeoutFailure)
2. Implement LoggingInterceptor — log method, URL, status code cho mỗi request/response
3. Implement ErrorMappingInterceptor — map `DioException` type sang Failure tương ứng
4. Cập nhật PostService trả về `Either<Failure, T>` thay vì throw exception
5. Cập nhật UI xử lý `Either` bằng `fold()` — hiển thị error message thân thiện

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// error_mapping_interceptor.dart
class ErrorMappingInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    final failure = switch (err.type) {
      DioExceptionType.connectionTimeout ||
      DioExceptionType.sendTimeout ||
      DioExceptionType.receiveTimeout =>
        TimeoutFailure('Kết nối quá chậm'),
      DioExceptionType.connectionError =>
        NetworkFailure('Không có kết nối mạng'),
      _ => ServerFailure(
        'Lỗi server',
        statusCode: err.response?.statusCode,
      ),
    };
    // TODO: attach failure to error and pass to handler
    handler.next(err);
  }
}
```

</details>

### Test scenarios

Bạn cần test các scenario sau:

| Scenario | Cách test | Expected |
|----------|-----------|----------|
| Success | Gọi API bình thường | Hiển thị danh sách posts |
| Network error | Tắt WiFi/Airplane mode | "Không có kết nối mạng" |
| Timeout | Set timeout = 1ms | "Kết nối quá chậm" |
| Server error | Gọi endpoint không tồn tại | "Lỗi server (404)" |
| Cancel | Navigate away while loading | Không crash |

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| Logging Interceptor hoạt động đúng | 2đ |
| Failure classes đầy đủ | 1đ |
| Error mapping chính xác | 2đ |
| UI hiển thị error thân thiện | 2đ |
| Dùng Either pattern | 2đ |
| Handle tất cả test scenarios | 1đ |
| **Tổng** | **10đ** |

---

## BT3 ⭐⭐⭐: Full API Layer — Retrofit + Auth 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_retrofit_auth` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Notes app CRUD với Retrofit, auth interceptor |
| **Dependencies** | `flutter pub add dio retrofit json_annotation fpdart` + `flutter pub add dev:json_serializable dev:build_runner dev:retrofit_generator` |

### Yêu cầu

Xây dựng API layer hoàn chỉnh cho ứng dụng Notes theo Clean Architecture.

**Chức năng:**
1. **CRUD Notes** — Tạo, đọc, sửa, xóa notes qua API
2. **Retrofit client** — Type-safe API definitions
3. **Auth interceptor** — Inject Bearer token vào mọi request
4. **Token refresh** — Khi nhận 401, tự refresh token và retry
5. **Error handling** — `Either<Failure, T>` xuyên suốt

**Kiến trúc:**

```
┌─────────────────────────┐
│    Presentation Layer    │
│    (NotesCubit/BLoC)     │
└────────────┬────────────┘
             │  Either<Failure, T>
┌────────────▼────────────┐
│      Domain Layer        │
│  UseCase + Repository    │
│  (interface)             │
└────────────┬────────────┘
             │
┌────────────▼────────────┐
│       Data Layer         │
│  RepositoryImpl          │
│  RemoteDataSource        │
│  (Retrofit + Dio)        │
└────────────┬────────────┘
             │
┌────────────▼────────────┐
│    Network Layer         │
│  Dio + Interceptors      │
│  (Auth, Log, Error)      │
└─────────────────────────┘
```

### Bước 1: Models

```dart
// lib/features/notes/data/models/note_model.dart
import 'package:json_annotation/json_annotation.dart';

part 'note_model.g.dart';

@JsonSerializable()
class NoteModel {
  final int id;
  final String title;
  final String content;

  @JsonKey(name: 'user_id')
  final int userId;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  @JsonKey(name: 'updated_at')
  final DateTime? updatedAt;

  const NoteModel({
    required this.id,
    required this.title,
    required this.content,
    required this.userId,
    required this.createdAt,
    this.updatedAt,
  });

  factory NoteModel.fromJson(Map<String, dynamic> json) =>
      _$NoteModelFromJson(json);
  Map<String, dynamic> toJson() => _$NoteModelToJson(this);
}
```

### Bước 2: Retrofit Client

```dart
// lib/features/notes/data/datasources/note_api_client.dart
import 'package:retrofit/retrofit.dart';
import 'package:dio/dio.dart';
import '../models/note_model.dart';

part 'note_api_client.g.dart';

@RestApi()
abstract class NoteApiClient {
  factory NoteApiClient(Dio dio) = _NoteApiClient;

  @GET('/notes')
  Future<List<NoteModel>> getNotes();

  @GET('/notes/{id}')
  Future<NoteModel> getNoteById(@Path('id') int id);

  @POST('/notes')
  Future<NoteModel> createNote(@Body() Map<String, dynamic> body);

  @PUT('/notes/{id}')
  Future<NoteModel> updateNote(
    @Path('id') int id,
    @Body() Map<String, dynamic> body,
  );

  @DELETE('/notes/{id}')
  Future<void> deleteNote(@Path('id') int id);
}
```

### Bước 3: Auth Token Interceptor

Implement `TokenRefreshInterceptor` với flow:
1. Inject `Bearer` token vào mọi request
2. Khi nhận 401 → gọi refresh endpoint
3. Refresh thành công → lưu token mới + retry request gốc
4. Refresh thất bại → clear tokens, emit logout event

> **Gợi ý:** Dùng `QueuedInterceptor` và tham khảo VD trong `01-ly-thuyet.md` phần 5.3

### Bước 4: Repository

```dart
// Implement NoteRepositoryImpl theo pattern trong VD5 (02-vi-du.md)
// Mỗi method: try-catch → Either<Failure, T>
```

### Bước 5: Mock API

Chọn **một** trong các cách sau để có API test:

**Option A: JSONPlaceholder** (đơn giản nhất)
- Dùng `/posts` thay cho `/notes`
- Không có auth, nhưng mock token interceptor

**Option B: json-server** (khuyến khích)
```bash
# Cài json-server
npm install -g json-server

# Tạo db.json
{
  "notes": [
    {"id": 1, "title": "Note 1", "content": "Content 1", "user_id": 1, "created_at": "2026-01-01"},
    {"id": 2, "title": "Note 2", "content": "Content 2", "user_id": 1, "created_at": "2026-01-02"}
  ]
}

# Chạy server
json-server --watch db.json --port 3000

# API tự động có: GET/POST/PUT/DELETE /notes
```

**Option C: reqres.in** (có auth)
- Dùng `/api/login` để lấy token
- Dùng `/api/users` thay cho notes

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

1. Tạo `NoteModel` với `@JsonSerializable()` — define fields và `fromJson`/`toJson`
2. Define Retrofit client với `@RestApi()` — annotation cho GET, POST, PUT, DELETE endpoints
3. Implement `TokenRefreshInterceptor` extends `QueuedInterceptor` — inject Bearer token, handle 401 refresh + retry
4. Implement `NoteRepositoryImpl` — wrap API calls trong try-catch, trả về `Either<Failure, T>`
5. Setup mock API (json-server hoặc JSONPlaceholder) và tạo simple UI hiển thị notes

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// token_refresh_interceptor.dart
class TokenRefreshInterceptor extends QueuedInterceptor {
  final Dio _dio;
  String? _accessToken;
  String? _refreshToken;

  TokenRefreshInterceptor(this._dio);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (_accessToken != null) {
      options.headers['Authorization'] = 'Bearer $_accessToken';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // TODO: refresh token and retry request
    }
    handler.next(err);
  }
}
```

</details>

### Deliverables

1. **Retrofit API client** hoàn chỉnh (CRUD endpoints)
2. **RemoteDataSource** wrapper (hoặc dùng Retrofit trực tiếp)
3. **Repository implementation** với `Either` error handling
4. **Auth Interceptor** inject token + refresh flow
5. **Logging Interceptor**
6. **Dio configuration** module (injectable)
7. **Ít nhất 1 UseCase** hoạt động end-to-end
8. **Simple UI** hiển thị notes (ListView + error/loading states)

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| Retrofit client đúng annotation | 1.5đ |
| Models với json_serializable | 1đ |
| Repository impl với Either | 2đ |
| Auth interceptor (inject token) | 1.5đ |
| Token refresh flow | 1.5đ |
| Logging interceptor | 0.5đ |
| Error mapping đầy đủ | 1đ |
| UI hiển thị được data | 0.5đ |
| Code organization (Clean Arch) | 0.5đ |
| **Tổng** | **10đ** |

---

## 💬 Câu hỏi thảo luận

### Câu 1: Khi nào chọn `http` vs `dio`?

Bạn đang bắt đầu một project mới. Hãy đưa ra **tiêu chí** cụ thể để quyết định dùng `http` hay `dio`. Xem xét các yếu tố:

- Quy mô project (MVP vs long-term)
- Số lượng API endpoints
- Cần auth management không?
- Cần file upload không?
- Team size và experience

Liệu có case nào bắt đầu với `http` rồi migrate sang `dio` được không? Trade-offs?

### Câu 2: REST vs GraphQL trong Flutter

So sánh việc sử dụng REST API (dio/retrofit) vs GraphQL (`graphql_flutter`) trong Flutter app:

- Khi nào REST phù hợp hơn GraphQL?
- Khi nào GraphQL mang lại lợi ích rõ ràng?
- Với team từ React/Vue có kinh nghiệm GraphQL, có nên dùng GraphQL cho Flutter?
- Hạn chế của GraphQL trong Flutter so với JavaScript ecosystem?

### Câu 3: Offline-first Strategy

Khi app cần hoạt động offline:

1. Luồng data sẽ thay đổi thế nào? (Remote → Local cache → UI)
2. Xử lý conflict khi sync thế nào?
3. Repository pattern cần thay đổi gì? (Hint: RemoteDataSource + LocalDataSource)
4. Có pattern/package nào hỗ trợ? (Hint: `hive`, `isar`, `drift`)

Vẽ architecture diagram cho offline-first note-taking app.

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 6:** Focus vào gen complete service scaffolds và review error/security handling.

### AI-BT1: Gen Dio Service hoàn chỉnh (Interceptor + 401 Refresh + Error Mapping) ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Dio, interceptors, JSON serialization, Retrofit, auth token management, error handling.
- **Task thực tế:** Backend team vừa hoàn thành API — cần setup Dio client hoàn chỉnh với auth interceptor (401 refresh), error mapping, logging, retry logic. AI gen scaffold, bạn review race condition + security.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần Dio HTTP client hoàn chỉnh cho Flutter production app.
Tech stack: Flutter 3.x, dio ^5.x, flutter_secure_storage ^9.x, dartz.
API base: https://api.example.com/v1

Gen các components:
1. DioClient: BaseOptions (30s timeout), interceptors order: Auth → Log → Error.
2. AuthInterceptor: thêm 'Authorization: Bearer $token' từ SecureStorage.
3. TokenRefreshInterceptor:
   - Khi nhận 401 → call /auth/refresh với refresh_token.
   - Race condition: nếu 2+ requests cùng 401, chỉ refresh 1 lần, queue others.
   - Refresh fail → clear tokens → navigate to Login.
4. ErrorInterceptor: DioException → Failure sealed class mapping.
5. RetryInterceptor: retry 3x (exponential backoff 1s/2s/4s) cho GET timeout/network errors.

Constraints:
- Token storage: flutter_secure_storage (KHÔNG SharedPreferences).
- Retry chỉ cho GET (idempotent), KHÔNG retry POST/PUT/DELETE.
- Error messages: tiếng Việt cho user, English cho developer logs.
- Race condition handling bắt buộc trong token refresh.
Output: 5 files (dio_client, auth_interceptor, token_refresh_interceptor, error_interceptor, retry_interceptor).
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 5 interceptor/client files.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Token refresh có lock/mutex khi multiple 401s? (race condition) | ☐ |
| 2 | Retry chỉ cho GET? POST/PUT/DELETE bị skip? | ☐ |
| 3 | Token lưu SecureStorage (không phải SharedPreferences)? | ☐ |
| 4 | Interceptors order đúng: Auth → Log → Error? | ☐ |
| 5 | 401 refresh fail → clear tokens + redirect Login? | ☐ |
| 6 | Exponential backoff: 1s → 2s → 4s (không fixed delay)? | ☐ |
| 7 | Error mapping: DioException → sealed Failure class? | ☐ |

**4. Customize:**
Thêm `CancelToken` support: khi user navigate away → cancel pending requests. Thêm `If-Modified-Since` header cho caching. AI thường không gen — tự thêm CancelToken pool manager.
