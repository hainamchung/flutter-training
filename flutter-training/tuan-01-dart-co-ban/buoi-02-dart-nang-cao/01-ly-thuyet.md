# Buổi 02 — Lý thuyết: Dart nâng cao — OOP, Async, Collections

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 2/16** · **Thời lượng tự học:** ~1 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 01 (lý thuyết + ít nhất BT1-BT2)

---

## Mục lục

1. [OOP trong Dart](#1-oop-trong-dart)
2. [Enum nâng cao](#2-enum-nâng-cao)
3. [Collections deep dive](#3-collections-deep-dive)
4. [Async Programming](#4-async-programming)
5. [Extension Methods](#5-extension-methods)
6. [Records & Pattern Matching (Dart 3)](#6-records--pattern-matching-dart-3)
7. [Error Handling](#7-error-handling)
8. [Best Practices & Lỗi thường gặp](#8-best-practices--lỗi-thường-gặp)
9. [💡 FE → Flutter: Góc nhìn chuyển đổi](#9--fe--flutter-góc-nhìn-chuyển-đổi-)
10. [Tổng kết](#10-tổng-kết)

---

## 1. OOP trong Dart 🔴

> **Buổi 01 bạn đã biết:** variables, types, functions, null safety.
> **Bây giờ:** Dart là ngôn ngữ **object-oriented** — mọi thứ đều là object (kể cả `int`, `String`). Hãy học cách tổ chức code bằng class.

### 1.1 Class cơ bản & Constructors

#### Định nghĩa

**Class** = bản thiết kế (blueprint) để tạo object. Mỗi object là một **instance** của class.

#### Tại sao cần class?

Khi code phức tạp, bạn cần gom dữ liệu (properties) và hành vi (methods) lại một chỗ. Thay vì truyền 10 biến riêng lẻ, bạn truyền 1 object.

#### Cách hoạt động

Dart có **4 loại constructor**:

| Loại Constructor         | Cú pháp                          | Khi nào dùng                                    |
|--------------------------|-----------------------------------|-------------------------------------------------|
| **Default**              | `Person(this.name, this.age)`     | Trường hợp thông thường                          |
| **Named**                | `Person.guest()`                  | Tạo object với giá trị đặc biệt / mặc định     |
| **Factory**              | `factory Person.fromJson(...)`    | Khi cần logic trước khi tạo object (parse, cache)|
| **Const**                | `const Person('Dart', 10)`        | Object bất biến, tạo tại compile-time            |

```dart
class Person {
  final String name;
  final int age;

  // Default constructor — dùng `this.` để gán trực tiếp
  Person(this.name, this.age);

  // Named constructor — tạo "guest" mặc định
  Person.guest()
      : name = 'Guest',
        age = 0;

  // Factory constructor — chứa logic, trả về instance
  factory Person.fromJson(Map<String, dynamic> json) {
    return Person(json['name'] as String, json['age'] as int);
  }

  // Const constructor — object bất biến
  // Yêu cầu: tất cả fields phải là final
  const Person.constant(this.name, this.age);

  @override
  String toString() => 'Person(name: $name, age: $age)';
}
```

**Quy tắc:**
- `this.name` trong constructor = tự gán parameter vào field (syntactic sugar)
- `factory` constructor **không** truy cập `this` — nó trả về một instance
- `const` constructor yêu cầu **tất cả fields là `final`** và không có body

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD1](./02-vi-du.md#vd1-class-hierarchy)

### 1.2 Getters & Setters

#### Định nghĩa

**Getter/Setter** = cách kiểm soát việc đọc/ghi vào property. Dart cho phép tạo "computed property" bằng getter.

#### Tại sao cần?

- Getter: tính toán giá trị dựa trên fields khác (không lưu trữ riêng)
- Setter: validate dữ liệu trước khi gán

```dart
class Rectangle {
  double width;
  double height;

  Rectangle(this.width, this.height);

  // Getter — computed property, không lưu trữ
  double get area => width * height;
  double get perimeter => 2 * (width + height);

  // Setter — validate trước khi gán
  set widthValue(double value) {
    if (value <= 0) throw ArgumentError('Width must be positive');
    width = value;
  }
}
```

### 1.3 Operator Overloading

#### Định nghĩa

**Operator overloading** = định nghĩa lại hành vi của toán tử (+, -, ==, ...) cho class của bạn.

#### Khi nào dùng?

Khi class đại diện cho giá trị toán học / so sánh được (vector, money, point).

```dart
class Vector2D {
  final double x, y;
  const Vector2D(this.x, this.y);

  // Overload toán tử +
  Vector2D operator +(Vector2D other) {
    return Vector2D(x + other.x, y + other.y);
  }

  // Overload toán tử ==
  @override
  bool operator ==(Object other) {
    return other is Vector2D && x == other.x && y == other.y;
  }

  @override
  int get hashCode => Object.hash(x, y);

  @override
  String toString() => 'Vector2D($x, $y)';
}
```

### 1.4 Inheritance (extends)

#### Định nghĩa

**Inheritance** = class con kế thừa toàn bộ properties và methods từ class cha. Dùng keyword `extends`.

#### Tại sao cần?

Tái sử dụng code. Nếu `Student` và `Teacher` đều có `name`, `age` — đừng viết lại 2 lần. Đặt vào class `Person` rồi kế thừa.

#### Cách hoạt động

```dart
class Person {
  final String name;
  final int age;
  Person(this.name, this.age);

  String introduce() => 'Tôi là $name, $age tuổi';
}

class Student extends Person {
  final String studentId;

  // Gọi constructor cha bằng super
  Student(super.name, super.age, this.studentId);

  // Override method — thay đổi hành vi
  @override
  String introduce() => '${super.introduce()}, MSSV: $studentId';
}
```

**Quy tắc:**
- Dart chỉ hỗ trợ **single inheritance** (1 class chỉ extends 1 class cha)
- Dùng `super` để gọi constructor/method của class cha
- Dùng `@override` khi ghi đè method (bắt buộc về convention)

### 1.5 Abstract Class

#### Định nghĩa

**Abstract class** = class **không thể tạo instance trực tiếp**. Nó định nghĩa interface (hợp đồng) mà các class con **bắt buộc** phải implement.

#### Tại sao cần?

Khi bạn muốn **ép buộc** tất cả class con phải có method nào đó, nhưng mỗi class con implement khác nhau.

```dart
abstract class Shape {
  // Method abstract — không có body, class con PHẢI implement
  double area();
  double perimeter();

  // Method có body — class con kế thừa, không cần ghi đè
  String describe() => 'Shape: area=${area()}, perimeter=${perimeter()}';
}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);

  @override
  double area() => 3.14159 * radius * radius;

  @override
  double perimeter() => 2 * 3.14159 * radius;
}
```

### 1.6 Interface (Implicit)

#### Định nghĩa

Trong Dart, **mọi class đều tự động là interface**. Không có keyword `interface` riêng (trước Dart 3). Bạn dùng `implements` để implement interface.

#### extends vs implements vs with — khác nhau thế nào?

| Keyword      | Ý nghĩa                                                   | Kế thừa code? | Số lượng    |
|--------------|------------------------------------------------------------|----------------|-------------|
| `extends`    | Kế thừa class cha (cả code lẫn interface)                  | ✅ Có          | Chỉ 1       |
| `implements` | Implement interface (chỉ "hợp đồng", phải viết lại code)  | ❌ Không       | Nhiều       |
| `with`       | Trộn (mix in) code từ mixin                                | ✅ Có          | Nhiều       |

```dart
class Printable {
  void printInfo() => print('Printable');
}

class Logger {
  void log(String msg) => print('[LOG] $msg');
}

// implements — phải viết lại TẤT CẢ method
class MyService implements Printable, Logger {
  @override
  void printInfo() => print('MyService info');

  @override
  void log(String msg) => print('[MyService] $msg');
}
```

### 1.7 Mixins

#### Định nghĩa

**Mixin** = cách chia sẻ code giữa nhiều class **không cùng hierarchy**. Dùng keyword `mixin` + `with`.

#### Tại sao cần mixin?

Dart chỉ cho single inheritance. Nếu `Bird` cần cả `fly()` và `swim()` nhưng 2 ability này thuộc 2 nhóm khác nhau → dùng mixin.

#### Cách hoạt động

```dart
mixin Flyable {
  void fly() => print('Đang bay...');
}

mixin Swimmable {
  void swim() => print('Đang bơi...');
}

class Duck extends Animal with Flyable, Swimmable {
  Duck(super.name);
}

// Duck có thể fly() VÀ swim() mà không cần viết lại code
```

#### Khi nào dùng mixin vs inheritance?

| Tình huống                                        | Dùng         |
|---------------------------------------------------|--------------|
| Quan hệ "is-a" (Student **is a** Person)          | `extends`    |
| Chia sẻ ability (Bird **can** fly, **can** swim)  | `mixin` + `with` |
| Chỉ cần "hợp đồng" method                        | `implements` |

#### Mixin với constraint (on)

```dart
mixin Printable on Object {
  void printDetails();
}

// Chỉ class extends Object (tức mọi class) mới dùng được Printable
// Hữu ích khi mixin cần gọi method từ class cụ thể
mixin DatabaseMixin on DatabaseConnection {
  void query(String sql) {
    // Có thể gọi method của DatabaseConnection tại đây
    final conn = getConnection(); // method từ DatabaseConnection
    print('Querying: $sql');
  }
}
```

### 1.8 Sealed Class (Dart 3)

#### Định nghĩa

**Sealed class** = class **chỉ có thể được kế thừa trong cùng file**. Compiler biết chính xác có bao nhiêu subclass → hỗ trợ **exhaustive switch** (kiểm tra đủ tất cả trường hợp).

#### Tại sao cần? — Giải thích từng bước

**Bước 1 — Vấn đề:** Bạn muốn biểu diễn kết quả API — hoặc thành công, hoặc thất bại:

```dart
// Cách cũ (trước Dart 3): dùng abstract class
abstract class Result {}
class Success extends Result { final data; Success(this.data); }
class Failure extends Result { final String error; Failure(this.error); }

// Vấn đề: Compiler KHÔNG biết chỉ có 2 subclass
// → dùng switch sẽ cần default case
// → nếu thêm subclass mới, compiler không cảnh báo
```

**Bước 2 — Giải pháp: sealed class**

```dart
// Dart 3: sealed class
sealed class Result<T> {}
class Success<T> extends Result<T> { final T data; Success(this.data); }
class Failure<T> extends Result<T> { final String error; Failure(this.error); }

// Bây giờ compiler BIẾT chỉ có Success và Failure
// → switch PHẢI cover cả 2 → không cần default
// → thêm subclass mới → compiler báo lỗi ở MỌI switch chưa handle
```

**Bước 3 — Exhaustive switch:**

```dart
String handleResult(Result<String> result) {
  // Compiler kiểm tra: đã handle tất cả subclass chưa?
  return switch (result) {
    Success(data: final d) => 'Thành công: $d',
    Failure(error: final e) => 'Lỗi: $e',
    // Không cần default! Compiler đã biết đủ 2 case.
  };
}
```

**Lợi ích:**
- **Type-safe**: compiler đảm bảo bạn handle hết mọi trường hợp
- **Refactoring-safe**: thêm subclass mới → compiler báo lỗi ở mọi nơi chưa update
- **Tự tài liệu hóa**: nhìn sealed class là biết ngay có bao nhiêu variant

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD2](./02-vi-du.md#vd2-sealed-class--pattern-matching)

---

> 💼 **Gặp trong dự án:** API error handling (network/parse/auth errors phải handle riêng), Form validation (nhiều loại lỗi khác nhau), State machine cho order/payment flow
> 🤖 **Keywords bắt buộc trong prompt:** `sealed class`, `exhaustive switch`, `pattern matching Dart 3`, `factory constructor`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Error handling:** App cần phân biệt NetworkError (timeout, no internet), ParseError (JSON sai format), AuthError (token hết hạn) — mỗi loại xử lý khác nhau trên UI
- **Payment flow:** Trạng thái đơn hàng: Pending → Processing → Success/Failed — cần exhaustive switch để không quên handle trạng thái nào
- **Form validation:** Validate email/phone/password trả về các loại lỗi khác nhau để hiển thị message phù hợp

**Tại sao cần các keyword trên:**
- **`sealed class`** (không phải `abstract class`) — để compiler enforce exhaustive check, AI hay dùng abstract class kiểu cũ
- **`exhaustive switch`** — AI phải dùng switch expression Dart 3, không dùng if-else chain
- **`pattern matching Dart 3`** — destructuring trong switch case, AI có thể gen syntax Dart 2 cũ
- **`factory constructor`** — để tạo Error từ HTTP status code hoặc exception type

**Prompt mẫu — Domain Error Handling:**
```text
Tôi cần implement error handling cho Flutter app dùng sealed class Dart 3.
Context: App gọi REST API, cần handle 4 loại lỗi riêng biệt.
Tech stack: Dart 3.x, sound null safety.
Constraints:
- Dùng sealed class AppError, 4 subclass: NetworkError, ServerError, ParseError, AuthError.
- Mỗi subclass có field riêng phù hợp (message, statusCode, originalException).
- Factory constructor AppError.fromException(Object e) → tự map Exception type sang đúng subclass.
- Extension method trên AppError: String get userMessage → message thân thiện cho user (tiếng Việt).
- 1 hàm handleError(AppError error) dùng exhaustive switch expression.
Output: 1 file app_error.dart hoàn chỉnh.
```

**Expected Output:** AI sẽ gen 1 file với `sealed class AppError` + 4 subclass + factory constructor + extension method + `handleError` function dùng switch expression.

⚠️ **Giới hạn AI hay mắc:** AI đôi khi dùng `abstract class` thay vì `sealed class` — check dòng khai báo. AI cũng hay thiếu `factory AppError.fromException` hoặc viết switch với `default` case (không cần với sealed class).

</details>

> 🔗 **FE Bridge:** Dart OOP ≈ TypeScript class nhưng **khác ở**: Dart = **OOP-first** (single inheritance + mixin), TypeScript/JS = multi-paradigm (prototype + class sugar). Dart `mixin` = composition pattern không có native equivalent trong TS. `abstract class` ≈ TS `abstract class` — mapping 1:1.

---

## 2. Enum nâng cao 🟡

### Định nghĩa

Từ Dart 2.17, **enum có thể có fields, methods, và implement interface** — mạnh hơn nhiều so với enum trong hầu hết ngôn ngữ.

### Tại sao dùng enum nâng cao?

Thay vì viết `if (status == 'active')` (dễ sai chính tả), dùng enum để compiler kiểm tra.

### Cách hoạt động

```dart
enum OrderStatus implements Comparable<OrderStatus> {
  pending(label: 'Chờ xử lý', priority: 3),
  processing(label: 'Đang xử lý', priority: 2),
  shipped(label: 'Đã gửi', priority: 1),
  delivered(label: 'Đã nhận', priority: 0);

  // Field
  final String label;
  final int priority;

  // Constructor
  const OrderStatus({required this.label, required this.priority});

  // Method
  bool get isCompleted => this == delivered;

  // Implement interface
  @override
  int compareTo(OrderStatus other) => priority.compareTo(other.priority);
}
```

**Sử dụng:**

```dart
void main() {
  final status = OrderStatus.processing;
  print(status.label);       // Đang xử lý
  print(status.isCompleted); // false

  // Enum trong switch — exhaustive!
  final message = switch (status) {
    OrderStatus.pending    => 'Đơn hàng đang chờ',
    OrderStatus.processing => 'Đang xử lý đơn hàng',
    OrderStatus.shipped    => 'Đã gửi hàng',
    OrderStatus.delivered  => 'Đã giao thành công',
  };
  print(message);
}
```

---

## 3. Collections deep dive 🟡

### Định nghĩa

Dart có 3 collection chính: `List` (mảng), `Map` (key-value), `Set` (tập hợp không trùng).

### Tại sao cần hiểu sâu?

Trong Flutter, bạn sẽ **liên tục** dùng collections: hiển thị danh sách sản phẩm, filter đơn hàng, đếm tổng giá, ... Nắm vững các operators sẽ giúp code ngắn gọn và rõ ràng hơn.

### 3.1 Các method quan trọng

| Method     | Mô tả                                    | Ví dụ                                          |
|------------|-------------------------------------------|-------------------------------------------------|
| `map`      | Biến đổi mỗi phần tử                     | `[1,2,3].map((e) => e * 2)` → `(2, 4, 6)`     |
| `where`    | Lọc phần tử thỏa điều kiện               | `[1,2,3,4].where((e) => e.isEven)` → `(2, 4)` |
| `fold`     | Gộp tất cả thành 1 giá trị (có initial)  | `[1,2,3].fold(0, (sum, e) => sum + e)` → `6`   |
| `reduce`   | Giống fold nhưng không có initial value   | `[1,2,3].reduce((a, b) => a + b)` → `6`        |
| `any`      | Có ít nhất 1 phần tử thỏa điều kiện?     | `[1,2,3].any((e) => e > 2)` → `true`           |
| `every`    | TẤT CẢ phần tử thỏa điều kiện?           | `[1,2,3].every((e) => e > 0)` → `true`         |
| `firstWhere` | Tìm phần tử đầu tiên thỏa điều kiện   | `[1,2,3].firstWhere((e) => e > 1)` → `2`       |
| `expand`   | Flatten — mỗi phần tử trả về iterable    | `[[1,2],[3]].expand((e) => e)` → `(1,2,3)`     |
| `toList()` | Chuyển Iterable thành List                | `.map(...).toList()`                            |
| `toSet()`  | Chuyển thành Set (loại trùng)             | `[1,1,2].toSet()` → `{1, 2}`                   |

### 3.2 Spread Operator (`...`)

```dart
final base = [1, 2, 3];
final extended = [0, ...base, 4]; // [0, 1, 2, 3, 4]

// Null-aware spread
List<int>? maybeNull;
final safe = [0, ...?maybeNull, 4]; // [0, 4]
```

### 3.3 Collection if / Collection for

Dart cho phép dùng `if` và `for` **bên trong collection literal** — cực kỳ hữu ích trong Flutter khi build widget list.

```dart
final isAdmin = true;
final items = [
  'Home',
  'Profile',
  if (isAdmin) 'Admin Panel', // Chỉ thêm nếu isAdmin = true
];

final numbers = [1, 2, 3];
final doubled = [
  for (final n in numbers) n * 2, // [2, 4, 6]
];
```

### 3.4 Map & Set operations

```dart
// Map
final scores = {'Alice': 90, 'Bob': 85, 'Charlie': 92};
final topStudents = scores.entries
    .where((e) => e.value >= 90)
    .map((e) => e.key)
    .toList(); // ['Alice', 'Charlie']

// Set — tự loại trùng
final Set<String> tags = {'dart', 'flutter', 'dart'}; // {'dart', 'flutter'}
final a = {1, 2, 3};
final b = {2, 3, 4};
print(a.intersection(b)); // {2, 3}
print(a.union(b));         // {1, 2, 3, 4}
print(a.difference(b));    // {1}
```

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD6](./02-vi-du.md#vd6-collection-operators)

---

> 💼 **Gặp trong dự án:** Filter/sort danh sách sản phẩm theo nhiều tiêu chí, transform API response list thành UI model, aggregate data (tổng tiền giỏ hàng, đếm item theo category)
> 🤖 **Keywords bắt buộc trong prompt:** `collection operators (map, where, fold)`, `spread operator`, `collection if/for`, `null-aware spread ...?`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **E-commerce:** Filter sản phẩm theo category + price range + rating, sort theo giá/mới nhất, tính tổng tiền có discount
- **Dashboard:** Aggregate dữ liệu từ List transactions → group by month → tính tổng/trung bình cho chart
- **Form builder:** Xây dựng list widget động từ config JSON, dùng collection if để ẩn/hiện field, spread để merge default + custom fields

**Tại sao cần các keyword trên:**
- **`map, where, fold`** — AI hay dùng `for` loop thủ công thay vì functional operators, code dài hơn cần thiết
- **`spread operator`** — khi merge lists/maps, AI có thể dùng `addAll` mutable thay vì `[...list1, ...list2]` immutable
- **`collection if/for`** — đặc biệt hữu ích trong Flutter widget list, AI cần biết dùng inline thay vì tách ra biến trước
- **`null-aware spread ...?`** — khi list có thể null, AI hay quên xử lý

**Prompt mẫu — Product filter & sort:**
```text
Tôi cần viết Dart utility functions cho e-commerce product list.
Tech stack: Dart 3.x, sound null safety, dùng data class Product(id, name, price, category, rating, isAvailable).
Constraints:
- Dùng functional operators (map, where, fold, expand) — không dùng for loop thủ công.
- Filter: theo category (String?), price range (min, max: double?), rating tối thiểu (double?). Null parameter = bỏ qua filter đó.
- Sort: enum SortBy { priceAsc, priceDesc, ratingDesc, nameAsc }.
- Aggregate: tổng tiền giỏ hàng (List<CartItem>), đếm sản phẩm theo category (Map<String, int>).
- Tất cả hàm phải trả về List/Map mới (immutable), không mutate input.
Output: 1 file product_utils.dart với 4 functions: filterProducts, sortProducts, calcCartTotal, countByCategory.
```

**Expected Output:** AI sẽ gen file với `Product` class + 4 utility functions dùng `where`, `fold`, `sort` (trên copy), `groupBy` logic.

⚠️ **Giới hạn AI hay mắc:** AI hay mutate list gốc khi sort (`list.sort()` thay vì `[...list]..sort()`). Check kỹ: input list có bị thay đổi không? Ngoài ra AI đôi khi dùng `.toList()` thừa sau `where` rồi lại `.map`.

</details>

---

## 4. Async Programming 🔴

### Tại sao cần async?

Dart là **single-threaded** — chỉ có 1 thread chạy code. Nếu gọi API mất 3 giây mà code chờ (blocking), UI sẽ đứng hình. Async cho phép "chờ" mà **không block** thread chính.

### 4.1 Future&lt;T&gt;

#### Định nghĩa

**Future&lt;T&gt;** = một giá trị **sẽ có trong tương lai**. Giống `Promise<T>` trong JavaScript.

#### Trạng thái của Future

```
Future được tạo
       │
       ▼
  ┌──────────┐
  │ Pending  │  ← Đang chờ kết quả
  └──────────┘
       │
       ├──── thành công ──▶ Completed (with value)
       │
       └──── thất bại ──▶ Completed (with error)
```

#### async/await vs .then()

```dart
// Cách 1: async/await — đọc dễ hơn, KHUYẾN KHÍCH dùng
Future<String> fetchUser() async {
  final response = await fetchFromApi('/user'); // "chờ" mà không block
  return response.data;
}

// Cách 2: .then() chain — dùng khi cần chain ngắn
fetchFromApi('/user')
    .then((response) => response.data)
    .catchError((error) => 'Lỗi: $error');
```

#### Future.delayed — giả lập API

```dart
Future<String> simulateApiCall() {
  // Giả lập API mất 2 giây
  return Future.delayed(
    const Duration(seconds: 2),
    () => 'Data from API',
  );
}
```

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD3](./02-vi-du.md#vd3-future--async-api-call)

### 4.2 Stream&lt;T&gt;

#### Định nghĩa

**Stream&lt;T&gt;** = chuỗi giá trị async **theo thời gian**. Nếu Future là "1 lần giao hàng", thì Stream là "đăng ký nhận hàng định kỳ".

#### Khi nào dùng Future vs Stream?

| Tình huống                    | Dùng          | Ví dụ thực tế                     |
|-------------------------------|---------------|------------------------------------|
| 1 kết quả, 1 lần             | `Future<T>`   | Gọi API, đọc file                 |
| Nhiều giá trị theo thời gian  | `Stream<T>`   | WebSocket, sensor, countdown, DB listen |

#### Tạo Stream

```dart
// Cách 1: async* + yield (khuyến khích)
Stream<int> countdown(int from) async* {
  for (int i = from; i >= 0; i--) {
    yield i;  // "phát" giá trị ra stream
    await Future.delayed(const Duration(seconds: 1));
  }
}

// Cách 2: StreamController
final controller = StreamController<String>();
controller.sink.add('Event 1');  // Gửi dữ liệu vào stream
controller.sink.add('Event 2');
controller.close();              // Đóng stream

// Lắng nghe
controller.stream.listen(
  (data) => print('Received: $data'),
  onDone: () => print('Stream closed'),
  onError: (error) => print('Error: $error'),
);
```

#### Single-subscription vs Broadcast Stream

```dart
// Single — chỉ 1 listener (mặc định)
final stream = countdown(5);
stream.listen(print); // OK
// stream.listen(print); // ❌ LỖI! Đã có listener rồi

// Broadcast — nhiều listener
final broadcastStream = countdown(5).asBroadcastStream();
broadcastStream.listen((n) => print('Listener 1: $n'));
broadcastStream.listen((n) => print('Listener 2: $n')); // OK
```

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD4](./02-vi-du.md#vd4-stream--countdown-timer)

### 4.3 Event Loop

#### Cách Dart xử lý async

Dart có 1 thread duy nhất với 1 **event loop** quản lý 2 hàng đợi:

```
                    ┌─────────────────────────┐
                    │      Dart Event Loop     │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
          ┌────────│   Có Microtask chờ?      │
          │ YES    └────────────┬────────────┘
          │                     │ NO
          ▼                     ▼
  ┌───────────────┐   ┌───────────────────┐
  │ MICROTASK     │   │   EVENT QUEUE     │
  │ QUEUE         │   │                   │
  │ (ưu tiên cao)│   │ (ưu tiên thường)  │
  │               │   │                   │
  │ • scheduleMicro│  │ • Future.then()   │
  │   task()      │   │ • Timer callback  │
  │ • Future body │   │ • I/O events      │
  │               │   │ • UI events       │
  └───────────────┘   └───────────────────┘

  Luồng xử lý:
  1. Chạy code synchronous trong main()
  2. Xử lý TẤT CẢ microtask queue (hết mới dừng)
  3. Lấy 1 event từ event queue → xử lý
  4. Quay lại bước 2
```

**Quy tắc quan trọng:**
- Microtask queue **luôn chạy trước** event queue
- `Future.then()` callback → event queue
- `scheduleMicrotask()` → microtask queue
- **Đừng** đặt code nặng trong microtask — sẽ block event loop

> 🔗 **FE Bridge:** `Future` ≈ `Promise`, `async/await` = gần **1:1**. Nhưng **Stream** ≈ RxJS `Observable` — và Stream **phổ biến hơn** trong Dart so với Observable trong FE. `Stream.listen()` ≈ `observable.subscribe()`, `StreamController` ≈ `Subject`. FE dev cần invest thời gian hiểu Stream vì Flutter dùng rất nhiều.

---

## 5. Extension Methods 🟢

### Định nghĩa

**Extension methods** = thêm method vào class **có sẵn** mà không cần kế thừa hay sửa source code. Giống "mở rộng" thư viện mà bạn không sở hữu.

### Tại sao cần?

Bạn muốn thêm method `capitalize()` cho `String` — nhưng `String` là class của Dart SDK, bạn không thể sửa. Extension methods giải quyết vấn đề này.

### Cách hoạt động

```dart
extension StringExtension on String {
  // Viết hoa chữ cái đầu
  String get capitalize {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }

  // Kiểm tra email đơn giản
  bool get isEmail {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);
  }

  // Rút gọn text
  String truncate(int maxLength, {String suffix = '...'}) {
    if (length <= maxLength) return this;
    return '${substring(0, maxLength)}$suffix';
  }
}

// Sử dụng — giống method có sẵn của String!
void main() {
  print('hello'.capitalize);         // Hello
  print('test@email.com'.isEmail);   // true
  print('Long text here'.truncate(8)); // Long tex...
}
```

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD5](./02-vi-du.md#vd5-extension-methods)

> 🔗 **FE Bridge:** Extension methods ≈ **prototype extension** trong JS hoặc TS augmentation — nhưng **khác ở**: Dart extension = compile-time, type-safe, không modify runtime prototype. Giống concept "add methods to existing types" nhưng an toàn hơn monkey-patching JS prototype.

---

## 6. Records & Pattern Matching (Dart 3) 🟡

### 6.1 Records

#### Định nghĩa

**Record** = kiểu dữ liệu nhẹ, giống tuple, cho phép nhóm nhiều giá trị mà **không cần tạo class**.

#### Tại sao cần?

Khi function trả về 2-3 giá trị, trước Dart 3 phải tạo class riêng hoặc dùng `List`. Record giải quyết gọn hơn.

```dart
// Record — literal type
(String, int) getNameAndAge() {
  return ('Nguyễn Văn A', 25);
}

// Named fields
({String name, int age}) getUserInfo() {
  return (name: 'Trần Thị B', age: 30);
}

void main() {
  // Positional access
  final record = getNameAndAge();
  print(record.$1); // Nguyễn Văn A
  print(record.$2); // 25

  // Named access
  final user = getUserInfo();
  print(user.name); // Trần Thị B
  print(user.age);  // 30

  // Destructuring
  final (name, age) = getNameAndAge();
  print('$name, $age tuổi'); // Nguyễn Văn A, 25 tuổi
}
```

### 6.2 Pattern Matching

#### Định nghĩa

**Pattern matching** = so khớp và phân rã (destructure) dữ liệu. Mạnh hơn `switch` cũ rất nhiều.

#### Các loại pattern

```dart
void main() {
  // 1. Variable pattern — destructure
  final (x, y) = (10, 20);
  print('$x, $y'); // 10, 20

  // 2. List pattern
  final [a, b, ...rest] = [1, 2, 3, 4, 5];
  print('$a, $b, $rest'); // 1, 2, [3, 4, 5]

  // 3. Map pattern
  final {'name': name, 'age': age} = {'name': 'Dart', 'age': 3};
  print('$name, $age'); // Dart, 3

  // 4. Object pattern — destructure object
  final point = Point(10, 20);
  final Point(:x, :y) = point;  // lấy x, y từ point
  print('$x, $y'); // 10, 20
}
```

### 6.3 Exhaustive Switch với Sealed Class

Đây là **combo mạnh nhất của Dart 3**: `sealed class` + pattern matching = type-safe handling cho mọi trường hợp.

```dart
sealed class Shape {}
class Circle extends Shape { final double radius; Circle(this.radius); }
class Rectangle extends Shape { final double w, h; Rectangle(this.w, this.h); }
class Triangle extends Shape { final double base, height; Triangle(this.base, this.height); }

double calculateArea(Shape shape) {
  return switch (shape) {
    Circle(radius: final r)           => 3.14159 * r * r,
    Rectangle(w: final w, h: final h) => w * h,
    Triangle(base: final b, height: final h) => 0.5 * b * h,
    // Không cần default — compiler biết đã cover hết!
    // Nếu thêm class Pentagon extends Shape → compiler báo lỗi ở đây
  };
}
```

> 📖 **Xem code đầy đủ:** [02-vi-du.md → VD2](./02-vi-du.md#vd2-sealed-class--pattern-matching)

> 🔗 **FE Bridge:** Dart `sealed class` + pattern matching ≈ TypeScript **discriminated union** + `switch` exhaustive check. Mapping concept gần 1:1 — nhưng Dart pattern matching mạnh hơn: `switch` expression, guard clause, destructuring pattern trong `case`. TS 5.x đang catch up nhưng chưa bằng.

---

## 7. Error Handling 🔴

### 7.1 Exception vs Error

| Loại          | Ý nghĩa                                    | Xử lý?             | Ví dụ                                     |
|---------------|---------------------------------------------|---------------------|--------------------------------------------|
| `Exception`   | Lỗi **dự kiến được**, code gây ra          | ✅ Nên catch         | FormatException, HttpException, IOException |
| `Error`       | Lỗi **lập trình viên**, bug trong code      | ❌ Không nên catch   | StackOverflowError, TypeError              |

### 7.2 try / catch / finally

```dart
Future<void> loadData() async {
  try {
    final data = await fetchFromApi();
    print('Data: $data');
  } on FormatException catch (e) {
    // Catch lỗi cụ thể
    print('Dữ liệu sai format: $e');
  } on Exception catch (e, stackTrace) {
    // Catch mọi Exception (nhưng không catch Error)
    print('Exception: $e');
    print('StackTrace: $stackTrace');
  } finally {
    // LUÔN chạy, dù có lỗi hay không
    print('Cleanup xong');
  }
}
```

### 7.3 Custom Exception

```dart
class ApiException implements Exception {
  final String message;
  final int statusCode;

  const ApiException(this.message, this.statusCode);

  @override
  String toString() => 'ApiException($statusCode): $message';
}

// Throw
Future<String> fetchUser(int id) async {
  if (id <= 0) {
    throw ApiException('Invalid user ID', 400);
  }
  // ... fetch logic
  return 'User $id';
}

// Catch
void main() async {
  try {
    await fetchUser(-1);
  } on ApiException catch (e) {
    print('API Error: ${e.message} (${e.statusCode})');
  }
}
```

### 7.4 Khi nào throw?

| Tình huống                                   | Nên throw?          |
|----------------------------------------------|---------------------|
| Input từ user không hợp lệ                   | ✅ Throw exception   |
| API trả về lỗi 404, 500                      | ✅ Throw exception   |
| Logic không thể xảy ra (assert)              | Dùng `assert()`     |
| Muốn biểu diễn success/failure               | Dùng `Result<T>` pattern |

> 🔗 **FE Bridge:** `try/catch/finally` = **giống hệt** JS/TS. Nhưng **khác ở**: Dart có thể `throw` bất kỳ object nào (không chỉ Error), và convention `on SpecificException catch(e)` cho phép catch theo type — giống TS type guard nhưng built-in syntax.

---

## 8. Best Practices & Lỗi thường gặp 🟡

### ✅ Nên làm

| Practice                                                        | Lý do                                              |
|-----------------------------------------------------------------|----------------------------------------------------|
| Dùng `final` cho fields không đổi sau khi tạo                  | Immutability = ít bug hơn                          |
| Prefer `async/await` over `.then()` chain                       | Dễ đọc, dễ debug, rõ flow hơn                     |
| Dùng `sealed class` cho union types                             | Exhaustive check tại compile-time                  |
| `implements` khi chỉ cần interface, `extends` khi cần code      | Tránh kế thừa thừa code không dùng                |
| Luôn `await` Future trong `try/catch`                            | Nếu không `await`, error không bắt được            |
| Đóng `StreamController` khi không dùng nữa                      | Tránh memory leak                                  |

### ❌ Lỗi thường gặp

| Lỗi                                                            | Giải thích                                         |
|-----------------------------------------------------------------|----------------------------------------------------|
| Quên `await` → Future không chạy / error bị nuốt               | `await fetchData()` chứ không phải `fetchData()`   |
| Dùng `var` thay vì type cụ thể cho public API                  | API nên rõ ràng: `List<String>` chứ không phải `var` |
| Listen stream 2 lần (single-subscription)                       | Dùng `.asBroadcastStream()` hoặc redesign          |
| Factory constructor truy cập `this`                             | Factory không có `this` — nó trả về instance        |
| `catch (e)` không có type → catch cả Error                     | Dùng `on Exception catch (e)` để chỉ catch Exception |
| `const` constructor nhưng field không `final`                   | Const yêu cầu tất cả fields phải là `final`        |

---

## 9. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

> **Phần này dành cho bạn đã biết React/Vue/TypeScript.** Nếu chưa quen frontend, có thể bỏ qua.

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue (TypeScript) Mindset | Dart Mindset | Tại sao khác |
|---|-------------------------------|--------------|--------------|
| 1 | Multi-paradigm: functional + OOP | **OOP-first**: class, mixin, inheritance là core | Dart thiết kế cho OOP, TS thiết kế cho JS compatibility |
| 2 | Promise-based async, RxJS optional | Future + **Stream everywhere** — Stream là first-class | Flutter/Dart ecosystem dùng Stream rất heavy |
| 3 | Interface = structural typing (duck typing) | Interface = `abstract class` hoặc `class` — **nominal typing** | Dart check type by name, TS check by structure |
| 4 | Prototype extension / augmentation | Extension methods — compile-time, type-safe, scoped | Dart extension không modify runtime, TS augmentation = declaration merging |
| 5 | `try/catch` generic, type guard manual | `on SpecificType catch(e)` — type-safe catch built-in | Dart catch by type natively, TS cần `instanceof` check |

### Bảng so sánh tổng quan

| Dart                                | JavaScript / TypeScript              | Ghi chú                                          |
|-------------------------------------|--------------------------------------|---------------------------------------------------|
| `class Person { ... }`             | `class Person { ... }`              | Tương tự, nhưng Dart class mạnh hơn (mixin, sealed) |
| `extends`                           | `extends`                           | Giống nhau                                        |
| `implements` (every class = interface)| `implements` (TypeScript interface) | Dart: mọi class tự động là interface              |
| `mixin` + `with`                   | React HOC / Composition             | Mixin = chia sẻ behavior, HOC = chia sẻ logic     |
| `Future<T>`                        | `Promise<T>`                        | Gần như giống nhau                                |
| `async/await`                      | `async/await`                       | Cú pháp giống nhau, cùng concept                  |
| `Stream<T>`                        | `Observable` (RxJS)                 | Stream ≈ Observable. `.listen()` ≈ `.subscribe()`  |
| `sealed class`                     | TypeScript discriminated union       | `sealed` = union types + exhaustive switch         |
| `factory` constructor              | Static factory method                | JS dùng static method, Dart có syntax riêng        |
| `extension` methods                | Prototype extension (không khuyến khích) | Dart extension an toàn hơn — scoped, explicit    |
| Records `(String, int)`            | Tuple (TypeScript `[string, number]`)| Dart 3 records = TypeScript tuples                 |
| Pattern matching (`switch`)        | Pattern matching (TC39 proposal)     | Dart 3 đã có, JS đang proposal                    |

### 💡 Nếu bạn từ React/Vue

| Concept Flutter/Dart                          | Bạn đã biết                                             |
|-----------------------------------------------|----------------------------------------------------------|
| Mixin                                         | Giống React HOC nhưng ở cấp class. Hoặc giống Vue composables. |
| `sealed class Result<T>` (Success/Failure)    | Giống pattern `{ success: true, data } \| { success: false, error }` nhưng type-safe. |
| `Stream.listen()`                             | Giống `observable.subscribe()` trong RxJS hoặc `useEffect` listen events. |
| Collection if/for trong list literal           | Giống JSX `{isAdmin && <AdminPanel />}` hoặc `.map()`.   |
| `Future.delayed()`                            | Giống `new Promise(resolve => setTimeout(resolve, ms))`. |
| Extension methods                             | Giống lodash utility nhưng gắn vào type, có IDE autocomplete. |

---

## 10. Tổng kết

### ✅ Checklist — Bạn đã nắm được chưa?

| #  | Concept                                                   | Hiểu? | Làm được? |
|----|-----------------------------------------------------------|-------|-----------|
| 1  | Tạo class với default, named, factory, const constructor  | ☐     | ☐         |
| 2  | Phân biệt `extends` vs `implements` vs `with` (mixin)    | ☐     | ☐         |
| 3  | Giải thích sealed class và viết exhaustive switch          | ☐     | ☐         |
| 4  | Dùng `map`, `where`, `fold`, spread, collection if/for   | ☐     | ☐         |
| 5  | Viết async/await với Future, xử lý error đúng cách       | ☐     | ☐         |
| 6  | Tạo Stream với `async*`/`yield` và `StreamController`     | ☐     | ☐         |
| 7  | Giải thích Event Loop (microtask vs event queue)          | ☐     | ☐         |
| 8  | Viết extension method cho class có sẵn                    | ☐     | ☐         |
| 9  | Dùng Records và pattern matching (Dart 3)                 | ☐     | ☐         |
| 10 | Viết custom exception và dùng try/catch đúng              | ☐     | ☐         |

### 🔑 Điểm quan trọng nhất

1. **OOP trong Dart** mạnh hơn JS: có mixin, sealed class, factory constructor
2. **`sealed class`** = union types + exhaustive check → rất hữu ích cho error handling, state management
3. **Future = 1 giá trị async, Stream = nhiều giá trị async** — đừng nhầm lẫn
4. **Extension methods** = cách mở rộng class an toàn, không cần sửa source
5. **Pattern matching (Dart 3)** = switch mạnh mẽ hơn, destructure dữ liệu dễ dàng

### ➡️ Tiếp theo

Chuyển sang [02-vi-du.md](./02-vi-du.md) để xem **6 ví dụ hoàn chỉnh** minh họa các concept trên, sau đó làm bài tập tại [03-thuc-hanh.md](./03-thuc-hanh.md).

---

### ➡️ Buổi tiếp theo

> **Buổi 03: Widget Tree Cơ Bản** — StatelessWidget, StatefulWidget, BuildContext, Widget Lifecycle và Key system.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Cài đặt Flutter SDK và tạo project mới bằng `flutter create`
