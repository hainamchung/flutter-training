# Buổi 07: Riverpod Deep Dive — Tài liệu tham khảo

---

## 📖 Tài liệu chính thức

| Nguồn | Link | Ghi chú |
|-------|------|---------|
| Riverpod Official Docs | https://riverpod.dev | Tài liệu chính, đầy đủ nhất |
| Riverpod Getting Started | https://riverpod.dev/docs/introduction/getting_started | Hướng dẫn bắt đầu |
| Riverpod — All Providers | https://riverpod.dev/docs/concepts/providers | Giải thích từng loại provider |
| flutter_riverpod (pub.dev) | https://pub.dev/packages/flutter_riverpod | Package chính cho Flutter |
| riverpod_annotation (pub.dev) | https://pub.dev/packages/riverpod_annotation | Annotations cho code-gen |
| riverpod_generator (pub.dev) | https://pub.dev/packages/riverpod_generator | Code generator |
| Riverpod GitHub | https://github.com/rrousselGit/riverpod | Source code, issues |

---

## 🔄 Migration Guide: Riverpod 2.x → 3.x {#migration}

Riverpod 3.x (từ `riverpod_generator ^3.0.0`) có một số thay đổi breaking quan trọng:

### Thay đổi chính

| Riverpod 2.x | Riverpod 3.x | Ghi chú |
|---|---|---|
| `FutureProviderRef`, `StreamProviderRef` | `Ref` (chung) | Tất cả provider dùng chung type `Ref` |
| `@riverpod` (auto-dispose mặc định) | `@riverpod` (giữ nguyên) | Behavior không đổi |
| `autoDispose: false` | `@Riverpod(keepAlive: true)` | Cú pháp mới cho non-auto-dispose |
| `StateProvider` | `NotifierProvider` | `StateProvider` deprecated |
| `StateNotifierProvider` | `NotifierProvider` / `AsyncNotifierProvider` | Migration sang Notifier class |

### Ví dụ migration

```dart
// Riverpod 2.x
@riverpod
Future<User> user(UserRef ref) async { ... }

// Riverpod 3.x — Ref thay thế UserRef
@riverpod
Future<User> user(Ref ref) async { ... }
```

```dart
// Riverpod 2.x — keepAlive
final myProvider = Provider<String>((ref) => 'hello');

// Riverpod 3.x — code-gen
@Riverpod(keepAlive: true)
String myValue(Ref ref) => 'hello';
```

### Tài liệu migration

| Nguồn | Link |
|-------|------|
| Official Migration Guide | https://riverpod.dev/docs/migration/from_riverpod_2_to_riverpod_3 |
| Changelog | https://pub.dev/packages/flutter_riverpod/changelog |
| Remi's announcement | https://github.com/rrousselGit/riverpod/discussions |

> 💡 **Tip**: Dùng `riverpod_lint` để tự động phát hiện code cần migrate. Chạy `dart fix --apply` sau khi cập nhật package.

---

## 📝 Blog & Bài viết

| Tiêu đề | Link | Mô tả |
|---------|------|--------|
| Migrating from Provider to Riverpod | https://riverpod.dev/docs/migration/from_provider | Hướng dẫn chuyển từ Provider sang Riverpod |
| Riverpod 2.0 — What's New | https://codewithandrea.com/articles/flutter-riverpod-2/ | Tổng quan thay đổi Riverpod 2.0 |
| Flutter Riverpod 2.0: The Ultimate Guide | https://codewithandrea.com/articles/flutter-state-management-riverpod/ | Hướng dẫn toàn diện (Andrea Bizzotto) |
| Riverpod — Complete Guide | https://codewithandrea.com/articles/flutter-riverpod-async-notifier/ | AsyncNotifier pattern |
| When to use each provider type | https://riverpod.dev/docs/concepts/providers#which-provider-should-i-use | Chọn đúng provider type |

---

## 🎥 Video hướng dẫn

| Tiêu đề | Nguồn | Mô tả |
|---------|-------|--------|
| Flutter Riverpod 2.0 Course | Andrea Bizzotto (YouTube) | Khóa học miễn phí, cập nhật Riverpod 2.x |
| Riverpod Explained | Reso Coder (YouTube) | Giải thích concept rõ ràng |
| Riverpod State Management | The Net Ninja (YouTube) | Series cho người mới |
| Flutter Riverpod Tutorial | Vandad Nahavandipoor (YouTube) | Ví dụ thực tế |

---

## 📦 Packages liên quan

| Package | pub.dev | Mục đích |
|---------|---------|----------|
| `flutter_riverpod` | https://pub.dev/packages/flutter_riverpod | Core Riverpod cho Flutter |
| `riverpod_annotation` | https://pub.dev/packages/riverpod_annotation | `@riverpod` annotation |
| `riverpod_generator` | https://pub.dev/packages/riverpod_generator | Code generator (dev_dependency) |
| `build_runner` | https://pub.dev/packages/build_runner | Chạy code generation |
| `riverpod_lint` | https://pub.dev/packages/riverpod_lint | Lint rules riêng cho Riverpod |
| `hooks_riverpod` | https://pub.dev/packages/hooks_riverpod | Riverpod + Flutter Hooks |
| `riverpod` | https://pub.dev/packages/riverpod | Riverpod cho Dart thuần (không Flutter) |

### pubspec.yaml mẫu

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
  riverpod_lint: ^2.3.13
```

---

## 🤖 AI Prompt Library — Buổi 07: Riverpod

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Flutter Riverpod. Background: 4+ năm React (useState, useContext, Redux, React Query).
Câu hỏi: Provider trong Riverpod giống hook nào trong React? FutureProvider giống React Query useQuery? ref.watch giống dependency array trong useEffect? NotifierProvider giống useReducer hay Redux slice?
Yêu cầu: mapping 1-1 với React concepts, giải thích bằng tiếng Việt, kèm code Riverpod minh họa.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần implement Weather App dùng Riverpod.
Tech stack: Flutter 3.x, flutter_riverpod ^2.x.
Cần 3 providers:
1. weatherProvider: FutureProvider.autoDispose.family<WeatherData, String> — fetch API by city.
2. favoritesProvider: NotifierProvider — manage list of favorite cities.
3. selectedCityProvider: StateProvider<String?> — track selected city.
Constraints:
- ref.watch trong build cho weather data display.
- ref.read trong onPressed cho add/remove favorites.
- AsyncValue.when cho loading/data/error UI.
- autoDispose cho API providers.
Output: provider files + 1 screen file.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Riverpod code sau:
[paste code]

Kiểm tra theo thứ tự:
1. ref.watch dùng trong build? ref.read dùng trong callbacks? (ngược lại = BUG)
2. FutureProvider có dùng autoDispose? (không → memory leak khi rời screen)
3. Notifier dùng state = newState (không dùng notifyListeners — đó là ChangeNotifier)?
4. Providers khai báo top-level global? (trong class = sai pattern)
5. AsyncValue.when handle đủ 3 cases? (hay chỉ handle data?)
6. family provider: parameter type có implement == và hashCode? (Record hoặc freezed)
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter Riverpod:
[paste error message]

Code liên quan:
[paste provider definition + widget code gây lỗi]

Context: dùng flutter_riverpod ^2.x, ProviderScope ở root.
Cần: (1) Giải thích nguyên nhân, (2) Check ref.watch/read usage, (3) Check ProviderScope tồn tại, (4) Fix cụ thể.
```

---

## 🔗 Tham khảo thêm

### So sánh State Management

| Tài liệu | Link |
|-----------|------|
| Flutter State Management Comparison | https://docs.flutter.dev/data-and-backend/state-mgmt/options |
| Provider vs Riverpod vs Bloc | https://codewithandrea.com/articles/flutter-state-management-riverpod/ |

### Code-gen & Build Runner

| Tài liệu | Link |
|-----------|------|
| build_runner docs | https://pub.dev/packages/build_runner |
| Dart Code Generation | https://dart.dev/tools/build_runner |

### Testing

| Tài liệu | Link |
|-----------|------|
| Testing Riverpod | https://riverpod.dev/docs/essentials/testing |
| Flutter Testing Guide | https://docs.flutter.dev/testing |

---

## 💡 Tips để đọc tài liệu hiệu quả

1. **Bắt đầu từ riverpod.dev** — Getting Started → Concepts → Providers
2. **Đọc migration guide** nếu đã biết Provider package
3. **Thực hành song song** — đọc đến đâu, code đến đó
4. **Dùng riverpod_lint** — package lint sẽ cảnh báo anti-patterns
5. **Tham khảo Andrea Bizzotto** — bài viết và video chất lượng cao, cập nhật

---

## 📅 Buổi tiếp theo

**Buổi 08: BLoC Pattern** — event-driven state management, khi nào chọn BLoC thay vì Riverpod.
