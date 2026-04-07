# Module 22: Fastlane & Store Deployment

## Tổng quan

Module này đi sâu vào **Fastlane automation** cho cả iOS và Android deployment. Bạn sẽ đọc và hiểu cách Fastlane tự động hóa build, signing, testing, và deployment lên App Store / Play Store.

> **Note:** M22 tập trung vào **Fastlane** (store deployment). Các CI platforms khác (Bitbucket Pipelines, Codemagic) đã được cover trong [Module 19](../module-19-cicd/). M22 yêu cầu hoàn thành M19 trước.

**Cycle:** CODE (đọc Fastlane configs) → EXPLAIN (hiểu concepts) → PRACTICE (configure lanes) → VERIFY.

**Prerequisite:** Hoàn thành [Module 19 — CI/CD](../module-19-cicd/) (CI infrastructure, build variants).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Setup được Fastlane lanes cho iOS (TestFlight) và Android (Play Store)?
2. Configure được Gym cho iOS build + signing?
3. Setup được Match để quản lý certificates và profiles?
4. Configure được Supply cho Android Play Store deployment?
5. Quản lý được secrets (API keys, certificates) trong Fastlane?
6. Phân biệt được các loại certificates: Development, Ad Hoc, Distribution?
7. Explain được build number management (increment + rollback)?

→ Nếu **7/7 Yes** — chuyển thẳng [Module 23 — Performance](../module-23-performance/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## 🏷️ Badge Summary

7 concepts rút ra từ code walk, phân loại theo mức độ cần nắm:

| # | Concept | Badge | Ý nghĩa |
|---|---------|-------|----------|
| 1 | Fastlane Architecture | 🔴 MUST-KNOW | Tool ecosystem, lane concept |
| 2 | iOS Fastfile Structure | 🔴 MUST-KNOW | Gym, Scan, Match, Deliver |
| 3 | Android Fastfile Structure | 🔴 MUST-KNOW | Supply, Screengrab |
| 4 | Certificates & Signing | 🔴 MUST-KNOW | Provisioning profiles, keystores |
| 5 | Build Number Management | 🟡 SHOULD-KNOW | Increment, fetch, rollback |
| 6 | Environment Variables | 🟡 SHOULD-KNOW | Secrets management |
| 7 | Deployment Automation | 🟢 AI-GENERATE | Store upload, release notes |

**Phân bố:** 🔴 ~57% · 🟡 ~29% · 🟢 ~14%

---

## 📂 Files trong Module này

| File | Nội dung | Vai trò |
|------|----------|---------|
| [01-code-walk.md](./01-code-walk.md) | Đọc iOS Fastfile, Android Fastfile, Makefile integration | CODE — quan sát |
| [02-concept.md](./02-concept.md) | 7 concepts: Fastlane ecosystem, iOS/Android patterns | EXPLAIN — giải thích |
| [03-exercise.md](./03-exercise.md) | 5 bài tập: trace lanes → configure → deploy | PRACTICE — làm tay |
| [04-verify.md](./04-verify.md) | Checklist tự đánh giá + cross-check | VERIFY — kiểm tra |

### Exercises tóm tắt

| # | Bài tập | Độ khó |
|---|---------|--------|
| 1 | Trace iOS Fastlane Lane Flow | ⭐ |
| 2 | Trace Android Fastlane Lane Flow | ⭐ |
| 3 | Analyze Makefile Integration | ⭐⭐ |
| 4 | Add New Fastlane Lane | ⭐⭐ |
| 5 | AI Prompt Dojo — Fastlane Review | ⭐⭐⭐ |

---

## 🔗 Liên kết

- [ios/fastlane/Fastfile](base_flutter/ios/fastlane/Fastfile) — iOS Fastlane configuration
- [android/fastlane/Fastfile](base_flutter/android/fastlane/Fastfile) — Android Fastlane configuration
- [makefile](base_flutter/makefile) — Build orchestration với Fastlane integration

---

## 💡 FE Perspective

| Flutter (Fastlane) | FE Equivalent |
|--------------------|---------------|
| Gym | `npm run build` cho web |
| Match | Certificate management — **mobile-specific** |
| Deliver | Deployment — tương đương SCP/rsync upload |
| Supply | Play Store console API — **mobile-specific** |
| Screengrab | Screenshot automation — **mobile-specific** |

---

## Unlocks (Module 23+)

Sau khi hoàn thành Module 22, bạn sẽ:

- **Module 23 — Performance Optimization:** Optimize app performance sau khi deployment pipeline hoàn chỉnh.
- **Capstone Project:** Fastlane ready cho capstone deployment.

→ Bắt đầu: [01-code-walk.md](./01-code-walk.md)

<!-- AI_VERIFY: generation-complete -->
