# Buổi 01 — Ví dụ: Giới thiệu Dart & Flutter

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`ng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 1/16** · **5 ví dụ hoàn chỉnh** · **Cập nhật:** 2026-03-31

---

## Mục lục

1. [VD1: Hello World Dart CLI](#vd1-hello-world-dart-cli)
2. [VD2: Variables, Types & Null Safety](#vd2-variables-types--null-safety)
3. [VD3: Functions](#vd3-functions)
4. [VD4: Flutter Hello World](#vd4-flutter-hello-world)
5. [VD5: Hot Reload Demo](#vd5-hot-reload-demo)

---

## VD1: Hello World Dart CLI 🟢

### Mục đích
Viết chương trình Dart đầu tiên. Hiểu `void main()`, `print()`, và cách chạy bằng `dart run`.

> **Liên quan tới:** [5. Chương trình Dart đầu tiên 🟢](01-ly-thuyet.md#5-chương-trình-dart-đầu-tiên)

### Chuẩn bị

```bash
# Tạo project Dart mới
dart create hello_dart
cd hello_dart
```

### Code

Mở file `bin/hello_dart.dart`, xóa hết nội dung mặc định và thay bằng:

```dart
/// Chương trình Dart đầu tiên
/// File: bin/hello_dart.dart

void main() {
  // print() in text ra terminal — giống console.log() trong JavaScript
  print('=== Chương trình Dart đầu tiên ===');
  print('Xin chào, Dart!');
  print('Hôm nay là ngày học Flutter đầu tiên.');

  // Phép tính đơn giản
  int a = 10;
  int b = 20;
  int sum = a + b;
  print('$a + $b = $sum');

  // String interpolation
  String name = 'Flutter Developer';
  print('Tôi là một $name!');
}
```

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
=== Chương trình Dart đầu tiên ===
Xin chào, Dart!
Hôm nay là ngày học Flutter đầu tiên.
10 + 20 = 30
Tôi là một Flutter Developer!
```

### Giải thích

| Dòng code                    | Giải thích                                                     |
|------------------------------|----------------------------------------------------------------|
| `void main() { ... }`       | Entry point — Dart luôn bắt đầu chạy từ hàm `main()`          |
| `void`                       | Hàm `main` không trả về giá trị nào                           |
| `print('...')`               | In text ra terminal (stdout)                                   |
| `int a = 10;`               | Khai báo biến kiểu `int` (số nguyên) với giá trị 10           |
| `'$a + $b = $sum'`          | String interpolation — nhúng biến vào chuỗi bằng dấu `$`     |
| `'Tôi là một $name!'`       | `$name` được thay thế bằng giá trị của biến `name`            |

- 🔗 **FE tương đương:** Cú pháp Dart gần giống TypeScript — `main()` ≈ entry point, `print()` ≈ `console.log()`, type annotation giống TS.

---

## VD2: Variables, Types & Null Safety 🔴

### Mục đích
Thực hành `var`, `final`, `const`, `late`, kiểu dữ liệu, và null safety (`?`, `!`, `??`).

> **Liên quan tới:** [4. Dart Fundamentals 🔴](01-ly-thuyet.md#4-dart-fundamentals)

### Chuẩn bị

Dùng lại project `hello_dart` từ VD1. Tạo file mới `bin/variables_demo.dart`:

### Code

```dart
/// Demo variables, types, null safety
/// File: bin/variables_demo.dart
/// Chạy: dart run bin/variables_demo.dart

void main() {
  print('=== 1. VAR — Biến có thể thay đổi ===');
  var language = 'Dart'; // Dart suy ra type là String
  print('Ngôn ngữ: $language');
  language = 'Flutter'; // ✅ OK — thay đổi giá trị cùng type
  print('Thay đổi: $language');
  // language = 42; // ❌ LỖI — không thể đổi type String → int

  print('');
  print('=== 2. FINAL — Gán một lần ===');
  final city = 'Hà Nội';
  print('Thành phố: $city');
  // city = 'HCM'; // ❌ LỖI — final không cho gán lại
  final now = DateTime.now(); // ✅ Giá trị xác định lúc runtime
  print('Thời gian: $now');

  print('');
  print('=== 3. CONST — Hằng số compile-time ===');
  const pi = 3.14159;
  const appName = 'Hello Flutter';
  print('Pi = $pi');
  print('App: $appName');
  // const time = DateTime.now(); // ❌ LỖI — DateTime.now() không biết lúc compile

  print('');
  print('=== 4. LATE — Khai báo trước, gán sau ===');
  late String description;
  // print(description); // ❌ LỖI runtime nếu print trước khi gán
  description = 'Đây là biến late — khởi tạo trễ';
  print(description);

  print('');
  print('=== 5. KIỂU DỮ LIỆU CƠ BẢN ===');
  int age = 25;
  double height = 1.75;
  String name = 'Nguyễn Văn A';
  bool isStudent = true;
  print('$name, $age tuổi, cao ${height}m, sinh viên: $isStudent');

  print('');
  print('=== 6. COLLECTIONS ===');
  List<String> fruits = ['Táo', 'Cam', 'Xoài'];
  print('Fruits: $fruits');
  print('Fruit đầu tiên: ${fruits[0]}');

  Map<String, int> scores = {'Toán': 9, 'Lý': 8, 'Hóa': 7};
  print('Scores: $scores');
  print('Điểm Toán: ${scores['Toán']}');

  Set<int> uniqueNumbers = {1, 2, 3, 2, 1};
  print('Set (tự loại trùng): $uniqueNumbers');

  print('');
  print('=== 7. NULL SAFETY ===');

  // Mặc định: biến KHÔNG thể null
  String greeting = 'Xin chào';
  // greeting = null; // ❌ LỖI — String không chấp nhận null

  // Thêm ? để cho phép null
  String? nickname;
  print('Nickname (chưa gán): $nickname'); // null

  nickname = 'Dev Flutter';
  print('Nickname (đã gán): $nickname');

  // ?? — giá trị mặc định nếu null
  String? middleName;
  String displayMiddle = middleName ?? 'Không có';
  print('Middle name: $displayMiddle');

  // ! — khẳng định không null (cẩn thận!)
  String? fullName = 'Nguyễn Văn B';
  int nameLength = fullName!.length; // Chắc chắn fullName không null
  print('Độ dài tên: $nameLength');

  print('');
  print('=== HOÀN THÀNH VD2 ===');
}
```

### Chạy

```bash
dart run bin/variables_demo.dart
```

### Kết quả mong đợi

```
=== 1. VAR — Biến có thể thay đổi ===
Ngôn ngữ: Dart
Thay đổi: Flutter

=== 2. FINAL — Gán một lần ===
Thành phố: Hà Nội
Thời gian: 2026-03-31 10:30:00.000000

=== 3. CONST — Hằng số compile-time ===
Pi = 3.14159
App: Hello Flutter

=== 4. LATE — Khai báo trước, gán sau ===
Đây là biến late — khởi tạo trễ

=== 5. KIỂU DỮ LIỆU CƠ BẢN ===
Nguyễn Văn A, 25 tuổi, cao 1.75m, sinh viên: true

=== 6. COLLECTIONS ===
Fruits: [Táo, Cam, Xoài]
Fruit đầu tiên: Táo
Scores: {Toán: 9, Lý: 8, Hóa: 7}
Điểm Toán: 9
Set (tự loại trùng): {1, 2, 3}

=== 7. NULL SAFETY ===
Nickname (chưa gán): null
Nickname (đã gán): Dev Flutter
Middle name: Không có
Độ dài tên: 12

=== HOÀN THÀNH VD2 ===
```

> **Lưu ý:** Dòng "Thời gian" sẽ hiển thị thời điểm bạn chạy chương trình.

### Giải thích

| Dòng code                          | Giải thích                                              |
|-------------------------------------|----------------------------------------------------------|
| `var language = 'Dart';`           | Dart suy ra `language` là `String`, cho phép thay đổi giá trị |
| `final city = 'Hà Nội';`          | Gán một lần duy nhất, giá trị xác định lúc runtime      |
| `const pi = 3.14159;`             | Hằng số biết trước lúc compile, không bao giờ thay đổi  |
| `late String description;`        | Khai báo non-nullable nhưng chưa gán — hứa sẽ gán sau  |
| `String? nickname;`               | Thêm `?` → biến có thể là `String` hoặc `null`          |
| `middleName ?? 'Không có'`        | Nếu `middleName` null → trả về `'Không có'`             |
| `fullName!.length`                | `!` khẳng định `fullName` không null → truy cập `.length` |

---

## VD3: Functions 🟡

### Mục đích
Thực hành khai báo hàm, named parameters, optional parameters, arrow syntax, và `required` keyword.

> **Liên quan tới:** [4. Dart Fundamentals 🔴](01-ly-thuyet.md#4-dart-fundamentals)

### Chuẩn bị

Trong project `hello_dart`, tạo file `bin/functions_demo.dart`:

### Code

```dart
/// Demo functions trong Dart
/// File: bin/functions_demo.dart
/// Chạy: dart run bin/functions_demo.dart

void main() {
  print('=== 1. HÀM CƠ BẢN ===');
  int result = add(10, 20);
  print('10 + 20 = $result');

  sayHello('Dart');

  print('');
  print('=== 2. NAMED PARAMETERS ===');
  // Truyền theo tên — dễ đọc, không cần đúng thứ tự
  createUser(name: 'Nguyễn Văn A', age: 25);
  createUser(age: 30, name: 'Trần Thị B', role: 'admin');

  print('');
  print('=== 3. OPTIONAL POSITIONAL PARAMETERS ===');
  print(greet('An'));
  print(greet('Bình', 'Anh'));

  print('');
  print('=== 4. ARROW FUNCTIONS ===');
  print('3 x 7 = ${multiply(3, 7)}');
  print('Bình phương 5 = ${square(5)}');
  logMessage('Ứng dụng khởi động');

  print('');
  print('=== 5. HÀM TRẢ VỀ NULLABLE ===');
  String? found = findUser(1);
  print('User #1: ${found ?? "Không tìm thấy"}');

  String? notFound = findUser(99);
  print('User #99: ${notFound ?? "Không tìm thấy"}');

  print('');
  print('=== 6. HÀM VỚI DEFAULT VALUES ===');
  printInfo('Flutter');
  printInfo('Dart', version: '3.5');

  print('');
  print('=== HOÀN THÀNH VD3 ===');
}

// --- Định nghĩa các hàm ---

/// Hàm cơ bản với positional parameters
int add(int a, int b) {
  return a + b;
}

/// Hàm void — không trả về giá trị
void sayHello(String name) {
  print('Xin chào $name!');
}

/// Hàm với named parameters
/// {required} = bắt buộc phải truyền
/// Có default value = không bắt buộc
void createUser({
  required String name,
  required int age,
  String role = 'member',
}) {
  print('Tạo user: $name, $age tuổi, vai trò: $role');
}

/// Hàm với optional positional parameters
/// [String? title] — không bắt buộc, mặc định null
String greet(String name, [String? title]) {
  if (title != null) {
    return 'Xin chào $title $name!';
  }
  return 'Xin chào $name!';
}

/// Arrow function — khi body chỉ có 1 expression
int multiply(int a, int b) => a * b;

/// Arrow function — square
int square(int n) => n * n;

/// Arrow void function
void logMessage(String msg) => print('[LOG] $msg');

/// Hàm trả về nullable
String? findUser(int id) {
  // Giả lập database
  final users = {1: 'Admin', 2: 'User'};
  return users[id]; // Trả về null nếu không tìm thấy
}

/// Hàm với named parameters có default value
void printInfo(String name, {String version = '1.0'}) {
  print('$name v$version');
}
```

### Chạy

```bash
dart run bin/functions_demo.dart
```

### Kết quả mong đợi

```
=== 1. HÀM CƠ BẢN ===
10 + 20 = 30
Xin chào Dart!

=== 2. NAMED PARAMETERS ===
Tạo user: Nguyễn Văn A, 25 tuổi, vai trò: member
Tạo user: Trần Thị B, 30 tuổi, vai trò: admin

=== 3. OPTIONAL POSITIONAL PARAMETERS ===
Xin chào An!
Xin chào Anh Bình!

=== 4. ARROW FUNCTIONS ===
3 x 7 = 21
Bình phương 5 = 25
[LOG] Ứng dụng khởi động

=== 5. HÀM TRẢ VỀ NULLABLE ===
User #1: Admin
User #99: Không tìm thấy

=== 6. HÀM VỚI DEFAULT VALUES ===
Flutter v1.0
Dart v3.5

=== HOÀN THÀNH VD3 ===
```

### Giải thích

| Dòng code                              | Giải thích                                           |
|-----------------------------------------|------------------------------------------------------|
| `int add(int a, int b)`               | Hàm trả về `int`, nhận 2 tham số positional          |
| `void sayHello(String name)`           | Hàm không trả về gì (`void`)                         |
| `{required String name}`              | Named parameter bắt buộc — gọi bằng `name: 'A'`     |
| `String role = 'member'`              | Named parameter có default value — không bắt buộc    |
| `[String? title]`                      | Optional positional — có thể không truyền             |
| `int multiply(int a, int b) => a * b;`| Arrow syntax — shorthand cho `{ return a * b; }`     |
| `String? findUser(int id)`            | Hàm có thể trả về `null` → kiểu trả về là `String?` |

---

## VD4: Flutter Hello World 🟡

### Mục đích
Tạo ứng dụng Flutter đầu tiên. Hiểu `MaterialApp`, `Scaffold`, `AppBar`, `Center`, `Text`, và cách widget lồng nhau.

> **Liên quan tới:** [6. Chương trình Flutter đầu tiên 🟡](01-ly-thuyet.md#6-chương-trình-flutter-đầu-tiên)

### Chuẩn bị

```bash
# Tạo project Flutter MỚI (khác với Dart CLI ở trên)
flutter create hello_flutter
cd hello_flutter
```

### Code

Mở `lib/main.dart`, xóa **toàn bộ** nội dung mặc định và thay bằng:

```dart
/// Flutter Hello World
/// File: lib/main.dart
/// Chạy: flutter run

import 'package:flutter/material.dart';

// Entry point — mọi ứng dụng Dart/Flutter bắt đầu từ main()
void main() {
  // runApp() là hàm đặc biệt của Flutter
  // Nó nhận một Widget và gắn vào màn hình
  runApp(const MyApp());
}

// MyApp — Widget gốc của ứng dụng
// StatelessWidget = widget KHÔNG có state thay đổi (bất biến)
class MyApp extends StatelessWidget {
  // Constructor — const để Flutter tối ưu performance
  const MyApp({super.key});

  // build() — method BẮT BUỘC của mọi widget
  // Trả về cây widget mô tả UI
  @override
  Widget build(BuildContext context) {
    // MaterialApp — widget gốc, cung cấp Material Design
    return MaterialApp(
      // Tắt banner "DEBUG" góc phải
      debugShowCheckedModeBanner: false,

      // Trang chủ
      home: Scaffold(
        // Scaffold cung cấp layout cơ bản: AppBar, Body, FAB, ...

        // AppBar — thanh bar phía trên
        appBar: AppBar(
          title: const Text('Hello Flutter'),
          backgroundColor: Colors.blue,
          foregroundColor: Colors.white,
        ),

        // body — nội dung chính
        body: const Center(
          // Center — canh giữa widget con
          child: Column(
            // Column — xếp widget theo chiều dọc
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Icon lớn
              Icon(
                Icons.flutter_dash,
                size: 80,
                color: Colors.blue,
              ),

              // Khoảng cách 16px
              SizedBox(height: 16),

              // Text chính
              Text(
                'Xin chào, Flutter!',
                style: TextStyle(
                  fontSize: 28,
                  fontWeight: FontWeight.bold,
                  color: Colors.black87,
                ),
              ),

              // Khoảng cách 8px
              SizedBox(height: 8),

              // Text phụ
              Text(
                'Đây là ứng dụng Flutter đầu tiên của tôi 🎉',
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.grey,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Chạy

```bash
# Mở emulator/simulator trước, sau đó:
flutter run

# Hoặc trong VS Code: F5
```

### Kết quả mong đợi

```
┌─────────────────────────────────┐
│  ◀  Hello Flutter               │  ← AppBar (thanh bar xanh)
├─────────────────────────────────┤
│                                 │
│                                 │
│          🐦 (Flutter Dash)      │  ← Icon Flutter
│                                 │
│     Xin chào, Flutter!          │  ← Text đậm, size 28
│                                 │
│  Đây là ứng dụng Flutter đầu   │  ← Text xám, size 16
│  tiên của tôi 🎉                │
│                                 │
│                                 │
└─────────────────────────────────┘
```

### Giải thích — Widget tree

```
MyApp
 └── MaterialApp
      └── Scaffold
           ├── AppBar
           │    └── Text('Hello Flutter')
           └── body: Center
                └── Column
                     ├── Icon(flutter_dash)
                     ├── SizedBox(height: 16)
                     ├── Text('Xin chào, Flutter!')
                     ├── SizedBox(height: 8)
                     └── Text('Đây là ứng dụng...')
```

| Widget                | Vai trò                                              |
|-----------------------|------------------------------------------------------|
| `MaterialApp`         | Widget gốc, cung cấp theme, navigation, locale       |
| `Scaffold`            | Khung layout cơ bản với AppBar, body, FAB, drawer    |
| `AppBar`              | Thanh bar phía trên màn hình                          |
| `Center`              | Canh giữa widget con (cả ngang và dọc)               |
| `Column`              | Xếp các widget con theo chiều dọc                     |
| `Icon`                | Hiển thị icon từ Material Icons                       |
| `SizedBox`            | Tạo khoảng cách cố định giữa các widget              |
| `Text`                | Hiển thị text                                         |
| `TextStyle`           | Tùy chỉnh font size, weight, color cho Text           |

- 🔗 **FE tương đương:** `MaterialApp` + `Scaffold` ≈ React `<App>` + layout wrapper. `Text('Hello')` ≈ `<p>Hello</p>` — nhưng mọi thứ là Widget, không có HTML tags.

---

## VD5: Hot Reload Demo 🟢

### Mục đích
Trải nghiệm Hot Reload — thay đổi code → thấy kết quả ngay lập tức trên emulator mà không cần khởi động lại app.

> **Liên quan tới:** [6. Chương trình Flutter đầu tiên 🟡](01-ly-thuyet.md#6-chương-trình-flutter-đầu-tiên)

### Chuẩn bị

Sử dụng project `hello_flutter` từ VD4. Đảm bảo app đang chạy (`flutter run` hoặc F5 trong VS Code).

### Bước 1 — Trạng thái ban đầu

App đang hiển thị "Xin chào, Flutter!" với text màu đen và icon xanh (code từ VD4).

### Bước 2 — Đổi text và màu sắc

**Không dừng app.** Mở `lib/main.dart` và thay đổi phần `body`:

```dart
/// Hot Reload Demo — Thay đổi sau khi app đang chạy
/// File: lib/main.dart (thay thế toàn bộ)

import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        // THAY ĐỔI 1: Đổi màu AppBar
        appBar: AppBar(
          title: const Text('Hot Reload Demo'),
          backgroundColor: Colors.deepPurple, // Từ blue → deepPurple
          foregroundColor: Colors.white,
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // THAY ĐỔI 2: Đổi icon và màu
              const Icon(
                Icons.rocket_launch, // Từ flutter_dash → rocket_launch
                size: 100,           // Từ 80 → 100
                color: Colors.deepPurple, // Từ blue → deepPurple
              ),
              const SizedBox(height: 16),

              // THAY ĐỔI 3: Đổi text
              const Text(
                'Hot Reload thật tuyệt! 🚀',
                style: TextStyle(
                  fontSize: 28,
                  fontWeight: FontWeight.bold,
                  color: Colors.deepPurple, // Từ black87 → deepPurple
                ),
              ),
              const SizedBox(height: 8),

              // THAY ĐỔI 4: Thêm text mô tả
              const Text(
                'Thay đổi code → Save → UI cập nhật ngay!',
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.grey,
                ),
              ),
              const SizedBox(height: 24),

              // THAY ĐỔI 5: Thêm container
              Container(
                padding: const EdgeInsets.symmetric(
                  horizontal: 24,
                  vertical: 12,
                ),
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(20),
                  border: Border.all(
                    color: Colors.deepPurple.shade200,
                  ),
                ),
                child: const Text(
                  '⏱️ Chỉ ~1 giây',
                  style: TextStyle(
                    fontSize: 18,
                    color: Colors.deepPurple,
                    fontWeight: FontWeight.w500,
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Bước 3 — Hot Reload

```
Cách 1: Trong terminal đang chạy flutter run → nhấn phím 'r'
Cách 2: Trong VS Code → Cmd+S (save file, tự động hot reload)
```

### Kết quả mong đợi

**TRƯỚC Hot Reload:**
```
┌──────────────────────────┐
│  Hello Flutter     (xanh)│
├──────────────────────────┤
│       🐦 (xanh)          │
│   Xin chào, Flutter!     │
│   Đây là ứng dụng...     │
└──────────────────────────┘
```

**SAU Hot Reload (~1 giây):**
```
┌──────────────────────────┐
│  Hot Reload Demo   (tím) │
├──────────────────────────┤
│       🚀 (tím, lớn hơn) │
│  Hot Reload thật tuyệt!  │
│  Thay đổi code → Save... │
│    ┌──────────────┐      │
│    │ ⏱️ Chỉ ~1 giây│     │
│    └──────────────┘      │
└──────────────────────────┘
```

### Những gì đã thay đổi

| #  | Thay đổi                      | Trước              | Sau                     |
|----|-------------------------------|---------------------|--------------------------|
| 1  | Màu AppBar                    | `Colors.blue`       | `Colors.deepPurple`      |
| 2  | Icon                          | `flutter_dash`, 80  | `rocket_launch`, 100     |
| 3  | Text chính                    | "Xin chào, Flutter!"| "Hot Reload thật tuyệt!" |
| 4  | Màu text                      | `black87`           | `deepPurple`             |
| 5  | Thêm widget mới               | —                   | Container với border     |

### Lưu ý quan trọng về Hot Reload

```
✅ Hot Reload hoạt động khi:
   - Thay đổi UI: text, color, size, layout
   - Thay đổi logic trong build()
   - Thêm/xóa widget

❌ Hot Reload KHÔNG hoạt động khi (cần Hot Restart):
   - Thay đổi hàm main()
   - Thêm/xóa field trong class
   - Thay đổi constructor signature
   - Thay đổi generic types

Phím tắt:
   r  → Hot Reload (nhanh, giữ state)
   R  → Hot Restart (chậm hơn, reset state)
   q  → Thoát flutter run
```

---

> **Tiếp theo:** [03-thuc-hanh.md](./03-thuc-hanh.md) — 3 bài tập thực hành
