# Buổi 16: CI/CD & Production — Ví Dụ

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

## VD1: Build Commands 🟡

> **Liên quan tới:** [1. Build Modes 🟡](01-ly-thuyet.md#1-build-modes)

### 1.1. Build APK (Android)

```bash
# Debug APK (mặc định)
flutter build apk --debug
# Output: build/app/outputs/flutter-apk/app-debug.apk
# Size: ~80-100 MB

# Release APK
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
# Size: ~15-25 MB

# Release APK với obfuscation
flutter build apk \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info
# Size: ~12-20 MB

# Split APK theo ABI (giảm size cho từng kiến trúc CPU)
flutter build apk --release --split-per-abi
# Output:
#   app-armeabi-v7a-release.apk  (~10 MB, ARM 32-bit)
#   app-arm64-v8a-release.apk    (~12 MB, ARM 64-bit)
#   app-x86_64-release.apk       (~12 MB, x86 64-bit)
```

### 1.2. Build App Bundle (Android — Recommended)

```bash
# App Bundle cho Google Play (recommended)
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab

# Với obfuscation
flutter build appbundle \
  --release \
  --obfuscate \
  --split-debug-info=build/debug-info

# Với dart-define cho environment config
flutter build appbundle \
  --release \
  --dart-define=API_URL=https://api.example.com \
  --dart-define=ENV=production
```

### 1.3. Build iOS

```bash
# Build iOS (chỉ chạy trên macOS)
flutter build ios --release
# Output: build/ios/iphoneos/Runner.app

# Build IPA cho distribution
flutter build ipa --release
# Output: build/ios/ipa/Runner.ipa

# Với export options
flutter build ipa \
  --release \
  --export-options-plist=ios/ExportOptions.plist \
  --obfuscate \
  --split-debug-info=build/debug-info
```

### 1.4. Kiểm tra app size

```bash
# Phân tích app size chi tiết
flutter build apk --analyze-size
# Tạo report tại: build/apk-code-size-analysis_*.json

flutter build appbundle --analyze-size
flutter build ios --analyze-size

# Mở DevTools để xem visualization
flutter pub global activate devtools
dart devtools --appSizeBase=build/apk-code-size-analysis_*.json
```

### 1.5. Sử dụng dart-define trong code

```dart
// Đọc giá trị từ --dart-define
class EnvConfig {
  static const apiUrl = String.fromEnvironment(
    'API_URL',
    defaultValue: 'https://api-dev.example.com',
  );
  
  static const env = String.fromEnvironment(
    'ENV',
    defaultValue: 'dev',
  );
  
  static const bool isProduction = env == 'production';
}

// Sử dụng
void main() {
  print('API URL: ${EnvConfig.apiUrl}');
  print('Environment: ${EnvConfig.env}');
}
```

- 🔗 **FE tương đương:** `flutter build apk` / `flutter build ipa` ≈ `npm run build` — nhưng output = platform-specific binary (không phải static files), cần signing identity riêng.

---

## VD2: GitHub Actions Workflow 🟢

> **Liên quan tới:** [3. GitHub Actions cho Flutter 🟢](01-ly-thuyet.md#3-github-actions-cho-flutter)

### Complete CI Workflow

```yaml
# .github/workflows/flutter-ci.yml
name: Flutter CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Hủy workflow cũ nếu có push mới trên cùng branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ─── Job 1: Analyze & Test ───
  analyze-and-test:
    name: Analyze & Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
          cache: true
      
      - name: Cache pub dependencies
        uses: actions/cache@v4
        with:
          path: |
            ~/.pub-cache
            .dart_tool
          key: ${{ runner.os }}-pub-${{ hashFiles('**/pubspec.lock') }}
          restore-keys: |
            ${{ runner.os }}-pub-
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Verify formatting
        run: dart format --output=none --set-exit-if-changed .
      
      - name: Analyze code
        run: flutter analyze --fatal-infos
      
      - name: Run tests
        run: flutter test --coverage
      
      - name: Upload coverage
        if: success()
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/lcov.info

  # ─── Job 2: Build Android ───
  build-android:
    name: Build Android
    needs: analyze-and-test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
          cache: true
      
      - name: Install dependencies
        run: flutter pub get
      
      # Decode signing key từ secrets
      - name: Setup Android signing
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        env:
          KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          KEY_PROPERTIES: ${{ secrets.KEY_PROPERTIES }}
        run: |
          echo "$KEYSTORE_BASE64" | base64 --decode > android/app/keystore.jks
          echo "$KEY_PROPERTIES" > android/key.properties
      
      - name: Build APK
        run: |
          flutter build apk --release \
            --obfuscate \
            --split-debug-info=build/debug-info
      
      - name: Build App Bundle
        run: |
          flutter build appbundle --release \
            --obfuscate \
            --split-debug-info=build/debug-info
      
      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: android-apk
          path: build/app/outputs/flutter-apk/app-release.apk
          retention-days: 14
      
      - name: Upload AAB artifact
        uses: actions/upload-artifact@v4
        with:
          name: android-aab
          path: build/app/outputs/bundle/release/app-release.aab
          retention-days: 14
      
      - name: Upload debug info
        uses: actions/upload-artifact@v4
        with:
          name: debug-info-android
          path: build/debug-info/
          retention-days: 30

  # ─── Job 3: Build iOS ───
  build-ios:
    name: Build iOS
    needs: analyze-and-test
    runs-on: macos-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
          cache: true
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Build iOS (no codesign for CI)
        run: flutter build ios --release --no-codesign
      
      # Chỉ build IPA khi push to main (có signing)
      # - name: Build IPA
      #   if: github.ref == 'refs/heads/main'
      #   run: flutter build ipa --release
```

### Giải thích flow:

```
Push / PR
    │
    ▼
┌──────────────────┐
│ analyze-and-test │  ← Chạy trước, fail fast
│  • format check  │
│  • analyze       │
│  • test          │
└───────┬──────────┘
        │ (pass)
   ┌────┴────┐
   ▼         ▼
┌────────┐ ┌────────┐
│ Build  │ │ Build  │  ← Chạy song song
│Android │ │  iOS   │
│  • APK │ │  • IPA │
│  • AAB │ │        │
└────────┘ └────────┘
```

- 🔗 **FE tương đương:** GitHub Actions workflow ≈ FE CI/CD — nhưng mobile cần thêm macOS runner cho iOS, signing certificate setup, và artifact upload to stores.

---

## VD3: Fastlane Setup 🟢

> **Liên quan tới:** [4. Fastlane 🟢](01-ly-thuyet.md#4-fastlane)

### 3.1. iOS Fastfile — TestFlight Deployment

```ruby
# ios/fastlane/Fastfile
default_platform(:ios)

platform :ios do
  # ─── Shared setup ───
  before_all do
    setup_ci if ENV['CI']
  end

  # ─── Beta: Upload to TestFlight ───
  desc "Push a new beta build to TestFlight"
  lane :beta do
    # Sync certificates (dùng Match)
    match(
      type: "appstore",
      readonly: is_ci,  # CI chỉ đọc, không tạo mới
      git_url: ENV['MATCH_GIT_URL']
    )

    # Tăng build number
    increment_build_number(
      xcodeproj: "Runner.xcodeproj",
      build_number: ENV['BUILD_NUMBER'] || latest_testflight_build_number + 1
    )

    # Build Flutter app trước
    sh "cd ../.. && flutter build ios --release --no-codesign"

    # Build và sign IPA
    build_ios_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store",
      output_directory: "./build",
      output_name: "Runner.ipa",
      export_options: {
        provisioningProfiles: {
          "com.company.appname" => "match AppStore com.company.appname"
        }
      }
    )

    # Upload lên TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true,
      apple_id: ENV['APPLE_APP_ID']
    )

    # Gửi thông báo (optional)
    # slack(
    #   message: "New iOS beta uploaded to TestFlight! 🚀",
    #   slack_url: ENV['SLACK_WEBHOOK_URL']
    # )
  end

  # ─── Release: Submit to App Store ───
  desc "Submit to App Store for review"
  lane :release do
    match(type: "appstore", readonly: true)

    build_ios_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )

    upload_to_app_store(
      force: true,
      submit_for_review: true,
      automatic_release: false,  # Manual release sau khi review pass
      skip_screenshots: true,
      precheck_include_in_app_purchases: false,
      submission_information: {
        add_id_info_uses_idfa: false
      }
    )
  end

  # ─── Error handling ───
  error do |lane, exception|
    # slack(
    #   message: "iOS #{lane} failed: #{exception.message}",
    #   success: false,
    #   slack_url: ENV['SLACK_WEBHOOK_URL']
    # )
  end
end
```

### 3.2. Appfile (iOS)

```ruby
# ios/fastlane/Appfile
app_identifier("com.company.appname")
apple_id(ENV['APPLE_ID'])           # Apple Developer email
itc_team_id(ENV['ITC_TEAM_ID'])     # App Store Connect Team ID
team_id(ENV['TEAM_ID'])             # Developer Portal Team ID
```

### 3.3. Android Fastfile

```ruby
# android/fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Deploy to Google Play Internal Testing"
  lane :beta do
    # Build Flutter app bundle
    sh "cd ../.. && flutter build appbundle --release " \
       "--obfuscate --split-debug-info=build/debug-info"

    # Upload to Play Store (internal track)
    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab',
      json_key: ENV['PLAY_STORE_JSON_KEY_PATH'] || 'play-store-key.json',
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
      release_status: 'draft'  # hoặc 'completed' để auto-publish
    )
  end

  desc "Promote from internal to production"
  lane :release do
    upload_to_play_store(
      track: 'internal',
      track_promote_to: 'production',
      skip_upload_aab: true,
      skip_upload_metadata: false,
      json_key: ENV['PLAY_STORE_JSON_KEY_PATH'] || 'play-store-key.json'
    )
  end

  desc "Build and distribute via Firebase App Distribution"
  lane :distribute do
    sh "cd ../.. && flutter build apk --release"

    firebase_app_distribution(
      app: ENV['FIREBASE_APP_ID'],
      apk_path: '../build/app/outputs/flutter-apk/app-release.apk',
      groups: "internal-testers",
      release_notes: "New build from CI"
    )
  end
end
```

---

## VD4: Android Signing Config 🟡

> **Liên quan tới:** [2. Code Signing 🟡](01-ly-thuyet.md#2-code-signing)

### 4.1. Tạo Keystore

```bash
# Tạo keystore mới
keytool -genkey -v \
  -keystore ~/my-app-release-key.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias my-app-key

# Sẽ hỏi:
# - Keystore password
# - Key password
# - First and Last name
# - Organization
# - City, State, Country

# Kiểm tra keystore
keytool -list -v -keystore ~/my-app-release-key.jks
```

### 4.2. File key.properties

```properties
# android/key.properties
# ⚠️ PHẢI thêm vào .gitignore!

storePassword=your_secure_store_password
keyPassword=your_secure_key_password
keyAlias=my-app-key
storeFile=keystore.jks
```

### 4.3. Cấu hình build.gradle

```groovy
// android/app/build.gradle

plugins {
    id "com.android.application"
    id "kotlin-android"
    id "dev.flutter.flutter-gradle-plugin"
}

// ─── Đọc signing config ───
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(
        new FileInputStream(keystorePropertiesFile)
    )
}

android {
    namespace "com.company.appname"
    compileSdk = flutter.compileSdkVersion

    defaultConfig {
        applicationId "com.company.appname"
        minSdk = flutter.minSdkVersion
        targetSdk = flutter.targetSdkVersion
        versionCode = flutter.versionCode
        versionName = flutter.versionName
    }

    // ─── Signing configs ───
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile']
                ? file(keystoreProperties['storeFile'])
                : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            signingConfig signingConfigs.release
            
            // Code shrinking & obfuscation
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile(
                'proguard-android-optimize.txt'
            ), 'proguard-rules.pro'
        }
    }
}

flutter {
    source '../..'
}

dependencies {}
```

### 4.4. .gitignore cho signing files

```gitignore
# android/.gitignore (thêm vào cuối)

# Signing
key.properties
*.jks
*.keystore
play-store-key.json

# Fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots
fastlane/test_output
```

### 4.5. Encode keystore cho CI secrets

```bash
# macOS/Linux: Encode keystore thành base64
base64 -i my-app-release-key.jks -o keystore_base64.txt

# Hoặc copy trực tiếp vào clipboard (macOS)
base64 -i my-app-release-key.jks | pbcopy

# Sau đó paste vào GitHub Secrets:
# Repo → Settings → Secrets → New repository secret
# Name: ANDROID_KEYSTORE_BASE64
# Value: (paste base64 string)

# Tương tự cho key.properties:
# Name: KEY_PROPERTIES
# Value: (nội dung file key.properties)
```

---

## VD5: Release Checklist Script 🟡

> **Liên quan tới:** [Pre-Release Checklist](01-ly-thuyet.md#pre-release-checklist)

### 5.1. Script tự động kiểm tra trước release

```bash
#!/bin/bash
# scripts/pre-release-check.sh
# Chạy: chmod +x scripts/pre-release-check.sh && ./scripts/pre-release-check.sh

set -e  # Dừng nếu có lỗi

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

PASS="${GREEN}✅ PASS${NC}"
FAIL="${RED}❌ FAIL${NC}"
WARN="${YELLOW}⚠️  WARN${NC}"

echo "╔══════════════════════════════════════╗"
echo "║   Flutter Pre-Release Check Script   ║"
echo "╚══════════════════════════════════════╝"
echo ""

ERRORS=0

# ─── 1. Check Flutter version ───
echo "📋 Step 1: Flutter version"
flutter --version
echo ""

# ─── 2. Clean build ───
echo "🧹 Step 2: Clean build artifacts"
flutter clean
flutter pub get
echo -e "$PASS Clean & pub get"
echo ""

# ─── 3. Format check ───
echo "📝 Step 3: Code formatting"
if dart format --output=none --set-exit-if-changed .; then
    echo -e "$PASS Code formatting OK"
else
    echo -e "$FAIL Code formatting issues found"
    echo "   Run: dart format ."
    ERRORS=$((ERRORS + 1))
fi
echo ""

# ─── 4. Analyze ───
echo "🔍 Step 4: Static analysis"
if flutter analyze --no-fatal-infos; then
    echo -e "$PASS No analysis issues"
else
    echo -e "$FAIL Analysis issues found"
    ERRORS=$((ERRORS + 1))
fi
echo ""

# ─── 5. Tests ───
echo "🧪 Step 5: Running tests"
if flutter test; then
    echo -e "$PASS All tests passed"
else
    echo -e "$FAIL Some tests failed"
    ERRORS=$((ERRORS + 1))
fi
echo ""

# ─── 6. Build Android ───
echo "🤖 Step 6: Build Android APK"
if flutter build apk --release --obfuscate --split-debug-info=build/debug-info; then
    APK_SIZE=$(du -h build/app/outputs/flutter-apk/app-release.apk | cut -f1)
    echo -e "$PASS Android APK built (Size: $APK_SIZE)"
else
    echo -e "$FAIL Android build failed"
    ERRORS=$((ERRORS + 1))
fi
echo ""

# ─── 7. Build App Bundle ───
echo "📦 Step 7: Build Android App Bundle"
if flutter build appbundle --release --obfuscate --split-debug-info=build/debug-info; then
    AAB_SIZE=$(du -h build/app/outputs/bundle/release/app-release.aab | cut -f1)
    echo -e "$PASS Android AAB built (Size: $AAB_SIZE)"
else
    echo -e "$FAIL App Bundle build failed"
    ERRORS=$((ERRORS + 1))
fi
echo ""

# ─── 8. Build iOS (macOS only) ───
if [[ "$OSTYPE" == "darwin"* ]]; then
    echo "🍎 Step 8: Build iOS"
    if flutter build ios --release --no-codesign; then
        echo -e "$PASS iOS build successful"
    else
        echo -e "$FAIL iOS build failed"
        ERRORS=$((ERRORS + 1))
    fi
else
    echo "🍎 Step 8: Build iOS"
    echo -e "$WARN Skipped (not macOS)"
fi
echo ""

# ─── 9. Check pubspec version ───
echo "🏷️  Step 9: Version check"
VERSION=$(grep '^version:' pubspec.yaml | head -1 | awk '{print $2}')
echo "   Current version: $VERSION"
echo -e "$PASS Version found in pubspec.yaml"
echo ""

# ─── 10. Check for TODOs ───
echo "📌 Step 10: TODO/FIXME check"
TODO_COUNT=$(grep -rn "TODO\|FIXME\|HACK\|XXX" lib/ --include="*.dart" | wc -l | tr -d ' ')
if [ "$TODO_COUNT" -gt 0 ]; then
    echo -e "$WARN Found $TODO_COUNT TODO/FIXME comments in lib/"
    grep -rn "TODO\|FIXME" lib/ --include="*.dart" | head -10
    if [ "$TODO_COUNT" -gt 10 ]; then
        echo "   ... and $((TODO_COUNT - 10)) more"
    fi
else
    echo -e "$PASS No TODO/FIXME found"
fi
echo ""

# ─── Summary ───
echo "╔══════════════════════════════════════╗"
echo "║            SUMMARY                   ║"
echo "╚══════════════════════════════════════╝"

if [ $ERRORS -eq 0 ]; then
    echo -e "${GREEN}"
    echo "  ✅ All checks passed!"
    echo "  Ready for release! 🚀"
    echo -e "${NC}"
    exit 0
else
    echo -e "${RED}"
    echo "  ❌ $ERRORS check(s) failed!"
    echo "  Fix the issues above before releasing."
    echo -e "${NC}"
    exit 1
fi
```

### 5.2. Makefile cho common commands

```makefile
# Makefile
.PHONY: analyze test build-android build-ios release-check clean

# ─── Development ───
run:
	flutter run

run-release:
	flutter run --release

# ─── Quality ───
analyze:
	flutter analyze --fatal-infos

format:
	dart format .

format-check:
	dart format --output=none --set-exit-if-changed .

test:
	flutter test

test-coverage:
	flutter test --coverage
	genhtml coverage/lcov.info -o coverage/html
	open coverage/html/index.html

# ─── Build ───
clean:
	flutter clean && flutter pub get

build-android:
	flutter build apk --release --obfuscate --split-debug-info=build/debug-info

build-aab:
	flutter build appbundle --release --obfuscate --split-debug-info=build/debug-info

build-ios:
	flutter build ios --release --no-codesign

build-ipa:
	flutter build ipa --release --obfuscate --split-debug-info=build/debug-info

# ─── Release ───
release-check:
	./scripts/pre-release-check.sh

# ─── CI simulation ───
ci: format-check analyze test build-android
	@echo "✅ CI simulation passed!"
```

**Sử dụng:**

```bash
# Chạy CI locally
make ci

# Chỉ test
make test

# Build Android
make build-aab

# Pre-release check
make release-check
```

---

## VD6: 🤖 AI Gen → Review — GitHub Actions CI Pipeline 🟢

> **Mục đích:** Luyện workflow "AI gen CI/CD config → bạn review secrets + build flags + cache → fix"

> **Liên quan tới:** [3. GitHub Actions cho Flutter 🟢](01-ly-thuyet.md#3-github-actions-cho-flutter)

### Bước 1: Prompt cho AI

```text
Tạo GitHub Actions CI workflow cho Flutter.
Trigger: on PR to main.
Steps: checkout → Flutter setup → pub get → analyze → test → build APK.
Cache: pub dependencies.
Output: .github/workflows/ci.yml.
```

### Bước 2: Review output AI theo checklist

| # | Điểm review | Tìm gì |
|---|---|---|
| 1 | **Flutter version** | Pinned version (3.x)? Không dùng 'any' hoặc 'latest'? |
| 2 | **Steps order** | analyze (lint) TRƯỚC test TRƯỚC build? |
| 3 | **Secrets** | Có secret nào hardcoded trong YAML? |
| 4 | **Cache** | pub cache configured? Gradle cache? |

### Bước 3: Các lỗi AI thường mắc (tự verify)

```yaml
# ❌ LỖI 1: Hardcode secret trong YAML
env:
  API_KEY: "sk-abc123def456"  # EXPOSED — ai cũng thấy!

# ✅ FIX: Dùng GitHub Secrets
env:
  API_KEY: ${{ secrets.API_KEY }}  # Encrypted, không visible
```

```yaml
# ❌ LỖI 2: Thiếu analyze step
steps:
  - run: flutter test        # Test chạy, nhưng lint errors ko bị catch!
  - run: flutter build apk

# ✅ FIX: Analyze trước test
steps:
  - run: flutter analyze --fatal-infos  # Catch lint issues FIRST
  - run: flutter test --coverage
  - run: flutter build apk
```

### Kết quả mong đợi

Sau bài tập này, bạn sẽ:
- ✅ Biết KHÔNG BAO GIỜ hardcode secrets trong CI config
- ✅ Hiểu CI steps order quan trọng (analyze → test → build)
- ✅ Verify cache setup để CI chạy nhanh hơn
