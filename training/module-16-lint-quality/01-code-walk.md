# Module 16: Lint & Code Quality — Code Walk

## CODE — Quan sát lint & code quality patterns

**Mục tiêu:** Đọc và hiểu lint configuration, flutter_lints, custom_lint rules.

**Lưu ý:** Snippets trong file này được đánh dấu `AI_VERIFY` — chúng được extract từ source code thật trong `base_flutter/`.

---

## 📁 Bước 1: analysis_options.yaml Structure

### 1.1 Full Configuration

**File:** [analysis_options.yaml](../../base_flutter/analysis_options.yaml)

> ⚠️ **Note:** Snippet below shows key rules only. The full `analysis_options.yaml` contains ~80+ linter rules. See the actual file for complete configuration.

```yaml
# AI_VERIFY: analysis_options.yaml — 233 lines total (snippet shows key rules below)
include: package:flutter_lints/flutter.yaml

analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/*.gr.dart"
    - "build/**"
    - "test/.test_coverage.dart"
    - "lib/generated/**"
  
  errors:
    invalid_annotation_target: ignore
    missing_required_param: error
    missing_return: error
    todo: ignore
  
  language:
    strict-casts: true
    strict-raw-types: true

linter:
  rules:
    # Enabled rules
    - always_declare_return_types
    - always_require_non_null_named_parameters
    - annotate_overrides
    - avoid_empty_else
    - avoid_print
    - avoid_relative_lib_imports
    - avoid_returning_null_for_future
    - avoid_slow_async_io
    - avoid_types_as_parameter_names
    - avoid_unnecessary_containers
    - avoid_web_libraries_in_flutter
    - camel_case_extensions
    - camel_case_types
    - cancel_subscriptions
    - close_sinks
    - constant_identifier_names
    - curly_braces_in_flow_control_structures
    - empty_catches
    - empty_constructor_bodies
    - prefer_const_constructors
    - prefer_const_literals_to_create_immutables
    - prefer_contains
    - prefer_final_fields
    - prefer_final_locals
    - prefer_if_null_operators
    - prefer_interpolation_to_compose_strings
    - prefer_is_empty
    - prefer_is_not_empty
    - prefer_single_quotes
    - sort_pub_dependencies
    - unnecessary_brace_in_string_interps
    - unnecessary_const
    - unnecessary_new
    - unnecessary_null_in_if_null_operators
    - unnecessary_this
    - use_key_in_widget_constructors
    - use_rethrow_when_possible
```

**Câu hỏi gợi suy nghĩ:**
> Tại sao cần `exclude` các generated files? Điều gì xảy ra nếu không exclude chúng?

---

### 1.2 Rule Categories

**Key rules grouped by purpose:**

```yaml
# Style & Formatting
- prefer_single_quotes              # ' vs "
- prefer_const_constructors         # const when possible
- prefer_const_literals_to_create_immutables
- unnecessary_const
- unnecessary_new                  # new keyword unnecessary

# Best Practices
- avoid_print                       # No print in production
- avoid_returning_null_for_future  # Return Future.value(null) instead
- cancel_subscriptions              # Cancel streams
- close_sinks                       # Close IO sinks
- prefer_final_locals               # Final when not reassigned

# Errors Prevention
- always_declare_return_types       # Return types required
- always_require_non_null_named_parameters
- missing_return                    # Error if no return
- prefer_contains                  # Use .contains() instead of indexOf
- prefer_is_empty                  # .isEmpty instead of length == 0
- prefer_is_not_empty              # .isNotEmpty instead of length != 0

# Naming
- camel_case_types
- camel_case_extensions
- constant_identifier_names
```

---

## 📁 Bước 2: flutter_lints Configuration

### 2.1 Enable/Disable Specific Rules

```yaml
# Disable specific rule
linter:
  rules:
    - prefer_const_constructors: false    # Disable this rule
    - avoid_print: false                  # Allow print temporarily

# Enable with severity override
analyzer:
  errors:
    prefer_const_constructors: warning    # Treat as warning instead of error
```

### 2.2 Severity Levels

| Level | Meaning | In YAML |
|-------|---------|---------|
| Error | Compilation fails | `error` (default for most) |
| Warning | Code works but suboptimal | `warning` |
| Info | Hint, style suggestion | `info` |
| Ignore | No feedback | `ignore` |

---

## 📁 Bước 3: custom_lint Setup

### 3.1 pubspec.yaml Dev Dependencies

**File:** [pubspec.yaml](../../base_flutter/pubspec.yaml)

```yaml
# AI_VERIFY: pubspec.yaml dev_dependencies — ACTUAL VERSIONS
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: 6.0.0
  custom_lint: 0.8.0
  build_runner: 2.7.0
  # super_lint is a local package:
  super_lint:
    path: super_lint
```

### 3.2 Enable in analysis_options.yaml

```yaml
# Add custom_lint ruleset
include: package:custom_lint/lint.yaml

analyzer:
  plugins:
    - custom_lint
```

---

## 📁 Bước 4: super_lint Package Structure

### 4.1 Package Location

```
super_lint/
├── lib/
│   ├── super_lint.dart           # Main export
│   ├── rules/
│   │   ├── avoid_legacy_color.dart
│   │   ├── prefer_datetime_from_timestamp.dart
│   │   └── prefer_intl_dateformat.dart
│   └── visitors/
│       └── avoid_legacy_color_visitor.dart
├── test/
│   ├── avoid_legacy_color_test.dart
│   └── prefer_datetime_test.dart
├── pubspec.yaml
└── README.md
```

### 4.2 Custom Lint Rule Example

**File:** [super_lint/lib/src/lints/avoid_using_datetime_now.dart](https://github.com/nals-lab/nals-flutter/tree/main/super_lint/lib/src/lints) (actual lint rules in `src/lints/`)

**Note:** The actual project uses a different lint rule pattern than shown. Below is the **actual** `super_lint.dart` entry point showing all real lint rules registered:

```dart
// AI_VERIFY: super_lint/lib/super_lint.dart — ACTUAL entry point
import 'src/index.dart';

PluginBase createPlugin() => _SuperLintPlugin();

class _SuperLintPlugin extends PluginBase {
  @override
  List<LintRule> getLintRules(CustomLintConfigs configs) {
    return [
      AvoidUnnecessaryAsyncFunction(configs),
      PreferNamedParameters(configs),
      PreferIsEmptyString(configs),
      PreferIsNotEmptyString(configs),
      PreferAsyncAwait(configs),
      TestFolderMustMirrorLibFolder(configs),
      AvoidHardCodedColors(configs),
      PreferCommonWidgets(configs),
      AvoidHardCodedStrings(configs),
      IncorrectParentClass(configs),
      PreferImportingIndexFile(configs),
      AvoidUsingTextStyleConstructorDirectly(configs),
      IncorrectScreenNameParameterValue(configs),
      IncorrectEventParameterName(configs),
      IncorrectEventParameterType(configs),
      IncorrectEventName(configs),
      IncorrectScreenNameEnumValue(configs),
      AvoidDynamic(configs),
      AvoidUsingEnumNameAsKey(configs),
      AvoidUsingUnsafeCast(configs),
      MissingRunCatching(configs),
      UtilFunctionsMustBeStatic(configs),
      MissingCommonScrollbar(configs),
      IncorrectFreezedDefaultValueType(configs),
      PreferSingleWidgetPerFile(configs),
      RequireMatchingFileAndClassName(configs),
      MissingGoldenTest(configs),
      MissingTestGroup(configs),
      AvoidUsingDateTimeNow(configs),
      IncorrectGoldenImageName(configs),
    ];
  }
}
```

**Key API patterns from actual source:**

```dart
// Method signature: getLintRules receives CustomLintConfigs, not CustomLintOptions
@override
List<LintRule> getLintRules(CustomLintConfigs configs) {
  return [ /* list of LintRule instances */ ];
}

// LintRule base class — all rules extend this
class AvoidUsingDateTimeNow extends LintRule {
  const AvoidUsingDateTimeNow(CustomLintConfigs configs) : super(configs);

  @override
  void run(CustomLintResolver resolver, File file, CompilableUnit unit,
            LintCode lintCode, ErrorReporter reporter) {
    // Walk the AST to find DateTime.now() calls
    // Report using: reporter.reportErrorForNode(lintCode, node);
  }
}
```

**Câu hỏi gợi suy nghĩ:**
> Tại sao dự án có tới **30 custom lint rules**? Custom lint rules được dùng để enforce những conventions nào mà `flutter_lints` không cover? (VD: `AvoidHardCodedColors`, `TestFolderMustMirrorLibFolder`, `MissingGoldenTest`, etc.)

---

### 4.3 super_lint.dart Entry Point — Actual Source

> ⚠️ **Note:** Section 4.2 above showed the **same source** as section 4.3. The super_lint.dart file registers **30 custom lint rules** (not 21 as previously mentioned).

<!-- AI_VERIFY: base_flutter/super_lint/lib/super_lint.dart -->

→ [Mở file gốc: `super_lint/lib/super_lint.dart`](../../base_flutter/super_lint/lib/super_lint.dart)

**30 custom lint rules registered:**

| Category | Rules |
|----------|-------|
| **Code Style** | `PreferNamedParameters`, `PreferSingleWidgetPerFile` |
| **String/Type** | `PreferIsEmptyString`, `PreferIsNotEmptyString`, `AvoidDynamic`, `AvoidUsingEnumNameAsKey` |
| **Async** | `AvoidUnnecessaryAsyncFunction`, `PreferAsyncAwait` |
| **Architecture** | `TestFolderMustMirrorLibFolder`, `PreferImportingIndexFile`, `RequireMatchingFileAndClassName`, `UtilFunctionsMustBeStatic` |
| **UI/Theme** | `AvoidHardCodedColors`, `AvoidHardCodedStrings`, `PreferCommonWidgets`, `AvoidUsingTextStyleConstructorDirectly` |
| **Error Handling** | `MissingRunCatching`, `AvoidUsingUnsafeCast` |
| **Analytics/Events** | `IncorrectScreenNameParameterValue`, `IncorrectScreenNameEnumValue`, `IncorrectEventParameterName`, `IncorrectEventParameterType`, `IncorrectEventName` |
| **Testing** | `MissingGoldenTest`, `MissingTestGroup`, `IncorrectGoldenImageName` |
| **Layout** | `MissingCommonScrollbar` |
| **Freezed** | `IncorrectFreezedDefaultValueType` |
| **Other** | `IncorrectParentClass` |

---

## 📁 Bước 5: Running Analysis

### 5.1 Command Line

```bash
# Run analysis
flutter analyze

# Analyze specific file
flutter analyze lib/exception/

# Analyze with fix
flutter analyze --fix

# Show machine-readable output
flutter analyze --machine
```

### 5.2 Makefile Integration

```makefile
# AI_VERIFY: makefile analyze target
analyze:
	@echo "Running Flutter analysis..."
	flutter analyze --fatal-infos --fatal-warnings

lint: analyze
```

---

## 📁 Bước 6: Analysis Server Management

### 6.1 Restart Analysis Server (VS Code)

```bash
# Command Palette (Cmd+Shift+P)
# > "Dart: Restart Analysis Server"
```

### 6.2 When to Restart

- Sau khi thay đổi `analysis_options.yaml`
- Sau khi thêm custom lint rules
- Khi warnings/errors không update
- RAM usage cao

### 6.3 IDE Integration

**VS Code settings (`.vscode/settings.json`):**

```json
{
  "dart.enableSdkFormatter": true,
  "editor.formatOnSave": true,
  "editor.rulers": [100],
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

## 🎯 Checkpoint

Sau khi đọc xong:

1. ✅ Hiểu cấu trúc `analysis_options.yaml`
2. ✅ Biết cách enable/disable lint rules
3. ✅ Hiểu super_lint package structure
4. ✅ Biết cách restart Analysis Server

→ Tiếp tục: [02-concept.md](./02-concept.md)

<!-- AI_VERIFY: generation-complete -->
