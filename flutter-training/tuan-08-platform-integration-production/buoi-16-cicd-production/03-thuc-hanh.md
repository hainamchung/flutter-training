# Buổi 16: CI/CD & Production — Thực Hành

> **Hướng dẫn học từng phần:**
> - 🔴 **Tự code tay — Hiểu sâu:** Bắt buộc tự viết, không dùng AI generate. Dùng AI chỉ để *hỏi khi không hiểu* — prompt mẫu: `"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue, có điểm tương đồng nào không?"`
> - 🟡 **Code cùng AI:** Tự nghĩ logic → dùng AI gen boilerplate → đọc hiểu → customize. Prompt mẫu: `"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."`
> - 🟢 **AI gen → Bạn review:** Để AI viết code → dùng checklist trong bài để review → fix → chạy thử. Prompt mẫu: `"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."`

---

## ⚡ FE → Flutter: Chú ý khi thực hành

> CI/CD cho mobile **phức tạp hơn nhiều** so với FE deployment.
> FE developer quen "push → deploy → done" cần chuẩn bị cho multi-step release process.

| FE Deploy Habit | Flutter Reality | Bài tập liên quan |
|-----------------|-----------------|---------------------|
| `npm run build` → upload CDN | Build APK/IPA + signing certificates + store upload | BT1, BT2 |
| CI = GitHub Actions, simple YAML | CI cần **macOS runner** cho iOS, signing setup in CI environment | BT2 |
| Deploy = instant rollback available | Store review process, phased rollout, **không instant rollback** | BT3 |

---

## BT1 ⭐: Build Release APK & App Bundle 🟡

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | Flutter app (dùng project hiện có) |
| **Tạo project** | Dùng project Flutter hiện có hoặc `flutter create bt1_release_build` |
| **Cách chạy** | `flutter build apk --release` / `flutter build appbundle --release` |
| **Output** | File APK/AAB trong `build/app/outputs/` |

### Mục tiêu
Thực hành build ứng dụng Flutter ở release mode, tạo keystore, sign app, và kiểm tra kích thước.

### Yêu cầu

#### Phần A: Build cơ bản

1. **Clean và build debug APK:**
   ```bash
   flutter clean
   flutter pub get
   flutter build apk --debug
   ```
   Ghi lại kích thước file APK.

2. **Build release APK:**
   ```bash
   flutter build apk --release
   ```
   So sánh kích thước với debug APK.

3. **Build release APK với obfuscation:**
   ```bash
   flutter build apk --release \
     --obfuscate \
     --split-debug-info=build/debug-info
   ```
   So sánh kích thước với release APK không obfuscate.

4. **Build App Bundle:**
   ```bash
   flutter build appbundle --release
   ```

#### Phần B: Android Signing

5. **Tạo keystore:**
   ```bash
   keytool -genkey -v \
     -keystore android/app/my-release-key.jks \
     -keyalg RSA \
     -keysize 2048 \
     -validity 10000 \
     -alias my-key-alias
   ```

6. **Tạo file `android/key.properties`:**
   ```properties
   storePassword=<password bạn đã nhập>
   keyPassword=<password bạn đã nhập>
   keyAlias=my-key-alias
   storeFile=my-release-key.jks
   ```

7. **Cấu hình `android/app/build.gradle`** để sử dụng signing config (xem VD4 trong 02-vi-du.md).

8. **Thêm vào `.gitignore`:**
   ```
   android/key.properties
   *.jks
   *.keystore
   ```

9. **Build lại release APK** và verify app đã được sign:
   ```bash
   flutter build apk --release
   
   # Verify signing
   jarsigner -verify -verbose -certs \
     build/app/outputs/flutter-apk/app-release.apk
   ```

#### Phần C: Báo cáo kích thước

10. **Điền bảng so sánh:**

| Build Mode | APK Size | Ghi chú |
|------------|----------|---------|
| Debug | _____ MB | |
| Release | _____ MB | |
| Release + obfuscate | _____ MB | |
| Release + split-per-abi (arm64) | _____ MB | |

### Kết quả mong đợi

- ✅ Build thành công ở tất cả các modes
- ✅ Keystore được tạo và cấu hình đúng
- ✅ Release APK đã được sign
- ✅ Bảng kích thước hoàn chỉnh, debug >> release
- ✅ Files nhạy cảm (keystore, key.properties) đã được thêm vào .gitignore

---

## BT2 ⭐⭐: Setup GitHub Actions CI 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | CI/CD config (YAML) |
| **Tạo project** | Tạo trong project Flutter hiện có: `mkdir -p .github/workflows` |
| **Cách chạy** | `git push` → GitHub Actions tự động chạy |
| **Output** | Workflow pass trên GitHub Actions tab |

### Mục tiêu
Tạo GitHub Actions workflow để tự động chạy analyze, test, và build mỗi khi push code.

### Yêu cầu

#### Phần A: Tạo workflow file

1. **Tạo thư mục và file:**
   ```bash
   mkdir -p .github/workflows
   ```

2. **Tạo file `.github/workflows/flutter-ci.yml`** với nội dung:

```yaml
name: Flutter CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    name: Build & Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'
          cache: true
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .
      
      - name: Analyze
        run: flutter analyze --fatal-infos
      
      - name: Run tests
        run: flutter test
      
      - name: Build APK
        run: flutter build apk --release
      
      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: release-apk
          path: build/app/outputs/flutter-apk/app-release.apk
          retention-days: 7
```

#### Phần B: Test workflow

3. **Push code lên GitHub:**
   ```bash
   git add .github/workflows/flutter-ci.yml
   git commit -m "ci: add Flutter CI workflow"
   git push origin main
   ```

4. **Kiểm tra trên GitHub:**
   - Vào tab **Actions** trên GitHub repository
   - Xem workflow đang chạy
   - Kiểm tra từng step có pass không

#### Phần C: Cải thiện workflow

5. **Thêm caching cho pub dependencies** (ngoài Flutter SDK cache):
   ```yaml
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

6. **Thêm badge vào README.md:**
   ```markdown
   ![Flutter CI](https://github.com/<username>/<repo>/actions/workflows/flutter-ci.yml/badge.svg)
   ```

7. **Tạo một PR** và verify workflow chạy trên pull request.

### Kết quả mong đợi

- ✅ File `.github/workflows/flutter-ci.yml` tồn tại và đúng cú pháp
- ✅ Workflow trigger thành công khi push
- ✅ Tất cả steps (format, analyze, test, build) pass
- ✅ APK artifact có thể download từ Actions tab
- ✅ Badge hiển thị trên README
- ✅ Workflow cũng chạy khi tạo PR

---

## BT3 ⭐⭐⭐: Full CI/CD Pipeline 🟢

| Mục | Chi tiết |
|-----|----------|
| **Loại project** | CI/CD pipeline (YAML + Fastlane) |
| **Tạo project** | Tạo trong project Flutter hiện có: Fastlane + GitHub Actions |
| **Cách chạy** | `git tag v1.0.0 && git push --tags` → pipeline trigger |
| **Output** | APK/AAB được deploy lên Internal Testing / TestFlight |

### Mục tiêu
Xây dựng pipeline hoàn chỉnh: GitHub Actions trigger → Fastlane build → Deploy lên TestFlight (iOS) hoặc Internal Testing (Android).

### Yêu cầu

#### Phần A: Setup Fastlane

1. **Cài đặt Fastlane:**
   ```bash
   # Tạo Gemfile ở root project
   cat > Gemfile << 'EOF'
   source "https://rubygems.org"
   gem "fastlane"
   EOF
   
   bundle install
   ```

2. **Init Fastlane cho Android:**
   ```bash
   cd android
   bundle exec fastlane init
   ```

3. **Tạo `android/fastlane/Fastfile`:**
   ```ruby
   default_platform(:android)

   platform :android do
     desc "Deploy to Internal Testing"
     lane :beta do
       sh "cd ../.. && flutter build appbundle --release " \
          "--obfuscate --split-debug-info=build/debug-info"

       upload_to_play_store(
         track: 'internal',
         aab: '../build/app/outputs/bundle/release/app-release.aab',
         json_key: ENV['PLAY_STORE_JSON_KEY_PATH'] || 'play-store-key.json',
         skip_upload_metadata: true,
         skip_upload_images: true,
         skip_upload_screenshots: true
       )
     end
   end
   ```

4. **(Optional — nếu có Mac) Init Fastlane cho iOS:**
   ```bash
   cd ios
   bundle exec fastlane init
   ```

#### Phần B: Setup Secrets trên GitHub

5. **Encode keystore thành base64:**
   ```bash
   base64 -i android/app/my-release-key.jks | pbcopy  # macOS
   # hoặc
   base64 -i android/app/my-release-key.jks > keystore_base64.txt  # Linux
   ```

6. **Thêm secrets vào GitHub:**
   Vào Repo → Settings → Secrets and variables → Actions → New repository secret:

   | Secret Name | Value |
   |-------------|-------|
   | `ANDROID_KEYSTORE_BASE64` | Base64 string của keystore |
   | `KEY_PROPERTIES` | Nội dung file key.properties |
   | `PLAY_STORE_JSON_KEY` | Service account JSON (từ Google Cloud Console) |

   > 💡 **Service Account JSON:** Tạo tại Google Cloud Console → IAM → Service Accounts → tạo key JSON → cấp quyền trên Play Console.

#### Phần C: Tạo Deploy Workflow

7. **Tạo `.github/workflows/deploy.yml`:**

```yaml
name: Deploy to Stores

on:
  push:
    tags:
      - 'v*'  # Trigger khi tạo tag: git tag v1.0.0 && git push --tags

jobs:
  # ─── CI: Test trước ───
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          cache: true
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test

  # ─── CD: Deploy Android ───
  deploy-android:
    name: Deploy Android
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          cache: true
      
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      
      - run: flutter pub get
      
      # Setup signing
      - name: Decode keystore
        env:
          KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          KEY_PROPERTIES: ${{ secrets.KEY_PROPERTIES }}
        run: |
          echo "$KEYSTORE_BASE64" | base64 --decode > android/app/keystore.jks
          echo "$KEY_PROPERTIES" > android/key.properties
      
      # Setup Play Store credentials
      - name: Setup Play Store key
        env:
          PLAY_STORE_JSON: ${{ secrets.PLAY_STORE_JSON_KEY }}
        run: |
          echo "$PLAY_STORE_JSON" > android/play-store-key.json
      
      # Deploy via Fastlane
      - name: Deploy to Internal Testing
        working-directory: android
        run: bundle exec fastlane beta
      
      # Cleanup sensitive files
      - name: Cleanup
        if: always()
        run: |
          rm -f android/app/keystore.jks
          rm -f android/key.properties
          rm -f android/play-store-key.json
```

#### Phần D: Test Pipeline

8. **Tạo tag và push:**
   ```bash
   # Update version trong pubspec.yaml
   # version: 1.0.0+1
   
   git add .
   git commit -m "chore: prepare release v1.0.0"
   git tag v1.0.0
   git push origin main --tags
   ```

9. **Monitor trên GitHub Actions:**
   - Verify test job pass
   - Verify deploy job trigger
   - Check Fastlane output logs

10. **Verify trên Play Console** (nếu configured):
    - Internal Testing track có build mới

### Kết quả mong đợi

- ✅ Fastlane installed và Fastfile configured
- ✅ Secrets thêm vào GitHub repository
- ✅ Deploy workflow trigger khi tạo tag
- ✅ Pipeline: test → build → sign → deploy
- ✅ Sensitive files được cleanup sau deploy
- ✅ Hiểu flow từ code push đến app trên store

### Sơ đồ pipeline hoàn chỉnh

```
Developer
    │
    ├── git push (code change)
    │       └── flutter-ci.yml ──▶ analyze + test + build
    │
    └── git tag v1.0.0 + push
            └── deploy.yml
                    │
                    ├── Test job (analyze + test)
                    │       │ (pass)
                    │       ▼
                    ├── Deploy Android
                    │   ├── Decode secrets
                    │   ├── Fastlane beta
                    │   │   ├── flutter build appbundle
                    │   │   └── upload_to_play_store (internal)
                    │   └── Cleanup secrets
                    │
                    └── Deploy iOS (nếu có)
                        ├── Match certificates
                        ├── Fastlane beta
                        │   ├── flutter build ios
                        │   └── upload_to_testflight
                        └── Cleanup
```

---

## 💬 Câu hỏi thảo luận

### Câu 1: Đầu tư CI/CD

> **Khi nào nên đầu tư thời gian setup CI/CD? Ngay từ đầu project hay khi project đã ổn định?**

Gợi ý thảo luận:
- CI cơ bản (analyze + test) → nên setup **ngay từ đầu**. Chi phí thấp, lợi ích cao.
- CD (auto deploy) → có thể setup **sau khi có users** hoặc khi team > 1 người.
- ROI: Project nhỏ, solo dev → CI đủ. Project lớn, nhiều người → CI/CD bắt buộc.
- So sánh với web: Vercel auto-deploy rất dễ setup. Mobile phức tạp hơn, nhưng lợi ích cũng rõ ràng khi team lớn.

### Câu 2: Deploy Strategy

> **Bạn sẽ deploy app như thế nào? Internal testing → Closed beta → Production? Hay push thẳng production? Khi nào nên dùng chiến lược nào?**

Gợi ý thảo luận:
- **Phased rollout** (Google Play): Release cho 5% → 20% → 50% → 100% users. Phát hiện bugs sớm.
- **Internal → Beta → Production**: An toàn, nhưng chậm. Phù hợp khi update lớn.
- **Hotfix**: Đôi khi cần push nhanh. Khi đó CI/CD giúp tiết kiệm thời gian enormously.
- **Shorebird/Code Push**: Quick fixes cho Dart code, bypass store review. Trade-offs?

### Câu 3: Monitoring sau khi launch

> **App đã lên store. Bạn cần monitor những gì? Tools nào? Khi nào cần action?**

Gợi ý thảo luận:
- **Crash-free rate**: Mục tiêu > 99.5%. Dùng Firebase Crashlytics hoặc Sentry.
- **ANR rate** (Android): Application Not Responding. Mục tiêu < 0.5%.
- **User reviews**: Monitor 1-star reviews, respond nhanh.
- **Performance metrics**: App startup time, frame rendering time.
- **Analytics**: User retention, feature usage, funnel analysis.
- So sánh với web: Web có Lighthouse, Core Web Vitals. Mobile có tương đương: startup time, jank rate, battery usage.

---

## 🤖 Thực hành cùng AI

> Bài tập dưới đây rèn kỹ năng **giao việc cho AI đúng cách** trong môi trường production:
> biết task thực tế cần gì → viết prompt đúng context & constraints → review output → customize.
>
> **Tuần 8 (Final):** Focus vào full AI-first feature delivery — từ scaffold → CI/CD → production checklist.

### AI-BT1: Full AI-First Feature Delivery — CI/CD Pipeline + Production Checklist ⭐⭐⭐

**1. Từ Concept đến Task thực tế:**
- **Concept vừa học:** Build modes, code signing, GitHub Actions, Fastlane, deployment, build size optimization, production monitoring.
- **Task thực tế:** Team lead giao: "Setup CI/CD pipeline hoàn chỉnh + production checklist trước release tuần sau." AI gen pipeline config + Fastlane lanes + production checklist. Bạn review secrets handling + build optimization + monitoring setup.

**2. Viết Prompt có Context & Constraints:**

```text
Tôi cần setup CI/CD pipeline hoàn chỉnh cho Flutter app trước production release.
Tech stack: Flutter 3.x, GitHub Actions, Fastlane, Firebase Crashlytics.

Gen tất cả config files:
1. .github/workflows/ci.yml:
   - Trigger: on PR to main.
   - Steps: checkout → setup Flutter → pub get → analyze → test --coverage → check coverage 80% → build APK.
   - Cache: pub cache, Gradle.
2. .github/workflows/cd.yml:
   - Trigger: on push to main.
   - Steps: build release APK (--split-per-abi --obfuscate --split-debug-info) → build IPA → upload artifacts → deploy.
   - Android: upload to Play Store Internal Testing (via Fastlane).
   - iOS: upload to TestFlight (via Fastlane).
3. Fastlane:
   - Android: Fastfile with lanes internal_test, beta, production.
   - iOS: Fastfile with lanes testflight, app_store.
   - match setup cho code signing.
4. Production checklist: markdown checklist (API endpoints, logging, security, monitoring, versioning).

Constraints:
- ALL secrets in GitHub Secrets (KHÔNG hardcode).
- Build size: APK target < 30MB, IPA < 50MB.
- Crashlytics: upload dSYM + debug symbols.
Output: ci.yml + cd.yml + Fastfile (Android) + Fastfile (iOS) + PRODUCTION_CHECKLIST.md.
```

**3. Expected Output & Review Checklist:**

*AI sẽ gen:* 5 config files.

| # | Kiểm tra | ✅ |
|---|---|---|
| 1 | Secrets NOT hardcoded? All via ${{ secrets.XXX }}? | ☐ |
| 2 | CI: analyze + test + build order correct? | ☐ |
| 3 | CD: --obfuscate + --split-debug-info present? | ☐ |
| 4 | Cache: pub + Gradle + CocoaPods cached? | ☐ |
| 5 | Fastlane: match for iOS code signing? | ☐ |
| 6 | Debug symbols uploaded for crash reporting? | ☐ |
| 7 | Production checklist covers: API URLs, logging, security, monitoring? | ☐ |

**4. Customize:**
Thêm Slack notification khi build fail. Thêm automatic version bump (build number increment). Thêm staged rollout (1% → 10% → 50% → 100%). AI gen basic pipeline — tự thêm notifications + version management + staged rollout.
