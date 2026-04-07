# 02-concept.md — Fastlane & Store Deployment

## CONCEPTS: Fastlane Automation

> Mỗi concept dưới đây được trích từ code đã đọc trong [01-code-walk.md](./01-code-walk.md).

---

## 1. Fastlane Architecture 🔴 MUST-KNOW

### Why Fastlane?

Fastlane tự động hóa các task lặp đi lặp lại trong mobile development:
- **Build**: iOS (Gym), Android (Gradle)
- **Test**: iOS (Scan), Android (Gradle test)
- **Sign**: iOS (Match), Android (signingConfigs)
- **Deploy**: iOS (Deliver/Pilot), Android (Supply/Firebase)

### FE Comparison

| Fastlane | FE Equivalent |
|----------|---------------|
| Gym | `npm run build` |
| Match | Certificate management — **mobile only** |
| Deliver | SCP/rsync upload |
| Supply | Play Store API — **mobile only** |

---

## 2. iOS Fastfile Structure 🔴 MUST-KNOW

### Lane Pattern

```ruby
# 1 lane per environment
lane :increase_version_build_and_up_testflight_develop do
  increase_version_and_build_and_deploy_to_test_flight(DEV_FLAVOR)
end

# Shared implementation
def increase_version_and_build_and_deploy_to_test_flight(flavor)
  # 1. Get API key
  # 2. Fetch latest build
  # 3. Increment version
  # 4. Build IPA
  # 5. Revert version
  # 6. Upload
  # 7. Slack
end
```

### Key Components

| Component | Purpose |
|-----------|---------|
| `app_store_connect_api_key()` | Authenticate với Apple |
| `latest_testflight_build_number()` | Get last build number |
| `upload_to_testflight()` | Upload to TestFlight |
| `slack()` | Send notifications |

---

## 3. Android Fastfile Structure 🔴 MUST-KNOW

### Lane Pattern

```ruby
lane :increase_version_build_and_up_firebase_develop do
  increase_version_and_build_and_deploy_to_firebase(DEV_FIREBASE_APP_ID, DEV_FLAVOR)
end

def increase_version_and_build_and_deploy_to_firebase(app_id, flavor)
  # 1. Fetch latest release
  # 2. Get current build
  # 3. Increment
  # 4. Build APK
  # 5. Revert
  # 6. Deploy to Firebase
  # 7. Slack
end
```

### Key Components

| Component | Purpose |
|-----------|---------|
| `firebase_app_distribution_get_latest_release()` | Get last release |
| `firebase_app_distribution()` | Upload to Firebase |
| `build_apk()` | Build via Makefile |

---

## 4. Certificates & Signing 🔴 MUST-KNOW

### iOS Certificates

| Type | Purpose | Usage |
|------|---------|-------|
| **Development** | Debug builds | Local dev |
| **Ad Hoc** | Testing | Internal testers |
| **Distribution** | App Store | Production |

### Project Uses

- **App Store Connect API Key** — modern authentication
- **Provisioning Profiles** — device whitelisting
- **Match** — automated certificate management (optional)

---

## 5. Build Number Management 🟡 SHOULD-KNOW

### How It Works

```
pubspec.yaml: "1.0.0+5"
                    ↑     ↑
               version  build

1. Fetch latest: 5
2. Increment: +1 → 6
3. Build with: 6
4. Revert to: 5 (keep local unchanged)
```

### Why Revert?

- CI modifies `pubspec.yaml` for build
- Local version should stay unchanged
- Only CI environment gets the incremented version

---

## 6. Environment Variables 🟡 SHOULD-KNOW

### .env.default Pattern

```ruby
# Load from .env.default
Dotenv.load(File.expand_path('../..', __dir__) + '/.env.default')

# Access as ENV["KEY"]
SLACK_HOOKS_URL = ENV["SLACK_HOOKS_URL_SUCCESS"]
FIREBASE_TOKEN = ENV["FIREBASE_TOKEN"]
```

### Secrets Structure

| Type | Storage | Example |
|------|---------|---------|
| Tokens | CI/CD env vars | `FIREBASE_TOKEN` |
| App IDs | CI/CD env vars | `DEV_FIREBASE_APP_ID` |
| API Keys | CI/CD env vars | `APP_STORE_CONNECT_KEY` |

---

## 7. Deployment Automation 🟢 AI-GENERATE

### Deployment Flow

```
┌─────────────┐
│  Developer  │
│  triggers   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Fastlane   │
│  Lane       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Makefile  │
│   Build     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Store     │
│   Deploy    │
└─────────────┘
```

### iOS: TestFlight → App Store

1. TestFlight (internal testing)
2. Beta testers
3. App Store (requires review)

### Android: Firebase → Play Store

1. Firebase App Distribution (internal)
2. Internal testing track
3. Production (requires review)

---

→ Tiếp theo: [03-exercise.md](./03-exercise.md)

<!-- AI_VERIFY: generation-complete -->
