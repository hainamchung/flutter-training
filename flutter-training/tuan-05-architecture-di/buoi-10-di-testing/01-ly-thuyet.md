# Buổi 10: Dependency Injection & Testing — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 10/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 09 (lý thuyết + ít nhất BT1-BT2)

## Mục lục

1. [DI là gì? Tại sao cần trong Flutter?](#1-di-là-gì-tại-sao-cần-trong-flutter)
2. [get_it + injectable](#2-get_it--injectable)
3. [Testing Strategy](#3-testing-strategy)
4. [Mocking với mocktail](#4-mocking-với-mocktail)
5. [Code Generation](#5-code-generation)
6. [Best Practices & Lỗi thường gặp](#6-best-practices--lỗi-thường-gặp)
7. [💡 FE → Flutter: Góc nhìn chuyển đổi](#7--fe--flutter-góc-nhìn-chuyển-đổi)
8. [Tổng kết](#8-tổng-kết)

---

## 1. DI là gì? Tại sao cần trong Flutter? 🔴

### 1.1 Khái niệm Dependency Injection

**Dependency Injection (DI)** là một design pattern trong đó một object nhận dependencies từ bên ngoài thay vì tự tạo chúng.

```
❌ KHÔNG có DI (tight coupling):
┌─────────────┐
│  NotesBloc   │──── tự tạo ──▶ NotesRepositoryImpl()
│              │──── tự tạo ──▶ ApiClient()
└─────────────┘

✅ CÓ DI (loose coupling):
┌─────────────┐
│  NotesBloc   │◀── inject ── NotesRepository (interface)
└─────────────┘
       ▲
       │ inject bởi DI Container
       │
┌─────────────────┐
│   DI Container   │── biết cách tạo NotesRepositoryImpl
│   (get_it)       │── biết cách tạo ApiClient
└─────────────────┘
```

### 1.2 Tại sao cần DI trong Flutter?

**a) Decouple các layer trong Clean Architecture:**

Ở buổi 09, chúng ta đã tách ứng dụng thành 3 layer. DI là "keo" nối chúng lại mà không tạo coupling:

```dart
// ❌ Không DI — Domain layer phụ thuộc trực tiếp vào Data layer
class GetNotesUseCase {
  // Vi phạm dependency rule!
  final repo = NotesRepositoryImpl(NotesRemoteDataSource(ApiClient()));
  
  Future<List<Note>> call() => repo.getNotes();
}

// ✅ Có DI — Domain layer chỉ phụ thuộc vào interface
class GetNotesUseCase {
  final NotesRepository repo; // Interface, không phải implementation
  
  GetNotesUseCase(this.repo); // Inject từ bên ngoài
  
  Future<List<Note>> call() => repo.getNotes();
}
```

**b) Enable Testing:**

Khi dependencies được inject, ta có thể thay thế bằng mock object khi test:

```dart
// Trong production: dùng real repository
final useCase = GetNotesUseCase(NotesRepositoryImpl(...));

// Trong test: dùng mock repository
final useCase = GetNotesUseCase(MockNotesRepository());
```

**c) Swap implementations dễ dàng:**

```dart
// Development: dùng local data source
getIt.registerSingleton<NotesDataSource>(LocalNotesDataSource());

// Production: dùng remote data source
getIt.registerSingleton<NotesDataSource>(RemoteNotesDataSource(apiClient));
```

### 1.3 Service Locator Pattern

**Service Locator** là một pattern mà trong đó có một "registry" trung tâm biết cách tạo và cung cấp dependencies.

```
┌──────────────────────────────┐
│      Service Locator          │
│  ┌────────────────────────┐  │
│  │ ApiClient ──▶ instance │  │
│  │ UserRepo  ──▶ instance │  │
│  │ NotesRepo ──▶ instance │  │
│  │ GetNotes  ──▶ instance │  │
│  │ NotesBloc ──▶ factory  │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
         ▲          ▲
         │          │
    NotesBloc    NotesPage
    (resolve)    (resolve)
```

> **Lưu ý:** Service Locator khác với "pure" DI (constructor injection). Trong Flutter, get_it là Service Locator nhưng khi kết hợp injectable, nó hoạt động tương tự DI container.

> 🔗 **FE Bridge:** DI concept **tương đồng** — inject dependencies thay vì hardcode. Nhưng **khác ở**: FE ít dùng DI container (Angular là ngoại lệ), thường import trực tiếp. Flutter dùng **get_it** (service locator) hoặc **injectable** (code gen DI) — giống Angular DI pattern hơn React/Vue pattern.

### 1.4 Manual DI vs DI Framework

**Manual DI** — tự viết code tạo và truyền dependencies:

```dart
// manual_injection.dart
NotesBloc createNotesBloc() {
  final apiClient = ApiClient(baseUrl: 'https://api.example.com');
  final dataSource = NotesRemoteDataSource(apiClient);
  final repository = NotesRepositoryImpl(dataSource);
  final getNotesUseCase = GetNotesUseCase(repository);
  return NotesBloc(getNotesUseCase);
}
```

| | Manual DI | DI Framework (get_it + injectable) |
|---|-----------|-------------------------------------|
| Setup | Không cần package | Cần thêm packages + build_runner |
| Boilerplate | Nhiều code lặp lại | Annotations ngắn gọn |
| Maintainability | Khó maintain khi app lớn | Tự generate, dễ maintain |
| Compile-time safety | ✅ Type-safe | ✅ Type-safe (generated code) |
| Learning curve | Thấp | Trung bình |

**Kết luận:** Với app nhỏ, manual DI đủ dùng. Khi app phức tạp (nhiều dependencies), dùng get_it + injectable để giảm boilerplate.

---

## 2. get_it + injectable 🟡

### 2.1 get_it — Service Locator cho Dart

**get_it** là một Service Locator đơn giản, nhanh và type-safe cho Dart/Flutter.

**Cài đặt:**

```yaml
# pubspec.yaml
dependencies:
  get_it: ^7.6.4
```

**Các cách đăng ký dependency:**

```dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setupDependencies() {
  // 1. registerSingleton — tạo ngay, dùng chung 1 instance
  // Dùng cho: ApiClient, Database, SharedPreferences
  getIt.registerSingleton<ApiClient>(ApiClient(baseUrl: 'https://api.example.com'));

  // 2. registerLazySingleton — tạo khi cần lần đầu, sau đó dùng lại
  // Dùng cho: Repository, DataSource (khởi tạo tốn resource)
  getIt.registerLazySingleton<NotesRepository>(
    () => NotesRepositoryImpl(getIt<NotesRemoteDataSource>()),
  );

  // 3. registerFactory — tạo instance MỚI mỗi lần resolve
  // Dùng cho: Bloc, Cubit (mỗi screen cần instance riêng)
  getIt.registerFactory<NotesBloc>(
    () => NotesBloc(getIt<GetNotesUseCase>()),
  );
}
```

**Resolve dependency:**

```dart
// Lấy instance từ get_it
final notesBloc = getIt<NotesBloc>(); // Tạo mới vì là Factory
final apiClient = getIt<ApiClient>(); // Trả về singleton
```

**So sánh 3 cách đăng ký:**

```
registerSingleton     ──▶ Tạo NGAY khi app start ──▶ Luôn cùng 1 instance
registerLazySingleton ──▶ Tạo KHI CẦN lần đầu   ──▶ Luôn cùng 1 instance
registerFactory       ──▶ Tạo MỚI mỗi lần gọi   ──▶ Instance khác nhau
```

### 2.2 injectable — Code Generation cho DI

Viết tay getIt.register... cho mỗi class rất tedious. **injectable** dùng annotations để tự generate code đăng ký.

**Cài đặt:**

```yaml
# pubspec.yaml
dependencies:
  get_it: ^7.6.4
  injectable: ^2.3.0

dev_dependencies:
  injectable_generator: ^2.3.0
  build_runner: ^2.4.0
```

**Các annotations chính:**

| Annotation | Tương đương get_it | Dùng cho |
|------------|---------------------|----------|
| `@injectable` | `registerFactory` | Bloc, Cubit, UseCase |
| `@singleton` | `registerSingleton` | ApiClient, Database |
| `@lazySingleton` | `registerLazySingleton` | Repository, DataSource |
| `@module` | N/A — nhóm external deps | Third-party packages |
| `@Environment('dev')` | N/A — conditional register | Dev/Prod switching |

### 2.3 Setup injectable

**Bước 1: Tạo injection.dart**

```dart
// lib/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';

import 'injection.config.dart'; // File sẽ được generate

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();
```

**Bước 2: Annotate các class**

```dart
// Data layer
@lazySingleton
class NotesRemoteDataSource {
  final ApiClient apiClient;
  NotesRemoteDataSource(this.apiClient);

  Future<List<NoteModel>> getNotes() async {
    final response = await apiClient.get('/notes');
    return (response as List).map((e) => NoteModel.fromJson(e)).toList();
  }
}

// Đăng ký interface ──▶ implementation
@LazySingleton(as: NotesRepository)
class NotesRepositoryImpl implements NotesRepository {
  final NotesRemoteDataSource remoteDataSource;
  NotesRepositoryImpl(this.remoteDataSource);

  @override
  Future<List<Note>> getNotes() async {
    final models = await remoteDataSource.getNotes();
    return models.map((m) => m.toDomain()).toList();
  }
}

// Domain layer
@injectable
class GetNotesUseCase {
  final NotesRepository repository;
  GetNotesUseCase(this.repository);

  Future<List<Note>> call() => repository.getNotes();
}

// Presentation layer
@injectable
class NotesBloc extends Bloc<NotesEvent, NotesState> {
  final GetNotesUseCase getNotesUseCase;
  NotesBloc(this.getNotesUseCase) : super(NotesInitial());
}
```

**Bước 3: Đăng ký external dependencies với @module**

```dart
// lib/injection/register_module.dart
@module
abstract class RegisterModule {
  @singleton
  ApiClient get apiClient => ApiClient(baseUrl: 'https://api.example.com');

  @preResolve // Cho async dependencies
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}
```

**Bước 4: Chạy build_runner**

```bash
dart run build_runner build --delete-conflicting-outputs
```

File `injection.config.dart` sẽ được generate tự động với tất cả registrations.

**Bước 5: Gọi trong main()**

```dart
void main() {
  configureDependencies();
  runApp(const MyApp());
}
```

> 🔗 **FE Bridge:** `GetIt` ≈ Angular `Injector` hoặc InversifyJS container — register service → resolve by type. Nhưng **khác ở**: `get_it` = **runtime service locator** (không injection), `injectable` generates registration code. FE dev quen direct import → cần shift sang "register once, resolve anywhere" mindset.

### 2.4 Environment — Switching Dev/Prod

```dart
// Chỉ đăng ký trong môi trường 'dev'
@Environment('dev')
@LazySingleton(as: NotesDataSource)
class FakeNotesDataSource implements NotesDataSource {
  @override
  Future<List<NoteModel>> getNotes() async {
    return [NoteModel(id: '1', title: 'Fake note', content: 'Test content')];
  }
}

// Chỉ đăng ký trong môi trường 'prod'
@Environment('prod')
@LazySingleton(as: NotesDataSource)
class RemoteNotesDataSource implements NotesDataSource {
  final ApiClient apiClient;
  RemoteNotesDataSource(this.apiClient);

  @override
  Future<List<NoteModel>> getNotes() async {
    final response = await apiClient.get('/notes');
    return (response as List).map((e) => NoteModel.fromJson(e)).toList();
  }
}

// injection.dart — chọn environment
@InjectableInit()
void configureDependencies({String environment = 'prod'}) =>
    getIt.init(environment: environment);

// main_dev.dart
void main() {
  configureDependencies(environment: 'dev');
  runApp(const MyApp());
}
```

---

> 💼 **Gặp trong dự án:** Setup DI cho project có 20+ services, register đúng lifecycle (singleton vs factory vs lazy), environment switching (dev mock / prod real API)
> 🤖 **Keywords bắt buộc trong prompt:** `get_it`, `injectable`, `@singleton`, `@lazySingleton`, `@injectable`, `@module`, `@Environment`, `configureDependencies()`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **New project:** Setup DI container cho toàn bộ app — Repository, DataSource, UseCase, Bloc — đúng lifecycle
- **Dev/Prod:** Team cần MockAuthRepository khi dev, RealAuthRepository khi production
- **Lifecycle bug:** Register ApiClient là factory thay vì singleton → mỗi screen tạo connection mới

**Tại sao cần các keyword trên:**
- **`@singleton`** — 1 instance duy nhất toàn app (ApiClient, Database), AI hay dùng factory (sai)
- **`@lazySingleton`** — singleton nhưng chỉ tạo khi cần lần đầu (tối ưu startup time)
- **`@injectable`** — tự động register, AI phải annotate đúng class
- **`@module`** — register third-party packages (Dio, SharedPreferences) không thể annotate source code
- **`@Environment`** — switch dev/prod, AI hay hardcode thay vì dùng environment

**Prompt mẫu — DI Setup:**
```text
Tôi cần setup get_it + injectable cho Flutter project theo Clean Architecture.
Tech stack: Flutter 3.x, get_it ^7.x, injectable ^2.x, injectable_generator.
Services cần register:
1. Dio (HTTP client) — @singleton, cần @module vì third-party.
2. SharedPreferences — @preResolve (async init), @module.
3. UserRemoteDataSource — @lazySingleton, depends on Dio.
4. UserLocalDataSource — @lazySingleton, depends on SharedPreferences.
5. UserRepositoryImpl — @LazySingleton(as: UserRepository), depends on cả 2 DataSources.
6. GetUserProfile UseCase — @injectable, depends on UserRepository.
7. UserBloc — @injectable, depends on GetUserProfile.
Constraints:
- 2 environments: 'dev' (MockUserRemoteDataSource) và 'prod' (RealUserRemoteDataSource).
- @module class cho third-party dependencies.
- configureDependencies() gọi trong main().
Output: injection.dart + injection.config.dart (mock) + register_module.dart + annotation trên mỗi class.
```

**Expected Output:** AI gen DI setup + class annotations.

⚠️ **Giới hạn AI hay mắc:** AI hay quên `@preResolve` cho async dependencies (SharedPreferences). AI cũng hay dùng `@singleton` cho Bloc (SAI — Bloc có lifecycle, phải dùng `@injectable` factory).

</details>

---

## 3. Testing Strategy 🔴

### 3.1 Test Pyramid

```
        ╱╲
       ╱  ╲         Integration Tests (ít)
      ╱    ╲        → Test toàn bộ flow
     ╱──────╲       → Chạy chậm, dễ flaky
    ╱        ╲
   ╱          ╲      Widget Tests (vừa)
  ╱            ╲     → Test UI components
 ╱──────────────╲    → Chạy nhanh hơn integration
╱                ╲
╱                  ╲   Unit Tests (nhiều)
╱────────────────────╲  → Test logic thuần
                        → Chạy rất nhanh
```

| Level | Test gì? | Tốc độ | Số lượng | Ví dụ |
|-------|----------|--------|----------|-------|
| **Unit** | Logic thuần, không UI | ⚡ Rất nhanh | Nhiều nhất | UseCase, Repository, Bloc, Utils |
| **Widget** | UI component đơn lẻ | 🔵 Nhanh | Vừa phải | NotesListPage, NoteCard, LoginForm |
| **Integration** | Toàn bộ user flow | 🐌 Chậm | Ít nhất | Login → Add note → View note |

### 3.2 Testing trong Clean Architecture

```
┌─────────────────────────────────────────────┐
│              Presentation Layer              │
│   Widget Test: NotesListPage, NoteCard       │
│   Unit Test: NotesBloc, NotesCubit           │
├─────────────────────────────────────────────┤
│              Domain Layer                    │
│   Unit Test: GetNotesUseCase, entities       │
├─────────────────────────────────────────────┤
│              Data Layer                      │
│   Unit Test: NotesRepositoryImpl,            │
│              NoteModel.fromJson/toJson       │
└─────────────────────────────────────────────┘

Integration Test: Xuyên suốt tất cả layers
```

**Nguyên tắc mock theo layer:**

- Test **UseCase** → mock **Repository** (interface)
- Test **Repository** → mock **DataSource**
- Test **Bloc/Cubit** → mock **UseCase**
- Test **Widget** → mock **Bloc/Cubit**

### 3.3 Test nào nên viết trước?

1. **Unit test cho Domain layer** — logic quan trọng nhất, ít thay đổi
2. **Unit test cho Data layer** — đảm bảo mapping data đúng
3. **Unit test cho Bloc/Cubit** — kiểm tra state transitions
4. **Widget test cho screen chính** — đảm bảo UI hiển thị đúng
5. **Integration test cho flow quan trọng** — happy path của feature chính

> 💼 **Gặp trong dự án:** Testing legacy code không có DI rất khó — class phụ thuộc trực tiếp vào network/database. Giải pháp: wrap dependency bằng interface, inject qua constructor, rồi mock trong test. Đây là pattern "extract and wrap" rất phổ biến khi refactor.

> 🔗 **FE Bridge:** Testing pyramid **giống hệt** FE: Unit → Widget (≈ Component test) → Integration (≈ E2E). `flutter test` ≈ `jest`/`vitest`. Widget testing dùng `pumpWidget()` ≈ React Testing Library `render()`. Nhưng **khác ở**: Flutter widget test cần `tester.pump()` để advance time/animation — React auto-flushes.

---

## 4. Mocking với mocktail 🟡

### So sánh mocktail vs mockito

| Tiêu chí | mocktail | mockito |
|----------|----------|----------|
| Code generation | ❌ Không cần | ✅ Cần `build_runner` |
| Cú pháp | `when(() => mock.method()).thenReturn(...)` | `when(mock.method()).thenReturn(...)` |
| Null safety | ✅ Native | ✅ Từ v5+ |
| Popularity | Đang tăng nhanh | Lâu đời, phổ biến hơn |
| Recommendation | ✅ **Training này dùng mocktail** | Dùng khi dự án đã có sẵn |

> 💡 **Tại sao chọn mocktail?** Không cần code generation → setup nhanh hơn, CI nhanh hơn. API tương tự mockito nên dễ switch.

### 4.1 Tại sao mocktail?

**mocktail** là package mocking **không cần code generation**, hỗ trợ Dart 3 null-safety đầy đủ. So với **mockito**, mocktail đơn giản hơn và không cần annotations hay build_runner.

```yaml
# pubspec.yaml
dev_dependencies:
  mocktail: ^1.0.4
  bloc_test: ^9.1.1  # Hữu ích nếu dùng BLoC
```

### 4.2 Tạo Mock class

```dart
import 'package:mocktail/mocktail.dart';

// Tạo mock cho interface/abstract class
class MockNotesRepository extends Mock implements NotesRepository {}
class MockGetNotesUseCase extends Mock implements GetNotesUseCase {}
class MockApiClient extends Mock implements ApiClient {}
```

**Không cần code gen!** Chỉ cần `extends Mock` + `implements <Interface>`.

### 4.3 Các hàm quan trọng

**`when()` — Định nghĩa behavior khi method được gọi:**

```dart
final mockRepo = MockNotesRepository();

// Trả về giá trị
when(() => mockRepo.getNotes()).thenAnswer(
  (_) async => [Note(id: '1', title: 'Test', content: 'Content')],
);

// Throw exception
when(() => mockRepo.getNotes()).thenThrow(ServerException('Error'));

// Trả về giá trị khác nhau theo lần gọi
when(() => mockRepo.deleteNote(any())).thenAnswer((_) async {});
```

**`verify()` — Kiểm tra method có được gọi không:**

```dart
// Kiểm tra đã gọi getNotes() đúng 1 lần
verify(() => mockRepo.getNotes()).called(1);

// Kiểm tra KHÔNG gọi deleteNote
verifyNever(() => mockRepo.deleteNote(any()));
```

**`any()` — Match bất kỳ argument nào:**

```dart
when(() => mockRepo.getNoteById(any())).thenAnswer(
  (_) async => Note(id: '1', title: 'Test', content: 'Content'),
);

verify(() => mockRepo.getNoteById('1')).called(1);
```

### 4.4 Mock vs Fake vs Stub

| Khái niệm | Mô tả | Khi dùng |
|-----------|--------|----------|
| **Mock** | Object giả, verify interactions | Kiểm tra method có được gọi đúng |
| **Fake** | Implementation thật nhưng đơn giản | Thay thế DB bằng in-memory list |
| **Stub** | Trả về giá trị cố định | Luôn trả về data cụ thể |

```dart
// Mock — dùng mocktail
class MockNotesRepo extends Mock implements NotesRepository {}

// Fake — implementation đơn giản
class FakeNotesRepo implements NotesRepository {
  final List<Note> _notes = [];

  @override
  Future<List<Note>> getNotes() async => _notes;

  @override
  Future<void> addNote(Note note) async => _notes.add(note);

  @override
  Future<void> deleteNote(String id) async =>
      _notes.removeWhere((n) => n.id == id);
}

// Stub — trả về giá trị cố định (dùng Mock + when)
final stubRepo = MockNotesRepo();
when(() => stubRepo.getNotes()).thenAnswer(
  (_) async => [Note(id: '1', title: 'Stub Note', content: 'Fixed content')],
);
```

### 4.5 Cấu trúc test file

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// Mocks
class MockNotesRepository extends Mock implements NotesRepository {}

void main() {
  // Khai báo late variables
  late MockNotesRepository mockRepository;
  late GetNotesUseCase useCase;

  // setUp chạy TRƯỚC MỖI test
  setUp(() {
    mockRepository = MockNotesRepository();
    useCase = GetNotesUseCase(mockRepository);
  });

  // tearDown chạy SAU MỖI test (nếu cần cleanup)
  tearDown(() {
    // Cleanup nếu cần
  });

  group('GetNotesUseCase', () {
    test('should return list of notes from repository', () async {
      // Arrange — chuẩn bị data và mock
      final notes = [Note(id: '1', title: 'Test', content: 'Content')];
      when(() => mockRepository.getNotes()).thenAnswer((_) async => notes);

      // Act — thực hiện action
      final result = await useCase.call();

      // Assert — kiểm tra kết quả
      expect(result, equals(notes));
      verify(() => mockRepository.getNotes()).called(1);
    });
  });
}
```

### Test Coverage

#### Chạy coverage report

```bash
# Generate coverage
flutter test --coverage

# Xem report (cần cài lcov)
# macOS: brew install lcov
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

#### Coverage thresholds

| Level | Target | Giải thích |
|-------|--------|------------|
| Unit tests | ≥ 80% | Business logic, use cases |
| Widget tests | ≥ 60% | UI components chính |
| Integration tests | Core flows | Happy path + edge cases |

> ⚠️ **Coverage ≠ Quality**. 100% coverage không có nghĩa code đúng. Focus vào test behavior, không test implementation.

> 🔗 **FE Bridge:** `mocktail` ≈ `jest.mock()` / `vitest.mock()` — mock dependency để isolate unit test. `when().thenReturn()` ≈ `jest.fn().mockReturnValue()`. Nhưng **khác ở**: Dart mock extends class (OOP approach), JS mock replaces module (functional approach).

---

> 💼 **Gặp trong dự án:** Viết unit tests cho UseCase + Repository (mock dependencies), tìm missing test cases, coverage report phân tích, team cần 80%+ coverage cho merge request
> 🤖 **Keywords bắt buộc trong prompt:** `mocktail`, `when().thenAnswer`, `verify()`, `group + setUp + tearDown`, `coverage report`, `edge cases`, `parametrized tests`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Code review:** Senior yêu cầu unit tests cho mỗi UseCase + Repository implementation trước merge
- **Coverage gap:** CI report thấy coverage 45% — cần viết tests cho uncovered branches
- **Mock setup:** Repository depends on RemoteDataSource + LocalDataSource — cần mock cả 2

**Tại sao cần các keyword trên:**
- **`mocktail`** — mocking library không cần code generation (unlike mockito)
- **`when().thenAnswer`** — setup mock behavior cho async methods, AI hay dùng `thenReturn` cho Future (SAI)
- **`verify()`** — kiểm tra method có được gọi đúng số lần
- **`edge cases`** — AI hay chỉ test happy path, thiếu error cases

**Prompt mẫu — Gen test suite:**
```text
Tôi cần viết unit tests cho User feature (Clean Architecture).
Tech stack: Flutter 3.x, flutter_test, mocktail ^1.x.
Classes cần test:
1. GetUserProfile UseCase — depends on UserRepository (abstract).
2. UserRepositoryImpl — depends on UserRemoteDataSource, UserLocalDataSource.

Test requirements:
- Mock: MockUserRepository, MockUserRemoteDataSource, MockUserLocalDataSource (dùng mocktail).
- GetUserProfile tests:
  a. Success: repository returns UserEntity → usecase returns Right(user).
  b. Failure: repository throws → usecase returns Left(ServerFailure).
  c. Verify: repository.getUser() called exactly once.
- UserRepositoryImpl tests:
  a. Remote success: fetch → cache → return Right(user).
  b. Remote fail + cache hit: return Right(cachedUser).
  c. Remote fail + cache miss: return Left(ServerFailure).
  d. Verify: localDataSource.cacheUser() called on remote success.
- Identify 3 MISSING test cases tôi chưa nghĩ tới.
Output: get_user_profile_test.dart + user_repository_impl_test.dart.
```

**Expected Output:** AI gen 2 test files + 3 suggested missing test cases.

⚠️ **Giới hạn AI hay mắc:** AI hay dùng `when(() => mock.method()).thenReturn(future)` thay vì `thenAnswer((_) async => value)` cho async methods. AI cũng hay thiếu `setUp` + `tearDown` cho mock initialization.

</details>

---

## 5. Code Generation 🟢

### 5.1 build_runner — Bộ máy code generation của Dart

**Tại sao Dart cần code generation?**

Khác với JavaScript/TypeScript (có runtime reflection), Dart (đặc biệt khi compile AOT cho Flutter) **không hỗ trợ runtime reflection**. Do đó, nhiều tác vụ cần code gen tại compile-time:

```
JavaScript/TypeScript          Dart/Flutter
─────────────────              ─────────────
Runtime reflection     →       Không có (AOT compiled)
Decorators at runtime  →       Annotations + build_runner
Proxy objects         →        Generated classes
Dynamic JSON parsing  →        Generated fromJson/toJson
```

**Cách chạy build_runner:**

```bash
# Build 1 lần
dart run build_runner build --delete-conflicting-outputs

# Watch mode — tự rebuild khi file thay đổi
dart run build_runner watch --delete-conflicting-outputs
```

**Workflow:**

```
Bạn viết         build_runner        Bạn sử dụng
annotations  ──▶  generates    ──▶   generated code
(@freezed,       (.g.dart,           (fromJson, copyWith,
 @injectable,     .freezed.dart,      DI registration,
 @JsonSerializable) .config.dart)     type unions)
```

### 5.2 freezed — Immutable Data Classes

**freezed** tạo immutable data classes với `copyWith`, `==`, `hashCode`, `toString`, và union types.

> 📖 **JSON serialization chi tiết** sẽ được học ở [Buổi 11 — Networking](../../tuan-06-networking-data/buoi-11-networking/01-ly-thuyet.md). Ở đây chỉ cần biết dùng `freezed` để tạo immutable data models cho testing.

**Cài đặt:**

```yaml
dependencies:
  freezed_annotation: ^2.4.0

dev_dependencies:
  freezed: ^2.4.0
  build_runner: ^2.4.0
```

**Tạo immutable class:**

```dart
// lib/domain/entities/note.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'note.freezed.dart';

@freezed
class Note with _$Note {
  const factory Note({
    required String id,
    required String title,
    required String content,
    @Default(false) bool isPinned,
    DateTime? createdAt,
  }) = _Note;
}
```

> ⚠️ **Lưu ý**: Buổi 09 khuyến nghị domain entities nên pure Dart. Dùng `@freezed` cho domain entities là cách tiếp cận pragmatic — đánh đổi pure domain để giảm boilerplate. Trong dự án thực tế, đây là lựa chọn phổ biến và chấp nhận được.

**Sau khi chạy build_runner, bạn có:**

```dart
// copyWith — tạo copy với một số field thay đổi
final note = Note(id: '1', title: 'Hello', content: 'World');
final updated = note.copyWith(title: 'Updated Title');
// Note(id: '1', title: 'Updated Title', content: 'World', isPinned: false)

// == và hashCode — so sánh theo value
final note1 = Note(id: '1', title: 'A', content: 'B');
final note2 = Note(id: '1', title: 'A', content: 'B');
print(note1 == note2); // true
```

**Union types (sealed class) với freezed:**

```dart
@freezed
class NotesState with _$NotesState {
  const factory NotesState.initial() = NotesInitial;
  const factory NotesState.loading() = NotesLoading;
  const factory NotesState.loaded(List<Note> notes) = NotesLoaded;
  const factory NotesState.error(String message) = NotesError;
}

// Sử dụng pattern matching
Widget build(BuildContext context) {
  return state.when(
    initial: () => const SizedBox.shrink(),
    loading: () => const CircularProgressIndicator(),
    loaded: (notes) => NotesList(notes: notes),
    error: (message) => ErrorWidget(message: message),
  );
}
```

### 5.3 Tổng quan Code Gen Packages

| Package | Tạo gì? | Dùng cho |
|---------|---------|----------|
| **freezed** | `.freezed.dart` — copyWith, ==, when/map | Domain entities, State classes |
| **json_serializable** | `.g.dart` — fromJson, toJson | Data models — xem [Buổi 11](../../tuan-06-networking-data/buoi-11-networking/01-ly-thuyet.md) |
| **injectable_generator** | `.config.dart` — DI registration | Dependency injection setup |
| **build_runner** | Orchestrator — chạy tất cả generators | N/A — là tool, không phải generator |

### 5.4 File structure sau code gen

```
lib/
├── domain/entities/
│   ├── note.dart              ← Bạn viết
│   └── note.freezed.dart      ← Generated (copyWith, ==, when)
├── data/models/
│   ├── note_model.dart        ← Bạn viết
│   └── note_model.g.dart      ← Generated (fromJson, toJson — xem Buổi 11)
├── injection.dart             ← Bạn viết
└── injection.config.dart      ← Generated (DI registrations)
```

> **Quy tắc:** Không bao giờ edit file `.g.dart`, `.freezed.dart`, `.config.dart`. Chúng sẽ bị overwrite khi chạy build_runner.

---

## 6. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

**DI:**

| Practice | Giải thích |
|----------|-----------|
| Đăng ký interface, không phải implementation | `@LazySingleton(as: NotesRepository)` thay vì `@lazySingleton` trên impl |
| Dùng `registerFactory` cho Bloc/Cubit | Mỗi screen cần instance mới để tránh state leak |
| Dùng `registerLazySingleton` cho Repository | Khởi tạo tốn resource, chỉ cần 1 instance |
| Gọi `configureDependencies()` đầu tiên trong `main()` | Trước `runApp()` để đảm bảo mọi DI đã sẵn sàng |

**Testing:**

| Practice | Giải thích |
|----------|-----------|
| Arrange-Act-Assert pattern | Mỗi test gồm 3 phần rõ ràng |
| Một assertion per test | Mỗi test kiểm tra 1 điều duy nhất |
| Test tên mô tả rõ ràng | `'should return notes when repository succeeds'` |
| setUp() cho common setup | Tránh copy-paste setup code |
| Mock ở đúng boundary | Mock interface, không mock implementation |

**Code Gen:**

| Practice | Giải thích |
|----------|-----------|
| Thêm `*.g.dart`, `*.freezed.dart` vào `.gitignore` (hoặc không) | Team nên thống nhất: commit hay generate lại |
| Dùng `--delete-conflicting-outputs` | Tránh conflict khi regenerate |
| Watch mode khi develop | `dart run build_runner watch` để auto-rebuild |

### ❌ Lỗi thường gặp

```dart
// ❌ Sai: Quên chạy build_runner sau khi thêm annotations
// Error: "injection.config.dart not found"
// Fix: chạy `dart run build_runner build --delete-conflicting-outputs`

// ❌ Sai: Đăng ký implementation thay vì interface
@lazySingleton  // Đăng ký NotesRepositoryImpl
class NotesRepositoryImpl implements NotesRepository { ... }
// Fix:
@LazySingleton(as: NotesRepository)  // Đăng ký với interface
class NotesRepositoryImpl implements NotesRepository { ... }

// ❌ Sai: Quên đăng ký dependency mà injectable cần
// Error: "Object/factory with type X is not registered inside GetIt"
// Fix: Đảm bảo tất cả dependencies trong constructor đều được đăng ký

// ❌ Sai: Dùng registerSingleton cho Bloc
getIt.registerSingleton<NotesBloc>(NotesBloc(...));
// Tất cả screen chia sẻ cùng 1 Bloc instance → state leak!
// Fix: dùng registerFactory cho Bloc/Cubit

// ❌ Sai: Quên `part` directive cho generated files
// File note.dart thiếu:
part 'note.freezed.dart';
part 'note.g.dart';

// ❌ Sai: Mock concrete class thay vì interface
class MockNotesRepoImpl extends Mock implements NotesRepositoryImpl {}
// Fix: Mock interface
class MockNotesRepo extends Mock implements NotesRepository {}
```

---

## 7. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

> Nếu bạn đến từ React/Vue/Angular, phần này giúp map kiến thức cũ sang Flutter.

### DI Framework

| Flutter (get_it + injectable) | Frontend tương đương |
|-------------------------------|----------------------|
| `getIt.registerSingleton<T>()` | Angular: `providers: [{ provide: T, useValue: ... }]` |
| `getIt.registerFactory<T>()` | Angular: `providers: [{ provide: T, useFactory: ... }]` |
| `@injectable` annotation | Angular: `@Injectable()` decorator |
| `getIt<NotesRepository>()` | Angular: `inject(NotesRepository)` |
| **Không có built-in DI** (cần package) | Angular: DI built-in; React: không có DI pattern chuẩn |

> **React dev?** React không có DI framework chuẩn. Thường dùng Context API hoặc truyền props. get_it giống concept của "service container" — nơi bạn đăng ký và resolve services.

### Mocking

| mocktail (Flutter) | Jest (JavaScript) |
|--------------------|-------------------|
| `class MockRepo extends Mock implements Repo {}` | `jest.mock('./repo')` |
| `when(() => mock.method()).thenAnswer(...)` | `mockRepo.method.mockResolvedValue(...)` |
| `verify(() => mock.method()).called(1)` | `expect(mockRepo.method).toHaveBeenCalledTimes(1)` |
| `any()` | `expect.anything()` |
| **Type-safe mocking** | Dynamic mocking (JS reflection) |

### Code Generation

| Flutter | Frontend tương đương |
|---------|----------------------|
| `freezed` — immutable data + union types | **Immer** (immutable) + **TypeScript discriminated unions** |
| `json_serializable` — JSON parsing | **Zod** / **io-ts** (schema validation + parsing) |
| `build_runner` — runs code generators | **Babel** / **Webpack** (transforms code at build time) |
| `*.g.dart`, `*.freezed.dart` | Transpiled `.js` output |

> **Điểm khác biệt lớn nhất:** Trong JS/TS, bạn có runtime reflection và dynamic typing nên ít cần code gen. Dart compile AOT nên nhiều thứ phải generated tại build time.

### Testing

| Flutter | React/Vue |
|---------|-----------|
| `flutter test` | `npm test` (Jest) |
| `WidgetTester` + `pumpWidget()` | React Testing Library + `render()` |
| `find.text('Hello')` | `screen.getByText('Hello')` |
| `tester.tap(find.byType(ElevatedButton))` | `fireEvent.click(screen.getByRole('button'))` |
| `tester.pump()` — rebuild widget | `waitFor()` — wait for async updates |
| Integration test (real device/emulator) | Cypress / Playwright (real browser) |

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|--------------|
| 1 | Import module trực tiếp | Register in DI container → resolve by type | Flutter DI pattern = Angular-like, không React-like |
| 2 | `jest.mock()` mock module path | `mocktail` extends class — OOP mock approach | Dart mock = class extension, JS mock = module replacement |
| 3 | Component test = render + query DOM | Widget test = `pumpWidget()` + `find.byType()` + `tester.pump()` | Phải manually advance frame, không auto-flush |

---

## 8. Tổng kết

### ✅ Checklist kiến thức buổi 10

Sau buổi học, hãy tự kiểm tra:

**Dependency Injection:**
- [ ] Giải thích DI là gì và tại sao cần trong Flutter
- [ ] Phân biệt 3 cách đăng ký: singleton, lazySingleton, factory
- [ ] Setup get_it + injectable trong project
- [ ] Đăng ký interface thay vì implementation
- [ ] Sử dụng @module cho external dependencies
- [ ] Sử dụng @Environment để switch dev/prod

**Testing:**
- [ ] Giải thích Test Pyramid — unit, widget, integration
- [ ] Viết unit test theo Arrange-Act-Assert pattern
- [ ] Tạo mock class với mocktail
- [ ] Sử dụng `when()`, `verify()`, `any()`
- [ ] Phân biệt Mock vs Fake vs Stub
- [ ] Viết widget test với WidgetTester

**Code Generation:**
- [ ] Giải thích tại sao Dart cần code generation
- [ ] Sử dụng freezed cho immutable data classes
- [ ] Sử dụng json_serializable cho JSON parsing
- [ ] Chạy build_runner (build & watch mode)
- [ ] Hiểu workflow: annotations → build_runner → generated code

### 🔗 Kết nối kiến thức

```
Buổi 09: Clean Architecture        Buổi 10: DI & Testing
─────────────────────────          ─────────────────────
Domain ← Data ← Presentation  ──▶  get_it nối các layer
Repository interface           ──▶  Mock để test
Entity classes                 ──▶  freezed cho immutable
                                    ──▶  Buổi tiếp: Networking & API
```

---

### ➡️ Buổi tiếp theo

> **Buổi 11: Networking** — HTTP client (Dio), Interceptors, Retrofit, JSON serialization, và error handling cho network layer.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Tạo tài khoản test API tại jsonplaceholder.typicode.com
