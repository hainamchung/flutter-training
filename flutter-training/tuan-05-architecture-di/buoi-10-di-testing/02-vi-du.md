# Buổi 10: Dependency Injection & Testing — Ví dụ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

1. [VD1: get_it setup — Đăng ký và resolve dependencies thủ công](#vd1-get_it-setup)
2. [VD2: injectable setup — Annotations + build_runner](#vd2-injectable-setup)
3. [VD3: Unit test — Test UseCase với mocked Repository](#vd3-unit-test-usecase)
4. [VD4: Widget test — Test screen với WidgetTester](#vd4-widget-test)
5. [VD5: freezed — Immutable data class với copyWith, fromJson](#vd5-freezed)

---

## VD1: get_it setup 🟡

> **Liên quan tới:** [2. get_it + injectable](01-ly-thuyet.md#2-get_it--injectable)

> **Mục tiêu:** Đăng ký và resolve dependencies thủ công với get_it, không dùng code gen.

### Bước 1: Entity và Repository interface

```dart
// lib/domain/entities/note.dart
class Note {
  final String id;
  final String title;
  final String content;

  const Note({required this.id, required this.title, required this.content});
}

// lib/domain/repositories/notes_repository.dart
abstract class NotesRepository {
  Future<List<Note>> getNotes();
  Future<void> addNote(Note note);
  Future<void> deleteNote(String id);
}
```

### Bước 2: Implementation

```dart
// lib/data/datasources/notes_local_data_source.dart
class NotesLocalDataSource {
  final List<Map<String, dynamic>> _storage = [];

  Future<List<Map<String, dynamic>>> getAll() async => _storage;

  Future<void> insert(Map<String, dynamic> data) async => _storage.add(data);

  Future<void> delete(String id) async =>
      _storage.removeWhere((item) => item['id'] == id);
}

// lib/data/repositories/notes_repository_impl.dart
class NotesRepositoryImpl implements NotesRepository {
  final NotesLocalDataSource dataSource;

  NotesRepositoryImpl(this.dataSource);

  @override
  Future<List<Note>> getNotes() async {
    final data = await dataSource.getAll();
    return data
        .map((e) => Note(id: e['id'], title: e['title'], content: e['content']))
        .toList();
  }

  @override
  Future<void> addNote(Note note) async {
    await dataSource.insert({
      'id': note.id,
      'title': note.title,
      'content': note.content,
    });
  }

  @override
  Future<void> deleteNote(String id) async {
    await dataSource.delete(id);
  }
}
```

### Bước 3: UseCase

```dart
// lib/domain/usecases/get_notes_usecase.dart
class GetNotesUseCase {
  final NotesRepository repository;

  GetNotesUseCase(this.repository);

  Future<List<Note>> call() => repository.getNotes();
}

// lib/domain/usecases/add_note_usecase.dart
class AddNoteUseCase {
  final NotesRepository repository;

  AddNoteUseCase(this.repository);

  Future<void> call(Note note) => repository.addNote(note);
}
```

### Bước 4: Setup get_it thủ công

```dart
// lib/injection.dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setupDependencies() {
  // Data layer
  getIt.registerLazySingleton<NotesLocalDataSource>(
    () => NotesLocalDataSource(),
  );

  // Repository — đăng ký interface, resolve implementation
  getIt.registerLazySingleton<NotesRepository>(
    () => NotesRepositoryImpl(getIt<NotesLocalDataSource>()),
  );

  // Use cases
  getIt.registerFactory<GetNotesUseCase>(
    () => GetNotesUseCase(getIt<NotesRepository>()),
  );
  getIt.registerFactory<AddNoteUseCase>(
    () => AddNoteUseCase(getIt<NotesRepository>()),
  );
}
```

### Bước 5: Sử dụng trong main() và Widget

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'injection.dart';

void main() {
  setupDependencies();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Notes App',
      home: NotesPage(),
    );
  }
}

// lib/presentation/pages/notes_page.dart
class NotesPage extends StatefulWidget {
  const NotesPage({super.key});

  @override
  State<NotesPage> createState() => _NotesPageState();
}

class _NotesPageState extends State<NotesPage> {
  // Resolve từ get_it
  final getNotesUseCase = getIt<GetNotesUseCase>();
  final addNoteUseCase = getIt<AddNoteUseCase>();

  List<Note> _notes = [];

  @override
  void initState() {
    super.initState();
    _loadNotes();
  }

  Future<void> _loadNotes() async {
    final notes = await getNotesUseCase();
    setState(() => _notes = notes);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Notes')),
      body: ListView.builder(
        itemCount: _notes.length,
        itemBuilder: (context, index) {
          final note = _notes[index];
          return ListTile(
            title: Text(note.title),
            subtitle: Text(note.content),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          final note = Note(
            id: DateTime.now().millisecondsSinceEpoch.toString(),
            title: 'New Note',
            content: 'Content here',
          );
          await addNoteUseCase(note);
          await _loadNotes();
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

**📌 Kết quả:** Dependencies được wire qua get_it. `NotesPage` không cần biết `NotesRepositoryImpl` hay `NotesLocalDataSource` — chỉ dùng qua UseCase.

- 🔗 **FE tương đương:** Tương tự Angular DI `@Injectable()` hoặc InversifyJS container — register service rồi resolve by type. React/Vue ít dùng pattern này.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_get_it
cd vidu_get_it
flutter pub add get_it
# Thay nội dung lib/ bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ App hiển thị trang Notes với AppBar "Notes"
✅ Nhấn FAB (+) → thêm note mới "New Note" vào danh sách
✅ Dependencies được resolve qua get_it — không có tight coupling
```

---

## VD2: injectable setup 🟡

> **Mục tiêu:** Dùng annotations thay vì viết tay registration. build_runner tự generate code.

> **Liên quan tới:** [2. get_it + injectable](01-ly-thuyet.md#2-get_it--injectable)

### Bước 1: Thêm dependencies

```yaml
# pubspec.yaml
dependencies:
  get_it: ^7.6.4
  injectable: ^2.3.0

dev_dependencies:
  injectable_generator: ^2.3.0
  build_runner: ^2.4.0
```

### Bước 2: injection.dart

```dart
// lib/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';

import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();
```

### Bước 3: Annotate classes

```dart
// lib/data/datasources/notes_local_data_source.dart
import 'package:injectable/injectable.dart';

@lazySingleton
class NotesLocalDataSource {
  final List<Map<String, dynamic>> _storage = [];

  Future<List<Map<String, dynamic>>> getAll() async => _storage;
  Future<void> insert(Map<String, dynamic> data) async => _storage.add(data);
  Future<void> delete(String id) async =>
      _storage.removeWhere((item) => item['id'] == id);
}

// lib/data/repositories/notes_repository_impl.dart
import 'package:injectable/injectable.dart';

@LazySingleton(as: NotesRepository) // ← Đăng ký với interface
class NotesRepositoryImpl implements NotesRepository {
  final NotesLocalDataSource dataSource;

  NotesRepositoryImpl(this.dataSource); // ← injectable tự resolve

  @override
  Future<List<Note>> getNotes() async {
    final data = await dataSource.getAll();
    return data
        .map((e) => Note(id: e['id'], title: e['title'], content: e['content']))
        .toList();
  }

  @override
  Future<void> addNote(Note note) async {
    await dataSource.insert({
      'id': note.id,
      'title': note.title,
      'content': note.content,
    });
  }

  @override
  Future<void> deleteNote(String id) async {
    await dataSource.delete(id);
  }
}

// lib/domain/usecases/get_notes_usecase.dart
import 'package:injectable/injectable.dart';

@injectable
class GetNotesUseCase {
  final NotesRepository repository;
  GetNotesUseCase(this.repository);
  Future<List<Note>> call() => repository.getNotes();
}

// lib/domain/usecases/add_note_usecase.dart
import 'package:injectable/injectable.dart';

@injectable
class AddNoteUseCase {
  final NotesRepository repository;
  AddNoteUseCase(this.repository);
  Future<void> call(Note note) => repository.addNote(note);
}
```

### Bước 4: Đăng ký external dependency với @module

```dart
// lib/injection/register_module.dart
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';

@module
abstract class RegisterModule {
  @preResolve
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}
```

### Bước 5: Chạy build_runner

```bash
dart run build_runner build --delete-conflicting-outputs
```

**File generated `injection.config.dart`** (tự động):

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND
// ****************************************************
//  injectable configuration
// ****************************************************

extension GetItInjectableX on GetIt {
  Future<GetIt> init({String? environment, ...}) async {
    final gh = GetItHelper(this, environment);

    // External modules
    final registerModule = _$RegisterModule();
    await gh.factoryAsync<SharedPreferences>(() => registerModule.prefs,
        preResolve: true);

    // Data layer
    gh.lazySingleton<NotesLocalDataSource>(() => NotesLocalDataSource());
    gh.lazySingleton<NotesRepository>(
        () => NotesRepositoryImpl(gh<NotesLocalDataSource>()));

    // Domain layer
    gh.factory<GetNotesUseCase>(() => GetNotesUseCase(gh<NotesRepository>()));
    gh.factory<AddNoteUseCase>(() => AddNoteUseCase(gh<NotesRepository>()));

    return this;
  }
}
```

### Bước 6: Gọi trong main()

```dart
// lib/main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await configureDependencies(); // async vì có @preResolve
  runApp(const MyApp());
}
```

**📌 So sánh:** VD1 viết ~20 dòng registration code. VD2 chỉ cần thêm annotation trên mỗi class → build_runner generate toàn bộ. Khi project có 50+ classes, injectable tiết kiệm rất nhiều boilerplate.

---

## VD3: Unit test UseCase 🔴

> **Liên quan tới:** [3. Testing Strategy](01-ly-thuyet.md#3-testing-strategy) · [4. Mocking với mocktail](01-ly-thuyet.md#4-mocking-với-mocktail)

> **Mục tiêu:** Test `GetNotesUseCase` bằng cách mock `NotesRepository` với mocktail.

### Setup

```yaml
# pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
```

### Test file

```dart
// test/domain/usecases/get_notes_usecase_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:notes_app/domain/entities/note.dart';
import 'package:notes_app/domain/repositories/notes_repository.dart';
import 'package:notes_app/domain/usecases/get_notes_usecase.dart';

// ======== Tạo Mock ========
class MockNotesRepository extends Mock implements NotesRepository {}

void main() {
  late MockNotesRepository mockRepository;
  late GetNotesUseCase useCase;

  // setUp chạy trước MỖI test case
  setUp(() {
    mockRepository = MockNotesRepository();
    useCase = GetNotesUseCase(mockRepository);
  });

  group('GetNotesUseCase', () {
    // ======== Test case 1: Happy path ========
    test('should return list of notes when repository succeeds', () async {
      // Arrange
      final expectedNotes = [
        const Note(id: '1', title: 'Note 1', content: 'Content 1'),
        const Note(id: '2', title: 'Note 2', content: 'Content 2'),
      ];
      when(() => mockRepository.getNotes())
          .thenAnswer((_) async => expectedNotes);

      // Act
      final result = await useCase();

      // Assert
      expect(result, equals(expectedNotes));
      expect(result.length, 2);
      verify(() => mockRepository.getNotes()).called(1);
    });

    // ======== Test case 2: Empty list ========
    test('should return empty list when no notes exist', () async {
      // Arrange
      when(() => mockRepository.getNotes()).thenAnswer((_) async => []);

      // Act
      final result = await useCase();

      // Assert
      expect(result, isEmpty);
      verify(() => mockRepository.getNotes()).called(1);
    });

    // ======== Test case 3: Error handling ========
    test('should throw exception when repository fails', () async {
      // Arrange
      when(() => mockRepository.getNotes())
          .thenThrow(Exception('Database error'));

      // Act & Assert
      expect(() => useCase(), throwsException);
      verify(() => mockRepository.getNotes()).called(1);
    });
  });
}
```

### Test Repository impl

```dart
// test/data/repositories/notes_repository_impl_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:notes_app/data/datasources/notes_local_data_source.dart';
import 'package:notes_app/data/repositories/notes_repository_impl.dart';

class MockNotesLocalDataSource extends Mock implements NotesLocalDataSource {}

void main() {
  late MockNotesLocalDataSource mockDataSource;
  late NotesRepositoryImpl repository;

  setUp(() {
    mockDataSource = MockNotesLocalDataSource();
    repository = NotesRepositoryImpl(mockDataSource);
  });

  group('NotesRepositoryImpl', () {
    group('getNotes', () {
      test('should return notes mapped from data source', () async {
        // Arrange
        final rawData = [
          {'id': '1', 'title': 'Test', 'content': 'Content'},
        ];
        when(() => mockDataSource.getAll()).thenAnswer((_) async => rawData);

        // Act
        final result = await repository.getNotes();

        // Assert
        expect(result.length, 1);
        expect(result.first.id, '1');
        expect(result.first.title, 'Test');
        verify(() => mockDataSource.getAll()).called(1);
      });

      test('should return empty list when data source is empty', () async {
        // Arrange
        when(() => mockDataSource.getAll()).thenAnswer((_) async => []);

        // Act
        final result = await repository.getNotes();

        // Assert
        expect(result, isEmpty);
      });
    });

    group('addNote', () {
      test('should call data source insert with correct data', () async {
        // Arrange
        const note = Note(id: '1', title: 'New', content: 'Content');
        when(() => mockDataSource.insert(any())).thenAnswer((_) async {});

        // Act
        await repository.addNote(note);

        // Assert
        verify(() => mockDataSource.insert({
              'id': '1',
              'title': 'New',
              'content': 'Content',
            })).called(1);
      });
    });

    group('deleteNote', () {
      test('should call data source delete with correct id', () async {
        // Arrange
        when(() => mockDataSource.delete(any())).thenAnswer((_) async {});

        // Act
        await repository.deleteNote('1');

        // Assert
        verify(() => mockDataSource.delete('1')).called(1);
      });
    });
  });
}
```

**Chạy test:**

```bash
flutter test test/domain/usecases/get_notes_usecase_test.dart
flutter test test/data/repositories/notes_repository_impl_test.dart

# Chạy tất cả tests
flutter test
```

**📌 Kết quả:** Mỗi layer được test độc lập. UseCase test mock Repository, Repository test mock DataSource. Không cần database hay network thật.

- 🔗 **FE tương đương:** `when(mock.method()).thenReturn(value)` ≈ `jest.fn().mockReturnValue(value)` — cùng pattern mock dependency, khác syntax OOP vs functional.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_unit_test
cd vidu_unit_test
flutter pub add dev:mocktail
# Tạo test files trong test/ theo cấu trúc trên, rồi:
flutter test
```

### 📋 Kết quả mong đợi

```
✅ All tests passed!
✅ GetNotesUseCase: 3 tests passed (happy path, empty list, error)
✅ NotesRepositoryImpl: 3 tests passed (getNotes, addNote, deleteNote)
```

---

## VD4: Widget test 🟡

> **Mục tiêu:** Test `NotesListPage` hiển thị đúng UI, xử lý empty state, và tương tác button.

> **Liên quan tới:** [3. Testing Strategy](01-ly-thuyet.md#3-testing-strategy)

### Widget cần test

```dart
// lib/presentation/pages/notes_list_page.dart
import 'package:flutter/material.dart';

class NotesListPage extends StatelessWidget {
  final List<Note> notes;
  final VoidCallback? onAddPressed;
  final Function(String id)? onDeletePressed;

  const NotesListPage({
    super.key,
    required this.notes,
    this.onAddPressed,
    this.onDeletePressed,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My Notes')),
      body: notes.isEmpty
          ? const Center(
              child: Text('No notes yet. Tap + to add one!'),
            )
          : ListView.builder(
              itemCount: notes.length,
              itemBuilder: (context, index) {
                final note = notes[index];
                return ListTile(
                  key: ValueKey(note.id),
                  title: Text(note.title),
                  subtitle: Text(note.content),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: () => onDeletePressed?.call(note.id),
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: onAddPressed,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Widget test file

```dart
// test/presentation/pages/notes_list_page_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:notes_app/domain/entities/note.dart';
import 'package:notes_app/presentation/pages/notes_list_page.dart';

void main() {
  // Helper: wrap widget với MaterialApp (cần cho Scaffold, AppBar, etc.)
  Widget createTestWidget({
    required List<Note> notes,
    VoidCallback? onAddPressed,
    Function(String)? onDeletePressed,
  }) {
    return MaterialApp(
      home: NotesListPage(
        notes: notes,
        onAddPressed: onAddPressed,
        onDeletePressed: onDeletePressed,
      ),
    );
  }

  group('NotesListPage', () {
    // ======== Test 1: Hiển thị AppBar ========
    testWidgets('should display app bar with title', (tester) async {
      // Arrange & Act
      await tester.pumpWidget(createTestWidget(notes: []));

      // Assert
      expect(find.text('My Notes'), findsOneWidget);
    });

    // ======== Test 2: Empty state ========
    testWidgets('should show empty message when no notes', (tester) async {
      // Arrange & Act
      await tester.pumpWidget(createTestWidget(notes: []));

      // Assert
      expect(find.text('No notes yet. Tap + to add one!'), findsOneWidget);
      expect(find.byType(ListView), findsNothing);
    });

    // ======== Test 3: Hiển thị danh sách notes ========
    testWidgets('should display list of notes', (tester) async {
      // Arrange
      final notes = [
        const Note(id: '1', title: 'First Note', content: 'Content 1'),
        const Note(id: '2', title: 'Second Note', content: 'Content 2'),
      ];

      // Act
      await tester.pumpWidget(createTestWidget(notes: notes));

      // Assert
      expect(find.text('First Note'), findsOneWidget);
      expect(find.text('Content 1'), findsOneWidget);
      expect(find.text('Second Note'), findsOneWidget);
      expect(find.text('Content 2'), findsOneWidget);
      expect(find.byType(ListTile), findsNWidgets(2));
    });

    // ======== Test 4: Tap FAB ========
    testWidgets('should call onAddPressed when FAB is tapped', (tester) async {
      // Arrange
      var addPressed = false;

      // Act
      await tester.pumpWidget(createTestWidget(
        notes: [],
        onAddPressed: () => addPressed = true,
      ));
      await tester.tap(find.byType(FloatingActionButton));
      await tester.pump(); // Rebuild sau tap

      // Assert
      expect(addPressed, isTrue);
    });

    // ======== Test 5: Tap delete button ========
    testWidgets('should call onDeletePressed with correct id', (tester) async {
      // Arrange
      String? deletedId;
      final notes = [
        const Note(id: '42', title: 'Delete Me', content: 'Content'),
      ];

      // Act
      await tester.pumpWidget(createTestWidget(
        notes: notes,
        onDeletePressed: (id) => deletedId = id,
      ));
      await tester.tap(find.byIcon(Icons.delete));
      await tester.pump();

      // Assert
      expect(deletedId, '42');
    });

    // ======== Test 6: FAB luôn hiển thị ========
    testWidgets('should always show FAB', (tester) async {
      await tester.pumpWidget(createTestWidget(notes: []));
      expect(find.byType(FloatingActionButton), findsOneWidget);
      expect(find.byIcon(Icons.add), findsOneWidget);
    });
  });
}
```

### Các hàm find thường dùng

```dart
// Tìm theo text
find.text('Hello')

// Tìm theo widget type
find.byType(ElevatedButton)
find.byType(ListTile)

// Tìm theo icon
find.byIcon(Icons.add)

// Tìm theo Key
find.byKey(const ValueKey('note_1'))

// Tìm con cháu trong widget cụ thể
find.descendant(
  of: find.byType(AppBar),
  matching: find.text('My Notes'),
)
```

### Các matcher thường dùng

```dart
findsOneWidget      // Đúng 1 widget
findsNothing        // Không tìm thấy
findsNWidgets(3)    // Đúng 3 widgets
findsAtLeastNWidgets(1) // Ít nhất 1
```

**Chạy:**

```bash
flutter test test/presentation/pages/notes_list_page_test.dart
```

**📌 Kết quả:** Widget test chạy rất nhanh (không cần emulator), kiểm tra UI render đúng và user interaction hoạt động.

---

## VD5: freezed — Immutable Data Class 🟢

> **Liên quan tới:** [5. Code Generation](01-ly-thuyet.md#5-code-generation)

> **Mục tiêu:** Tạo immutable `Note` class với copyWith, ==, fromJson/toJson bằng freezed.

### Bước 1: Cài đặt

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0

dev_dependencies:
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

### Bước 2: Viết class với annotations

```dart
// lib/domain/entities/note.dart
import 'package:freezed_annotation/freezed_annotation.dart';

// 2 part directives cho 2 file generated
part 'note.freezed.dart'; // copyWith, ==, hashCode, toString
part 'note.g.dart';       // fromJson, toJson

@freezed
class Note with _$Note {
  const factory Note({
    required String id,
    required String title,
    required String content,
    @Default(false) bool isPinned,
    @Default([]) List<String> tags,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) = _Note;

  // fromJson factory
  factory Note.fromJson(Map<String, dynamic> json) => _$NoteFromJson(json);
}
```

### Bước 3: Chạy build_runner

```bash
dart run build_runner build --delete-conflicting-outputs
```

### Bước 4: Sử dụng generated code

```dart
void main() {
  // ======== Tạo instance ========
  final note = Note(
    id: '1',
    title: 'Learning Flutter',
    content: 'DI and Testing are important',
    tags: ['flutter', 'testing'],
    createdAt: DateTime.now(),
  );
  print(note);
  // Note(id: 1, title: Learning Flutter, content: DI and Testing are important,
  //   isPinned: false, tags: [flutter, testing], createdAt: ...)

  // ======== copyWith — tạo copy, thay đổi một số field ========
  final pinned = note.copyWith(isPinned: true);
  print(pinned.isPinned); // true
  print(note.isPinned);   // false — note gốc KHÔNG thay đổi (immutable)

  final updated = note.copyWith(
    title: 'Updated Title',
    updatedAt: DateTime.now(),
  );

  // ======== == (value equality) ========
  final note1 = Note(id: '1', title: 'A', content: 'B');
  final note2 = Note(id: '1', title: 'A', content: 'B');
  print(note1 == note2); // true (so sánh value, không phải reference)

  final note3 = note1.copyWith(title: 'C');
  print(note1 == note3); // false (title khác)

  // ======== fromJson / toJson ========
  final json = {
    'id': '2',
    'title': 'From JSON',
    'content': 'Parsed content',
    'isPinned': true,
    'tags': ['json', 'test'],
  };
  final fromJson = Note.fromJson(json);
  print(fromJson.title); // From JSON

  final toJson = fromJson.toJson();
  print(toJson); // {id: 2, title: From JSON, content: Parsed content, ...}
}
```

### Bonus: Union types cho State

```dart
// lib/presentation/bloc/notes_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'notes_state.freezed.dart';

@freezed
class NotesState with _$NotesState {
  const factory NotesState.initial() = NotesInitial;
  const factory NotesState.loading() = NotesLoading;
  const factory NotesState.loaded(List<Note> notes) = NotesLoaded;
  const factory NotesState.error(String message) = NotesError;
}
```

**Sử dụng với pattern matching:**

```dart
// Trong widget
@override
Widget build(BuildContext context) {
  final state = context.watch<NotesCubit>().state;

  // when — phải handle TẤT CẢ cases (exhaustive)
  return state.when(
    initial: () => const SizedBox.shrink(),
    loading: () => const Center(child: CircularProgressIndicator()),
    loaded: (notes) => ListView.builder(
      itemCount: notes.length,
      itemBuilder: (context, index) => ListTile(title: Text(notes[index].title)),
    ),
    error: (message) => Center(child: Text('Error: $message')),
  );
}

// maybeWhen — handle một số cases, có orElse
return state.maybeWhen(
  loaded: (notes) => Text('${notes.length} notes'),
  orElse: () => const CircularProgressIndicator(),
);

// map — nhận typed object thay vì destructured params
return state.map(
  initial: (_) => const SizedBox.shrink(),
  loading: (_) => const CircularProgressIndicator(),
  loaded: (state) => NotesList(notes: state.notes),
  error: (state) => ErrorView(message: state.message),
);
```

**📌 Kết quả:** Một annotation `@freezed` generate toàn bộ boilerplate cho immutable class. Đặc biệt hữu ích cho entities (Domain layer) và state classes (BLoC/Cubit).

### Test

```dart
// test/models/user_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/models/user.dart';

void main() {
  group('User freezed model', () {
    test('copyWith creates modified copy', () {
      final user = User(id: 1, name: 'Alice', email: 'alice@test.com');
      final updated = user.copyWith(name: 'Bob');
      
      expect(updated.name, 'Bob');
      expect(updated.id, 1); // unchanged
      expect(user.name, 'Alice'); // immutable
    });

    test('equality works by value', () {
      final u1 = User(id: 1, name: 'Alice', email: 'a@test.com');
      final u2 = User(id: 1, name: 'Alice', email: 'a@test.com');
      expect(u1, equals(u2));
    });

    test('fromJson / toJson roundtrip', () {
      final user = User(id: 1, name: 'Alice', email: 'a@test.com');
      final json = user.toJson();
      final restored = User.fromJson(json);
      expect(restored, equals(user));
    });
  });
}
```

> ✅ **Freezed models rất dễ test** — vì immutable, không cần lo side-effects. copyWith + equality là 2 feature được test nhiều nhất.

### ▶️ Chạy ví dụ

```bash
flutter create vidu_freezed
cd vidu_freezed
flutter pub add freezed_annotation json_annotation
flutter pub add dev:freezed dev:json_serializable dev:build_runner
# Tạo file note.dart với annotations, rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ build_runner generate note.freezed.dart và note.g.dart
✅ copyWith hoạt động: note.copyWith(isPinned: true) trả về instance mới
✅ Value equality: Note(id: '1', ...) == Note(id: '1', ...) → true
✅ fromJson/toJson serialize/deserialize chính xác
```

---

## VD6: 🤖 AI Gen → Review — Unit Test với Mocking 🟢

> **Mục đích:** Luyện workflow "AI gen test scaffold → bạn review mock setup + coverage gaps → fix"

> **Liên quan tới:** [4. Mocking với mocktail](01-ly-thuyet.md#4-mocking-với-mocktail)

### Bước 1: Prompt cho AI

```text
Tạo unit tests cho GetNotes UseCase trong Flutter Clean Architecture.
UseCase depends on NoteRepository (abstract).
Tests: success returns list, failure returns Failure, verify repo called once.
Dùng mocktail. Output: get_notes_test.dart.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Mock pattern** | `class MockNoteRepository extends Mock implements NoteRepository`? |
| 2 | **Async mock** | `thenAnswer((_) async => ...)` cho Future methods? (thenReturn = sai cho Future) |
| 3 | **Verify** | `verify(() => repo.getNotes()).called(1)` — side effect checked? |
| 4 | **Edge cases** | Empty list? Null input? Network timeout? AI hay thiếu edge cases |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: thenReturn cho async method
when(() => mockRepo.getNotes())
    .thenReturn(Future.value([note]));  // WRONG — thenReturn cho Future

// ✅ FIX: thenAnswer cho async
when(() => mockRepo.getNotes())
    .thenAnswer((_) async => Right([note]));  // CORRECT

// ❌ LỖI 2: Thiếu setUp — mock không fresh mỗi test
test('test 1', () { /* uses stale mock from previous test */ });

// ✅ FIX: setUp tạo mới mỗi test
setUp(() {
  mockRepo = MockNoteRepository();
  useCase = GetNotes(mockRepo);
});
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Phân biệt `thenReturn` vs `thenAnswer` cho async mocking
- ✅ Biết verify() kiểm tra side effects quan trọng
- ✅ Nhận ra AI hay thiếu edge case tests — cần tự thêm
