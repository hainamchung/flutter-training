# Buổi 06: State Management Cơ Bản — Lý Thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 6/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 05 (lý thuyết + ít nhất BT1-BT2)

## 1. State trong Flutter là gì? 🔴

### 1.1 Định nghĩa

**State** = dữ liệu có thể thay đổi theo thời gian và ảnh hưởng đến UI.

Khi state thay đổi → Flutter rebuild widget → UI cập nhật.

```
User Action ──▶ State thay đổi ──▶ Widget rebuild ──▶ UI cập nhật
```

### 1.2 Hai loại State

#### Ephemeral State (Local State)

- Chỉ thuộc về **một widget duy nhất**
- Không cần chia sẻ với widget khác
- Quản lý bằng `setState()` trong `StatefulWidget`

**Ví dụ**: animation progress, current tab index, text field input đang gõ, checkbox checked/unchecked.

```dart
// Ephemeral state — chỉ widget này cần biết
class _CounterState extends State<Counter> {
  int _count = 0; // ← ephemeral state

  @override
  Widget build(BuildContext context) {
    return Text('$_count');
  }
}
```

#### App State (Shared State)

- Cần **chia sẻ** giữa nhiều widget / screen
- Tồn tại xuyên suốt nhiều phần của app
- Cần giải pháp state management (Provider, BLoC, Riverpod...)

**Ví dụ**: user authentication, shopping cart, theme preference, notification count.

```
                    ┌─── ProductList (đọc cart count)
                    │
App State (Cart) ───┼─── CartScreen (đọc + sửa cart items)
                    │
                    └─── AppBar badge (đọc cart count)
```

### 1.3 Decision Tree: Chọn cái nào?

```
State này có cần chia sẻ giữa nhiều widget không?
│
├── KHÔNG → Chỉ 1 widget cần?
│           │
│           ├── CÓ → setState() là đủ ✅
│           │
│           └── Widget cha-con trực tiếp? → Truyền qua constructor ✅
│
└── CÓ → Nhiều widget ở các nhánh khác nhau cần?
          │
          ├── Data đơn giản, ít thay đổi → InheritedWidget
          │
          └── Data phức tạp, thay đổi thường xuyên → Provider / Riverpod / BLoC
```

> **Quy tắc ngón cái**: Nếu bạn phải truyền callback/data qua 3+ level widget → đã đến lúc dùng state management solution.

> 🔗 **FE Bridge:** Ephemeral State ≈ `useState` (local component state), App State ≈ Redux/Zustand store (global state) — mapping concept **gần 1:1**. Nhưng **khác ở**: Flutter phân chia rõ ràng hơn — ephemeral state KHÔNG BAO GIỜ nên dùng Provider, chỉ `setState`. FE dev thường đưa mọi thứ vào global store — Flutter khuyến khích local-first.

---

## 2. setState() — Review Deep Dive 🔴

### 2.1 Cách hoạt động bên trong

```dart
void setState(VoidCallback fn) {
  fn();                    // 1. Chạy callback → cập nhật biến
  _element.markNeedsBuild(); // 2. Đánh dấu element là "dirty"
  // 3. Framework schedule rebuild trong frame tiếp theo
  // 4. build() được gọi lại → tạo widget tree mới
  // 5. Element tree reconciliation — so sánh widget mới vs cũ tại mỗi vị trí,
  //    chỉ cập nhật RenderObject khi cần (KHÔNG phải VDOM diffing như React)
}
```

```
setState() được gọi
    │
    ▼
markNeedsBuild() ← đánh dấu Element dirty
    │
    ▼
Scheduler đặt lịch rebuild
    │
    ▼
build() chạy lại ← toàn bộ method build()
    │
    ▼
Element tree reconciliation & RenderObject update
(so sánh runtimeType + key tại mỗi vị trí, cập nhật tối thiểu)
```

### 2.2 Cách dùng đúng

```dart
// ✅ ĐÚNG — thay đổi state bên trong callback
setState(() {
  _count++;
  _label = 'Count: $_count';
});

// ✅ ĐÚNG — thay đổi trước rồi gọi setState
_count++;
_label = 'Count: $_count';
setState(() {});

// ❌ SAI — gọi setState trong build()
@override
Widget build(BuildContext context) {
  setState(() { _count++; }); // INFINITE LOOP!
  return Text('$_count');
}

// ❌ SAI — gọi setState sau dispose
void _fetchData() async {
  final data = await api.getData();
  setState(() { _data = data; }); // Widget có thể đã bị dispose!
}

// ✅ SỬA — check mounted trước khi setState
void _fetchData() async {
  final data = await api.getData();
  if (mounted) {
    setState(() { _data = data; });
  }
}
```

### 2.3 Giới hạn của setState()

| Giới hạn | Giải thích |
|-----------|------------|
| **Chỉ trong 1 widget** | Không thể notify widget khác khi state thay đổi |
| **Rebuild cả subtree** | `build()` chạy lại toàn bộ, mặc dù Flutter tối ưu bằng Element tree reconciliation |
| **Prop drilling** | Phải truyền data qua constructor từ cha → con → cháu → ... |
| **Khó test** | Logic lẫn trong UI, không tách riêng được |
| **Không scale** | App lớn → setState rải rác khắp nơi → khó maintain |

```
// Prop drilling hell — truyền callback qua 4 levels
App → HomePage → ProductSection → ProductList → ProductCard
        ↓              ↓               ↓             ↓
    onAddToCart     onAddToCart     onAddToCart    onAddToCart
```

### 2.4 Lifting State Up

Khi 2 widget cần chia sẻ cùng state, **nâng state lên widget cha chung gần nhất** (lifting state up). Cha giữ state và truyền xuống qua constructor.

> 💡 Pattern này giống hệt **"Lifting State Up"** trong React — [React docs](https://react.dev/learn/sharing-state-between-components).

```dart
// Parent quản lý state, 2 children đọc/ghi qua callback
class TemperatureConverter extends StatefulWidget {
  const TemperatureConverter({super.key});

  @override
  State<TemperatureConverter> createState() => _TemperatureConverterState();
}

class _TemperatureConverterState extends State<TemperatureConverter> {
  double _celsius = 0;

  void _updateTemperature(double value) {
    setState(() => _celsius = value);
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Child 1: input celsius
        CelsiusInput(
          celsius: _celsius,
          onChanged: _updateTemperature,
        ),
        // Child 2: hiển thị fahrenheit (derived từ cùng state)
        FahrenheitDisplay(celsius: _celsius),
      ],
    );
  }
}

class CelsiusInput extends StatelessWidget {
  final double celsius;
  final ValueChanged<double> onChanged;
  const CelsiusInput({super.key, required this.celsius, required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return Slider(
      value: celsius, min: -40, max: 100,
      onChanged: onChanged,
    );
  }
}

class FahrenheitDisplay extends StatelessWidget {
  final double celsius;
  const FahrenheitDisplay({super.key, required this.celsius});

  @override
  Widget build(BuildContext context) {
    return Text('${(celsius * 9 / 5 + 32).toStringAsFixed(1)}°F');
  }
}
```

**Khi nào dùng**: 2–3 widget cần chung state, nằm gần nhau trong tree. Khi phải lift quá 3 level → chuyển sang InheritedWidget hoặc Provider.

> ⚠️ **FE Trap:** FE dev thường map `setState` = `useState` setter. **Nguy hiểm!** React `setCount(n)` chỉ trigger re-render component đó. Flutter `setState()` gọi `markNeedsBuild()` → rebuild **toàn bộ `build()` method + tất cả child widgets**. Đây là lý do Flutter CẦN tách widget nhỏ hơn React cần tách component.

---

## 3. InheritedWidget — Cơ chế gốc của Flutter 🟡

### 3.1 Vấn đề InheritedWidget giải quyết

Thay vì truyền data từ cha → con → cháu (prop drilling), InheritedWidget cho phép **bất kỳ widget con nào** truy cập data trực tiếp.

```
// TRƯỚC: Prop drilling
        App
         │ (truyền theme)
      HomePage
         │ (truyền theme)
      Section
         │ (truyền theme)
      Button ← chỉ mình nó cần theme!

// SAU: InheritedWidget
        App
         │
    InheritedWidget (giữ theme) ← cung cấp cho mọi widget con
         │
      HomePage
         │
      Section
         │
      Button ← truy cập theme trực tiếp qua context
```

### 3.2 Cách hoạt động

InheritedWidget là một widget đặc biệt:
- Nằm trong widget tree như widget bình thường
- Con cháu có thể **lookup** nó qua `context.dependOnInheritedWidgetOfExactType<T>()` — phép tìm **O(1)** (không phải đi ngược cây)
- Khi data thay đổi + `updateShouldNotify` trả về `true` → tất cả widget đã đăng ký sẽ rebuild

```dart
// Tạo InheritedWidget
class CounterInherited extends InheritedWidget {
  final int count;
  final VoidCallback increment;

  const CounterInherited({
    super.key,
    required this.count,
    required this.increment,
    required super.child,
  });

  // Pattern phổ biến: static method of() để truy cập dễ hơn
  static CounterInherited of(BuildContext context) {
    final result =
        context.dependOnInheritedWidgetOfExactType<CounterInherited>();
    assert(result != null, 'CounterInherited not found in context');
    return result!;
  }

  @override
  bool updateShouldNotify(CounterInherited oldWidget) {
    return count != oldWidget.count; // Chỉ rebuild khi count thay đổi
  }
}
```

### 3.3 Pattern of()

```dart
// Widget con truy cập data:
class CounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // O(1) lookup — Flutter lưu map type → widget ở mỗi Element
    final counter = CounterInherited.of(context);
    return Text('Count: ${counter.count}');
  }
}
```

> Bạn đã dùng pattern này rồi! `Theme.of(context)`, `MediaQuery.of(context)`, `Navigator.of(context)` — tất cả đều dùng InheritedWidget bên dưới.

### 3.4 Nhược điểm

| Nhược điểm | Chi tiết |
|-------------|----------|
| **Boilerplate nhiều** | Phải viết class riêng, `of()` method, `updateShouldNotify` |
| **Không có notify mechanism** | InheritedWidget chỉ **truyền data xuống**, không tự thông báo khi data thay đổi — cần kết hợp với `StatefulWidget` ở trên |
| **Khó tái sử dụng** | Mỗi loại data cần 1 InheritedWidget riêng |

→ Đây là lý do **Provider** ra đời — nó wrap InheritedWidget và thêm các tiện ích cần thiết.

---

> 💼 **Gặp trong dự án:** Truyền theme, locale, auth state xuyên suốt widget tree mà không phải pass qua constructor từng widget. Senior yêu cầu hiểu InheritedWidget trước khi dùng Provider.
> 🤖 **Keywords bắt buộc trong prompt:** `InheritedWidget`, `of(context)`, `updateShouldNotify`, `ChangeNotifier wrap`, `Provider package vs raw InheritedWidget`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Theme switching:** App cần đổi giữa light/dark theme, tất cả widgets phải update khi user toggle — dùng InheritedWidget để truyền ThemeData
- **So sánh:** PM hỏi "tại sao không dùng Provider luôn?" — cần hiểu InheritedWidget là foundation, Provider chỉ là wrapper

**Tại sao cần các keyword trên:**
- **`InheritedWidget`** — AI cần gen class đúng pattern: extend InheritedWidget, override `updateShouldNotify`
- **`of(context)`** — pattern truy cập data từ ancestor, AI hay quên null safety (`of(context)!` vs `maybeOf`)
- **`updateShouldNotify`** — quyết định khi nào rebuild dependents, AI hay return true mọi lúc (tốn performance)
- **`Provider package vs raw InheritedWidget`** — AI phải giải thích trade-off, không chỉ nói "dùng Provider"

**Prompt mẫu — InheritedWidget custom:**
```text
Tạo InheritedWidget custom cho theme switching trong Flutter.
Context: App cần toggle light/dark theme, tất cả child widgets phải rebuild khi theme đổi.
Constraints:
- Viết KHÔNG dùng Provider package — chỉ raw InheritedWidget + StatefulWidget.
- Class ThemeConfig chứa: brightness, primaryColor, backgroundColor, textColor.
- of(context) method trả về ThemeConfig, throw nếu không tìm thấy.
- updateShouldNotify: chỉ notify khi brightness thay đổi (tối ưu rebuild).
- Demo: 1 screen với background, text, và toggle button.
Output: 3 files — theme_config.dart, theme_inherited_widget.dart, demo_screen.dart.
Sau đó: viết lại bằng Provider để so sánh code — highlight chỗ nào Provider giảm boilerplate.
```

**Expected Output:** AI gen 3 files raw InheritedWidget + 1 file Provider version + bảng so sánh code lines/complexity.

⚠️ **Giới hạn AI hay mắc:** AI hay quên wrap InheritedWidget trong StatefulWidget (InheritedWidget là immutable, cần StatefulWidget ở trên để thay đổi data). AI cũng hay return `true` trong `updateShouldNotify` thay vì compare actual values.

</details>

> 🔗 **FE Bridge:** InheritedWidget ≈ `React.createContext` + `useContext` — nhưng **khác ở**: InheritedWidget là **Widget class** (nằm trong tree), React Context là API (tách biệt). InheritedWidget dùng O(1) lookup qua `context.dependOnInheritedWidgetOfExactType()`, không phải traverse tree.

---

## 4. ChangeNotifier + Provider 🔴

### 4.1 Provider là gì?

**Provider** = package wrapping InheritedWidget, do **Remi Rousselet** viết và được **Google recommend**.

```
InheritedWidget (khó dùng) + ChangeNotifier (observable)
    ↓ wrap lại
Provider (dễ dùng, ít boilerplate, type-safe)
```

Thêm dependency:

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.2
```

### 4.2 ChangeNotifier — Observable Object

`ChangeNotifier` là class trong Flutter SDK cho phép notify listeners khi data thay đổi.

```dart
import 'package:flutter/foundation.dart';

class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count; // Getter — expose read-only

  void increment() {
    _count++;
    notifyListeners(); // ← Thông báo cho tất cả listeners
  }

  void decrement() {
    _count--;
    notifyListeners();
  }

  void reset() {
    _count = 0;
    notifyListeners();
  }
}
```

**Quy tắc quan trọng**:
- Biến state nên là **private** (`_count`), expose qua **getter**
- **Luôn gọi `notifyListeners()`** sau khi thay đổi state
- Đặt **logic** ở đây, không phải trong widget

> 🔗 **FE Bridge:** Provider ≈ Redux/Pinia store + React Context Provider — nhưng **khác ở**: Provider là **wrapper trên InheritedWidget**, không phải pub/sub pattern. `ChangeNotifier.notifyListeners()` = explicit trigger (phải gọi thủ công), React state tự trigger khi set. Quên `notifyListeners()` = UI không update.

### 4.3 ChangeNotifierProvider — Cung cấp state cho widget tree

```dart
// main.dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: const MyApp(),
    ),
  );
}
```

Khi cần nhiều Provider:

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => CounterModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
        ChangeNotifierProvider(create: (_) => ThemeModel()),
      ],
      child: const MyApp(),
    ),
  );
}
```

### 4.4 Đọc state — watch, read, select

```dart
// 1. context.watch<T>() — LISTEN to changes (rebuild khi state thay đổi)
// Dùng trong build()
@override
Widget build(BuildContext context) {
  final counter = context.watch<CounterModel>();
  return Text('${counter.count}');
}

// 2. context.read<T>() — ONE-TIME read (KHÔNG rebuild)
// Dùng trong callbacks/event handlers
ElevatedButton(
  onPressed: () {
    context.read<CounterModel>().increment(); // Chỉ đọc 1 lần, gọi method
  },
  child: const Text('Increment'),
)

// 3. context.select<T, R>() — Chỉ listen MỘT PHẦN state
// Rebuild chỉ khi phần được select thay đổi
@override
Widget build(BuildContext context) {
  // Chỉ rebuild khi count thay đổi, ignore các field khác
  final count = context.select<CounterModel, int>((m) => m.count);
  return Text('$count');
}
```

### 4.5 Consumer Widget

Alternative cho `context.watch()` — giới hạn phạm vi rebuild:

```dart
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Header — KHÔNG rebuild'), // ← không bị rebuild
      Consumer<CounterModel>(
        builder: (context, counter, child) {
          return Text('Count: ${counter.count}'); // ← CHỈ phần này rebuild
        },
      ),
      const Text('Footer — KHÔNG rebuild'), // ← không bị rebuild
    ],
  );
}
```

### 4.6 Tóm tắt Provider API

| API | Khi nào dùng | Rebuild? |
|-----|--------------|----------|
| `context.watch<T>()` | Trong `build()` — cần hiển thị state | ✅ Có |
| `context.read<T>()` | Trong callback — gọi method | ❌ Không |
| `context.select<T, R>()` | Trong `build()` — chỉ cần 1 phần state | ✅ Chỉ khi phần đó đổi |
| `Consumer<T>` | Giới hạn vùng rebuild trong widget tree | ✅ Trong builder |

> ⚠️ **Lỗi phổ biến**: Dùng `context.read()` trong `build()` → UI không cập nhật khi state thay đổi!

> 🆕 **Concept mới hoàn toàn:** `context.watch()` vs `context.read()` vs `context.select()` không có mapping trực tiếp trong React. React `useContext` luôn subscribe (giống `watch`). Flutter tách rõ: `watch` = reactive, `read` = one-time, `select` = chỉ listen một field. Dùng sai watch/read → bug khó debug hoặc performance kém.

---

> 💼 **Gặp trong dự án:** Shopping cart state, user preferences, notification count — cần chia sẻ state giữa nhiều screens, watch vs read dùng sai là bug khó debug
> 🤖 **Keywords bắt buộc trong prompt:** `ChangeNotifier`, `ChangeNotifierProvider`, `context.watch vs context.read`, `Consumer widget`, `ProxyProvider`, `MultiProvider`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Shopping cart:** Nhiều screens cần biết cart count (AppBar badge), cart items (CartScreen), total price (CheckoutScreen)
- **Bug thực tế:** Junior dùng `context.read<CartModel>()` trong `build()` → UI không update khi add item vào cart
- **MultiProvider:** App production có 5-10 providers — cần organize đúng thứ tự

**Tại sao cần các keyword trên:**
- **`context.watch vs context.read`** — AI phải dùng `watch` trong build (cần rebuild) và `read` trong callbacks (không cần rebuild)
- **`Consumer widget`** — giới hạn vùng rebuild, AI hay wrap cả screen trong Consumer (thừa)
- **`ProxyProvider`** — khi 1 provider phụ thuộc provider khác, AI hay tạo circular dependency
- **`MultiProvider`** — thứ tự providers quan trọng (provider phụ thuộc phải đặt SAU)

**Prompt mẫu — Shopping Cart Provider:**
```text
Tôi cần implement Shopping Cart dùng Provider pattern.
Tech stack: Flutter 3.x, provider ^6.x.
Features:
- CartModel (ChangeNotifier): addItem, removeItem, clearCart, totalPrice, itemCount.
- Hiển thị cart badge count trên AppBar (mọi screen).
- CartScreen: list items + swipe to delete + total price.
- CheckoutScreen: summary + place order button.
Constraints:
- Dùng context.watch trong build cho UI cần update.
- Dùng context.read trong onPressed callbacks.
- Consumer widget: chỉ wrap phần cần rebuild (badge, list), KHÔNG wrap cả Scaffold.
- MultiProvider setup cho CartModel + UserModel (2 providers).
Output: cart_model.dart + main.dart (MultiProvider) + cart_screen.dart + checkout_screen.dart.
```

**Expected Output:** AI gen 4 files với proper watch/read usage, Consumer placement, MultiProvider setup.

⚠️ **Giới hạn AI hay mắc:** AI hay dùng `context.watch` trong `onPressed` callback (gây lỗi). AI cũng hay quên `Consumer` và wrap toàn bộ widget tree trong `context.watch` (rebuild thừa).

</details>

---

## 5. Forms & Validation 🟡

### 5.1 Tại sao cần Form?

Hầu hết mọi app đều có form: login, register, search, settings... Flutter cung cấp `Form` widget để quản lý trạng thái và validation của nhiều input fields cùng lúc.

### 5.2 Các thành phần chính

```
Form (container)
 │
 ├── GlobalKey<FormState> ← key để truy cập form state
 │
 ├── TextFormField
 │    ├── decoration: InputDecoration ← giao diện
 │    ├── validator: (value) => ... ← hàm validate
 │    ├── onSaved: (value) => ... ← lưu giá trị khi form.save()
 │    └── controller: TextEditingController ← điều khiển text
 │
 └── TextFormField (nhiều fields...)
```

### 5.3 GlobalKey\<FormState\>

```dart
class _LoginFormState extends State<LoginForm> {
  // Key để truy cập FormState
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          // ... form fields
          ElevatedButton(
            onPressed: () {
              // validate() gọi validator() của TẤT CẢ TextFormField
              if (_formKey.currentState!.validate()) {
                _formKey.currentState!.save(); // Gọi onSaved() của tất cả fields
                // Xử lý submit...
              }
            },
            child: const Text('Submit'),
          ),
        ],
      ),
    );
  }
}
```

### 5.4 TextFormField & Validator

```dart
TextFormField(
  decoration: const InputDecoration(
    labelText: 'Email',
    hintText: 'you@example.com',
    prefixIcon: Icon(Icons.email),
    border: OutlineInputBorder(),
  ),
  keyboardType: TextInputType.emailAddress,

  // Validator: trả về null = hợp lệ, String = error message
  validator: (value) {
    if (value == null || value.isEmpty) {
      return 'Vui lòng nhập email';
    }
    if (!RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(value)) {
      return 'Email không hợp lệ';
    }
    return null; // Hợp lệ
  },

  onSaved: (value) {
    _email = value!;
  },
)
```

### 5.5 TextEditingController

```dart
class _MyFormState extends State<MyForm> {
  final _nameController = TextEditingController();

  @override
  void dispose() {
    _nameController.dispose(); // ⚠️ PHẢI dispose để tránh memory leak
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      controller: _nameController,
      // Đọc giá trị: _nameController.text
      // Gán giá trị: _nameController.text = 'Hello'
      // Listen thay đổi: _nameController.addListener(() { ... })
    );
  }
}
```

### 5.6 Các phương thức Form

| Method | Chức năng |
|--------|-----------|
| `_formKey.currentState!.validate()` | Chạy tất cả validators, trả về `true` nếu tất cả hợp lệ |
| `_formKey.currentState!.save()` | Gọi `onSaved` callback trên tất cả fields |
| `_formKey.currentState!.reset()` | Reset tất cả fields về giá trị ban đầu |

### 5.7 AutovalidateMode

```dart
Form(
  key: _formKey,
  // KHÔNG validate tự động (default) — chỉ khi submit
  autovalidateMode: AutovalidateMode.disabled,

  // Validate khi user tương tác (gõ, blur...)
  // autovalidateMode: AutovalidateMode.onUserInteraction,

  // Validate luôn từ đầu
  // autovalidateMode: AutovalidateMode.always,
  child: ...
)
```

> 📖 **Đọc thêm — Form nâng cao**
>
> Các chủ đề nâng cao về Forms không nằm trong scope buổi này:
> - Custom `FormField` widgets (DatePicker, Dropdown có validation)
> - Async validation (kiểm tra email tồn tại qua API)
> - Multi-step forms (wizard pattern)
> - Package hỗ trợ: [`reactive_forms`](https://pub.dev/packages/reactive_forms), [`flutter_form_builder`](https://pub.dev/packages/flutter_form_builder)
>
> Xem thêm tại [Tài liệu tham khảo](04-tai-lieu-tham-khao.md).

> 🔗 **FE Bridge:** `Form` + `TextFormField` ≈ React Hook Form / Formik — nhưng **khác ở**: Flutter Form dùng `GlobalKey<FormState>` để validate, không dùng hook pattern. `TextEditingController` = controlled input (giống React controlled component), nhưng controller phải `dispose()` thủ công.

---

## 6. Tổng quan các giải pháp State Management 🟡

### 6.1 Bảng so sánh

| Tiêu chí | **Provider** | **Riverpod** | **BLoC** | **GetX** |
|-----------|-------------|-------------|---------|---------|
| Độ khó | ⭐ Dễ | ⭐⭐ Trung bình | ⭐⭐⭐ Khó | ⭐ Dễ |
| Boilerplate | Ít | Ít | Nhiều | Rất ít |
| Google backed | ✅ | ❌ (cùng tác giả) | ❌ | ❌ |
| Compile-safe | ❌ | ✅ | ✅ | ❌ |
| Testability | Tốt | ✅ Rất tốt | ✅ Rất tốt | Trung bình |
| DevTools integration | ✅ | ✅ | ✅ (BlocObserver) | ❌ |
| Code generation | ❌ | ✅ (riverpod_generator) | ❌ | ❌ |
| Auto-disposal | ❌ | ✅ (autoDispose mặc định) | ❌ (manual close) | ⚠️ |
| Type safety | ⚠️ Runtime | ✅ Compile-time | ✅ Compile-time | ❌ Runtime |
| Learning curve | Thấp | Trung bình | Cao | Thấp |
| Enterprise ready | ✅ | ✅ | ✅ | ⚠️ |
| Cộng đồng | Lớn | Đang lớn | Lớn | Lớn nhưng tranh cãi |

### 6.2 Khi nào dùng gì?

```
App nhỏ, prototype, học Flutter
    → Provider ✅

App trung bình, team nhỏ
    → Provider hoặc Riverpod ✅

App lớn, team lớn, cần event-driven architecture
    → BLoC ✅

Cần compile-safe, không phụ thuộc BuildContext
    → Riverpod ✅
```

### 6.3 Decision Tree

```
Bạn mới học Flutter?
│
├── CÓ → Học Provider trước ✅
│         (Đơn giản, Google recommend, tài liệu nhiều)
│
└── KHÔNG → App có yêu cầu đặc biệt?
             │
             ├── Cần compile-safe + no context → Riverpod
             │
             ├── Cần event-driven + enterprise → BLoC
             │
             └── Không có yêu cầu đặc biệt → Provider vẫn ổn ✅
```

> ⚠️ **Tại sao không dạy GetX?**
>
> GetX phổ biến nhưng có nhiều vấn đề:
> - Magic globals — khó test, khó debug
> - Không tương thích với devtools inspection
> - Breaking changes thường xuyên giữa các version
> - Cộng đồng enterprise và Google chọn Riverpod/BLoC
>
> Nếu dự án yêu cầu GetX, bạn có thể tự học — nhưng training này focus vào patterns được industry recommend.

> **Lời khuyên**: Đừng quá đau đầu chọn giải pháp. Học Provider cho vững cái nền, sau đó thử Riverpod hoặc BLoC khi project yêu cầu. Các concepts (observable, listen, rebuild) đều giống nhau.

---

## 7. Best Practices & Lỗi thường gặp 🟡

### 7.1 Best Practices

```dart
// ✅ 1. Tách state logic ra khỏi widget
// SAI: logic trong widget
setState(() { _items.add(item); _total += item.price; });

// ĐÚNG: logic trong ChangeNotifier
class CartModel extends ChangeNotifier {
  void addItem(Product item) {
    _items.add(item);
    notifyListeners();
  }
  double get total => _items.fold(0, (sum, item) => sum + item.price);
}

// ✅ 2. Private state, public getters
class UserModel extends ChangeNotifier {
  String _name = '';
  String get name => _name; // read-only access
}

// ✅ 3. Dùng context.read() trong callback, context.watch() trong build
onPressed: () => context.read<CartModel>().addItem(product); // ✅
final cart = context.watch<CartModel>(); // ✅ trong build()

// ✅ 4. Dùng select() khi chỉ cần một phần state
final count = context.select<CartModel, int>((cart) => cart.itemCount);

// ✅ 5. Dispose controllers
@override
void dispose() {
  _emailController.dispose();
  _passwordController.dispose();
  super.dispose();
}
```

### 7.2 Lỗi thường gặp

| Lỗi | Hậu quả | Cách sửa |
|------|---------|----------|
| Quên `notifyListeners()` | UI không cập nhật | Luôn gọi sau khi đổi state |
| `context.read()` trong `build()` | UI không reactive | Dùng `context.watch()` hoặc `Consumer` |
| `context.watch()` trong callback | Lỗi/hành vi không mong muốn | Dùng `context.read()` |
| Quên dispose `TextEditingController` | Memory leak | Override `dispose()` |
| setState sau dispose | Exception | Check `mounted` trước |
| Form không có `GlobalKey` | Không truy cập được FormState | Tạo `GlobalKey<FormState>` |
| Logic xử lý trong widget | Khó test, khó maintain | Tách vào ChangeNotifier |

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Flutter Mindset | Tại sao khác |
|---|-------------------|-----------------|---------------|
| 1 | `useState` re-render component đó | `setState` rebuild toàn bộ subtree → cần tách widget nhỏ | Flutter Widget Tree rebuild scope lớn hơn React |
| 2 | Mọi state đưa vào global store (Redux/Zustand) | Ephemeral state dùng `setState`, chỉ app state dùng Provider | Flutter khuyến khích local-first state management |
| 3 | `useContext` = subscribe tự động | `watch` vs `read` vs `select` — phải chọn đúng | Flutter tách rõ reactive vs one-time read |
| 4 | State update → auto re-render | `notifyListeners()` phải gọi **thủ công** | ChangeNotifier = explicit notification, không auto |
| 5 | Controlled input = `value` + `onChange` | Controlled input = `TextEditingController` + phải `dispose()` | Controller là object cần lifecycle management |

Nếu bạn đến từ React hoặc Vue, đây là mapping giúp hiểu nhanh:

| Flutter | React | Vue |
|---------|-------|-----|
| `setState()` | `useState()` / `this.setState()` | `ref()` / `reactive()` |
| `InheritedWidget` | `React.createContext()` + `useContext()` | `provide()` / `inject()` |
| `Provider` | React Context + `useReducer()` hoặc Redux | Pinia / Vuex |
| `ChangeNotifier` | Redux Store / Zustand Store | Pinia Store |
| `context.watch()` | `useContext()` (auto re-render) | Computed property |
| `context.read()` | `useRef` (no re-render) hoặc callback read | Không trigger watch |
| `context.select()` | ⛔ Không có tương đương trực tiếp — React `useContext` luôn subscribe | ⛔ Computed với deep watch |
| ⛔ `watch/read/select` pattern | ⛔ Không có tương đương trực tiếp — React useContext luôn subscribe | ⛔ Vue reactivity tự động track |
| `Consumer` | `Context.Consumer` | `<slot>` + scoped |
| `Form` + `GlobalKey<FormState>` | React Hook Form / Formik | VeeValidate / FormKit |
| `TextFormField.validator` | Yup schema / Zod | Validation rules |
| `notifyListeners()` | `dispatch()` / Store update | Tự động (reactivity) |

### So sánh cụ thể

**setState vs useState:**
```
// Flutter
setState(() { _count++; });  // Explicit call → rebuild

// React
const [count, setCount] = useState(0);
setCount(count + 1);  // Trigger re-render
```
→ Giống nhau: đều trigger rebuild/re-render. Khác: Flutter `setState` gộp thay đổi trong 1 callback, React có batching tự động.

**Provider vs Redux/Context:**
```
// Flutter Provider
ChangeNotifierProvider(create: (_) => CartModel(), child: App())
final cart = context.watch<CartModel>();

// React Context + useContext
<CartContext.Provider value={cartState}><App /></CartContext.Provider>
const cart = useContext(CartContext);
```
→ Provider = React Context nhưng tích hợp sẵn notify mechanism.

**Key takeaway**: Flutter state management quen thuộc nhưng **explicit hơn** — bạn phải tự gọi `notifyListeners()`, tự chọn `watch()` vs `read()`. Trong React/Vue, reactivity phần lớn tự động.

---

## 9. Tổng kết

### ✅ Checklist kiến thức buổi 6

Sau buổi này, bạn phải tự tin nói "Có" với tất cả:

- [ ] Tôi phân biệt được **ephemeral state** vs **app state**
- [ ] Tôi biết khi nào `setState()` là đủ, khi nào cần state management
- [ ] Tôi hiểu `setState()` hoạt động thế nào bên trong (mark dirty → rebuild)
- [ ] Tôi hiểu `InheritedWidget` và pattern `of(context)` (O(1) lookup)
- [ ] Tôi dùng được `ChangeNotifier` + `notifyListeners()` để tạo observable object
- [ ] Tôi dùng được `ChangeNotifierProvider` để cung cấp state cho widget tree
- [ ] Tôi phân biệt `context.watch()` vs `context.read()` vs `context.select()`
- [ ] Tôi dùng `Consumer` để giới hạn vùng rebuild
- [ ] Tôi tạo được `Form` với `GlobalKey<FormState>`
- [ ] Tôi viết được `validator` cho `TextFormField`
- [ ] Tôi biết gọi `validate()`, `save()`, `reset()` trên Form
- [ ] Tôi dispose `TextEditingController` đúng cách
- [ ] Tôi biết tổng quan Provider vs Riverpod vs BLoC vs GetX
- [ ] Tôi biết best practices: tách logic, private state, chọn watch/read đúng

### 🔑 Key takeaways

1. **setState()** chỉ dùng cho ephemeral state đơn giản
2. **InheritedWidget** là nền tảng, nhưng quá nhiều boilerplate → dùng **Provider**
3. **Provider** = ChangeNotifier + ChangeNotifierProvider + watch/read — đủ cho hầu hết app
4. **Form** = `Form` + `GlobalKey<FormState>` + `TextFormField` + `validator`
5. Luôn **tách state logic khỏi widget** — đặt trong ChangeNotifier

### ➡️ Buổi tiếp theo

Buổi 7 sẽ đi vào **Riverpod** — state management solution hiện đại, type-safe và testable cho Flutter.
