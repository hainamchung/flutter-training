# Buổi 02 — Ví dụ: Dart nâng cao — OOP, Async, Collections

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 2/16** · **6 ví dụ hoàn chỉnh** · **Cập nhật:** 2026-03-31

---

## Mục lục

1. [VD1: Class Hierarchy](#vd1-class-hierarchy)
2. [VD2: Sealed Class & Pattern Matching](#vd2-sealed-class--pattern-matching)
3. [VD3: Future — Async API Call](#vd3-future--async-api-call)
4. [VD4: Stream — Countdown Timer](#vd4-stream--countdown-timer)
5. [VD5: Extension Methods](#vd5-extension-methods)
6. [VD6: Collection Operators](#vd6-collection-operators)

---

## VD1: Class Hierarchy 🔴

### Mục đích

Thực hành class, constructor (default, named, factory), inheritance (`extends`), `@override`, `toString()`.

> **Liên quan tới:** [1. OOP trong Dart 🔴](01-ly-thuyet.md#1-oop-trong-dart)

### Chuẩn bị

```bash
dart create class_demo
cd class_demo
```

### Code

Mở file `bin/class_demo.dart`, xóa nội dung mặc định và thay bằng:

```dart
/// Demo: Class hierarchy — Person, Student
/// File: bin/class_demo.dart
/// Chạy: dart run

// ─── Base Class ──────────────────────────────────────────────

class Person {
  final String name;
  final int age;

  // Default constructor — dùng this. để gán trực tiếp
  Person(this.name, this.age);

  // Named constructor — tạo một "guest" mặc định
  Person.guest()
      : name = 'Guest',
        age = 0;

  // Factory constructor — chứa logic, trả về instance
  // Hữu ích khi parse JSON hoặc cache
  factory Person.fromJson(Map<String, dynamic> json) {
    final name = json['name'] as String? ?? 'Unknown';
    final age = json['age'] as int? ?? 0;
    return Person(name, age);
  }

  // Override toString để print dễ đọc
  @override
  String toString() => 'Person(name: $name, age: $age)';

  // Method instance
  String introduce() => 'Xin chào, tôi là $name, $age tuổi.';
}

// ─── Student extends Person ──────────────────────────────────

class Student extends Person {
  final String studentId;
  final double gpa;

  // Constructor gọi super — truyền name, age lên Person
  Student(super.name, super.age, {required this.studentId, this.gpa = 0.0});

  // Named constructor
  Student.freshman(String name, String studentId)
      : studentId = studentId,
        gpa = 0.0,
        super(name, 18);

  // Factory constructor — parse từ JSON
  factory Student.fromJson(Map<String, dynamic> json) {
    return Student(
      json['name'] as String? ?? 'Unknown',
      json['age'] as int? ?? 18,
      studentId: json['studentId'] as String? ?? 'N/A',
      gpa: (json['gpa'] as num?)?.toDouble() ?? 0.0,
    );
  }

  // Override method của Person
  @override
  String introduce() => '${super.introduce()} MSSV: $studentId, GPA: $gpa';

  @override
  String toString() =>
      'Student(name: $name, age: $age, id: $studentId, gpa: $gpa)';
}

// ─── Teacher extends Person ──────────────────────────────────

class Teacher extends Person {
  final String subject;
  final int yearsOfExperience;

  Teacher(super.name, super.age,
      {required this.subject, this.yearsOfExperience = 0});

  @override
  String introduce() =>
      '${super.introduce()} Tôi dạy môn $subject ($yearsOfExperience năm KN).';

  @override
  String toString() =>
      'Teacher(name: $name, age: $age, subject: $subject, exp: $yearsOfExperience)';
}

// ─── Main ────────────────────────────────────────────────────

void main() {
  print('=== VD1: Class Hierarchy ===\n');

  // 1. Default constructor
  final person = Person('Nguyễn Văn A', 30);
  print('Default: $person');
  print('Giới thiệu: ${person.introduce()}');

  // 2. Named constructor
  final guest = Person.guest();
  print('\nNamed: $guest');
  print('Giới thiệu: ${guest.introduce()}');

  // 3. Factory constructor — từ JSON
  final json = {'name': 'Trần Thị B', 'age': 28};
  final fromJson = Person.fromJson(json);
  print('\nFactory: $fromJson');

  // 4. Student — kế thừa Person
  final student = Student('Lê Văn C', 20, studentId: 'SV001', gpa: 3.5);
  print('\n--- Student ---');
  print('$student');
  print('Giới thiệu: ${student.introduce()}');

  // 5. Student — named constructor
  final freshman = Student.freshman('Phạm Thị D', 'SV002');
  print('\nFreshman: $freshman');

  // 6. Student — factory từ JSON
  final studentJson = {
    'name': 'Hoàng Văn E',
    'age': 21,
    'studentId': 'SV003',
    'gpa': 3.8
  };
  final studentFromJson = Student.fromJson(studentJson);
  print('From JSON: $studentFromJson');

  // 7. Teacher
  final teacher =
      Teacher('Võ Thị F', 35, subject: 'Toán', yearsOfExperience: 10);
  print('\n--- Teacher ---');
  print('$teacher');
  print('Giới thiệu: ${teacher.introduce()}');

  // 8. Polymorphism — danh sách Person chứa cả Student, Teacher
  print('\n--- Polymorphism ---');
  final people = <Person>[person, student, teacher, freshman];
  for (final p in people) {
    print(p.introduce());
  }
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD1: Class Hierarchy ===

Default: Person(name: Nguyễn Văn A, age: 30)
Giới thiệu: Xin chào, tôi là Nguyễn Văn A, 30 tuổi.

Named: Person(name: Guest, age: 0)
Giới thiệu: Xin chào, tôi là Guest, 0 tuổi.

Factory: Person(name: Trần Thị B, age: 28)

--- Student ---
Student(name: Lê Văn C, age: 20, id: SV001, gpa: 3.5)
Giới thiệu: Xin chào, tôi là Lê Văn C, 20 tuổi. MSSV: SV001, GPA: 3.5

Freshman: Student(name: Phạm Thị D, age: 18, id: SV002, gpa: 0.0)
From JSON: Student(name: Hoàng Văn E, age: 21, id: SV003, gpa: 3.8)

--- Teacher ---
Teacher(name: Võ Thị F, age: 35, subject: Toán, exp: 10)
Giới thiệu: Xin chào, tôi là Võ Thị F, 35 tuổi. Tôi dạy môn Toán (10 năm KN).

--- Polymorphism ---
Xin chào, tôi là Nguyễn Văn A, 30 tuổi.
Xin chào, tôi là Lê Văn C, 20 tuổi. MSSV: SV001, GPA: 3.5
Xin chào, tôi là Võ Thị F, 35 tuổi. Tôi dạy môn Toán (10 năm KN).
Xin chào, tôi là Phạm Thị D, 18 tuổi. MSSV: SV002, GPA: 0.0
```

### Giải thích

| Concept                 | Dòng code                                    | Giải thích                                         |
|-------------------------|----------------------------------------------|-----------------------------------------------------|
| Default constructor     | `Person(this.name, this.age)`                | `this.` = gán parameter vào field tự động           |
| Named constructor       | `Person.guest()`                             | Tạo instance với giá trị mặc định                  |
| Factory constructor     | `factory Person.fromJson(...)`               | Logic trước khi tạo instance (parse JSON)           |
| Inheritance             | `class Student extends Person`               | Student kế thừa `name`, `age`, `introduce()`        |
| Super constructor       | `Student(super.name, super.age, ...)`        | Truyền args lên constructor cha                     |
| Override                | `@override String introduce()`               | Ghi đè method cha, dùng `super.introduce()` gộp    |
| Polymorphism            | `List<Person>` chứa Student, Teacher         | Gọi `introduce()` → Dart tự chọn method đúng       |

- 🔗 **FE tương đương:** Dart class ≈ TS class, nhưng `mixin` không có equivalent trong TS — Dart mixin = composition pattern cho code reuse giữa các class không liên quan.

---

## VD2: Sealed Class & Pattern Matching 🟡

### Mục đích

Thực hành `sealed class`, generic `<T>`, exhaustive switch, pattern matching — `Result<T>` pattern giống cách handle API response thực tế.

> **Liên quan tới:** [6. Records & Pattern Matching (Dart 3) 🟡](01-ly-thuyet.md#6-records--pattern-matching-dart-3)

### Chuẩn bị

```bash
dart create sealed_demo
cd sealed_demo
```

### Code

Mở file `bin/sealed_demo.dart`:

```dart
/// Demo: Sealed class + Pattern matching — Result<T>
/// File: bin/sealed_demo.dart
/// Chạy: dart run

// ─── Sealed Class: Result<T> ─────────────────────────────────
// sealed = chỉ có thể extends trong cùng file
// Compiler biết chính xác có bao nhiêu subclass
// → exhaustive switch (không cần default)

sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);

  @override
  String toString() => 'Success(data: $data)';
}

class Failure<T> extends Result<T> {
  final String message;
  final int? code;
  const Failure(this.message, {this.code});

  @override
  String toString() => 'Failure(message: $message, code: $code)';
}

class Loading<T> extends Result<T> {
  const Loading();

  @override
  String toString() => 'Loading()';
}

// ─── Model ───────────────────────────────────────────────────

class User {
  final int id;
  final String name;
  final String email;

  const User({required this.id, required this.name, required this.email});

  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';
}

// ─── "API" giả lập ──────────────────────────────────────────

Future<Result<User>> fetchUser(int id) async {
  // Giả lập API delay 1 giây
  await Future.delayed(const Duration(seconds: 1));

  // Giả lập các trường hợp
  if (id <= 0) {
    return Failure('Invalid user ID: $id', code: 400);
  }
  if (id > 100) {
    return Failure('User not found', code: 404);
  }
  return Success(User(id: id, name: 'User #$id', email: 'user$id@example.com'));
}

// ─── Handler: exhaustive switch ──────────────────────────────

String handleResult(Result<User> result) {
  // Pattern matching — compiler kiểm tra đã cover hết chưa
  return switch (result) {
    Success(data: final user) => '✅ Thành công: ${user.name} (${user.email})',
    Failure(message: final msg, code: final c) =>
      '❌ Lỗi ${c ?? 'unknown'}: $msg',
    Loading() => '⏳ Đang tải...',
    // Không cần default! Compiler biết chỉ có 3 subclass.
    // Thử comment 1 case → compiler báo lỗi.
  };
}

// ─── Guard clause pattern ────────────────────────────────────

String handleWithGuard(Result<User> result) {
  return switch (result) {
    Success(data: final user) when user.id == 1 => '👑 Admin: ${user.name}',
    Success(data: final user) => '👤 User: ${user.name}',
    Failure(code: final c?) when c >= 500 => '🔥 Server error!',
    Failure(message: final msg) => '⚠️ Error: $msg',
    Loading() => '⏳ Loading...',
  };
}

// ─── Main ────────────────────────────────────────────────────

void main() async {
  print('=== VD2: Sealed Class & Pattern Matching ===\n');

  // 1. Loading state
  print(handleResult(const Loading<User>()));

  // 2. Success case
  final result1 = await fetchUser(1);
  print(handleResult(result1));
  print(handleWithGuard(result1)); // Admin case

  // 3. Success case (non-admin)
  final result42 = await fetchUser(42);
  print(handleResult(result42));
  print(handleWithGuard(result42));

  // 4. Failure case — invalid ID
  final resultBad = await fetchUser(-1);
  print(handleResult(resultBad));

  // 5. Failure case — not found
  final resultNotFound = await fetchUser(999);
  print(handleResult(resultNotFound));

  // 6. If-case — pattern matching trong if
  print('\n--- If-case ---');
  final result = await fetchUser(5);
  if (result case Success(data: final user)) {
    print('Got user: ${user.name}');
  }

  // 7. Destructuring result
  print('\n--- Destructuring ---');
  final results = await Future.wait([
    fetchUser(1),
    fetchUser(2),
    fetchUser(-1),
  ]);
  for (final (index, r) in results.indexed) {
    print('Result #$index: ${handleResult(r)}');
  }
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD2: Sealed Class & Pattern Matching ===

⏳ Đang tải...
✅ Thành công: User #1 (user1@example.com)
👑 Admin: User #1
✅ Thành công: User #42 (user42@example.com)
👤 User: User #42
❌ Lỗi 400: Invalid user ID: -1
❌ Lỗi 404: User not found

--- If-case ---
Got user: User #5

--- Destructuring ---
Result #0: ✅ Thành công: User #1 (user1@example.com)
Result #1: ✅ Thành công: User #2 (user2@example.com)
Result #2: ❌ Lỗi 400: Invalid user ID: -1
```

### Giải thích

| Concept                  | Dòng code                                    | Giải thích                                      |
|--------------------------|----------------------------------------------|--------------------------------------------------|
| `sealed class`           | `sealed class Result<T> {}`                  | Chỉ extends trong file này → compiler biết hết  |
| Exhaustive switch        | `switch (result) { ... }`                    | Không cần `default`, compiler check đủ case     |
| Pattern destructuring    | `Success(data: final user)`                  | Rút `data` field, gán vào biến `user`           |
| Guard clause (`when`)    | `... when user.id == 1`                      | Điều kiện thêm sau pattern                      |
| If-case                  | `if (result case Success(...))`              | Pattern matching trong `if`                     |
| Generics                 | `Result<User>`, `Result<T>`                  | Type-safe: Success giữ User, không phải dynamic |

---

## VD3: Future — Async API Call 🔴

### Mục đích

Thực hành `Future`, `async/await`, `Future.delayed`, chained futures, error handling.

> **Liên quan tới:** [4. Async Programming 🔴](01-ly-thuyet.md#4-async-programming)

### Chuẩn bị

```bash
dart create async_demo
cd async_demo
```

### Code

Mở file `bin/async_demo.dart`:

```dart
/// Demo: Future — Simulated API calls
/// File: bin/async_demo.dart
/// Chạy: dart run

// ─── Models ──────────────────────────────────────────────────

class User {
  final int id;
  final String name;
  const User({required this.id, required this.name});

  @override
  String toString() => 'User(id: $id, name: $name)';
}

class Order {
  final int id;
  final int userId;
  final String product;
  final double price;
  const Order({
    required this.id,
    required this.userId,
    required this.product,
    required this.price,
  });

  @override
  String toString() => 'Order(id: $id, product: $product, price: $price)';
}

// ─── Simulated API calls ─────────────────────────────────────

Future<User> fetchUser(int userId) async {
  print('  📡 Đang fetch user $userId...');
  await Future.delayed(const Duration(seconds: 1));

  if (userId <= 0) {
    throw Exception('User ID phải > 0');
  }
  return User(id: userId, name: 'Nguyễn Văn $userId');
}

Future<List<Order>> fetchOrders(int userId) async {
  print('  📡 Đang fetch orders cho user $userId...');
  await Future.delayed(const Duration(seconds: 1));

  return [
    Order(id: 1, userId: userId, product: 'Laptop', price: 25000000),
    Order(id: 2, userId: userId, product: 'Mouse', price: 500000),
    Order(id: 3, userId: userId, product: 'Keyboard', price: 1200000),
  ];
}

Future<double> calculateTotal(List<Order> orders) async {
  print('  🧮 Đang tính tổng...');
  await Future.delayed(const Duration(milliseconds: 500));

  return orders.fold(0.0, (sum, order) => sum + order.price);
}

// ─── Chained async/await ─────────────────────────────────────

Future<void> processUserOrders(int userId) async {
  print('\n--- Bắt đầu xử lý user $userId ---');

  // Bước 1: Fetch user
  final user = await fetchUser(userId);
  print('  ✅ User: $user');

  // Bước 2: Fetch orders (phụ thuộc vào user)
  final orders = await fetchOrders(user.id);
  print('  ✅ Tìm thấy ${orders.length} đơn hàng');
  for (final order in orders) {
    print('     • $order');
  }

  // Bước 3: Calculate total (phụ thuộc vào orders)
  final total = await calculateTotal(orders);
  print('  ✅ Tổng: ${total.toStringAsFixed(0)} VND');
}

// ─── Parallel futures ────────────────────────────────────────

Future<void> fetchMultipleUsers() async {
  print('\n--- Fetch song song nhiều user ---');
  final stopwatch = Stopwatch()..start();

  // Future.wait — chạy song song, nhanh hơn
  final users = await Future.wait([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3),
  ]);

  stopwatch.stop();
  print('  ✅ Fetched ${users.length} users trong ${stopwatch.elapsedMilliseconds}ms');
  for (final user in users) {
    print('     • $user');
  }
}

// ─── Error handling ──────────────────────────────────────────

Future<void> fetchWithErrorHandling() async {
  print('\n--- Error handling ---');

  try {
    final user = await fetchUser(-1); // Sẽ throw exception
    print('User: $user'); // Dòng này không chạy
  } on Exception catch (e) {
    print('  ❌ Caught exception: $e');
  }

  // .then() chain — ít dùng hơn nhưng nên biết
  print('\n--- .then() chain ---');
  await fetchUser(5)
      .then((user) => print('  ✅ .then(): $user'))
      .catchError((Object error) => print('  ❌ .catchError(): $error'));
}

// ─── Timeout ─────────────────────────────────────────────────

Future<void> fetchWithTimeout() async {
  print('\n--- Timeout ---');
  try {
    // Giả lập API chậm — timeout sau 500ms
    final user = await fetchUser(1).timeout(
      const Duration(milliseconds: 500),
      onTimeout: () => throw Exception('Request timeout!'),
    );
    print('  User: $user');
  } on Exception catch (e) {
    print('  ⏰ Timeout: $e');
  }
}

// ─── Main ────────────────────────────────────────────────────

void main() async {
  print('=== VD3: Future — Async API Call ===');

  // 1. Sequential chain: user → orders → total
  await processUserOrders(1);

  // 2. Parallel futures
  await fetchMultipleUsers();

  // 3. Error handling
  await fetchWithErrorHandling();

  // 4. Timeout
  await fetchWithTimeout();

  print('\n=== Hoàn tất! ===');
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD3: Future — Async API Call ===

--- Bắt đầu xử lý user 1 ---
  📡 Đang fetch user 1...
  ✅ User: User(id: 1, name: Nguyễn Văn 1)
  📡 Đang fetch orders cho user 1...
  ✅ Tìm thấy 3 đơn hàng
     • Order(id: 1, product: Laptop, price: 25000000.0)
     • Order(id: 2, product: Mouse, price: 500000.0)
     • Order(id: 3, product: Keyboard, price: 1200000.0)
  🧮 Đang tính tổng...
  ✅ Tổng: 26700000 VND

--- Fetch song song nhiều user ---
  📡 Đang fetch user 1...
  📡 Đang fetch user 2...
  📡 Đang fetch user 3...
  ✅ Fetched 3 users trong ~1000ms
     • User(id: 1, name: Nguyễn Văn 1)
     • User(id: 2, name: Nguyễn Văn 2)
     • User(id: 3, name: Nguyễn Văn 3)

--- Error handling ---
  📡 Đang fetch user -1...
  ❌ Caught exception: Exception: User ID phải > 0

--- .then() chain ---
  📡 Đang fetch user 5...
  ✅ .then(): User(id: 5, name: Nguyễn Văn 5)

--- Timeout ---
  📡 Đang fetch user 1...
  ⏰ Timeout: Exception: Request timeout!

=== Hoàn tất! ===
```

### Giải thích

| Concept            | Dòng code                                           | Giải thích                                     |
|--------------------|------------------------------------------------------|------------------------------------------------|
| `async/await`      | `final user = await fetchUser(1)`                    | Chờ Future resolve, nhận giá trị               |
| Sequential chain   | `await A(); await B(); await C();`                   | Tuần tự — B chờ A xong mới chạy               |
| `Future.wait()`    | `Future.wait([f1, f2, f3])`                          | Chạy song song — nhanh hơn tuần tự            |
| `try/catch`        | `try { await ... } on Exception catch (e) { ... }`  | Bắt lỗi từ Future                              |
| `.then()` chain    | `fetchUser(5).then(...)...catchError(...)`            | Cách cũ, ít readable hơn async/await           |
| `.timeout()`       | `.timeout(Duration(milliseconds: 500))`               | Cancel nếu quá lâu                             |

- 🔗 **FE tương đương:** `Future` ≈ `Promise`, `async/await` gần 1:1. `Future.wait()` ≈ `Promise.all()` — syntax khác nhưng behavior giống hệt.

---

## VD4: Stream — Countdown Timer 🔴

### Mục đích

Thực hành `Stream`, `async*`/`yield`, `StreamController`, `listen`, broadcast stream.

> **Liên quan tới:** [4. Async Programming 🔴](01-ly-thuyet.md#4-async-programming)

### Chuẩn bị

```bash
dart create stream_demo
cd stream_demo
```

### Code

Mở file `bin/stream_demo.dart`:

```dart
/// Demo: Stream — Countdown timer, StreamController
/// File: bin/stream_demo.dart
/// Chạy: dart run

import 'dart:async';

// ─── 1. Stream bằng async* / yield ──────────────────────────

Stream<int> countdown(int from) async* {
  for (int i = from; i >= 0; i--) {
    yield i; // "Phát" giá trị ra stream
    if (i > 0) {
      await Future.delayed(const Duration(milliseconds: 500));
    }
  }
}

// ─── 2. Stream transform ────────────────────────────────────

Stream<String> countdownMessages(int from) async* {
  await for (final count in countdown(from)) {
    if (count == 0) {
      yield '🚀 GO!';
    } else if (count <= 3) {
      yield '🔴 $count...';
    } else {
      yield '⏳ $count';
    }
  }
}

// ─── 3. StreamController ────────────────────────────────────

class EventBus {
  // Broadcast controller — nhiều listener
  final _controller = StreamController<String>.broadcast();

  // Getter cho stream
  Stream<String> get events => _controller.stream;

  // Phát event
  void emit(String event) {
    if (!_controller.isClosed) {
      _controller.add(event);
    }
  }

  // Phát lỗi
  void emitError(String error) {
    if (!_controller.isClosed) {
      _controller.addError(Exception(error));
    }
  }

  // Đóng stream — QUAN TRỌNG: tránh memory leak
  void dispose() {
    _controller.close();
  }
}

// ─── 4. Stream with data processing ─────────────────────────

Stream<double> stockPriceStream() async* {
  final prices = [150.0, 152.5, 148.0, 155.0, 153.5, 160.0];
  for (final price in prices) {
    await Future.delayed(const Duration(milliseconds: 300));
    yield price;
  }
}

// ─── Main ────────────────────────────────────────────────────

void main() async {
  print('=== VD4: Stream — Countdown Timer ===\n');

  // ── Demo 1: Countdown cơ bản ──
  print('--- 1. Countdown cơ bản ---');
  await for (final count in countdown(5)) {
    print('  Count: $count');
  }
  print('  Done!\n');

  // ── Demo 2: Countdown với messages ──
  print('--- 2. Countdown messages ---');
  await for (final msg in countdownMessages(5)) {
    print('  $msg');
  }
  print('');

  // ── Demo 3: StreamController (EventBus) ──
  print('--- 3. EventBus (StreamController) ---');
  final bus = EventBus();

  // Listener 1
  final sub1 = bus.events.listen(
    (event) => print('  [Listener 1] $event'),
    onError: (Object error) => print('  [Listener 1] ERROR: $error'),
    onDone: () => print('  [Listener 1] Stream closed'),
  );

  // Listener 2 — chỉ lắng nghe event chứa "user"
  final sub2 = bus.events
      .where((event) => event.toLowerCase().contains('user'))
      .listen(
        (event) => print('  [Listener 2 - user only] $event'),
      );

  // Phát events
  bus.emit('App started');
  bus.emit('User logged in');
  bus.emit('Data loaded');
  bus.emit('User updated profile');
  bus.emitError('Network timeout');

  // Cho event loop xử lý các events
  await Future.delayed(const Duration(milliseconds: 100));

  // Cleanup
  await sub1.cancel();
  await sub2.cancel();
  bus.dispose();
  print('');

  // ── Demo 4: Stream operators ──
  print('--- 4. Stream operators ---');
  print('  Stock prices:');

  final prices = <double>[];
  await for (final price in stockPriceStream()) {
    prices.add(price);
    final avg = prices.fold(0.0, (sum, p) => sum + p) / prices.length;
    final change = prices.length > 1 ? price - prices[prices.length - 2] : 0.0;
    final arrow = change >= 0 ? '📈' : '📉';
    print('  $arrow \$$price (avg: \$${avg.toStringAsFixed(1)}, '
        'change: ${change >= 0 ? '+' : ''}${change.toStringAsFixed(1)})');
  }

  // Stream → toList, first, last
  print('\n--- 5. Stream utility methods ---');
  final allCounts = await countdown(3).toList();
  print('  toList(): $allCounts');

  final firstCount = await countdown(3).first;
  print('  first: $firstCount');

  final length = await countdown(3).length;
  print('  length: $length');

  print('\n=== Hoàn tất! ===');
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD4: Stream — Countdown Timer ===

--- 1. Countdown cơ bản ---
  Count: 5
  Count: 4
  Count: 3
  Count: 2
  Count: 1
  Count: 0
  Done!

--- 2. Countdown messages ---
  ⏳ 5
  ⏳ 4
  🔴 3...
  🔴 2...
  🔴 1...
  🚀 GO!

--- 3. EventBus (StreamController) ---
  [Listener 1] App started
  [Listener 1] User logged in
  [Listener 2 - user only] User logged in
  [Listener 1] Data loaded
  [Listener 1] User updated profile
  [Listener 2 - user only] User updated profile
  [Listener 1] ERROR: Exception: Network timeout
  [Listener 1] Stream closed

--- 4. Stream operators ---
  Stock prices:
  📈 $150.0 (avg: $150.0, change: +0.0)
  📈 $152.5 (avg: $151.2, change: +2.5)
  📉 $148.0 (avg: $150.2, change: -4.5)
  📈 $155.0 (avg: $151.4, change: +7.0)
  📉 $153.5 (avg: $151.8, change: -1.5)
  📈 $160.0 (avg: $153.2, change: +6.5)

--- 5. Stream utility methods ---
  toList(): [3, 2, 1, 0]
  first: 3
  length: 4

=== Hoàn tất! ===
```

### Giải thích

| Concept              | Dòng code                                        | Giải thích                                      |
|----------------------|---------------------------------------------------|-------------------------------------------------|
| `async*` / `yield`   | `Stream<int> countdown(int from) async* { yield i; }` | Tạo stream generator, `yield` phát từng giá trị |
| `await for`          | `await for (final count in stream)`               | Lắng nghe stream, tự dừng khi stream đóng       |
| `StreamController`   | `StreamController<String>.broadcast()`            | Tạo stream thủ công, broadcast = nhiều listener  |
| `.add()` / `.sink`   | `controller.add('event')`                         | Gửi data vào stream                             |
| `.listen()`          | `stream.listen((data) => ...)`                    | Đăng ký nhận data từ stream                     |
| `.where()`           | `stream.where((e) => e.contains('user'))`         | Filter stream — chỉ nhận event thỏa điều kiện  |
| `.close()`           | `controller.close()`                              | Đóng stream — **QUAN TRỌNG** tránh memory leak  |
| `.cancel()`          | `subscription.cancel()`                           | Hủy đăng ký lắng nghe                           |

- 🔗 **FE tương đương:** `Stream` ≈ RxJS `Observable` — `stream.listen()` ≈ `observable.subscribe()`, `stream.map()` ≈ `pipe(map())`. Stream phổ biến hơn trong Dart so với Observable trong FE.

---

## VD5: Extension Methods 🟢

### Mục đích

Thực hành viết extension methods cho `String`, `int`, `List` — thêm method hữu ích mà không sửa source code gốc.

> **Liên quan tới:** [5. Extension Methods 🟢](01-ly-thuyet.md#5-extension-methods)

### Chuẩn bị

```bash
dart create extension_demo
cd extension_demo
```

### Code

Mở file `bin/extension_demo.dart`:

```dart
/// Demo: Extension methods on String, int, List
/// File: bin/extension_demo.dart
/// Chạy: dart run

// ─── Extension on String ─────────────────────────────────────

extension StringExtension on String {
  /// Viết hoa chữ cái đầu tiên
  String get capitalize {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }

  /// Viết hoa chữ cái đầu mỗi từ
  String get titleCase {
    if (isEmpty) return this;
    return split(' ').map((word) => word.capitalize).join(' ');
  }

  /// Kiểm tra email hợp lệ (basic)
  bool get isEmail {
    return RegExp(r'^[\w\-\.]+@([\w\-]+\.)+[\w\-]{2,4}$').hasMatch(this);
  }

  /// Kiểm tra URL hợp lệ (basic)
  bool get isUrl {
    return RegExp(r'^https?://').hasMatch(this);
  }

  /// Rút gọn text
  String truncate(int maxLength, {String suffix = '...'}) {
    if (length <= maxLength) return this;
    return '${substring(0, maxLength)}$suffix';
  }

  /// Đếm số từ
  int get wordCount {
    if (trim().isEmpty) return 0;
    return trim().split(RegExp(r'\s+')).length;
  }

  /// Reverse string
  String get reversed => split('').reversed.join();
}

// ─── Extension on int ────────────────────────────────────────

extension IntExtension on int {
  /// Chuyển sang Duration
  Duration get seconds => Duration(seconds: this);
  Duration get milliseconds => Duration(milliseconds: this);
  Duration get minutes => Duration(minutes: this);

  /// Format number với dấu phẩy ngăn cách hàng nghìn
  String get formatted {
    final str = abs().toString();
    final result = StringBuffer();
    for (int i = 0; i < str.length; i++) {
      if (i > 0 && (str.length - i) % 3 == 0) {
        result.write(',');
      }
      result.write(str[i]);
    }
    return isNegative ? '-${result.toString()}' : result.toString();
  }

  /// Kiểm tra chẵn/lẻ (readable hơn)
  bool get isPrime {
    if (this < 2) return false;
    for (int i = 2; i * i <= this; i++) {
      if (this % i == 0) return false;
    }
    return true;
  }
}

// ─── Extension on List<T> ────────────────────────────────────

extension ListExtension<T> on List<T> {
  /// Lấy phần tử đầu tiên hoặc null (tránh exception)
  T? get firstOrNull => isEmpty ? null : first;

  /// Lấy phần tử cuối hoặc null
  T? get lastOrNull => isEmpty ? null : last;

  /// Chunk — chia list thành các nhóm nhỏ
  List<List<T>> chunk(int size) {
    final chunks = <List<T>>[];
    for (int i = 0; i < length; i += size) {
      final end = (i + size < length) ? i + size : length;
      chunks.add(sublist(i, end));
    }
    return chunks;
  }
}

// ─── Main ────────────────────────────────────────────────────

void main() {
  print('=== VD5: Extension Methods ===\n');

  // ── String extensions ──
  print('--- String Extensions ---');
  print("'hello'.capitalize: ${'hello'.capitalize}");
  print("'hello world dart'.titleCase: ${'hello world dart'.titleCase}");
  print("'test@email.com'.isEmail: ${'test@email.com'.isEmail}");
  print("'not-email'.isEmail: ${'not-email'.isEmail}");
  print("'https://dart.dev'.isUrl: ${'https://dart.dev'.isUrl}");
  print("'Một chuỗi rất dài'.truncate(8): ${'Một chuỗi rất dài'.truncate(8)}");
  print("'Hello Dart world'.wordCount: ${'Hello Dart world'.wordCount}");
  print("'Dart'.reversed: ${'Dart'.reversed}");

  // ── Int extensions ──
  print('\n--- Int Extensions ---');
  print('5.seconds: ${5.seconds}');
  print('3.minutes: ${3.minutes}');
  print('1500000.formatted: ${1500000.formatted}');
  print('26700000.formatted: ${26700000.formatted}');
  print('7.isPrime: ${7.isPrime}');
  print('10.isPrime: ${10.isPrime}');

  // ── List extensions ──
  print('\n--- List Extensions ---');
  final emptyList = <int>[];
  final numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

  print('emptyList.firstOrNull: ${emptyList.firstOrNull}');
  print('numbers.firstOrNull: ${numbers.firstOrNull}');
  print('numbers.lastOrNull: ${numbers.lastOrNull}');
  print('numbers.chunk(3): ${numbers.chunk(3)}');
  print('numbers.chunk(4): ${numbers.chunk(4)}');
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD5: Extension Methods ===

--- String Extensions ---
'hello'.capitalize: Hello
'hello world dart'.titleCase: Hello World Dart
'test@email.com'.isEmail: true
'not-email'.isEmail: false
'https://dart.dev'.isUrl: true
'Một chuỗi rất dài'.truncate(8): Một chuỗi r...
'Hello Dart world'.wordCount: 3
'Dart'.reversed: traD

--- Int Extensions ---
5.seconds: 0:00:05.000000
3.minutes: 0:03:00.000000
1500000.formatted: 1,500,000
26700000.formatted: 26,700,000
7.isPrime: true
10.isPrime: false

--- List Extensions ---
emptyList.firstOrNull: null
numbers.firstOrNull: 1
numbers.lastOrNull: 10
numbers.chunk(3): [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
numbers.chunk(4): [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]
```

### Giải thích

| Concept             | Dòng code                                    | Giải thích                                       |
|---------------------|----------------------------------------------|---------------------------------------------------|
| `extension on`      | `extension StringExtension on String`        | Mở rộng class String — thêm methods mới          |
| Getter extension    | `String get capitalize`                      | Dùng như property: `'hello'.capitalize`            |
| Method extension    | `String truncate(int maxLength)`             | Dùng như method: `'hello'.truncate(3)`            |
| Generic extension   | `extension ListExtension<T> on List<T>`      | Extension có generic — áp dụng cho mọi List       |
| Scope              | Import file chứa extension → có method       | Extension chỉ available khi file được import      |

---

## VD6: Collection Operators 🟢

### Mục đích

Thực hành `map`, `where`, `fold`, `reduce`, `any`, `every`, spread operator, collection if/for — thao tác dữ liệu ngắn gọn.

> **Liên quan tới:** [3. Collections deep dive 🟡](01-ly-thuyet.md#3-collections-deep-dive)

### Chuẩn bị

```bash
dart create collection_demo
cd collection_demo
```

### Code

Mở file `bin/collection_demo.dart`:

```dart
/// Demo: Collection operators — map, where, fold, spread, collection if/for
/// File: bin/collection_demo.dart
/// Chạy: dart run

// ─── Model ───────────────────────────────────────────────────

class Product {
  final String name;
  final double price;
  final String category;
  final bool inStock;

  const Product({
    required this.name,
    required this.price,
    required this.category,
    this.inStock = true,
  });

  @override
  String toString() =>
      'Product($name, ${price.toStringAsFixed(0)}đ, $category'
      '${inStock ? '' : ', HẾT HÀNG'})';
}

// ─── Sample Data ─────────────────────────────────────────────

final products = [
  Product(name: 'MacBook Pro', price: 52000000, category: 'Laptop'),
  Product(name: 'Dell XPS 13', price: 35000000, category: 'Laptop'),
  Product(name: 'iPhone 16', price: 27000000, category: 'Phone'),
  Product(name: 'Samsung S24', price: 25000000, category: 'Phone', inStock: false),
  Product(name: 'AirPods Pro', price: 6000000, category: 'Accessory'),
  Product(name: 'Magic Mouse', price: 2500000, category: 'Accessory'),
  Product(name: 'iPad Air', price: 18000000, category: 'Tablet'),
];

// ─── Main ────────────────────────────────────────────────────

void main() {
  print('=== VD6: Collection Operators ===\n');

  // ── 1. map — biến đổi ──
  print('--- 1. map ---');
  final names = products.map((p) => p.name).toList();
  print('Tên SP: $names');

  final priceStrings = products
      .map((p) => '${p.name}: ${p.price.toStringAsFixed(0)}đ')
      .toList();
  print('Giá: $priceStrings');

  // ── 2. where — lọc ──
  print('\n--- 2. where ---');
  final laptops = products.where((p) => p.category == 'Laptop').toList();
  print('Laptops: $laptops');

  final affordable = products.where((p) => p.price < 20000000).toList();
  print('Dưới 20tr: $affordable');

  final inStock = products.where((p) => p.inStock).toList();
  print('Còn hàng: ${inStock.length}/${products.length} sản phẩm');

  // ── 3. fold — gộp thành 1 giá trị ──
  print('\n--- 3. fold ---');
  final totalPrice = products.fold(0.0, (sum, p) => sum + p.price);
  print('Tổng giá trị: ${totalPrice.toStringAsFixed(0)}đ');

  final avgPrice = totalPrice / products.length;
  print('Giá trung bình: ${avgPrice.toStringAsFixed(0)}đ');

  // Tính tổng theo category
  final categoryTotals = products.fold<Map<String, double>>(
    {},
    (map, p) {
      map[p.category] = (map[p.category] ?? 0) + p.price;
      return map;
    },
  );
  print('Tổng theo category: $categoryTotals');

  // ── 4. reduce ──
  print('\n--- 4. reduce ---');
  final mostExpensive = products.reduce(
    (a, b) => a.price > b.price ? a : b,
  );
  print('Đắt nhất: $mostExpensive');

  final cheapest = products.reduce(
    (a, b) => a.price < b.price ? a : b,
  );
  print('Rẻ nhất: $cheapest');

  // ── 5. any & every ──
  print('\n--- 5. any & every ---');
  print('Có SP > 50tr? ${products.any((p) => p.price > 50000000)}');
  print('Tất cả còn hàng? ${products.every((p) => p.inStock)}');
  print('Tất cả > 0đ? ${products.every((p) => p.price > 0)}');

  // ── 6. Spread operator (...) ──
  print('\n--- 6. Spread operator ---');
  final baseMenu = ['Home', 'Products'];
  final isAdmin = true;
  final isLoggedIn = true;

  final menu = [
    ...baseMenu,
    if (isLoggedIn) 'Profile',
    if (isAdmin) 'Admin',
    'About',
  ];
  print('Menu: $menu');

  // Merge maps
  final defaults = {'theme': 'light', 'lang': 'vi', 'page': 1};
  final overrides = {'theme': 'dark', 'page': 2};
  final settings = {...defaults, ...overrides};
  print('Settings: $settings');

  // ── 7. Collection if/for ──
  print('\n--- 7. Collection if/for ---');
  final showOutOfStock = false;
  final displayProducts = [
    for (final p in products)
      if (showOutOfStock || p.inStock)
        '${p.name} (${p.price.toStringAsFixed(0)}đ)',
  ];
  print('Display: $displayProducts');

  // ── 8. Chaining operators ──
  print('\n--- 8. Chain operators ---');
  final report = products
      .where((p) => p.inStock)           // Còn hàng
      .where((p) => p.price >= 10000000) // >= 10tr
      .map((p) => p.name)                // Lấy tên
      .toList()
    ..sort();                             // Sắp xếp A-Z

  print('Còn hàng, >= 10tr, sorted: $report');

  // ── 9. Set operations ──
  print('\n--- 9. Set operations ---');
  final frontendStack = {'Dart', 'Flutter', 'HTML', 'CSS'};
  final mobileStack = {'Dart', 'Flutter', 'Kotlin', 'Swift'};
  print('Intersection: ${frontendStack.intersection(mobileStack)}');
  print('Union: ${frontendStack.union(mobileStack)}');
  print('Only frontend: ${frontendStack.difference(mobileStack)}');

  // ── 10. Map operations ──
  print('\n--- 10. Map operations ---');
  final scores = {'Alice': 90, 'Bob': 75, 'Charlie': 85, 'Diana': 95};

  // Filter by value
  final honors = Map.fromEntries(
    scores.entries.where((e) => e.value >= 85),
  );
  print('Honors (>=85): $honors');

  // Transform values
  final grades = scores.map((name, score) {
    final grade = switch (score) {
      >= 90 => 'A',
      >= 80 => 'B',
      >= 70 => 'C',
      _ => 'D',
    };
    return MapEntry(name, grade);
  });
  print('Grades: $grades');
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== VD6: Collection Operators ===

--- 1. map ---
Tên SP: [MacBook Pro, Dell XPS 13, iPhone 16, Samsung S24, AirPods Pro, Magic Mouse, iPad Air]
Giá: [MacBook Pro: 52000000đ, Dell XPS 13: 35000000đ, ...]

--- 2. where ---
Laptops: [Product(MacBook Pro, 52000000đ, Laptop), Product(Dell XPS 13, 35000000đ, Laptop)]
Dưới 20tr: [Product(AirPods Pro, ...), Product(Magic Mouse, ...), Product(iPad Air, ...)]
Còn hàng: 6/7 sản phẩm

--- 3. fold ---
Tổng giá trị: 165500000đ
Giá trung bình: 23642857đ
Tổng theo category: {Laptop: 87000000.0, Phone: 52000000.0, Accessory: 8500000.0, Tablet: 18000000.0}

--- 4. reduce ---
Đắt nhất: Product(MacBook Pro, 52000000đ, Laptop)
Rẻ nhất: Product(Magic Mouse, 2500000đ, Accessory)

--- 5. any & every ---
Có SP > 50tr? true
Tất cả còn hàng? false
Tất cả > 0đ? true

--- 6. Spread operator ---
Menu: [Home, Products, Profile, Admin, About]
Settings: {theme: dark, lang: vi, page: 2}

--- 7. Collection if/for ---
Display: [MacBook Pro (52000000đ), Dell XPS 13 (35000000đ), iPhone 16 (27000000đ), AirPods Pro (6000000đ), Magic Mouse (2500000đ), iPad Air (18000000đ)]

--- 8. Chain operators ---
Còn hàng, >= 10tr, sorted: [Dell XPS 13, MacBook Pro, iPhone 16, iPad Air]

--- 9. Set operations ---
Intersection: {Dart, Flutter}
Union: {Dart, Flutter, HTML, CSS, Kotlin, Swift}
Only frontend: {HTML, CSS}

--- 10. Map operations ---
Honors (>=85): {Alice: 90, Charlie: 85, Diana: 95}
Grades: {Alice: A, Bob: C, Charlie: B, Diana: A}
```

### Giải thích

| Operator          | Mô tả                                        | Hay dùng trong Flutter                          |
|-------------------|-----------------------------------------------|-------------------------------------------------|
| `map`             | Biến đổi mỗi phần tử                         | Product → Widget, JSON → Model                  |
| `where`           | Lọc theo điều kiện                            | Filter danh sách hiển thị                       |
| `fold`            | Gộp thành 1 giá trị (có giá trị ban đầu)     | Tính tổng, thống kê                             |
| `reduce`          | Gộp (không có giá trị ban đầu)                | Tìm min/max                                     |
| `any` / `every`   | Check điều kiện                               | Validate form, check permission                 |
| `...` spread      | Trải collection vào collection khác            | Merge widget lists                              |
| Collection if/for | Thêm phần tử có điều kiện / vòng lặp         | Build widget list trong Flutter (`children: []`) |
| `..sort()`        | Cascade operator + sort                       | Chain operations                                |
