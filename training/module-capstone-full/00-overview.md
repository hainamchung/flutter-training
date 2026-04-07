# Capstone Module — Full Capstone Project

> **⏱️ Thời lượng ước tính:** 1 tuần (full-time)
>
> 🗺️ **Capstone Path:** Đây là **Full Capstone** - capstone cuối cùng và quan trọng nhất trong chương trình đào tạo Flutter. Hoàn thành M0-M19 và các Optional modules (MA, MB, MC) trước khi bắt đầu.

---

## Mục tiêu

Sau capstone này, bạn sẽ:
- Xây dựng feature hoàn chỉnh end-to-end từ spec → implementation → testing → CI/CD
- Áp dụng tất cả kiến thức từ M0-M19 vào production-ready code
- Đánh giá năng lực middle-level Flutter developer
- Tạo portfolio piece để demonstrate skills

---

## Prerequisites

### Required Modules Completed

| Module | Concept cần nắm | Relevance |
|--------|-----------------|-----------|
| **M0-M19** | Tất cả core modules | Foundation |
| **MA** | Security, performance monitoring | Production readiness |
| **MB** | Native features | User profile với camera, biometrics |
| **MC** | Advanced patterns | State management, GraphQL (optional) |

### Technical Requirements

- Flutter 3.x + Dart 3.x
- base_flutter project setup
- Git workflow knowledge
- CI/CD basic understanding (M19)

---

## Nội dung

| File | Nội dung | Thời lượng |
|------|----------|------------|
| [01-code-walk.md](./01-code-walk.md) | Full implementation walkthrough | ~2 hrs |
| [02-concept.md](./02-concept.md) | Capstone-level concepts | ~1 hr |
| [03-exercise.md](./03-exercise.md) | Full project exercises | 5 days |
| [04-verify.md](./04-verify.md) | Capstone evaluation rubric | ~1 hr |

---

## Capstone Feature: User Profile Feature

### Overview

Build **User Profile Feature** trên `base_flutter` project — feature hoàn chỉnh bao gồm:

```
┌─────────────────────────────────────────────────────────────────┐
│                    USER PROFILE FEATURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Profile Page (/profile)          Edit Profile (/profile/edit)  │
│  ┌─────────────────────┐         ┌─────────────────────────┐   │
│  │ 👤 Avatar           │         │ 📷 Change Avatar        │   │
│  │    (tap to change)   │         │ 📝 Name: [___________] │   │
│  │                     │         │ 📧 Email: [___________] │   │
│  │ Name: Nguyễn Văn A  │         │ 📱 Phone: [___________] │   │
│  │ Email: a@example.com│         │ 📅 DOB: [___/___/___]   │   │
│  │ Phone: 0901234567   │         │ 📖 Bio: [___________]   │   │
│  │                     │         │                         │   │
│  │ ─── Settings ───    │         │ [Cancel]    [Save]      │   │
│  │ 🔔 Notifications: ON│         └─────────────────────────┘   │
│  │ 🌙 Dark Mode: OFF  │                                          │
│  │ 🌐 Language: Tiếng Việt│                                     │
│  │                     │                                          │
│  │ [Chỉnh sửa] [Đăng xuất]│                                     │
│  └─────────────────────┘                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Feature Scope

#### ✅ In Scope (Required)

1. **Profile View Page**
   - Display user info: avatar, name, email, phone, date of birth, bio
   - Settings section: notifications, dark mode, language
   - Edit button → navigate to edit page
   - Logout button → confirm dialog → logout

2. **Edit Profile Page**
   - Form fields: avatar, name, phone, date of birth, bio
   - Avatar: camera capture, gallery picker, crop to square, compress, upload
   - Form validation: required fields, format validation
   - Unsaved changes warning on back navigation
   - Save: API call → loading → success toast / error message

3. **API Integration**
   - GET /api/v1/users/me — fetch profile
   - PUT /api/v1/users/me — update profile
   - POST /api/v1/users/me/avatar — upload avatar (multipart)

4. **Local Storage**
   - Cache profile locally for offline access
   - Persist settings (dark mode, language)

5. **Testing**
   - Unit tests: ≥70% coverage
   - Widget tests: ProfilePage, EditProfilePage
   - Golden tests: light/dark mode

6. **CI/CD**
   - PR check: lint + test + build
   - CI pipeline green

#### ❌ Out of Scope

- Social features (follow, share)
- Chat/messaging
- Real-time updates
- Push notifications
- Complex animations

---

## Technical Architecture

### Module Mapping

| Module | Requirements |
|--------|-------------|
| **M0** | Null safety, extension methods, generics |
| **M1** | Environment-specific builds |
| **M2** | Clean architecture layers |
| **M3** | AppTheme, assets |
| **M4** | AppException, Result type |
| **M5** | Built-in Widgets: layout, input, display, list, navigation widgets |
| **M6** | Custom Widgets & Animation: widget lifecycle, composition, animation basics |
| **M7** | BaseViewModel, loading/error states |
| **M8** | Riverpod state management |
| **M9** | Page UI structure |
| **M10** | BaseViewModel Page, BasePage, runCatching |
| **M11** | Riverpod Advanced: provider types, ref API, autoDispose |
| **M12** | Dio, multipart upload |
| **M13** | Error handling, interceptors |
| **M14** | SharedPreferences, secure storage |
| **M15** | Popup, Dialog & Paging: BasePopup, paging executor |
| **M16** | Lint & Code Quality: analysis options, custom lint |
| **M17** | Architecture & DI: get_it, dependency injection |
| **M18** | Unit, widget, golden tests |
| **M19** | CI/CD pipeline |
| **MA** | Performance monitoring, secure storage |
| **MB** | Camera, biometrics (optional) |
| **MC** | Advanced patterns (optional) |

### Folder Structure

```
lib/
├── features/
│   └── profile/
│       ├── data/
│       │   ├── profile_repository.dart
│       │   └── profile_api_service.dart
│       ├── domain/
│       │   └── profile_model.dart
│       └── presentation/
│           ├── profile_page.dart
│           ├── profile_view_model.dart
│           ├── edit_profile_page.dart
│           ├── edit_profile_view_model.dart
│           └── widgets/
│               ├── avatar_widget.dart
│               ├── profile_info_tile.dart
│               └── settings_section.dart
```

---

## Deliverables

| # | Deliverable | Mô tả | Deadline |
|---|-------------|--------|----------|
| 1 | **Source Code** | Feature branch: `feature/user-profile` | End of Week |
| 2 | **Unit Tests** | ≥70% coverage cho profile module | End of Week |
| 3 | **Widget Tests** | ProfilePage, EditProfilePage states | End of Week |
| 4 | **Golden Tests** | Light + dark mode | End of Week |
| 5 | **CI Pipeline** | PR check workflow pass | End of Week |
| 6 | **PR Description** | Changes, screenshots, testing notes | End of Week |
| 7 | **Demo** | Live demo + Q&A | Demo Day |

---

## Evaluation Rubric

### Scoring Breakdown

| Tiêu chí | Trọng số | Chi tiết |
|-----------|----------|----------|
| **Architecture** | 25% | Đúng layer, đúng pattern, clean dependencies |
| **Functionality** | 20% | Feature hoạt động đúng spec, edge cases handled |
| **Code Quality** | 20% | Readable, maintainable, follows conventions |
| **Testing** | 20% | Coverage ≥70%, test quality, assertions |
| **Production Readiness** | 15% | CI pass, error handling, performance, UX |

### Score Levels

| Score | Level | Mô tả |
|-------|-------|--------|
| **90-100%** | **Excellent** | Vượt expectations, production-ready |
| **75-89%** | **Pass** ✅ | Đạt chuẩn middle-level |
| **60-74%** | **Conditional Pass** 🟡 | Cần fix trong 3 ngày |
| **< 60%** | **Fail** 🔴 | Cần thêm training time |

---

## Timeline — 1 Tuần

### Day 1: Setup & Architecture

| Task | Deliverable |
|------|-------------|
| Create branch `feature/user-profile` | Git branch |
| Create folder structure | Clean architecture |
| Create data models | Profile model với freezed |
| Create API service | Dio client, mock interceptor |
| Create repository | Data layer abstraction |

### Day 2: Profile View

| Task | Deliverable |
|------|-------------|
| ProfileViewModel | State management |
| ProfilePage UI | View profile screen |
| Settings section | Dark mode, language, notifications |
| Navigation guard | Auth check |
| Image caching | CachedNetworkImage |

### Day 3: Edit Profile

| Task | Deliverable |
|------|-------------|
| EditProfileViewModel | Form state |
| EditProfilePage UI | Form fields |
| Avatar widget | Camera/gallery picker |
| Image crop/compress | image_cropper, flutter_image_compress |
| Form validation | Required fields, format |

### Day 4: Testing

| Task | Deliverable |
|------|-------------|
| Unit tests | Repository, ViewModel tests |
| Widget tests | Page states (loading, data, error) |
| Golden tests | Light/dark mode |
| Coverage check | ≥70% |

### Day 5: CI/CD & Polish

| Task | Deliverable |
|------|-------------|
| CI pipeline | lint + test + build |
| Error handling | Global error handler |
| Performance | Performance monitoring |
| Documentation | PR description, comments |
| Demo prep | Slides, demo flow |

---

## Milestone Checkpoints

| Day | Milestone | Review | Criteria |
|-----|-----------|--------|----------|
| Day 1 | M1: Setup | Self-check | Folder structure, models, API |
| Day 2 | M2: View | Facilitator | Architecture, navigation |
| Day 3 | M3: Edit | Peer review | UI, validation, upload |
| Day 4 | M4: Tests | Facilitator | Coverage, test quality |
| Day 5 | M5: CI + Demo | Team | Pipeline green, demo ready |

---

## Resources

- **Base Flutter Project:** `../../base_flutter/`
- **Training Modules:** `../module-00-dart-primer/` → `../module-19-cicd/`
- **Advanced Modules:** `../module-advanced-A-performance-security/`, `../module-advanced-B-native-features/`, `../module-advanced-C-patterns-tooling/`
- **Rubric:** `../tieu-chuan/middle-level-rubric.md`
- **AI Toolkit:** `../ai-toolkit/ai-driven-development.md`, `../ai-toolkit/prompt-dojo.md`

---

<!-- AI_VERIFY: generation-complete -->
