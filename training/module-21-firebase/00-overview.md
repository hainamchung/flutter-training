# Module 21 – Firebase

## Tổng quan

Module này survey toàn bộ Firebase integration của dự án Flutter — từ Firebase Core setup, Analytics tracking, Crashlytics reporting, Auth (email/password, Google), Remote Config, Cloud Messaging (push notifications), Storage, và Firestore. Đây là **Advanced Survey**: nắm Firebase services overview + patterns, không setup Firebase project từ đầu.

**Depth**: Advanced Survey — đọc hiểu Firebase setup và patterns, trace flow giữa Flutter và Firebase console.

**Cycle:** CODE (Firebase services setup) → EXPLAIN (analytics, auth, messaging) → PRACTICE (trace flows + configuration).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được Firebase initialization flow — `Firebase.initializeApp()` → platform plugins?
2. Trace được Firebase Analytics flow — `logEvent()` → Firebase console?
3. Hiểu Firebase Auth state management — `authStateChanges()` → UI updates?
4. Configure được Firebase Remote Config — fetch → activate → use values?
5. Setup được Firebase Cloud Messaging — foreground/background handlers?

→ Nếu **5/5 Yes** — Hoàn thành Flutter Training!
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## Bạn sẽ học

1. **Firebase Core** — Initialization, configuration, google-services.json/plist
2. **Firebase Analytics** — Event tracking, screen tracking, user properties
3. **Firebase Crashlytics** — Exception reporting, custom keys, breadcrumbs
4. **Firebase Auth** — Email/password, Google Sign-In, auth state
5. **Firebase Remote Config** — Parameters, conditions, fetch & activate
6. **Firebase Cloud Messaging** — Token management, foreground/background handling
7. **Firebase Storage & Firestore** — File uploads, real-time database

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## Kiến thức cần có

| Module | Nội dung | Vai trò trong M21 |
|--------|----------|-------------------|
| **M1** | WidgetsFlutterBinding | Bootstrap layer for Firebase |
| **M3** | Config/env | dart_defines cho Firebase env |
| **M20** | Platform Channels | Firebase plugins dùng channels internally |

---

## Cấu trúc files

| File | Nội dung | Thời gian |
|------|----------|-----------|
| [01-code-walk.md](./01-code-walk.md) | Walk-through: main.dart init, analytics, auth, messaging, storage | 35 min |
| [02-concept.md](./02-concept.md) | 7 concepts: Core, Analytics, Crashlytics, Auth, Remote Config, FCM, Storage/Firestore | 30 min |
| [03-exercise.md](./03-exercise.md) | 4 exercises: ⭐ trace → ⭐⭐ config → ⭐⭐ analytics → ⭐⭐⭐ FCM | 60 min |
| [04-verify.md](./04-verify.md) | Verification checklist | 10 min |

---

## 💡 FE Perspective

| Flutter Firebase | FE Equivalent |
|-----------------|---------------|
| `firebase_core` | Firebase JS SDK initialization |
| `firebase_analytics` | `firebase.analytics()` |
| `firebase_crashlytics` | Sentry SDK |
| `firebase_auth` | Firebase Auth JS (`signInWithEmailAndPassword`) |
| `firebase_remote_config` | Firebase Remote Config JS |
| `firebase_messaging` | FCM JS SDK |
| `firebase_storage` | Firebase Storage JS SDK |
| `cloud_firestore` | Firestore JS SDK |

---

## Key Files trong Codebase

```
lib/
├── main.dart                      ← Firebase.initializeApp()
├── common/firebase/               ← Firebase configuration
│   ├── firebase_analytics_service.dart
│   ├── firebase_crashlytics_service.dart
│   ├── firebase_auth_service.dart
│   └── firebase_messaging_service.dart
└── ui/page/splash/
    └── splash_page.dart           ← FCM token registration

ios/Runner/
└── GoogleService-Info.plist      ← iOS Firebase config

android/app/
└── google-services.json          ← Android Firebase config
```

---

## Firebase Services Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Firebase                               │
├─────────────────────────────────────────────────────────────┤
│  Analytics ──────────► User behavior tracking                │
│  Crashlytics ────────► Exception & crash reporting         │
│  Auth ──────────────► User authentication                  │
│  Remote Config ─────► Server-side configuration             │
│  Cloud Messaging ───► Push notifications                    │
│  Storage ───────────► File storage                         │
│  Firestore ─────────► Real-time database                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Forward Reference

→ **Capstone Project**: Sử dụng Firebase Auth cho login, Analytics cho tracking, Crashlytics cho monitoring.

<!-- AI_VERIFY: generation-complete -->
