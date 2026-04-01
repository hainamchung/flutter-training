# Buổi 12: Local Storage & Data Persistence — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

1. [BT1 ⭐ — App Settings với SharedPreferences](#bt1--app-settings-với-sharedpreferences)
2. [BT2 ⭐⭐ — Todo App với Hive](#bt2--todo-app-với-hive)
3. [BT3 ⭐⭐⭐ — Offline-First Notes App](#bt3--offline-first-notes-app)
4. [Câu hỏi thảo luận](#câu-hỏi-thảo-luận)

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới local storage — FE developer sẽ tìm thấy **parallels với localStorage & IndexedDB** nhưng API rất khác.
> Đọc bảng dưới TRƯỚC khi code để tránh nhầm lẫn.

| FE Storage Habit | Flutter Reality | Bài tập liên quan |
|------------------|-----------------|---------------------|
| `localStorage.setItem(key, JSON.stringify(obj))` | SharedPreferences: `prefs.setString(key, value)` — **async** + typed | BT1 |
| Hiếm khi dùng IndexedDB | SQLite/Drift = **standard** cho structured data — phải biết SQL basics | BT2 |
| Service Worker cache responses | Tự implement cache logic: check local → fetch → save local | BT2, BT3 |

---

## BT1 ⭐ — App Settings với SharedPreferences 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_app_settings` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — màn hình Settings với dark mode, language, font size |
| **Dependencies** | `flutter pub add shared_preferences` |

### Yêu cầu

Xây dựng màn hình Settings cho phép user thay đổi các cài đặt app. Tất cả settings phải **persist qua app restart**.

### Chức năng

| Setting | Loại | Mặc định |
|---------|------|----------|
| Dark Mode | `bool` (toggle) | `false` |
| Language | `String` (dropdown: 'vi', 'en', 'ja') | `'vi'` |
| Font Size | `double` (slider: 10.0 → 24.0) | `14.0` |
| Notifications | `bool` (toggle) | `true` |
| Username | `String` (text field) | `''` |

### Yêu cầu kỹ thuật

1. Tạo `SettingsService` class wrap SharedPreferences
2. Mỗi setting có getter (đọc) và setter (ghi)
3. Có nút "Reset to Defaults" xóa tất cả settings
4. App phải tự apply dark mode dựa trên saved setting khi khởi động
5. Font size thay đổi phải phản ánh ngay trên UI

### Skeleton code

```dart
// settings_service.dart
import 'package:shared_preferences/shared_preferences.dart';

class SettingsService {
  final SharedPreferences _prefs;

  SettingsService(this._prefs);

  // TODO: Implement getters & setters cho mỗi setting
  // Gợi ý: dùng _prefs.getBool(), _prefs.getString(), _prefs.getDouble()
  //         dùng _prefs.setBool(), _prefs.setString(), _prefs.setDouble()

  // Dark Mode
  bool get isDarkMode => /* TODO */;
  Future<void> setDarkMode(bool value) async {
    // TODO
  }

  // Language
  String get language => /* TODO */;
  Future<void> setLanguage(String value) async {
    // TODO
  }

  // Font Size
  double get fontSize => /* TODO */;
  Future<void> setFontSize(double value) async {
    // TODO
  }

  // Notifications
  bool get notificationsEnabled => /* TODO */;
  Future<void> setNotificationsEnabled(bool value) async {
    // TODO
  }

  // Username
  String get username => /* TODO */;
  Future<void> setUsername(String value) async {
    // TODO
  }

  // Reset all
  Future<void> resetToDefaults() async {
    // TODO: clear all settings
  }
}
```

```dart
// settings_screen.dart
import 'package:flutter/material.dart';
import 'settings_service.dart';

class SettingsScreen extends StatefulWidget {
  final SettingsService settings;
  const SettingsScreen({super.key, required this.settings});

  @override
  State<SettingsScreen> createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings'),
        actions: [
          TextButton(
            onPressed: () async {
              await widget.settings.resetToDefaults();
              setState(() {});
            },
            child: const Text('Reset'),
          ),
        ],
      ),
      body: ListView(
        children: [
          // TODO: SwitchListTile cho Dark Mode
          // TODO: ListTile với DropdownButton cho Language
          // TODO: ListTile với Slider cho Font Size
          // TODO: SwitchListTile cho Notifications
          // TODO: ListTile với TextField cho Username
        ],
      ),
    );
  }
}
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

- Dùng `SwitchListTile` cho bool settings
- Dùng `DropdownButton` cho language selection  
- Dùng `Slider` cho font size
- Truyền `SettingsService` từ `main()` sau khi `SharedPreferences.getInstance()`
- Để apply dark mode: dùng `MaterialApp(themeMode: settings.isDarkMode ? ThemeMode.dark : ThemeMode.light)`

</details>

### Tiêu chí hoàn thành

- [ ] Tất cả 5 settings hoạt động (đọc/ghi)
- [ ] App restart vẫn giữ settings cũ
- [ ] Dark mode apply lên toàn app
- [ ] Font size thay đổi ngay trên UI
- [ ] Reset to Defaults hoạt động đúng

---

## BT2 ⭐⭐ — Todo App với Hive 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_hive_todo` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Todo app CRUD với 3 tabs (All/Active/Completed) |
| **Dependencies** | `flutter pub add hive hive_flutter uuid` + `flutter pub add dev:hive_generator dev:build_runner` |

### Yêu cầu

Xây dựng Todo app đầy đủ chức năng CRUD sử dụng Hive cho persistence. Data phải sống sót qua app restart.

### Chức năng

1. **Xem** danh sách todos (show tất cả, filter: all/active/completed)
2. **Thêm** todo mới (title + optional description)
3. **Sửa** todo (thay đổi title, description)
4. **Toggle** trạng thái done/undone
5. **Xóa** todo đơn lẻ
6. **Xóa** tất cả todos đã completed
7. **Đếm** active/completed todos

### Data Model

```dart
// todo_model.dart
import 'package:hive/hive.dart';

part 'todo_model.g.dart';

@HiveType(typeId: 0)
class TodoModel extends HiveObject {
  @HiveField(0)
  final String id;

  @HiveField(1)
  String title;

  @HiveField(2)
  String? description;

  @HiveField(3)
  bool isDone;

  @HiveField(4)
  final DateTime createdAt;

  // TODO: Constructor, copyWith method
}
```

### Yêu cầu kỹ thuật

1. Tạo `TodoModel` với `@HiveType` annotation
2. Tạo `TodoRepository` class wrap Hive Box
3. Implement đầy đủ CRUD methods
4. UI có 3 tabs: All / Active / Completed
5. Hiển thị counter: "3 items left"
6. Chạy `build_runner` để generate TypeAdapter

### Skeleton — TodoRepository

```dart
// todo_repository.dart
import 'package:hive/hive.dart';
import 'todo_model.dart';

class TodoRepository {
  static const _boxName = 'todos';

  Future<Box<TodoModel>> _getBox() async {
    // TODO: Return opened box (check if already open)
  }

  // CREATE
  Future<void> addTodo(TodoModel todo) async {
    // TODO: put todo vào box với todo.id làm key
  }

  // READ
  Future<List<TodoModel>> getAllTodos() async {
    // TODO: return box.values.toList(), sort by createdAt desc
  }

  Future<List<TodoModel>> getActiveTodos() async {
    // TODO: filter isDone == false
  }

  Future<List<TodoModel>> getCompletedTodos() async {
    // TODO: filter isDone == true
  }

  // UPDATE
  Future<void> updateTodo(TodoModel todo) async {
    // TODO: put updated todo
  }

  Future<void> toggleDone(String id) async {
    // TODO: get todo, toggle isDone, put back
  }

  // DELETE
  Future<void> deleteTodo(String id) async {
    // TODO: box.delete(id)
  }

  Future<void> deleteCompleted() async {
    // TODO: find all completed, deleteAll
  }

  // COUNT
  Future<int> get activeCount async {
    // TODO: count where isDone == false
  }
}
```

### Skeleton — UI

```dart
// todo_screen.dart
class TodoScreen extends StatefulWidget {
  const TodoScreen({super.key});

  @override
  State<TodoScreen> createState() => _TodoScreenState();
}

class _TodoScreenState extends State<TodoScreen>
    with SingleTickerProviderStateMixin {
  final _repo = TodoRepository();
  late TabController _tabController;

  // TODO: TabController với 3 tabs (All, Active, Completed)

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo App'),
        bottom: TabBar(
          controller: _tabController,
          tabs: const [
            Tab(text: 'All'),
            Tab(text: 'Active'),
            Tab(text: 'Completed'),
          ],
        ),
        actions: [
          // TODO: Button xóa completed todos
        ],
      ),
      body: TabBarView(
        controller: _tabController,
        children: [
          // TODO: _buildTodoList(_repo.getAllTodos()),
          // TODO: _buildTodoList(_repo.getActiveTodos()),
          // TODO: _buildTodoList(_repo.getCompletedTodos()),
        ],
      ),
      // TODO: FAB để thêm todo mới (show dialog)
      // TODO: Bottom bar hiển thị "X items left"
    );
  }

  // TODO: _buildTodoList(Future<List<TodoModel>> todosFuture)
  // TODO: _showAddTodoDialog()
  // TODO: _showEditTodoDialog(TodoModel todo)
}
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

- Dùng `uuid` package để tạo unique id: `const Uuid().v4()`
- `box.put(todo.id, todo)` — dùng id làm key cho easy lookup
- Sau mỗi write operation, gọi `setState(() {})` để refresh FutureBuilder
- Dùng `showDialog()` cho add/edit forms
- Dùng `Dismissible` widget cho swipe-to-delete

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
class TodoModel extends HiveObject {
  final String id;
  String title;
  String? description;
  bool isDone;
  final DateTime createdAt;

  TodoModel({
    required this.id,
    required this.title,
    this.description,
    this.isDone = false,
    required this.createdAt,
  });

  TodoModel copyWith({String? title, String? description, bool? isDone}) {
    return TodoModel(
      id: id,
      title: title ?? this.title,
      description: description ?? this.description,
      isDone: isDone ?? this.isDone,
      createdAt: createdAt,
    );
  }
}
```

</details>

### Setup

```bash
# 1. Thêm dependencies
flutter pub add hive hive_flutter uuid
flutter pub add --dev hive_generator build_runner

# 2. Tạo model với annotations

# 3. Generate code
dart run build_runner build

# 4. Khởi tạo trong main()
# await Hive.initFlutter();
# Hive.registerAdapter(TodoModelAdapter());
```

### Tiêu chí hoàn thành

- [ ] CRUD hoạt động đầy đủ (tạo, đọc, sửa, xóa)
- [ ] 3 tabs filter hoạt động (All, Active, Completed)
- [ ] Toggle done/undone hoạt động
- [ ] Delete completed hoạt động
- [ ] Counter hiển thị đúng
- [ ] Data persist qua app restart
- [ ] TypeAdapter generated thành công

---

## BT3 ⭐⭐⭐ — Offline-First Notes App 🟡

> 💡 **BT3 là bài khó nhất buổi này.** Gợi ý chia thành 3 bước:
> 1. **Bước 1**: Làm CRUD với Drift/Hive hoạt động offline (dựa trên BT2)
> 2. **Bước 2**: Thêm Dio fetch data từ API
> 3. **Bước 3**: Implement sync logic (pull → merge → push)

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_offline_notes` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Notes app offline-first với Drift + sync engine |
| **Dependencies** | `flutter pub add drift sqlite3_flutter_libs path_provider path dio connectivity_plus` + `flutter pub add dev:drift_dev dev:build_runner` |

### Yêu cầu

Xây dựng Notes app với offline-first architecture:
- **Drift** cho local SQLite database
- **Dio** cho API calls  
- Cache-first read strategy
- Queue writes khi offline
- Sync khi connectivity trở lại

### Architecture

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌─────────┐
│    UI    │────▶│   NoteRepo   │────▶│  Local DB    │     │   API   │
│          │◀────│              │◀────│   (Drift)    │     │ Server  │
│ (Screen) │     │              │     └──────────────┘     └─────────┘
└──────────┘     │  ┌────────┐  │            │                  │
                 │  │ Cache  │  │            │                  │
                 │  │Strategy│  │            │                  │
                 │  └────────┘  │            │                  │
                 │              │     ┌──────────────┐          │
                 │  ┌────────┐  │     │ Pending Ops  │──sync──▶│
                 │  │ Sync   │  │     │   Queue      │          │
                 │  │ Engine │  │     └──────────────┘          │
                 │  └────────┘  │                               │
                 └──────────────┘                               │
```

### Data Model

```dart
// Drift table
class Notes extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 200)();
  TextColumn get content => text()();
  TextColumn get color => text().withDefault(const Constant('#FFFFFF'))();
  BoolColumn get isPinned => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();

  // Sync metadata
  IntColumn get remoteId => integer().nullable()();  // server ID
  BoolColumn get isSynced => boolean().withDefault(const Constant(false))();
}

// Pending operations queue
class PendingOperations extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get operationType => text()();   // 'create', 'update', 'delete'
  TextColumn get entityType => text()();      // 'note'
  IntColumn get entityId => integer()();
  TextColumn get payload => text()();         // JSON
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}
```

### Các component cần implement

#### 1. NoteDao (Drift DAO)

```dart
// TODO: Implement
@DriftAccessor(tables: [Notes, PendingOperations])
class NoteDao extends DatabaseAccessor<AppDatabase> with _$NoteDaoMixin {
  NoteDao(AppDatabase db) : super(db);

  // CRUD cho Notes
  Stream<List<Note>> watchAllNotes();
  Future<Note?> getNoteById(int id);
  Future<int> insertNote(NotesCompanion note);
  Future<bool> updateNote(Note note);
  Future<int> deleteNoteById(int id);

  // Pending operations
  Future<void> addPendingOperation(PendingOperationsCompanion op);
  Future<List<PendingOperation>> getPendingOperations();
  Future<void> markOperationComplete(int opId);
}
```

#### 2. NoteApiService (Remote)

```dart
// TODO: Implement
class NoteApiService {
  final Dio _dio;

  NoteApiService(this._dio);

  Future<List<NoteDto>> fetchNotes();
  Future<NoteDto> createNote(NoteDto note);
  Future<NoteDto> updateNote(int id, NoteDto note);
  Future<void> deleteNote(int id);
}
```

#### 3. ConnectivityService

```dart
// TODO: Implement
class ConnectivityService {
  final Connectivity _connectivity;

  Future<bool> get hasConnection;
  Stream<bool> get onConnectivityChanged;
}
```

#### 4. SyncEngine

```dart
// TODO: Implement
class SyncEngine {
  final NoteApiService _api;
  final NoteDao _dao;
  final ConnectivityService _connectivity;

  /// Push pending local changes to server
  Future<void> pushPendingChanges();

  /// Pull latest data from server
  Future<void> pullLatestData();

  /// Full sync (push then pull)
  Future<void> sync();

  /// Start listening for connectivity changes
  void startAutoSync();

  void dispose();
}
```

#### 5. NoteRepository (Orchestrator)

```dart
// TODO: Implement
class NoteRepository {
  final NoteDao _dao;
  final NoteApiService _api;
  final SyncEngine _syncEngine;
  final ConnectivityService _connectivity;

  /// READ: luôn từ local DB (Stream)
  Stream<List<Note>> watchNotes() => _dao.watchAllNotes();

  /// CREATE: lưu local + queue sync
  Future<void> createNote({required String title, required String content}) async {
    // 1. Insert vào local DB
    // 2. Add pending operation (type: 'create')
    // 3. Trigger sync nếu online
  }

  /// UPDATE: update local + queue sync
  Future<void> updateNote(Note note) async {
    // 1. Update local DB
    // 2. Add pending operation (type: 'update')
    // 3. Trigger sync nếu online
  }

  /// DELETE: delete local + queue sync
  Future<void> deleteNote(int noteId) async {
    // 1. Delete from local DB
    // 2. Add pending operation (type: 'delete')
    // 3. Trigger sync nếu online
  }
}
```

### UI Screen

```dart
// note_list_screen.dart
class NoteListScreen extends StatelessWidget {
  final NoteRepository repository;

  const NoteListScreen({super.key, required this.repository});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Notes'),
        actions: [
          // TODO: Online/offline indicator icon
          // TODO: Manual sync button
        ],
      ),
      body: StreamBuilder<List<Note>>(
        stream: repository.watchNotes(),
        builder: (context, snapshot) {
          // TODO: Build note list
          // Hiển thị sync status (synced/pending) cho mỗi note
        },
      ),
      // TODO: FAB thêm note mới
    );
  }
}
```

### Flow chính

```
=== ONLINE ===
User tạo note → Save local DB → Add to sync queue → Push to API → Mark synced

=== OFFLINE ===
User tạo note → Save local DB → Add to sync queue → (đợi)

=== Connectivity restored ===
connectivity_plus detect online → SyncEngine.sync() →
  Push pending ops → Pull latest → Update local DB → UI auto-updates via Stream
```

### Setup

```bash
flutter pub add drift sqlite3_flutter_libs path_provider path dio connectivity_plus
flutter pub add --dev drift_dev build_runner
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

- Dùng **Stream** (Drift `.watch()`) cho UI — tự update khi local DB thay đổi
- `PendingOperations` table lưu các ops chưa sync
- `SyncEngine` chạy khi online, xử lý từng pending op
- Dùng `connectivity_plus` để detect online/offline
- Hiển thị icon nhỏ trên mỗi note cho biết đã sync hay chưa
- Khi offline, UI vẫn hoạt động bình thường vì đọc từ local DB
- Có thể mock API bằng JSONPlaceholder hoặc json-server

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// sync_engine.dart
class SyncEngine {
  final NoteApiService _api;
  final NoteDao _dao;
  final ConnectivityService _connectivity;
  StreamSubscription? _connectivitySub;

  SyncEngine(this._api, this._dao, this._connectivity);

  Future<void> sync() async {
    await pushPendingChanges();
    await pullLatestData();
  }

  Future<void> pushPendingChanges() async {
    final ops = await _dao.getPendingOperations();
    for (final op in ops) {
      // TODO: execute op based on operationType
      // TODO: mark complete after success
    }
  }

  void startAutoSync() {
    _connectivitySub = _connectivity.onConnectivityChanged.listen((online) {
      if (online) sync();
    });
  }

  void dispose() => _connectivitySub?.cancel();
}
```

</details>

### Tiêu chí hoàn thành

- [ ] Local CRUD hoạt động (Drift)
- [ ] UI dùng StreamBuilder, auto-update
- [ ] Pending operations được queue khi offline
- [ ] Sync tự động khi online trở lại
- [ ] UI hiển thị online/offline status
- [ ] UI hiển thị sync status cho mỗi note
- [ ] Data persist qua app restart

---

## Câu hỏi thảo luận

### Câu 1: Chọn storage solution

> Bạn đang xây dựng một app thương mại điện tử. Cần lưu trữ:
> - User preferences (theme, language)
> - Danh sách sản phẩm yêu thích (có thể hàng trăm items)
> - Giỏ hàng với quan hệ product ↔ quantity ↔ price
> - Auth token
>
> **Hỏi:** Bạn sẽ chọn storage nào cho mỗi loại dữ liệu? Giải thích lý do.

**Gợi ý trả lời:**
- User preferences → SharedPreferences (key-value đơn giản, ít data)
- Sản phẩm yêu thích → Hive (structured objects, không cần relational, fast read)
- Giỏ hàng → Drift (relational: product ↔ cart_item, cần queries phức tạp)
- Auth token → flutter_secure_storage (sensitive data, cần mã hóa)

### Câu 2: Offline-first complexity

> Team bạn đang tranh luận về offline-first approach cho app đọc tin tức:
> - Dev A: "Cứ network-first, offline thì show error"
> - Dev B: "Phải offline-first, user cần đọc tin khi không có mạng"
>
> **Hỏi:** Bạn đứng về phía nào? Phân tích trade-offs của mỗi approach. Có middle-ground nào không?

**Gợi ý thảo luận:**
- Network-first: đơn giản, luôn có data mới nhất, nhưng UX kém khi offline
- Offline-first: UX tốt, phức tạp hơn (sync engine, conflict resolution, storage)
- Middle-ground: **Stale-while-revalidate** — show cached data ngay, fetch mới ở background
- Phụ thuộc vào use case: tin tức có thể dùng stale data, banking cần data mới nhất

### Câu 3: Data migration

> App bạn đang chạy production với Hive, có model:
> ```dart
> @HiveType(typeId: 0)
> class UserProfile extends HiveObject {
>   @HiveField(0)
>   String name;
>   @HiveField(1)
>   String email;
> }
> ```
> Giờ cần thêm field `phoneNumber` và `avatarUrl`.
>
> **Hỏi:** Làm sao thêm fields mà không break existing data? Nếu dùng Drift thay vì Hive, cách handle migration có gì khác?

**Gợi ý trả lời:**
- **Hive**: Thêm `@HiveField(2) String? phoneNumber` và `@HiveField(3) String? avatarUrl` — PHẢI dùng field number mới (2, 3), KHÔNG thay đổi field numbers cũ. Fields mới phải nullable (hoặc có default). Hive tự handle: records cũ trả về null cho fields mới.
- **Drift**: Tăng `schemaVersion`, viết `onUpgrade` migration:
  ```dart
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.addColumn(userProfiles, userProfiles.phoneNumber);
      await m.addColumn(userProfiles, userProfiles.avatarUrl);
    }
  }
  ```
- Key difference: Hive migration là "free" cho additive changes, Drift cần explicit migration code nhưng an toàn hơn (compile-time checks).

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 6:** Focus vào gen complete data layer và review caching/security.

### AI-BT1: Gen Caching Layer với Repository Pattern (Cache-first + TTL + Offline Queue) ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** SharedPreferences, Hive, Drift, caching strategies, offline-first pattern, secure storage.
- **Task thực tế:** PM yêu cầu app Notes hoạt động offline — cached data hiển thị ngay, write operations queued khi offline, sync khi online. AI gen caching layer, bạn review TTL + cache invalidation + sync conflicts.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần implement caching layer cho Notes feature theo Repository pattern.
Tech stack: Flutter 3.x, hive ^2.x, dio ^5.x, connectivity_plus ^5.x.
Yêu cầu:
1. NoteRepository: cache-first strategy.
   - Read: return cache → fetch background → update cache → emit stream.
   - Write online: API → cache → success.
   - Write offline: queue → show "pending" badge → sync khi online.
2. CacheManager: setWithTTL(key, data, 5.minutes), get(key), isExpired(key).
3. SyncManager: offline write queue (Hive box), replay khi online, conflict = server wins.
4. ConnectivityService: Stream<bool> isOnline.
Constraints:
- Repository return Stream<List<Note>> (not Future — reactive).
- TTL: 5 minutes, stale → background refresh.
- Queue max: 50 items.
Output: 4 files.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 4 files caching layer.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Cache-first: return cache trước, fetch background? (không await API) | ☐ |
| 2 | TTL implemented? Cache expire sau 5 minutes? | ☐ |
| 3 | Offline queue: writes queued khi offline? | ☐ |
| 4 | Sync: replay queue khi online? Handle failures? | ☐ |
| 5 | Repository return Stream (not Future)? | ☐ |
| 6 | Queue max 50? Oldest dropped khi full? | ☐ |
| 7 | Cache invalidation: khi write thành công → invalidate related cache? | ☐ |

**4. Customize:**
Thêm optimistic update: UI update ngay (assume success) → rollback nếu API fail. AI gen pessimistic (wait for API) — tự đổi sang optimistic + rollback logic.
