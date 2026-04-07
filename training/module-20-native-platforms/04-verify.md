# Verification — Kiểm tra kết quả Module 20

> Đối chiếu bài làm với iOS/Android documentation và platform channel best practices.
>
> ⚠️ **Lưu ý quan trọng**: iOS có MethodChannel thực tế, Android không có. Verification được điều chỉnh phù hợp.

---

## 1. Self-Assessment Checklist

Trả lời **Yes / No** cho từng câu. Nếu **No** → quay lại concept tương ứng trong [02-concept.md](./02-concept.md).

|| # | Câu hỏi | Concept | Badge |
|---|---------|---------|-------|
| 1 | Tôi xác nhận được iOS có MethodChannel `"jp.flutter.app"` với method `clearBadgeCount`? | iOS MethodChannel | 🔴 |
| 2 | Tôi xác nhận được Android (MainActivity.kt) **không có** MethodChannel? | Android Native | 🔴 |
| 3 | Tôi phân biệt được MethodChannel vs EventChannel — khi nào dùng cái nào? | Platform Channels | 🔴 |
| 4 | Tôi giải thích được sự khác nhau giữa `minSdk`, `targetSdk`, `compileSdk`? | Android Configuration | 🟡 |
| 5 | Tôi mô tả được Info.plist permissions cần thiết cho camera, location, push? | iOS Configuration | 🟡 |
| 6 | Tôi hiểu Firebase plugins dùng platform channels internally? | Firebase Integration | 🟢 |
| 7 | Tôi thiết kế được MethodChannel setup cho Android (theoretical)? | Android MethodChannel | 🟢 |

**Target:** 3/3 Yes cho 🔴 MUST-KNOW, tối thiểu 5/7 tổng.

---

## 2. Exercise Verification

### Exercise 1 — Trace ACTUAL iOS MethodChannel ⭐

**File:** `base_flutter/ios/Runner/AppDelegate.swift`

- [ ] Đọc AppDelegate.swift — xác định FlutterMethodChannel
- [ ] Tìm method name: `clearBadgeCount`
- [ ] Xác định function `didInitializeImplicitFlutterEngine` là nơi register
- [ ] Trace flow: Flutter invokeMethod → Swift handler → result

### Exercise 2 — Android: No Platform Channel ⭐⭐

**File:** `base_flutter/android/app/src/main/kotlin/.../MainActivity.kt`

- [ ] Đọc MainActivity.kt — xác nhận không có MethodChannel
- [ ] Xác định native functionality được handle bởi Firebase plugins
- [ ] Trả lời: Khi nào cần thêm MethodChannel vào Android

### Exercise 3 — iOS Configuration ⭐⭐

- [ ] Đọc và hiểu Info.plist permissions
- [ ] Đọc và hiểu Podfile configuration
- [ ] Trả lời: UIBackgroundModes khi nào cần
- [ ] Trả lời: Sự khác nhau giữa usage descriptions

### Exercise 4 — Android Configuration ⭐⭐

- [ ] Đọc build.gradle — SDK versions
- [ ] Đọc AndroidManifest.xml — permissions
- [ ] Trả lời: minSdk vs targetSdk vs compileSdk
- [ ] Trả lời: ProGuard/R8 purpose

### Exercise 5 — Theoretical Android MethodChannel ⭐⭐⭐

- [ ] Hiểu pattern từ iOS implementation
- [ ] Thiết kế Android equivalent với `configureFlutterEngine`
- [ ] Trả lời: Sự khác nhau iOS vs Android setup
- [ ] Trả lời: Khi nào cần custom MethodChannel

---

## 3. Codebase Verification

### Check 1: iOS MethodChannel

```bash
# Kiểm tra AppDelegate.swift có MethodChannel
grep -n "FlutterMethodChannel" base_flutter/ios/Runner/AppDelegate.swift
grep -n "jp.flutter.app" base_flutter/ios/Runner/AppDelegate.swift
grep -n "clearBadgeCount" base_flutter/ios/Runner/AppDelegate.swift
```

**Expected:** Tìm thấy cả 3 patterns

### Check 2: Android NO MethodChannel

```bash
# Kiểm tra MainActivity.kt KHÔNG có MethodChannel
grep -n "MethodChannel" base_flutter/android/app/src/main/kotlin/jp/flutter/app/MainActivity.kt
```

**Expected:** Không tìm thấy MethodChannel

### Check 3: Platform Channel Summary

| Platform | Channel Name | Method | Status |
|----------|-------------|--------|--------|
| iOS | `jp.flutter.app` | `clearBadgeCount` | ✅ Exists |
| Android | - | - | ❌ Not exists |

---

## 4. Quick Quiz

<details>
<summary>Q1: MethodChannel vs EventChannel — khi nào dùng cái nào?</summary>

MethodChannel dùng cho **one-time request-response** operations: gọi method, nhận result. VD: `getDeviceInfo()`, `clearBadgeCount()`. EventChannel dùng cho **continuous streaming** data: subscribe, nhận events liên tục. VD: `batteryLevelStream`, `sensorDataStream`. MethodChannel = `fetch()`, EventChannel = WebSocket/SSE.

</details>

<details>
<summary>Q2: Tại sao iOS có MethodChannel mà Android không có?</summary>

Trong base_flutter project, iOS cần clear badge count (notification management) nên có MethodChannel. Android dựa vào Firebase plugins cho native functionality. Đây là architectural decision của team — không phải Flutter requirement. Thực tế, cả hai platform đều có thể có MethodChannel nếu cần.

</details>

<details>
<summary>Q3: Info.plist vs AndroidManifest.xml — khác nhau gì?</summary>

Info.plist là **file-based configuration** (XML property list) cho iOS — khai báo permissions, capabilities, và app metadata. AndroidManifest.xml là **manifest declaration** cho Android — khai báo permissions, app components (activities, services), và intent filters. Cả hai đều cần khai báo permissions, nhưng format và cách tổ chức khác nhau.

</details>

<details>
<summary>Q4: Sự khác nhau giữa `minSdk`, `targetSdk`, và `compileSdk`?</summary>

`minSdk` = minimum Android version app hỗ trợ. App không được cài trên device có API thấp hơn. `targetSdk` = Android version mà app được optimize cho. Ảnh hưởng behavior changes (VD: background start restrictions). `compileSdk` = Android SDK version dùng để compile. Phải ≥ targetSdk. Nên match với latest stable.

</details>

<details>
<summary>Q5: Firebase plugins dùng platform channels như thế nào?</summary>

Firebase SDK (Auth, Messaging, Analytics, etc.) dùng platform channels internally để giao tiếp với native SDKs:
- iOS: Gọi Firebase Swift SDK qua MethodChannel
- Android: Gọi Firebase Kotlin SDK qua MethodChannel

Flutter developer không cần viết MethodChannel code — Firebase plugin đã handle rồi. Tuy nhiên, nếu cần custom native functionality không có plugin, phải tự viết MethodChannel.

</details>

---

## 5. FE Perspective Mapping

|| Flutter | FE Equivalent |
|---------|---------------|
| `MethodChannel` | WebView `postMessage` / RN `NativeModules` |
| `EventChannel` | `EventSource` (SSE) |
| Info.plist | React Native `Info.plist` |
| AndroidManifest.xml | React Native `AndroidManifest.xml` |
| `Platform.isIOS` | `Platform.OS === 'ios'` |
| App signing | App Store certificates |

---

## 6. Backward Reference Check

- [ ] Nhận ra MethodChannel tương tự WebView bridge (M1)
- [ ] Hiểu Platform.isIOS/isAndroid liên quan đến environment config (M3)
- [ ] Firebase dùng platform channels internally (M20)

---

## 7. Forward Reference

- [ ] Biết M21 (Firebase) plugins dùng platform channels internally
- [ ] Biết Firebase Analytics, Crashlytics, Messaging đều qua native

---

## ✅ Module Complete

Hoàn thành khi:

- [ ] Self-assessment: ≥ 5/7 Yes (3/3 🔴 bắt buộc)
- [ ] Exercise 1 + 2 + 3 hoàn thành
- [ ] Quick Quiz trả lời đúng ≥ 4/5
- [ ] Codebase verification: iOS MethodChannel ✅, Android no MethodChannel ✅

---

## ➡️ Next Module

Hoàn thành Module 20! Bạn đã hiểu cách Flutter giao tiếp với native code.

→ Tiến sang **[Module 21 — Firebase](../module-21-firebase/)** để học Firebase services integration: Analytics, Crashlytics, Auth, Remote Config, và Messaging.

<!-- AI_VERIFY: generation-complete -->
