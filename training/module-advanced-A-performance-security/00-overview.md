# Advanced Module MA — Performance & Security

> **Depth:** Advanced Survey — optional module, lighter scaffolding

---

## Mục tiêu

Sau module này, bạn sẽ:
- Hiểu security hardening: code obfuscation, SSL pinning, secure storage, API key protection, debug detection
- Nắm advanced performance: monitoring, memory leak detection, large dataset handling, image optimization
- Phân biệt khi nào cần security hardening vs performance optimization
- Apply security + performance best practices vào production-ready apps

---

## Prerequisites

| Module | Cần nắm | Relevance cho MA |
|--------|---------|------------------|
| **M0** | Dart basics, async/await — foundation for performance monitoring | Performance tracking |
| **M1** | App entrypoint, platform flags | Debug mode detection |
| **M3** | Config/Constants — environment values | API key protection |
| **M12** | AppApiService, Dio — network layer | SSL pinning, secure API |
| **M14** | AppPreferences, secure storage | Secure storage patterns |
| **M20** | Native platforms (MethodChannel) | SSL pinning implementation |
| **MA** | Performance Optimization — profiling basics | Performance monitoring |

---

## Nội dung

| File | Nội dung | Thời lượng |
|------|----------|------------|
| [01-code-walk.md](./01-code-walk.md) | Security + performance patterns trong codebase | ~30 min |
| [02-concept.md](./02-concept.md) | 6 concepts: obfuscation, SSL pinning, secure storage, API keys, debug detection, performance monitoring | ~25 min |
| [03-exercise.md](./03-exercise.md) | 3 exercises: security audit → SSL pinning → performance monitoring | ~2-4 hrs |
| [04-verify.md](./04-verify.md) | Checklist xác nhận hoàn thành | ~10 min |

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## Anchor Files

```
lib/data_source/preference/app_preferences.dart    — FlutterSecureStorage (secure storage patterns)
lib/data_source/api/app_api_service.dart           — Dio (SSL pinning, API security)
lib/main.dart                                     — App initialization (debug flags, environment)
pubspec.yaml                                       — Dependencies (obfuscation config)
android/app/build.gradle                          — R8/ProGuard configuration
ios/Runner/Info.plist                             — iOS security configs
```

---

## 💡 FE Perspective Summary

| Flutter | Frontend Equivalent |
|---------|---------------------|
| Code obfuscation (R8/ProGuard) | Webpack `mode: production` minification |
| SSL Certificate Pinning | Web TLS certificate pinning (rare in web) |

> ⚠️ **Web note:** TLS/certificate pinning on web is not practical due to browser security restrictions. SSL pinning is primarily a **mobile** concern (iOS/Android).
| `flutter_secure_storage` | `sessionStorage` / `localStorage` với encryption |
| API key protection | Environment variables (`.env`) |
| Debug mode detection | `process.env.NODE_ENV === 'development'` |
| Performance monitoring | `PerformanceObserver` API (web) |

---

## Forward Reference

→ **Advanced B (MB):** Native Features sử dụng secure token storage từ module này.
→ **Advanced C (MC):** Patterns & Tooling sử dụng performance monitoring techniques.

<!-- AI_VERIFY: generation-complete -->
