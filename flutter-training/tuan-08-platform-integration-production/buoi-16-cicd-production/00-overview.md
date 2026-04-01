# Buổi 16: CI/CD & Production 🎉

> 🟢 **Mức độ ưu tiên: AI-OK — AI hỗ trợ tốt**
> API/config đơn giản, AI generate tốt. Tập trung đọc hiểu + review + verify.

> **Buổi 16/16** · Tuần 8 · CI/CD, Deployment & Capstone Review · **BUỔI CUỐI CÙNG!**

## 🗺️ Vị trí trong lộ trình

```
Tuần 1  ✅ Dart Cơ Bản
Tuần 2  ✅ Widget Fundamentals
Tuần 3  ✅ Navigation & State Cơ Bản
Tuần 4  ✅ State Management Nâng Cao
Tuần 5  ✅ Architecture & DI
Tuần 6  ✅ Networking & Data
Tuần 7  ✅ Performance & Animation
Tuần 8  ✅ Platform Integration & Production
  │
  ├── Buổi 15 ✅ Platform Integration
  └── Buổi 16 ◀── BẠN ĐANG Ở ĐÂY
```

```
Platform Integration ──▶ [BẠN ĐANG Ở ĐÂY: CI/CD & Production] ──▶ 🎓 HOÀN THÀNH!
```

## 🌍 Vai trò trong hệ sinh thái Flutter

CI/CD & Production là bước cuối cùng đưa app từ code thành sản phẩm trên tay user. Build modes, code signing, GitHub Actions, và Fastlane tạo thành release pipeline tự động. Không có CI/CD, mỗi lần release là một cơn ác mộng manual. Đây là kỹ năng bắt buộc cho mọi Flutter developer làm việc trong team.

## 💼 Đóng góp vào dự án thực tế

- **GitHub Actions** — CI tự động: lint, test, build trên mỗi PR → catch bugs sớm
- **Fastlane** — automate deployment lên TestFlight/Play Store Internal Testing
- **Code signing** — ký app đúng cách cho iOS/Android → publish lên store
- **Build modes** — Debug/Profile/Release, biết khi nào dùng mode nào
- **Release pipeline** — từ merge PR → tự động build → upload store → notify team

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- Build modes: Debug vs Profile vs Release — khi nào dùng cái nào
- Code signing flow cho iOS (certificates, provisioning profiles) và Android (keystore)
- CI/CD pipeline concept: trigger → build → test → deploy
- Fastlane lanes và match cho team signing

### ✅ Làm được
- Build release APK/IPA với proper signing
- Setup GitHub Actions CI cho Flutter project (analyze + test + build)
- Sử dụng Fastlane để automate deployment lên TestFlight/Internal Testing
- Deploy ứng dụng lên store (hoặc internal testing track)

### 🚫 Chưa cần biết
- Custom CI servers (Jenkins, TeamCity self-hosted)
- Kubernetes deployment cho backend
- App Store Optimization (ASO) chuyên sâu

## ⏰ Phân bổ thời gian

| Phần | Nội dung | Thời gian |
|------|----------|-----------|
| 1 | Build Modes (Debug/Profile/Release) | ~15 phút |
| 2 | Code Signing (iOS & Android) | ~25 phút |
| 3 | GitHub Actions cho Flutter | ~30 phút |
| 4 | Fastlane Automation | ~25 phút |
| 5 | App Deployment (Store submission) | ~25 phút |
| 6 | Capstone Project Review & Tổng kết | ~30 phút |
| | **Tổng** | **~150 phút** |

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| Build Modes (Debug/Profile/Release) | 🟡 | Phải biết 3 modes và khi nào dùng |
| Tree Shaking, AOT | 🟡 | Hiểu tại sao release build nhỏ hơn |
| Build Flavors | 🟡 | Dev/Staging/Prod — dự án thực bắt buộc |
| Code Signing | 🟡 | Keystore, provisioning — làm 1 lần, AI guide |
| GitHub Actions | 🟢 | YAML workflow — AI viết rất tốt |
| Fastlane | 🟢 | Automation scripts — AI generate |
| App Deployment | 🟡 | Play Store / App Store — checklist cần biết |
| Shorebird (Code Push) | 🟢 | AI docs tốt |
| Release Checklist | 🟡 | Pre-release verification |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|----------|
| VD GitHub Actions workflow | 🟢 | AI viết YAML tốt |
| VD Fastlane setup | 🟢 | AI generate lane scripts |
| VD Build Flavors | 🟡 | Cần tự chạy 1 lần |
| BT1 Build Release | 🟡 | Tự chạy 1 lần |
| BT2-3 CI/CD pipeline | 🟢 | AI viết, bạn review |

---

## 📋 Tổng quan bài tập

| Bài | Độ khó | Loại | Chạy | Mô tả |
|-----|--------|------|------|--------|
| BT1 | ⭐ | CLI | `flutter build` | Build release APK/IPA — tạo keystore, sign app, kiểm tra size |
| BT2 | ⭐⭐ | CI/CD | GitHub Actions | Setup GitHub Actions CI — analyze + test + build trên mỗi push |
| BT3 | ⭐⭐⭐ | CI/CD | GitHub Actions + Fastlane | Full CI/CD pipeline — GitHub Actions + Fastlane → TestFlight/Internal Testing |
| 🤖 AI-BT1 | ⭐⭐⭐ | CI/CD | GitHub Actions | Full AI-first CI/CD setup + production checklist → review secrets + optimization |

## 🔗 Liên kết bài học

| File | Nội dung |
|------|----------|
| [01-ly-thuyet.md](./01-ly-thuyet.md) | Build modes, code signing, CI/CD, Fastlane, deployment, capstone review |
| [02-vi-du.md](./02-vi-du.md) | 5 ví dụ thực tế: build commands, GitHub Actions, Fastlane, signing, release script |
| [03-thuc-hanh.md](./03-thuc-hanh.md) | 3 bài tập + câu hỏi thảo luận + 🤖 bài tập AI |
| [04-tai-lieu-tham-khao.md](./04-tai-lieu-tham-khao.md) | Tài liệu, tools, lộ trình học tiếp |

## 📅 Tiến độ chương trình

> Buổi 16/16 — Hoàn thành 100% lộ trình
> ████████████████ 🎉🎓

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | ✅ Hoàn thành |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ✅ Hoàn thành |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ✅ Hoàn thành |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ✅ Hoàn thành |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ✅ Hoàn thành |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ✅ Hoàn thành |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | 🔵 Đang học |

> **🎉 Chúc mừng!** Đây là buổi học cuối cùng. Sau buổi này, bạn đã sẵn sàng xây dựng và phát hành ứng dụng Flutter hoàn chỉnh!
