# Buổi 12: Local Storage & Data Persistence — Ví dụ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

1. [VD1: SharedPreferences — Theme & Language Settings](#vd1-sharedpreferences--theme--language-settings)
2. [VD2: Hive — CRUD Task Model](#vd2-hive--crud-task-model)
3. [VD3: Drift — Tasks Table với DAO](#vd3-drift--tasks-table-với-dao)
4. [VD4: Cache Layer — Repository Pattern](#vd4-cache-layer--repository-pattern)
5. [VD5: Secure Storage — Auth Token](#vd5-secure-storage--auth-token)

---

## VD1: SharedPreferences — Theme & Language Settings 🟢

> **Liên quan tới:** [1. SharedPreferences — Key-Value Storage](01-ly-thuyet.md#1-sharedpreferences--key-value-storage)

### Mục đích

Lưu và tải lại theme mode (light/dark) và language preference bằng SharedPreferences. Khi app restart, settings vẫn giữ nguyên.

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  shared_preferences: ^2.2.3
```

### settings_service.dart

```dart
import 'package:shared_preferences/shared_preferences.dart';

class SettingsService {
  static const _keyThemeMode = 'theme_mode';
  static const _keyLanguage = 'language';
  static const _keyFontSize = 'font_size';

  final SharedPreferences _prefs;

  // Inject SharedPreferences instance (testable)
  SettingsService(this._prefs);

  // --- Theme Mode ---
  /// Trả về 'light', 'dark', hoặc 'system' (default)
  String get themeMode => _prefs.getString(_keyThemeMode) ?? 'system';

  Future<void> setThemeMode(String mode) async {
    await _prefs.setString(_keyThemeMode, mode);
  }

  // --- Language ---
  /// Trả về 'vi', 'en', hoặc 'vi' (default)
  String get language => _prefs.getString(_keyLanguage) ?? 'vi';

  Future<void> setLanguage(String lang) async {
    await _prefs.setString(_keyLanguage, lang);
  }

  // --- Font Size ---
  double get fontSize => _prefs.getDouble(_keyFontSize) ?? 14.0;

  Future<void> setFontSize(double size) async {
    await _prefs.setDouble(_keyFontSize, size);
  }
}
```

### settings_screen.dart

```dart
import 'package:flutter/material.dart';
import 'settings_service.dart';
import 'package:shared_preferences/shared_preferences.dart';

class SettingsScreen extends StatefulWidget {
  const SettingsScreen({super.key});

  @override
  State<SettingsScreen> createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  late SettingsService _settings;
  bool _isLoaded = false;

  @override
  void initState() {
    super.initState();
    _initSettings();
  }

  Future<void> _initSettings() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _settings = SettingsService(prefs);
      _isLoaded = true;
    });
  }

  @override
  Widget build(BuildContext context) {
    if (!_isLoaded) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    return Scaffold(
      appBar: AppBar(title: const Text('Settings')),
      body: ListView(
        children: [
          // Theme Mode
          ListTile(
            title: const Text('Theme Mode'),
            subtitle: Text(_settings.themeMode),
            trailing: DropdownButton<String>(
              value: _settings.themeMode,
              items: const [
                DropdownMenuItem(value: 'light', child: Text('Light')),
                DropdownMenuItem(value: 'dark', child: Text('Dark')),
                DropdownMenuItem(value: 'system', child: Text('System')),
              ],
              onChanged: (value) async {
                if (value != null) {
                  await _settings.setThemeMode(value);
                  setState(() {});
                }
              },
            ),
          ),

          // Language
          ListTile(
            title: const Text('Language'),
            subtitle: Text(_settings.language == 'vi' ? 'Tiếng Việt' : 'English'),
            trailing: DropdownButton<String>(
              value: _settings.language,
              items: const [
                DropdownMenuItem(value: 'vi', child: Text('🇻🇳 Tiếng Việt')),
                DropdownMenuItem(value: 'en', child: Text('🇺🇸 English')),
              ],
              onChanged: (value) async {
                if (value != null) {
                  await _settings.setLanguage(value);
                  setState(() {});
                }
              },
            ),
          ),

          // Font Size
          ListTile(
            title: const Text('Font Size'),
            subtitle: Text('${_settings.fontSize.toInt()}'),
          ),
          Slider(
            min: 10,
            max: 24,
            divisions: 14,
            value: _settings.fontSize,
            label: '${_settings.fontSize.toInt()}',
            onChanged: (value) async {
              await _settings.setFontSize(value);
              setState(() {});
            },
          ),
        ],
      ),
    );
  }
}
```

### Giải thích flow

```
1. App khởi động → _initSettings() → SharedPreferences.getInstance()
2. SettingsService đọc saved values (hoặc dùng default nếu chưa có)
3. User thay đổi setting → setThemeMode/setLanguage/setFontSize → ghi async
4. Restart app → _initSettings() đọc lại values đã lưu → UI hiển thị đúng
```

### ▶️ Chạy ví dụ

```bash
flutter create vidu_shared_prefs
cd vidu_shared_prefs
flutter pub add shared_preferences
# Thay nội dung lib/ bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Trang Settings hiển thị dropdown Theme (Light/Dark/System)
✅ Dropdown Language (Tiếng Việt / English)
✅ Slider Font Size (10–24)
✅ Thay đổi setting → restart app → settings vẫn giữ nguyên giá trị đã chọn
```

- 🔗 **FE tương đương:** `prefs.setString('key', value)` ≈ `localStorage.setItem('key', value)` — nhưng SharedPreferences là async và hỗ trợ typed getter (`getInt`, `getBool`).

---

## VD2: Hive — CRUD Task Model 🟢

> **Liên quan tới:** [2. Hive — NoSQL Database](01-ly-thuyet.md#2-hive--nosql-database)

### Mục đích

CRUD operations trên Task model sử dụng Hive với TypeAdapter (code generation).

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  hive: ^2.2.3
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.1
  build_runner: ^2.4.0
```

### task_model.dart

```dart
import 'package:hive/hive.dart';

part 'task_model.g.dart';

@HiveType(typeId: 0)
class TaskModel extends HiveObject {
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

  @HiveField(5)
  DateTime? updatedAt;

  TaskModel({
    required this.id,
    required this.title,
    this.description,
    this.isDone = false,
    DateTime? createdAt,
    this.updatedAt,
  }) : createdAt = createdAt ?? DateTime.now();

  /// Tạo bản copy với các field mới
  TaskModel copyWith({
    String? title,
    String? description,
    bool? isDone,
  }) {
    return TaskModel(
      id: id,
      title: title ?? this.title,
      description: description ?? this.description,
      isDone: isDone ?? this.isDone,
      createdAt: createdAt,
      updatedAt: DateTime.now(),
    );
  }
}
```

### task_repository.dart (Hive)

```dart
import 'package:hive/hive.dart';
import 'task_model.dart';

class HiveTaskRepository {
  static const String _boxName = 'tasks';

  /// Mở box — gọi 1 lần khi khởi tạo
  Future<Box<TaskModel>> _openBox() async {
    if (Hive.isBoxOpen(_boxName)) {
      return Hive.box<TaskModel>(_boxName);
    }
    return await Hive.openBox<TaskModel>(_boxName);
  }

  // --- CREATE ---
  Future<void> addTask(TaskModel task) async {
    final box = await _openBox();
    await box.put(task.id, task);  // dùng id làm key
  }

  // --- READ ---
  Future<List<TaskModel>> getAllTasks() async {
    final box = await _openBox();
    return box.values.toList();
  }

  Future<TaskModel?> getTaskById(String id) async {
    final box = await _openBox();
    return box.get(id);
  }

  Future<List<TaskModel>> getIncompleteTasks() async {
    final box = await _openBox();
    return box.values.where((task) => !task.isDone).toList();
  }

  // --- UPDATE ---
  Future<void> updateTask(TaskModel task) async {
    final box = await _openBox();
    final updated = task.copyWith(
      title: task.title,
      description: task.description,
      isDone: task.isDone,
    );
    await box.put(task.id, updated);
  }

  Future<void> toggleTaskDone(String id) async {
    final box = await _openBox();
    final task = box.get(id);
    if (task != null) {
      await box.put(id, task.copyWith(isDone: !task.isDone));
    }
  }

  // --- DELETE ---
  Future<void> deleteTask(String id) async {
    final box = await _openBox();
    await box.delete(id);
  }

  Future<void> deleteAllCompleted() async {
    final box = await _openBox();
    final completedKeys = box.values
        .where((task) => task.isDone)
        .map((task) => task.id)
        .toList();
    await box.deleteAll(completedKeys);
  }

  // --- WATCH (reactive) ---
  Stream<BoxEvent> watchTasks() async* {
    final box = await _openBox();
    yield* box.watch();
  }
}
```

### main.dart — Khởi tạo Hive

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'task_model.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 1. Khởi tạo Hive
  await Hive.initFlutter();

  // 2. Đăng ký adapter (TRƯỚC khi mở box)
  Hive.registerAdapter(TaskModelAdapter());

  runApp(const MyApp());
}
```

### task_list_screen.dart — UI sử dụng Hive

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'task_model.dart';
import 'task_repository.dart';
import 'package:uuid/uuid.dart';

class TaskListScreen extends StatefulWidget {
  const TaskListScreen({super.key});

  @override
  State<TaskListScreen> createState() => _TaskListScreenState();
}

class _TaskListScreenState extends State<TaskListScreen> {
  final _repo = HiveTaskRepository();
  final _titleController = TextEditingController();

  @override
  void dispose() {
    _titleController.dispose();
    super.dispose();
  }

  Future<void> _addTask() async {
    final title = _titleController.text.trim();
    if (title.isEmpty) return;

    final task = TaskModel(
      id: const Uuid().v4(),
      title: title,
    );
    await _repo.addTask(task);
    _titleController.clear();
    setState(() {}); // refresh UI
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tasks (Hive)')),
      body: Column(
        children: [
          // Input row
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _titleController,
                    decoration: const InputDecoration(hintText: 'New task...'),
                    onSubmitted: (_) => _addTask(),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.add),
                  onPressed: _addTask,
                ),
              ],
            ),
          ),

          // Task list
          Expanded(
            child: FutureBuilder<List<TaskModel>>(
              future: _repo.getAllTasks(),
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return const Center(child: CircularProgressIndicator());
                }
                final tasks = snapshot.data!;
                if (tasks.isEmpty) {
                  return const Center(child: Text('No tasks yet'));
                }
                return ListView.builder(
                  itemCount: tasks.length,
                  itemBuilder: (context, index) {
                    final task = tasks[index];
                    return ListTile(
                      leading: Checkbox(
                        value: task.isDone,
                        onChanged: (_) async {
                          await _repo.toggleTaskDone(task.id);
                          setState(() {});
                        },
                      ),
                      title: Text(
                        task.title,
                        style: TextStyle(
                          decoration: task.isDone
                              ? TextDecoration.lineThrough
                              : null,
                        ),
                      ),
                      subtitle: task.description != null
                          ? Text(task.description!)
                          : null,
                      trailing: IconButton(
                        icon: const Icon(Icons.delete, color: Colors.red),
                        onPressed: () async {
                          await _repo.deleteTask(task.id);
                          setState(() {});
                        },
                      ),
                    );
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### Điểm chú ý

```
✦ TaskModel extends HiveObject → có sẵn save(), delete() methods
✦ @HiveType(typeId: 0) — typeId phải unique giữa các model
✦ @HiveField(0) — fieldId phải unique trong cùng class, KHÔNG BAO GIỜ thay đổi
✦ registerAdapter() PHẢI gọi trước openBox()
✦ box.put(key, value) — dùng id làm key để dễ lookup
✦ box.watch() — trả về Stream<BoxEvent> cho reactive UI
```

### ▶️ Chạy ví dụ

```bash
flutter create vidu_hive_crud
cd vidu_hive_crud
flutter pub add hive hive_flutter uuid
flutter pub add dev:hive_generator dev:build_runner
# Tạo TaskModel với @HiveType annotations, rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Trang "Tasks (Hive)" với input field và nút thêm task
✅ Nhập tên task → nhấn (+) → task xuất hiện trong danh sách
✅ Checkbox toggle done/undone (gạch ngang title khi done)
✅ Nút xóa (🗑) xóa task khỏi danh sách
✅ Restart app → tasks vẫn còn (persisted trong Hive box)
```

---

## VD3: Drift — Tasks Table với DAO 🟢

> **Liên quan tới:** [3. Drift (SQLite) — Relational Database](01-ly-thuyet.md#3-drift-sqlite--relational-database)

### Mục đích

Định nghĩa tasks table, DAO với đầy đủ queries (insert, select, update, delete, watch) bằng Drift.

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  drift: ^2.14.0
  sqlite3_flutter_libs: ^0.5.18
  path_provider: ^2.1.0
  path: ^1.8.3

dev_dependencies:
  drift_dev: ^2.14.0
  build_runner: ^2.4.0
```

### database/tables.dart

```dart
import 'package:drift/drift.dart';

/// Tasks table — Drift tự sinh code tạo SQL CREATE TABLE
class Tasks extends Table {
  // Primary key auto-increment
  IntColumn get id => integer().autoIncrement()();

  // Title: required, max 200 chars
  TextColumn get title => text().withLength(min: 1, max: 200)();

  // Content: optional
  TextColumn get content => text().nullable()();

  // Category: optional, default = 'general'
  TextColumn get category =>
      text().withDefault(const Constant('general')).nullable()();

  // Priority: 0=low, 1=medium, 2=high
  IntColumn get priority => integer().withDefault(const Constant(1))();

  // Status
  BoolColumn get isDone => boolean().withDefault(const Constant(false))();

  // Due date: optional
  DateTimeColumn get dueDate => dateTime().nullable()();

  // Timestamps
  DateTimeColumn get createdAt =>
      dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}
```

### database/task_dao.dart

```dart
import 'package:drift/drift.dart';
import 'app_database.dart';
import 'tables.dart';

part 'task_dao.g.dart';

@DriftAccessor(tables: [Tasks])
class TaskDao extends DatabaseAccessor<AppDatabase> with _$TaskDaoMixin {
  TaskDao(AppDatabase db) : super(db);

  // ============ SELECT ============

  /// Lấy tất cả tasks, sắp xếp theo createdAt mới nhất
  Future<List<Task>> getAllTasks() {
    return (select(tasks)
          ..orderBy([(t) => OrderingTerm.desc(t.createdAt)]))
        .get();
  }

  /// Lấy tasks theo category
  Future<List<Task>> getTasksByCategory(String category) {
    return (select(tasks)..where((t) => t.category.equals(category))).get();
  }

  /// Lấy tasks chưa hoàn thành
  Future<List<Task>> getIncompleteTasks() {
    return (select(tasks)
          ..where((t) => t.isDone.equals(false))
          ..orderBy([
            (t) => OrderingTerm.desc(t.priority),
            (t) => OrderingTerm.asc(t.dueDate),
          ]))
        .get();
  }

  /// Lấy task theo id
  Future<Task?> getTaskById(int id) {
    return (select(tasks)..where((t) => t.id.equals(id))).getSingleOrNull();
  }

  /// Đếm tasks theo status
  Future<int> countTasks({bool? isDone}) {
    final query = selectOnly(tasks)..addColumns([tasks.id.count()]);
    if (isDone != null) {
      query.where(tasks.isDone.equals(isDone));
    }
    return query
        .map((row) => row.read(tasks.id.count())!)
        .getSingle();
  }

  // ============ WATCH (Stream) ============

  /// Watch tất cả tasks — Stream tự emit khi data thay đổi
  Stream<List<Task>> watchAllTasks() {
    return (select(tasks)
          ..orderBy([(t) => OrderingTerm.desc(t.createdAt)]))
        .watch();
  }

  /// Watch tasks chưa hoàn thành
  Stream<List<Task>> watchIncompleteTasks() {
    return (select(tasks)
          ..where((t) => t.isDone.equals(false))
          ..orderBy([(t) => OrderingTerm.desc(t.priority)]))
        .watch();
  }

  // ============ INSERT ============

  /// Thêm task mới, trả về id
  Future<int> insertTask(TasksCompanion task) {
    return into(tasks).insert(task);
  }

  /// Thêm nhiều tasks cùng lúc
  Future<void> insertMultipleTasks(List<TasksCompanion> taskList) async {
    await batch((batch) {
      batch.insertAll(tasks, taskList);
    });
  }

  // ============ UPDATE ============

  /// Update toàn bộ task
  Future<bool> updateTask(Task task) {
    return update(tasks).replace(task);
  }

  /// Toggle isDone
  Future<void> toggleDone(int taskId) {
    return (update(tasks)..where((t) => t.id.equals(taskId))).write(
      TasksCompanion.custom(
        isDone: tasks.isDone.not(),
        updatedAt: Variable(DateTime.now()),
      ),
    );
  }

  // ============ DELETE ============

  /// Xóa task theo id
  Future<int> deleteTaskById(int id) {
    return (delete(tasks)..where((t) => t.id.equals(id))).go();
  }

  /// Xóa tất cả tasks đã hoàn thành
  Future<int> deleteCompletedTasks() {
    return (delete(tasks)..where((t) => t.isDone.equals(true))).go();
  }
}
```

### database/app_database.dart

```dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;
import 'tables.dart';
import 'task_dao.dart';

part 'app_database.g.dart';

@DriftDatabase(tables: [Tasks], daos: [TaskDao])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  // Tăng version khi schema thay đổi
  @override
  int get schemaVersion => 1;

  // Migration strategy
  @override
  MigrationStrategy get migration => MigrationStrategy(
        onCreate: (m) => m.createAll(),
        onUpgrade: (m, from, to) async {
          // Ví dụ: thêm column 'category' ở version 2
          // if (from < 2) {
          //   await m.addColumn(tasks, tasks.category);
          // }
        },
      );
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'tasks.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

### Sử dụng trong UI với StreamBuilder

```dart
class DriftTaskScreen extends StatelessWidget {
  final AppDatabase database;

  const DriftTaskScreen({super.key, required this.database});

  @override
  Widget build(BuildContext context) {
    final taskDao = database.taskDao;

    return Scaffold(
      appBar: AppBar(title: const Text('Tasks (Drift)')),
      body: StreamBuilder<List<Task>>(
        // watchAllTasks() tự emit mỗi khi data thay đổi
        stream: taskDao.watchAllTasks(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          final tasks = snapshot.data ?? [];
          if (tasks.isEmpty) {
            return const Center(child: Text('No tasks'));
          }

          return ListView.builder(
            itemCount: tasks.length,
            itemBuilder: (context, index) {
              final task = tasks[index];
              return ListTile(
                leading: Checkbox(
                  value: task.isDone,
                  onChanged: (_) => taskDao.toggleDone(task.id),
                ),
                title: Text(task.title),
                subtitle: task.content != null ? Text(task.content!) : null,
                trailing: IconButton(
                  icon: const Icon(Icons.delete),
                  onPressed: () => taskDao.deleteTaskById(task.id),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          await taskDao.insertTask(
            TasksCompanion.insert(
              title: 'New Task ${DateTime.now().millisecondsSinceEpoch}',
              content: Value('Description here'),
            ),
          );
          // Không cần setState! StreamBuilder tự update
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Giải thích key concepts

```
✦ Tasks extends Table → Drift generate SQL CREATE TABLE
✦ TasksCompanion → Drift-generated class cho INSERT (xử lý nullable/default)
✦ Task → Drift-generated data class cho SELECT results
✦ .watch() → trả về Stream, tự emit khi bất kỳ write operation nào chạy
✦ batch() → multiple operations trong 1 transaction
✦ NativeDatabase.createInBackground() → chạy DB trên isolate riêng
✦ LazyDatabase → chỉ mở connection khi cần
```

### ▶️ Chạy ví dụ

```bash
flutter create vidu_drift
cd vidu_drift
flutter pub add drift sqlite3_flutter_libs path_provider path
flutter pub add dev:drift_dev dev:build_runner
# Tạo tables.dart, task_dao.dart, app_database.dart, rồi:
dart run build_runner build --delete-conflicting-outputs
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Trang "Tasks (Drift)" với danh sách tasks từ SQLite
✅ Nhấn FAB (+) → thêm task mới (auto-increment ID)
✅ Checkbox toggle done → StreamBuilder tự update UI (không cần setState)
✅ Nút xóa hoạt động — task biến mất khỏi danh sách
✅ Restart app → tasks vẫn còn (SQLite persistent)
```

- 🔗 **FE tương đương:** Drift ORM ≈ Prisma/Drizzle — type-safe database queries từ schema definition. Khác biệt: mobile dùng embedded SQLite (full SQL), FE thường dùng IndexedDB (NoSQL, cursor-based).

---

## VD4: Cache Layer — Repository Pattern 🟡

### Mục đích

Implement Repository với cache-first strategy: check cache → if stale → fetch API → update cache → return data.

> **Liên quan tới:** [4. Caching Strategy](01-ly-thuyet.md#4-caching-strategy)

### models/article.dart

```dart
class Article {
  final int id;
  final String title;
  final String body;

  Article({required this.id, required this.title, required this.body});

  factory Article.fromJson(Map<String, dynamic> json) => Article(
        id: json['id'] as int,
        title: json['title'] as String,
        body: json['body'] as String,
      );

  Map<String, dynamic> toJson() => {
        'id': id,
        'title': title,
        'body': body,
      };
}
```

### data/cache/cached_response.dart

```dart
import 'dart:convert';

class CachedResponse {
  final String data;       // JSON string
  final DateTime cachedAt;
  final Duration ttl;

  CachedResponse({
    required this.data,
    required this.cachedAt,
    this.ttl = const Duration(minutes: 10),
  });

  bool get isExpired => DateTime.now().difference(cachedAt) > ttl;
  bool get isFresh => !isExpired;

  // Serialize cho Hive storage
  Map<String, dynamic> toJson() => {
        'data': data,
        'cachedAt': cachedAt.toIso8601String(),
        'ttlSeconds': ttl.inSeconds,
      };

  factory CachedResponse.fromJson(Map<String, dynamic> json) => CachedResponse(
        data: json['data'] as String,
        cachedAt: DateTime.parse(json['cachedAt'] as String),
        ttl: Duration(seconds: json['ttlSeconds'] as int),
      );
}
```

### data/remote/article_api_service.dart

```dart
import 'package:dio/dio.dart';
import '../../models/article.dart';

class ArticleApiService {
  final Dio _dio;

  ArticleApiService(this._dio);

  Future<List<Article>> fetchArticles() async {
    final response = await _dio.get('/articles');
    final list = response.data as List;
    return list.map((json) => Article.fromJson(json)).toList();
  }

  Future<Article> fetchArticleById(int id) async {
    final response = await _dio.get('/articles/$id');
    return Article.fromJson(response.data);
  }
}
```

### data/local/article_local_data_source.dart

```dart
import 'dart:convert';
import 'package:hive/hive.dart';
import '../../models/article.dart';
import '../cache/cached_response.dart';

class ArticleLocalDataSource {
  static const _boxName = 'article_cache';

  Future<Box<String>> _getBox() async {
    if (Hive.isBoxOpen(_boxName)) return Hive.box<String>(_boxName);
    return Hive.openBox<String>(_boxName);
  }

  /// Lưu danh sách articles vào cache
  Future<void> cacheArticles(List<Article> articles) async {
    final box = await _getBox();
    final cached = CachedResponse(
      data: jsonEncode(articles.map((a) => a.toJson()).toList()),
      cachedAt: DateTime.now(),
      ttl: const Duration(minutes: 10),
    );
    await box.put('all_articles', jsonEncode(cached.toJson()));
  }

  /// Lấy danh sách articles từ cache
  Future<CachedResponse?> getCachedArticles() async {
    final box = await _getBox();
    final raw = box.get('all_articles');
    if (raw == null) return null;
    return CachedResponse.fromJson(jsonDecode(raw));
  }

  /// Parse cached data thành List<Article>
  List<Article> parseArticles(CachedResponse cached) {
    final list = jsonDecode(cached.data) as List;
    return list.map((json) => Article.fromJson(json)).toList();
  }

  /// Xóa cache
  Future<void> clearCache() async {
    final box = await _getBox();
    await box.clear();
  }
}
```

### data/repository/article_repository.dart

```dart
import '../../models/article.dart';
import '../remote/article_api_service.dart';
import '../local/article_local_data_source.dart';

class ArticleRepository {
  final ArticleApiService _api;
  final ArticleLocalDataSource _local;

  ArticleRepository(this._api, this._local);

  /// Cache-first strategy:
  /// 1. Check local cache
  /// 2. If fresh → return cached data
  /// 3. If stale/miss → fetch from API
  /// 4. Update cache
  /// 5. If API fails → return stale cache as fallback
  Future<List<Article>> getArticles({bool forceRefresh = false}) async {
    // Step 1: Check cache (unless force refresh)
    if (!forceRefresh) {
      final cached = await _local.getCachedArticles();
      if (cached != null && cached.isFresh) {
        // Cache hit & fresh → return immediately
        return _local.parseArticles(cached);
      }
    }

    // Step 2: Fetch from network
    try {
      final articles = await _api.fetchArticles();

      // Step 3: Update cache
      await _local.cacheArticles(articles);

      return articles;
    } catch (e) {
      // Step 4: Network failed → try stale cache
      final staleCache = await _local.getCachedArticles();
      if (staleCache != null) {
        // Return stale data — better than nothing
        return _local.parseArticles(staleCache);
      }

      // No cache at all → rethrow
      rethrow;
    }
  }

  /// Network-first strategy (cho data cần mới nhất)
  Future<List<Article>> getArticlesNetworkFirst() async {
    try {
      final articles = await _api.fetchArticles();
      await _local.cacheArticles(articles);
      return articles;
    } catch (e) {
      final cached = await _local.getCachedArticles();
      if (cached != null) return _local.parseArticles(cached);
      rethrow;
    }
  }
}
```

### Sử dụng

```dart
final dio = Dio(BaseOptions(baseUrl: 'https://jsonplaceholder.typicode.com'));
final apiService = ArticleApiService(dio);
final localSource = ArticleLocalDataSource();
final repository = ArticleRepository(apiService, localSource);

// Cache-first (default)
final articles = await repository.getArticles();

// Force refresh (skip cache)
final freshArticles = await repository.getArticles(forceRefresh: true);

// Network-first
final latestArticles = await repository.getArticlesNetworkFirst();
```

### Flow diagram

```
getArticles(forceRefresh: false)
│
├── Check cache
│   ├── Cache HIT & fresh → return cached ✅
│   └── Cache MISS or stale
│       │
│       ├── Fetch API
│       │   ├── Success → update cache → return API data ✅
│       │   └── Failure
│       │       ├── Stale cache exists → return stale ✅ (with warning)
│       │       └── No cache → throw error ❌
│       │
└── forceRefresh: true → skip cache → go to Fetch API
```

---

## VD5: Secure Storage — Auth Token 🔴

### Mục đích

Lưu và đọc auth token (access token, refresh token) an toàn với `flutter_secure_storage`.

> **Liên quan tới:** [6. Secure Storage](01-ly-thuyet.md#6-secure-storage)

### pubspec.yaml

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

### auth_storage.dart

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class AuthStorage {
  static const _keyAccessToken = 'access_token';
  static const _keyRefreshToken = 'refresh_token';
  static const _keyUserId = 'user_id';

  final FlutterSecureStorage _storage;

  AuthStorage({FlutterSecureStorage? storage})
      : _storage = storage ?? const FlutterSecureStorage(
          aOptions: AndroidOptions(encryptedSharedPreferences: true),
          iOptions: IOSOptions(
            accessibility: KeychainAccessibility.first_unlock,
          ),
        );

  // --- Access Token ---
  Future<void> saveAccessToken(String token) async {
    await _storage.write(key: _keyAccessToken, value: token);
  }

  Future<String?> getAccessToken() async {
    return await _storage.read(key: _keyAccessToken);
  }

  Future<bool> hasAccessToken() async {
    return await _storage.containsKey(key: _keyAccessToken);
  }

  // --- Refresh Token ---
  Future<void> saveRefreshToken(String token) async {
    await _storage.write(key: _keyRefreshToken, value: token);
  }

  Future<String?> getRefreshToken() async {
    return await _storage.read(key: _keyRefreshToken);
  }

  // --- User ID ---
  Future<void> saveUserId(String userId) async {
    await _storage.write(key: _keyUserId, value: userId);
  }

  Future<String?> getUserId() async {
    return await _storage.read(key: _keyUserId);
  }

  // --- Save all tokens at once (after login) ---
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
    required String userId,
  }) async {
    await Future.wait([
      saveAccessToken(accessToken),
      saveRefreshToken(refreshToken),
      saveUserId(userId),
    ]);
  }

  // --- Clear all (logout) ---
  Future<void> clearAll() async {
    await _storage.deleteAll();
  }

  // --- Check if logged in ---
  Future<bool> get isLoggedIn async {
    final token = await getAccessToken();
    return token != null && token.isNotEmpty;
  }
}
```

### Sử dụng với Dio Interceptor

```dart
import 'package:dio/dio.dart';
import 'auth_storage.dart';

class AuthInterceptor extends Interceptor {
  final AuthStorage _authStorage;

  AuthInterceptor(this._authStorage);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    // Đọc token từ secure storage
    final token = await _authStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    // Nếu 401 → thử refresh token
    if (err.response?.statusCode == 401) {
      final refreshToken = await _authStorage.getRefreshToken();
      if (refreshToken != null) {
        try {
          // Gọi API refresh
          final newTokens = await _refreshTokens(refreshToken);

          // Lưu tokens mới
          await _authStorage.saveTokens(
            accessToken: newTokens['access_token']!,
            refreshToken: newTokens['refresh_token']!,
            userId: (await _authStorage.getUserId()) ?? '',
          );

          // Retry request ban đầu với token mới
          final retryOptions = err.requestOptions;
          retryOptions.headers['Authorization'] =
              'Bearer ${newTokens['access_token']}';
          final response = await Dio().fetch(retryOptions);
          handler.resolve(response);
          return;
        } catch (_) {
          // Refresh thất bại → logout
          await _authStorage.clearAll();
        }
      }
    }
    handler.next(err);
  }

  Future<Map<String, String>> _refreshTokens(String refreshToken) async {
    final dio = Dio();
    final response = await dio.post(
      'https://api.example.com/auth/refresh',
      data: {'refresh_token': refreshToken},
    );
    return {
      'access_token': response.data['access_token'],
      'refresh_token': response.data['refresh_token'],
    };
  }
}
```

### Login flow

```dart
class AuthService {
  final Dio _dio;
  final AuthStorage _authStorage;

  AuthService(this._dio, this._authStorage);

  Future<void> login(String email, String password) async {
    final response = await _dio.post('/auth/login', data: {
      'email': email,
      'password': password,
    });

    // Lưu tokens vào secure storage
    await _authStorage.saveTokens(
      accessToken: response.data['access_token'],
      refreshToken: response.data['refresh_token'],
      userId: response.data['user_id'],
    );
  }

  Future<void> logout() async {
    // Xóa tất cả tokens
    await _authStorage.clearAll();
  }
}
```

### Điểm chú ý

```
✦ FlutterSecureStorage dùng Keychain (iOS) và EncryptedSharedPreferences (Android)
✦ Dữ liệu được mã hóa ở OS level — an toàn hơn SharedPreferences rất nhiều
✦ AndroidOptions(encryptedSharedPreferences: true) → dùng EncryptedSharedPreferences API 23+
✦ IOSOptions(accessibility: .first_unlock) → accessible sau khi device unlock lần đầu
✦ KHÔNG BAO GIỜ lưu token trong SharedPreferences hoặc Hive không mã hóa
✦ clearAll() khi logout để xóa sạch sensitive data
```

---

## VD6: 🤖 AI Gen → Review — Hive Data Layer 🟢

> **Mục đích:** Luyện workflow "AI gen Hive setup → bạn review adapter registration + field numbers + migration → fix"

> **Liên quan tới:** [2. Hive — NoSQL Database](01-ly-thuyet.md#2-hive--nosql-database)

### Bước 1: Prompt cho AI

```text
Tạo Hive setup cho "Favorite Articles" feature.
Model: Article (id, title, url, savedAt DateTime, tags List<String>).
Output: article_model.dart (@HiveType) + article_data_source.dart (save, remove, getAll, getByTag).
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Field numbers** | @HiveField(0), (1), (2)... sequential? Không skip? |
| 2 | **DateTime** | Dùng millisecondsSinceEpoch? (DateTime directly = crash!) |
| 3 | **Adapter registration** | `Hive.registerAdapter(ArticleAdapter())` TRƯỚC `Hive.openBox`? |
| 4 | **List<String> handling** | Hive hỗ trợ List<String> natively, nhưng kiểm tra nullable? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: DateTime field directly trong Hive model
@HiveType(typeId: 0)
class Article {
  @HiveField(0) final String id;
  @HiveField(1) final DateTime savedAt; // CRASH — Hive ko hỗ trợ DateTime!
}

// ✅ FIX: Lưu millisecondsSinceEpoch
@HiveType(typeId: 0)
class Article {
  @HiveField(0) final String id;
  @HiveField(1) final int savedAtMillis; // millisecondsSinceEpoch

  DateTime get savedAt => DateTime.fromMillisecondsSinceEpoch(savedAtMillis);
}
```

```dart
// ❌ LỖI 2: openBox TRƯỚC registerAdapter
Future<void> main() async {
  await Hive.initFlutter();
  final box = await Hive.openBox<Article>('articles'); // CRASH!
  Hive.registerAdapter(ArticleAdapter()); // Quá trễ!
}

// ✅ FIX: Register adapter TRƯỚC openBox
Future<void> main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(ArticleAdapter()); // TRƯỚC
  final box = await Hive.openBox<Article>('articles'); // SAU
}
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết Hive không hỗ trợ DateTime directly — phải convert
- ✅ Nhận ra adapter registration order quan trọng
- ✅ Kiểm tra field numbers stability cho migration safety
