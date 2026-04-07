# 03-exercise.md — Fastlane & Store Deployment

## PRACTICE: Fastlane Exercises

---

## Exercise 1: Trace iOS Fastlane Lane Flow ⭐

### Objective

Trace và hiểu iOS Fastlane lane flow từ code đến deployment.

### Steps

1. **Open iOS Fastfile:**
   ```bash
   cat base_flutter/ios/fastlane/Fastfile
   ```

2. **Identify these components:**
   - Public lanes (4 lanes: develop, qa, staging, production)
   - Private methods (helper functions)
   - API authentication method
   - Version management approach

3. **Trace the flow:**
   ```
   increase_version_build_and_up_testflight_develop
   └── increase_version_and_build_and_deploy_to_test_flight(DEV_FLAVOR)
       ├── 1. Get xcconfig path
       ├── 2. Get bundle_id
       ├── 3. Get App Store ID
       ├── 4. Create API key
       ├── 5. Get version from pubspec
       ├── 6. Fetch latest TestFlight build
       ├── 7. Increment build number
       ├── 8. Build IPA (via Makefile)
       ├── 9. Revert pubspec
       ├── 10. Upload to TestFlight
       └── 11. Send Slack notification
   ```

### Questions

1. Tại sao cần increment build number trước build?
2. Tại sao revert pubspec.yaml sau build?
3. App Store Connect API key được dùng để làm gì?

---

## Exercise 2: Trace Android Fastlane Lane Flow ⭐

### Objective

Trace và hiểu Android Fastlane lane flow.

### Steps

1. **Open Android Fastfile:**
   ```bash
   cat base_flutter/android/fastlane/Fastfile
   ```

2. **Identify these components:**
   - Public lanes (4 lanes)
   - Firebase App Distribution configuration
   - Version management

3. **Compare with iOS:**

   | Aspect | iOS | Android |
   |--------|-----|---------|
   | Deployment target | TestFlight | Firebase |
   | API authentication | App Store Connect | Firebase Token |
   | Build command | make build_*_ipa | make build_*_apk |

### Questions

1. Firebase App Distribution khác gì Google Play Store?
2. Tại sao Android không cần certificates như iOS?

---

## Exercise 3: Analyze Makefile Integration ⭐⭐

### Objective

Understand cách Fastlane gọi Makefile để build.

### Steps

1. **Read Makefile:**
   ```bash
   cat base_flutter/makefile | grep -A 5 "build_dev_ipa"
   cat base_flutter/makefile | grep -A 5 "build_dev_apk"
   ```

2. **Identify:**
   - Build commands cho iOS (4 flavors)
   - Build commands cho Android (4 flavors)
   - Deployment commands (cd_*)

3. **Trace complete flow:**

   ```
   fastlane increase_version_build_and_up_testflight_develop
   └── build_ipa(DEV_FLAVOR)
       └── make build_dev_ipa
           └── flutter build ipa --release --flavor develop ...
   ```

### Bonus

1. Tìm các build commands khác (qa, staging, production)
2. So sánh với Codemagic build commands

---

## Exercise 4: Add New Fastlane Lane ⭐⭐

### Objective

Thêm một Fastlane lane mới cho một use case mới.

### Scenario

Team muốn thêm lane để build và deploy **adhoc** build (không cần TestFlight).

### Steps

1. **Create new lane:**

   ```ruby
   # Trong ios/fastlane/Fastfile
   desc "Adhoc: Build for testing without TestFlight"
   lane :build_adhoc_develop do
     # 1. Build IPA
     # 2. Export without signing
     # 3. Save to artifacts
   end
   ```

2. **Integration với Makefile:**
   
   Thêm vào `makefile`:
   ```makefile
   build_adhoc_ios:
   	flutter build ipa --release --flavor develop \
   	  --export-options-plist=ios/exportOptionsAdhoc.plist
   ```

### Questions

1. Adhoc build khác gì TestFlight build?
2. Cần cấu hình gì thêm cho adhoc?

---

## Exercise 5: AI Prompt Dojo — Fastlane Review ⭐⭐⭐

### Objective

Viết prompt để AI review Fastlane configuration.

### Scenario

Team mới có Fastlane setup nhưng chưa tối ưu. Cần AI giúp review.

### ❌ Bad Prompt

```
Review my Fastlane setup
```

### ✅ Good Prompt

```
Review my Fastlane configuration for a Flutter iOS app:

1. I have 4 lanes (develop/qa/staging/production) that all call the same helper function
2. Each lane increments build number before build
3. I use App Store Connect API for authentication
4. I deploy to TestFlight after build

Problems I'm facing:
- Build sometimes fails with "Duplicate Bundle ID"
- Slack notifications not sent on failure
- Version management seems fragile

Analyze and suggest:
1. Code improvements for the Fastfile
2. Better error handling
3. Version management optimization
```

### Your Task

1. Viết prompt để review Android Fastfile
2. Include specific questions about Firebase App Distribution
3. Ask about version management best practices

### Output Format

```
Prompt:
[Your prompt here]

Expected Review Areas:
1. [Area 1]
2. [Area 2]
3. [Area 3]
```

---

## Exercise Verification

Sau khi hoàn thành các bài tập, verify bằng cách:

### Exercise 1: iOS Lane Flow

- [ ] Đọc `ios/fastlane/Fastfile`
- [ ] Tracer được 4 public lanes
- [ ] Hiểu được `increase_version_and_build_and_deploy_to_test_flight()` flow
- [ ] Trả lời được 3 câu hỏi

### Exercise 2: Android Lane Flow

- [ ] Đọc `android/fastlane/Fastfile`
- [ ] Tracer được 4 public lanes
- [ ] So sánh được với iOS
- [ ] Trả lời được 2 câu hỏi

### Exercise 3: Makefile Integration

- [ ] Đọc `makefile`
- [ ] Tìm được build commands cho cả iOS và Android
- [ ] Tracer được complete flow
- [ ] So sánh được với Codemagic

### Exercise 4: New Lane

- [ ] Viết được lane code mới
- [ ] Tích hợp được với Makefile
- [ ] Hiểu được adhoc vs TestFlight

### Exercise 5: AI Prompt

- [ ] Viết được prompt chi tiết cho iOS review
- [ ] Viết được prompt cho Android review
- [ ] Cover đủ các areas cần review

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
