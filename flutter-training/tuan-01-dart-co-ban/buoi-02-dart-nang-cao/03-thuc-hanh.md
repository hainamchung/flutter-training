# Buổi 02 — Thực hành: Dart nâng cao — OOP, Async, Collections

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 2/16** · **3 bài tập + 3 câu hỏi thảo luận** · **Cập nhật:** 2026-03-31

---

## Mục lục

1. [BT1 ⭐ Class Hierarchy](#bt1--class-hierarchy)
2. [BT2 ⭐⭐ Async Chain](#bt2--async-chain)
3. [BT3 ⭐⭐⭐ Result&lt;T&gt; Pattern](#bt3--resultt-pattern)
4. [Câu hỏi thảo luận](#câu-hỏi-thảo-luận)

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ TypeScript/JavaScript**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| TypeScript/JS Habit | Dart Reality | Bài tập liên quan |
|---------------------|--------------|---------------------|
| Interface = structural typing (duck typing) | Dart dùng **nominal typing** — phải `implements` explicit, cùng shape ≠ cùng type | BT1 |
| `Promise.all()` cho parallel async | Dart: `Future.wait()` — tương đương nhưng tên khác | BT2 |
| RxJS ít dùng trong FE thông thường | **Stream** là first-class trong Dart — phải nắm vững map/where/listen | BT2, BT3 |
| `try/catch` bắt mọi Error | Dart: dùng `on SpecificException catch(e)` để bắt đúng type | BT3 |

---

## BT1 ⭐ Class Hierarchy 🔴

### Thông tin

| Mục           | Chi tiết                                     |
|---------------|----------------------------------------------|
| **Loại project**    | Dart CLI                                     |
| **Độ khó**    | ⭐ Cơ bản                                    |
| **Thời gian** | ~30 phút                                     |
| **Output**    | Terminal                                     |
| **Concepts**  | Class, constructor, extends, override, mixin |

### Yêu cầu

Xây dựng hệ thống class mô tả nhân viên trường học:

1. **Base class `Person`**: `name` (String), `age` (int), method `introduce()` trả về String
2. **Class `Student extends Person`**: thêm `studentId`, `gpa`, override `introduce()`
3. **Class `Teacher extends Person`**: thêm `subject`, `yearsOfExperience`, override `introduce()`
4. **Mixin `Printable`**: method `printInfo()` — in thông tin dạng đẹp ra console
5. Tạo danh sách `List<Person>` chứa cả Student và Teacher → dùng `for` loop gọi `introduce()` (polymorphism)

### Các bước thực hiện

#### Bước 1: Tạo project

```bash
dart create bt1_class_hierarchy
cd bt1_class_hierarchy
```

#### Bước 2: Code

Mở `bin/bt1_class_hierarchy.dart` và implement:

```dart
/// BT1: Class Hierarchy
/// File: bin/bt1_class_hierarchy.dart

// TODO: 1. Tạo mixin Printable
//   - method printInfo() in ra "=== <tên class> ===" và gọi introduce()

// TODO: 2. Tạo class Person
//   - fields: name (String), age (int)
//   - default constructor
//   - named constructor Person.anonymous() — name = 'Ẩn danh', age = 0
//   - method introduce() trả về 'Xin chào, tôi là $name, $age tuổi.'
//   - override toString()

// TODO: 3. Tạo class Student extends Person with Printable
//   - fields: studentId (String), gpa (double)
//   - constructor gọi super
//   - override introduce() — thêm MSSV và GPA
//   - factory Student.fromJson(Map<String, dynamic> json)

// TODO: 4. Tạo class Teacher extends Person with Printable
//   - fields: subject (String), yearsOfExperience (int)
//   - constructor gọi super
//   - override introduce() — thêm môn dạy và kinh nghiệm

// TODO: 5. Hàm main()
//   - Tạo 2 Student, 2 Teacher
//   - Thêm vào List<Person>
//   - Duyệt list, gọi introduce() cho mỗi person
//   - Với person là Printable → gọi printInfo()

void main() {
  // TODO: implement
}
```

#### Bước 3: Chạy

```bash
dart run
```

### Output mong đợi

```
=== Danh sách nhân viên trường học ===

Xin chào, tôi là Nguyễn Văn A, 20 tuổi. MSSV: SV001, GPA: 3.5
=== Student ===
Xin chào, tôi là Nguyễn Văn A, 20 tuổi. MSSV: SV001, GPA: 3.5

Xin chào, tôi là Trần Thị B, 22 tuổi. MSSV: SV002, GPA: 3.8
=== Student ===
Xin chào, tôi là Trần Thị B, 22 tuổi. MSSV: SV002, GPA: 3.8

Xin chào, tôi là Lê Văn C, 35 tuổi. Dạy môn Toán (10 năm KN).
=== Teacher ===
Xin chào, tôi là Lê Văn C, 35 tuổi. Dạy môn Toán (10 năm KN).

Xin chào, tôi là Phạm Thị D, 40 tuổi. Dạy môn Văn (15 năm KN).
=== Teacher ===
Xin chào, tôi là Phạm Thị D, 40 tuổi. Dạy môn Văn (15 năm KN).
```

### Tiêu chí hoàn thành

| #  | Tiêu chí                                                    | ✅ |
|----|--------------------------------------------------------------|----|
| 1  | `Person` có default constructor và named constructor         | ☐  |
| 2  | `Student` extends `Person`, override `introduce()`           | ☐  |
| 3  | `Teacher` extends `Person`, override `introduce()`           | ☐  |
| 4  | `Printable` mixin được `with` vào Student và Teacher         | ☐  |
| 5  | `List<Person>` chứa cả Student và Teacher (polymorphism)     | ☐  |
| 6  | `factory Student.fromJson()` hoạt động                       | ☐  |
| 7  | `dart run` chạy không lỗi, output đúng                      | ☐  |

### Gợi ý

<details>
<summary>💡 Gợi ý 1: Mixin syntax</summary>

```dart
mixin Printable {
  String introduce(); // abstract — class dùng mixin phải có method này

  void printInfo() {
    print('=== ${runtimeType} ===');
    print(introduce());
  }
}
```

</details>

<details>
<summary>💡 Gợi ý 2: Kiểm tra type trong runtime</summary>

```dart
for (final person in people) {
  print(person.introduce());
  if (person is Printable) {
    (person as Printable).printInfo();
  }
}
```

</details>

<details>
<summary>🔑 Lời giải hoàn chỉnh</summary>

```dart
mixin Printable {
  String introduce();

  void printInfo() {
    print('=== ${runtimeType} ===');
    print(introduce());
  }
}

class Person {
  final String name;
  final int age;

  Person(this.name, this.age);
  Person.anonymous()
      : name = 'Ẩn danh',
        age = 0;

  String introduce() => 'Xin chào, tôi là $name, $age tuổi.';

  @override
  String toString() => 'Person(name: $name, age: $age)';
}

class Student extends Person with Printable {
  final String studentId;
  final double gpa;

  Student(super.name, super.age,
      {required this.studentId, this.gpa = 0.0});

  factory Student.fromJson(Map<String, dynamic> json) {
    return Student(
      json['name'] as String? ?? 'Unknown',
      json['age'] as int? ?? 18,
      studentId: json['studentId'] as String? ?? 'N/A',
      gpa: (json['gpa'] as num?)?.toDouble() ?? 0.0,
    );
  }

  @override
  String introduce() =>
      '${super.introduce()} MSSV: $studentId, GPA: $gpa';
}

class Teacher extends Person with Printable {
  final String subject;
  final int yearsOfExperience;

  Teacher(super.name, super.age,
      {required this.subject, this.yearsOfExperience = 0});

  @override
  String introduce() =>
      '${super.introduce()} Dạy môn $subject ($yearsOfExperience năm KN).';
}

void main() {
  print('=== Danh sách nhân viên trường học ===\n');

  final people = <Person>[
    Student('Nguyễn Văn A', 20, studentId: 'SV001', gpa: 3.5),
    Student('Trần Thị B', 22, studentId: 'SV002', gpa: 3.8),
    Teacher('Lê Văn C', 35, subject: 'Toán', yearsOfExperience: 10),
    Teacher('Phạm Thị D', 40, subject: 'Văn', yearsOfExperience: 15),
  ];

  for (final person in people) {
    print(person.introduce());
    if (person is Printable) {
      (person as Printable).printInfo();
    }
    print('');
  }
}
```

</details>

---

## BT2 ⭐⭐ Async Chain 🔴

### Thông tin

| Mục           | Chi tiết                                                   |
|---------------|------------------------------------------------------------|
| **Loại project**    | Dart CLI                                                   |
| **Độ khó**    | ⭐⭐ Trung bình                                             |
| **Thời gian** | ~40 phút                                                   |
| **Output**    | Terminal                                                   |
| **Concepts**  | Future, async/await, Future.wait, error handling, chaining |

### Yêu cầu

Xây dựng flow xử lý đơn hàng giả lập:

1. **`fetchUser(int id)`** → trả về `Future<User>` (delay 1s). Nếu `id <= 0` → throw Exception.
2. **`fetchOrders(int userId)`** → trả về `Future<List<Order>>` (delay 1s). Mỗi order có `productName`, `price`, `quantity`.
3. **`calculateDiscount(double total)`** → trả về `Future<double>` (delay 500ms). Giảm giá 10% nếu total > 1 triệu.
4. **`processOrder(int userId)`** → chain 3 hàm trên tuần tự: fetch user → fetch orders → tính tổng → tính discount → in kết quả.
5. **Main**: gọi `processOrder()` cho user hợp lệ và user không hợp lệ, xử lý error.
6. **Bonus**: dùng `Future.wait()` gọi song song cho 3 user.

### Các bước thực hiện

#### Bước 1: Tạo project

```bash
dart create bt2_async_chain
cd bt2_async_chain
```

#### Bước 2: Code

Mở `bin/bt2_async_chain.dart` và implement:

```dart
/// BT2: Async Chain — Fetch user → orders → total → discount
/// File: bin/bt2_async_chain.dart

// TODO: 1. Tạo class User (id, name)
// TODO: 2. Tạo class Order (id, productName, price, quantity)
// TODO: 3. Implement fetchUser(int id) — Future.delayed 1s
//          Nếu id <= 0 → throw Exception
// TODO: 4. Implement fetchOrders(int userId) — Future.delayed 1s
//          Trả về 3 orders giả lập
// TODO: 5. Implement calculateDiscount(double total) — Future.delayed 500ms
//          total > 1,000,000 → giảm 10%
// TODO: 6. Implement processOrder(int userId) — chain tất cả
// TODO: 7. Main:
//          a. processOrder(1) — happy path
//          b. processOrder(-1) — error path
//          c. Future.wait() cho 3 users song song

void main() async {
  // TODO: implement
}
```

#### Bước 3: Chạy

```bash
dart run
```

### Output mong đợi

```
=== BT2: Async Chain ===

--- Process user 1 ---
📡 Fetching user 1...
✅ User: User(1, Nguyễn Văn 1)
📡 Fetching orders for user 1...
✅ 3 orders found:
   • Laptop x1 — 25,000,000đ
   • Mouse x2 — 1,000,000đ
   • Keyboard x1 — 1,200,000đ
🧮 Total: 27,200,000đ
💰 After discount (10%): 24,480,000đ

--- Process user -1 (error) ---
📡 Fetching user -1...
❌ Error: Exception: Invalid user ID: -1

--- Parallel fetch ---
📡 Fetching user 1...
📡 Fetching user 2...
📡 Fetching user 3...
✅ All users fetched in ~1 second (parallel!)
```

### Tiêu chí hoàn thành

| #  | Tiêu chí                                                  | ✅ |
|----|------------------------------------------------------------|----|
| 1  | `fetchUser()` hoạt động với delay giả lập                 | ☐  |
| 2  | `fetchOrders()` trả về list orders                        | ☐  |
| 3  | `calculateDiscount()` tính đúng discount                  | ☐  |
| 4  | Chaining tuần tự: user → orders → total → discount        | ☐  |
| 5  | Error handling: user ID không hợp lệ → catch đúng         | ☐  |
| 6  | `Future.wait()` chạy song song (thời gian ~1s, không ~3s) | ☐  |
| 7  | `dart run` chạy không lỗi                                 | ☐  |

### Gợi ý

<details>
<summary>💡 Gợi ý 1: Chaining tuần tự</summary>

```dart
Future<void> processOrder(int userId) async {
  final user = await fetchUser(userId);
  final orders = await fetchOrders(user.id);
  final total = orders.fold(0.0, (sum, o) => sum + o.price * o.quantity);
  final discounted = await calculateDiscount(total);
  print('Tổng sau giảm giá: $discounted');
}
```

</details>

<details>
<summary>💡 Gợi ý 2: Future.wait song song</summary>

```dart
final stopwatch = Stopwatch()..start();
final users = await Future.wait([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3),
]);
stopwatch.stop();
print('Done in ${stopwatch.elapsedMilliseconds}ms');
```

</details>

<details>
<summary>🔑 Lời giải hoàn chỉnh</summary>

```dart
class User {
  final int id;
  final String name;
  const User({required this.id, required this.name});
  @override
  String toString() => 'User($id, $name)';
}

class Order {
  final int id;
  final String productName;
  final double price;
  final int quantity;
  const Order({
    required this.id,
    required this.productName,
    required this.price,
    required this.quantity,
  });
  double get total => price * quantity;
  @override
  String toString() =>
      '$productName x$quantity — ${total.toStringAsFixed(0)}đ';
}

Future<User> fetchUser(int id) async {
  print('📡 Fetching user $id...');
  await Future.delayed(const Duration(seconds: 1));
  if (id <= 0) throw Exception('Invalid user ID: $id');
  return User(id: id, name: 'Nguyễn Văn $id');
}

Future<List<Order>> fetchOrders(int userId) async {
  print('📡 Fetching orders for user $userId...');
  await Future.delayed(const Duration(seconds: 1));
  return [
    Order(id: 1, productName: 'Laptop', price: 25000000, quantity: 1),
    Order(id: 2, productName: 'Mouse', price: 500000, quantity: 2),
    Order(id: 3, productName: 'Keyboard', price: 1200000, quantity: 1),
  ];
}

Future<double> calculateDiscount(double total) async {
  print('🧮 Calculating discount...');
  await Future.delayed(const Duration(milliseconds: 500));
  if (total > 1000000) return total * 0.9;
  return total;
}

Future<void> processOrder(int userId) async {
  final user = await fetchUser(userId);
  print('✅ User: $user');

  final orders = await fetchOrders(user.id);
  print('✅ ${orders.length} orders found:');
  for (final o in orders) {
    print('   • $o');
  }

  final total = orders.fold(0.0, (sum, o) => sum + o.total);
  print('🧮 Total: ${total.toStringAsFixed(0)}đ');

  final discounted = await calculateDiscount(total);
  final pct = ((total - discounted) / total * 100).round();
  print('💰 After discount ($pct%): ${discounted.toStringAsFixed(0)}đ');
}

void main() async {
  print('=== BT2: Async Chain ===\n');

  // Happy path
  print('--- Process user 1 ---');
  await processOrder(1);

  // Error path
  print('\n--- Process user -1 (error) ---');
  try {
    await processOrder(-1);
  } on Exception catch (e) {
    print('❌ Error: $e');
  }

  // Parallel
  print('\n--- Parallel fetch ---');
  final sw = Stopwatch()..start();
  final users = await Future.wait([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3),
  ]);
  sw.stop();
  print('✅ All ${users.length} users fetched in '
      '${sw.elapsedMilliseconds}ms (parallel!)');
}
```

</details>

---

## BT3 ⭐⭐⭐ Result&lt;T&gt; Pattern 🟡

### Thông tin

| Mục           | Chi tiết                                                              |
|---------------|-----------------------------------------------------------------------|
| **Loại project**    | Dart CLI                                                              |
| **Độ khó**    | ⭐⭐⭐ Nâng cao                                                        |
| **Thời gian** | ~45 phút                                                              |
| **Output**    | Terminal                                                              |
| **Concepts**  | Sealed class, generics, pattern matching, exhaustive switch, error handling |

### Yêu cầu

Implement `Result<T>` pattern — cách handle API response phổ biến trong Flutter thực tế:

1. **`sealed class Result<T>`** với 3 subclass:
   - `Success<T>` — chứa `T data`
   - `Failure<T>` — chứa `String message` và `int? code`
   - `Loading<T>` — không chứa data
2. **Extension method trên `Result<T>`**:
   - `isSuccess` → bool
   - `isFailure` → bool
   - `dataOrNull` → T?
   - `map<R>(R Function(T) fn)` → Result<R>
3. **Simulated API functions** dùng `Result<T>` thay vì throw:
   - `login(String email, String password)` → `Future<Result<User>>`
   - `fetchProfile(int userId)` → `Future<Result<Profile>>`
4. **Handler functions** dùng exhaustive switch + pattern matching:
   - `handleLogin()` → xử lý đăng nhập
   - `handleProfile()` → chain login → fetchProfile
5. **Main**: demo tất cả scenarios (success, wrong password, network error, chaining)

### Các bước thực hiện

#### Bước 1: Tạo project

```bash
dart create bt3_result_pattern
cd bt3_result_pattern
```

#### Bước 2: Code

Mở `bin/bt3_result_pattern.dart` và implement:

```dart
/// BT3: Result<T> Pattern — Sealed class + Pattern matching
/// File: bin/bt3_result_pattern.dart

// TODO: 1. Sealed class Result<T> (Success, Failure, Loading)

// TODO: 2. Extension on Result<T>:
//          isSuccess, isFailure, dataOrNull, map<R>()

// TODO: 3. Models: User (id, name, email), Profile (userId, avatar, bio)

// TODO: 4. login(email, password) → Future<Result<User>>
//          - "admin@test.com" + "123456" → Success
//          - Sai password → Failure(code: 401)
//          - Email trống → Failure(code: 400)

// TODO: 5. fetchProfile(userId) → Future<Result<Profile>>
//          - userId > 0 → Success
//          - Else → Failure(code: 404)

// TODO: 6. handleLogin() — exhaustive switch xử lý kết quả

// TODO: 7. handleChain() — login → dùng Result để fetchProfile

// TODO: 8. main() — demo tất cả scenarios

void main() async {
  // TODO: implement
}
```

#### Bước 3: Chạy

```bash
dart run
```

### Output mong đợi

```
=== BT3: Result<T> Pattern ===

--- Login: admin@test.com ---
✅ Đăng nhập thành công: User(1, Admin)

--- Login: wrong password ---
❌ Lỗi 401: Sai mật khẩu

--- Login: empty email ---
❌ Lỗi 400: Email không được để trống

--- Chain: login → profile ---
✅ Login: User(1, Admin)
✅ Profile: Profile(userId: 1, avatar: avatar_1.png, bio: Hello!)

--- Result extensions ---
isSuccess: true
dataOrNull: User(1, Admin)
map: Success(data: ADMIN)
```

### Tiêu chí hoàn thành

| #  | Tiêu chí                                                      | ✅ |
|----|----------------------------------------------------------------|----|
| 1  | `sealed class Result<T>` với Success, Failure, Loading         | ☐  |
| 2  | Exhaustive switch — không cần `default` case                   | ☐  |
| 3  | Pattern matching destructure: `Success(data: final user)`      | ☐  |
| 4  | Extension methods trên Result<T> (isSuccess, dataOrNull, map)  | ☐  |
| 5  | Login xử lý đúng 3 scenarios (success, wrong pass, empty email)| ☐  |
| 6  | Chain login → fetchProfile sử dụng Result, không throw         | ☐  |
| 7  | `dart run` chạy không lỗi                                      | ☐  |

### Gợi ý

<details>
<summary>💡 Gợi ý 1: Extension method map()</summary>

```dart
extension ResultExtension<T> on Result<T> {
  Result<R> map<R>(R Function(T data) fn) {
    return switch (this) {
      Success(data: final d) => Success(fn(d)),
      Failure(message: final m, code: final c) => Failure(m, code: c),
      Loading() => Loading(),
    };
  }
}
```

</details>

<details>
<summary>💡 Gợi ý 2: Chain pattern — không dùng throw</summary>

```dart
Future<Result<Profile>> loginAndFetchProfile(
    String email, String password) async {
  final loginResult = await login(email, password);
  return switch (loginResult) {
    Success(data: final user) => await fetchProfile(user.id),
    Failure(message: final m, code: final c) => Failure(m, code: c),
    Loading() => Loading(),
  };
}
```

</details>

<details>
<summary>🔑 Lời giải hoàn chỉnh</summary>

```dart
// ─── Result<T> ───
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
  String toString() => 'Failure($code: $message)';
}
class Loading<T> extends Result<T> {
  const Loading();
  @override
  String toString() => 'Loading()';
}

// ─── Extensions ───
extension ResultExtension<T> on Result<T> {
  bool get isSuccess => this is Success<T>;
  bool get isFailure => this is Failure<T>;
  T? get dataOrNull => switch (this) {
    Success(data: final d) => d,
    _ => null,
  };
  Result<R> map<R>(R Function(T) fn) => switch (this) {
    Success(data: final d) => Success(fn(d)),
    Failure(message: final m, code: final c) => Failure(m, code: c),
    Loading() => Loading(),
  };
}

// ─── Models ───
class User {
  final int id;
  final String name;
  final String email;
  const User({required this.id, required this.name, required this.email});
  @override
  String toString() => 'User($id, $name)';
}

class Profile {
  final int userId;
  final String avatar;
  final String bio;
  const Profile(
      {required this.userId, required this.avatar, required this.bio});
  @override
  String toString() =>
      'Profile(userId: $userId, avatar: $avatar, bio: $bio)';
}

// ─── API ───
Future<Result<User>> login(String email, String password) async {
  await Future.delayed(const Duration(milliseconds: 500));
  if (email.isEmpty) return Failure('Email không được để trống', code: 400);
  if (email == 'admin@test.com' && password == '123456') {
    return Success(User(id: 1, name: 'Admin', email: email));
  }
  return Failure('Sai mật khẩu', code: 401);
}

Future<Result<Profile>> fetchProfile(int userId) async {
  await Future.delayed(const Duration(milliseconds: 500));
  if (userId <= 0) return Failure('User not found', code: 404);
  return Success(
      Profile(userId: userId, avatar: 'avatar_$userId.png', bio: 'Hello!'));
}

// ─── Handler ───
String handleResult<T>(Result<T> result) => switch (result) {
  Success(data: final d) => '✅ Thành công: $d',
  Failure(message: final m, code: final c) => '❌ Lỗi $c: $m',
  Loading() => '⏳ Đang tải...',
};

Future<Result<Profile>> loginAndFetchProfile(
    String email, String password) async {
  final loginResult = await login(email, password);
  return switch (loginResult) {
    Success(data: final user) => await fetchProfile(user.id),
    Failure(message: final m, code: final c) => Failure(m, code: c),
    Loading() => Loading(),
  };
}

void main() async {
  print('=== BT3: Result<T> Pattern ===\n');

  print('--- Login: admin@test.com ---');
  final r1 = await login('admin@test.com', '123456');
  print(handleResult(r1));

  print('\n--- Login: wrong password ---');
  final r2 = await login('admin@test.com', 'wrong');
  print(handleResult(r2));

  print('\n--- Login: empty email ---');
  final r3 = await login('', '123456');
  print(handleResult(r3));

  print('\n--- Chain: login → profile ---');
  final r4 = await login('admin@test.com', '123456');
  print('Login: ${handleResult(r4)}');
  final profileResult =
      await loginAndFetchProfile('admin@test.com', '123456');
  print('Profile: ${handleResult(profileResult)}');

  print('\n--- Result extensions ---');
  print('isSuccess: ${r1.isSuccess}');
  print('dataOrNull: ${r1.dataOrNull}');
  final mapped = r1.map((user) => user.name.toUpperCase());
  print('map: $mapped');
}
```

</details>

---

## Câu hỏi thảo luận

> Thảo luận trong nhóm hoặc tự trả lời để củng cố kiến thức.

### Câu 1: Mixin vs Inheritance — khi nào dùng cái nào?

**Tình huống:** Bạn đang xây dựng game đơn giản. Có các entity:
- `Bird` — có thể bay, có thể hót
- `Fish` — có thể bơi
- `Duck` — có thể bay VÀ bơi
- `Penguin` — có thể bơi, KHÔNG bay

**Câu hỏi:**
1. Nếu chỉ dùng `extends` (single inheritance), bạn gặp vấn đề gì?
2. Thiết kế với mixin sẽ trông như thế nào? Viết skeleton code.
3. Khi nào **KHÔNG** nên dùng mixin?

---

### Câu 2: Future vs Stream — chọn đúng tool

**Cho các tình huống sau, bạn sẽ dùng `Future<T>` hay `Stream<T>`? Giải thích tại sao.**

| Tình huống                                    | Future hay Stream? | Tại sao? |
|-----------------------------------------------|---------------------|----------|
| Gọi REST API lấy thông tin user               |                     |          |
| Lắng nghe vị trí GPS realtime                 |                     |          |
| Upload file lên server                        |                     |          |
| Nhận message từ WebSocket chat                |                     |          |
| Đọc file JSON từ ổ đĩa                       |                     |          |
| Đếm ngược 10 → 0 hiển thị trên UI            |                     |          |
| Lắng nghe thay đổi data từ Firestore realtime |                     |          |

---

### Câu 3: final vs const — quyết định trong dự án thực tế

**Câu hỏi:**
1. `final` khác `const` ở điểm nào cơ bản nhất?
2. Tại sao Flutter khuyến khích dùng `const` cho Widget khi có thể?
3. Cho đoạn code sau, dòng nào lỗi? Tại sao?

```dart
final a = DateTime.now();   // 1
const b = DateTime.now();   // 2
final c = [1, 2, 3];       // 3
const d = [1, 2, 3];       // 4
c.add(4);                   // 5
d.add(4);                   // 6
```

4. Trong dự án Flutter thực tế, `const` constructor quan trọng thế nào cho performance?

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 1:** Focus vào đọc hiểu code AI gen và verify tính đúng đắn.

### AI-BT1: Gen Sealed Class + Pattern Matching cho Error Handling ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Sealed class, pattern matching, exhaustive switch (Dart 3 OOP).
- **Task thực tế:** PM yêu cầu "app phải hiển thị error message phù hợp cho từng loại lỗi — lỗi mạng hiện nút Retry, lỗi auth redirect về login, lỗi server hiện mã lỗi". Cần error handling type-safe, compiler phải bắt được nếu quên handle 1 loại lỗi.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần implement domain error handling cho Flutter app dùng sealed class Dart 3.
Context: App gọi REST API, backend trả về các HTTP status code khác nhau.
Tech stack: Dart 3.x, sound null safety.
Constraints:
- Sealed class AppFailure với 4 subclass: NetworkFailure, ServerFailure, AuthFailure, ParseFailure.
- NetworkFailure: field message (String), isTimeout (bool).
- ServerFailure: field statusCode (int), serverMessage (String?).
- AuthFailure: field reason (enum AuthFailReason { tokenExpired, unauthorized, accountLocked }).
- ParseFailure: field rawData (String), expectedType (Type).
- Factory: AppFailure.fromStatusCode(int code, {String? body}) → mapping logic: 401→Auth, 5xx→Server, etc.
- Extension: String get userFriendlyMessage → message tiếng Việt cho mỗi loại.
- 1 hàm Widget buildErrorWidget(AppFailure failure) dùng exhaustive switch expression, trả về mô tả widget phù hợp.
Output: 1 file app_failure.dart hoàn chỉnh.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file `app_failure.dart` với `sealed class AppFailure`, 4 subclass, factory constructor, extension method, `buildErrorWidget` function.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Khai báo là `sealed class` (KHÔNG phải `abstract class`)? | ☐ |
| 2 | Switch expression KHÔNG có `default` case (exhaustive tự động với sealed)? | ☐ |
| 3 | Pattern matching dùng destructuring: `ServerFailure(statusCode: final code)` thay vì cast? | ☐ |
| 4 | `AuthFailReason` là enum riêng biệt, không dùng String? | ☐ |
| 5 | Factory constructor handle đúng status code ranges (400-499, 500-599)? | ☐ |
| 6 | `dart analyze` không có warning? | ☐ |

**4. Customize:**
Tự thêm: method `bool get isRetryable` — trả về `true` nếu lỗi có thể retry (NetworkFailure timeout, ServerFailure 503). AI chưa làm phần này. Implement logic và viết thêm 3 test cases dùng `assert` để verify.

### AI-BT2: Gen Collection Utilities + Verify Immutability ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Collection operators (map, where, fold), spread operator, collection if/for.
- **Task thực tế:** Feature "Thống kê đơn hàng" — cần filter đơn hàng theo date range và status, group by tháng, tính tổng doanh thu mỗi tháng, sort theo tổng giảm dần. Dữ liệu thô là List từ API.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần viết Dart utility cho feature thống kê đơn hàng.
Tech stack: Dart 3.x, sound null safety.
Data class Order: id (int), customerId (int), amount (double), status (enum OrderStatus: pending, confirmed, shipped, delivered, cancelled), createdAt (DateTime).
Constraints:
- Dùng functional operators (where, map, fold) — KHÔNG dùng for loop thủ công.
- filterOrders(List<Order>, {DateTimeRange? dateRange, Set<OrderStatus>? statuses}) → trả về List mới, KHÔNG mutate input.
- groupByMonth(List<Order>) → Map<String, List<Order>> (key: "2024-01", "2024-02"...).
- monthlyRevenue(Map<String, List<Order>>) → List<MonthlyStats> sorted by revenue descending.
- MonthlyStats: month (String), totalRevenue (double), orderCount (int).
- Tất cả functions PHẢI immutable — trả về collection mới, không sửa input.
Output: 1 file order_stats.dart hoàn chỉnh với classes và functions.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file với `Order` class, `OrderStatus` enum, `MonthlyStats` class, 3 utility functions.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Tất cả functions trả về collection MỚI (không `list.sort()`, phải `[...list]..sort()`)? | ☐ |
| 2 | `filterOrders` xử lý null params đúng (null = bỏ qua filter đó)? | ☐ |
| 3 | `groupByMonth` dùng `fold` hoặc `forEach` trên Map đúng cách? | ☐ |
| 4 | Sort không mutate list gốc (dùng `toList()..sort()` hoặc spread)? | ☐ |
| 5 | Không có `dynamic` type nào? | ☐ |
| 6 | `dart analyze` không có warning? | ☐ |

**4. Customize:**
Tự thêm: function `topCustomers(List<Order> orders, {int limit = 5})` — trả về top N customers theo tổng amount, dùng fold + entries.toList()..sort. AI chưa làm phần này.
