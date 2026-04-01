# Buổi 03: Widget Tree — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 3/16** · **Thời lượng tự học:** ~1 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 02 (lý thuyết + ít nhất BT1-BT2)

> Đọc kỹ từng phần. Mỗi concept đều là nền tảng cho mọi thứ bạn sẽ xây dựng trong Flutter.

---

## 1. Widget là gì? — "Everything is a Widget" 🔴

### 1.1 Concept cốt lõi

Trong Flutter, **mọi thứ bạn nhìn thấy trên màn hình đều là Widget** — từ một đoạn text, một nút bấm, đến cả padding, margin, layout.

```
┌─────────────────────────────────────────┐
│              MaterialApp                 │
│  ┌───────────────────────────────────┐  │
│  │            Scaffold               │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │         AppBar              │  │  │
│  │  │    ┌─────────────────┐      │  │  │
│  │  │    │     Text        │      │  │  │
│  │  │    └─────────────────┘      │  │  │
│  │  └─────────────────────────────┘  │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │         Column              │  │  │
│  │  │   ┌──────┐  ┌───────────┐  │  │  │
│  │  │   │ Icon │  │ Text      │  │  │  │
│  │  │   └──────┘  └───────────┘  │  │  │
│  │  │   ┌──────────────────────┐ │  │  │
│  │  │   │  ElevatedButton      │ │  │  │
│  │  │   └──────────────────────┘ │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 1.2 Đặc tính quan trọng của Widget

| Đặc tính | Giải thích |
|-----------|-----------|
| **Immutable** | Widget không thể thay đổi sau khi tạo. Muốn thay đổi UI? Tạo widget mới. |
| **Lightweight** | Widget chỉ là "bản mô tả" (configuration), không phải pixel thực tế trên màn hình. |
| **Composable** | Widget lớn được tạo bằng cách ghép nhiều widget nhỏ lại. |
| **Declarative** | Bạn mô tả UI trông như thế nào, Flutter lo việc render. |

```dart
// Widget chỉ là một "đơn đặt hàng" mô tả UI
// Nó KHÔNG phải element trên màn hình
class Text extends Widget {
  final String data;        // Immutable — không thể thay đổi
  final TextStyle? style;   // Chỉ mô tả: "Tôi muốn text trông thế này"
  // ...
}
```

### 1.3 So sánh tư duy

```
// Android (Imperative):
textView.setText("Hello");   // Trực tiếp thay đổi UI element

// Flutter (Declarative):
Text("Hello")                // Mô tả "tôi muốn hiển thị text Hello"
                             // Flutter tự lo việc render
```

> 💡 **Nhớ:** Widget = **blueprint** (bản thiết kế), không phải ngôi nhà thật.

> 🔗 **FE Bridge:** Tương tự `Component` trong React/Vue — nhưng **khác ở**: Widget là **immutable blueprint**, không phải instance. Mỗi lần rebuild, Widget cũ bị discard và tạo mới hoàn toàn. React component giữ reference, Flutter Widget không giữ.

---

## 2. StatelessWidget vs StatefulWidget 🔴

### 2.1 StatelessWidget — Widget "tĩnh"

**StatelessWidget** không có state (trạng thái) bên trong. Nó chỉ phụ thuộc vào **input** (constructor parameters). Cùng input → cùng output, mọi lúc.

```dart
class GreetingCard extends StatelessWidget {
  final String name;      // Chỉ phụ thuộc vào input
  final String message;

  const GreetingCard({
    super.key,
    required this.name,
    required this.message,
  });

  @override
  Widget build(BuildContext context) {
    // build() chỉ được gọi khi:
    // 1. Widget được tạo lần đầu
    // 2. Parent rebuild và truyền input khác
    return Card(
      child: Text('$name: $message'),
    );
  }
}
```

**Khi nào dùng StatelessWidget:**
- Hiển thị data tĩnh (text, icon, hình ảnh)
- Widget chỉ phụ thuộc vào input từ parent
- Không có gì thay đổi theo thời gian bên trong widget

### 2.2 StatefulWidget — Widget "động"

**StatefulWidget** có một **State object** đi kèm. State object này có thể thay đổi theo thời gian, và khi thay đổi → UI được rebuild.

```dart
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;  // State — có thể thay đổi!

  void _increment() {
    setState(() {
      _count++;    // Thay đổi state → trigger rebuild
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Tăng'),
        ),
      ],
    );
  }
}
```

### 2.3 Bảng so sánh

| Tiêu chí | StatelessWidget | StatefulWidget |
|----------|----------------|----------------|
| Internal state | ❌ Không có | ✅ Có (State object) |
| Thay đổi theo thời gian | ❌ Không | ✅ Có |
| `build()` gọi khi nào | Parent rebuild | Parent rebuild HOẶC `setState()` |
| Dùng khi | Hiển thị data tĩnh | Tương tác user, animation, data thay đổi |
| Cấu trúc class | 1 class | 2 class (Widget + State) |
| Performance | Nhẹ hơn | Nặng hơn một chút |

### 2.4 Quy tắc chọn

```
Widget của bạn có cần thay đổi gì sau khi render không?
│
├── KHÔNG → StatelessWidget ✅
│   (VD: Text, Icon, Avatar, Label...)
│
└── CÓ → StatefulWidget ✅
    (VD: Counter, Form, Animation, Toggle...)
```

> ⚠️ **Quy tắc vàng:** Luôn bắt đầu với **StatelessWidget**. Chỉ chuyển sang StatefulWidget khi thực sự cần state bên trong.

> 🔗 **FE Bridge:** StatelessWidget ≈ `Pure Functional Component` (no hooks), StatefulWidget ≈ `Component + useState`. Nhưng **khác ở**: `setState()` trong Flutter trigger rebuild **TOÀN BỘ subtree**, không chỉ component đó. React hooks chỉ re-render component sử dụng state.

---

## 3. Widget Tree, Element Tree, RenderObject Tree — Kiến trúc 3 cây 🔴

Đây là kiến thức **quan trọng nhất** buổi hôm nay. Hiểu 3 trees = hiểu tại sao Flutter nhanh.

### 3.1 Tổng quan 3 Trees

```
    Widget Tree              Element Tree           RenderObject Tree
  (Configuration)           (Instances)              (Rendering)
                                                    
  ┌──────────┐           ┌──────────────┐         ┌──────────────┐
  │MaterialApp│──creates──▶│MaterialApp   │──creates─▶│RenderView   │
  └────┬─────┘           │  Element     │         └──────┬───────┘
       │                 └──────┬───────┘                │
  ┌────▼─────┐           ┌──────▼───────┐         ┌──────▼───────┐
  │ Scaffold  │──creates──▶│Scaffold     │──creates─▶│RenderFlex   │
  └────┬─────┘           │  Element     │         └──────┬───────┘
       │                 └──────┬───────┘                │
  ┌────▼─────┐           ┌──────▼───────┐         ┌──────▼───────┐
  │  Text     │──creates──▶│Text Element  │──creates─▶│RenderPara-  │
  │ "Hello"   │           │              │         │  graph       │
  └──────────┘           └──────────────┘         └──────────────┘

   Immutable              Mutable                  Layout + Paint
   Lightweight            Manages lifecycle        Expensive
   Bạn viết cái này       Flutter tự quản lý       Flutter tự quản lý
```

### 3.2 Chi tiết từng Tree

#### Widget Tree — Bản thiết kế

```dart
// Bạn viết code → Flutter tạo Widget Tree
Scaffold(
  appBar: AppBar(title: Text('Demo')),  // Widget
  body: Center(                          // Widget
    child: Text('Hello'),                // Widget
  ),
)
```

- **Bạn tạo ra** bằng code
- **Immutable** — không thay đổi, chỉ tạo mới
- **Lightweight** — chỉ chứa configuration
- Được tạo lại mỗi khi `build()` chạy

#### Element Tree — Quản lý viên

- Flutter tự tạo từ Widget Tree
- **Mutable** — có thể thay đổi
- Quản lý **lifecycle** của widget
- **Giữ reference** đến cả Widget và RenderObject
- **Tồn tại lâu hơn** Widget (Widget bị tạo mới, Element cũ được tái sử dụng)

#### RenderObject Tree — Hoạ sĩ

- Flutter tự tạo từ Element Tree
- Thực hiện **layout** (tính toán kích thước, vị trí)
- Thực hiện **painting** (vẽ pixel lên màn hình)
- **Expensive** — tốn tài nguyên nhất
- Chỉ update khi thực sự cần thiết

### 3.3 Tại sao cần 3 Trees? — Performance!

```
Khi setState() được gọi:

1. Widget Tree: Tạo MỚI hoàn toàn (rẻ — chỉ là object nhẹ)
           ↓
2. Element Tree: SO SÁNH widget cũ vs mới
   ├── Cùng type & key? → Tái sử dụng Element, chỉ update
   └── Khác type?       → Xoá Element cũ, tạo mới
           ↓
3. RenderObject Tree: Chỉ update phần THAY ĐỔI (đắt — nhưng ít xảy ra)
```

> 💡 **Ví dụ thực tế:** Trong app 1000 widget, khi user nhấn nút → chỉ 1 Text thay đổi. Flutter tạo lại 1000 widget (rẻ), nhưng chỉ update 1 RenderObject (đắt). Kết quả: **60fps mượt mà**.

### 3.4 So sánh dễ hiểu

| | Widget Tree | Element Tree | RenderObject Tree |
|---|---|---|---|
| Ví dụ đời thực | Bản vẽ kiến trúc | Đội thợ xây | Ngôi nhà thật |
| Bản chất | Configuration | Manager | Renderer |
| Tạo mới khi rebuild? | ✅ Có (rẻ) | ❌ Tái sử dụng | ❌ Chỉ update nếu cần |
| Ai tạo? | Developer (bạn) | Flutter framework | Flutter framework |
| Mutable? | ❌ Immutable | ✅ Mutable | ✅ Mutable |

---

> 💼 **Gặp trong dự án:** Debug performance issues (tại sao UI giật?), hiểu tại sao `const` widget cải thiện performance, giải thích cho junior tại sao thay đổi parent không nhất thiết rebuild child
> 🤖 **Keywords bắt buộc trong prompt:** `Widget Tree vs Element Tree vs RenderObject Tree`, `widget rebuild cost`, `const constructor optimization`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Performance review:** Lead hỏi "tại sao màn hình danh sách 500 item vẫn mượt?" — cần giải thích cơ chế 3 trees
- **Code review:** Reviewer comment "thiếu const ở widget không có dynamic data" — cần hiểu tại sao const widget skip rebuild
- **Debug:** Widget không update dù setState rồi — có thể Element Tree tái sử dụng widget cũ vì thiếu Key

**Tại sao cần các keyword trên:**
- **`Widget Tree vs Element Tree vs RenderObject Tree`** — AI cần phân biệt rõ 3 layers, không nhầm lẫn
- **`widget rebuild cost`** — để AI giải thích "rebuild widget rẻ, chỉ render object update mới đắt"
- **`const constructor optimization`** — AI phải nói rõ: const widget = same instance = Element Tree skip canUpdate

**Prompt mẫu — Giải thích 3 Trees cho team:**
```text
Tôi cần giải thích Flutter 3 Trees architecture cho team FE chuyển sang Flutter.
Context: Team có background React, quen Virtual DOM diffing.
Yêu cầu:
- So sánh React Virtual DOM vs Flutter 3 Trees (Widget/Element/RenderObject).
- Giải thích tại sao Flutter rebuild toàn bộ Widget Tree mỗi frame mà vẫn 60fps.
- Demo: const widget giúp skip rebuild như thế nào (code + sơ đồ text).
- Khi nào cần Key và Key giúp Element Tree thế nào.
Output: Giải thích ngắn gọn bằng tiếng Việt, kèm sơ đồ text ASCII và code Dart minh họa.
```

**Expected Output:** AI sẽ giải thích parallel React VDOM ↔ Flutter 3 Trees, kèm ASCII diagram showing rebuild flow, code demo const vs non-const widget.

⚠️ **Giới hạn AI hay mắc:** AI hay nói "Flutter dùng Virtual DOM giống React" — SAI, Flutter KHÔNG có Virtual DOM. Element Tree khác VDOM ở chỗ element tồn tại persistent, không tạo mới mỗi render.

</details>

> ⚠️ **FE Trap:** FE dev thường map Widget Tree = Virtual DOM. **Sai!** React dùng 2 layers (Virtual DOM → Real DOM), Flutter dùng **3 layers** (Widget → Element → RenderObject). Element Tree giữ state + lifecycle — đây là layer KHÔNG CÓ trong React. Quên điều này → không hiểu tại sao Widget immutable mà State vẫn sống.

---

## 4. BuildContext — Chìa khoá truy cập Widget Tree 🔴

### 4.1 BuildContext là gì?

**BuildContext** = **reference đến vị trí** của Widget trong Element Tree.

```
               MaterialApp
                    │
               Scaffold         ← context ở đây chỉ "thấy" ancestor phía trên
                    │
                 Column
               ┌────┴────┐
             Text     ElevatedButton  ← context ở đây "thấy" Column, Scaffold, MaterialApp
```

Mỗi Widget có **context riêng** — nó biết nó nằm ở đâu trong cây.

### 4.2 Dùng BuildContext để làm gì?

```dart
@override
Widget build(BuildContext context) {
  // 1. Lấy Theme
  final theme = Theme.of(context);
  final primaryColor = theme.colorScheme.primary;

  // 2. Lấy kích thước màn hình
  final screenWidth = MediaQuery.of(context).size.width;

  // 3. Navigate
  Navigator.of(context).push(...);

  // 4. Hiển thị SnackBar
  ScaffoldMessenger.of(context).showSnackBar(...);

  return Container(
    color: primaryColor,
    width: screenWidth * 0.8,
    child: const Text('Hello'),
  );
}
```

### 4.3 Pattern `.of(context)` — Đi lên tìm ancestor

```dart
// Pattern: WidgetTên.of(context)
// Nghĩa: "Đi ngược lên Widget Tree từ vị trí hiện tại, tìm WidgetTên gần nhất"

Theme.of(context)           // Tìm Theme widget gần nhất phía trên
MediaQuery.of(context)      // Tìm MediaQuery gần nhất phía trên  
Navigator.of(context)       // Tìm Navigator gần nhất phía trên
Scaffold.of(context)        // Tìm Scaffold gần nhất phía trên
```

```
        Theme  ◄─── Theme.of(context) tìm thấy ở đây
          │
       Scaffold  ◄─── Scaffold.of(context) tìm thấy ở đây
          │
        Column
          │
     MyWidget  ◄─── context xuất phát từ đây, đi NGƯỢC LÊN
```

### 4.4 Lỗi thường gặp với BuildContext

#### ❌ Lỗi 1: Dùng context trước khi widget mounted

```dart
class _MyState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    // ❌ SAI — context chưa sẵn sàng đầy đủ trong initState
    final theme = Theme.of(context);
  }
}
```

```dart
class _MyState extends State<MyWidget> {
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // ✅ ĐÚNG — context đã sẵn sàng ở đây
    final theme = Theme.of(context);
  }
}
```

#### ❌ Lỗi 2: Dùng context của Scaffold để tìm chính Scaffold

```dart
// ❌ SAI — context của Scaffold chưa "thấy" Scaffold (nó ở cùng level)
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: ElevatedButton(
      onPressed: () {
        // context ở đây là của Widget CHA, không phải của Scaffold
        Scaffold.of(context).openDrawer(); // Lỗi!
      },
      child: const Text('Open Drawer'),
    ),
  );
}
```

```dart
// ✅ ĐÚNG — Dùng Builder để lấy context BÊN TRONG Scaffold
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Builder(
      builder: (scaffoldContext) {
        // scaffoldContext nằm BÊN TRONG Scaffold → tìm được Scaffold
        return ElevatedButton(
          onPressed: () {
            Scaffold.of(scaffoldContext).openDrawer();
          },
          child: const Text('Open Drawer'),
        );
      },
    ),
  );
}
```

> 🆕 **Concept mới hoàn toàn:** BuildContext không có tương đương trực tiếp trong React/Vue. Gần nhất là `useContext` + vị trí component trong tree — nhưng BuildContext là **DANH TÍNH CỤ THỂ** của một Element, không chỉ là "nơi lấy data". Dùng `context.findAncestorWidgetOfExactType()` = traverse tree thực, không phải lookup store.

---

## 5. Key trong Flutter — Giúp Flutter nhận diện Widget 🟡

### 5.1 Tại sao cần Key?

Khi Flutter rebuild, nó so sánh Widget Tree cũ và mới. Mặc định, Flutter dùng **type** và **vị trí** để match widget.

**Vấn đề xuất hiện khi danh sách thay đổi thứ tự:**

```
Trước:                    Sau khi đảo vị trí:
┌─────────────┐          ┌─────────────┐
│ Item A      │ pos 0    │ Item C      │ pos 0  ← Flutter nghĩ A đổi text thành C
│ Item B      │ pos 1    │ Item A      │ pos 1  ← Flutter nghĩ B đổi text thành A
│ Item C      │ pos 2    │ Item B      │ pos 2  ← Flutter nghĩ C đổi text thành B
└─────────────┘          └─────────────┘
 
Không có Key: Flutter update text nhưng GIỮ NGUYÊN state (color, scroll position...)
→ State bị gán SAI widget!
```

### 5.2 Các loại Key

| Key | Dùng khi | Ví dụ |
|-----|---------|-------|
| **ValueKey** | Widget có giá trị unique (id, name) | `ValueKey(todo.id)` |
| **ObjectKey** | Widget gắn với một object cụ thể | `ObjectKey(todoItem)` |
| **UniqueKey** | Muốn widget luôn là unique, mỗi lần build | `UniqueKey()` |
| **GlobalKey** | Cần truy cập State từ bên ngoài | `GlobalKey<FormState>()` |

```dart
// ValueKey — phổ biến nhất
ListView.builder(
  itemCount: todos.length,
  itemBuilder: (context, index) {
    final todo = todos[index];
    return TodoTile(
      key: ValueKey(todo.id),  // ✅ Flutter nhận diện đúng widget khi reorder
      todo: todo,
    );
  },
)

// GlobalKey — truy cập State từ bên ngoài
final formKey = GlobalKey<FormState>();

Form(
  key: formKey,
  child: Column(children: [...]),
)

// Validate form từ bên ngoài
if (formKey.currentState!.validate()) {
  formKey.currentState!.save();
}
```

### 5.3 Khi nào CẦN dùng Key?

- ✅ **Danh sách** có thể reorder, thêm/xoá item
- ✅ **Animation** cần giữ state khi widget đổi vị trí
- ✅ **Form** cần truy cập state từ bên ngoài (GlobalKey)
- ❌ Không cần Key cho widget tĩnh, không thay đổi thứ tự

> 🔗 **FE Bridge:** Tương tự `key` prop trong React — nhưng **khác ở**: React key chỉ dùng cho **list rendering** (`.map()`), Flutter Key dùng rộng hơn: **preserve state** khi widget đổi vị trí, **force rebuild**, và **identify widget** trong animation. Nếu chỉ nghĩ Key = React key → sẽ thiếu nhiều use case.

---

## 6. Widget Lifecycle — Vòng đời StatefulWidget 🔴

### 6.1 Lifecycle Flow

```
                    ┌─────────────────────┐
                    │    createState()     │ ← StatefulWidget tạo State object
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │    initState()       │ ← Khởi tạo (gọi 1 lần duy nhất)
                    │  - Init controllers  │   Setup ban đầu: controllers, listeners
                    │  - Subscribe streams │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────────┐
                    │ didChangeDependencies()  │ ← InheritedWidget thay đổi
                    │  - Theme changed         │   Gọi ngay sau initState() và
                    │  - MediaQuery changed     │   mỗi khi dependency thay đổi
                    └──────────┬───────────────┘
                               │
                    ┌──────────▼──────────┐
            ┌──────│      build()         │ ← Xây dựng UI (gọi nhiều lần)
            │      │  - Return Widget tree │   KHÔNG làm việc nặng ở đây!
            │      └──────────┬──────────┘
            │                 │
            │      ┌──────────▼──────────┐
            │      │  didUpdateWidget()   │ ← Parent rebuild, truyền widget mới
            │      │  - Compare old/new   │   So sánh widget cũ vs mới
            │      └──────────┬──────────┘
            │                 │
            └─────────────────┘  (loop: build ↔ didUpdateWidget)
                               │
                    ┌──────────▼──────────┐
                    │    deactivate()      │ ← Widget bị gỡ khỏi tree
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │     dispose()        │ ← Dọn dẹp (gọi 1 lần cuối cùng)
                    │  - Cancel timers     │   PHẢI cleanup ở đây!
                    │  - Close streams     │
                    │  - Dispose controllers│
                    └─────────────────────┘
```

### 6.2 Chi tiết từng bước

| Method | Gọi khi nào | Làm gì |
|--------|-------------|--------|
| `createState()` | Widget được tạo | Return State object mới |
| `initState()` | State được tạo (1 lần) | Init controllers, listeners, fetch data ban đầu |
| `didChangeDependencies()` | Sau initState + khi InheritedWidget thay đổi | Phản ứng với thay đổi từ ancestor (Theme, MediaQuery) |
| `build()` | Sau initState, sau setState, sau didUpdateWidget | Return Widget Tree — phải pure, không side effect |
| `didUpdateWidget()` | Parent rebuild với config mới | So sánh old widget vs new widget, react accordingly |
| `deactivate()` | Widget bị remove khỏi tree | Hiếm khi override |
| `dispose()` | State bị destroy vĩnh viễn | **BẮT BUỘC** cleanup: dispose controllers, cancel subscriptions |

### 6.3 Ví dụ lifecycle đầy đủ

```dart
class _TimerWidgetState extends State<TimerWidget> {
  late final TextEditingController _controller;
  late final Timer _timer;

  @override
  void initState() {
    super.initState();  // LUÔN gọi super trước
    _controller = TextEditingController();
    _timer = Timer.periodic(
      const Duration(seconds: 1),
      (_) => _updateTime(),
    );
    debugPrint('initState — chạy 1 lần');
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    debugPrint('didChangeDependencies — Theme/MediaQuery thay đổi');
  }

  @override
  void didUpdateWidget(covariant TimerWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    debugPrint('didUpdateWidget — parent rebuild');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('build — xây dựng UI');
    return TextField(controller: _controller);
  }

  @override
  void dispose() {
    _controller.dispose();  // ✅ Cleanup controller
    _timer.cancel();         // ✅ Cancel timer
    debugPrint('dispose — dọn dẹp');
    super.dispose();  // LUÔN gọi super cuối cùng
  }
}
```

> ⚠️ **Quy tắc vàng:** Mọi thứ bạn tạo trong `initState()` → PHẢI dọn trong `dispose()`.

---

> 💼 **Gặp trong dự án:** Memory leak do quên dispose (Timer, StreamSubscription, AnimationController), widget crash khi setState sau dispose, quản lý TextEditingController/FocusNode
> 🤖 **Keywords bắt buộc trong prompt:** `StatefulWidget lifecycle`, `initState dispose pattern`, `mounted check`, `controller cleanup`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Timer widget:** OTP countdown, auto-refresh data mỗi 30s — cần Timer trong initState, cancel trong dispose
- **Form screen:** Nhiều TextEditingController + FocusNode — quên dispose 1 cái = memory leak
- **Async callback:** Gọi API trong initState → response về sau khi user đã navigate away → setState crash

**Tại sao cần các keyword trên:**
- **`StatefulWidget lifecycle`** — AI cần follow đúng thứ tự: createState → initState → didChangeDependencies → build → dispose
- **`initState dispose pattern`** — AI phải tạo và cleanup resource thành cặp, không bỏ sót
- **`mounted check`** — AI phải bọc setState trong `if (mounted)` khi có async callback
- **`controller cleanup`** — AI phải dispose mọi controller trong dispose(), gọi trước super.dispose()

**Prompt mẫu — StatefulWidget với lifecycle đầy đủ:**
```text
Tôi cần tạo Flutter StatefulWidget cho màn hình Chat — có auto-scroll, message polling, và text input.
Context: app chat realtime, widget này trên navigation stack có thể bị pop bất kỳ lúc nào.
Tech stack: Flutter 3.x, Dart 3.x.
Constraints:
- ScrollController cho auto-scroll to bottom khi có message mới.
- Timer.periodic(Duration(seconds: 5)) để poll messages.
- TextEditingController + FocusNode cho input field.
- TẤT CẢ controllers/timer PHẢI dispose đúng trong dispose().
- Mọi setState PHẢI check mounted trước.
- initState chỉ gọi _startPolling(), không gọi async trực tiếp.
Output: 1 file chat_screen.dart với đầy đủ lifecycle methods, có comment giải thích từng bước.
```

**Expected Output:** AI sẽ gen StatefulWidget với 3-4 controllers, initState setup, dispose cleanup, Timer polling với mounted check.

⚠️ **Giới hạn AI hay mắc:** AI hay gọi `async` function trực tiếp trong `initState` (sai — initState không được async). AI cũng hay quên `super.dispose()` hoặc đặt sai thứ tự (phải gọi cuối cùng).

</details>

> ⚠️ **FE Trap:** FE dev thường map `initState` = `useEffect([], ...)` và `dispose` = cleanup function. **Nguy hiểm!** Flutter lifecycle gắn với **State object** (persist qua rebuild), không phải Widget. `didUpdateWidget` được gọi khi parent rebuild → KHÔNG có equivalent trong React hooks. Lifecycle Flutter = **class-based**, không phải hook-based.

---

## 7. setState() — Trigger rebuild UI 🔴

### 7.1 Cách hoạt động

```dart
void _increment() {
  setState(() {
    _count++;  // 1. Thay đổi state
  });
  // 2. setState() đánh dấu Element là "dirty"
  // 3. Flutter lên lịch rebuild
  // 4. build() được gọi lại
  // 5. Widget Tree mới được tạo
  // 6. Element Tree so sánh cũ vs mới
  // 7. RenderObject Tree chỉ update phần thay đổi
}
```

### 7.2 Phạm vi rebuild

```
setState() gọi trong _CounterState
          │
          ▼
    CounterWidget rebuild   ← Từ widget này TRỞ XUỐNG
    ├── Text rebuild         ← Con cũng rebuild  
    ├── Button rebuild       ← Con cũng rebuild
    └── Icon rebuild         ← Con cũng rebuild

    OtherWidget             ← KHÔNG rebuild (không liên quan)
```

> 💡 **Scope:** `setState()` chỉ rebuild widget **chứa nó** và **tất cả con** bên dưới.

### 7.3 Lỗi thường gặp với setState()

#### ❌ Lỗi 1: Gọi setState() sau dispose()

```dart
// ❌ SAI — widget đã bị remove khỏi tree
Future<void> _fetchData() async {
  final data = await api.getData();
  setState(() {     // Lỗi! Widget có thể đã dispose rồi
    _data = data;
  });
}
```

```dart
// ✅ ĐÚNG — kiểm tra mounted trước
Future<void> _fetchData() async {
  final data = await api.getData();
  if (mounted) {                    // Kiểm tra widget còn sống không
    setState(() {
      _data = data;
    });
  }
}
```

#### ❌ Lỗi 2: Gọi setState() trong build()

```dart
// ❌ SAI — gây loop vô hạn: build → setState → build → setState...
@override
Widget build(BuildContext context) {
  setState(() { _count++; });  // ĐỪNG BAO GIỜ LÀM THẾ NÀY
  return Text('$_count');
}
```

#### ❌ Lỗi 3: Thay đổi state NGOÀI setState()

```dart
// ❌ SAI — state thay đổi nhưng UI không rebuild
void _increment() {
  _count++;  // State thay đổi nhưng Flutter không biết!
}
```

```dart
// ✅ ĐÚNG — bọc trong setState()
void _increment() {
  setState(() {
    _count++;
  });
}
```

#### ❌ Lỗi 4: Làm việc nặng trong setState()

```dart
// ❌ SAI — setState() nên chỉ thay đổi state, không làm compute nặng
setState(() {
  _items = heavyComputation();  // Tốn 500ms → UI lag
  _count = _items.length;
});
```

```dart
// ✅ ĐÚNG — compute trước, setState chỉ gán kết quả
final result = heavyComputation();
setState(() {
  _items = result;
  _count = _items.length;
});
```

> 🔗 **FE Bridge:** Tương tự `useState` setter trong React — nhưng **khác ở**: React `setCount(n)` chỉ re-render component đó, Flutter `setState()` gọi `markNeedsBuild()` → rebuild **TOÀN BỘ `build()` method** của widget + tất cả child widgets. Đây là lý do #1 cần tách widget nhỏ trong Flutter.

---

## 8. Common Widgets — Bộ công cụ hàng ngày 🟡

### 8.1 Layout & Container

| Widget | Mục đích | Properties thường dùng |
|--------|---------|----------------------|
| **Container** | Box model — padding, margin, decoration | `width`, `height`, `padding`, `margin`, `decoration`, `color` |
| **SizedBox** | Tạo khoảng trống hoặc fixed size | `width`, `height` |
| **Padding** | Thêm padding (chuyên dụng hơn Container) | `padding` |

```dart
Container(
  width: 200,
  height: 100,
  padding: const EdgeInsets.all(16),
  margin: const EdgeInsets.symmetric(horizontal: 8),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(color: Colors.black26, blurRadius: 4),
    ],
  ),
  child: const Text('Hello Container'),
)
```

### 8.2 Text & Display

| Widget | Mục đích | Properties thường dùng |
|--------|---------|----------------------|
| **Text** | Hiển thị text | `style`, `textAlign`, `maxLines`, `overflow` |
| **Icon** | Hiển thị icon | `Icons.xxx`, `size`, `color` |
| **Image** | Hiển thị hình ảnh | `Image.asset()`, `Image.network()`, `fit` |

```dart
const Text(
  'Hello Flutter!',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,  // "..." khi text quá dài
)
```

### 8.3 Interactive Widgets

| Widget | Mục đích | Properties thường dùng |
|--------|---------|----------------------|
| **ElevatedButton** | Nút bấm nổi | `onPressed`, `child`, `style` |
| **TextButton** | Nút bấm phẳng | `onPressed`, `child` |
| **IconButton** | Nút icon | `onPressed`, `icon` |
| **TextField** | Ô nhập text | `controller`, `decoration`, `onChanged` |

```dart
ElevatedButton(
  onPressed: () {
    debugPrint('Đã nhấn!');
  },
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue,
    padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: const Text('Nhấn tôi'),
)
```

```dart
TextField(
  controller: _textController,
  decoration: const InputDecoration(
    labelText: 'Nhập tên',
    hintText: 'Ví dụ: Nguyễn Văn A',
    border: OutlineInputBorder(),
    prefixIcon: Icon(Icons.person),
  ),
  onChanged: (value) {
    debugPrint('Đang nhập: $value');
  },
)
```

### 8.4 Card & ListTile — Combo phổ biến

```dart
Card(
  elevation: 4,
  margin: const EdgeInsets.all(8),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(12),
  ),
  child: const ListTile(
    leading: CircleAvatar(child: Icon(Icons.person)),
    title: Text('Nguyễn Văn A'),
    subtitle: Text('Flutter Developer'),
    trailing: Icon(Icons.arrow_forward_ios),
  ),
)
```

---

## 9. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

| # | Practice | Giải thích |
|---|---------|-----------|
| 1 | **Bắt đầu với StatelessWidget** | Chỉ chuyển sang Stateful khi thực sự cần |
| 2 | **Dùng `const` constructor** | `const Text('Hello')` — Flutter tái sử dụng, không tạo mới |
| 3 | **Tách widget nhỏ** | Thay vì 1 build() 200 dòng → tách thành nhiều widget con |
| 4 | **Cleanup trong dispose()** | Controller, Timer, Stream subscription → phải dispose |
| 5 | **Kiểm tra `mounted`** | Trước setState() sau async operation |
| 6 | **Dùng Key cho lists** | Khi list có thể reorder, add, remove |
| 7 | **Không làm nặng trong build()** | build() có thể gọi 60 lần/giây! |

### ❌ Lỗi thường gặp

| # | Lỗi | Hậu quả | Cách fix |
|---|-----|---------|---------|
| 1 | Không `dispose()` controller | Memory leak | Luôn dispose trong `dispose()` |
| 2 | `setState()` sau dispose | Crash | Check `mounted` trước |
| 3 | Dùng context sai scope | Widget not found error | Dùng Builder hoặc context đúng |
| 4 | Build() method quá dài | Khó maintain, performance kém | Tách widget con |
| 5 | Quên `super.initState()` | Undefined behavior | Luôn gọi super đầu tiên |
| 6 | setState() trong build() | Infinite loop | Chỉ gọi setState trong event handler |

---

## 10. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | React/Vue Mindset | Flutter Mindset | Tại sao khác |
|---|-------------------|-----------------|--------------|
| 1 | Component là instance, mutate props/state | Widget là immutable config, bị tạo mới mỗi build() | Flutter dùng Element Tree để giữ state, Widget chỉ là blueprint |
| 2 | Virtual DOM diff rồi patch minimal | Widget Tree diff qua Element Tree, rebuild subtree | Flutter có 3 layers thay vì 2, Element là layer trung gian |
| 3 | Context = cách truyền data xuống | BuildContext = danh tính vị trí + cách truy cập ancestor | BuildContext gắn chặt vào Element, không chỉ là data tunnel |
| 4 | Component unmount → cleanup effect | Widget dispose → State.dispose() → Element deactivate | Lifecycle gắn với State object, không phải Widget |
| 5 | `key` chỉ dùng khi render list | Key dùng rộng: preserve state, force rebuild, identify widget | Flutter Key liên kết Widget ↔ Element, không chỉ list optimization |

Nếu bạn đến từ React hoặc Vue, bảng này sẽ giúp bạn "map" kiến thức:

| Concept | Flutter | React | Vue |
|---------|---------|-------|-----|
| UI component | **Widget** | Component | Component |
| Static component | **StatelessWidget** | Functional Component (no state) | Component (no data) |
| Dynamic component | **StatefulWidget** | `useState` / Class Component | `ref()` / `reactive()` |
| Internal state | **State object + setState()** | `useState()` hook | `ref()` / `reactive()` |
| Rebuild trigger | **setState()** | State change → re-render | Reactivity system |
| Tree diffing | **Element Tree** so sánh Widget Tree | **Virtual DOM** diffing | **Virtual DOM** diffing |
| Access ancestor | **BuildContext + .of(context)** | ⛔ Không có tương đương trực tiếp (gần nhất: `useContext()`) | ⛔ Không có tương đương trực tiếp (gần nhất: `provide/inject`) |
| Component identity | **Key** | `key` prop | `:key` binding |
| Lifecycle: mount | **initState()** | `useEffect(() => {}, [])` | `onMounted()` |
| Lifecycle: cleanup | **dispose()** | `useEffect` return cleanup | `onUnmounted()` |
| Lifecycle: update | **didUpdateWidget()** | `useEffect` with deps | `watch()` |

### Sự khác biệt quan trọng

```
React: Virtual DOM → Real DOM (2 layers)
Flutter: Widget Tree → Element Tree → RenderObject Tree (3 layers)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
         Flutter có thêm 1 layer → granular hơn, nhanh hơn trong nhiều trường hợp
```

| | React | Flutter |
|---|---|---|
| Render target | DOM (browser) | Skia/Impeller (canvas) |
| Diffing | VDOM vs VDOM | Widget vs Widget (qua Element) |
| Identity hint | `key` prop | `Key` object |
| Khi nào cần key | List rendering | List rendering, form preservation |

> 💡 **Tóm lại:** Nếu bạn hiểu React's Virtual DOM, bạn đã hiểu 70% ý tưởng của Flutter's 3 trees. Điểm khác biệt lớn nhất: Flutter có **thêm 1 layer** (Element Tree) giữa configuration và rendering.

---

## 11. Tổng kết — Checklist kiến thức

### ✅ Sau buổi này, bạn phải trả lời được:

- [ ] Widget là gì? Tại sao Widget là immutable?
- [ ] StatelessWidget vs StatefulWidget — khi nào dùng cái nào?
- [ ] 3 trees là gì? Tại sao Flutter cần 3 trees thay vì 1?
- [ ] BuildContext là gì? Pattern `.of(context)` hoạt động ra sao?
- [ ] Key dùng để làm gì? Khi nào cần dùng Key?
- [ ] Lifecycle của StatefulWidget — thứ tự các method?
- [ ] `setState()` hoạt động ra sao? Phạm vi rebuild là gì?
- [ ] Kể tên 5 common widgets và mục đích sử dụng?

### 🧪 Quick test — Trả lời nhanh:

1. **Widget Tree** được tạo bởi ai? → _____
2. **Element Tree** có mutable không? → _____
3. **RenderObject Tree** làm gì? → _____
4. `initState()` gọi mấy lần? → _____
5. `dispose()` dùng để làm gì? → _____
6. `setState()` chỉ dùng trong ___Widget? → _____
7. `Theme.of(context)` tìm kiếm theo hướng nào trong tree? → _____

> 📝 **Đáp án:** 1. Developer (bạn) | 2. Có | 3. Layout + Paint | 4. 1 lần | 5. Cleanup resources | 6. Stateful | 7. Đi ngược lên (ancestor)

---

> **Tiếp theo:** Hãy chuyển sang [02-vi-du.md](./02-vi-du.md) để xem code minh hoạ cho từng concept.

---

### ➡️ Buổi tiếp theo

> **Buổi 04: Layout System** — Constraints model, Row/Column, Flex system, Scrollable widgets và responsive design.
>
> **Chuẩn bị:**
> - Hoàn thành BT1 + BT2 buổi này
> - Đọc trước về Flutter constraints: "Constraints go down, Sizes go up"
