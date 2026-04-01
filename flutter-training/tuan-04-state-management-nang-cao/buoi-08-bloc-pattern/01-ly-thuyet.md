# Buổi 08: BLoC Pattern — Lý Thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 8/16** · **Thời lượng tự học:** ~2 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 06 (parallel với Buổi 07 nếu muốn)

## 1. BLoC là gì? 🔴

### 1.1 Định nghĩa

**BLoC** = **B**usiness **Lo**gic **C**omponent

BLoC là một **design pattern** được các kỹ sư Google giới thiệu tại DartConf 2018. Mục tiêu:
- **Tách biệt hoàn toàn** UI khỏi business logic
- **Unidirectional data flow** — luồng dữ liệu một chiều, dễ dự đoán
- **Testable** — logic nằm ngoài widget, test dễ dàng
- **Platform-independent** — cùng BLoC có thể dùng cho Flutter, Angular Dart, web

### 1.2 Luồng dữ liệu một chiều

```
┌──────────────────────────────────────────────────┐
│                                                  │
│   UI (Widget)                                    │
│     │                                            │
│     │ ① User action → add Event                  │
│     ▼                                            │
│   Event ──────▶ Bloc ──────▶ State               │
│                  │             │                  │
│                  │ ② Process   │ ③ Emit new       │
│                  │   logic     │   state           │
│                  │             ▼                  │
│                              UI rebuilds          │
│                              (BlocBuilder)        │
│                                                  │
└──────────────────────────────────────────────────┘
```

**Giải thích:**
1. **User tương tác** → UI dispatch một **Event** vào Bloc
2. **Bloc xử lý logic** → dựa trên Event, chạy business logic (API call, validation...)
3. **Emit State mới** → UI listen State, tự rebuild khi State thay đổi

### 1.3 Tại sao BLoC phổ biến?

| Đặc điểm | Giải thích |
|-----------|------------|
| Predictable | Cùng Event → cùng State, dễ debug |
| Traceable | Mọi thay đổi đều qua Event, dễ log |
| Testable | Business logic tách biệt, test không cần UI |
| Scalable | Pattern chuẩn cho enterprise, team lớn |
| Ecosystem | `flutter_bloc`, `bloc_test`, `hydrated_bloc`, `replay_bloc` |

> 💡 BLoC đã phát triển từ pattern ban đầu thành **standard enterprise pattern** trong hệ sinh thái Flutter. Nhiều công ty lớn sử dụng BLoC cho production apps.

> 🔗 **FE Bridge:** BLoC (Business Logic Component) ≈ Redux pattern — cùng triết lý **unidirectional data flow**: UI dispatch Event → BLoC xử lý → emit State → UI rebuild. Nhưng **khác ở**: BLoC dùng `Stream` thay vì store subscription, và mỗi BLoC là **isolated unit** thay vì một global store.

---

## 2. Events, States, Bloc Class 🔴

### 2.1 Events — Hành động từ user

**Event** đại diện cho những gì **đã xảy ra** (past tense): người dùng nhấn nút, data load xong, form thay đổi.

```dart
// ✅ Dùng sealed class (Dart 3+)
sealed class CounterEvent {}

class IncrementPressed extends CounterEvent {}

class DecrementPressed extends CounterEvent {}

class ResetPressed extends CounterEvent {}
```

**Nguyên tắc đặt tên Event:**
- Dùng **past tense** hoặc mô tả hành động: `ButtonPressed`, `DataLoaded`, `TextChanged`
- Mỗi Event là một class riêng — rõ ràng, dễ handle
- Dùng `sealed class` để compiler kiểm tra exhaustive switch

```dart
// Event có data
sealed class TodoEvent {}

class TodoAdded extends TodoEvent {
  final String title;
  TodoAdded(this.title);
}

class TodoToggled extends TodoEvent {
  final String id;
  TodoToggled(this.id);
}

class TodoDeleted extends TodoEvent {
  final String id;
  TodoDeleted(this.id);
}
```

### 2.2 States — Trạng thái UI

**State** đại diện cho **trạng thái hiện tại** của UI tại một thời điểm.

```dart
// ✅ Sealed class cho States
sealed class CounterState {
  final int count;
  const CounterState(this.count);
}

class CounterInitial extends CounterState {
  const CounterInitial() : super(0);
}

class CounterUpdated extends CounterState {
  const CounterUpdated(super.count);
}
```

**Pattern phổ biến cho async states:**

```dart
sealed class TodoState {}

class TodoInitial extends TodoState {}

class TodoLoading extends TodoState {}

class TodoLoaded extends TodoState {
  final List<Todo> todos;
  TodoLoaded(this.todos);
}

class TodoError extends TodoState {
  final String message;
  TodoError(this.message);
}
```

> 🎯 **Pattern I-L-S-E** (Initial → Loading → Success → Error) là pattern state phổ biến nhất trong BLoC.

### 2.3 Equatable — So sánh State

Mặc định, Dart so sánh object bằng reference (`identical`). Để BLoC biết khi nào State thực sự thay đổi, dùng `equatable`:

> 📌 **Convention: `extends Equatable` vs `with EquatableMixin`**
>
> - **`extends Equatable`** — Dùng làm **mặc định** cho BLoC states/events. API sạch hơn, ít boilerplate.
> - **`with EquatableMixin`** — Dùng khi class **đã extends class khác** (ví dụ: `extends Exception`, `extends SomeBaseClass`), vì Dart không hỗ trợ multiple inheritance.
>
> Trong tài liệu này, chúng ta thống nhất dùng `extends Equatable` cho states/events.

```dart
import 'package:equatable/equatable.dart';

sealed class TodoState extends Equatable {
  @override
  List<Object?> get props => [];
}

class TodoLoaded extends TodoState {
  final List<Todo> todos;
  TodoLoaded(this.todos);

  @override
  List<Object?> get props => [todos];
}
```

**Tại sao cần Equatable?**
- Không có → BlocBuilder rebuild mỗi khi `emit()` được gọi, kể cả state giống nhau
- Có Equatable → chỉ rebuild khi state **thực sự** thay đổi (so sánh `props`)

### 2.4 Bloc Class — Trung tâm xử lý

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterInitial()) {
    // Đăng ký handler cho từng event
    on<IncrementPressed>(_onIncrement);
    on<DecrementPressed>(_onDecrement);
    on<ResetPressed>(_onReset);
  }

  void _onIncrement(IncrementPressed event, Emitter<CounterState> emit) {
    emit(CounterUpdated(state.count + 1));
  }

  void _onDecrement(DecrementPressed event, Emitter<CounterState> emit) {
    emit(CounterUpdated(state.count - 1));
  }

  void _onReset(ResetPressed event, Emitter<CounterState> emit) {
    emit(const CounterInitial());
  }
}
```

**Giải thích:**
- `extends Bloc<CounterEvent, CounterState>` — generic: Event type + State type
- `super(const CounterInitial())` — state khởi tạo
- `on<Event>(handler)` — đăng ký handler cho từng loại Event
- `emit(newState)` — phát ra state mới, UI tự rebuild
- `state` — truy cập state hiện tại

### 2.5 Async trong Bloc

```dart
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  final TodoRepository repository;

  TodoBloc(this.repository) : super(TodoInitial()) {
    on<TodosFetched>(_onFetched);
    on<TodoAdded>(_onAdded);
  }

  Future<void> _onFetched(
    TodosFetched event,
    Emitter<TodoState> emit,
  ) async {
    emit(TodoLoading());
    try {
      final todos = await repository.fetchTodos();
      emit(TodoLoaded(todos));
    } catch (e) {
      emit(TodoError(e.toString()));
    }
  }

  Future<void> _onAdded(
    TodoAdded event,
    Emitter<TodoState> emit,
  ) async {
    final currentState = state;
    if (currentState is TodoLoaded) {
      final updatedTodos = [...currentState.todos, Todo(title: event.title)];
      emit(TodoLoaded(updatedTodos));
    }
  }
}
```

---

> 💼 **Gặp trong dự án:** Define Event và State classes cho feature phức tạp (login flow, shopping cart, order management), sealed class cho Events giúp exhaustive switch, Equatable cho State comparison
> 🤖 **Keywords bắt buộc trong prompt:** `sealed class Event`, `Equatable State`, `copyWith`, `emit`, `on<Event>`, `EventTransformer`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Login flow:** Events: LoginSubmitted, LoginWithGoogle, LogoutRequested. States: LoginInitial, LoginLoading, LoginSuccess(user), LoginFailure(error)
- **Bug thực tế:** Junior quên Equatable trên State → BlocBuilder rebuild ngay cả khi state giống nhau
- **Event transformer:** Search feature cần debounce — `EventTransformer` cho SearchQueryChanged event

**Tại sao cần các keyword trên:**
- **`sealed class Event`** — Dart 3 sealed classes giúp exhaustive pattern matching, AI hay dùng abstract class kiểu cũ
- **`Equatable State`** — so sánh state bằng value (không phải reference), thiếu → rebuild thừa
- **`copyWith`** — update 1 field trong state mà giữ nguyên các field khác, AI hay tạo state mới thiếu fields
- **`emit`** — chỉ emit trong event handler, AI hay emit ngoài on<Event> handler
- **`EventTransformer`** — debounce, throttle, sequential cho events

**Prompt mẫu — Convert Riverpod Weather sang BLoC:**
```text
Tôi cần convert Weather feature từ Riverpod sang BLoC pattern.
Tech stack: Flutter 3.x, flutter_bloc ^8.x, equatable ^2.x, freezed (cho Events/States).
Feature: Weather app — fetch weather by city, manage favorites.
Events (sealed class):
- WeatherFetchRequested(String city)
- WeatherRefreshRequested
- FavoriteCityAdded(String city)
- FavoriteCityRemoved(String city)
States (Equatable):
- WeatherState với fields: status (enum: initial/loading/success/failure), weather (WeatherData?), favorites (List<String>), errorMessage (String?)
- Dùng copyWith pattern.
Constraints:
- WeatherBloc extends Bloc<WeatherEvent, WeatherState>.
- Mỗi event handler dùng emit để update state.
- WeatherFetchRequested: emit loading → call API → emit success/failure.
- Events dùng sealed class (Dart 3), States dùng Equatable + copyWith.
- EventTransformer: debounce 300ms cho WeatherFetchRequested (user đang gõ city name).
Output: weather_event.dart + weather_state.dart + weather_bloc.dart.
```

**Expected Output:** AI gen 3 files với sealed events, equatable state, bloc với event handlers.

⚠️ **Giới hạn AI hay mắc:** AI hay dùng `abstract class` thay vì `sealed class` cho Events (không exhaustive). AI cũng hay quên `Equatable` hoặc quên override `props` list → state comparison bằng reference thay vì value.

</details>

> 🔗 **FE Bridge:** `Event` ≈ Redux `Action`, `State` ≈ Redux `State`, `on<Event>` handler ≈ Redux `Reducer`. Mapping gần 1:1. Nhưng **khác ở**: BLoC Event → State là **async by default** (có thể `await` trong handler), Redux reducer phải pure/sync (side effects trong middleware/thunk).

---

## 3. Cubit vs Bloc 🔴

### 3.1 Cubit — Phiên bản đơn giản

**Cubit** là phiên bản lightweight của Bloc — **không cần Events**, gọi method trực tiếp để emit state.

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0); // State khởi tạo = 0

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}
```

**So sánh với Bloc:**

```dart
// ❌ Bloc: Cần Event classes
sealed class CounterEvent {}
class IncrementPressed extends CounterEvent {}
class DecrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementPressed>((event, emit) => emit(state + 1));
    on<DecrementPressed>((event, emit) => emit(state - 1));
  }
}

// ✅ Cubit: Gọi method trực tiếp
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}
```

### 3.2 So sánh chi tiết

| Tiêu chí | Cubit | Bloc |
|-----------|-------|------|
| **Boilerplate** | Ít — chỉ cần method | Nhiều — cần Event classes |
| **Traceability** | Thấp — khó trace nguồn gốc state change | Cao — mọi thay đổi qua Event |
| **Complexity** | Đơn giản | Phức tạp hơn |
| **Reactiveness** | Method → emit | Event → handler → emit |
| **Testing** | Test methods trực tiếp | Test event-state mapping |
| **Event transforming** | Không hỗ trợ | `transformer` cho debounce, throttle |
| **Use case** | Counter, toggle, simple form | Search, auth flow, complex logic |

### 3.3 Decision Tree — Khi nào dùng cái nào?

```
Bạn cần state management →
├── Logic đơn giản (toggle, counter, simple form)?
│   └── ✅ Cubit
├── Cần trace mọi event (audit log, analytics)?
│   └── ✅ Bloc
├── Cần event transforming (debounce search, throttle)?
│   └── ✅ Bloc
├── Complex async flow (auth, multi-step process)?
│   └── ✅ Bloc
└── Không chắc?
    └── 🟡 Bắt đầu Cubit, upgrade lên Bloc khi cần
```

> 💡 **Thực tế:** Trong một project, có thể dùng **cả Cubit lẫn Bloc** — Cubit cho feature đơn giản, Bloc cho feature phức tạp. Không cần chọn "chỉ một".

> 🔗 **FE Bridge:** Cubit ≈ `useReducer` đơn giản (emit state trực tiếp), Bloc ≈ full Redux (event → handler → state). Chọn Cubit khi logic đơn giản (không cần track event history), chọn Bloc khi cần **event transforming** hoặc **debug event stream**. FE equivalent: `useState` vs full Redux.

---

## 4. flutter_bloc Widgets 🟡

### 4.1 BlocProvider — Cung cấp Bloc cho widget tree

```dart
// Tạo và provide bloc
BlocProvider(
  create: (context) => CounterCubit(),
  child: const CounterPage(),
)

// Provide nhiều bloc cùng lúc
MultiBlocProvider(
  providers: [
    BlocProvider(create: (context) => CounterCubit()),
    BlocProvider(create: (context) => TodoBloc(TodoRepository())),
    BlocProvider(create: (context) => AuthBloc(AuthRepository())),
  ],
  child: const MyApp(),
)
```

**Lưu ý:**
- `BlocProvider` tự động `close()` bloc khi widget bị dispose
- Dùng `BlocProvider.value()` khi provide bloc **đã tạo sẵn** (không tự close)

### 4.2 BlocBuilder — Rebuild UI khi State thay đổi

```dart
BlocBuilder<CounterCubit, int>(
  builder: (context, count) {
    return Text('Count: $count', style: const TextStyle(fontSize: 24));
  },
)
```

**buildWhen** — Tối ưu rebuild:

```dart
BlocBuilder<TodoBloc, TodoState>(
  // Chỉ rebuild khi state là TodoLoaded
  buildWhen: (previous, current) => current is TodoLoaded,
  builder: (context, state) {
    if (state is TodoLoaded) {
      return TodoList(todos: state.todos);
    }
    return const SizedBox.shrink();
  },
)
```

### 4.3 BlocListener — Side effects khi State thay đổi

Dùng khi cần **thực hiện action** (navigation, snackbar, dialog) — không rebuild UI.

```dart
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthSuccess) {
      Navigator.pushReplacementNamed(context, '/home');
    }
    if (state is AuthError) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.message)),
      );
    }
  },
  child: const LoginForm(),
)
```

**listenWhen** — Lọc state:

```dart
BlocListener<TodoBloc, TodoState>(
  listenWhen: (previous, current) => current is TodoError,
  listener: (context, state) {
    if (state is TodoError) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.message)),
      );
    }
  },
  child: const TodoPage(),
)
```

**MultiBlocListener** — nhiều listener:

```dart
MultiBlocListener(
  listeners: [
    BlocListener<AuthBloc, AuthState>(
      listener: (context, state) { /* handle auth */ },
    ),
    BlocListener<TodoBloc, TodoState>(
      listener: (context, state) { /* handle todo */ },
    ),
  ],
  child: const MyPage(),
)
```

### 4.4 BlocConsumer — Builder + Listener kết hợp

Khi cần **cả rebuild UI lẫn side effects**:

```dart
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthSuccess) {
      Navigator.pushReplacementNamed(context, '/home');
    }
  },
  builder: (context, state) {
    if (state is AuthLoading) {
      return const CircularProgressIndicator();
    }
    return const LoginForm();
  },
)
```

### 4.5 context.read vs context.watch

```dart
// context.read<T>() — Lấy bloc instance, KHÔNG listen
// Dùng trong event handlers, callbacks
ElevatedButton(
  onPressed: () {
    context.read<CounterCubit>().increment();
  },
  child: const Text('Increment'),
)

// context.watch<T>() — Listen state changes, trigger rebuild
// Dùng trong build method
@override
Widget build(BuildContext context) {
  final count = context.watch<CounterCubit>().state;
  return Text('Count: $count');
}

// context.select<T, R>() — Listen một phần state cụ thể
@override
Widget build(BuildContext context) {
  final count = context.select<CounterCubit, int>(
    (cubit) => cubit.state,
  );
  return Text('Count: $count');
}
```

> ⚠️ **Quan trọng:**
> - `context.read` → dùng trong **callbacks**, event handlers
> - `context.watch` → dùng trong **build method**
> - Dùng `context.watch` trong callback = BUG (rebuild không cần thiết)

---

> 💼 **Gặp trong dự án:** BlocBuilder rebuild quá nhiều, BlocListener không trigger, MultiBlocProvider setup sai thứ tự, BlocConsumer kết hợp cả UI rebuild + side effects
> 🤖 **Keywords bắt buộc trong prompt:** `BlocBuilder buildWhen`, `BlocListener listenWhen`, `BlocConsumer`, `MultiBlocProvider`, `context.read<Bloc>().add(Event)`, `BlocSelector`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Performance:** BlocBuilder rebuild cả screen khi bất kỳ state field đổi — cần `buildWhen` filter
- **Navigation:** Login success → navigate to Home — dùng BlocListener, AI hay dùng BlocBuilder (SAI — build chạy nhiều lần)
- **Dual need:** Vừa rebuild UI vừa show SnackBar → BlocConsumer

**Tại sao cần các keyword trên:**
- **`BlocBuilder buildWhen`** — chỉ rebuild khi condition true, AI hay bỏ qua → rebuild thừa
- **`BlocListener listenWhen`** — side effects (navigate, snackbar), chỉ chạy 1 lần khi state change
- **`BlocConsumer`** — kết hợp builder + listener, AI hay tách thành 2 widgets lồng nhau (verbose)
- **`MultiBlocProvider`** — thứ tự providers, AI hay tạo nested BlocProvider (pyramid of doom)

**Prompt mẫu — BLoC Widgets setup:**
```text
Tôi cần setup flutter_bloc widgets cho Auth flow.
Tech stack: Flutter 3.x, flutter_bloc ^8.x.
Screens: LoginScreen, HomeScreen.
Requirements:
- LoginScreen dùng BlocConsumer<AuthBloc, AuthState>:
  - builder: hiển thị form + loading indicator khi state.isSubmitting.
  - listener: navigate to Home khi state == AuthSuccess, show SnackBar khi AuthFailure.
  - buildWhen: chỉ rebuild khi isSubmitting đổi hoặc errorMessage đổi.
  - listenWhen: chỉ listen khi status đổi (không phải khi form field đổi).
- MultiBlocProvider ở root: AuthBloc + ThemeBloc.
- Button onPressed: context.read<AuthBloc>().add(LoginSubmitted(email, password)).
Output: login_screen.dart + app.dart (MultiBlocProvider setup).
```

**Expected Output:** AI gen login_screen.dart + app.dart với BlocConsumer, buildWhen/listenWhen, MultiBlocProvider.

⚠️ **Giới hạn AI hay mắc:** AI hay dùng BlocBuilder cho navigation (navigator.push trong builder — SAI, chạy mỗi rebuild). AI cũng hay quên `buildWhen`/`listenWhen` optimization.

</details>

> 🔗 **FE Bridge:** `BlocBuilder` ≈ `useSelector` (rebuild UI khi state thay đổi), `BlocListener` ≈ `useEffect` trên state (side effects: navigation, snackbar), `BlocConsumer` = cả hai combined. Nhưng **khác ở**: Flutter tách rõ Builder (UI) vs Listener (side effect) — React dùng `useEffect` cho cả hai.

---

## 5. Testing BLoC 🟡

### 5.1 bloc_test package

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:test/test.dart';

void main() {
  group('CounterCubit', () {
    // Test state khởi tạo
    test('initial state is 0', () {
      final cubit = CounterCubit();
      expect(cubit.state, 0);
      cubit.close();
    });

    // blocTest — cách test chuẩn
    blocTest<CounterCubit, int>(
      'emits [1] when increment is called',
      build: () => CounterCubit(),
      act: (cubit) => cubit.increment(),
      expect: () => [1],
    );

    blocTest<CounterCubit, int>(
      'emits [1, 2, 3] when increment is called 3 times',
      build: () => CounterCubit(),
      act: (cubit) {
        cubit.increment();
        cubit.increment();
        cubit.increment();
      },
      expect: () => [1, 2, 3],
    );

    blocTest<CounterCubit, int>(
      'emits [-1] when decrement is called',
      build: () => CounterCubit(),
      act: (cubit) => cubit.decrement(),
      expect: () => [-1],
    );
  });
}
```

### 5.2 Testing Bloc (event-driven)

```dart
blocTest<TodoBloc, TodoState>(
  'emits [TodoLoading, TodoLoaded] when TodosFetched is added',
  build: () {
    // Mock repository
    when(() => mockRepository.fetchTodos())
        .thenAnswer((_) async => [Todo(title: 'Test')]);
    return TodoBloc(mockRepository);
  },
  act: (bloc) => bloc.add(TodosFetched()),
  expect: () => [
    TodoLoading(),
    TodoLoaded([Todo(title: 'Test')]),
  ],
);

blocTest<TodoBloc, TodoState>(
  'emits [TodoLoading, TodoError] when fetchTodos throws',
  build: () {
    when(() => mockRepository.fetchTodos())
        .thenThrow(Exception('Network error'));
    return TodoBloc(mockRepository);
  },
  act: (bloc) => bloc.add(TodosFetched()),
  expect: () => [
    TodoLoading(),
    isA<TodoError>(),
  ],
);
```

### 5.3 Mocking dependencies

```dart
import 'package:mocktail/mocktail.dart';

class MockTodoRepository extends Mock implements TodoRepository {}

void main() {
  late MockTodoRepository mockRepository;

  setUp(() {
    mockRepository = MockTodoRepository();
  });

  group('TodoBloc', () {
    blocTest<TodoBloc, TodoState>(
      'emits [TodoLoading, TodoLoaded] on success',
      setUp: () {
        when(() => mockRepository.fetchTodos())
            .thenAnswer((_) async => <Todo>[]);
      },
      build: () => TodoBloc(mockRepository),
      act: (bloc) => bloc.add(TodosFetched()),
      expect: () => [
        TodoLoading(),
        const TodoLoaded([]),
      ],
      verify: (_) {
        verify(() => mockRepository.fetchTodos()).called(1);
      },
    );
  });
}
```

### 5.4 Anatomy của blocTest

```dart
blocTest<MyBloc, MyState>(
  'description',           // Mô tả test case
  build: () => MyBloc(),   // Tạo bloc instance
  seed: () => MyState(),   // (Optional) Set state ban đầu
  act: (bloc) => ...,      // Thực hiện action (add event / call method)
  wait: Duration(...),     // (Optional) Đợi async
  expect: () => [...],     // Danh sách states expected (theo thứ tự)
  verify: (bloc) => ...,   // (Optional) Verify thêm
  errors: () => [...],     // (Optional) Expect errors
);
```

> 🔗 **FE Bridge:** `blocTest()` ≈ Redux test pattern: given initial state → when dispatch actions → then expect state sequence. `bloc_test` package cung cấp `act` + `expect` pattern quen thuộc với FE dev dùng Redux test.

---

## 6. So sánh Riverpod vs BLoC 🟡

### 6.1 Trade-offs table

| Tiêu chí | Riverpod | BLoC |
|-----------|----------|------|
| **Learning curve** | Trung bình — concept provider, ref | Cao — Event/State/Bloc pattern |
| **Boilerplate** | Ít — đặc biệt với code generation | Nhiều — Event + State classes |
| **Testability** | Tốt — ProviderContainer | Rất tốt — bloc_test, isolated |
| **Traceability** | Trung bình | Rất cao — mọi action qua Event |
| **Scalability** | Tốt | Rất tốt — pattern chuẩn enterprise |
| **Community** | Đang phát triển mạnh | Lớn, mature, nhiều resource |
| **Code generation** | Hỗ trợ (riverpod_generator) | Không cần |
| **DevTools** | Riverpod DevTools | BLoC Observer, DevTools |
| **Dependency injection** | Built-in (ref.read/watch) | Cần setup thêm (get_it...) |
| **Side effects** | ref.listen, AsyncValue | BlocListener, integrated |

### 6.2 Khi nào chọn cái nào?

**Chọn Riverpod khi:**
- Team nhỏ, cần move nhanh
- Muốn ít boilerplate, code gọn
- Cần dependency injection tích hợp
- Dự án mới, muốn dùng approach hiện đại

**Chọn BLoC khi:**
- Team lớn, cần pattern rõ ràng
- Yêu cầu traceability cao (audit, logging)
- Enterprise project, cần chuẩn hóa
- Team có background Redux/Vuex

### 6.3 Có thể dùng cùng nhau không?

**Có, nhưng KHÔNG nên.** Lý do:
- Complexity tăng — team phải hiểu cả hai
- Inconsistent patterns — khó maintain
- **Recommendation:** Chọn **MỘT** cho mỗi project, giữ consistency

> 💡 **Team recommendation:** Nếu bắt đầu project mới và team chưa có kinh nghiệm state management → **Riverpod**. Nếu team lớn, cần pattern strict, hoặc đã quen BLoC → **BLoC**.

> 🔗 **FE Bridge:** Riverpod vs BLoC ≈ Zustand/Jotai vs Redux trong FE world. Riverpod = flexible, ít boilerplate. BLoC = structured, nhiều boilerplate nhưng predictable. Cùng trade-off như FE: team lớn thường chọn BLoC/Redux vì enforced patterns, team nhỏ chọn Riverpod/Zustand vì rapid development.

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

```dart
// 1. Một Bloc cho một feature
// ✅ Tốt
class AuthBloc extends Bloc<AuthEvent, AuthState> { ... }
class TodoBloc extends Bloc<TodoEvent, TodoState> { ... }

// ❌ Xấu — God Bloc
class AppBloc extends Bloc<AppEvent, AppState> { ... } // Làm mọi thứ

// 2. Dùng Equatable cho States
// ✅ Mặc định — dùng extends Equatable (API sạch hơn)
class TodoLoaded extends TodoState {
  final List<Todo> todos;
  TodoLoaded(this.todos);
  @override
  List<Object?> get props => [todos];
}

// ✅ Dùng EquatableMixin khi đã extends class khác
class ApiException extends AppException with EquatableMixin {
  final int statusCode;
  ApiException(this.statusCode);
  @override
  List<Object?> get props => [statusCode];
}

// 3. BlocObserver cho logging
class AppBlocObserver extends BlocObserver {
  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    debugPrint('${bloc.runtimeType} $transition');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    super.onError(bloc, error, stackTrace);
    debugPrint('${bloc.runtimeType} $error');
  }
}

// main.dart
void main() {
  Bloc.observer = AppBlocObserver();
  runApp(const MyApp());
}

// 4. Đặt tên Event theo past tense
// ✅ Tốt
class TodoAdded extends TodoEvent { ... }
class TodoDeleted extends TodoEvent { ... }

// ❌ Xấu — imperative
class AddTodo extends TodoEvent { ... }
class DeleteTodo extends TodoEvent { ... }
```

### ❌ Lỗi thường gặp

```dart
// 1. Emit sau khi Bloc đã close
// ❌ Xấu
Future<void> _onFetch(FetchEvent event, Emitter<MyState> emit) async {
  final data = await repository.fetch(); // Nếu bloc close trong lúc await...
  emit(Loaded(data)); // 💥 Error: emit after close
}

// ✅ Sửa — check isClosed hoặc dùng emit.isDone
Future<void> _onFetch(FetchEvent event, Emitter<MyState> emit) async {
  emit(Loading());
  try {
    final data = await repository.fetch();
    emit(Loaded(data)); // flutter_bloc 8+ tự handle nếu bloc closed
  } catch (e) {
    emit(ErrorState(e.toString()));
  }
}

// 2. Dùng context.watch trong callback
// ❌ Xấu
onPressed: () {
  final bloc = context.watch<CounterCubit>(); // 💥 KHÔNG dùng watch trong callback
  bloc.increment();
}

// ✅ Sửa
onPressed: () {
  context.read<CounterCubit>().increment(); // Dùng read trong callback
}

// 3. Quên close Bloc
// ❌ Nếu tạo Bloc thủ công (không qua BlocProvider)
final bloc = CounterCubit();
// ... quên bloc.close() → memory leak

// ✅ BlocProvider tự close — luôn dùng BlocProvider
BlocProvider(create: (_) => CounterCubit(), child: ...)
```

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Flutter/BLoC Mindset | Tại sao khác |
|---|-------------------|----------------------|--------------|
| 1 | Redux reducer = pure sync function | BLoC `on<Event>` handler = **async by default** | BLoC xử lý side effects ngay trong handler |
| 2 | `useEffect` cho cả UI update và side effects | `BlocBuilder` cho UI, `BlocListener` cho side effects — **tách rõ** | Flutter enforce separation of concerns |
| 3 | Global Redux store = single source of truth | Mỗi BLoC = isolated unit, không share state trực tiếp | BLoC = scoped state thay vì global state |
| 4 | Action creator → middleware → reducer chain | Event → `on<Event>` handler → emit State — đơn giản hơn | Không có middleware layer riêng trong BLoC |
| 5 | Redux DevTools track action history | `BlocObserver` track event + state transitions | Tương đương concept, khác tool |

Nếu bạn đến từ React hoặc Vue, đây là mapping giúp bạn hiểu BLoC nhanh hơn:

| Flutter BLoC | React/Redux | Vue/Vuex |
|-------------|-------------|----------|
| **Event** | Action (dispatch) | Action/Mutation |
| **State** | Store state | State |
| **Bloc** | Reducer + Middleware | Store module |
| **Cubit** | `useReducer` hook | Simple store |
| **emit()** | `return newState` (reducer) | `commit()` |
| **BlocBuilder** | `connect()` / `useSelector` | `mapState` / computed |
| **BlocListener** | `useEffect` with selector | `watch` |
| **BlocProvider** | `<Provider store={}>` | `app.use(store)` |
| **BlocObserver** | Redux DevTools middleware | Vuex plugins |

**Sự khác biệt chính:**
- BLoC **enforce** unidirectional flow chặt chẽ hơn Redux
- **Cubit ≈ useReducer** — dispatch function trực tiếp, không cần action type
- **BlocListener** = side effect handler — giống `useEffect` react nhưng chỉ cho state changes
- Flutter không có "connect HOC" — dùng **BlocBuilder widget** thay thế

---

## 9. Tổng kết

### Checklist kiến thức buổi 08

- [ ] Hiểu BLoC pattern — unidirectional data flow (UI → Event → Bloc → State → UI)
- [ ] Viết được Event sealed class, State sealed class, Bloc class
- [ ] Dùng `on<Event>()` để đăng ký handler, `emit()` để phát state
- [ ] Hiểu và dùng **Equatable** cho state comparison
- [ ] Phân biệt **Cubit** (method-based, đơn giản) vs **Bloc** (event-driven, traceable)
- [ ] Biết khi nào dùng Cubit, khi nào cần Bloc
- [ ] Sử dụng **BlocProvider** để provide bloc cho widget tree
- [ ] Sử dụng **BlocBuilder** để rebuild UI theo state
- [ ] Sử dụng **BlocListener** cho side effects (navigation, snackbar)
- [ ] Sử dụng **BlocConsumer** = Builder + Listener
- [ ] Phân biệt `context.read()` (callback) vs `context.watch()` (build)
- [ ] Viết test với **bloc_test**: `blocTest()` — build, act, expect
- [ ] Mock dependencies với `mocktail`
- [ ] So sánh được **Riverpod vs BLoC** — trade-offs, khi nào chọn cái nào
- [ ] Áp dụng BLoC best practices: naming, observer, equatable, one-bloc-per-feature

---

### ➡️ Buổi tiếp theo

> **Buổi 09: Clean Architecture** — Domain/Data/Presentation layers, Dependency Rule, và folder structure chuẩn cho dự án production.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Đọc SOLID principles (đặc biệt Dependency Inversion)
