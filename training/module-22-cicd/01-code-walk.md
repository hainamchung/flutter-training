# 01-code-walk.md — Fastlane & Store Deployment

## CODE Walk: Fastlane Infrastructure

> **Note:** Module 19 covered CI/CD infrastructure (Bitbucket Pipelines, Codemagic, Lefthook). This module focuses on **Fastlane** — the tool for building and deploying to app stores.

---

## 1. Fastlane Architecture Overview

### 1.1 What is Fastlane?

Fastlane là tool automation cho iOS/Android để:
- Build apps (Gym for iOS, Gradle for Android)
- Run tests (Scan for iOS, Gradle for Android)
- Sign code (Match for iOS, Gradle signing for Android)
- Deploy to stores (Deliver/pilot for iOS, Supply for Android)
- Screenshots (Snapshot for iOS, Screengrab for Android)

### 1.2 Tool Ecosystem

| Tool | Platform | Purpose |
|------|---------|---------|
| **Gym** | iOS | Build .ipa |
| **Scan** | iOS | Run tests |
| **Match** | iOS | Certificate/profile management |
| **Deliver/Pilot** | iOS | App Store deployment |
| **Supply** | Android | Play Store deployment |
| **Screengrab** | Android | Screenshots |

---

## 2. iOS Fastfile Deep Dive

### 2.1 Full Source Code

<!-- AI_VERIFY: base_flutter/ios/fastlane/Fastfile -->

→ [Mở file gốc: `ios/fastlane/Fastfile`](../../base_flutter/ios/fastlane/Fastfile)

```ruby
# ACTUAL SOURCE: ios/fastlane/Fastfile (276 lines)

require 'dotenv'
require 'yaml'
Dotenv.load(File.expand_path('../..', __dir__) + '/.env.default')

default_platform(:ios)

platform :ios do
  # Load env variables from .env.default
  SLACK_HOOKS_URL_SUCCESS = ENV["SLACK_HOOKS_URL_SUCCESS"]
  SLACK_HOOKS_URL_ERROR = ENV["SLACK_HOOKS_URL_ERROR"]
  MENTIONS_SUCCESS = ENV["MENTIONS_SUCCESS"] || "@channel"
  MENTIONS_ERROR = ENV["MENTIONS_ERROR"] || "@minhnt3"
  DEV_FLAVOR = ENV["DEV_FLAVOR"] || "develop"
  QA_FLAVOR = ENV["QA_FLAVOR"] || "qa"
  STG_FLAVOR = ENV["STG_FLAVOR"] || "staging"
  PROD_FLAVOR = ENV["PROD_FLAVOR"] || "production"

  # ===== LANES =====
  desc "Develop: Increase version, build & deploy app to TestFlight"
  lane :increase_version_build_and_up_testflight_develop do
    increase_version_and_build_and_deploy_to_test_flight(DEV_FLAVOR)
  end

  desc "Qa: Increase version, build & deploy app to TestFlight"
  lane :increase_version_build_and_up_testflight_qa do
    increase_version_and_build_and_deploy_to_test_flight(QA_FLAVOR)
  end

  desc "Staging: Increase version, build & deploy app to TestFlight"
  lane :increase_version_build_and_up_testflight_staging do
    increase_version_and_build_and_deploy_to_test_flight(STG_FLAVOR)
  end

  desc "Production: Increase version, build & deploy app to TestFlight"
  lane :increase_version_build_and_up_testflight_production do
    increase_version_and_build_and_deploy_to_test_flight(PROD_FLAVOR)
  end

  # ===== IMPLEMENTATION =====
  def increase_version_and_build_and_deploy_to_test_flight(flavor)
    begin
      xcconfig = get_xcconfig_path(flavor)
      bundle_id = get_bundle_id(flavor)
      app_store_id = get_app_store_id(flavor)
      ipa_path = get_ipa_file_path(flavor)

      # App Store Connect API Key authentication
      api_key = app_store_connect_api_key(
        key_id: get_key_id(flavor),
        issuer_id: get_issuer_id(flavor),
        key_filepath: get_key_filepath(flavor),
        duration: 1200,
        in_house: false
      )

      version_name = get_version_name_of_pubspec()
      latest_release = latest_testflight_build_number(
        api_key: api_key, team_id: get_team_id(flavor),
        app_identifier: bundle_id, version: version_name
      )

      build_number = get_build_number_of_pubspec()
      project_root = get_project_root_path()
      new_build_number = latest_release.nil? ? 1 : latest_release.to_i + 1

      # Increment build number via Dart tool
      sh("cd ../.. && dart run #{project_root}/tools/dart_tools/lib/set_build_number_pubspec.dart #{new_build_number}")

      # Build .ipa via Makefile
      build_ipa(flavor)

      # Revert pubspec.yaml build number
      sh("cd ../.. && dart run #{project_root}/tools/dart_tools/lib/set_build_number_pubspec.dart #{build_number}")

      # Upload to TestFlight
      deploy_to_test_flight(bundle_id, app_store_id, ipa_path, xcconfig, flavor, api_key)

      version = "#{version_name}(#{new_build_number})"
      changelog = get_changelog()
      send_slack("#{MENTIONS_SUCCESS} *Deployed #{flavor} to TestFlight. iOS #{version}*\n*Release notes:* #{changelog}")
    rescue => exception
      error(exception)
    end
  end

  def build_ipa(flavor)
    begin
      if flavor == DEV_FLAVOR
        sh("cd ../.. && make build_dev_ipa")
      elsif flavor == QA_FLAVOR
        sh("cd ../.. && make build_qa_ipa")
      elsif flavor == STG_FLAVOR
        sh("cd ../.. && make build_stg_ipa")
      elsif flavor == PROD_FLAVOR
        sh("cd ../.. && make build_prod_ipa")
      else
        raise "flavor #{flavor} is invalid"
      end
    rescue => exception
      error(exception)
    end
  end

  def deploy_to_test_flight(bundle_id, app_store_id, ipa_path, xcconfig, flavor, api_key)
    begin
      changelog = get_changelog()
      upload_to_testflight(
        api_key: api_key,
        changelog: changelog,
        ipa: ipa_path,
        distribute_external: true,
        notify_external_testers: true,
        groups: get_test_flight_groups(flavor),
        apple_id: app_store_id,
        itc_provider: get_team_id(flavor)
      )
    rescue => exception
      error(exception)
    end
  end

  # ===== HELPERS =====
  def get_app_store_id(flavor)
    case flavor
    when DEV_FLAVOR then return ENV["DEV_APP_STORE_ID"]
    when QA_FLAVOR then return ENV["QA_APP_STORE_ID"]
    when STG_FLAVOR then return ENV["STG_APP_STORE_ID"]
    when PROD_FLAVOR then return ENV["PROD_APP_STORE_ID"]
    else raise "Unknown flavor: #{flavor}"
    end
  end

  def get_team_id(flavor)
    flavor == PROD_FLAVOR ? ENV["PROD_TEAM_ID"] : ENV["QA_TEAM_ID"]
  end

  def get_key_id(flavor)
    flavor == PROD_FLAVOR ? ENV["PROD_KEY_ID"] : ENV["QA_KEY_ID"]
  end

  def get_key_filepath(flavor)
    flavor == PROD_FLAVOR ? ENV["PROD_KEY_FILEPATH"] : ENV["QA_KEY_FILEPATH"]
  end

  def get_issuer_id(flavor)
    flavor == PROD_FLAVOR ? ENV["PROD_ISSUER_ID"] : ENV["QA_ISSUER_ID"]
  end

  def get_test_flight_groups(flavor)
    case flavor
    when DEV_FLAVOR then return ENV["DEV_TEST_FLIGHT_EXTERNAL_GROUPS"] || "testers"
    when QA_FLAVOR then return ENV["QA_TEST_FLIGHT_EXTERNAL_GROUPS"] || "testers"
    when STG_FLAVOR then return ENV["STG_TEST_FLIGHT_EXTERNAL_GROUPS"] || "testers"
    when PROD_FLAVOR then return ENV["PROD_TEST_FLIGHT_EXTERNAL_GROUPS"] || "testers"
    else raise "Unknown flavor: #{flavor}"
    end
  end

  def get_bundle_id(flavor)
    path = get_xcconfig_path(flavor)
    return get_xcconfig_value(path: path, name: "PRODUCT_BUNDLE_IDENTIFIER")
  end

  def get_xcconfig_path(flavor)
    return "Flutter/#{flavor.capitalize}.xcconfig"
  end

  def get_ipa_file_path(flavor)
    path = get_xcconfig_path(flavor)
    app_name = get_xcconfig_value(path: path, name: "APP_DISPLAY_NAME")
    return "../build/ios/ipa/#{app_name}.ipa"
  end

  def get_project_root_path()
    return File.expand_path("../..", __dir__)
  end

  def get_version_name_of_pubspec()
    pubspec_path = File.join(get_project_root_path(), "pubspec.yaml")
    pubspec = YAML.load_file(pubspec_path)
    return pubspec["version"].split("+").first
  end

  def get_build_number_of_pubspec()
    pubspec_path = File.join(get_project_root_path(), "pubspec.yaml")
    pubspec = YAML.load_file(pubspec_path)
    return pubspec["version"].split("+").last.to_i
  end

  def get_changelog()
    changelog_path = File.join(get_project_root_path(), "RELEASE_NOTES.md")
    return File.exist?(changelog_path) ? File.read(changelog_path) : "No release notes provided."
  end

  def send_slack(message, success = true)
    slack(message: message, success: success,
          slack_url: success ? SLACK_HOOKS_URL_SUCCESS : SLACK_HOOKS_URL_ERROR,
          link_names: true, default_payloads: [:git_branch, :lane])
  end

  def error(exception)
    send_slack("#{MENTIONS_ERROR} Build failed: #{exception.to_s}", false)
  end
end
```

### 2.2 Key Observations

| Aspect | Pattern |
|--------|----------|
| **4 lanes** | develop, qa, staging, production |
| **API Auth** | App Store Connect API Key (not password) |
| **Build delegation** | Calls `make build_*_ipa` |
| **Version mgmt** | Custom Dart tool (`set_build_number_pubspec.dart`) |
| **Config source** | `.env.default` file |
| **Notifications** | Slack on success/failure |
| **Upload** | `upload_to_testflight` action |

### 2.3 Lane Flow Diagram

```
increase_version_build_and_up_testflight_develop
    │
    ├── 1. Get API key (App Store Connect)
    ├── 2. Fetch latest TestFlight build number
    ├── 3. Increment pubspec.yaml build number
    ├── 4. Run: make build_dev_ipa
    ├── 5. Revert pubspec.yaml
    ├── 6. Upload to TestFlight
    └── 7. Send Slack notification
```

---

## 3. Android Fastfile Deep Dive

### 3.1 Full Source Code

<!-- AI_VERIFY: base_flutter/android/fastlane/Fastfile -->

→ [Mở file gốc: `android/fastlane/Fastfile`](../../base_flutter/android/fastlane/Fastfile)

```ruby
# ACTUAL SOURCE: android/fastlane/Fastfile
# Uses Firebase App Distribution — NOT Google Play Supply

require 'dotenv'
Dotenv.load(File.expand_path('../..', __dir__) + '/.env.default')

default_platform(:android)

platform :android do
  # secrets & configs (from .env.default)
  FIREBASE_TOKEN = ENV["FIREBASE_TOKEN"]
  SLACK_HOOKS_URL_SUCCESS = ENV["SLACK_HOOKS_URL_SUCCESS"]
  SLACK_HOOKS_URL_ERROR = ENV["SLACK_HOOKS_URL_ERROR"]
  MENTIONS_SUCCESS = ENV["MENTIONS_SUCCESS"] || "@channel"
  MENTIONS_ERROR = ENV["MENTIONS_ERROR"] || "@minhnt3"
  DEV_FLAVOR = ENV["DEV_FLAVOR"] || "develop"
  QA_FLAVOR = ENV["QA_FLAVOR"] || "qa"
  STG_FLAVOR = ENV["STG_FLAVOR"] || "staging"
  PROD_FLAVOR = ENV["PROD_FLAVOR"] || "production"

  # Firebase App Distribution IDs
  DEV_FIREBASE_APP_ID = ENV["DEV_FIREBASE_APP_ID"]
  QA_FIREBASE_APP_ID = ENV["QA_FIREBASE_APP_ID"]
  STG_FIREBASE_APP_ID = ENV["STG_FIREBASE_APP_ID"]
  PROD_FIREBASE_APP_ID = ENV["PROD_FIREBASE_APP_ID"]

  # Firebase groups
  DEV_FIREBASE_GROUPS = ENV["DEV_FIREBASE_GROUPS"] || "testers"
  QA_FIREBASE_GROUPS = ENV["QA_FIREBASE_GROUPS"] || "testers"
  STG_FIREBASE_GROUPS = ENV["STG_FIREBASE_GROUPS"] || "testers"
  PROD_FIREBASE_GROUPS = ENV["PROD_FIREBASE_GROUPS"] || "testers"

  # ===== LANES =====
  desc "Develop: Build & deploy to Firebase Distribution"
  lane :increase_version_build_and_up_firebase_develop do
    increase_version_and_build_and_deploy_to_firebase(DEV_FIREBASE_APP_ID, DEV_FLAVOR)
  end

  desc "Qa: Build & deploy to Firebase Distribution"
  lane :increase_version_build_and_up_firebase_qa do
    increase_version_and_build_and_deploy_to_firebase(QA_FIREBASE_APP_ID, QA_FLAVOR)
  end

  desc "Staging: Build & deploy to Firebase Distribution"
  lane :increase_version_build_and_up_firebase_staging do
    increase_version_and_build_and_deploy_to_firebase(STG_FIREBASE_APP_ID, STG_FLAVOR)
  end

  desc "Production: Build & deploy to Firebase Distribution"
  lane :increase_version_build_and_up_firebase_production do
    increase_version_and_build_and_deploy_to_firebase(PROD_FIREBASE_APP_ID, PROD_FLAVOR)
  end

  # ===== IMPLEMENTATION =====
  def increase_version_and_build_and_deploy_to_firebase(app_id, flavor)
    begin
      # Fetch latest release
      latest_release = firebase_app_distribution_get_latest_release(
        app: app_id,
        firebase_cli_token: FIREBASE_TOKEN
      )

      build_number = get_build_number_of_pubspec()
      project_root = get_project_root_path()
      new_build_number = latest_release.nil? ? 1 : latest_release[:buildVersion].to_i + 1

      # Increment build number
      sh("cd ../.. && dart run #{project_root}/tools/dart_tools/lib/set_build_number_pubspec.dart #{new_build_number}")

      # Build APK
      build_apk(flavor)

      # Revert build number
      sh("cd ../.. && dart run #{project_root}/tools/dart_tools/lib/set_build_number_pubspec.dart #{build_number}")

      # Deploy to Firebase
      changelog_path = get_changelog()
      deploy_to_firebase(flavor, changelog_path)

      # Slack notification
      changelog = File.exist?(changelog_path) ? File.read(changelog_path) : "No release notes provided."
      version = "#{get_version_name_of_pubspec()}(#{new_build_number})"
      send_slack("#{MENTIONS_SUCCESS} *Deployed #{flavor} to Firebase. Android #{version}*\n*Release notes:* #{changelog}")
    rescue => exception
      error(exception)
    end
  end

  def build_apk(flavor)
    if flavor == DEV_FLAVOR
      sh("cd ../.. && make build_dev_apk")
    elsif flavor == QA_FLAVOR
      sh("cd ../.. && make build_qa_apk")
    elsif flavor == STG_FLAVOR
      sh("cd ../.. && make build_stg_apk")
    elsif flavor == PROD_FLAVOR
      sh("cd ../.. && make build_prod_apk")
    else
      raise "flavor #{flavor} is invalid"
    end
  end

  def deploy_to_firebase(flavor, changelog_path)
    app_id = get_firebase_app_id(flavor)
    apk_path = get_apk_file_path(flavor)
    firebase_app_distribution(
      firebase_cli_token: FIREBASE_TOKEN,
      app: app_id,
      groups: get_firebase_groups(flavor),
      android_artifact_path: apk_path,
      release_notes_file: changelog_path
    )
  end

  def get_firebase_app_id(flavor)
    case flavor
      when DEV_FLAVOR then return DEV_FIREBASE_APP_ID
      when QA_FLAVOR then return QA_FIREBASE_APP_ID
      when STG_FLAVOR then return STG_FIREBASE_APP_ID
      when PROD_FLAVOR then return PROD_FIREBASE_APP_ID
      else raise "Unknown flavor: #{flavor}"
    end
  end

  def get_apk_file_path(flavor)
    return "../build/app/outputs/flutter-apk/app-#{flavor}-release.apk"
  end

  def get_project_root_path()
    return File.expand_path("../..", __dir__)
  end

  def get_build_number_of_pubspec()
    pubspec_path = File.join(get_project_root_path(), "pubspec.yaml")
    pubspec = YAML.load_file(pubspec_path)
    return pubspec["version"].split("+").last.to_i
  end

  def get_version_name_of_pubspec()
    pubspec_path = File.join(get_project_root_path(), "pubspec.yaml")
    pubspec = YAML.load_file(pubspec_path)
    return pubspec["version"].split("+").first
  end

  def get_changelog()
    changelog_path = File.join(get_project_root_path(), "RELEASE_NOTES.md")
    return changelog_path
  end

  def send_slack(message, success = true)
    slack(message: message, success: success,
          slack_url: success ? SLACK_HOOKS_URL_SUCCESS : SLACK_HOOKS_URL_ERROR,
          link_names: true, default_payloads: [:git_branch, :lane])
  end

  def error(exception)
    send_slack("#{MENTIONS_ERROR} Build failed: #{exception.to_s}", false)
  end
end
```

### 3.2 Key Observations

| Aspect | Pattern |
|--------|----------|
| **4 lanes** | develop, qa, staging, production |
| **Deployment** | Firebase App Distribution (not Google Play) |
| **Build delegation** | Calls `make build_*_apk` |
| **Version mgmt** | Same Dart tool as iOS |
| **Config source** | `.env.default` file |
| **Notifications** | Slack on success/failure |

### 3.3 Lane Flow Diagram

```
increase_version_build_and_up_firebase_develop
    │
    ├── 1. Fetch latest Firebase release
    ├── 2. Get current build number from pubspec
    ├── 3. Increment build number
    ├── 4. Run: make build_dev_apk
    ├── 5. Revert pubspec.yaml
    ├── 6. Deploy to Firebase App Distribution
    └── 7. Send Slack notification
```

---

## 4. Makefile Integration

### 4.1 Fastlane-Related Commands

<!-- AI_VERIFY: base_flutter/makefile -->

→ [Mở file gốc: `makefile`](../../base_flutter/makefile)

```makefile
# === iOS Builds ===
build_dev_ipa:
	flutter build ipa --release --flavor develop -t lib/main.dart \
	  --dart-define-from-file=dart_defines/develop.json \
	  --export-options-plist=ios/exportOptions.plist

# Similar for: build_qa_ipa, build_stg_ipa, build_prod_ipa

# === Android Builds ===
build_dev_apk:
	flutter build apk --flavor develop -t lib/main.dart \
	  --dart-define-from-file=dart_defines/develop.json

# Similar for: build_qa_apk, build_stg_apk, build_prod_apk

build_dev_aab:
	flutter build appbundle --flavor develop -t lib/main.dart \
	  --dart-define-from-file=dart_defines/develop.json

# === Continuous Deployment ===
cd_dev_android:
	cd android && fastlane increase_version_build_and_up_firebase_develop

cd_dev_ios:
	cd ios && fastlane increase_version_build_and_up_testflight_develop
```

### 4.2 Build Command Pattern

| Command | Purpose |
|---------|---------|
| `make build_dev_ipa` | Build iOS for develop |
| `make build_dev_apk` | Build Android APK for develop |
| `make build_dev_aab` | Build Android App Bundle |
| `make cd_dev_android` | Deploy Android to Firebase |
| `make cd_dev_ios` | Deploy iOS to TestFlight |

---

## 5. Version Management

### 5.1 How It Works

The project uses a **Dart tool** to manage build numbers:

```bash
# Increment build number
dart run tools/dart_tools/lib/set_build_number_pubspec.dart 42

# This modifies pubspec.yaml version field
# From: "1.0.0+5" → "1.0.0+42"
```

### 5.2 Why Increment Before Build?

1. App Store requires **unique build numbers** for each upload
2. Fetching latest build number → incrementing → building ensures no collision
3. Reverting pubspec.yaml after build keeps local version unchanged

---

## Summary

Qua bước code walk này, bạn đã:

1. **iOS Fastfile:** Hiểu lane structure, App Store Connect API, TestFlight deployment
2. **Android Fastfile:** Hiểu Firebase App Distribution, version management
3. **Makefile:** Thấy build commands được gọi từ Fastlane
4. **Version Management:** Hiểu cách build numbers được tự động tăng

→ Tiếp theo: [02-concept.md](./02-concept.md)

<!-- AI_VERIFY: generation-complete -->
