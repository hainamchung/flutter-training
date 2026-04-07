# Module 16: Lint & Code Quality — Verification

## VERIFY — Kiểm tra sau khi hoàn thành

---

## 🧑‍💻 Self-Check Checklist

Đánh dấu ✅ mỗi câu bạn trả lời được. Cần đạt target để pass.

### 🔴 MUST-KNOW (3 câu — 100% required)

| # | Câu hỏi | Trả lời được | Ghi chú |
|---|---------|--------------|---------|
| 1 | Config được `analysis_options.yaml` với include, analyzer, linter sections | ☐ | |
| 2 | Giải thích được sự khác nhau giữa error, warning, info severity | ☐ | |
| 3 | Enable/disable được specific lint rules | ☐ | |

### 🟡 SHOULD-KNOW (4 câu — ≥3 required)

| # | Câu hỏi | Trả lời được | Ghi chú |
|---|---------|--------------|---------|
| 4 | Hiểu được 5+ key lint rules (prefer_const, avoid_print, etc.) | ☐ | |
| 5 | Setup và enable được custom_lint rules | ☐ | |
| 6 | Restart được Analysis Server khi cần | ☐ | |
| 7 | Tạo được custom lint rule trong super_lint package | ☐ | |

### 🟢 AI-GENERATE (2 câu — ≥1 required)

| # | Câu hỏi | Trả lời được | Ghi chú |
|---|---------|--------------|---------|
| 8 | Understand Effective Dart guidelines | ☐ | |
| 9 | Integrate custom lint vào CI/CD | ☐ | |

---

## 🎯 Pass Criteria

| Category | Required | Notes |
|----------|----------|-------|
| 🔴 MUST-KNOW | **3/3** | Không được miss bất kỳ câu nào |
| 🟡 SHOULD-KNOW | **≥3/4** | Ít nhất 3 câu |
| 🟢 AI-GENERATE | **≥1/2** | Ít nhất 1 câu |
| **Total** | **≥7/9** | Tổng cộng |

---

## 🔍 Cross-Check với Codebase

### Check 1: super_lint Structure

```bash
# Kiểm tra super_lint package structure
ls -la base_flutter/super_lint/lib/src/
ls -la base_flutter/super_lint/lib/src/lints/
ls -la base_flutter/super_lint/lib/src/base/
```

**Expected:**
- Có `base/` directory với `common_lint_rule.dart`, `common_quick_fix.dart`, `rule_config.dart`
- Có `lints/` directory với nhiều lint rules
- `index.dart` export tất cả rules

### Check 2: CommonLintRule Pattern

Đọc file `super_lint/lib/src/base/common_lint_rule.dart`:

```bash
cat base_flutter/super_lint/lib/src/base/common_lint_rule.dart
```

**Expected pattern:**
- Class kế thừa `DartLintRule`
- Có generic parameter `<T extends CommonLintParameter>`
- Method chính là `check()` (KHÔNG PHẢI `run()`)
- Có `RuleConfig<T> config`

### Check 3: RuleConfig Structure

Đọc file `super_lint/lib/src/base/rule_config.dart`:

```bash
cat base_flutter/super_lint/lib/src/base/rule_config.dart
```

**Expected:**
- `RuleConfig` class với generic `<T extends CommonLintParameter>`
- Constructor nhận `name`, `configs`, `problemMessage`, `paramsParser`
- Có properties: `name`, `enabled`, `parameters`, `lintCode`

### Check 4: CommonQuickFix Pattern

Đọc file `super_lint/lib/src/base/common_quick_fix.dart`:

```bash
cat base_flutter/super_lint/lib/src/base/common_quick_fix.dart
```

**Expected:**
- Class kế thừa `DartFix`
- Có generic parameter `<T extends CommonLintParameter>`
- Có `RuleConfig<T> config`

### Check 5: Example Implementation

Đọc `super_lint/lib/src/lints/avoid_using_datetime_now.dart`:

```bash
cat base_flutter/super_lint/lib/src/lints/avoid_using_datetime_now.dart
```

**Expected structure:**
```dart
class AvoidUsingDateTimeNow extends CommonLintRule<_Parameter> {
  AvoidUsingDateTimeNow(CustomLintConfigs configs) : super(
    RuleConfig(
      name: 'avoid_using_datetime_now',
      configs: configs,
      paramsParser: _Parameter.fromMap,
      problemMessage: (_) => '...',
    ),
  );

  @override
  Future<void> check(...) async { ... }

  @override
  List<Fix> getFixes() => [_Fix(config)];
}
```

### Check 6: Verify Exercise Implementation

```bash
# Kiểm tra rule mới đã được export
grep "prefer_const_list" base_flutter/super_lint/lib/src/index.dart

# Kiểm tra rule đã được register
grep "PreferConstList" base_flutter/super_lint/lib/super_lint.dart

# Kiểm tra rule đã được enable
grep "prefer_const_list" base_flutter/analysis_options.yaml
```

---

## 🏋️ Practical Test

### Test 1: Analyze Existing Lint Rule (15 phút)

1. Đọc `super_lint/lib/src/lints/avoid_using_datetime_now.dart`
2. Trace flow: constructor → RuleConfig → check() → getFixes()
3. Xác định 3 thành phần chính và giải thích vai trò

### Test 2: Create Custom Lint Rule (30 phút)

1. Tạo `PreferConstList` rule theo `CommonLintRule<T>` pattern
2. Implement `check()` method (KHÔNG dùng `run()`)
3. Implement `getFixes()` trả về list of Fix
4. Đăng ký rule trong `super_lint.dart`
5. Verify rule hoạt động với sample code

### Test 3: Quick Fix Implementation (15 phút)

1. Thêm `CommonQuickFix<T>` subclass cho rule đã tạo
2. Implement `run()` method với ChangeReporter
3. Verify quick fix xuất hiện trong IDE

---

## 📊 Score Sheet

| Section | Score | Max | Notes |
|---------|-------|-----|-------|
| 🔴 MUST-KNOW | __/3 | 3 | |
| 🟡 SHOULD-KNOW | __/4 | 4 | |
| 🟢 AI-GENERATE | __/2 | 2 | |
| **Total** | **__/9** | 9 | |

**Pass threshold:** ≥7/9 với 3/3 🔴

---

## 🚩 Nếu chưa đạt

Quay lại và ôn lại:

| Missed | Resource | Action |
|--------|----------|--------|
| CommonLintRule pattern | [01-code-walk.md](./01-code-walk.md) | Đọc phần super_lint structure |
| check() vs run() | [01-code-walk.md](./01-code-walk.md) | Hiểu method override đúng |
| RuleConfig | [01-code-walk.md](./01-code-walk.md) | Đọc rule_config.dart |
| Custom lint creation | [03-exercise.md](./03-exercise.md) | Làm Bài 3 |

---

## ✅ Khi đã pass

1. **Commit exercises** lên branch `feature/m16-lint-quality`
2. **Update README.md** — thêm notes về lint rules convention
3. **Sang Module 17:** [Codebase Architecture & DI](../module-17-architecture-di/)

---

<!-- AI_VERIFY: generation-complete -->
