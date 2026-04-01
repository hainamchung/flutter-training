# Buổi 14: Animation trong Flutter — Lý thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 14/16** · **Thời lượng tự học:** ~1.5 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 03-04 (Widget basics + Layout system)

## Mục lục

1. [Implicit Animations](#1-implicit-animations)
2. [Explicit Animations](#2-explicit-animations)
3. [Hero Transitions](#3-hero-transitions)
4. [CustomPainter + Animation](#4-custompainter--animation)
5. [Rive / Lottie Integration](#5-rive--lottie-integration)
6. [Animation Performance Tips](#6-animation-performance-tips)
7. [Best Practices & Lỗi thường gặp](#7-best-practices--lỗi-thường-gặp)
8. [💡 FE → Flutter: Góc nhìn chuyển đổi](#8--fe--flutter-góc-nhìn-chuyển-đổi)
9. [Tổng kết](#9-tổng-kết)

---

## 1. Implicit Animations 🟡

### 1.1. Khái niệm

Implicit animations là cách đơn giản nhất để thêm animation trong Flutter. Bạn **chỉ cần thay đổi property**, widget tự animate đến giá trị mới. Không cần controller, không cần quản lý lifecycle.

> **Quy tắc:** Nếu có widget tên `Foo`, hãy tìm `AnimatedFoo`. Nếu tồn tại → dùng implicit animation.

### 1.2. Các Implicit Animation Widgets phổ biến

#### AnimatedContainer

Animate **mọi property** của Container: size, color, padding, margin, decoration, alignment...

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _expanded ? 200 : 100,
  height: _expanded ? 200 : 100,
  decoration: BoxDecoration(
    color: _expanded ? Colors.blue : Colors.red,
    borderRadius: BorderRadius.circular(_expanded ? 20 : 8),
  ),
  child: const Icon(Icons.star),
)
```

Khi `_expanded` thay đổi (qua `setState`), container tự animate mượt mà đến giá trị mới.

#### AnimatedOpacity

Animate opacity (fade in/out):

```dart
AnimatedOpacity(
  duration: const Duration(milliseconds: 500),
  opacity: _visible ? 1.0 : 0.0,
  child: const Text('Tôi sẽ fade!'),
)
```

#### AnimatedPositioned

Animate vị trí bên trong `Stack`:

```dart
Stack(
  children: [
    AnimatedPositioned(
      duration: const Duration(milliseconds: 400),
      curve: Curves.elasticOut,
      left: _moved ? 200 : 0,
      top: _moved ? 100 : 0,
      child: Container(width: 50, height: 50, color: Colors.green),
    ),
  ],
)
```

#### AnimatedDefaultTextStyle

Animate style của text (font size, color, weight...):

```dart
AnimatedDefaultTextStyle(
  duration: const Duration(milliseconds: 300),
  style: _highlighted
      ? const TextStyle(fontSize: 24, color: Colors.orange, fontWeight: FontWeight.bold)
      : const TextStyle(fontSize: 16, color: Colors.black),
  child: const Text('Animated Text'),
)
```

#### AnimatedCrossFade

Chuyển đổi giữa 2 widget với crossfade effect:

```dart
AnimatedCrossFade(
  duration: const Duration(milliseconds: 300),
  crossFadeState: _showFirst
      ? CrossFadeState.showFirst
      : CrossFadeState.showSecond,
  firstChild: const Icon(Icons.favorite, size: 48, color: Colors.red),
  secondChild: const Icon(Icons.favorite_border, size: 48, color: Colors.grey),
)
```

#### TweenAnimationBuilder

Widget linh hoạt nhất cho implicit animation — animate **bất kỳ giá trị nào**:

```dart
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0.0, end: _targetAngle),
  duration: const Duration(milliseconds: 600),
  builder: (context, angle, child) {
    return Transform.rotate(
      angle: angle,
      child: child,
    );
  },
  child: const Icon(Icons.refresh, size: 48), // child không rebuild
)
```

> **Quan trọng:** Truyền `child` vào `TweenAnimationBuilder` để tránh rebuild widget con mỗi frame.

### 1.3. Duration & Curves

**Duration** — thời gian animation:
- Micro-interaction: 150–300ms
- Page transition: 300–500ms  
- Complex animation: 500–1000ms

**Curves** — tốc độ thay đổi theo thời gian:

```
Curves.linear        ──  tốc độ đều
Curves.easeIn        ──  bắt đầu chậm, kết thúc nhanh
Curves.easeOut       ──  bắt đầu nhanh, kết thúc chậm
Curves.easeInOut     ──  chậm → nhanh → chậm (phổ biến nhất)
Curves.elasticOut    ──  nảy elastic ở cuối
Curves.bounceOut     ──  nảy bounce ở cuối
Curves.fastOutSlowIn ──  Material Design standard curve
```

> 🔗 **FE Bridge:** Implicit animation ≈ **CSS transition** — declare target state, framework tự animate. `AnimatedContainer` ≈ `transition: all 300ms ease`. FE dev sẽ thấy khá quen — nhưng **khác ở**: Flutter implicit = widget wrapper, CSS transition = property-based.

### 1.4. Khi nào dùng Implicit Animation?

✅ Dùng khi:
- Animation đơn giản: thay đổi size, color, position
- Animation chỉ chạy từ A → B (không lặp, không reverse tự động)
- Muốn code gọn, không cần quản lý controller

❌ Không dùng khi:
- Cần loop/repeat animation
- Cần control chi tiết (pause, resume, seek)
- Cần staggered animation (nhiều animation phối hợp)

---

## 2. Explicit Animations 🟡

### 2.1. Kiến trúc tổng quan

```
AnimationController ──▶ Tween ──▶ Animation<T> ──▶ Widget
       │                                              ▲
       │            CurvedAnimation                   │
       └──────────────────────────────────────────────┘
                    AnimatedBuilder
```

Explicit animation cho bạn **toàn quyền kiểm soát**: start, stop, repeat, reverse, seek to position, phối hợp nhiều animation.

### 2.2. AnimationController

Trái tim của explicit animation. Tạo giá trị từ 0.0 → 1.0 theo thời gian.

```dart
class _MyWidgetState extends State<MyWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this, // TickerProvider — đồng bộ với refresh rate
      duration: const Duration(milliseconds: 800),
    );
  }

  @override
  void dispose() {
    _controller.dispose(); // ⚠️ BẮT BUỘC dispose!
    super.dispose();
  }
}
```

**vsync** — Tại sao cần `TickerProviderStateMixin`?
- `vsync` đồng bộ animation với screen refresh rate (60fps / 120fps)
- Khi widget không visible, ticker tự động pause → tiết kiệm battery
- `SingleTickerProviderStateMixin`: 1 controller
- `TickerProviderStateMixin`: nhiều controllers

**Điều khiển AnimationController:**

```dart
_controller.forward();          // chạy 0.0 → 1.0
_controller.reverse();          // chạy 1.0 → 0.0
_controller.repeat();           // loop vô hạn
_controller.repeat(reverse: true); // loop ping-pong
_controller.stop();             // dừng tại vị trí hiện tại
_controller.reset();            // về 0.0
_controller.animateTo(0.5);     // animate đến giá trị cụ thể
```

### 2.3. Tween

Chuyển đổi range 0.0–1.0 của controller sang range bạn cần:

```dart
// Double
final sizeTween = Tween<double>(begin: 50, end: 200);

// Color
final colorTween = ColorTween(begin: Colors.red, end: Colors.blue);

// Offset (cho SlideTransition)
final slideTween = Tween<Offset>(
  begin: const Offset(-1, 0), // ngoài màn hình bên trái
  end: Offset.zero,           // vị trí gốc
);

// BorderRadius
final radiusTween = BorderRadiusTween(
  begin: BorderRadius.circular(0),
  end: BorderRadius.circular(24),
);
```

### 2.4. Animation<T>

Kết hợp AnimationController + Tween thành giá trị animated:

```dart
late final Animation<double> _sizeAnimation;

@override
void initState() {
  super.initState();
  _controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 800),
  );

  _sizeAnimation = Tween<double>(begin: 50, end: 200).animate(_controller);
  // _sizeAnimation.value sẽ đi từ 50 → 200 khi controller chạy
}
```

### 2.5. CurvedAnimation

Thêm easing curve cho explicit animation:

```dart
final curvedAnimation = CurvedAnimation(
  parent: _controller,
  curve: Curves.elasticOut,
  reverseCurve: Curves.easeIn, // curve khác khi reverse
);

_sizeAnimation = Tween<double>(begin: 50, end: 200).animate(curvedAnimation);
```

### 2.6. AnimatedBuilder & AnimatedWidget

**AnimatedBuilder** — cách phổ biến nhất, tách biệt animation logic khỏi widget:

```dart
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Transform.scale(
      scale: _sizeAnimation.value,
      child: child, // child không rebuild
    );
  },
  child: const FlutterLogo(size: 100), // static child
)
```

**AnimatedWidget** — tạo reusable animated widget:

```dart
class SpinningLogo extends AnimatedWidget {
  const SpinningLogo({super.key, required Animation<double> animation})
      : super(listenable: animation);

  @override
  Widget build(BuildContext context) {
    final animation = listenable as Animation<double>;
    return Transform.rotate(
      angle: animation.value * 2 * pi,
      child: const FlutterLogo(size: 80),
    );
  }
}

// Sử dụng:
SpinningLogo(animation: _controller)
```

**Pre-built Transition Widgets** — Flutter cung cấp sẵn:

```dart
// Fade
FadeTransition(opacity: _fadeAnimation, child: ...)

// Scale
ScaleTransition(scale: _scaleAnimation, child: ...)

// Slide
SlideTransition(position: _slideAnimation, child: ...)

// Rotation
RotationTransition(turns: _rotationAnimation, child: ...)

// Size
SizeTransition(sizeFactor: _sizeAnimation, child: ...)
```

> **Khuyến nghị:** Ưu tiên dùng `FadeTransition` thay vì `AnimatedBuilder` + `Opacity` widget. Transition widgets tối ưu hơn vì không trigger rebuild.

> 🔗 **FE Bridge:** `AnimatedBuilder` ≈ React Spring / Framer Motion `useSpring` — chỉ rebuild phần animated, giữ phần static. Nhưng **khác ở**: Flutter approach = **widget composition** (builder pattern), FE approach = hook/HOC wrapping.

### 2.7. Staggered Animations

Nhiều animation phối hợp, mỗi animation chạy ở interval khác nhau trong cùng controller:

```dart
late final Animation<double> _opacityAnim;
late final Animation<double> _scaleAnim;
late final Animation<Offset> _slideAnim;

@override
void initState() {
  super.initState();
  _controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 1500),
  );

  // 0% → 30%: fade in
  _opacityAnim = Tween(begin: 0.0, end: 1.0).animate(
    CurvedAnimation(
      parent: _controller,
      curve: const Interval(0.0, 0.3, curve: Curves.easeIn),
    ),
  );

  // 20% → 60%: scale up
  _scaleAnim = Tween(begin: 0.5, end: 1.0).animate(
    CurvedAnimation(
      parent: _controller,
      curve: const Interval(0.2, 0.6, curve: Curves.elasticOut),
    ),
  );

  // 50% → 100%: slide vào vị trí
  _slideAnim = Tween(
    begin: const Offset(0, 0.3),
    end: Offset.zero,
  ).animate(
    CurvedAnimation(
      parent: _controller,
      curve: const Interval(0.5, 1.0, curve: Curves.easeOut),
    ),
  );
}
```

```
Timeline:  0%────────30%────────60%────────100%
Opacity:   [████████]
Scale:          [████████████████]
Slide:                     [████████████████]
```

---

> � **FE Bridge:** `AnimationController` ≈ **Web Animations API** / GSAP timeline — full control: play, pause, reverse, repeat. `Tween` ≈ keyframe interpolation. Nhưng **khác ở**: Flutter animation = **tick-based** (vsync), FE thường dùng `requestAnimationFrame`. AnimationController cần `dispose()` — FE animation tự cleanup.

> �💼 **Gặp trong dự án:** Tạo complex animation sequences (staggered, chained), AnimationController lifecycle (init, dispose), Tween + CurvedAnimation combinations, animate multiple properties simultaneously
> 🤖 **Keywords bắt buộc trong prompt:** `AnimationController`, `vsync`, `Tween`, `CurvedAnimation`, `AnimatedBuilder`, `addStatusListener`, `staggered animation`, `dispose controller`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Design handoff:** Designer giao Figma prototype có loading animation — 3 dots bounce sequentially (staggered)
- **Onboarding:** Cần intro screen với text fade-in → image slide-up → button scale-in (chained)
- **Micro-interaction:** Button press có bounce + color change + ripple effect (multi-property)

**Tại sao cần các keyword trên:**
- **`AnimationController`** — central controller, PHẢI dispose! AI hay quên dispose → memory leak
- **`vsync`** — tie animation to frame refresh, cần `TickerProviderStateMixin`
- **`Tween`** — define start/end values, AI hay hardcode thay vì dùng Tween
- **`CurvedAnimation`** — apply easing curves, AI hay dùng linear (boring UX)
- **`staggered animation`** — multiple animations start tại different times, cần Interval

**Prompt mẫu — Staggered Animation:**
```text
Tôi cần tạo staggered animation cho onboarding screen trong Flutter.
Sequence:
1. Logo: fade in (0.0 → 0.3 duration, Curves.easeIn).
2. Title text: slide up from bottom + fade in (0.2 → 0.5, Curves.easeOut).
3. Subtitle: slide up + fade in (0.4 → 0.7, Curves.easeOut).
4. CTA button: scale from 0 → 1 + fade in (0.6 → 1.0, Curves.elasticOut).
Requirements:
1. Single AnimationController (2000ms total duration).
2. 4 separate Tween + CurvedAnimation với Interval timing.
3. AnimatedBuilder cho mỗi element.
4. Auto-play khi screen mở, addStatusListener log completion.
5. dispose controller trong dispose().
Constraints:
- TickerProviderStateMixin (KHÔNG SingleTickerProviderStateMixin — multiple tweens).
- Curves phải UX-friendly (không linear).
- Total duration configurable.
Output: onboarding_animation_screen.dart.
```

**Expected Output:** AI gen staggered animation screen hoàn chỉnh.

⚠️ **Giới hạn AI hay mắc:** AI hay dùng `SingleTickerProviderStateMixin` cho multiple animations (sai — cần `TickerProviderStateMixin`). AI hay quên `dispose()` controller. AI cũng hay dùng `Curves.linear` thay vì UX-appropriate curves.

</details>

---

## 3. Hero Transitions 🟡

### 3.1. Khái niệm

Hero animation tạo hiệu ứng **"bay"** giữa 2 routes — widget bay từ vị trí ở route cũ sang vị trí ở route mới. Flutter tự động animate size, position, và shape.

### 3.2. Cách hoạt động

```
Route A                          Route B
┌────────────────┐               ┌────────────────┐
│  ┌──────┐      │               │                │
│  │ Hero │      │  ──animate──▶ │   ┌────────┐   │
│  │tag="a"│     │               │   │ Hero   │   │
│  └──────┘      │               │   │tag="a" │   │
│                │               │   └────────┘   │
└────────────────┘               └────────────────┘
```

**Quy tắc:** 2 widget `Hero` ở 2 route khác nhau có **cùng `tag`** → Flutter tự animate.

### 3.3. Triển khai cơ bản

**Route A — Danh sách:**

```dart
// Trong GridView/ListView
GestureDetector(
  onTap: () => Navigator.push(
    context,
    MaterialPageRoute(builder: (_) => DetailPage(imageUrl: url)),
  ),
  child: Hero(
    tag: 'photo-$id', // tag UNIQUE cho mỗi item
    child: Image.network(url, fit: BoxFit.cover),
  ),
)
```

**Route B — Chi tiết:**

```dart
class DetailPage extends StatelessWidget {
  final String imageUrl;
  const DetailPage({super.key, required this.imageUrl});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Hero(
          tag: 'photo-$id', // CÙNG tag với route A
          child: Image.network(imageUrl, fit: BoxFit.contain),
        ),
      ),
    );
  }
}
```

### 3.4. Custom Flight Shape

Mặc định Hero giữ hình chữ nhật khi bay. Dùng `flightShuttleBuilder` để customize:

```dart
Hero(
  tag: 'avatar-$id',
  flightShuttleBuilder: (flightContext, animation, direction, fromContext, toContext) {
    return AnimatedBuilder(
      animation: animation,
      builder: (context, child) {
        return Material(
          color: Colors.transparent,
          child: ClipOval( // giữ hình tròn khi bay
            child: toContext.widget,
          ),
        );
      },
    );
  },
  child: CircleAvatar(backgroundImage: NetworkImage(avatarUrl)),
)
```

### 3.5. Photo Gallery Use Case

Pattern phổ biến: grid ảnh → tap → full screen với Hero:

```dart
// Grid thumbnail
Hero(
  tag: 'photo-${photo.id}',
  child: ClipRRect(
    borderRadius: BorderRadius.circular(8),
    child: Image.network(photo.thumbUrl, fit: BoxFit.cover),
  ),
)

// Detail page — full screen
Hero(
  tag: 'photo-${photo.id}',
  child: InteractiveViewer( // cho phép pinch-to-zoom
    child: Image.network(photo.fullUrl, fit: BoxFit.contain),
  ),
)
```

### 3.6. Lưu ý

- Tag **phải unique** trong cùng route (không trùng tag giữa 2 Hero visible cùng lúc)
- Hero animation hoạt động với `Navigator.push`/`Navigator.pop` (cả named routes)
- Widget con của Hero nên có **kích thước xác định** để animation mượt
- Tránh Hero quá nhiều widget cùng lúc — gây janky animation

> 🔗 **FE Bridge:** Hero animation ≈ **View Transitions API** (Chrome) / Framer Motion `layoutId` — shared element transition giữa 2 "pages". Flutter `Hero` = wrap widget + same `tag` = auto animate. FE API mới hơn và ít mature hơn Flutter Hero.

---

## 4. CustomPainter + Animation 🟢

### 4.1. Khái niệm CustomPainter

`CustomPainter` cho phép vẽ **bất kỳ hình gì** trực tiếp lên Canvas — giống Canvas API trong Web.

```dart
class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Vẽ ở đây
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}

// Sử dụng:
CustomPaint(
  size: const Size(300, 300),
  painter: MyPainter(),
)
```

### 4.2. Canvas API

**Paint object** — cọ vẽ:

```dart
final paint = Paint()
  ..color = Colors.blue
  ..strokeWidth = 3.0
  ..style = PaintingStyle.stroke; // stroke = viền, fill = tô đặc
```

**Vẽ các hình cơ bản:**

```dart
@override
void paint(Canvas canvas, Size size) {
  final paint = Paint()..color = Colors.blue..strokeWidth = 2;

  // Đường thẳng
  canvas.drawLine(
    const Offset(0, 0),
    Offset(size.width, size.height),
    paint,
  );

  // Hình tròn
  canvas.drawCircle(
    Offset(size.width / 2, size.height / 2), // tâm
    50, // bán kính
    paint..style = PaintingStyle.fill,
  );

  // Hình chữ nhật
  canvas.drawRect(
    Rect.fromLTWH(10, 10, 100, 60),
    paint..style = PaintingStyle.stroke,
  );

  // Cung tròn (arc)
  canvas.drawArc(
    Rect.fromCircle(center: Offset(size.width / 2, size.height / 2), radius: 80),
    -pi / 2,       // startAngle (12 giờ)
    pi * 1.5,      // sweepAngle (270 độ)
    false,          // useCenter
    paint..strokeCap = StrokeCap.round,
  );

  // Path tùy ý
  final path = Path()
    ..moveTo(0, size.height)
    ..quadraticBezierTo(size.width / 2, 0, size.width, size.height)
    ..close();
  canvas.drawPath(path, paint..style = PaintingStyle.fill);
}
```

### 4.3. shouldRepaint Optimization

`shouldRepaint` quyết định khi nào cần vẽ lại. **Tối ưu đúng cách:**

```dart
class ProgressPainter extends CustomPainter {
  final double progress;
  final Color color;

  ProgressPainter({required this.progress, required this.color});

  @override
  void paint(Canvas canvas, Size size) {
    // vẽ dựa trên progress...
  }

  @override
  bool shouldRepaint(ProgressPainter oldDelegate) {
    // Chỉ vẽ lại khi giá trị thay đổi
    return oldDelegate.progress != progress || oldDelegate.color != color;
  }
}
```

> ⚠️ Đừng return `true` vô điều kiện — gây vẽ lại mỗi frame dù không cần.

### 4.4. Animated CustomPainter

Kết hợp `AnimationController` + `CustomPainter` để tạo animation tùy chỉnh:

```dart
class _AnimatedProgressState extends State<AnimatedProgress>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return CustomPaint(
          size: const Size(200, 200),
          painter: CircularProgressPainter(
            progress: _controller.value, // 0.0 → 1.0
            color: Colors.blue,
          ),
        );
      },
    );
  }
}
```

### 4.5. Use Cases

- **Charts:** bar chart, line chart, pie chart tùy chỉnh
- **Custom shapes:** logo, biểu tượng phức tạp
- **Drawing app:** cho phép user vẽ tự do
- **Game elements:** sprite đơn giản, background effects
- **Custom progress indicators:** circular, linear với gradient

> 🔗 **FE Bridge:** `CustomPainter` ≈ **Canvas API** — draw shapes, paths, gradients tự do. Flutter Canvas API **tương đồng** HTML5 Canvas: `canvas.drawRect()` ≈ `ctx.fillRect()`, `canvas.drawPath()` ≈ `ctx.stroke()`. Nhưng Flutter Canvas chạy trên **Skia/Impeller engine**, performance tốt hơn.

---

## 5. Rive / Lottie Integration 🟢

### 5.1. Lottie

**Lottie** chuyển đổi animation từ After Effects → JSON → render trên Flutter.

**Workflow:**
```
Designer (After Effects) ──▶ Bodymovin plugin ──▶ .json file ──▶ Flutter (lottie package)
```

**Setup:**

```yaml
# pubspec.yaml
dependencies:
  lottie: ^3.1.0
```

**Sử dụng:**

```dart
// Từ assets
Lottie.asset(
  'assets/animations/loading.json',
  width: 200,
  height: 200,
  repeat: true,
)

// Từ network
Lottie.network(
  'https://lottie.host/xxx/animation.json',
  width: 200,
  height: 200,
)

// Với controller (kiểm soát playback)
class _LottiePageState extends State<LottiePage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Lottie.asset(
      'assets/animations/success.json',
      controller: _controller,
      onLoaded: (composition) {
        _controller
          ..duration = composition.duration
          ..forward();
      },
    );
  }
}
```

### 5.2. Rive

**Rive** là nền tảng animation tương tác — hỗ trợ **state machines** (animation thay đổi dựa trên input).

**Setup:**

```yaml
dependencies:
  rive: ^0.13.0
```

**Sử dụng:**

```dart
// Cơ bản
const RiveAnimation.asset(
  'assets/animations/button.riv',
  fit: BoxFit.contain,
)

// Với State Machine
class _RiveButtonState extends State<RiveButton> {
  SMIBool? _isPressed;

  void _onRiveInit(Artboard artboard) {
    final controller = StateMachineController.fromArtboard(artboard, 'State Machine 1');
    if (controller != null) {
      artboard.addController(controller);
      _isPressed = controller.findInput<bool>('isPressed') as SMIBool?;
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => _isPressed?.value = true,
      onTapUp: (_) => _isPressed?.value = false,
      child: RiveAnimation.asset(
        'assets/animations/button.riv',
        onInit: _onRiveInit,
      ),
    );
  }
}
```

### 5.3. Khi nào dùng Lottie vs Rive?

| Tiêu chí | Lottie | Rive |
|-----------|--------|------|
| Source | After Effects | Rive Editor (web) |
| Tương tác | Playback only | State machines, interactive |
| File size | JSON (có thể lớn) | Binary (nhỏ hơn) |
| Ecosystem | Rất lớn (LottieFiles.com) | Đang phát triển |
| Learning curve | Thấp | Trung bình |
| Use case | Loading, success/error states, onboarding | Interactive buttons, game elements, dynamic UI |

### 5.4. Khi nào dùng Lottie/Rive vs code animation?

- **Complex visual:** hình ảnh phức tạp, particle effects → Lottie/Rive
- **Simple UI animation:** size, color, position → Implicit/Explicit animation
- **Designer-driven:** designer tạo animation → Lottie/Rive
- **Data-driven:** animation dựa trên data thay đổi → Code animation
- **Performance critical:** cần kiểm soát tuyệt đối → Code animation

---

> 💼 **Gặp trong dự án:** Integrate Lottie/Rive animation files từ designer, optimize animation file size, handle animation lifecycle (play/pause/stop khi visibility change), coordinate animation với app state
> 🤖 **Keywords bắt buộc trong prompt:** `Lottie.asset`, `RiveAnimation`, `AnimationController with Lottie`, `animation file optimization`, `visibility-aware animation`, `mounted check`, `animation cache`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **Asset delivery:** Designer giao 10 Lottie files cho app — cần integrate, manage lifecycle, optimize performance
- **Loading UX:** Splash screen dùng Rive animation — phải play once → navigate, handle error nếu file corrupt
- **Performance:** Lottie animation 500KB gây jank trên low-end devices — cần optimize

**Tại sao cần các keyword trên:**
- **`Lottie.asset`** — load từ assets, AI hay dùng URL (slow, no offline)
- **`AnimationController with Lottie`** — control play/pause, duration sync
- **`visibility-aware animation`** — pause khi off-screen (battery + CPU saving)
- **`mounted check`** — check `mounted` trước `setState` sau async animation load
- **`animation cache`** — Lottie compositions cacheable, avoid re-parse

**Prompt mẫu — Lottie Integration:**
```text
Tôi cần integrate 5 Lottie animations vào Flutter app.
Tech stack: Flutter 3.x, lottie ^3.x.
Animations: loading.json, success.json, error.json, empty_state.json, onboarding.json.
Requirements:
1. Reusable LottieWidget: asset path, loop/oneShot mode, size.
2. Lifecycle-aware: pause khi off-screen (VisibilityDetector), resume khi visible.
3. Controller: AnimationController sync với Lottie duration.
4. Error handling: fallback widget nếu file corrupt/missing.
5. Performance: cache compositions, limit concurrent animations.
Constraints:
- PHẢI dispose controller.
- PHẢI check mounted trước setState.
- Max file size: 100KB per animation.
- Fallback: SizedBox + error icon nếu load fail.
Output: lottie_widget.dart (reusable) + usage example.
```

**Expected Output:** AI gen reusable Lottie widget + lifecycle management.

⚠️ **Giới hạn AI hay mắc:** AI quên check `mounted` trước setState (crash khi widget disposed). AI cũng hay quên dispose AnimationController (memory leak). AI hay load Lottie từ network thay vì assets (slow + no offline).

</details>

---

## 6. Animation Performance Tips 🟡

### 6.1. Mục tiêu: 60fps (16.67ms per frame)

Mỗi frame có **16.67ms** để build + paint. Animation vượt thời gian này → janky (giật).

### 6.2. Các nguyên tắc chính

#### Tránh trigger rebuild trong animation

```dart
// ❌ BAD — setState mỗi frame → rebuild toàn bộ build method
_controller.addListener(() {
  setState(() {}); // rebuild mọi thứ
});

// ✅ GOOD — AnimatedBuilder chỉ rebuild phần cần thiết
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Transform.scale(
      scale: _scaleAnimation.value,
      child: child, // KHÔNG rebuild
    );
  },
  child: const ExpensiveWidget(), // build 1 lần duy nhất
)
```

#### Dùng Transition widgets thay vì Opacity widget

```dart
// ❌ BAD — Opacity widget tạo separate layer, tốn memory
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Opacity(
      opacity: _controller.value,
      child: child, // vẫn paint dù opacity = 0
    );
  },
  child: const ComplexWidget(),
)

// ✅ GOOD — FadeTransition tối ưu hơn
FadeTransition(
  opacity: _fadeAnimation,
  child: const ComplexWidget(),
)
```

#### RepaintBoundary

Cô lập vùng animate khỏi phần còn lại:

```dart
// Widget phức tạp xung quanh không bị repaint khi animation chạy
RepaintBoundary(
  child: AnimatedBuilder(
    animation: _controller,
    builder: (context, child) {
      return CustomPaint(
        painter: MyAnimatedPainter(progress: _controller.value),
      );
    },
  ),
)
```

#### Tránh animate complex subtrees

```dart
// ❌ BAD — animate opacity trên cây widget phức tạp
FadeTransition(
  opacity: _fadeAnimation,
  child: ListView.builder( // hàng trăm items!
    itemCount: 500,
    itemBuilder: (_, i) => ComplexListItem(i),
  ),
)

// ✅ GOOD — animate chỉ phần nhỏ, hoặc dùng AnimatedList
AnimatedList(
  // mỗi item có animation riêng, chỉ animate item visible
)
```

### 6.3. Debugging Animation Performance

```dart
// Bật performance overlay
MaterialApp(
  showPerformanceOverlay: true, // hiện fps chart
)
```

- **Raster thread > 16ms:** GPU đang làm việc nhiều → giảm layers, tránh ClipPath phức tạp
- **UI thread > 16ms:** CPU đang build lâu → giảm rebuild, dùng const widgets

---

## 7. Best Practices & Lỗi thường gặp 🟡

### 7.1. Best Practices

| Practice | Giải thích |
|----------|------------|
| Luôn `dispose()` AnimationController | Memory leak nếu quên |
| Dùng `SingleTickerProviderStateMixin` | Chỉ khi có 1 controller; `TickerProviderStateMixin` cho nhiều controller |
| Truyền `child` vào AnimatedBuilder | Tránh rebuild subtree mỗi frame |
| Dùng `const` cho static widgets | Compiler tối ưu, không rebuild |
| Chọn đúng loại animation | Simple → implicit, Complex → explicit |
| Test trên device thật | Emulator không phản ánh đúng performance |

### 7.2. Lỗi thường gặp

#### Quên dispose controller

```dart
// ❌ Memory leak!
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: ...);
  }
  // Quên dispose ở đây!
}

// ✅ Luôn dispose
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

#### Quên mixin TickerProvider

```dart
// ❌ LỖI RUNTIME: vsync requires TickerProvider
class _MyState extends State<MyWidget> {
  late final AnimationController _controller;
  // ...
  _controller = AnimationController(vsync: this); // ERROR!
}

// ✅ Thêm mixin
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
```

#### Dùng setState trong animation loop

```dart
// ❌ Rebuild toàn bộ widget tree mỗi frame
_controller.addListener(() {
  setState(() {});
});

// ✅ Dùng AnimatedBuilder hoặc ValueListenableBuilder
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) => ...,
)
```

#### Hero tag trùng lặp

```dart
// ❌ CRASH: duplicate Hero tag
ListView(
  children: items.map((item) =>
    Hero(tag: 'photo', child: ...) // tag giống nhau!
  ).toList(),
)

// ✅ Tag unique
Hero(tag: 'photo-${item.id}', child: ...)
```

---

## 8. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|---------------|
| 1 | CSS transition/animation = declarative | Implicit ≈ CSS transition, Explicit = **imperative controller** | Flutter explicit animation cần AnimationController + dispose |
| 2 | `requestAnimationFrame` browser-managed | **vsync** + TickerProvider — manual frame sync | Flutter animation tick = manual, browser auto-manages RAF |
| 3 | Framer Motion / GSAP = optional library | Animation = **built-in framework** — không cần thêm package | Flutter animation API là first-class, FE cần 3rd party |
| 4 | View Transitions API mới, limited support | Hero animation = **mature, built-in** — wrap + tag = done | Flutter Hero animation production-ready, FE View Transitions đang evolve |

### Implicit Animation vs CSS Transition

```
/* CSS */                              // Flutter
.box {                                 AnimatedContainer(
  transition: all 0.3s ease-in-out;      duration: Duration(milliseconds: 300),
  width: 100px;                          curve: Curves.easeInOut,
}                                        width: _expanded ? 200 : 100,
.box:hover {                           )
  width: 200px;                        // Thay đổi _expanded → auto animate
}
```

**Giống:** Khai báo trạng thái đích, framework tự animate.
**Khác:** Flutter dùng widget dedicated thay vì CSS property chung.

### Explicit Animation vs requestAnimationFrame / GSAP

```javascript
// GSAP                                // Flutter
gsap.to('.box', {                      _controller = AnimationController(
  duration: 0.8,                         duration: Duration(milliseconds: 800),
  x: 200,                               vsync: this,
  ease: 'elastic.out',                );
  repeat: -1,                          _animation = Tween(begin: 0, end: 200)
  yoyo: true,                           .animate(CurvedAnimation(
});                                        parent: _controller,
                                           curve: Curves.elasticOut,
                                         ));
                                       _controller.repeat(reverse: true);
```

**Giống:** Direct control, timeline, easing curves.
**Khác:** Flutter dùng `AnimationController` thay cho `requestAnimationFrame`. `vsync` tự đồng bộ refresh rate.

### Hero vs Shared Layout Animation (Framer Motion)

```jsx
// Framer Motion                       // Flutter
<motion.div layoutId="photo-1">        Hero(
  <img src={url} />                      tag: 'photo-1',
</motion.div>                            child: Image.network(url),
                                       )
// Framer auto-animate khi            // Flutter auto-animate khi
// layoutId giống nhau                 // Hero tag giống nhau giữa routes
```

**Giống:** Shared element giữa 2 layout/route animate tự động.
**Khác:** Framer Motion hoạt động trong cùng page, Hero Flutter chỉ hoạt động giữa routes.

### CustomPainter vs Canvas API / SVG

```javascript
// Web Canvas API                      // Flutter CustomPainter
const ctx = canvas.getContext('2d');    @override
ctx.beginPath();                       void paint(Canvas canvas, Size size) {
ctx.arc(100, 100, 50, 0, Math.PI*2);   canvas.drawCircle(
ctx.fillStyle = 'blue';                  Offset(100, 100), 50,
ctx.fill();                              Paint()..color = Colors.blue,
                                        );
                                       }
```

**Giống:** API gần như 1:1 — draw, path, arc, bezier...
**Khác:** Flutter `shouldRepaint` cho phép tối ưu repaint. Web Canvas vẽ lại toàn bộ mỗi frame.

### Lottie — Giống hệt!

```jsx
// React                               // Flutter
import Lottie from 'lottie-react';     import 'package:lottie/lottie.dart';

<Lottie                                Lottie.asset(
  animationData={animationJson}          'assets/loading.json',
  loop={true}                            repeat: true,
  style={{ width: 200 }}                 width: 200,
/>                                     )
```

**100% giống** — cùng file JSON, cùng API pattern. Đây là lợi thế lớn cho cross-platform workflow.

---

## 9. Tổng kết

### Checklist kiến thức

| # | Chủ đề | Tự đánh giá |
|---|--------|-------------|
| 1 | Dùng được AnimatedContainer, AnimatedOpacity, AnimatedCrossFade | ⬜ |
| 2 | Dùng TweenAnimationBuilder cho custom implicit animation | ⬜ |
| 3 | Tạo AnimationController với đúng mixin, đúng dispose | ⬜ |
| 4 | Kết hợp Tween + CurvedAnimation + AnimatedBuilder | ⬜ |
| 5 | Tạo staggered animations với Interval | ⬜ |
| 6 | Implement Hero transition giữa routes | ⬜ |
| 7 | Vẽ với CustomPainter (drawCircle, drawArc, drawPath) | ⬜ |
| 8 | Kết hợp AnimationController + CustomPainter | ⬜ |
| 9 | Tích hợp Lottie animation | ⬜ |
| 10 | Tối ưu animation performance (RepaintBoundary, FadeTransition) | ⬜ |

### Decision Tree: Chọn loại Animation nào?

```
Bạn cần animation gì?
│
├─ Thay đổi UI property đơn giản?
│  └─ ✅ Implicit Animation (AnimatedContainer, AnimatedOpacity...)
│
├─ Cần loop / reverse / control chi tiết?
│  └─ ✅ Explicit Animation (AnimationController + Tween)
│
├─ Chuyển element giữa 2 routes?
│  └─ ✅ Hero Transition
│
├─ Cần vẽ hình tùy chỉnh / chart / gauge?
│  └─ ✅ CustomPainter + AnimationController
│
├─ Animation phức tạp từ designer?
│  ├─ Chỉ playback → ✅ Lottie
│  └─ Cần tương tác → ✅ Rive
│
└─ Nhiều animation phối hợp?
   └─ ✅ Staggered Animation (Interval)
```

### Buổi tiếp theo

**Buổi 15: Platform Integration** — Platform channels, native code integration, plugins.
