# Exercise — Full Capstone Project

> 📌 **Prerequisites:**
> - Hoàn thành tất cả modules M0-M19
> - Có `base_flutter` project setup
> - Hiểu clean architecture, Riverpod, Dio, freezed, testing
> - Giai đoạn: **5 ngày (1 tuần)**

---

## Exercise Timeline

| Day | Focus | Deliverable |
|-----|-------|-------------|
| Day 1 | Setup & Architecture | Folder structure, models, API |
| Day 2 | Profile View | ViewModel, Page, Settings |
| Day 3 | Edit Profile | Form, Avatar, Validation |
| Day 4 | Testing | Unit, Widget, Golden, Coverage |
| Day 5 | CI/CD & Polish | Pipeline, Error handling, Demo |

---

## Day 1: Setup & Architecture

### Task 1.1: Create Branch & Structure

```bash
# Create feature branch
git checkout -b feature/user-profile

# Create folder structure
mkdir -p lib/features/profile/{data,domain,presentation/widgets}
mkdir -p test/features/profile
```

### Task 1.2: Create Profile Model

```dart
// lib/features/profile/domain/profile_model.dart
// TODO: Create ProfileModel với freezed
// Fields: id, name, email, phone, dateOfBirth, bio, avatarUrl, settings
// Settings: language, darkMode, notificationsEnabled
```

### Task 1.3: Create API Service

```dart
// lib/features/profile/data/profile_api_service.dart
// TODO: Create ProfileApiService
// Methods: getProfile(), updateProfile(), uploadAvatar(File)
// Use Dio client
```

### Task 1.4: Create Mock Interceptor

```dart
// lib/features/profile/data/mock_profile_interceptor.dart
// TODO: Create MockProfileInterceptor
// Mock response cho GET /users/me
// Mock response cho PUT /users/me
// Mock response cho POST /users/me/avatar
```

### Task 1.5: Create Repository

```dart
// lib/features/profile/data/profile_repository.dart
// TODO: Create ProfileRepository
// Methods: getProfile(), updateProfile(), uploadAvatar()
// Implement caching
```

### Deliverable Day 1

- [ ] Branch `feature/user-profile` created
- [ ] Folder structure matches clean architecture
- [ ] Profile model với freezed
- [ ] API service với mock interceptor
- [ ] Repository với caching

---

## Day 2: Profile View

### Task 2.1: Create ProfileViewModel

```dart
// lib/features/profile/presentation/profile_view_model.dart
// TODO: Create @riverpod ProfileViewModel
// Methods: build(), refresh(), updateSettings(), logout()
// State: AsyncValue<ProfileModel?>
```

### Task 2.2: Create ProfilePage

```dart
// lib/features/profile/presentation/profile_page.dart
// TODO: Create ProfilePage
// States: loading (shimmer), error (retry), data (content)
// AppBar với edit button
// Body: avatar, info, settings, logout
```

### Task 2.3: Create UI Widgets

```dart
// lib/features/profile/presentation/widgets/
// TODO: Create widgets:
// - avatar_widget.dart: CachedNetworkImage
// - profile_info_tile.dart: info display
// - settings_section.dart: settings toggles
```

### Task 2.4: Add Navigation Route

```dart
// lib/navigation/app_router.dart
// TODO: Add routes:
// - /profile → ProfileRoute
// - /profile/edit → EditProfileRoute
```

### Task 2.5: Add Auth Guard

```dart
// TODO: Profile routes require authentication
// Add guard với AppNavigator
```

### Deliverable Day 2

- [ ] ProfileViewModel working
- [ ] ProfilePage với all states
- [ ] UI widgets extracted
- [ ] Navigation routes configured
- [ ] Auth guard applied

---

## Day 3: Edit Profile

### Task 3.1: Create EditProfileViewModel

```dart
// lib/features/profile/presentation/edit_profile_view_model.dart
// TODO: Create EditProfileState + EditProfileViewModel
// Methods: updateName(), updatePhone(), updateDateOfBirth(), updateBio(), updateAvatar()
// Form validation
// hasChanges check
// save() method
```

### Task 3.2: Create EditProfilePage

```dart
// lib/features/profile/presentation/edit_profile_page.dart
// TODO: Create EditProfilePage
// Form fields: name, phone, dateOfBirth, bio
// Unsaved changes warning
// Save button
```

### Task 3.3: Implement Avatar Widget

```dart
// lib/features/profile/presentation/widgets/avatar_edit_widget.dart
// TODO: Create AvatarEditWidget
// Camera capture
// Gallery picker
// Image crop (1:1)
// Image compress
// Upload
```

### Task 3.4: Add Form Validation

```dart
// TODO: Add validation rules:
// - Name: required, 2-50 chars
// - Phone: optional, 10-11 digits
// - DateOfBirth: optional, not future, age ≥ 16
// - Bio: optional, max 200 chars
```

### Task 3.5: Implement Unsaved Changes

```dart
// TODO: Implement PopScope
// Show confirmation dialog if hasChanges
// Discard or save options
```

### Deliverable Day 3

- [ ] EditProfileViewModel với form state
- [ ] EditProfilePage với form
- [ ] Avatar widget với camera/gallery
- [ ] Form validation working
- [ ] Unsaved changes handled

---

## Day 4: Testing

### Task 4.1: Unit Tests - Repository

```dart
// test/features/profile/profile_repository_test.dart
// TODO: Test cases:
// - getProfile success
// - getProfile with cache
// - getProfile network error
// - updateProfile success
// - uploadAvatar success
```

### Task 4.2: Unit Tests - ViewModel

```dart
// test/features/profile/profile_view_model_test.dart
// TODO: Test cases:
// - Initial load
// - Refresh
// - Update settings
// - Logout
```

### Task 4.3: Widget Tests

```dart
// test/features/profile/profile_page_test.dart
// TODO: Test cases:
// - Loading state
// - Error state
// - Data state
// - Settings toggles
```

### Task 4.4: Golden Tests

```dart
// test/features/profile/golden/
// TODO: Golden tests:
// - profile_page_light.png
// - profile_page_dark.png
// - edit_profile_page_light.png
// - edit_profile_page_dark.png
```

### Task 4.5: Coverage Check

```bash
# Run coverage
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Verify coverage ≥ 70%
```

### Deliverable Day 4

- [ ] Unit tests với ≥70% coverage
- [ ] Widget tests với all states
- [ ] Golden tests với light/dark
- [ ] Coverage report generated
- [ ] All tests pass

---

## Day 5: CI/CD & Polish

### Task 5.1: Setup CI Pipeline

```yaml
# .github/workflows/profile-feature.yml
# TODO: Create CI pipeline
# Jobs: lint, test, build
# Quality gates: analyze, format, coverage
```

### Task 5.2: Add Error Handling

```dart
// TODO: Review error handling
// - Network errors
// - Validation errors
// - Unknown errors
// - Graceful degradation
```

### Task 5.3: Performance Optimization

```dart
// TODO: Performance checks:
// - Image caching (CachedNetworkImage)
// - Const constructors
// - Selective rebuilds
// - No unnecessary rebuilds
```

### Task 5.4: Create PR

```markdown
# TODO: Create PR với description:
## Changes
- Profile view page
- Edit profile page
- Avatar upload
- Settings section

## Screenshots
| Light | Dark |
|-------|------|
| [img] | [img] |

## Testing
- Unit test coverage: XX%
- Golden tests: 4
- CI status: ✅

## Checklist
- [ ] Lint pass
- [ ] All tests pass
- [ ] Coverage ≥70%
- [ ] CI green
```

### Task 5.5: Demo Prep

```markdown
# TODO: Prepare demo flow:
1. View profile → show all info
2. Edit profile → change name → save
3. Upload avatar → camera → crop → upload
4. Settings → toggle dark mode
5. Logout → confirm dialog
```

### Deliverable Day 5

- [ ] CI pipeline green
- [ ] Error handling complete
- [ ] Performance optimized
- [ ] PR submitted
- [ ] Demo ready

---

## Bonus Challenges ⭐

### Bonus 1: Offline Support

```dart
// TODO: Add offline-first support
// - Show cached profile when offline
// - Queue updates for when online
// - Sync indicator
```

### Bonus 2: Biometric Lock

```dart
// TODO: Add biometric authentication
// - Check biometric on profile page access
// - Fallback to PIN
// - Settings toggle
```

### Bonus 3: Profile Activity Timeline

```dart
// TODO: Add activity timeline
// - Recent changes
// - Avatar updates
// - Login history
```

---

## Submission Checklist

### Code Quality

- [ ] Clean architecture (Presentation → Domain → Data)
- [ ] No circular dependencies
- [ ] Meaningful variable/function names
- [ ] Comments for complex logic
- [ ] No hardcoded strings (use l10n)
- [ ] No hardcoded colors (use AppTheme)

### Functionality

- [ ] Profile view works (loading, data, error)
- [ ] Edit profile works (validation, save)
- [ ] Avatar upload works (camera, gallery, crop, compress)
- [ ] Settings persist
- [ ] Logout clears data

### Testing

- [ ] Unit tests ≥70% coverage
- [ ] Widget tests pass
- [ ] Golden tests match
- [ ] All CI checks green

### Production Ready

- [ ] CI pipeline passes
- [ ] Error handling complete
- [ ] Performance acceptable
- [ ] Demo ready

---

## Hints

### Hint 1: Freezed Setup

```bash
# Add dependencies
flutter pub add freezed_annotation
flutter pub add --dev freezed build_runner json_serializable

# Generate
dart run build_runner build --delete-conflicting-outputs
```

### Hint 2: Mock Setup

```dart
// Use mocktail for mocking
flutter pub add --dev mocktail

// In tests
import 'package:mocktail/mocktail.dart';

class MockProfileRepository extends Mock implements ProfileRepository {}
```

### Hint 3: Golden Test Setup

```dart
// Add golden_file_comparator
flutter pub add --dev flutter_test_goldens

// In test
await expectLater(find.byType(Widget), matchesGoldenFile('golden.png'));
```

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
