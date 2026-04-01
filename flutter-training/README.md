# 🚀 Flutter Training — Frontend Developer → Middle Mobile Developer

> **85 files** · **16 buổi** · **8 tuần** · **AI-Integrated** · **Peer-to-Peer**

Bạn là Frontend Developer (React / Vue). Bạn hiểu component lifecycle, state management, async patterns, clean architecture. Bạn không cần học lại những thứ đó.

Chương trình này **ánh xạ trực tiếp** từ kiến thức FE bạn đã có sang Flutter — kết hợp **AI workflow chuẩn 2026** — để bạn trở thành **Middle Mobile Developer trong 8 tuần**, không cần trainer chuyên nghiệp.

---

## 📌 Mục lục

- [Vì sao chương trình này khác](#-vì-sao-chương-trình-này-khác)
- [AI-First — Workflow chuẩn 2026](#-ai-first--workflow-chuẩn-2026)
- [Hệ thống ưu tiên 🔴🟡🟢](#-hệ-thống-ưu-tiên-)
- [Lộ trình 16 buổi](#-lộ-trình-16-buổi)
- [Mục tiêu đầu ra](#-mục-tiêu-đầu-ra)
- [Cài đặt môi trường](#%EF%B8%8F-cài-đặt-môi-trường-phát-triển)
- [Cách đọc tài liệu](#-cách-đọc-tài-liệu)
- [Tài liệu bổ sung](#-tài-liệu-bổ-sung)

---

## ✨ Vì sao chương trình này khác

### 🔗 Tận dụng kiến thức đã có

Không dạy lại "component là gì" hay "state management hoạt động thế nào". Thay vào đó, mỗi buổi ánh xạ trực tiếp:

| Bạn đã biết (React/Vue) | Bạn sẽ học (Flutter) |
|---|---|
| `useState` / `ref()` | `setState`, `ValueNotifier` |
| Context API / Provide/Inject | `InheritedWidget`, `Provider` |
| Redux / Pinia / Zustand | `BLoC`, `Riverpod` |
| React Router / Vue Router | `GoRouter`, Navigator 2.0 |
| Flexbox / CSS Grid | `Row`, `Column`, `Stack`, `Flex` |
| `useEffect` cleanup | `dispose()`, widget lifecycle |

Kết quả: tiết kiệm 60–70% thời gian so với học từ zero.

### 🤖 AI-First từ ngày 1

Không phải "dùng AI nếu muốn". Đây là workflow bắt buộc. Mỗi buổi học đều tích hợp:
- **Task → Keywords & Constraints** — prompt AI ra code production-ready
- **Expected Output** — biết kết quả đúng trông như thế nào
- **Review Checklist** — những lỗi AI hay mắc (và cách bắt)
- **Manual Override** — phần nào bắt buộc tự viết tay

Chi tiết ở [section AI-First bên dưới](#-ai-first--workflow-chuẩn-2026).

### 👥 Học bằng cách dạy

Mô hình peer-to-peer: mỗi buổi 1 người present, rotate. Người trình bày phải hiểu sâu hơn để dạy — đây là [hiệu ứng Feynman](https://en.wikipedia.org/wiki/Learning_by_teaching). 85 file chuẩn hoá đảm bảo chất lượng nhất quán bất kể ai là presenter. Chi phí training: **gần bằng 0**.

---

## 🤖 AI-First — Workflow chuẩn 2026

> **Triết lý:** AI không dạy Flutter. Tài liệu làm việc đó. AI là **người dịch Task** — giúp bạn map từ task thực tế sang code production-ready. Bạn cần hiểu concept đủ sâu để **ra đề đúng** và **review kết quả**.

### AI xuất hiện thế nào trong mỗi buổi học?

**Trong lý thuyết (`01-ly-thuyet.md`):**

Mỗi file lý thuyết bắt đầu với hướng dẫn AI theo 3 mức:

> - 🔴 **Tự code tay:** Dùng AI chỉ để hỏi khi không hiểu
>   - Prompt mẫu: _"Giải thích [concept] trong Dart/Flutter. Background của tôi là React/Vue."_
> - 🟡 **Code cùng AI:** Tự nghĩ logic → AI gen boilerplate → review + customize
>   - Prompt mẫu: _"Tạo [component] với Flutter, theo [pattern], có [constraint cụ thể]."_
> - 🟢 **AI gen → Review:** AI viết code → dùng checklist review → fix → chạy thử
>   - Prompt mẫu: _"Generate [feature] hoàn chỉnh theo [architecture], [tech stack]. Bao gồm error handling."_

**Trong thực hành (`03-thuc-hanh.md`):**

Bài tập theo quy trình 4 bước:

```
1. CONCEPT  — Hiểu bài toán và constraints
2. TASK     — Xác định yêu cầu cụ thể
3. PROMPT   — Ra đề cho AI với đúng keywords
4. REVIEW   — Kiểm tra output theo checklist, customize phần quan trọng
```

**Trong tham khảo (`04-tai-lieu-tham-khao.md`):**

Mỗi buổi có AI Prompt Library sẵn sàng copy-paste cho các task phổ biến.

### So sánh: Có AI workflow vs. Không có

| Tình huống | Không có AI | Có AI + Prompt đúng |
|---|---|---|
| Chưa thuộc Dart syntax | 30–60 phút Google/StackOverflow | 5 phút: mô tả ý định → AI gen đúng |
| Gặp lỗi Flutter lạ | 40 phút debug + search | 5 phút: paste error + context → AI giải thích |
| Implement Clean Architecture | 2 ngày scaffold từ đầu | Nửa ngày: AI gen structure → bạn customize business logic |
| Ship production-ready code | ~60% quality lần đầu | 85%+ quality với review checklist |
| Viết unit test | 1–2 giờ viết tay | 20 phút: AI gen → review edge cases → bổ sung |

### Ví dụ cụ thể

> Bạn vừa học xong `NotifierProvider` ở buổi 07.
>
> PM giao task: "Cart với realtime badge update."
>
> Bạn prompt AI: _"Tạo CartNotifier dùng NotifierProvider trong Riverpod. State là immutable list CartItem. Cần: addItem, removeItem, clearCart, getter totalPrice và itemCount. Dùng copyWith pattern. Include badge count cho BottomNavigationBar."_
>
> AI gen code → bạn review theo checklist: ✅ immutable state? ✅ copyWith? ✅ dispose? ✅ error handling? → Fix nếu cần → Commit.

Đây không phải "dùng AI nếu muốn." Đây là cách developer năm 2026 làm việc.

---

## 🎯 Hệ thống ưu tiên 🔴🟡🟢

> **Nguyên tắc:** Trong thời đại AI, không phải mọi thứ cần học giống nhau. Mỗi heading trong tài liệu được gắn tag:

| Icon | Mức | Chiến lược học | Ví dụ |
|:---:|------|---------------|-------|
| 🔴 | **MUST** — Bắt buộc hiểu sâu | Tự code tay. AI chỉ giải thích khi không hiểu | Widget Lifecycle, Null Safety, Clean Architecture |
| 🟡 | **SHOULD** — AI hỗ trợ được | Hiểu concept → AI gen boilerplate → review + customize | GoRouter setup, BlocBuilder, JSON serialization |
| 🟢 | **AI-OK** — AI viết tốt | Đọc lướt concept → AI generate → dùng checklist review | GitHub Actions YAML, Animation code, Plugin config |
| ⚪ | **REF** — Tra cứu | Bookmark, đọc khi cần | So sánh React/Vue, tài liệu tham khảo |

### Phân bổ effort đề xuất

```
🔴 MUST   — 60% thời gian │████████████████████░░░░░░░░░░░░│ Buổi 01, 02, 03, 06, 09, 13
🟡 SHOULD — 30% thời gian │██████████░░░░░░░░░░░░░░░░░░░░░│ Buổi 04, 05, 07, 08, 10, 11
🟢 AI-OK  — 10% thời gian │███░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ Buổi 12, 14, 15, 16
```

### 🏆 Top 10 — Bắt buộc hiểu sâu (AI KHÔNG thay thế được)

| # | Chủ đề | Buổi | Tại sao |
|---|--------|:---:|---------|
| 1 | **Null Safety** (?, !, ??, late) | 01 | AI generate code null-unsafe. Bạn phải catch |
| 2 | **Async/Await, Future, Stream** | 02 | Race condition, deadlock — AI không biết context app bạn |
| 3 | **Widget/Element/Render Tree** | 03 | Debug performance cần hiểu 3-tree architecture |
| 4 | **setState & Widget Lifecycle** | 03 | Sai = memory leak, crash. AI không biết lifecycle context |
| 5 | **Constraints Model (3 quy tắc vàng)** | 04 | 90% layout bug là do không hiểu constraints |
| 6 | **State Architecture** | 06 | AI không biết business context để chọn pattern đúng |
| 7 | **ref.watch vs ref.read** | 07 | Sai = rebuild toàn bộ app, performance collapse |
| 8 | **Clean Architecture & SOLID** | 09 | AI không thể architect hệ thống cho bạn |
| 9 | **Performance: const, rebuild, DevTools** | 13 | AI KHÔNG biết app bạn lag ở đâu |
| 10 | **Auth Token & Security** | 11-12 | Sai = security breach. Không chấp nhận AI guess |

### 🤖 Top 10 — AI hỗ trợ rất tốt (tập trung review thay vì viết tay)

| # | Chủ đề | Buổi | Tại sao AI làm tốt |
|---|--------|:---:|-----------|
| 1 | JSON Serialization | 11 | Mechanical mapping — AI gen model class hoàn chỉnh |
| 2 | freezed Immutable Classes | 10 | Pattern-based — AI gen + build_runner config chuẩn |
| 3 | Code Generation (@riverpod) | 07, 10 | Annotation-based — AI rất chuẩn với declarative code |
| 4 | CRUD Storage (Hive, SQLite) | 12 | API đơn giản, AI viết CRUD operations tốt |
| 5 | GitHub Actions YAML | 16 | Template-based — AI viết CI/CD workflow chuẩn |
| 6 | Fastlane Scripts | 16 | Well-documented — AI generate lane scripts chính xác |
| 7 | Layout Code (Row/Column) | 04 | Visual → code — AI translate UI design tốt |
| 8 | Animation Code | 14 | API declarative — AI viết AnimatedX, Tween tốt |
| 9 | Platform Plugin Config | 15 | Config-heavy — permission setup, camera config |
| 10 | Retrofit API Client | 11 | Pattern matching — AI generate type-safe API layer |

---

## 👥 Đối tượng & Điều kiện

| Tiêu chí | Yêu cầu |
|---|---|
| Vị trí hiện tại | Frontend Developer (React hoặc Vue) |
| Kinh nghiệm | ≥ 1 năm làm việc với SPA framework |
| JavaScript / TypeScript | Kiểu dữ liệu, async/await, class, generic |
| React hoặc Vue | Component lifecycle, props/state, hooks hoặc Composition API |
| HTML / CSS | Flexbox, Grid (ánh xạ sang Flutter Layout) |
| Git | Branch, merge, pull request workflow |
| Dart / Flutter | **Không yêu cầu** — Tuần 1 cover từ đầu |

---

## 📚 Mô hình học tập

| Thông số | Chi tiết |
|---|---|
| Tổng thời gian | **8 tuần** |
| Số buổi | **16 buổi** |
| Tần suất | **2 buổi / tuần** |
| Thời lượng mỗi buổi | **90 phút** (sync) |
| Hình thức | **Peer-to-peer** — học viên luân phiên trình bày |
| Presenter | **Rotating** — mỗi buổi 1–2 người present |

### Quy trình mỗi buổi

1. **Presenter** đọc tài liệu + chuẩn bị demo trước buổi học
2. **Trình bày** nội dung chính (45–60 phút)
3. **Live coding / Demo** cùng nhóm (15–20 phút)
4. **Q&A + Thảo luận** so sánh với React/Vue (10–15 phút)

### Cấu trúc tài liệu mỗi buổi (5 file)

```
buoi-XX/
├── 00-overview.md     ← Tổng quan + mục tiêu + đánh giá ưu tiên
├── 01-ly-thuyet.md    ← Lý thuyết chi tiết kèm so sánh React/Vue + AI prompts
├── 02-vi-du.md        ← 5+ code ví dụ hoàn chỉnh, chạy được
├── 03-thuc-hanh.md    ← 3+ bài tập (⭐ → ⭐⭐⭐) + câu hỏi thảo luận
└── 04-tai-lieu-tham-khao.md ← Link tham khảo + AI Prompt Library
```

> Mỗi file lý thuyết và ví dụ đều có **priority icon** (🔴🟡🟢) trên từng heading — scan nhanh để biết mục nào cần đầu tư, mục nào để AI hỗ trợ.

---

## 📅 Lộ trình 16 buổi

| Tuần | Buổi | Chủ đề | Nội dung chính | AI Focus |
|---|---|---|---|---|
| **Tuần 1** | 01 | Giới thiệu Dart & Flutter | Dart syntax, types, functions — so sánh TypeScript | Setup AI tools, prompt basics |
| | 02 | Dart nâng cao | OOP, generics, async/await, Stream, null safety | AI giải thích concept, debug errors |
| **Tuần 2** | 03 | Widget Tree | StatelessWidget, StatefulWidget, BuildContext, lifecycle | AI gen widget boilerplate |
| | 04 | Layout System | Row, Column, Stack, Flex — so sánh Flexbox/Grid | AI translate UI design → Flutter code |
| **Tuần 3** | 05 | Navigation & Routing | Navigator 2.0, GoRouter, deep linking | AI gen route config, guard logic |
| | 06 | State Management cơ bản | setState, InheritedWidget, Provider | Hiểu concept trước — AI hỗ trợ sau |
| **Tuần 4** | 07 | Riverpod | Provider types, ref, family, autoDispose | AI gen provider + state class |
| | 08 | BLoC Pattern | Cubit, Bloc, Event/State, BlocBuilder | AI gen Bloc scaffold |
| **Tuần 5** | 09 | Clean Architecture | Layer separation, domain/data/presentation, use cases | AI gen architecture scaffold |
| | 10 | DI & Testing | get_it, injectable, unit test, widget test, mockito | AI gen test cases + mock |
| **Tuần 6** | 11 | Networking | Dio, Retrofit, interceptors, error handling | AI gen API client + models |
| | 12 | Local Storage | SharedPreferences, Hive, SQLite, secure storage | AI gen CRUD + migration |
| **Tuần 7** | 13 | Performance | DevTools, profiling, const widgets, lazy loading | DevTools tự dùng — AI explain bottleneck |
| | 14 | Animation | Implicit/Explicit animation, Hero, Lottie, Rive | AI gen animation code |
| **Tuần 8** | 15 | Platform Integration | Method channels, camera, permissions | AI gen platform config |
| | 16 | CI/CD & Production | Flavors, Fastlane, GitHub Actions, store deploy | AI gen CI/CD pipeline |

---

## 🏆 Mục tiêu đầu ra

Sau 8 tuần, bạn có thể:

**Flutter Development**
- [ ] Viết ứng dụng Flutter hoàn chỉnh với navigation, state management, networking
- [ ] Áp dụng Clean Architecture + Dependency Injection trong dự án thực tế
- [ ] Sử dụng thành thạo Riverpod hoặc BLoC cho state management
- [ ] Viết unit test, widget test, và integration test
- [ ] Tối ưu performance và sử dụng DevTools để profile
- [ ] Tích hợp native platform (camera, storage, permissions)
- [ ] Cấu hình CI/CD pipeline và release production build

**AI-Augmented Development**
- [ ] Viết prompt có Context & Constraints để AI gen code 85%+ production-ready
- [ ] Review AI-generated code theo production checklist — catch lỗi AI hay mắc
- [ ] Biết khi nào nhờ AI, khi nào tự code — **AI-aware developer**
- [ ] Dùng AI tools (Copilot, Cursor, Claude) như force multiplier, không phải crutch

**Đánh giá:** Đạt tiêu chuẩn [Middle Mobile Developer](tieu-chuan/middle-level-rubric.md) — tự triển khai feature end-to-end.

---

## ⚙️ Cài đặt môi trường phát triển

### 1. VS Code (IDE chính)

VS Code là IDE chính cho khoá học — nhẹ, quen thuộc với Frontend developer, và hỗ trợ Flutter rất tốt.

```bash
# Tải VS Code tại: https://code.visualstudio.com/
# Hoặc qua Homebrew (macOS):
brew install --cask visual-studio-code
```

### 2. Flutter SDK

```bash
# macOS — cài qua Homebrew (khuyến nghị):
brew install --cask flutter

# Hoặc tải thủ công:
# https://docs.flutter.dev/get-started/install/macos

# Thêm Flutter vào PATH (nếu cài thủ công):
# Mở ~/.zshrc hoặc ~/.bashrc và thêm:
export PATH="$HOME/development/flutter/bin:$PATH"

# Reload terminal:
source ~/.zshrc
```

Kiểm tra version:

```bash
flutter --version
# Kết quả mong đợi: Flutter 3.x.x • channel stable
```

### 3. VS Code Extensions

Mở VS Code → Extensions (`Cmd + Shift + X`) → tìm và cài đặt:

| Extension | ID | Mô tả |
|---|---|---|
| **Dart** | `Dart-Code.dart-code` | Syntax highlighting, IntelliSense, debugging cho Dart |
| **Flutter** | `Dart-Code.flutter` | Hot reload, device selector, widget inspector |

```bash
# Hoặc cài qua terminal:
code --install-extension Dart-Code.dart-code
code --install-extension Dart-Code.flutter
```

### 4. iOS Simulator (macOS)

```bash
# Bước 1: Cài Xcode từ App Store
# https://apps.apple.com/us/app/xcode/id497799835

# Bước 2: Chấp nhận license
sudo xcodebuild -license accept

# Bước 3: Cài Xcode Command Line Tools
sudo xcode-select --install

# Bước 4: Mở iOS Simulator
open -a Simulator
```

> 📱 Simulator sẽ xuất hiện — chọn device trong **File → Open Simulator → iPhone 15 Pro** (hoặc bất kỳ model nào).

### 5. Android Emulator

```bash
# Bước 1: Tải Android Studio
# https://developer.android.com/studio

# Bước 2: Mở Android Studio → More Actions → Virtual Device Manager (AVD Manager)
# Bước 3: Create Virtual Device → chọn Pixel 7 → chọn API level mới nhất → Finish
# Bước 4: Nhấn ▶️ để khởi chạy emulator

# Bước 5: Chấp nhận Android licenses
flutter doctor --android-licenses
```

> ⚠️ Sau khi tạo xong emulator, **không cần mở Android Studio nữa**. VS Code sẽ tự detect emulator đang chạy. Chỉ dùng Android Studio cho việc quản lý AVD.

### 6. Kiểm tra toàn bộ — `flutter doctor`

```bash
flutter doctor
```

Kết quả mong đợi — tất cả đều ✅:

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x)
[✓] Android toolchain
[✓] Xcode
[✓] Chrome
[✓] Android Studio
[✓] VS Code
[✓] Connected device
[✓] Network resources
```

> Nếu có mục nào ❌, chạy `flutter doctor -v` để xem chi tiết lỗi và hướng xử lý.

### 7. Chọn device trong VS Code

1. Mở Flutter project trong VS Code
2. Nhìn **góc dưới bên phải** status bar → click vào tên device
3. Chọn device muốn chạy: iOS Simulator, Android Emulator, Chrome, hoặc macOS
4. Hoặc dùng Command Palette (`Cmd + Shift + P`) → `Flutter: Select Device`

---

## ▶️ Cách chạy các loại bài tập

| Loại bài tập | Lệnh / Cách chạy | Khi nào dùng |
|---|---|---|
| **Dart CLI** (console app) | `dart run lib/main.dart` | Bài tập Dart thuần — tuần 1 |
| **Flutter app** | `flutter run` hoặc nhấn `F5` trong VS Code | Bài tập UI — từ tuần 2 trở đi |
| **Widget test** | `flutter test` | Chạy tất cả unit/widget test |
| **Widget test (1 file)** | `flutter test test/widget_test.dart` | Chạy 1 file test cụ thể |
| **Integration test** | `flutter test integration_test/` | Test end-to-end trên device/emulator |
| **DartPad** (online) | Mở [dartpad.dev](https://dartpad.dev) | Thử nhanh code Dart/Flutter không cần IDE |

> 💡 **Hot Reload**: Khi app đang chạy, nhấn `r` trong terminal hoặc `Cmd + S` trong VS Code để hot reload — thay đổi hiện ngay lập tức mà không mất state.

---

## 📁 Cấu trúc thư mục

```
flutter-training/
├── README.md                        ← 📍 Bạn đang ở đây
├── ai-toolkit/                      ← Hướng dẫn AI-driven development
│   └── ai-driven-development.md
├── project-mau/                     ← Kiến trúc tham chiếu cho dự án thực hành
│   └── reference-architecture.md
├── tieu-chuan/                      ← Tiêu chuẩn đánh giá
│   └── middle-level-rubric.md
├── tuan-01-dart-co-ban/             ← Tuần 1: Dart cơ bản
│   ├── buoi-01-gioi-thieu-dart-flutter/
│   │   ├── 00-overview.md           ← Tổng quan buổi học
│   │   ├── 01-ly-thuyet.md          ← Lý thuyết chi tiết + AI prompts
│   │   ├── 02-vi-du.md              ← Code ví dụ
│   │   ├── 03-thuc-hanh.md          ← Bài tập thực hành (4 bước AI workflow)
│   │   └── 04-tai-lieu-tham-khao.md ← Tài liệu tham khảo + AI Prompt Library
│   └── buoi-02-dart-nang-cao/
│       └── (cùng cấu trúc 5 file)
├── tuan-02-widget-fundamentals/     ← Tuần 2: Widget & Layout
│   ├── buoi-03-widget-tree-co-ban/
│   └── buoi-04-layout-system/
├── tuan-03-navigation-state-co-ban/ ← Tuần 3: Navigation & State cơ bản
│   ├── buoi-05-navigation-routing/
│   └── buoi-06-state-management-co-ban/
├── tuan-04-state-management-nang-cao/ ← Tuần 4: State Management nâng cao
│   ├── buoi-07-riverpod/
│   └── buoi-08-bloc-pattern/
├── tuan-05-architecture-di/         ← Tuần 5: Architecture & DI
│   ├── buoi-09-clean-architecture/
│   └── buoi-10-di-testing/
├── tuan-06-networking-data/         ← Tuần 6: Networking & Data
│   ├── buoi-11-networking/
│   └── buoi-12-local-storage/
├── tuan-07-performance-animation/   ← Tuần 7: Performance & Animation
│   ├── buoi-13-performance/
│   └── buoi-14-animation/
├── tuan-08-platform-integration-production/ ← Tuần 8: Platform & Production
│   ├── buoi-15-platform-integration/
│   └── buoi-16-cicd-production/
└── van-hanh-nhom/                   ← Hướng dẫn vận hành nhóm học
    └── study-group-operations.md
```

---

## 📖 Cách đọc tài liệu

### Thứ tự đọc khuyến nghị

1. **README.md** (file này) — nắm tổng quan chương trình và cài đặt môi trường
2. **van-hanh-nhom/study-group-operations.md** — hiểu cách vận hành nhóm học peer-to-peer
3. **tieu-chuan/middle-level-rubric.md** — xem tiêu chuẩn đánh giá Middle Developer
4. **ai-toolkit/ai-driven-development.md** — setup AI tools và học workflow
5. **tuan-XX/buoi-YY/00-overview.md** — đọc overview trước mỗi buổi học
6. **tuan-XX/buoi-YY/01 → 02 → 03 → 04** — đọc tuần tự theo lịch

### Quy ước trong tài liệu

| Ký hiệu | Ý nghĩa |
|---|---|
| 🔄 **React/Vue ↔ Flutter** | Phần so sánh với framework Frontend |
| 🔴🟡🟢 **Priority** | Mức ưu tiên: MUST / SHOULD / AI-OK |
| 💡 **Tip** | Mẹo hay, best practice |
| ⚠️ **Chú ý** | Điểm dễ sai, cần lưu ý |
| 🏋️ **Bài tập** | Phần thực hành |
| 📱 **Demo** | Code chạy được trên device/emulator |

### Format AI blocks trong tài liệu

Mỗi file lý thuyết mở đầu bằng **AI learning guide** — hướng dẫn cách dùng AI cho từng mức priority (🔴🟡🟢) kèm prompt mẫu. Ví dụ:

> - 🔴 **Tự code tay:** Prompt mẫu: _"Giải thích [concept]. Background: React/Vue."_
> - 🟡 **Code cùng AI:** Prompt mẫu: _"Tạo [component] theo [pattern], có [constraint]."_
> - 🟢 **AI gen → Review:** Prompt mẫu: _"Generate [feature] theo [architecture]. Bao gồm error handling."_

Khi gặp các block này, đọc kỹ prompt mẫu — chúng được thiết kế để bạn copy-paste và customize cho task thực tế.

---

## 📎 Tài liệu bổ sung

| Tài liệu | Mô tả |
|---|---|
| [Tiêu chuẩn Middle Developer](tieu-chuan/middle-level-rubric.md) | Rubric đánh giá — biết mình cần đạt gì |
| [AI-Driven Development](ai-toolkit/ai-driven-development.md) | Setup AI tools + workflow chi tiết |
| [Reference Architecture](project-mau/reference-architecture.md) | Kiến trúc tham chiếu cho dự án Flutter thực tế |
| [Vận hành nhóm học](van-hanh-nhom/study-group-operations.md) | Hướng dẫn tổ chức peer-to-peer learning |

---

## 🔗 Tài liệu tham khảo

### Flutter & Dart chính thức

- [Flutter Documentation](https://docs.flutter.dev/) — Tài liệu chính thức Flutter
- [Dart Documentation](https://dart.dev/guides) — Hướng dẫn ngôn ngữ Dart
- [Flutter API Reference](https://api.flutter.dev/) — API reference đầy đủ
- [DartPad](https://dartpad.dev/) — Thử code Dart/Flutter online
- [Flutter Widget Catalog](https://docs.flutter.dev/ui/widgets) — Danh mục tất cả widget
- [Flutter Cookbook](https://docs.flutter.dev/cookbook) — Recipes cho các tác vụ phổ biến
- [Dart Language Tour](https://dart.dev/language) — Tour tổng quan ngôn ngữ Dart

### Packages quan trọng trong khoá học

- [Riverpod](https://riverpod.dev/) — State management
- [BLoC](https://bloclibrary.dev/) — State management pattern
- [GoRouter](https://pub.dev/packages/go_router) — Declarative routing
- [Dio](https://pub.dev/packages/dio) — HTTP client
- [get_it](https://pub.dev/packages/get_it) — Dependency Injection
- [Hive](https://pub.dev/packages/hive) — Local database

### Dành cho Frontend developer chuyển sang Flutter

- [Flutter for Web Developers](https://docs.flutter.dev/get-started/flutter-for/web-devs) — So sánh HTML/CSS ↔ Flutter
- [Flutter for React Native Developers](https://docs.flutter.dev/get-started/flutter-for/react-native-devs) — So sánh React Native ↔ Flutter

---

<p align="center">
  <strong>85 files · 16 buổi · 8 tuần · AI-Integrated · Peer-to-Peer</strong><br/>
  <em>Bạn đã là Frontend Developer. Giờ hãy trở thành Mobile Developer.</em>
</p>
