# Buổi 16: CI/CD & Production — Tài Liệu Tham Khảo

## 📚 Tài liệu chính thức

| Tài liệu | Link | Ghi chú |
|-----------|------|---------|
| Flutter Deployment (Android) | https://docs.flutter.dev/deployment/android | Build, sign, deploy lên Play Store |
| Flutter Deployment (iOS) | https://docs.flutter.dev/deployment/ios | Build, sign, deploy lên App Store |
| Flutter Build Modes | https://docs.flutter.dev/testing/build-modes | Debug, Profile, Release |
| Flutter Obfuscation | https://docs.flutter.dev/deployment/obfuscate | Obfuscate Dart code |
| Fastlane Docs | https://docs.fastlane.tools | Automation cho iOS & Android |
| Fastlane — Flutter | https://docs.flutter.dev/deployment/cd#fastlane | Tích hợp Fastlane với Flutter |
| GitHub Actions Docs | https://docs.github.com/en/actions | CI/CD platform |
| GitHub Actions — Flutter | https://github.com/subosito/flutter-action | Flutter action cho GitHub Actions |
| App Store Connect | https://developer.apple.com/app-store-connect/ | Quản lý app trên App Store |
| Google Play Console | https://play.google.com/console | Quản lý app trên Play Store |

## 📦 Packages hữu ích

### Error Tracking & Monitoring

| Package | Mô tả | Link |
|---------|--------|------|
| `sentry_flutter` | Error tracking & performance monitoring | https://pub.dev/packages/sentry_flutter |
| `firebase_crashlytics` | Crash reporting từ Firebase | https://pub.dev/packages/firebase_crashlytics |
| `firebase_analytics` | App analytics từ Firebase | https://pub.dev/packages/firebase_analytics |
| `firebase_performance` | Performance monitoring | https://pub.dev/packages/firebase_performance |

### OTA Updates & Distribution

| Package/Tool | Mô tả | Link |
|--------------|--------|------|
| Shorebird | Code push / OTA updates cho Flutter | https://shorebird.dev |
| Firebase App Distribution | Distribute pre-release builds | https://firebase.google.com/docs/app-distribution |

### App Configuration

| Package | Mô tả | Link |
|---------|--------|------|
| `firebase_remote_config` | Feature flags & remote configuration | https://pub.dev/packages/firebase_remote_config |
| `package_info_plus` | Lấy app version, build number | https://pub.dev/packages/package_info_plus |
| `flutter_dotenv` | Load .env files | https://pub.dev/packages/flutter_dotenv |

## 🛠️ CI/CD Tools & Platforms

| Tool | Mô tả | Free Tier | Link |
|------|--------|-----------|------|
| **GitHub Actions** | CI/CD tích hợp GitHub | 2,000 min/tháng (free) | https://github.com/features/actions |
| **Fastlane** | Automation cho mobile deployment | Open source | https://fastlane.tools |
| **Codemagic** | CI/CD chuyên cho Flutter | 500 min/tháng (free) | https://codemagic.io |
| **Bitrise** | CI/CD cho mobile | Free cho open source | https://bitrise.io |
| **App Center** | Build, test, distribute (Microsoft) | Free tier available | https://appcenter.ms |
| **CircleCI** | CI/CD platform | Free tier available | https://circleci.com |
| **GitLab CI** | CI/CD tích hợp GitLab | 400 min/tháng (free) | https://docs.gitlab.com/ee/ci/ |

### So sánh CI/CD platforms cho Flutter

| Feature | GitHub Actions | Codemagic | Bitrise |
|---------|---------------|-----------|---------|
| Flutter-specific | Action cộng đồng | Native support | Có Flutter steps |
| iOS builds | macOS runners | macOS M1/M2 | macOS runners |
| Pricing model | Per-minute | Per-minute | Per-minute |
| Fastlane integration | Manual setup | Built-in option | Built-in option |
| Ease of setup | YAML config | GUI + YAML | GUI + YAML |
| GitHub integration | Native | Tốt | Tốt |

## 📖 Blogs & Tutorials

### CI/CD

- [Flutter CI/CD with GitHub Actions](https://docs.flutter.dev/deployment/cd) — Official guide
- [Automating Flutter builds with Fastlane](https://docs.fastlane.tools/getting-started/cross-platform/flutter/) — Fastlane + Flutter
- [Codemagic Flutter CI/CD Guide](https://blog.codemagic.io/getting-started-with-codemagic/) — Alternative CI/CD platform

### App Store Submission

- [Preparing an iOS app for release](https://docs.flutter.dev/deployment/ios) — Official Flutter guide
- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) — Apple's requirements
- [Google Play Launch Checklist](https://developer.android.com/distribute/best-practices/launch/launch-checklist) — Google's requirements

### Code Signing

- [Android App Signing](https://developer.android.com/studio/publish/app-signing) — Official Android guide
- [iOS Code Signing Guide](https://developer.apple.com/support/code-signing/) — Apple's guide
- [Fastlane Match](https://docs.fastlane.tools/actions/match/) — Certificate management

### Monitoring & Analytics

- [Firebase Crashlytics for Flutter](https://firebase.google.com/docs/crashlytics/get-started?platform=flutter) — Crash reporting setup
- [Sentry for Flutter](https://docs.sentry.io/platforms/flutter/) — Error monitoring

---

## 🤖 AI Prompt Library — Buổi 16: CI/CD & Production

Copy và dùng trực tiếp với ChatGPT, Claude, hoặc Copilot Chat.

### 1. Prompt hiểu concept (khi đọc lý thuyết không hiểu)
```text
Tôi đang học CI/CD cho Flutter. Background: 4+ năm React (GitHub Actions, Vercel, Netlify CI).
Câu hỏi: Flutter CI/CD flow khác web app CI/CD thế nào? Code signing iOS/Android equivalent gì trên web? Fastlane giống tool nào? App bundle vs APK giống gì?
Yêu cầu: mapping với web deployment concepts, highlight mobile-specific (code signing, store review, staged rollout).
```

### 2. Prompt gen code (khi bắt đầu làm bài tập)
```text
Tôi cần GitHub Actions workflow cho Flutter CI.
Trigger: PR to main.
Steps: analyze → test --coverage → coverage check 80% → build APK.
Cache: pub + Gradle.
Secrets: none needed for CI.
Output: .github/workflows/ci.yml.
```

### 3. Prompt review code (khi muốn kiểm tra chất lượng)
```text
Review đoạn CI/CD config sau:
[paste YAML workflow + Fastlane config]

Kiểm tra:
1. Secrets hardcoded? (API keys, passwords in YAML/Fastfile?)
2. Build flags: --obfuscate, --split-debug-info, --tree-shake-icons?
3. Cache: pub, Gradle, CocoaPods?
4. CI steps order: analyze → test → build?
5. CD: debug symbols uploaded for crash reporting?
6. iOS: code signing via match? (not manual)
```

### 4. Prompt debug lỗi (khi gặp error không hiểu)
```text
Tôi gặp lỗi CI/CD cho Flutter:
[paste error from GitHub Actions / Fastlane]

Workflow:
[paste YAML]

Context: [build step nào fail, trên platform nào]
Cần: (1) Root cause, (2) Fix, (3) Prevention (add check to prevent recurrence).
```

---

## 🎓 Post-Training: Lộ trình học tiếp

### Advanced Flutter Topics

| Chủ đề | Mô tả | Tài liệu |
|--------|--------|-----------|
| **Flutter Web** | Build web apps với Flutter (PWA, SPA) | https://docs.flutter.dev/platform-integration/web |
| **Flutter Desktop** | macOS, Windows, Linux apps | https://docs.flutter.dev/platform-integration/desktop |
| **Custom RenderObjects** | Vẽ UI custom ở level thấp | https://api.flutter.dev/flutter/rendering/RenderObject-class.html |
| **Platform Views** | Embed native views trong Flutter | https://docs.flutter.dev/platform-integration/platform-views |
| **FFI** | Gọi C/C++ code từ Dart | https://docs.flutter.dev/platform-integration/android/c-interop |
| **Flame** | Game engine cho Flutter | https://flame-engine.org |
| **Custom Paint** | Canvas API, custom drawings | https://api.flutter.dev/flutter/widgets/CustomPaint-class.html |
| **Isolates** | Multi-threaded Dart | https://dart.dev/language/isolates |
| **Flutter Internals** | Rendering pipeline, widget lifecycle | https://docs.flutter.dev/resources/architectural-overview |
| **Package Development** | Tạo và publish packages | https://docs.flutter.dev/packages-and-plugins/developing-packages |

### Backend & Full-stack

| Chủ đề | Mô tả | Link |
|--------|--------|------|
| **Dart Frog** | Backend framework bằng Dart | https://dartfrog.vgv.dev |
| **Serverpod** | Full-stack Dart framework | https://serverpod.dev |
| **Firebase** | BaaS: Auth, Firestore, Storage, Functions | https://firebase.google.com/docs/flutter |
| **Supabase** | Open-source Firebase alternative | https://supabase.com/docs/guides/getting-started/tutorials/with-flutter |
| **GraphQL** | Ferry/Artemis cho GraphQL | https://pub.dev/packages/ferry |

### Design & UX

| Chủ đề | Link |
|--------|------|
| Material 3 Design | https://m3.material.io |
| Cupertino (iOS) Design | https://developer.apple.com/design/human-interface-guidelines |
| Flutter Adaptive Design | https://docs.flutter.dev/ui/layout/responsive-adaptive |

---

## 👥 Community & Resources

### Cộng đồng chính thức

| Cộng đồng | Link | Mô tả |
|-----------|------|--------|
| Flutter Community | https://flutter.dev/community | Trang cộng đồng chính thức |
| Flutter Discord | https://discord.gg/flutter | Chat trực tiếp, Q&A |
| r/FlutterDev | https://reddit.com/r/FlutterDev | Reddit community |
| Stack Overflow | https://stackoverflow.com/questions/tagged/flutter | Q&A |
| Flutter GitHub | https://github.com/flutter/flutter | Source code, issues |

### YouTube Channels

| Channel | Nội dung |
|---------|----------|
| Flutter (official) | Widget of the Week, tutorials |
| Fireship | Quick explainers, comparisons |
| Robert Brunhage | Flutter tutorials, tips |
| Tadas Petra | Flutter content |
| Reso Coder | Architecture, clean code |
| Code with Andrea | In-depth tutorials |

### Newsletters & Blogs

| Resource | Link |
|----------|------|
| Flutter Weekly | https://flutterweekly.net |
| Medium — Flutter | https://medium.com/flutter |
| dev.to — Flutter | https://dev.to/t/flutter |
| Flutter Official Blog | https://medium.com/flutter |

### Conferences

| Conference | Link |
|-----------|------|
| FlutterCon | https://fluttercon.dev |
| Flutter Vikings | https://fluttervikings.com |
| Droidcon | https://www.droidcon.com |

---

## 📋 Tổng kết tài liệu tham khảo toàn khóa học

| Tuần | Buổi | Chủ đề | Tài liệu chính |
|------|------|--------|-----------------|
| 1 | 01-02 | Dart Cơ Bản | dart.dev, DartPad |
| 2 | 03-04 | Widget & Layout | flutter.dev/ui |
| 3 | 05-06 | Navigation & State | GoRouter, setState |
| 4 | 07-08 | Riverpod & BLoC | riverpod.dev, bloclibrary.dev |
| 5 | 09-10 | Architecture & DI | Clean Architecture, get_it |
| 6 | 11-12 | Networking & Data | Dio, Drift, SharedPreferences |
| 7 | 13-14 | Performance & Animation | DevTools, Animations |
| 8 | 15-16 | Platform & CI/CD | Method channels, Fastlane, GitHub Actions |

---

> 🎓 **Cảm ơn bạn đã hoàn thành chương trình Flutter Training!**
> Hãy tiếp tục build, deploy, và chia sẻ những gì bạn đã học. **Happy coding! 🚀**
