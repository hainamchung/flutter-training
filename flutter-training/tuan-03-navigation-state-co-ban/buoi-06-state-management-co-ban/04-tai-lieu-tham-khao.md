# Buổi 06: State Management Cơ Bản — Tài liệu tham khảo

## 📚 Tài liệu chính thức (Official)

### Flutter Docs

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| State Management Introduction | https://docs.flutter.dev/data-and-backend/state-mgmt/intro | Tổng quan ephemeral vs app state |
| Simple State Management | https://docs.flutter.dev/data-and-backend/state-mgmt/simple | Hướng dẫn Provider từ Flutter team |
| State Management Options | https://docs.flutter.dev/data-and-backend/state-mgmt/options | Danh sách các giải pháp |
| Flutter Forms | https://docs.flutter.dev/cookbook/forms | Cookbook chính thức về Forms |
| Form Validation | https://docs.flutter.dev/cookbook/forms/validation | Hướng dẫn validation step-by-step |
| Retrieve Text Input | https://docs.flutter.dev/cookbook/forms/retrieve-input | TextEditingController |
| Handle Changes to TextField | https://docs.flutter.dev/cookbook/forms/text-field-changes | onChanged vs controller |

### API Reference

| Class / Widget | Link | Mô tả |
|----------------|------|-------|
| `ChangeNotifier` | https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html | Base class cho observable objects |
| `InheritedWidget` | https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html | Cơ chế truyền data xuống tree |
| `Form` | https://api.flutter.dev/flutter/widgets/Form-class.html | Form container widget |
| `FormState` | https://api.flutter.dev/flutter/widgets/FormState-class.html | State của Form — validate, save, reset |
| `TextFormField` | https://api.flutter.dev/flutter/material/TextFormField-class.html | Input field tích hợp form validation |
| `TextEditingController` | https://api.flutter.dev/flutter/widgets/TextEditingController-class.html | Controller cho text input |
| `GlobalKey` | https://api.flutter.dev/flutter/widgets/GlobalKey-class.html | Key để truy cập state từ bên ngoài |

### Provider Package

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Provider (pub.dev) | https://pub.dev/packages/provider | Package chính, README rất chi tiết |
| Provider API Docs | https://pub.dev/documentation/provider/latest/ | Tất cả classes và methods |
| Migration Guide | https://pub.dev/packages/provider#migration-from-v3-to-v4 | Nếu gặp code cũ |

---

## 📝 Bài viết & Blog

### State Management

| Bài viết | Nguồn | Nội dung chính |
|----------|-------|---------------|
| Flutter State Management in 2024 | Medium / Flutter Community | So sánh tổng quan các giải pháp |
| Provider – A pragmatic state management | Reso Coder | Tutorial Provider step-by-step |
| InheritedWidget explained | Didier Boelens | Deep dive cách InheritedWidget hoạt động |
| setState vs Provider vs BLoC | Fireship.io | So sánh nhanh 3 approaches |

### Forms

| Bài viết | Nguồn | Nội dung chính |
|----------|-------|---------------|
| Flutter Form Validation – Complete Guide | FilledStacks | Form patterns production-ready |
| Building Forms in Flutter | Medium / Flutter | Best practices cho complex forms |

---

## 🎥 Video

| Video | Kênh | Thời lượng | Nội dung |
|-------|------|-----------|----------|
| State Management Explained | Flutter (official) | ~12 min | Tổng quan từ Flutter team |
| Provider Tutorial for Beginners | Net Ninja | ~30 min | Hướng dẫn Provider từ đầu |
| Flutter State Management – Full Course | Vandad Nahavandipoor | ~2 hrs | Khóa học đầy đủ |
| InheritedWidget Explained | Flutter Widget of the Week | ~3 min | Giải thích nhanh |
| Flutter Forms & Validation | HeyFlutter | ~20 min | Form tutorial đầy đủ |

---

## 📦 Packages liên quan

### State Management

| Package | pub.dev | Mô tả | Khi nào dùng |
|---------|---------|-------|-------------|
| `provider` | https://pub.dev/packages/provider | State management đơn giản, Google-backed | Default choice, đặc biệt cho người mới |
| `flutter_riverpod` | https://pub.dev/packages/flutter_riverpod | Provider v2 — compile-safe, no context | Khi cần type-safe + testability cao |
| `flutter_bloc` | https://pub.dev/packages/flutter_bloc | Event-driven state management | Enterprise apps, team lớn |
| `get` | https://pub.dev/packages/get | All-in-one solution | Prototype nhanh (cẩn thận code quality) |

### Forms

| Package | pub.dev | Mô tả | Khi nào dùng |
|---------|---------|-------|-------------|
| `flutter_form_builder` | https://pub.dev/packages/flutter_form_builder | Form builder với nhiều field types | Forms phức tạp, nhiều loại input |
| `form_validator` | https://pub.dev/packages/form_validator | Validator utilities | Validators tái sử dụng |

---

## 🔗 Tài liệu bổ sung

### So sánh với Frontend

| Nếu bạn biết... | Đọc thêm... | Giúp hiểu... |
|-----------------|-------------|--------------|
| React Context | Provider README (phần comparison) | Provider wraps InheritedWidget giống Context wraps values |
| Redux | BLoC documentation | BLoC pattern tương tự Redux (actions → reducer → state) |
| Vue Pinia | Provider examples | ChangeNotifier tương tự Pinia store |
| React Hook Form | Flutter Forms cookbook | Cách tiếp cận khác nhau nhưng concept giống (validate, submit, error display) |

---

## 📖 Thứ tự đọc khuyến nghị

Cho người mới bắt đầu:

```
1. Flutter docs: State Management Introduction
   → Hiểu khái niệm cơ bản

2. Flutter docs: Simple State Management
   → Học Provider cách chính thống

3. Provider README trên pub.dev
   → Nắm đầy đủ API

4. Flutter docs: Form Validation cookbook
   → Học Form cơ bản

5. Video: Net Ninja Provider Tutorial
   → Xem demo thực tế

6. Flutter docs: State Management Options
   → Mở rộng hiểu biết về các giải pháp khác
```

---

## ⚡ Cheat Sheet nhanh

### Provider

```dart
// 1. Model
class MyModel extends ChangeNotifier {
  int _value = 0;
  int get value => _value;
  void update() { _value++; notifyListeners(); }
}

// 2. Provide
ChangeNotifierProvider(create: (_) => MyModel(), child: App())

// 3. Consume
context.watch<MyModel>()    // rebuild khi thay đổi
context.read<MyModel>()     // đọc 1 lần
context.select<MyModel, int>((m) => m.value) // chỉ listen 1 field
Consumer<MyModel>(builder: (ctx, model, child) => ...)
```

### Form

```dart
// 1. Key
final _formKey = GlobalKey<FormState>();

// 2. Form widget
Form(key: _formKey, child: Column(children: [
  TextFormField(validator: (v) => v!.isEmpty ? 'Required' : null),
]))

// 3. Actions
_formKey.currentState!.validate()  // chạy tất cả validators
_formKey.currentState!.save()      // gọi onSaved trên tất cả fields
_formKey.currentState!.reset()     // reset về giá trị ban đầu
```

---

## 🤖 AI Prompt Library — Buổi 06: State Management cơ bản

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Flutter State Management cơ bản. Background: 4+ năm React (useState, useContext, Redux).
Câu hỏi: setState trong Flutter giống useState React ở điểm nào? InheritedWidget giống useContext hay React Context? Provider giống Redux hay Context + useReducer?
Yêu cầu: mapping 1-1 với React concepts, giải thích bằng tiếng Việt, kèm code Flutter minh họa.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần implement state management cho shopping cart dùng Provider.
Tech stack: Flutter 3.x, provider ^6.x.
Features: addItem, removeItem, totalPrice, itemCount, clearCart.
Constraints:
- CartModel extends ChangeNotifier.
- Dùng context.watch trong build() cho UI update.
- Dùng context.read trong onPressed callbacks.
- Consumer widget chỉ wrap phần cần rebuild (badge count, cart list).
- MultiProvider nếu có thêm UserModel.
Output: cart_model.dart + main.dart + cart_screen.dart.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn Provider setup sau:
[paste code]

Kiểm tra theo thứ tự:
1. context.watch dùng trong build()? context.read dùng trong callbacks?
2. Consumer widget có wrap đúng phần cần rebuild không (hay wrap cả Scaffold)?
3. ChangeNotifier có gọi notifyListeners() sau mỗi state change?
4. MultiProvider: thứ tự providers đúng? (dependent provider đặt sau)
5. Dispose: ChangeNotifier có cần dispose manual không?
6. Performance: có select() cho trường hợp chỉ cần 1 field?
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter Provider:
[paste error message]

Code liên quan:
[paste provider setup + screen code gây lỗi]

Context: dùng provider ^6.x, có MultiProvider với CartModel + UserModel.
Cần: (1) Giải thích nguyên nhân, (2) Check watch vs read usage, (3) Fix cụ thể, (4) Pattern chuẩn để tránh lỗi Provider này.
```
