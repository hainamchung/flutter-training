# Buổi 10: Dependency Injection & Testing — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

- [BT1 ⭐: Setup get_it + injectable](#bt1--setup-get_it--injectable)
- [BT2 ⭐⭐: Unit test Repository & UseCase](#bt2--unit-test-repository--usecase)
- [BT3 ⭐⭐⭐: Widget test + Integration test](#bt3--widget-test--integration-test)
- [Câu hỏi thảo luận](#câu-hỏi-thảo-luận)

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen testing FE**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| FE Testing Habit | Flutter Reality | Bài tập liên quan |
|------------------|-----------------|---------------------|
| `jest.mock('module')` mock by path | `mocktail`: tạo mock class extends/implements — OOP approach | BT1, BT2 |
| `render(<Component/>)` auto-complete | `tester.pumpWidget()` + `tester.pump()` — phải advance frame thủ công | BT2, BT3 |
| Direct import dependency trong test | DI override: `getIt.registerSingleton<T>(mockInstance)` | BT1, BT2 |

---

## BT1 ⭐: Setup get_it + injectable 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_di_setup` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Notes app với Dependency Injection hoạt động |
| **Dependencies** | `flutter pub add get_it injectable` + `flutter pub add dev:injectable_generator dev:build_runner dev:mocktail dev:bloc_test` |

### 🎯 Mục tiêu

Áp dụng Dependency Injection với get_it + injectable vào Notes app từ Buổi 09 (Clean Architecture).

### 📋 Yêu cầu

1. **Thêm dependencies** vào `pubspec.yaml`:
   - `get_it`, `injectable` (dependencies)
   - `injectable_generator`, `build_runner` (dev_dependencies)

2. **Tạo file `lib/injection.dart`**:
   - Import get_it và injectable
   - Khai báo `getIt = GetIt.instance`
   - Annotate với `@InjectableInit()`
   - Tạo hàm `configureDependencies()`

3. **Annotate các class** trong Notes app:

   | Class | Annotation | Lý do |
   |-------|-----------|-------|
   | `NotesLocalDataSource` | `@lazySingleton` | Chỉ cần 1 instance |
   | `NotesRepositoryImpl` | `@LazySingleton(as: NotesRepository)` | Đăng ký với interface |
   | `GetNotesUseCase` | `@injectable` | Factory — mỗi lần dùng tạo mới |
   | `AddNoteUseCase` | `@injectable` | Factory |
   | `DeleteNoteUseCase` | `@injectable` | Factory |
   | `NotesCubit` / `NotesBloc` | `@injectable` | Factory — mỗi screen cần instance riêng |

4. **Chạy build_runner** và kiểm tra file generated

5. **Cập nhật `main.dart`** — gọi `configureDependencies()` trước `runApp()`

6. **Cập nhật widget** — resolve dependencies từ `getIt` thay vì tạo trực tiếp

### 🔨 Hướng dẫn từng bước

**Bước 1: pubspec.yaml**

```yaml
dependencies:
  flutter:
    sdk: flutter
  get_it: ^7.6.4
  injectable: ^2.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  injectable_generator: ^2.3.0
  build_runner: ^2.4.0
```

```bash
flutter pub get
```

**Bước 2: Tạo injection.dart**

```dart
// lib/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';

import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();
```

**Bước 3: Annotate 1 class ví dụ**

```dart
// lib/data/datasources/notes_local_data_source.dart
import 'package:injectable/injectable.dart';

@lazySingleton  // ← Thêm annotation
class NotesLocalDataSource {
  // ... giữ nguyên code
}
```

> 💡 **Tự annotate các class còn lại!** (Repository, UseCases, Bloc/Cubit)

**Bước 4: Chạy build_runner**

```bash
dart run build_runner build --delete-conflicting-outputs
```

Kiểm tra file `lib/injection.config.dart` được tạo thành công.

**Bước 5: main.dart**

```dart
void main() {
  configureDependencies();
  runApp(const MyApp());
}
```

### ✅ Tiêu chí hoàn thành

- [ ] `flutter pub get` chạy không lỗi
- [ ] `dart run build_runner build` generate `injection.config.dart` thành công
- [ ] App chạy bình thường (`flutter run`) với DI
- [ ] Tất cả dependencies được resolve đúng (không có runtime error)
- [ ] Không có `NotesRepositoryImpl(...)` nào được tạo trực tiếp trong widget code

### ⚠️ Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `injection.config.dart not found` | Chưa chạy build_runner | Chạy `dart run build_runner build` |
| `Object/factory with type X is not registered` | Quên annotate một dependency | Thêm `@injectable` / `@lazySingleton` |
| `The following assertion was thrown...` | Gọi getIt trước configureDependencies() | Đảm bảo gọi `configureDependencies()` trong `main()` |

---

## BT2 ⭐⭐: Unit test Repository & UseCase 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_unit_testing` |
| **Cách chạy** | `flutter test` |
| **Output** | Terminal — tất cả unit tests PASS |
| **Dependencies** | `flutter pub add dev:mocktail dev:bloc_test` |

### 🎯 Mục tiêu

Viết unit test cho business logic: test UseCase và Repository implementation bằng mock.

### 📋 Yêu cầu

1. **Thêm test dependencies**:

   ```yaml
   dev_dependencies:
     flutter_test:
       sdk: flutter
     mocktail: ^1.0.4
   ```

2. **Tạo file test** theo cấu trúc:

   ```
   test/
   ├── domain/
   │   └── usecases/
   │       ├── get_notes_usecase_test.dart
   │       └── add_note_usecase_test.dart
   └── data/
       └── repositories/
           └── notes_repository_impl_test.dart
   ```

3. **Test GetNotesUseCase** — ít nhất 3 test cases:

   | Test case | Mock behavior | Expected |
   |-----------|--------------|----------|
   | Happy path | Repository trả về list notes | UseCase trả về đúng list |
   | Empty | Repository trả về `[]` | UseCase trả về `[]` |
   | Error | Repository throw Exception | UseCase throw Exception |

4. **Test AddNoteUseCase** — ít nhất 2 test cases:

   | Test case | Mock behavior | Expected |
   |-----------|--------------|----------|
   | Success | Repository.addNote() thành công | Verify method called 1 lần |
   | Verify data | Truyền note cụ thể | Verify đúng note được truyền |

5. **Test NotesRepositoryImpl** — ít nhất 3 test cases:

   | Test case | Mock behavior | Expected |
   |-----------|--------------|----------|
   | getNotes | DataSource trả về raw data | Repository trả về mapped Notes |
   | addNote | Verify DataSource.insert() | Đúng data format |
   | deleteNote | Verify DataSource.delete() | Đúng id |

### 🔨 Template

**Mock class:**

```dart
class MockNotesRepository extends Mock implements NotesRepository {}
class MockNotesDataSource extends Mock implements NotesLocalDataSource {}
```

**Test structure:**

```dart
void main() {
  late MockNotesRepository mockRepo;
  late GetNotesUseCase useCase;

  setUp(() {
    mockRepo = MockNotesRepository();
    useCase = GetNotesUseCase(mockRepo);
  });

  group('GetNotesUseCase', () {
    test('should return notes when repository succeeds', () async {
      // Arrange
      // TODO: setup mock với when()

      // Act
      // TODO: gọi useCase()

      // Assert
      // TODO: expect() và verify()
    });

    // TODO: thêm test cases
  });
}
```

### 🚀 Chạy test

```bash
# Chạy tất cả unit tests
flutter test

# Chạy 1 file test cụ thể
flutter test test/domain/usecases/get_notes_usecase_test.dart

# Chạy với verbose output
flutter test --reporter expanded

# Chạy với coverage
flutter test --coverage
```

### ✅ Tiêu chí hoàn thành

- [ ] Tất cả tests PASS (`flutter test` không có lỗi đỏ)
- [ ] Ít nhất 3 test cases cho GetNotesUseCase
- [ ] Ít nhất 2 test cases cho AddNoteUseCase
- [ ] Ít nhất 3 test cases cho NotesRepositoryImpl
- [ ] Mỗi test theo Arrange-Act-Assert pattern
- [ ] Sử dụng `verify()` để kiểm tra method calls
- [ ] Không dùng real implementation (chỉ mock/fake)

### 💡 Gợi ý thêm (nâng cao)

- Test edge case: note với title rỗng, content rất dài
- Test concurrent calls: gọi useCase 2 lần
- Dùng `setUpAll()` cho one-time setup nếu cần

---

## BT3 ⭐⭐⭐: Widget test + Integration test 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_widget_integration_test` |
| **Cách chạy** | `flutter test` + `flutter test integration_test/` |
| **Output** | Terminal — widget tests & integration tests PASS |
| **Dependencies** | `flutter pub add dev:mocktail dev:bloc_test` |

### 🎯 Mục tiêu

Test UI và toàn bộ user flow của Notes app.

### 📋 Phần A: Widget test

1. **Tạo file test**:

   ```
   test/
   └── presentation/
       └── pages/
           └── notes_list_page_test.dart
   ```

2. **Viết widget test cho `NotesListPage`** — ít nhất 5 test cases:

   | # | Test case | Kiểm tra |
   |---|-----------|----------|
   | 1 | App bar | Hiển thị title "My Notes" |
   | 2 | Empty state | Hiển thị message khi không có notes |
   | 3 | Notes list | Hiển thị đúng số notes, title, content |
   | 4 | Add button | FAB hiển thị và gọi callback khi tap |
   | 5 | Delete button | Icon delete hiển thị và gọi callback đúng id |

3. **Helper function:**

   ```dart
   Widget createTestWidget({required List<Note> notes, ...}) {
     return MaterialApp(
       home: NotesListPage(notes: notes, ...),
     );
   }
   ```

4. **Chạy:**

   ```bash
   flutter test test/presentation/pages/notes_list_page_test.dart
   ```

### 📋 Phần B: Integration test

1. **Thêm dependency:**

   ```yaml
   dev_dependencies:
     integration_test:
       sdk: flutter
   ```

2. **Tạo file:**

   ```
   integration_test/
   └── notes_flow_test.dart
   ```

3. **Viết integration test cho flow thêm note:**

   ```dart
   // integration_test/notes_flow_test.dart
   import 'package:flutter_test/flutter_test.dart';
   import 'package:integration_test/integration_test.dart';
   import 'package:notes_app/main.dart' as app;

   void main() {
     IntegrationTestWidgetsFlutterBinding.ensureInitialized();

     group('Notes flow', () {
       testWidgets('add a new note and see it in list', (tester) async {
         // Arrange — khởi chạy app
         app.main();
         await tester.pumpAndSettle();

         // Act — tap FAB
         await tester.tap(find.byType(FloatingActionButton));
         await tester.pumpAndSettle();

         // TODO: Điền form thêm note (nếu có form)
         // await tester.enterText(find.byKey(Key('title_field')), 'Test Note');
         // await tester.enterText(find.byKey(Key('content_field')), 'Test Content');
         // await tester.tap(find.text('Save'));
         // await tester.pumpAndSettle();

         // Assert — note xuất hiện trong list
         // expect(find.text('Test Note'), findsOneWidget);
       });
     });
   }
   ```

4. **Chạy trên device/emulator:**

   ```bash
   flutter test integration_test/notes_flow_test.dart
   ```

### ✅ Tiêu chí hoàn thành

**Widget test:**
- [ ] Ít nhất 5 test cases PASS
- [ ] Test empty state
- [ ] Test hiển thị data
- [ ] Test user interaction (tap)
- [ ] Dùng `pumpWidget` và `pump` đúng chỗ

**Integration test:**
- [ ] Test file chạy trên emulator không lỗi
- [ ] Test flow thêm note end-to-end
- [ ] Sử dụng `pumpAndSettle()` cho animations

### ⚠️ Phân biệt pump

| Method | Khi dùng |
|--------|----------|
| `pump()` | Rebuild widget tree 1 frame |
| `pump(Duration(...))` | Rebuild sau khoảng thời gian |
| `pumpAndSettle()` | Rebuild cho đến khi không còn animation |
| `pumpWidget(widget)` | Mount widget lần đầu |

---

## Câu hỏi thảo luận

### ❓ Câu hỏi 1: Khi nào dùng get_it trực tiếp vs injectable?

**Gợi ý suy nghĩ:**

- App nhỏ (< 20 classes) → get_it đủ dùng, không cần build_runner overhead
- App lớn (> 50 classes) → injectable giảm rất nhiều boilerplate
- Team quen code gen → injectable hợp lý
- Trường hợp nào manual DI (không dùng get_it) vẫn ổn?

**Thảo luận:**
- Singleton vs Factory — khi nào dùng cái nào?
- Có nên dùng `getIt` trực tiếp trong widget không? Hay nên truyền qua constructor?
- So sánh: get_it (Service Locator) vs Riverpod (cũng resolve dependencies)

---

### ❓ Câu hỏi 2: Nên test gì và không nên test gì?

**Gợi ý suy nghĩ:**

Nên test:
- Business logic (UseCase, Repository mapping)
- State management (Bloc/Cubit state transitions)
- Edge cases (empty data, error, null)
- UI rendering (đúng widget hiển thị)

Không cần test:
- Code generated (freezed, json_serializable output)
- Framework behavior (Flutter rendering engine)
- Simple getters/setters không có logic
- Third-party packages

**Thảo luận:**
- Test coverage bao nhiêu % là đủ?
- Khi nào unit test không đủ, cần widget test?
- Integration test có nên chạy trong CI/CD?

---

### ❓ Câu hỏi 3: Code generation — lợi và hại?

**Gợi ý suy nghĩ:**

Lợi ích:
- Giảm boilerplate, ít bug do code lặp
- Type-safe (compile-time errors thay vì runtime)
- Consistency (code generated luôn follow pattern)

Bất lợi:
- Build time chậm hơn (phải chạy build_runner)
- Thêm complexity (team mới phải học)
- Generated files lớn, IDE chậm hơn
- Debugging khó hơn (stack trace qua generated code)

**Thảo luận:**
- Có nên commit generated files vào git không?
- Khi nào chọn freezed, khi nào viết tay `==`, `copyWith`?
- Nếu build_runner chạy 5 phút mỗi lần, có chấp nhận được không?

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 5:** Focus vào gen test scaffold và review mock coverage.

### AI-BT1: Gen Unit Tests cho UseCase + Repository ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** DI with get_it/injectable, mocking with mocktail, testing strategy (unit/widget/integration), coverage.
- **Task thực tế:** CI pipeline yêu cầu 80% coverage trước khi merge. Cần viết unit tests cho tất cả UseCases và Repository implementations. AI gen test scaffold, bạn review mock setup + missing edge cases.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần viết unit tests cho User feature theo Clean Architecture.
Tech stack: Flutter 3.x, flutter_test, mocktail ^1.x.
Code hiện tại (đã implement):
- GetUserProfile(UserRepository) → Future<Either<Failure, UserEntity>>
- UserRepositoryImpl(UserRemoteDataSource, UserLocalDataSource)
  - getUser(id): try remote → cache → if fail → try local → if fail → Left(Failure)

Test requirements:
- Mock tất cả dependencies dùng mocktail.
- UseCase tests: success, failure, verify repository called once.
- Repository tests: remote success (cache result), remote fail + cache hit, remote fail + cache miss.
- setUp/tearDown đúng pattern.
- Tìm 3 missing test cases tôi chưa nghĩ tới.
Output: 2 test files + suggested missing tests.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 2 test files + suggested missing test cases.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Mock classes dùng `class MockX extends Mock implements X`? | ☐ |
| 2 | `when(() => mock.method()).thenAnswer((_) async => ...)` cho async? | ☐ |
| 3 | `verify(() => mock.method()).called(1)` kiểm tra side effects? | ☐ |
| 4 | `setUp` initialize mocks + SUT (system under test)? | ☐ |
| 5 | Test cả happy path + error path? | ☐ |
| 6 | Missing test cases hữu ích (không phải trivial)? | ☐ |
| 7 | Không test implementation detail (chỉ test behavior)? | ☐ |

**4. Customize:**
Thêm widget test cho ProfileScreen: mock UserBloc, verify UI hiển thị đúng khi state = loading/success/error. AI chỉ gen unit tests — tự thêm `testWidgets` + `BlocProvider` override.
