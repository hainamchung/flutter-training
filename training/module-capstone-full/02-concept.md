# Concepts — Full Capstone Project

> 📌 **Module context:** Capstone-level concepts tổng hợp kiến thức từ M0-M19 và Optional modules (MA, MB, MC).

---

## Concept 1: Clean Architecture in Capstone 🔴 MUST-KNOW

**WHY:** Capstone yêu cầu clean architecture hoàn chỉnh — đây là điểm đánh giá quan trọng nhất.

### Clean Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLEAN ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    PRESENTATION LAYER                       │ │
│  │  ProfilePage → ProfileViewModel → EditProfilePage          │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                      DOMAIN LAYER                           │ │
│  │  ProfileModel → ProfileRepository (interface)               │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                       DATA LAYER                            │ │
│  │  ProfileApiService → MockInterceptor → Repository impl     │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Dependency Rule

```dart
// ✅ CORRECT: Presentation → Domain → Data
// Presentation depends on Domain
// Domain depends on abstract Data (Repository interface)
// Data implements Domain interfaces

// ❌ WRONG: Data depends on Presentation
// ❌ WRONG: Domain depends on Data
```

### Feature-First Structure

```
lib/features/profile/
├── data/
│   ├── profile_api_service.dart    // API calls
│   ├── profile_repository_impl.dart // Repository implementation
│   └── mock_profile_interceptor.dart // Mock data
├── domain/
│   ├── profile_model.dart         // Data model
│   └── profile_repository.dart     // Repository interface
└── presentation/
    ├── profile_page.dart          // UI
    ├── profile_view_model.dart    // State
    ├── edit_profile_page.dart     // Edit UI
    ├── edit_profile_view_model.dart // Edit state
    └── widgets/                    // Reusable widgets
        ├── avatar_widget.dart
        ├── profile_info_tile.dart
        └── settings_section.dart
```

> 💡 **FE Perspective**
> **Flutter:** Clean architecture = Presentation → Domain → Data layers.
> **React/Vue tương đương:** Presentation → Business Logic → Data layers.
> **Khác biệt quan trọng:** Flutter có thêm Base classes (BaseViewModel, BasePage) trong base_flutter.

---

## Concept 2: State Management Selection 🟡 SHOULD-KNOW

**WHY:** Chọn đúng state management approach ảnh hưởng đến code quality và testability.

### Riverpod for Profile Feature

```dart
// ProfileViewModel — StateNotifier pattern với Riverpod
@riverpod
class ProfileViewModel extends _$ProfileViewModel {
  @override
  Future<ProfileModel?> build() async {
    // Initial state: fetch profile
    return await _repository.getProfile();
  }
  
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _repository.getProfile());
  }
}
```

### AsyncValue State Pattern

```dart
// AsyncValue = Loading + Error + Data states
Widget build(BuildContext context, WidgetRef ref) {
  final profileAsync = ref.watch(profileViewModelProvider);
  
  return profileAsync.when(
    loading: () => const ShimmerLoading(),
    error: (error, stack) => ErrorWidget(
      error: error,
      onRetry: () => ref.invalidate(profileViewModelProvider),
    ),
    data: (profile) => profile != null
        ? ProfileContent(profile: profile)
        : const ProfileEmpty(),
  );
}
```

### Form State Management

```dart
// EditProfileState — Form fields
@freezed
class EditProfileState with _$EditProfileState {
  const factory EditProfileState({
    required String name,
    required String phone,
    String? dateOfBirth,
    required String bio,
    File? avatarFile,
    @Default(false) bool isLoading,
    String? error,
  }) = _EditProfileState;
}

// Form changes update state
void updateName(String name) {
  state = state.copyWith(name: name);
}
```

### Optimistic Updates

```dart
// Update settings với optimistic UI
Future<void> updateSettings(ProfileSettings settings) async {
  final currentProfile = state.value;
  
  // 1. Optimistic update
  state = AsyncData(currentProfile.copyWith(settings: settings));
  
  try {
    // 2. API call
    await _repository.updateProfile();
  } catch (e) {
    // 3. Rollback on error
    state = AsyncData(currentProfile);
    rethrow;
  }
}
```

---

## Concept 3: Testing Strategy 🟡 SHOULD-KNOW

**WHY:** Testing quality quyết định 20% của capstone score.

### Test Pyramid

```
┌─────────────────────────────────────────────────────────────────┐
│                      TEST PYRAMID                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                         ┌───────┐                               │
│                        │ Golden │                               │
│                       │  Tests  │  ← Few, high confidence      │
│                      └─────────┘                                │
│                    ┌───────────────┐                            │
│                   │  Widget Tests  │  ← Medium, UI testing      │
│                  └─────────────────┘                            │
│               ┌───────────────────────────┐                     │
│              │       Unit Tests          │  ← Many, fast        │
│             └─────────────────────────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Unit Tests

```dart
// test/features/profile/profile_repository_test.dart
void main() {
  group('ProfileRepository', () {
    late ProfileRepository repository;
    late MockProfileApiService mockApi;
    
    setUp(() {
      mockApi = MockProfileApiService();
      repository = ProfileRepository(apiService: mockApi);
    });
    
    test('getProfile returns profile on success', () async {
      // Arrange
      final mockProfile = ProfileModel(
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
      );
      when(() => mockApi.getProfile())
          .thenAnswer((_) async => mockProfile);
      
      // Act
      final result = await repository.getProfile();
      
      // Assert
      expect(result, equals(mockProfile));
      verify(() => mockApi.getProfile()).called(1);
    });
    
    test('getProfile throws on network error', () async {
      // Arrange
      when(() => mockApi.getProfile())
          .thenThrow(NetworkException('No connection'));
      
      // Assert
      expect(
        () => repository.getProfile(),
        throwsA(isA<NetworkException>()),
      );
    });
  });
}
```

### Widget Tests

```dart
// test/features/profile/profile_page_test.dart
void main() {
  group('ProfilePage', () {
    testWidgets('shows loading state', (tester) async {
      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            profileViewModelProvider.overrideWith(
              () => ProfileViewModel(MockRef()),
            ),
          ],
          child: const MaterialApp(home: ProfilePage()),
        ),
      );
      
      expect(find.byType(CircularProgressIndicator), findsOneWidget);
    });
    
    testWidgets('shows profile data when loaded', (tester) async {
      final mockProfile = ProfileModel(
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
      );
      
      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            profileViewModelProvider.overrideWith(
              () => ProfileViewModel(MockRef(mockProfile)),
            ),
          ],
          child: const MaterialApp(home: ProfilePage()),
        ),
      );
      
      expect(find.text('Test User'), findsOneWidget);
      expect(find.text('test@example.com'), findsOneWidget);
    });
  });
}
```

### Coverage Requirements

| Module | Coverage Required |
|--------|------------------|
| `profile_repository.dart` | ≥80% |
| `profile_view_model.dart` | ≥80% |
| `edit_profile_view_model.dart` | ≥70% |
| **Total** | **≥70%** |

---

## Concept 4: CI/CD Pipeline 🟡 SHOULD-KNOW

**WHY:** CI/CD là 15% của score — phải green.

### Pipeline Stages

```
┌─────────────────────────────────────────────────────────────────┐
│                      CI/CD PIPELINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Lint   │───▶│  Test   │───▶│  Build  │───▶│ Deploy  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│                                                                  │
│  flutter     flutter      flutter        flutter                │
│  analyze     test         build apk      build ios              │
│                                                                  │
│  • dart      • unit       • debug        • release             │
│    format    • widget     • release      • sign                 │
│  • dart      • coverage   • split-debug  • artifact            │
│    fix       • goldens                    • upload              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Quality Gates

| Gate | Threshold | Action on Fail |
|------|-----------|----------------|
| Lint | 0 warnings | Block PR |
| Format | No diff | Block PR |
| Unit Tests | 100% pass | Block PR |
| Coverage | ≥70% | Block PR |
| Build | Success | Block PR |

### PR Requirements

```markdown
## PR Checklist
- [ ] Lint pass (flutter analyze)
- [ ] Format pass (dart format)
- [ ] All tests pass
- [ ] Coverage ≥70%
- [ ] Build success
- [ ] No hardcoded strings (i18n)
- [ ] No hardcoded colors (AppTheme)
- [ ] Error handling complete
```

---

## Concept 5: Error Handling & Edge Cases 🟡 SHOULD-KNOW

**WHY:** Production-ready code phải handle errors gracefully.

### Error Scenarios

| Scenario | Handling |
|----------|----------|
| Network error on load | Show cached data + error banner |
| Network error on save | Show error snackbar, keep form data |
| Validation error | Show inline error message |
| Avatar upload fail | Show error, keep current avatar |
| Session expired | Redirect to login |
| Unknown error | Generic error message + log |

### Error State UI

```dart
class ProfileErrorWidget extends StatelessWidget {
  final Object error;
  final VoidCallback onRetry;
  
  const ProfileErrorWidget({
    super.key,
    required this.error,
    required this.onRetry,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            Icons.error_outline,
            size: 64,
            color: Theme.of(context).colorScheme.error,
          ),
          const SizedBox(height: 16),
          Text(
            l10n.errorLoadingProfile,
            style: Theme.of(context).textTheme.titleMedium,
          ),
          const SizedBox(height: 8),
          Text(
            error.toString(),
            style: Theme.of(context).textTheme.bodySmall,
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 24),
          ElevatedButton.icon(
            onPressed: onRetry,
            icon: const Icon(Icons.refresh),
            label: Text(l10n.retry),
          ),
        ],
      ),
    );
  }
}
```

### Unsaved Changes Handling

```dart
// PopScope for unsaved changes
PopScope(
  canPop: !state.hasChanges,
  onPopInvokedWithResult: (didPop, result) async {
    if (didPop) return;
    
    final discard = await showConfirmDialog(
      context: context,
      title: l10n.discardChanges,
      message: l10n.discardChangesMessage,
      confirmText: l10n.discard,
      isDestructive: true,
    );
    
    if (discard && context.mounted) {
      Navigator.of(context).pop();
    }
  },
  child: Scaffold(
    // ...
  ),
)
```

---

## Concept 6: Performance Considerations 🟢 AI-GENERATE

**WHY:** Profile page phải load nhanh và scroll mượt.

### Image Optimization

```dart
// CachedNetworkImage for avatar
CachedNetworkImage(
  imageUrl: avatarUrl,
  imageBuilder: (context, imageProvider) => CircleAvatar(
    backgroundImage: imageProvider,
    radius: 60,
  ),
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const DefaultAvatar(),
  // Cache options
  cacheManager: DefaultCacheManager(),
  memCacheWidth: 200,
  memCacheHeight: 200,
)
```

### Lazy Loading

```dart
// Lazy load heavy sections
@override
Widget build(BuildContext context) {
  return CustomScrollView(
    slivers: [
      const SliverToBoxAdapter(child: ProfileHeader()), // Always visible
      SliverToBoxAdapter(
        child: LazyLoad(
          onVisible: () => ref.read(analyticsProvider).trackProfileView(),
          child: ProfileAnalytics(),
        ),
      ),
      // More slivers as needed
    ],
  );
}
```

### Const Constructors

```dart
// Use const where possible
class ProfileContent extends StatelessWidget {
  const ProfileContent({super.key, required this.profile});
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: const [
        ProfileAvatar(),      // const
        SizedBox(height: 24), // const
        ProfileInfoSection(), // const
        SizedBox(height: 24), // const
      ],
    );
  }
}
```

---

## Concept Map — Capstone Integration

```
Concept 1: Clean Architecture → Structure tổ chức code
Concept 2: State Management   → Riverpod + AsyncValue
Concept 3: Testing Strategy  → Unit + Widget + Golden + Coverage
Concept 4: CI/CD Pipeline    → GitHub Actions workflow
Concept 5: Error Handling    → Graceful error states
Concept 6: Performance      → Image cache, lazy load, const
```

**Capstone Success Formula:**

```
Clean Architecture × State Management × Testing × CI/CD × Error Handling × Performance
```

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
