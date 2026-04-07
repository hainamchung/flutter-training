# Code Walk — Full Capstone Implementation

> 📌 **Prerequisites:**
> - Hoàn thành tất cả modules M0-M19 và Optional (MA, MB, MC)
> - Có `base_flutter` project setup
> - Hiểu clean architecture, Riverpod, Dio, freezed

---

## Walk Order

```
Profile Model (data model + freezed)
    ↓
Profile API Service (Dio + mock interceptor)
    ↓
Profile Repository (data layer abstraction)
    ↓
ProfileViewModel (state management)
    ↓
ProfilePage (view UI)
    ↓
EditProfileViewModel (form state)
    ↓
EditProfilePage (form UI + avatar)
    ↓
CI/CD Pipeline (GitHub Actions)
```

---

## 1. Profile Model

```dart
// lib/features/profile/domain/profile_model.dart
@freezed
class ProfileModel with _$ProfileModel {
  const factory ProfileModel({
    required String id,
    required String name,
    required String email,
    String? phone,
    String? dateOfBirth,
    String? bio,
    String? avatarUrl,
    @Default(ProfileSettings()) ProfileSettings settings,
  }) = _ProfileModel;

  factory ProfileModel.fromJson(Map<String, dynamic> json) =>
      _$ProfileModelFromJson(json);
}

@freezed
class ProfileSettings with _$ProfileSettings {
  const factory ProfileSettings({
    @Default('vi') String language,
    @Default(false) bool darkMode,
    @Default(true) bool notificationsEnabled,
  }) = _ProfileSettings;

  factory ProfileSettings.fromJson(Map<String, dynamic> json) =>
      _$ProfileSettingsFromJson(json);
}
```

### Freezed Generated Code

```dart
// lib/features/profile/domain/profile_model.g.dart (auto-generated)
part of 'profile_model.dart';

_$_ProfileModel _$$_ProfileModelToJson(_$_ProfileModel instance) =>
    <String, dynamic>{
      'id': instance.id,
      'name': instance.name,
      'email': instance.email,
      'phone': instance.phone,
      'dateOfBirth': instance.dateOfBirth,
      'bio': instance.bio,
      'avatarUrl': instance.avatarUrl,
      'settings': instance.settings,
    };

// ... more generated code
```

---

## 2. Profile API Service

```dart
// lib/features/profile/data/profile_api_service.dart
class ProfileApiService {
  final Dio _dio;
  
  ProfileApiService({Dio? dio}) : _dio = dio ?? Dio();
  
  Future<ProfileModel> getProfile() async {
    final response = await _dio.get('/api/v1/users/me');
    return ProfileModel.fromJson(response.data);
  }
  
  Future<ProfileModel> updateProfile({
    String? name,
    String? phone,
    String? dateOfBirth,
    String? bio,
  }) async {
    final response = await _dio.put(
      '/api/v1/users/me',
      data: {
        if (name != null) 'name': name,
        if (phone != null) 'phone': phone,
        if (dateOfBirth != null) 'dateOfBirth': dateOfBirth,
        if (bio != null) 'bio': bio,
      },
    );
    return ProfileModel.fromJson(response.data);
  }
  
  Future<String> uploadAvatar(File file) async {
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(
        file.path,
        filename: 'avatar_${DateTime.now().millisecondsSinceEpoch}.jpg',
      ),
    });
    
    final response = await _dio.post(
      '/api/v1/users/me/avatar',
      data: formData,
    );
    
    return response.data['avatar_url'] as String;
  }
}
```

### Mock Interceptor

```dart
// lib/features/profile/data/mock_profile_interceptor.dart
class MockProfileInterceptor extends Interceptor {
  static const _mockProfile = {
    'id': 'user_001',
    'name': 'Nguyễn Văn A',
    'email': 'a@example.com',
    'phone': '0901234567',
    'dateOfBirth': '1995-06-15',
    'bio': 'Flutter Developer',
    'avatarUrl': null,
    'settings': {
      'language': 'vi',
      'darkMode': false,
      'notificationsEnabled': true,
    },
  };
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (options.path.contains('/users/me') && options.method == 'GET') {
      handler.resolve(Response(
        requestOptions: options,
        statusCode: 200,
        data: _mockProfile,
      ));
      return;
    }
    
    if (options.path.contains('/users/me') && options.method == 'PUT') {
      final updated = {..._mockProfile, ...options.data as Map};
      handler.resolve(Response(
        requestOptions: options,
        statusCode: 200,
        data: updated,
      ));
      return;
    }
    
    handler.next(options);
  }
}
```

---

## 3. Profile Repository

```dart
// lib/features/profile/data/profile_repository.dart
class ProfileRepository {
  final ProfileApiService _apiService;
  final AppPreferences _preferences;
  
  ProfileRepository({
    ProfileApiService? apiService,
    AppPreferences? preferences,
  }) : _apiService = apiService ?? ProfileApiService(),
       _preferences = preferences ?? AppPreferences();
  
  Future<ProfileModel> getProfile() async {
    try {
      final profile = await _apiService.getProfile();
      await _cacheProfile(profile);
      return profile;
    } catch (e) {
      // Try cache if network fails
      final cached = await _getCachedProfile();
      if (cached != null) return cached;
      rethrow;
    }
  }
  
  Future<ProfileModel> updateProfile({
    String? name,
    String? phone,
    String? dateOfBirth,
    String? bio,
  }) async {
    return await _apiService.updateProfile(
      name: name,
      phone: phone,
      dateOfBirth: dateOfBirth,
      bio: bio,
    );
  }
  
  Future<String> uploadAvatar(File file) async {
    // Compress before upload
    final compressed = await _compressImage(file);
    return await _apiService.uploadAvatar(compressed);
  }
  
  Future<void> _cacheProfile(ProfileModel profile) async {
    await _preferences.saveProfileCache(profile.toJson());
  }
  
  Future<ProfileModel?> _getCachedProfile() async {
    final cached = await _preferences.getProfileCache();
    if (cached != null) {
      return ProfileModel.fromJson(cached);
    }
    return null;
  }
  
  Future<File> _compressImage(File file) async {
    final result = await FlutterImageCompress.compressAndGetFile(
      file.absolute.path,
      '${file.path}_compressed.jpg',
      quality: 70,
      minWidth: 800,
      minHeight: 800,
    );
    return result ?? file;
  }
}
```

---

## 4. Profile ViewModel

```dart
// lib/features/profile/presentation/profile_view_model.dart
@riverpod
class ProfileViewModel extends _$ProfileViewModel {
  @override
  Future<ProfileModel?> build() async {
    return await ref.read(profileRepositoryProvider).getProfile();
  }
  
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => 
      ref.read(profileRepositoryProvider).getProfile()
    );
  }
  
  Future<void> updateSettings(ProfileSettings settings) async {
    final currentProfile = state.value;
    if (currentProfile == null) return;
    
    // Optimistic update
    state = AsyncData(currentProfile.copyWith(settings: settings));
    
    try {
      await ref.read(profileRepositoryProvider).updateProfile();
    } catch (e) {
      // Rollback on error
      state = AsyncData(currentProfile);
      rethrow;
    }
  }
  
  Future<void> logout() async {
    await ref.read(appPreferencesProvider).clearAll();
    ref.read(authStateProvider.notifier).logout();
  }
}
```

---

## 5. Profile Page

```dart
// lib/features/profile/presentation/profile_page.dart
class ProfilePage extends ConsumerWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final profileAsync = ref.watch(profileViewModelProvider);
    
    return profileAsync.when(
      loading: () => const ProfileShimmer(),
      error: (error, stack) => ProfileErrorWidget(
        error: error,
        onRetry: () => ref.invalidate(profileViewModelProvider),
      ),
      data: (profile) => profile != null
          ? ProfileContent(profile: profile)
          : const ProfileEmpty(),
    );
  }
}

class ProfileContent extends ConsumerWidget {
  final ProfileModel profile;
  
  const ProfileContent({super.key, required this.profile});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(
        title: Text(l10n.profile),
        actions: [
          IconButton(
            icon: const Icon(Icons.edit),
            onPressed: () => AppNavigator.push(const EditProfileRoute()),
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            ProfileAvatar(
              avatarUrl: profile.avatarUrl,
              onTap: () => AppNavigator.push(const EditProfileRoute()),
            ),
            const SizedBox(height: 24),
            ProfileInfoSection(profile: profile),
            const SizedBox(height: 24),
            SettingsSection(
              settings: profile.settings,
              onSettingsChanged: (settings) {
                ref.read(profileViewModelProvider.notifier)
                    .updateSettings(settings);
              },
            ),
            const SizedBox(height: 24),
            LogoutButton(
              onPressed: () => _showLogoutDialog(context, ref),
            ),
          ],
        ),
      ),
    );
  }
  
  Future<void> _showLogoutDialog(BuildContext context, WidgetRef ref) async {
    final confirmed = await showConfirmDialog(
      context: context,
      title: l10n.logout,
      message: l10n.logoutConfirmMessage,
      confirmText: l10n.logout,
      isDestructive: true,
    );
    
    if (confirmed) {
      await ref.read(profileViewModelProvider.notifier).logout();
    }
  }
}
```

---

## 6. Edit Profile ViewModel

```dart
// lib/features/profile/presentation/edit_profile_view_model.dart
@riverpod
class EditProfileViewModel extends _$EditProfileViewModel {
  @override
  EditProfileState build(ProfileModel initialProfile) {
    return EditProfileState(
      name: initialProfile.name,
      phone: initialProfile.phone ?? '',
      dateOfBirth: initialProfile.dateOfBirth,
      bio: initialProfile.bio ?? '',
      avatarFile: null,
      isLoading: false,
      error: null,
    );
  }
  
  void updateName(String name) {
    state = state.copyWith(name: name);
  }
  
  void updatePhone(String phone) {
    state = state.copyWith(phone: phone);
  }
  
  void updateDateOfBirth(String dateOfBirth) {
    state = state.copyWith(dateOfBirth: dateOfBirth);
  }
  
  void updateBio(String bio) {
    state = state.copyWith(bio: bio);
  }
  
  void updateAvatar(File file) {
    state = state.copyWith(avatarFile: file);
  }
  
  bool get hasChanges {
    // Compare with initial profile
    final initial = ref.read(initialProfileProvider);
    return state.name != initial.name ||
           state.phone != (initial.phone ?? '') ||
           state.dateOfBirth != (initial.dateOfBirth ?? '') ||
           state.bio != (initial.bio ?? '') ||
           state.avatarFile != null;
  }
  
  Future<void> save() async {
    if (!hasChanges) return;
    
    state = state.copyWith(isLoading: true, error: null);
    
    try {
      // Upload avatar if changed
      // Note: avatar upload is handled separately and returns the new URL
      // In this flow, we show the pattern but the avatar URL is managed by the API
      final avatarUrl = state.avatarFile != null
          ? await ref.read(profileRepositoryProvider)
              .uploadAvatar(state.avatarFile!)
          : null;
      // The API might return the updated profile with new avatar URL automatically

      // Update profile
      await ref.read(profileRepositoryProvider).updateProfile(
        name: state.name,
        phone: state.phone.isNotEmpty ? state.phone : null,
        dateOfBirth: state.dateOfBirth.isNotEmpty ? state.dateOfBirth : null,
        bio: state.bio.isNotEmpty ? state.bio : null,
        // avatarUrl is updated via uploadAvatar, not updateProfile
      );
      
      // Refresh profile
      ref.invalidate(profileViewModelProvider);
      
      state = state.copyWith(isLoading: false);
    } catch (e) {
      state = state.copyWith(isLoading: false, error: e.toString());
      rethrow;
    }
  }
}

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
```

---

## 7. Edit Profile Page

```dart
// lib/features/profile/presentation/edit_profile_page.dart
class EditProfilePage extends ConsumerStatefulWidget {
  final ProfileModel initialProfile;
  
  const EditProfilePage({super.key, required this.initialProfile});

  @override
  ConsumerState<EditProfilePage> createState() => _EditProfilePageState();
}

class _EditProfilePageState extends ConsumerState<EditProfilePage> {
  late final TextEditingController _nameController;
  late final TextEditingController _phoneController;
  late final TextEditingController _bioController;
  
  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController(text: widget.initialProfile.name);
    _phoneController = TextEditingController(text: widget.initialProfile.phone ?? '');
    _bioController = TextEditingController(text: widget.initialProfile.bio ?? '');
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    _bioController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final state = ref.watch(editProfileViewModelProvider(widget.initialProfile));
    final viewModel = ref.read(editProfileViewModelProvider(widget.initialProfile).notifier);
    
    return PopScope(
      canPop: !state.hasChanges,
      onPopInvokedWithResult: (didPop, result) async {
        if (didPop) return;
        
        final discard = await _showDiscardDialog(context);
        if (discard && context.mounted) {
          Navigator.of(context).pop();
        }
      },
      child: Scaffold(
        appBar: AppBar(
          title: Text(l10n.editProfile),
          actions: [
            TextButton(
              onPressed: state.isLoading ? null : () => _save(context, viewModel),
              child: state.isLoading
                  ? const SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    )
                  : Text(l10n.save),
            ),
          ],
        ),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              AvatarEditWidget(
                currentAvatarUrl: widget.initialProfile.avatarUrl,
                onAvatarChanged: viewModel.updateAvatar,
              ),
              const SizedBox(height: 24),
              PrimaryTextField(
                controller: _nameController,
                label: l10n.name,
                onChanged: viewModel.updateName,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return l10n.nameRequired;
                  }
                  if (value.length < 2) {
                    return l10n.nameTooShort;
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              PrimaryTextField(
                controller: _phoneController,
                label: l10n.phone,
                keyboardType: TextInputType.phone,
                onChanged: viewModel.updatePhone,
              ),
              const SizedBox(height: 16),
              DatePickerField(
                label: l10n.dateOfBirth,
                value: state.dateOfBirth,
                onChanged: viewModel.updateDateOfBirth,
              ),
              const SizedBox(height: 16),
              PrimaryTextField(
                controller: _bioController,
                label: l10n.bio,
                maxLines: 3,
                maxLength: 200,
                onChanged: viewModel.updateBio,
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  Future<void> _save(BuildContext context, EditProfileViewModel vm) async {
    if (!Form.of(context).validate()) return;
    
    try {
      await vm.save();
      if (context.mounted) {
        AppNavigator.pop();
        AppSnackbar.showSuccess(context, l10n.profileUpdated);
      }
    } catch (e) {
      if (context.mounted) {
        AppSnackbar.showError(context, e.toString());
      }
    }
  }
  
  Future<bool> _showDiscardDialog(BuildContext context) async {
    return await showConfirmDialog(
      context: context,
      title: l10n.discardChanges,
      message: l10n.discardChangesMessage,
      confirmText: l10n.discard,
      isDestructive: true,
    ) ?? false;
  }
}
```

---

## 8. CI/CD Pipeline

> ⚠️ **Implementation note:** The actual `base_flutter` project uses **Bitbucket Pipelines + Codemagic** for CI/CD, not GitHub Actions. The GitHub Actions example below is shown as a template pattern — see M19 for the actual CI/CD configuration used in this project.

```yaml
# .github/workflows/profile-feature.yml
name: Profile Feature CI

on:
  push:
    branches: [feature/user-profile]
  pull_request:
    branches: [develop, main]

jobs:
  lint:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          channel: 'stable'
      
      - run: flutter pub get
      - run: flutter analyze
      - run: dart format --check

  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - run: flutter pub get
      - run: flutter test --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  build:
    runs-on: ubuntu-24.04
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - run: flutter pub get
      - run: flutter build apk --release
      - uses: actions/upload-artifact@v4
        with:
          name: apk
          path: build/app/outputs/flutter-apk/app-release.apk
```

---

## Summary — Implementation Checklist

| Component | File | Status |
|-----------|------|--------|
| Model | `profile_model.dart` | ✅ freezed |
| API | `profile_api_service.dart` | ✅ Dio |
| Repository | `profile_repository.dart` | ✅ Cache |
| ViewModel | `profile_view_model.dart` | ✅ Riverpod |
| Page | `profile_page.dart` | ✅ UI |
| Edit VM | `edit_profile_view_model.dart` | ✅ Form state |
| Edit Page | `edit_profile_page.dart` | ✅ Form + validation |
| CI/CD | `profile-feature.yml` | ✅ GitHub Actions |

<!-- AI_VERIFY: generation-complete -->
