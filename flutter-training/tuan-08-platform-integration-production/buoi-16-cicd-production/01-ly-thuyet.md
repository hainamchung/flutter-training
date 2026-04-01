# Buổi 16: CI/CD & Production — Lý Thuyết

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

> **Buổi 16/16** · **Thời lượng tự học:** ~2 giờ · **Cập nhật:** 2026-03-31  
> **Yêu cầu trước khi học:** Hoàn thành Buổi 15 (lý thuyết + ít nhất BT1-BT2)

## 1. Build Modes 🟡

### 1.1. Ba chế độ build trong Flutter

Flutter có **3 build modes**, mỗi mode phục vụ mục đích khác nhau trong quá trình phát triển:

| Mode | Mục đích | Compilation | Performance | Debugging |
|------|----------|-------------|-------------|-----------|
| **Debug** | Phát triển | JIT (Just-In-Time) | Chậm | Đầy đủ |
| **Profile** | Profiling | AOT | Gần release | Một phần |
| **Release** | Production | AOT (Ahead-Of-Time) | Tối ưu nhất | Không có |

### 1.2. Debug Mode

```bash
# Mặc định khi chạy
flutter run
flutter run --debug
```

**Đặc điểm:**
- **Hot Reload** hoạt động — thay đổi code thấy ngay
- **Assertions** được bật — `assert()` sẽ throw nếu fail
- **DevTools** có thể kết nối — inspect widget tree, memory, network
- **JIT compilation** — compile khi cần, nên khởi động chậm hơn
- **App size lớn** — chứa debug symbols, source maps
- Hiển thị **debug banner** ở góc phải

> ⚠️ **Không bao giờ** đo performance ở Debug mode. JIT compilation làm kết quả sai lệch hoàn toàn.

### 1.3. Profile Mode

```bash
flutter run --profile
```

**Đặc điểm:**
- **AOT compilation** — performance gần giống Release
- Một số debug tools vẫn hoạt động (DevTools, Observatory)
- **Assertions bị tắt** — giống Release
- Chỉ dùng trên **thiết bị thật** (không hỗ trợ emulator/simulator)

**Khi nào dùng:** Kiểm tra performance, tìm jank, profiling render time.

### 1.4. Release Mode

```bash
flutter run --release
flutter build apk --release
flutter build ios --release
```

**Đặc điểm:**
- **AOT compilation** — code Dart biên dịch hoàn toàn sang native
- **Tree shaking** — loại bỏ code không sử dụng, giảm app size
- **Tối ưu tối đa** — no debug info, no assertions, no DevTools
- App size nhỏ nhất có thể

### 1.5. Tree Shaking và AOT

```
Debug Mode:                    Release Mode:
┌─────────────────┐           ┌─────────────────┐
│ Toàn bộ code    │           │ Code được dùng  │
│ + Debug symbols │  ──AOT──▶ │ + Native binary │
│ + Source maps   │  +shake   │ (tối ưu)        │
│ ~80-150 MB      │           │ ~15-30 MB       │
└─────────────────┘           └─────────────────┘
```

**Tree shaking:** Compiler phân tích toàn bộ code, loại bỏ functions/classes/imports không được gọi tới. Giống webpack tree shaking ở React/Vue nhưng ở mức Dart VM.

**AOT compilation:** Dart code → native ARM/x86 code. Khác với web (JS bundle), mobile app cần native binary.

### 1.6. Obfuscation

Bảo vệ source code khỏi reverse engineering:

```bash
flutter build apk --obfuscate --split-debug-info=build/debug-info

flutter build appbundle --obfuscate --split-debug-info=build/debug-info

flutter build ios --obfuscate --split-debug-info=build/debug-info
```

- `--obfuscate`: Đổi tên classes, methods thành ký tự vô nghĩa
- `--split-debug-info`: Tách debug symbols ra folder riêng (cần giữ để đọc crash reports)

### 1.7. So sánh app size (ước tính)

| Mode | APK Size | IPA Size |
|------|----------|----------|
| Debug | ~80-150 MB | ~200+ MB |
| Release | ~15-25 MB | ~30-50 MB |
| Release + obfuscate | ~12-20 MB | ~25-45 MB |

### 1.8. Build Flavors (Gradle/Xcode)

`--dart-define` phù hợp cho config đơn giản. Cho production apps cần:

#### Android: `build.gradle` flavors
```groovy
// android/app/build.gradle
android {
    flavorDimensions "environment"
    
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            resValue "string", "app_name", "MyApp Dev"
        }
        staging {
            dimension "environment"  
            applicationIdSuffix ".staging"
            resValue "string", "app_name", "MyApp Staging"
        }
        production {
            dimension "environment"
            resValue "string", "app_name", "MyApp"
        }
    }
}
```

#### iOS: Xcode Schemes

1. Tạo 3 schemes: `dev`, `staging`, `production`
2. Mỗi scheme → Edit Scheme → Build Configuration
3. Dùng `xcconfig` files cho mỗi environment

#### Chạy với flavor

```bash
# Development
flutter run --flavor dev -t lib/main_dev.dart

# Staging  
flutter run --flavor staging -t lib/main_staging.dart

# Production
flutter run --flavor production -t lib/main.dart
```

> 💡 **Package gợi ý**: [`flutter_flavorizr`](https://pub.dev/packages/flutter_flavorizr) tự động generate cả Android và iOS config.

---

## 2. Code Signing 🟡

### 2.1. Tại sao cần Code Signing?

```
Developer ──sign──▶ App Binary ──verify──▶ App Store / Play Store ──▶ User
    │                                            │
    └── "Tôi là developer thật"                  └── "App này an toàn, từ developer đã xác minh"
```

**Lý do:**
- **Xác minh danh tính** — Store biết ai là người tạo app
- **Bảo đảm toàn vẹn** — App không bị sửa đổi sau khi sign
- **Bắt buộc** — Cả App Store lẫn Play Store đều yêu cầu
- **Cập nhật** — Chỉ app signed cùng key mới update được app cũ

> 💡 **Góc nhìn từ Frontend:** Web không có khái niệm code signing. Deploy lên Vercel/Netlify chỉ cần push code. Mobile phức tạp hơn nhiều vì app chạy trực tiếp trên thiết bị người dùng.

### 2.2. Android Code Signing

#### Bước 1: Tạo Keystore

```bash
keytool -genkey -v \
  -keystore ~/my-release-key.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias my-key-alias
```

> ⚠️ **QUAN TRỌNG:** Giữ keystore file an toàn! Mất keystore = không thể update app trên Play Store. Backup ở nơi an toàn, **KHÔNG** commit vào git.

#### Bước 2: Tạo file key.properties

```properties
# android/key.properties
# ⚠️ KHÔNG commit file này vào git!
storePassword=your_store_password
keyPassword=your_key_password
keyAlias=my-key-alias
storeFile=/path/to/my-release-key.jks
```

Thêm vào `.gitignore`:
```
# Android signing
android/key.properties
*.jks
*.keystore
```

#### Bước 3: Cấu hình build.gradle

```groovy
// android/app/build.gradle

// Đọc key.properties
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(
        new FileInputStream(keystorePropertiesFile)
    )
}

android {
    // ...

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
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile(
                'proguard-android-optimize.txt'
            ), 'proguard-rules.pro'
        }
    }
}
```

### 2.3. iOS Code Signing

#### Yêu cầu

- **Apple Developer Program** ($99/năm) — bắt buộc để phát hành
- **Xcode** trên macOS — build iOS chỉ chạy trên Mac
- **Provisioning Profile** — liên kết app ID + certificates + devices

#### Quy trình signing

```
Apple Developer Account
    │
    ├── Certificate (ai được sign)
    │     ├── Development Certificate
    │     └── Distribution Certificate
    │
    ├── App ID (app nào)
    │     └── com.company.appname
    │
    └── Provisioning Profile (gộp tất cả)
          ├── Development (test trên device)
          ├── Ad Hoc (test nội bộ, giới hạn devices)
          └── Distribution (App Store)
```

#### Automatic vs Manual Signing

| | Automatic | Manual |
|---|-----------|--------|
| **Setup** | Xcode tự quản lý | Tự tạo trên Apple Developer Portal |
| **CI/CD** | Khó dùng trên CI | Phù hợp CI/CD |
| **Khi nào** | Development | Production, CI/CD |
| **Config** | Tick "Automatically manage signing" | Chọn profile cụ thể |

**Trong Xcode:**
1. Mở `ios/Runner.xcworkspace`
2. Chọn Runner target → Signing & Capabilities
3. Chọn Team (Apple Developer Account)
4. Automatic signing: Xcode tạo profiles tự động
5. Manual signing: Chọn provisioning profile đã tạo

#### App Store Connect

Là portal để quản lý app trên App Store:
- Upload builds (qua Xcode hoặc Transporter)
- Quản lý TestFlight
- Submit for review
- Quản lý metadata (screenshots, description, pricing)
- Xem analytics và crash reports

---

> 💼 **Gặp trong dự án:** Setup CI/CD pipeline hoàn chỉnh (GitHub Actions + Fastlane), automate build/test/deploy, code signing cho iOS (certificates, provisioning profiles), environment-specific builds (dev/staging/prod)
> 🤖 **Keywords bắt buộc trong prompt:** `GitHub Actions workflow`, `flutter analyze + test + build`, `code signing`, `Fastlane match`, `environment variables`, `secrets management`, `artifact upload`, `deployment lanes`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **New project:** Team cần setup CI/CD từ scratch — mỗi PR chạy analyze + test, mỗi merge to main build + deploy
- **Code signing pain:** iOS certificates expire, provisioning profiles mismatch, team members can't build
- **Multi-environment:** Dev (mock API), Staging (test API), Prod (live API) — mỗi environment cần separate build

**Tại sao cần các keyword trên:**
- **`GitHub Actions workflow`** — YAML config, AI cần biết Flutter-specific actions (subosito/flutter-action)
- **`flutter analyze + test + build`** — CI steps order matters (analyze → test → build), AI hay thiếu analyze
- **`Fastlane match`** — manage iOS code signing (certificates + profiles) — AI hay manual setup (không scale)
- **`secrets management`** — KHÔNG hardcode API keys, signing passwords in code/workflow
- **`artifact upload`** — upload APK/IPA as build artifact cho testing

**Prompt mẫu — GitHub Actions CI/CD:**
```text
Tôi cần setup GitHub Actions CI/CD cho Flutter project.
Tech stack: Flutter 3.x, GitHub Actions, Fastlane.
Requirements:
1. CI workflow (trên mỗi PR):
   - Run flutter analyze (strict — treat warnings as errors).
   - Run flutter test --coverage.
   - Check coverage >= 80% (fail PR nếu < 80%).
   - Build APK (verify build passes).
2. CD workflow (trên merge to main):
   - Build release APK + App Bundle (Android).
   - Build release IPA (iOS) — code signing với Fastlane match.
   - Upload artifacts (APK/IPA) to GitHub Releases.
   - Deploy: Android → Google Play Internal Testing, iOS → TestFlight (via Fastlane).
3. Environment configs: dev/staging/prod — different API URLs, different app IDs.
Constraints:
- Secrets: KEYSTORE_PASSWORD, MATCH_PASSWORD, PLAY_STORE_KEY, APP_STORE_CONNECT_KEY — stored in GitHub Secrets.
- Matrix build: test trên Flutter stable + beta.
- Cache: pub cache + Gradle cache + CocoaPods cache.
- Notifications: Slack webhook khi build fail.
Output: .github/workflows/ci.yml + .github/workflows/cd.yml + Fastlane config files.
```

**Expected Output:** AI gen 2 workflow files + Fastlane configuration.

⚠️ **Giới hạn AI hay mắc:** AI hay hardcode secrets trong YAML (SECURITY RISK!). AI cũng hay thiếu cache configuration (build chậm gấp 3x). AI hay quên `flutter analyze` step (chỉ test + build). AI hay generate Fastlane config cho wrong platform.

</details>

> 🔗 **FE Bridge:** Build process ≈ `npm run build` — nhưng **khác ở**: mobile = **2 platforms** (Android APK/AAB + iOS IPA), mỗi platform cần signing certificate riêng. FE build = 1 bundle cho tất cả browsers. Mobile signing = identity verification (Apple/Google), FE deploy = just upload static files.

---

## 3. GitHub Actions cho Flutter 🟢

### 3.1. GitHub Actions là gì?

CI/CD platform tích hợp sẵn trong GitHub. Chạy workflows tự động khi có events (push, PR, release...).

```
Push code ──▶ GitHub Actions trigger ──▶ Workflow chạy ──▶ Kết quả
                                              │
                                    ┌─────────┼─────────┐
                                    ▼         ▼         ▼
                                 Analyze    Test     Build
                                    │         │         │
                                    └─────────┼─────────┘
                                              ▼
                                    ✅ Pass / ❌ Fail
```

> 💡 **Góc nhìn từ Frontend:** Nếu bạn đã dùng GitHub Actions cho React/Vue (lint, test, deploy), thì concepts hoàn toàn giống! Chỉ khác ở steps cụ thể (Flutter SDK thay vì Node.js).

### 3.2. Cấu trúc Workflow File

```yaml
# .github/workflows/flutter-ci.yml
name: Flutter CI

# Khi nào chạy
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Các jobs
jobs:
  build:
    runs-on: ubuntu-latest  # hoặc macos-latest cho iOS
    
    steps:
      # Step 1: Checkout code
      - uses: actions/checkout@v4
      
      # Step 2: Setup Flutter
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
          cache: true  # Cache Flutter SDK
      
      # Step 3: Cài dependencies
      - run: flutter pub get
      
      # Step 4: Analyze code
      - run: flutter analyze
      
      # Step 4b: Kiểm tra dart fix suggestions (fail nếu có code cần update)
      - run: dart fix --dry-run
      
      # Step 5: Chạy tests
      - run: flutter test
      
      # Step 6: Build
      - run: flutter build apk --release
```

### 3.3. Caching Dependencies

Tăng tốc CI bằng cách cache `.pub-cache`:

```yaml
steps:
  - uses: subosito/flutter-action@v2
    with:
      flutter-version: '3.24.0'
      cache: true  # Cache Flutter SDK tự động

  # Cache pub dependencies riêng (optional, chi tiết hơn)
  - name: Cache pub dependencies
    uses: actions/cache@v4
    with:
      path: |
        ~/.pub-cache
        .dart_tool
      key: ${{ runner.os }}-pub-${{ hashFiles('**/pubspec.lock') }}
      restore-keys: |
        ${{ runner.os }}-pub-
```

### 3.4. Matrix Strategy

Test trên nhiều Flutter versions cùng lúc:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        flutter-version: ['3.22.0', '3.24.0']
        # Có thể thêm: os: [ubuntu-latest, macos-latest]
    
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ matrix.flutter-version }}
      - run: flutter pub get
      - run: flutter test
```

### 3.5. Upload Artifacts

Lưu APK/IPA sau khi build để download:

```yaml
- name: Build APK
  run: flutter build apk --release

- name: Upload APK
  uses: actions/upload-artifact@v4
  with:
    name: release-apk
    path: build/app/outputs/flutter-apk/app-release.apk
    retention-days: 14
```

### 3.6. Secrets Management

Lưu trữ sensitive data (signing keys, API keys) an toàn:

```
GitHub Repo → Settings → Secrets and variables → Actions → New repository secret
```

```yaml
# Sử dụng secrets trong workflow
- name: Setup Android signing
  env:
    KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
    KEY_PROPERTIES: ${{ secrets.KEY_PROPERTIES }}
  run: |
    echo "$KEYSTORE_BASE64" | base64 --decode > android/app/keystore.jks
    echo "$KEY_PROPERTIES" > android/key.properties
```

**Encode keystore thành base64 để lưu vào secrets:**
```bash
base64 -i my-release-key.jks | pbcopy  # macOS: copy vào clipboard
```

> ⚠️ **KHÔNG BAO GIỜ** hardcode passwords, keys, hoặc certificates vào workflow file.

---

## 4. Fastlane 🟢

### 4.1. Fastlane là gì?

**Fastlane** là công cụ automation cho iOS và Android deployment. Thay vì làm thủ công hàng chục bước, Fastlane tự động hóa toàn bộ quy trình.

```
                        Fastlane
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         iOS Lane    Android Lane   Shared
              │            │            │
         build_ios    build_android  run_tests
         testflight   play_store    slack_notify
         app_store    firebase      versioning
```

> 💡 **Góc nhìn từ Frontend:** Fastlane giống Vercel/Netlify ở chỗ tự động deploy, nhưng khác hoàn toàn ở quy trình. Web: push → build → deploy (vài phút). Mobile: push → build → sign → upload → review (vài ngày cho iOS). App Store review process là điều web developer không quen.

### 4.2. Cài đặt Fastlane

```bash
# macOS (recommended)
brew install fastlane

# Hoặc qua gem
gem install fastlane

# Hoặc qua bundler (recommended cho CI)
# Gemfile
source "https://rubygems.org"
gem "fastlane"
```

### 4.3. iOS Lane — TestFlight Deployment

```ruby
# ios/fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    # Increment build number tự động
    increment_build_number(
      xcodeproj: "Runner.xcodeproj"
    )

    # Build iOS app
    build_ios_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store",
      output_directory: "./build",
      output_name: "Runner.ipa"
    )

    # Upload lên TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )

    # Thông báo qua Slack (optional)
    # slack(message: "New beta build uploaded to TestFlight!")
  end

  desc "Push a new release to App Store"
  lane :release do
    build_ios_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )

    upload_to_app_store(
      force: true,
      skip_metadata: false,
      skip_screenshots: true
    )
  end
end
```

### 4.4. Android Lane — Play Store Deployment

```ruby
# android/fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Deploy a new beta to Google Play Internal Testing"
  lane :beta do
    # Build app bundle
    sh "cd ../.. && flutter build appbundle --release"

    # Upload lên Play Store (internal track)
    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab',
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end

  desc "Promote internal to production"
  lane :release do
    upload_to_play_store(
      track: 'internal',
      track_promote_to: 'production',
      skip_upload_aab: true,
      skip_upload_metadata: true
    )
  end
end
```

### 4.5. Match — Quản lý Certificates (iOS)

**Match** là tool của Fastlane giúp quản lý iOS certificates và provisioning profiles qua Git repo:

```ruby
# ios/fastlane/Matchfile
git_url("https://github.com/your-org/certificates")
storage_mode("git")
type("appstore")  # development, adhoc, appstore
app_identifier("com.company.appname")
```

```ruby
# Trong Fastfile
lane :beta do
  match(type: "appstore")  # Tự động setup certificates
  build_ios_app(...)
  upload_to_testflight(...)
end
```

### 4.6. Tích hợp Fastlane với GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Stores

on:
  push:
    tags:
      - 'v*'  # Trigger khi tạo tag v1.0.0, v1.1.0...

jobs:
  deploy-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
          working-directory: android
      
      - run: flutter pub get
      
      # Decode secrets
      - name: Setup signing
        env:
          KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          PLAY_STORE_JSON: ${{ secrets.PLAY_STORE_JSON_KEY }}
        run: |
          echo "$KEYSTORE_BASE64" | base64 --decode > android/app/keystore.jks
          echo "$PLAY_STORE_JSON" > android/play-store-key.json
      
      - name: Deploy to Play Store
        working-directory: android
        run: bundle exec fastlane beta

  deploy-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
          working-directory: ios
      
      - run: flutter pub get
      
      - name: Setup signing
        env:
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
          MATCH_GIT_TOKEN: ${{ secrets.MATCH_GIT_TOKEN }}
        run: cd ios && bundle exec fastlane match appstore
      
      - name: Deploy to TestFlight
        working-directory: ios
        run: bundle exec fastlane beta
```

> 🔗 **FE Bridge:** CI/CD concept **tương đồng**: push → build → test → deploy. GitHub Actions ≈ **giống hệt** FE workflow. Nhưng **khác ở**: mobile CI cần **macOS runner** cho iOS build (expensive), platform signing setup complex hơn. `Fastlane` ≈ deployment script cho mobile — FE không cần equivalent vì deploy = copy files.

---

## 5. App Deployment 🟡

### 5.1. Google Play Store (Android)

#### Quy trình triển khai

```
Build APK/AAB ──▶ Internal Testing ──▶ Closed Testing ──▶ Open Testing ──▶ Production
                       │                      │                  │              │
                   Dev team            Invited users       Public beta     Everyone
                   (max 100)          (email list)        (opt-in)
```

**Các bước:**
1. **Tạo Google Play Developer Account** ($25 một lần)
2. **Tạo app trên Play Console** — thêm metadata, screenshots, descriptions
3. **Upload AAB** (App Bundle ưu tiên hơn APK — Google tự optimize cho từng device)
4. **Internal testing** — test nội bộ nhanh, không cần review
5. **Closed/Open testing** — mở rộng tester, thu thập feedback
6. **Production** — phát hành cho everyone

#### Lưu ý quan trọng:
- **App Bundle (.aab)** được khuyến nghị thay vì APK — Google Play tự tạo optimized APKs
- **Play App Signing** — Google quản lý signing key (recommended)
- **Review time:** Android thường review nhanh hơn iOS (vài giờ đến 1-2 ngày)

### 5.2. Apple App Store (iOS)

#### Quy trình triển khai

```
Build IPA ──▶ Upload (Xcode/Transporter) ──▶ TestFlight ──▶ App Store Review ──▶ Release
                                                  │                │                │
                                            Internal testers    1-3 ngày        Everyone
                                            External testers   (có thể reject)
```

**Các bước:**
1. **Apple Developer Program** ($99/năm)
2. **Tạo app trên App Store Connect** — metadata, screenshots, age rating
3. **Upload build** qua Xcode hoặc `xcrun altool`
4. **TestFlight** — internal (tự động) và external (cần review ngắn) testing
5. **Submit for Review** — Apple review team kiểm tra app (1-3 ngày)
6. **Release** — Manual release hoặc automatic after approval

#### Lý do phổ biến bị reject:
- App crashes hoặc bugs nghiêm trọng
- Thiếu privacy policy
- Sử dụng private APIs
- Metadata không chính xác
- Thiếu tính năng (app quá đơn giản)
- Vi phạm App Store Review Guidelines

### 5.3. Code Push / OTA Updates (Shorebird)

**Vấn đề:** Mỗi lần fix bug nhỏ → build lại → submit review → chờ 1-3 ngày (iOS).

**Shorebird** cho phép **over-the-air updates** cho Flutter apps — push code changes trực tiếp đến users mà không cần qua store review:

```bash
# Cài đặt Shorebird CLI
curl --proto '=https' --tlsv1.2 https://raw.githubusercontent.com/shorebirdtech/install/main/install.sh -sSf | bash

# Init trong project
shorebird init

# Release
shorebird release android
shorebird release ios

# Patch (OTA update)
shorebird patch android
shorebird patch ios
```

**Lưu ý:** Chỉ update Dart code, không thể update native code hay assets. Tuân thủ store policies.

### 5.4. App Versioning

Trong `pubspec.yaml`:

```yaml
# format: major.minor.patch+buildNumber
version: 1.2.3+45
#         │ │ │  │
#         │ │ │  └── Build number (tăng mỗi lần upload lên store)
#         │ │ └───── Patch: bug fixes
#         │ └─────── Minor: features mới, backward compatible
#         └───────── Major: breaking changes
```

**Semantic Versioning rules:**
- **1.0.0** → Release đầu tiên
- **1.0.1** → Bug fix
- **1.1.0** → Feature mới
- **2.0.0** → Redesign, breaking changes

**Build number** phải **luôn tăng** khi upload lên store. Store dùng build number để phân biệt các builds.

### 5.5. Release Checklist

```markdown
## Pre-Release Checklist

### Code Quality
- [ ] `flutter analyze` — không có warnings/errors
- [ ] `flutter test` — tất cả tests pass
- [ ] Code review đã hoàn thành
- [ ] Không có TODO/FIXME trong production code

### Build
- [ ] Version number đã cập nhật trong pubspec.yaml
- [ ] Build number đã tăng
- [ ] Build release thành công: `flutter build appbundle --release`
- [ ] Build iOS thành công: `flutter build ios --release`
- [ ] Obfuscation enabled: `--obfuscate --split-debug-info`

### Signing
- [ ] Android keystore configured
- [ ] iOS certificates and profiles valid
- [ ] Signing config tested

### Testing
- [ ] Test trên thiết bị thật (Android + iOS)
- [ ] Test trên nhiều screen sizes
- [ ] Test offline behavior
- [ ] Test deep links
- [ ] Performance profiling (profile mode)

### Store
- [ ] App metadata updated (description, screenshots, changelog)
- [ ] Privacy policy URL valid
- [ ] Age rating configured
- [ ] Contact information updated

### Post-Release
- [ ] Error monitoring active (Sentry / Crashlytics)
- [ ] Analytics configured
- [ ] Monitor crash-free rate
- [ ] Plan cho hotfix nếu cần
```

---

> 💼 **Gặp trong dự án:** Optimize build size cho App Store/Play Store limits, tree shaking, deferred components, obfuscation, production monitoring + crash reporting setup
> 🤖 **Keywords bắt buộc trong prompt:** `--split-debug-info`, `--obfuscate`, `--tree-shake-icons`, `deferred components`, `app bundle size analyze`, `Firebase Crashlytics`, `Sentry`, `production checklist`

<details>
<summary>📋 Prompt mẫu + Expected Output + Giới hạn AI</summary>

**Tình huống thực tế:**
- **App Store rejection:** Build 150MB → Google Play reject (100MB limit cho APK) → cần optimize
- **Production issue:** App crash trên user device → cần crash reporting setup + symbolicated stack traces
- **Release checklist:** QA yêu cầu production readiness checklist trước release

**Tại sao cần các keyword trên:**
- **`--obfuscate`** — obfuscate Dart code (reverse engineering protection), AI hay thiếu
- **`--split-debug-info`** — tách debug symbols (giảm size + cho crash reporting)
- **`--tree-shake-icons`** — remove unused Material icons (tiết kiệm ~1MB)
- **`deferred components`** — lazy load features, giảm initial download size
- **`Firebase Crashlytics`** — crash reporting với symbolicated stack traces

**Prompt mẫu — Production Checklist + Optimization:**
```text
Tôi cần production readiness checklist cho Flutter app trước release.
App hiện tại: 85MB APK, 120MB IPA, no crash reporting, no obfuscation.
Requirements:
1. Build size optimization:
   - flutter build apk --split-per-abi --obfuscate --split-debug-info=build/symbols --tree-shake-icons
   - Analyze: flutter build apk --analyze-size → identify biggest dependencies.
   - Target: APK < 30MB per ABI, IPA < 50MB.
2. Crash reporting: Firebase Crashlytics setup + upload debug symbols cho symbolicated traces.
3. Production checklist:
   - API endpoints: staging → production URLs.
   - Logging: disable debug logs, enable production-only logging.
   - Performance: no debugPrint, no print statements.
   - Security: ProGuard rules, network security config, certificate pinning.
   - Environment: verify app ID, version, build number.
4. Monitoring: Sentry OR Firebase Performance — response times, render times.
Constraints:
- KHÔNG include --release flags đã default.
- Debug symbols PHẢI upload cho crash reporting (symbolication).
- ProGuard: keep Flutter engine classes.
Output: build commands + Crashlytics setup + production checklist markdown.
```

**Expected Output:** AI gen build scripts + Crashlytics config + production checklist.

⚠️ **Giới hạn AI hay mắc:** AI hay quên `--split-debug-info` (crash reports unreadable without symbols!). AI cũng hay suggest ProGuard rules quá aggressive (strip Flutter engine → crash). AI hay thiếu `--tree-shake-icons` (easy 1MB win).

</details>

> 🔗 **FE Bridge:** Store distribution = **Kiểu C — không có FE equivalent**. FE deploy = upload to CDN, instantly available. Mobile = **review process** (1-3 ngày), versioning mandatory, phased rollout, user phải update. Deployment cadence & rollback completely different.

---

## 6. Build Size Optimization

### 6.1 Tại sao size matters?

| Platform | Tác động |
|---|---|
| Android | App > 150MB không download tự động qua mobile data |
| iOS | App > 200MB cảnh báo download trên cellular |
| Cả hai | User bounce rate tăng ~1% với mỗi 6MB tăng thêm (Google data) |

### 6.2 Phân tích bundle size

```bash
# Build với flag --analyze-size
flutter build apk --analyze-size
flutter build ipa --analyze-size

# Kết quả: file .json phân tích chi tiết trong build/
# Mở bằng DevTools: dart devtools --appSizeBase=<path-to-json>
```

```bash
# Xem size breakdown nhanh
flutter build apk --release --split-per-abi
# Output 3 APK riêng cho arm64-v8a, armeabi-v7a, x86_64
# Thay vì 1 fat APK chứa tất cả architectures
```

### 6.3 Kỹ thuật giảm size

#### a. Tree Shaking (tự động)

Flutter compiler tự động loại bỏ code không dùng. Để tối ưu hơn:

```dart
// ❌ Import toàn bộ package
import 'package:font_awesome_flutter/font_awesome_flutter.dart';

// ✅ Import chỉ icon cần dùng — giúp tree shaking hiệu quả hơn
import 'package:font_awesome_flutter/font_awesome_flutter.dart' show FaIcon;
```

#### b. Deferred Components (Android)

```dart
// Tách feature lớn thành deferred component — download khi cần
import 'package:heavy_feature/heavy_feature.dart' deferred as heavy;

Future<void> loadFeature() async {
  await heavy.loadLibrary(); // Download component on-demand
  heavy.showHeavyScreen();
}
```

#### c. Asset optimization

```yaml
# pubspec.yaml — chỉ include asset cần thiết
flutter:
  assets:
    - assets/icons/    # ✅ Chỉ folder cần
    # - assets/        # ❌ Không include toàn bộ root

  fonts:
    - family: Roboto
      fonts:
        - asset: fonts/Roboto-Regular.ttf
        # Chỉ include weight cần dùng, không include tất cả
```

```bash
# Compress PNG → WebP (giảm 25-34%)
# Hoặc dùng tool: https://squoosh.app/
```

#### d. ProGuard / R8 (Android)

```groovy
// android/app/build.gradle
android {
    buildTypes {
        release {
            shrinkResources true  // Loại bỏ resources không dùng
            minifyEnabled true    // Code shrinking với R8
        }
    }
}
```

### 6.4 Checklist trước release

- [ ] Build với `--split-per-abi` (Android) — giảm ~40% size
- [ ] Chạy `--analyze-size` và review top contributors
- [ ] Compress images (PNG → WebP hoặc AVIF)
- [ ] Remove unused packages từ `pubspec.yaml`
- [ ] Enable ProGuard/R8 cho Android release build
- [ ] Kiểm tra font files — chỉ include weight cần dùng

### 6.5 So sánh React/Vue ↔ Flutter

| React/Vue | Flutter |
|---|---|
| `webpack-bundle-analyzer` | `flutter build --analyze-size` + DevTools |
| Code splitting (`React.lazy`) | Deferred components |
| Tree shaking (webpack/rollup) | Tree shaking (Dart compiler, tự động) |
| Image optimization (next/image) | `cacheWidth` + compressed assets |
| Minification (terser) | R8/ProGuard (Android), bitcode (iOS) |

---

## 7. Capstone Project Review

### 7.1. Task Management App

Trong suốt 8 tuần, chúng ta đã xây dựng kiến thức để tạo một **Task Management App** hoàn chỉnh (theo reference architecture). App này áp dụng tất cả kiến thức đã học:

```
Task Management App
├── Dart fundamentals (Tuần 1)
│   └── Models, null safety, async/await
├── UI Layer (Tuần 2)
│   └── Widget tree, layouts, responsive design
├── Navigation (Tuần 3)
│   └── GoRouter, deep linking
├── State Management (Tuần 4)
│   └── Riverpod / BLoC
├── Architecture (Tuần 5)
│   └── Clean Architecture, DI with get_it
├── Data Layer (Tuần 6)
│   └── REST API, local storage, caching
├── Polish (Tuần 7)
│   └── Performance optimization, animations
└── Production (Tuần 8)
    └── Platform features, CI/CD, deployment
```

### 7.2. Architecture Review Checklist

```markdown
## Architecture Review

### Project Structure
- [ ] Clean Architecture layers rõ ràng (data, domain, presentation)
- [ ] Dependency rule: inner layers không biết outer layers
- [ ] Folder structure organized by feature

### State Management
- [ ] State management nhất quán (Riverpod hoặc BLoC)
- [ ] Không có business logic trong Widgets
- [ ] Loading/Error/Success states handled properly

### Data Layer
- [ ] Repository pattern implemented
- [ ] Network errors handled gracefully
- [ ] Local caching strategy

### Code Quality
- [ ] Null safety — không dùng `!` operator bừa bãi
- [ ] Const constructors where possible
- [ ] No magic numbers/strings
- [ ] Widget decomposition — không có widget quá 200 dòng

### Testing
- [ ] Unit tests cho business logic
- [ ] Widget tests cho UI components
- [ ] Integration tests cho critical flows
- [ ] Code coverage > 60%
```

### 7.3. Code Quality Criteria

| Tiêu chí | Kém | Trung bình | Tốt |
|-----------|-----|------------|-----|
| **Architecture** | God class, spaghetti | Có layers, nhưng leaky | Clean Architecture rõ ràng |
| **State Mgmt** | setState everywhere | Riverpod/BLoC nhưng lộn xộn | Consistent, well-organized |
| **Error Handling** | App crash khi no internet | Catch nhưng không show UI | Graceful degradation, retry |
| **Testing** | Không có tests | Unit tests only | Unit + Widget + Integration |
| **Performance** | Jank, rebuild toàn bộ | OK nhưng chưa optimize | const, keys, lazy loading |
| **Code Style** | Inconsistent, no lint | Linter nhưng nhiều ignores | Clean, consistent, documented |

### 7.4. Nhìn lại 8 tuần

```
Tuần 1: 🧱 Nền tảng
  "Hello World" → Dart syntax, null safety, async/await
  "Wow, Dart giống TypeScript quá!"

Tuần 2: 🎨 Giao diện
  Widget tree, StatelessWidget, StatefulWidget, Layouts
  "Hmm, flexbox → Row/Column, khá quen!"

Tuần 3: 🧭 Điều hướng
  Navigation, routing, state cơ bản
  "React Router → GoRouter, OK understood!"

Tuần 4: 💡 State lên level
  Riverpod, BLoC pattern
  "Redux/Vuex → Riverpod, concepts tương tự!"

Tuần 5: 🏗️ Kiến trúc
  Clean Architecture, DI, Testing
  "Separation of concerns, giống backend! Thích!"

Tuần 6: 🌐 Dữ liệu
  REST API, local storage, caching
  "fetch/axios → Dio, localStorage → SharedPreferences/Drift"

Tuần 7: ⚡ Nâng cao
  Performance, animations
  "requestAnimationFrame → AnimationController, cool!"

Tuần 8: 🚀 Production
  Platform integration, CI/CD, deployment
  "GitHub Actions quen rồi! Nhưng code signing... 😅"
```

---

## 8. Best Practices & Lỗi thường gặp

### ✅ Best Practices

```dart
// 1. Environment-based configuration
// lib/config/env.dart
enum Environment { dev, staging, production }

class AppConfig {
  static late Environment environment;
  
  static String get apiBaseUrl {
    switch (environment) {
      case Environment.dev:
        return 'https://api-dev.example.com';
      case Environment.staging:
        return 'https://api-staging.example.com';
      case Environment.production:
        return 'https://api.example.com';
    }
  }
}

// 2. Flavor-based builds
// flutter run --flavor dev -t lib/main_dev.dart
// flutter run --flavor prod -t lib/main_prod.dart
```

```yaml
# 3. CI: Fail fast — analyze trước, test sau, build cuối
steps:
  - run: flutter analyze     # Nhanh nhất, fail sớm
  - run: flutter test         # Nếu analyze pass
  - run: flutter build apk    # Chỉ build nếu test pass
```

```yaml
# 4. Semantic versioning tự động
# pubspec.yaml
version: 1.2.3+${{ github.run_number }}
```

### ❌ Lỗi thường gặp

| Lỗi | Hậu quả | Cách sửa |
|-----|---------|----------|
| Commit keystore vào git | Leak signing key | Dùng `.gitignore`, lưu trong CI secrets |
| Quên tăng build number | Store reject upload | Tự động hóa build number trong CI |
| Không test trên thiết bị thật | Bugs chỉ xuất hiện trên device | Luôn test release build trên device thật |
| Đo performance ở debug mode | Kết quả sai lệch | Luôn dùng profile/release mode |
| Không obfuscate release build | Code dễ bị reverse engineer | Thêm `--obfuscate --split-debug-info` |
| Hardcode API keys trong code | Security vulnerability | Dùng `--dart-define` hoặc env config |
| Không setup error monitoring | Không biết app crash | Setup Sentry/Crashlytics trước khi release |
| Không có CI/CD | Manual process error-prone | Setup GitHub Actions từ ngày đầu |

> 🔗 **FE Bridge:** Crashlytics ≈ **Sentry** / Datadog RUM — crash reporting + performance monitoring. Concept **tương đồng**: capture errors, stack traces, user sessions. Nhưng **khác ở**: mobile crash = **app terminate** (nghiêm trọng hơn web error), cần monitor ANR (App Not Responding) — FE không có equivalent.

---

## 9. 💡 FE → Flutter: Góc nhìn chuyển đổi 🟢

### GitHub Actions: Quen thuộc!

```
React/Vue CI:                    Flutter CI:
─────────────                    ──────────
checkout                         checkout
setup Node.js                    setup Flutter      ← Khác SDK
npm install                      flutter pub get    ← Tương tự
npm run lint                     flutter analyze    ← Tương tự
                                 dart fix --dry-run ← Auto-fix check (như eslint --fix --dry-run)
npm test                         flutter test       ← Tương tự
npm run build                    flutter build apk  ← Output khác
deploy to Vercel                 upload artifact    ← KHÁC HOÀN TOÀN
```

**Giống:** Trigger, caching, secrets, matrix — concepts hoàn toàn giống.
**Khác:** Output là binary (APK/IPA), không phải static files. Deploy lên store, không phải CDN.

### Fastlane vs Vercel/Netlify

| Aspect | Vercel/Netlify (Web) | Fastlane (Mobile) |
|--------|---------------------|-------------------|
| **Deploy time** | Vài phút | Vài giờ đến vài ngày |
| **Review** | Không có | App Store: 1-3 ngày |
| **Rollback** | Instant | Phải submit version mới |
| **Environment** | Preview deploys | TestFlight / Internal testing |
| **Signing** | Không cần | Bắt buộc (certificates) |
| **Cost** | Free tier generous | $99/năm (Apple), $25 (Google) |

### Code Signing = Khái niệm mới

Web: Bạn push code → build → deploy. **Không ai hỏi bạn là ai.**

Mobile: Bạn phải **chứng minh danh tính** (certificates) → **đăng ký app** (provisioning profiles) → **sign binary** → store verify → publish.

Đây là khác biệt lớn nhất giữa web và mobile deployment.

### Build Modes vs NODE_ENV

```
Web:                              Flutter:
────                              ────────
NODE_ENV=development              flutter run --debug
NODE_ENV=production               flutter run --release
                                  flutter run --profile   ← Không có tương đương web
```

Khác biệt: Flutter build modes ảnh hưởng **cách compile** (JIT vs AOT), không chỉ env variables.

### TestFlight vs Staging Deploy

```
Web staging:                      Mobile:
────────────                      ──────
Push to staging branch            Build + Upload to TestFlight
Vercel preview URL                Install qua TestFlight app
Anyone with URL can access        Phải invite testers by email
Instant update                    Apple review (external testers)
```

### Mindset Shifts — Thay đổi tư duy quan trọng

| # | FE Mindset | Flutter Mindset | Tại sao khác |
|---|-----------|-----------------|---------------|
| 1 | `npm run build` → upload CDN → done | Build **2 platforms** + signing + store upload + **review wait** | Mobile release = process nặng hơn FE rất nhiều |
| 2 | Deploy = instant, rollback = redeploy | Store review 1-3 ngày, **phased rollout**, user phải update | Không thể hotfix production instantly |
| 3 | Sentry captures JS errors | Crashlytics: crash = **app killed** + ANR monitoring | Mobile crash nghiêm trọng hơn web error |

---

## 10. 🎓 Tổng kết chương trình

### 16 buổi recap

| Buổi | Chủ đề | Skill chính |
|------|--------|-------------|
| 01 | Giới thiệu Dart & Flutter | Dart syntax, setup environment |
| 02 | Dart Nâng Cao | OOP, generics, async/await, null safety |
| 03 | Widget Tree Cơ Bản | StatelessWidget, StatefulWidget, lifecycle |
| 04 | Layout System | Row, Column, Stack, responsive design |
| 05 | Navigation & Routing | GoRouter, deep linking, route guards |
| 06 | State Management Cơ Bản | setState, InheritedWidget, lifting state |
| 07 | Riverpod | Provider types, code generation, StateNotifier |
| 08 | BLoC Pattern | Events, States, Cubit, BlocBuilder |
| 09 | Clean Architecture | Layers, entities, use cases, repositories |
| 10 | DI & Testing | get_it, injectable, unit/widget/integration tests |
| 11 | Networking | Dio, interceptors, error handling, retry |
| 12 | Local Storage | SharedPreferences, Drift, Hive, caching |
| 13 | Performance | DevTools, const, keys, lazy loading, isolates |
| 14 | Animation | Implicit, explicit, hero, Rive/Lottie |
| 15 | Platform Integration | Method channels, permissions, camera, maps |
| 16 | CI/CD & Production | Build modes, signing, GitHub Actions, Fastlane |

### Bạn có thể làm gì bây giờ?

Sau 8 tuần, bạn đã có khả năng:

- ✅ **Xây dựng Flutter app hoàn chỉnh** từ đầu đến cuối
- ✅ **Áp dụng Clean Architecture** — code maintainable, testable
- ✅ **State management** chuyên nghiệp với Riverpod hoặc BLoC
- ✅ **Networking & caching** — REST API, offline support
- ✅ **Performance optimization** — smooth 60fps
- ✅ **CI/CD** — tự động hóa build và deploy
- ✅ **Publish app** lên App Store và Google Play

### Lộ trình học tiếp (Advanced Topics)

```
Bạn đang ở đây
       │
       ▼
┌──────────────────────────────────────────────┐
│  Flutter Advanced                             │
│  ├── Flutter Web (PWA, SPA)                   │
│  ├── Flutter Desktop (macOS, Windows, Linux)  │
│  ├── Custom RenderObjects                     │
│  ├── Platform Views (native UI embedding)     │
│  ├── FFI (Foreign Function Interface)         │
│  ├── Flame (game development)                 │
│  ├── Custom Paint & Canvas                    │
│  ├── Isolates & Compute (heavy computation)   │
│  ├── Flutter Internals (rendering pipeline)   │
│  └── Package development & publishing         │
├──────────────────────────────────────────────┤
│  Backend & Full-stack                         │
│  ├── Dart Frog / Serverpod (Dart backend)     │
│  ├── Firebase suite (Auth, Firestore, etc.)   │
│  ├── Supabase (open-source Firebase alt)      │
│  └── GraphQL with Ferry/Artemis               │
├──────────────────────────────────────────────┤
│  DevOps & Scaling                             │
│  ├── Feature flags (remote config)            │
│  ├── A/B testing                              │
│  ├── App analytics (deep)                     │
│  ├── Codemagic / Bitrise (CI/CD platforms)    │
│  └── Multi-module architecture                │
└──────────────────────────────────────────────┘
```

### Community Resources

- **Flutter Discord** — [discord.gg/flutter](https://discord.gg/flutter)
- **r/FlutterDev** — Reddit community
- **flutter.dev/community** — Official community page
- **Medium / dev.to** — Flutter articles
- **YouTube** — Flutter channel, Fireship, Robert Brunhage, Tadas Petra
- **Twitter/X** — Follow Flutter team, community leaders

### Lời cuối

> 🎓 **Chúc mừng bạn đã hoàn thành chương trình Flutter Training!**
>
> Từ một frontend developer (React/Vue), bạn đã trải qua 8 tuần để trở thành một mobile developer có khả năng xây dựng, test, và deploy ứng dụng Flutter production-ready.
>
> Hành trình không dừng ở đây. Flutter đang phát triển nhanh chóng, và cộng đồng rất active. Hãy tiếp tục build, học hỏi, và chia sẻ kiến thức.
>
> **Keep building! 🚀**
