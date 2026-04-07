# Module 16: Lint & Code Quality — Exercises

## PRACTICE — Làm tay trên codebase thật

---

## Bài 1: Analyze Existing Lint Config ⭐

**Mục tiêu:** Hiểu current lint configuration.

**Hướng dẫn:**

1. Mở `analysis_options.yaml` trong `base_flutter/`
2. Đọc và phân loại các rules:
   - Style rules
   - Best practices rules
   - Error prevention rules
3. Count số lượng enabled rules
4. Identify rules được disabled (`rule: false`)

**Deliverable:**
- Bảng phân loại 10 rules theo categories
- Số lượng enabled vs disabled rules
- 3 rules bạn muốn enable/disable và tại sao

**Checklist:**
- [ ] Đọc full analysis_options.yaml
- [ ] Phân loại được rules
- [ ] Count enabled/disabled
- [ ] Giải thích được tại sao chọn 3 rules

---

## Bài 2: Enable/Disable Specific Rules ⭐

**Mục tiêu:** Thực hành modify lint configuration.

**Scenario:** Team muốn enable thêm rules để improve code quality.

**Hướng dẫn:**

1. Thêm các rules sau vào `analysis_options.yaml`:

```yaml
linter:
  rules:
    # Thêm các rules mới
    - avoid_function_literals_in_foreach_calls
    - avoid_return_types_on_setters
    - avoid_setters_without_backers
    - cascade_invocations
    - collection_methods_unrelated_type
    - combinators_ordering
    - conditional_uri_does_not_exist
    - deprecated_member_use_from_same_package
    - empty_statements
    - unnecessary_await_in_return
```

2. Run `flutter analyze` để xem có violations không
3. Fix hoặc disable rules nếu cần

**Bonus:** Tạo script để count violations:

```bash
flutter analyze --machine | grep -c "error\|warning"
```

**Checklist:**
- [ ] Thêm 10 rules mới
- [ ] Run flutter analyze
- [ ] Identify violations
- [ ] Fix hoặc disable rules

---

## Bài 3: Create Custom Lint Rule với CommonLintRule Pattern ⭐⭐

**Mục tiêu:** Tạo custom lint rule theo đúng pattern của super_lint framework.

**Scenario:** Team muốn enforce convention: không dùng `DateTime.now()` trực tiếp, phải dùng `DateTimeUtil.now`.

**Hướng dẫn:**

### Part A — Tạo Rule File

Tạo file mới theo pattern có sẵn trong `super_lint/lib/src/lints/`:

1. Đọc file reference để hiểu pattern:
   - `super_lint/lib/src/lints/avoid_using_datetime_now.dart`
   - `super_lint/lib/src/base/common_lint_rule.dart`
   - `super_lint/lib/src/base/common_quick_fix.dart`
   - `super_lint/lib/src/base/rule_config.dart`

2. Tạo file `super_lint/lib/src/lints/prefer_const_list.dart`:

```dart
import '../index.dart';

class PreferConstList extends CommonLintRule<_PreferConstListParameter> {
  PreferConstList(CustomLintConfigs configs) : super(
    RuleConfig(
      name: 'prefer_const_list',
      configs: configs,
      paramsParser: _PreferConstListParameter.fromMap,
      problemMessage: (_) =>
          'Use const keyword for lists that are never modified.',
    ),
  );

  @override
  Future<void> check(
    CustomLintResolver resolver,
    ErrorReporter reporter,
    CustomLintContext context,
    String rootPath,
  ) async {
    // TODO: Implement lint logic
    // 1. Kiểm tra node là ListLiteral
    // 2. Check xem có const keyword chưa
    // 3. Nếu chưa có và list được khởi tạo rỗng hoặc với literal values
    // 4. Report violation
    context.registry.addListLiteral((node) {
      if (!node.constKeyword.present) {
        reporter.atNode(node, code);
      }
    });
  }

  @override
  List<Fix> getFixes() => [
    _PreferConstListFix(config),
  ];
}
```

3. Tạo Quick Fix class:

```dart
class _PreferConstListFix extends CommonQuickFix<_PreferConstListParameter> {
  _PreferConstListFix(super.config);

  @override
  Future<void> run(
    CustomLintResolver resolver,
    ChangeReporter reporter,
    CustomLintContext context,
    AnalysisError analysisError,
    List<AnalysisError> others,
  ) async {
    context.registry.addListLiteral((node) {
      if (!node.sourceRange.intersects(analysisError.sourceRange)) return;
      if (!node.constKeyword.present) {
        final changeBuilder = reporter.createChangeBuilder(
          message: 'Add const keyword',
          priority: 70,
        );
        changeBuilder.addDartFileEdit((builder) {
          builder.addSimpleInsertion(node.offset, 'const ');
        });
      }
    });
  }
}
```

4. Tạo Parameter class:

```dart
class _PreferConstListParameter extends CommonLintParameter {
  const _PreferConstListParameter({
    super.excludes,
    super.includes,
    super.severity,
  });

  static _PreferConstListParameter fromMap(Map<String, dynamic> map) {
    return _PreferConstListParameter(
      excludes: safeCastToListString(map['excludes']),
      includes: safeCastToListString(map['includes']),
      severity: convertStringToErrorSeverity(map['severity']),
    );
  }
}
```

### Part B — Register Rule

1. Export trong `super_lint/lib/src/index.dart`:
```dart
export 'lints/prefer_const_list.dart';
```

2. Register trong `super_lint/lib/super_lint.dart`:
```dart
class _SuperLintPlugin extends PluginBase {
  @override
  List<LintRule> getLintRules(CustomLintConfigs configs) => [
    const AvoidUsingDateTimeNow(),
    // ... existing rules ...
    PreferConstList(configs),  // Thêm rule mới
  ];
}
```

3. Enable trong `analysis_options.yaml`:
```yaml
custom_lint:
  rules:
    - prefer_const_list: true
```

### Checklist:
- [ ] Đọc và hiểu pattern từ 4 reference files
- [ ] Implement `check()` method (KHÔNG phải `run()`)
- [ ] Implement `getFixes()` trả về `List<Fix>`
- [ ] Tạo QuickFix kế thừa `CommonQuickFix<T>`
- [ ] Tạo Parameter class kế thừa `CommonLintParameter`
- [ ] Export và register rule
- [ ] Test với sample code

---

## Bài 4: Analyze Existing Lint Rule Implementation ⭐⭐⭐

**Mục tiêu:** Phân tích cách một lint rule thực sự được implement.

**Hướng dẫn:**

1. Đọc `super_lint/lib/src/lints/avoid_using_datetime_now.dart`

2. Trả lời các câu hỏi:
   - Rule kế thừa class nào? (`CommonLintRule<T>`)
   - Method chính để check là gì? (`check()` vs `run()`)
   - `RuleConfig` chứa những gì?
   - QuickFix được implement như thế nào?

3. Trace flow:
   ```
   LintRule created
   → RuleConfig initialized
   → check() called with resolver, reporter, context
   → Registry.addInstanceCreationExpression()
   → Node matches → reporter.atNode()
   → Fixes accessed via getFixes()
   ```

4. So sánh với Exercise 3 — bạn đã implement đúng pattern chưa?

**Deliverable:**
- Bảng phân tích 5 thành phần của `AvoidUsingDateTimeNow`
- Sơ đồ flow của một lint rule từ creation đến reporting

**Checklist:**
- [ ] Đọc full implementation
- [ ] Trả lời 4 câu hỏi phân tích
- [ ] Trace complete flow
- [ ] So sánh với implementation của mình

---

## Bài 5: AI Prompt Dojo — Code Quality Review ⭐⭐⭐

**Mục tiêu:** Viết prompt để AI review code quality.

### ❌ Bad Prompt

```
Review my Flutter code.
```

### ✅ Good Prompt

```
Perform code quality review on my Flutter codebase:

1. **Lint Analysis**
   - Run `flutter analyze` and list all errors/warnings
   - Identify patterns (same error repeated)

2. **Code Style**
   - Check for inconsistent naming (files, classes, variables)
   - Check import organization (should use package: imports)
   - Check for magic numbers/strings that should be constants

3. **Best Practices**
   - Find `print()` statements that should be Log.d()
   - Find missing const constructors
   - Find potential memory leaks (unclosed streams/sinks)

4. **Architecture**
   - Check if files follow Clean Architecture layers
   - Identify circular dependencies
   - Check barrel file usage

5. **Suggestions**
   - Prioritize fixes by impact
   - Provide before/after code examples

Focus on: lib/exception/, lib/common/, lib/data/
Output format: Markdown table with file, line, issue, severity, suggestion
```

### 🎯 Challenge

1. Chạy prompt trên với Claude/Copilot
2. Review output
3. Tạo action plan từ suggestions
4. Fix top 3 issues
5. Run `flutter analyze` lại để verify

**Deliverable:**
- Copy prompt đã dùng
- Issues identified
- Action plan
- Before/after for 3 fixes

---

## 📤 Submit

1. Push code lên branch `feature/m16-lint-quality`
2. Tạo PR với description:
   ```
   ## M16: Lint & Code Quality

   - [ ] Bài 1: Lint config analyzed
   - [ ] Bài 2: Rules enabled/disabled
   - [ ] Bài 3: Custom lint rule created
   - [ ] Bài 4: super_lint package configured
   - [ ] Bài 5: AI review prompt + output
   ```
3. Gửi link PR cho peer review

---

→ Tiếp tục: [04-verify.md](./04-verify.md)

---

## ↩️ Revert Changes

Sau khi hoàn thành mỗi bài tập, revert changes để không ảnh hưởng codebase:

```bash
# Revert tất cả thay đổi trong bài tập
git checkout -- lib/

# Hoặc revert cụ thể file
# git checkout -- lib/path/to/modified/file.dart

# Nếu đã chạy codegen (make gen, make ep):
# 1. Revert barrel/file changes
git checkout -- lib/index.dart

# 2. Chạy lại make để clean
make gen
```

> ⚠️ **Quan trọng:** Luôn revert trước khi chuyển bài tập hoặc trước khi `git commit`. Code của bạn chỉ nên ở trong branch feature, không nên modify các base files trực tiếp.



<!-- AI_VERIFY: generation-complete -->
