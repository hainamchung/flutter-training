# Module 21 – Exercises: Firebase

> **Mục tiêu**: Thực hành Firebase integration — Analytics, Crashlytics, và Cloud Messaging với actual services trong base_flutter.

📌 **Recap**: M1 (WidgetsFlutterBinding) · M3 (Config/env) · M17 (DI/Injectable) · M18 (Testing)

---

## Exercise 1 ⭐ — Trace Firebase Services Integration

<!-- AI_VERIFY: exercise-1 -->

### Mục tiêu

Hiểu cách Firebase services được integrate trong base_flutter thông qua Riverpod và Injectable patterns.

### Acceptance Criteria

- [ ] Đọc `AnalyticsHelper` — tìm file: `base_flutter/lib/common/helper/analytics/analytics_helper.dart`
- [ ] Đọc `CrashlyticsHelper` — tìm file: `base_flutter/lib/common/helper/crashlytics_helper.dart`
- [ ] Đọc `FirebaseMessagingService` — tìm file: `base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart`
- [ ] Trả lời: Tại sao AnalyticsHelper dùng Riverpod Provider thay vì Injectable?
- [ ] Trả lời: Tại sao CrashlyticsHelper và FirebaseMessagingService dùng @LazySingleton?

<details>
<summary>🏗️ Architecture Hint</summary>

**AnalyticsHelper Pattern (Riverpod):**
```dart
// base_flutter/lib/common/helper/analytics/analytics_helper.dart
final analyticsHelperProvider = Provider<AnalyticsHelper>(
  (ref) => AnalyticsHelper(ref: ref),
);

class AnalyticsHelper {
  AnalyticsHelper({
    FirebaseAnalytics? firebaseAnalytics,
    required this.ref,
  }) : _firebaseAnalytics = firebaseAnalytics ?? FirebaseAnalytics.instance;
  
  Future<void> logEvent(NormalEvent event) async { ... }
  Future<void> logScreenView(ScreenViewEvent screenViewEvent) async { ... }
  Future<void> logPurchase({ ... }) async { ... }
  Future<void> setUserId(String userId) { ... }
}
```

**CrashlyticsHelper Pattern (Injectable):**
```dart
// base_flutter/lib/common/helper/crashlytics_helper.dart
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

**FirebaseMessagingService Pattern (Injectable):**
```dart
// base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart
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

</details>

---

## Exercise 2 ⭐⭐ — Extend AnalyticsHelper

### Mục tiêu

Thêm một custom analytics event vào AnalyticsHelper để track user interaction.

### Acceptance Criteria

- [ ] Thêm method `logButtonTap(String buttonName, String page)` vào AnalyticsHelper
- [ ] Method phải reuse `_commonParameters` để include deviceId
- [ ] Method phải log event với format: `button_tap_{buttonName}` và page name
- [ ] Viết unit test cho method mới (mock FirebaseAnalytics)

<details>
<summary>🏗️ Architecture Hint</summary>

**Source Code Reference:** `base_flutter/lib/common/helper/analytics/analytics_helper.dart`

**Pattern to follow:**
```dart
// Trong AnalyticsHelper, thêm method mới:
Future<void> logButtonTap(String buttonName, String page) async {
  final parameters = {
    ...await _commonParameters,
    'button_name': buttonName,
    'page': page,
  };
  if (kDebugMode) {
    Log.d('logButtonTap: $buttonName on $page', color: LogColor.cyan);
  }
  return _firebaseAnalytics.logEvent(
    name: 'button_tap_${buttonName}',
    parameters: parameters,
  );
}
```

**Test Pattern:**
```dart
// test/unit_test/common/helper/analytics_helper_test.dart
test('logButtonTap should call firebaseAnalytics.logEvent', () async {
  // Setup mock
  when(mockFirebaseAnalytics.logEvent(
    name: anyNamed('name'),
    parameters: anyNamed('parameters'),
  )).thenAnswer((_) async {});
  
  // Call method
  await analyticsHelper.logButtonTap('submit', 'HomePage');
  
  // Verify
  verify(mockFirebaseAnalytics.logEvent(
    name: 'button_tap_submit',
    parameters: anyNamed('parameters'),
  )).called(1);
});
```

</details>

---

## Exercise 3 ⭐⭐ — Extend CrashlyticsHelper

### Mục tiêu

Thêm method để log custom keys cho crash analytics.

### Acceptance Criteria

- [ ] Thêm method `setCustomKey(String key, dynamic value)` vào CrashlyticsHelper
- [ ] Thêm method `setUserIdentifier(String userId)` để link crash với user
- [ ] Thêm method `log(String message)` để log custom messages
- [ ] Viết unit test cho các method mới

<details>
<summary>🏗️ Architecture Hint</summary>

**Source Code Reference:** `base_flutter/lib/common/helper/crashlytics_helper.dart`

**Pattern to follow:**
```dart
// Trong CrashlyticsHelper, thêm các methods mới:
Future<void> setCustomKey(String key, dynamic value) {
  return FirebaseCrashlytics.instance.setCustomKey(key, value.toString());
}

Future<void> setUserIdentifier(String userId) {
  return FirebaseCrashlytics.instance.setUserIdentifier(userId);
}

Future<void> log(String message) {
  return FirebaseCrashlytics.instance.log(message);
}
```

**Usage Example:**
```dart
// Khi user login thành công:
crashlyticsHelper.setUserIdentifier(user.id);
crashlyticsHelper.setCustomKey('user_email', user.email);
crashlyticsHelper.setCustomKey('user_tier', user.tier);

// Khi có non-critical error:
crashlyticsHelper.log('API timeout exceeded');
```

</details>

---

## Exercise 4 ⭐⭐⭐ — Integrate FirebaseMessagingService

### Mục tiêu

Sử dụng FirebaseMessagingService để subscribe/unsubscribe topics.

### Acceptance Criteria

- [ ] Tạo một Page mới để test FCM topic subscription
- [ ] Sử dụng `FirebaseMessagingService` (inject qua Riverpod)
- [ ] Implement button để subscribe vào topic `test_notifications`
- [ ] Implement button để unsubscribe khỏi topic
- [ ] Hiển thị current device token (sử dụng `deviceToken`)
- [ ] Trả lời: Khi nào nên dùng topic subscription vs device token?

<details>
<summary>🏗️ Architecture Hint</summary>

**Source Code Reference:** `base_flutter/lib/data_source/firebase/messaging/firebase_messaging_service.dart`

**Page Implementation Pattern:**
```dart
// lib/ui/page/test_fcm_page.dart
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:base_flutter/common/helper/crashlytics_helper.dart';
import 'package:base_flutter/data_source/firebase/messaging/firebase_messaging_service.dart';

final firebaseMessagingServiceProvider = Provider((ref) => getIt.get<FirebaseMessagingService>());

class TestFCMPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagingService = ref.watch(firebaseMessagingServiceProvider);
    
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Display device token
            FutureBuilder<String?>(
              future: messagingService.deviceToken,
              builder: (context, snapshot) {
                return Text('Token: ${snapshot.data ?? "loading..."}');
              },
            ),
            // Subscribe button
            ElevatedButton(
              onPressed: () async {
                await messagingService.subscribeToTopic('test_notifications');
              },
              child: Text('Subscribe'),
            ),
            // Unsubscribe button
            ElevatedButton(
              onPressed: () async {
                await messagingService.unsubscribeFromTopic('test_notifications');
              },
              child: Text('Unsubscribe'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Topic vs Token:**
| Aspect | Topic Subscription | Device Token |
|--------|--------------------|--------------|
| Use Case | Broadcast to group | Direct notification |
| Flexibility | Easy to add/remove | Manual tracking needed |
| Scalability | Great for many users | Good for targeted |
| Example | "new_promo", "update_v2" | User ID based |

</details>

---

## Exercise 5 ⭐⭐⭐ — Error Reporting with Crashlytics

### Mục tiêu

Tích hợp CrashlyticsHelper vào một service để track errors.

### Acceptance Criteria

- [ ] Tạo một wrapper method trong CrashlyticsHelper để handle API errors
- [ ] Method phải capture: exception, stack trace, và custom context
- [ ] Implement `recordApiError()` method với parameters cho endpoint, status code
- [ ] Tích hợp vào một API service (mock hoặc thật)

<details>
<summary>🏗️ Architecture Hint</summary>

**Extended CrashlyticsHelper:**
```dart
// Thêm vào CrashlyticsHelper:
Future<void> recordApiError({
  required String endpoint,
  required int statusCode,
  dynamic exception,
  StackTrace? stack,
}) {
  return recordError(
    exception: exception,
    stack: stack,
    reason: 'API Error: $endpoint returned $statusCode',
    information: ['endpoint: $endpoint', 'statusCode: $statusCode'],
  );
}
```

**Usage in API Service:**
```dart
// Trong API service:
try {
  final response = await dio.get(endpoint);
  return response.data;
} catch (e, stack) {
  await crashlyticsHelper.recordApiError(
    endpoint: endpoint,
    statusCode: e.response?.statusCode ?? 0,
    exception: e,
    stack: stack,
  );
  rethrow;
}
```

</details>

---

## Tổng kết Exercises

| # | Độ khó | Skill |
|---|--------|-------|
| 1 | ⭐ | Trace Firebase services patterns |
| 2 | ⭐⭐ | Extend AnalyticsHelper |
| 3 | ⭐⭐ | Extend CrashlyticsHelper |
| 4 | ⭐⭐⭐ | Integrate FirebaseMessagingService |
| 5 | ⭐⭐⭐ | Error reporting with Crashlytics |

---

## Stretch Goals

- ⭐⭐⭐⭐ Tạo một analytics event registry để type-safe analytics events
- ⭐⭐⭐⭐⭐ Implement foreground notification handler với custom UI

→ **Tiếp theo**: [04-verify.md](./04-verify.md) — Verification checklist.

<!-- AI_VERIFY: generation-complete -->
