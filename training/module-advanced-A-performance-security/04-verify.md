# Verify — Advanced Performance & Security

> 📌 **Hoàn thành [03-exercise.md](./03-exercise.md) trước khi làm verify.**

---

## Verify Checklist

### Phần 1: Security Fundamentals 🔴

| # | Criteria | Check |
|---|----------|-------|
| 1.1 | Hiểu code obfuscation workflow (source → minify → rename → output) | [ ] |
| 1.2 | Giải thích được R8/ProGuard configuration trong Android | [ ] |
| 1.3 | Biết cách enable/disable minification cho debug vs release builds | [ ] |
| 1.4 | Hiểu obfuscation dictionary và tại sao cần nó | [ ] |
| 1.5 | Nêu được 3-5 items cần keep trong ProGuard rules | [ ] |

### Phần 2: SSL Pinning 🟡

| # | Criteria | Check |
|---|----------|-------|
| 2.1 | Giải thích được SSL pinning prevents MITM attacks | [ ] |
| 2.2 | Biết cách extract certificate fingerprint từ server | [ ] |
| 2.3 | Implement được basic SSL pinning với Dio | [ ] |
| 2.4 | Hiểu certificate rotation strategy (primary + backup pins) | [ ] |
| 2.5 | Xử lý được certificate validation errors gracefully | [ ] |

### Phần 3: Secure Storage 🟡

| # | Criteria | Check |
|---|----------|-------|
| 3.1 | Phân biệt được `FlutterSecureStorage` vs `SharedPreferences` | [ ] |
| 3.2 | Explain được Android KeyStore và iOS Keychain encryption | [ ] |
| 3.3 | Implement được secure token storage pattern | [ ] |
| 3.4 | Biết cách configure `AndroidOptions` và `IOSOptions` | [ ] |
| 3.5 | Hiểu khi nào nên dùng encrypted Hive vs SecureStorage | [ ] |

### Phần 4: API Key Protection 🟡

| # | Criteria | Check |
|---|----------|-------|
| 4.1 | Biết cách sử dụng `String.fromEnvironment()` cho API keys | [ ] |
| 4.2 | Explain được cách inject secrets qua CI/CD | [ ] |
| 4.3 | Hiểu được backend proxy pattern cho API key protection | [ ] |
| 4.4 | Tránh được hardcoded secrets trong source code | [ ] |
| 4.5 | Có thể setup environment-specific builds | [ ] |

### Phần 5: Debug Detection 🟢

| # | Criteria | Check |
|---|----------|-------|
| 5.1 | Sử dụng được `kDebugMode` và `kAssertsEnabled` | [ ] |
| 5.2 | Implement được emulator/simulator detection | [ ] |
| 5.3 | Detect được rooted/jailbroken devices | [ ] |
| 5.4 | Apply được debug detection để enforce security policies | [ ] |
| 5.5 | Hiểu security gate pattern cho unsafe environments | [ ] |

### Phần 6: Performance Monitoring 🟢

| # | Criteria | Check |
|---|----------|-------|
| 6.1 | Implement được custom performance monitoring service | [ ] |
| 6.2 | Track được navigation performance với RouteObserver | [ ] |
| 6.3 | Track được API call performance với Interceptor | [ ] |
| 6.4 | Log performance metrics trong debug mode | [ ] |
| 6.5 | Thiết kế được custom metrics cho app-specific monitoring | [ ] |

---

## Quick Quiz

### Question 1: Security

**SSL Pinning ngăn chặn loại attack nào?**

A) SQL Injection
B) Man-in-the-Middle (MITM)
C) Cross-Site Scripting (XSS)
D) Distributed Denial of Service (DDoS)

<details>
<summary>Answer</summary>

**B) Man-in-the-Middle (MITM)**

SSL Pinning đảm bảo app chỉ trust specific certificate, ngăn attacker intercept traffic với fake certificate.

</details>

### Question 2: Storage

**Nên lưu JWT token ở đâu?**

A) `SharedPreferences` vì nhanh và đơn giản
B) `FlutterSecureStorage` vì encrypted và secure
C) Plain file trong app documents directory
D) In-memory variable vì không persist

<details>
<summary>Answer</summary>

**B) `FlutterSecureStorage` vì encrypted và secure**

JWT tokens là sensitive data cần encrypt. `SharedPreferences` lưu plaintext.

</details>

### Question 3: Obfuscation

**Để obfuscate Flutter app, cần làm gì?**

A) Enable ProGuard/R8 trong `build.gradle`
B) Chạy `flutter build apk --obfuscate`
C) Cả A và B
D) Sử dụng third-party obfuscation tool

<details>
<summary>Answer</summary>

**C) Cả A và B**

Enable ProGuard trong Gradle config VÀ sử dụng `--obfuscate` flag khi build.

</details>

---

## Practical Demonstration

### Task 1: Show Security Audit (5 min)

Mở codebase và demonstrate:

1. Tìm `FlutterSecureStorage` usage → giải thích platform-specific encryption
2. Tìm `Dio` configuration → explain SSL consideration
3. Chạy `flutter analyze` → show potential security warnings

### Task 2: Demonstrate SSL Pinning (5 min)

1. Mở `SSLPinningService` → trace through code
2. Show cách extract certificate fingerprint
3. Explain certificate rotation strategy

### Task 3: Show Performance Monitor (5 min)

1. Mở `PerformanceMonitor` → trace through implementation
2. Chạy app → show console logs cho navigation/API timing
3. Explain cách extend cho custom metrics

---

## Reflection Questions

Trả lời ngắn gọn (1-2 sentences):

1. **Tại sao không nên hardcode API keys trong source code?**
   
   _______________________________________________________________

2. **Sự khác biệt giữa R8 (Android) và symbol stripping (iOS)?**
   
   _______________________________________________________________

3. **Khi nào nên sử dụng SSL pinning, khi nào không cần?**
   
   _______________________________________________________________

4. **Tại sao debug mode detection quan trọng cho security?**
   
   _______________________________________________________________

---

## Completion Criteria

Để hoàn thành module này, bạn cần:

- [ ] ✅ Hoàn thành **tất cả 3 exercises**
- [ ] ✅ Pass **tất cả quiz questions** (3/3)
- [ ] ✅ Demonstrate được **security patterns** trong codebase
- [ ] ✅ Pass **8/10 practical criteria** hoặc hơn

**Points breakdown:**

| Section | Max Points | Passing |
|---------|------------|---------|
| Security Fundamentals | 25 | 18 |
| SSL Pinning | 20 | 14 |
| Secure Storage | 20 | 14 |
| API Key Protection | 15 | 11 |
| Debug Detection | 10 | 7 |
| Performance Monitoring | 10 | 7 |
| **Total** | **100** | **70** |

---

## Next Steps

✅ **Hoàn thành module MA** → Chuyển sang:
- [Module MB: Advanced Native Features](../module-advanced-B-native-features/) (Push, Camera, Biometrics, Deep Linking)
- [Module MC: Advanced Patterns & Tooling](../module-advanced-C-patterns-tooling/) (State Management, GraphQL, WebSocket, Melos)

❌ **Chưa đạt yêu cầu** → Review lại:
- Đọc kỹ concepts chưa nắm vững
- Làm lại exercises
- Hỏi facilitator

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
