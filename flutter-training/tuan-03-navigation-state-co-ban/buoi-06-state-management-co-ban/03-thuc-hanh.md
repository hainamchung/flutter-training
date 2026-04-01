# Buổi 06: State Management Cơ Bản — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Hướng dẫn chung

- Tạo Flutter project mới cho mỗi bài hoặc dùng chung 1 project với nhiều screen
- Thêm `provider: ^6.1.2` vào `pubspec.yaml` cho BT1 và BT2
- Chạy `flutter pub get` sau khi thêm dependency
- Đọc kỹ yêu cầu và gợi ý trước khi code

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ React State**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| React/Vue Habit | Flutter Reality | Bài tập liên quan |
|-----------------|-----------------|---------------------|
| `useState` chỉ re-render component dùng state | `setState` rebuild TOÀN BỘ subtree → cần tách widget nhỏ | BT1, BT2 |
| State lift-up + prop drilling | Dùng `Provider` — prop drilling KHÔNG phải pattern Flutter | BT2 |
| `useEffect` cleanup return | `dispose()` trong State class — phải handle controller lifecycle | BT1, BT3 |
| `useContext` để subscribe state | `context.watch()` cho UI, `context.read()` cho event handler — dùng sai → bug | BT2 |
| Quên `notifyListeners()` — React tự trigger | ChangeNotifier PHẢI gọi `notifyListeners()` thủ công → quên = UI không update | BT2 |

---

## BT1 ⭐ Theme Switcher với Provider 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_theme_switcher` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app chuyển đổi Dark/Light theme |

### Yêu cầu

Tạo app cho phép chuyển đổi Dark/Light theme, state được quản lý bằng Provider.

### Chức năng cần có

1. **Toggle dark/light mode** bằng Switch hoặc IconButton trên AppBar
2. **Theme áp dụng cho toàn bộ app** (không chỉ 1 screen)
3. **Hiển thị trạng thái hiện tại**: "Dark Mode" hoặc "Light Mode"
4. **Demo screen** với một vài widget (Card, ListTile, Button) để thấy theme thay đổi
5. **NavigatE sang screen khác** → theme vẫn nhất quán (chứng minh state là global)

### Mockup

```
┌──────────────────────────────┐
│ Theme Switcher    [🌙/☀️]   │ ← AppBar với toggle icon
├──────────────────────────────┤
│                              │
│  Chế độ hiện tại: Dark Mode  │
│                              │
│  ┌──────────────────────┐    │
│  │  Card demo            │    │
│  │  Nội dung mẫu         │    │
│  └──────────────────────┘    │
│                              │
│  ● ListTile 1               │
│  ● ListTile 2               │
│  ● ListTile 3               │
│                              │
│  [  Go to Settings  ]       │ ← Navigate sang screen khác
│                              │
└──────────────────────────────┘
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

```
Bước 1: Tạo ThemeModel extends ChangeNotifier
         - Field: bool _isDarkMode = false
         - Getter: isDarkMode, themeData (trả về ThemeData)
         - Method: toggleTheme()

Bước 2: Wrap MaterialApp bằng ChangeNotifierProvider<ThemeModel>

Bước 3: Trong MaterialApp, dùng context.watch<ThemeModel>().themeData
         cho property theme:

Bước 4: Tạo HomeScreen với toggle button và demo widgets

Bước 5: Tạo SettingsScreen — navigate từ Home, theme vẫn đúng
```

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// theme_model.dart
class ThemeModel extends ChangeNotifier {
  bool _isDarkMode = false;

  bool get isDarkMode => _isDarkMode;

  ThemeData get themeData => _isDarkMode
      ? ThemeData.dark(useMaterial3: true)
      : ThemeData.light(useMaterial3: true);

  void toggleTheme() {
    // TODO: implement
  }
}

// main.dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => ThemeModel(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // TODO: dùng context.watch để lấy themeData
    return MaterialApp(
      theme: /* ??? */,
      home: const HomeScreen(),
    );
  }
}
```

</details>

### Tiêu chí hoàn thành

- [ ] `ThemeModel` extends `ChangeNotifier` đúng cách
- [ ] Toggle theme hoạt động real-time
- [ ] Theme áp dụng cho toàn bộ app (cả screen khác)
- [ ] Dùng `context.watch()` ở nơi cần reactive, `context.read()` ở callback
- [ ] Code sạch, tách model riêng file

---

## BT2 ⭐⭐ Shopping Cart với Provider 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_shopping_cart` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app shopping cart với danh sách sản phẩm và giỏ hàng |

### Yêu cầu

Tạo app shopping đơn giản: danh sách sản phẩm, thêm/bớt vào giỏ hàng, xem giỏ hàng với tổng tiền.

### Chức năng cần có

1. **Product List Screen**: Hiển thị danh sách sản phẩm (ít nhất 6 items)
2. **Thêm/bớt sản phẩm** từ product list (nút +/-)
3. **Cart badge** trên AppBar hiển thị số lượng items
4. **Cart Screen**: Hiển thị items trong giỏ, số lượng, đơn giá, thành tiền
5. **Tổng tiền** ở cuối Cart Screen
6. **Xóa item** khỏi giỏ hàng
7. **Xóa tất cả** (Clear cart)

### Mockup

```
// Screen 1: Product List
┌──────────────────────────────┐
│ Shop              🛒 [3]    │
├──────────────────────────────┤
│ ☕ Cà phê          35,000đ  │
│              [-] 2 [+]      │
│─────────────────────────────│
│ 🥖 Bánh mì         25,000đ  │
│                    [+]      │
│─────────────────────────────│
│ 🍜 Phở             55,000đ  │
│              [-] 1 [+]      │
│─────────────────────────────│
│ ...                          │
└──────────────────────────────┘

// Screen 2: Cart
┌──────────────────────────────┐
│ ← Giỏ hàng      [Xóa tất cả]│
├──────────────────────────────┤
│ ☕ Cà phê                     │
│    35,000đ × 2 = 70,000đ  🗑│
│─────────────────────────────│
│ 🍜 Phở                       │
│    55,000đ × 1 = 55,000đ  🗑│
│─────────────────────────────│
│                              │
│                              │
├──────────────────────────────┤
│ Tổng cộng:       125,000đ   │
│ [     Thanh toán            ]│
└──────────────────────────────┘
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

```
Bước 1: Tạo model classes
         - Product (id, name, price, emoji)
         - Danh sách sản phẩm mẫu (const list)

Bước 2: Tạo CartModel extends ChangeNotifier
         - Map<String, int> _items (productId → quantity)
         - Getters: items, totalItems, totalPrice
         - Methods: addItem, removeItem, removeProduct, clearCart

Bước 3: Setup Provider ở main.dart

Bước 4: Tạo ProductListScreen
         - ListView.builder với sampleProducts
         - Mỗi tile: tên, giá, emoji, nút +/-
         - AppBar có cart icon + badge (Consumer)

Bước 5: Tạo CartScreen
         - Hiển thị chi tiết items
         - Tính tổng tiền
         - Nút xóa từng item, xóa tất cả

Bước 6: Navigate giữa 2 screen
```

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

```dart
// cart_model.dart
class CartModel extends ChangeNotifier {
  final Map<String, int> _items = {};

  Map<String, int> get items => Map.unmodifiable(_items);
  int get totalItems => _items.values.fold(0, (sum, qty) => sum + qty);
  // TODO: totalPrice cần danh sách sản phẩm để tính

  void addItem(String productId) {
    // TODO: implement
  }

  void removeItem(String productId) {
    // TODO: implement
  }

  void removeProduct(String productId) {
    // TODO: implement
  }

  void clearCart() {
    // TODO: implement
  }
}
```

</details>

### Tiêu chí hoàn thành

- [ ] `CartModel` chứa toàn bộ logic (add, remove, calculate)
- [ ] Dùng `ChangeNotifierProvider` ở root
- [ ] Cart badge cập nhật real-time khi thêm/bớt
- [ ] `context.select()` dùng ở product tile (tối ưu rebuild)
- [ ] Cart screen hiển thị đúng items, quantity, tổng tiền
- [ ] Xóa item và xóa tất cả hoạt động
- [ ] Navigate giữa 2 screen — state nhất quán
- [ ] Không có logic tính toán trong widget (tất cả ở CartModel)

---

## BT3 ⭐⭐⭐ Registration Form với Multi-field Validation 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_registration_form` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — form đăng ký với multi-field validation |

### Yêu cầu

Tạo form đăng ký với nhiều fields và validation phức tạp.

### Chức năng cần có

1. **Form fields**: Họ tên, Email, Số điện thoại, Mật khẩu, Xác nhận mật khẩu
2. **Validation rules**:
   - Họ tên: không trống, ít nhất 2 từ
   - Email: format hợp lệ
   - Số điện thoại: format Việt Nam (bắt đầu bằng 0, 10 số)
   - Mật khẩu: ≥ 8 ký tự, có chữ hoa, chữ thường, số, ký tự đặc biệt
   - Xác nhận mật khẩu: khớp với mật khẩu
3. **Password strength indicator** (thanh tiến trình hoặc text thay đổi)
4. **Show/hide password** toggle
5. **Auto-validate** sau lần submit đầu tiên
6. **Submit**: validate toàn bộ form, hiển thị kết quả (SnackBar hoặc Dialog)
7. **Reset form** button

### Mockup

```
┌──────────────────────────────┐
│ ← Đăng ký tài khoản         │
├──────────────────────────────┤
│                              │
│  👤 Họ và tên                │
│  ┌──────────────────────┐    │
│  │ Nguyễn Văn A          │    │
│  └──────────────────────┘    │
│                              │
│  📧 Email                    │
│  ┌──────────────────────┐    │
│  │ email@example.com     │    │
│  └──────────────────────┘    │
│                              │
│  📱 Số điện thoại            │
│  ┌──────────────────────┐    │
│  │ 0901234567             │    │
│  └──────────────────────┘    │
│                              │
│  🔒 Mật khẩu                │
│  ┌──────────────────────┐    │
│  │ ••••••••           👁 │    │
│  └──────────────────────┘    │
│  Độ mạnh: ████████░░ Khá    │
│                              │
│  🔒 Xác nhận mật khẩu       │
│  ┌──────────────────────┐    │
│  │ ••••••••           👁 │    │
│  └──────────────────────┘    │
│                              │
│  [      Đăng ký            ] │
│  [      Reset              ] │
│                              │
└──────────────────────────────┘
```

<details>
<summary>💡 Gợi ý hướng tiếp cận</summary>

```
Bước 1: Tạo StatefulWidget với GlobalKey<FormState>
         Tạo TextEditingController cho mỗi field (5 controllers)

Bước 2: Viết validator functions:
         - validateName(String?)
         - validateEmail(String?)
         - validatePhone(String?)
         - validatePassword(String?)
         - validateConfirmPassword(String?)
         Chú ý: confirmPassword cần truy cập _passwordController.text

Bước 3: Build Form UI
         - Form wrapper với key
         - 5 TextFormField với decoration đẹp
         - Password toggle (obscureText)

Bước 4: Password strength indicator
         - Hàm tính strength dựa trên criteria: length, uppercase,
           lowercase, digit, special char
         - LinearProgressIndicator hoặc Row of colored boxes
         - Dùng TextEditingController.addListener để cập nhật

Bước 5: Submit handler
         - if (_formKey.currentState!.validate())
         - Hiển thị success dialog/snackbar

Bước 6: Reset handler
         - _formKey.currentState!.reset()
         - Clear tất cả controllers
         - Reset password strength

Bước 7: Dispose tất cả controllers
```

</details>

<details>
<summary>🔑 Gợi ý code structure</summary>

### Validation Rules Chi Tiết

```dart
// Họ tên: không trống, ít nhất 2 từ
String? validateName(String? value) {
  if (value == null || value.trim().isEmpty) return 'Vui lòng nhập họ tên';
  if (value.trim().split(' ').length < 2) return 'Vui lòng nhập đầy đủ họ tên';
  return null;
}

// Email
String? validateEmail(String? value) {
  if (value == null || value.trim().isEmpty) return 'Vui lòng nhập email';
  if (!RegExp(r'^[^@\s]+@[^@\s]+\.[^@\s]+$').hasMatch(value.trim())) {
    return 'Email không hợp lệ';
  }
  return null;
}

// Số điện thoại Việt Nam
String? validatePhone(String? value) {
  if (value == null || value.trim().isEmpty) return 'Vui lòng nhập số điện thoại';
  if (!RegExp(r'^0\d{9}$').hasMatch(value.trim())) {
    return 'Số điện thoại không hợp lệ (VD: 0901234567)';
  }
  return null;
}

// Mật khẩu: ≥ 8 ký tự, chữ hoa, chữ thường, số, ký tự đặc biệt
String? validatePassword(String? value) {
  if (value == null || value.isEmpty) return 'Vui lòng nhập mật khẩu';
  if (value.length < 8) return 'Mật khẩu phải có ít nhất 8 ký tự';
  if (!RegExp(r'[A-Z]').hasMatch(value)) return 'Cần có ít nhất 1 chữ hoa';
  if (!RegExp(r'[a-z]').hasMatch(value)) return 'Cần có ít nhất 1 chữ thường';
  if (!RegExp(r'[0-9]').hasMatch(value)) return 'Cần có ít nhất 1 số';
  if (!RegExp(r'[!@#\$%\^&\*]').hasMatch(value)) {
    return 'Cần có ít nhất 1 ký tự đặc biệt (!@#\$%^&*)';
  }
  return null;
}

// Xác nhận mật khẩu
String? validateConfirmPassword(String? value) {
  if (value == null || value.isEmpty) return 'Vui lòng xác nhận mật khẩu';
  if (value != _passwordController.text) return 'Mật khẩu không khớp';
  return null;
}
```

### Password Strength Indicator

```dart
// Tính điểm strength (0.0 → 1.0)
double _calculateStrength(String password) {
  double strength = 0;
  if (password.length >= 8) strength += 0.2;
  if (password.length >= 12) strength += 0.1;
  if (RegExp(r'[A-Z]').hasMatch(password)) strength += 0.2;
  if (RegExp(r'[a-z]').hasMatch(password)) strength += 0.1;
  if (RegExp(r'[0-9]').hasMatch(password)) strength += 0.2;
  if (RegExp(r'[!@#\$%\^&\*]').hasMatch(password)) strength += 0.2;
  return strength.clamp(0.0, 1.0);
}

String _strengthLabel(double strength) {
  if (strength < 0.3) return 'Yếu';
  if (strength < 0.6) return 'Trung bình';
  if (strength < 0.8) return 'Khá';
  return 'Mạnh';
}

Color _strengthColor(double strength) {
  if (strength < 0.3) return Colors.red;
  if (strength < 0.6) return Colors.orange;
  if (strength < 0.8) return Colors.blue;
  return Colors.green;
}
```

</details>

### Tiêu chí hoàn thành

- [ ] Form có đủ 5 fields với `InputDecoration` rõ ràng
- [ ] Mỗi field có validator riêng, hiển thị error message bên dưới
- [ ] Confirm password validate khớp với password
- [ ] Password strength indicator cập nhật real-time
- [ ] Show/hide password toggle hoạt động
- [ ] Auto-validate sau lần submit đầu (hoặc dùng `onUserInteraction`)
- [ ] Submit chỉ thành công khi tất cả fields hợp lệ
- [ ] Reset form và clear tất cả controllers
- [ ] Tất cả controllers được dispose đúng cách

---

## 💬 Câu hỏi thảo luận

### Câu hỏi 1: Khi nào setState() là đủ?

> Bạn đang xây app Todo List. Mỗi task có checkbox "done". Trong bao nhiêu trường hợp sau đây, `setState()` là đủ? Giải thích.
>
> a) Toggle checkbox của 1 task (UI cập nhật ngay trong list)
> b) Đếm số task đã hoàn thành và hiển thị ở AppBar
> c) Lưu danh sách task để khi navigate sang Settings screen rồi quay lại vẫn còn
> d) Hiển thị badge số task chưa hoàn thành ở BottomNavigationBar (khác tab)

**Gợi ý trả lời**:
- (a): setState đủ — chỉ widget chứa list cần biết
- (b): setState đủ — nếu AppBar và list cùng trong 1 StatefulWidget
- (c): setState đủ — nếu state nằm ở widget cha (không bị dispose khi navigate push)
- (d): setState KHÔNG đủ — BottomNavigationBar và list ở 2 nhánh khác nhau → cần Provider

### Câu hỏi 2: Provider vs InheritedWidget trực tiếp

> Team lead nói: "Mình không muốn dependency bên ngoài, dùng InheritedWidget thuần thôi". Bạn sẽ đưa ra những argument nào để convince dùng Provider?

**Gợi ý trả lời**:
- Provider giảm boilerplate đáng kể (không cần viết InheritedWidget class, of() method, StatefulWidget wrapper)
- Provider tự handle dispose (ChangeNotifierProvider tự dispose ChangeNotifier)
- Provider có `context.select()` — tối ưu rebuild
- Provider là Google-recommended, maintained tốt, community lớn
- Nhưng cũng tôn trọng: nếu chỉ cần 1-2 shared state đơn giản, InheritedWidget thuần có thể chấp nhận được

### Câu hỏi 3: Form Validation Patterns

> App của bạn có 5 screens, mỗi screen có form riêng. Nhiều fields lặp lại (email, phone). Bạn sẽ tổ chức code validation như thế nào để tránh duplicate?

**Gợi ý trả lời**:
- Tạo file `validators.dart` chứa tất cả validator functions dùng chung
- Mỗi validator là pure function: `String? validateEmail(String? value)`
- Có thể tạo class `Validators` với static methods
- Cho các validation logic phức tạp, có thể dùng validator composition:
  ```dart
  FormFieldValidator<String> compose(List<FormFieldValidator<String>> validators) {
    return (value) {
      for (final validator in validators) {
        final error = validator(value);
        if (error != null) return error;
      }
      return null;
    };
  }
  ```
- Tái sử dụng: `validator: compose([Validators.required, Validators.email])`

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 3:** Focus vào gen config phức tạp và review đúng pattern.

### AI-BT1: Gen InheritedWidget custom cho Theme Switching ⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** InheritedWidget, Provider, ChangeNotifier, watch vs read.
- **Task thực tế:** Tech lead yêu cầu "viết theme switching bằng raw InheritedWidget trước, rồi refactor sang Provider — để hiểu Provider đang wrap gì". Mục đích: trainee phải hiểu layer dưới trước khi dùng abstraction.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần implement theme switching 2 cách để so sánh.
Tech stack: Flutter 3.x.
Cách 1 — Raw InheritedWidget:
- ThemeConfig class: brightness (light/dark), primaryColor, backgroundColor, textColor.
- ThemeInheritedWidget extends InheritedWidget, có of(context) static method.
- StatefulWidget wrapper để thay đổi ThemeConfig.
- updateShouldNotify: chỉ notify khi brightness đổi.
- Demo screen: background color, text color theo theme + FAB toggle.

Cách 2 — Provider:
- Refactor cách 1 sang ChangeNotifierProvider.
- ThemeNotifier extends ChangeNotifier, toggleTheme() method.
- Demo screen dùng context.watch<ThemeNotifier>().

Output: 4 files (2 cho mỗi cách) + bảng so sánh (Lines of code, Boilerplate, testability, scalability).
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 4 files + comparison table.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | InheritedWidget có wrap trong StatefulWidget? (InheritedWidget immutable) | ☐ |
| 2 | `updateShouldNotify` có compare giá trị cụ thể (không return true cứng)? | ☐ |
| 3 | `of(context)` có throw error rõ ràng khi InheritedWidget không tìm thấy? | ☐ |
| 4 | Provider version dùng `context.watch` trong build (không phải read)? | ☐ |
| 5 | Toggle button dùng `context.read` (trong callback)? | ☐ |
| 6 | So sánh table accurate (Provider ít boilerplate hơn)? | ☐ |
| 7 | Cả 2 cách đều chạy được standalone (no missing imports)? | ☐ |

**4. Customize:**
Thêm persistence: lưu theme preference vào SharedPreferences → khi mở app lại, theme giữ nguyên. AI chưa handle phần này. Implement `loadSavedTheme()` trong `initState` và `saveTheme()` mỗi khi toggle.
