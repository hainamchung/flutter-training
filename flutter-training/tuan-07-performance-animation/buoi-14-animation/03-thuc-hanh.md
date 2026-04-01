# Buổi 14: Animation trong Flutter — Thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## Mục lục

1. [BT1 ⭐ — Animated Expandable Card](#bt1--animated-expandable-card)
2. [BT2 ⭐⭐ — Custom Animated Button](#bt2--custom-animated-button)
3. [BT3 ⭐⭐⭐ — Product Catalog với Hero + Staggered Animation](#bt3--product-catalog-với-hero--staggered-animation)
4. [Câu hỏi thảo luận](#câu-hỏi-thảo-luận)

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Animation trong Flutter **khác biệt lớn** so với CSS animation/transition.
> FE developer cần shift mindset từ "declare CSS property" sang "compose widget + controller".

| FE Animation Habit | Flutter Reality | Bài tập liên quan |
|--------------------|-----------------|---------------------|
| `transition: opacity 300ms ease` | `AnimatedOpacity(duration: Duration(ms: 300))` — widget wrapper | BT1 |
| GSAP / Framer Motion timeline | `AnimationController` + `Tween` — phải `dispose()` khi unmount | BT2 |
| CSS `@keyframes` multi-step | `TweenSequence` hoặc `AnimationController` với `Interval` | BT2, BT3 |
| Canvas API `ctx.fillRect()` | `CustomPainter` + `canvas.drawRect()` — tương tự nhưng integrated với widget tree | BT3 |

---

## BT1 ⭐ — Animated Expandable Card 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_expandable_card` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Card mở rộng/thu gọn với AnimatedContainer |

### Yêu cầu

Tạo một card có thể **mở rộng/thu gọn** khi tap, sử dụng **AnimatedContainer** + **AnimatedCrossFade**.

### Đặc tả chi tiết

**Trạng thái thu gọn (collapsed):**
- Card nhỏ gọn, hiển thị: avatar, tên, subtitle ngắn
- Chiều cao cố định ~80px
- Border radius nhỏ (8px)
- Shadow nhẹ

**Trạng thái mở rộng (expanded):**
- Card mở rộng, hiển thị thêm: description, tags, action buttons
- Chiều cao tự động theo nội dung (~250px)
- Border radius lớn hơn (16px)
- Shadow đậm hơn

**Animation:**
- Dùng `AnimatedContainer` cho: height, borderRadius, shadow, padding
- Dùng `AnimatedCrossFade` cho: chuyển đổi giữa nội dung collapsed ↔ expanded
- Duration: 300ms, Curve: `Curves.easeInOut`
- Icon mũi tên xoay 180° khi mở/đóng (dùng `AnimatedRotation`)

### Gợi ý cấu trúc

```
ExpandableCard
├── AnimatedContainer (wrapper — animate size, decoration)
│   ├── Row (header — luôn hiển thị)
│   │   ├── CircleAvatar
│   │   ├── Column (name + subtitle)
│   │   └── AnimatedRotation (arrow icon)
│   └── AnimatedCrossFade
│       ├── firstChild: SizedBox.shrink() (collapsed — empty)
│       └── secondChild: Column (expanded content)
│           ├── Divider
│           ├── Text (description)
│           ├── Wrap (tags)
│           └── Row (action buttons)
```

### Dữ liệu mẫu

```dart
// Có thể dùng data cứng
final name = 'Nguyễn Văn A';
final subtitle = 'Flutter Developer';
final description = 'Kinh nghiệm 3 năm phát triển ứng dụng mobile...';
final tags = ['Flutter', 'Dart', 'Firebase', 'Clean Architecture'];
```

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| AnimatedContainer animate đúng properties | 30% |
| AnimatedCrossFade chuyển đổi nội dung smooth | 25% |
| Icon arrow xoay đúng hướng | 15% |
| Layout đẹp ở cả 2 trạng thái | 20% |
| Code sạch, đặt tên rõ ràng | 10% |

---

## BT2 ⭐⭐ — Custom Animated Button 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_animated_button` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Button với scale/bounce animation khi nhấn |

### Yêu cầu

Tạo một custom button có animation 3 trạng thái: **idle → pressed (scale down) → released (bounce back)**. Sử dụng `AnimationController` với custom `Curves`.

### Đặc tả chi tiết

**Trạng thái Idle:**
- Button bình thường, scale = 1.0
- Có subtle glow effect (shadow mềm)

**Trạng thái Pressed (onTapDown):**
- Scale xuống 0.92 (thu nhỏ nhẹ)
- Shadow giảm (nhấn xuống)
- Duration: 100ms
- Curve: `Curves.easeIn`

**Trạng thái Released (onTapUp / onTapCancel):**
- Scale bounce back lên 1.0
- Shadow trở lại bình thường
- Duration: 400ms
- Curve: `Curves.elasticOut` (nảy nhẹ)

**Bonus — Ripple effect:**
- Khi tap, hiển thị 1 vòng tròn mở rộng (dùng thêm 1 AnimationController)
- Vòng tròn fade out khi mở rộng hết

### Gợi ý kỹ thuật

```dart
class AnimatedButton extends StatefulWidget {
  final String label;
  final VoidCallback onPressed;

  const AnimatedButton({
    super.key,
    required this.label,
    required this.onPressed,
  });

  @override
  State<AnimatedButton> createState() => _AnimatedButtonState();
}

class _AnimatedButtonState extends State<AnimatedButton>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scaleAnimation;

  // Controller chạy forward khi pressed, reverse khi released
  // forward: 1.0 → 0.92 (scale down)
  // reverse: 0.92 → 1.0 (bounce back)

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 100),  // press duration
      reverseDuration: const Duration(milliseconds: 400), // release duration
    );

    _scaleAnimation = Tween<double>(begin: 1.0, end: 0.92).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeIn,
        reverseCurve: Curves.elasticOut,
      ),
    );
  }

  // Dùng GestureDetector với onTapDown, onTapUp, onTapCancel
  // onTapDown → _controller.forward()
  // onTapUp → _controller.reverse() + widget.onPressed()
  // onTapCancel → _controller.reverse()

  // ...
}
```

### Yêu cầu bổ sung

1. Button phải **reusable** — nhận `label`, `onPressed`, `color` từ ngoài
2. Khi button đang disabled (onPressed = null) → không có animation, màu xám
3. Accessibility: button phải wrap trong `Semantics` widget

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| AnimationController setup đúng (vsync, dispose) | 20% |
| Scale animation smooth: idle → pressed → released | 25% |
| Bounce effect tự nhiên với elasticOut | 15% |
| Button reusable (nhận props từ ngoài) | 15% |
| Shadow animate theo press state | 15% |
| Code sạch, dispose đúng | 10% |

---

## BT3 ⭐⭐⭐ — Product Catalog với Hero + Staggered Animation 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_product_catalog` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — Product grid với Hero transition và staggered animation |

### Yêu cầu

Xây dựng một **product catalog** hoàn chỉnh với:
1. **Hero transitions** giữa grid và detail page
2. **Staggered list animation** khi page load
3. **Custom animated AppBar** (thay đổi khi scroll)

### Đặc tả chi tiết

#### Phần 1: Product Grid Page

**Layout:**
- AppBar với title + search icon
- GridView 2 cột hiển thị product cards
- Mỗi card: image (Hero), tên, giá, rating

**Staggered Animation khi load:**
- Các card xuất hiện lần lượt (không cùng lúc)
- Mỗi card: fade in + slide up
- Card thứ i delay thêm `i * 100ms`
- Dùng `AnimationController` + `Interval` cho mỗi card

```
Timeline:
Card 0:  [████████████████]
Card 1:      [████████████████]
Card 2:          [████████████████]
Card 3:              [████████████████]
```

**Gợi ý staggered animation:**

```dart
// Trong AnimatedBuilder:
final itemAnimation = Tween(begin: 0.0, end: 1.0).animate(
  CurvedAnimation(
    parent: _controller,
    curve: Interval(
      index * 0.1,           // start: mỗi item delay thêm 10%
      (index * 0.1) + 0.5,   // end: mỗi item chạy 50% timeline
      curve: Curves.easeOut,
    ),
  ),
);
```

#### Phần 2: Product Detail Page

**Layout:**
- Hero image ở top (chiếm ~40% màn hình)
- Product name, price, rating
- Description text
- "Add to Cart" button (có thể dùng AnimatedButton từ BT2)

**Hero Animation:**
- Image trong grid → bay sang detail page full-width
- Tag: `'product-${product.id}'`

**Custom Animated AppBar:**
- Khi scroll up: AppBar fade in background color + show title
- Khi scroll down: AppBar transparent + hide title
- Dùng `ScrollController` + `AnimatedOpacity` / `AnimatedContainer`

```dart
// Gợi ý: nghe scroll position
_scrollController.addListener(() {
  final showTitle = _scrollController.offset > 200;
  if (showTitle != _showAppBarTitle) {
    setState(() => _showAppBarTitle = showTitle);
  }
});
```

#### Phần 3: Transitions

- Dùng custom `PageRouteBuilder` cho page transition:

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    transitionDuration: const Duration(milliseconds: 500),
    pageBuilder: (_, __, ___) => ProductDetailPage(product: product),
    transitionsBuilder: (_, animation, __, child) {
      return FadeTransition(
        opacity: animation,
        child: child,
      );
    },
  ),
);
```

### Dữ liệu mẫu

```dart
class Product {
  final String id;
  final String name;
  final String imageUrl;
  final double price;
  final double rating;
  final String description;

  const Product({
    required this.id,
    required this.name,
    required this.imageUrl,
    required this.price,
    required this.rating,
    required this.description,
  });
}

final sampleProducts = [
  Product(
    id: '1',
    name: 'Wireless Headphones',
    imageUrl: 'https://picsum.photos/id/1/400/400',
    price: 1299000,
    rating: 4.5,
    description: 'Tai nghe không dây chất lượng cao, chống ồn chủ động...',
  ),
  Product(
    id: '2',
    name: 'Smart Watch',
    imageUrl: 'https://picsum.photos/id/2/400/400',
    price: 2499000,
    rating: 4.8,
    description: 'Đồng hồ thông minh với nhiều tính năng theo dõi sức khỏe...',
  ),
  // ... thêm 4-6 sản phẩm
];
```

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| Hero transition hoạt động mượt giữa grid ↔ detail | 20% |
| Staggered animation khi load grid page | 20% |
| Custom animated AppBar (scroll-aware) | 15% |
| Page transition custom (không dùng default) | 10% |
| UI đẹp, responsive | 15% |
| Code architecture tốt (tách widget, reusable) | 10% |
| Performance (RepaintBoundary, const widgets) | 10% |

### Cấu trúc file gợi ý

```
lib/
├── models/
│   └── product.dart
├── data/
│   └── sample_products.dart
├── widgets/
│   ├── animated_product_card.dart    // Card with staggered animation
│   ├── animated_app_bar.dart         // Scroll-aware AppBar
│   └── animated_button.dart          // Từ BT2
├── pages/
│   ├── product_grid_page.dart        // Grid + staggered load
│   └── product_detail_page.dart      // Detail + Hero + scroll AppBar
└── main.dart
```

---

## Câu hỏi thảo luận

### Câu 1: Implicit vs Explicit — Khi nào chọn cái nào?

**Tình huống:** Bạn cần tạo các animation sau. Với mỗi cái, bạn sẽ dùng implicit hay explicit animation? Giải thích lý do.

1. Button thay đổi màu khi hover/focus
2. Loading spinner quay liên tục
3. Toast notification slide vào rồi tự biến mất sau 3 giây
4. Onboarding page với 5 element xuất hiện lần lượt (staggered)
5. Card flip 180° khi tap (front ↔ back)

**Gợi ý trả lời:**

| # | Animation | Loại | Lý do |
|---|-----------|------|-------|
| 1 | Button color | Implicit | Chỉ A→B, không loop |
| 2 | Loading spinner | Explicit | Loop liên tục |
| 3 | Toast notification | Explicit | Cần sequence: slide in → wait → slide out |
| 4 | Staggered onboarding | Explicit | Nhiều animation phối hợp (Interval) |
| 5 | Card flip | Explicit | Cần control direction, có thể pause |

### Câu 2: Animation Performance Budget

**Tình huống:** App của bạn có một trang feed hiển thị 20+ cards. Mỗi card có:
- Ảnh fade in khi load xong
- Like button có heart animation khi tap
- Shimmer loading effect khi đang fetch data

Team report rằng trang này bị janky trên thiết bị cũ (mid-range Android).

**Câu hỏi:**
1. Bạn sẽ debug performance bằng cách nào? (công cụ nào?)
2. Kể ra 3 optimization bạn sẽ áp dụng
3. Animation nào có thể bỏ/đơn giản hóa nếu cần?

**Gợi ý các điểm thảo luận:**
- Dùng Flutter DevTools → Performance tab → xem frame time
- `showPerformanceOverlay: true` để xem realtime
- `RepaintBoundary` cho mỗi card
- Chỉ animate items visible trên màn hình
- Shimmer dùng `ShaderMask` có thể tốn GPU — cân nhắc dùng alternative
- Heart animation: dùng pre-built Lottie thay vì code phức tạp

### Câu 3: Designer-Developer Workflow với Rive/Lottie

**Tình huống:** Team bạn có designer dùng Figma/After Effects. Designer muốn tạo các animation cho app:
- Splash screen animation (logo morph)
- Pull-to-refresh custom animation
- Empty state illustration (animated)
- Onboarding carousel (3 pages, mỗi page có animation)
- Micro-interactions (button, toggle, checkbox)

**Câu hỏi:**
1. Animation nào nên dùng Lottie? Animation nào nên code tay?
2. Workflow giữa designer và developer như thế nào?
3. Khi designer deliver file Lottie quá nặng (> 500KB), bạn làm gì?

**Gợi ý các điểm thảo luận:**
- Splash, empty state, onboarding → Lottie (visual phức tạp, designer control)
- Pull-to-refresh, micro-interactions → Code (cần interactive, phản hồi nhanh)
- Workflow: Designer export → Drop vào assets → Dev tích hợp + set triggers
- File nặng: giảm keyframes, bỏ layer thừa, dùng dotLottie (compressed), hoặc convert sang Rive
- Đặt spec: max 100KB per animation, max 5 giây duration, test trên thiết bị cũ

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 7:** Focus vào AI gen animation code và review lifecycle/performance.

### AI-BT1: Gen Hero + Staggered Animation cho Product Detail ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Implicit animations, explicit animations (AnimationController, Tween), Hero transitions, CustomPainter + animation, Lottie/Rive integration, performance tips.
- **Task thực tế:** Designer giao prototype: Product List → tap → Hero transition to Detail screen → staggered fade-in (image, title, price, description, CTA button). AI gen animation scaffolding, bạn review dispose + curve choices + performance.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần Hero transition + staggered animation cho Product Detail screen.
Tech stack: Flutter 3.x.
Flow: ProductListItem → Hero(tag: 'product-${id}') → ProductDetailScreen.
Detail screen staggered animation sequence:
1. Hero image: handled by Hero widget (0.0-0.3).
2. Product title: slide up + fade in (0.2-0.5, Curves.easeOut).
3. Price: slide up + fade in (0.3-0.6, Curves.easeOut).
4. Description: fade in (0.5-0.8, Curves.easeIn).
5. "Add to Cart" button: scale up + fade in (0.7-1.0, Curves.elasticOut).

Requirements:
1. Single AnimationController, 5 Interval-based animations.
2. TickerProviderStateMixin (multiple animations).
3. Hero tag unique per product.
4. dispose controller trong dispose().
5. Curves: UX-friendly (no linear).
Constraints:
- mounted check trước setState.
- addStatusListener: log animation complete.
- Reverse animation khi pop back.
Output: product_detail_screen.dart + product_list_item.dart (Hero source).
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 2 files (detail screen + list item).

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | `TickerProviderStateMixin` (not Single)? Multiple animations? | ☐ |
| 2 | Controller `dispose()` trong `dispose()` method? | ☐ |
| 3 | `mounted` check trước `setState`? | ☐ |
| 4 | Hero tag unique per product (không hardcode)? | ☐ |
| 5 | Curves hợp lý? (elasticOut cho button, easeOut cho slides) | ☐ |
| 6 | Interval timing sequential? Không overlap quá nhiều? | ☐ |
| 7 | Reverse animation khi navigate back? | ☐ |

**4. Customize:**
Thêm parallax effect cho image khi scroll detail screen. Thêm shimmer loading placeholder trước data load. AI gen static animation — tự thêm scroll-aware parallax + shimmer.
