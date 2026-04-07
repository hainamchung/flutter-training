# Module 20 – Exercises: Native Platforms Bridge

> **Mục tiêu**: Thực hành tracing platform channel flow, iOS/Android configuration, và native debugging.
>
> ⚠️ **Lưu ý quan trọng**: Trong base_flutter project, **iOS có MethodChannel** nhưng **Android không có**. Exercises được thiết kế để thực hành với code có sẵn (iOS) và học lý thuyết (Android/General).

📌 **Recap**: M0 (Dart basics) · M1 (WidgetsFlutterBinding) · M3 (Config/env)

---

## Exercise 1 ⭐ — Trace ACTUAL iOS MethodChannel

### Mục tiêu

Hiểu MethodChannel thực tế trong iOS app.

### Thực tế trong base_flutter

**iOS (`AppDelegate.swift`)** có một MethodChannel:

```swift
let notificationChannel = FlutterMethodChannel(
  name: "jp.flutter.app",
  binaryMessenger: engine.applicationRegistrar.messenger()
)
notificationChannel.setMethodCallHandler({
  (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
  if call.method == "clearBadgeCount" {
    clearBadgeCount()
    result(nil)
  } else {
    result(FlutterMethodNotImplemented)
  }
})
```

### Hướng dẫn

1. Đọc `base_flutter/ios/Runner/AppDelegate.swift`
2. Tìm FlutterMethodChannel với name `"jp.flutter.app"`
3. Tìm method `clearBadgeCount` implementation
4. Trace flow: Flutter → `invokeMethod('clearBadgeCount')` → Swift handler → `result(nil)`

### Acceptance Criteria

- [ ] Đọc `AppDelegate.swift` và xác định MethodChannel setup
- [ ] Xác định method name được handle: `clearBadgeCount`
- [ ] Trả lời: MethodChannel này dùng để làm gì?
- [ ] Trả lời: Kênh này được register ở function nào? (`didInitializeImplicitFlutterEngine`)

---

## Exercise 2 ⭐⭐ — Android: Không có Platform Channel

### Mục tiêu

Hiểu rằng base_flutter **không có** MethodChannel trên Android, và tại sao Firebase plugins thường dùng.

### Thực tế trong base_flutter

Đọc `base_flutter/android/app/src/main/kotlin/.../MainActivity.kt`:

```kotlin
class MainActivity : FlutterActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Không có MethodChannel setup
    }
}
```

**Nhận xét**: Android side không có custom platform channel. Native functionality được handle bởi Firebase plugins (FirebaseApp.configure()).

### Hướng dẫn

1. Đọc `MainActivity.kt` — xác nhận không có MethodChannel
2. Đọc `AppDelegate.swift` — xác nhận có MethodChannel
3. Trả lời: Tại sao iOS có mà Android không có?

### Acceptance Criteria

- [ ] Đọc và xác nhận MainActivity.kt không có MethodChannel
- [ ] Xác định cách Android handle native functionality (Firebase plugins)
- [ ] Trả lời: Trong thực tế, khi nào cần thêm MethodChannel vào Android?

---

## Exercise 3 ⭐⭐ — iOS Configuration Review

### Mục tiêu

Phân tích iOS configuration và hiểu permissions setup.

### Hướng dẫn

1. Đọc `ios/Runner/Info.plist` — hiểu tất cả permissions
2. Đọc `ios/Podfile` — hiểu configuration
3. Trả lời: Tại sao cần `UIBackgroundModes`? Khi nào cần thêm?
4. Trả lời: Sự khác nhau giữa `NSCameraUsageDescription` và `NSPhotoLibraryAddUsageDescription`?

### Acceptance Criteria

- [ ] Đọc Info.plist — hiểu permissions structure
- [ ] Đọc Podfile — hiểu Firebase/M10 configuration
- [ ] Trả lời: UIBackgroundModes khi nào cần
- [ ] Trả lời: Sự khác nhau giữa các usage descriptions

---

## Exercise 4 ⭐⭐ — Android Configuration Review

### Mục tiêu

Phân tích Android configuration và hiểu build setup.

### Hướng dẫn

1. Đọc `android/app/build.gradle` — hiểu SDK versions và dependencies
2. Đọc `android/app/src/main/AndroidManifest.xml` — hiểu permissions
3. Trả lời: Sự khác nhau giữa `minSdk`, `targetSdk`, và `compileSdk`?
4. Trả lời: ProGuard/R8 dùng để làm gì?

### Acceptance Criteria

- [ ] Đọc build.gradle — hiểu SDK versions
- [ ] Đọc AndroidManifest.xml — hiểu permissions
- [ ] Trả lời: minSdk vs targetSdk vs compileSdk
- [ ] Trả lời: ProGuard/R8 purpose

---

## Exercise 5 ⭐⭐⭐ — THEORETICAL: Thêm MethodChannel vào Android

### Mục tiêu

Học cách thêm MethodChannel vào Android (lý thuyết, vì base_flutter không có).

### Hướng dẫn

1. **Nghiên cứu pattern từ iOS:**
   - iOS có `FlutterMethodChannel` với handler
   - Method: `clearBadgeCount` → gọi `clearBadgeCount()` function

2. **Thiết kế Android equivalent:**
   ```kotlin
   // MainActivity.kt - THEORETICAL (chưa có trong project)
   class MainActivity : FlutterActivity() {
       override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
           super.configureFlutterEngine(flutterEngine)
           
           MethodChannel(
               flutterEngine.dartExecutor.binaryMessenger,
               "jp.flutter.app"
           ).setMethodCallHandler { call, result ->
               when (call.method) {
                   "clearBadgeCount" -> {
                       clearBadgeCount()
                       result.success(null)
                   }
                   else -> result.notImplemented()
               }
           }
       }
       
       private fun clearBadgeCount() {
           // Android implementation
       }
   }
   ```

3. **Dart side (đã có):**
   ```dart
   const channel = MethodChannel('jp.flutter.app');
   await channel.invokeMethod('clearBadgeCount');
   ```

### Acceptance Criteria

- [ ] Hiểu pattern MethodChannel từ iOS implementation
- [ ] Thiết kế Android equivalent
- [ ] Trả lời: Sự khác nhau giữa iOS và Android MethodChannel setup
- [ ] Trả lời: Khi nào cần custom MethodChannel thay vì dùng plugin

---

## Tổng kết Exercises

|| # | Độ khó | Skill |
|---|--------|-------|
|| 1 | ⭐ | Trace actual iOS MethodChannel |
|| 2 | ⭐⭐ | Android: Observe no platform channel |
|| 3 | ⭐⭐ | iOS configuration review |
|| 4 | ⭐⭐ | Android configuration review |
|| 5 | ⭐⭐⭐ | Theoretical: Add MethodChannel to Android |

---

## ⚠️ Quan trọng về Platform Channels trong base_flutter

| Platform | MethodChannel | Purpose | Implementation |
|----------|--------------|---------|----------------|
| iOS | ✅ Có | `clearBadgeCount` | `AppDelegate.swift` |
| Android | ❌ Không | - | (relies on Firebase plugins) |

**Firebase Plugins** dùng platform channels internally cho:
- Firebase Auth
- Firebase Messaging
- Firebase Analytics
- Crashlytics

→ **Tiếp theo**: [04-verify.md](./04-verify.md) — Verification checklist.

<!-- AI_VERIFY: generation-complete -->
