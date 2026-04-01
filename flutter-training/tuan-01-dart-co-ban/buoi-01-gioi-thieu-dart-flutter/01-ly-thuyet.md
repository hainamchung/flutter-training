# Buổi 01 — Lý thuyết: Giới thiệu Dart & Flutter

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 1/16** · **Thời lượng tự học:** ~45 phút · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Biết 1 ngôn ngữ lập trình (JS/TS recommended)

---

## Mục lục

1. [Flutter là gì?](#1-flutter-là-gì)
2. [Dart là gì?](#2-dart-là-gì)
3. [Cài đặt môi trường](#3-cài-đặt-môi-trường)
4. [Dart Fundamentals](#4-dart-fundamentals)
5. [Chương trình Dart đầu tiên](#5-chương-trình-dart-đầu-tiên)
6. [Chương trình Flutter đầu tiên](#6-chương-trình-flutter-đầu-tiên)
7. [pubspec.yaml](#7-pubspecyaml)
8. [Best Practices & Lỗi thường gặp](#8-best-practices--lỗi-thường-gặp)
9. [💡 FE → Flutter: Góc nhìn chuyển đổi](#9--fe--flutter-góc-nhìn-chuyển-đổi)
10. [Tổng kết](#10-tổng-kết)

---

## 1. Flutter là gì? 🔴

### Định nghĩa

**Flutter** là một **UI toolkit mã nguồn mở** do Google phát triển, giúp bạn xây dựng ứng dụng đẹp, natively compiled cho **mobile** (iOS, Android), **web**, và **desktop** (Windows, macOS, Linux) — tất cả từ **một codebase duy nhất**.

### Tại sao Flutter tồn tại?

Trước Flutter, nếu muốn làm app cho cả iOS và Android, bạn có 2 lựa chọn:

| Cách tiếp cận       | Ưu điểm              | Nhược điểm                        |
|----------------------|-----------------------|-----------------------------------|
| Native (Swift/Kotlin)| Performance tốt nhất | Viết 2 lần, tốn gấp đôi chi phí  |
| Hybrid (React Native)| Một codebase          | Bridge overhead, UI không nhất quán|

Google tạo Flutter để giải quyết cả hai vấn đề: **một codebase** nhưng **performance gần native** vì Flutter không dùng bridge — nó tự vẽ mọi pixel trên màn hình bằng engine riêng (Skia/Impeller).

### Kiến trúc Flutter

Flutter có 3 lớp chính:

```
┌─────────────────────────────────────────────────┐
│              FRAMEWORK LAYER (Dart)              │
│                                                   │
│  Material / Cupertino (Design widgets)           │
│  Widgets (Text, Row, Column, Stack, ...)         │
│  Rendering (Layout, Painting, Hit testing)       │
│  Foundation (basic classes, utilities)            │
│                                                   │
│  → Đây là phần BẠN viết code. Toàn bộ bằng Dart │
├─────────────────────────────────────────────────┤
│              ENGINE LAYER (C/C++)                │
│                                                   │
│  Skia / Impeller (2D rendering engine)           │
│  Dart Runtime & Compiler                          │
│  Text Layout (libTxt)                             │
│  Platform Channels                                │
│                                                   │
│  → Flutter tự vẽ UI, KHÔNG dùng native widgets   │
├─────────────────────────────────────────────────┤
│            EMBEDDER LAYER (Platform)             │
│                                                   │
│  iOS (Objective-C/Swift)                          │
│  Android (Java/Kotlin)                            │
│  Web (JavaScript)                                 │
│  Desktop (C++/ObjC)                               │
│                                                   │
│  → Giao tiếp với OS: input, rendering surface    │
└─────────────────────────────────────────────────┘
```

**Điểm quan trọng:**
- **Framework layer** — nơi bạn viết code, hoàn toàn bằng Dart
- **Engine layer** — tự vẽ UI pixel-by-pixel (không dùng UIKit hay Android View)
- **Embedder** — kết nối engine với OS cụ thể

### Lợi thế của Flutter

| Lợi thế               | Giải thích                                           |
|------------------------|------------------------------------------------------|
| One codebase           | Viết một lần, chạy trên iOS, Android, Web, Desktop   |
| Hot Reload             | Thay đổi code → thấy kết quả ngay ~1 giây            |
| Pixel-perfect UI       | Flutter tự vẽ, nên UI giống nhau 100% trên mọi nền tảng |
| Performance            | Compile ra native ARM code (không qua bridge)         |
| Hệ sinh thái mạnh     | pub.dev có 40,000+ packages                          |
| Backed by Google       | Dùng cho Google Pay, Google Ads, Alibaba, BMW, ...    |

> 🔗 **FE Bridge:** Flutter ≈ React/Vue nhưng **compile to native** thay vì chạy trên browser engine. Không có DOM, không có CSS, không có HTML — tất cả UI là Widget code. FE dev quen "HTML structure + CSS style + JS logic" → Flutter gộp cả 3 thành Widget tree.

---

## 2. Dart là gì? 🔴

### Định nghĩa

**Dart** là một **ngôn ngữ lập trình** do Google tạo ra, được thiết kế đặc biệt để xây dựng ứng dụng trên nhiều nền tảng. Dart là ngôn ngữ **duy nhất** bạn cần biết để viết Flutter.

### Tại sao Google tạo Dart?

Google tạo Dart vào năm 2011 với mục tiêu ban đầu thay thế JavaScript trên trình duyệt. Sau đó, Dart trở thành ngôn ngữ chính của Flutter vì nó có 3 đặc tính **không ngôn ngữ nào khác có đủ cùng lúc**:

| Đặc tính                        | Tại sao quan trọng cho Flutter                     |
|----------------------------------|-----------------------------------------------------|
| **AOT Compilation** (Ahead-of-Time) | Compile ra machine code → app khởi động nhanh, chạy mượt |
| **JIT Compilation** (Just-in-Time)  | Hot reload trong lúc develop → thay đổi thấy ngay    |
| **Sound Null Safety**            | Bắt lỗi null tại compile time → ít crash hơn         |

- **AOT** dùng khi build production (release mode) → performance cao
- **JIT** dùng khi đang develop (debug mode) → hot reload nhanh
- **Null safety** bắt buộc từ Dart 2.12+ → bạn phải khai báo rõ biến nào có thể null

### Vai trò của Dart trong Flutter

```
Bạn viết Dart code
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  Debug mode  │     │ Release mode │
│  (JIT)       │     │ (AOT)        │
│  → Hot Reload│     │ → ARM binary │
└──────────────┘     └──────────────┘
       │                     │
       ▼                     ▼
  Chạy trên            Cài trên điện
  emulator/device       thoại thật
  (nhanh khi dev)      (nhanh khi dùng)
```

Mọi thứ trong Flutter — từ UI (widget), logic, navigation, state, animation — đều viết bằng Dart. Không cần HTML, CSS, XML, hay ngôn ngữ nào khác.

> 🔗 **FE Bridge:** Dart ≈ TypeScript với **sound null safety** — nhưng **khác ở**: Dart là **compiled language** (AOT cho production, JIT cho development), TypeScript transpile → JavaScript. Dart null safety = **runtime guaranteed**, TypeScript strict mode = chỉ compile-time check. Dart single-threaded + event loop giống JavaScript.

---

## 3. Cài đặt môi trường 🟢

### 3.1 Cài Flutter SDK

**macOS (dùng Homebrew — cách nhanh nhất):**

```bash
# Bước 1: Cài Homebrew nếu chưa có
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Bước 2: Cài Flutter
brew install --cask flutter

# Bước 3: Kiểm tra
flutter --version
```

**macOS (cài thủ công):**

```bash
# Bước 1: Download Flutter SDK
# Vào https://docs.flutter.dev/get-started/install/macos chọn phiên bản mới nhất

# Bước 2: Giải nén vào thư mục bạn chọn (ví dụ ~/development)
mkdir -p ~/development
cd ~/development
unzip ~/Downloads/flutter_macos_arm64_*.zip

# Bước 3: Thêm Flutter vào PATH
# Mở file ~/.zshrc (macOS dùng zsh mặc định)
nano ~/.zshrc

# Thêm dòng này vào cuối file:
export PATH="$HOME/development/flutter/bin:$PATH"

# Lưu file (Ctrl+X → Y → Enter), sau đó:
source ~/.zshrc

# Bước 4: Kiểm tra
flutter --version
```

> **PATH là gì?** PATH là biến môi trường cho hệ điều hành biết tìm các chương trình ở đâu. Khi bạn gõ `flutter` trong terminal, OS sẽ tìm trong các thư mục liệt kê trong PATH. Nếu không thêm Flutter vào PATH, bạn phải gõ đường dẫn đầy đủ mỗi lần.

### 3.2 Cài VS Code + Extensions

```
Bước 1: Tải VS Code từ https://code.visualstudio.com/
Bước 2: Mở VS Code
Bước 3: Vào Extensions (Cmd+Shift+X trên macOS)
Bước 4: Tìm và cài 2 extension:
  - "Flutter" (by Dart-Code) — bao gồm cả Dart extension
  - "Dart" (by Dart-Code) — thường được cài tự động cùng Flutter
```

Sau khi cài, VS Code sẽ tự nhận diện Flutter SDK và cung cấp:
- Code completion (gợi ý code)
- Error highlighting (gạch đỏ lỗi)
- Debug tools
- Nút Run/Debug trên thanh toolbar

### 3.3 iOS Simulator (macOS only)

```bash
# Bước 1: Cài Xcode từ App Store (hoặc dùng lệnh)
xcode-select --install

# Bước 2: Mở Xcode lần đầu, accept license
sudo xcodebuild -license accept

# Bước 3: Cài iOS Simulator runtime (nếu cần)
xcodebuild -downloadPlatform iOS

# Bước 4: Mở Simulator  
open -a Simulator
```

> Khi Simulator mở lên, bạn sẽ thấy một chiếc iPhone ảo trên màn hình. Flutter sẽ tự nhận device này.

### 3.4 Android Emulator

```
Bước 1: Tải Android Studio từ https://developer.android.com/studio
Bước 2: Mở Android Studio → More Actions → Virtual Device Manager
        (hoặc Tools → Device Manager nếu đã có project)
Bước 3: Click "Create Virtual Device"
Bước 4: Chọn device (ví dụ: Pixel 7) → Next
Bước 5: Chọn System Image (ví dụ: API 34) → Download nếu cần → Next
Bước 6: Finish
Bước 7: Click nút Play ▶ để khởi động emulator
```

> **Lưu ý:** Bạn chỉ cần Android Studio để tạo Emulator. Viết code vẫn dùng VS Code.

### 3.5 Kiểm tra với `flutter doctor`

```bash
flutter doctor
```

Kết quả mong đợi (tất cả dấu ✓):

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x)
[✓] Android toolchain - develop for Android devices
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio
[✓] VS Code
[✓] Connected device (2 available)
[✓] Network resources

• No issues found!
```

Nếu có dấu ✗ hoặc !, đọc lời hướng dẫn bên dưới mỗi dòng để fix. Lỗi phổ biến:
- **Android license not accepted** → chạy `flutter doctor --android-licenses`
- **Xcode not installed** → cài Xcode từ App Store
- **No connected device** → mở Simulator hoặc Emulator trước

### 3.6 Chọn device trong VS Code

```
Cách 1: Nhìn góc dưới phải VS Code → click tên device → chọn
Cách 2: Cmd+Shift+P → "Flutter: Select Device"
```

Bạn sẽ thấy danh sách:
- iPhone 16 Pro (ios simulator)
- Pixel 7 (android emulator)
- Chrome (web)
- macOS (desktop)

### 3.7 Dart CLI vs Flutter app

| Đặc điểm     | Dart CLI (`dart run`)       | Flutter App (`flutter run`)     |
|---------------|-----------------------------|---------------------------------|
| Tạo project   | `dart create my_app`        | `flutter create my_app`         |
| Chạy          | `dart run`                  | `flutter run`                   |
| Output        | Text trong terminal          | UI trên emulator/device         |
| Cần emulator? | ❌ Không                     | ✅ Có                            |
| Dùng khi nào  | Học Dart thuần, viết script  | Xây dựng app có giao diện       |
| File chính    | `bin/my_app.dart`           | `lib/main.dart`                 |

> Trong buổi này, chúng ta sẽ dùng **cả hai**: Dart CLI để học ngôn ngữ Dart, và Flutter app để tạo UI đầu tiên.

---

## 4. Dart Fundamentals 🔴

### 4.1 Variables (Biến)

Trong Dart, bạn khai báo biến với các keyword khác nhau tùy mục đích:

#### `var` — Biến có thể thay đổi giá trị

```dart
var name = 'Flutter'; // Dart tự suy ra type là String
name = 'Dart';        // ✅ OK — giá trị thay đổi được
// name = 42;          // ❌ LỖI — không thể đổi type (String → int)
```

`var` để Dart **tự suy luận kiểu dữ liệu** (type inference). Một khi đã gán giá trị, kiểu dữ liệu bị cố định.

#### `final` — Gán một lần, không đổi được

```dart
final city = 'Hà Nội';
// city = 'HCM';  // ❌ LỖI — final chỉ gán một lần

final now = DateTime.now(); // ✅ Giá trị được xác định lúc RUNTIME
```

Dùng `final` khi bạn biết biến **chỉ cần gán một lần** nhưng giá trị có thể được tính lúc chương trình chạy.

#### `const` — Hằng số compile-time

```dart
const pi = 3.14159;
// pi = 3.0;  // ❌ LỖI — const không thể thay đổi

// const now = DateTime.now(); // ❌ LỖI — DateTime.now() chỉ biết lúc runtime
```

Dùng `const` khi giá trị **đã biết trước** lúc compile (ví dụ: số pi, tên app, URL cố định).

#### `late` — Khai báo trước, gán sau

```dart
late String description;

// ... code khác ...

description = 'Đây là biến late'; // Gán sau khi khai báo
print(description); // ✅ OK

// Nếu print TRƯỚC khi gán → LỖI runtime: LateInitializationError
```

Dùng `late` khi bạn **chắc chắn** sẽ gán giá trị trước khi sử dụng, nhưng chưa thể gán ngay lúc khai báo.

#### Bảng tổng hợp

| Keyword | Thay đổi giá trị? | Khi nào xác định giá trị? | Dùng khi                          |
|---------|--------------------|---------------------------|-----------------------------------|
| `var`   | ✅ Có              | Runtime                   | Biến thông thường, cần thay đổi   |
| `final` | ❌ Không           | Runtime                   | Gán 1 lần, giá trị tính lúc chạy  |
| `const` | ❌ Không           | Compile-time              | Hằng số đã biết trước             |
| `late`  | ✅ Có              | Runtime (gán sau)         | Khởi tạo trễ, chắc chắn sẽ gán   |

### 4.2 Types (Kiểu dữ liệu)

Dart là ngôn ngữ **strongly typed** — mỗi biến có một kiểu dữ liệu rõ ràng.

#### Kiểu cơ bản

```dart
// Số nguyên
int age = 25;
int year = 2026;

// Số thực (có phần thập phân)
double height = 1.75;
double pi = 3.14159;

// Chuỗi ký tự
String name = 'Nguyễn Văn A';
String greeting = "Xin chào"; // Dùng ' hoặc " đều được

// Boolean (đúng/sai)
bool isStudent = true;
bool isWorking = false;
```

#### Collections (Tập hợp)

```dart
// List — danh sách có thứ tự (giống Array trong JS)
List<String> fruits = ['Táo', 'Cam', 'Xoài'];
print(fruits[0]);      // Táo
print(fruits.length);  // 3

// Map — cặp key-value (giống Object/Map trong JS)
Map<String, int> scores = {
  'Toán': 9,
  'Lý': 8,
  'Hóa': 7,
};
print(scores['Toán']); // 9

// Set — tập hợp không trùng lặp (giống Set trong JS)
Set<int> uniqueNumbers = {1, 2, 3, 2, 1};
print(uniqueNumbers); // {1, 2, 3} — tự loại bỏ trùng
```

#### `dynamic` — Kiểu bất kỳ

```dart
dynamic anything = 'Hello';
anything = 42;    // ✅ OK — dynamic cho phép đổi type
anything = true;  // ✅ OK

// ⚠️ NGUY HIỂM: Dart không kiểm tra type lúc compile
// → Dễ gây lỗi runtime. HẠN CHẾ dùng dynamic!
```

#### Bảng tổng hợp kiểu dữ liệu

| Kiểu      | Ví dụ                      | Mô tả                        |
|-----------|-----------------------------|-------------------------------|
| `int`     | `42`, `-7`, `0`             | Số nguyên                     |
| `double`  | `3.14`, `-0.5`              | Số thực                       |
| `String`  | `'Hello'`, `"World"`        | Chuỗi ký tự                  |
| `bool`    | `true`, `false`             | Giá trị logic                 |
| `List<T>` | `[1, 2, 3]`                | Danh sách có thứ tự          |
| `Map<K,V>`| `{'key': 'value'}`          | Cặp key-value                 |
| `Set<T>`  | `{1, 2, 3}`                | Tập hợp không trùng          |
| `dynamic` | bất kỳ giá trị nào          | Kiểu động (hạn chế dùng)     |

> 🔗 **FE Bridge:** `var` ≈ `let`, `final` ≈ `const` (runtime constant), `const` ≈ **compile-time constant** (không có equivalent trong JS/TS). Chú ý: `const` trong Dart ≠ `const` trong JavaScript! Dart `const` = giá trị xác định lúc compile, JS `const` = chỉ không reassign.

### 4.3 Null Safety

Null safety là tính năng **bắt buộc** trong Dart (từ phiên bản 2.12+). Mục đích: **ngăn lỗi null tại compile time** thay vì để crash lúc runtime.

#### Mặc định: biến KHÔNG thể null

```dart
String name = 'Flutter';
// name = null;  // ❌ LỖI COMPILE — String không chấp nhận null
```

#### `?` — Cho phép null

```dart
String? nickname;        // nickname có thể là String hoặc null
print(nickname);         // null (giá trị mặc định)

nickname = 'Dev';
print(nickname);         // Dev

nickname = null;         // ✅ OK — vì đã khai báo String?
```

#### `!` — Khẳng định không null (bang operator)

```dart
String? maybeName = 'Flutter';

// Dart yêu cầu kiểm tra null trước khi dùng String? như String
String definitelyName = maybeName!; // "Tôi chắc chắn nó không null"

// ⚠️ NẾU maybeName là null → crash với _CastError
// → Chỉ dùng ! khi THỰC SỰ chắc chắn
```

#### `??` — Giá trị mặc định nếu null

```dart
String? input;
String result = input ?? 'Mặc định'; // Nếu input null → dùng 'Mặc định'
print(result); // Mặc định
```

#### `late` với null safety

```dart
late String description; // Non-nullable nhưng chưa gán ngay

void initialize() {
  description = 'Đã khởi tạo';
}

// Gọi initialize() trước khi dùng description
// Nếu quên gọi → LateInitializationError lúc runtime
```

#### Bảng tổng hợp Null Safety

| Cú pháp | Ý nghĩa                                | Ví dụ                            |
|---------|-----------------------------------------|----------------------------------|
| `T`     | Không bao giờ null                      | `String name = 'A';`            |
| `T?`    | Có thể null                             | `String? name;`                  |
| `!`     | Khẳng định không null (nguy hiểm)       | `name!.length`                   |
| `??`    | Giá trị thay thế nếu null              | `name ?? 'default'`              |
| `late`  | Sẽ gán sau, Dart tin tưởng bạn          | `late String name;`              |

---

> 💼 **Gặp trong dự án:** Parse API response (field null khi schema thay đổi), nhận params từ route (null nếu navigate sai), async data chưa load xong
> 🤖 **Keywords bắt buộc trong prompt:** `sound null safety`, `null-aware operators (?. ?? ??=)`, không dùng `!` operator

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **API response parsing:** `response.data['user']` có thể null khi server trả về khác schema
- **Navigation arguments:** params nhận từ route có thể null nếu user navigate sai flow
- **Async data:** widget render trước khi future complete → data chưa có → null

**Tại sao cần các keyword trên:**
- **`sound null safety`** — để AI không dùng `dynamic` để né null check (dynamic phá vỡ type system)
- **`null-aware operators`** — AI phải dùng `?.`, `??`, `??=` thay vì `if (x != null)` verbose
- **Không dùng `!`** — nếu không nói rõ, AI hay dùng `json['field']!` gây runtime crash

**Prompt mẫu — Parse API response an toàn:**
```text
Tôi cần parse JSON response từ API vào Dart class. Flutter 3.x, Dart 3.x, sound null safety.
Constraints:
- Không dùng dynamic, không dùng ! operator trừ khi thực sự chắc chắn.
- Dùng null-aware operators (?., ??, ??=) đúng chỗ.
- fromJson factory phải xử lý field bị null hoặc sai kiểu.
Input JSON: {"id": 1, "name": "Nam", "avatar": null, "score": "42"}
Output: Dart class User với fromJson an toàn. Giải thích ngắn lý do từng xử lý null.
```

**Expected Output:** AI sẽ gen class `User` với `factory User.fromJson(Map<String, dynamic> json)`, dùng `??` cho `avatar` null, parse `score` từ String sang int an toàn bằng `int.tryParse`.

⚠️ **Giới hạn AI hay mắc:** AI đôi khi vẫn dùng `json['field']!` hoặc `as String` không an toàn — check kỹ mọi dòng có ký tự `!` và từ khóa `as` trong output.

→ Prompt nâng cao: [ai-toolkit/ai-driven-development.md](../../ai-toolkit/ai-driven-development.md)

</details>

### 4.4 Functions (Hàm)

#### Hàm cơ bản

```dart
// Khai báo hàm với kiểu trả về
int add(int a, int b) {
  return a + b;
}

// Hàm không trả về gì
void sayHello(String name) {
  print('Xin chào $name!');
}

// Gọi hàm
int result = add(3, 5);    // result = 8
sayHello('Flutter');         // In: Xin chào Flutter!
```

#### Positional Parameters (Tham số theo vị trí)

```dart
// Tham số bắt buộc, truyền theo thứ tự
String fullName(String first, String last) {
  return '$first $last';
}

print(fullName('Nguyễn', 'Văn A')); // Nguyễn Văn A
// Phải truyền đúng 2 tham số, đúng thứ tự
```

#### Named Parameters (Tham số đặt tên)

```dart
// Dùng {} để tạo named parameters
// {required} bắt buộc phải truyền
void createUser({
  required String name,
  required int age,
  String role = 'member', // Có giá trị mặc định → không bắt buộc
}) {
  print('$name, $age tuổi, vai trò: $role');
}

// Gọi hàm — truyền theo TÊN, không cần đúng thứ tự
createUser(age: 25, name: 'An');             // An, 25 tuổi, vai trò: member
createUser(name: 'Bình', age: 30, role: 'admin'); // Bình, 30 tuổi, vai trò: admin
```

> **Tại sao named parameters quan trọng?** Trong Flutter, hầu hết widget dùng named parameters. Ví dụ: `Text('Hello', style: TextStyle(fontSize: 20))`. Đọc code rõ ràng hơn nhiều so với positional.

#### Optional Positional Parameters

```dart
// Dùng [] để tạo optional positional parameters
String greet(String name, [String? title]) {
  if (title != null) {
    return 'Xin chào $title $name!';
  }
  return 'Xin chào $name!';
}

print(greet('An'));            // Xin chào An!
print(greet('An', 'Anh'));     // Xin chào Anh An!
```

#### Arrow Syntax (Cú pháp mũi tên)

```dart
// Khi hàm chỉ có MỘT biểu thức → dùng =>
int multiply(int a, int b) => a * b;
void logMessage(String msg) => print('[LOG] $msg');

// Tương đương với:
// int multiply(int a, int b) { return a * b; }
```

---

> 💼 **Gặp trong dự án:** Tạo widget Flutter (mọi widget đều dùng named params), viết utility functions cho business logic, xử lý callback
> 🤖 **Keywords bắt buộc trong prompt:** `named parameters`, `required keyword`, `arrow syntax =>`, `typedef` cho callback

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Custom widget:** Tạo widget `ProfileCard` nhận `name`, `avatar`, `onTap` callback — tất cả dùng named params
- **Utility functions:** Viết hàm format tiền tệ, validate email, parse date — dùng arrow syntax cho hàm ngắn
- **Callback handling:** Truyền `VoidCallback`, `ValueChanged<T>` giữa các widget

**Tại sao cần các keyword trên:**
- **`named parameters`** — Flutter convention bắt buộc, nếu thiếu AI sẽ gen positional params (khó đọc)
- **`required keyword`** — đảm bảo không quên truyền params quan trọng
- **`arrow syntax =>`** — convention cho hàm 1 dòng, AI cần biết để gen code idiomatic
- **`typedef`** — cho callback type rõ ràng thay vì `Function` chung chung

**Prompt mẫu — Tạo utility functions:**
```text
Tôi cần viết 3 utility functions cho Flutter app. Dart 3.x, null safety.
1. formatCurrency(double amount, {String symbol = '₫'}) → String — format số tiền có dấu phẩy ngàn
2. validateEmail(String? email) → bool — kiểm tra email hợp lệ bằng RegExp
3. parseDate(String? dateStr, {DateTime? fallback}) → DateTime — parse ISO 8601, có fallback
Constraints:
- Dùng named parameters cho optional params.
- Dùng arrow syntax (=>) khi hàm chỉ có 1 expression.
- Null safety: xử lý input null an toàn, không dùng !.
```

**Expected Output:** 3 functions với named params đúng convention, arrow syntax cho hàm ngắn, null safety đầy đủ.

⚠️ **Giới hạn AI hay mắc:** AI hay quên `required` keyword cho params bắt buộc, hoặc dùng `Function` thay vì typedef cụ thể cho callbacks.

→ Prompt nâng cao: [ai-toolkit/ai-driven-development.md](../../ai-toolkit/ai-driven-development.md)

</details>

### 4.5 Strings (Chuỗi)

#### String Interpolation

```dart
String name = 'Flutter';
int version = 3;

// Dùng $ với biến đơn giản
print('Xin chào $name!');           // Xin chào Flutter!

// Dùng ${} với biểu thức
print('Version: ${version + 1}');    // Version: 4
print('Chữ hoa: ${name.toUpperCase()}'); // Chữ hoa: FLUTTER

// Multi-line string dùng '''
String bio = '''
Tên: $name
Version: $version
Released: ${2026 - version} năm trước
''';
```

### 4.6 Control Flow (Luồng điều khiển)

#### if / else

```dart
int score = 85;

if (score >= 90) {
  print('Xuất sắc');
} else if (score >= 70) {
  print('Khá'); // ← In ra dòng này
} else {
  print('Cần cố gắng');
}
```

#### for loop

```dart
// for truyền thống
for (int i = 0; i < 5; i++) {
  print('Lần $i');
}

// for-in (duyệt qua collection)
List<String> languages = ['Dart', 'Kotlin', 'Swift'];
for (String lang in languages) {
  print(lang);
}
```

#### while loop

```dart
int count = 0;
while (count < 3) {
  print('Count: $count');
  count++;
}
// Count: 0
// Count: 1
// Count: 2
```

#### switch

```dart
String command = 'start';

switch (command) {
  case 'start':
    print('Bắt đầu');
    break;
  case 'stop':
    print('Dừng lại');
    break;
  default:
    print('Không rõ lệnh');
}
```

---

## 5. Chương trình Dart đầu tiên 🟢

### `void main()` — Điểm bắt đầu

Mọi chương trình Dart đều bắt đầu từ hàm `main()`. Đây là **entry point** — nơi Dart bắt đầu thực thi code.

```dart
void main() {
  print('Xin chào, Dart!');
}
```

Giải thích:
- `void` — hàm `main` không trả về giá trị nào
- `main()` — tên hàm đặc biệt, Dart luôn tìm hàm này để bắt đầu
- `print()` — in text ra terminal (giống `console.log` trong JS)

### Tạo và chạy project Dart

```bash
# Bước 1: Tạo project Dart mới
dart create hello_dart

# Bước 2: Vào thư mục project
cd hello_dart

# Bước 3: Xem cấu trúc
# hello_dart/
# ├── bin/
# │   └── hello_dart.dart    ← File chính, chứa main()
# ├── lib/
# │   └── hello_dart.dart    ← Library code
# ├── test/
# │   └── hello_dart_test.dart
# ├── pubspec.yaml            ← File cấu hình project
# ├── analysis_options.yaml   ← Quy tắc lint
# └── README.md

# Bước 4: Chạy
dart run
```

---

## 6. Chương trình Flutter đầu tiên 🟡

### Tạo project Flutter

```bash
# Bước 1: Tạo project
flutter create hello_flutter

# Bước 2: Vào thư mục
cd hello_flutter
```

### Cấu trúc project Flutter

```
hello_flutter/
├── android/          ← Code native Android (không cần sửa)
├── ios/              ← Code native iOS (không cần sửa)
├── web/              ← Code cho web platform
├── lib/              ← ⭐ CODE CỦA BẠN Ở ĐÂY
│   └── main.dart     ← Entry point — bắt đầu từ đây
├── test/             ← Unit tests
├── pubspec.yaml      ← ⭐ File cấu hình & dependencies
├── pubspec.lock      ← Lock file (tự sinh, không sửa)
├── analysis_options.yaml  ← Quy tắc lint
└── README.md
```

**Quan trọng:** 95% thời gian bạn chỉ làm việc trong thư mục `lib/` và file `pubspec.yaml`.

### Code Flutter đầu tiên

Mở `lib/main.dart`, xóa hết nội dung mặc định và thay bằng:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Hello Flutter'),
        ),
        body: const Center(
          child: Text(
            'Xin chào, Flutter!',
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```

Giải thích từng phần:

| Code                            | Ý nghĩa                                                |
|---------------------------------|---------------------------------------------------------|
| `import 'package:flutter/...'`  | Import thư viện Flutter Material Design                 |
| `void main()`                   | Entry point — Dart bắt đầu ở đây                       |
| `runApp()`                      | Hàm đặc biệt của Flutter — gắn widget vào màn hình     |
| `StatelessWidget`               | Widget không có state thay đổi (bất biến)               |
| `build()`                       | Method trả về **cây widget** — mô tả UI                 |
| `MaterialApp`                   | Widget gốc, cung cấp Material Design theme              |
| `Scaffold`                      | Cung cấp layout cơ bản: AppBar, Body, FloatingButton... |
| `AppBar`                        | Thanh bar phía trên màn hình                            |
| `Center`                        | Widget canh giữa con của nó                             |
| `Text`                          | Widget hiển thị text                                    |
| `const`                         | Đánh dấu widget bất biến → Flutter tối ưu performance  |

### Chạy Flutter app

```bash
# Cách 1: Terminal
flutter run

# Cách 2: VS Code
# Mở lib/main.dart → F5 (hoặc Run → Start Debugging)
# Hoặc click nút ▶ trên toolbar
```

### Hot Reload

Khi app đang chạy:
1. Thay đổi code trong `lib/main.dart` (ví dụ: đổi text, đổi màu)
2. Nhấn `r` trong terminal (hoặc Cmd+S trong VS Code)
3. App cập nhật **ngay lập tức** (~1 giây), **không mất state**

```
Hot Reload:   Nhấn 'r' → cập nhật UI, giữ state
Hot Restart:  Nhấn 'R' → restart toàn bộ app, mất state
Full Restart: Dừng app, flutter run lại
```

> 🔗 **FE Bridge:** Hot Reload ≈ HMR (Hot Module Replacement) — nhưng **khác ở**: Flutter Hot Reload **giữ state** đáng tin cậy hơn React HMR. Hot Restart = full restart (mất state) ≈ page refresh. FE dev sẽ thấy Hot Reload là upgrade so với HMR.

---

## 7. pubspec.yaml 🟡

### Nó là gì?

`pubspec.yaml` là file cấu hình **bắt buộc** của mọi project Dart/Flutter. Nó khai báo:
- Tên, phiên bản project
- Dart/Flutter SDK version
- Dependencies (thư viện bên ngoài)
- Assets (hình ảnh, fonts, ...)

### Cấu trúc cơ bản

```yaml
# Tên project — phải viết thường, dùng underscore
name: hello_flutter

# Mô tả ngắn
description: "My first Flutter app"

# Phiên bản app
version: 1.0.0+1

# Yêu cầu phiên bản Dart/Flutter
environment:
  sdk: ^3.5.0   # Dart SDK >= 3.5.0

# Dependencies — thư viện app CẦN để chạy
dependencies:
  flutter:
    sdk: flutter
  # Ví dụ thêm package:
  # http: ^1.2.0

# Dev dependencies — chỉ dùng khi develop (test, lint, ...)
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

# Assets (hình ảnh, fonts) — sẽ học ở buổi sau
# flutter:
#   assets:
#     - assets/images/
```

### Thêm dependencies

```bash
# Cách 1: Sửa pubspec.yaml → chạy lệnh
flutter pub get

# Cách 2: Dùng lệnh (tự thêm vào pubspec.yaml)
flutter pub add http
```

Sau khi chạy `flutter pub get`, Dart sẽ:
1. Download packages từ pub.dev  
2. Tạo/cập nhật file `pubspec.lock` (lock phiên bản cụ thể)
3. Tạo thư mục `.dart_tool/` (cache)

### So sánh nhanh với package.json

| Đặc điểm      | `pubspec.yaml` (Dart/Flutter) | `package.json` (Node.js)       |
|----------------|-------------------------------|--------------------------------|
| Định dạng      | YAML                          | JSON                           |
| Tên project    | `name:`                       | `"name":`                      |
| Dependencies   | `dependencies:`               | `"dependencies":`              |
| Dev deps       | `dev_dependencies:`           | `"devDependencies":`           |
| Install        | `flutter pub get`             | `npm install`                  |
| Add package    | `flutter pub add http`        | `npm install axios`            |
| Lock file      | `pubspec.lock`                | `package-lock.json`            |
| Registry       | pub.dev                       | npmjs.com                      |

> 🔗 **FE Bridge:** `pubspec.yaml` ≈ `package.json` — quản lý dependencies, metadata, scripts. `flutter pub get` ≈ `npm install`. `pub.dev` ≈ `npmjs.com`. Cấu trúc tương đồng, chỉ khác format (YAML vs JSON).

---

## 8. Best Practices & Lỗi thường gặp 🟡

### ✅ 5 Best Practices cho người mới

| #  | Practice                         | Giải thích                                          |
|----|----------------------------------|------------------------------------------------------|
| 1  | Dùng `final` thay `var` khi có thể | Nếu biến không cần thay đổi → `final`. Ít bug hơn. |
| 2  | Luôn khai báo type rõ ràng       | `int count = 0;` thay vì `var count = 0;` khi type không rõ ràng |
| 3  | Tránh `dynamic`                  | `dynamic` tắt type checking → dễ crash runtime       |
| 4  | Dùng `const` cho widget bất biến | `const Text('Hello')` → Flutter skip rebuild → nhanh hơn |
| 5  | Chạy `flutter doctor` thường xuyên | Đảm bảo môi trường luôn sẵn sàng                   |

### ❌ 5 Lỗi thường gặp

| #  | Lỗi                                      | Nguyên nhân & Cách fix                              |
|----|-------------------------------------------|------------------------------------------------------|
| 1  | `Null check operator used on a null value`| Dùng `!` trên biến null → kiểm tra null trước khi dùng `!` |
| 2  | `LateInitializationError`                 | Dùng biến `late` trước khi gán → đảm bảo gán trước khi dùng |
| 3  | `type 'Null' is not a subtype of type 'String'` | Truyền null vào nơi yêu cầu non-null → dùng `??` hoặc kiểm tra |
| 4  | `flutter run` không tìm thấy device      | Chưa mở emulator/simulator → mở trước khi chạy     |
| 5  | Hot reload không hoạt động                | Thay đổi ở `main()` hoặc thêm field mới → cần Hot Restart (`R`) |

---

## 9. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

> **Phần này dành cho bạn đã từng làm React/Vue.** Nếu chưa biết React/Vue, bạn có thể bỏ qua — không ảnh hưởng đến việc học.

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Dart/Flutter Mindset | Tại sao khác |
|---|-------------------|----------------------|--------------|
| 1 | HTML + CSS + JS tách biệt | Mọi thứ là Dart code — UI, style, logic gộp trong Widget | Flutter không có separation of concerns kiểu web |
| 2 | TypeScript = compile-time types only | Dart = **sound type system** — type safety cả runtime | Dart compiled, TS transpiled to untyped JS |
| 3 | `const` = cannot reassign variable | `const` = **compile-time constant** (giá trị biết trước lúc build) | Hoàn toàn khác ngữ nghĩa JS const |
| 4 | npm/yarn + package.json | pub + pubspec.yaml — tương đồng workflow | Cùng concept, khác tool + format |
| 5 | Browser DevTools debug | Flutter DevTools + native debugger | Tooling khác hoàn toàn, workflow tương tự |

### Flutter vs React/Vue — So sánh tổng quan

| Đặc điểm            | Flutter                     | React (Native)             | Vue                        |
|----------------------|-----------------------------|----------------------------|----------------------------|
| Ngôn ngữ            | Dart                        | JavaScript/TypeScript       | JavaScript/TypeScript       |
| UI rendering         | Tự vẽ pixel (Skia/Impeller) | Native components (bridge) | Virtual DOM → Real DOM      |
| Styling             | Widget properties            | StyleSheet / CSS-in-JS     | CSS / Scoped CSS            |
| State management    | setState, Provider, Riverpod| useState, Redux, Zustand   | ref, reactive, Pinia        |
| Hot reload          | ✅ Stateful Hot Reload       | ✅ Fast Refresh              | ✅ HMR                      |
| Layout              | Widget tree (Row, Column)    | Flexbox                    | Flexbox / CSS Grid          |
| Entry point          | `void main()` + `runApp()`  | `createRoot().render()`    | `createApp().mount()`       |

### Dart vs TypeScript

| Đặc điểm            | Dart                        | TypeScript                  |
|----------------------|-----------------------------|------------------------------|
| Null safety          | Sound (100% bắt lúc compile)| Strict mode (vẫn có escape) |
| Compilation          | AOT + JIT                   | Transpile → JavaScript       |
| Type system          | Bắt buộc (sound)            | Tùy chọn (gradual)          |
| OOP                  | Class-based, single inherit | Class-based, interface       |
| Async                | Future + Stream              | Promise + Observable (RxJS)  |
| Enum                 | Enhanced enum (có method)    | Enum cơ bản                  |

### pubspec.yaml vs package.json

| Thao tác             | Flutter                      | Node.js                      |
|----------------------|------------------------------|-------------------------------|
| Khai báo deps        | `pubspec.yaml`               | `package.json`                |
| Install deps         | `flutter pub get`            | `npm install`                 |
| Add package          | `flutter pub add <pkg>`      | `npm install <pkg>`           |
| Lock file            | `pubspec.lock`               | `package-lock.json`           |
| Run scripts          | Không có field scripts       | `"scripts": { "start": ... }`|
| Registry             | pub.dev                      | npmjs.com                     |

### Hot Reload vs HMR

| Đặc điểm            | Flutter Hot Reload           | React Fast Refresh / Vue HMR |
|----------------------|------------------------------|-------------------------------|
| Tốc độ              | ~1 giây                      | ~1-3 giây                     |
| Giữ state           | ✅ (StatefulWidget state)     | ✅ (component-level state)     |
| Khi nào cần restart | Thay đổi main(), thêm field  | Thay đổi module boundary      |
| Cơ chế              | JIT recompile + inject       | Module replace + reconcile    |

---

## 10. Tổng kết

### ✅ Checklist tự kiểm tra

Sau buổi học, hãy tự trả lời các câu hỏi sau. Nếu trả lời được hết → bạn đã nắm vững buổi 1:

| #  | Câu hỏi                                                          | ✅ |
|----|-------------------------------------------------------------------|----|
| 1  | Flutter là gì? Tại sao Google tạo Flutter?                       | ☐  |
| 2  | Kiến trúc Flutter có mấy lớp? Kể tên                             | ☐  |
| 3  | Dart là gì? Tại sao Flutter dùng Dart thay vì JavaScript?        | ☐  |
| 4  | AOT và JIT compilation khác nhau thế nào?                         | ☐  |
| 5  | `flutter doctor` dùng để làm gì?                                 | ☐  |
| 6  | Phân biệt `var`, `final`, `const`, `late`                        | ☐  |
| 7  | Null safety là gì? Giải thích `?`, `!`, `??`                     | ☐  |
| 8  | Named parameters khác positional parameters thế nào?              | ☐  |
| 9  | `pubspec.yaml` dùng để làm gì?                                   | ☐  |
| 10 | Hot reload khác hot restart thế nào?                              | ☐  |

### Bước tiếp theo

1. Hoàn thành 3 bài tập trong [03-thuc-hanh.md](./03-thuc-hanh.md)
2. Đọc thêm tài liệu tham khảo trong [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md)
3. Chuẩn bị cho **Buổi 02: Dart nâng cao** — OOP, Async, Collections

---

> **Tiếp theo:** [02-vi-du.md](./02-vi-du.md) — 5 ví dụ hoàn chỉnh, chạy được

---

### ➡️ Buổi tiếp theo

> **Buổi 02: Dart Nâng Cao** — OOP, Async/Await, Collections nâng cao, và Pattern Matching trong Dart 3.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Tự viết thử 1 class đơn giản với constructor
