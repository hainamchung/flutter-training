# Buổi 07: Riverpod Deep Dive — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> ⚠️ Tất cả ví dụ cần thêm dependencies sau vào `pubspec.yaml`:
>
> ```yaml
> dependencies:
>   flutter:
>     sdk: flutter
>   flutter_riverpod: ^2.5.1
>   riverpod_annotation: ^2.3.5
>
> dev_dependencies:
>   riverpod_generator: ^2.4.0
>   build_runner: ^2.4.0
> ```

---

## Ví dụ 1: Basic Provider + ConsumerWidget — Counter đơn giản 🔴

> 📖 **Liên quan:** [Phần 2.3.2 — StateProvider](01-ly-thuyet.md#232-stateprovider--state-đơn-giản-deprecated) · [Phần 2.5 — ConsumerWidget](01-ly-thuyet.md#25-consumerwidget-vs-consumer-vs-consumerstatefulwidget)

### 🎯 Mục tiêu

Làm quen với `StateProvider`, `ConsumerWidget`, `ref.watch`, `ref.read`.

> **Liên quan tới:** [2. Core Concepts 🔴](01-ly-thuyet.md#2-core-concepts)

### Code

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. Khai báo provider — global, compile-time safe
final counterProvider = StateProvider<int>((ref) => 0);

// 2. Main — wrap ProviderScope
void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Counter',
      theme: ThemeData(
        colorSchemeSeed: Colors.blue,
        useMaterial3: true,
      ),
      home: const CounterPage(),
    );
  }
}

// 3. ConsumerWidget — thay StatelessWidget để dùng ref
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref.watch trong build → rebuild khi counter thay đổi
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Riverpod Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('You have pushed the button this many times:'),
            Text(
              '$count',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            heroTag: 'increment',
            // ref.read trong callback → không subscribe
            onPressed: () => ref.read(counterProvider.notifier).state++,
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'decrement',
            onPressed: () => ref.read(counterProvider.notifier).state--,
            child: const Icon(Icons.remove),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'reset',
            onPressed: () => ref.read(counterProvider.notifier).state = 0,
            child: const Icon(Icons.refresh),
          ),
        ],
      ),
    );
  }
}
```

### 📝 Giải thích

| Dòng | Giải thích |
|------|-----------|
| `final counterProvider = StateProvider<int>((ref) => 0)` | Khai báo provider global, giá trị ban đầu = 0 |
| `ProviderScope(child: MyApp())` | Khởi tạo Riverpod, bắt buộc ở root |
| `ConsumerWidget` | Widget có thể dùng `ref` để đọc providers |
| `ref.watch(counterProvider)` | Subscribe counter — rebuild khi thay đổi |
| `ref.read(counterProvider.notifier).state++` | Đọc 1 lần + thay đổi state |

- 🔗 **FE tương đương:** `ref.watch` ≈ `useSelector` (reactive subscribe), `ref.read` ≈ `store.getState()` (one-time read) — nhưng Riverpod enforce rule này tại compile-time.

> ⚠️ **Deprecation Notice:** `StateProvider` đã deprecated trong Riverpod 2.x+. Ở đây dùng để hiểu concept cơ bản. Trong production, hãy dùng `NotifierProvider` thay thế (xem VD2).

- 🔗 **FE tương đương:** Tương tự Zustand `create(set => ({ count: 0 }))` hoặc Jotai `atom(0)` — nhưng Riverpod compile-safe và có `autoDispose` built-in.

### 🔍 So sánh với Provider (package cũ)

```dart
// Provider (cũ) — cần context, có thể ProviderNotFoundException
final count = context.watch<CounterModel>().value;

// Riverpod — không cần context, compile-time safe
final count = ref.watch(counterProvider);
```

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_riverpod_counter
cd vidu_riverpod_counter
# Thêm dependency
flutter pub add flutter_riverpod
# Thay nội dung lib/main.dart bằng code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Text "You have pushed the button this many times:" với số đếm bên dưới
✅ 3 FloatingActionButton: +, −, reset
✅ Nhấn + → count tăng, nhấn − → count giảm, nhấn reset → count về 0
```

---

## Ví dụ 2: NotifierProvider — Todo List 🔴

> 📖 **Liên quan:** [Phần 2.3.6 — NotifierProvider / AsyncNotifierProvider](01-ly-thuyet.md#236-notifierprovider--asyncnotifierprovider---khuyến-khích-dùng)

### 🎯 Mục tiêu

Sử dụng `NotifierProvider` (modern approach) cho state phức tạp với add, toggle, remove.

> **Liên quan tới:** [2. Core Concepts 🔴](01-ly-thuyet.md#2-core-concepts)

### Code

```dart
// lib/models/todo.dart
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  const Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  Todo copyWith({String? id, String? title, bool? isCompleted}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}
```

```dart
// lib/providers/todo_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/todo.dart';

class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    // State ban đầu — list rỗng
    return [];
  }

  void addTodo(String title) {
    state = [
      ...state,
      Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: title,
      ),
    ];
  }

  void toggleTodo(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList();
  }

  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}

// Provider declaration
final todosProvider =
    NotifierProvider<TodosNotifier, List<Todo>>(TodosNotifier.new);

// Computed providers — derived state
enum TodoFilter { all, active, completed }

final todoFilterProvider = StateProvider<TodoFilter>((ref) => TodoFilter.all);

final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(todosProvider);
  final filter = ref.watch(todoFilterProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.active:
      return todos.where((t) => !t.isCompleted).toList();
    case TodoFilter.completed:
      return todos.where((t) => t.isCompleted).toList();
  }
});

final todoCountProvider = Provider<int>((ref) {
  return ref.watch(todosProvider).length;
});
```

```dart
// lib/screens/todo_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/todo_provider.dart';

class TodoScreen extends ConsumerWidget {
  const TodoScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final filteredTodos = ref.watch(filteredTodosProvider);
    final totalCount = ref.watch(todoCountProvider);
    final currentFilter = ref.watch(todoFilterProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('Todo List ($totalCount)'),
      ),
      body: Column(
        children: [
          // Filter buttons
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: SegmentedButton<TodoFilter>(
              segments: const [
                ButtonSegment(value: TodoFilter.all, label: Text('All')),
                ButtonSegment(value: TodoFilter.active, label: Text('Active')),
                ButtonSegment(
                    value: TodoFilter.completed, label: Text('Done')),
              ],
              selected: {currentFilter},
              onSelectionChanged: (selection) {
                ref.read(todoFilterProvider.notifier).state = selection.first;
              },
            ),
          ),
          // Todo list
          Expanded(
            child: filteredTodos.isEmpty
                ? const Center(child: Text('No todos yet!'))
                : ListView.builder(
                    itemCount: filteredTodos.length,
                    itemBuilder: (_, index) {
                      final todo = filteredTodos[index];
                      return ListTile(
                        leading: Checkbox(
                          value: todo.isCompleted,
                          onChanged: (_) {
                            ref
                                .read(todosProvider.notifier)
                                .toggleTodo(todo.id);
                          },
                        ),
                        title: Text(
                          todo.title,
                          style: TextStyle(
                            decoration: todo.isCompleted
                                ? TextDecoration.lineThrough
                                : null,
                          ),
                        ),
                        trailing: IconButton(
                          icon: const Icon(Icons.delete, color: Colors.red),
                          onPressed: () {
                            ref
                                .read(todosProvider.notifier)
                                .removeTodo(todo.id);
                          },
                        ),
                      );
                    },
                  ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddTodoDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddTodoDialog(BuildContext context, WidgetRef ref) {
    final controller = TextEditingController();
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Add Todo'),
        content: TextField(
          controller: controller,
          autofocus: true,
          decoration: const InputDecoration(hintText: 'Enter todo title'),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () {
              final title = controller.text.trim();
              if (title.isNotEmpty) {
                ref.read(todosProvider.notifier).addTodo(title);
                Navigator.pop(context);
              }
            },
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }
}
```

### 📝 Giải thích

- **`Notifier<List<Todo>>`**: Class quản lý state, method `build()` trả về state ban đầu
- **Immutable state**: Mỗi lần thay đổi tạo list mới (`[...state, newTodo]`), không mutate
- **Computed providers**: `filteredTodosProvider` tự động cập nhật khi `todosProvider` hoặc `todoFilterProvider` thay đổi
- **Separation of concerns**: Model → Provider → UI, mỗi layer độc lập

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_riverpod_todo
cd vidu_riverpod_todo
# Thêm dependency
flutter pub add flutter_riverpod
# Tạo các file: lib/models/todo.dart, lib/providers/todo_provider.dart, lib/screens/todo_screen.dart
# Cập nhật lib/main.dart để import và chạy TodoScreen với ProviderScope, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ AppBar hiện "Todo List (0)" — count tự cập nhật
✅ SegmentedButton filter: All / Active / Done
✅ Nhấn FAB (+) → dialog nhập todo → thêm vào list
✅ Tap checkbox → todo gạch ngang (completed), filter "Done" chỉ hiện completed
✅ Nhấn icon delete → xóa todo khỏi list
```

---

## Ví dụ 3: FutureProvider — Fetch Data từ Mock API 🔴

> 📖 **Liên quan:** [Phần 2.3.4 — FutureProvider](01-ly-thuyet.md#234-futureprovider--dữ-liệu-async)

### 🎯 Mục tiêu

Sử dụng `FutureProvider` + `AsyncValue.when()` để fetch và hiển thị data.

> **Liên quan tới:** [2. Core Concepts 🔴](01-ly-thuyet.md#2-core-concepts)

### Code

```dart
// lib/models/post.dart
class Post {
  final int id;
  final String title;
  final String body;

  const Post({required this.id, required this.title, required this.body});

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}
```

```dart
// lib/providers/post_provider.dart
import 'dart:convert';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:http/http.dart' as http;
import '../models/post.dart';

// Mock API service
class PostRepository {
  Future<List<Post>> fetchPosts() async {
    // Simulate network delay
    await Future.delayed(const Duration(seconds: 1));

    // Mock data — thay bằng API thật trong production
    final mockData = List.generate(
      20,
      (i) => {
        'id': i + 1,
        'title': 'Post ${i + 1}: Lorem ipsum dolor sit amet',
        'body':
            'This is the body of post ${i + 1}. Lorem ipsum dolor sit amet, '
                'consectetur adipiscing elit.',
      },
    );

    return mockData.map((json) => Post.fromJson(json)).toList();
  }
}

// Repository provider — DI
final postRepositoryProvider = Provider<PostRepository>((ref) {
  return PostRepository();
});

// FutureProvider — auto handle loading/error/data
final postsProvider = FutureProvider.autoDispose<List<Post>>((ref) async {
  final repository = ref.watch(postRepositoryProvider);
  return repository.fetchPosts();
});
```

```dart
// lib/screens/posts_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/post_provider.dart';

class PostsScreen extends ConsumerWidget {
  const PostsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // AsyncValue — tự động wrap loading/error/data
    final postsAsync = ref.watch(postsProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Posts — FutureProvider'),
        actions: [
          // Refresh button — invalidate để fetch lại
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => ref.invalidate(postsProvider),
          ),
        ],
      ),
      body: postsAsync.when(
        // Khi đang loading
        loading: () => const Center(
          child: CircularProgressIndicator(),
        ),

        // Khi có lỗi
        error: (error, stackTrace) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.error, size: 48, color: Colors.red),
              const SizedBox(height: 16),
              Text('Error: $error'),
              const SizedBox(height: 16),
              FilledButton(
                onPressed: () => ref.invalidate(postsProvider),
                child: const Text('Retry'),
              ),
            ],
          ),
        ),

        // Khi có data
        data: (posts) => RefreshIndicator(
          onRefresh: () async => ref.invalidate(postsProvider),
          child: ListView.builder(
            itemCount: posts.length,
            itemBuilder: (_, index) {
              final post = posts[index];
              return Card(
                margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                child: ListTile(
                  leading: CircleAvatar(child: Text('${post.id}')),
                  title: Text(
                    post.title,
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  subtitle: Text(
                    post.body,
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}
```

### 📝 Giải thích

| Concept | Giải thích |
|---------|-----------|
| `FutureProvider.autoDispose` | Fetch data + tự cleanup khi rời screen |
| `AsyncValue.when()` | Pattern matching cho 3 trạng thái: loading, error, data |
| `ref.invalidate(postsProvider)` | Force re-fetch data (giống pull-to-refresh) |
| `postRepositoryProvider` | Dependency injection — dễ mock trong tests |

- 🔗 **FE tương đương:** Tương tự React Query `useQuery(['todos'], fetchTodos)` — FutureProvider tự handle loading/error states giống React Query.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_future_provider
cd vidu_future_provider
# Thêm dependency
flutter pub add flutter_riverpod
# Tạo các file: lib/models/post.dart, lib/providers/post_provider.dart, lib/screens/posts_screen.dart
# Cập nhật lib/main.dart để import và chạy PostsScreen với ProviderScope, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Loading spinner hiện 1 giây (simulate network delay)
✅ Danh sách 20 posts hiện dạng Card với id, title, body
✅ Nhấn icon refresh trên AppBar → loading lại từ đầu
✅ Kéo xuống (pull-to-refresh) → cũng reload data
```

---

## Ví dụ 4: family + autoDispose — User Detail by ID 🟡

### 🎯 Mục tiêu

Sử dụng `family` modifier để tạo parameterized provider, kết hợp `autoDispose`.

> **Liên quan tới:** [4. Modifiers 🟡](01-ly-thuyet.md#4-modifiers)

### Code

```dart
// lib/models/user.dart
class User {
  final String id;
  final String name;
  final String email;
  final String avatarUrl;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.avatarUrl,
  });
}
```

```dart
// lib/providers/user_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../models/user.dart';

// Mock user database
final _mockUsers = {
  '1': const User(
    id: '1',
    name: 'Nguyen Van A',
    email: 'a@example.com',
    avatarUrl: 'https://i.pravatar.cc/150?img=1',
  ),
  '2': const User(
    id: '2',
    name: 'Tran Thi B',
    email: 'b@example.com',
    avatarUrl: 'https://i.pravatar.cc/150?img=2',
  ),
  '3': const User(
    id: '3',
    name: 'Le Van C',
    email: 'c@example.com',
    avatarUrl: 'https://i.pravatar.cc/150?img=3',
  ),
};

// family + autoDispose — provider khác nhau cho mỗi userId
final userDetailProvider = FutureProvider.autoDispose
    .family<User, String>((ref, userId) async {
  // Log để thấy khi nào provider được tạo/dispose
  ref.onDispose(() {
    // Provider cho userId này đã được dispose
  });

  // Simulate API call
  await Future.delayed(const Duration(milliseconds: 800));

  final user = _mockUsers[userId];
  if (user == null) {
    throw Exception('User not found: $userId');
  }
  return user;
});

// Provider cho danh sách user IDs
final userIdsProvider = Provider<List<String>>((ref) {
  return ['1', '2', '3'];
});
```

```dart
// lib/screens/user_list_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/user_provider.dart';
import 'user_detail_screen.dart';

class UserListScreen extends ConsumerWidget {
  const UserListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userIds = ref.watch(userIdsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Users — family modifier')),
      body: ListView.builder(
        itemCount: userIds.length,
        itemBuilder: (_, index) {
          final userId = userIds[index];
          // Mỗi userId tạo 1 instance riêng của userDetailProvider
          final userAsync = ref.watch(userDetailProvider(userId));

          return userAsync.when(
            loading: () => const ListTile(
              leading: CircleAvatar(child: CircularProgressIndicator()),
              title: Text('Loading...'),
            ),
            error: (err, _) => ListTile(
              leading: const CircleAvatar(
                  child: Icon(Icons.error, color: Colors.red)),
              title: Text('Error: $err'),
            ),
            data: (user) => ListTile(
              leading: CircleAvatar(
                backgroundImage: NetworkImage(user.avatarUrl),
              ),
              title: Text(user.name),
              subtitle: Text(user.email),
              trailing: const Icon(Icons.chevron_right),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => UserDetailScreen(userId: user.id),
                  ),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

```dart
// lib/screens/user_detail_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/user_provider.dart';

class UserDetailScreen extends ConsumerWidget {
  final String userId;

  const UserDetailScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Dùng lại provider với cùng userId — cached, không fetch lại
    final userAsync = ref.watch(userDetailProvider(userId));

    return Scaffold(
      appBar: AppBar(title: const Text('User Detail')),
      body: userAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (err, _) => Center(child: Text('Error: $err')),
        data: (user) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              CircleAvatar(
                radius: 60,
                backgroundImage: NetworkImage(user.avatarUrl),
              ),
              const SizedBox(height: 16),
              Text(
                user.name,
                style: Theme.of(context).textTheme.headlineMedium,
              ),
              const SizedBox(height: 8),
              Text(
                user.email,
                style: Theme.of(context).textTheme.bodyLarge,
              ),
              const SizedBox(height: 8),
              Text(
                'ID: ${user.id}',
                style: Theme.of(context).textTheme.bodySmall,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 📝 Giải thích

| Concept | Giải thích |
|---------|-----------|
| `.family<User, String>` | Provider nhận parameter `String` (userId), trả về `User` |
| `userDetailProvider(userId)` | Mỗi userId khác nhau tạo 1 instance provider riêng |
| `autoDispose` | Khi navigate back (không còn widget listen), provider tự dispose |
| Cache behavior | Nếu 2 widget cùng watch `userDetailProvider('1')`, chỉ fetch 1 lần |

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §4.2 family — Provider có tham số](01-ly-thuyet.md#42-family--provider-có-tham-số) · [§4.3 Kết hợp autoDispose + family](01-ly-thuyet.md#43-kết-hợp-autodispose--family)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_riverpod_family
cd vidu_riverpod_family
# Thêm dependency
flutter pub add flutter_riverpod
# Tạo các file: lib/models/user.dart, lib/providers/user_provider.dart,
# lib/screens/user_list_screen.dart, lib/screens/user_detail_screen.dart
# Cập nhật lib/main.dart để import và chạy UserListScreen với ProviderScope, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Danh sách 3 users với avatar, tên, email (loading 0.8s mỗi user)
✅ Tap vào user → UserDetailScreen hiện avatar lớn + thông tin chi tiết
✅ Quay lại list → provider đã cache, không fetch lại (autoDispose chỉ cleanup khi rời screen)
✅ Mỗi userId khác nhau có loading state riêng biệt (family)
```

---

## Ví dụ 5: @riverpod Code Generation 🟢

### 🎯 Mục tiêu

Viết lại counter và todo bằng code-gen approach — giảm boilerplate.

> **Liên quan tới:** [2. Core Concepts 🔴](01-ly-thuyet.md#2-core-concepts)

### Setup

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
```

### Code

```dart
// lib/providers/counter_codegen.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_codegen.g.dart';

// ✅ Code-gen: Provider read-only (function → Provider)
@riverpod
String appTitle(AppTitleRef ref) {
  return 'Riverpod Code-Gen Demo';
}

// ✅ Code-gen: NotifierProvider (class → NotifierProvider)
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0; // Initial state

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}
```

```dart
// lib/providers/todo_codegen.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../models/todo.dart';

part 'todo_codegen.g.dart';

// ✅ Code-gen: NotifierProvider cho Todo list
@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() => [];

  void addTodo(String title) {
    state = [
      ...state,
      Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: title,
      ),
    ];
  }

  void toggleTodo(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList();
  }

  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}

// ✅ Code-gen: FutureProvider với family (parameterized)
@riverpod
Future<Todo?> todoById(TodoByIdRef ref, String id) async {
  final todos = ref.watch(todosProvider);
  return todos.where((t) => t.id == id).firstOrNull;
}

// ✅ Code-gen: Computed provider
@riverpod
int completedTodoCount(CompletedTodoCountRef ref) {
  final todos = ref.watch(todosProvider);
  return todos.where((t) => t.isCompleted).length;
}
```

```dart
// lib/screens/codegen_demo_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_codegen.dart';
import '../providers/todo_codegen.dart';

class CodeGenDemoScreen extends ConsumerWidget {
  const CodeGenDemoScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Sử dụng y hệt — generated providers có cùng API
    final appTitle = ref.watch(appTitleProvider);
    final count = ref.watch(counterProvider);
    final todos = ref.watch(todosProvider);
    final completedCount = ref.watch(completedTodoCountProvider);

    return Scaffold(
      appBar: AppBar(title: Text(appTitle)),
      body: Column(
        children: [
          // Counter section
          Card(
            margin: const EdgeInsets.all(16),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  IconButton(
                    onPressed: () =>
                        ref.read(counterProvider.notifier).decrement(),
                    icon: const Icon(Icons.remove),
                  ),
                  Text(
                    '$count',
                    style: Theme.of(context).textTheme.headlineLarge,
                  ),
                  IconButton(
                    onPressed: () =>
                        ref.read(counterProvider.notifier).increment(),
                    icon: const Icon(Icons.add),
                  ),
                ],
              ),
            ),
          ),

          // Stats
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Text(
              'Todos: ${todos.length} | Completed: $completedCount',
              style: Theme.of(context).textTheme.titleMedium,
            ),
          ),

          // Todo list
          Expanded(
            child: ListView.builder(
              itemCount: todos.length,
              itemBuilder: (_, index) {
                final todo = todos[index];
                return ListTile(
                  leading: Checkbox(
                    value: todo.isCompleted,
                    onChanged: (_) {
                      ref.read(todosProvider.notifier).toggleTodo(todo.id);
                    },
                  ),
                  title: Text(todo.title),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: () {
                      ref.read(todosProvider.notifier).removeTodo(todo.id);
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          final controller = TextEditingController();
          showDialog(
            context: context,
            builder: (context) => AlertDialog(
              title: const Text('New Todo'),
              content: TextField(
                controller: controller,
                autofocus: true,
              ),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('Cancel'),
                ),
                FilledButton(
                  onPressed: () {
                    final title = controller.text.trim();
                    if (title.isNotEmpty) {
                      ref.read(todosProvider.notifier).addTodo(title);
                      Navigator.pop(context);
                    }
                  },
                  child: const Text('Add'),
                ),
              ],
            ),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Chạy code generation

```bash
# Build 1 lần
dart run build_runner build --delete-conflicting-outputs

# Watch mode — tự generate khi file thay đổi
dart run build_runner watch --delete-conflicting-outputs
```

### 📝 So sánh Manual vs Code-Gen

| Aspect | Manual | Code-Gen (@riverpod) |
|--------|--------|---------------------|
| Khai báo | `final p = NotifierProvider<T, S>(T.new)` | `@riverpod class T extends _$T` |
| autoDispose | `.autoDispose` phải thêm thủ công | Tự động (mặc định) |
| family | `.family<T, P>` phải thêm thủ công | Tự động từ function params |
| Type safety | Phải khai báo đúng generic types | Tự infer từ code |
| Boilerplate | Nhiều hơn | Ít hơn đáng kể |
| Build step | Không cần | Cần `build_runner` |

### Generated file (tham khảo)

File `.g.dart` được tạo tự động sẽ trông như:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND
// counter_codegen.g.dart

part of 'counter_codegen.dart';

String _$appTitleHash() => r'abc123...';

/// See also [appTitle].
@ProviderFor(appTitle)
final appTitleProvider = AutoDisposeProvider<String>.internal(
  appTitle,
  name: r'appTitleProvider',
  ...
);

/// See also [Counter].
@ProviderFor(Counter)
final counterProvider = AutoDisposeNotifierProvider<Counter, int>.internal(
  Counter.new,
  name: r'counterProvider',
  ...
);
```

> 💡 **Không cần đọc file `.g.dart`** — chỉ cần biết nó tạo ra các provider giống manual approach.

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §2.6 Code Generation — @riverpod annotation](01-ly-thuyet.md#26-code-generation--riverpod-annotation)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_riverpod_codegen
cd vidu_riverpod_codegen
# Thêm dependencies
flutter pub add flutter_riverpod riverpod_annotation
flutter pub add dev:riverpod_generator dev:build_runner
# Tạo các file: lib/models/todo.dart, lib/providers/counter_codegen.dart,
# lib/providers/todo_codegen.dart, lib/screens/codegen_demo_screen.dart
# Chạy code generation:
dart run build_runner build --delete-conflicting-outputs
# Cập nhật lib/main.dart để import và chạy CodeGenDemoScreen với ProviderScope, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ AppBar hiện title "Riverpod Code-Gen Demo" (từ appTitleProvider)
✅ Counter section: nhấn +/− → count thay đổi
✅ Stats hiện "Todos: X | Completed: Y" — cập nhật real-time
✅ Nhấn FAB (+) → dialog nhập todo → thêm vào list
✅ Tap checkbox → toggle completed, count cập nhật
```

---

## VD6: 🤖 AI Gen → Review — Riverpod Weather Provider 🟢

> **Mục đích:** Luyện workflow "AI gen Riverpod providers → bạn review ref usage + dispose → fix issues"

> **Liên quan tới:** [3. ref.watch, ref.read, ref.listen 🔴](01-ly-thuyet.md#3-refwatch-refread-reflisten)

### Bước 1: Prompt cho AI

```text
Tạo Riverpod providers cho Weather feature:
1. weatherProvider: FutureProvider.family fetch weather by city name (mock data OK).
2. favoritesNotifier: NotifierProvider manage list of favorite cities.
Dùng ConsumerWidget hiển thị weather data với AsyncValue.when.
Output: 2 provider files + 1 screen widget.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **ref.watch vs ref.read** | `ref.watch` trong build? `ref.read` trong callbacks? Ngược lại = BUG |
| 2 | **autoDispose** | FutureProvider có `.autoDispose`? Thiếu → không cleanup khi rời screen |
| 3 | **state = newState** | Notifier dùng `state =` hay `notifyListeners()`? (notifyListeners là ChangeNotifier, SAI) |
| 4 | **AsyncValue.when** | Handle đủ `data`, `loading`, `error`? Hay chỉ handle data? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Dùng ChangeNotifier pattern trong Riverpod Notifier
class FavoritesNotifier extends Notifier<List<String>> {
  void addCity(String city) {
    state.add(city);           // WRONG — mutate existing list
    notifyListeners();         // WRONG — ChangeNotifier method, không có trong Notifier
  }
}

// ✅ FIX: Immutable state update
class FavoritesNotifier extends Notifier<List<String>> {
  @override
  List<String> build() => [];
  
  void addCity(String city) {
    if (!state.contains(city)) {
      state = [...state, city];  // CORRECT — tạo list mới, trigger rebuild tự động
    }
  }
}
```

```dart
// ❌ LỖI 2: Thiếu autoDispose cho API provider
final weatherProvider = FutureProvider.family<Weather, String>(...);
// → Provider tồn tại mãi, mỗi city tạo 1 instance → memory leak

// ✅ FIX: Thêm autoDispose
final weatherProvider = FutureProvider.autoDispose.family<Weather, String>(...);
// → Tự cleanup khi không có widget nào watch
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Phân biệt Riverpod Notifier (state =) vs ChangeNotifier (notifyListeners)
- ✅ Biết khi nào cần autoDispose cho FutureProvider
- ✅ Hiểu immutable state update pattern trong Riverpod
