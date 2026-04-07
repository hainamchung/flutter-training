# Module 16: Lint & Code Quality — Concepts

## EXPLAIN — Giải thích patterns từ code

---

## 🔴 Concept 1: analysis_options.yaml Structure

### 📖 Giải thích

`analysis_options.yaml` là configuration file cho Dart analyzer và linter. Nó nằm ở root của project.

### 📦 4 Main Sections

```yaml
include: package:flutter_lints/flutter.yaml    # Include preset

analyzer:                                        # Analyzer config
  exclude: [...]                                 # Skip files
  errors: {...}                                 # Severity levels
  language: {...}                                # Language features

linter:                                         # Linter config
  rules:                                        # Enable/disable rules
    - rule1
    - rule2: false
```

### 🎯 Key Points

| Section | Purpose | Common Use |
|---------|---------|------------|
| `include` | Include preset configs | `package:flutter_lints/flutter.yaml` |
| `analyzer.exclude` | Skip files from analysis | Generated files, build |
| `analyzer.errors` | Override severity | Treat warnings as errors |
| `linter.rules` | Enable/disable lint rules | Project-specific preferences |

### 💡 FE Perspective

| Flutter | JavaScript/TypeScript |
|---------|----------------------|
| `analysis_options.yaml` | `tsconfig.json` + `.eslintrc` |
| `analyzer.exclude` | `exclude` in tsconfig.json |
| `linter.rules` | `.eslintrc` rules |
| `flutter analyze` | `eslint`, `tsc --noEmit` |

---

## 🔴 Concept 2: flutter_lints Configuration

### 📖 Giải thích

`flutter_lints` là preset chứa 100+ recommended lint rules từ Dart team.

### 📦 Rule Severity Levels

| Level | Meaning | Default |
|-------|---------|---------|
| Error | Code won't compile | Yes |
| Warning | Suboptimal code | Yes |
| Info | Style hint | No |

### 📦 Override Severity

```yaml
analyzer:
  errors:
    missing_return: error          # Already error (can't return null)
    prefer_const_constructors: warning  # Downgrade to warning
    avoid_print: ignore            # Disable completely
```

### 💡 FE Perspective

| Flutter | TypeScript |
|---------|------------|
| `flutter_lints/flutter.yaml` | `@typescript-eslint/recommended` |
| Severity override | `eslint --rule 'rule: error'` |
| Disable rule | `eslint-disable-line` comment |

---

## 🔴 Concept 3: Key Lint Rules

### 📖 Giải thích

Top 10 lint rules bạn cần biết:

| Rule | What it catches | Why important |
|------|----------------|---------------|
| `prefer_const_constructors` | `Widget()` → `const Widget()` | Performance |
| `avoid_print` | `print()` statements | Security/logging |
| `prefer_final_locals` | Mutable vars that could be final | Immutability |
| `prefer_const_literals_to_create_immutables` | `['a', 'b']` → `const ['a', 'b']` | Performance |
| `avoid_relative_lib_imports` | `import 'package:app/...'` vs relative | Consistency |
| `sort_pub_dependencies` | Unsorted dependencies | Maintainability |
| `use_key_in_widget_constructors` | Missing `key:` in lists | Performance |
| `cancel_subscriptions` | Unclosed streams | Memory leaks |
| `close_sinks` | Unclosed IO sinks | Resource leaks |
| `prefer_single_quotes` | `"string"` → `'string'` | Style consistency |

### 📦 Examples

```dart
// ❌ Violates prefer_const_constructors
Widget build(BuildContext context) {
  return Container(
    padding: EdgeInsets.all(8),  // Can be const
    child: Text('Hello'),
  );
}

// ✅ Fixed
Widget build(BuildContext context) {
  return Container(
    padding: const EdgeInsets.all(8),
    child: const Text('Hello'),
  );
}
```

### 💡 FE Perspective

| Flutter | ESLint (TypeScript) |
|---------|---------------------|
| `prefer_const_constructors` | `prefer-const` |
| `avoid_print` | `no-console` |
| `prefer_final_locals` | `prefer-const` |
| `avoid_relative_lib_imports` | `no-relative-imports` |

---

## 🟡 Concept 4: custom_lint Setup

### 📖 Giải thích

`custom_lint` cho phép tạo lint rules tùy chỉnh cho project.

### 📦 Setup Steps

1. Add dependencies:
   ```yaml
   dev_dependencies:
     custom_lint: ^0.6.0
   ```

2. Enable in `analysis_options.yaml`:
   ```yaml
   include: package:custom_lint/lint.yaml
   analyzer:
     plugins:
       - custom_lint
   ```

3. Create lint rules in `super_lint/` package

### 💡 FE Perspective

| Flutter | JavaScript |
|---------|------------|
| `custom_lint` | `eslint-plugin-*` packages |
| Create custom rules | `eslint-plugin-mine` |
| Enable in config | `"plugins": ["mine"]` |

---

## 🟡 Concept 5: super_lint Package

### 📖 Giải thích

`super_lint` là custom lint package chứa project-specific rules.

### 📦 Package Structure

```
super_lint/
├── lib/
│   ├── super_lint.dart           # Plugin entry
│   └── lints/
│       ├── rule1.dart
│       └── rule2.dart
├── test/
└── pubspec.yaml
```

### 📦 Rule Template

```dart
import '../index.dart';

class MyCustomRule extends CommonLintRule<_MyCustomRuleParameter> {
  MyCustomRule(CustomLintConfigs configs) : super(
    RuleConfig(
      name: 'my_custom_rule',
      configs: configs,
      paramsParser: _MyCustomRuleParameter.fromMap,
      problemMessage: (_) => 'Description of the problem',
    ),
  );

  @override
  Future<void> run(
    CustomLintResolver resolver,
    ErrorReporter reporter,
    CustomLintContext context,
    String rootPath,
  ) async {
    // 1. Register what to watch (e.g., instance creation)
    context.registry.addInstanceCreationExpression((node) {
      // 2. Check conditions
      // 3. Report if violation found
      reporter.atNode(node, code);
    });
  }

  @override
  List<Fix> getFixes() => [
    _MyCustomRuleFix(config),
  ];
}

// Parameter class for rule configuration
class _MyCustomRuleParameter extends CommonLintParameter {
  const _MyCustomRuleParameter({
    super.excludes,
    super.includes,
    super.severity,
  });

  static _MyCustomRuleParameter fromMap(Map<String, dynamic> map) {
    return _MyCustomRuleParameter(
      excludes: safeCastToListString(map['excludes']),
      includes: safeCastToListString(map['includes']),
      severity: convertStringToErrorSeverity(map['severity']),
    );
  }
}

// Quick fix implementation
class _MyCustomRuleFix extends CommonQuickFix<_MyCustomRuleParameter> {
  _MyCustomRuleFix(super.config);

  @override
  Future<void> run(
    CustomLintResolver resolver,
    ChangeReporter reporter,
    CustomLintContext context,
    AnalysisError analysisError,
    List<AnalysisError> others,
  ) async {
    context.registry.addInstanceCreationExpression((node) {
      // ... apply fix
    });
  }
}
```

<!-- AI_VERIFY: based-on avoid_using_datetime_now.dart and common_lint_rule.dart -->

---

## 🟡 Concept 6: Linter Rule Categories

### 📖 Giải thích

Lint rules được chia thành categories:

| Category | Examples | Purpose |
|----------|----------|---------|
| **Style** | `prefer_single_quotes`, `camel_case_types` | Code formatting |
| **Best practices** | `avoid_print`, `prefer_final_locals` | Correctness |
| **Errors** | `missing_return`, `invalid_assignment` | Bug prevention |
| **Pub** | `sort_pub_dependencies` | Package management |

### 💡 FE Perspective

| Flutter | ESLint |
|---------|--------|
| Style rules | `Formatting` rules |
| Best practices | `Best Practices` rules |
| Errors | `Problems` rules |
| Pub rules | Custom project rules |

---

## 🟡 Concept 7: Analysis Server Management

### 📖 Giải thích

Dart Analysis Server phân tích code liên tục trong background.

### 📦 When to Restart

| Trigger | Action |
|---------|--------|
| Edit `analysis_options.yaml` | Restart server |
| Add custom lint package | Restart server |
| IDE shows stale warnings | Restart server |
| High RAM usage | Restart server |

### 📦 How to Restart

**VS Code:**
1. Cmd+Shift+P → "Dart: Restart Analysis Server"

**IntelliJ/Android Studio:**
1. File → Invalidate Caches → Restart

### 💡 FE Perspective

| Flutter | TypeScript |
|---------|------------|
| Dart Analysis Server | TypeScript Language Server |
| Restart Analysis Server | "TypeScript: Restart TS Server" |

---

## 🟡 Concept 8: IDE Integration

### 📖 Giải thích

Lint warnings hiển thị trực tiếp trong IDE.

### 📦 VS Code Integration

| Feature | How to Enable |
|---------|---------------|
| Inline warnings | Default |
| Quick fix (lightbulb) | Default |
| Problems panel | Cmd+Shift+M |
| Format on save | `editor.formatOnSave: true` |

### 📦 Settings Example

```json
{
  "dart.enableSdkFormatter": true,
  "editor.formatOnSave": true,
  "[dart]": {
    "editor.defaultFormatter": "Dart-Code.dart-code",
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    }
  }
}
```

---

## 🟢 Concept 9: Creating Custom Lints

### 📖 Giải thích

Tạo lint rule mới trong super_lint package.

### 📦 Step-by-Step

1. **Create rule file:**
   ```dart
   // super_lint/lib/rules/prefer_intl_dateformat.dart
   import '../index.dart';

   class PreferIntlDateformat extends CommonLintRule<_PreferIntlDateformatParameter> {
     PreferIntlDateformat(CustomLintConfigs configs) : super(
       RuleConfig(
         name: 'prefer_intl_dateformat',
         configs: configs,
         paramsParser: _PreferIntlDateformatParameter.fromMap,
         problemMessage: (_) => 'Use intl DateFormat instead of DateTime.toString()',
       ),
     );

     @override
     Future<void> check(
       CustomLintResolver resolver,
       ErrorReporter reporter,
       CustomLintContext context,
       String rootPath,
     ) async {
       // Implementation using context.registry to watch AST nodes
     }

     @override
     List<Fix> getFixes() => [];
   }
   ```

2. **Export in super_lint.dart:**
   ```dart
   export 'rules/prefer_intl_dateformat.dart';
   ```

3. **Register in Plugin** (via injectable pattern):
   ```dart
   // Rules are auto-registered via getLintRules() in the plugin
   // See _SuperLintPlugin.getLintRules() for the list of rule instances
   ```

4. **Test the rule:**
   ```dart
   test('prefer_intl_dateformat', () {
     testLint('lib/main.dart', PreferIntlDateformat(CustomLintConfigs({})));
   });
   ```

### 📦 QuickFix Pattern

QuickFix được implement bằng cách extend `CommonQuickFix`:

```dart
class _MyRuleFix extends CommonQuickFix<_MyRuleParameter> {
  _MyRuleFix(super.config);

  @override
  Future<void> run(
    CustomLintResolver resolver,
    ChangeReporter reporter,
    CustomLintContext context,
    AnalysisError analysisError,
    List<AnalysisError> others,
  ) async {
    // Get the source range of the error
    final sourceRange = analysisError.sourceRange;

    // Create change builder
    final changeBuilder = reporter.createChangeBuilder(
      message: 'Apply this fix',
      priority: 70,
    );

    // Apply the fix
    changeBuilder.addDartFileEdit((builder) {
      builder.addSimpleReplacement(sourceRange, 'replacement');
    });
  }
}
```

> 📝 **QuickFix signature khác với Rule:** `run()` nhận thêm `AnalysisError` và `List<AnalysisError>` để biết chính xác vị trí và các lỗi khác trong file.

> 📝 **Lưu ý quan trọng:** Method chính là `run()` chứ KHÔNG phải `check()`. Method `run()` được định nghĩa trong abstract class `CommonLintRule` và gọi `check()` sau khi kiểm tra `shouldSkipAnalysis`.

<!-- AI_VERIFY: based-on common_lint_rule.dart line 10-28 -->

---

## 🟢 Concept 10: Effective Dart Guidelines

### 📖 Giải thích

[Effective Dart](https://dart.dev/guides/language/effective-dart) là style guide chính thức.

### 📦 Key Sections

| Section | Focus |
|---------|-------|
| Style | Formatting, naming |
| Documentation | Comments, docs |
| Usage | Library usage |
| Design | API design |

### 📦 Quick Summary

```dart
// DO: Use const constructors
const items = ['a', 'b', 'c'];

// DON'T: Avoid print
print('debug'); // Remove before commit

// DO: Use final for locals
final name = getName();

// DON'T: Avoid relative imports
import 'package:app/models/user.dart';  // Preferred
// vs
import '../../models/user.dart';        // Avoid
```

---

## 🔗 Bridges Summary

| Flutter | Frontend | Backend |
|---------|----------|---------|
| `analysis_options.yaml` | `tsconfig.json` + `.eslintrc` | `.eslintrc` |
| `flutter_lints` | `@typescript-eslint/recommended` | ESLint recommended |
| `custom_lint` | `eslint-plugin-*` | Custom ESLint plugin |
| `flutter analyze` | `eslint` | `eslint` |
| `prefer_const_constructors` | `prefer-const` | `prefer-const` |
| `avoid_print` | `no-console` | `no-console` |
| Restart Analysis Server | Reload ESLint | Restart linter |
| Effective Dart | ESLint rules + Prettier | Style guides |

---

## 🎯 Micro-Task: Practice ngay

**Exercise 1: Find violation**

1. Mở file bất kỳ trong `lib/`
2. Tìm 1 violation của `prefer_const_constructors`
3. Fix nó
4. Run `flutter analyze` để verify

**Exercise 2: Add rule**

1. Thêm 1 rule vào `analysis_options.yaml`:
   ```yaml
   linter:
     rules:
       - always_put_control_body_on_new_line
   ```
2. Run `flutter analyze` để xem effect

---

→ Tiếp tục: [03-exercise.md](./03-exercise.md)

<!-- AI_VERIFY: generation-complete -->
