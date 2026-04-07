# 04-verify.md — CI/CD Pipeline

## VERIFY: Self-Assessment Checklist

---

## Before You Start

Đảm bảo đã hoàn thành:

- [ ] Đọc `01-code-walk.md` và trace các pipeline configs
- [ ] Đọc `02-concept.md` và hiểu concepts
- [ ] Hoàn thành ít nhất Exercise 1 và 2
- [ ] Reviewed Fastlane configuration files

---

## Self-Assessment Questions

### Section 1: Fastlane iOS

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 1.1 | Lane naming convention: `increase_version_build_and_up_testflight_*`? | ___ |
| 1.2 | Helper function: `increase_version_and_build_and_deploy_to_test_flight()`? | ___ |
| 1.3 | TestFlight deployment action: `upload_to_testflight`? | ___ |
| 1.4 | App Store Connect auth: `app_store_connect_api_key()`? | ___ |

### Section 2: Fastlane Android

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 2.1 | Lane naming convention: `increase_version_build_and_up_firebase_*`? | ___ |
| 2.2 | Helper function: `increase_version_and_build_and_deploy_to_firebase()`? | ___ |
| 2.3 | Firebase deployment action: `firebase_app_distribution`? | ___ |
| 2.4 | Firebase App Distribution groups: phân nhóm testers? | ___ |

### Section 3: Makefile Integration

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 3.1 | CI targets format: `cd_*_*` (VD: `cd_dev_android`)? | ___ |
| 3.2 | Build targets format: `build_*_*` (VD: `build_dev_apk`)? | ___ |
| 3.3 | Makefile gọi fastlane qua `cd {platform} && fastlane {lane}`? | ___ |
| 3.4 | Flavors được định nghĩa trong Fastfile và `.env.default`? | ___ |

### Section 4: Version Management

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 4.1 | Version name từ `pubspec.yaml`? | ___ |
| 4.2 | Build number được tăng tự động dựa trên TestFlight/Firebase release? | ___ |
| 4.3 | Build number được revert sau khi build (để giữ pubspec.yaml original)? | ___ |
| 4.4 | Dart tool `set_build_number_pubspec.dart` được dùng để update version? | ___ |

### Section 5: Notifications

| # | Câu hỏi | ✓/✗ |
|---|---------|-----|
| 5.1 | Slack notification được gửi sau khi deploy thành công? | ___ |
| 5.2 | Slack notification được gửi khi deploy thất bại? | ___ |
| 5.3 | Changelog được đọc từ `RELEASE_NOTES.md`? | ___ |
| 5.4 | `@mentions` được sử dụng trong Slack messages? | ___ |

---

## Answer Key

### Section 1: Fastlane iOS

| # | Answer |
|---|--------|
| 1.1 | ✅ Đúng: `increase_version_build_and_up_testflight_develop/qa/staging/production` |
| 1.2 | ✅ Đúng: Helper function nhận flavor và gọi các sub-functions |
| 1.3 | ✅ Đúng: `upload_to_testflight` action được sử dụng trong `deploy_to_test_flight` |
| 1.4 | ✅ Đúng: `app_store_connect_api_key()` với key_id, issuer_id, key_filepath |

### Section 2: Fastlane Android

| # | Answer |
|---|--------|
| 2.1 | ✅ Đúng: `increase_version_build_and_up_firebase_develop/qa/staging/production` |
| 2.2 | ✅ Đúng: Helper function nhận app_id và flavor |
| 2.3 | ✅ Đúng: `firebase_app_distribution` action được sử dụng trong `deploy_to_firebase` |
| 2.4 | ✅ Đúng: Firebase groups để phân nhóm testers (VD: "testers") |

### Section 3: Makefile Integration

| # | Answer |
|---|--------|
| 3.1 | ✅ Đúng: `cd_dev_android`, `cd_qa_android`, `cd_dev_ios`, v.v. |
| 3.2 | ✅ Đúng: `build_dev_apk`, `build_qa_apk`, `build_dev_ipa`, v.v. |
| 3.3 | ✅ Đúng: `cd android && fastlane increase_version_build_and_up_firebase_develop` |
| 3.4 | ✅ Đúng: Flavors được define trong cả Fastfile và .env.default |

### Section 4: Version Management

| # | Answer |
|---|--------|
| 4.1 | ✅ Đúng: `get_version_name_of_pubspec()` đọc từ pubspec.yaml |
| 4.2 | ✅ Đúng: `latest_testflight_build_number` hoặc `firebase_app_distribution_get_latest_release` |
| 4.3 | ✅ Đúng: Revert sau khi build để giữ pubspec.yaml clean |
| 4.4 | ✅ Đúng: Dart tool trong `tools/dart_tools/lib/set_build_number_pubspec.dart` |

### Section 5: Notifications

| # | Answer |
|---|--------|
| 5.1 | ✅ Đúng: `send_slack(message, success = true)` sau khi deploy thành công |
| 5.2 | ✅ Đúng: `error(exception)` → `send_slack(..., success = false)` |
| 5.3 | ✅ Đúng: `get_changelog()` đọc từ `RELEASE_NOTES.md` |
| 5.4 | ✅ Đúng: `MENTIONS_SUCCESS` và `MENTIONS_ERROR` được sử dụng |

---

## Badge Targets

### 🔴 MUST-KNOW (Phải trả lời đúng 100%)

- [ ] 1.1: iOS lane naming convention
- [ ] 1.3: TestFlight deployment action
- [ ] 2.1: Android lane naming convention
- [ ] 2.3: Firebase deployment action
- [ ] 3.1: Makefile CI target format

**Target: 5/5 correct** → Foundation concepts

### 🟡 SHOULD-KNOW (Nên trả lời đúng ≥80%)

- [ ] Section 1: 3+/4 correct
- [ ] Section 2: 3+/4 correct
- [ ] Section 3: 3+/4 correct
- [ ] Section 4: 3+/4 correct
- [ ] Section 5: 3+/4 correct

**Target: ≥16/20 correct**

### 🟢 AI-GENERATE (Understand when to use)

- [ ] Understand how to add a new lane
- [ ] Understand version management flow
- [ ] Understand Slack notification integration

---

## Practical Verification

### Verify Fastlane Configuration

```bash
# 1. List iOS lanes
cd base_flutter/ios && bundle exec fastlane lanes

# 2. List Android lanes
cd base_flutter/android && bundle exec fastlane lanes

# 3. Check lane syntax
cd base_flutter/ios && bundle exec fastlane --help
```

### Verify Makefile Targets

```bash
# List all CI/CD related targets
grep -E "^(cd_|build_)" base_flutter/makefile
```

### Verify Source Code Integration

<!-- AI_VERIFY: ios-fastlane-source -->

**iOS Fastfile Lanes:**
```ruby
# base_flutter/ios/fastlane/Fastfile

# Lanes có sẵn:
lane :increase_version_build_and_up_testflight_develop do
  increase_version_and_build_and_deploy_to_test_flight(DEV_FLAVOR)
end

lane :increase_version_build_and_up_testflight_qa do
  increase_version_and_build_and_deploy_to_test_flight(QA_FLAVOR)
end

# Helper function:
def increase_version_and_build_and_deploy_to_test_flight(flavor)
  # 1. Get build info
  # 2. Fetch latest TestFlight build
  # 3. Increase build number
  # 4. Build IPA
  # 5. Deploy to TestFlight
  # 6. Send Slack notification
end
```

<!-- AI_VERIFY: android-fastlane-source -->

**Android Fastfile Lanes:**
```ruby
# base_flutter/android/fastlane/Fastfile

# Lanes có sẵn:
lane :increase_version_build_and_up_firebase_develop do
  increase_version_and_build_and_deploy_to_firebase(DEV_FIREBASE_APP_ID, DEV_FLAVOR)
end

lane :increase_version_build_and_up_firebase_qa do
  increase_version_and_build_and_deploy_to_firebase(QA_FIREBASE_APP_ID, QA_FLAVOR)
end

# Helper function:
def increase_version_and_build_and_deploy_to_firebase(app_id, flavor)
  # 1. Get latest Firebase release
  # 2. Increase build number
  # 3. Build APK
  # 4. Deploy to Firebase App Distribution
  # 5. Send Slack notification
end
```

<!-- AI_VERIFY: makefile-source -->

**Makefile Targets:**
```makefile
# base_flutter/makefile

# CI targets (gọi Fastlane):
cd_dev_android:
    cd android && fastlane increase_version_build_and_up_firebase_develop

cd_qa_android:
    cd android && fastlane increase_version_build_and_up_firebase_qa

cd_dev_ios:
    cd ios && fastlane increase_version_build_and_up_testflight_develop

# Build targets (chỉ build, không deploy):
build_dev_apk:
    flutter build apk --flavor develop ...

build_dev_ipa:
    flutter build ipa --release --flavor develop ...
```

---

## Cross-Check with Module 22

Kiến thức từ Module 22 được sử dụng trong:

| Context | Module Usage |
|---------|--------------|
| CI/CD automation | Automated deployments for testing |
| Version management | Consistent build numbers across platforms |
| Notification integration | Team communication on deployments |

---

## Sign-Off

Sau khi hoàn thành checklist và đạt badge targets:

```markdown
## M22 Completion Status

- [ ] All 🔴 MUST-KNOW answered correctly
- [ ] ≥80% 🟡 SHOULD-KNOW answered correctly
- [ ] Can trace Fastlane lane flow
- [ ] Can explain Makefile integration
- [ ] Understand version management

**Verdict:** [ ] PASS / [ ] NEEDS REVIEW
**Date:** ___
**Notes:** ___
```

---

## If You Need Review

Nếu chưa đạt targets:

1. **Đọc lại** `01-code-walk.md` và `02-concept.md` - tập trung vào 🔴 concepts
2. **Trace** actual Fastlane files trong `base_flutter/ios/fastlane/` và `base_flutter/android/fastlane/`
3. **Hoàn thành** Exercise 3 (thêm lane mới)
4. **Hỏi** team member hoặc mentor

---

## Next Module

→ [Module 23 — Performance (Advanced)](../module-23-performance/)

<!-- AI_VERIFY: generation-complete -->
