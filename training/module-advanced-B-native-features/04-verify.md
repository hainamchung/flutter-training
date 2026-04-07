# Verify — Advanced Native Features

> 📌 **Hoàn thành [03-exercise.md](./03-exercise.md) trước khi làm verify.**

---

## Verify Checklist

### Phần 1: Camera Integration 🔴

| # | Criteria | Check |
|---|----------|-------|
| 1.1 | Sử dụng được `image_picker` để pick/capture images | [ ] |
| 1.2 | Implement được image crop với `image_cropper` | [ ] |
| 1.3 | Compress được image trước upload | [ ] |
| 1.4 | Handle được camera permission (request, denied, permanently denied) | [ ] |
| 1.5 | Implement được complete avatar upload pipeline | [ ] |

### Phần 2: Location Services 🟡

| # | Criteria | Check |
|---|----------|-------|
| 2.1 | Check được location service enabled/disabled | [ ] |
| 2.2 | Handle được all permission states (denied, deniedForever, etc.) | [ ] |
| 2.3 | Get được current position với accuracy options | [ ] |
| 2.4 | Calculate được distance giữa 2 positions | [ ] |
| 2.5 | Open được app settings khi permission permanently denied | [ ] |

### Phần 3: Biometric Authentication 🟡

| # | Criteria | Check |
|---|----------|-------|
| 3.1 | Check được device biometric support | [ ] |
| 3.2 | Get được available biometric types (fingerprint, face) | [ ] |
| 3.3 | Implement được biometric authentication với fallback | [ ] |
| 3.4 | Handle được authentication errors (notAvailable, notEnrolled, lockedOut) | [ ] |
| 3.5 | Configure được iOS Info.plist và Android manifest | [ ] |

### Phần 4: Push Notifications 🔴

| # | Criteria | Check |
|---|----------|-------|
| 4.1 | Explain được 3 app states và notification handling (foreground, background, terminated) | [ ] |
| 4.2 | Implement được FCM token management (get, refresh) | [ ] |
| 4.3 | Handle được foreground message (show in-app notification) | [ ] |
| 4.4 | Handle được background/terminated message tap (navigate) | [ ] |
| 4.5 | Setup được Android notification channel | [ ] |

### Phần 5: Deep Linking 🟡

| # | Criteria | Check |
|---|----------|-------|
| 5.1 | Phân biệt được Custom Scheme vs App Links/Universal Links | [ ] |
| 5.2 | Configure được custom scheme (myapp://) trên Android | [ ] |
| 5.3 | Configure được App Links (https://) trên Android với autoVerify | [ ] |
| 5.4 | Configure được Universal Links trên iOS | [ ] |
| 5.5 | Implement được URI parsing và navigation | [ ] |

---

## Quick Quiz

### Question 1: Camera

**Để upload avatar, thứ tự xử lý nào đúng?**

A) Capture → Upload → Crop → Compress
B) Capture → Crop → Compress → Upload
C) Crop → Capture → Compress → Upload
D) Capture → Compress → Crop → Upload

<details>
<summary>Answer</summary>

**B) Capture → Crop → Compress → Upload**

Crop trước để loại bỏ phần không mong muốn, compress để giảm kích thước trước khi upload.

</details>

### Question 2: Location

**Khi user permanently denied location permission, app nên làm gì?**

A) Ignore và tiếp tục không dùng location
B) Retry request permission
C) Open app settings để user enable manually
D) Show error và exit app

<details>
<summary>Answer</summary>

**C) Open app settings để user enable manually**

Khi `deniedForever`, user phải manually enable trong Settings. App không thể request lại.

</details>

### Question 3: Push Notifications

**Khi app đang terminated và nhận push notification, user tap notification. App lifecycle nào đúng?**

A) Foreground → handle via onMessage
B) Background → handle via onMessageOpenedApp
C) Terminated → app launches → getInitialMessage
D) App restart từ đầu, không có data

<details>
<summary>Answer</summary>

**C) Terminated → app launches → getInitialMessage**

`getInitialMessage()` được gọi một lần khi app launch từ terminated state.

</details>

### Question 4: Deep Linking

**Sự khác biệt chính giữa Custom Scheme và App Links?**

A) Custom Scheme nhanh hơn
B) App Links dùng HTTPS và được domain verified
C) Custom Scheme chỉ hoạt động trên iOS
D) Không có khác biệt

<details>
<summary>Answer</summary>

**B) App Links dùng HTTPS và được domain verified**

App Links/Universal Links sử dụng HTTPS URLs được verified qua Digital Asset Links, security cao hơn Custom Scheme.

</details>

---

## Practical Demonstration

### Task 1: Camera Pipeline (5 min)

Demonstrate:
1. Pick image from gallery → show image
2. Capture photo with camera → show image
3. Crop to square → show cropped result
4. Show upload progress → confirm upload

### Task 2: Location Feature (3 min)

Demonstrate:
1. Show distance to a fixed location
2. Simulate denied permission → show explanation dialog
3. Show settings opening flow

### Task 3: Biometric Login (3 min)

Demonstrate:
1. Show biometric availability status
2. Trigger biometric prompt
3. Handle success/failure

### Task 4: Push Notifications (5 min)

Demonstrate:
1. Send test notification → show in-app handling
2. Background notification → tap → navigate
3. Explain background handler setup

### Task 5: Deep Links (4 min)

Demonstrate:
1. Open URL scheme → app opens → navigate
2. Open HTTPS link → app opens → navigate
3. Show URI parsing logic

---

## Completion Criteria

Để hoàn thành module này, bạn cần:

- [ ] ✅ Hoàn thành **ít nhất 3/5 exercises**
- [ ] ✅ Pass **ít nhất 3/4 quiz questions** (75%)
- [ ] ✅ Demonstrate được **tất cả 5 features** hoạt động
- [ ] ✅ Pass **15/20 practical criteria** hoặc hơn

**Points breakdown:**

| Section | Max Points | Passing |
|---------|------------|---------|
| Camera Integration | 25 | 18 |
| Location Services | 20 | 14 |
| Biometric Auth | 15 | 11 |
| Push Notifications | 25 | 18 |
| Deep Linking | 15 | 11 |
| **Total** | **100** | **70** |

---

## Next Steps

✅ **Hoàn thành module MB** → Chuyển sang:
- [Module MA: Performance & Security](../module-advanced-A-performance-security/) (Security hardening, monitoring)
- [Module MC: Advanced Patterns & Tooling](../module-advanced-C-patterns-tooling/) (State Management, GraphQL, WebSocket)

❌ **Chưa đạt yêu cầu** → Review lại:
- Đọc kỹ concepts chưa nắm vững
- Làm lại exercises trên device
- Hỏi facilitator

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
