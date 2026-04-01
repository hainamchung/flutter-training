# Buổi 14: Animation trong Flutter — Ví dụ minh hoạ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

1. [VD1: AnimatedContainer — Tap to Animate](#vd1-animatedcontainer--tap-to-animate)
2. [VD2: Pulsating Button — AnimationController + ScaleTransition](#vd2-pulsating-button--animationcontroller--scaletransition)
3. [VD3: Hero Transition — Photo Grid → Detail](#vd3-hero-transition--photo-grid--detail)
4. [VD4: Animated Circular Progress — CustomPainter](#vd4-animated-circular-progress--custompainter)
5. [VD5: Lottie Loading Animation](#vd5-lottie-loading-animation)

---

## VD1: AnimatedContainer — Tap to Animate 🟡

> **Mục tiêu:** Tap vào card → thay đổi size, color, borderRadius với smooth animation.

> **Liên quan tới:** [1. Implicit Animations 🟡](01-ly-thuyet.md#1-implicit-animations)

### Giải thích

- Dùng `AnimatedContainer` — implicit animation đơn giản nhất
- Mỗi khi `setState` thay đổi giá trị, container tự animate sang giá trị mới
- Không cần AnimationController, không cần dispose

### Code

```dart
import 'package:flutter/material.dart';

class AnimatedContainerDemo extends StatefulWidget {
  const AnimatedContainerDemo({super.key});

  @override
  State<AnimatedContainerDemo> createState() => _AnimatedContainerDemoState();
}

class _AnimatedContainerDemoState extends State<AnimatedContainerDemo> {
  bool _expanded = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedContainer Demo')),
      body: Center(
        child: GestureDetector(
          onTap: () => setState(() => _expanded = !_expanded),
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 400),
            curve: Curves.easeInOut,
            width: _expanded ? 280 : 160,
            height: _expanded ? 200 : 120,
            padding: EdgeInsets.all(_expanded ? 24 : 16),
            decoration: BoxDecoration(
              color: _expanded ? Colors.deepPurple : Colors.indigo,
              borderRadius: BorderRadius.circular(_expanded ? 24 : 12),
              boxShadow: [
                BoxShadow(
                  color: (_expanded ? Colors.deepPurple : Colors.indigo)
                      .withValues(alpha: 0.4),
                  blurRadius: _expanded ? 20 : 8,
                  offset: Offset(0, _expanded ? 10 : 4),
                ),
              ],
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  _expanded ? Icons.unfold_less : Icons.unfold_more,
                  color: Colors.white,
                  size: _expanded ? 36 : 24,
                ),
                const SizedBox(height: 8),
                Text(
                  _expanded ? 'Tap to collapse' : 'Tap to expand',
                  style: const TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.w600,
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### Kết quả

- Tap vào card: size tăng từ 160×120 → 280×200
- Màu chuyển từ indigo → deepPurple
- Border radius từ 12 → 24
- Shadow mở rộng
- Tất cả animate mượt trong 400ms với easeInOut curve
- Tap lại: animate ngược lại

### Điểm chú ý

- `AnimatedContainer` animate **tất cả** property cùng lúc: size, color, borderRadius, shadow
- `duration` và `curve` áp dụng chung cho mọi property
- Widget con (Column chứa Icon và Text) **không animate** — chỉ Container bọc ngoài animate

- 🔗 **FE tương đương:** `AnimatedContainer` changing color/size ≈ CSS `transition: all 300ms ease` — declare desired state, framework handles interpolation.

---

## VD2: Pulsating Button — AnimationController + ScaleTransition 🟡

> **Mục tiêu:** Tạo button nhấp nháy (pulse) liên tục bằng explicit animation.

> **Liên quan tới:** [2. Explicit Animations 🟡](01-ly-thuyet.md#2-explicit-animations)

### Giải thích

- Dùng `AnimationController` + `ScaleTransition` cho animation repeat
- Controller chạy ping-pong (forward → reverse → forward...)
- `CurvedAnimation` thêm easing cho tự nhiên hơn

### Code

```dart
import 'package:flutter/material.dart';

class PulsatingButton extends StatefulWidget {
  const PulsatingButton({super.key});

  @override
  State<PulsatingButton> createState() => _PulsatingButtonState();
}

class _PulsatingButtonState extends State<PulsatingButton>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();

    // 1. Tạo controller — vsync đồng bộ với refresh rate
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    );

    // 2. Tạo curved animation để easing tự nhiên
    final curvedAnimation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );

    // 3. Tween: scale từ 1.0 → 1.15 (to hơn 15%)
    _scaleAnimation = Tween<double>(
      begin: 1.0,
      end: 1.15,
    ).animate(curvedAnimation);

    // 4. Repeat ping-pong
    _controller.repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose(); // ⚠️ QUAN TRỌNG: luôn dispose
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Pulsating Button')),
      body: Center(
        child: ScaleTransition(
          scale: _scaleAnimation,
          child: ElevatedButton.icon(
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Button pressed!')),
              );
            },
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.red,
              foregroundColor: Colors.white,
              padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(30),
              ),
              elevation: 8,
            ),
            icon: const Icon(Icons.favorite),
            label: const Text(
              'Tap Me!',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ),
        ),
      ),
    );
  }
}
```

### Kết quả

- Button liên tục phóng to (1.0 → 1.15) và thu nhỏ (1.15 → 1.0)
- Animation mượt mà với easeInOut curve
- Vẫn nhấn được bình thường (onPressed hoạt động)
- Khi rời screen → dispose controller → animation dừng

### Điểm chú ý

- `SingleTickerProviderStateMixin` vì chỉ có 1 controller
- `ScaleTransition` là pre-built transition widget — tối ưu hơn `AnimatedBuilder` + `Transform.scale`
- `controller.repeat(reverse: true)` = ping-pong loop vô hạn
- **PHẢI `dispose()`** controller trong `dispose()` — nếu không sẽ memory leak

- 🔗 **FE tương đương:** `AnimationController` + `Tween` ≈ GSAP `gsap.to(element, { duration: 1, x: 100 })` — nhưng Flutter cần manual controller lifecycle (init + dispose).

---

## VD3: Hero Transition — Photo Grid → Detail 🟡

> **Mục tiêu:** Grid ảnh, tap vào ảnh → bay sang trang detail với Hero animation.

> **Liên quan tới:** [3. Hero Transitions 🟡](01-ly-thuyet.md#3-hero-transitions)

### Giải thích

- `Hero` widget wrap ảnh ở cả 2 routes
- Cùng `tag` → Flutter tự animate vị trí + kích thước
- Không cần code animation nào cả — framework xử lý hết

### Code

```dart
import 'package:flutter/material.dart';

// === Model ===
class Photo {
  final String id;
  final String url;
  final String title;

  const Photo({required this.id, required this.url, required this.title});
}

// === Data mẫu ===
const samplePhotos = [
  Photo(id: '1', url: 'https://picsum.photos/id/10/400/400', title: 'Forest'),
  Photo(id: '2', url: 'https://picsum.photos/id/20/400/400', title: 'Beach'),
  Photo(id: '3', url: 'https://picsum.photos/id/30/400/400', title: 'Mountain'),
  Photo(id: '4', url: 'https://picsum.photos/id/40/400/400', title: 'City'),
  Photo(id: '5', url: 'https://picsum.photos/id/50/400/400', title: 'Lake'),
  Photo(id: '6', url: 'https://picsum.photos/id/60/400/400', title: 'Desert'),
];

// === Grid Page ===
class PhotoGridPage extends StatelessWidget {
  const PhotoGridPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Photo Gallery')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
        ),
        itemCount: samplePhotos.length,
        itemBuilder: (context, index) {
          final photo = samplePhotos[index];
          return GestureDetector(
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => PhotoDetailPage(photo: photo),
                ),
              );
            },
            child: ClipRRect(
              borderRadius: BorderRadius.circular(12),
              child: Hero(
                tag: 'photo-${photo.id}', // Tag UNIQUE cho mỗi ảnh
                child: Image.network(
                  photo.url,
                  fit: BoxFit.cover,
                  // Placeholder khi loading
                  loadingBuilder: (context, child, loadingProgress) {
                    if (loadingProgress == null) return child;
                    return Container(
                      color: Colors.grey[200],
                      child: const Center(child: CircularProgressIndicator()),
                    );
                  },
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// === Detail Page ===
class PhotoDetailPage extends StatelessWidget {
  final Photo photo;

  const PhotoDetailPage({super.key, required this.photo});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        foregroundColor: Colors.white,
        title: Text(photo.title),
      ),
      body: Center(
        child: Hero(
          tag: 'photo-${photo.id}', // CÙNG tag với grid page
          child: InteractiveViewer(
            child: Image.network(
              photo.url,
              fit: BoxFit.contain,
              width: double.infinity,
            ),
          ),
        ),
      ),
    );
  }
}
```

### Kết quả

- Grid hiển thị 6 ảnh, 2 cột
- Tap vào ảnh: ảnh **bay** từ vị trí trong grid sang full screen ở detail page
- Back: ảnh **bay ngược** về vị trí cũ trong grid
- `InteractiveViewer` cho phép pinch-to-zoom ở detail page

### Điểm chú ý

- Hero `tag` **phải unique** — dùng `photo-${photo.id}`, không dùng string cố định
- Cùng `tag` ở 2 routes → Flutter tự detect và animate
- Ảnh tự smooth resize từ nhỏ (grid cell) → lớn (full screen)
- Hoạt động với cả `Navigator.push` và `Navigator.pop` (back button)

- 🔗 **FE tương đương:** Hero animation ≈ Framer Motion `layoutId` / View Transitions API — shared element auto-animates khi navigate. Flutter implementation mature hơn FE equivalents.

---

## VD4: Animated Circular Progress — CustomPainter 🟢

> **Mục tiêu:** Vẽ vòng tròn progress có animation từ 0% → target value.

> **Liên quan tới:** [4. CustomPainter + Animation 🟢](01-ly-thuyet.md#4-custompainter--animation)

### Giải thích

- `CustomPainter` vẽ arc (cung tròn) trên Canvas
- `AnimationController` drive giá trị progress từ 0 → target
- `shouldRepaint` so sánh progress cũ vs mới để tối ưu

### Code

```dart
import 'dart:math';
import 'package:flutter/material.dart';

// === CustomPainter ===
class CircularProgressPainter extends CustomPainter {
  final double progress; // 0.0 → 1.0
  final Color progressColor;
  final Color backgroundColor;
  final double strokeWidth;

  CircularProgressPainter({
    required this.progress,
    this.progressColor = Colors.blue,
    this.backgroundColor = Colors.grey,
    this.strokeWidth = 12,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = (size.width - strokeWidth) / 2;

    // 1. Vẽ vòng tròn nền (background track)
    final bgPaint = Paint()
      ..color = backgroundColor.withValues(alpha: 0.2)
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round;

    canvas.drawCircle(center, radius, bgPaint);

    // 2. Vẽ arc progress
    final progressPaint = Paint()
      ..color = progressColor
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round;

    final sweepAngle = 2 * pi * progress;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -pi / 2, // bắt đầu từ 12 giờ
      sweepAngle,
      false, // không nối về tâm
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(CircularProgressPainter oldDelegate) {
    return oldDelegate.progress != progress ||
        oldDelegate.progressColor != progressColor;
  }
}

// === Widget sử dụng Painter ===
class AnimatedCircularProgress extends StatefulWidget {
  final double targetProgress; // 0.0 → 1.0

  const AnimatedCircularProgress({
    super.key,
    required this.targetProgress,
  });

  @override
  State<AnimatedCircularProgress> createState() =>
      _AnimatedCircularProgressState();
}

class _AnimatedCircularProgressState extends State<AnimatedCircularProgress>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late Animation<double> _progressAnimation;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1500),
    );

    _progressAnimation = Tween<double>(
      begin: 0,
      end: widget.targetProgress,
    ).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOutCubic),
    );

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Circular Progress')),
      body: Center(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Stack(
              alignment: Alignment.center,
              children: [
                CustomPaint(
                  size: const Size(200, 200),
                  painter: CircularProgressPainter(
                    progress: _progressAnimation.value,
                    progressColor: Colors.teal,
                    strokeWidth: 14,
                  ),
                ),
                // Text phần trăm ở giữa
                Text(
                  '${(_progressAnimation.value * 100).toInt()}%',
                  style: const TextStyle(
                    fontSize: 40,
                    fontWeight: FontWeight.bold,
                    color: Colors.teal,
                  ),
                ),
              ],
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _controller.reset();
          _controller.forward();
        },
        child: const Icon(Icons.replay),
      ),
    );
  }
}

// Sử dụng: AnimatedCircularProgress(targetProgress: 0.75)
```

### Kết quả

- Vòng tròn progress animate từ 0% → 75% (hoặc giá trị target)
- Text phần trăm ở giữa cập nhật realtime theo animation
- Stroke tròn đầu (StrokeCap.round) cho đẹp hơn
- Nút replay để chạy lại animation
- Background track hiển thị (dạng vòng tròn mờ)

### Điểm chú ý

- `shouldRepaint` so sánh `progress` — chỉ vẽ lại khi giá trị thay đổi
- Bắt đầu arc từ `-pi / 2` (vị trí 12 giờ) thay vì 3 giờ (default)
- `AnimatedBuilder` + `CustomPaint` = pattern chuẩn cho animated painting
- Dùng `Stack` để overlay text lên CustomPaint

---

## VD5: Lottie Loading Animation 🟢

> **Mục tiêu:** Hiển thị Lottie animation làm loading indicator.

> **Liên quan tới:** [5. Rive / Lottie Integration 🟢](01-ly-thuyet.md#5-rive--lottie-integration)

### Giải thích

- Package `lottie` render file JSON từ LottieFiles
- Có thể dùng từ assets hoặc network
- Tích hợp với AnimationController để control playback

### Setup

```yaml
# pubspec.yaml
dependencies:
  lottie: ^3.1.0
```

Tải file Lottie JSON từ [LottieFiles.com](https://lottiefiles.com/) và đặt vào `assets/animations/`.

```yaml
# pubspec.yaml
flutter:
  assets:
    - assets/animations/
```

### Code

```dart
import 'package:flutter/material.dart';
import 'package:lottie/lottie.dart';

class LottieLoadingDemo extends StatefulWidget {
  const LottieLoadingDemo({super.key});

  @override
  State<LottieLoadingDemo> createState() => _LottieLoadingDemoState();
}

class _LottieLoadingDemoState extends State<LottieLoadingDemo>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _isLoading = false;
  String _status = 'Nhấn nút để bắt đầu loading';

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

  Future<void> _simulateLoading() async {
    setState(() {
      _isLoading = true;
      _status = 'Đang tải...';
    });

    _controller.repeat(); // loop animation

    // Giả lập loading 3 giây
    await Future.delayed(const Duration(seconds: 3));

    _controller.stop();
    _controller.reset();

    if (mounted) {
      setState(() {
        _isLoading = false;
        _status = 'Tải xong! ✅';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Lottie Loading')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // === Lottie Animation ===
            if (_isLoading)
              Lottie.asset(
                'assets/animations/loading.json',
                controller: _controller,
                width: 200,
                height: 200,
                onLoaded: (composition) {
                  // Set duration từ file Lottie
                  _controller.duration = composition.duration;
                },
              )
            else
              const SizedBox(
                width: 200,
                height: 200,
                child: Icon(
                  Icons.cloud_download_outlined,
                  size: 80,
                  color: Colors.grey,
                ),
              ),

            const SizedBox(height: 24),

            // === Status text ===
            Text(
              _status,
              style: const TextStyle(fontSize: 18),
            ),

            const SizedBox(height: 32),

            // === Button ===
            ElevatedButton.icon(
              onPressed: _isLoading ? null : _simulateLoading,
              icon: const Icon(Icons.download),
              label: Text(_isLoading ? 'Đang tải...' : 'Bắt đầu tải'),
            ),
          ],
        ),
      ),
    );
  }
}

// === Ví dụ đơn giản hơn — không cần controller ===
class SimpleLottieDemo extends StatelessWidget {
  const SimpleLottieDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Simple Lottie')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Auto-play, loop
            Lottie.asset(
              'assets/animations/loading.json',
              width: 150,
              height: 150,
              repeat: true,
            ),
            const SizedBox(height: 16),
            const Text('Loading...', style: TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }
}
```

### Kết quả

- Nhấn button → hiển thị Lottie loading animation (loop)
- Sau 3 giây → animation dừng, hiển thị "Tải xong! ✅"
- `SimpleLottieDemo`: version đơn giản — chỉ hiển thị Lottie auto-loop

### Điểm chú ý

- `onLoaded` callback lấy duration từ file Lottie gốc — đảm bảo tốc độ đúng ý designer
- Dùng `controller.repeat()` để loop, `controller.stop()` để dừng
- Check `mounted` trước `setState` sau async — tránh set state trên widget đã dispose
- Version đơn giản (`SimpleLottieDemo`) chỉ cần `Lottie.asset()` + `repeat: true`, không cần controller
- Cần khai báo `assets/animations/` trong `pubspec.yaml`

---

## VD6: 🤖 AI Gen → Review — Staggered Animation 🟢

> **Mục đích:** Luyện workflow "AI gen animation code → bạn review lifecycle + curves + performance → fix"

> **Liên quan tới:** [2. Explicit Animations 🟡](01-ly-thuyet.md#2-explicit-animations)

### Bước 1: Prompt cho AI

```text
Tạo staggered animation cho Flutter onboarding screen.
3 elements: logo (fade 0-0.3), text (slide+fade 0.3-0.7), button (scale 0.7-1.0).
Single AnimationController, dispose properly.
Output: onboarding_screen.dart.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **dispose** | `controller.dispose()` trong `dispose()` method? (quên = memory leak!) |
| 2 | **vsync** | `TickerProviderStateMixin` cho multiple animations? (Single = crash với 2+ controllers) |
| 3 | **Curves** | UX-friendly curves? (elasticOut cho button, easeOut cho slides, không linear) |
| 4 | **mounted** | Check mounted trước setState trong callbacks? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```dart
// ❌ LỖI 1: Quên dispose AnimationController
class _OnboardingState extends State<OnboardingScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    _controller = AnimationController(vsync: this, duration: 2.seconds);
    _controller.forward();
  }
  // THIẾU dispose() → memory leak khi navigate away!
}

// ✅ FIX: Luôn dispose
@override
void dispose() {
  _controller.dispose(); // PHẢI có!
  super.dispose();
}
```

```dart
// ❌ LỖI 2: Dùng Curves.linear cho UI animation
final animation = CurvedAnimation(
  parent: _controller,
  curve: Curves.linear, // Robot-like movement → bad UX
);

// ✅ FIX: Dùng natural curves
final animation = CurvedAnimation(
  parent: _controller,
  curve: Curves.easeOutCubic, // Natural deceleration → good UX
);
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết AnimationController PHẢI dispose (memory leak pattern)
- ✅ Chọn curves phù hợp UX (không dùng linear)
- ✅ Phân biệt TickerProviderStateMixin vs SingleTickerProviderStateMixin
