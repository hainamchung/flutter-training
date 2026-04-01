# Buổi 05: Navigation & Routing — Tài liệu tham khảo

---

## 📖 Tài liệu chính thức

| Tài liệu | Link | Mô tả |
|-----------|------|--------|
| Flutter Navigation Cookbook | https://docs.flutter.dev/cookbook/navigation | Cookbook chính thức với các recipe push/pop, named routes, pass data, return data |
| Flutter Navigation Overview | https://docs.flutter.dev/ui/navigation | Tổng quan navigation trong Flutter, so sánh Navigator 1.0 và 2.0 |
| go_router package (pub.dev) | https://pub.dev/packages/go_router | Package chính thức, README chi tiết, API reference |
| go_router migration guide | https://docs.flutter.dev/ui/navigation/url-strategies | Hướng dẫn URL strategies và navigation patterns |
| Deep Linking (Flutter docs) | https://docs.flutter.dev/ui/navigation/deep-linking | Cấu hình deep linking cho iOS và Android |

## 📝 Blog & Bài viết

| Bài viết | Link | Mô tả |
|----------|------|--------|
| Flutter Navigation 2.0 Guide | https://medium.com/flutter/learning-flutters-new-navigation-and-routing-system-7c9068155ade | Giải thích chi tiết Navigator 2.0 (by Flutter team) |
| GoRouter Complete Guide | https://codewithandrea.com/articles/flutter-navigation-gorouter-go-vs-push/ | So sánh `go()` vs `push()` chi tiết (Andrea Bizzotto) |
| Migrate to GoRouter | https://docs.flutter.dev/ui/navigation | Migration guide từ Navigator 1.0 sang GoRouter |
| StatefulShellRoute Guide | https://pub.dev/documentation/go_router/latest/topics/Configuration-topic.html | Cách dùng StatefulShellRoute cho tab navigation |

## 🎥 Video

| Video | Link | Mô tả |
|-------|------|--------|
| Navigation & Routing (Flutter YouTube) | https://www.youtube.com/watch?v=nyvwx7o277U | Video chính thức về navigation patterns |
| GoRouter Package (Package of the Week) | https://www.youtube.com/watch?v=b6Z885Z46cU | Giới thiệu GoRouter từ Flutter team |
| Deep Linking in Flutter | https://www.youtube.com/watch?v=KNAb2XL7k2g | Hướng dẫn setup deep linking |

## 📦 Packages liên quan

| Package | Link | Mô tả | Stars |
|---------|------|--------|-------|
| `go_router` | https://pub.dev/packages/go_router | Declarative routing, deep linking, URL sync | ⭐ Official |
| `auto_route` | https://pub.dev/packages/auto_route | Code-generation based routing, type-safe | ⭐⭐ Popular |
| `beamer` | https://pub.dev/packages/beamer | Navigator 2.0 wrapper, URL-based | Alternative |

### So sánh nhanh packages

| Feature | go_router | auto_route | beamer |
|---------|-----------|------------|--------|
| Maintained by | Flutter team | Community | Community |
| Code generation | Không | Có (build_runner) | Không |
| Type-safe params | Manual | Tự động generate | Manual |
| Learning curve | Thấp | Trung bình | Trung bình |
| Deep linking | ✅ | ✅ | ✅ |
| Nested navigation | ✅ (ShellRoute) | ✅ (AutoTabsRouter) | ✅ (BeamLocation) |
| Web URL sync | ✅ | ✅ | ✅ |

> 💡 **Khuyến nghị**: Dùng **go_router** cho hầu hết project. Nó là package chính thức, đơn giản, và đủ mạnh cho đa số use cases.

## 🔧 Tools hỗ trợ

| Tool | Mô tả |
|------|--------|
| Flutter DevTools | Xem widget tree, navigation stack, performance |
| `adb shell am start` | Test deep link trên Android emulator |
| `xcrun simctl openurl` | Test deep link trên iOS Simulator |
| `flutter run -d chrome` | Chạy app trên web để test URL routing |

---

## 🤖 AI Prompt Library — Buổi 05: Navigation & Routing

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học Flutter Navigation với GoRouter. Background: 4+ năm React (react-router-dom v6).
Câu hỏi: go() vs push() trong GoRouter khác gì? So sánh với navigate() vs Link trong react-router. Khi nào dùng ShellRoute vs StatefulShellRoute? Tương đương Layout Route trong react-router không?
Yêu cầu: giải thích bằng tiếng Việt, mapping 1-1 với react-router concepts, kèm code GoRouter v14 minh họa.
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần setup GoRouter cho app có bottom navigation + auth flow.
Tech stack: Flutter 3.x, go_router ^14.x.
Constraints:
- 3 bottom tabs: Home, Search, Profile (StatefulShellRoute).
- Nested routes: /home/product/:id (push trong tab).
- Auth routes: /login, /register (ngoài ShellRoute, ẩn bottom nav).
- Redirect: Profile tab yêu cầu auth, exclude /login khỏi redirect.
- Route names dùng enum.
Output: 1 file router.dart hoàn chỉnh + 1 file route_names.dart.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn GoRouter config sau:
[paste code]

Kiểm tra theo thứ tự:
1. Redirect loop: login route có bị redirect lại không?
2. Tab state: dùng StatefulShellRoute hay ShellRoute? State giữ đúng?
3. Route params: có validate type (int.tryParse)? Null handling?
4. Naming: dùng enum/constants hay String literals dễ typo?
5. Error handling: có errorBuilder cho 404?
6. go() vs push(): dùng đúng context? (go = replace stack, push = add)
Liệt kê: Critical → Warning → Suggestion.
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi Flutter navigation:
[paste error message]

Code liên quan:
[paste router config + screen code gây lỗi]

Context: dùng GoRouter ^14.x, có bottom tabs + redirect guard.
Cần: (1) Giải thích nguyên nhân, (2) Check xem có phải redirect loop không, (3) Fix cụ thể, (4) Pattern chuẩn để tránh lỗi navigation này.
```

## 📐 Tham khảo thêm

- **Material Design Navigation** : https://m3.material.io/foundations/layout/understanding-layout/overview — Hướng dẫn UX/UI cho navigation patterns
- **Human Interface Guidelines (iOS)** : https://developer.apple.com/design/human-interface-guidelines/navigation — Apple guidelines cho navigation
- **Android Navigation** : https://developer.android.com/guide/navigation — Android navigation component (tham khảo concept)
