# Buổi 10: Dependency Injection & Testing — Tài liệu tham khảo

## 📚 Tài liệu chính thức

### Flutter & Dart

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Flutter Testing Cookbook | https://docs.flutter.dev/cookbook/testing | Hướng dẫn từng bước: unit, widget, integration test |
| Dart Testing Docs | https://dart.dev/guides/testing | Tổng quan testing trong Dart |
| Flutter Integration Testing | https://docs.flutter.dev/testing/integration-tests | Hướng dẫn integration test trên device |
| Flutter Widget Testing | https://docs.flutter.dev/cookbook/testing/widget/introduction | Giới thiệu widget testing |

---

## 📦 Packages

### Dependency Injection

| Package | Pub.dev | Mô tả |
|---------|---------|--------|
| **get_it** | https://pub.dev/packages/get_it | Service Locator cho Dart — đơn giản, nhanh, type-safe |
| **injectable** | https://pub.dev/packages/injectable | Code generation cho get_it — giảm boilerplate DI |
| **injectable_generator** | https://pub.dev/packages/injectable_generator | Generator đi kèm injectable (dev_dependency) |

### Testing

| Package | Pub.dev | Mô tả |
|---------|---------|--------|
| **mocktail** | https://pub.dev/packages/mocktail | Mocking library — không cần code gen, hỗ trợ null-safety |
| **bloc_test** | https://pub.dev/packages/bloc_test | Tiện ích test BLoC/Cubit — `blocTest()` helper |
| **integration_test** | Flutter SDK | Framework cho integration testing |

### Code Generation

| Package | Pub.dev | Mô tả |
|---------|---------|--------|
| **freezed** | https://pub.dev/packages/freezed | Immutable data classes, union types, copyWith |
| **freezed_annotation** | https://pub.dev/packages/freezed_annotation | Annotations cho freezed (dependency) |
| **json_serializable** | https://pub.dev/packages/json_serializable | JSON serialization code generation |
| **json_annotation** | https://pub.dev/packages/json_annotation | Annotations cho json_serializable (dependency) |
| **build_runner** | https://pub.dev/packages/build_runner | Build system chạy code generators |

---

## 📝 Bài viết & Blog

### Dependency Injection

| Bài viết | Link | Tóm tắt |
|----------|------|---------|
| Dependency Injection in Flutter (Reso Coder) | https://resocoder.com/2020/02/04/injectable-flutter-dart-equivalent-to-dagger-angular-dependency-injection/ | Hướng dẫn chi tiết injectable + get_it |
| Flutter Dependency Injection (Medium) | https://medium.com/flutter-community/flutter-dependency-injection-a-beginners-guide-a903db1e3bc6 | Giới thiệu DI cho beginner |
| get_it: Simple Service Locator | https://pub.dev/packages/get_it#getting-started | Getting Started guide chính thức |

### Testing

| Bài viết | Link | Tóm tắt |
|----------|------|---------|
| Testing Flutter Apps (Reso Coder) | https://resocoder.com/category/tutorials/flutter/tdd-clean-architecture/ | TDD + Clean Architecture series |
| Testing Best Practices | https://docs.flutter.dev/testing/overview | Tổng quan best practices |
| Mocktail Guide | https://pub.dev/packages/mocktail#usage | Hướng dẫn sử dụng mocktail |

### Code Generation

| Bài viết | Link | Tóm tắt |
|----------|------|---------|
| Freezed Guide (Reso Coder) | https://resocoder.com/2021/01/25/freezed-data-class-union-in-one-dart-package/ | Hướng dẫn chi tiết freezed |
| JSON Serialization in Flutter | https://docs.flutter.dev/data-and-backend/serialization/json | Hướng dẫn chính thức JSON serialization |
| Code Generation in Dart | https://dart.dev/tools/build_runner | build_runner documentation |

---

## 🎥 Video

### Dependency Injection

| Video | Kênh | Nội dung |
|-------|------|----------|
| Flutter Dependency Injection For Beginners | Reso Coder | get_it + injectable setup |
| GetIt & Injectable - Full Setup | FilledStacks | Production-ready DI setup |
| Service Locator Pattern in Flutter | Flutter Mapp | Giải thích Service Locator pattern |

### Testing

| Video | Kênh | Nội dung |
|-------|------|----------|
| Flutter Testing For Beginners | Reso Coder | Unit test, widget test, integration test |
| Widget Testing - The Boring Flutter Show | Flutter (Official) | Widget testing deep dive |
| Mocktail Tutorial | Very Good Ventures | Mocking với mocktail |
| Integration Testing in Flutter | Flutter (Official) | Integration test guide |

### Code Generation

| Video | Kênh | Nội dung |
|-------|------|----------|
| Freezed Tutorial | Reso Coder | Immutable data classes với freezed |
| JSON Serialization in Flutter | Flutter (Official) | json_serializable guide |
| Build Runner Deep Dive | Andrea Bizzotto | Hiểu cách build_runner hoạt động |

---

## 🗂️ Tóm tắt pubspec.yaml

```yaml
# pubspec.yaml — tất cả packages cho Buổi 10
dependencies:
  flutter:
    sdk: flutter
  # DI
  get_it: ^7.6.4
  injectable: ^2.3.0
  # Code gen annotations
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  # DI code gen
  injectable_generator: ^2.3.0
  # Testing
  mocktail: ^1.0.4
  bloc_test: ^9.1.1
  # Code gen
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

---

## 🤖 AI Prompt Library — Buổi 10: DI & Testing

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Dependency Injection trong Flutter. Background: 4+ năm React (Context, Redux store, dependency injection containers).
Câu hỏi: get_it giống gì trong React ecosystem? @injectable annotation giống gì? Service Locator pattern vs Constructor Injection — khi nào dùng cái nào?
Yêu cầu: mapping với frontend concepts quen thuộc, giải thích bằng tiếng Việt.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần viết unit tests cho Clean Architecture Flutter project.
Tech stack: flutter_test, mocktail ^1.x.
Classes: GetProducts UseCase, ProductRepositoryImpl.
Mock: ProductRepository, ProductRemoteDataSource.
Constraints:
- mocktail (không phải mockito — không cần code gen).
- thenAnswer cho async methods.
- verify() cho side effects.
- Test cả success + failure paths.
Output: 2 test files hoàn chỉnh.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn test code sau:
[paste code]

Kiểm tra:
1. Mock setup đúng? extends Mock implements Interface?
2. thenAnswer vs thenReturn — dùng đúng cho async?
3. verify: side effects checked?
4. setUp/tearDown: fresh mocks mỗi test?
5. Coverage: happy path + error path + edge cases?
6. Test isolation: tests không depend on nhau?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi khi chạy Flutter tests:
[paste error message]

Test code:
[paste test file]

Context: dùng mocktail ^1.x, get_it ^7.x.
Cần: (1) Nguyên nhân (mock setup sai?), (2) Fix, (3) Pattern chuẩn.
```

---

## 🔗 Liên kết buổi học

- [Tổng quan buổi 10](00-overview.md)
- [Lý thuyết](01-ly-thuyet.md)
- [Ví dụ](02-vi-du.md)
- [Thực hành](03-thuc-hanh.md)
- [Buổi trước: Clean Architecture](../buoi-09-clean-architecture/00-overview.md)
