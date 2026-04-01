# Buổi 07: Riverpod Deep Dive — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 7/16** · **Thời lượng tự học:** ~2 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 06 (lý thuyết + ít nhất BT1-BT2)

## 1. Tại sao Riverpod? 🔴

### 1.1 Vấn đề của Provider (package cũ)

Provider là package state management phổ biến nhất trong Flutter, nhưng có nhiều hạn chế quan trọng:

#### ❌ Runtime errors — ProviderNotFoundException

```dart
// Provider: lỗi chỉ phát hiện khi chạy app!
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Nếu quên wrap ChangeNotifierProvider ở trên → CRASH lúc runtime
    final counter = context.watch<CounterNotifier>();
    return Text('${counter.value}');
  }
}
```

Lỗi `ProviderNotFoundException` chỉ xảy ra **khi chạy app**, không thể phát hiện lúc compile.

#### ❌ Phụ thuộc BuildContext

```dart
// Provider: PHẢI có BuildContext
void _onButtonPressed(BuildContext context) {
  // Không thể dùng Provider bên ngoài widget tree
  final auth = context.read<AuthService>();
  auth.login();
}

// Ở service layer, không có context → không lấy được provider
class ApiService {
  // ❌ Không có cách nào lấy AuthService ở đây nếu dùng Provider
}
```

#### ❌ Không thể có 2 Provider cùng type

```dart
// Provider: ❌ Không phân biệt được
Provider<String>(create: (_) => 'Hello'),
Provider<String>(create: (_) => 'World'),

// Widget con chỉ nhận được 1 trong 2
final value = context.watch<String>(); // 'Hello' hay 'World'?
```

#### ❌ Không có compile-time safety

Tất cả lỗi Provider (thiếu ancestor, sai type) đều là **runtime errors**.

### 1.2 Riverpod giải quyết như thế nào?

| Vấn đề Provider | Giải pháp Riverpod |
|---|---|
| Runtime errors | ✅ **Compile-time safety** — lỗi phát hiện lúc code |
| Phụ thuộc BuildContext | ✅ **Không cần BuildContext** — providers là global |
| Không thể 2 cùng type | ✅ **Mỗi provider là biến riêng** — khai báo bao nhiêu tùy ý |
| Khó test | ✅ **Dễ test** — ProviderContainer, override dễ dàng |
| Không tự dispose | ✅ **autoDispose** — tự cleanup khi không dùng |

```dart
// Riverpod: providers khai báo global, compile-time safe
final counterProvider = StateProvider<int>((ref) => 0);
final greetingProvider = Provider<String>((ref) => 'Hello');
final anotherGreetingProvider = Provider<String>((ref) => 'World');

// Không cần BuildContext, không bao giờ ProviderNotFoundException
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

### 1.3 Riverpod = "Provider" nhưng viết ngược lại

> **Riverpod** là anagram (đảo chữ) của **Provider** — cùng tác giả (Remi Rousselet), là phiên bản "viết lại từ đầu" để sửa mọi hạn chế.

```
P-r-o-v-i-d-e-r  →  R-i-v-e-r-p-o-d
```

> 🔗 **FE Bridge:** Riverpod giải quyết vấn đề tương tự React Query / Zustand / Jotai — state management compile-safe, không dùng Context. **Khác ở**: Riverpod = compile-time safety cho provider dependencies, FE libs dùng runtime resolution. Nếu quen React Query → Riverpod's `FutureProvider` sẽ rất familiar.

---

## 2. Core Concepts 🔴

### 2.1 Cài đặt

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5
  http: ^1.2.0  # Cần cho các ví dụ API call (family provider, FutureProvider...)

dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
```

> 📖 Chi tiết migration guide Riverpod 2.x → 3.x xem tại [Tài liệu tham khảo](04-tai-lieu-tham-khao.md#migration).
>
> Tài liệu này giảng theo Riverpod 2.x API.

### 2.2 ProviderScope — Bắt buộc

```dart
void main() {
  runApp(
    // ProviderScope PHẢI wrap toàn bộ app
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

> ⚠️ **Bắt buộc**: Mọi app dùng Riverpod phải có `ProviderScope` ở root. Khác với Provider package — ProviderScope là nơi lưu trữ state của tất cả providers.

### 2.3 Các loại Provider

#### 2.3.1 Provider — Giá trị read-only

Dùng cho giá trị **không thay đổi** hoặc **computed values**.

```dart
// Provider read-only
final appNameProvider = Provider<String>((ref) => 'My Riverpod App');

// Computed provider — phụ thuộc provider khác
final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(todosProvider);
  final filter = ref.watch(filterProvider);

  switch (filter) {
    case TodoFilter.completed:
      return todos.where((t) => t.isCompleted).toList();
    case TodoFilter.active:
      return todos.where((t) => !t.isCompleted).toList();
    case TodoFilter.all:
      return todos;
  }
});
```

#### 2.3.2 StateProvider — State đơn giản (deprecated)

> ⚠️ **Riverpod 2.x+**: `StateProvider` vẫn hoạt động nhưng khuyến khích dùng `NotifierProvider` cho code mới. `StateNotifierProvider` cũng deprecated → dùng `Notifier` + `NotifierProvider`.

```dart
// ❌ StateProvider (legacy) — chỉ dùng cho code cũ
final counterProvider = StateProvider<int>((ref) => 0);
// Sử dụng: ref.read(counterProvider.notifier).state++;

// ✅ NotifierProvider (recommended) — dùng cho code mới
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

final counterProvider =
    NotifierProvider<CounterNotifier, int>(CounterNotifier.new);

// Sử dụng
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return ElevatedButton(
      onPressed: () => ref.read(counterProvider.notifier).increment(),
      child: Text('Count: $count'),
    );
  }
}
```

#### 2.3.3 StateNotifierProvider — Legacy (vẫn còn dùng nhiều)

```dart
// StateNotifier class — legacy, gặp nhiều trong codebases cũ
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

// StateNotifierProvider
final counterProvider =
    StateNotifierProvider<CounterNotifier, int>((ref) => CounterNotifier());
```

> ⚠️ `StateNotifierProvider` cũng deprecated. Với code mới, dùng `Notifier` + `NotifierProvider` (xem 2.3.6). Chỉ giữ `StateNotifierProvider` khi maintain code cũ.

#### 2.3.4 FutureProvider — Dữ liệu async

Dùng cho **API calls**, **đọc file**, **shared preferences** — bất cứ gì trả về `Future`.

```dart
final userProvider = FutureProvider<User>((ref) async {
  final repository = ref.watch(userRepositoryProvider);
  return repository.fetchUser();
});

// Sử dụng — tự động handle loading/error/data
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

#### 2.3.5 StreamProvider — Stream data

Dùng cho **Firebase Realtime**, **WebSocket**, **SSE** — bất cứ gì là `Stream`.

```dart
final messagesProvider = StreamProvider<List<Message>>((ref) {
  final chatService = ref.watch(chatServiceProvider);
  return chatService.messagesStream;
});

// Sử dụng — giống FutureProvider, dùng .when()
class ChatWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return messagesAsync.when(
      data: (messages) => ListView.builder(
        itemCount: messages.length,
        itemBuilder: (_, i) => Text(messages[i].text),
      ),
      loading: () => const CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

#### 2.3.6 NotifierProvider / AsyncNotifierProvider — ⭐ Khuyến khích dùng

Đây là **cách hiện đại nhất** để quản lý state trong Riverpod.

```dart
// Notifier — cho synchronous state
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    // Initial state — thay vì constructor
    return [];
  }

  void addTodo(String title) {
    state = [
      ...state,
      Todo(id: DateTime.now().toString(), title: title),
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

final todosProvider =
    NotifierProvider<TodosNotifier, List<Todo>>(TodosNotifier.new);
```

```dart
// AsyncNotifier — cho async state (API calls)
class UsersNotifier extends AsyncNotifier<List<User>> {
  @override
  Future<List<User>> build() async {
    // Fetch initial data
    return _fetchUsers();
  }

  Future<List<User>> _fetchUsers() async {
    final repository = ref.read(userRepositoryProvider);
    return repository.getUsers();
  }

  Future<void> addUser(String name) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(userRepositoryProvider);
      await repository.addUser(name);
      return _fetchUsers();
    });
  }
}

final usersProvider =
    AsyncNotifierProvider<UsersNotifier, List<User>>(UsersNotifier.new);
```

### 2.4 Bảng so sánh các loại Provider

| Loại | Khi nào dùng | State | Async | Recommended |
|------|-------------|-------|-------|-------------|
| `Provider` | Computed values, DI | Read-only | ❌ | ✅ |
| `StateProvider` | Counter, toggle | Mutable đơn giản | ❌ | ⚠️ Deprecated |
| `StateNotifierProvider` | Complex state | Mutable | ❌ | ⚠️ Legacy |
| `FutureProvider` | API call 1 lần | Async data | ✅ | ✅ |
| `StreamProvider` | Realtime data | Stream | ✅ | ✅ |
| `NotifierProvider` | Complex sync state | Mutable | ❌ | ⭐ Khuyến khích |
| `AsyncNotifierProvider` | Complex async state | Mutable + Async | ✅ | ⭐ Khuyến khích |

### 2.5 ConsumerWidget vs Consumer vs ConsumerStatefulWidget

```dart
// ConsumerWidget — thay StatelessWidget
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}

// Consumer — rebuild chỉ 1 phần widget tree
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Phần này KHÔNG rebuild'),
        Consumer(
          builder: (context, ref, child) {
            final value = ref.watch(myProvider);
            return Text('Chỉ phần này rebuild: $value');
          },
        ),
      ],
    );
  }
}

// ConsumerStatefulWidget — khi cần initState, dispose
class MyStatefulWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends ConsumerState<MyStatefulWidget> {
  @override
  void initState() {
    super.initState();
    // Có thể dùng ref ở đây
  }

  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

### 2.6 Code Generation — @riverpod annotation

Thay vì viết provider thủ công, dùng **code generation** để giảm boilerplate:

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'my_providers.g.dart';

// Tự động tạo Provider (read-only)
@riverpod
String appName(AppNameRef ref) {
  return 'My Riverpod App';
}

// Tự động tạo FutureProvider
@riverpod
Future<User> user(UserRef ref) async {
  final repository = ref.watch(userRepositoryProvider);
  return repository.fetchUser();
}

// Tự động tạo NotifierProvider
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// Tự động tạo AsyncNotifierProvider
@riverpod
class UserList extends _$UserList {
  @override
  Future<List<User>> build() async {
    return _fetchUsers();
  }

  Future<List<User>> _fetchUsers() async {
    // fetch logic
    return [];
  }
}
```

Chạy code generation:

```bash
dart run build_runner build
# hoặc watch mode
dart run build_runner watch
```

> 💡 **Tip**: Code-gen tự động xác định `autoDispose` và giảm lỗi typo. Với project mới, nên dùng code-gen approach.

> 🔗 **FE Bridge:** Provider types mapping: `Provider` ≈ computed value (Zustand derived), `StateProvider` ≈ `atom` (Jotai), `FutureProvider` ≈ `useQuery` (React Query), `StreamProvider` ≈ RxJS Observable subscription, `StateNotifierProvider` ≈ `useReducer` + external store. Nhưng **khác ở**: tất cả đều **compile-safe** — sai type = compile error, không phải runtime crash.

---

## 3. ref.watch, ref.read, ref.listen 🔴

### 3.1 ref.watch — Theo dõi & rebuild

```dart
class TodoListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ ref.watch trong build → widget rebuild khi todos thay đổi
    final todos = ref.watch(todosProvider);

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (_, i) => Text(todos[i].title),
    );
  }
}
```

**Quy tắc**: Dùng `ref.watch` trong `build()` method.

### 3.2 ref.read — Đọc 1 lần, không rebuild

```dart
class AddTodoButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () {
        // ✅ ref.read trong callback → đọc 1 lần, không subscribe
        ref.read(todosProvider.notifier).addTodo('New todo');
      },
      child: const Text('Add Todo'),
    );
  }
}
```

**Quy tắc**: Dùng `ref.read` trong callbacks (`onPressed`, `onTap`, etc.).

### 3.3 ref.listen — Phản ứng khi thay đổi

```dart
class MyWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    // ref.listen — chạy callback khi giá trị thay đổi
    ref.listen<AsyncValue<void>>(
      submitProvider,
      (previous, next) {
        next.whenOrNull(
          error: (err, stack) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('Error: $err')),
            );
          },
          data: (_) {
            Navigator.of(context).pop();
          },
        );
      },
    );

    return const SizedBox();
  }
}
```

**Quy tắc**: Dùng `ref.listen` khi cần **side effects** (show snackbar, navigate, log).

### 3.4 Decision Tree — Chọn ref method nào?

```
Bạn cần gì?
│
├── Hiển thị data trong UI?
│   └── ✅ ref.watch (trong build)
│
├── Thực hiện action (button, gesture)?
│   └── ✅ ref.read (trong callback)
│
└── Phản ứng khi thay đổi (snackbar, navigate)?
    └── ✅ ref.listen (trong build hoặc initState)
```

### 3.5 ⚠️ Anti-patterns

```dart
class BadWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ❌ KHÔNG dùng ref.read trong build — sẽ không rebuild
    final count = ref.read(counterProvider);

    // ❌ KHÔNG dùng ref.watch trong callback — subscribe không cần thiết
    return ElevatedButton(
      onPressed: () {
        final value = ref.watch(counterProvider); // ❌ Sai!
      },
      child: Text('$count'),
    );
  }
}

class GoodWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ ref.watch trong build
    final count = ref.watch(counterProvider);

    return ElevatedButton(
      // ✅ ref.read trong callback
      onPressed: () => ref.read(counterProvider.notifier).state++,
      child: Text('$count'),
    );
  }
}
```

---

> 💼 **Gặp trong dự án:** Chọn sai ref method (watch/read/listen) gây bug UI không update hoặc rebuild quá nhiều, team member dùng ref.read trong build → state stale
> 🤖 **Keywords bắt buộc trong prompt:** `ref.watch vs ref.read`, `ConsumerWidget`, `ref.listen side effects`, `select filter`, `WidgetRef`, `Ref`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Bug thực tế:** Junior dùng `ref.read(counterProvider)` trong `build()` → UI không update khi counter thay đổi
- **Performance:** Senior review thấy widget rebuild khi bất kỳ field nào trong state đổi — cần `select` để filter
- **Side effects:** Show SnackBar khi API fail — dùng `ref.listen` thay vì `ref.watch` trong build

**Tại sao cần các keyword trên:**
- **`ref.watch vs ref.read`** — AI phải dùng `watch` trong build (reactive) và `read` trong callbacks (one-shot)
- **`ConsumerWidget`** — AI cần extend đúng base class, không dùng `StatelessWidget`
- **`ref.listen`** — cho side effects (snackbar, navigation), AI hay quên và dùng watch + if check thay thế (SAI)
- **`select filter`** — `ref.watch(userProvider.select((u) => u.name))` chỉ rebuild khi name đổi

**Prompt mẫu — Refactor ref usage:**
```text
Review và refactor widget dùng Riverpod sau — fix ref.watch/ref.read usage:
Tech stack: Flutter 3.x, flutter_riverpod ^2.x.

Code hiện tại (có bug):
class ProfileScreen extends ConsumerWidget {
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.read(userProvider);  // BUG 1: read trong build
    final theme = ref.watch(themeProvider);
    
    return Column(children: [
      Text(user.name),
      Text(user.email),
      ElevatedButton(
        onPressed: () {
          ref.watch(userProvider.notifier).updateName('New');  // BUG 2: watch trong callback
        },
        child: Text('Update'),
      ),
    ]);
  }
}

Yêu cầu:
1. Fix tất cả ref usage bugs.
2. Dùng ref.select cho Text(user.name) — chỉ rebuild khi name đổi.
3. Thêm ref.listen show SnackBar khi userProvider error.
4. Giải thích từng fix.
Output: code đã fix + bảng so sánh before/after.
```

**Expected Output:** AI gen code fixed + explanation table cho mỗi bug.

⚠️ **Giới hạn AI hay mắc:** AI hay fix đúng watch/read nhưng quên `select` optimization. AI cũng hay đặt `ref.listen` sai vị trí (trong initState thay vì trong build khi dùng ConsumerWidget).

</details>

> ⚠️ **FE Trap:** FE dev thường dùng `useSelector` (Redux) hoặc `useContext` cho mọi thứ. Riverpod **BẮT BUỘC** phân biệt: `ref.watch` = reactive rebuild (dùng trong `build()`), `ref.read` = one-time read (dùng trong event handlers), `ref.listen` = side effect khi state thay đổi. Dùng `ref.watch` trong event handler → unnecessary rebuild. Dùng `ref.read` trong `build()` → UI không update.

---

## 4. Modifiers 🟡

### 4.1 autoDispose — Tự động cleanup

Khi không còn widget nào listen provider → provider tự hủy và giải phóng resources.

```dart
// Cách 1: Manual
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  // Provider tự dispose khi widget unmount
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});

// Cách 2: Code-gen — mặc định đã autoDispose
@riverpod
Future<User> user(UserRef ref) async {
  // Tự động autoDispose khi dùng code-gen
  return fetchUser();
}
```

**Khi nào dùng autoDispose:**
- API calls — giải phóng khi rời trang
- Temporary state — filter, search query
- WebSocket connections — đóng khi không cần

**Giữ provider alive tạm thời:**

```dart
final myProvider = FutureProvider.autoDispose<Data>((ref) async {
  // Giữ provider alive thêm 30 giây sau khi mất listeners
  ref.keepAlive();

  // Hoặc cancel keepAlive sau timeout
  final link = ref.keepAlive();
  final timer = Timer(const Duration(seconds: 30), link.close);
  ref.onDispose(timer.cancel);

  return fetchData();
});
```

### 4.2 family — Provider có tham số

Tạo **nhiều instance** của cùng 1 provider dựa trên parameter.

```dart
// Provider nhận parameter userId
final userProvider =
    FutureProvider.autoDispose.family<User, String>((ref, userId) async {
  final response =
      await http.get(Uri.parse('https://api.example.com/users/$userId'));
  return User.fromJson(jsonDecode(response.body));
});

// Sử dụng — truyền ID
class UserDetailScreen extends ConsumerWidget {
  final String userId;
  const UserDetailScreen({required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId));

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

### 4.3 Kết hợp autoDispose + family

```dart
// autoDispose + family: tự dispose + nhận parameter
final productDetailProvider = FutureProvider.autoDispose
    .family<Product, int>((ref, productId) async {
  final repo = ref.watch(productRepositoryProvider);
  return repo.getProduct(productId);
});

// Code-gen approach — tự động autoDispose + family
@riverpod
Future<Product> productDetail(ProductDetailRef ref, int productId) async {
  final repo = ref.watch(productRepositoryProvider);
  return repo.getProduct(productId);
}
```

### 4.4 family với nhiều parameters

```dart
// Dùng Record (Dart 3)
final searchProvider = FutureProvider.autoDispose
    .family<List<Product>, ({String query, int page})>((ref, params) async {
  return searchProducts(query: params.query, page: params.page);
});

// Sử dụng
ref.watch(searchProvider((query: 'phone', page: 1)));
```

> 🔗 **FE Bridge:** `autoDispose` ≈ React `useEffect` cleanup — tự dispose khi không còn listener. `family` ≈ parameterized query key trong React Query (`useQuery(['user', id])`). Nhưng **khác ở**: Riverpod autoDispose = provider-level lifecycle, React useEffect = component-level lifecycle.

---

## 5. Testing với Riverpod 🟡

### 5.1 ProviderContainer cho unit tests

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  test('Counter starts at 0 and increments', () {
    // Tạo container — tương đương ProviderScope
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // Đọc giá trị ban đầu
    expect(container.read(counterProvider), 0);

    // Increment
    container.read(counterProvider.notifier).state++;

    // Kiểm tra
    expect(container.read(counterProvider), 1);
  });
}
```

### 5.2 Override providers trong tests

```dart
// Mock repository
class MockUserRepository implements UserRepository {
  @override
  Future<List<User>> getUsers() async {
    return [
      User(id: '1', name: 'Test User'),
    ];
  }
}

void main() {
  test('Users provider returns mock data', () async {
    final container = ProviderContainer(
      overrides: [
        // Override provider với mock
        userRepositoryProvider.overrideWithValue(MockUserRepository()),
      ],
    );
    addTearDown(container.dispose);

    // FutureProvider → cần await
    final users = await container.read(usersProvider.future);
    expect(users.length, 1);
    expect(users.first.name, 'Test User');
  });
}
```

### 5.3 Widget testing

```dart
testWidgets('Counter widget displays count', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        // Override cho widget test
        counterProvider.overrideWith((ref) => 42),
      ],
      child: const MaterialApp(
        home: CounterPage(),
      ),
    ),
  );

  expect(find.text('42'), findsOneWidget);
});
```

### 5.4 Testing Notifiers trực tiếp

```dart
test('TodosNotifier - add and toggle', () {
  final container = ProviderContainer();
  addTearDown(container.dispose);

  final notifier = container.read(todosProvider.notifier);

  // Add todo
  notifier.addTodo('Buy milk');
  expect(container.read(todosProvider).length, 1);
  expect(container.read(todosProvider).first.title, 'Buy milk');
  expect(container.read(todosProvider).first.isCompleted, false);

  // Toggle
  final todoId = container.read(todosProvider).first.id;
  notifier.toggleTodo(todoId);
  expect(container.read(todosProvider).first.isCompleted, true);
});
```

---

> 💼 **Gặp trong dự án:** Test Riverpod providers isolated (mock dependencies), widget test với ProviderScope overrides, async provider testing cần await .future
> 🤖 **Keywords bắt buộc trong prompt:** `ProviderContainer`, `overrideWithValue`, `overrideWith`, `ProviderScope overrides`, `AsyncValue`, `container.read vs container.listen`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Unit test:** Test NotifierProvider logic independent khỏi UI — dùng ProviderContainer
- **Mock API:** Override repository provider với MockRepository trong test
- **Widget test:** Test screen hiển thị data đúng khi provider trả về mock data

**Tại sao cần các keyword trên:**
- **`ProviderContainer`** — test container cho unit tests (không cần widget)
- **`overrideWithValue`** — inject mock trực tiếp, AI hay quên và tạo real instance
- **`ProviderScope overrides`** — cho widget tests, AI phải wrap trong ProviderScope
- **`AsyncValue`** — test FutureProvider/StreamProvider cần handle loading/data/error states

**Prompt mẫu — Gen test suite cho Riverpod:**
```text
Tôi cần viết test suite cho Weather App dùng Riverpod.
Tech stack: Flutter 3.x, flutter_riverpod ^2.x, flutter_test, mocktail.
Providers cần test:
1. weatherProvider (FutureProvider.family) — fetch weather by city name.
2. favoriteCitiesProvider (NotifierProvider) — add/remove/list favorite cities.
3. weatherRepositoryProvider — dependency injection point.
Constraints:
- Mock WeatherRepository dùng mocktail.
- Override weatherRepositoryProvider trong ProviderContainer.
- Test FutureProvider: loading state, data state, error state.
- Test Notifier: add city, remove city, duplicate check.
- Widget test: WeatherScreen hiển thị temperature khi data loaded.
- Tối thiểu 8 test cases.
Output: weather_provider_test.dart + weather_screen_test.dart.
```

**Expected Output:** AI gen 2 test files với 8+ test cases, mock repository, ProviderContainer usage.

⚠️ **Giới hạn AI hay mắc:** AI hay quên `addTearDown(container.dispose)` → memory leak trong tests. AI cũng hay dùng `container.read(asyncProvider)` trực tiếp thay vì `await container.read(asyncProvider.future)` cho FutureProvider.

</details>

> 🔗 **FE Bridge:** `ProviderContainer` override ≈ Jest `mockImplementation` + React Testing Library providers — nhưng **khác ở**: Riverpod override tại provider level, không cần wrapper component. Test isolation tốt hơn FE testing patterns.

---

## 6. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

| # | Practice | Lý do |
|---|----------|-------|
| 1 | Dùng `NotifierProvider` thay `StateProvider` cho project mới | Modern, flexible, dễ mở rộng |
| 2 | `ref.watch` trong `build`, `ref.read` trong callbacks | Tránh unnecessary rebuilds |
| 3 | Dùng `autoDispose` cho API calls | Giải phóng resources, tránh memory leak |
| 4 | Dùng `family` thay vì lưu param trong Notifier | Idiomatic Riverpod, dễ cache |
| 5 | Dùng `AsyncValue.when()` cho async data | Handle loading/error/data đầy đủ |
| 6 | Provider declarations ở top-level (global) | Riverpod design — providers là global variables |
| 7 | Code-gen (`@riverpod`) cho project mới | Ít boilerplate, ít bug, tự autoDispose |

### ❌ Lỗi thường gặp

#### Lỗi 1: Quên ProviderScope

```dart
// ❌ App crash
void main() {
  runApp(MyApp()); // Thiếu ProviderScope!
}

// ✅ Đúng
void main() {
  runApp(const ProviderScope(child: MyApp()));
}
```

#### Lỗi 2: ref.watch trong callback

```dart
// ❌ Sai — subscribe trong callback không cần thiết
onPressed: () {
  final count = ref.watch(counterProvider); // ❌
},

// ✅ Đúng
onPressed: () {
  ref.read(counterProvider.notifier).state++;
},
```

#### Lỗi 3: Mutate state trực tiếp

```dart
// ❌ Sai — mutate list trực tiếp
void addTodo(Todo todo) {
  state.add(todo); // Riverpod không detect thay đổi!
}

// ✅ Đúng — tạo list mới
void addTodo(Todo todo) {
  state = [...state, todo];
}
```

#### Lỗi 4: Không handle AsyncValue

```dart
// ❌ Sai — bỏ qua loading và error
final data = ref.watch(myFutureProvider).value; // value có thể null!

// ✅ Đúng — handle đầy đủ
ref.watch(myFutureProvider).when(
  data: (data) => Text('$data'),
  loading: () => const CircularProgressIndicator(),
  error: (err, stack) => Text('Error: $err'),
);
```

#### Lỗi 5: Quên `.notifier` khi read state

```dart
// ❌ Đọc state thay vì notifier
ref.read(counterProvider)++; // Lỗi — int không có ++

// ✅ Đúng
ref.read(counterProvider.notifier).state++;
// Hoặc với Notifier class:
ref.read(todosProvider.notifier).addTodo('New');
```

---

## 7. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Flutter/Riverpod Mindset | Tại sao khác |
|---|-------------------|--------------------------|--------------|
| 1 | Context Provider wrap component tree | Riverpod Provider là global, không cần wrap | Riverpod không dùng widget tree cho DI |
| 2 | `useSelector` / `useContext` = luôn subscribe | `ref.watch` vs `ref.read` — **phải chọn đúng** | Sai = unnecessary rebuild hoặc stale UI |
| 3 | State cleanup = `useEffect` return | `autoDispose` modifier — provider tự dispose | Lifecycle ở provider level, không phải component |
| 4 | Runtime error khi dependency sai | **Compile error** khi dependency sai | Riverpod = compile-time safety |
| 5 | React Query cache = key-based | Riverpod `.family` = parameter-based provider | Tương tự concept, khác implementation |

> Nếu bạn đến từ React/Vue, bảng so sánh dưới đây giúp bạn "map" kiến thức cũ sang Riverpod.

| Riverpod Concept | React Equivalent | Vue Equivalent |
|-----------------|-----------------|----------------|
| `Provider` | React Context + useMemo | computed property |
| `StateProvider` *(deprecated)* | useState | ref() |
| `NotifierProvider` | useReducer | Pinia store |
| `FutureProvider` | React Query useQuery | composable with async |
| `ref.watch` | useSelector (Redux) | computed |
| `ref.read` | dispatch (Redux) | store action call |
| `autoDispose` | cleanup in useEffect | onUnmounted |

> **Key insight:** Riverpod kết hợp mô hình data fetching của React Query với state management predictable của Redux. Nếu bạn biết React hooks, `ref.watch` / `ref.read` tương đương `useSelector` / `dispatch`.

Nếu bạn từng làm React hoặc Vue, đây là bảng mapping quen thuộc:

| Riverpod | React/Vue Equivalent | Giải thích |
|----------|---------------------|------------|
| `Provider` (read-only) | `useMemo` / `computed` | Giá trị derived, cached |
| `StateProvider` | `useState` đơn giản | State primitive (counter, bool) |
| `NotifierProvider` | Zustand store / Jotai atom | Complex mutable state với actions |
| `FutureProvider` | React Query / SWR `useQuery` | Fetch data, auto handle loading/error |
| `StreamProvider` | RxJS Observable + `useSubscription` | Realtime data stream |
| `ref.watch` | `useSyncExternalStore` / reactive dependency | Subscribe & re-render khi data thay đổi |
| `ref.read` | Direct store access (zustand `getState()`) | Đọc 1 lần, không subscribe |
| `ref.listen` | `useEffect` với dependency | Side effect khi data thay đổi |
| `autoDispose` | `useEffect` cleanup / React Query `gcTime` | Tự cleanup khi unmount |
| `family` | Factory pattern / dynamic query key | Parameterized data fetching |
| `ProviderScope` | React Context Provider / Vue `provide` | DI container cho toàn app |
| `ProviderContainer` | Testing utilities | Test isolation |

### So sánh cụ thể

**FutureProvider vs React Query:**

```dart
// Riverpod FutureProvider
final todosProvider = FutureProvider.autoDispose<List<Todo>>((ref) async {
  return fetchTodos(); // Auto loading/error handling
});

// Tương đương React Query
// const { data, isLoading, error } = useQuery(['todos'], fetchTodos);
```

**autoDispose vs useEffect cleanup:**

```dart
// Riverpod autoDispose → tự cleanup khi rời trang
final wsProvider = StreamProvider.autoDispose<Message>((ref) {
  final ws = WebSocket('ws://...');
  ref.onDispose(() => ws.close()); // cleanup
  return ws.stream;
});

// Tương đương React
// useEffect(() => {
//   const ws = new WebSocket('ws://...');
//   return () => ws.close(); // cleanup
// }, []);
```

---

## 8. Tổng kết

### Checklist kiến thức buổi 07

| # | Nội dung | Tự đánh giá |
|---|----------|-------------|
| 1 | Giải thích được tại sao Riverpod tốt hơn Provider | ⬜ |
| 2 | Sử dụng được `ProviderScope` và `ConsumerWidget` | ⬜ |
| 3 | Phân biệt được Provider, NotifierProvider, FutureProvider, StreamProvider | ⬜ |
| 4 | Dùng đúng `ref.watch`, `ref.read`, `ref.listen` | ⬜ |
| 5 | Áp dụng được `autoDispose` và `family` | ⬜ |
| 6 | Viết được provider bằng code-gen (`@riverpod`) | ⬜ |
| 7 | Viết được unit test cho Riverpod providers | ⬜ |
| 8 | Handle `AsyncValue` đầy đủ (loading/error/data) | ⬜ |

### Công thức ghi nhớ

```
Provider Type = Loại data bạn quản lý
  ├── Sync read-only     → Provider
  ├── Sync mutable       → NotifierProvider ⭐
  ├── Async one-shot     → FutureProvider
  ├── Async stream       → StreamProvider
  └── Async mutable      → AsyncNotifierProvider ⭐

ref method = Cách bạn đọc data
  ├── Cần rebuild UI     → ref.watch (trong build)
  ├── Cần action 1 lần   → ref.read (trong callback)
  └── Cần side effect    → ref.listen (show snackbar, navigate)

Modifier = Tùy chỉnh behavior
  ├── Tự cleanup         → autoDispose
  └── Có parameter       → family
```

### 🔑 Key takeaways

- Riverpod = compile-safe, testable, context-free state management
- Luôn dùng `ref.watch` trong `build()`, `ref.read` trong callbacks
- `autoDispose` là default — chỉ `keepAlive` khi thật sự cần cache
- Provider là đơn vị nhỏ nhất — mỗi provider chỉ quản lý 1 concern
- Testing dễ dàng vì provider independent với widget tree

### Buổi tiếp theo

**Buổi 08: BLoC Pattern** — event-driven state management, Cubit vs Bloc, khi nào dùng BLoC thay Riverpod.
