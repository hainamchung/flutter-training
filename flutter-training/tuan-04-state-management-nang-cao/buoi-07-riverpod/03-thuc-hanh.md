# Buổi 07: Riverpod Deep Dive — Bài tập thực hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> Các bài tập dưới đây liên quan tới concepts mà FE developer dễ mắc sai lầm do **thói quen từ React state libs**.
> Đọc bảng dưới TRƯỚC khi code để tránh debug không cần thiết.

| React/Vue Habit | Riverpod Reality | Bài tập liên quan |
|-----------------|------------------|---------------------|
| `useSelector`/`useContext` cho mọi nơi | `ref.watch` trong `build()`, `ref.read` trong callbacks — dùng sai → bug | BT1, BT2 |
| Global state wrap bằng Provider component | Riverpod providers = global by default, không cần `ProviderScope` wrapper cho từng provider | BT1 |
| `useEffect` để fetch data | `FutureProvider` hoặc `AsyncNotifierProvider` — không dùng `initState` + `setState` | BT2, BT3 |
| Cache invalidation = manual | `ref.invalidate()` hoặc `autoDispose` — Riverpod quản lý cache lifecycle | BT3 |

---

## BT1 ⭐: Todo App với Riverpod 🔴

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt1_todo_riverpod` |
| **Setup** | `flutter pub add flutter_riverpod riverpod_annotation` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — ứng dụng Todo List với Riverpod |

### Yêu cầu

Xây dựng ứng dụng **Todo List** hoàn chỉnh sử dụng Riverpod `NotifierProvider`.

### Chức năng bắt buộc

- [ ] Thêm todo mới (TextField + nút Add)
- [ ] Toggle trạng thái completed/active
- [ ] Xóa todo (swipe to delete hoặc icon button)
- [ ] Filter: All / Active / Completed
- [ ] Hiển thị số lượng todos (total và completed count)

### Cấu trúc project gợi ý

```
lib/
├── main.dart
├── models/
│   └── todo.dart
├── providers/
│   └── todo_provider.dart
└── screens/
    └── todo_screen.dart
```

### Yêu cầu kỹ thuật

| # | Yêu cầu | Chi tiết |
|---|---------|----------|
| 1 | `NotifierProvider` | Dùng `Notifier<List<Todo>>` cho state chính |
| 2 | `Provider` (computed) | `filteredTodosProvider` dựa trên filter hiện tại |
| 3 | `StateProvider` | `todoFilterProvider` cho enum filter |
| 4 | `ConsumerWidget` | Tất cả widget cần Riverpod |
| 5 | Immutable state | `state = [...state, newTodo]` — KHÔNG mutate trực tiếp |

### Model gợi ý

```dart
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  const Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  Todo copyWith({String? id, String? title, bool? isCompleted}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}
```

### Gợi ý từng bước

<details>
<summary>Bước 1: Setup project</summary>

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
```

```dart
// main.dart
void main() {
  runApp(const ProviderScope(child: MyApp()));
}
```
</details>

<details>
<summary>Bước 2: Tạo TodosNotifier</summary>

```dart
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(String title) {
    // Tạo todo mới, thêm vào state
    // Nhớ: state = [...state, newTodo]
  }

  void toggleTodo(String id) {
    // Map qua list, toggle todo có id trùng
    // Nhớ: dùng copyWith, tạo list mới
  }

  void removeTodo(String id) {
    // Filter bỏ todo có id trùng
  }
}
```
</details>

<details>
<summary>Bước 3: Tạo computed providers</summary>

```dart
// Filter enum
enum TodoFilter { all, active, completed }

final todoFilterProvider = StateProvider<TodoFilter>((ref) => TodoFilter.all);

final filteredTodosProvider = Provider<List<Todo>>((ref) {
  final todos = ref.watch(todosProvider);
  final filter = ref.watch(todoFilterProvider);
  // Switch theo filter, trả về list phù hợp
});
```
</details>

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| App chạy không lỗi, ProviderScope đúng | 2 |
| Dùng NotifierProvider đúng cách | 2 |
| Filter hoạt động (3 trạng thái) | 2 |
| Immutable state update | 2 |
| UI clean, UX hợp lý | 2 |
| **Tổng** | **10** |

---

## BT2 ⭐⭐: Weather App với FutureProvider 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt2_weather_app` |
| **Setup** | `flutter pub add flutter_riverpod riverpod_annotation` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app xem thời tiết theo thành phố |

### Yêu cầu

Xây dựng ứng dụng **xem thời tiết** sử dụng `FutureProvider` để fetch data theo thành phố.

### Chức năng bắt buộc

- [ ] Input field nhập tên thành phố
- [ ] Fetch weather data (mock API) khi nhấn Search
- [ ] Hiển thị loading indicator khi đang fetch
- [ ] Hiển thị error message khi lỗi
- [ ] Hiển thị weather data: temperature, description, icon
- [ ] Pull-to-refresh để fetch lại

### Cấu trúc project gợi ý

```
lib/
├── main.dart
├── models/
│   └── weather.dart
├── providers/
│   └── weather_provider.dart
├── repositories/
│   └── weather_repository.dart
└── screens/
    └── weather_screen.dart
```

### Yêu cầu kỹ thuật

| # | Yêu cầu | Chi tiết |
|---|---------|----------|
| 1 | `FutureProvider.autoDispose.family` | Fetch weather theo city name |
| 2 | `StateProvider` | Lưu city name hiện tại |
| 3 | `AsyncValue.when()` | Handle loading/error/data đầy đủ |
| 4 | `ref.invalidate()` | Pull-to-refresh |
| 5 | Repository pattern | Tách logic fetch data ra repository |

### Mock data gợi ý

```dart
class WeatherRepository {
  Future<Weather> getWeather(String city) async {
    // Simulate API call
    await Future.delayed(const Duration(seconds: 1));

    final mockWeatherData = {
      'hanoi': const Weather(
        city: 'Ha Noi',
        temperature: 28,
        description: 'Partly Cloudy',
        icon: '⛅',
        humidity: 75,
        windSpeed: 12,
      ),
      'hochiminh': const Weather(
        city: 'Ho Chi Minh',
        temperature: 33,
        description: 'Sunny',
        icon: '☀️',
        humidity: 65,
        windSpeed: 8,
      ),
      'danang': const Weather(
        city: 'Da Nang',
        temperature: 30,
        description: 'Rainy',
        icon: '🌧️',
        humidity: 85,
        windSpeed: 20,
      ),
    };

    final normalizedCity = city.toLowerCase().replaceAll(' ', '');
    final weather = mockWeatherData[normalizedCity];

    if (weather == null) {
      throw Exception('City not found: $city');
    }

    return weather;
  }
}
```

### Model gợi ý

```dart
class Weather {
  final String city;
  final double temperature;
  final String description;
  final String icon;
  final int humidity;
  final double windSpeed;

  const Weather({
    required this.city,
    required this.temperature,
    required this.description,
    required this.icon,
    required this.humidity,
    required this.windSpeed,
  });
}
```

### Gợi ý từng bước

<details>
<summary>Bước 1: Tạo providers</summary>

```dart
// DI cho repository
final weatherRepositoryProvider = Provider<WeatherRepository>((ref) {
  return WeatherRepository();
});

// State: city hiện tại
final selectedCityProvider = StateProvider<String?>((ref) => null);

// FutureProvider: fetch weather
final weatherProvider = FutureProvider.autoDispose
    .family<Weather, String>((ref, city) async {
  final repo = ref.watch(weatherRepositoryProvider);
  return repo.getWeather(city);
});
```
</details>

<details>
<summary>Bước 2: UI với AsyncValue</summary>

```dart
// Trong build method:
final city = ref.watch(selectedCityProvider);

if (city == null || city.isEmpty) {
  return const Center(child: Text('Nhập tên thành phố để xem thời tiết'));
}

final weatherAsync = ref.watch(weatherProvider(city));

return weatherAsync.when(
  loading: () => // CircularProgressIndicator
  error: (err, _) => // Error message + Retry button
  data: (weather) => // Weather card UI
);
```
</details>

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| FutureProvider.family dùng đúng | 2 |
| AsyncValue.when() handle đủ 3 states | 2 |
| Repository pattern tách biệt | 2 |
| autoDispose hoạt động | 1 |
| Search + refresh hoạt động | 2 |
| UI hiển thị weather info rõ ràng | 1 |
| **Tổng** | **10** |

---

## BT3 ⭐⭐⭐: Full Riverpod App + Unit Tests 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app |
| **Tạo project** | `flutter create bt3_product_catalog` |
| **Setup** | `flutter pub add flutter_riverpod riverpod_annotation` |
| **Cách chạy** | `flutter run` |
| **Output** | UI trên emulator — app Product Catalog với giỏ hàng và unit tests |

### Yêu cầu

Xây dựng ứng dụng **Product Catalog** đầy đủ với:
- Danh sách sản phẩm (FutureProvider)
- Chi tiết sản phẩm (family + autoDispose)
- Giỏ hàng (NotifierProvider)
- Unit tests cho tất cả providers

### Chức năng bắt buộc

- [ ] Hiển thị danh sách sản phẩm từ mock API
- [ ] Xem chi tiết sản phẩm (navigate, family provider)
- [ ] Thêm/xóa sản phẩm vào giỏ hàng
- [ ] Hiển thị tổng số item và tổng giá trong giỏ hàng
- [ ] Unit tests (ít nhất 5 test cases)

### Cấu trúc project gợi ý

```
lib/
├── main.dart
├── models/
│   ├── product.dart
│   └── cart_item.dart
├── providers/
│   ├── product_provider.dart
│   └── cart_provider.dart
├── repositories/
│   └── product_repository.dart
└── screens/
    ├── product_list_screen.dart
    ├── product_detail_screen.dart
    └── cart_screen.dart

test/
├── providers/
│   ├── product_provider_test.dart
│   └── cart_provider_test.dart
└── repositories/
    └── product_repository_test.dart
```

### Yêu cầu kỹ thuật

| # | Yêu cầu | Chi tiết |
|---|---------|----------|
| 1 | `FutureProvider.autoDispose` | Fetch danh sách products |
| 2 | `FutureProvider.autoDispose.family` | Fetch product detail theo ID |
| 3 | `NotifierProvider` | Cart state (add, remove, clear, total) |
| 4 | `Provider` (computed) | `cartTotalProvider`, `cartItemCountProvider` |
| 5 | Repository pattern | `ProductRepository` — inject vào providers |
| 6 | Unit tests | ProviderContainer + override providers |
| 7 | autoDispose | Product detail tự dispose khi rời trang |

### Models gợi ý

```dart
class Product {
  final int id;
  final String name;
  final String description;
  final double price;
  final String imageUrl;

  const Product({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    required this.imageUrl,
  });
}

class CartItem {
  final Product product;
  final int quantity;

  const CartItem({required this.product, this.quantity = 1});

  double get totalPrice => product.price * quantity;

  CartItem copyWith({Product? product, int? quantity}) {
    return CartItem(
      product: product ?? this.product,
      quantity: quantity ?? this.quantity,
    );
  }
}
```

### Test cases bắt buộc

```dart
// test/providers/cart_provider_test.dart
void main() {
  late ProviderContainer container;

  setUp(() {
    container = ProviderContainer();
  });

  tearDown(() {
    container.dispose();
  });

  test('Cart starts empty', () {
    // TODO: verify cart is initially empty
  });

  test('Add product to cart', () {
    // TODO: add a product, verify cart has 1 item
  });

  test('Add same product increases quantity', () {
    // TODO: add same product twice, verify quantity is 2
  });

  test('Remove product from cart', () {
    // TODO: add then remove, verify cart is empty
  });

  test('Cart total calculates correctly', () {
    // TODO: add products, verify total price
  });
}
```

### Gợi ý test với mock repository

<details>
<summary>Mock repository cho test</summary>

```dart
class MockProductRepository implements ProductRepository {
  @override
  Future<List<Product>> getProducts() async {
    return const [
      Product(
        id: 1,
        name: 'Test Product',
        description: 'Test Description',
        price: 29.99,
        imageUrl: 'https://example.com/img.png',
      ),
    ];
  }

  @override
  Future<Product> getProductById(int id) async {
    return const Product(
      id: 1,
      name: 'Test Product',
      description: 'Test Description',
      price: 29.99,
      imageUrl: 'https://example.com/img.png',
    );
  }
}

// Trong test:
final container = ProviderContainer(
  overrides: [
    productRepositoryProvider.overrideWithValue(MockProductRepository()),
  ],
);
```
</details>

### Tiêu chí đánh giá

| Tiêu chí | Điểm |
|----------|------|
| Product list hiển thị với FutureProvider | 1 |
| Product detail với family + autoDispose | 2 |
| Cart hoạt động (add/remove/clear) | 2 |
| Computed providers (total, count) | 1 |
| Repository pattern + DI qua providers | 1 |
| Unit tests pass (≥ 5 test cases) | 2 |
| Code clean, structure rõ ràng | 1 |
| **Tổng** | **10** |

---

## 💬 Câu hỏi thảo luận

### Câu 1: Migration Provider → Riverpod

> Bạn đang maintain 1 app Flutter dùng Provider package (cũ). Team quyết định migrate sang Riverpod. Bạn sẽ:
>
> 1. Migrate toàn bộ 1 lần hay dần dần (incremental)?
> 2. Bước nào migrate trước: Models, Providers, hay Widgets?
> 3. Làm sao đảm bảo app vẫn hoạt động đúng trong quá trình migrate?

**Gợi ý thảo luận:**
- Riverpod và Provider có thể **cùng tồn tại** trong 1 app (dùng cả `ProviderScope` và Provider package)
- Migration guide: https://riverpod.dev/docs/migration/from_provider
- Bắt đầu từ leaf providers (không phụ thuộc provider khác)
- Viết test trước khi migrate

### Câu 2: Chọn đúng loại Provider

> Cho các use case sau, bạn sẽ dùng loại provider nào và tại sao?
>
> 1. Theme mode (light/dark) — toggle bởi user
> 2. Danh sách products từ API
> 3. Chat messages realtime từ WebSocket
> 4. Auth state (logged in user info, null nếu chưa login)
> 5. Filtered list dựa trên search query + category filter
> 6. User profile by user ID (navigate từ list sang detail)

**Gợi ý:**
| Use case | Provider type | Modifier |
|----------|--------------|----------|
| Theme | `NotifierProvider<ThemeNotifier, ThemeMode>` | — |
| Products | `FutureProvider<List<Product>>` hoặc `AsyncNotifierProvider` | `autoDispose` |
| Chat | `StreamProvider<List<Message>>` | `autoDispose` |
| Auth | `AsyncNotifierProvider<AuthNotifier, User?>` | — (keep alive) |
| Filtered list | `Provider<List<Product>>` (computed) | — |
| User by ID | `FutureProvider.family<User, String>` | `autoDispose.family` |

### Câu 3: Testing Strategies

> Team bạn tranh luận về chiến lược test cho app Riverpod:
>
> - **Dev A**: "Test Notifier trực tiếp, không cần widget test"
> - **Dev B**: "Widget test quan trọng hơn vì user tương tác qua UI"
> - **Dev C**: "Integration test là đủ, unit test tốn thời gian"
>
> Bạn đồng ý với ai? Chiến lược test nào hợp lý nhất?

**Gợi ý thảo luận:**
- **Testing pyramid**: Unit tests (nhiều nhất) → Widget tests → Integration tests (ít nhất)
- Unit test Notifiers: nhanh, isolated, test logic thuần
- Widget test: verify UI phản ứng đúng với state changes
- Override providers trong tests → không cần mock phức tạp
- `ProviderContainer` cho unit test vs `ProviderScope` cho widget test
- Cả 3 dev đều đúng một phần — balanced approach là tốt nhất

---

## 📋 Checklist nộp bài

| # | Item | BT1 | BT2 | BT3 |
|---|------|-----|-----|-----|
| 1 | App chạy không crash | ✅ | ✅ | ✅ |
| 2 | ProviderScope ở root | ✅ | ✅ | ✅ |
| 3 | Dùng đúng provider types | ✅ | ✅ | ✅ |
| 4 | ref.watch trong build, ref.read trong callback | ✅ | ✅ | ✅ |
| 5 | AsyncValue.when() cho async providers | — | ✅ | ✅ |
| 6 | autoDispose khi phù hợp | — | ✅ | ✅ |
| 7 | family cho parameterized data | — | ✅ | ✅ |
| 8 | Repository pattern | — | ✅ | ✅ |
| 9 | Unit tests (≥ 5 test cases) | — | — | ✅ |
| 10 | Code structure rõ ràng | ✅ | ✅ | ✅ |

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 4:** Focus vào gen state management code và review reactive patterns.

### AI-BT1: Gen Riverpod Providers cho Weather App ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** FutureProvider, NotifierProvider, family modifier, ref.watch/read/listen, autoDispose.
- **Task thực tế:** PM giao "build Weather App hiển thị thời tiết theo thành phố, có favorite cities list". Cần providers cho API call (FutureProvider.family) và local state (NotifierProvider).

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần setup Riverpod providers cho Weather App.
Tech stack: Flutter 3.x, flutter_riverpod ^2.x, freezed (cho immutable state).
Features:
1. weatherProvider (FutureProvider.autoDispose.family<Weather, String>) — fetch weather by city name từ OpenWeatherMap API.
2. favoriteCitiesProvider (NotifierProvider<FavoriteCitiesNotifier, List<String>>) — add/remove/list favorite cities, persist vào SharedPreferences.
3. selectedCityProvider (StateProvider<String?>) — city đang được chọn.
Constraints:
- weatherProvider dùng autoDispose + family (tự cleanup khi rời screen, nhận city parameter).
- FavoriteCitiesNotifier: addCity (check duplicate), removeCity, loadFromStorage (init).
- ref.watch(weatherProvider(selectedCity)) trong build để hiển thị weather data.
- ref.read(favoriteCitiesProvider.notifier).addCity() trong onPressed callback.
- Error handling: AsyncValue.when cho loading/data/error states.
Output: 3 files — weather_provider.dart, favorite_cities_provider.dart, weather_screen.dart.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 3 files với providers + screen sử dụng providers.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | `weatherProvider` dùng `autoDispose.family` (tự cleanup + nhận city param)? | ☐ |
| 2 | `ref.watch` trong build, `ref.read` trong callbacks? | ☐ |
| 3 | `AsyncValue.when` handle cả 3 states (loading, data, error)? | ☐ |
| 4 | FavoriteCitiesNotifier có check duplicate khi addCity? | ☐ |
| 5 | `notifyListeners()` KHÔNG cần (Riverpod Notifier dùng `state =` để trigger rebuild)? | ☐ |
| 6 | Providers khai báo top-level (không trong class)? | ☐ |
| 7 | ConsumerWidget hoặc ConsumerStatefulWidget (không phải StatelessWidget)? | ☐ |

**4. Customize:**
Thêm caching: khi user quay lại city đã xem, không gọi API lại trong 5 phút. Implement `ref.keepAlive()` với Timer cancel sau 5 phút. AI chưa handle phần này — tự thêm keepAlive logic trong weatherProvider.
