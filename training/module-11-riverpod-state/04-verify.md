# Verify — Riverpod & State Management

> Self-assessment checklist và exercise verification.

---

## 📋 Self-Assessment Checklist

Trả lời từng câu hỏi. Nếu câu nào không trả lời được → quay lại [02-concept.md](./02-concept.md).

### ProviderScope & Container

- [ ] **1.** Giải thích được `ProviderScope` tạo `ProviderContainer` ẩn, vai trò `overrides` cho testing?
- [ ] **2.** Trình bày được container lazy-create behavior: declaration ≠ creation.

### Provider Types

- [ ] **3.** Chọn đúng provider type (Provider / StateProvider / StateNotifierProvider / FutureProvider / StreamProvider) trong 10 giây cho bất kỳ use case?
- [ ] **4.** Phân biệt được mutable vs read-only providers.
- [ ] **5.** Giải thích được khi nào dùng `.autoDispose` vs no modifier.

### ref API

- [ ] **6.** Phân biệt `ref.read` (one-shot) / `ref.watch` (rebuild) / `ref.listen` (side-effect) / `ref.select` (granular)?
- [ ] **7.** Giải thích được tại sao `ref.watch` KHÔNG được dùng trong callbacks — crash vs warning.
- [ ] **8.** Viết được code dùng `.select()` để tối ưu rebuild.

### Provider Patterns

- [ ] **9.** Viết được DI bridge pattern: `Provider((ref) => getIt.get<T>())`?
- [ ] **10.** Giải thích được `ProviderContainer` cho manual container management?
- [ ] **11.** Mô tả được 4 hooks của `ProviderObserver`: didAdd, didUpdate, didDispose, didFail?

### Advanced Patterns

- [ ] **12.** Khi nào dùng `FutureProvider` vs `StateNotifierProvider`?
- [ ] **13.** Khi nào dùng `StreamProvider` cho real-time data?
- [ ] **14.** Viết được testing override với `overrideWithValue`?

### Decision Tree

- [ ] **15.** Vẽ được decision tree chọn provider type cho: theme, form, API fetch, chat, navigation.
- [ ] **16.** Phân biệt shared state (app-wide) vs page state (autoDispose)?
- [ ] **17.** Giải thích được tại sao page VM luôn dùng `.autoDispose`.

---

## ✅ Exercise Verification

### Exercise 1: Provider Lifecycle Trace

**Yêu cầu:** Hoàn thành timeline diagram.

**Checklist:**
- [ ] `didAddProvider` fires khi first `ref.watch` called
- [ ] `didUpdateProvider` fires khi state changes
- [ ] `didDisposeProvider` fires khi no more listeners
- [ ] State wiped khi disposed

### Exercise 2: Classify Provider Types

**Yêu cầu:** Phân loại 10 providers.

**Đáp án:**

| # | Type |
|---|------|
| 1 | Provider |
| 2 | StateProvider |
| 3 | StateNotifierProvider.autoDispose |
| 4 | FutureProvider |
| 5 | StreamProvider |
| 6 | StateNotifierProvider |
| 7 | Provider |
| 8 | StateNotifierProvider.autoDispose |
| 9 | StateNotifierProvider.autoDispose |
| 10 | Provider |

### Exercise 3: SettingsViewModel

**Yêu cầu:** Tạo complete Settings implementation.

**Checklist:**
- [ ] SettingsState với 4 fields
- [ ] SettingsViewModel với setters + load/save methods
- [ ] StateNotifierProvider.autoDispose
- [ ] runCatching với actionName
- [ ] SettingsPage dùng selectors

### Exercise 4: Selector Optimization

**Yêu cầu:** Hiểu performance impact.

**Checklist:**
- [ ] Giải thích được: without selector = rebuild entire widget
- [ ] Giải thích được: with selector = rebuild only affected widget
- [ ] Tính được rebuild counts cho scenario

### Exercise 5: AI Review

**Yêu cầu:** Complete AI Prompt Dojo.

**Checklist:**
- [ ] Prompt được execute
- [ ] 5+ issues identified
- [ ] Refactored code provided
- [ ] Unit test skeleton provided

---

## 🎯 Completion Criteria

Để hoàn thành module này:

| Criteria | Điều kiện |
|----------|-----------|
| **Core Understanding** | Trả lời được 14/17 câu trong checklist |
| **Practical Skill** | Hoàn thành Exercise 1-4 |
| **Advanced Exploration** | Hoàn thành Exercise 5 |
| **Code Reading** | Đọc và hiểu tất cả files trong [🔗 Liên kết](../module-11-riverpod-state/00-overview.md#-liên-kết) |

---

## 🔄 Next Module Prep

Sau khi hoàn thành Module 11:

### Đọc trước

1. [Module 12 — Data Layer](../module-12-data-layer/):
   - HookConsumerWidget deep-dive
   - Built-in hooks: useState, useEffect, useCallback
   - Custom hooks composition
   - StatefulHookConsumerWidget

### Chuẩn bị questions cho mentor

1. Khi nào dùng `FutureProvider` vs `StateNotifierProvider` cho async data?
2. Làm sao để test provider behavior không cần widget?
3. Tại sao cần cả `Provider` và `StateProvider` khi cả hai đều read-only?

---

## 📖 Glossary Terms

| Term | Definition |
|------|------------|
| **ProviderScope** | Widget tạo ProviderContainer cho toàn bộ app |
| **ProviderContainer** | Hidden in-memory store chứa tất cả provider instances |
| **autoDispose** | Modifier tự dispose provider khi không còn listeners |
| **family** | Modifier tạo parameterized provider (1 provider per parameter value) |
| **ref.watch** | Subscribe + rebuild widget khi value thay đổi |
| **ref.read** | One-shot access — không subscribe |
| **ref.listen** | Subscribe + callback — cho side effects |
| **ref.select** | Granular subscription — chỉ rebuild khi selected value đổi |
| **overrideWith** | Testing pattern để inject mock providers |

---

## 🏆 Module Achievement

Sau khi hoàn thành checklist + exercises:

```
┌─────────────────────────────────────────────────────┐
│           MODULE 11 COMPLETE ✅                     │
│                                                     │
│  Concepts Mastered:                                 │
│  ├── Provider Types Taxonomy                     │
│  ├── ref API (read/watch/listen/select)          │
│  ├── autoDispose & family                        │
│  ├── DI Bridge Pattern                            │
│  ├── ProviderContainer                           │
│  ├── ProviderObserver                            │
│  ├── FutureProvider & StreamProvider             │
│  └── Testing Overrides                           │
│                                                     │
│  Next: Module 12 — Data Layer                │
└─────────────────────────────────────────────────────┘
```

---

**Tiếp theo:** [Module 12 — Data Layer](../module-12-data-layer/)

<!-- AI_VERIFY: generation-complete -->
