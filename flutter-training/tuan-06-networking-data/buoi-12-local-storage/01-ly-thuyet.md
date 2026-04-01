# Buổi 12: Local Storage & Data Persistence — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 12/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 09 (lý thuyết + ít nhất BT1-BT2) và Buổi 11 (Networking)

## Mục lục

1. [SharedPreferences — Key-Value Storage](#1-sharedpreferences--key-value-storage)
2. [Hive — NoSQL Database](#2-hive--nosql-database)
3. [Drift (SQLite) — Relational Database](#3-drift-sqlite--relational-database)
4. [Caching Strategy](#4-caching-strategy)
5. [Offline-First Pattern](#5-offline-first-pattern)
6. [Secure Storage](#6-secure-storage)
7. [Best Practices & Lỗi thường gặp](#7-best-practices--lỗi-thường-gặp)
8. [💡 FE → Flutter: Góc nhìn chuyển đổi](#8--fe--flutter-góc-nhìn-chuyển-đổi)
9. [Tổng kết](#9-tổng-kết)

---

## 1. SharedPreferences — Key-Value Storage 🟡

### 1.1 Khái niệm

SharedPreferences là cơ chế lưu trữ **key-value đơn giản** trên cả Android (SharedPreferences) và iOS (NSUserDefaults). Nó hoạt động tương tự `localStorage` trong web.

```
┌─────────────────────────────────┐
│        SharedPreferences        │
│                                 │
│   Key (String) → Value          │
│   ─────────────────────         │
│   "theme"      → "dark"        │
│   "language"   → "vi"          │
│   "fontSize"   → 16.0          │
│   "isLoggedIn" → true          │
│   "tags"       → ["a","b"]     │
│                                 │
│   Supported types:              │
│   String, int, double,          │
│   bool, List<String>            │
└─────────────────────────────────┘
```

### 1.2 Các kiểu dữ liệu hỗ trợ

| Kiểu | Ví dụ | Method |
|------|-------|--------|
| `String` | `"dark"` | `setString()` / `getString()` |
| `int` | `42` | `setInt()` / `getInt()` |
| `double` | `16.0` | `setDouble()` / `getDouble()` |
| `bool` | `true` | `setBool()` / `getBool()` |
| `List<String>` | `["vi", "en"]` | `setStringList()` / `getStringList()` |

### 1.3 API cơ bản

```dart
// Tất cả operations đều async
final prefs = await SharedPreferences.getInstance();

// Write
await prefs.setString('theme', 'dark');
await prefs.setInt('fontSize', 16);
await prefs.setBool('notifications', true);

// Read — trả về null nếu key chưa tồn tại
final theme = prefs.getString('theme');          // String?
final fontSize = prefs.getInt('fontSize');        // int?
final notif = prefs.getBool('notifications');     // bool?

// Delete
await prefs.remove('theme');

// Clear all
await prefs.clear();

// Check key existence
final hasTheme = prefs.containsKey('theme');     // bool
```

### 1.4 Khi nào dùng / không dùng

✅ **Nên dùng:**
- App settings (dark mode, language, font size)
- User preferences (notification on/off)
- Simple flags (onboarding completed, first launch)
- Last selected tab, last search query

❌ **KHÔNG nên dùng:**
- Dữ liệu phức tạp (nested objects, lists of objects)
- Dữ liệu lớn (> vài KB)
- Dữ liệu nhạy cảm (token, password) → dùng Secure Storage
- Dữ liệu cần query/filter → dùng Hive hoặc Drift

### 1.5 Lưu ý quan trọng

- SharedPreferences **không mã hóa** dữ liệu — lưu dạng plain text
- Trên Android: lưu trong file XML tại `data/data/<package>/shared_prefs/`
- Trên iOS: lưu trong `NSUserDefaults`
- **Async API** — tất cả read/write đều trả về `Future`
- `getInstance()` tốn chi phí lần đầu, nên cache instance

> 🔗 **FE Bridge:** `SharedPreferences` ≈ **localStorage** — key-value string storage, persist across sessions. API khác: `prefs.getString(key)` thay vì `localStorage.getItem(key)`. Nhưng **khác ở**: SharedPreferences = async (cần `await`), localStorage = sync. SharedPreferences hỗ trợ typed getter (`getInt`, `getBool`), localStorage chỉ string.

---

## 2. Hive — NoSQL Database 🟡

### 2.1 Khái niệm

Hive là **lightweight NoSQL database** được viết hoàn toàn bằng Dart, tối ưu cho Flutter. Nó nhanh hơn SharedPreferences cho dữ liệu có cấu trúc và không cần native dependencies.

```
┌──────────────────────────────────────────┐
│                  HIVE                     │
│                                           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │  Box:    │  │  Box:    │  │  Box:    │ │
│  │  tasks   │  │  users   │  │ settings │ │
│  │         │  │         │  │         │  │
│  │ key→Task│  │ key→User│  │ key→val │  │
│  │ key→Task│  │ key→User│  │ key→val │  │
│  └─────────┘  └─────────┘  └─────────┘  │
│                                           │
│  ✦ Pure Dart — no native dependency       │
│  ✦ Fast read/write (binary format)        │
│  ✦ Encryption support                     │
│  ✦ TypeAdapter for custom objects          │
└──────────────────────────────────────────┘
```

### 2.2 Các khái niệm chính

**Box** = một collection (tương tự table/collection trong database):
- `Box` — load toàn bộ data vào memory khi mở → đọc cực nhanh
- `LazyBox` — chỉ load keys vào memory, value đọc từ disk khi cần → tiết kiệm RAM cho data lớn

**TypeAdapter** = bộ serialize/deserialize cho custom objects:
- Viết tay hoặc dùng code generation với `hive_generator`
- Mỗi class cần đăng ký adapter trước khi dùng

**Encrypted Box** = box được mã hóa bằng AES-256:

```dart
// Tạo encryption key
final encryptionKey = Hive.generateSecureKey();

// Mở encrypted box
final box = await Hive.openBox('secrets',
  encryptionCipher: HiveAesCipher(encryptionKey),
);
```

> 🔒 **Security:** Encryption key phải được lưu an toàn — KHÔNG hardcode trong source code.
> Dùng [`flutter_secure_storage`](https://pub.dev/packages/flutter_secure_storage) để lưu key trên Keychain (iOS) / Keystore (Android).
>
> ```dart
> final secureStorage = FlutterSecureStorage();
> // Tạo key lần đầu
> var key = await secureStorage.read(key: 'hive_key');
> if (key == null) {
>   final newKey = Hive.generateSecureKey();
>   await secureStorage.write(key: 'hive_key', value: base64UrlEncode(newKey));
>   key = base64UrlEncode(newKey);
> }
> final encryptionKey = base64Url.decode(key);
> await Hive.openBox('secrets', encryptionCipher: HiveAesCipher(encryptionKey));
> ```

### 2.3 Setup & API cơ bản

```dart
// Khởi tạo Hive
await Hive.initFlutter();

// Đăng ký TypeAdapter (trước khi mở box)
Hive.registerAdapter(TaskAdapter());

// Mở box
final box = await Hive.openBox<Task>('tasks');

// CRUD operations
// Create / Update
await box.put('task_1', Task(title: 'Learn Hive', done: false));
// hoặc auto-increment key
final key = await box.add(Task(title: 'Learn Hive', done: false));

// Read
final task = box.get('task_1');           // Task?
final allTasks = box.values.toList();     // List<Task>

// Update
await box.put('task_1', Task(title: 'Learn Hive', done: true));

// Delete
await box.delete('task_1');

// Watch changes (reactive)
box.watch().listen((event) {
  print('Key: ${event.key}, Deleted: ${event.deleted}');
});
```

### 2.4 TypeAdapter với code generation

```dart
// task.dart
import 'package:hive/hive.dart';

part 'task.g.dart';

@HiveType(typeId: 0)  // mỗi class cần typeId unique
class Task extends HiveObject {
  @HiveField(0)
  final String title;

  @HiveField(1)
  final bool isDone;

  @HiveField(2)
  final DateTime createdAt;

  Task({
    required this.title,
    this.isDone = false,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();
}
```

```bash
# Generate TypeAdapter
dart run build_runner build
```

### 2.5 LazyBox cho dữ liệu lớn

```dart
// LazyBox — không load tất cả values vào memory
final lazyBox = await Hive.openLazyBox<Task>('largeTasks');

// Read — async vì phải đọc từ disk
final task = await lazyBox.get('task_1');  // Future<Task?>

// Write — giống Box thường
await lazyBox.put('task_1', task);
```

### 2.6 Khi nào dùng Hive

✅ **Nên dùng:**
- Structured local data (model objects)
- Offline cache (API responses)
- Fast read/write cần thiết
- Data không cần relational queries phức tạp
- App không cần native SQLite dependency

❌ **Không nên dùng:**
- Dữ liệu cần JOIN, GROUP BY, complex queries → dùng Drift
- Dữ liệu cần ACID transactions phức tạp → dùng Drift
- Chỉ cần lưu vài settings đơn giản → dùng SharedPreferences

> ⚠️ **Hive maintenance status (2025+)**: Hive v2 (Community Edition) đang maintenance mode — ít update mới. Các alternatives:
> - **Isar** — cùng tác giả với Hive, hiệu năng tốt hơn (đang beta cho v4)
> - **ObjectBox** — NoSQL hiệu năng cao, production-ready
> - **Drift** (đã học) — nếu cần SQL
> 
> Hive vẫn hoạt động tốt cho simple key-value storage. Dùng Drift/ObjectBox cho data phức tạp.

### ObjectBox — Alternative NoSQL

ObjectBox là database NoSQL hiệu năng cao, thay thế Hive cho use cases cần query phức tạp:

```yaml
dependencies:
  objectbox: ^4.0.0
  objectbox_flutter_libs: any

dev_dependencies:
  objectbox_generator: ^4.0.0  
  build_runner: ^2.4.0
```

```dart
@Entity()
class Note {
  @Id()
  int id = 0;
  String title;
  String content;
  
  @Property(type: PropertyType.date)
  DateTime createdAt;
  
  Note({required this.title, required this.content, DateTime? createdAt})
      : createdAt = createdAt ?? DateTime.now();
}

// CRUD
final box = store.box<Note>();
box.put(Note(title: 'Hello', content: 'World'));
final notes = box.getAll();
```

> 💡 **Khi nào dùng ObjectBox?** Khi cần query phức tạp (filter, sort, relations) mà không muốn overhead của SQL. Nhanh hơn Hive cho dataset lớn (>10,000 records).

> 💼 **Gặp trong dự án:** Khi Hive schema thay đổi (thêm/xóa field), app cũ crash vì data format không match. Giải pháp: version hóa TypeAdapter, viết migration logic trong `initHive()`, và luôn có fallback khi parse thất bại — xóa box cũ và tạo mới.

> 🔗 **FE Bridge:** Hive ≈ **IndexedDB** — NoSQL, key-value & document storage. Nhưng **khác ở**: Hive = type-safe với adapters (Dart objects), IndexedDB = schemaless JSON. Hive performance tốt hơn nhiều so với IndexedDB. Alternative: `ObjectBox` ≈ Realm — binary-optimized NoSQL.

---

## 3. Drift (SQLite) — Relational Database 🟢

### 3.1 Khái niệm

Drift (tên cũ: Moor) là **type-safe SQLite wrapper** cho Dart/Flutter. Nó cho phép định nghĩa tables, queries, và migrations bằng Dart code, với compile-time safety.

```
┌──────────────────────────────────────────────┐
│                   DRIFT                       │
│                                               │
│   Dart Classes ──▶ SQL Tables                 │
│   Dart Queries ──▶ SQL Queries                │
│   Compile-time type checking                  │
│                                               │
│   ┌──────────┐    ┌──────────┐               │
│   │  Tasks    │    │  Tags    │               │
│   │──────────│    │──────────│               │
│   │ id (PK)  │───▶│ taskId   │               │
│   │ title    │    │ name     │               │
│   │ content  │    │ color    │               │
│   │ dueDate  │    └──────────┘               │
│   │ isDone   │                                │
│   └──────────┘                                │
│                                               │
│   ✦ Type-safe queries                         │
│   ✦ Auto-generated code                       │
│   ✦ Stream queries (reactive)                 │
│   ✦ Migrations support                        │
│   ✦ DAO pattern                               │
└──────────────────────────────────────────────┘
```

### 3.2 Định nghĩa Tables

```dart
// tables.dart
import 'package:drift/drift.dart';

class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get content => text().nullable()();
  DateTimeColumn get dueDate => dateTime().nullable()();
  BoolColumn get isDone => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

class Tags extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get color => integer()();
  IntColumn get taskId => integer().references(Tasks, #id)();
}
```

### 3.3 Database class

```dart
// database.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Tasks, Tags], daos: [TaskDao])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (m) => m.createAll(),
    onUpgrade: (m, from, to) async {
      // Handle schema migrations
      if (from < 2) {
        await m.addColumn(tasks, tasks.content);
      }
    },
  );
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'app.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

### 3.4 DAO Pattern

```dart
// task_dao.dart
part 'task_dao.g.dart';

@DriftAccessor(tables: [Tasks])
class TaskDao extends DatabaseAccessor<AppDatabase> with _$TaskDaoMixin {
  TaskDao(AppDatabase db) : super(db);

  // SELECT all
  Future<List<Task>> getAllTasks() => select(tasks).get();

  // SELECT with filter
  Future<List<Task>> getIncompleteTasks() {
    return (select(tasks)..where((t) => t.isDone.equals(false))).get();
  }

  // WATCH — trả về Stream, UI tự update khi data thay đổi
  Stream<List<Task>> watchAllTasks() => select(tasks).watch();

  // INSERT
  Future<int> insertTask(TasksCompanion task) {
    return into(tasks).insert(task);
  }

  // UPDATE
  Future<bool> updateTask(Task task) => update(tasks).replace(task);

  // DELETE
  Future<int> deleteTask(Task task) => delete(tasks).delete(task);

  // Complex query
  Future<List<Task>> getTasksDueBefore(DateTime date) {
    return (select(tasks)
          ..where((t) => t.dueDate.isSmallerThanValue(date))
          ..orderBy([(t) => OrderingTerm.asc(t.dueDate)]))
        .get();
  }
}
```

### 3.5 Khi nào dùng Drift

✅ **Nên dùng:**
- Dữ liệu có **quan hệ** (relational) — tasks + tags, users + posts
- Cần **complex queries**: JOIN, GROUP BY, aggregate functions
- Cần **data integrity**: foreign keys, constraints, transactions
- Cần **migrations** khi schema thay đổi
- Dữ liệu lớn cần **indexed queries**

❌ **Không nên dùng:**
- Dữ liệu đơn giản, không cần relational → dùng Hive
- Chỉ cần key-value → dùng SharedPreferences

### 3.6 So sánh Hive vs Drift

| Tiêu chí | Hive | Drift (SQLite) |
|-----------|------|----------------|
| Kiểu DB | NoSQL (key-value) | Relational (SQL) |
| Tốc độ đọc | ⚡ Rất nhanh (in-memory) | 🔵 Nhanh (indexed) |
| Tốc độ ghi | ⚡ Nhanh | 🔵 Tốt |
| Complex queries | ❌ Hạn chế | ✅ Full SQL power |
| Relations | ❌ Không hỗ trợ | ✅ Foreign keys, JOINs |
| Type safety | 🔵 Runtime | ✅ Compile-time |
| Code gen | Optional | Required |
| Migrations | Manual | Built-in |
| Encryption | ✅ Built-in | 🔵 Cần thêm package |
| Native deps | Không (pure Dart) | SQLite native lib |

### sqflite — Raw SQLite Access

`sqflite` là package truy cập SQLite trực tiếp, không cần ORM:

```yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.8.0
```

```dart
// Mở database
final db = await openDatabase(
  join(await getDatabasesPath(), 'notes.db'),
  onCreate: (db, version) {
    return db.execute(
      'CREATE TABLE notes(id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT, content TEXT)',
    );
  },
  version: 1,
);

// CRUD
await db.insert('notes', {'title': 'Hello', 'content': 'World'});
final notes = await db.query('notes');
await db.update('notes', {'title': 'Updated'}, where: 'id = ?', whereArgs: [1]);
await db.delete('notes', where: 'id = ?', whereArgs: [1]);
```

> 💡 **sqflite vs Drift**: sqflite = raw SQL, full control. Drift = type-safe ORM built on sqflite. **Dùng Drift** cho production apps, sqflite khi cần raw queries hoặc migration phức tạp.

> 🔗 **FE Bridge:** SQLite ≈ **IndexedDB** nhưng dùng SQL thay vì cursor API. `Drift` (ORM cho SQLite) ≈ Prisma/Drizzle cho FE — type-safe queries, migration support. Nhưng **khác ở**: mobile có full SQL database locally (không giới hạn 5MB như localStorage), Drift generate type-safe Dart code từ schema.

---

> 💼 **Gặp trong dự án:** Setup Hive hoặc Drift cho feature cần local persistence, migrate data khi add/remove fields, encrypt sensitive data, setup TypeAdapters cho custom objects
> 🤖 **Keywords bắt buộc trong prompt:** `Hive`, `@HiveType`, `@HiveField`, `TypeAdapter`, `Drift`, `schema migration`, `encrypt box`, `lazy box vs box`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Feature:** Todo app cần persist data offline — user tạo todos, close app, mở lại phải có data
- **Migration:** V1 có `name, email`, V2 thêm `phoneNumber, avatarUrl` — existing records phải backward compatible
- **Security:** User profile có token/sensitive fields — cần encrypted storage

**Tại sao cần các keyword trên:**
- **`Hive`** — NoSQL database cho Flutter, nhanh, không cần native dependencies
- **`@HiveType/@HiveField`** — annotations cho code generation, field number PHẢI stable (không thay đổi)
- **`TypeAdapter`** — serialize custom objects (AI hay quên register adapter)
- **`schema migration`** — Drift cần explicit migration, Hive "free" cho additive changes
- **`encrypt box`** — Hive encrypted box cho sensitive data

**Prompt mẫu — Hive Setup:**
```text
Tôi cần setup Hive cho Todo app với Flutter.
Tech stack: Flutter 3.x, hive ^2.x, hive_flutter, hive_generator.
Model TodoItem: id (String), title, description, isCompleted (bool), createdAt (DateTime), priority (enum: low/medium/high).
Requirements:
1. @HiveType + @HiveField annotations cho TodoItem.
2. PriorityEnum adapter (custom TypeAdapter hoặc @HiveType).
3. TodoLocalDataSource: addTodo, updateTodo, deleteTodo, getAllTodos, getTodosByPriority.
4. Init Hive trong main() (initFlutter, openBox, registerAdapters).
5. Migration plan: V2 sẽ thêm `dueDate` field — thiết kế field numbers để dễ migrate.
Constraints:
- DateTime serialize dùng millisecondsSinceEpoch (Hive không hỗ trợ DateTime directly).
- Field numbers: 0-based, PHẢI sequential, KHÔNG thay đổi existing numbers.
- Box name: 'todos' — PHẢI consistent toàn app.
- Register ALL adapters TRƯỚC openBox.
Output: todo_item.dart (model) + todo_local_data_source.dart + main.dart (init code).
```

**Expected Output:** AI gen 3 files Hive setup hoàn chỉnh.

⚠️ **Giới hạn AI hay mắc:** AI hay quên register adapter trước khi openBox. AI cũng hay dùng DateTime directly (Hive không hỗ trợ, phải dùng millisecondsSinceEpoch hoặc custom adapter). AI hay assign field number không sequential.

</details>

---

## 4. Caching Strategy 🟡

### 4.1 Các chiến lược caching

```
┌─────────────────────────────────────────────────────┐
│              CACHING STRATEGIES                      │
│                                                      │
│  1. Cache-First (Offline-First)                      │
│     Cache ──▶ Nếu có & fresh → return                │
│           ──▶ Nếu stale/miss → Network → Update cache│
│                                                      │
│  2. Network-First                                    │
│     Network ──▶ Thành công → Update cache → return   │
│             ──▶ Thất bại → Fallback to cache         │
│                                                      │
│  3. Stale-While-Revalidate                           │
│     Return cache ngay (dù stale)                     │
│     Đồng thời fetch network → Update cache           │
│     Next request sẽ có data mới                      │
└─────────────────────────────────────────────────────┘
```

**Chọn strategy nào?**

| Strategy | Use case | UX |
|----------|----------|----|
| Cache-First | Dữ liệu ít thay đổi (profile, settings) | Nhanh, ít loading |
| Network-First | Dữ liệu realtime (chat, feed) | Luôn mới nhất |
| Stale-While-Revalidate | Dữ liệu thay đổi vừa (product list) | Nhanh + cập nhật sau |

### 4.2 Time-Based Invalidation (TTL)

```dart
class CachedData<T> {
  final T data;
  final DateTime cachedAt;
  final Duration ttl;

  CachedData({
    required this.data,
    required this.cachedAt,
    required this.ttl,
  });

  bool get isExpired => DateTime.now().difference(cachedAt) > ttl;
  bool get isFresh => !isExpired;
}
```

### 4.3 Repository Pattern với Cache Layer

```
┌──────────┐     ┌──────────────┐     ┌────────────┐     ┌─────────┐
│    UI    │────▶│  Repository  │────▶│ Local Cache │     │   API   │
│          │◀────│              │◀────│  (Hive/DB)  │     │ Server  │
└──────────┘     │              │     └────────────┘     └─────────┘
                 │  1. Check    │            │                 │
                 │     cache    │◀───────────┘                 │
                 │  2. If stale │                              │
                 │     → fetch  │────────────────────────────▶│
                 │  3. Update   │                              │
                 │     cache    │◀─────────────────────────────│
                 │  4. Return   │                              │
                 └──────────────┘                              │
```

```dart
class TaskRepository {
  final TaskApiService _api;
  final TaskLocalDataSource _local;

  TaskRepository(this._api, this._local);

  /// Cache-first strategy
  Future<List<Task>> getTasks({bool forceRefresh = false}) async {
    // 1. Check cache
    if (!forceRefresh) {
      final cached = await _local.getCachedTasks();
      if (cached != null && cached.isFresh) {
        return cached.data;
      }
    }

    // 2. Fetch from network
    try {
      final tasks = await _api.fetchTasks();
      // 3. Update cache
      await _local.cacheTasks(tasks);
      return tasks;
    } catch (e) {
      // 4. Fallback to stale cache if network fails
      final staleCache = await _local.getCachedTasks();
      if (staleCache != null) {
        return staleCache.data;
      }
      rethrow;
    }
  }
}
```

### 4.4 Memory Cache vs Disk Cache

```
┌─────────────────────────────────────────────┐
│           CACHING LAYERS                     │
│                                              │
│  Layer 1: Memory Cache (Map/LRU)            │
│  ├── Tốc độ: ⚡⚡⚡ Cực nhanh                │
│  ├── Persist: ❌ Mất khi kill app            │
│  └── Size: Hạn chế (RAM)                    │
│                                              │
│  Layer 2: Disk Cache (Hive/SQLite/File)     │
│  ├── Tốc độ: ⚡ Nhanh                       │
│  ├── Persist: ✅ Sống sót restart            │
│  └── Size: Lớn (disk space)                 │
│                                              │
│  Layer 3: Network (API)                     │
│  ├── Tốc độ: 🐌 Chậm nhất                  │
│  ├── Persist: ✅ Server                      │
│  └── Size: Không giới hạn                   │
└─────────────────────────────────────────────┘

Request Flow:
  Memory → miss → Disk → miss → Network
                                  │
  Memory ◀── Disk ◀── ───────────┘
                       (update cả 2 layers)
```

```dart
class TwoLayerCache<T> {
  final Map<String, CachedData<T>> _memoryCache = {};
  final Box<String> _diskCache;
  final T Function(Map<String, dynamic>) _fromJson;
  final Map<String, dynamic> Function(T) _toJson;

  TwoLayerCache(this._diskCache, this._fromJson, this._toJson);

  Future<T?> get(String key, {Duration ttl = const Duration(minutes: 5)}) async {
    // Layer 1: Memory
    final memEntry = _memoryCache[key];
    if (memEntry != null && memEntry.isFresh) {
      return memEntry.data;
    }

    // Layer 2: Disk
    final diskJson = _diskCache.get(key);
    if (diskJson != null) {
      final data = _fromJson(jsonDecode(diskJson));
      // Promote to memory cache
      _memoryCache[key] = CachedData(
        data: data,
        cachedAt: DateTime.now(),
        ttl: ttl,
      );
      return data;
    }

    return null;
  }

  Future<void> put(String key, T data, {Duration ttl = const Duration(minutes: 5)}) async {
    // Write to both layers
    _memoryCache[key] = CachedData(data: data, cachedAt: DateTime.now(), ttl: ttl);
    await _diskCache.put(key, jsonEncode(_toJson(data)));
  }
}
```

---

## 5. Offline-First Pattern 🟡

### 5.1 Kiến trúc tổng quan

Offline-first = **UI luôn đọc từ local database**, không bao giờ đọc trực tiếp từ API. Sync engine chạy ngầm để đồng bộ data.

```
┌─────────────────────────────────────────────────────┐
│                OFFLINE-FIRST ARCHITECTURE           │
│                                                      │
│  ┌──────┐     ┌──────────┐     ┌──────────┐        │
│  │  UI  │────▶│ Local DB │     │  Sync    │        │
│  │      │◀────│ (Drift)  │◀───▶│  Engine  │        │
│  └──────┘     └──────────┘     └────┬─────┘        │
│                                      │               │
│                                      │ background    │
│                                      ▼               │
│                               ┌──────────┐          │
│                               │   API    │          │
│                               │  Server  │          │
│                               └──────────┘          │
│                                                      │
│  Flow:                                               │
│  1. UI reads from Local DB (always fast)             │
│  2. UI writes to Local DB + adds to sync queue       │
│  3. Sync Engine pushes pending changes to API        │
│  4. Sync Engine pulls new data from API → Local DB   │
│  5. UI reactively updates via Stream                 │
└─────────────────────────────────────────────────────┘
```

### 5.2 Sync Engine

```dart
class SyncEngine {
  final ApiService _api;
  final LocalDatabase _db;
  final ConnectivityService _connectivity;
  Timer? _syncTimer;

  SyncEngine(this._api, this._db, this._connectivity);

  void startPeriodicSync({Duration interval = const Duration(minutes: 5)}) {
    _syncTimer = Timer.periodic(interval, (_) => sync());

    // Sync ngay khi có internet trở lại
    _connectivity.onConnectivityChanged.listen((status) {
      if (status != ConnectivityResult.none) {
        sync();
      }
    });
  }

  Future<void> sync() async {
    if (!await _connectivity.hasConnection) return;

    // 1. Push: gửi pending changes lên server
    await _pushPendingChanges();

    // 2. Pull: lấy data mới từ server
    await _pullLatestData();
  }

  Future<void> _pushPendingChanges() async {
    final pendingOps = await _db.getPendingOperations();
    for (final op in pendingOps) {
      try {
        await _api.execute(op);
        await _db.markSynced(op.id);
      } catch (e) {
        // Retry later — không xóa khỏi queue
        break;
      }
    }
  }

  Future<void> _pullLatestData() async {
    final lastSync = await _db.getLastSyncTimestamp();
    final newData = await _api.fetchChangesSince(lastSync);
    await _db.applyRemoteChanges(newData);
  }

  void dispose() {
    _syncTimer?.cancel();
  }
}
```

### 5.3 Pending Operations Queue

```dart
// Lưu các operation chưa sync vào local DB
class PendingOperations extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get type => text()();        // 'create', 'update', 'delete'
  TextColumn get entity => text()();      // 'task', 'note'
  TextColumn get payload => text()();     // JSON data
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  BoolColumn get isSynced => boolean().withDefault(const Constant(false))();
}
```

```dart
// Khi user tạo note mới (offline hoặc online)
Future<void> createNote(Note note) async {
  // 1. Lưu vào local DB ngay
  await _db.insertNote(note);

  // 2. Thêm vào sync queue
  await _db.addPendingOperation(
    PendingOperation(
      type: 'create',
      entity: 'note',
      payload: jsonEncode(note.toJson()),
    ),
  );

  // 3. Trigger sync nếu có internet
  _syncEngine.trySyncNow();
}
```

### 5.4 Conflict Resolution

Khi cùng một record bị thay đổi ở cả client và server:

```
┌──────────────────────────────────────────────┐
│         CONFLICT RESOLUTION STRATEGIES       │
│                                              │
│  1. Last-Write-Wins (LWW)                   │
│     → Dùng timestamp, bản mới nhất thắng    │
│     → Đơn giản, có thể mất data             │
│                                              │
│  2. Server-Wins                              │
│     → Server data luôn ưu tiên              │
│     → An toàn, nhưng mất local changes      │
│                                              │
│  3. Client-Wins                              │
│     → Client data ưu tiên                   │
│     → User control, có thể ghi đè data khác │
│                                              │
│  4. Manual Merge                             │
│     → Hiển thị cho user chọn               │
│     → Best UX, phức tạp nhất                │
└──────────────────────────────────────────────┘
```

### 5.5 Connectivity Detection

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class ConnectivityService {
  final Connectivity _connectivity = Connectivity();

  /// Check hiện tại có internet không
  Future<bool> get hasConnection async {
    final result = await _connectivity.checkConnectivity();
    return !result.contains(ConnectivityResult.none);
  }

  /// Stream theo dõi thay đổi connectivity
  Stream<bool> get onConnectivityChanged {
    return _connectivity.onConnectivityChanged.map(
      (results) => !results.contains(ConnectivityResult.none),
    );
  }
}
```

> ⚠️ **Lưu ý:** `connectivity_plus` chỉ kiểm tra kết nối mạng, **không đảm bảo có internet thật**. Để chắc chắn, nên thêm bước ping server hoặc check DNS.

> 🔗 **FE Bridge:** Caching strategy **tương đồng** FE: stale-while-revalidate, TTL, cache-first vs network-first. Nhưng **khác ở**: FE dùng Service Worker / Cache API (browser-managed), Flutter phải **tự implement** cache layer hoặc dùng package như `cached_network_image`, `dio_cache_interceptor`.

---

> 💼 **Gặp trong dự án:** Implement cache-first strategy cho API data, TTL (time-to-live) cho cached data, offline queue cho write operations, sync conflict resolution khi back online
> 🤖 **Keywords bắt buộc trong prompt:** `cache-first strategy`, `TTL`, `stale-while-revalidate`, `offline queue`, `sync conflict resolution`, `Repository pattern with cache`, `connectivity check`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Offline UX:** PM yêu cầu app hoạt động offline — data cũ vẫn hiển thị, new data queue chờ sync
- **Performance:** API chậm 2-3s — cần show cached data ngay + background refresh
- **Conflict:** User sửa offline, backend cũng sửa → cần merge strategy

**Tại sao cần các keyword trên:**
- **`cache-first strategy`** — show cache ngay → fetch → update cache → notify UI
- **`TTL`** — cached data hết hạn sau N minutes, force refetch
- **`stale-while-revalidate`** — display stale data while fetching fresh (best UX)
- **`offline queue`** — queue write operations (POST/PUT/DELETE) khi offline, replay khi online
- **`sync conflict resolution`** — last-write-wins vs merge vs user-decides

**Prompt mẫu — Caching Layer:**
```text
Tôi cần implement caching layer cho Flutter app theo Repository pattern.
Tech stack: Flutter 3.x, hive ^2.x, dio ^5.x, connectivity_plus.
Feature: Notes (CRUD) — cần cache-first strategy.
Requirements:
1. NoteRepository with cache-first strategy:
   - Read: Return cache immediately → fetch API in background → update cache → notify stream.
   - Write online: API first → cache → return success.
   - Write offline: Queue in Hive → show as "pending" → sync when online.
2. Cache TTL: 5 minutes — stale data triggers background refresh.
3. CacheManager: isExpired(key), setWithTTL(key, value, duration), get(key).
4. SyncManager: queue offline writes, replay when connectivity restored, handle conflicts (last-write-wins).
5. ConnectivityService: stream of online/offline status.
Constraints:
- Stream-based: Repository returns Stream<List<Note>> (not Future).
- TTL stored as metadata alongside cached data.
- Offline queue: max 50 items, oldest dropped if exceeded.
- Conflict: server timestamp wins (last-write-wins).
Output: note_repository.dart, cache_manager.dart, sync_manager.dart, connectivity_service.dart.
```

**Expected Output:** AI gen 4 files caching layer hoàn chỉnh.

⚠️ **Giới hạn AI hay mắc:** AI hay implement cache-first nhưng quên TTL (cache never expires). AI cũng hay quên handle case "sync queue full" và "duplicate sync items". AI hay dùng Future thay vì Stream cho cache-first (mất reactivity).

</details>

---

## 6. Secure Storage 🔴

### 6.1 Khi nào cần Secure Storage

- **Auth tokens** (access token, refresh token)
- **API keys** nhạy cảm
- **User credentials** (nếu cần lưu)
- **Encryption keys**
- Bất kỳ dữ liệu nào cần **bảo mật cao**

### 6.2 flutter_secure_storage

Package này sử dụng:
- **iOS**: Keychain Services
- **Android**: EncryptedSharedPreferences (API 23+) hoặc Keystore

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

final storage = FlutterSecureStorage();

// Write
await storage.write(key: 'auth_token', value: 'eyJhbGciOiJIUzI1NiIs...');
await storage.write(key: 'refresh_token', value: 'dGhpcyBpcyBhIHJl...');

// Read
final token = await storage.read(key: 'auth_token');  // String?

// Delete
await storage.delete(key: 'auth_token');

// Delete all
await storage.deleteAll();

// Check existence
final hasToken = await storage.containsKey(key: 'auth_token');

// Read all
final allValues = await storage.readAll();  // Map<String, String>
```

### 6.3 So sánh SharedPreferences vs Secure Storage

| | SharedPreferences | FlutterSecureStorage |
|---|---|---|
| Mã hóa | ❌ Plain text | ✅ Encrypted |
| Tốc độ | ⚡ Nhanh | 🔵 Chậm hơn |
| Use case | Settings, preferences | Tokens, credentials |
| Platform | NSUserDefaults / SharedPrefs | Keychain / Keystore |

---

## 7. Best Practices & Lỗi thường gặp 🟡

### 7.1 Best Practices

```
✅ DO:
├── Chọn đúng storage cho đúng use case
│   ├── Simple settings → SharedPreferences
│   ├── Structured data, fast R/W → Hive
│   ├── Relational, complex queries → Drift
│   └── Sensitive data → flutter_secure_storage
│
├── Wrap storage bằng Repository pattern
│   └── Dễ swap implementation, dễ test
│
├── Handle migration khi schema thay đổi
│   ├── Hive: typeId và fieldId không đổi
│   └── Drift: Migration strategy
│
├── Cache strategy phù hợp với data type
│   ├── Static data → Cache-first, TTL dài
│   └── Dynamic data → Network-first hoặc stale-while-revalidate
│
└── Luôn handle trường hợp data null/missing
    └── Default values cho SharedPreferences
```

### 7.2 Lỗi thường gặp

```
❌ DON'T:
├── Lưu token trong SharedPreferences
│   └── FIX: Dùng flutter_secure_storage
│
├── Quên đăng ký Hive TypeAdapter trước khi openBox
│   └── FIX: registerAdapter() trong main() trước openBox()
│
├── Thay đổi Hive typeId hoặc fieldId
│   └── FIX: Chỉ thêm field mới, không đổi/xóa field cũ
│
├── Quên close Hive boxes
│   └── FIX: await Hive.close() trong app lifecycle
│
├── Gọi SharedPreferences.getInstance() nhiều lần
│   └── FIX: Cache instance hoặc dùng DI
│
├── Không handle migration khi thêm column Drift
│   └── FIX: Tăng schemaVersion + viết onUpgrade
│
├── Chạy heavy DB operations trên main thread
│   └── FIX: Drift.createInBackground() hoặc compute()
│
└── Assume connectivity_plus = có internet thật
    └── FIX: Thêm actual HTTP check
```

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

Nếu bạn đến từ React/Vue, đây là bảng mapping:

### 8.1 Storage Mapping

| Flutter | Web Equivalent | Ghi chú |
|---------|---------------|---------|
| `SharedPreferences` | `localStorage` | Key-value, sync bên web nhưng async bên Flutter |
| `Hive` | `IndexedDB` | NoSQL, structured data, nhưng Hive API đơn giản hơn nhiều |
| `Drift (SQLite)` | `WebSQL` (deprecated) / `better-sqlite3` | Relational, Drift type-safe hơn |
| `flutter_secure_storage` | `HttpOnly Cookies` / `Web Crypto API` | Web không có Keychain equivalent trực tiếp |

### 8.2 Offline-First Mapping

| Flutter | Web (PWA) | Ghi chú |
|---------|-----------|---------|
| Connectivity detection | `navigator.onLine` + Service Worker | Flutter dùng `connectivity_plus` |
| Sync Engine | Background Sync API | Flutter linh hoạt hơn, kiểm soát tốt hơn |
| Local DB (Drift/Hive) | IndexedDB + Cache API | Flutter có SQLite native, mạnh hơn |
| Pending operation queue | SWR / React Query offline | Tự build hoặc dùng packages |

### 8.3 Khác biệt chính cần chú ý

```
Web (React/Vue)                    Flutter
────────────────                   ──────────────────
localStorage sync API         →   SharedPreferences async API
IndexedDB phức tạp            →   Hive đơn giản hơn nhiều  
Service Worker caching        →   Repository pattern + Hive/Drift
5MB localStorage limit        →   Gần như không giới hạn (disk)
CORS restrictions             →   Không có CORS issues
Browser manages storage       →   App manages own storage
```

> 💡 **Key insight:** Flutter apps có **nhiều quyền kiểm soát hơn** over local storage so với web apps. Bạn có full access đến file system, SQLite, và Keychain/Keystore — những thứ mà web browser giới hạn rất nhiều.

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|---------------|
| 1 | `localStorage` = sync, simple | SharedPreferences = **async**, typed getters | Mobile storage API is async by design |
| 2 | IndexedDB = complex, rarely used | SQLite/Drift = **powerful, commonly used** — full SQL | Mobile có embedded database, web thì IndexedDB limited |
| 3 | Service Worker handles caching | **Tự implement** cache layer — không có built-in Service Worker | Mobile app = no browser cache layer |

---

## 9. Tổng kết

### Checklist kiến thức buổi 12

```
□ Hiểu 4 giải pháp storage: SharedPreferences, Hive, Drift, Secure Storage
□ Biết khi nào dùng giải pháp nào
□ Implement CRUD với Hive (TypeAdapter, Box, LazyBox)
□ Implement CRUD với Drift (Tables, DAO, Queries, Streams)
□ Hiểu 3 caching strategies: cache-first, network-first, stale-while-revalidate
□ Implement Repository pattern với cache layer
□ Hiểu offline-first architecture: local DB + sync engine
□ Implement pending operations queue
□ Dùng connectivity_plus để detect online/offline
□ Dùng flutter_secure_storage cho sensitive data
□ Hiểu conflict resolution strategies
□ Biết handle schema migration (Hive & Drift)
```

### Decision Tree — Chọn Storage nào?

```
Dữ liệu cần lưu là gì?
│
├── Settings/preferences đơn giản?
│   └── ✅ SharedPreferences
│
├── Tokens/passwords/sensitive data?
│   └── ✅ flutter_secure_storage
│
├── Structured objects, không cần relational queries?
│   └── ✅ Hive
│
├── Relational data, cần JOINs/complex queries?
│   └── ✅ Drift (SQLite)
│
└── Large files (images, documents)?
    └── ✅ File system (path_provider)
```

---

### ➡️ Buổi tiếp theo

> **Buổi 13: Performance Optimization** — Rebuild optimization, DevTools profiling, memory management, và Isolates cho heavy computation.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Cài Flutter DevTools extension trong VS Code
