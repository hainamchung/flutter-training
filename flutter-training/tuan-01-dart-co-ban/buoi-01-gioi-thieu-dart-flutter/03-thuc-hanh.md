# Buổi 01 — Thực hành: Giới thiệu Dart & Flutter

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 1/16** · **4 bài tập + 3 câu hỏi thảo luận** · **Cập nhật:** 2026-04-01

---

## Mục lục

1. [BT1 ⭐ Setup Environment](#bt1--setup-environment)
2. [BT2 ⭐ Dart CLI Calculator](#bt2--dart-cli-calculator)
3. [BT3 ⭐⭐ Flutter Profile Card](#bt3--flutter-profile-card)
4. [BT4 ⭐⭐⭐ Dart Type-Safe Data Processor](#bt4--dart-type-safe-data-processor)
5. [Câu hỏi thảo luận](#câu-hỏi-thảo-luận)

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ TypeScript/JavaScript**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| TypeScript/JS Habit | Dart Reality | Bài tập liên quan |
|---------------------|--------------|---------------------|
| `const x = [1,2,3]` — array vẫn mutable | Dart `const` = **compile-time immutable** — `const [1,2,3]` không thể add/remove | BT1, BT2 |
| String interpolation: `` `Hello ${name}` `` | Dart: `'Hello $name'` hoặc `'Hello ${expression}'` — dùng `$` thay vì `${}` | BT1 |
| `null` check bằng `?.` optional | Dart null safety tương tự nhưng **bắt buộc** — `String?` phải handle, không thể bỏ qua | BT2, BT3 |

---

## BT1 ⭐ Setup Environment 🔴

### Thông tin

| Mục          | Chi tiết                                  |
|--------------|-------------------------------------------|
| **Loại project**   | Setup                                     |
| **Độ khó**   | ⭐ Cơ bản                                 |
| **Thời gian** | ~30 phút                                 |
| **Output**   | App mẫu Flutter chạy trên emulator        |

### Yêu cầu

Cài đặt môi trường phát triển Flutter hoàn chỉnh và chạy thành công app mẫu.

### Các bước thực hiện

#### Bước 1: Cài Flutter SDK

```bash
# macOS với Homebrew
brew install --cask flutter

# Kiểm tra
flutter --version
```

#### Bước 2: Cài VS Code Extensions

```
1. Mở VS Code
2. Cmd+Shift+X (Extensions)
3. Tìm "Flutter" → Install (by Dart-Code)
4. Extension "Dart" sẽ được cài tự động
```

#### Bước 3: Cài Emulator

```bash
# iOS Simulator (macOS)
sudo xcodebuild -license accept
open -a Simulator

# HOẶC Android Emulator
# Mở Android Studio → Virtual Device Manager → Create Device → Start
```

#### Bước 4: Chạy flutter doctor

```bash
flutter doctor
```

**Mục tiêu:** Tất cả dòng hiển thị dấu ✓. Nếu có ✗ hoặc !, đọc hướng dẫn fix bên dưới mỗi dòng.

#### Bước 5: Tạo và chạy app Flutter

```bash
# Tạo project mới
flutter create bt1_setup

# Vào thư mục project
cd bt1_setup

# Chạy app
flutter run
```

#### Bước 6: Chụp ảnh màn hình

Chụp ảnh emulator đang chạy app mẫu Flutter (app counter mặc định).

### Tiêu chí hoàn thành

| #  | Tiêu chí                                    | ✅ |
|----|----------------------------------------------|----|
| 1  | `flutter --version` hiển thị version         | ☐  |
| 2  | `flutter doctor` không có lỗi critical       | ☐  |
| 3  | VS Code có Flutter & Dart extensions          | ☐  |
| 4  | Emulator/Simulator mở thành công              | ☐  |
| 5  | App mẫu chạy trên emulator                   | ☐  |
| 6  | Nhấn nút + (FAB) → số tăng lên               | ☐  |

---

## BT2 ⭐ Dart CLI Calculator 🟡

### Thông tin

| Mục          | Chi tiết                                  |
|--------------|-------------------------------------------|
| **Loại project**   | Dart CLI                                  |
| **Độ khó**   | ⭐ Cơ bản                                 |
| **Thời gian** | ~30 phút                                 |
| **Tạo project** | `dart create`                             |
| **Cách chạy**   | `dart run`                                |
| **Output**   | Text kết quả trong terminal               |

### Yêu cầu

Viết chương trình Dart CLI thực hiện các phép tính cơ bản. Chương trình phải sử dụng:
- Variables: `var`, `final`, `const`
- Types: `int`, `double`, `String`, `bool`
- Null safety: `?`, `??`
- Functions: named parameters, arrow functions
- String interpolation
- Control flow: `if/else`, `switch`

### Các bước thực hiện

#### Bước 1: Tạo project

```bash
dart create dart_calculator
cd dart_calculator
```

#### Bước 2: Viết code

Mở file `bin/dart_calculator.dart`, xóa nội dung mặc định và viết chương trình máy tính.

### Gợi ý cấu trúc (đọc trước khi xem đáp án)

```
1. Định nghĩa hằng số: const appName, const version
2. Viết hàm calculate():
   - Nhận named parameters: {required double a, required double b, required String operator}
   - Dùng switch để xử lý +, -, *, /
   - Xử lý chia cho 0 → trả về null (double?)
3. Viết hàm formatResult():
   - Nhận double? → trả về String
   - Dùng ?? để xử lý null
4. Trong main():
   - In tên app + version
   - Gọi calculate() với nhiều phép tính
   - In kết quả đẹp bằng string interpolation
```

### Đáp án tham khảo

> 💡 **Dart 3 Records** — Cú pháp `(Type1, Type2)` là Records trong Dart 3. Nếu chưa quen, hãy đọc trước phần Records trong [Buổi 02: Dart nâng cao](../buoi-02-dart-nang-cao/01-ly-thuyet.md) rồi quay lại làm bài này.

> 💡 **Records syntax**: Dart 3.0+ hỗ trợ Records — kiểu dữ liệu nhóm nhiều giá trị: `(int, String)` tạo tuple. Nếu chưa quen, thay bằng `class` hoặc `Map` đều được.

> **⚠️ Cố gắng tự làm trước khi xem đáp án!**

<details>
<summary>👉 Click để xem đáp án</summary>

```dart
/// BT2: Dart CLI Calculator
/// File: bin/dart_calculator.dart
/// Chạy: dart run

void main() {
  // Hằng số compile-time
  const appName = 'Dart Calculator';
  const version = '1.0.0';

  print('================================');
  print(' $appName v$version');
  print('================================');
  print('');

  // Các phép tính demo
  final calculations = [
    (a: 10.0, b: 3.0, op: '+'),
    (a: 25.5, b: 4.5, op: '-'),
    (a: 7.0, b: 8.0, op: '*'),
    (a: 20.0, b: 3.0, op: '/'),
    (a: 10.0, b: 0.0, op: '/'), // Chia cho 0
  ];

  // Dùng for-in loop để tính toàn bộ
  for (final calc in calculations) {
    double? result = calculate(
      a: calc.a,
      b: calc.b,
      operator: calc.op,
    );

    String formatted = formatResult(result);
    print('  ${calc.a} ${calc.op} ${calc.b} = $formatted');
  }

  print('');

  // Demo null safety
  print('--- Demo Null Safety ---');
  String? userName;
  print('User (chưa gán): ${userName ?? "Khách"}');

  userName = 'Flutter Dev';
  print('User (đã gán): ${userName}');

  print('');

  // Demo final vs var
  print('--- Demo Variables ---');
  var counter = 0;
  counter++;
  counter++;
  print('Counter (var, thay đổi được): $counter');

  final createdAt = DateTime.now();
  print('Created at (final): $createdAt');

  // Demo bool
  bool isPositive = isNumberPositive(42);
  print('42 là số dương? $isPositive');

  isPositive = isNumberPositive(-5);
  print('-5 là số dương? $isPositive');

  print('');
  print('================================');
  print(' Hoàn thành! 🎉');
  print('================================');
}

/// Hàm tính toán — dùng named parameters
/// Trả về double? vì có thể thất bại (chia cho 0)
double? calculate({
  required double a,
  required double b,
  required String operator,
}) {
  switch (operator) {
    case '+':
      return a + b;
    case '-':
      return a - b;
    case '*':
      return a * b;
    case '/':
      if (b == 0) return null; // Không chia được cho 0
      return a / b;
    default:
      return null; // Operator không hợp lệ
  }
}

/// Format kết quả — xử lý null bằng ??
String formatResult(double? value) =>
    value != null ? value.toStringAsFixed(2) : 'Lỗi (chia cho 0)';

/// Arrow function — kiểm tra số dương
bool isNumberPositive(double n) => n > 0;
```

</details>

### Chạy

```bash
dart run
```

### Kết quả mong đợi

```
================================
 Dart Calculator v1.0.0
================================

  10.0 + 3.0 = 13.00
  25.5 - 4.5 = 21.00
  7.0 * 8.0 = 56.00
  20.0 / 3.0 = 6.67
  10.0 / 0.0 = Lỗi (chia cho 0)

--- Demo Null Safety ---
User (chưa gán): Khách
User (đã gán): Flutter Dev

--- Demo Variables ---
Counter (var, thay đổi được): 2
Created at (final): 2026-03-31 10:30:00.000000
42 là số dương? true
-5 là số dương? false

================================
 Hoàn thành! 🎉
================================
```

### Tiêu chí hoàn thành

| #  | Tiêu chí                                      | ✅ |
|----|------------------------------------------------|----|
| 1  | Chạy `dart run` thành công, không lỗi          | ☐  |
| 2  | Có dùng `const` cho hằng số                    | ☐  |
| 3  | Có dùng `final` cho biến gán một lần           | ☐  |
| 4  | Hàm `calculate()` dùng named parameters        | ☐  |
| 5  | Xử lý chia cho 0 trả về `null` (dùng `double?`)| ☐  |
| 6  | Có dùng `??` để xử lý null                     | ☐  |
| 7  | Có arrow function (`=>`)                        | ☐  |
| 8  | String interpolation hoạt động đúng            | ☐  |

---

## BT3 ⭐⭐ Flutter Profile Card 🟡

### Thông tin

| Mục          | Chi tiết                                        |
|--------------|-------------------------------------------------|
| **Loại project**   | Flutter UI                                      |
| **Độ khó**   | ⭐⭐ Trung bình                                 |
| **Thời gian** | ~45 phút                                       |
| **Tạo project** | `flutter create`                               |
| **Cách chạy**   | `flutter run`                                  |
| **Output**   | UI profile card trên emulator                   |

### Yêu cầu

Tạo Flutter app hiển thị thẻ thông tin cá nhân (profile card) với:
- Ảnh đại diện (dùng `CircleAvatar` với `Icon`)
- Tên, chức danh
- Thông tin liên hệ (email, điện thoại, địa chỉ)
- Layout dùng `Column`, `Row`, `Container`, `Text`, `Icon`

### Mockup

```
┌────────────────────────────────┐
│           PROFILE              │  ← AppBar
├────────────────────────────────┤
│                                │
│          ┌──────┐              │
│          │  👤  │              │  ← CircleAvatar
│          └──────┘              │
│                                │
│       Nguyễn Văn A             │  ← Tên (bold, size 28)
│     Flutter Developer          │  ← Chức danh (grey)
│                                │
│  ─────────────────────────     │  ← Divider
│                                │
│  📧  email@example.com         │  ← Row: Icon + Text
│  📱  +84 123 456 789           │  ← Row: Icon + Text
│  📍  Hà Nội, Việt Nam          │  ← Row: Icon + Text
│                                │
└────────────────────────────────┘
```

### Các bước thực hiện

#### Bước 1: Tạo project

```bash
flutter create profile_card
cd profile_card
```

#### Bước 2: Viết code

Mở `lib/main.dart`, xóa nội dung mặc định.

### Gợi ý cấu trúc

```
MaterialApp
 └── Scaffold
      ├── AppBar('Profile')
      └── body: Center
           └── Padding
                └── Column (center)
                     ├── CircleAvatar (radius: 50, Icon person)
                     ├── SizedBox(16)
                     ├── Text(tên, bold, size 28)
                     ├── SizedBox(4)
                     ├── Text(chức danh, grey)
                     ├── SizedBox(24)
                     ├── Divider
                     ├── SizedBox(16)
                     ├── _buildInfoRow(icon: email, text: ...)
                     ├── _buildInfoRow(icon: phone, text: ...)
                     └── _buildInfoRow(icon: location, text: ...)
```

### Đáp án tham khảo

> **⚠️ Cố gắng tự làm trước khi xem đáp án!**

<details>
<summary>👉 Click để xem đáp án</summary>

```dart
/// BT3: Flutter Profile Card
/// File: lib/main.dart
/// Chạy: flutter run

import 'package:flutter/material.dart';

void main() {
  runApp(const ProfileApp());
}

class ProfileApp extends StatelessWidget {
  const ProfileApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.teal,
        useMaterial3: true,
      ),
      home: const ProfileScreen(),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  // Thông tin cá nhân — dùng final vì không thay đổi
  final String name = 'Nguyễn Văn A';
  final String title = 'Flutter Developer';
  final String email = 'nguyenvana@example.com';
  final String phone = '+84 123 456 789';
  final String location = 'Hà Nội, Việt Nam';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // AppBar
      appBar: AppBar(
        title: const Text('Profile'),
        centerTitle: true,
      ),

      // Body
      body: Center(
        child: Padding(
          // Padding — thêm khoảng cách xung quanh
          padding: const EdgeInsets.all(24.0),
          child: Column(
            // Canh giữa theo chiều dọc
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Avatar — hình tròn chứa icon
              const CircleAvatar(
                radius: 50,
                backgroundColor: Colors.teal,
                child: Icon(
                  Icons.person,
                  size: 50,
                  color: Colors.white,
                ),
              ),

              const SizedBox(height: 16),

              // Tên
              Text(
                name,
                style: const TextStyle(
                  fontSize: 28,
                  fontWeight: FontWeight.bold,
                ),
              ),

              const SizedBox(height: 4),

              // Chức danh
              Text(
                title,
                style: TextStyle(
                  fontSize: 18,
                  color: Colors.grey[600],
                ),
              ),

              const SizedBox(height: 24),

              // Đường kẻ ngang
              const Divider(thickness: 1),

              const SizedBox(height: 16),

              // Thông tin liên hệ
              _buildInfoRow(
                icon: Icons.email_outlined,
                text: email,
              ),

              const SizedBox(height: 12),

              _buildInfoRow(
                icon: Icons.phone_outlined,
                text: phone,
              ),

              const SizedBox(height: 12),

              _buildInfoRow(
                icon: Icons.location_on_outlined,
                text: location,
              ),
            ],
          ),
        ),
      ),
    );
  }

  /// Helper method — tạo 1 row thông tin (icon + text)
  /// Dùng named parameters cho dễ đọc
  Widget _buildInfoRow({
    required IconData icon,
    required String text,
  }) {
    return Row(
      children: [
        // Icon bên trái
        Icon(
          icon,
          color: Colors.teal,
          size: 24,
        ),

        // Khoảng cách giữa icon và text
        const SizedBox(width: 16),

        // Text thông tin
        Text(
          text,
          style: const TextStyle(fontSize: 16),
        ),
      ],
    );
  }
}
```

</details>

### Chạy

```bash
flutter run
```

### Kết quả mong đợi

Trên emulator hiển thị:
- AppBar "Profile" ở trên cùng
- Hình tròn teal chứa icon người
- Tên in đậm, size 28
- Chức danh màu xám
- Đường kẻ ngang
- 3 dòng thông tin: email, phone, location — mỗi dòng có icon bên trái

### Thử Hot Reload

Sau khi app chạy, thử thay đổi:
1. Đổi `name` thành tên của bạn → Save → xem app cập nhật
2. Đổi `Colors.teal` thành `Colors.indigo` → Save
3. Đổi `radius: 50` thành `radius: 70` → Save

### Tiêu chí hoàn thành

| #  | Tiêu chí                                      | ✅ |
|----|------------------------------------------------|----|
| 1  | `flutter run` chạy thành công                  | ☐  |
| 2  | Có `CircleAvatar` hiển thị avatar               | ☐  |
| 3  | Tên hiển thị đậm, size lớn                      | ☐  |
| 4  | Chức danh hiển thị màu xám                      | ☐  |
| 5  | Có `Divider` ngăn cách                          | ☐  |
| 6  | 3 dòng info dùng `Row` (Icon + Text)            | ☐  |
| 7  | Dùng helper method `_buildInfoRow()`            | ☐  |
| 8  | Hot reload hoạt động khi thay đổi thông tin     | ☐  |

---

## BT4 ⭐⭐⭐ Dart Type-Safe Data Processor 🟡

### Thông tin

| Mục          | Chi tiết                                  |
|--------------|-------------------------------------------|
| **Loại project**   | Dart CLI                                  |
| **Độ khó**   | ⭐⭐⭐ Nâng cao                             |
| **Thời gian** | ~60 phút                                 |
| **Output**   | Dart CLI app xử lý dữ liệu có type safety |

### Yêu cầu

Tạo một Dart CLI app xử lý danh sách nhân viên từ data hardcode:
1. Định nghĩa class `Employee` với: `name` (String), `department` (String?), `salary` (double)
2. Sử dụng `final`, `const`, null safety (`?`, `??`, `!`) đúng cách
3. Viết hàm: lọc theo department, tính trung bình lương, tìm max/min
4. In kết quả dạng formatted table

### Tiêu chí hoàn thành

- [ ] Không dùng `var` khi có thể dùng `final`
- [ ] Xử lý null safety cho `department` (nullable)
- [ ] Code clean, tên biến rõ ràng, theo Dart conventions
- [ ] Chạy tốt bằng `dart run`

### 🤖 Thực hành cùng AI

**Prompt gợi ý:**
```text
Review Dart code này. Kiểm tra: null safety usage, final/const usage, naming conventions.
Gợi ý cải thiện mà không viết lại toàn bộ.
```

---

## Câu hỏi thảo luận

Suy nghĩ và trả lời 3 câu hỏi sau. Không cần code — chỉ cần giải thích bằng lời:

### Câu 1: Dart vs TypeScript

> Dart có **Sound Null Safety** — nghĩa là nếu code compile thành công thì **chắc chắn** không có lỗi null ở runtime (trừ khi dùng `!` sai). TypeScript cũng có strict mode nhưng vẫn có thể bypass.
>
> **Câu hỏi:** Tại sao "sound" null safety quan trọng hơn cho mobile app so với web app? Nghĩ về hậu quả khi app crash trên điện thoại người dùng vs crash trên browser.

### Câu 2: Flutter vs React Native Architecture

> Flutter tự vẽ mọi pixel bằng engine Skia/Impeller. React Native dùng bridge để giao tiếp với native components.
>
> **Câu hỏi:** Cách tiếp cận nào cho UI consistent hơn giữa iOS và Android? Tại sao? Cách nào cho cảm giác "native" hơn? Có trade-off gì?

### Câu 3: Hot Reload

> Khi bạn nhấn hot reload, Flutter chỉ recompile các hàm đã thay đổi (JIT), inject vào Dart VM đang chạy, và gọi lại `build()` trên các widget bị ảnh hưởng.
>
> **Câu hỏi:** Tại sao thay đổi hàm `main()` hoặc thêm field mới vào class lại KHÔNG thể hot reload mà cần hot restart? (Gợi ý: nghĩ về thứ tự khởi tạo và memory layout)

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 1:** Focus vào đọc hiểu code AI gen và verify tính đúng đắn.

### AI-BT1: Gen Dart code + Verify Null Safety ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Null Safety — `?`, `!`, `??`, `late`, sound null safety.
- **Task thực tế:** PM giao task "Parse user data từ API response, field avatar và bio có thể null, field score trả về dạng String thay vì int". Cần viết Dart class parse an toàn, không crash khi server trả về data bất thường.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần tạo Dart data class để parse JSON response từ REST API.
Context: Flutter app, dùng Dart 3.x, sound null safety.
Input JSON mẫu: {"id": 1, "name": "Nam", "avatar": null, "bio": null, "score": "85", "isActive": true}
Constraints:
- KHÔNG dùng dynamic type ở bất kỳ đâu.
- KHÔNG dùng ! operator (bang operator). Xử lý null bằng ??, ?., ??=.
- fromJson factory phải xử lý: field null, field sai kiểu (score là String thay vì int), field thiếu hoàn toàn.
- Tạo thêm toJson method.
- Dùng final cho tất cả fields.
Output: 1 file user.dart hoàn chỉnh với class User, fromJson, toJson, và toString.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file `user.dart` với class `User`, `factory User.fromJson(Map<String, dynamic> json)`, `Map<String, dynamic> toJson()`, `@override String toString()`.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Không có ký tự `!` nào trong code (trừ trong comment)? | ☐ |
| 2 | Không có `dynamic` nào (ngoài parameter `Map<String, dynamic> json`)? | ☐ |
| 3 | `avatar` và `bio` khai báo là `String?` (nullable)? | ☐ |
| 4 | `score` parse từ String sang int bằng `int.tryParse` + `?? 0`? | ☐ |
| 5 | Tất cả fields dùng `final`? | ☐ |
| 6 | `dart analyze` không có warning? | ☐ |

**4. Customize:**
Tự thêm: method `copyWith()` cho class User — cho phép update 1 field mà giữ nguyên các field còn lại. AI chưa làm phần này. Implement `copyWith({String? name, String? avatar, ...})` với null-aware logic.

---

### AI-BT2: Gen Flutter Widget + Verify Lifecycle ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** StatelessWidget, `runApp`, `MaterialApp`, `Scaffold`, widget composition.
- **Task thực tế:** Tạo widget đồng hồ đếm ngược (OTP countdown) — loại component xuất hiện trong mọi app có tính năng xác thực OTP. Cần update UI mỗi giây và hiển thị nút "Gửi lại" khi hết giờ.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần tạo Flutter widget OtpCountdownTimer — đếm ngược từ 60 giây xuống 0.
Context: app xác thực OTP, widget này sẽ được dùng trong nhiều màn hình. Flutter 3.x, Dart 3.x.
Constraints:
- Dùng StatefulWidget (vì cần Timer state).
- Timer.periodic trong initState, cancel timer trong dispose() để tránh memory leak.
- Check mounted trước khi gọi setState.
- Khi hết giờ: ẩn countdown, hiện nút "Gửi lại mã" có onResend callback.
- Dùng const constructor cho những phần static.
- Widget nhận 2 named params: {required int seconds, required VoidCallback onResend}.
Output: 1 file widget hoàn chỉnh, tự chứa, có thể import vào màn hình khác.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file `otp_countdown_timer.dart` với `StatefulWidget`, `Timer` biến private, `initState`, `dispose`, `build` trả về layout với Text countdown và nút Resend.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | `_timer` được gán vào biến private, không gọi fire-and-forget? | ☐ |
| 2 | `_timer?.cancel()` có trong `dispose()` TRƯỚC `super.dispose()`? | ☐ |
| 3 | `setState` có được bọc trong `if (mounted)`? | ☐ |
| 4 | Khi `_secondsLeft == 0`, timer có được cancel ngay không? | ☐ |
| 5 | `const` được dùng đúng chỗ (Text static, Icon static)? | ☐ |
| 6 | Widget nhận `onResend` callback qua named parameter? | ☐ |
| 7 | `flutter analyze` không có warning? | ☐ |

**4. Customize:**
Tự thêm: khi user nhấn "Gửi lại", countdown reset về 60 và đếm lại (AI chưa làm phần này). Implement hàm `_resetTimer()` và gắn vào nút. Thêm visual: đổi màu text sang đỏ khi còn 10 giây cuối.

---

> **Tiếp theo:** [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md) — Tài liệu & link tham khảo
