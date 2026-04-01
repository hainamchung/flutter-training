# Buổi 13: Performance Optimization — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 13/16** · **Thời lượng tự học:** ~2 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 12 (toàn bộ lý thuyết + bài tập)

## 1. Flutter Rendering Pipeline 🔴

### 1.1 Ba giai đoạn render

Flutter render mỗi frame qua 3 giai đoạn tuần tự:

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌───────────┐     ┌──────────┐
│  BUILD   │────▶│  LAYOUT  │────▶│  PAINT   │────▶│ COMPOSITE │────▶│RASTERIZE │
│          │     │          │     │          │     │           │     │          │
│ Widget   │     │ Tính size│     │ Vẽ pixel │     │ Merge     │     │ GPU      │
│ Tree →   │     │ & vị trí │     │ lên      │     │ layers    │     │ render   │
│ Element  │     │ từng     │     │ canvas   │     │ thành     │     │ ra       │
│ Tree     │     │ RenderObj│     │          │     │ scene     │     │ pixels   │
└──────────┘     └──────────┘     └──────────┘     └───────────┘     └──────────┘
```

**Build phase:**
- Flutter so sánh Widget tree mới với Element tree hiện tại
- Quyết định Element nào cần update
- Tạo/cập nhật RenderObject tương ứng

**Layout phase:**
- Duyệt RenderObject tree từ trên xuống (top-down constraints)
- Mỗi RenderObject tính toán size dựa trên constraints từ parent
- Trả size về cho parent (bottom-up sizes)

**Paint phase:**
- Duyệt RenderObject tree, vẽ từng object lên Layer tree
- Layer tree được gửi đến engine (Skia hoặc Impeller) để rasterize

**Composite phase:**
- Sau khi paint, Flutter gộp (composite) các Layer outputs thành một **Scene** duy nhất
- Scene này là biểu diễn cuối cùng của frame, sẵn sàng gửi cho GPU
- Phase này xử lý việc merge các layer (bao gồm RepaintBoundary layers, Opacity layers, Transform layers) theo đúng thứ tự z-index

**Rasterize phase:**
- Engine (Skia/Impeller) nhận Scene và chuyển thành pixels trên GPU
- Đây là bước cuối cùng trước khi hiển thị lên màn hình

> 💡 **Impeller**: Từ Flutter 3.16+, **Impeller** là rendering engine mặc định trên iOS (thay thế Skia). Impeller pre-compile tất cả shader lúc build, loại bỏ hoàn toàn **shader compilation jank** — vấn đề phổ biến nhất gây frame drop lần đầu trên iOS. Trên Android, Impeller đang ở giai đoạn preview (opt-in via `--enable-impeller`).

### 1.2 Ba loại Tree

```
Widget Tree          Element Tree         RenderObject Tree
(Blueprint)          (Instantiation)      (Layout & Paint)
┌─────────┐         ┌─────────┐          ┌─────────┐
│MaterialApp│───────▶│Element  │─────────▶│RenderObj│
├─────────┤         ├─────────┤          ├─────────┤
│ Scaffold │───────▶│Element  │─────────▶│RenderObj│
├─────────┤         ├─────────┤          ├─────────┤
│  Column  │───────▶│Element  │─────────▶│RenderObj│
├─────────┤         ├─────────┤          ├─────────┤
│  Text    │───────▶│Element  │─────────▶│RenderObj│
└─────────┘         └─────────┘          └─────────┘

Immutable            Mutable              Mutable
(Tạo mới khi         (Được reuse          (Thực hiện
 rebuild)             nếu có thể)          layout/paint)
```

> **Quan trọng:** Widget được tạo mới mỗi lần rebuild, nhưng Element và RenderObject được **reuse** nếu Widget type và key không đổi. Đây là cơ chế tối ưu cốt lõi của Flutter.

### 1.3 Frame Budget

```
60 FPS: 1000ms / 60 = 16.67ms mỗi frame
        ┌───────────────────────────────┐
        │      16.67ms frame budget      │
        ├──────────┬────────┬───────────┤
        │  Build   │ Layout │   Paint   │
        │  ~4ms    │ ~4ms   │  ~4ms     │  ← Lý tưởng
        └──────────┴────────┴───────────┘
        (Còn ~4ms buffer cho engine rasterize)

120 FPS: 1000ms / 120 = 8.33ms mỗi frame  ← iPhone ProMotion, flagship Android
        ┌───────────────────┐
        │  8.33ms budget!   │  ← Khắt khe hơn nhiều
        └───────────────────┘
```

Mỗi frame, Flutter cần hoàn thành cả 3 phase trong budget. Nếu vượt quá → **dropped frame** → **jank**.

### 1.4 VSync

**VSync (Vertical Sync):** Tín hiệu đồng bộ từ display hardware, báo cho Flutter biết khi nào nên bắt đầu render frame tiếp theo.

```
VSync ──┤    ├────┤    ├────┤    ├────┤
Frame   │ F1 │    │ F2 │    │ F3 │    │
        └────┘    └────┘    └────┘
        16ms      16ms      16ms
```

Flutter sử dụng `SchedulerBinding` để lắng nghe vsync signal và trigger build.

### 1.5 💡 So sánh với React/Vue

| Khía cạnh | React/Vue | Flutter |
|-----------|-----------|---------|
| Rendering | Virtual DOM → DOM diffing → Browser paint | Widget → Element → RenderObject → Skia/Impeller |
| Diffing | Virtual DOM diff (reconciliation) | Element tree reuse (canUpdate check) |
| Engine | Dùng browser engine (V8 + Blink) | Rendering engine riêng (Skia/Impeller) |
| Frame control | Browser quản lý (`requestAnimationFrame`) | Flutter tự quản lý (vsync + SchedulerBinding) |
| Layout | CSS Box Model (browser tính) | RenderObject constraints (Flutter tự tính) |

> **Key insight:** Flutter **không có** Virtual DOM. Thay vào đó, nó có 3 trees riêng biệt. Widget tree là "blueprint" (tương tự JSX), Element tree là "instance" (tương tự Fiber node), RenderObject tree thực hiện layout/paint (tương tự DOM node).

---

## 2. FPS & Jank Detection 🔴

### 2.1 Jank là gì?

**Jank** = khi app không kịp render frame trong budget → frame bị bỏ qua → người dùng thấy giật, lag.

```
Bình thường (60fps mượt):
Frame: │F1│F2│F3│F4│F5│F6│F7│F8│
       ├──┼──┼──┼──┼──┼──┼──┼──┤
       16 16 16 16 16 16 16 16  (ms)

Jank (frame bị drop):
Frame: │F1│F2│  F3 (chậm)  │F5│F6│F7│
       ├──┼──┼──────────────┼──┼──┼──┤
       16 16      48ms       16 16 16
              ↑
        F4 bị drop! User thấy giật
```

### 2.2 Mức FPS mục tiêu

| Thiết bị | FPS | Frame budget |
|----------|-----|-------------|
| Hầu hết Android | 60 fps | 16.67ms |
| Flagship Android (120Hz) | 120 fps | 8.33ms |
| iPhone 13 Pro+ (ProMotion) | 120 fps | 8.33ms |
| iPad Pro (ProMotion) | 120 fps | 8.33ms |

### 2.3 Performance Overlay

Bật nhanh để xem realtime FPS:

```dart
// Cách 1: Trong MaterialApp
MaterialApp(
  showPerformanceOverlay: true, // Hiển thị GPU & UI thread timing
  home: MyHomePage(),
);

// Cách 2: Từ DevTools
// Run app ở profile mode: flutter run --profile
// Mở DevTools → Performance → Toggle Performance Overlay
```

```
┌─────────────────────────────┐
│ Performance Overlay          │
│ ┌─────────────────────────┐ │
│ │ ▓▓▓▓░░░░░░░░░░░░ UI    │ │  ← Thanh xanh = OK
│ │ ▓▓░░░░░░░░░░░░░░ Raster│ │  ← Thanh đỏ = vượt budget!
│ └─────────────────────────┘ │
│ Đường kẻ = 16ms budget line │
└─────────────────────────────┘
```

> **Lưu ý:** Luôn profile ở **profile mode** (`flutter run --profile`), KHÔNG phải debug mode. Debug mode chậm hơn nhiều do assertions, debug checks.

### 2.4 Nguyên nhân jank phổ biến

```
🔴 Heavy build() method
   └── Tính toán phức tạp trong build()
   └── Tạo nhiều object không cần thiết
   └── Build lại toàn bộ widget tree

🔴 Synchronous I/O trên main thread
   └── Đọc file lớn
   └── Parse JSON lớn
   └── Image processing

🔴 Large images
   └── Load ảnh resolution cao không resize
   └── Decode ảnh trên main thread
   └── Không cache ảnh

🔴 Complex layouts
   └── Nested ListView (đặc biệt không có shrinkWrap)
   └── Intrinsic dimensions (IntrinsicWidth/Height)
   └── saveLayer (ClipRRect, Opacity, ShaderMask)
```

---

## 3. Rebuild Optimization 🔴

### 3.1 const Constructors — Vũ khí số 1

**`const` constructor** là cách tối ưu quan trọng nhất trong Flutter. Widget được đánh dấu `const` sẽ **KHÔNG bao giờ rebuild** khi parent rebuild.

```dart
// ❌ KHÔNG const — rebuild mỗi lần parent rebuild
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('Hello World'),      // Tạo mới mỗi lần build
      Icon(Icons.star),          // Tạo mới mỗi lần build
      SizedBox(height: 16),     // Tạo mới mỗi lần build
    ],
  );
}

// ✅ const — KHÔNG rebuild khi parent rebuild
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Hello World'),      // Reuse instance cũ
      const Icon(Icons.star),          // Reuse instance cũ
      const SizedBox(height: 16),     // Reuse instance cũ
    ],
  );
}
```

**Tại sao `const` hiệu quả?**
- `const` Widget được tạo tại **compile-time** → cùng 1 instance trong memory
- Flutter so sánh bằng `identical()` (so sánh reference) → O(1)
- Nếu Widget giống hệt → Element được reuse → KHÔNG trigger layout/paint

> **Từ React/Vue:** `const` Widget ≈ `React.memo()` với shallow compare, nhưng mạnh hơn vì nó là **compile-time guarantee**, không phải runtime check.

### 3.2 Widget Splitting — Granular Widgets

Tách widget lớn thành widget nhỏ để giới hạn phạm vi rebuild:

```dart
// ❌ Mega-widget: toàn bộ rebuild khi counter thay đổi
class MyHomePage extends StatefulWidget {
  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Demo')),
      body: Column(
        children: [
          const Text('Static header'),          // Rebuild không cần thiết!
          const Icon(Icons.star, size: 100),    // Rebuild không cần thiết!
          Text('Count: $_counter'),              // Chỉ phần này cần rebuild
          Image.network('https://...'),          // Rebuild không cần thiết!
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _counter++),
        child: const Icon(Icons.add),
      ),
    );
  }
}

// ✅ Split widget: chỉ CounterDisplay rebuild
class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Demo')),
      body: const Column(
        children: [
          Text('Static header'),
          Icon(Icons.star, size: 100),
          CounterDisplay(),    // Chỉ widget này quản lý state riêng
          CachedImage(),       // Widget riêng, rebuild riêng
        ],
      ),
    );
  }
}

class CounterDisplay extends StatefulWidget {
  const CounterDisplay({super.key});

  @override
  State<CounterDisplay> createState() => _CounterDisplayState();
}

class _CounterDisplayState extends State<CounterDisplay> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    // Chỉ widget nhỏ này rebuild khi _counter thay đổi
    return TextButton(
      onPressed: () => setState(() => _counter++),
      child: Text('Count: $_counter'),
    );
  }
}
```

### 3.3 RepaintBoundary

`RepaintBoundary` tạo một layer riêng, cách ly phần paint tốn kém:

```dart
// ❌ Animation repaint toàn bộ screen
Stack(
  children: [
    ComplexBackground(),     // Bị repaint mỗi frame do animation!
    AnimatedWidget(),        // Animation 60fps
    StaticFooter(),          // Bị repaint mỗi frame do animation!
  ],
)

// ✅ Cách ly animation trong RepaintBoundary
Stack(
  children: [
    const ComplexBackground(),
    RepaintBoundary(          // Layer riêng cho animation
      child: AnimatedWidget(),
    ),
    const StaticFooter(),
  ],
)
```

**Khi nào dùng RepaintBoundary:**
- Animation liên tục bên cạnh static content
- Custom paint phức tạp
- Phần của screen thay đổi thường xuyên

**Khi nào KHÔNG dùng:**
- Mọi nơi (overhead tạo layer > lợi ích)
- Widget đơn giản, ít paint cost

### 3.4 ListView.builder vs ListView

```dart
// ❌ ListView: tạo TẤT CẢ 10,000 items cùng lúc
ListView(
  children: List.generate(10000, (i) => ListTile(title: Text('Item $i'))),
  // → Tạo 10,000 Widget + Element + RenderObject → chậm, tốn RAM
)

// ✅ ListView.builder: chỉ tạo items visible trên screen
ListView.builder(
  itemCount: 10000,
  itemBuilder: (context, index) {
    return ListTile(title: Text('Item $index'));
  },
  // → Chỉ tạo ~15-20 items visible → nhanh, tiết kiệm RAM
)
```

```
ListView (10,000 items):
┌──────────────────┐
│ Item 0           │ ← Visible
│ Item 1           │ ← Visible
│ Item 2           │ ← Visible
│ ─ ─ ─ ─ ─ ─ ─ ─│ ← Screen edge
│ Item 3           │ ← Built nhưng KHÔNG visible!
│ Item 4           │ ← Built nhưng KHÔNG visible!
│ ...              │
│ Item 9999        │ ← Built nhưng KHÔNG visible!
└──────────────────┘
10,000 widgets trong memory 😱

ListView.builder (10,000 items):
┌──────────────────┐
│ Item 0           │ ← Built & Visible
│ Item 1           │ ← Built & Visible
│ Item 2           │ ← Built & Visible
│ ─ ─ ─ ─ ─ ─ ─ ─│ ← Screen edge
│ (cache vùng đệm)│ ← Vài items buffer
└──────────────────┘
~20 widgets trong memory ✅
```

> **Từ React/Vue:** `ListView.builder` ≈ `react-window` hoặc `vue-virtual-scroller`. Cùng concept: chỉ render items trong viewport.

### 3.5 ValueListenableBuilder & AnimatedBuilder

Dùng builder widgets để chỉ rebuild phần cần thiết:

```dart
// ❌ setState rebuild toàn bộ
class _MyPageState extends State<MyPage> {
  final _counter = ValueNotifier<int>(0);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ExpensiveWidget(),      // Bị rebuild!
        Text('${_counter.value}'),    // Cần rebuild
        const AnotherExpensiveWidget(), // Bị rebuild!
      ],
    );
  }
}

// ✅ ValueListenableBuilder: chỉ rebuild Text
class _MyPageState extends State<MyPage> {
  final _counter = ValueNotifier<int>(0);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ExpensiveWidget(),          // KHÔNG rebuild ✅
        ValueListenableBuilder<int>(
          valueListenable: _counter,
          builder: (context, value, child) {
            return Text('$value');         // Chỉ phần này rebuild
          },
        ),
        const AnotherExpensiveWidget(),   // KHÔNG rebuild ✅
      ],
    );
  }

  @override
  void dispose() {
    _counter.dispose();
    super.dispose();
  }
}
```

### 3.6 Keys — Preserving State

Keys giúp Flutter identify đúng Element khi Widget tree thay đổi:

```dart
// ❌ Không có key: Flutter có thể reuse sai Element khi reorder
ListView(
  children: [
    for (final item in items)
      ItemWidget(item: item),  // Reorder → sai state!
  ],
)

// ✅ Có key: Flutter match đúng Element
ListView(
  children: [
    for (final item in items)
      ItemWidget(key: ValueKey(item.id), item: item),
  ],
)
```

| Key type | Khi nào dùng |
|----------|-------------|
| `ValueKey` | Khi item có unique value (id, email) |
| `ObjectKey` | Khi item object là unique |
| `UniqueKey` | Khi cần force rebuild (dùng ít) |
| `GlobalKey` | Khi cần access state từ bên ngoài (dùng hạn chế) |

> 🔗 **FE Bridge:** `const` widget ≈ `React.memo()` — skip rebuild khi props không đổi. `Key` ≈ React `key` prop — giúp reconciliation. Nhưng **khác ở**: Flutter `const` = compile-time constant (mạnh hơn memo), và **thiếu key** gây bug rebuild giống React nhưng khó detect hơn.

### 3.7 BLoC: buildWhen / listenWhen

```dart
// ❌ Rebuild khi BẤT KỲ state nào thay đổi
BlocBuilder<ProfileBloc, ProfileState>(
  builder: (context, state) {
    return Text(state.name);  // Rebuild cả khi state.avatar thay đổi!
  },
)

// ✅ Chỉ rebuild khi name thay đổi
BlocBuilder<ProfileBloc, ProfileState>(
  buildWhen: (previous, current) => previous.name != current.name,
  builder: (context, state) {
    return Text(state.name);  // Chỉ rebuild khi name thực sự đổi
  },
)
```

### 3.8 Selector trong Provider / Riverpod

```dart
// Riverpod: select chỉ field cần thiết
// ❌ Rebuild khi BẤT KỲ field nào trong UserState thay đổi
final userState = ref.watch(userProvider);
Text(userState.name);

// ✅ Chỉ rebuild khi name thay đổi
final name = ref.watch(userProvider.select((state) => state.name));
Text(name);
```

---

> 💼 **Gặp trong dự án:** App giật khi scroll danh sách dài, UI freeze khi load data, rebuild quá nhiều widget không cần thiết, DevTools flamechart analysis
> 🤖 **Keywords bắt buộc trong prompt:** `const constructor`, `widget splitting`, `RepaintBoundary`, `ListView.builder`, `shouldRebuild`, `ValueListenableBuilder`, `Selector`, `DevTools timeline`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **App giật:** QA report "scroll lag" trên danh sách 5000 items — cần profiling + fix
- **Rebuild over:** Senior review thấy parent widget setState → toàn bộ subtree rebuild (300+ widgets)
- **DevTools confusion:** Junior không biết đọc flamechart, không biết widget nào gây jank

**Tại sao cần các keyword trên:**
- **`const constructor`** — widget immutable, Flutter skip rebuild (zero cost), AI hay thiếu `const`
- **`widget splitting`** — tách widget lớn thành nhiều widgets nhỏ, `setState` chỉ rebuild phần cần
- **`RepaintBoundary`** — isolate paint region, tránh repaint cascading
- **`ListView.builder`** — lazy rendering, chỉ build visible items (AI hay dùng `ListView(children:)` = build ALL)
- **`DevTools timeline`** — trace build/layout/paint phases, tìm bottleneck

**Prompt mẫu — Performance Audit:**
```text
Tôi cần audit performance cho Flutter screen có danh sách 5000 contacts.
Hiện tại: scroll giật, FPS drop xuống 20fps trên Android mid-range.
Code hiện tại:
[paste widget code]

Yêu cầu:
1. Identify: tất cả unnecessary rebuilds (setState scope quá rộng, thiếu const).
2. List: widgets nào PHẢI thêm const constructor.
3. Suggest: widget splitting strategy — tách StatefulWidget nào ra.
4. Recommend: RepaintBoundary positions (đâu cần, đâu không cần).
5. Check: ListView dùng .builder? itemExtent set? cacheExtent hợp lý?
6. Bonus: DevTools profiling steps — record timeline, đọc flamechart, tìm bottleneck.
Output: Annotated code + optimization checklist + DevTools guide.
```

**Expected Output:** AI gen annotated code + optimization checklist.

⚠️ **Giới hạn AI hay mắc:** AI hay suggest `RepaintBoundary` everywhere (overhead! chỉ dùng cho expensive paint operations). AI cũng hay nói "dùng const" nhưng không chỉ ra cụ thể widget NÀO cần const. AI hay thiếu `itemExtent` cho ListView.builder (quan trọng cho scroll performance).

</details>

---

## 4. Flutter DevTools 🔴

### 4.1 Tổng quan

Flutter DevTools là bộ công cụ profiling và debugging:

```
┌────────────────────────────────────────────────────────┐
│                    Flutter DevTools                      │
├──────────┬───────────┬────────┬────────┬───────────────┤
│ Inspector│Performance│  CPU   │ Memory │   Network     │
│          │           │Profiler│        │               │
├──────────┼───────────┼────────┼────────┼───────────────┤
│ Widget   │ Frame     │Function│Snapshot│ HTTP request  │
│ tree     │ chart     │ time   │& alloc │ monitoring    │
│ Rebuild  │ Timeline  │ Call   │ Leak   │ Payload size  │
│ counts   │ Jank ID   │ stack  │ detect │ Timing        │
└──────────┴───────────┴────────┴────────┴───────────────┘
```

### 4.2 Cách khởi chạy

```bash
# Cách 1: Từ terminal
flutter run --profile
# → DevTools URL hiển thị trong terminal output
# → Click URL hoặc copy vào browser

# Cách 2: Từ VS Code
# Run app → Click "Open DevTools" trong Debug sidebar

# Cách 3: Từ Android Studio
# Run app → Click "Open DevTools" trên toolbar
```

> **Quan trọng:** Luôn dùng `--profile` mode. Debug mode KHÔNG phản ánh performance thực tế.

### 4.3 Widget Inspector

```
Widget Inspector:
┌──────────────────────────────────────────────┐
│ Widget Tree                 │ Details        │
│ ├─ MaterialApp              │                │
│ │  ├─ Scaffold              │ renderObject:  │
│ │  │  ├─ AppBar             │  RenderFlex    │
│ │  │  ├─ Column             │  size: 360x640 │
│ │  │  │  ├─ Text ⟳ 3       │  constraints:  │
│ │  │  │  └─ Image ⟳ 1      │   0≤w≤360     │
│ │  │  └─ FAB               │   0≤h≤640     │
│                              │                │
│ ⟳ = rebuild count            │ Rebuild count: │
│ (toggle trong DevTools)      │  Text: 3 lần   │
└──────────────────────────────────────────────┘
```

**Bật rebuild tracking:**
1. Mở DevTools → Widget Inspector
2. Click icon "Track Widget Rebuilds" (biểu tượng refresh)
3. Thao tác với app → quan sát rebuild count trên mỗi widget

### 4.4 Performance View

```
Performance View:
┌──────────────────────────────────────────────┐
│ Frame Chart:                                  │
│ ██ ██ ██ ██ ██████ ██ ██ ██ ██ ██ ██ ██     │
│ ── ── ── ── ────── ── ── ── ── ── ── ──     │
│                 ↑                              │
│         Frame chậm (jank!)                    │
│                                               │
│ Timeline (cho frame được chọn):               │
│ ┌──Build──┐┌──Layout──┐┌──Paint──┐           │
│ │  12ms   ││   3ms    ││  2ms   │← Budget:   │
│ └─────────┘└──────────┘└────────┘  16ms      │
│                                               │
│ Build phase quá lâu! → Optimize build()       │
└──────────────────────────────────────────────┘
```

> 🔗 **FE Bridge:** Flutter DevTools ≈ Chrome DevTools Performance tab — timeline, frame budget, jank detection. `Performance Overlay` ≈ FPS counter. Nhưng **khác ở**: Flutter có **2 threads** (UI + Raster) cần monitor riêng — FE chỉ có 1 main thread + requestAnimationFrame.

### 4.5 Memory View

```
Memory View:
┌──────────────────────────────────────────────┐
│ Heap Usage Over Time:                         │
│     ╱╲                                        │
│   ╱    ╲    ╱╲╱╲╱╲  ← GC events              │
│ ╱        ╲╱                                   │
│                         ╱                     │
│                       ╱   ← Memory tăng liên  │
│                     ╱       tục = LEAK!        │
│                                               │
│ Snapshot: Class instances count                │
│ ┌──────────────┬───────┐                      │
│ │ _StreamSub   │ 1,234 │ ← Quá nhiều!        │
│ │ AnimController│  567  │ ← Chưa dispose?     │
│ └──────────────┴───────┘                      │
└──────────────────────────────────────────────┘
```

### 4.6 💡 So sánh DevTools

| Feature | Flutter DevTools | React DevTools | Chrome DevTools |
|---------|-----------------|----------------|-----------------|
| Widget tree | ✅ Inspector | ✅ Components | ✅ Elements |
| Performance | ✅ Frame chart | ✅ Profiler | ✅ Performance |
| Memory | ✅ Heap snapshot | ❌ | ✅ Memory |
| Network | ✅ Network tab | ❌ | ✅ Network |
| Rebuild tracking | ✅ Rebuild counts | ✅ Highlight updates | ❌ |

---

## 5. Memory Leaks 🔴

### 5.1 Memory Leak là gì?

Memory leak xảy ra khi object không còn cần thiết nhưng vẫn được giữ tham chiếu → GC không thể thu hồi → memory tăng dần → app crash.

```
Bình thường:
Screen A mở → allocate memory → Screen A đóng → GC thu hồi ✅

Memory leak:
Screen A mở → allocate memory → Screen A đóng → reference vẫn tồn tại → GC KHÔNG thu hồi ❌
→ Lặp lại nhiều lần → Out of Memory → Crash 💥
```

### 5.2 Nguyên nhân phổ biến

**1. Stream subscription không cancel:**

```dart
// ❌ LEAK: subscription không bao giờ được cancel
class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    FirebaseFirestore.instance
        .collection('messages')
        .snapshots()
        .listen((snapshot) {  // ← Subscription sống mãi!
          setState(() {
            _messages = snapshot.docs;
          });
        });
  }
  // Widget bị dispose nhưng listener vẫn chạy!
}

// ✅ FIX: cancel trong dispose()
class _MyWidgetState extends State<MyWidget> {
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = FirebaseFirestore.instance
        .collection('messages')
        .snapshots()
        .listen((snapshot) {
          setState(() {
            _messages = snapshot.docs;
          });
        });
  }

  @override
  void dispose() {
    _subscription?.cancel();  // ← Cancel khi widget bị dispose
    super.dispose();
  }
}
```

**2. AnimationController không dispose:**

```dart
// ❌ LEAK: controller không dispose
class _AnimatedWidgetState extends State<AnimatedWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
  }
  // _controller không bao giờ được dispose!
}

// ✅ FIX: dispose controller
class _AnimatedWidgetState extends State<AnimatedWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
  }

  @override
  void dispose() {
    _controller.dispose();  // ← PHẢI dispose!
    super.dispose();
  }
}
```

**3. Timer/Periodic không cancel:**

```dart
// ❌ LEAK
Timer.periodic(const Duration(seconds: 1), (timer) {
  _updateClock();  // Chạy mãi sau khi widget dispose!
});

// ✅ FIX
late final Timer _timer;

@override
void initState() {
  super.initState();
  _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
    _updateClock();
  });
}

@override
void dispose() {
  _timer.cancel();
  super.dispose();
}
```

### 5.3 Checklist dispose()

```dart
@override
void dispose() {
  // 1. Animation controllers
  _animationController.dispose();

  // 2. Stream subscriptions
  _subscription?.cancel();

  // 3. Timers
  _timer?.cancel();

  // 4. TextEditingControllers
  _textController.dispose();

  // 5. FocusNodes
  _focusNode.dispose();

  // 6. ScrollControllers
  _scrollController.dispose();

  // 7. ValueNotifiers
  _valueNotifier.dispose();

  // Luôn gọi super.dispose() CUỐI CÙNG
  super.dispose();
}
```

> 🔗 **FE Bridge:** `dispose()` pattern ≈ React `useEffect` cleanup / Vue `onUnmounted` — release resources khi widget unmount. Nhưng **khác ở**: Flutter **bắt buộc** dispose controllers (Animation, Text, Scroll) — miss = memory leak. FE garbage collector tự handle hầu hết cases.

### 5.4 WeakReference

Khi cần giữ reference nhưng không muốn ngăn GC:

```dart
// Giữ reference yếu — GC có thể thu hồi object
final weakRef = WeakReference(heavyObject);

// Sử dụng
final obj = weakRef.target;  // null nếu đã bị GC
if (obj != null) {
  obj.doSomething();
}
```

### 5.5 LeakTracking trong tests

```dart
// Dùng leak_tracker package để detect leak trong test
import 'package:leak_tracker/leak_tracker.dart';

testWidgets('MyWidget does not leak', (tester) async {
  await tester.pumpWidget(const MyWidget());
  // Navigate away
  await tester.pumpWidget(const SizedBox());
  // LeakTracker sẽ báo nếu có object không được dispose
});
```

---

## 6. Isolates 🟡

### 6.1 Dart Event Loop — Giống JavaScript!

Dart là **single-threaded** với event loop, giống hệt JavaScript:

```
Main Isolate (Single Thread):
┌─────────────────────────────────────────────┐
│                Event Loop                     │
│  ┌────────────────────────────────────────┐  │
│  │ Microtask Queue: Future.then, async    │  │
│  │  ┌───┐ ┌───┐ ┌───┐                    │  │
│  │  │ M1│ │ M2│ │ M3│ → Xử lý trước     │  │
│  │  └───┘ └───┘ └───┘                    │  │
│  ├────────────────────────────────────────┤  │
│  │ Event Queue: I/O, Timer, UI events     │  │
│  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐             │  │
│  │  │ E1│ │ E2│ │ E3│ │ E4│ → Xử lý sau │  │
│  │  └───┘ └───┘ └───┘ └───┘             │  │
│  └────────────────────────────────────────┘  │
│                                               │
│  Loop: Microtasks → Events → Microtasks → ... │
└─────────────────────────────────────────────┘
```

> **Từ JavaScript:** `async/await` trong Dart hoạt động GIỐNG HỆT JavaScript. Nó **KHÔNG** tạo thread mới. Nó chỉ schedule task vào event loop. Code nặng vẫn block main thread!

```dart
// ⚠️ async/await KHÔNG giải quyết CPU-heavy task!
Future<List<Item>> parseJson(String json) async {
  // Vẫn chạy trên main thread!
  // Nếu json lớn (10MB) → block UI → jank!
  return jsonDecode(json);  // CPU-heavy, block main thread
}
```

### 6.2 Isolate là gì?

**Isolate** = thread riêng biệt với **heap riêng**, **KHÔNG chia sẻ memory** với main isolate.

```
Main Isolate                    Worker Isolate
┌─────────────────┐            ┌─────────────────┐
│  Heap            │            │  Heap            │
│  ┌─────────┐    │            │  ┌─────────┐    │
│  │ UI State │    │            │  │ JSON     │    │
│  │ Widgets  │    │   Message  │  │ Parser   │    │
│  │ Render   │    │◄──────────▶│  │ Data     │    │
│  └─────────┘    │  Passing   │  └─────────┘    │
│                  │ (SendPort/ │                  │
│  Event Loop      │ ReceivePort│  Event Loop      │
└─────────────────┘)           └─────────────────┘
     Không chia sẻ memory!
```

### 6.3 compute() — Cách đơn giản nhất

```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';

// Hàm parse phải là top-level hoặc static (không capture state)
List<Map<String, dynamic>> _parseJsonInBackground(String jsonString) {
  final List<dynamic> decoded = jsonDecode(jsonString);
  return decoded.cast<Map<String, dynamic>>();
}

// Sử dụng
Future<void> loadData() async {
  final response = await http.get(Uri.parse('https://api.example.com/data'));

  // ✅ Parse JSON trên isolate riêng — main thread tự do render!
  final items = await compute(_parseJsonInBackground, response.body);

  setState(() {
    _items = items;
  });
}
```

**Lưu ý về `compute()`:**
- Hàm callback phải là **top-level** hoặc **static** (không phải closure)
- Tham số và kết quả phải **serializable** (primitive types, List, Map)
- Có overhead tạo/hủy isolate → chỉ dùng cho task nặng (>16ms)

### 6.3b Isolate.run() — Preferred từ Dart 2.19+

```dart
// Dart 2.19+ — preferred over compute()
final result = await Isolate.run(() {
  return heavyComputation(data);
});
```

> 💡 **`Isolate.run()` vs `compute()`**: Từ Dart 2.19+, `Isolate.run()` là API chính thức thay thế `compute()`. Ưu điểm: type-safe hơn, không cần top-level function, hỗ trợ closure trực tiếp. `compute()` vẫn hoạt động nhưng `Isolate.run()` là hướng đi khuyến nghị.

### 6.4 Isolate.spawn — Cho task phức tạp

```dart
import 'dart:isolate';

Future<ProcessedImage> processImage(Uint8List imageBytes) async {
  final receivePort = ReceivePort();

  await Isolate.spawn(
    _processImageIsolate,
    _ImageProcessingRequest(
      imageBytes: imageBytes,
      sendPort: receivePort.sendPort,
    ),
  );

  final result = await receivePort.first as ProcessedImage;
  receivePort.close();
  return result;
}

// Chạy trên isolate riêng
void _processImageIsolate(_ImageProcessingRequest request) {
  // Heavy image processing...
  final processed = _applyFilters(request.imageBytes);
  final resized = _resize(processed, width: 800);
  final compressed = _compress(resized, quality: 85);

  request.sendPort.send(ProcessedImage(data: compressed));
}

class _ImageProcessingRequest {
  final Uint8List imageBytes;
  final SendPort sendPort;

  _ImageProcessingRequest({
    required this.imageBytes,
    required this.sendPort,
  });
}
```

### 6.5 Khi nào dùng Isolate?

```
Dùng async/await (Event Loop):          Dùng Isolate:
├── HTTP requests                        ├── Parse JSON lớn (>1MB)
├── File I/O                             ├── Image processing
├── Database queries                     ├── Complex calculations
├── Timer/Delays                         ├── Encryption/Decryption
└── Bất kỳ I/O-bound task               ├── Data compression
    (chờ response, không tính toán)      └── Bất kỳ CPU-bound task
                                             (tính toán nặng >16ms)
```

### 6.6 💡 So sánh với Web Workers

| Khía cạnh | Dart Isolate | JavaScript Web Worker |
|-----------|-------------|----------------------|
| Memory | Heap riêng biệt | Heap riêng biệt |
| Communication | SendPort/ReceivePort | postMessage/onmessage |
| Shared memory | ❌ Không | SharedArrayBuffer (hạn chế) |
| Spawn | `Isolate.spawn()`, `compute()` | `new Worker('file.js')` |
| Data transfer | Copy data (hoặc TransferableTypedData) | Structured clone (hoặc transfer) |

---

> 💼 **Gặp trong dự án:** Heavy computation (JSON parsing 10MB, image processing) block main thread, compute() vs Isolate.spawn decision, isolate communication patterns
> 🤖 **Keywords bắt buộc trong prompt:** `compute()`, `Isolate.spawn`, `SendPort/ReceivePort`, `compute overhead`, `isolate pool`, `data serialization cost`, `background processing`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Big data parse:** Server trả JSON 5MB — parsing block main thread 2-3 giây, app freeze
- **Image processing:** Apply filter lên ảnh 4K — phải offload sang isolate
- **Batch import:** Import 10,000 records từ CSV — cần background processing + progress reporting

**Tại sao cần các keyword trên:**
- **`compute()`** — simple one-shot function, spawn isolate + run + return result + kill isolate
- **`Isolate.spawn`** — long-running isolate, bidirectional communication
- **`SendPort/ReceivePort`** — IPC (inter-process communication) cho isolates
- **`compute overhead`** — spawning isolate có cost (~50ms), không dùng cho task < 100ms
- **`data serialization cost`** — data truyền vào isolate phải copy (không share memory)

**Prompt mẫu — Isolate cho heavy task:**
```text
Tôi cần offload heavy JSON parsing sang isolate trong Flutter.
Context: API trả response 5-10MB JSON, parsing trên main thread gây freeze 2-3s.
Requirements:
1. compute() wrapper: parseJsonInBackground(String rawJson) → List<Product>.
2. Isolate pool (2 isolates): cho trường hợp cần parse nhiều responses cùng lúc.
3. Progress reporting: khi parse 10,000 items → report progress 0-100% cho UI progress bar.
4. Error handling: parse fail → trả Failure (không crash isolate).
5. Cancel support: user navigate away → cancel parsing.
Constraints:
- compute() cho single parse, Isolate.spawn cho pool.
- Data > 1MB PHẢI dùng isolate.
- Return type: Either<Failure, List<Product>>.
Output: json_parser_service.dart, isolate_pool.dart.
```

**Expected Output:** AI gen isolate-based JSON parser + pool manager.

⚠️ **Giới hạn AI hay mắc:** AI thường quên rằng `compute()` serialize/deserialize data qua isolate boundary (cost!). AI hay suggest isolate cho tasks < 100ms (overhead > benefit). AI hay quên kill isolate khi done (memory leak).

</details>

---

## 7. Best Practices & Lỗi thường gặp 🟡

### ✅ Best Practices

```
1. Profile TRƯỚC khi optimize
   └── "Premature optimization is the root of all evil" — Donald Knuth
   └── Dùng DevTools đo trước, fix sau

2. const everywhere
   └── Thêm const cho mọi widget static
   └── IDE thường gợi ý (prefer_const_constructors lint)

3. Lazy loading
   └── ListView.builder cho lists
   └── FutureBuilder/StreamBuilder cho data
   └── Late initialization cho heavy objects

4. Minimize rebuilds
   └── Widget splitting
   └── buildWhen/select cho state management
   └── ValueListenableBuilder cho targeted updates

5. Profile mode for benchmarking
   └── flutter run --profile (KHÔNG phải debug)
   └── Test trên device thật (KHÔNG phải emulator)
```

### ❌ Lỗi thường gặp

| # | Lỗi | Hậu quả | Fix |
|---|------|---------|-----|
| 1 | Quên `const` cho static widgets | Rebuild không cần thiết | Thêm `const`, bật lint `prefer_const_constructors` |
| 2 | ListView cho list lớn | Load toàn bộ vào memory | Dùng `ListView.builder` |
| 3 | Heavy computation trong `build()` | Jank mỗi frame | Move ra ngoài build, cache kết quả |
| 4 | Quên `dispose()` controllers | Memory leak | Luôn dispose trong `dispose()` |
| 5 | Profile ở debug mode | Kết quả không chính xác | Luôn dùng `--profile` mode |
| 6 | Nested ListView không `shrinkWrap` | Crash hoặc chậm | Dùng `SliverList` trong `CustomScrollView` |
| 7 | `async/await` cho CPU-heavy task | Block main thread | Dùng `compute()` hoặc `Isolate` |
| 8 | Opacity widget cho fade effect | `saveLayer` tốn kém | Dùng `FadeTransition` hoặc `AnimatedOpacity` |

---

## 8. Image Optimization 🟡

### 8.1 Vấn đề thường gặp

Image là nguyên nhân phổ biến nhất gây jank và memory pressure trong Flutter app:
- Load ảnh full-size cho thumbnail → OOM crash
- Không cache → re-download mỗi lần rebuild
- Decode trên main thread → UI freeze

### 8.2 Best Practices

#### a. Resize tại source

```dart
// ❌ Load full 4K image cho avatar 48px
Image.network('https://example.com/photo-4000x3000.jpg')

// ✅ Request kích thước phù hợp (nếu server hỗ trợ)
Image.network('https://example.com/photo-4000x3000.jpg?w=96&q=80')
```

#### b. `cacheWidth` / `cacheHeight`

```dart
// Flutter tự resize image trong memory trước khi render
Image.asset(
  'assets/hero-banner.png',
  cacheWidth: 800, // pixels — Flutter decode ở kích thước này thay vì full
)
```

> 💡 `cacheWidth` / `cacheHeight` giúp giảm memory usage đáng kể khi hiển thị ảnh lớn ở kích thước nhỏ.

#### c. `cached_network_image` package

```dart
// pubspec.yaml
// cached_network_image: ^3.3.1

CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
  memCacheWidth: 200, // cache ở kích thước nhỏ trong memory
)
```

#### d. Precache images

```dart
// Precache trong didChangeDependencies hoặc initState
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  precacheImage(
    const AssetImage('assets/hero-banner.png'),
    context,
  );
}
```

### 8.3 So sánh React/Vue ↔ Flutter

| React/Vue | Flutter |
|---|---|
| `<img loading="lazy">` | `Image` widget (lazy by default khi trong `ListView`) |
| `srcset` / `<picture>` | `cacheWidth` / `cacheHeight` |
| `next/image` optimization | `cached_network_image` package |
| CDN image resizing | Server-side resize + `cacheWidth` |

> 🔗 **FE Bridge:** Image optimization **tương đồng** FE: lazy loading, caching, resize. `cached_network_image` ≈ `next/image` lazy + cache. Nhưng **khác ở**: Flutter render images ở **native layer** (Skia/Impeller), không qua browser image decoding pipeline.

---

## 9. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

Bảng so sánh tổng hợp cho developer từ React/Vue:

| Concept | React/Vue | Flutter |
|---------|-----------|---------|
| **Background thread** | Web Workers | Isolates |
| **Skip re-render** | `React.memo`, `useMemo` | `const` constructor |
| **Profiler** | React DevTools Profiler | Flutter DevTools Performance |
| **Memory profiler** | Chrome DevTools Memory | Flutter DevTools Memory |
| **Virtualized list** | `react-window`, `vue-virtual-scroller` | `ListView.builder` |
| **Frame budget** | `requestAnimationFrame` (~16ms) | vsync signal (~16ms) — cùng concept! |
| **Selective rebuild** | `useSelector`, computed | `select()`, `buildWhen` |
| **Render isolation** | `React.memo` boundary | `RepaintBoundary` |
| **Cleanup** | `useEffect` cleanup, `beforeUnmount` | `dispose()` trong StatefulWidget |

### Mental Model chuyển đổi:

```
React:                              Flutter:
useEffect cleanup ──────────────▶  dispose()
React.memo(() => ...) ─────────▶  const MyWidget()
useMemo(expensiveFn) ──────────▶  Cache ngoài build()
react-window ──────────────────▶  ListView.builder
Web Worker ────────────────────▶  Isolate / compute()
React DevTools Profiler ───────▶  DevTools Performance View
React.lazy + Suspense ─────────▶  FutureBuilder + lazy route
requestAnimationFrame ─────────▶  Ticker / vsync
```

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|--------------|
| 1 | Virtual DOM diffing auto-optimized | Widget rebuild = **explicit** — cần `const`, proper state scope | Flutter không có Virtual DOM, rebuild = real render tree |
| 2 | Browser handles frame scheduling | **2 threads**: UI thread + Raster thread — monitor cả hai | Jank có thể từ UI hoặc Raster, khác FE single thread |
| 3 | Garbage collector handles cleanup | `dispose()` **bắt buộc** cho controllers — miss = memory leak | Dart GC không handle native resources (streams, listeners) |
| 4 | Bundle size = main concern | APK/IPA size + **runtime memory** = dual concern | Mobile có memory limit, FE chỉ focus download size |

---

## 10. Tổng kết

### Checklist Performance Optimization

```
□ Rendering Pipeline
  ├── □ Hiểu 3 phase: Build → Layout → Paint
  ├── □ Hiểu frame budget (16ms/60fps, 8ms/120fps)
  └── □ Hiểu 3 trees: Widget → Element → RenderObject

□ FPS & Jank
  ├── □ Biết bật Performance Overlay
  ├── □ Profile ở --profile mode (không phải debug)
  └── □ Nhận diện jank trong DevTools

□ Rebuild Optimization
  ├── □ const constructors cho static widgets
  ├── □ Widget splitting cho granular rebuilds
  ├── □ RepaintBoundary cho expensive paints
  ├── □ ListView.builder cho large lists
  ├── □ ValueListenableBuilder cho targeted rebuilds
  ├── □ Keys cho reorderable/dynamic lists
  └── □ buildWhen/select cho state management

□ DevTools
  ├── □ Widget Inspector — rebuild tracking
  ├── □ Performance View — frame chart, timeline
  └── □ Memory View — snapshots, leak detection

□ Memory Leaks
  ├── □ dispose() cho tất cả controllers/subscriptions
  ├── □ Cancel streams, timers
  └── □ Detect leaks với DevTools Memory View

□ Isolates
  ├── □ Hiểu event loop (giống JavaScript)
  ├── □ compute() cho simple offloading
  ├── □ Isolate.spawn cho complex work
  └── □ Phân biệt I/O-bound (async) vs CPU-bound (isolate)
```

> **Buổi tiếp theo:** Animation — Biến app mượt mà thành app đẹp mắt với animations và transitions! 🎬
