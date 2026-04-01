# Buổi 09: Clean Architecture trong Flutter — Tài liệu tham khảo

---

## 📚 Bài viết & Blog

### Clean Architecture trong Flutter

| Tài liệu | Mô tả | Link |
|-----------|--------|------|
| **Reso Coder — Flutter Clean Architecture** | Series kinh điển, giải thích từng layer chi tiết với TDD. Đây là tài liệu #1 cho Flutter Clean Architecture | https://resocoder.com/flutter-clean-architecture-tdd/ |
| **Robert C. Martin — The Clean Architecture** | Bài gốc của Uncle Bob (2012) — nền tảng lý thuyết | https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html |
| **Very Good Ventures — Flutter Architecture** | Enterprise-level architecture patterns từ team Flutter consultancy hàng đầu | https://verygood.ventures/blog |

### Dependency Rule & SOLID

| Tài liệu | Mô tả |
|-----------|--------|
| **SOLID Principles in Dart** | 5 nguyên tắc SOLID áp dụng trong Dart, đặc biệt Dependency Inversion |
| **Dependency Inversion Principle** | Giải thích tại sao Domain layer định nghĩa interface, Data layer implement |

---

## 🎥 Video Tutorials

| Video | Nội dung | Ghi chú |
|-------|----------|---------|
| **Reso Coder — Flutter TDD Clean Architecture** (YouTube series) | Series ~13 videos, xây dựng app hoàn chỉnh với Clean Architecture + TDD | Kinh điển, khuyến khích xem |
| **Flutter Clean Architecture Full Course** trên YouTube | Nhiều channel có full course, search "Flutter Clean Architecture 2024" | Chọn video có nhiều views + recent |
| **Code With Andrea — Flutter App Architecture** | Andrea Bizzotto giải thích Riverpod + Architecture patterns | Phù hợp nếu dùng Riverpod thay BLoC |

> **Lưu ý:** Một số video có thể dùng phiên bản Flutter/Dart cũ. Hãy kiểm tra phiên bản trước khi làm theo.

---

## 📦 Packages hữu ích

### Core packages cho Clean Architecture

| Package | Mục đích | Sử dụng |
|---------|----------|---------|
| **fpdart** | Functional programming: `Either<Failure, Success>` để xử lý error | Domain & Data layer — thay thế try/catch bằng Either type |
| **equatable** | So sánh object theo value (không cần override == thủ công) | Entities, States, Events |
| **flutter_bloc** | State management (BLoC/Cubit) | Presentation layer |
| **get_it** | Service Locator cho Dependency Injection | Wiring các layers (Buổi 10) |
| **injectable** | Code generation cho get_it | Tự động generate DI code (Buổi 10) |

> ℹ️ **`fpdart`** là successor được maintain tích cực của `dartz` (đã ngừng maintain từ 2021). API tương tự: `Either<Failure, T>`, `Option<A>`, `right()`, `left()`.

### Data layer packages

| Package | Mục đích |
|---------|----------|
| **http** / **dio** | HTTP client cho API calls |
| **json_annotation** + **json_serializable** | Code generation cho fromJson/toJson |
| **shared_preferences** | Local storage đơn giản (key-value) |
| **sqflite** / **drift** | SQLite database cho local storage phức tạp |
| **hive** | NoSQL database nhẹ, nhanh |

### Ví dụ sử dụng `fpdart` Either

```dart
// Thay vì throw exception:
Future<User> getUser(String id) async {
  try {
    return await remoteDataSource.getUser(id);
  } catch (e) {
    throw ServerFailure(e.toString()); // ❌ throw thì caller phải try/catch
  }
}

// Dùng Either (fpdart) — explicit error handling:
Future<Either<Failure, User>> getUser(String id) async {
  try {
    final user = await remoteDataSource.getUser(id);
    return Right(user);     // ✅ Success
  } catch (e) {
    return Left(ServerFailure(e.toString()));  // ✅ Failure
  }
}

// Caller:
final result = await getUserUseCase('123');
result.fold(
  (failure) => emit(UserError(failure.message)),  // Left = error
  (user) => emit(UserLoaded(user)),                // Right = success
);
```

> **Ghi chú:** Dùng `fpdart` Either là optional — nhiều team chỉ dùng try/catch thông thường. Either hữu ích khi muốn **ép buộc** caller xử lý cả success và failure.

### Ví dụ sử dụng `equatable`

```dart
// Không có equatable — phải override thủ công:
class User {
  final String id;
  final String name;
  // ...

  @override
  bool operator ==(Object other) =>
    identical(this, other) ||
    other is User && id == other.id && name == other.name;

  @override
  int get hashCode => id.hashCode ^ name.hashCode;
}

// Với equatable — tự động:
class User extends Equatable {
  final String id;
  final String name;

  const User({required this.id, required this.name});

  @override
  List<Object?> get props => [id, name]; // Chỉ cần list props
}
```

---

## 🏗️ Project Templates & Examples

### GitHub repositories tham khảo

| Repo | Mô tả |
|------|--------|
| Search "flutter clean architecture" trên GitHub | Nhiều boilerplate projects để tham khảo cấu trúc |
| **Reso Coder's TDD Clean Architecture repo** | Source code đi kèm series blog/video |

> **Cách học hiệu quả:** Clone 1 repo, đọc từ Domain layer ra → hiểu flow trước khi tự viết.

---

## 📖 Sách

| Sách | Tác giả | Ghi chú |
|------|---------|---------|
| **Clean Architecture: A Craftsman's Guide** | Robert C. Martin | Sách gốc — lý thuyết nền tảng, không specific cho Flutter |
| **Clean Code** | Robert C. Martin | Bổ trợ — viết code sạch ở mức function/class |

---

## ⚠️ Lưu ý quan trọng

1. **Không có official Flutter docs cho Clean Architecture** — đây là community-driven pattern, Flutter không recommend architecture cụ thể nào
2. **Clean Architecture có nhiều biến thể** — mỗi team/blog có thể có cách interpret khác nhau. Nguyên tắc cốt lõi (Dependency Rule, layer separation) là giống nhau
3. **Không phải one-size-fits-all** — Đánh giá quy mô dự án trước khi áp dụng
4. Một số tài liệu dùng `fpdart` + `Either`, một số dùng `try/catch` thuần — cả hai đều hợp lệ

---

## 🤖 AI Prompt Library — Buổi 09: Clean Architecture

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Clean Architecture cho Flutter. Background: 4+ năm React (feature folders, hooks, context).
Câu hỏi: Domain/Data/Presentation trong Clean Architecture tương đương pattern nào trong React? Entity giống gì? Use Case giống custom hook? Repository Pattern giống service layer không?
Yêu cầu: mapping 1-1 với React/frontend concepts, giải thích Dependency Rule bằng analogy đơn giản.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần scaffold Clean Architecture cho "Product Catalog" feature.
Tech stack: Flutter 3.x, flutter_bloc ^8.x, dartz (Either), injectable.
3 Layers cần tạo:
- Domain: ProductEntity, ProductRepository (abstract), GetProducts use case.
- Data: ProductModel (DTO), ProductRemoteDataSource, ProductRepositoryImpl.
- Presentation: ProductBloc, ProductScreen.
Constraints:
- Dependency Rule: Domain → nothing, Data → Domain, Presentation → Domain.
- Return Either<Failure, List<ProductEntity>> cho use case.
- DataSource abstract (để mock).
Output: folder structure + all files (class stubs, no implementation).
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Clean Architecture code sau:
[paste folder structure + key files]

Kiểm tra theo thứ tự:
1. Dependency Rule: Domain imports Data? Presentation imports Data directly?
2. Repository: abstract ở Domain, impl ở Data?
3. Use Cases: single responsibility? Return Either?
4. DTO vs Entity: UserModel ≠ UserEntity? Mapping method tồn tại?
5. DataSource: abstract class (testable)?
6. Presentation: dùng Use Cases, không dùng Repository trực tiếp?
Liệt kê: Critical (Dependency Rule violations) → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp vấn đề architecture trong Flutter Clean Architecture project:
[mô tả vấn đề: import circular, layer violation, unclear responsibility]

Folder structure:
[paste tree]

Cần: (1) Identify layer violations, (2) Suggest correct import direction, (3) Refactoring steps, (4) Prevention rules.
```

---

## 🔗 Liên kết buổi tiếp theo

**Buổi 10: Dependency Injection (get_it + injectable)** — Học cách tự động hóa việc wiring các layers mà buổi này đang làm thủ công.

Packages cần chuẩn bị:
```yaml
dependencies:
  get_it: ^7.6.4
  injectable: ^2.3.0

dev_dependencies:
  injectable_generator: ^2.3.0
  build_runner: ^2.4.0
```
