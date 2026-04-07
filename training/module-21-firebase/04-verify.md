# Verification — Kiểm tra kết quả Module 21

> Đối chiếu bài làm với Firebase documentation và actual source code trong base_flutter.

---

## 1. Self-Assessment Checklist

Trả lời **Yes / No** cho từng câu. Nếu **No** → quay lại concept tương ứng trong [02-concept.md](./02-concept.md).

| # | Câu hỏi | Concept | Badge |
|---|---------|---------|-------|
| 1 | Tôi giải thích được sự khác biệt giữa AnalyticsHelper (Riverpod) và CrashlyticsHelper (Injectable)? | Firebase Services Pattern | 🔴 |
| 2 | Tôi trace được Analytics flow — `logEvent()` → Firebase console? | Firebase Analytics | 🔴 |
| 3 | Tôi hiểu Firebase Crashlytics dùng để track exceptions và crashes? | Crashlytics | 🔴 |
| 4 | Tôi configure được Firebase Cloud Messaging service? | FCM | 🟡 |
| 5 | Tôi phân biệt được Crashlytics vs Analytics — exception vs behavior tracking? | Crashlytics vs Analytics | 🟢 |
| 6 | Tôi sử dụng được Injectable patterns (@LazySingleton) cho Firebase services? | DI/Injectable | 🟢 |
| 7 | Tôi hiểu khi nào dùng topic subscription vs device token? | FCM Advanced | 🟢 |

**Target:** 3/3 Yes cho 🔴 MUST-KNOW, tối thiểu 5/7 tổng.

---

## 2. Exercise Verification

### Exercise 1 — Trace Firebase Services ⭐

<!-- AI_VERIFY: exercise-1-verification -->

- [ ] Đọc `AnalyticsHelper` tại `base_flutter/lib/common/helper/analytics/analytics_helper.dart`
- [ ] Đọc `CrashlyticsHelper` tại `base_flutter/lib/common/helper/crashlytics_helper.dart`
- [ ] Đọc `FirebaseMessagingService` tại `base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart`
- [ ] Trả lời: Tại sao AnalyticsHelper dùng `Provider<>` thay vì `@LazySingleton()`?
- [ ] Trả lời: Tại sao CrashlyticsHelper và FirebaseMessagingService dùng `@LazySingleton()`?

### Exercise 2 — Extend AnalyticsHelper ⭐⭐

<!-- AI_VERIFY: exercise-2-verification -->

- [ ] Thêm method `logButtonTap(String buttonName, String page)` vào AnalyticsHelper
- [ ] Method reuse `_commonParameters` để include deviceId
- [ ] Event name format: `button_tap_{buttonName}`
- [ ] Viết unit test với mock FirebaseAnalytics

### Exercise 3 — Extend CrashlyticsHelper ⭐⭐

<!-- AI_VERIFY: exercise-3-verification -->

- [ ] Thêm method `setCustomKey(String key, dynamic value)`
- [ ] Thêm method `setUserIdentifier(String userId)`
- [ ] Thêm method `log(String message)`
- [ ] Viết unit test cho các method mới

### Exercise 4 — Integrate FirebaseMessagingService ⭐⭐⭐

<!-- AI_VERIFY: exercise-4-verification -->

- [ ] Tạo test page với FCM integration
- [ ] Sử dụng `firebaseMessagingServiceProvider` để inject service
- [ ] Implement subscribe/unsubscribe buttons
- [ ] Display device token
- [ ] Trả lời: Topic subscription vs device token - khi nào dùng cái nào?

### Exercise 5 — Error Reporting ⭐⭐⭐

<!-- AI_VERIFY: exercise-5-verification -->

- [ ] Thêm method `recordApiError()` vào CrashlyticsHelper
- [ ] Method capture: endpoint, status code, exception, stack trace
- [ ] Tích hợp vào API service pattern

---

## 3. Quick Quiz

<details>
<summary>Q1: Tại sao AnalyticsHelper dùng Riverpod Provider thay vì Injectable @LazySingleton?</summary>

AnalyticsHelper sử dụng Riverpod Provider vì nó cần `Ref` để access các dependencies khác (VD: `deviceHelperProvider` để lấy deviceId). Với Riverpod, ta có thể pass `ref` vào constructor, trong khi `@LazySingleton` từ Injectable không hỗ trợ pattern này một cách tự nhiên. Ngoài ra, Analytics cần được instantiate sớm trong app lifecycle, không phải lazy.

</details>

<details>
<summary>Q2: Sự khác nhau giữa Crashlytics và Analytics?</summary>

**Crashlytics** dùng để track exceptions và crashes — stack traces, non-fatal errors, crash reports. Dữ liệu được gửi real-time khi app crash hoặc có error. **Analytics** dùng để track user behavior — events, screen views, user properties. Dữ liệu được batched và gửi periodically. Crashlytics phục vụ debugging và stability, Analytics phục vụ business intelligence.

</details>

<details>
<summary>Q3: Topic subscription vs Device token — khi nào dùng cái nào?</summary>

**Topic Subscription** phù hợp khi: (1) Muốn broadcast message đến nhiều users cùng lúc, (2) Không cần track individual device, (3) Dùng cho promotions, announcements. **Device Token** phù hợp khi: (1) Cần gửi message đến specific user, (2) Cần store và manage tokens, (3) Dùng cho personalized notifications. Topic subscription dễ scale hơn, device token linh hoạt hơn nhưng cần backend quản lý.

</details>

<details>
<summary>Q4: @LazySingleton vs @Singleton trong Injectable — khác nhau gì?</summary>

**@Singleton** được instantiate ngay khi app start (eager initialization). **@LazySingleton** chỉ được instantiate khi được access lần đầu tiên (lazy initialization). Firebase services thường dùng @LazySingleton để trì hoãn initialization cho đến khi thực sự cần, giảm startup time. Tuy nhiên, với Firebase initialization (đã gọi trong main), @Singleton cũng acceptable.

</details>

<details>
<summary>Q5: Làm sao để test Firebase services?</summary>

**Mock Firebase SDK**: Sử dụng các packages như `mocktail` hoặc `Mockito` để mock Firebase classes. **Pattern trong base_flutter**: Inject dependencies qua constructor (hoặc Provider), sau đó trong test có thể pass mock implementations. VD: `AnalyticsHelper(firebaseAnalytics: mockFirebaseAnalytics, ref: mockRef)`.

</details>

---

## 4. Code Quality Checks

- [ ] Firebase services có proper dependency injection pattern
- [ ] Error handling cho tất cả async operations
- [ ] Tests cho Firebase services (mock Firebase SDK)
- [ ] Follow existing code patterns trong base_flutter
- [ ] Proper logging trong debug mode (sử dụng `kDebugMode` check)

---

## 5. Cross-Check with Actual Source

### AnalyticsHelper Pattern

<!-- AI_VERIFY: analytics-helper-source -->

Kiểm tra source: `base_flutter/lib/common/helper/analytics/analytics_helper.dart`

```dart
// ✅ Correct: Riverpod Provider pattern
final analyticsHelperProvider = Provider<AnalyticsHelper>(
  (ref) => AnalyticsHelper(ref: ref),
);

// ✅ Correct: Uses _commonParameters
Future<Map<String, Object>> get _commonParameters async => {
  ParameterConstants.userId: await ref.read(deviceHelperProvider).deviceId,
};
```

### CrashlyticsHelper Pattern

<!-- AI_VERIFY: crashlytics-helper-source -->

Kiểm tra source: `base_flutter/lib/common/helper/crashlytics_helper.dart`

```dart
// ✅ Correct: @LazySingleton pattern
@LazySingleton()
class CrashlyticsHelper {
  Future<void> recordError({
    dynamic exception,
    dynamic reason,
    StackTrace? stack,
    bool fatal = false,
  }) {
    return FirebaseCrashlytics.instance.recordError(...);
  }
}
```

### FirebaseMessagingService Pattern

<!-- AI_VERIFY: firebase-messaging-service-source -->

Kiểm tra source: `base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart`

```dart
// ✅ Correct: @LazySingleton + Riverpod Provider
@LazySingleton()
class FirebaseMessagingService {
  final _messaging = FirebaseMessaging.instance;
  
  Stream<String> get onTokenRefresh => _messaging.onTokenRefresh;
  Future<String?> get deviceToken async { ... }
  Stream<RemoteMessage> get onMessage => FirebaseMessaging.onMessage;
  Stream<RemoteMessage> get onMessageOpenedApp => FirebaseMessaging.onMessageOpenedApp;
  Future<void> subscribeToTopic(String topic) async { ... }
  Future<void> unsubscribeFromTopic(String topic) async { ... }
}
```

---

## 6. FE Perspective Mapping

| Flutter Firebase | FE Equivalent |
|-----------------|---------------|
| `AnalyticsHelper` (Provider) | Custom analytics wrapper |
| `CrashlyticsHelper` (@LazySingleton) | Sentry SDK |
| `FirebaseMessagingService` (@LazySingleton) | FCM JS SDK |
| `firebase_analytics` | `firebase.analytics()` |
| `firebase_crashlytics` | Sentry SDK |
| `firebase_messaging` | FCM JS SDK |

---

## 7. Backward Reference Check

- [ ] Nhận ra Firebase plugins dùng platform channels (M20)
- [ ] Hiểu Firebase initialization liên quan đến M1 (WidgetsFlutterBinding)
- [ ] Biết Firebase config có thể dùng dart_defines (M3)
- [ ] Hiểu dependency injection patterns từ M17

---

## 8. Forward Reference

- [ ] **Capstone Project**: Sử dụng Firebase services cho app
- [ ] **Production**: Firebase Analytics + Crashlytics cho monitoring
- [ ] **Next Steps**: Explore Firebase Remote Config, Firestore

---

## ✅ Module Complete

Hoàn thành khi:

- [ ] Self-assessment: ≥ 5/7 Yes (3/4 🔴 bắt buộc)
- [ ] Exercise 1 + 2 hoàn thành
- [ ] Quick Quiz trả lời đúng ≥ 3/5
- [ ] Cross-check với actual source code thành công

---

## 🎉 Module 21 Complete

Bạn đã hoàn thành **Module 21: Firebase Integration**! 🎉

**Đã học được:**
- Firebase Analytics với custom event tracking
- Firebase Crashlytics cho crash reporting
- Firebase Cloud Messaging (FCM) cho push notifications
- Dependency injection patterns cho Firebase services

**Next Steps:**
- [ ] **[M22: CI/CD Pipeline](https://github.com/nalslab/base_flutter)** — Tự động hóa build & deploy
- [ ] **[M23: Performance](https://github.com/nalslab/base_flutter)** — Tối ưu hóa app performance
- [ ] **Capstone Project** — Áp dụng tất cả kiến thức đã học

<!-- AI_VERIFY: generation-complete -->
