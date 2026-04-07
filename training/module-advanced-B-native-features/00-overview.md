# Advanced Module MB — Native Features

> **Depth:** Advanced Survey — optional module, lighter scaffolding

---

## Mục tiêu

Sau module này, bạn sẽ:
- Hiểu Camera integration: `image_picker`, `camera` package, cropping, compression
- Nắm Location services: `geolocator`, permissions, background location, geofencing
- Xử lý Biometric authentication: `local_auth`, fingerprint, Face ID, fallback
- Implement Push Notifications: Firebase Messaging, local notifications
- Configure Deep Linking: custom schemes, App Links, Universal Links

---

## Prerequisites

| Module | Cần nắm | Relevance cho MB |
|--------|---------|-----------------|
| **M0** | Dart basics, async/await — foundation for platform APIs | All features |
| **M1** | App entrypoint, platform channels | Permission handling |
| **M9** | `AppNavigator`, `AppRoute` pattern, navigation guards | Deep link navigation |
| **M12** | AppApiService, Dio — network layer | Upload photos, location API |
| **MA** | Security — secure storage, debug detection | Biometric security |

---

## Nội dung

| File | Nội dung | Thời lượng |
|------|----------|------------|
| [01-code-walk.md](./01-code-walk.md) | Native features patterns: camera, location, biometrics, notifications, deep links | ~45 min |
| [02-concept.md](./02-concept.md) | 6 concepts: camera pipeline, location API, auth patterns, notification architecture, deep link routing | ~30 min |
| [03-exercise.md](./03-exercise.md) | 5 exercises: camera → location → biometrics → push → deep link | ~3-5 hrs |
| [04-verify.md](./04-verify.md) | Checklist xác nhận hoàn thành | ~10 min |

**Phân bố:** 🔴 ~25% · 🟡 ~50% · 🟢 ~25%

---

## Anchor Files

```
pubspec.yaml                              — Native feature packages
lib/main.dart                            — Permission initialization
lib/data_source/                         — API services for uploads
lib/common/helper/                       — Native feature helpers
```

---

## 💡 FE Perspective Summary

| Flutter | Frontend Equivalent |
|---------|---------------------|
| `image_picker` / `camera` | HTML `<input type="file">` / MediaStream API |
| `geolocator` | `navigator.geolocation` API |
| `local_auth` | WebAuthn / Web Authentication API |
| `firebase_messaging` | Web Push API |
| Deep linking | URL routing (react-router) |

---

## Forward Reference

→ **Module MC:** State Management patterns cho handle native feature state.

<!-- AI_VERIFY: generation-complete -->
