# Buổi 11: Networking & API Integration — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## VD1: Basic HTTP GET Request — Fetch Users (⭐) 🟡

> **Liên quan tới:** [1. http vs dio — So sánh & Lựa chọn](01-ly-thuyet.md#1-http-vs-dio--so-sánh--lựa-chọn)

> **Mục tiêu:** Gọi API đơn giản với package `http`, parse JSON thủ công.

### Model

```dart
class User {
  final int id;
  final String name;
  final String email;
  final String phone;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.phone,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      phone: json['phone'] as String,
    );
  }
}
```

### Service với `http`

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class UserService {
  static const _baseUrl = 'https://jsonplaceholder.typicode.com';

  Future<List<User>> getUsers() async {
    final uri = Uri.parse('$_baseUrl/users');
    final response = await http.get(uri);

    if (response.statusCode == 200) {
      final List<dynamic> jsonList = jsonDecode(response.body);
      return jsonList.map((json) => User.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load users: ${response.statusCode}');
    }
  }

  Future<User> getUserById(int id) async {
    final uri = Uri.parse('$_baseUrl/users/$id');
    final response = await http.get(uri);

    if (response.statusCode == 200) {
      final json = jsonDecode(response.body) as Map<String, dynamic>;
      return User.fromJson(json);
    } else if (response.statusCode == 404) {
      throw Exception('User not found');
    } else {
      throw Exception('Failed to load user: ${response.statusCode}');
    }
  }
}
```

### Widget hiển thị

```dart
class UserListScreen extends StatefulWidget {
  const UserListScreen({super.key});

  @override
  State<UserListScreen> createState() => _UserListScreenState();
}

class _UserListScreenState extends State<UserListScreen> {
  late Future<List<User>> _usersFuture;
  final _userService = UserService();

  @override
  void initState() {
    super.initState();
    _usersFuture = _userService.getUsers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Users')),
      body: FutureBuilder<List<User>>(
        future: _usersFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text('Error: ${snapshot.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      setState(() {
                        _usersFuture = _userService.getUsers();
                      });
                    },
                    child: const Text('Retry'),
                  ),
                ],
              ),
            );
          }

          final users = snapshot.data!;
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return ListTile(
                leading: CircleAvatar(child: Text(user.name[0])),
                title: Text(user.name),
                subtitle: Text(user.email),
                trailing: Text(user.phone),
              );
            },
          );
        },
      ),
    );
  }
}
```

### Giải thích flow

```
UserListScreen
     │
     │ initState()
     ▼
UserService.getUsers()
     │
     │ http.get('https://jsonplaceholder.../users')
     ▼
Server trả JSON array
     │
     │ jsonDecode() → List<dynamic>
     │ .map() → List<User>
     ▼
FutureBuilder render ListView
```

> **💡 Từ Frontend:** Giống `fetch('/api/users').then(res => res.json())` trong JavaScript. Đơn giản nhưng phải handle mọi thứ manually.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_http_basic
cd vidu_http_basic
flutter pub add http
# Thay nội dung lib/ bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ App hiển thị danh sách 10 users từ jsonplaceholder API
✅ Mỗi user hiện avatar (chữ cái đầu), tên, email, phone
✅ Loading spinner khi đang fetch
✅ Nút Retry khi có lỗi mạng
```

---

## VD2: Dio với Interceptors — Logging + Auth (⭐⭐) 🔴

> **Liên quan tới:** [2. Interceptors — Middleware cho HTTP](01-ly-thuyet.md#2-interceptors--middleware-cho-http)

> **Mục tiêu:** Cấu hình Dio với BaseOptions, thêm interceptor logging và auth token.

### Cấu hình Dio

```dart
import 'package:dio/dio.dart';

class DioClient {
  late final Dio _dio;

  DioClient({required String baseUrl, String? authToken}) {
    _dio = Dio(
      BaseOptions(
        baseUrl: baseUrl,
        connectTimeout: const Duration(seconds: 10),
        receiveTimeout: const Duration(seconds: 15),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    // Thêm interceptors theo thứ tự
    _dio.interceptors.addAll([
      _AuthInterceptor(authToken),
      _LoggingInterceptor(),
    ]);
  }

  Dio get dio => _dio;
}
```

### Auth Interceptor

```dart
class _AuthInterceptor extends Interceptor {
  String? _token;

  _AuthInterceptor(this._token);

  void updateToken(String? token) {
    _token = token;
  }

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (_token != null && _token!.isNotEmpty) {
      options.headers['Authorization'] = 'Bearer $_token';
    }
    handler.next(options);
  }
}
```

### Logging Interceptor

```dart
class _LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final timestamp = DateTime.now().toIso8601String();
    print('');
    print('╔══════════════════════════════════════════');
    print('║ 📤 REQUEST [$timestamp]');
    print('║ ${options.method} ${options.uri}');
    if (options.headers.isNotEmpty) {
      print('║ Headers: ${_sanitizeHeaders(options.headers)}');
    }
    if (options.data != null) {
      print('║ Body: ${options.data}');
    }
    print('╚══════════════════════════════════════════');
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('');
    print('╔══════════════════════════════════════════');
    print('║ 📥 RESPONSE [${response.statusCode}]');
    print('║ ${response.requestOptions.method} ${response.requestOptions.uri}');
    print('║ Data: ${_truncate(response.data.toString(), 200)}');
    print('╚══════════════════════════════════════════');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print('');
    print('╔══════════════════════════════════════════');
    print('║ ❌ ERROR [${err.response?.statusCode ?? 'N/A'}]');
    print('║ ${err.requestOptions.method} ${err.requestOptions.uri}');
    print('║ Type: ${err.type}');
    print('║ Message: ${err.message}');
    print('╚══════════════════════════════════════════');
    handler.next(err);
  }

  /// Ẩn giá trị token trong log
  Map<String, dynamic> _sanitizeHeaders(Map<String, dynamic> headers) {
    return headers.map((key, value) {
      if (key.toLowerCase() == 'authorization') {
        return MapEntry(key, 'Bearer ***');
      }
      return MapEntry(key, value);
    });
  }

  String _truncate(String text, int maxLength) {
    if (text.length <= maxLength) return text;
    return '${text.substring(0, maxLength)}... [truncated]';
  }
}
```

### Sử dụng

```dart
void main() async {
  final client = DioClient(
    baseUrl: 'https://jsonplaceholder.typicode.com',
    authToken: 'my-secret-token-123',
  );

  try {
    // GET — Lấy danh sách posts
    final response = await client.dio.get('/posts');
    final posts = (response.data as List)
        .map((json) => Post.fromJson(json))
        .toList();
    print('Loaded ${posts.length} posts');

    // POST — Tạo post mới
    final createResponse = await client.dio.post('/posts', data: {
      'title': 'Dio with Interceptors',
      'body': 'This is a test post',
      'userId': 1,
    });
    print('Created post: ${createResponse.data}');
  } on DioException catch (e) {
    print('API Error: ${e.message}');
  }
}
```

### Output trong console

```
╔══════════════════════════════════════════
║ 📤 REQUEST [2026-03-31T10:30:00.000]
║ GET https://jsonplaceholder.typicode.com/posts
║ Headers: {Authorization: Bearer ***, Content-Type: application/json}
╚══════════════════════════════════════════

╔══════════════════════════════════════════
║ 📥 RESPONSE [200]
║ GET https://jsonplaceholder.typicode.com/posts
║ Data: [{id: 1, title: sunt aut facere...}, ...] [truncated]
╚══════════════════════════════════════════
```

> **💡 Từ Frontend:** Giống `axios.interceptors.request.use()` — cùng concept, khác syntax.

- 🔗 **FE tương đương:** `dio.get(url)` ≈ `axios.get(url)` — response structure tương tự: `.data` chứa body, `.statusCode` chứa status. FE dev sẽ thấy rất familiar.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_dio_interceptor
cd vidu_dio_interceptor
flutter pub add dio
# Thay nội dung lib/ bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Console hiển thị log request/response với format box (╔══...╚══)
✅ Auth header tự động thêm "Bearer ***" vào mọi request
✅ Sensitive headers được mask trong log (Authorization: Bearer ***)
✅ Error log hiển thị status code và message khi request fail
```

---

## VD3: json_serializable Model — User (⭐⭐) 🟢

> **Liên quan tới:** [3. JSON Serialization](01-ly-thuyet.md#3-json-serialization)

> **Mục tiêu:** Tạo model class với code generation, handle nested object và null safety.

### Setup pubspec.yaml

```yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

### Model: User

```dart
// lib/models/user.dart
import 'package:json_annotation/json_annotation.dart';
import 'address.dart';
import 'company.dart';

part 'user.g.dart';

@JsonSerializable(explicitToJson: true)
class User {
  final int id;
  final String name;

  @JsonKey(name: 'username')
  final String userName;

  final String email;
  final String? phone;
  final String? website;

  final Address address;   // Nested object
  final Company? company;  // Nullable nested object

  @JsonKey(name: 'created_at')
  final DateTime? createdAt;

  const User({
    required this.id,
    required this.name,
    required this.userName,
    required this.email,
    this.phone,
    this.website,
    required this.address,
    this.company,
    this.createdAt,
  });

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

### Model: Address (Nested)

```dart
// lib/models/address.dart
import 'package:json_annotation/json_annotation.dart';

part 'address.g.dart';

@JsonSerializable(explicitToJson: true)
class Address {
  final String street;
  final String suite;
  final String city;
  final String zipcode;
  final Geo? geo;

  const Address({
    required this.street,
    required this.suite,
    required this.city,
    required this.zipcode,
    this.geo,
  });

  factory Address.fromJson(Map<String, dynamic> json) =>
      _$AddressFromJson(json);
  Map<String, dynamic> toJson() => _$AddressToJson(this);
}

@JsonSerializable()
class Geo {
  final String lat;
  final String lng;

  const Geo({required this.lat, required this.lng});

  factory Geo.fromJson(Map<String, dynamic> json) => _$GeoFromJson(json);
  Map<String, dynamic> toJson() => _$GeoToJson(this);
}
```

### Model: Company (Nullable Nested)

```dart
// lib/models/company.dart
import 'package:json_annotation/json_annotation.dart';

part 'company.g.dart';

@JsonSerializable()
class Company {
  final String name;
  final String? catchPhrase;
  final String? bs;

  const Company({
    required this.name,
    this.catchPhrase,
    this.bs,
  });

  factory Company.fromJson(Map<String, dynamic> json) =>
      _$CompanyFromJson(json);
  Map<String, dynamic> toJson() => _$CompanyToJson(this);
}
```

### Chạy code generation

```bash
# Sau khi tạo models, chạy:
dart run build_runner build --delete-conflicting-outputs

# Hoặc watch mode (development):
dart run build_runner watch --delete-conflicting-outputs
```

### Sử dụng với API response

```dart
import 'dart:convert';

// Parse single object
final jsonString = '{"id":1,"name":"Leanne Graham","username":"Bret",...}';
final json = jsonDecode(jsonString) as Map<String, dynamic>;
final user = User.fromJson(json);
print(user.name);        // Leanne Graham
print(user.address.city); // Gwenborough

// Parse list
final listJson = jsonDecode(responseBody) as List<dynamic>;
final users = listJson
    .map((e) => User.fromJson(e as Map<String, dynamic>))
    .toList();

// Serialize back to JSON
final map = user.toJson();
final body = jsonEncode(map);
print(body); // {"id":1,"name":"Leanne Graham","username":"Bret",...}
```

### Lưu ý quan trọng

```dart
// ⚠️ PHẢI dùng explicitToJson: true cho nested objects!
@JsonSerializable(explicitToJson: true)  // ✅
class User { ... }

// Nếu không, nested objects sẽ output "Instance of 'Address'" thay vì JSON

// ⚠️ Mỗi lần thay đổi model → chạy lại build_runner
// Hoặc dùng watch mode để tự động
```

> **💡 Từ Frontend:** Giống TypeScript interface + runtime validation. Nhưng code gen ở build time → performance tốt hơn Zod/io-ts ở runtime.

- 🔗 **FE tương đương:** `fromJson()` factory ≈ `zod.parse()` hoặc manual mapping `new User(json)` — nhưng Dart dùng code generation tự động tạo parse logic.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_json_serializable
cd vidu_json_serializable
flutter pub add json_annotation
flutter pub add dev:json_serializable dev:build_runner
# Tạo model files (user.dart, address.dart, company.dart), rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ build_runner generate user.g.dart, address.g.dart, company.g.dart
✅ User.fromJson() parse nested Address và nullable Company chính xác
✅ user.toJson() output JSON với explicitToJson cho nested objects
✅ @JsonKey(name: 'username') map đúng field name từ API
```

---

## VD4: Retrofit API Client (⭐⭐) 🟡

> **Mục tiêu:** Tạo type-safe API client với annotation, kết hợp dio.

> **Liên quan tới:** [4. Retrofit — Type-safe HTTP Client](01-ly-thuyet.md#4-retrofit--type-safe-http-client)

### Setup

```yaml
dependencies:
  dio: ^5.4.3+1
  retrofit: ^4.1.0
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^8.1.0
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

### API Client

```dart
// lib/data/api/user_api_client.dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../../models/user.dart';

part 'user_api_client.g.dart';

@RestApi()
abstract class UserApiClient {
  factory UserApiClient(Dio dio, {String? baseUrl}) = _UserApiClient;

  /// GET /users — Lấy tất cả users
  @GET('/users')
  Future<List<User>> getUsers();

  /// GET /users/{id} — Lấy user theo ID
  @GET('/users/{id}')
  Future<User> getUserById(@Path('id') int userId);

  /// GET /users?name=... — Tìm user theo tên
  @GET('/users')
  Future<List<User>> searchUsers(@Query('name') String name);

  /// POST /users — Tạo user mới
  @POST('/users')
  Future<User> createUser(@Body() CreateUserRequest request);

  /// PUT /users/{id} — Cập nhật user
  @PUT('/users/{id}')
  Future<User> updateUser(
    @Path('id') int userId,
    @Body() UpdateUserRequest request,
  );

  /// DELETE /users/{id} — Xóa user
  @DELETE('/users/{id}')
  Future<void> deleteUser(@Path('id') int userId);

  /// GET /users/{id}/posts — Lấy posts của user
  @GET('/users/{id}/posts')
  Future<List<Post>> getUserPosts(@Path('id') int userId);
}
```

### Request Models

```dart
// lib/models/requests/create_user_request.dart
import 'package:json_annotation/json_annotation.dart';

part 'create_user_request.g.dart';

@JsonSerializable()
class CreateUserRequest {
  final String name;
  final String email;
  final String? phone;

  const CreateUserRequest({
    required this.name,
    required this.email,
    this.phone,
  });

  factory CreateUserRequest.fromJson(Map<String, dynamic> json) =>
      _$CreateUserRequestFromJson(json);
  Map<String, dynamic> toJson() => _$CreateUserRequestToJson(this);
}

@JsonSerializable()
class UpdateUserRequest {
  final String? name;
  final String? email;
  final String? phone;

  const UpdateUserRequest({this.name, this.email, this.phone});

  factory UpdateUserRequest.fromJson(Map<String, dynamic> json) =>
      _$UpdateUserRequestFromJson(json);
  Map<String, dynamic> toJson() => _$UpdateUserRequestToJson(this);
}
```

### Kết nối Retrofit + Dio

```dart
// lib/data/api/api_provider.dart

class ApiProvider {
  late final Dio _dio;
  late final UserApiClient _userApiClient;

  ApiProvider() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://jsonplaceholder.typicode.com',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 15),
      headers: {'Content-Type': 'application/json'},
    ));

    _dio.interceptors.addAll([
      LoggingInterceptor(),
      // AuthInterceptor(tokenStorage),
    ]);

    _userApiClient = UserApiClient(_dio);
  }

  UserApiClient get userApi => _userApiClient;
}
```

### Sử dụng

```dart
final apiProvider = ApiProvider();

// Tất cả đều type-safe!
final users = await apiProvider.userApi.getUsers();           // List<User>
final user = await apiProvider.userApi.getUserById(1);        // User
final posts = await apiProvider.userApi.getUserPosts(1);      // List<Post>

final newUser = await apiProvider.userApi.createUser(
  const CreateUserRequest(name: 'John', email: 'john@test.com'),
);
// newUser: User — type-safe response!

await apiProvider.userApi.deleteUser(1); // void
```

### So sánh: Trước và sau Retrofit

```dart
// ❌ TRƯỚC: Manual dio calls — verbose, không type-safe
Future<List<User>> getUsers() async {
  final response = await dio.get('/users');
  return (response.data as List)
      .map((json) => User.fromJson(json))
      .toList();
}

// ✅ SAU: Retrofit — clean, type-safe
@GET('/users')
Future<List<User>> getUsers();
// Retrofit tự động: gọi dio.get, parse JSON, map sang List<User>
```

> **💡 Từ Frontend:** Retrofit không có equivalent trực tiếp trong React/Vue. Gần nhất là tRPC hoặc OpenAPI codegen — ý tưởng là generate type-safe client từ API definition.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_retrofit
cd vidu_retrofit
flutter pub add dio retrofit json_annotation
flutter pub add dev:retrofit_generator dev:json_serializable dev:build_runner
# Tạo model files và API client file theo code trên, rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ build_runner generate user_api_client.g.dart và request model .g.dart files
✅ getUsers() trả về List<User> — type-safe, không cần parse JSON thủ công
✅ createUser() nhận CreateUserRequest object → tự serialize thành JSON body
✅ deleteUser() trả về void — không cần handle response body
✅ Tất cả API calls đều type-safe tại compile time
```

---

## VD5: Complete Network Layer — Clean Architecture (⭐⭐⭐) 🟡

> **Mục tiêu:** Xây dựng network layer hoàn chỉnh: DataSource → Repository → UseCase với error handling `Either` pattern.

> **Liên quan tới:** [6. Error Handling cho Network](01-ly-thuyet.md#6-error-handling-cho-network)

### Cấu trúc thư mục

```
lib/
├── core/
│   ├── error/
│   │   ├── failures.dart
│   │   └── network_error_mapper.dart
│   └── network/
│       ├── dio_client.dart
│       └── network_info.dart
├── features/
│   └── posts/
│       ├── data/
│       │   ├── datasources/
│       │   │   └── post_remote_datasource.dart
│       │   ├── models/
│       │   │   └── post_model.dart
│       │   └── repositories/
│       │       └── post_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── post.dart
│       │   ├── repositories/
│       │   │   └── post_repository.dart
│       │   └── usecases/
│       │       └── get_posts.dart
│       └── presentation/
│           └── ...
```

### Bước 1: Domain Layer — Entity & Repository Interface

```dart
// lib/features/posts/domain/entities/post.dart
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
}

// lib/features/posts/domain/repositories/post_repository.dart
import 'package:fpdart/fpdart.dart'; // fpdart is the actively maintained successor to dartz

abstract class PostRepository {
  Future<Either<Failure, List<Post>>> getPosts();
  Future<Either<Failure, Post>> getPostById(int id);
  Future<Either<Failure, Post>> createPost({
    required String title,
    required String body,
    required int userId,
  });
}
```

### Bước 2: Core — Failure & Error Mapper

```dart
// lib/core/error/failures.dart
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

// lib/core/error/network_error_mapper.dart
import 'package:dio/dio.dart';
import 'failures.dart';

class NetworkErrorMapper {
  static Failure mapDioException(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        return const TimeoutFailure('Request timed out. Vui lòng thử lại.');
      case DioExceptionType.connectionError:
        return const NetworkFailure('Không có kết nối mạng.');
      case DioExceptionType.badResponse:
        final statusCode = error.response?.statusCode ?? 500;
        final message = _extractMessage(error.response) ??
            'Server error ($statusCode)';
        return ServerFailure(message, statusCode: statusCode);
      default:
        return const ServerFailure('Đã xảy ra lỗi không xác định.');
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

### Bước 3: Data Layer — Model & DataSource

```dart
// lib/features/posts/data/models/post_model.dart
import 'package:json_annotation/json_annotation.dart';
import '../../domain/entities/post.dart';

part 'post_model.g.dart';

@JsonSerializable()
class PostModel extends Post {
  const PostModel({
    required super.id,
    required super.userId,
    required super.title,
    required super.body,
  });

  factory PostModel.fromJson(Map<String, dynamic> json) =>
      _$PostModelFromJson(json);
  Map<String, dynamic> toJson() => _$PostModelToJson(this);
}
```

```dart
// lib/features/posts/data/datasources/post_remote_datasource.dart
import 'package:retrofit/retrofit.dart';
import 'package:dio/dio.dart';
import '../models/post_model.dart';

part 'post_remote_datasource.g.dart';

@RestApi()
abstract class PostRemoteDataSource {
  factory PostRemoteDataSource(Dio dio) = _PostRemoteDataSource;

  @GET('/posts')
  Future<List<PostModel>> getPosts();

  @GET('/posts/{id}')
  Future<PostModel> getPostById(@Path('id') int id);

  @POST('/posts')
  Future<PostModel> createPost(@Body() Map<String, dynamic> body);
}
```

### Bước 4: Data Layer — Repository Implementation

```dart
// lib/features/posts/data/repositories/post_repository_impl.dart
import 'package:fpdart/fpdart.dart';
import 'package:dio/dio.dart';
import '../../../../core/error/failures.dart';
import '../../../../core/error/network_error_mapper.dart';
import '../../../../core/network/network_info.dart';
import '../../domain/entities/post.dart';
import '../../domain/repositories/post_repository.dart';
import '../datasources/post_remote_datasource.dart';

class PostRepositoryImpl implements PostRepository {
  final PostRemoteDataSource _remoteDataSource;
  final NetworkInfo _networkInfo;

  PostRepositoryImpl(this._remoteDataSource, this._networkInfo);

  @override
  Future<Either<Failure, List<Post>>> getPosts() async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('Không có kết nối mạng.'));
    }

    try {
      final posts = await _remoteDataSource.getPosts();
      return Right(posts);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }

  @override
  Future<Either<Failure, Post>> getPostById(int id) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('Không có kết nối mạng.'));
    }

    try {
      final post = await _remoteDataSource.getPostById(id);
      return Right(post);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }

  @override
  Future<Either<Failure, Post>> createPost({
    required String title,
    required String body,
    required int userId,
  }) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('Không có kết nối mạng.'));
    }

    try {
      final post = await _remoteDataSource.createPost({
        'title': title,
        'body': body,
        'userId': userId,
      });
      return Right(post);
    } on DioException catch (e) {
      return Left(NetworkErrorMapper.mapDioException(e));
    } catch (e) {
      return Left(ServerFailure('Unexpected error: $e'));
    }
  }
}
```

### Bước 5: Domain Layer — UseCase

```dart
// lib/features/posts/domain/usecases/get_posts.dart
import 'package:fpdart/fpdart.dart';
import '../../../../core/error/failures.dart';
import '../entities/post.dart';
import '../repositories/post_repository.dart';

class GetPosts {
  final PostRepository _repository;
  GetPosts(this._repository);

  Future<Either<Failure, List<Post>>> call() => _repository.getPosts();
}

class GetPostById {
  final PostRepository _repository;
  GetPostById(this._repository);

  Future<Either<Failure, Post>> call(int id) => _repository.getPostById(id);
}

class CreatePost {
  final PostRepository _repository;
  CreatePost(this._repository);

  Future<Either<Failure, Post>> call({
    required String title,
    required String body,
    required int userId,
  }) => _repository.createPost(title: title, body: body, userId: userId);
}
```

### Bước 6: Presentation — Sử dụng trong UI

```dart
// Ví dụ với Cubit
class PostCubit extends Cubit<PostState> {
  final GetPosts _getPosts;

  PostCubit(this._getPosts) : super(const PostState.initial());

  Future<void> loadPosts() async {
    emit(const PostState.loading());

    final result = await _getPosts();

    result.fold(
      (failure) => emit(PostState.error(failure.message)),
      (posts) => emit(PostState.loaded(posts)),
    );
  }
}

// PostState với freezed
@freezed
class PostState with _$PostState {
  const factory PostState.initial() = _Initial;
  const factory PostState.loading() = _Loading;
  const factory PostState.loaded(List<Post> posts) = _Loaded;
  const factory PostState.error(String message) = _Error;
}
```

### Full Flow Diagram

```
┌─────────┐    ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌────────┐
│   UI     │───▶│  Cubit   │───▶│   UseCase    │───▶│  Repository  │───▶│  API   │
│(Widget)  │    │(PostCubit│    │(GetPosts)    │    │  (Impl)      │    │(Server)│
│          │◀───│)         │◀───│              │◀───│              │◀───│        │
└─────────┘    └──────────┘    └──────────────┘    └──────────────┘    └────────┘
                                                          │
   PostState     Either<       Either<           try-catch DioException
   .loaded()     Failure,      Failure,          → Left(Failure)
   .error()      List<Post>>   List<Post>>       → Right(List<Post>)
```

> **💡 Từ Frontend:** Đây là version có cấu trúc của React Query + service layer. Rõ ràng hơn "gọi fetch trong component" nhưng cần nhiều boilerplate hơn. Trade-off: **maintainability vs simplicity**.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_network_layer
cd vidu_network_layer
flutter pub add dio retrofit json_annotation fpdart flutter_bloc freezed_annotation
flutter pub add dev:retrofit_generator dev:json_serializable dev:build_runner dev:freezed
# Tạo cấu trúc thư mục core/ và features/posts/ theo code trên, rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ build_runner generate post_model.g.dart, post_remote_datasource.g.dart
✅ UI hiển thị danh sách posts từ jsonplaceholder API
✅ Khi mất mạng → Left(NetworkFailure) → UI hiện thông báo lỗi
✅ Khi server trả 500 → Left(ServerFailure) → UI hiện thông báo lỗi
✅ Flow: UI → Cubit → UseCase → Repository → DataSource → API
```

---

## VD6: 🤖 AI Gen → Review — Dio Interceptor Chain 🟢

> **Mục đích:** Luyện workflow "AI gen Dio interceptors → bạn review order + race condition + security → fix"

> **Liên quan tới:** [2. Interceptors — Middleware cho HTTP](01-ly-thuyet.md#2-interceptors--middleware-cho-http)

### Bước 1: Prompt cho AI

```text
Tạo Dio interceptor chain cho Flutter app:
1. AuthInterceptor: thêm Bearer token header.
2. LoggingInterceptor: log request/response.
3. ErrorInterceptor: map DioException → custom Failure.
Output: 3 interceptor classes.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Interceptor order** | Auth → Log → Error? (sai order = token không log, error không có context) |
| 2 | **Token source** | Lấy token từ SecureStorage? (không phải SharedPreferences = insecure) |
| 3 | **Error mapping** | Tất cả DioExceptionType được handle? (connectionTimeout, receiveTimeout, badResponse...) |
| 4 | **Async handler** | `handler.next()` được gọi? (quên = request bị treo) |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Sai interceptor order
dio.interceptors.addAll([
  ErrorInterceptor(),   // Error trước Auth!
  AuthInterceptor(),    // Token chưa được thêm khi Error chạy
  LoggingInterceptor(),
]);

// ✅ FIX: Đúng order
dio.interceptors.addAll([
  AuthInterceptor(),    // 1. Thêm token
  LoggingInterceptor(), // 2. Log request (đã có token)
  ErrorInterceptor(),   // 3. Map error (cuối cùng)
]);
```

```dart
// ❌ LỖI 2: Token refresh không handle race condition
Future<void> onError(DioException err, handler) async {
  if (err.response?.statusCode == 401) {
    await refreshToken(); // 3 requests cùng refresh → 3 lần call!
    // ...
  }
}

// ✅ FIX: Dùng lock/Completer
Completer<String>? _refreshCompleter;
bool _isRefreshing = false;

Future<void> onError(DioException err, handler) async {
  if (err.response?.statusCode == 401) {
    if (_isRefreshing) {
      // Wait for ongoing refresh
      final token = await _refreshCompleter!.future;
      // Retry with new token
    } else {
      _isRefreshing = true;
      _refreshCompleter = Completer<String>();
      final newToken = await refreshToken();
      _refreshCompleter!.complete(newToken);
      _isRefreshing = false;
    }
  }
}
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết interceptor order ảnh hưởng behavior (Auth trước Error)
- ✅ Nhận ra AI không handle race condition trong token refresh
- ✅ Verify token storage security (SecureStorage, không phải SharedPreferences)
```
