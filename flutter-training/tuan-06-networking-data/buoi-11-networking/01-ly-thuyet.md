# Buổi 11: Networking & API Integration — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 11/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 09 (lý thuyết + ít nhất BT1-BT2)

## 1. http vs dio — So sánh & Lựa chọn 🟡

### 1.1 Package `http` — Từ Dart Team

Package `http` là thư viện HTTP chính thức từ Dart team. Đơn giản, nhẹ, phù hợp dự án nhỏ.

```dart
// pubspec.yaml
dependencies:
  http: ^1.2.0
```

```dart
import 'package:http/http.dart' as http;

// GET request
final response = await http.get(
  Uri.parse('https://jsonplaceholder.typicode.com/posts'),
  headers: {'Authorization': 'Bearer $token'},
);

if (response.statusCode == 200) {
  final data = jsonDecode(response.body);
} else {
  throw Exception('Failed to load posts');
}

// POST request
final response = await http.post(
  Uri.parse('https://jsonplaceholder.typicode.com/posts'),
  headers: {'Content-Type': 'application/json'},
  body: jsonEncode({'title': 'Hello', 'body': 'World'}),
);
```

**Ưu điểm:**
- Chính thức từ Dart team → ổn định, ít dependency
- API đơn giản, dễ học
- Nhẹ, ít overhead

**Nhược điểm:**
- Không hỗ trợ interceptor
- Không có cancel token
- Không hỗ trợ FormData natively
- Không có built-in retry, timeout config linh hoạt

### 1.2 Package `dio` — Powerful HTTP Client

`dio` là HTTP client mạnh mẽ nhất trong hệ sinh thái Flutter. Gần như là **tiêu chuẩn** cho production app.

```dart
// pubspec.yaml
dependencies:
  dio: ^5.4.3+1
```

```dart
import 'package:dio/dio.dart';

final dio = Dio(BaseOptions(
  baseUrl: 'https://jsonplaceholder.typicode.com',
  connectTimeout: const Duration(seconds: 10),
  receiveTimeout: const Duration(seconds: 10),
  headers: {'Content-Type': 'application/json'},
));

// GET request
final response = await dio.get('/posts');
final data = response.data; // Đã tự parse JSON!

// POST request
final response = await dio.post('/posts', data: {
  'title': 'Hello',
  'body': 'World',
});

// FormData (file upload)
final formData = FormData.fromMap({
  'name': 'dio',
  'file': await MultipartFile.fromFile('./example.txt'),
});
final response = await dio.post('/upload', data: formData);

// Cancel token
final cancelToken = CancelToken();
dio.get('/posts', cancelToken: cancelToken);
// Cancel request bất cứ lúc nào
cancelToken.cancel('User cancelled');
```

**Ưu điểm:**
- Interceptors (giống middleware)
- Cancel token — hủy request đang chạy
- FormData + file upload
- Auto JSON decode
- Global configuration (base URL, timeout, headers)
- Download progress tracking

> 🔗 **FE Bridge:** `Dio` ≈ **axios** — mapping gần 1:1: `dio.get()` ≈ `axios.get()`, `Interceptor` ≈ `axios interceptor`, `BaseOptions` ≈ `axios.create({ baseURL })`. FE dev sẽ thấy rất quen thuộc khi dùng Dio.

### 1.3 So sánh http vs dio

| Tính năng | `http` | `dio` |
|-----------|--------|-------|
| Maintainer | Dart team | Community (flutterchina) |
| Interceptors | ❌ | ✅ |
| Cancel token | ❌ | ✅ |
| FormData/Upload | Cần thêm package | ✅ Built-in |
| Auto JSON parse | ❌ (manual) | ✅ |
| Global config | Hạn chế | ✅ BaseOptions |
| Retry logic | ❌ | ✅ (via interceptor) |
| Download progress | ❌ | ✅ |
| Bundle size | Nhỏ hơn | Lớn hơn |

> **Khi nào dùng gì?**
> - `http`: Prototype nhanh, app đơn giản, ít API call, muốn ít dependency
> - `dio`: Production app, cần interceptor, auth management, file upload

> 🔗 **FE Bridge:** HTTP concepts **giống hệt** FE — GET/POST/PUT/DELETE, status codes, headers. Nhưng **khác ở**: Flutter dùng `http` package hoặc `dio` thay vì `fetch`/`axios`. `Dio` ≈ `axios` (interceptors, base URL, timeout config). Package `http` ≈ `fetch` API (low-level hơn).

### 1.4 💡 Góc nhìn từ Frontend: Migration từ axios/fetch

Nếu bạn đến từ React/Vue, `dio` chính là **axios của Flutter**:

```
JavaScript (axios)              Dart (dio)
──────────────                  ──────────
axios.create({                  Dio(BaseOptions(
  baseURL: '...',                 baseUrl: '...',
  timeout: 5000,                  connectTimeout: Duration(seconds: 5),
})                              ))

axios.get('/posts')             dio.get('/posts')
axios.post('/posts', data)      dio.post('/posts', data: data)

axios.interceptors.request      dio.interceptors.add(
  .use(config => ...)              InterceptorsWrapper(onRequest: ...)
                                )
```

Còn `http` giống như native `fetch` — đơn giản nhưng cần tự xử lý mọi thứ.

> 💼 **Gặp trong dự án:** API versioning là vấn đề thường gặp — backend update API v2 nhưng app cũ vẫn gọi v1. Dùng Dio interceptor để thêm `Api-Version` header và handle backward compatibility. Khi customer support báo lỗi, check version header trong log để debug nhanh hơn.

---

## 2. Interceptors — Middleware cho HTTP 🔴

### 2.1 Concept

Interceptor là **middleware** xử lý request/response trước khi chúng được gửi đi hoặc nhận về. Giống hệt concept middleware trong Express.js hoặc axios interceptors.

```
Request Flow:
App ──▶ [Interceptor 1] ──▶ [Interceptor 2] ──▶ Server
                                                    │
App ◀── [Interceptor 1] ◀── [Interceptor 2] ◀──────┘
Response Flow
```

### 2.2 Dio Interceptors: 3 Callbacks

```dart
dio.interceptors.add(
  InterceptorsWrapper(
    // Được gọi TRƯỚC khi request được gửi
    onRequest: (options, handler) {
      // Thêm headers, modify request, log...
      handler.next(options); // Tiếp tục
    },

    // Được gọi KHI nhận response thành công
    onResponse: (response, handler) {
      // Transform data, log response...
      handler.next(response); // Tiếp tục
    },

    // Được gọi KHI có lỗi
    onError: (error, handler) {
      // Handle error, retry, log...
      handler.next(error); // Truyền lỗi tiếp
      // hoặc handler.resolve(response) để "sửa" lỗi
    },
  ),
);
```

### 2.3 Logging Interceptor

```dart
class LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    log('──▶ ${options.method} ${options.uri}');
    log('Headers: ${options.headers}');
    if (options.data != null) {
      log('Body: ${options.data}');
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    log('◀── ${response.statusCode} ${response.requestOptions.uri}');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    log('✖ ERROR [${err.response?.statusCode}] ${err.requestOptions.uri}');
    log('Message: ${err.message}');
    handler.next(err);
  }
}

// Sử dụng
dio.interceptors.add(LoggingInterceptor());
```

### 2.4 Auth Token Interceptor

```dart
class AuthInterceptor extends Interceptor {
  final TokenStorage _tokenStorage;

  AuthInterceptor(this._tokenStorage);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await _tokenStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
}
```

### 2.5 Retry Interceptor

```dart
class RetryInterceptor extends Interceptor {
  final Dio _dio;
  final int maxRetries;

  RetryInterceptor(this._dio, {this.maxRetries = 3});

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (_shouldRetry(err) && err.requestOptions.extra['retryCount'] == null) {
      int retryCount = 0;
      while (retryCount < maxRetries) {
        retryCount++;
        try {
          log('Retry attempt $retryCount/${maxRetries}');
          final options = err.requestOptions;
          options.extra['retryCount'] = retryCount;
          final response = await _dio.fetch(options);
          return handler.resolve(response);
        } catch (e) {
          if (retryCount == maxRetries) {
            return handler.next(err);
          }
        }
        await Future.delayed(Duration(seconds: retryCount)); // Exponential backoff
      }
    }
    handler.next(err);
  }

  bool _shouldRetry(DioException err) {
    return err.type == DioExceptionType.connectionTimeout ||
        err.type == DioExceptionType.connectionError ||
        (err.response?.statusCode ?? 0) >= 500;
  }
}
```

---

## 3. JSON Serialization 🟡

### 3.1 Manual: fromJson / toJson

Cách cơ bản nhất — viết tay factory constructor:

```dart
class User {
  final int id;
  final String name;
  final String email;

  const User({
    required this.id,
    required this.name,
    required this.email,
  });

  // Factory constructor để parse JSON
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
    );
  }

  // Chuyển object thành JSON
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}

// Sử dụng
final json = {'id': 1, 'name': 'John', 'email': 'john@test.com'};
final user = User.fromJson(json);
final map = user.toJson();
```

**Nhược điểm manual:**
- Dễ sai khi model phức tạp
- Phải viết lại khi thêm/sửa field
- Không type-safe tuyệt đối (cast thủ công)

### 3.2 json_serializable — Code Generation

Dùng annotation + `build_runner` để tự sinh code:

```yaml
# pubspec.yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart'; // File generated

@JsonSerializable()
class User {
  final int id;
  final String name;
  final String email;

  @JsonKey(name: 'created_at') // Map snake_case → camelCase
  final DateTime? createdAt;

  @JsonKey(defaultValue: 'user')
  final String role;

  const User({
    required this.id,
    required this.name,
    required this.email,
    this.createdAt,
    this.role = 'user',
  });

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

Chạy code generation:

```bash
# Chạy một lần
dart run build_runner build

# Chạy và watch thay đổi (development)
dart run build_runner watch --delete-conflicting-outputs
```

### 3.3 Nested Objects & Lists

```dart
@JsonSerializable(explicitToJson: true) // Quan trọng cho nested!
class Post {
  final int id;
  final String title;
  final User author; // Nested object
  final List<Comment> comments; // List of objects

  @JsonKey(fromJson: _tagsFromJson, toJson: _tagsToJson)
  final List<String> tags;

  const Post({
    required this.id,
    required this.title,
    required this.author,
    required this.comments,
    required this.tags,
  });

  factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);
  Map<String, dynamic> toJson() => _$PostToJson(this);

  // Custom converter cho field đặc biệt
  static List<String> _tagsFromJson(dynamic json) {
    if (json is String) return json.split(',');
    if (json is List) return json.cast<String>();
    return [];
  }

  static dynamic _tagsToJson(List<String> tags) => tags;
}
```

### 3.4 Null Safety trong JSON

```dart
@JsonSerializable()
class Profile {
  final int id;
  final String name;
  final String? bio;           // nullable field
  final String? avatarUrl;

  @JsonKey(defaultValue: 0)    // default nếu null/missing
  final int postCount;

  @JsonKey(includeIfNull: false) // Không include trong toJson nếu null
  final String? phone;

  const Profile({
    required this.id,
    required this.name,
    this.bio,
    this.avatarUrl,
    this.postCount = 0,
    this.phone,
  });

  factory Profile.fromJson(Map<String, dynamic> json) =>
      _$ProfileFromJson(json);
  Map<String, dynamic> toJson() => _$ProfileToJson(this);
}
```

### 3.5 freezed — Immutable Models + JSON

`freezed` tạo immutable data class với `copyWith`, `==`, `toString` miễn phí:

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^2.4.0
  json_annotation: ^4.8.1

dev_dependencies:
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    required String email,
    @Default('user') String role,
    DateTime? createdAt,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Sử dụng
final user = User(id: 1, name: 'John', email: 'john@test.com');
final updated = user.copyWith(name: 'Jane'); // Immutable update
print(user == updated); // false — value equality tự động
```

### 3.6 TypeAdapter Pattern

Khi cần custom conversion cho types đặc biệt:

```dart
class DateTimeConverter implements JsonConverter<DateTime, String> {
  const DateTimeConverter();

  @override
  DateTime fromJson(String json) => DateTime.parse(json);

  @override
  String toJson(DateTime object) => object.toIso8601String();
}

class TimestampConverter implements JsonConverter<DateTime, int> {
  const TimestampConverter();

  @override
  DateTime fromJson(int json) =>
      DateTime.fromMillisecondsSinceEpoch(json * 1000);

  @override
  int toJson(DateTime object) =>
      object.millisecondsSinceEpoch ~/ 1000;
}

// Sử dụng
@JsonSerializable()
class Event {
  final int id;
  final String title;

  @DateTimeConverter()
  final DateTime startDate;

  @TimestampConverter()
  final DateTime createdAt;

  const Event({
    required this.id,
    required this.title,
    required this.startDate,
    required this.createdAt,
  });

  factory Event.fromJson(Map<String, dynamic> json) => _$EventFromJson(json);
  Map<String, dynamic> toJson() => _$EventToJson(this);
}
```

> 🔗 **FE Bridge:** JSON serialization ≈ Zod validation + type parsing trong TS. Nhưng **khác ở**: Dart cần **code generation** (`json_serializable`, `freezed`) để parse JSON → class. TS có runtime type checking với Zod, Dart approach = compile-time generated `fromJson`/`toJson`. Build runner ≈ `codegen` step trong TS + Zod.

---

> 💼 **Gặp trong dự án:** Tạo API service layer hoàn chỉnh (Dio + interceptors), setup JSON serialization cho 10+ model classes, handle token refresh khi 401, construct request/response logging
> 🤖 **Keywords bắt buộc trong prompt:** `Dio`, `interceptors`, `json_serializable`, `@JsonKey`, `fromJson/toJson`, `retrofit`, `base URL config`, `timeout config`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **New API layer:** Backend vừa cung cấp Swagger docs — cần tạo Dio client + API service + model classes
- **Interceptors:** Mọi request cần auth token header, response logging, error transformation 
- **JSON models:** 15 API endpoints, mỗi endpoint có request/response model cần `fromJson/toJson`

**Tại sao cần các keyword trên:**
- **`Dio`** — HTTP client mạnh mẽ hơn `http` package, hỗ trợ interceptors, cancel tokens, form data
- **`interceptors`** — middleware pattern cho request/response pipeline
- **`json_serializable`** — code generation cho JSON parsing, giảm boilerplate
- **`@JsonKey`** — customize field name mapping (snake_case → camelCase)
- **`retrofit`** — type-safe HTTP client (annotation-based, giống Retrofit trên Android)

**Prompt mẫu — Dio Service Setup:**
```text
Tôi cần setup Dio HTTP client hoàn chỉnh cho Flutter app.
Tech stack: Flutter 3.x, dio ^5.x, retrofit ^4.x, json_serializable ^6.x.
Requirements:
1. DioClient class: base URL, connectTimeout 30s, receiveTimeout 30s.
2. AuthInterceptor: thêm Bearer token từ SecureStorage cho mọi request.
3. LoggingInterceptor: log request method + URL + status code + response time.
4. ErrorInterceptor: catch DioException → map sang AppException hierarchy.
5. TokenRefreshInterceptor: nếu 401 → refresh token → retry original request → nếu refresh fail → logout.
6. Retrofit API service: @GET, @POST annotations cho /users, /posts endpoints.
Constraints:
- Token refresh phải handle race condition (2 requests cùng gặp 401).
- Interceptors order: Auth → Logging → Error (order quan trọng!).
- BaseOptions configurable per environment (dev/staging/prod).
Output: dio_client.dart, auth_interceptor.dart, logging_interceptor.dart, error_interceptor.dart, token_refresh_interceptor.dart, api_service.dart.
```

**Expected Output:** AI gen 6 files cho Dio setup hoàn chỉnh.

⚠️ **Giới hạn AI hay mắc:** AI thường quên handle race condition trong token refresh (2+ requests cùng 401 → chỉ refresh 1 lần rồi retry tất cả). AI cũng hay đặt sai thứ tự interceptors (Error trước Auth → token không được thêm vào retry request).

</details>

---

## 4. Retrofit — Type-safe HTTP Client 🟢

### 4.1 Giới thiệu

Retrofit sinh ra từ thế giới Android (Java/Kotlin), giờ có cho Dart. Dùng **annotation** để định nghĩa API, kết hợp với `dio` và **code generation**.

```yaml
# pubspec.yaml
dependencies:
  dio: ^5.4.3+1
  retrofit: ^4.1.0
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^8.1.0
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

### 4.2 Định nghĩa API Client

```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';

part 'api_client.g.dart';

@RestApi(baseUrl: 'https://jsonplaceholder.typicode.com')
abstract class ApiClient {
  factory ApiClient(Dio dio, {String baseUrl}) = _ApiClient;

  @GET('/posts')
  Future<List<Post>> getPosts();

  @GET('/posts/{id}')
  Future<Post> getPost(@Path('id') int postId);

  @GET('/posts')
  Future<List<Post>> getPostsByUser(@Query('userId') int userId);

  @POST('/posts')
  Future<Post> createPost(@Body() CreatePostRequest request);

  @PUT('/posts/{id}')
  Future<Post> updatePost(
    @Path('id') int postId,
    @Body() UpdatePostRequest request,
  );

  @DELETE('/posts/{id}')
  Future<void> deletePost(@Path('id') int postId);

  @POST('/upload')
  @MultiPart()
  Future<UploadResponse> uploadFile(
    @Part(name: 'file') File file,
    @Part(name: 'description') String description,
  );
}
```

### 4.3 Các Annotation quan trọng

```dart
// HTTP Methods
@GET('/path')       // GET request
@POST('/path')      // POST request
@PUT('/path')       // PUT request
@DELETE('/path')     // DELETE request
@PATCH('/path')      // PATCH request

// Parameters
@Path('name')        // URL path parameter  → /users/{name}
@Query('key')        // Query parameter     → ?key=value
@Queries()           // Map<String, dynamic> → multiple query params
@Body()              // Request body (auto serialize to JSON)
@Header('name')      // Single header
@Headers({'key': 'value'})  // Multiple headers

// Đặc biệt
@MultiPart()         // Multipart form data
@Part(name: 'key')   // Part trong multipart
@FormUrlEncoded()    // x-www-form-urlencoded
@Field('key')        // Field trong form
```

### 4.4 Kết hợp Retrofit + DI (injectable)

```dart
@module
abstract class NetworkModule {
  @lazySingleton
  Dio dio(AuthInterceptor authInterceptor) {
    final dio = Dio(BaseOptions(
      baseUrl: Environment.apiBaseUrl,
      connectTimeout: const Duration(seconds: 15),
      receiveTimeout: const Duration(seconds: 15),
    ));
    dio.interceptors.addAll([
      authInterceptor,
      LoggingInterceptor(),
    ]);
    return dio;
  }

  @lazySingleton
  ApiClient apiClient(Dio dio) => ApiClient(dio);
}
```

---

## 5. Auth Token Management 🔴

### 5.1 Bearer Token Flow

```
┌──────────┐     POST /login        ┌──────────┐
│  Client   │ ──────────────────────▶│  Server  │
│  (App)    │ { email, password }    │          │
│           │◀──────────────────────│          │
│           │ { accessToken,         │          │
│           │   refreshToken }       │          │
└──────────┘                        └──────────┘
     │
     │  Lưu tokens vào secure storage
     │
     ▼
┌──────────────────────────────────────────────┐
│  Mọi API call sau đó:                        │
│  Header: Authorization: Bearer <accessToken>  │
└──────────────────────────────────────────────┘
```

### 5.2 Token Storage với flutter_secure_storage

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
class TokenStorage {
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  final FlutterSecureStorage _storage;

  const TokenStorage(this._storage);

  Future<String?> getAccessToken() =>
      _storage.read(key: _accessTokenKey);

  Future<String?> getRefreshToken() =>
      _storage.read(key: _refreshTokenKey);

  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey, value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<void> clearTokens() async {
    await Future.wait([
      _storage.delete(key: _accessTokenKey),
      _storage.delete(key: _refreshTokenKey),
    ]);
  }

  Future<bool> hasTokens() async {
    final token = await getAccessToken();
    return token != null && token.isNotEmpty;
  }
}
```

### 5.3 Token Refresh Interceptor (Quan trọng!)

Đây là phần phức tạp nhất — xử lý khi access token hết hạn:

```
Request ──▶ Server trả 401
                │
                ▼
        Token hết hạn?
           │        │
          Có       Không
           │        │
           ▼        ▼
    Gọi /refresh   Throw error
    với refreshToken
           │
           ▼
    Nhận token mới?
      │         │
     Có        Không
      │         │
      ▼         ▼
  Lưu token   Logout user
  Retry request (clear tokens)
  ban đầu
```

```dart
class TokenRefreshInterceptor extends QueuedInterceptor {
  final Dio _dio;
  final TokenStorage _tokenStorage;
  final Dio _refreshDio; // Dio riêng để refresh (tránh vòng lặp)

  TokenRefreshInterceptor(this._dio, this._tokenStorage)
      : _refreshDio = Dio(BaseOptions(
          baseUrl: _dio.options.baseUrl,
        ));

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await _tokenStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(
    DioException err,
    ErrorInterceptorHandler handler,
  ) async {
    if (err.response?.statusCode == 401) {
      try {
        // Thử refresh token
        final refreshToken = await _tokenStorage.getRefreshToken();
        if (refreshToken == null) {
          return handler.next(err);
        }

        final response = await _refreshDio.post(
          '/auth/refresh',
          data: {'refresh_token': refreshToken},
        );

        final newAccessToken = response.data['access_token'] as String;
        final newRefreshToken = response.data['refresh_token'] as String;

        // Lưu token mới
        await _tokenStorage.saveTokens(
          accessToken: newAccessToken,
          refreshToken: newRefreshToken,
        );

        // Retry request ban đầu với token mới
        final options = err.requestOptions;
        options.headers['Authorization'] = 'Bearer $newAccessToken';
        final retryResponse = await _dio.fetch(options);
        return handler.resolve(retryResponse);
      } catch (e) {
        // Refresh thất bại → clear tokens (force logout)
        await _tokenStorage.clearTokens();
        return handler.next(err);
      }
    }
    handler.next(err);
  }
}
```

> **Lưu ý:** Dùng `QueuedInterceptor` thay vì `Interceptor` thông thường. `QueuedInterceptor` đảm bảo khi nhiều request cùng gặp 401, chỉ **một request refresh** được thực hiện, các request khác đợi trong queue.

---

## 6. Error Handling cho Network 🔴

### 6.1 DioException Types

```dart
switch (error.type) {
  case DioExceptionType.connectionTimeout:
    // Server không phản hồi trong thời gian cho phép
    break;
  case DioExceptionType.sendTimeout:
    // Gửi data quá lâu
    break;
  case DioExceptionType.receiveTimeout:
    // Nhận data quá lâu
    break;
  case DioExceptionType.badResponse:
    // Server trả về error (4xx, 5xx)
    final statusCode = error.response?.statusCode;
    break;
  case DioExceptionType.cancel:
    // Request bị cancel bởi CancelToken
    break;
  case DioExceptionType.connectionError:
    // Không có kết nối mạng
    break;
  case DioExceptionType.unknown:
    // Lỗi không xác định
    break;
  default:
    break;
}
```

### 6.2 Network Error → Domain Error Mapping

Từ Clean Architecture (Buổi 9), ta map network error sang domain error:

```dart
// Domain layer — Failure classes
abstract class Failure {
  final String message;
  const Failure(this.message);
}

class ServerFailure extends Failure {
  final int? statusCode;
  const ServerFailure(super.message, {this.statusCode});
}

class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

class TimeoutFailure extends Failure {
  const TimeoutFailure(super.message);
}

class UnauthorizedFailure extends Failure {
  const UnauthorizedFailure(super.message);
}

class NotFoundFailure extends Failure {
  const NotFoundFailure(super.message);
}
```

```dart
// Data layer — Error mapper
class NetworkErrorMapper {
  static Failure mapDioException(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        return const TimeoutFailure(
          'Kết nối quá chậm. Vui lòng thử lại.',
        );

      case DioExceptionType.connectionError:
        return const NetworkFailure(
          'Không có kết nối mạng. Kiểm tra WiFi/4G.',
        );

      case DioExceptionType.badResponse:
        return _mapStatusCode(error.response);

      case DioExceptionType.cancel:
        return const NetworkFailure('Request đã bị hủy.');

      default:
        return const ServerFailure('Đã xảy ra lỗi. Vui lòng thử lại.');
    }
  }

  static Failure _mapStatusCode(Response? response) {
    final statusCode = response?.statusCode;
    final message = _extractMessage(response);

    switch (statusCode) {
      case 400:
        return ServerFailure(message ?? 'Dữ liệu không hợp lệ.', statusCode: 400);
      case 401:
        return const UnauthorizedFailure('Phiên đăng nhập hết hạn.');
      case 403:
        return const ServerFailure('Bạn không có quyền truy cập.', statusCode: 403);
      case 404:
        return const NotFoundFailure('Không tìm thấy dữ liệu.');
      case 500:
        return const ServerFailure('Lỗi server. Vui lòng thử lại sau.', statusCode: 500);
      default:
        return ServerFailure(
          message ?? 'Lỗi không xác định.',
          statusCode: statusCode,
        );
    }
  }

  static String? _extractMessage(Response? response) {
    try {
      final data = response?.data;
      if (data is Map<String, dynamic>) {
        return data['message'] as String?;
      }
    } catch (_) {}
    return null;
  }
}
```

### 6.3 Either Pattern với fpdart

> 💡 **fpdart vs dartz**: `fpdart` là bản kế thừa actively maintained của `dartz`. API tương tự (`Either`, `Option`, `fold`). Training này dùng `fpdart` — nếu gặp code cũ dùng `dartz`, cú pháp gần như giống hệt.

`Either<L, R>` — trả về **hoặc** error (Left) **hoặc** success (Right):

```yaml
dependencies:
  fpdart: ^1.1.0  # fpdart is the actively maintained successor to dartz
```

```dart
import 'package:fpdart/fpdart.dart';

// Repository interface (Domain layer)
abstract class PostRepository {
  Future<Either<Failure, List<Post>>> getPosts();
  Future<Either<Failure, Post>> getPostById(int id);
  Future<Either<Failure, Post>> createPost(CreatePostRequest request);
}

// Repository implementation (Data layer)
class PostRepositoryImpl implements PostRepository {
  final ApiClient _apiClient;

  PostRepositoryImpl(this._apiClient);

  @override
  Future<Either<Failure, List<Post>>> getPosts() async {
    try {
      final posts = await _apiClient.getPosts();
      return Right(posts);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }

  @override
  Future<Either<Failure, Post>> getPostById(int id) async {
    try {
      final post = await _apiClient.getPost(id);
      return Right(post);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }

  @override
  Future<Either<Failure, Post>> createPost(CreatePostRequest request) async {
    try {
      final post = await _apiClient.createPost(request);
      return Right(post);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }
}
```

### 6.4 Sử dụng Either trong UseCase & UI

```dart
// UseCase
class GetPostsUseCase {
  final PostRepository _repository;
  GetPostsUseCase(this._repository);

  Future<Either<Failure, List<Post>>> call() => _repository.getPosts();
}

// Trong BLoC/Cubit
class PostCubit extends Cubit<PostState> {
  final GetPostsUseCase _getPostsUseCase;

  PostCubit(this._getPostsUseCase) : super(const PostState.initial());

  Future<void> loadPosts() async {
    emit(const PostState.loading());

    final result = await _getPostsUseCase();

    result.fold(
      (failure) => emit(PostState.error(failure.message)),
      (posts) => emit(PostState.loaded(posts)),
    );
  }
}
```

### 6.5 Kiểm tra kết nối mạng

```yaml
dependencies:
  connectivity_plus: ^6.0.0
```

```dart
class NetworkInfo {
  final Connectivity _connectivity;

  const NetworkInfo(this._connectivity);

  Future<bool> get isConnected async {
    final result = await _connectivity.checkConnectivity();
    return !result.contains(ConnectivityResult.none);
  }
}

// Sử dụng trong Repository
@override
Future<Either<Failure, List<Post>>> getPosts() async {
  if (!await _networkInfo.isConnected) {
    return const Left(
      NetworkFailure('Không có kết nối mạng.'),
    );
  }

  try {
    final posts = await _apiClient.getPosts();
    return Right(posts);
  } on DioException catch (e) {
    return Left(NetworkErrorMapper.mapDioException(e));
  }
}
```

> 🔗 **FE Bridge:** API error handling pattern **tương đồng**: try/catch cho network error, check status code cho business error. `DioException` ≈ `AxiosError` — cùng chứa `response`, `statusCode`. Pattern `Result<T>` / `Either<Failure, T>` = typed error handling, FE thường dùng discriminated union thay thế.

---

> 💼 **Gặp trong dự án:** Map network errors thành user-friendly messages, handle offline mode gracefully, retry logic cho unstable connections, circuit breaker pattern cho backend instability
> 🤖 **Keywords bắt buộc trong prompt:** `NetworkFailure`, `DioException mapping`, `error hierarchy`, `retry policy`, `offline detection`, `connectivity check`, `user-friendly error messages`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Production issue:** Users thấy "DioException [bad response]" thay vì message tiếng Việt dễ hiểu
- **QA report:** App crash khi mất mạng giữa chừng (no offline handling)
- **Retry logic:** Flaky API cần retry 3 lần với exponential backoff trước khi báo lỗi

**Tại sao cần các keyword trên:**
- **`NetworkFailure`** — custom Failure class hierarchy (ServerFailure, CacheFailure, NetworkFailure)
- **`DioException mapping`** — map DioException types thành app-level Failures
- **`error hierarchy`** — sealed class pattern cho exhaustive error handling
- **`retry policy`** — exponential backoff: 1s → 2s → 4s, max 3 retries, chỉ retry cho network/timeout errors

**Prompt mẫu — Error Handling Layer:**
```text
Tôi cần xây dựng error handling layer cho Flutter networking.
Tech stack: dio ^5.x, fpdart (Either), connectivity_plus.

> ℹ️ `dartz` is no longer maintained. Use [`fpdart`](https://pub.dev/packages/fpdart) — the actively maintained successor with better Dart 3.x support.
Requirements:
1. Failure hierarchy: sealed class Failure → ServerFailure (statusCode, message), NetworkFailure (message), TimeoutFailure, CacheFailure.
2. NetworkErrorMapper: DioException → Failure (map theo type: connectionTimeout, badResponse, cancel...).
3. Message mapping: Failure → user-friendly String tiếng Việt.
4. Retry interceptor: retry 3 lần với exponential backoff cho connectionError + timeout. Skip retry cho 4xx errors.
5. Connectivity guard: check network trước khi call API, return NetworkFailure ngay nếu offline.
Constraints:
- Failure dùng sealed class (Dart 3 pattern matching).
- Message mapping không hardcode — dùng Map hoặc extension.
- Retry chỉ cho idempotent operations (GET, không retry POST).
Output: failure.dart, network_error_mapper.dart, retry_interceptor.dart, connectivity_guard.dart.
```

**Expected Output:** AI gen 4 files error handling layer.

⚠️ **Giới hạn AI hay mắc:** AI hay retry tất cả HTTP methods (kể cả POST → duplicate data!). AI cũng hay quên handle case "refresh token expired" trong retry flow (vòng lặp vô tận).

</details>

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

1. **Luôn dùng `BaseOptions`** — Set base URL, timeout, default headers ở một chỗ
2. **Tạo Dio instance duy nhất** — Inject qua DI, không `Dio()` rải rác
3. **Dùng `QueuedInterceptor` cho auth** — Tránh race condition khi refresh token
4. **Dùng `freezed` cho model** — Immutable, copyWith, equality miễn phí
5. **Map error sang domain** — UI không nên biết về `DioException`
6. **Luôn dùng `Either`** — Explicit error handling, không exception ẩn
7. **Tách Dio cho refresh** — Tránh vòng lặp interceptor
8. **Set timeout hợp lý** — 10-15s connect, 15-30s receive

### ❌ Lỗi thường gặp

| Lỗi | Giải thích | Cách sửa |
|-----|-----------|----------|
| Quên `explicitToJson: true` | Nested object không serialize đúng | Thêm annotation vào `@JsonSerializable` |
| Quên chạy `build_runner` | Code gen files chưa được tạo | `dart run build_runner build` |
| Dùng chung Dio instance cho refresh | Interceptor chạy vòng lặp vô hạn | Tạo Dio riêng cho refresh endpoint |
| Không handle cancel | App crash khi navigate giữa request | Dùng `CancelToken`, cancel trong `dispose()` |
| Parse JSON trong main isolate | UI jank với data lớn | Dùng `compute()` cho large JSON |
| Hardcode base URL | Khó switch môi trường | Dùng env config hoặc flavor |
| Không check connectivity trước | UX tệ khi offline | Check `connectivity_plus` trước request |
| Token lưu SharedPreferences | Không an toàn | Dùng `flutter_secure_storage` |

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### dio vs axios

| Concept | axios (JS) | dio (Dart) |
|---------|-----------|-----------|
| Instance | `axios.create({...})` | `Dio(BaseOptions(...))` |
| GET | `axios.get('/path')` | `dio.get('/path')` |
| POST | `axios.post('/path', data)` | `dio.post('/path', data: data)` |
| Interceptor request | `axios.interceptors.request.use()` | `dio.interceptors.add(onRequest:)` |
| Interceptor response | `axios.interceptors.response.use()` | `dio.interceptors.add(onResponse:)` |
| Cancel | `AbortController` | `CancelToken` |
| Error type | `axios.isAxiosError()` | `DioException` |

> **Gần như 1-1!** Nếu bạn quen axios, bạn sẽ rất thoải mái với dio.

### Interceptors: axios vs dio

```javascript
// axios interceptor
axios.interceptors.request.use(
  config => {
    config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  error => Promise.reject(error)
);
```

```dart
// dio interceptor — tương tự nhưng structured hơn
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    options.headers['Authorization'] = 'Bearer $token';
    handler.next(options);
  },
  onError: (error, handler) {
    handler.next(error);
  },
));
```

### Retrofit vs React Query / SWR

> ⚠️ **Paradigm hoàn toàn khác!**

| | Retrofit (Dart) | React Query / SWR (JS) |
|--|-----------------|----------------------|
| Vai trò | **HTTP client** — chỉ gọi API | **Server state manager** — cache, refetch, invalidate |
| Focus | Type-safe API definition | Data synchronization & caching |
| Caching | Không có | Built-in |
| Tương đương Flutter | Retrofit | Riverpod + caching logic |

Retrofit chỉ là **tầng gọi API**, còn React Query/SWR quản lý cả **state của data từ server**. Trong Flutter, bạn kết hợp Retrofit + Riverpod/BLoC để đạt hiệu quả tương đương.

### json_serializable vs Zod / io-ts

| | json_serializable (Dart) | Zod / io-ts (TS) |
|--|-------------------------|------------------|
| Thời điểm | **Build time** (code gen) | **Runtime** (validation) |
| Cách hoạt động | Generate fromJson/toJson | Schema validation |
| Type safety | Strong (Dart's type system) | Runtime validation |
| Performance | Nhanh (pre-compiled) | Chậm hơn (runtime) |

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|--------------|
| 1 | `fetch()` / `axios` built-in | `Dio` / `http` package — phải add dependency | Dart standard library không có HTTP client built-in |
| 2 | JSON → object = tự động (JS dynamic) | JSON → class = **code generation** (`fromJson`/`toJson`) | Dart strongly typed → cần explicit deserialization |
| 3 | `zod.parse()` runtime validation | `freezed` + `json_serializable` compile-time gen | Dart approach = generate code, TS approach = runtime check |
| 4 | Error = `catch(error)` generic | Error = typed `DioException`, pattern `Either<Failure,T>` | Flutter community prefer typed error handling |

---

## 9. Tổng kết

### ✅ Checklist kiến thức

Sau buổi học, hãy đảm bảo bạn:

- [ ] Hiểu sự khác biệt `http` vs `dio`, biết khi nào dùng cái nào
- [ ] Viết được interceptor: logging, auth token injection, retry
- [ ] Tạo model với `json_serializable` (annotation, build_runner)
- [ ] Biết dùng `freezed` cho immutable models
- [ ] Handle nested objects, lists, null safety trong JSON
- [ ] Tạo Retrofit API client với các annotation cơ bản
- [ ] Triển khai auth token flow: save, inject, refresh, retry
- [ ] Lưu token an toàn với `flutter_secure_storage`
- [ ] Map `DioException` sang domain `Failure`
- [ ] Sử dụng `Either<Failure, Success>` pattern
- [ ] Check connectivity trước khi gọi API

### 🔗 Liên kết buổi tiếp theo

**Buổi 12: Local Storage & Persistence** — Bạn sẽ học cách lưu data offline, cache API response, và xây dựng offline-first experience.

```
Buổi 11: Networking ──▶ Buổi 12: Local Storage
(Remote data)            (Local data + caching)
       │                        │
       └────────────────────────┘
              Kết hợp = Offline-first app
```
