# Buổi 09: Clean Architecture trong Flutter — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen tổ chức code FE**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| FE Habit | Flutter Reality | Bài tập liên quan |
|----------|-----------------|---------------------|
| Import trực tiếp từ bất kỳ layer nào | **Dependency Rule**: domain KHÔNG import data/presentation | BT1, BT2 |
| API service = 1 file function exports | Repository = abstract class + impl — phải tạo cả hai | BT1, BT2, BT3 |
| Business logic trong component/hook | Business logic trong **Use Case class** — tách khỏi UI hoàn toàn | BT2, BT3 |

---

## BT1 ⭐ Xác định layers trong code spaghetti 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Phân tích (text/giấy) |
| **Tạo project** | Không cần |
| **Cách chạy** | Không |
| **Output** | Trả lời bằng text — phân tích code spaghetti và xác định layers |

### Mục tiêu
Đọc code "trộn lẫn" và phân loại từng phần thuộc Domain, Data, hay Presentation layer.

### Yêu cầu
Đọc đoạn code dưới đây. **Không cần viết code** — chỉ cần trả lời bằng text/giấy:
1. Đánh dấu từng đoạn code thuộc layer nào (Domain / Data / Presentation)
2. Xác định Entity, Use Case logic, Repository logic, Data Source logic, UI logic
3. Vẽ lại cấu trúc folder nếu refactor theo Clean Architecture

### Code cần phân tích

```dart
// File: lib/screens/todo_screen.dart
// Tất cả logic đều nằm trong 1 file

import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'package:shared_preferences/shared_preferences.dart';

class TodoScreen extends StatefulWidget {
  const TodoScreen({super.key});

  @override
  State<TodoScreen> createState() => _TodoScreenState();
}

class _TodoScreenState extends State<TodoScreen> {
  List<Map<String, dynamic>> todos = [];
  bool isLoading = false;
  String? errorMessage;

  @override
  void initState() {
    super.initState();
    loadTodos();
  }

  // ---------- A ----------
  Future<void> loadTodos() async {
    setState(() {
      isLoading = true;
      errorMessage = null;
    });

    try {
      // ---------- B ----------
      final response = await http.get(
        Uri.parse('https://jsonplaceholder.typicode.com/todos'),
      );

      if (response.statusCode == 200) {
        // ---------- C ----------
        final List<dynamic> data = jsonDecode(response.body);
        final allTodos = data.map((json) => {
          'id': json['id'],
          'title': json['title'],
          'completed': json['completed'],
          'userId': json['user_id'],
        }).toList();

        // ---------- D ----------
        final incompleteTodos = allTodos
          .where((todo) => todo['completed'] == false)
          .toList();

        // ---------- E ----------
        final prefs = await SharedPreferences.getInstance();
        await prefs.setString('cached_todos', jsonEncode(incompleteTodos));

        // ---------- F ----------
        setState(() {
          todos = incompleteTodos;
          isLoading = false;
        });
      }
    } catch (e) {
      // ---------- G ----------
      try {
        final prefs = await SharedPreferences.getInstance();
        final cached = prefs.getString('cached_todos');
        if (cached != null) {
          setState(() {
            todos = List<Map<String, dynamic>>.from(jsonDecode(cached));
            isLoading = false;
          });
        }
      } catch (_) {
        setState(() {
          errorMessage = 'Không thể tải dữ liệu';
          isLoading = false;
        });
      }
    }
  }

  // ---------- H ----------
  Future<void> toggleTodo(int index) async {
    final todo = todos[index];
    final newCompleted = !(todo['completed'] as bool);

    // ---------- I ----------
    final response = await http.patch(
      Uri.parse('https://jsonplaceholder.typicode.com/todos/${todo['id']}'),
      body: jsonEncode({'completed': newCompleted}),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 200) {
      setState(() {
        todos[index] = {...todo, 'completed': newCompleted};
      });
    }
  }

  // ---------- J ----------
  @override
  Widget build(BuildContext context) {
    if (isLoading) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    return Scaffold(
      appBar: AppBar(title: const Text('My Todos')),
      body: errorMessage != null
          ? Center(child: Text(errorMessage!))
          : ListView.builder(
              itemCount: todos.length,
              itemBuilder: (_, i) => CheckboxListTile(
                title: Text(todos[i]['title']),
                value: todos[i]['completed'],
                onChanged: (_) => toggleTodo(i),
              ),
            ),
    );
  }
}
```

### Câu hỏi cần trả lời

**Q1.** Phân loại từng đoạn (A → J):

| Đoạn | Layer | Vai trò cụ thể |
|------|-------|----------------|
| A | ? | ? |
| B | ? | ? |
| C | ? | ? |
| D | ? | ? |
| E | ? | ? |
| F | ? | ? |
| G | ? | ? |
| H | ? | ? |
| I | ? | ? |
| J | ? | ? |

**Q2.** Liệt kê các Entity cần tạo. Những field nào?

**Q3.** Cần bao nhiêu Use Case? Tên gì?

**Q4.** Repository interface cần những method nào?

**Q5.** Cần bao nhiêu Data Source? Remote hay Local?

**Q6.** Vẽ cấu trúc folder nếu refactor:
```
lib/features/todo/
├── domain/
│   ├── entities/
│   │   └── ???
│   ├── usecases/
│   │   └── ???
│   └── repositories/
│       └── ???
├── data/
│   ├── models/
│   │   └── ???
│   ├── datasources/
│   │   └── ???
│   └── repositories/
│       └── ???
└── presentation/
    ├── cubit/
    │   └── ???
    ├── pages/
    │   └── ???
    └── widgets/
        └── ???
```

### Đáp án tham khảo

<details>
<summary>👉 Nhấn để xem đáp án</summary>

| Đoạn | Layer | Vai trò |
|------|-------|---------|
| A | Presentation | Orchestration / state management logic (loading state) |
| B | Data | Remote Data Source — gọi HTTP API |
| C | Data | Model parsing — JSON → Map (nên là fromJson) |
| D | Domain | Business logic — filter incomplete todos (Use Case) |
| E | Data | Local Data Source — cache vào SharedPreferences |
| F | Presentation | UI state update (setState) |
| G | Data + Presentation | Repository logic (fallback cache) + UI state update |
| H | Presentation + Domain | Orchestration + business logic (toggle completed) |
| I | Data | Remote Data Source — PATCH API call |
| J | Presentation | Pure UI — widget build |

**Entity:** `Todo` với fields: `id` (int), `title` (String), `completed` (bool), `userId` (int)

**Use Cases:** `GetIncompleteTodos`, `ToggleTodoCompletion`

**Repository interface:**
```dart
abstract class TodoRepository {
  Future<List<Todo>> getIncompleteTodos();
  Future<Todo> toggleTodo(int id, bool completed);
}
```

**Data Sources:** `TodoRemoteDataSource` + `TodoLocalDataSource`

</details>

### Phần code (bắt buộc)

Tạo folder structure cho ứng dụng đã phân tích ở trên:

```bash
mkdir -p lib/{domain/{entities,repositories,usecases},data/{models,repositories,datasources},presentation/{pages,widgets,cubits}}
```

Trong mỗi folder, tạo 1 file placeholder với TODO comments:

```dart
// lib/domain/entities/task.dart
// TODO: Định nghĩa Task entity với các field cần thiết
// Gợi ý: id, title, isCompleted, createdAt
class Task {
  // Implement entity ở đây
}
```

**Output:** Screenshot folder structure + 3 placeholder files (1 entity, 1 repository interface, 1 use case).

---

## BT2 ⭐⭐ Tái cấu trúc Todo App theo Clean Architecture 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_clean_todo` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Todo app tái cấu trúc theo Clean Architecture |
| **Dependencies** | `flutter pub add get_it injectable fpdart` |

### Mục tiêu
Lấy todo app (dạng spaghetti từ BT1 hoặc từ buổi 08) và **tái cấu trúc** theo Clean Architecture.

### Yêu cầu

#### Bước 1: Tạo cấu trúc folder

```
lib/
├── core/
│   └── error/
│       └── exceptions.dart
├── features/
│   └── todo/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── todo.dart
│       │   ├── repositories/
│       │   │   └── todo_repository.dart
│       │   └── usecases/
│       │       ├── get_incomplete_todos.dart
│       │       └── toggle_todo.dart
│       ├── data/
│       │   ├── models/
│       │   │   └── todo_model.dart
│       │   ├── datasources/
│       │   │   ├── todo_remote_data_source.dart
│       │   │   └── todo_local_data_source.dart
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
├── injection_container.dart
└── main.dart
```

#### Bước 2: Domain Layer

**todo.dart** — Entity:
```dart
class Todo {
  final int id;
  final String title;
  final bool completed;
  final int userId;

  const Todo({
    required this.id,
    required this.title,
    required this.completed,
    required this.userId,
  });
}
```

**todo_repository.dart** — Interface:
```dart
abstract class TodoRepository {
  Future<List<Todo>> getAllTodos();
  Future<Todo> toggleTodo(int id, bool completed);
}
```

**get_incomplete_todos.dart** — Use Case:
```dart
class GetIncompleteTodosUseCase {
  final TodoRepository repository;

  GetIncompleteTodosUseCase(this.repository);

  Future<List<Todo>> call() async {
    final todos = await repository.getAllTodos();
    return todos.where((t) => !t.completed).toList();
  }
}
```

#### Bước 3: Data Layer

Implement `TodoModel`, `TodoRemoteDataSource`, `TodoLocalDataSource`, `TodoRepositoryImpl`.

- `TodoModel` extends `Todo`, thêm `fromJson`/`toJson`
- Remote gọi `https://jsonplaceholder.typicode.com/todos`
- Local dùng `SharedPreferences` để cache
- Repository: try remote → cache → fallback local

#### Bước 4: Presentation Layer

- `TodoCubit` sử dụng Use Cases
- `TodoState`: `Initial`, `Loading`, `Loaded(List<Todo>)`, `Error(String)`
- `TodoListPage` dùng `BlocBuilder`
- `TodoItem` widget riêng biệt

#### Bước 5: Wiring (Manual DI)

Trong `injection_container.dart`, tạo tất cả dependencies và inject.

### Checklist kiểm tra

- [ ] Domain layer KHÔNG import `package:flutter` hay `package:http`
- [ ] Entity không có `fromJson`/`toJson`
- [ ] Use Case chỉ gọi Repository interface (abstract class)
- [ ] Data layer import từ Domain (entities, repositories)
- [ ] Presentation KHÔNG import từ Data layer
- [ ] Cubit chỉ biết Use Cases và Entities
- [ ] App chạy được, hiển thị danh sách todos

### Gợi ý cho người mới

Nếu chưa quen, hãy làm theo thứ tự:
1. **Domain trước** — viết Entity → Repository interface → Use Cases
2. **Data tiếp** — viết Model → Data Sources → Repository impl
3. **Presentation cuối** — viết States → Cubit → Pages
4. **Wire lại** — injection_container.dart → main.dart

---

## BT3 ⭐⭐⭐ Xây dựng feature "Notes" từ đầu với Clean Architecture 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_clean_notes` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Notes app hoàn chỉnh theo Clean Architecture |
| **Dependencies** | `flutter pub add get_it injectable fpdart` |

### Mục tiêu
Build một feature hoàn chỉnh **từ số 0** theo Clean Architecture — không refactor code có sẵn.

### Yêu cầu chức năng

Feature "Notes" cho phép:
- Xem danh sách notes
- Tạo note mới (title + content)
- Xóa note
- Đánh dấu note là "pinned" (ghim)
- Hiển thị pinned notes trước

### Bước 1: Domain Layer

**Entity — Note:**
```dart
class Note {
  final String id;
  final String title;
  final String content;
  final bool isPinned;
  final DateTime createdAt;
  final DateTime updatedAt;

  const Note({
    required this.id,
    required this.title,
    required this.content,
    required this.isPinned,
    required this.createdAt,
    required this.updatedAt,
  });

  // Business rules
  bool get isEmpty => title.isEmpty && content.isEmpty;
  String get preview =>
      content.length > 100 ? '${content.substring(0, 100)}...' : content;
}
```

**Repository interface:**
```dart
abstract class NoteRepository {
  Future<List<Note>> getAllNotes();
  Future<Note> getNoteById(String id);
  Future<Note> createNote({required String title, required String content});
  Future<void> deleteNote(String id);
  Future<Note> togglePin(String id);
}
```

**Use Cases (tự implement):**
- `GetAllNotesUseCase` — trả về notes, pinned notes trước, rồi sort by updatedAt
- `CreateNoteUseCase` — validation: title không rỗng, content không rỗng
- `DeleteNoteUseCase` — xóa note theo id
- `TogglePinUseCase` — ghim/bỏ ghim note

### Bước 2: Data Layer

Vì chưa học API (Tuần 6), dùng **in-memory fake data source**:

```dart
/// Fake data source — giả lập API bằng List trong memory
class NoteLocalDataSourceImpl implements NoteLocalDataSource {
  final List<NoteModel> _notes = [
    NoteModel(
      id: '1',
      title: 'Welcome Note',
      content: 'This is your first note. Try creating more!',
      isPinned: true,
      createdAt: DateTime.now().subtract(const Duration(days: 1)),
      updatedAt: DateTime.now().subtract(const Duration(days: 1)),
    ),
    NoteModel(
      id: '2',
      title: 'Shopping List',
      content: 'Milk, Eggs, Bread, Coffee',
      isPinned: false,
      createdAt: DateTime.now().subtract(const Duration(hours: 5)),
      updatedAt: DateTime.now().subtract(const Duration(hours: 5)),
    ),
  ];

  @override
  Future<List<NoteModel>> getAllNotes() async {
    // Giả lập network delay
    await Future.delayed(const Duration(milliseconds: 500));
    return List.from(_notes);
  }

  @override
  Future<NoteModel> createNote({
    required String title,
    required String content,
  }) async {
    await Future.delayed(const Duration(milliseconds: 300));

    final note = NoteModel(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title,
      content: content,
      isPinned: false,
      createdAt: DateTime.now(),
      updatedAt: DateTime.now(),
    );

    _notes.add(note);
    return note;
  }

  // ... implement deleteNote, togglePin, getNoteById
}
```

### Bước 3: Presentation Layer

Dùng **Cubit** (hoặc BLoC nếu muốn thử):

**States:**
- `NotesInitial`
- `NotesLoading`
- `NotesLoaded(List<Note> notes)` — pinned notes đã được sort lên trước
- `NoteCreated(Note note)`
- `NoteDeleted`
- `NotesError(String message)`

**Pages/Widgets:**
- `NoteListPage` — hiển thị danh sách, FAB để tạo mới
- `CreateNotePage` — form tạo note (title + content)
- `NoteCard` — widget hiển thị 1 note (có icon pin, swipe to delete)

### Bước 4: Wiring + Main

Tạo `injection_container.dart` với manual DI, cung cấp `NoteCubit` qua `BlocProvider`.

### Cấu trúc folder hoàn chỉnh

```
lib/
├── core/
│   └── error/
│       └── exceptions.dart
├── features/
│   └── notes/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── note.dart
│       │   ├── repositories/
│       │   │   └── note_repository.dart
│       │   └── usecases/
│       │       ├── get_all_notes.dart
│       │       ├── create_note.dart
│       │       ├── delete_note.dart
│       │       └── toggle_pin.dart
│       ├── data/
│       │   ├── models/
│       │   │   └── note_model.dart
│       │   ├── datasources/
│       │   │   └── note_local_data_source.dart
│       │   └── repositories/
│       │       └── note_repository_impl.dart
│       └── presentation/
│           ├── cubit/
│           │   ├── note_cubit.dart
│           │   └── note_state.dart
│           ├── pages/
│           │   ├── note_list_page.dart
│           │   └── create_note_page.dart
│           └── widgets/
│               └── note_card.dart
├── injection_container.dart
└── main.dart
```

### Checklist kiểm tra

- [ ] Domain layer: 0 external imports (chỉ Dart core)
- [ ] Entity `Note` immutable, có business rules (`isEmpty`, `preview`)
- [ ] 4 Use Cases, mỗi cái 1 file
- [ ] `NoteModel` extends `Note`
- [ ] Data source dùng in-memory (fake API)
- [ ] Cubit chỉ import từ domain/
- [ ] UI hiển thị: danh sách notes, pinned lên trước
- [ ] Tạo note mới hoạt động
- [ ] Xóa note hoạt động
- [ ] Toggle pin hoạt động
- [ ] Không vi phạm Dependency Rule (search import paths)

### Bonus challenges

- Thêm **search/filter** notes (tạo `SearchNotesUseCase`)
- Thêm **edit note** (tạo `UpdateNoteUseCase`)
- Thay in-memory bằng `SharedPreferences` hoặc `sqflite`
- Thêm animation khi thêm/xóa note

---

## 💬 Câu hỏi thảo luận

### Q1: Khi nào Clean Architecture là "overkill"?

Suy nghĩ về các trường hợp:
- App có 1-2 trang đơn giản (ví dụ: calculator app)
- MVP / prototype cần ship nhanh trong 1 tuần
- Side project 1 người dev
- App production với 10+ features, 5+ developers

> **Gợi ý:** Không có đáp án tuyệt đối. Clean Architecture là trade-off giữa **setup cost** (phải tạo nhiều file/class) và **maintenance benefit** (dễ maintain, test, mở rộng). Với app nhỏ, chi phí setup > lợi ích. Với app lớn, lợi ích >> chi phí.

### Q2: Feature-first hay Layer-first?

So sánh 2 cách tổ chức:

```
Feature-first:                    Layer-first:
features/                         domain/
  auth/                             entities/
    domain/                           user.dart
    data/                             todo.dart
    presentation/                   usecases/
  todo/                               get_user.dart
    domain/                           create_todo.dart
    data/                         data/
    presentation/                   models/
                                    datasources/
                                  presentation/
                                    pages/
                                    widgets/
```

Câu hỏi:
- Khi nào feature-first tốt hơn?
- Khi nào layer-first tốt hơn?
- Nếu 2 features share chung 1 entity thì xử lý thế nào?
- Team bạn 5 người, mỗi người phụ trách 1 feature — structure nào hợp lý hơn?

### Q3: Lợi ích testing của Domain layer

Tại sao Domain layer (đặc biệt Use Cases) dễ test hơn các layer khác?

Gợi ý suy nghĩ:
- Domain không phụ thuộc framework → test không cần Flutter test environment
- Repository là abstract → dễ mock/fake
- Use Case có **1 nhiệm vụ** → test case rõ ràng

```dart
// Ví dụ test Use Case — không cần Flutter, không cần API thật
void main() {
  test('GetActiveUsersUseCase filters inactive users', () async {
    // Arrange: fake repository
    final fakeRepo = FakeUserRepository([
      User(id: '1', name: 'A', email: 'a@b.com', isActive: true, ...),
      User(id: '2', name: 'B', email: 'b@b.com', isActive: false, ...),
      User(id: '3', name: 'C', email: 'c@b.com', isActive: true, ...),
    ]);

    final useCase = GetActiveUsersUseCase(fakeRepo);

    // Act
    final result = await useCase();

    // Assert: chỉ có 2 active users
    expect(result.length, 2);
    expect(result.every((u) => u.isActive), true);
  });
}
```

Câu hỏi: Nếu business logic nằm trong Widget (như BT1), test có dễ không? Tại sao?

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 5:** Focus vào gen scaffold architecture và review dependency violations.

### AI-BT1: Gen Clean Architecture Scaffold cho "User Profile" Feature ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** 3 layers (Domain, Data, Presentation), Dependency Rule, Repository Pattern, Use Cases, Entity vs DTO.
- **Task thực tế:** PM giao "User Profile" feature — xem/sửa profile, upload avatar, change password. Cần scaffold toàn bộ Clean Architecture structure trước khi code logic.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần scaffold Clean Architecture cho "User Profile" feature.
Tech stack: Flutter 3.x, riverpod ^2.x, dartz (Either), freezed.
Feature scope:
- GetUserProfile(userId) → UserEntity
- UpdateUserProfile(UserEntity) → void
- ChangePassword(oldPass, newPass) → void

Scaffold 3 layers:
Domain:
- entities/user_entity.dart (freezed)
- repositories/user_repository.dart (abstract)
- usecases/get_user_profile.dart, update_user_profile.dart, change_password.dart

Data:
- models/user_model.dart (DTO, fromJson/toJson, toDomain)
- datasources/user_remote_data_source.dart (abstract + impl)
- repositories/user_repository_impl.dart (implements UserRepository)

Presentation:
- providers/user_profile_provider.dart (Riverpod)
- screens/user_profile_screen.dart (ConsumerWidget)
- widgets/profile_form.dart

Constraints:
- Dependency Rule: Domain imports NOTHING from Data/Presentation.
- Data: imports Domain entities/repos for interface, NEVER imports Presentation.
- Presentation: imports Domain use cases/entities, NEVER imports Data directly.
- Each Use Case: single method call(), returns Future<Either<Failure, T>>.
Output: folder structure + ALL file stubs (class declarations, method signatures, no implementation).
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* Folder structure + 10+ file stubs.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Domain layer có import Data hoặc Presentation không? (MUST = NO) | ☐ |
| 2 | Abstract `UserRepository` ở Domain, concrete `UserRepositoryImpl` ở Data? | ☐ |
| 3 | Use Cases có single responsibility (1 use case = 1 action)? | ☐ |
| 4 | `UserModel` (Data) khác `UserEntity` (Domain)? Có mapping method? | ☐ |
| 5 | DataSource là abstract class (để mock trong test)? | ☐ |
| 6 | Presentation dùng Use Cases (không dùng Repository trực tiếp)? | ☐ |
| 7 | Return types dùng `Either<Failure, T>` (không throw exception)? | ☐ |

**4. Customize:**
Thêm `UserLocalDataSource` (Hive cache) trong Data layer. Repository fallback: remote → cache → failure. AI gen chỉ có remote — tự thêm local data source + cache-first strategy.
