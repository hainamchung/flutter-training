# Buổi 08: BLoC Pattern — Ví dụ minh họa

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> 💡 Tất cả ví dụ đều là Flutter app hoàn chỉnh, có thể copy-paste và chạy.

---

## VD1: Counter với Cubit (Đơn giản nhất) 🔴

> 📖 **Liên quan:** [Phần 3.1 — Cubit — Phiên bản đơn giản](01-ly-thuyet.md#31-cubit--phiên-bản-đơn-giản)

> **Liên quan tới:** [3. Cubit vs Bloc 🔴](01-ly-thuyet.md#3-cubit-vs-bloc)

### Cubit

```dart
// lib/counter/counter_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}
```

### UI

```dart
// lib/counter/counter_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_cubit.dart';

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const CounterView(),
    );
  }
}

class CounterView extends StatelessWidget {
  const CounterView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter Cubit')),
      body: Center(
        child: BlocBuilder<CounterCubit, int>(
          builder: (context, count) {
            return Text(
              '$count',
              style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
            );
          },
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            heroTag: 'increment',
            onPressed: () => context.read<CounterCubit>().increment(),
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'decrement',
            onPressed: () => context.read<CounterCubit>().decrement(),
            child: const Icon(Icons.remove),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'reset',
            onPressed: () => context.read<CounterCubit>().reset(),
            child: const Icon(Icons.refresh),
          ),
        ],
      ),
    );
  }
}
```

### main.dart

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'counter/counter_page.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter Cubit Demo',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const CounterPage(),
    );
  }
}
```

**Điểm học được:**
- `CounterCubit extends Cubit<int>` — state type là `int`
- Gọi `emit(newValue)` để cập nhật state
- `BlocProvider` cung cấp Cubit cho widget tree
- `BlocBuilder` lắng nghe và rebuild khi state thay đổi
- `context.read<CounterCubit>()` lấy instance trong callback
- 🔗 **FE tương đương:** Tương tự Redux `dispatch({ type: 'INCREMENT' })` → reducer → new state — BLoC mapping gần 1:1 với Redux pattern.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_cubit_counter
cd vidu_cubit_counter
# Thêm dependency
flutter pub add flutter_bloc
# Tạo file lib/counter/counter_cubit.dart và lib/counter/counter_page.dart
# Cập nhật lib/main.dart theo code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Số lớn ở giữa màn hình hiển thị count (ban đầu = 0)
✅ 3 FloatingActionButton: +, −, reset
✅ Nhấn + → count tăng 1, nhấn − → count giảm 1, nhấn reset → về 0
```

---

## VD2: Counter với Bloc (Event-driven) 🔴

> **Liên quan tới:** [2. Events, States, Bloc Class 🔴](01-ly-thuyet.md#2-events-states-bloc-class)

### Events & States

```dart
// lib/counter_bloc/counter_event.dart
sealed class CounterEvent {}

class IncrementPressed extends CounterEvent {}

class DecrementPressed extends CounterEvent {}

class ResetPressed extends CounterEvent {}
```

```dart
// lib/counter_bloc/counter_state.dart
import 'package:equatable/equatable.dart';

class CounterState extends Equatable {
  final int count;
  const CounterState({this.count = 0});

  @override
  List<Object?> get props => [count];
}
```

### Bloc

```dart
// lib/counter_bloc/counter_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_event.dart';
import 'counter_state.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState()) {
    on<IncrementPressed>(_onIncrement);
    on<DecrementPressed>(_onDecrement);
    on<ResetPressed>(_onReset);
  }

  void _onIncrement(IncrementPressed event, Emitter<CounterState> emit) {
    emit(CounterState(count: state.count + 1));
  }

  void _onDecrement(DecrementPressed event, Emitter<CounterState> emit) {
    emit(CounterState(count: state.count - 1));
  }

  void _onReset(ResetPressed event, Emitter<CounterState> emit) {
    emit(const CounterState());
  }
}
```

### UI

```dart
// lib/counter_bloc/counter_bloc_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_bloc.dart';
import 'counter_event.dart';
import 'counter_state.dart';

class CounterBlocPage extends StatelessWidget {
  const CounterBlocPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterBloc(),
      child: const CounterBlocView(),
    );
  }
}

class CounterBlocView extends StatelessWidget {
  const CounterBlocView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter Bloc')),
      body: Center(
        child: BlocBuilder<CounterBloc, CounterState>(
          builder: (context, state) {
            return Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text('Count:', style: TextStyle(fontSize: 20)),
                Text(
                  '${state.count}',
                  style: const TextStyle(
                    fontSize: 48,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            );
          },
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            heroTag: 'bloc_inc',
            onPressed: () =>
                context.read<CounterBloc>().add(IncrementPressed()),
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'bloc_dec',
            onPressed: () =>
                context.read<CounterBloc>().add(DecrementPressed()),
            child: const Icon(Icons.remove),
          ),
          const SizedBox(height: 8),
          FloatingActionButton(
            heroTag: 'bloc_reset',
            onPressed: () => context.read<CounterBloc>().add(ResetPressed()),
            child: const Icon(Icons.refresh),
          ),
        ],
      ),
    );
  }
}
```

**So sánh với VD1 (Cubit):**
- Cubit: `context.read<CounterCubit>().increment()` — gọi method trực tiếp
- Bloc: `context.read<CounterBloc>().add(IncrementPressed())` — dispatch Event
- Bloc cần thêm Event classes, nhưng mọi thay đổi đều traceable qua Event

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §2.1 Events — Hành động từ user](01-ly-thuyet.md#21-events--hành-động-từ-user) · [§2.4 Bloc Class — Trung tâm xử lý](01-ly-thuyet.md#24-bloc-class--trung-tâm-xử-lý)

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_counter_bloc
cd vidu_counter_bloc
# Thêm dependencies
flutter pub add flutter_bloc equatable
# Tạo các file: lib/counter_bloc/counter_event.dart, counter_state.dart,
# counter_bloc.dart, counter_bloc_page.dart
# Cập nhật lib/main.dart để import CounterBlocPage, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Số lớn ở giữa hiển thị "Count:" và giá trị (ban đầu = 0)
✅ 3 FloatingActionButton: +, −, reset
✅ Nhấn + → dispatch IncrementPressed → count tăng 1
✅ Nhấn − → dispatch DecrementPressed → count giảm 1
✅ Nhấn reset → dispatch ResetPressed → count về 0
```

---

## VD3: BlocBuilder + BlocListener — Snackbar khi state thay đổi 🟡

> 📖 **Liên quan:** [Phần 4.4 — BlocConsumer](01-ly-thuyet.md#44-blocconsumer--builder--listener-kết-hợp) · [Phần 4.3 — BlocListener](01-ly-thuyet.md#43-bloclistener--side-effects-khi-state-thay-đổi)

> **Liên quan tới:** [4. flutter_bloc Widgets 🟡](01-ly-thuyet.md#4-flutter_bloc-widgets)

```dart
// lib/counter_listener/counter_listener_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// --- Cubit ---
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

// --- Page ---
class CounterListenerPage extends StatelessWidget {
  const CounterListenerPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const CounterListenerView(),
    );
  }
}

class CounterListenerView extends StatelessWidget {
  const CounterListenerView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('BlocListener Demo')),
      // BlocConsumer = BlocBuilder + BlocListener
      body: BlocConsumer<CounterCubit, int>(
        // Listener: side effects
        listener: (context, count) {
          final message = count > 0
              ? '🎉 Count tăng lên $count!'
              : count < 0
                  ? '📉 Count giảm xuống $count'
                  : '🔄 Count đã reset';

          ScaffoldMessenger.of(context)
            ..hideCurrentSnackBar()
            ..showSnackBar(
              SnackBar(
                content: Text(message),
                duration: const Duration(seconds: 1),
                behavior: SnackBarBehavior.floating,
              ),
            );
        },
        // Builder: rebuild UI
        builder: (context, count) {
          final color = count > 0
              ? Colors.green
              : count < 0
                  ? Colors.red
                  : Colors.grey;

          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  count >= 0 ? Icons.trending_up : Icons.trending_down,
                  size: 64,
                  color: color,
                ),
                const SizedBox(height: 16),
                Text(
                  '$count',
                  style: TextStyle(
                    fontSize: 72,
                    fontWeight: FontWeight.bold,
                    color: color,
                  ),
                ),
              ],
            ),
          );
        },
      ),
      floatingActionButton: Row(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            heroTag: 'listener_dec',
            onPressed: () => context.read<CounterCubit>().decrement(),
            child: const Icon(Icons.remove),
          ),
          const SizedBox(width: 8),
          FloatingActionButton(
            heroTag: 'listener_inc',
            onPressed: () => context.read<CounterCubit>().increment(),
            child: const Icon(Icons.add),
          ),
        ],
      ),
    );
  }
}
```

**Điểm học được:**
- `BlocConsumer` kết hợp **BlocBuilder** (rebuild UI) + **BlocListener** (side effect)
- Listener hiển thị SnackBar — đây là side effect, không phải rebuild
- Builder thay đổi màu sắc và icon — đây là UI rebuild
- SnackBar, Navigation, Dialog → luôn dùng **Listener**, không phải Builder
- 🔗 **FE tương đương:** `BlocBuilder` ≈ `useSelector` (react to state), `BlocListener` ≈ `useEffect` on state change (side effects) — Flutter tách rõ UI rebuild vs side effect.

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_bloc_listener
cd vidu_bloc_listener
# Thêm dependency
flutter pub add flutter_bloc
# Thay nội dung lib/main.dart bằng toàn bộ code trên, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ Số lớn ở giữa với icon trending up/down, màu thay đổi theo giá trị
✅ Nhấn + → count tăng (xanh), nhấn − → count giảm (đỏ)
✅ Mỗi lần thay đổi → SnackBar floating hiện message "🎉 Count tăng lên X!" hoặc "📉 Count giảm xuống X"
```

---

## VD4: Complete Todo Bloc 🟡

> 📖 **Liên quan:** [Phần 2.4 — Bloc Class — Trung tâm xử lý](01-ly-thuyet.md#24-bloc-class--trung-tâm-xử-lý) · [Phần 4.2 — BlocBuilder](01-ly-thuyet.md#42-blocbuilder--rebuild-ui-khi-state-thay-đổi)

> **Liên quan tới:** [2. Events, States, Bloc Class 🔴](01-ly-thuyet.md#2-events-states-bloc-class)

### Model

```dart
// lib/todo/models/todo.dart
import 'package:equatable/equatable.dart';

class Todo extends Equatable {
  final String id;
  final String title;
  final bool isCompleted;

  const Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  Todo copyWith({String? title, bool? isCompleted}) {
    return Todo(
      id: id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }

  @override
  List<Object?> get props => [id, title, isCompleted];
}
```

### Events

```dart
// lib/todo/bloc/todo_event.dart
sealed class TodoEvent {}

class TodoItemAdded extends TodoEvent {
  final String title;
  TodoItemAdded(this.title);
}

class TodoItemToggled extends TodoEvent {
  final String id;
  TodoItemToggled(this.id);
}

class TodoItemDeleted extends TodoEvent {
  final String id;
  TodoItemDeleted(this.id);
}
```

### States

```dart
// lib/todo/bloc/todo_state.dart
import 'package:equatable/equatable.dart';
import '../models/todo.dart';

sealed class TodoState extends Equatable {
  @override
  List<Object?> get props => [];
}

class TodoInitial extends TodoState {}

class TodoLoaded extends TodoState {
  final List<Todo> todos;

  TodoLoaded({this.todos = const []});

  int get completedCount => todos.where((t) => t.isCompleted).length;
  int get pendingCount => todos.where((t) => !t.isCompleted).length;

  @override
  List<Object?> get props => [todos];
}
```

### Bloc

```dart
// lib/todo/bloc/todo_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/todo.dart';
import 'todo_event.dart';
import 'todo_state.dart';

class TodoBloc extends Bloc<TodoEvent, TodoState> {
  TodoBloc() : super(TodoLoaded()) {
    on<TodoItemAdded>(_onAdded);
    on<TodoItemToggled>(_onToggled);
    on<TodoItemDeleted>(_onDeleted);
  }

  void _onAdded(TodoItemAdded event, Emitter<TodoState> emit) {
    final currentState = state;
    if (currentState is TodoLoaded) {
      final newTodo = Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: event.title,
      );
      emit(TodoLoaded(todos: [...currentState.todos, newTodo]));
    }
  }

  void _onToggled(TodoItemToggled event, Emitter<TodoState> emit) {
    final currentState = state;
    if (currentState is TodoLoaded) {
      final updatedTodos = currentState.todos.map((todo) {
        return todo.id == event.id
            ? todo.copyWith(isCompleted: !todo.isCompleted)
            : todo;
      }).toList();
      emit(TodoLoaded(todos: updatedTodos));
    }
  }

  void _onDeleted(TodoItemDeleted event, Emitter<TodoState> emit) {
    final currentState = state;
    if (currentState is TodoLoaded) {
      final updatedTodos =
          currentState.todos.where((todo) => todo.id != event.id).toList();
      emit(TodoLoaded(todos: updatedTodos));
    }
  }
}
```

### UI

```dart
// lib/todo/todo_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'bloc/todo_bloc.dart';
import 'bloc/todo_event.dart';
import 'bloc/todo_state.dart';

class TodoPage extends StatelessWidget {
  const TodoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => TodoBloc(),
      child: const TodoView(),
    );
  }
}

class TodoView extends StatelessWidget {
  const TodoView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo BLoC'),
        actions: [
          // Hiển thị count
          BlocBuilder<TodoBloc, TodoState>(
            builder: (context, state) {
              if (state is TodoLoaded) {
                return Center(
                  child: Padding(
                    padding: const EdgeInsets.only(right: 16),
                    child: Text(
                      '${state.completedCount}/${state.todos.length}',
                      style: const TextStyle(fontSize: 16),
                    ),
                  ),
                );
              }
              return const SizedBox.shrink();
            },
          ),
        ],
      ),
      body: BlocBuilder<TodoBloc, TodoState>(
        builder: (context, state) {
          if (state is TodoLoaded) {
            if (state.todos.isEmpty) {
              return const Center(
                child: Text(
                  'Chưa có todo nào.\nNhấn + để thêm!',
                  textAlign: TextAlign.center,
                  style: TextStyle(fontSize: 18, color: Colors.grey),
                ),
              );
            }

            return ListView.builder(
              itemCount: state.todos.length,
              itemBuilder: (context, index) {
                final todo = state.todos[index];
                return Dismissible(
                  key: Key(todo.id),
                  direction: DismissDirection.endToStart,
                  background: Container(
                    color: Colors.red,
                    alignment: Alignment.centerRight,
                    padding: const EdgeInsets.only(right: 16),
                    child: const Icon(Icons.delete, color: Colors.white),
                  ),
                  onDismissed: (_) {
                    context.read<TodoBloc>().add(TodoItemDeleted(todo.id));
                  },
                  child: ListTile(
                    leading: Checkbox(
                      value: todo.isCompleted,
                      onChanged: (_) {
                        context
                            .read<TodoBloc>()
                            .add(TodoItemToggled(todo.id));
                      },
                    ),
                    title: Text(
                      todo.title,
                      style: TextStyle(
                        decoration: todo.isCompleted
                            ? TextDecoration.lineThrough
                            : null,
                        color: todo.isCompleted ? Colors.grey : null,
                      ),
                    ),
                  ),
                );
              },
            );
          }
          return const Center(child: CircularProgressIndicator());
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context) {
    final controller = TextEditingController();

    showDialog(
      context: context,
      builder: (dialogContext) {
        return AlertDialog(
          title: const Text('Thêm Todo'),
          content: TextField(
            controller: controller,
            autofocus: true,
            decoration: const InputDecoration(
              hintText: 'Nhập tên todo...',
              border: OutlineInputBorder(),
            ),
            onSubmitted: (value) {
              if (value.trim().isNotEmpty) {
                context.read<TodoBloc>().add(TodoItemAdded(value.trim()));
                Navigator.pop(dialogContext);
              }
            },
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(dialogContext),
              child: const Text('Hủy'),
            ),
            FilledButton(
              onPressed: () {
                final text = controller.text.trim();
                if (text.isNotEmpty) {
                  context.read<TodoBloc>().add(TodoItemAdded(text));
                  Navigator.pop(dialogContext);
                }
              },
              child: const Text('Thêm'),
            ),
          ],
        );
      },
    );
  }
}
```

**Luồng dữ liệu VD4:**

```
User nhấn FAB (+)
  → showDialog, nhập title
  → bloc.add(TodoItemAdded(title))
  → _onAdded handler
  → emit(TodoLoaded(updatedList))
  → BlocBuilder rebuild ListView

User swipe todo
  → Dismissible onDismissed
  → bloc.add(TodoItemDeleted(id))
  → _onDeleted handler
  → emit(TodoLoaded(filteredList))
  → BlocBuilder rebuild ListView

User tap checkbox
  → bloc.add(TodoItemToggled(id))
  → _onToggled handler
  → emit(TodoLoaded(toggledList))
  → BlocBuilder rebuild ListTile
```

### ▶️ Chạy ví dụ

```bash
# Tạo project (nếu chưa có)
flutter create vidu_todo_bloc
cd vidu_todo_bloc
# Thêm dependencies
flutter pub add flutter_bloc equatable
# Tạo các file: lib/todo/models/todo.dart, lib/todo/bloc/todo_event.dart,
# lib/todo/bloc/todo_state.dart, lib/todo/bloc/todo_bloc.dart, lib/todo/todo_page.dart
# Cập nhật lib/main.dart để import TodoPage, rồi:
flutter run
```

### 📋 Kết quả mong đợi

```
✅ AppBar hiện "Todo BLoC" với counter completed/total bên phải
✅ Nhấn FAB (+) → dialog nhập tên todo → thêm vào list
✅ Tap checkbox → todo gạch ngang (completed), counter cập nhật
✅ Swipe todo sang trái → xóa todo (Dismissible animation)
```

---

## VD5: Unit Testing TodoBloc với bloc_test 🟢

> **Liên quan tới:** [5. Testing BLoC 🟡](01-ly-thuyet.md#5-testing-bloc)

```dart
// test/todo/bloc/todo_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';

// Import các file từ VD4
import 'package:my_app/todo/bloc/todo_bloc.dart';
import 'package:my_app/todo/bloc/todo_event.dart';
import 'package:my_app/todo/bloc/todo_state.dart';
import 'package:my_app/todo/models/todo.dart';

void main() {
  group('TodoBloc', () {
    // ---- Test initial state ----
    test('initial state is TodoLoaded with empty list', () {
      final bloc = TodoBloc();
      expect(bloc.state, isA<TodoLoaded>());
      expect((bloc.state as TodoLoaded).todos, isEmpty);
      bloc.close();
    });

    // ---- Test TodoItemAdded ----
    blocTest<TodoBloc, TodoState>(
      'emits TodoLoaded with 1 todo when TodoItemAdded is added',
      build: () => TodoBloc(),
      act: (bloc) => bloc.add(TodoItemAdded('Buy milk')),
      expect: () => [
        isA<TodoLoaded>().having(
          (s) => s.todos.length,
          'todos length',
          1,
        ),
      ],
      verify: (bloc) {
        final state = bloc.state as TodoLoaded;
        expect(state.todos.first.title, 'Buy milk');
        expect(state.todos.first.isCompleted, false);
      },
    );

    blocTest<TodoBloc, TodoState>(
      'emits TodoLoaded with 2 todos when 2 TodoItemAdded events',
      build: () => TodoBloc(),
      act: (bloc) {
        bloc.add(TodoItemAdded('Buy milk'));
        bloc.add(TodoItemAdded('Walk the dog'));
      },
      expect: () => [
        isA<TodoLoaded>().having((s) => s.todos.length, 'length', 1),
        isA<TodoLoaded>().having((s) => s.todos.length, 'length', 2),
      ],
    );

    // ---- Test TodoItemToggled ----
    blocTest<TodoBloc, TodoState>(
      'toggles todo completion status',
      build: () => TodoBloc(),
      seed: () => TodoLoaded(
        todos: [const Todo(id: '1', title: 'Test', isCompleted: false)],
      ),
      act: (bloc) => bloc.add(TodoItemToggled('1')),
      expect: () => [
        isA<TodoLoaded>().having(
          (s) => s.todos.first.isCompleted,
          'isCompleted',
          true,
        ),
      ],
    );

    blocTest<TodoBloc, TodoState>(
      'toggle twice returns to original state',
      build: () => TodoBloc(),
      seed: () => TodoLoaded(
        todos: [const Todo(id: '1', title: 'Test', isCompleted: false)],
      ),
      act: (bloc) {
        bloc.add(TodoItemToggled('1'));
        bloc.add(TodoItemToggled('1'));
      },
      expect: () => [
        isA<TodoLoaded>().having(
          (s) => s.todos.first.isCompleted,
          'isCompleted after first toggle',
          true,
        ),
        isA<TodoLoaded>().having(
          (s) => s.todos.first.isCompleted,
          'isCompleted after second toggle',
          false,
        ),
      ],
    );

    // ---- Test TodoItemDeleted ----
    blocTest<TodoBloc, TodoState>(
      'removes todo when TodoItemDeleted is added',
      build: () => TodoBloc(),
      seed: () => TodoLoaded(
        todos: [
          const Todo(id: '1', title: 'Todo 1'),
          const Todo(id: '2', title: 'Todo 2'),
        ],
      ),
      act: (bloc) => bloc.add(TodoItemDeleted('1')),
      expect: () => [
        isA<TodoLoaded>()
            .having((s) => s.todos.length, 'length', 1)
            .having((s) => s.todos.first.id, 'remaining id', '2'),
      ],
    );

    blocTest<TodoBloc, TodoState>(
      'emits empty TodoLoaded when last todo is deleted',
      build: () => TodoBloc(),
      seed: () => TodoLoaded(
        todos: [const Todo(id: '1', title: 'Only todo')],
      ),
      act: (bloc) => bloc.add(TodoItemDeleted('1')),
      expect: () => [
        isA<TodoLoaded>().having((s) => s.todos, 'todos', isEmpty),
      ],
    );

    // ---- Test computed properties ----
    test('completedCount and pendingCount are correct', () {
      final state = TodoLoaded(
        todos: [
          const Todo(id: '1', title: 'Done', isCompleted: true),
          const Todo(id: '2', title: 'Not done', isCompleted: false),
          const Todo(id: '3', title: 'Also done', isCompleted: true),
        ],
      );
      expect(state.completedCount, 2);
      expect(state.pendingCount, 1);
    });
  });
}
```

**Giải thích blocTest:**

```dart
blocTest<TodoBloc, TodoState>(
  'description',              // ① Tên test case
  build: () => TodoBloc(),    // ② Tạo bloc instance
  seed: () => TodoLoaded(...),// ③ (Optional) Set state ban đầu
  act: (bloc) => bloc.add(   // ④ Dispatch event / gọi method
    TodoItemAdded('test'),
  ),
  expect: () => [             // ⑤ Danh sách states kỳ vọng (theo thứ tự)
    isA<TodoLoaded>(),
  ],
  verify: (bloc) {            // ⑥ (Optional) Kiểm tra thêm sau khi chạy
    // ...
  },
);
```

**Điểm học được:**
- `blocTest` là cách chuẩn để test BLoC — declare: build → act → expect
- `seed` cho phép set state ban đầu (thay vì add event để warm up)
- `isA<Type>().having()` cho phép kiểm tra chi tiết type + property
- Test isolated — không cần Flutter widget, chỉ test pure logic
- Mỗi test case tạo bloc mới → không ảnh hưởng lẫn nhau
- 🔗 **FE tương đương:** Pattern giống Redux test: `given(initialState) → when(dispatch(action)) → then(expect(newState))` — `blocTest` API gần tương đương.

### 📖 Liên quan

> Xem lý thuyết: [01-ly-thuyet.md - §5.1 bloc_test package](01-ly-thuyet.md#51-bloc_test-package) · [§5.2 Testing Bloc (event-driven)](01-ly-thuyet.md#52-testing-bloc-event-driven)

### ▶️ Chạy ví dụ

```bash
# Trong project từ VD4 (vidu_todo_bloc)
# Thêm dependency test
flutter pub add dev:bloc_test
# Tạo file test/todo/bloc/todo_bloc_test.dart với code trên
# Chạy tests:
flutter test test/todo/bloc/todo_bloc_test.dart
```

### 📋 Kết quả mong đợi

```
✅ 7 tests pass: initial state, add 1 todo, add 2 todos,
   toggle, double toggle, delete, delete last
✅ completedCount và pendingCount tính đúng
✅ Tất cả tests chạy < 1 giây (pure logic, không cần Flutter engine)
```

---

## VD6: 🤖 AI Gen → Review — BLoC Events & States 🟢

> **Mục đích:** Luyện workflow "AI gen BLoC boilerplate → bạn review sealed/Equatable → fix issues"

> **Liên quan tới:** [2. Events, States, Bloc Class 🔴](01-ly-thuyet.md#2-events-states-bloc-class)

### Bước 1: Prompt cho AI

```text
Tạo BLoC cho Todo feature trong Flutter.
Events: AddTodo(title), ToggleTodo(id), DeleteTodo(id), LoadTodos.
States: TodoState(status, todos, errorMessage) dùng Equatable.
Bloc: TodoBloc xử lý tất cả events, emit state mới.
Output: todo_event.dart + todo_state.dart + todo_bloc.dart.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **sealed class** | Events dùng `sealed class` hay `abstract class`? (abstract = không exhaustive) |
| 2 | **Equatable props** | State override `props` đủ tất cả fields? (thiếu field → compare sai) |
| 3 | **copyWith usage** | `emit(state.copyWith(...))` giữ field cũ? Hay mất field? |
| 4 | **emit location** | `emit` chỉ trong `on<Event>` handler? Gọi ngoài handler = crash runtime |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Abstract class thay vì sealed class
abstract class TodoEvent {}  // WRONG — không exhaustive
class AddTodo extends TodoEvent { ... }

// ✅ FIX: Sealed class (Dart 3)
sealed class TodoEvent {}
class AddTodo extends TodoEvent { final String title; ... }
// → switch(event) phải handle tất cả cases, compiler báo nếu thiếu
```

```dart
// ❌ LỖI 2: Equatable props thiếu field
class TodoState extends Equatable {
  final TodoStatus status;
  final List<Todo> todos;
  final String? errorMessage;
  
  @override
  List<Object?> get props => [status]; // WRONG — thiếu todos và errorMessage!
  // → State có todos khác nhau nhưng Equatable nói "giống" → UI không rebuild
}

// ✅ FIX: Props list đủ tất cả fields
@override
List<Object?> get props => [status, todos, errorMessage];
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết sealed class cho Events giúp exhaustive pattern matching (Dart 3)
- ✅ Hiểu tầm quan trọng của Equatable props list đầy đủ
- ✅ Phân biệt khi nào AI gen đúng/sai BLoC boilerplate
