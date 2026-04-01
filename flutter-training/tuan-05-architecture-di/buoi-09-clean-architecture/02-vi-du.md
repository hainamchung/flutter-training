# Buổi 09: Clean Architecture trong Flutter — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> Tất cả ví dụ xây dựng quanh feature **User Management** để thấy rõ cách 3 layer kết nối.

---

## Ví dụ 1: Domain Layer — Entity + Use Case + Repository Interface 🔴

> **Liên quan tới:** [2. Domain Layer — Trái tim của ứng dụng](01-ly-thuyet.md#2-domain-layer--trái-tim-của-ứng-dụng)

### 1.1 Entity

```dart
// lib/features/user/domain/entities/user.dart

/// Entity: Pure Dart class — không import Flutter hay package nào
class User {
  final String id;
  final String name;
  final String email;
  final bool isActive;
  final DateTime createdAt;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.isActive,
    required this.createdAt,
  });

  /// Business rule: kiểm tra email hợp lệ
  bool get hasValidEmail => email.contains('@') && email.contains('.');

  /// Computed property
  String get displayName => name.isNotEmpty ? name : email.split('@').first;

  /// Business rule: account mới (< 30 ngày)
  bool get isNewAccount =>
      DateTime.now().difference(createdAt).inDays < 30;

  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is User &&
          runtimeType == other.runtimeType &&
          id == other.id;

  @override
  int get hashCode => id.hashCode;
}
```

### 1.2 Repository Interface

```dart
// lib/features/user/domain/repositories/user_repository.dart

import '../entities/user.dart';

/// Abstract class — chỉ định nghĩa contract
/// Data layer sẽ implement interface này
abstract class UserRepository {
  /// Lấy user theo ID
  Future<User> getUserById(String id);

  /// Lấy tất cả users
  Future<List<User>> getAllUsers();

  /// Tạo user mới
  Future<User> createUser({
    required String name,
    required String email,
  });

  /// Cập nhật user
  Future<User> updateUser(User user);

  /// Xóa user
  Future<void> deleteUser(String id);
}
```

### 1.3 Use Cases

```dart
// lib/features/user/domain/usecases/get_user.dart

import '../entities/user.dart';
import '../repositories/user_repository.dart';

/// Lấy thông tin 1 user theo ID
class GetUserUseCase {
  final UserRepository repository;

  GetUserUseCase(this.repository);

  /// Dùng call() để có thể gọi: getUserUseCase('123')
  Future<User> call(String userId) {
    return repository.getUserById(userId);
  }
}
```

```dart
// lib/features/user/domain/usecases/get_active_users.dart

import '../entities/user.dart';
import '../repositories/user_repository.dart';

/// Lấy danh sách users đang active
/// Business logic: filter + sort nằm ở Use Case, KHÔNG ở UI
class GetActiveUsersUseCase {
  final UserRepository repository;

  GetActiveUsersUseCase(this.repository);

  Future<List<User>> call() async {
    final allUsers = await repository.getAllUsers();

    return allUsers
        .where((user) => user.isActive)
        .toList()
      ..sort((a, b) => a.name.compareTo(b.name));
  }
}
```

```dart
// lib/features/user/domain/usecases/create_user.dart

import '../entities/user.dart';
import '../repositories/user_repository.dart';

/// Tạo user mới với validation
class CreateUserUseCase {
  final UserRepository repository;

  CreateUserUseCase(this.repository);

  Future<User> call({
    required String name,
    required String email,
  }) {
    // Business validation — nằm trong Use Case
    if (name.trim().isEmpty) {
      throw ArgumentError('Tên không được để trống');
    }
    if (name.trim().length < 2) {
      throw ArgumentError('Tên phải có ít nhất 2 ký tự');
    }
    if (!email.contains('@') || !email.contains('.')) {
      throw ArgumentError('Email không hợp lệ');
    }

    return repository.createUser(
      name: name.trim(),
      email: email.trim().toLowerCase(),
    );
  }
}
```

### 📌 Điểm cần chú ý VD1

```
✅ Không có import 'package:flutter/...'
✅ Không có import package nào (http, sqflite, shared_preferences...)
✅ Entity immutable (final fields, const constructor)
✅ Use Case nhận Repository qua constructor (Dependency Injection)
✅ Use Case chứa business validation
✅ Repository là abstract class (interface)
```

- 🔗 **FE tương đương:** Use Case ≈ custom hook chứa business logic (`useGetUsers`) — nhưng Flutter Use Case là class với method `call()`, không phải function/hook.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_clean_arch
cd vidu_clean_arch
mkdir -p lib/features/user/domain/{entities,usecases,repositories}
# Copy code Entity, Use Cases, Repository interface vào thư mục tương ứng
dart analyze lib/features/user/domain/
```

### 📋 Kết quả mong đợi

```
✅ dart analyze: No issues found!
✅ Entity, Use Case, Repository interface compile thành công với pure Dart
✅ Không có import 'package:flutter/...' hay third-party package trong domain/
```

---

## Ví dụ 2: Data Layer — Model + Repository Impl + DataSource 🔴

> **Liên quan tới:** [3. Data Layer — Nơi lấy và lưu trữ dữ liệu](01-ly-thuyet.md#3-data-layer--nơi-lấy-và-lưu-trữ-dữ-liệu)

### 2.1 Model (DTO)

```dart
// lib/features/user/data/models/user_model.dart

import 'dart:convert';
import '../../domain/entities/user.dart';

/// Model: extends Entity + thêm serialization
/// Biết cách chuyển đổi JSON ↔ Object
class UserModel extends User {
  const UserModel({
    required super.id,
    required super.name,
    required super.email,
    required super.isActive,
    required super.createdAt,
  });

  /// Tạo UserModel từ JSON (API response thường dùng snake_case)
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      isActive: json['is_active'] as bool,
      createdAt: DateTime.parse(json['created_at'] as String),
    );
  }

  /// Chuyển thành JSON để gửi lên API
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'is_active': isActive,
      'created_at': createdAt.toIso8601String(),
    };
  }

  /// Tạo UserModel từ Entity (khi cần convert ngược)
  factory UserModel.fromEntity(User user) {
    return UserModel(
      id: user.id,
      name: user.name,
      email: user.email,
      isActive: user.isActive,
      createdAt: user.createdAt,
    );
  }

  /// Parse list từ JSON string
  static List<UserModel> fromJsonList(String jsonString) {
    final List<dynamic> data = json.decode(jsonString) as List;
    return data
        .map((item) => UserModel.fromJson(item as Map<String, dynamic>))
        .toList();
  }
}
```

### 2.2 Remote Data Source

```dart
// lib/features/user/data/datasources/user_remote_data_source.dart

import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/user_model.dart';

/// Interface cho Remote Data Source
abstract class UserRemoteDataSource {
  Future<UserModel> getUserById(String id);
  Future<List<UserModel>> getAllUsers();
  Future<UserModel> createUser({required String name, required String email});
  Future<void> deleteUser(String id);
}

/// Implementation — gọi REST API thực tế
class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  final http.Client client;
  final String baseUrl;

  UserRemoteDataSourceImpl({
    required this.client,
    this.baseUrl = 'https://api.example.com',
  });

  @override
  Future<UserModel> getUserById(String id) async {
    final response = await client.get(
      Uri.parse('$baseUrl/users/$id'),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body) as Map<String, dynamic>;
      return UserModel.fromJson(data);
    } else if (response.statusCode == 404) {
      throw UserNotFoundException('User with id $id not found');
    } else {
      throw ServerException(
        'Server error: ${response.statusCode}',
      );
    }
  }

  @override
  Future<List<UserModel>> getAllUsers() async {
    final response = await client.get(
      Uri.parse('$baseUrl/users'),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 200) {
      final List<dynamic> data = json.decode(response.body) as List;
      return data
          .map((item) => UserModel.fromJson(item as Map<String, dynamic>))
          .toList();
    } else {
      throw ServerException(
        'Failed to load users: ${response.statusCode}',
      );
    }
  }

  @override
  Future<UserModel> createUser({
    required String name,
    required String email,
  }) async {
    final response = await client.post(
      Uri.parse('$baseUrl/users'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({'name': name, 'email': email}),
    );

    if (response.statusCode == 201) {
      final data = json.decode(response.body) as Map<String, dynamic>;
      return UserModel.fromJson(data);
    } else {
      throw ServerException(
        'Failed to create user: ${response.statusCode}',
      );
    }
  }

  @override
  Future<void> deleteUser(String id) async {
    final response = await client.delete(
      Uri.parse('$baseUrl/users/$id'),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode != 200 && response.statusCode != 204) {
      throw ServerException(
        'Failed to delete user: ${response.statusCode}',
      );
    }
  }
}

// Custom exceptions
class ServerException implements Exception {
  final String message;
  ServerException(this.message);

  @override
  String toString() => 'ServerException: $message';
}

class UserNotFoundException implements Exception {
  final String message;
  UserNotFoundException(this.message);

  @override
  String toString() => 'UserNotFoundException: $message';
}
```

### 2.3 Local Data Source

```dart
// lib/features/user/data/datasources/user_local_data_source.dart

import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';
import '../models/user_model.dart';

/// Interface cho Local Data Source
abstract class UserLocalDataSource {
  Future<List<UserModel>> getCachedUsers();
  Future<void> cacheUsers(List<UserModel> users);
  Future<void> clearCache();
}

/// Implementation — dùng SharedPreferences để cache
class UserLocalDataSourceImpl implements UserLocalDataSource {
  final SharedPreferences prefs;
  static const String _cachedUsersKey = 'CACHED_USERS';

  UserLocalDataSourceImpl({required this.prefs});

  @override
  Future<List<UserModel>> getCachedUsers() async {
    final jsonString = prefs.getString(_cachedUsersKey);

    if (jsonString != null) {
      return UserModel.fromJsonList(jsonString);
    }

    throw CacheException('No cached users found');
  }

  @override
  Future<void> cacheUsers(List<UserModel> users) async {
    final jsonList = users.map((u) => u.toJson()).toList();
    final jsonString = json.encode(jsonList);
    await prefs.setString(_cachedUsersKey, jsonString);
  }

  @override
  Future<void> clearCache() async {
    await prefs.remove(_cachedUsersKey);
  }
}

class CacheException implements Exception {
  final String message;
  CacheException(this.message);

  @override
  String toString() => 'CacheException: $message';
}
```

### 2.4 Repository Implementation

```dart
// lib/features/user/data/repositories/user_repository_impl.dart

import '../../domain/entities/user.dart';
import '../../domain/repositories/user_repository.dart';
import '../datasources/user_remote_data_source.dart';
import '../datasources/user_local_data_source.dart';
import '../models/user_model.dart';

/// Repository implementation — kết nối DataSources với Domain
///
/// Quyết định lấy data từ đâu:
/// - Online → Remote + cache kết quả
/// - Offline → Local (cached data)
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;
  final UserLocalDataSource localDataSource;

  UserRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });

  @override
  Future<User> getUserById(String id) async {
    // Lấy từ remote (luôn cần data mới nhất cho single user)
    return await remoteDataSource.getUserById(id);
  }

  @override
  Future<List<User>> getAllUsers() async {
    try {
      // 1. Thử lấy từ API
      final remoteUsers = await remoteDataSource.getAllUsers();

      // 2. Cache lại cho offline use
      await localDataSource.cacheUsers(remoteUsers);

      // 3. Return (UserModel extends User nên trả về List<User> OK)
      return remoteUsers;
    } catch (e) {
      // 4. Nếu không có mạng → lấy từ cache
      try {
        return await localDataSource.getCachedUsers();
      } catch (_) {
        // Không có cache → throw error gốc
        rethrow;
      }
    }
  }

  @override
  Future<User> createUser({
    required String name,
    required String email,
  }) async {
    final newUser = await remoteDataSource.createUser(
      name: name,
      email: email,
    );

    // Invalidate cache sau khi tạo user mới
    await localDataSource.clearCache();

    return newUser;
  }

  @override
  Future<User> updateUser(User user) {
    // TODO: Implement khi có API endpoint
    throw UnimplementedError('updateUser not yet implemented');
  }

  @override
  Future<void> deleteUser(String id) async {
    await remoteDataSource.deleteUser(id);
    // Invalidate cache
    await localDataSource.clearCache();
  }
}
```

### 📌 Điểm cần chú ý VD2

```
✅ UserModel extends User (Entity) — thêm fromJson/toJson
✅ Data Source tách Remote vs Local — mỗi cái có interface riêng
✅ Repository impl quyết định lấy data từ đâu (caching strategy)
✅ Repository impl chỉ return User (Entity), KHÔNG return UserModel ra ngoài
✅ Import từ domain/ (entities, repositories) — đúng Dependency Rule
```

- 🔗 **FE tương đương:** Tương tự API service abstraction (`userService.ts`) — nhưng Flutter tách rõ abstract interface + concrete implementation, FE thường gộp trong 1 file.

### ▶️ Chạy ví dụ

```bash
# Tiếp tục từ project vidu_clean_arch
flutter pub add http shared_preferences
mkdir -p lib/features/user/data/{models,datasources,repositories}
# Copy code Model, DataSources, Repository impl vào thư mục tương ứng
dart analyze lib/features/user/data/
```

### 📋 Kết quả mong đợi

```
✅ UserModel extends User (Entity) — fromJson/toJson hoạt động
✅ DataSource gọi API và parse response thành Model
✅ Repository impl quyết định remote vs local data source
```

---

## Ví dụ 3: Presentation Layer — Cubit + States + Widget 🔴

> **Liên quan tới:** [4. Presentation Layer — Nơi hiển thị và tương tác](01-ly-thuyet.md#4-presentation-layer--nơi-hiển-thị-và-tương-tác)

### 3.1 States

```dart
// lib/features/user/presentation/cubit/user_state.dart

import '../../domain/entities/user.dart';

/// Sealed class pattern cho states (Dart 3+)
sealed class UserState {
  const UserState();
}

/// Trạng thái ban đầu
class UserInitial extends UserState {
  const UserInitial();
}

/// Đang tải dữ liệu
class UserLoading extends UserState {
  const UserLoading();
}

/// Đã tải xong danh sách users
class UsersLoaded extends UserState {
  final List<User> users;
  const UsersLoaded(this.users);
}

/// Đã tải xong 1 user
class UserDetailLoaded extends UserState {
  final User user;
  const UserDetailLoaded(this.user);
}

/// Tạo user thành công
class UserCreated extends UserState {
  final User user;
  const UserCreated(this.user);
}

/// Có lỗi xảy ra
class UserError extends UserState {
  final String message;
  const UserError(this.message);
}
```

### 3.2 Cubit

```dart
// lib/features/user/presentation/cubit/user_cubit.dart

import 'package:flutter_bloc/flutter_bloc.dart';
import '../../domain/usecases/get_user.dart';
import '../../domain/usecases/get_active_users.dart';
import '../../domain/usecases/create_user.dart';
import 'user_state.dart';

/// UserCubit — chỉ biết về Use Cases và Entities
/// KHÔNG import bất kỳ thứ gì từ Data layer
class UserCubit extends Cubit<UserState> {
  final GetUserUseCase _getUserUseCase;
  final GetActiveUsersUseCase _getActiveUsersUseCase;
  final CreateUserUseCase _createUserUseCase;

  UserCubit({
    required GetUserUseCase getUserUseCase,
    required GetActiveUsersUseCase getActiveUsersUseCase,
    required CreateUserUseCase createUserUseCase,
  })  : _getUserUseCase = getUserUseCase,
        _getActiveUsersUseCase = getActiveUsersUseCase,
        _createUserUseCase = createUserUseCase,
        super(const UserInitial());

  /// Tải danh sách active users
  Future<void> loadActiveUsers() async {
    emit(const UserLoading());

    try {
      // Gọi Use Case — không biết data đến từ đâu
      final users = await _getActiveUsersUseCase();
      emit(UsersLoaded(users));
    } catch (e) {
      emit(UserError('Không thể tải danh sách users: ${e.toString()}'));
    }
  }

  /// Tải chi tiết 1 user
  Future<void> loadUser(String id) async {
    emit(const UserLoading());

    try {
      final user = await _getUserUseCase(id);
      emit(UserDetailLoaded(user));
    } catch (e) {
      emit(UserError('Không thể tải thông tin user: ${e.toString()}'));
    }
  }

  /// Tạo user mới
  Future<void> createUser({
    required String name,
    required String email,
  }) async {
    emit(const UserLoading());

    try {
      final user = await _createUserUseCase(name: name, email: email);
      emit(UserCreated(user));

      // Reload danh sách sau khi tạo
      await loadActiveUsers();
    } on ArgumentError catch (e) {
      // Business validation error từ Use Case
      emit(UserError(e.message));
    } catch (e) {
      emit(UserError('Không thể tạo user: ${e.toString()}'));
    }
  }
}
```

### 3.3 User List Page

```dart
// lib/features/user/presentation/pages/user_list_page.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../domain/entities/user.dart';
import '../cubit/user_cubit.dart';
import '../cubit/user_state.dart';
import '../widgets/user_card.dart';
import 'create_user_page.dart';

class UserListPage extends StatefulWidget {
  const UserListPage({super.key});

  @override
  State<UserListPage> createState() => _UserListPageState();
}

class _UserListPageState extends State<UserListPage> {
  @override
  void initState() {
    super.initState();
    // Tải users khi mở trang
    context.read<UserCubit>().loadActiveUsers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Users'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => context.read<UserCubit>().loadActiveUsers(),
          ),
        ],
      ),
      body: BlocConsumer<UserCubit, UserState>(
        listener: (context, state) {
          // Side effects: show snackbar khi tạo user thành công
          if (state is UserCreated) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text('Đã tạo user: ${state.user.displayName}'),
              ),
            );
          }
        },
        builder: (context, state) {
          // Loading
          if (state is UserLoading) {
            return const Center(child: CircularProgressIndicator());
          }

          // Error
          if (state is UserError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, size: 48, color: Colors.red),
                  const SizedBox(height: 16),
                  Text(state.message, textAlign: TextAlign.center),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () =>
                        context.read<UserCubit>().loadActiveUsers(),
                    child: const Text('Thử lại'),
                  ),
                ],
              ),
            );
          }

          // Loaded
          if (state is UsersLoaded) {
            if (state.users.isEmpty) {
              return const Center(child: Text('Chưa có user nào'));
            }

            return ListView.builder(
              padding: const EdgeInsets.all(16),
              itemCount: state.users.length,
              itemBuilder: (_, index) => UserCard(user: state.users[index]),
            );
          }

          // Initial
          return const Center(child: Text('Nhấn refresh để tải users'));
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.of(context).push(
            MaterialPageRoute(builder: (_) => const CreateUserPage()),
          );
        },
        child: const Icon(Icons.person_add),
      ),
    );
  }
}
```

### 3.4 Reusable Widget

```dart
// lib/features/user/presentation/widgets/user_card.dart

import 'package:flutter/material.dart';
import '../../domain/entities/user.dart';

class UserCard extends StatelessWidget {
  final User user;

  const UserCard({super.key, required this.user});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor:
              user.isActive ? Colors.green.shade100 : Colors.grey.shade100,
          child: Text(
            user.displayName[0].toUpperCase(),
            style: TextStyle(
              color: user.isActive ? Colors.green.shade700 : Colors.grey,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        title: Text(user.displayName),
        subtitle: Text(user.email),
        trailing: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              user.isActive ? Icons.check_circle : Icons.cancel,
              color: user.isActive ? Colors.green : Colors.red,
              size: 20,
            ),
            if (user.isNewAccount)
              Container(
                margin: const EdgeInsets.only(top: 4),
                padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                decoration: BoxDecoration(
                  color: Colors.blue.shade50,
                  borderRadius: BorderRadius.circular(4),
                ),
                child: const Text(
                  'NEW',
                  style: TextStyle(fontSize: 10, color: Colors.blue),
                ),
              ),
          ],
        ),
      ),
    );
  }
}
```

### 📌 Điểm cần chú ý VD3

```
✅ Cubit chỉ import từ domain/ (Use Cases + Entities)
✅ Widget dùng Entity (User), KHÔNG dùng Model (UserModel)
✅ States dùng sealed class (Dart 3+ pattern matching)
✅ BlocConsumer cho cả listener (side effects) + builder (UI)
✅ Widget tách riêng (UserCard) để reuse
```

### ▶️ Chạy ví dụ

```bash
# Tiếp tục từ project vidu_clean_arch
flutter pub add flutter_bloc
mkdir -p lib/features/user/presentation/{cubit,pages,widgets}
# Copy toàn bộ code VD1 → VD4 (Wiring) vào cấu trúc thư mục, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ App hiển thị danh sách Users từ API (hoặc cached data khi offline)
✅ CircularProgressIndicator khi đang loading
✅ Error message + nút "Thử lại" khi API fail
✅ FAB mở trang tạo user mới
```

---

## Ví dụ 4: Wiring — Kết nối tất cả Layers 🔴

> **Liên quan tới:** [5. Dependency Rule — Quy tắc quan trọng nhất](01-ly-thuyet.md#5-dependency-rule--quy-tắc-quan-trọng-nhất)

### 4.1 Manual Dependency Injection

Ở buổi này chúng ta dùng **Manual DI** — tự tạo và inject dependencies. Buổi 10 sẽ học `get_it` + `injectable` để tự động hóa.

```dart
// lib/injection_container.dart

import 'package:http/http.dart' as http;
import 'package:shared_preferences/shared_preferences.dart';

// Domain
import 'features/user/domain/repositories/user_repository.dart';
import 'features/user/domain/usecases/get_user.dart';
import 'features/user/domain/usecases/get_active_users.dart';
import 'features/user/domain/usecases/create_user.dart';

// Data
import 'features/user/data/datasources/user_remote_data_source.dart';
import 'features/user/data/datasources/user_local_data_source.dart';
import 'features/user/data/repositories/user_repository_impl.dart';

// Presentation
import 'features/user/presentation/cubit/user_cubit.dart';

/// Manual DI — tạo tất cả dependencies theo đúng thứ tự
class InjectionContainer {
  // Singleton
  static final InjectionContainer _instance = InjectionContainer._();
  factory InjectionContainer() => _instance;
  InjectionContainer._();

  // Dependencies
  late final UserCubit userCubit;

  /// Khởi tạo tất cả dependencies
  /// Gọi trong main() TRƯỚC runApp()
  Future<void> init() async {
    // External
    final prefs = await SharedPreferences.getInstance();
    final client = http.Client();

    // Data Sources
    final remoteDataSource = UserRemoteDataSourceImpl(client: client);
    final localDataSource = UserLocalDataSourceImpl(prefs: prefs);

    // Repository (Data layer implements Domain interface)
    final UserRepository userRepository = UserRepositoryImpl(
      remoteDataSource: remoteDataSource,
      localDataSource: localDataSource,
    );

    // Use Cases (Domain layer)
    final getUserUseCase = GetUserUseCase(userRepository);
    final getActiveUsersUseCase = GetActiveUsersUseCase(userRepository);
    final createUserUseCase = CreateUserUseCase(userRepository);

    // Cubit (Presentation layer)
    userCubit = UserCubit(
      getUserUseCase: getUserUseCase,
      getActiveUsersUseCase: getActiveUsersUseCase,
      createUserUseCase: createUserUseCase,
    );
  }
}
```

### 4.2 Main.dart

```dart
// lib/main.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'injection_container.dart';
import 'features/user/presentation/pages/user_list_page.dart';
import 'features/user/presentation/cubit/user_cubit.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Khởi tạo dependencies
  final di = InjectionContainer();
  await di.init();

  runApp(MyApp(di: di));
}

class MyApp extends StatelessWidget {
  final InjectionContainer di;

  const MyApp({super.key, required this.di});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider<UserCubit>.value(value: di.userCubit),
      ],
      child: MaterialApp(
        title: 'Clean Architecture Demo',
        theme: ThemeData(
          colorSchemeSeed: Colors.blue,
          useMaterial3: true,
        ),
        home: const UserListPage(),
      ),
    );
  }
}
```

### 4.3 Luồng data hoàn chỉnh

```
Người dùng mở app
    │
    ▼
main.dart
    │ InjectionContainer.init()
    │ Tạo: DataSources → Repository → UseCases → Cubit
    │
    ▼
UserListPage (initState)
    │ context.read<UserCubit>().loadActiveUsers()
    │
    ▼
UserCubit.loadActiveUsers()
    │ emit(UserLoading)
    │ await _getActiveUsersUseCase()
    │
    ▼
GetActiveUsersUseCase.call()
    │ await repository.getAllUsers()
    │ filter active + sort by name
    │
    ▼
UserRepositoryImpl.getAllUsers()
    │ try: remoteDataSource.getAllUsers()  ──▶  HTTP GET /users
    │                                            │
    │ cache: localDataSource.cacheUsers()        │ JSON response
    │                                            ▼
    │ catch: localDataSource.getCachedUsers()    UserModel.fromJson()
    │
    ▼
Return List<User>
    │
    ▼
UserCubit
    │ emit(UsersLoaded(users))
    │
    ▼
BlocBuilder rebuild
    │ ListView.builder + UserCard
    │
    ▼
UI hiển thị danh sách users ✅
```

### 📌 Điểm cần chú ý VD4

```
✅ Dependencies được tạo từ trong ra ngoài: DataSource → Repo → UseCase → Cubit
✅ UserRepository (abstract) được assign bằng UserRepositoryImpl (concrete)
✅ Cubit KHÔNG biết về UserRepositoryImpl — chỉ biết Use Cases
✅ Data flows: UI → Cubit → UseCase → Repo(impl) → DataSource → API
✅ Manual DI — ở buổi 10 sẽ thay bằng get_it + injectable
```

---

## VD5: 🤖 AI Gen → Review — Clean Architecture Scaffold 🟢

> **Mục đích:** Luyện workflow "AI gen architecture scaffold → bạn review Dependency Rule → fix violations"

> **Liên quan tới:** [6. Cấu trúc Folder chuẩn — Feature-first Approach](01-ly-thuyet.md#6-cấu-trúc-folder-chuẩn--feature-first-approach)

### Bước 1: Prompt cho AI

```text
Tạo Clean Architecture scaffold cho "Notes" feature trong Flutter.
3 layers: Domain (NoteEntity, NoteRepository abstract, GetNotes use case),
Data (NoteModel DTO, NoteRemoteDataSource, NoteRepositoryImpl),
Presentation (NotesScreen widget).
Output: folder structure + class stubs.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Dependency Rule** | Domain folder có import `data/` hoặc `presentation/`? (= violation!) |
| 2 | **Abstract vs Concrete** | Repository ở Domain là `abstract class`? Impl ở Data `implements` nó? |
| 3 | **DTO vs Entity** | `NoteModel` (Data) khác `NoteEntity` (Domain)? Có `toDomain()` method? |
| 4 | **Use Case pattern** | `GetNotes` class có single `call()` method trả `Either<Failure, T>`? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Domain import Data (vi phạm Dependency Rule)
// File: domain/usecases/get_notes.dart
import '../../data/repositories/note_repository_impl.dart'; // WRONG!
// Domain KHÔNG ĐƯỢC biết Data layer tồn tại

// ✅ FIX: Domain chỉ import chính nó
import '../repositories/note_repository.dart'; // abstract class trong Domain

class GetNotes {
  final NoteRepository repository; // abstract, không phải impl
  // ...
}
```

```dart
// ❌ LỖI 2: Dùng chung Model cho cả API response và Domain
class Note {
  final String id;
  final String title;
  final String created_at;  // snake_case từ API lộ ra Domain!
}

// ✅ FIX: Tách DTO (Data) và Entity (Domain)
// Data: NoteModel (fromJson, toJson, created_at)
// Domain: NoteEntity (createdAt — camelCase, business fields only)
// Mapping: NoteModel.toDomain() → NoteEntity
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Kiểm tra Dependency Rule bằng cách đọc import statements
- ✅ Phân biệt DTO (Data layer) vs Entity (Domain layer)
- ✅ Hiểu tại sao abstract Repository cần ở Domain (Dependency Inversion)
