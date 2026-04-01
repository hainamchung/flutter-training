# Buổi 09: Clean Architecture trong Flutter — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 9/16** · **Thời lượng tự học:** ~2 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 07 hoặc 08 (ít nhất 1 state management)

## 1. Clean Architecture là gì? 🔴

### 1.1 Nguồn gốc

Clean Architecture được giới thiệu bởi **Robert C. Martin (Uncle Bob)** năm 2012. Đây không phải là một framework hay thư viện — mà là một **tập hợp nguyên tắc** để tổ chức code sao cho:

- **Độc lập với framework:** Business logic không bị ràng buộc vào Flutter, BLoC, hay bất kỳ thư viện nào
- **Dễ test:** Business logic có thể test mà không cần UI, database, hay server
- **Độc lập với UI:** UI có thể thay đổi mà không ảnh hưởng business logic
- **Độc lập với database/API:** Có thể đổi từ REST sang GraphQL mà domain logic không đổi

### 1.2 Tại sao cần Clean Architecture?

**Không có architecture (code spaghetti):**

```dart
// ❌ Mọi thứ trộn lẫn trong một Widget
class UserScreen extends StatefulWidget {
  @override
  _UserScreenState createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  List<Map<String, dynamic>> users = [];

  Future<void> loadUsers() async {
    // API call trực tiếp trong widget
    final response = await http.get(Uri.parse('https://api.example.com/users'));
    final data = jsonDecode(response.body);

    // Business logic trong widget
    final activeUsers = data.where((u) => u['isActive'] == true).toList();
    activeUsers.sort((a, b) => a['name'].compareTo(b['name']));

    setState(() {
      users = activeUsers;
    });
  }

  @override
  Widget build(BuildContext context) {
    // UI trộn lẫn với logic
    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (_, i) => ListTile(title: Text(users[i]['name'])),
    );
  }
}
```

**Vấn đề:**
- Không thể test business logic riêng biệt
- Thay đổi API → sửa widget
- Nhiều người cùng sửa 1 file → conflict liên tục
- Code phình to, khó đọc, khó maintain

**Với Clean Architecture:**
- Business logic tách riêng → test dễ dàng
- Mỗi người phụ trách 1 layer → giảm conflict
- Đổi API? Chỉ sửa Data layer. Đổi UI? Chỉ sửa Presentation layer
- **Rõ ràng ai làm gì, ở đâu**

### SOLID Principles (tóm tắt)

> SOLID là nền tảng của Clean Architecture. Hiểu SOLID = hiểu tại sao chúng ta chia layer.

| Principle | Tên đầy đủ | Ý nghĩa | Ví dụ Dart |
|-----------|-----------|---------|------------|
| **S** | Single Responsibility | Mỗi class chỉ làm 1 việc | `UserRepository` chỉ xử lý data, không chứa UI logic |
| **O** | Open/Closed | Mở rộng được, không sửa code cũ | Dùng abstract class + implementation mới |
| **L** | Liskov Substitution | Subclass thay thế được parent | `MockUserRepo implements UserRepo` hoạt động đúng |
| **I** | Interface Segregation | Interface nhỏ, chuyên biệt | Tách `AuthRepo` và `UserRepo` thay vì 1 `BigRepo` |
| **D** | Dependency Inversion | Depend on abstractions | UseCase nhận `UserRepository` (abstract), không phải `UserRepositoryImpl` |

Trong Clean Architecture, mỗi nguyên tắc SOLID được áp dụng trực tiếp:
- **S** → Mỗi layer có trách nhiệm riêng
- **O** → Thêm DataSource mới không cần sửa UseCase
- **L** → Mock thay thế real implementation trong testing
- **I** → Repository interface chỉ chứa methods cần thiết
- **D** → Domain layer không phụ thuộc Data layer

### 1.3 Mô hình vòng tròn đồng tâm

Uncle Bob's original model:

```
┌─────────────────────────────────────────────────┐
│                 Frameworks & Drivers             │
│    ┌─────────────────────────────────────────┐   │
│    │          Interface Adapters              │   │
│    │    ┌─────────────────────────────────┐   │   │
│    │    │         Use Cases               │   │   │
│    │    │    ┌─────────────────────────┐   │   │   │
│    │    │    │       Entities          │   │   │   │
│    │    │    │                         │   │   │   │
│    │    │    │   (Business Objects)    │   │   │   │
│    │    │    └─────────────────────────┘   │   │   │
│    │    │                                 │   │   │
│    │    │    (Application Logic)          │   │   │
│    │    └─────────────────────────────────┘   │   │
│    │                                         │   │
│    │    (Controllers, Presenters, Gateways)  │   │
│    └─────────────────────────────────────────┘   │
│                                                 │
│    (Flutter, HTTP, Database, ...)               │
└─────────────────────────────────────────────────┘

Dependency Rule: Mũi tên luôn hướng VÀO TRONG ──▶
```

### 1.4 Adapted cho Flutter — 3 Layers

Trong Flutter, mô hình thường được **đơn giản hóa** thành 3 layer:

```
┌──────────────────────────────────────────────────────┐
│                  PRESENTATION LAYER                   │
│         (Widgets, BLoC/Cubit, Pages, UI)             │
│    ┌──────────────────────────────────────────────┐   │
│    │              DATA LAYER                       │   │
│    │    (Models, Repo Impl, DataSources)          │   │
│    │    ┌──────────────────────────────────────┐   │   │
│    │    │          DOMAIN LAYER                │   │   │
│    │    │   (Entities, Use Cases, Repo Iface)  │   │   │
│    │    │                                      │   │   │
│    │    │    ❌ KHÔNG phụ thuộc gì bên ngoài    │   │   │
│    │    └──────────────────────────────────────┘   │   │
│    └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

| Layer | Chứa gì | Phụ thuộc |
|-------|---------|-----------|
| **Domain** | Entities, Use Cases, Repository interfaces | Không phụ thuộc gì |
| **Data** | Models, Repository impl, DataSources | Phụ thuộc Domain |
| **Presentation** | Widgets, State (BLoC/Cubit), Pages | Phụ thuộc Domain |

> **Lưu ý quan trọng:** Cả Data và Presentation đều phụ thuộc Domain, nhưng **không phụ thuộc lẫn nhau**.

> 🔗 **FE Bridge:** Clean Architecture concept **tương đồng** giữa FE và Flutter — cùng Dependency Rule, cùng layered approach. Nhưng **khác ở**: Flutter community **enforce strict hơn** — FE thường có `services/`, `hooks/`, `utils/` loosely organized, Flutter thường có `domain/`, `data/`, `presentation/` với boundaries rõ ràng.

---

## 2. Domain Layer — Trái tim của ứng dụng 🔴

Domain layer là layer **quan trọng nhất** — nó chứa business logic thuần túy và **KHÔNG phụ thuộc bất kỳ thứ gì bên ngoài** (không Flutter, không package, không framework).

### 2.1 Entities

Entity là **đối tượng business cốt lõi** — đại diện cho dữ liệu quan trọng của ứng dụng.

```dart
// ✅ Entity: pure Dart class, không import Flutter hay package nào
class User {
  final String id;
  final String name;
  final String email;
  final bool isActive;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.isActive,
  });
}
```

**Đặc điểm của Entity:**
- ✅ Pure Dart — không `import 'package:flutter/...'`
- ✅ Immutable (dùng `final` + `const` constructor)
- ✅ Chứa business rules cơ bản (validation, computed properties)
- ❌ KHÔNG có `fromJson`/`toJson` (đó là việc của Data layer)
- ❌ KHÔNG phụ thuộc third-party package

> 💡 **Thực tế dự án**: Trong production, nhiều team cho phép `freezed` trong domain entities để tận dụng `copyWith`, `==`, `toString`. Đây là trade-off: thêm dependency nhưng giảm boilerplate đáng kể. Nếu muốn domain hoàn toàn pure, dùng `freezed` chỉ trong Data layer (DTOs/Models).

```dart
// Entity có thể chứa business logic đơn giản
class User {
  final String id;
  final String name;
  final String email;
  final bool isActive;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.isActive,
  });

  // ✅ Business rule: valid email
  bool get hasValidEmail => email.contains('@') && email.contains('.');

  // ✅ Computed property
  String get displayName => name.isNotEmpty ? name : email;
}
```

### 2.2 Use Cases (Interactors)

Use Case đại diện cho **một hành động business duy nhất**. Mỗi use case có đúng 1 nhiệm vụ.

```dart
// Mỗi Use Case = 1 business operation
// Convention: dùng method call() để có thể gọi như function

class GetUserUseCase {
  final UserRepository repository;

  GetUserUseCase(this.repository);

  Future<User> call(String userId) {
    return repository.getUserById(userId);
  }
}
```

```dart
class GetActiveUsersUseCase {
  final UserRepository repository;

  GetActiveUsersUseCase(this.repository);

  Future<List<User>> call() async {
    final users = await repository.getAllUsers();
    return users.where((user) => user.isActive).toList();
  }
}
```

```dart
class CreateUserUseCase {
  final UserRepository repository;

  CreateUserUseCase(this.repository);

  Future<User> call(String name, String email) {
    // ✅ Business validation trong Use Case
    if (name.isEmpty) {
      throw ArgumentError('Name cannot be empty');
    }
    if (!email.contains('@')) {
      throw ArgumentError('Invalid email format');
    }

    return repository.createUser(name: name, email: email);
  }
}
```

**Nguyên tắc Use Case:**
- ✅ Mỗi use case = **1 hành động** (Single Responsibility)
- ✅ Dùng `call()` method — cho phép gọi `useCase(params)` thay vì `useCase.execute(params)`
- ✅ Chứa business validation / orchestration logic
- ❌ KHÔNG biết về UI hay database cụ thể
- ❌ KHÔNG gọi network trực tiếp

> **Khi nào cần Use Case?** Nếu logic chỉ là "lấy data và trả về" — Use Case vẫn có giá trị vì nó tạo một **abstraction layer** giữa Presentation và Data. Khi business logic phức tạp hơn, Use Case sẽ phát huy tác dụng.

### 2.3 Repository Interfaces

Repository interface định nghĩa **contract** — "Data layer phải cung cấp những gì cho Domain layer?"

```dart
// ✅ Abstract class — chỉ định nghĩa contract, không implement
abstract class UserRepository {
  Future<User> getUserById(String id);
  Future<List<User>> getAllUsers();
  Future<User> createUser({required String name, required String email});
  Future<void> deleteUser(String id);
  Future<User> updateUser(User user);
}
```

**Tại sao dùng abstract class?**
- Domain layer **định nghĩa** interface nhưng **không implement**
- Data layer sẽ implement interface này
- → Domain không biết data đến từ API, database, hay memory
- → Dễ dàng swap implementation (mock cho testing)

```
Domain layer:        abstract class UserRepository { ... }
                              ▲
                              │ implements
                              │
Data layer:          class UserRepositoryImpl implements UserRepository { ... }
```

### 2.4 Tóm tắt Domain Layer

```
domain/
├── entities/
│   └── user.dart              ← Pure Dart class
├── usecases/
│   ├── get_user.dart          ← GetUserUseCase
│   ├── get_active_users.dart  ← GetActiveUsersUseCase
│   └── create_user.dart       ← CreateUserUseCase
└── repositories/
    └── user_repository.dart   ← Abstract class (interface)
```

**Quy tắc vàng:** Nếu bạn xóa toàn bộ Flutter SDK, Data layer, và tất cả packages — Domain layer vẫn phải compile được với pure Dart.

> 🔗 **FE Bridge:** Entity ≈ TypeScript `interface`/Zod schema, Use Case ≈ Custom Hook hoặc Service function. Nhưng **khác ở**: Dart Entity thường là **immutable class** (với `copyWith`), không phải interface structural typing. Use Case = **single responsibility class** thay vì function — Flutter OOP-focused hơn FE functional approach.

---

## 3. Data Layer — Nơi lấy và lưu trữ dữ liệu 🔴

Data layer chịu trách nhiệm **lấy dữ liệu từ bên ngoài** (API, database, cache) và **chuyển đổi** thành Entity cho Domain layer sử dụng.

### 3.1 Models (DTOs)

Model là **phiên bản "data" của Entity** — biết cách serialize/deserialize.

```dart
import '../../../domain/entities/user.dart';

// Model extends hoặc tương ứng với Entity
// Thêm fromJson/toJson cho serialization
class UserModel extends User {
  const UserModel({
    required super.id,
    required super.name,
    required super.email,
    required super.isActive,
  });

  // ✅ Factory từ JSON (API response)
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      isActive: json['is_active'] as bool,
    );
  }

  // ✅ Chuyển thành JSON (gửi lên API)
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'is_active': isActive,
    };
  }
}
```

**Tại sao tách Entity và Model?**

| Entity (Domain) | Model (Data) |
|-----------------|--------------|
| Business object thuần túy | Data transfer object (DTO) |
| Không biết JSON | Biết `fromJson`/`toJson` |
| Field name theo business | Field name có thể khác (snake_case từ API) |
| Không thay đổi khi API thay đổi | Thay đổi khi API response thay đổi |

### 3.2 Data Sources

Data Source là nơi **thực sự gọi API hoặc truy vấn database**.

```dart
// Remote Data Source — gọi API
abstract class UserRemoteDataSource {
  Future<UserModel> getUserById(String id);
  Future<List<UserModel>> getAllUsers();
  Future<UserModel> createUser({required String name, required String email});
  Future<void> deleteUser(String id);
}

class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  final http.Client client;

  UserRemoteDataSourceImpl(this.client);

  @override
  Future<UserModel> getUserById(String id) async {
    final response = await client.get(
      Uri.parse('https://api.example.com/users/$id'),
    );

    if (response.statusCode == 200) {
      return UserModel.fromJson(jsonDecode(response.body));
    } else {
      throw ServerException('Failed to load user');
    }
  }

  @override
  Future<List<UserModel>> getAllUsers() async {
    final response = await client.get(
      Uri.parse('https://api.example.com/users'),
    );

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body);
      return data.map((json) => UserModel.fromJson(json)).toList();
    } else {
      throw ServerException('Failed to load users');
    }
  }

  // ... createUser, deleteUser
}
```

```dart
// Local Data Source — cache/database
abstract class UserLocalDataSource {
  Future<List<UserModel>> getCachedUsers();
  Future<void> cacheUsers(List<UserModel> users);
}

class UserLocalDataSourceImpl implements UserLocalDataSource {
  final SharedPreferences prefs;

  UserLocalDataSourceImpl(this.prefs);

  @override
  Future<List<UserModel>> getCachedUsers() async {
    final jsonString = prefs.getString('cached_users');
    if (jsonString != null) {
      final List<dynamic> data = jsonDecode(jsonString);
      return data.map((json) => UserModel.fromJson(json)).toList();
    }
    throw CacheException('No cached users');
  }

  @override
  Future<void> cacheUsers(List<UserModel> users) async {
    final jsonList = users.map((u) => u.toJson()).toList();
    await prefs.setString('cached_users', jsonEncode(jsonList));
  }
}
```

### 3.3 Repository Implementation

Repository impl **kết nối Data Sources với Domain layer** — implement interface từ Domain.

```dart
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;
  final UserLocalDataSource localDataSource;

  UserRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });

  @override
  Future<User> getUserById(String id) async {
    return await remoteDataSource.getUserById(id);
  }

  @override
  Future<List<User>> getAllUsers() async {
    try {
      // Thử lấy từ remote trước
      final users = await remoteDataSource.getAllUsers();
      // Cache lại
      await localDataSource.cacheUsers(users);
      return users;
    } catch (e) {
      // Nếu không có mạng, lấy từ cache
      return await localDataSource.getCachedUsers();
    }
  }

  @override
  Future<User> createUser({required String name, required String email}) {
    return remoteDataSource.createUser(name: name, email: email);
  }

  @override
  Future<void> deleteUser(String id) {
    return remoteDataSource.deleteUser(id);
  }

  @override
  Future<User> updateUser(User user) {
    // Convert Entity → Model nếu cần
    throw UnimplementedError();
  }
}
```

**Repository Pattern — Giá trị cốt lõi:**
- **Orchestration:** Quyết định lấy data từ remote hay local
- **Caching strategy:** Lưu cache khi có data mới
- **Error handling:** Xử lý lỗi mạng, fallback sang cache
- **Data mapping:** Chuyển đổi Model ↔ Entity

### 3.4 Tóm tắt Data Layer

```
data/
├── models/
│   └── user_model.dart          ← extends User Entity + fromJson/toJson
├── datasources/
│   ├── user_remote_data_source.dart  ← API calls
│   └── user_local_data_source.dart   ← Cache/DB
└── repositories/
    └── user_repository_impl.dart     ← implements UserRepository
```

> 🔗 **FE Bridge:** Repository pattern ≈ API Service abstraction trong FE — cùng mục đích abstract data source. Nhưng **khác ở**: Flutter Repository có **interface + implementation split** rõ ràng (`abstract class UserRepository` + `UserRepositoryImpl`). FE thường chỉ có 1 file `userService.ts` = cả interface lẫn implementation.

---

> 💼 **Gặp trong dự án:** Setup Data Layer cho feature mới (API + local cache), implement Repository Pattern đúng chuẩn, map API response DTO → Domain Entity, handle error từ multiple data sources
> 🤖 **Keywords bắt buộc trong prompt:** `Repository Pattern`, `DataSource abstraction`, `DTO vs Entity mapping`, `Either<Failure, Success>`, `try-catch → Result type`, `data source fallback`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Feature mới:** PM giao "User Profile" feature — cần fetch từ API, cache local, handle offline mode
- **DTO mapping:** API trả về `UserResponse` (snake_case, extra fields) → Domain cần `UserEntity` (clean, only business fields)
- **Error handling:** API fail → try local cache → nếu cache cũng fail → return Failure

**Tại sao cần các keyword trên:**
- **`Repository Pattern`** — AI phải implement abstract repository (Domain) + concrete implementation (Data)
- **`DataSource abstraction`** — Remote (API) và Local (Hive/SharedPreferences) là 2 DataSource riêng
- **`DTO vs Entity mapping`** — AI hay trộn lẫn API response model với Domain entity (vi phạm Dependency Rule)
- **`Either<Failure, Success>`** — dapartz package hoặc custom Result type, AI hay dùng try-catch thay vì explicit error types

**Prompt mẫu — Data Layer scaffold:**
```text
Tôi cần scaffold Data Layer cho "User Profile" feature theo Clean Architecture.
Tech stack: Flutter 3.x, dio ^5.x, hive ^2.x, dartz ^0.10.x (Either).
Components cần tạo:
1. UserRemoteDataSource: fetchUser(id) → UserModel (from API).
2. UserLocalDataSource: cacheUser(UserModel), getCachedUser(id) → UserModel?.
3. UserRepositoryImpl implements UserRepository (from Domain layer).
4. UserModel (DTO): fromJson, toJson — mapping snake_case API response.
5. UserModel → UserEntity extension (toDomain).
Constraints:
- Repository: try remote first → cache result → if remote fails → try local → if local fails → return Left(ServerFailure).
- UserModel KHÔNG import từ Domain layer (chỉ mapping method).
- Return type: Future<Either<Failure, UserEntity>> (dartz package).
- Error handling: catch DioException → ServerFailure, catch CacheException → CacheFailure.
Output: 4 files: user_remote_data_source.dart, user_local_data_source.dart, user_repository_impl.dart, user_model.dart.
```

**Expected Output:** AI gen 4 files Data Layer hoàn chỉnh.

⚠️ **Giới hạn AI hay mắc:** AI hay import Domain entities vào Data layer (vi phạm Dependency Rule). AI cũng hay quên cache result sau khi fetch thành công từ remote. AI hay dùng try-catch thay vì Either type.

</details>

---

## 4. Presentation Layer — Nơi hiển thị và tương tác 🟡

Presentation layer chịu trách nhiệm **hiển thị UI** và **xử lý tương tác người dùng**. Layer này dùng Use Cases từ Domain layer để lấy/thao tác dữ liệu.

### 4.1 State Management (BLoC/Cubit)

```dart
// Cubit — quản lý state cho User feature
class UserCubit extends Cubit<UserState> {
  final GetUserUseCase getUserUseCase;
  final GetActiveUsersUseCase getActiveUsersUseCase;

  UserCubit({
    required this.getUserUseCase,
    required this.getActiveUsersUseCase,
  }) : super(UserInitial());

  Future<void> loadUser(String id) async {
    emit(UserLoading());
    try {
      final user = await getUserUseCase(id);
      emit(UserLoaded(user));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }

  Future<void> loadActiveUsers() async {
    emit(UserLoading());
    try {
      final users = await getActiveUsersUseCase();
      emit(ActiveUsersLoaded(users));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }
}
```

```dart
// States
abstract class UserState {}

class UserInitial extends UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}

class ActiveUsersLoaded extends UserState {
  final List<User> users;
  ActiveUsersLoaded(this.users);
}

class UserError extends UserState {
  final String message;
  UserError(this.message);
}
```

**Lưu ý quan trọng:** Cubit/BLoC **chỉ biết về Use Cases và Entities** — KHÔNG biết về Models, DataSources, hay API.

### 4.2 Widgets / Pages

```dart
class UserListPage extends StatelessWidget {
  const UserListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Users')),
      body: BlocBuilder<UserCubit, UserState>(
        builder: (context, state) {
          if (state is UserLoading) {
            return const Center(child: CircularProgressIndicator());
          }

          if (state is ActiveUsersLoaded) {
            return ListView.builder(
              itemCount: state.users.length,
              itemBuilder: (_, index) {
                final user = state.users[index];
                return ListTile(
                  title: Text(user.displayName),
                  subtitle: Text(user.email),
                  trailing: Icon(
                    user.isActive ? Icons.check_circle : Icons.cancel,
                    color: user.isActive ? Colors.green : Colors.red,
                  ),
                );
              },
            );
          }

          if (state is UserError) {
            return Center(child: Text('Error: ${state.message}'));
          }

          return const Center(child: Text('Nhấn để tải users'));
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          context.read<UserCubit>().loadActiveUsers();
        },
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

### 4.3 Tóm tắt Presentation Layer

```
presentation/
├── cubit/
│   ├── user_cubit.dart        ← State management
│   └── user_state.dart        ← State definitions
├── pages/
│   ├── user_list_page.dart    ← Full screen
│   └── user_detail_page.dart
└── widgets/
    ├── user_card.dart         ← Reusable widget
    └── user_avatar.dart
```

> 💼 **Gặp trong dự án:** Khi migrate legacy code sang Clean Architecture, bắt đầu từ Repository layer. Không cần refactor toàn bộ cùng lúc — áp dụng dần theo feature mới. Feature cũ giữ nguyên, feature mới theo Clean Architecture. Sau 2-3 sprints sẽ thấy codebase dần clean hơn.

---

## 5. Dependency Rule — Quy tắc quan trọng nhất 🔴

### 5.1 Nguyên tắc

> **"Source code dependencies must point only INWARD."**
> — Robert C. Martin

```
Presentation ──depends on──▶ Domain ◀──depends on── Data
     │                         ▲                       │
     │                         │                       │
     └─── KHÔNG depends on ────┘──── KHÔNG depends ────┘
              Data                    Presentation
```

### 5.2 Bảng phụ thuộc cho phép

| Layer | Được import từ | KHÔNG được import từ |
|-------|---------------|---------------------|
| **Domain** | Chỉ Dart core | Data, Presentation, Packages |
| **Data** | Domain, Packages (http, sqflite...) | Presentation |
| **Presentation** | Domain, Packages (flutter, bloc...) | Data |

### 5.3 Dependency Inversion trong thực tế

```
Luồng phụ thuộc CODE (compile-time):
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ Presentation │ ───▶ │    Domain    │ ◀─── │     Data     │
│              │      │              │      │              │
│  UserCubit   │ ───▶ │ GetUserUC    │      │ UserRepoImpl │
│              │      │ UserRepo     │ ◀─── │              │
│              │      │ (abstract)   │      │              │
└──────────────┘      └──────────────┘      └──────────────┘

Luồng DATA (runtime):
UI ──▶ Cubit ──▶ UseCase ──▶ RepoImpl ──▶ DataSource ──▶ API
                                ▲
                    Inject qua constructor
```

**Tại sao Data phụ thuộc Domain mà không ngược lại?**

Domain định nghĩa `abstract class UserRepository` — nói rằng "Tôi cần ai đó cung cấp data theo contract này". Data layer `implements UserRepository` — nói rằng "Tôi sẽ cung cấp data theo contract đó."

→ Domain **KHÔNG biết** data đến từ API, database, hay file. Nó chỉ biết contract.

### 5.4 Kiểm tra vi phạm Dependency Rule

```dart
// ❌ VIOLATION: Domain import Data layer
// File: domain/usecases/get_user.dart
import '../../data/models/user_model.dart';     // ❌ KHÔNG ĐƯỢC!
import '../../data/datasources/api_client.dart'; // ❌ KHÔNG ĐƯỢC!

// ❌ VIOLATION: Presentation import Data layer
// File: presentation/cubit/user_cubit.dart
import '../../data/repositories/user_repo_impl.dart'; // ❌ KHÔNG ĐƯỢC!

// ✅ CORRECT: Presentation chỉ import Domain
import '../../domain/usecases/get_user.dart';    // ✅ OK
import '../../domain/entities/user.dart';         // ✅ OK
```

**Mẹo:** Search `import '../../data/` trong presentation folder — nếu tìm thấy = đang vi phạm!

> 🔗 **FE Bridge:** Dependency Rule = inner layers không biết outer layers — **giống hệt** FE hexagonal/ports-adapters architecture. Tuy nhiên FE dev ít enforce rule này (import từ bất kỳ đâu), Flutter với DI container **bắt buộc** follow dependency inversion.

---

> 💼 **Gặp trong dự án:** Code review phát hiện Presentation import trực tiếp Data layer, team member mới không hiểu tại sao phải dùng abstract class cho Repository, Dependency Inversion confusion
> 🤖 **Keywords bắt buộc trong prompt:** `Dependency Rule`, `Dependency Inversion Principle`, `abstract Repository interface`, `import direction check`, `layer boundary enforcement`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Code review:** Senior thấy `import 'package:app/data/repositories/user_repo_impl.dart'` trong Presentation → vi phạm Dependency Rule
- **Onboarding:** New member hỏi "tại sao phải tạo abstract UserRepository rồi mới implement? Sao không dùng thẳng?"
- **Refactoring:** Cần đổi từ REST API sang GraphQL — nếu Dependency Rule đúng → chỉ đổi Data layer, Domain + Presentation không đổi

**Tại sao cần các keyword trên:**
- **`Dependency Rule`** — outer layers phụ thuộc inner layers, KHÔNG ngược lại
- **`Dependency Inversion Principle`** — Domain define interface, Data implement
- **`abstract Repository interface`** — ở Domain layer, không biết gì về Data
- **`import direction check`** — tool/script kiểm tra import violations tự động

**Prompt mẫu — Verify Dependency Rule:**
```text
Tôi cần verify Dependency Rule trong Flutter project theo Clean Architecture.
Context: project có 3 layers — domain/, data/, presentation/.
Yêu cầu:
1. Phân tích import statements trong tất cả files.
2. Identify violations: presentation → data (trực tiếp), data → presentation.
3. Cho mỗi violation: file nào, import gì, cách fix.
4. Suggest: Dart analysis_options rule hoặc custom lint rule để prevent future violations.
5. Tạo shell script kiểm tra import violations trong CI/CD.
Input: [paste folder structure]
Output: Violation report + fix suggestions + CI script.
```

**Expected Output:** AI gen violation report + fix guide + shell script cho CI.

⚠️ **Giới hạn AI hay mắc:** AI hay nói "dùng abstract class" nhưng quên giải thích abstract class phải ở Domain layer (không phải Data layer). AI cũng hay suggest lint rules chưa tồn tại trong Dart analyzer.

</details>

> 🔗 **FE Bridge:** Dependency Rule = inner layers không biết outer layers — **giống hệt** FE hexagonal/ports-adapters architecture. Tuy nhiên FE dev ít enforce rule này (import từ bất kỳ đâu), Flutter với DI container **bắt buộc** follow dependency inversion.

---

## 6. Cấu trúc Folder chuẩn — Feature-first Approach 🟡

### 6.1 Feature-first vs Layer-first

**Layer-first (KHÔNG khuyến khích cho dự án lớn):**

```
lib/
├── domain/
│   ├── entities/
│   │   ├── user.dart
│   │   ├── todo.dart          ← Khó tìm: user ở 3 folder khác nhau
│   │   └── note.dart
│   ├── usecases/
│   │   ├── get_user.dart
│   │   ├── create_todo.dart
│   │   └── delete_note.dart
│   └── repositories/
│       ├── user_repository.dart
│       ├── todo_repository.dart
│       └── note_repository.dart
├── data/
│   └── ...                    ← Tương tự: trộn lẫn các feature
└── presentation/
    └── ...
```

**Feature-first (KHUYẾN KHÍCH):**

```
lib/
├── core/                          ← Code dùng chung
│   ├── error/
│   │   ├── exceptions.dart        ← Custom exceptions
│   │   └── failures.dart          ← Failure classes
│   ├── network/
│   │   └── network_info.dart      ← Check connectivity
│   ├── usecases/
│   │   └── usecase.dart           ← Base UseCase class
│   └── utils/
│       ├── constants.dart
│       └── input_validator.dart
│
├── features/                      ← Mỗi feature = 1 folder chứa 3 layers
│   ├── auth/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       ├── login.dart
│   │   │       └── register.dart
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_data_source.dart
│   │   │   │   └── auth_local_data_source.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   ├── auth_bloc.dart
│   │       │   ├── auth_event.dart
│   │       │   └── auth_state.dart
│   │       ├── pages/
│   │       │   ├── login_page.dart
│   │       │   └── register_page.dart
│   │       └── widgets/
│   │           └── auth_form.dart
│   │
│   └── todo/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── todo.dart
│       │   ├── repositories/
│       │   │   └── todo_repository.dart
│       │   └── usecases/
│       │       ├── get_todos.dart
│       │       ├── create_todo.dart
│       │       └── toggle_todo.dart
│       ├── data/
│       │   ├── models/
│       │   │   └── todo_model.dart
│       │   ├── datasources/
│       │   │   └── todo_remote_data_source.dart
│       │   └── repositories/
│       │       └── todo_repository_impl.dart
│       └── presentation/
│           ├── cubit/
│           │   ├── todo_cubit.dart
│           │   └── todo_state.dart
│           ├── pages/
│           │   └── todo_list_page.dart
│           └── widgets/
│               └── todo_item.dart
│
├── injection_container.dart       ← Dependency Injection setup
└── main.dart
```

### 6.2 Tại sao Feature-first?

| Tiêu chí | Layer-first | Feature-first |
|-----------|-------------|---------------|
| Tìm code liên quan | Phải nhảy giữa nhiều folder | Tất cả trong 1 folder |
| Thêm feature mới | Sửa nhiều folder | Tạo 1 folder mới |
| Xóa feature | Xóa file ở nhiều nơi | Xóa 1 folder |
| Team phân chia | Khó phân công | Mỗi người/team 1 feature |
| Code conflict | Nhiều | Ít |

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

1. **Bắt đầu từ Domain layer** — viết Entity và Use Case trước, rồi mới Data và Presentation
2. **Mỗi Use Case = 1 file** — không gom nhiều use case vào 1 class
3. **Đặt tên rõ ràng** — `GetUserUseCase`, `CreateTodoUseCase`, không phải `UserService`
4. **Entity immutable** — dùng `final` fields và `const` constructor
5. **Repository interface ở Domain** — implementation ở Data
6. **Không import trực tiếp Data trong Presentation** — luôn qua Domain (Use Cases)

### ❌ Lỗi thường gặp

| Lỗi | Vấn đề | Cách sửa |
|-----|--------|---------|
| Entity có `fromJson` | Domain phụ thuộc serialization | Tách thành Model ở Data layer |
| UseCase gọi API trực tiếp | Domain phụ thuộc http package | Gọi qua Repository interface |
| Cubit tạo `UserRepositoryImpl` | Presentation phụ thuộc Data | Inject qua constructor |
| Chung 1 Model cho API + DB | Thay đổi API ảnh hưởng DB | Tách riêng RemoteModel + LocalModel |
| Quá nhiều UseCase cho logic đơn giản | Over-engineering | Đánh giá: nếu chỉ forward call, cân nhắc bỏ UseCase |
| Import `package:flutter` trong Domain | Domain phụ thuộc framework | Chỉ dùng pure Dart |

### ⚠️ Khi nào Clean Architecture là "quá mức"?

- App nhỏ, 1-2 màn hình → Không cần
- Prototype / MVP nhanh → Không cần
- Dự án production, nhiều feature, nhiều người → **Nên dùng**
- Logic business phức tạp → **Nên dùng**

> **Quy tắc ngón tay cái:** Nếu app có > 3 features và > 2 developers → nên dùng Clean Architecture.

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

Nếu bạn đến từ React hoặc Vue, đây là cách "map" các khái niệm:

### 8.1 Architecture Patterns

| Flutter Clean Architecture | React/Vue tương đương |
|---------------------------|----------------------|
| Clean Architecture (3 layers) | Không có standard rõ ràng (MVC/MVVM/Flux tùy team) |
| Feature-first folders | Feature-first (Next.js app router) vs Route-first (pages/) |
| Domain Entity | TypeScript interface / Zod schema |
| Use Case | Custom hook (`useGetUser`) / Service function |
| Repository interface | API service abstraction |
| Repository impl | Axios instance / fetch wrapper |
| Model (fromJson/toJson) | API response type + transformer |
| BLoC/Cubit | Redux reducer / Zustand store / Pinia store |

### 8.2 So sánh cụ thể

**Repository Pattern:**

```typescript
// React: API service (không có interface chính thức)
const apiService = {
  getUser: (id: string) => axios.get(`/users/${id}`),
  getUsers: () => axios.get('/users'),
};

// Hook sử dụng
function useUser(id: string) {
  return useQuery(['user', id], () => apiService.getUser(id));
}
```

```dart
// Flutter Clean Architecture: Interface + Implementation
// Domain (interface):
abstract class UserRepository {
  Future<User> getUserById(String id);
}

// Data (implementation):
class UserRepositoryImpl implements UserRepository {
  @override
  Future<User> getUserById(String id) async { ... }
}
```

**Use Case vs Custom Hook:**

```typescript
// React: Custom hook = gần giống Use Case
function useGetActiveUsers() {
  const { data: users } = useUsers();
  return users?.filter(u => u.isActive) ?? [];
}
```

```dart
// Flutter: Use Case class
class GetActiveUsersUseCase {
  final UserRepository repository;
  GetActiveUsersUseCase(this.repository);

  Future<List<User>> call() async {
    final users = await repository.getAllUsers();
    return users.where((u) => u.isActive).toList();
  }
}
```

### 8.3 Điểm khác biệt chính

| Khía cạnh | React/Vue | Flutter Clean Architecture |
|-----------|-----------|---------------------------|
| Enforced separation | Không — team tự quyết | Có — compile error nếu vi phạm import |
| Testability | Mock module/API | Mock repository interface (dễ hơn) |
| Dependency direction | Tùy ý | Bắt buộc hướng vào trong |
| Serialization | Tự động (JSON ↔ JS object) | Cần viết fromJson/toJson (hoặc code gen) |

> **Key insight:** Trong React, bạn _có thể_ tổ chức code tốt nhưng không bị _ép buộc_. Trong Flutter Clean Architecture, cấu trúc folder + import rules **ép bạn** phải tách biệt rõ ràng.

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|---------------|
| 1 | Folder structure = convention, không enforce | Layer boundaries = **strict**, DI enforce dependency rule | Flutter community coi architecture là requirement, FE coi là nice-to-have |
| 2 | Custom hooks = reusable logic | Use Case class = single responsibility unit | OOP-first thay vì functional-first approach |
| 3 | Service = function module | Repository = abstract class + implementation split | Interface/implementation separation là standard |

---

## 9. Tổng kết

### Checklist kiến thức buổi 09

| # | Nội dung | Hiểu? |
|---|---------|-------|
| 1 | Clean Architecture là gì, tại sao cần | ☐ |
| 2 | 3 layers: Domain, Data, Presentation — vai trò từng layer | ☐ |
| 3 | Entity: pure Dart, immutable, no framework dependency | ☐ |
| 4 | Use Case: 1 class = 1 business operation, `call()` method | ☐ |
| 5 | Repository interface (Domain) vs implementation (Data) | ☐ |
| 6 | Model: extends Entity, thêm fromJson/toJson | ☐ |
| 7 | DataSource: Remote (API) + Local (cache/DB) | ☐ |
| 8 | Dependency Rule: trong → ngoài, không bao giờ ngược | ☐ |
| 9 | Feature-first folder structure | ☐ |
| 10 | Biết khi nào nên/không nên dùng Clean Architecture | ☐ |

### Flow tổng quan

```
User nhấn nút
    │
    ▼
Widget ──▶ Cubit/BLoC ──▶ UseCase ──▶ Repository(impl) ──▶ DataSource ──▶ API
                                            │
                                            ▼
                                     Return Entity
                                            │
    ◀── Update UI ◀── Emit State ◀── Return ┘
```

### Buổi tiếp theo

**Buổi 10: DI & Testing** — Kết nối tất cả các layer lại với nhau bằng `get_it` và `injectable`, kết hợp testing để đảm bảo chất lượng code.
