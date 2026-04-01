# Buổi 12: Local Storage & Data Persistence — Tài liệu tham khảo

## 📦 Packages

### Core Storage

| Package | Mô tả | pub.dev |
|---------|--------|---------|
| `shared_preferences` | Key-value storage (NSUserDefaults / SharedPrefs) | [shared_preferences](https://pub.dev/packages/shared_preferences) |
| `hive` | Lightweight NoSQL database (pure Dart) | [hive](https://pub.dev/packages/hive) |
| `hive_flutter` | Hive extensions cho Flutter (initFlutter, ValueListenableBuilder) | [hive_flutter](https://pub.dev/packages/hive_flutter) |
| `hive_generator` | Code generator cho Hive TypeAdapter | [hive_generator](https://pub.dev/packages/hive_generator) |
| `drift` | Type-safe SQLite wrapper (formerly Moor) | [drift](https://pub.dev/packages/drift) |
| `sqlite3_flutter_libs` | SQLite native libraries cho Flutter | [sqlite3_flutter_libs](https://pub.dev/packages/sqlite3_flutter_libs) |
| `drift_dev` | Code generator cho Drift | [drift_dev](https://pub.dev/packages/drift_dev) |

### Security & Utilities

| Package | Mô tả | pub.dev |
|---------|--------|---------|
| `flutter_secure_storage` | Encrypted storage (Keychain/Keystore) | [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) |
| `connectivity_plus` | Network connectivity detection | [connectivity_plus](https://pub.dev/packages/connectivity_plus) |
| `path_provider` | Lấy đường dẫn thư mục app (documents, cache, temp) | [path_provider](https://pub.dev/packages/path_provider) |
| `build_runner` | Code generation tool cho Dart | [build_runner](https://pub.dev/packages/build_runner) |

---

## 📖 Tài liệu chính thức

### SharedPreferences
- [shared_preferences — Flutter Favorite](https://pub.dev/packages/shared_preferences)
- [Flutter Cookbook: Store key-value data on disk](https://docs.flutter.dev/cookbook/persistence/key-value)

### Hive
- [Hive Documentation](https://docs.hivedb.dev/)
- [Hive GitHub](https://github.com/isar/hive)

### Drift (SQLite)
- [Drift Documentation](https://drift.simonbinder.eu/)
- [Drift — Getting Started](https://drift.simonbinder.eu/getting-started/)
- [Drift — Writing Queries](https://drift.simonbinder.eu/dart_api/select/)
- [Drift — Migrations](https://drift.simonbinder.eu/migrations/)
- [Drift — DAOs](https://drift.simonbinder.eu/dart_api/daos/)
- [Drift GitHub](https://github.com/simolus3/drift)

### Secure Storage
- [flutter_secure_storage Documentation](https://pub.dev/packages/flutter_secure_storage)
- [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)

### Connectivity
- [connectivity_plus Documentation](https://pub.dev/packages/connectivity_plus)

---

## 📝 Blog & Articles

### SharedPreferences
- [Using SharedPreferences in Flutter (flutter.dev)](https://docs.flutter.dev/cookbook/persistence/key-value)
- [SharedPreferences in Flutter: A Practical Guide](https://medium.com/flutter-community/shared-preferences-in-flutter-a-practical-guide-cd5c4e5c6c5c)

### Hive
- [Hive — Lightweight and Blazing Fast Key-Value Database](https://docs.hivedb.dev/)
- [Flutter Local Database: Hive — Complete Tutorial](https://resocoder.com/2019/09/30/hive-flutter-tutorial-lightweight-fast-database/)

### Drift
- [Drift: Reactive Persistence Library for Flutter & Dart](https://drift.simonbinder.eu/)
- [Flutter SQLite Database with Drift (Moor)](https://resocoder.com/2021/06/21/moor-drift-flutter-sqlite-database-tutorial/)

### Offline-First
- [Building Offline-First Apps with Flutter](https://medium.com/flutter-community/building-offline-first-apps-with-flutter-1a5ef6a7f9d2)
- [Offline-First Architecture Patterns](https://developer.android.com/topic/architecture/data-layer/offline-first)

### Caching
- [Caching Strategies in Flutter](https://medium.com/@nicholasfarris/caching-strategies-in-flutter-2c0b2a1d9a7d)

---

## 🎥 Video

### SharedPreferences
- Tìm trên YouTube: "Flutter SharedPreferences tutorial" (The Net Ninja)
- Tìm trên YouTube: "Flutter SharedPreferences official tutorial"

### Hive
- Tìm trên YouTube: "Hive Flutter tutorial 2024"
- Tìm trên YouTube: "Flutter Hive Reso Coder"

### Drift
- Tìm trên YouTube: "Drift Flutter tutorial 2024"
- Tìm trên YouTube: "Flutter Drift SQLite tutorial"

### Offline-First
- Tìm trên YouTube: "Flutter offline first architecture"

---

## 🔧 Tools

| Tool | Mô tả |
|------|--------|
| `dart run build_runner build` | Generate code cho Hive TypeAdapter và Drift |
| `dart run build_runner watch` | Auto-generate khi file thay đổi |
| `dart run build_runner build --delete-conflicting-outputs` | Build & xóa file conflict |
| DB Browser for SQLite | GUI tool xem SQLite database file |
| Android Studio Device File Explorer | Xem files trên Android device/emulator |

---

## 🤖 AI Prompt Library — Buổi 12: Local Storage & Data Persistence

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Local Storage trong Flutter. Background: 4+ năm React (localStorage, IndexedDB, Redux Persist).
Câu hỏi: SharedPreferences giống localStorage? Hive giống IndexedDB? Drift giống gì? Khi nào dùng cái nào?
Yêu cầu: mapping 1-1 với web storage concepts, giải thích trade-offs (speed vs query power vs encryption).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần setup Hive cho "Bookmarks" feature trong Flutter.
Tech stack: hive ^2.x, hive_flutter, hive_generator.
Model Bookmark: id, url, title, tags (List<String>), createdAt (DateTime).
Output: bookmark_model.dart (@HiveType) + bookmark_local_data_source.dart (CRUD).
Constraints: DateTime dùng millisecondsSinceEpoch, field numbers sequential, register adapter trước openBox.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn local storage code sau:
[paste code]

Kiểm tra:
1. Hive: @HiveField numbers sequential và stable? (không thay đổi existing numbers)
2. TypeAdapter registered TRƯỚC openBox?
3. DateTime handling đúng? (millisecondsSinceEpoch, không dùng DateTime directly)
4. Box lifecycle: close box khi không cần? Memory leak?
5. Error handling: box corruption, disk full?
6. Encryption: sensitive data dùng encrypted box?
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi với Hive/Drift trong Flutter:
[paste error message]

Setup:
[paste model + init code]

Context: [mô tả khi nào lỗi xảy ra — cold start, migration, sau update app version?]
Cần: (1) Nguyên nhân, (2) Fix, (3) Prevention cho lần sau.
```

---

## 📐 So sánh nhanh các giải pháp

```
┌─────────────────┬────────────────┬───────────────┬────────────────┬──────────────────┐
│                 │ SharedPrefs    │ Hive          │ Drift (SQLite) │ Secure Storage   │
├─────────────────┼────────────────┼───────────────┼────────────────┼──────────────────┤
│ Kiểu            │ Key-Value      │ NoSQL         │ Relational     │ Key-Value        │
│ Data phức tạp   │ ❌             │ ✅            │ ✅✅           │ ❌               │
│ Queries         │ get/set        │ Basic filter  │ Full SQL       │ get/set          │
│ Tốc độ          │ ⚡             │ ⚡⚡          │ ⚡             │ 🐌               │
│ Mã hóa          │ ❌             │ Optional      │ Optional       │ ✅ Built-in      │
│ Code gen        │ ❌             │ Optional      │ Required       │ ❌               │
│ Reactive        │ ❌             │ ✅ watch()    │ ✅ Stream      │ ❌               │
│ Use case        │ Settings       │ Cache, models │ Complex data   │ Tokens, secrets  │
└─────────────────┴────────────────┴───────────────┴────────────────┴──────────────────┘
```

---

## 📋 Cài đặt nhanh

```bash
# SharedPreferences
flutter pub add shared_preferences

# Hive
flutter pub add hive hive_flutter
flutter pub add --dev hive_generator build_runner

# Drift
flutter pub add drift sqlite3_flutter_libs path_provider path
flutter pub add --dev drift_dev build_runner

# Secure Storage
flutter pub add flutter_secure_storage

# Connectivity
flutter pub add connectivity_plus

# Code generation (sau khi tạo models)
dart run build_runner build --delete-conflicting-outputs
```
