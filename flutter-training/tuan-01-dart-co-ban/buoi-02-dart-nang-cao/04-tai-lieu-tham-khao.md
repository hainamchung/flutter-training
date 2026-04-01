# Buổi 02 — Tài liệu tham khảo: Dart nâng cao — OOP, Async, Collections

> **Buổi 2/16** · **Cập nhật:** 2026-03-31

---

## 📚 Tài liệu chính thức (Official)

### Dart Language

| Tài liệu                        | Link                                                      | Ghi chú                                          |
|----------------------------------|------------------------------------------------------------|---------------------------------------------------|
| Dart Classes                     | https://dart.dev/language/classes                          | ⭐ Bắt buộc đọc — class, constructor, extends     |
| Dart Mixins                      | https://dart.dev/language/mixins                           | Mixin syntax và best practices                    |
| Dart Class Modifiers             | https://dart.dev/language/class-modifiers                  | sealed, final, base, interface (Dart 3)           |
| Dart Patterns                    | https://dart.dev/language/patterns                         | ⭐ Pattern matching, destructuring (Dart 3)        |
| Dart Branches (switch)           | https://dart.dev/language/branches                         | Switch expression, exhaustive check               |
| Dart Async Programming           | https://dart.dev/language/async                            | ⭐ async/await, Future, Stream fundamentals        |
| Dart Concurrency                 | https://dart.dev/language/concurrency                      | Event loop, isolates giải thích chi tiết          |
| Dart Collections                 | https://dart.dev/language/collections                      | List, Map, Set, spread, collection if/for         |
| Dart Extension Methods           | https://dart.dev/language/extension-methods                | Syntax và use cases                               |
| Dart Records                     | https://dart.dev/language/records                          | Record types (Dart 3)                             |
| Dart Error Handling              | https://dart.dev/language/error-handling                   | try/catch/finally, throw                          |
| Dart Enums                       | https://dart.dev/language/enums                            | Enhanced enums với fields, methods                |

### Dart Library Tour

| Tài liệu                        | Link                                                      | Ghi chú                                          |
|----------------------------------|------------------------------------------------------------|---------------------------------------------------|
| dart:core Library                | https://dart.dev/libraries/dart-core                       | String, int, List, Map, Set, DateTime             |
| dart:async Library               | https://dart.dev/libraries/dart-async                      | Future, Stream, Completer, StreamController       |

---

## 📝 Blogs & Articles

| Tài liệu                                          | Link                                                             | Ghi chú                                     |
|----------------------------------------------------|------------------------------------------------------------------|----------------------------------------------|
| Dart 3 Records & Patterns — Official Blog          | https://medium.com/dartlang/announcing-dart-3-53f065a10635       | Tổng quan tính năng mới Dart 3               |
| Sealed Classes in Dart 3                           | https://dart.dev/language/class-modifiers#sealed                 | Hướng dẫn sealed class chi tiết              |
| Understanding Dart Streams                         | https://dart.dev/tutorials/language/streams                      | ⭐ Tutorial stream chính thức                 |
| Effective Dart                                     | https://dart.dev/effective-dart                                  | ⭐ Style guide và best practices              |
| Effective Dart — Usage                             | https://dart.dev/effective-dart/usage                            | Cách dùng collections, async, classes đúng   |
| Dart Asynchronous Programming: Futures & Streams   | https://dart.dev/codelabs/async-await                            | Codelab hands-on cho async                   |

---

## 🎥 Video

| Video                                                | Kênh          | Link                                                          | Ghi chú                                |
|------------------------------------------------------|---------------|---------------------------------------------------------------|----------------------------------------|
| Dart OOP Crash Course                                | Vandad Nahavandipoor | https://www.youtube.com/watch?v=5WnODkC6gUE               | OOP concepts trong 30 phút             |
| Dart Async Explained — Futures, Streams, async/await | Fireship       | https://www.youtube.com/watch?v=OTS-ap9_aXc                  | ⭐ Async giải thích trực quan            |
| Dart Mixins Explained                                | Flutter Mapp   | https://www.youtube.com/watch?v=JegP4Wru1eo                  | Mixin nên xem nếu chưa rõ              |
| Dart 3 Records, Patterns, and Class Modifiers        | Flutter        | https://www.youtube.com/watch?v=KhYTFglbF2k                  | Tính năng mới Dart 3 chính thức         |
| Dart Sealed Classes & Pattern Matching               | Andrea Bizzotto | https://www.youtube.com/watch?v=FHeReTLKEZY                 | Sealed class thực hành                  |
| Understanding Event Loop in Dart                     | Majid Hajian   | https://www.youtube.com/watch?v=vl_AaCgudcY                  | Event loop, microtask queue giải thích  |

---

## 📦 Packages

> **Chưa cần package bên ngoài ở buổi này.**
>
> Buổi 2 tập trung vào Dart core language features — tất cả đều có sẵn trong Dart SDK.
> Các package bên ngoài sẽ được giới thiệu ở các buổi sau.
>
> Ghi chú: Một số package liên quan đến nội dung hôm nay (sẽ dùng sau):
>
> | Package       | Link                                | Liên quan             | Dùng ở buổi |
> |---------------|-------------------------------------|-----------------------|--------------|
> | `freezed`     | https://pub.dev/packages/freezed    | Code generation cho sealed class, union types | Buổi 09+ |
> | `fpdart`      | https://pub.dev/packages/fpdart     | Functional programming, Either type (successor của dartz) | Tham khảo |
> | `rxdart`      | https://pub.dev/packages/rxdart     | Extensions cho Stream (giống RxJS) | Buổi 09+ |
>
> Khi cần tìm package: https://pub.dev

---

## 🔗 Tham khảo nhanh

### Dart Cheatsheet

| Chủ đề                  | Syntax nhanh                                                |
|--------------------------|-------------------------------------------------------------|
| Class                    | `class A { final String x; A(this.x); }`                   |
| Named constructor        | `A.fromJson(Map j) : x = j['x'];`                          |
| Factory constructor      | `factory A.create() => A('default');`                       |
| Inheritance              | `class B extends A { B(super.x); }`                        |
| Mixin                    | `mixin M { void foo() {} }` → `class B with M {}`          |
| Sealed class             | `sealed class R {}` → exhaustive switch                     |
| Future                   | `Future<T> f() async { return await ...; }`                 |
| Stream                   | `Stream<T> s() async* { yield value; }`                     |
| Extension                | `extension on String { String get x => ...; }`              |
| Record                   | `(String, int) r = ('a', 1); r.$1;`                        |
| Pattern matching         | `switch (x) { Pattern() => result, }`                       |
| Collection if            | `[if (cond) item]`                                          |
| Collection for           | `[for (var i in list) transform(i)]`                        |
| Spread                   | `[...list1, ...list2]`                                      |

---

## 🤖 AI Prompt Library — Buổi 02: Dart nâng cao

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Sealed class và Pattern matching trong Dart 3. Background: 4+ năm TypeScript với discriminated unions.
Câu hỏi: Sealed class trong Dart 3 so sánh thế nào với TypeScript discriminated unions? Exhaustive switch check hoạt động ra sao? Khi nào dùng sealed class vs abstract class vs enum?
Yêu cầu: giải thích bằng tiếng Việt, so sánh side-by-side với TypeScript, kèm code Dart 3.x minh họa sealed class + switch expression.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần tạo Dart file implement Result pattern cho async operations.
Tech stack: Dart 3.x, sound null safety.
Constraints:
- Sealed class Result<T> với 2 subclass: Success<T>(T data), Failure<T>(AppError error).
- AppError cũng là sealed class: NetworkError, ServerError(int statusCode), ValidationError(Map<String, String> fieldErrors).
- Extension on Result<T>: fold<R>(R Function(T) onSuccess, R Function(AppError) onFailure).
- Helper: Future<Result<T>> tryCatch<T>(Future<T> Function() action) — wrap try/catch thành Result.
Output: 1 file result.dart hoàn chỉnh, sẵn sàng import trong project.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Dart code sau:
[paste code]

Kiểm tra theo thứ tự:
1. OOP: sealed class dùng đúng chưa? Có cần sealed không hay abstract đủ?
2. Collections: dùng functional operators (map, where, fold) thay vì for loop?
3. Async: Future chain có handle error đúng? Có missing await?
4. Pattern matching: switch expression exhaustive? Có default thừa?
5. Null safety: có dynamic hoặc ! operator không cần thiết?
6. Naming conventions: camelCase, PascalCase đúng chỗ?
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Dart:
[paste error message đầy đủ]

Code liên quan:
[paste đoạn code gây lỗi]

Context: đang implement OOP pattern (sealed class, mixin, generics) trong Dart 3.x, sound null safety.
Cần: (1) Giải thích nguyên nhân bằng tiếng Việt, (2) Cách fix cụ thể, (3) Nếu liên quan đến Dart 3 syntax mới (sealed, records, patterns), so sánh với cách cũ Dart 2.
```

---

## ➡️ Tiếp theo

Buổi 03 sẽ bắt đầu **Flutter cơ bản** — Widget tree, StatelessWidget, và tạo UI đầu tiên. Hãy đảm bảo bạn đã nắm vững Dart OOP và Async trước khi sang Flutter!
