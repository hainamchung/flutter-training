# Module 20 – Native Platforms Bridge & Configuration

## Tổng quan

Module này survey toàn bộ native platform integration của dự án Flutter — từ Platform Channels (MethodChannel, EventChannel), iOS/Android configuration (Info.plist, Podfile, AndroidManifest.xml, build.gradle), app signing, đến native debugging. Đây là **Advanced Survey**: nắm architectural overview + key patterns, không setup native code từ đầu.

**Depth**: Advanced Survey — đọc hiểu codebase native config, trace channel flow, không viết native code.

**Cycle:** CODE (platform channels + configs) → EXPLAIN (iOS/Android setup) → PRACTICE (trace flow + debug).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được `MethodChannel` vs `EventChannel` — khi nào dùng cái nào?
2. Trace được flow: Flutter gọi `channel.invokeMethod()` → Native handler → callback?
3. Mô tả được `Info.plist` permissions cần thiết cho camera, location, push?
4. Configure được `build.gradle` với `compileSdk`, `minSdk`, `targetSdk`?
5. Hiểu được conditional imports (`Platform.isIOS`) trong Dart code?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 21 — Firebase](../module-21-firebase/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## Bạn sẽ học

1. **Platform Channels Architecture** — MethodChannel (request-response), EventChannel (stream)
2. **iOS Configuration** — Info.plist, Podfile, AppDelegate, certificates
3. **Android Configuration** — AndroidManifest.xml, build.gradle, signing config
4. **App Signing** — iOS certificates/provisioning, Android keystore
5. **Native Debugging** — Xcode, Android Studio, Flutter DevTools
6. **Conditional Imports** — Platform-specific code trong Dart

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## Kiến thức cần có

| Module | Nội dung | Vai trò trong M20 |
|--------|----------|-------------------|
| **M0** | Dart basics | Platform channel API dựa trên Dart types |
| **M1** | WidgetsFlutterBinding | Bootstrap layer nơi platform messages dispatch |
| **M3** | Config/env | dart_defines cho platform-specific config |

---

## Cấu trúc files

| File | Nội dung | Thời gian |
|------|----------|-----------|
| [01-code-walk.md](./01-code-walk.md) | Walk-through: AppDelegate MethodChannel, Android MainActivity, Info.plist, build.gradle | 30 min |
| [02-concept.md](./02-concept.md) | 6 concepts: channels, iOS config, Android config, signing, debug, conditional imports | 30 min |
| [03-exercise.md](./03-exercise.md) | 4 exercises: ⭐ trace → ⭐⭐ iOS config → ⭐⭐ Android config → ⭐⭐⭐ debug | 60 min |
| [04-verify.md](./04-verify.md) | Verification checklist | 10 min |

---

## 💡 FE Perspective

| Flutter | FE Equivalent |
|---------|---------------|
| `MethodChannel` | WebView `postMessage` / React Native `NativeModules` |
| `EventChannel` | `EventSource` (SSE) / `addEventListener` |
| `FlutterEngine` | WebView JS runtime / RN Bridge (JSI) |
| Info.plist | `ios/Info.plist` (React Native) |
| AndroidManifest.xml | `AndroidManifest.xml` (React Native) |
| `Platform.isIOS` | `Platform.OS === 'ios'` |
| Conditional imports | Platform-specific modules |

---

## Key Files trong Codebase

```
ios/Runner/
├── AppDelegate.swift            ← FlutterMethodChannel setup (clearBadgeCount)
├── Info.plist                   ← Permissions (camera, location, etc.)
└── Runner.entitlements          ← Capabilities (push, background)

android/app/src/main/
├── kotlin/.../MainActivity.kt    ← FlutterActivity entry point
└── AndroidManifest.xml          ← Permissions, app components

lib/
└── common/helper/               ← Platform-specific helpers
```

---

## Forward Reference

→ **M20 (Firebase)**: Firebase plugins dùng platform channels internally — hiểu M20 giúp debug Firebase issues.

<!-- AI_VERIFY: generation-complete -->
