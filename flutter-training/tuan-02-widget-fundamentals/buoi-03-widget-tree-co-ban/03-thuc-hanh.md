# Buổi 03: Widget Tree — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> Tự tay code mới hiểu sâu. Mỗi bài tập là một Flutter app hoàn chỉnh.

---

## Cách setup mỗi bài tập

```bash
# Tạo project mới cho mỗi bài
flutter create bt1_counter_app
cd bt1_counter_app

# Thay code trong lib/main.dart

# Chạy app
flutter run
```

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ React/Vue**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| React/Vue Habit | Flutter Reality | Bài tập liên quan |
|-----------------|-----------------|---------------------|
| Component giữ reference, re-render chỉ phần thay đổi | Widget bị tạo mới mỗi `build()` — phải tách widget nhỏ để tối ưu | BT1, BT2 |
| `useState` chỉ re-render component dùng state | `setState` rebuild TOÀN BỘ subtree → cần tách StatefulWidget nhỏ | BT1 |
| `useEffect` cleanup = component unmount | `dispose()` trong State class — lifecycle gắn với State, không phải Widget | BT1, BT2 |
| CSS cho styling, component cho logic | Mọi thứ đều là Widget — cả styling (`Padding`, `Container`) cũng là Widget | BT2, BT3 |
| Conditional render: `{show && <Component/>}` | Dùng if/else trong `build()` hoặc `Visibility` widget — KHÔNG có JSX conditional | BT2, BT3 |

---

## BT1 ⭐ Counter App — Nhập môn StatefulWidget 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_counter_app` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Counter app với nút tăng/giảm/reset |

### Yêu cầu

Tạo ứng dụng đếm số với các tính năng:

| # | Tính năng | Chi tiết |
|---|----------|---------|
| 1 | Hiển thị số đếm | Số lớn ở giữa màn hình |
| 2 | Nút tăng (+) | Nhấn → count + 1 |
| 3 | Nút giảm (-) | Nhấn → count - 1 (không xuống dưới 0) |
| 4 | Nút reset | Đưa count về 0 |
| 5 | Đổi màu | Count > 0: xanh, count == 0: đen, count < 0: đỏ (nếu cho phép âm) |

### Wireframe

```
┌──────────────────────────┐
│       Counter App    [⟲] │  ← AppBar với nút reset
│──────────────────────────│
│                          │
│                          │
│    Bạn đã nhấn nút:     │
│                          │
│         42               │  ← Số lớn, đổi màu theo giá trị
│                          │
│                          │
│    [ - ]    [ + ]        │  ← 2 FloatingActionButton
│                          │
└──────────────────────────┘
```

### Gợi ý (chỉ đọc khi cần)

<details>
<summary>💡 Gợi ý cấu trúc</summary>

```dart
class CounterApp extends StatefulWidget {
  // ...
}

class _CounterAppState extends State<CounterApp> {
  int _count = 0;

  void _increment() {
    setState(() { /* ... */ });
  }

  void _decrement() {
    setState(() { /* ... */ });
  }

  void _reset() {
    setState(() { /* ... */ });
  }

  Color _getCounterColor() {
    if (_count > 0) return Colors.green;
    if (_count < 0) return Colors.red;
    return Colors.black;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Counter App'),
        actions: [
          IconButton(onPressed: _reset, icon: const Icon(Icons.refresh)),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Text hiển thị count...
            // Row chứa 2 nút...
          ],
        ),
      ),
    );
  }
}
```

</details>

### Tiêu chí đánh giá

- [ ] App chạy không lỗi
- [ ] Nút tăng/giảm hoạt động đúng
- [ ] Count không xuống dưới 0
- [ ] Nút reset đưa về 0
- [ ] Đổi màu số theo giá trị
- [ ] Dùng `const` constructor cho widget tĩnh
- [ ] Code sạch, không logic thừa

---

## BT2 ⭐⭐ Todo List UI — Quản lý danh sách với State 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_todo_list` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Todo list với thêm/xoá/toggle hoàn thành |

### Yêu cầu

Tạo ứng dụng Todo List đơn giản (không cần persistent storage):

| # | Tính năng | Chi tiết |
|---|----------|---------|
| 1 | Hiển thị danh sách todo | Dùng `ListView.builder` |
| 2 | Thêm todo mới | TextField + nút thêm |
| 3 | Đánh dấu hoàn thành | Nhấn vào item → toggle (gạch ngang text) |
| 4 | Xoá todo | Swipe hoặc nút xoá |
| 5 | Hiển thị tổng | "3/5 hoàn thành" |

### Wireframe

```
┌──────────────────────────┐
│      Todo List           │
│──────────────────────────│
│ ┌──────────────────┐ [+] │  ← TextField + nút thêm
│ │ Nhập todo mới... │     │
│ └──────────────────┘     │
│──────────────────────────│
│ 2/4 hoàn thành           │  ← Tổng kết
│──────────────────────────│
│ ☑ Học Flutter        [🗑] │  ← Hoàn thành (gạch ngang)
│ ☐ Làm bài tập       [🗑] │  ← Chưa hoàn thành
│ ☑ Đọc tài liệu      [🗑] │
│ ☐ Code demo          [🗑] │
│                          │
└──────────────────────────┘
```

### Gợi ý (chỉ đọc khi cần)

<details>
<summary>💡 Gợi ý data model</summary>

```dart
class Todo {
  final String id;        // Unique identifier
  final String title;
  bool isDone;

  Todo({
    required this.id,
    required this.title,
    this.isDone = false,
  });
}
```

</details>

<details>
<summary>💡 Gợi ý cấu trúc State</summary>

```dart
class _TodoScreenState extends State<TodoScreen> {
  final List<Todo> _todos = [];
  final TextEditingController _textController = TextEditingController();

  void _addTodo() {
    final text = _textController.text.trim();
    if (text.isEmpty) return;

    setState(() {
      _todos.add(Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: text,
      ));
    });
    _textController.clear();
  }

  void _toggleTodo(int index) {
    setState(() {
      _todos[index].isDone = !_todos[index].isDone;
    });
  }

  void _removeTodo(int index) {
    setState(() {
      _todos.removeAt(index);
    });
  }

  int get _completedCount => _todos.where((t) => t.isDone).length;

  @override
  void dispose() {
    _textController.dispose();  // ⚠️ Đừng quên dispose!
    super.dispose();
  }

  // ...
}
```

</details>

<details>
<summary>💡 Gợi ý ListView với Key</summary>

```dart
ListView.builder(
  itemCount: _todos.length,
  itemBuilder: (context, index) {
    final todo = _todos[index];
    return Dismissible(
      key: ValueKey(todo.id),  // ← Key quan trọng cho list!
      onDismissed: (_) => _removeTodo(index),
      background: Container(
        color: Colors.red,
        alignment: Alignment.centerRight,
        padding: const EdgeInsets.only(right: 16),
        child: const Icon(Icons.delete, color: Colors.white),
      ),
      child: CheckboxListTile(
        value: todo.isDone,
        onChanged: (_) => _toggleTodo(index),
        title: Text(
          todo.title,
          style: TextStyle(
            decoration: todo.isDone ? TextDecoration.lineThrough : null,
            color: todo.isDone ? Colors.grey : null,
          ),
        ),
      ),
    );
  },
)
```

</details>

### Tiêu chí đánh giá

- [ ] Thêm todo mới hoạt động (enter hoặc nhấn nút)
- [ ] Toggle hoàn thành/chưa hoàn thành
- [ ] Xoá todo bằng swipe hoặc nút xoá
- [ ] Hiển thị "X/Y hoàn thành"
- [ ] TextField có `controller` và `dispose()` đúng
- [ ] Dùng `ValueKey` cho list items
- [ ] Không cho thêm todo rỗng
- [ ] UI clean, có AppBar

---

## BT3 ⭐⭐⭐ Custom Widget Composition — UserCard tái sử dụng 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_user_card` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — User directory với expandable cards và follow button |

### Yêu cầu

Tạo widget `UserCard` tái sử dụng, hiển thị thông tin user:

| # | Tính năng | Chi tiết |
|---|----------|---------|
| 1 | UserCard widget | StatelessWidget, nhận User data qua constructor |
| 2 | Expandable detail | Nhấn vào card → expand/collapse chi tiết (StatefulWidget) |
| 3 | Action buttons | Nút "Follow" có toggle state (Following/Follow) |
| 4 | Nhiều instances | Hiển thị 5+ UserCard với data khác nhau |
| 5 | Composition pattern | Tách nhỏ: `UserAvatar`, `UserInfo`, `FollowButton` |

### Wireframe

```
┌──────────────────────────────────┐
│          User Directory          │
│──────────────────────────────────│
│ ┌──────────────────────────────┐ │
│ │ (A)  Nguyễn Văn A            │ │  ← UserCard (collapsed)
│ │      Flutter Dev   [Follow]  │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ (B)  Trần Thị B              │ │  ← UserCard (expanded)
│ │      iOS Developer [Following]│ │
│ │ ─────────────────────────── │ │
│ │ 📧 b@email.com              │ │  ← Expanded detail
│ │ 📱 0901234567               │ │
│ │ 📍 Hà Nội                   │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ (C)  Lê Văn C                │ │
│ │      Backend Dev    [Follow] │ │
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

### Gợi ý (chỉ đọc khi cần)

<details>
<summary>💡 Gợi ý cấu trúc widget</summary>

```
UserDirectoryScreen (StatelessWidget)
 └── ListView
      ├── UserCard (StatefulWidget — vì có expand + follow state)
      │    ├── UserAvatar (StatelessWidget)
      │    ├── UserInfo (StatelessWidget)
      │    ├── FollowButton (StatefulWidget — vì toggle state)
      │    └── UserDetail (StatelessWidget — chỉ hiện khi expanded)
      ├── UserCard
      └── UserCard
```

</details>

<details>
<summary>💡 Gợi ý data model</summary>

```dart
class User {
  final String id;
  final String name;
  final String role;
  final String email;
  final String phone;
  final String location;
  final Color avatarColor;

  const User({
    required this.id,
    required this.name,
    required this.role,
    required this.email,
    required this.phone,
    required this.location,
    required this.avatarColor,
  });
}

// Sample data
final sampleUsers = [
  const User(
    id: '1',
    name: 'Nguyễn Văn A',
    role: 'Flutter Developer',
    email: 'a@nals.vn',
    phone: '0901234567',
    location: 'Đà Nẵng',
    avatarColor: Colors.blue,
  ),
  // ... thêm users
];
```

</details>

<details>
<summary>💡 Gợi ý widget composition</summary>

```dart
/// Widget avatar riêng — StatelessWidget
class UserAvatar extends StatelessWidget {
  final String name;
  final Color color;

  const UserAvatar({super.key, required this.name, required this.color});

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      backgroundColor: color,
      radius: 24,
      child: Text(
        name[0].toUpperCase(),
        style: const TextStyle(color: Colors.white, fontSize: 20),
      ),
    );
  }
}

/// Widget thông tin riêng — StatelessWidget
class UserInfo extends StatelessWidget {
  final String name;
  final String role;

  const UserInfo({super.key, required this.name, required this.role});

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(name, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
        Text(role, style: TextStyle(color: Colors.grey[600])),
      ],
    );
  }
}

/// Nút Follow — StatefulWidget vì có toggle state
class FollowButton extends StatefulWidget {
  const FollowButton({super.key});

  @override
  State<FollowButton> createState() => _FollowButtonState();
}

class _FollowButtonState extends State<FollowButton> {
  bool _isFollowing = false;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        setState(() { _isFollowing = !_isFollowing; });
      },
      style: ElevatedButton.styleFrom(
        backgroundColor: _isFollowing ? Colors.grey : Colors.blue,
      ),
      child: Text(_isFollowing ? 'Following' : 'Follow'),
    );
  }
}
```

</details>

### Tiêu chí đánh giá

- [ ] Widget composition rõ ràng: UserAvatar, UserInfo, FollowButton là widget riêng
- [ ] Expand/collapse hoạt động
- [ ] Follow/Following toggle đúng
- [ ] Mỗi UserCard hoạt động độc lập (follow card A không ảnh hưởng card B)
- [ ] Dùng `const` constructor khi có thể
- [ ] Dùng `ValueKey` cho list items
- [ ] Code clean, tách file nếu cần
- [ ] `dispose()` controllers nếu có

---

## 🤔 Câu hỏi thảo luận

Suy nghĩ và ghi lại câu trả lời. Thảo luận với nhóm hoặc mentor.

### Câu hỏi 1: StatelessWidget hay StatefulWidget?

Cho các widget sau, bạn chọn StatelessWidget hay StatefulWidget? Giải thích tại sao.

| Widget | Mô tả | Chọn? | Tại sao? |
|--------|-------|-------|---------|
| A | Header hiển thị logo + app name | _____ | _____ |
| B | Form đăng nhập (email + password) | _____ | _____ |
| C | Card hiển thị thông tin sản phẩm (từ API) | _____ | _____ |
| D | Bottom navigation bar (đổi tab) | _____ | _____ |
| E | Loading spinner | _____ | _____ |
| F | Ô nhập tìm kiếm với debounce | _____ | _____ |

<details>
<summary>💡 Đáp án tham khảo</summary>

| Widget | Chọn | Giải thích |
|--------|------|-----------|
| A | **Stateless** | Chỉ hiển thị data tĩnh, không thay đổi |
| B | **Stateful** | Cần quản lý TextEditingController, validate state |
| C | **Stateless** | Nếu data truyền từ parent → Stateless. Parent lo việc fetch |
| D | **Stateful** | Cần track tab nào đang active |
| E | **Stateless** | AnimationController nằm trong AnimatedWidget, spinner chỉ hiển thị |
| F | **Stateful** | Cần Timer cho debounce, TextEditingController |

</details>

### Câu hỏi 2: Three Trees & Performance

Giả sử app có 500 widget trên màn hình. User nhấn nút "Like" → chỉ icon ❤️ đổi màu.

1. Widget Tree có bao nhiêu widget được tạo lại?
2. Element Tree có bao nhiêu element bị thay đổi?
3. RenderObject Tree có bao nhiêu render object được repaint?
4. Tại sao Flutter vẫn nhanh dù tạo lại toàn bộ Widget Tree?

<details>
<summary>💡 Đáp án tham khảo</summary>

1. **Widget Tree**: Toàn bộ subtree chứa nút Like bị tạo lại (widget nhẹ, rẻ)
2. **Element Tree**: Phần lớn Element được **tái sử dụng** (canUpdate = true). Chỉ 1-2 element cần update
3. **RenderObject Tree**: Chỉ **1** RenderObject (icon) được repaint
4. Vì Widget chỉ là object nhẹ (configuration). Tạo 500 widget ≈ tạo 500 plain object → rất nhanh. Phần đắt (render) chỉ update khi thực sự cần.

</details>

### Câu hỏi 3: BuildContext — Debug lỗi

Xem đoạn code sau, tìm lỗi:

```dart
class MyScreen extends StatelessWidget {
  const MyScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Hiển thị SnackBar
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(content: Text('Hello!')),
            );
          },
          child: const Text('Show SnackBar'),
        ),
      ),
    );
  }
}
```

1. Code này có chạy được không?
2. Nếu `Scaffold` nằm ở widget khác (không phải trong cùng `build()`), liệu có lỗi không?
3. Khi nào pattern này GÂY LỖI? Và cách fix?

<details>
<summary>💡 Đáp án tham khảo</summary>

1. **Chạy được** trong trường hợp này vì `ScaffoldMessenger` được cung cấp bởi `MaterialApp` (ancestor), không phải `Scaffold` trong cùng build. `ScaffoldMessenger.of(context)` tìm `ScaffoldMessenger` gần nhất phía trên.

2. **Không lỗi** nếu có `MaterialApp` (hoặc `ScaffoldMessenger`) ở ancestor. `ScaffoldMessenger` khác với `Scaffold.of()`.

3. **Gây lỗi** nếu không có `MaterialApp` hoặc `ScaffoldMessenger` ở phía trên trong tree. Fix: đảm bảo widget nằm trong MaterialApp, hoặc dùng `Builder` nếu cần context bên trong Scaffold.

</details>

---

## 📝 Checklist sau khi hoàn thành

- [ ] BT1: Counter app chạy đúng, có tăng/giảm/reset/đổi màu
- [ ] BT2: Todo list thêm/xoá/toggle/hiển thị tổng
- [ ] BT3: UserCard composition với expand/collapse + follow toggle
- [ ] Trả lời được 3 câu hỏi thảo luận
- [ ] Đã `dispose()` tất cả controllers
- [ ] Đã dùng `Key` cho list items
- [ ] Hiểu lifecycle qua debug log

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 2:** Focus vào verify lifecycle correctness và widget composition.

### AI-BT1: Gen StatefulWidget Counter + Verify Lifecycle ⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** StatefulWidget lifecycle — `createState`, `initState`, `setState`, `dispose`, `mounted`.
- **Task thực tế:** Tạo widget Stopwatch (đồng hồ bấm giờ) — component hay gặp trong app fitness/cooking. Cần Timer chạy liên tục update UI mỗi 100ms, pause/resume/reset, và cleanup khi navigate away.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần tạo Flutter widget StopwatchWidget — đồng hồ bấm giờ với Start/Pause/Reset.
Context: app fitness tracker, widget nằm trong tab có thể bị dispose khi chuyển tab.
Tech stack: Flutter 3.x, Dart 3.x.
Constraints:
- Dùng StatefulWidget, Timer.periodic trong state.
- Hiển thị format MM:SS.ms (phút:giây.mili giây).
- 3 nút: Start, Pause, Reset — chỉ hiện nút phù hợp với trạng thái hiện tại.
- Timer PHẢI cancel trong dispose() để tránh memory leak.
- setState PHẢI check mounted trước khi gọi.
- const constructor cho những widget static (Icon, Text label cố định).
- Tách logic ra method riêng: _startTimer(), _pauseTimer(), _resetTimer().
Output: 1 file stopwatch_widget.dart hoàn chỉnh, tự chứa.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 1 file `stopwatch_widget.dart` với `StatefulWidget`, `Timer` biến private, `initState` (có thể trống), `dispose` cancel timer, `build` với Row/Column chứa Text + 3 nút.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | `_timer?.cancel()` có trong `dispose()` TRƯỚC `super.dispose()`? | ☐ |
| 2 | `setState` có được bọc trong `if (mounted)`? | ☐ |
| 3 | Timer cancel khi nhấn Pause (không chỉ trong dispose)? | ☐ |
| 4 | Reset cũng cancel timer cũ trước khi có thể start lại? | ☐ |
| 5 | `const` dùng cho Icon và Text label cố định? | ☐ |
| 6 | Không có logic async trực tiếp trong `initState`? | ☐ |
| 7 | `flutter analyze` không có warning? | ☐ |

**4. Customize:**
Tự thêm: "Lap" feature — nút Lap ghi lại thời gian hiện tại vào List<Duration>, hiển thị danh sách laps bên dưới stopwatch. AI chưa làm phần này. Implement thêm `_laps` list + `_addLap()` method + ListView hiển thị.

---

> **Tiếp theo:** Xem [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md) để tìm thêm tài liệu học.
