# Buổi 08: BLoC Pattern — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> ⏱ Tổng thời gian: ~120 phút
> 🎯 Mục tiêu: Thực hành Cubit, Bloc, BlocListener, và testing

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ Redux/Vuex**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| Redux/Vuex Habit | BLoC Reality | Bài tập liên quan |
|------------------|--------------|---------------------|
| Reducer phải pure sync | BLoC handler **async by default** — có thể await API calls trực tiếp | BT1, BT2 |
| `useSelector` + `useEffect` trong component | `BlocBuilder` cho UI rebuild, `BlocListener` cho side effects — tách rõ | BT1, BT3 |
| Dispatch action từ bất kỳ đâu | `bloc.add(Event)` — phải có reference đến đúng BLoC instance | BT2 |
| State update = new object (immutable) | State class phải **copyWith** hoặc tạo mới — tương tự Redux | BT1, BT2, BT3 |

---

## BT1 ⭐ Counter App với Cubit 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_counter_cubit` |
| **Setup** | `flutter pub add flutter_bloc bloc equatable` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — counter app với Cubit và màu sắc theo giá trị |

**Thời gian:** ~25 phút

### Yêu cầu

Tạo một Flutter app counter sử dụng **Cubit**:

1. **CounterCubit** với 3 operations:
   - `increment()` — tăng 1
   - `decrement()` — giảm 1
   - `reset()` — về 0

2. **UI cần có:**
   - Hiển thị số đếm giữa màn hình (font lớn)
   - 3 nút: ➕ Increment, ➖ Decrement, 🔄 Reset
   - Màu số thay đổi theo giá trị: xanh (> 0), đỏ (< 0), xám (= 0)

3. **Cấu trúc file:**
   ```
   lib/
   ├── main.dart
   └── counter/
       ├── counter_cubit.dart
       └── counter_page.dart
   ```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

```dart
// counter_cubit.dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  // TODO: implement increment, decrement, reset
}
```

```dart
// counter_page.dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const CounterView(),
    );
  }
}

class CounterView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: BlocBuilder<CounterCubit, int>(
          builder: (context, count) {
            // TODO: hiển thị count với màu sắc
          },
        ),
      ),
      // TODO: 3 FloatingActionButton
    );
  }
}
```

</details>

### Tiêu chí hoàn thành

- [ ] CounterCubit hoạt động đúng (increment, decrement, reset)
- [ ] BlocProvider wrap đúng vị trí
- [ ] BlocBuilder rebuild khi state thay đổi
- [ ] `context.read` dùng trong callback (onPressed)
- [ ] Màu sắc thay đổi theo giá trị

---

## BT2 ⭐⭐ Todo App với Full Bloc Pattern 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_todo_bloc` |
| **Setup** | `flutter pub add flutter_bloc bloc equatable` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Todo app với full Bloc pattern (Events + States) |

**Thời gian:** ~45 phút

### Yêu cầu

Tạo một Flutter Todo app sử dụng **full Bloc pattern** (Events + States):

1. **Model:**
   ```dart
   class Todo {
     final String id;
     final String title;
     final bool isCompleted;
   }
   ```

2. **Events:**
   - `TodoItemAdded(String title)` — thêm todo mới
   - `TodoItemToggled(String id)` — toggle hoàn thành
   - `TodoItemDeleted(String id)` — xoá todo

3. **States** (dùng sealed class):
   - `TodoInitial` — chưa có data
   - `TodoLoaded(List<Todo> todos)` — đã có data

4. **TodoBloc:**
   - Handle cả 3 events
   - Emit state mới sau mỗi thao tác

5. **UI cần có:**
   - AppBar hiển thị "X/Y completed"
   - ListView hiển thị danh sách todo
   - Checkbox để toggle
   - Swipe để xoá (Dismissible)
   - FAB + Dialog để thêm todo mới
   - Empty state khi chưa có todo

6. **Cấu trúc file:**
   ```
   lib/
   ├── main.dart
   └── todo/
       ├── models/
       │   └── todo.dart
       ├── bloc/
       │   ├── todo_event.dart
       │   ├── todo_state.dart
       │   └── todo_bloc.dart
       └── todo_page.dart
   ```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

1. Tạo model `Todo` với id, title, isCompleted
2. Định nghĩa sealed class `TodoEvent` với 3 events: Added, Toggled, Deleted
3. Định nghĩa sealed class `TodoState` với Equatable — `TodoLoaded` chứa `List<Todo>`
4. Implement `TodoBloc` handle 3 events, emit state mới sau mỗi thao tác
5. Tạo UI: AppBar hiển thị count, ListView với Checkbox và Dismissible
6. FAB + Dialog để thêm todo mới, wrap page trong `BlocProvider`

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// todo_event.dart
sealed class TodoEvent {}

class TodoItemAdded extends TodoEvent {
  final String title;
  TodoItemAdded(this.title);
}
// TODO: thêm TodoItemToggled, TodoItemDeleted

// todo_state.dart
sealed class TodoState extends Equatable { ... }
class TodoLoaded extends TodoState {
  final List<Todo> todos;
  // TODO: computed properties: completedCount, pendingCount
}

// todo_bloc.dart
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  TodoBloc() : super(TodoLoaded()) {
    on<TodoItemAdded>(_onAdded);
    on<TodoItemToggled>(_onToggled);
    on<TodoItemDeleted>(_onDeleted);
  }
  // TODO: implement handlers
}
```

</details>

### Tiêu chí hoàn thành

- [ ] Event sealed class với 3 events
- [ ] State sealed class với Equatable
- [ ] TodoBloc handle 3 events đúng logic
- [ ] UI hiển thị danh sách, toggle, xoá hoạt động
- [ ] Dialog thêm todo hoạt động
- [ ] Empty state hiển thị khi không có todo
- [ ] Không có warning/error khi chạy

### Bonus ⭐

- [ ] Thêm filter: All / Active / Completed
- [ ] Viết ít nhất 3 unit tests cho TodoBloc

---

## BT3 ⭐⭐⭐ Authentication Flow với Bloc + BlocListener 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_auth_bloc` |
| **Setup** | `flutter pub add flutter_bloc bloc equatable` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — authentication flow với login/logout và navigation |

**Thời gian:** ~50 phút

### Yêu cầu

Tạo một Flutter app mô phỏng **authentication flow** hoàn chỉnh:

1. **Flow:**
   ```
   LoginPage ──(submit)──▶ Loading ──▶ Success ──▶ HomePage
                                    └──▶ Error ──▶ Show SnackBar
   HomePage ──(logout)──▶ LoginPage
   ```

2. **Events:**
   ```dart
   sealed class AuthEvent {}
   class LoginRequested extends AuthEvent {
     final String email;
     final String password;
   }
   class LogoutRequested extends AuthEvent {}
   ```

3. **States:**
   ```dart
   sealed class AuthState {}
   class AuthInitial extends AuthState {}      // Chưa login
   class AuthLoading extends AuthState {}      // Đang xử lý
   class AuthSuccess extends AuthState {       // Login thành công
     final String userName;
   }
   class AuthFailure extends AuthState {       // Login thất bại
     final String errorMessage;
   }
   ```

4. **AuthBloc:**
   - `LoginRequested` → emit Loading → (giả lập 2s delay) → Success hoặc Failure
   - Rule: email = "test@test.com" và password = "123456" → Success, còn lại → Failure
   - `LogoutRequested` → emit AuthInitial

5. **LoginPage:**
   - Form với email + password TextFields
   - Login button (disabled khi loading)
   - **BlocListener**: navigate tới HomePage khi AuthSuccess
   - **BlocListener**: show SnackBar khi AuthFailure
   - **BlocBuilder**: show CircularProgressIndicator khi AuthLoading

6. **HomePage:**
   - Hiển thị "Welcome, {userName}!"
   - Logout button
   - **BlocListener**: navigate về LoginPage khi AuthInitial

7. **Cấu trúc file:**
   ```
   lib/
   ├── main.dart
   └── auth/
       ├── bloc/
       │   ├── auth_event.dart
       │   ├── auth_state.dart
       │   └── auth_bloc.dart
       ├── login_page.dart
       └── home_page.dart
   ```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

1. Tạo sealed class `AuthEvent` gồm `LoginRequested` (email, password) và `LogoutRequested`
2. Tạo sealed class `AuthState`: AuthInitial, AuthLoading, AuthSuccess(userName), AuthFailure(errorMessage)
3. Implement `AuthBloc`: LoginRequested → emit Loading → delay 2s → Success/Failure; LogoutRequested → AuthInitial
4. Tạo LoginPage với `BlocConsumer`: listener navigate khi Success, show SnackBar khi Failure
5. Tạo HomePage hiển thị userName, logout button với `BlocListener` navigate về LoginPage
6. Wrap app trong `BlocProvider` ở main.dart

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// auth_bloc.dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
    on<LogoutRequested>(_onLogoutRequested);
  }

  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    await Future.delayed(const Duration(seconds: 2)); // Giả lập API

    if (event.email == 'test@test.com' && event.password == '123456') {
      emit(AuthSuccess(userName: 'Test User'));
    } else {
      emit(AuthFailure(errorMessage: 'Email hoặc mật khẩu không đúng'));
    }
  }

  // TODO: implement _onLogoutRequested
}
```

```dart
// login_page.dart — key pattern
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: BlocConsumer<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthSuccess) {
            // TODO: Navigate to HomePage
          }
          if (state is AuthFailure) {
            // TODO: Show SnackBar
          }
        },
        builder: (context, state) {
          final isLoading = state is AuthLoading;
          // TODO: Build login form
          // Disable button when isLoading
          // Show CircularProgressIndicator when isLoading
        },
      ),
    );
  }
}
```

```dart
// main.dart — BlocProvider ở root
void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => AuthBloc(),
      child: MaterialApp(
        home: const LoginPage(),
      ),
    );
  }
}
```

</details>

### Tiêu chí hoàn thành

- [ ] AuthBloc xử lý Login + Logout đúng logic
- [ ] LoginPage: form + validation cơ bản (email & password không trống)
- [ ] Loading state: button disabled + indicator hiển thị
- [ ] Login thành công: navigate tới HomePage
- [ ] Login thất bại: SnackBar hiển thị error message
- [ ] HomePage hiển thị userName
- [ ] Logout: navigate về LoginPage
- [ ] BlocListener dùng cho navigation + snackbar (KHÔNG phải BlocBuilder)
- [ ] Không crash, không leak

### Bonus ⭐

- [ ] Thêm "Remember me" checkbox
- [ ] Thêm form validation chi tiết (email format, password length)
- [ ] Viết unit tests cho AuthBloc
- [ ] Thêm BlocObserver log transitions

---

## 💬 Câu hỏi thảo luận

### Câu 1: Cubit vs Bloc — Quyết định như thế nào?

> Bạn đang bắt đầu một feature mới: **search bar tìm kiếm sản phẩm** (gõ text → gọi API → hiển thị kết quả, có debounce).
>
> Bạn sẽ chọn **Cubit** hay **Bloc**? Giải thích lý do.

**Gợi ý suy nghĩ:**
- Search cần debounce → Bloc có `transformer` hỗ trợ
- Cubit có thể tự implement debounce với Timer
- Traceability: mỗi keystroke có cần là event riêng?

---

### Câu 2: Riverpod vs BLoC — Cho team của bạn?

> Team bạn gồm 5 Flutter developers (2 senior, 3 junior). Đang bắt đầu project mới — e-commerce app, dự kiến phát triển 1 năm.
>
> Bạn sẽ recommend **Riverpod** hay **BLoC** cho team? Giải thích trade-offs.

**Gợi ý suy nghĩ:**
- Team size & experience level
- Project complexity & duration
- Onboarding time cho junior
- Long-term maintainability
- Testability requirements

---

### Câu 3: BLoC Testing Best Practices

> Bạn đang viết tests cho một `OrderBloc` phức tạp (create order → payment → confirmation). Bloc phụ thuộc vào `OrderRepository` và `PaymentService`.
>
> Hãy mô tả chiến lược testing: cần mock gì, test cases nào là quan trọng nhất, dùng `seed` hay `act` để set up state?

**Gợi ý suy nghĩ:**
- Mock dependencies vs real implementation
- Happy path + error cases
- Edge cases: network error giữa chừng, duplicate events
- `seed` cho test từ state cụ thể vs `act` cho test full flow

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 4:** Focus vào gen state management code và review event/state architecture.

### AI-BT1: Convert Riverpod Weather App sang BLoC ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Events (sealed), States (Equatable), Bloc class, BlocBuilder/BlocListener/BlocConsumer, testing.
- **Task thực tế:** Tech lead quyết định chuyển project từ Riverpod sang BLoC (team quen BLoC hơn, traceability tốt hơn cho debug). Cần convert Weather feature: giữ nguyên UI, chỉ đổi state management layer.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần convert Weather feature từ Riverpod sang BLoC pattern.
Tech stack: Flutter 3.x, flutter_bloc ^8.x, equatable ^2.x.
Feature gốc (Riverpod):
- weatherProvider: FutureProvider.family fetch weather by city
- favoritesProvider: NotifierProvider manage favorites list
- UI: ConsumerWidget, ref.watch, ref.read, AsyncValue.when

Convert sang BLoC:
Events (sealed class):
1. WeatherFetchRequested(String city)
2. FavoriteCityToggled(String city)
3. WeatherRefreshed

States (Equatable + copyWith):
- WeatherState: status (initial/loading/loaded/error), weather?, favorites, errorMessage?

Constraints:
- Bloc extends Bloc<WeatherEvent, WeatherState>.
- Sealed class cho Events (Dart 3, exhaustive switch).
- Equatable + copyWith cho State (value comparison).
- BlocConsumer: builder cho UI, listener cho error SnackBar.
- buildWhen: chỉ rebuild khi status hoặc weather đổi.
- Unit tests dùng bloc_test: verify state transitions cho mỗi event.
Output: weather_event.dart + weather_state.dart + weather_bloc.dart + weather_screen.dart + weather_bloc_test.dart.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 5 files với full BLoC setup + tests.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Events dùng `sealed class` (không phải abstract class)? | ☐ |
| 2 | State extend `Equatable`, override `props` list đủ fields? | ☐ |
| 3 | `copyWith` method update đúng (giữ field cũ khi không truyền param mới)? | ☐ |
| 4 | `emit(state.copyWith(status: ...))` — không tạo state mới bỏ field? | ☐ |
| 5 | BlocListener cho side effects (SnackBar), BlocBuilder cho UI rebuild? | ☐ |
| 6 | `buildWhen` có filter để tránh rebuild thừa? | ☐ |
| 7 | Tests dùng `blocTest<>()` với `expect: [...]` states sequence? | ☐ |
| 8 | `flutter analyze` không warning? Sealed class exhaustive? | ☐ |

**4. Customize:**
Thêm EventTransformer: debounce 300ms cho `WeatherFetchRequested` khi user đang gõ city name. AI chưa handle phần này — import `stream_transformers` và implement `restartable()` transformer.
