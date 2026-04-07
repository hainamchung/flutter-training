# Module 18 – Testing (Unit + Widget + Golden)

## Tổng quan

Module này tập trung vào các kỹ thuật testing: Provider testing, mocking nâng cao với mocktail, golden testing workflow hoàn chỉnh, và CI/CD integration cho tests.

**Depth**: Advanced Deep-Dive — không chỉ đọc hiểu mà còn thực hành các patterns phức tạp.

**Cycle:** CODE (Provider testing patterns) → EXPLAIN (Mocktail advanced usage) → PRACTICE (Write comprehensive tests).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Viết được Provider testing với `ProviderContainer` và `overrideWithValue`?
2. Phân biệt được `when().thenReturn` vs `when().thenAnswer` vs `when().thenThrow` trong mocktail?
3. Config được golden test với `golden_toolkit` và multi-device testing?
4. Hiểu coverage reporting với `lcov` và branch coverage?
5. Setup được test CI/CD integration với `make te` trong pipeline?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 19 — CI/CD](../module-19-cicd/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

## Bạn sẽ học

1. **Provider Testing** — `ProviderContainer`, `overrideWithValue`, `overrideWith`, StateNotifier testing
2. **Mocktail Advanced** — `registerFallbackValue`, `resetMocktailState`, `captureAny`, nested mocks
3. **Golden Testing** — multi-device, auto-update, threshold configuration
4. **Coverage Analysis** — line/branch coverage, `lcov` report, CI integration
5. **Test Organization** — folder mirroring, naming conventions, fast vs slow tests
6. **CI/CD Test Integration** — `make te`, `make cov`, Bitbucket Pipelines integration

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## Kiến thức cần có

| Module | Nội dung | Vai trò trong M18 |
|--------|----------|-------------------|
| **M7** | BaseViewModel | Target chính để Provider testing |
| **M8** | Riverpod providers | Mock pattern: override providers |
| **M18** | Testing basics | Foundation cho advanced patterns |

---

## Cấu trúc files

| File | Nội dung | Thời gian |
|------|----------|-----------|
| [01-code-walk.md](./01-code-walk.md) | Provider testing, mocktail advanced, golden toolkit | 35 min |
| [02-concept.md](./02-concept.md) | 6 concepts: Provider testing, mocktail advanced, golden workflow, coverage, CI integration | 30 min |
| [03-exercise.md](./03-exercise.md) | 4 exercises: ⭐ run → ⭐⭐ Provider test → ⭐⭐ golden → ⭐⭐⭐ CI integration | 90 min |
| [04-verify.md](./04-verify.md) | Verification checklist | 10 min |

---

## 💡 FE Perspective

| Flutter | FE Equivalent |
|---------|---------------|
| `ProviderContainer` (unit test) | React Testing Library `renderHook` with wrapper |
| `ProviderScope.overrides` (widget test) | React Testing Library `MockedProvider` |
| `golden_toolkit` multi-device | Chromatic multi-viewport screenshot |
| `lcov` coverage report | Jest `--coverage` with Istanbul |
| `make te` in CI | `npm test` in GitHub Actions |

---

## Key Files trong Codebase

```
test/
├── common/
│   ├── base_test.dart           ← Global mocks registration
│   ├── test_util.dart           ← createContainer, buildWidget
│   └── test_config.dart         ← Device sizes, locale, theme
├── unit_test/
│   └── ui/page/login/view_model/
│       └── login_view_model_test.dart  ← Provider testing pattern
├── widget_test/
│   └── ui/component/primary_text_field/
│       └── primary_text_field_test.dart  ← Golden test pattern
└── flutter_test_config.dart     ← Global config (golden, fonts, threshold)
```

---

## Forward Reference

→ **M20 (Native Platforms)**: Native debugging cần hiểu platform channels để test native code.
→ **M21 (Firebase)**: Firebase services cần mock trong tests (MockFirebaseAuth, etc.).

<!-- AI_VERIFY: generation-complete -->
