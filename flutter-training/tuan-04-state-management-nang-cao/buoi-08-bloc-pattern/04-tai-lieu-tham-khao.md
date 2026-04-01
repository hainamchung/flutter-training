# Buổi 08: BLoC Pattern — Tài liệu tham khảo

---

## 📖 Tài liệu chính thức

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| BLoC Library | [bloclibrary.dev](https://bloclibrary.dev) | Trang chủ chính thức — docs, tutorials, examples |
| flutter_bloc (pub.dev) | [pub.dev/packages/flutter_bloc](https://pub.dev/packages/flutter_bloc) | Package chính cho Flutter |
| bloc (pub.dev) | [pub.dev/packages/bloc](https://pub.dev/packages/bloc) | Core library (platform-independent) |
| bloc_test (pub.dev) | [pub.dev/packages/bloc_test](https://pub.dev/packages/bloc_test) | Testing utilities cho BLoC |
| BLoC GitHub | [github.com/felangel/bloc](https://github.com/felangel/bloc) | Source code + examples |

---

## 📝 Bài viết quan trọng

### BLoC Fundamentals
- **[Flutter BLoC Pattern — Getting Started](https://bloclibrary.dev/getting-started/)** — Hướng dẫn bắt đầu chính thức
- **[BLoC Architecture](https://bloclibrary.dev/architecture/)** — Kiến trúc và best practices
- **[BLoC Naming Conventions](https://bloclibrary.dev/naming-conventions/)** — Quy tắc đặt tên Event, State, Bloc

### Cubit vs Bloc
- **[Cubit vs Bloc](https://bloclibrary.dev/bloc-concepts/#cubit-vs-bloc)** — So sánh chính thức từ tác giả
- **[When to use Cubit vs Bloc](https://github.com/felangel/bloc/issues/3432)** — Discussion từ community

### Testing
- **[BLoC Testing](https://bloclibrary.dev/testing/)** — Guide testing chính thức
- **[bloc_test API Reference](https://pub.dev/documentation/bloc_test/latest/)** — API docs cho bloc_test

### So sánh State Management
- **[Flutter State Management Comparison](https://docs.flutter.dev/data-and-backend/state-mgmt/options)** — Danh sách từ Flutter team
- **[Riverpod vs BLoC](https://codewithandrea.com/articles/flutter-state-management-riverpod/)** — So sánh chi tiết

---

## 🎥 Video hướng dẫn

| Video | Kênh | Nội dung |
|-------|------|----------|
| Flutter BLoC Tutorial (Full Course) | Reso Coder | Bloc pattern từ cơ bản đến nâng cao |
| Flutter Bloc Library Tutorial | Felix Angelov | Tutorial từ tác giả BLoC |
| Cubit vs Bloc - When to use what | Vandad Nahavandipoor | Giải thích khi nào dùng Cubit vs Bloc |
| Flutter BLoC State Management | The Net Ninja | Series BLoC cho beginner |
| BLoC Testing Deep Dive | Reso Coder | Testing strategies cho BLoC |

---

## 📦 Packages liên quan

### Core

| Package | Version | Mô tả |
|---------|---------|--------|
| `flutter_bloc` | ^8.1.6 | BLoC widgets cho Flutter (BlocProvider, BlocBuilder...) |
| `bloc` | ^8.1.4 | Core BLoC library (platform-independent) |
| `bloc_test` | ^9.1.7 | Testing utilities (blocTest function) |
| `equatable` | ^2.0.5 | Value equality cho Event/State classes |

### Testing

| Package | Version | Mô tả |
|---------|---------|--------|
| `mocktail` | ^1.0.4 | Mocking library (thay thế mockito, không cần codegen) |
| `bloc_test` | ^9.1.7 | blocTest() function cho testing Bloc/Cubit |

### Mở rộng

| Package | Version | Mô tả |
|---------|---------|--------|
| `hydrated_bloc` | ^9.1.5 | Tự động persist/restore state (local storage) |
| `replay_bloc` | ^0.2.7 | Undo/redo cho Bloc/Cubit |
| `bloc_concurrency` | ^0.2.5 | Event transformers (sequential, droppable, restartable) |

### Setup nhanh

```yaml
# pubspec.yaml
dependencies:
  flutter_bloc: ^8.1.5
  equatable: ^2.0.5

dev_dependencies:
  bloc_test: ^9.1.1
  mocktail: ^1.0.4
```

---

## 🔗 Tài liệu bổ sung

### Dart Language (liên quan)
- **[Sealed Classes](https://dart.dev/language/class-modifiers#sealed)** — Dùng cho Event/State hierarchy
- **[Pattern Matching](https://dart.dev/language/patterns)** — Switch expression với sealed class
- **[Streams](https://dart.dev/tutorials/language/streams)** — BLoC sử dụng Streams internally

### Flutter Architecture
- **[Flutter Architecture Guide](https://docs.flutter.dev/app-architecture)** — Guide chính thức từ Flutter team
- **[Clean Architecture with BLoC](https://resocoder.com/flutter-clean-architecture-tdd/)** — Kết hợp Clean Architecture + BLoC

### DevTools
- **[Flutter DevTools](https://docs.flutter.dev/tools/devtools)** — Debug BLoC state changes
- **[BlocObserver](https://bloclibrary.dev/bloc-concepts/#blocobserver)** — Log tất cả transitions

---

## 🗂 Cấu trúc project mẫu (reference)

```
lib/
├── main.dart
├── app/
│   ├── app.dart              # MaterialApp + MultiBlocProvider
│   └── bloc_observer.dart    # AppBlocObserver
├── features/
│   ├── counter/
│   │   ├── cubit/
│   │   │   └── counter_cubit.dart
│   │   └── view/
│   │       └── counter_page.dart
│   ├── todo/
│   │   ├── bloc/
│   │   │   ├── todo_bloc.dart
│   │   │   ├── todo_event.dart
│   │   │   └── todo_state.dart
│   │   ├── models/
│   │   │   └── todo.dart
│   │   └── view/
│   │       └── todo_page.dart
│   └── auth/
│       ├── bloc/
│       │   ├── auth_bloc.dart
│       │   ├── auth_event.dart
│       │   └── auth_state.dart
│       └── view/
│           ├── login_page.dart
│           └── home_page.dart
test/
├── features/
│   ├── counter/
│   │   └── cubit/
│   │       └── counter_cubit_test.dart
│   ├── todo/
│   │   └── bloc/
│   │       └── todo_bloc_test.dart
│   └── auth/
│       └── bloc/
│           └── auth_bloc_test.dart
```

---

## 🤖 AI Prompt Library — Buổi 08: BLoC Pattern

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Flutter BLoC pattern. Background: 4+ năm React (Redux, useReducer).
Câu hỏi: Event trong BLoC giống Action trong Redux? State trong BLoC giống Redux store? Bloc class giống Reducer? emit giống dispatch? BlocBuilder giống useSelector? BlocListener giống middleware?
Yêu cầu: mapping 1-1 với Redux concepts, giải thích bằng tiếng Việt, kèm code flutter_bloc minh họa.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần implement Authentication flow dùng BLoC.
Tech stack: Flutter 3.x, flutter_bloc ^8.x, equatable ^2.x.
Events: LoginSubmitted(email, password), LogoutRequested, AuthCheckRequested.
States: AuthState(status, user, errorMessage) dùng Equatable + copyWith.
Constraints:
- Events dùng sealed class (Dart 3).
- BlocConsumer cho LoginScreen: builder + listener.
- buildWhen: chỉ rebuild khi status đổi.
- BlocListener: navigate khi success, SnackBar khi failure.
Output: auth_event.dart + auth_state.dart + auth_bloc.dart + login_screen.dart.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn BLoC code sau:
[paste code]

Kiểm tra theo thứ tự:
1. Events: sealed class? Dùng abstract class = không exhaustive.
2. States: Equatable? props list đủ fields? Thiếu → state comparison sai.
3. copyWith: giữ nguyên field cũ? Hay tạo state mới mất field?
4. emit: chỉ trong on<Event> handler? Gọi ngoài = crash.
5. BlocBuilder vs BlocListener: builder cho UI? listener cho side effects?
6. buildWhen/listenWhen: có filter optimization?
7. Tests: dùng blocTest? Verify state sequence?
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter BLoC:
[paste error message]

Code liên quan:
[paste event + state + bloc + widget code gây lỗi]

Context: dùng flutter_bloc ^8.x, có BlocProvider ở trên widget tree.
Cần: (1) Giải thích nguyên nhân, (2) Check emit usage, (3) Check Equatable props, (4) Fix cụ thể.
```

---

## 📌 Quick Reference

### BLoC Lifecycle

```
Bloc created → Initial State
  ↓
Event added → on<Event>() handler called
  ↓
Handler runs → emit(newState)
  ↓
State emitted → BlocBuilder/BlocListener triggered
  ↓
Bloc closed → Resources released
```

### Cheat Sheet

```dart
// Cubit
class MyCubit extends Cubit<MyState> {
  MyCubit() : super(InitialState());
  void doSomething() => emit(NewState());
}

// Bloc
class MyBloc extends Bloc<MyEvent, MyState> {
  MyBloc() : super(InitialState()) {
    on<MyEvent>((event, emit) => emit(NewState()));
  }
}

// Provide
BlocProvider(create: (_) => MyCubit(), child: ...)

// Build UI
BlocBuilder<MyCubit, MyState>(builder: (ctx, state) => ...)

// Side effects
BlocListener<MyCubit, MyState>(listener: (ctx, state) => ...)

// Both
BlocConsumer<MyCubit, MyState>(listener: ..., builder: ...)

// Access (callback)
context.read<MyCubit>().doSomething();

// Access (build)
final state = context.watch<MyCubit>().state;

// Test
blocTest<MyCubit, MyState>(
  'description',
  build: () => MyCubit(),
  act: (cubit) => cubit.doSomething(),
  expect: () => [NewState()],
);
```
