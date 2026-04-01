# Kiến Trúc Tham Chiếu Cho Production Flutter Project

> Tài liệu mô tả cấu trúc mẫu, conventions, patterns, và yêu cầu capstone project cho chương trình Flutter Training.

---

## 1. Cấu Trúc Folder Mẫu (Feature-First Clean Architecture)

```
lib/
├── main.dart                          # Entry point
├── app.dart                           # MaterialApp, theme, router setup
├── bootstrap.dart                     # App initialization (DI, Hive, etc.)
│
├── core/                              # Shared across all features
│   ├── constants/
│   │   ├── app_constants.dart         # App-wide constants
│   │   ├── api_constants.dart         # API endpoints, base URL
│   │   └── storage_keys.dart          # Hive/SharedPreferences keys
│   │
│   ├── error/
│   │   ├── app_exception.dart         # Custom exception classes
│   │   ├── failure.dart               # Failure classes cho Result type
│   │   └── error_handler.dart         # Centralized error handling
│   │
│   ├── network/
│   │   ├── dio_client.dart            # Dio instance configuration
│   │   ├── api_interceptors.dart      # Auth, logging, error interceptors
│   │   └── network_info.dart          # Connectivity checker
│   │
│   ├── storage/
│   │   ├── hive_storage.dart          # Hive initialization & boxes
│   │   └── secure_storage.dart        # Flutter secure storage wrapper
│   │
│   ├── router/
│   │   ├── app_router.dart            # GoRouter configuration
│   │   └── route_names.dart           # Route name constants
│   │
│   ├── theme/
│   │   ├── app_theme.dart             # ThemeData (light/dark)
│   │   ├── app_colors.dart            # Color palette
│   │   ├── app_text_styles.dart       # Typography
│   │   └── app_spacing.dart           # Padding, margin constants
│   │
│   ├── utils/
│   │   ├── result.dart                # Result<T> type (Success/Failure)
│   │   ├── date_utils.dart            # Date formatting helpers
│   │   ├── validators.dart            # Input validation functions
│   │   └── extensions/
│   │       ├── context_extensions.dart
│   │       ├── string_extensions.dart
│   │       └── datetime_extensions.dart
│   │
│   └── widgets/                       # Shared reusable widgets
│       ├── app_button.dart
│       ├── app_text_field.dart
│       ├── loading_overlay.dart
│       ├── error_widget.dart
│       ├── empty_state_widget.dart
│       └── responsive_builder.dart
│
├── features/                          # Feature modules
│   ├── auth/                          # Authentication feature
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart        # Abstract interface
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       ├── logout_usecase.dart
│   │   │       └── check_auth_usecase.dart
│   │   │
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   ├── user_model.dart             # freezed + json
│   │   │   │   ├── login_request.dart
│   │   │   │   └── login_response.dart
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_datasource.dart # Retrofit API
│   │   │   │   └── auth_local_datasource.dart  # Token storage
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart   # Implementation
│   │   │
│   │   └── presentation/
│   │       ├── bloc/                           # Hoặc providers/
│   │       │   ├── auth_bloc.dart
│   │       │   ├── auth_event.dart
│   │       │   └── auth_state.dart
│   │       ├── pages/
│   │       │   ├── login_page.dart
│   │       │   └── register_page.dart
│   │       └── widgets/
│   │           ├── login_form.dart
│   │           └── social_login_buttons.dart
│   │
│   ├── tasks/                         # Task management feature
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── task.dart
│   │   │   │   └── task_category.dart
│   │   │   ├── repositories/
│   │   │   │   └── task_repository.dart
│   │   │   └── usecases/
│   │   │       ├── get_tasks_usecase.dart
│   │   │       ├── create_task_usecase.dart
│   │   │       ├── update_task_usecase.dart
│   │   │       ├── delete_task_usecase.dart
│   │   │       └── filter_tasks_usecase.dart
│   │   │
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   ├── task_model.dart
│   │   │   │   └── task_category_model.dart
│   │   │   ├── datasources/
│   │   │   │   ├── task_remote_datasource.dart
│   │   │   │   └── task_local_datasource.dart
│   │   │   └── repositories/
│   │   │       └── task_repository_impl.dart
│   │   │
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   ├── task_list_bloc.dart
│   │       │   ├── task_detail_bloc.dart
│   │       │   └── task_form_bloc.dart
│   │       ├── pages/
│   │       │   ├── task_list_page.dart
│   │       │   ├── task_detail_page.dart
│   │       │   └── task_form_page.dart
│   │       └── widgets/
│   │           ├── task_card.dart
│   │           ├── task_filter_bar.dart
│   │           ├── priority_badge.dart
│   │           └── category_chip.dart
│   │
│   ├── categories/                    # Category management feature
│   │   ├── domain/
│   │   ├── data/
│   │   └── presentation/
│   │
│   ├── notifications/                 # Push notification feature
│   │   ├── domain/
│   │   ├── data/
│   │   └── presentation/
│   │
│   └── settings/                      # App settings feature
│       ├── domain/
│       ├── data/
│       └── presentation/
│
├── l10n/                              # Localization
│   ├── app_en.arb
│   └── app_vi.arb
│
└── di/                                # Dependency Injection setup
    ├── injection.dart                 # get_it configuration
    └── modules/
        ├── network_module.dart
        ├── storage_module.dart
        └── feature_modules.dart

test/
├── core/
│   ├── network/
│   │   └── dio_client_test.dart
│   └── utils/
│       └── result_test.dart
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl_test.dart
│   │   ├── domain/
│   │   │   └── usecases/
│   │   │       └── login_usecase_test.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   └── auth_bloc_test.dart
│   │       └── pages/
│   │           └── login_page_test.dart
│   │
│   └── tasks/
│       ├── data/
│       ├── domain/
│       └── presentation/
│
├── helpers/
│   ├── test_helpers.dart
│   ├── mock_classes.dart
│   └── fake_data.dart
│
└── integration_test/
    ├── app_test.dart
    └── auth_flow_test.dart
```

---

## 2. Giải Thích Từng Layer

### 2.1. Domain Layer (Lõi ứng dụng)

```
domain/
├── entities/       # Business objects thuần túy
├── repositories/   # Abstract interfaces (contracts)
└── usecases/       # Business logic cụ thể
```

**Nguyên tắc:**
- **Không phụ thuộc** vào bất kỳ layer nào khác
- **Không import** Flutter, Dio, Hive, hay bất kỳ package bên ngoài nào
- Chỉ chứa Dart thuần túy (pure Dart)
- Entities là plain Dart classes, không có annotation `@freezed`, `@JsonSerializable`

**Entities:**
```dart
// domain/entities/task.dart
class Task {
  final String id;
  final String title;
  final String description;
  final TaskPriority priority;
  final DateTime? dueDate;
  final bool isCompleted;
  final String categoryId;
  final DateTime createdAt;
  final DateTime updatedAt;

  const Task({
    required this.id,
    required this.title,
    required this.description,
    required this.priority,
    this.dueDate,
    this.isCompleted = false,
    required this.categoryId,
    required this.createdAt,
    required this.updatedAt,
  });
}

enum TaskPriority { low, medium, high, urgent }
```

**Repository Interface:**
```dart
// domain/repositories/task_repository.dart
import '../entities/task.dart';
import '../../core/utils/result.dart';

abstract class TaskRepository {
  Future<Result<List<Task>>> getTasks({
    int page = 1,
    int limit = 20,
    TaskFilter? filter,
  });
  Future<Result<Task>> getTaskById(String id);
  Future<Result<Task>> createTask(CreateTaskParams params);
  Future<Result<Task>> updateTask(UpdateTaskParams params);
  Future<Result<void>> deleteTask(String id);
}
```

**UseCase:**
```dart
// domain/usecases/get_tasks_usecase.dart
import '../entities/task.dart';
import '../repositories/task_repository.dart';
import '../../core/utils/result.dart';

class GetTasksUseCase {
  final TaskRepository _repository;

  const GetTasksUseCase(this._repository);

  Future<Result<List<Task>>> call({
    int page = 1,
    int limit = 20,
    TaskFilter? filter,
  }) {
    return _repository.getTasks(
      page: page,
      limit: limit,
      filter: filter,
    );
  }
}
```

### 2.2. Data Layer (Triển khai chi tiết)

```
data/
├── models/         # Data transfer objects (DTO) với serialization
├── datasources/    # Remote (API) và Local (Hive/SQLite)
└── repositories/   # Implementation của domain repository interfaces
```

**Nguyên tắc:**
- Phụ thuộc vào Domain layer (implement interfaces)
- Chứa tất cả chi tiết kỹ thuật: JSON parsing, API calls, database queries
- Models map qua lại với Entities thông qua extension methods hoặc mapper

**Model (DTO):**
```dart
// data/models/task_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../domain/entities/task.dart';

part 'task_model.freezed.dart';
part 'task_model.g.dart';

@freezed
class TaskModel with _$TaskModel {
  const TaskModel._();

  const factory TaskModel({
    required String id,
    required String title,
    required String description,
    required String priority,
    @JsonKey(name: 'due_date') DateTime? dueDate,
    @JsonKey(name: 'is_completed') @Default(false) bool isCompleted,
    @JsonKey(name: 'category_id') required String categoryId,
    @JsonKey(name: 'created_at') required DateTime createdAt,
    @JsonKey(name: 'updated_at') required DateTime updatedAt,
  }) = _TaskModel;

  factory TaskModel.fromJson(Map<String, dynamic> json) =>
      _$TaskModelFromJson(json);

  // Mapper: Model → Entity
  Task toEntity() => Task(
        id: id,
        title: title,
        description: description,
        priority: TaskPriority.values.byName(priority),
        dueDate: dueDate,
        isCompleted: isCompleted,
        categoryId: categoryId,
        createdAt: createdAt,
        updatedAt: updatedAt,
      );

  // Mapper: Entity → Model
  factory TaskModel.fromEntity(Task task) => TaskModel(
        id: task.id,
        title: task.title,
        description: task.description,
        priority: task.priority.name,
        dueDate: task.dueDate,
        isCompleted: task.isCompleted,
        categoryId: task.categoryId,
        createdAt: task.createdAt,
        updatedAt: task.updatedAt,
      );
}
```

**Remote Datasource (Retrofit):**
```dart
// data/datasources/task_remote_datasource.dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../models/task_model.dart';

part 'task_remote_datasource.g.dart';

@RestApi()
abstract class TaskRemoteDatasource {
  factory TaskRemoteDatasource(Dio dio) = _TaskRemoteDatasource;

  @GET('/tasks')
  Future<ApiResponse<List<TaskModel>>> getTasks(
    @Query('page') int page,
    @Query('limit') int limit,
    @Query('status') String? status,
    @Query('priority') String? priority,
    @Query('category_id') String? categoryId,
  );

  @GET('/tasks/{id}')
  Future<ApiResponse<TaskModel>> getTaskById(@Path('id') String id);

  @POST('/tasks')
  Future<ApiResponse<TaskModel>> createTask(@Body() CreateTaskRequest request);

  @PUT('/tasks/{id}')
  Future<ApiResponse<TaskModel>> updateTask(
    @Path('id') String id,
    @Body() UpdateTaskRequest request,
  );

  @DELETE('/tasks/{id}')
  Future<void> deleteTask(@Path('id') String id);
}
```

**Local Datasource (Hive):**
```dart
// data/datasources/task_local_datasource.dart
import 'package:hive/hive.dart';
import '../models/task_model.dart';

class TaskLocalDatasource {
  static const String _boxName = 'tasks';

  Future<Box<Map>> _getBox() async => Hive.openBox<Map>(_boxName);

  Future<List<TaskModel>> getCachedTasks() async {
    final box = await _getBox();
    return box.values
        .map((e) => TaskModel.fromJson(Map<String, dynamic>.from(e)))
        .toList();
  }

  Future<void> cacheTasks(List<TaskModel> tasks) async {
    final box = await _getBox();
    await box.clear();
    for (final task in tasks) {
      await box.put(task.id, task.toJson());
    }
  }

  Future<void> cacheTask(TaskModel task) async {
    final box = await _getBox();
    await box.put(task.id, task.toJson());
  }

  Future<void> removeTask(String id) async {
    final box = await _getBox();
    await box.delete(id);
  }
}
```

**Repository Implementation:**
```dart
// data/repositories/task_repository_impl.dart
import '../../core/error/app_exception.dart';
import '../../core/network/network_info.dart';
import '../../core/utils/result.dart';
import '../../domain/entities/task.dart';
import '../../domain/repositories/task_repository.dart';
import '../datasources/task_local_datasource.dart';
import '../datasources/task_remote_datasource.dart';
import '../models/task_model.dart';

class TaskRepositoryImpl implements TaskRepository {
  final TaskRemoteDatasource _remoteDatasource;
  final TaskLocalDatasource _localDatasource;
  final NetworkInfo _networkInfo;

  const TaskRepositoryImpl(
    this._remoteDatasource,
    this._localDatasource,
    this._networkInfo,
  );

  @override
  Future<Result<List<Task>>> getTasks({
    int page = 1,
    int limit = 20,
    TaskFilter? filter,
  }) async {
    try {
      if (await _networkInfo.isConnected) {
        final response = await _remoteDatasource.getTasks(
          page,
          limit,
          filter?.status,
          filter?.priority,
          filter?.categoryId,
        );
        final tasks = response.data;

        // Cache locally
        if (page == 1) {
          await _localDatasource.cacheTasks(tasks);
        }

        return Result.success(tasks.map((m) => m.toEntity()).toList());
      } else {
        // Offline: return cached data
        final cachedTasks = await _localDatasource.getCachedTasks();
        return Result.success(
          cachedTasks.map((m) => m.toEntity()).toList(),
        );
      }
    } on AppException catch (e) {
      return Result.failure(e.toFailure());
    } catch (e) {
      return Result.failure(Failure.unexpected(e.toString()));
    }
  }

  // ... other methods
}
```

### 2.3. Presentation Layer (Giao diện người dùng)

```
presentation/
├── bloc/ (hoặc providers/)  # State management
├── pages/                    # Full screen widgets
└── widgets/                  # Feature-specific reusable widgets
```

**Nguyên tắc:**
- Phụ thuộc vào Domain layer (dùng UseCases)
- **Không import** trực tiếp Data layer
- BLoC/Provider chỉ gọi UseCases, không gọi Repository trực tiếp
- Pages compose widgets, widgets chứa UI logic
- Không có business logic trong UI

**BLoC:**
```dart
// presentation/bloc/task_list_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../../domain/entities/task.dart';
import '../../../domain/usecases/get_tasks_usecase.dart';
import '../../../domain/usecases/delete_task_usecase.dart';

part 'task_list_bloc.freezed.dart';

// Events
@freezed
class TaskListEvent with _$TaskListEvent {
  const factory TaskListEvent.loadTasks({TaskFilter? filter}) = _LoadTasks;
  const factory TaskListEvent.loadMore() = _LoadMore;
  const factory TaskListEvent.deleteTask(String id) = _DeleteTask;
  const factory TaskListEvent.refresh() = _Refresh;
}

// States
@freezed
class TaskListState with _$TaskListState {
  const factory TaskListState.initial() = _Initial;
  const factory TaskListState.loading() = _Loading;
  const factory TaskListState.loaded({
    required List<Task> tasks,
    required bool hasMore,
    required int currentPage,
    TaskFilter? activeFilter,
  }) = _Loaded;
  const factory TaskListState.error(String message) = _Error;
}

// BLoC
class TaskListBloc extends Bloc<TaskListEvent, TaskListState> {
  final GetTasksUseCase _getTasksUseCase;
  final DeleteTaskUseCase _deleteTaskUseCase;

  TaskListBloc(this._getTasksUseCase, this._deleteTaskUseCase)
      : super(const TaskListState.initial()) {
    on<_LoadTasks>(_onLoadTasks);
    on<_LoadMore>(_onLoadMore);
    on<_DeleteTask>(_onDeleteTask);
    on<_Refresh>(_onRefresh);
  }

  Future<void> _onLoadTasks(
    _LoadTasks event,
    Emitter<TaskListState> emit,
  ) async {
    emit(const TaskListState.loading());

    final result = await _getTasksUseCase(
      page: 1,
      filter: event.filter,
    );

    result.when(
      success: (tasks) => emit(TaskListState.loaded(
        tasks: tasks,
        hasMore: tasks.length >= 20,
        currentPage: 1,
        activeFilter: event.filter,
      )),
      failure: (failure) => emit(TaskListState.error(failure.message)),
    );
  }

  // ... other handlers
}
```

### 2.4. Core Layer (Chia sẻ chung)

Chứa utilities, configurations, và shared widgets dùng chung giữa tất cả features:

| Folder | Mục đích |
|--------|---------|
| `constants/` | Hằng số toàn app: API URLs, storage keys, magic numbers |
| `error/` | Custom exceptions, Failure classes, centralized error handling |
| `network/` | Dio client, interceptors, connectivity checker |
| `storage/` | Hive setup, secure storage wrapper |
| `router/` | GoRouter config, route names, guards |
| `theme/` | Colors, typography, spacing, ThemeData |
| `utils/` | Helpers, extensions, validators, Result type |
| `widgets/` | Reusable widgets dùng chung (buttons, text fields, loading) |

---

## 3. Capstone Project: Task Management App

### 3.1. Tổng Quan

Xây dựng ứng dụng **Task Management** (todo app nâng cao) áp dụng toàn bộ kiến thức đã học trong 8 tuần training.

### 3.2. Features Yêu Cầu

#### F1: Authentication
- Đăng nhập bằng email + password
- Đăng ký tài khoản mới
- Logout
- Auto-login nếu token còn hạn
- Refresh token khi hết hạn

#### F2: Task CRUD
- Tạo task mới: title, description, priority, due date, category
- Xem danh sách tasks (pagination, pull-to-refresh)
- Xem chi tiết task
- Cập nhật task (edit tất cả fields, toggle completed)
- Xóa task (soft delete với confirm dialog)

#### F3: Categories
- Tạo/sửa/xóa categories
- Mỗi category có: name, color, icon
- Gán task vào category
- Xem tasks theo category

#### F4: Filters & Search
- Filter theo: status (all/active/completed), priority, category, due date
- Search tasks theo title/description
- Sort theo: created date, due date, priority
- Persist filter preferences

#### F5: Offline Support
- Cache tasks locally bằng Hive
- Hiển thị cached data khi offline
- Sync khi có lại kết nối
- Hiển thị trạng thái online/offline

#### F6: Push Notifications
- Nhắc nhở task sắp đến hạn (local notification)
- Nhận notification từ server (Firebase Cloud Messaging)
- Deep link từ notification đến task detail

### 3.3. Yêu Cầu Kỹ Thuật

| Hạng mục | Yêu cầu |
|----------|---------|
| **Architecture** | Clean Architecture, feature-first folder structure |
| **State Management** | Riverpod **hoặc** BLoC (chọn 1, dùng nhất quán) |
| **Networking** | Dio + Retrofit, proper error handling |
| **Local Storage** | Hive cho data caching, flutter_secure_storage cho tokens |
| **Navigation** | GoRouter với nested navigation |
| **Code Generation** | freezed, json_serializable, retrofit_generator |
| **Testing** | Unit tests (core logic), Widget tests (key screens) |
| **CI** | GitHub Actions chạy analyze + test |
| **Minimum Dart/Flutter** | Dart 3.x, Flutter 3.x |

### 3.4. Mock API Endpoints

> Base URL: `https://api.taskapp.example.com/v1`

#### Authentication

**POST /auth/login**
```json
// Request
{
  "email": "user@example.com",
  "password": "password123"
}

// Response 200
{
  "data": {
    "access_token": "eyJhbGciOi...",
    "refresh_token": "eyJhbGciOi...",
    "expires_in": 3600,
    "user": {
      "id": "usr_001",
      "email": "user@example.com",
      "name": "Nguyen Van A",
      "avatar_url": "https://api.taskapp.example.com/avatars/usr_001.jpg"
    }
  },
  "message": "Login successful"
}

// Response 401
{
  "error": {
    "code": 401,
    "message": "Invalid email or password"
  }
}
```

**POST /auth/register**
```json
// Request
{
  "email": "newuser@example.com",
  "password": "password123",
  "name": "Nguyen Van B"
}

// Response 201
{
  "data": {
    "access_token": "eyJhbGciOi...",
    "refresh_token": "eyJhbGciOi...",
    "expires_in": 3600,
    "user": {
      "id": "usr_002",
      "email": "newuser@example.com",
      "name": "Nguyen Van B",
      "avatar_url": null
    }
  },
  "message": "Registration successful"
}
```

**POST /auth/refresh**
```json
// Request
{
  "refresh_token": "eyJhbGciOi..."
}

// Response 200
{
  "data": {
    "access_token": "eyJhbGciOi...(new)",
    "refresh_token": "eyJhbGciOi...(new)",
    "expires_in": 3600
  },
  "message": "Token refreshed"
}
```

**POST /auth/logout**
```json
// Headers: Authorization: Bearer <access_token>
// Response 200
{
  "message": "Logged out successfully"
}
```

#### Tasks

**GET /tasks?page=1&limit=20&status=active&priority=high&category_id=cat_001&search=keyword&sort_by=due_date&sort_order=asc**
```json
// Response 200
{
  "data": [
    {
      "id": "task_001",
      "title": "Setup Flutter project",
      "description": "Initialize project with Clean Architecture structure",
      "priority": "high",
      "due_date": "2026-04-15T23:59:59Z",
      "is_completed": false,
      "category_id": "cat_001",
      "category": {
        "id": "cat_001",
        "name": "Development",
        "color": "#4CAF50",
        "icon": "code"
      },
      "created_at": "2026-04-01T10:00:00Z",
      "updated_at": "2026-04-01T10:00:00Z"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 45,
    "total_pages": 3
  },
  "message": "Tasks retrieved successfully"
}
```

**GET /tasks/:id**
```json
// Response 200
{
  "data": {
    "id": "task_001",
    "title": "Setup Flutter project",
    "description": "Initialize project with Clean Architecture structure",
    "priority": "high",
    "due_date": "2026-04-15T23:59:59Z",
    "is_completed": false,
    "category_id": "cat_001",
    "category": {
      "id": "cat_001",
      "name": "Development",
      "color": "#4CAF50",
      "icon": "code"
    },
    "subtasks": [
      {
        "id": "sub_001",
        "title": "Create folder structure",
        "is_completed": true
      },
      {
        "id": "sub_002",
        "title": "Add dependencies to pubspec.yaml",
        "is_completed": false
      }
    ],
    "created_at": "2026-04-01T10:00:00Z",
    "updated_at": "2026-04-01T10:00:00Z"
  },
  "message": "Task retrieved successfully"
}

// Response 404
{
  "error": {
    "code": 404,
    "message": "Task not found"
  }
}
```

**POST /tasks**
```json
// Request
{
  "title": "Implement login screen",
  "description": "Build login UI with email and password fields",
  "priority": "medium",
  "due_date": "2026-04-20T23:59:59Z",
  "category_id": "cat_001"
}

// Response 201
{
  "data": {
    "id": "task_002",
    "title": "Implement login screen",
    "description": "Build login UI with email and password fields",
    "priority": "medium",
    "due_date": "2026-04-20T23:59:59Z",
    "is_completed": false,
    "category_id": "cat_001",
    "created_at": "2026-04-02T09:00:00Z",
    "updated_at": "2026-04-02T09:00:00Z"
  },
  "message": "Task created successfully"
}

// Response 422 (validation error)
{
  "error": {
    "code": 422,
    "message": "Validation failed",
    "details": {
      "title": ["Title is required", "Title must be at least 3 characters"]
    }
  }
}
```

**PUT /tasks/:id**
```json
// Request (partial update allowed)
{
  "title": "Implement login screen (updated)",
  "is_completed": true
}

// Response 200
{
  "data": {
    "id": "task_002",
    "title": "Implement login screen (updated)",
    "is_completed": true,
    "...": "..."
  },
  "message": "Task updated successfully"
}
```

**DELETE /tasks/:id**
```json
// Response 200
{
  "message": "Task deleted successfully"
}
```

#### Categories

**GET /categories**
```json
// Response 200
{
  "data": [
    {
      "id": "cat_001",
      "name": "Development",
      "color": "#4CAF50",
      "icon": "code",
      "task_count": 12
    },
    {
      "id": "cat_002",
      "name": "Design",
      "color": "#2196F3",
      "icon": "palette",
      "task_count": 5
    },
    {
      "id": "cat_003",
      "name": "Meeting",
      "color": "#FF9800",
      "icon": "people",
      "task_count": 8
    }
  ],
  "message": "Categories retrieved successfully"
}
```

**POST /categories**
```json
// Request
{
  "name": "Research",
  "color": "#9C27B0",
  "icon": "search"
}

// Response 201
{
  "data": {
    "id": "cat_004",
    "name": "Research",
    "color": "#9C27B0",
    "icon": "search",
    "task_count": 0
  },
  "message": "Category created successfully"
}
```

**PUT /categories/:id**
```json
// Request
{
  "name": "Research & Analysis",
  "color": "#7B1FA2"
}

// Response 200
{
  "data": {
    "id": "cat_004",
    "name": "Research & Analysis",
    "color": "#7B1FA2",
    "icon": "search",
    "task_count": 0
  },
  "message": "Category updated successfully"
}
```

**DELETE /categories/:id**
```json
// Response 200
{
  "message": "Category deleted successfully"
}

// Response 409 (conflict - category has tasks)
{
  "error": {
    "code": 409,
    "message": "Cannot delete category with existing tasks. Move or delete tasks first."
  }
}
```

#### Common Error Responses

```json
// 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "Bad request"
  }
}

// 401 Unauthorized
{
  "error": {
    "code": 401,
    "message": "Unauthorized. Token expired or invalid."
  }
}

// 403 Forbidden
{
  "error": {
    "code": 403,
    "message": "You don't have permission to perform this action"
  }
}

// 500 Internal Server Error
{
  "error": {
    "code": 500,
    "message": "Internal server error. Please try again later."
  }
}
```

### 3.5. Tiêu Chí Đánh Giá Capstone

| # | Tiêu chí | Điểm tối đa | Mô tả |
|---|----------|-------------|-------|
| 1 | **Architecture** | 20 | Clean Architecture đúng chuẩn, layer separation rõ ràng |
| 2 | **State Management** | 15 | Dùng BLoC/Riverpod đúng cách, state handling tốt |
| 3 | **UI/UX** | 15 | UI responsive, smooth animations, good UX |
| 4 | **Networking** | 15 | Dio + Retrofit, error handling, auth flow hoàn chỉnh |
| 5 | **Local Storage** | 10 | Offline support, caching strategy hợp lý |
| 6 | **Testing** | 10 | Unit tests + widget tests, coverage ≥ 60% core logic |
| 7 | **Code Quality** | 10 | Clean code, naming conventions, no lint warnings |
| 8 | **Bonus** | 5 | Push notifications, CI/CD, dark mode, animations nâng cao |
| | **TỔNG** | **100** | |

**Thang đánh giá:**

| Điểm | Kết quả |
|------|---------|
| 90-100 | Xuất sắc - Vượt mong đợi |
| 80-89 | Tốt - Đạt chuẩn Middle |
| 70-79 | Đạt - Cần cải thiện một số điểm |
| 60-69 | Cần bổ sung - Hoàn thành thêm requirements |
| < 60 | Chưa đạt - Cần làm lại |

---

## 4. Naming Conventions

### 4.1. Files

| Loại | Convention | Ví dụ |
|------|-----------|-------|
| Dart files | `snake_case.dart` | `task_repository.dart` |
| Test files | `*_test.dart` | `task_repository_test.dart` |
| Generated files | `*.g.dart`, `*.freezed.dart` | `task_model.g.dart` |
| Feature folders | `snake_case/` | `task_management/` |
| Asset files | `snake_case` | `ic_arrow_back.svg` |

### 4.2. Classes

| Loại | Convention | Ví dụ |
|------|-----------|-------|
| Widget | `PascalCase` + hậu tố ngữ cảnh | `TaskListPage`, `TaskCard` |
| BLoC | `PascalCase` + `Bloc` | `TaskListBloc` |
| Cubit | `PascalCase` + `Cubit` | `TaskFormCubit` |
| Event | `PascalCase` + `Event` | `TaskListEvent` |
| State | `PascalCase` + `State` | `TaskListState` |
| Model | `PascalCase` + `Model` | `TaskModel` |
| Entity | `PascalCase` (plain) | `Task` |
| Repository (interface) | `PascalCase` + `Repository` | `TaskRepository` |
| Repository (impl) | `PascalCase` + `RepositoryImpl` | `TaskRepositoryImpl` |
| UseCase | `PascalCase` + `UseCase` | `GetTasksUseCase` |
| Datasource | `PascalCase` + `Datasource` | `TaskRemoteDatasource` |
| Provider | `camelCase` + `Provider` | `taskListProvider` |
| Extension | `PascalCase` + `Extension` / `X` | `StringExtension`, `ContextX` |

### 4.3. Methods & Variables

| Loại | Convention | Ví dụ |
|------|-----------|-------|
| Methods | `camelCase`, động từ đầu | `getTasks()`, `deleteTask()` |
| Variables | `camelCase` | `taskList`, `isLoading` |
| Constants | `camelCase` hoặc `SCREAMING_SNAKE_CASE` | `defaultPageSize`, `API_BASE_URL` |
| Private | `_camelCase` | `_taskRepository`, `_onLoadTasks()` |
| Boolean | `is`/`has`/`should` prefix | `isCompleted`, `hasMore`, `shouldRefresh` |
| Callbacks | `on` prefix | `onTap`, `onTaskCreated` |
| Builder | `build` prefix | `_buildTaskItem()`, `_buildEmptyState()` |

### 4.4. Quy Tắc Đặt Tên Khác

**BLoC Events** - dùng past tense hoặc imperative:
```dart
// ✅ Good
TaskListEvent.loadTasks()
TaskListEvent.taskDeleted(String id)
TaskListEvent.filterChanged(TaskFilter filter)

// ❌ Bad
TaskListEvent.loading()
TaskListEvent.delete()
```

**BLoC States** - dùng tính từ hoặc trạng thái:
```dart
// ✅ Good
TaskListState.initial()
TaskListState.loading()
TaskListState.loaded(tasks)
TaskListState.error(message)

// ❌ Bad
TaskListState.load()
TaskListState.getTasks()
```

**Riverpod Providers**:
```dart
// ✅ Good
final taskListProvider = StateNotifierProvider<TaskListNotifier, TaskListState>(...);
final currentTaskProvider = FutureProvider.family<Task, String>(...);
final isLoggedInProvider = Provider<bool>(...);

// ❌ Bad
final tasks = StateNotifierProvider(...);  // Quá ngắn
final getTaskListStateNotifierProvider = ...;  // Quá dài
```

---

## 5. Common Patterns

### 5.1. Result Type Pattern

Dùng để xử lý success/failure mà không dùng try-catch ở tầng business logic.

```dart
// core/utils/result.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../error/failure.dart';

part 'result.freezed.dart';

@freezed
sealed class Result<T> with _$Result<T> {
  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(Failure failure) = ResultFailure<T>;
}

// Sử dụng:
final result = await getTasksUseCase();
result.when(
  success: (tasks) => emit(TaskListState.loaded(tasks: tasks)),
  failure: (failure) => emit(TaskListState.error(failure.message)),
);
```

**Failure class:**
```dart
// core/error/failure.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'failure.freezed.dart';

@freezed
class Failure with _$Failure {
  const factory Failure.server({
    required String message,
    required int statusCode,
  }) = ServerFailure;

  const factory Failure.network({
    @Default('No internet connection') String message,
  }) = NetworkFailure;

  const factory Failure.cache({
    @Default('Cache error') String message,
  }) = CacheFailure;

  const factory Failure.unexpected({
    @Default('An unexpected error occurred') String message,
  }) = UnexpectedFailure;

  const factory Failure.validation({
    required String message,
    required Map<String, List<String>> errors,
  }) = ValidationFailure;
}
```

### 5.2. Repository Pattern

Abstract interface ở Domain layer, implementation ở Data layer.

```dart
// Domain: định nghĩa "CẦN GÌ"
abstract class TaskRepository {
  Future<Result<List<Task>>> getTasks({int page, TaskFilter? filter});
  Future<Result<Task>> getTaskById(String id);
  Future<Result<Task>> createTask(CreateTaskParams params);
  Future<Result<Task>> updateTask(UpdateTaskParams params);
  Future<Result<void>> deleteTask(String id);
}

// Data: định nghĩa "LÀM THẾ NÀO"
class TaskRepositoryImpl implements TaskRepository {
  final TaskRemoteDatasource _remote;
  final TaskLocalDatasource _local;
  final NetworkInfo _networkInfo;

  // Implementation xử lý:
  // - Gọi API (remote) khi online
  // - Đọc cache (local) khi offline
  // - Xử lý errors, mapping models → entities
}
```

**Lợi ích:**
- Domain layer không biết data đến từ đâu (API? Database? File?)
- Dễ dàng swap implementation (mock cho testing)
- Dễ thêm caching, offline support mà không ảnh hưởng business logic

### 5.3. UseCase Pattern

Mỗi UseCase đại diện cho 1 hành động cụ thể của người dùng.

```dart
// Base UseCase (optional)
abstract class UseCase<T, Params> {
  Future<Result<T>> call(Params params);
}

class NoParams {
  const NoParams();
}

// Concrete UseCase
class GetTasksUseCase {
  final TaskRepository _repository;

  const GetTasksUseCase(this._repository);

  Future<Result<List<Task>>> call({
    int page = 1,
    int limit = 20,
    TaskFilter? filter,
  }) {
    return _repository.getTasks(page: page, limit: limit, filter: filter);
  }
}

class CreateTaskUseCase {
  final TaskRepository _repository;

  const CreateTaskUseCase(this._repository);

  Future<Result<Task>> call(CreateTaskParams params) {
    // Có thể thêm validation logic ở đây
    if (params.title.length < 3) {
      return Future.value(
        const Result.failure(
          Failure.validation(
            message: 'Title too short',
            errors: {'title': ['Title must be at least 3 characters']},
          ),
        ),
      );
    }
    return _repository.createTask(params);
  }
}
```

**Khi nào cần UseCase:**
- Có business logic cần xử lý trước khi gọi repository (validation, transformation)
- Cần combine data từ nhiều repositories
- Muốn tách biệt rõ ràng cho testing

**Khi nào có thể bỏ qua UseCase:**
- CRUD đơn giản, chỉ pass-through đến repository
- Project nhỏ, không cần abstraction layer thêm
- Team đồng ý dùng repository trực tiếp từ BLoC/Provider

### 5.4. Dependency Injection Pattern

**Cách 1: Với get_it**
```dart
// di/injection.dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

Future<void> setupDI() async {
  // Network
  getIt.registerLazySingleton<Dio>(() => createDioClient());
  getIt.registerLazySingleton<NetworkInfo>(() => NetworkInfoImpl());

  // Datasources
  getIt.registerLazySingleton<TaskRemoteDatasource>(
    () => TaskRemoteDatasource(getIt<Dio>()),
  );
  getIt.registerLazySingleton<TaskLocalDatasource>(
    () => TaskLocalDatasource(),
  );

  // Repositories
  getIt.registerLazySingleton<TaskRepository>(
    () => TaskRepositoryImpl(
      getIt<TaskRemoteDatasource>(),
      getIt<TaskLocalDatasource>(),
      getIt<NetworkInfo>(),
    ),
  );

  // UseCases
  getIt.registerFactory(() => GetTasksUseCase(getIt<TaskRepository>()));
  getIt.registerFactory(() => CreateTaskUseCase(getIt<TaskRepository>()));

  // BLoCs
  getIt.registerFactory(() => TaskListBloc(
        getIt<GetTasksUseCase>(),
        getIt<DeleteTaskUseCase>(),
      ));
}
```

**Cách 2: Với Riverpod (không cần get_it)**
```dart
// features/tasks/presentation/providers/task_providers.dart

final dioProvider = Provider<Dio>((ref) => createDioClient());

final taskRemoteDatasourceProvider = Provider<TaskRemoteDatasource>(
  (ref) => TaskRemoteDatasource(ref.read(dioProvider)),
);

final taskLocalDatasourceProvider = Provider<TaskLocalDatasource>(
  (ref) => TaskLocalDatasource(),
);

final taskRepositoryProvider = Provider<TaskRepository>(
  (ref) => TaskRepositoryImpl(
    ref.read(taskRemoteDatasourceProvider),
    ref.read(taskLocalDatasourceProvider),
    ref.read(networkInfoProvider),
  ),
);

final taskListProvider =
    StateNotifierProvider<TaskListNotifier, TaskListState>(
  (ref) => TaskListNotifier(ref.read(taskRepositoryProvider)),
);
```

---

## Phụ Lục: Checklist Trước Khi Submit Capstone

- [ ] Clean Architecture: 3 layers rõ ràng (Domain, Data, Presentation)
- [ ] Không có dependency ngược (Domain không import Data/Presentation)
- [ ] State management nhất quán (BLoC hoặc Riverpod xuyên suốt)
- [ ] Error handling ở mọi API call
- [ ] Offline support: hiển thị cached data khi mất mạng
- [ ] Unit tests cho UseCases/Repositories ≥ 60% coverage
- [ ] Widget tests cho ít nhất 3 screens chính
- [ ] `flutter analyze` không có warnings
- [ ] `dart format .` đã chạy
- [ ] README.md có hướng dẫn setup, run, architecture overview
- [ ] Không hardcode API keys, secrets trong source code
- [ ] Git history sạch, commit messages rõ ràng

---

## 📚 Tài liệu liên quan

| Tài liệu | Mô tả |
|---|---|
| [README — Tổng quan chương trình](../README.md) | Cài đặt môi trường, lộ trình 16 buổi, hướng dẫn sử dụng |
| [Tiêu chuẩn Middle Developer](../tieu-chuan/middle-level-rubric.md) | Rubric đánh giá năng lực Middle Flutter Developer |
| [AI-Driven Development](../ai-toolkit/ai-driven-development.md) | Hướng dẫn sử dụng AI tools trong phát triển Flutter |
| [Vận hành nhóm học](../van-hanh-nhom/study-group-operations.md) | Quy trình tổ chức buổi học peer-to-peer |

---

*Tài liệu thuộc chương trình Flutter Training. Cập nhật lần cuối: 2026-03-31.*
