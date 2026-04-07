# 03-exercise.md — Performance Optimization

## PRACTICE: Performance Optimization Exercises

---

## Exercise 1: Profile App with DevTools ⭐

### Objective

Learn to use Flutter DevTools để profile performance.

### Steps

1. **Start DevTools:**
   ```bash
   # Run app với DevTools
   flutter run --observe
   
   # Hoặc attach to running app
   flutter attach
   ```

2. **Open DevTools:**
   - Navigate to `http://localhost:8080` (hoặc port khác)
   - Hoặc dùng `flutter devtools` command

3. **Use Performance Tab:**
   ```
   a. Click "Performance" tab
   b. Click "Record" button
   c. Interact với app (scroll, tap buttons)
   d. Click "Stop" button
   e. Analyze timeline:
      - Look for red bars (slow frames)
      - Check UI thread vs GPU thread
      - Identify jank patterns
   ```

4. **Use CPU Profiler:**
   ```
   a. Click "CPU Profiler" tab
   b. Select "Dart" view
   c. Click "Record"
   d. Run some operations
   e. Click "Stop"
   f. Analyze flame chart
   ```

5. **Use Memory Tab:**
   ```
   a. Click "Memory" tab
   b. Take snapshot
   c. Interact với app
   d. Take another snapshot
   e. Compare allocations
   ```

### Questions

- Frame nào bị slow (>16ms)?
- Widget nào rebuild nhiều nhất?
- Có memory leaks không?

### Verification

- [ ] Can open DevTools
- [ ] Can record and analyze timeline
- [ ] Can identify slow frames

---

## Exercise 2: Optimize Widget Rebuilds ⭐

### Objective

Reduce unnecessary widget rebuilds.

### Steps

### Part A: Add Const Constructors

1. **Identify non-const widgets:**
   ```bash
   flutter analyze --no-fatal-infos
   # Look for: "This class is marked as 'immutable' but..."
   ```

2. **Add const constructors:**
   
   Before:
   ```dart
   class MyWidget extends StatelessWidget {
     final String title;
     
     MyWidget({required this.title});
     
     @override
     Widget build(BuildContext context) {
       return Column(
         children: [
           Text(title),
           Container(height: 16, color: Colors.grey),
         ],
       );
     }
   }
   ```
   
   After:
   ```dart
   class MyWidget extends StatelessWidget {
     final String title;
     
     const MyWidget({super.key, required this.title});
     
     @override
     Widget build(BuildContext context) {
       return Column(
         children: [
           Text(title),
           const SizedBox(height: 16),  // const!
           Container(height: 16, color: Colors.grey),
         ],
       );
     }
   }
   ```

3. **Verify with DevTools:**
   - Enable "Show widget rebuilds" in Inspector
   - Check rebuild count decreases

### Part B: Add Keys to List

1. **Find list without keys:**
   ```dart
   // ❌ Without key
   ListView.builder(
     itemBuilder: (context, index) {
       return ListTile(title: Text(items[index].name));
     },
   )
   ```

2. **Add ValueKey:**
   ```dart
   // ✅ With key
   ListView.builder(
     itemBuilder: (context, index) {
       return ListTile(
         key: ValueKey(items[index].id),  // ✅
         title: Text(items[index].name),
       );
     },
   )
   ```

### Questions

- Sự khác nhau giữa const và non-const?
- Tại sao cần keys trong list?

### Verification

- [ ] At least 5 widgets use const
- [ ] ListView has keys
- [ ] Rebuild count reduced (check in DevTools)

---

## Exercise 3: Fix Memory Leaks ⭐⭐

### Objective

Identify và fix memory leaks trong codebase.

### Steps

### Part A: Add Leak Detection

1. **Add debug prints in dispose:**
   ```dart
   @override
   void dispose() {
     debugPrint('WidgetName disposing...');
     _controller.dispose();
     _subscription?.cancel();
     super.dispose();
   }
   ```

2. **Create memory leak detector:**
   ```dart
   // lib/utils/memory_leak_detector.dart
   
   class MemoryLeakDetector {
     static final Set<String> _activeWidgets = {};
     
     static void register(String widgetName) {
       _activeWidgets.add(widgetName);
       debugPrint('Registered: $widgetName (total: ${_activeWidgets.length})');
     }
     
     static void unregister(String widgetName) {
       _activeWidgets.remove(widgetName);
       debugPrint('Unregistered: $widgetName (remaining: ${_activeWidgets.length})');
     }
     
     static void report() {
       debugPrint('=== Memory Leak Report ===');
       debugPrint('Active widgets: ${_activeWidgets.length}');
       for (final name in _activeWidgets) {
         debugPrint('  - $name');
       }
     }
   }
   ```

### Part B: Fix Common Leaks

1. **Fix StreamSubscription:**
   ```dart
   // Before
   class StreamWidget extends StatefulWidget {
     @override
     State<StreamWidget> createState() => _StreamWidgetState();
   }
   
   class _StreamWidgetState extends State<StreamWidget> {
     late StreamSubscription _subscription;
     
     @override
     void initState() {
       super.initState();
       _subscription = stream.listen((_) {});
       // ❌ Missing dispose
     }
   }
   
   // After
   class StreamWidget extends StatefulWidget {
     @override
     State<StreamWidget> createState() => _StreamWidgetState();
   }
   
   class _StreamWidgetState extends State<StreamWidget> {
     StreamSubscription? _subscription;
     
     @override
     void initState() {
       super.initState();
       _subscription = stream.listen((_) {});
     }
     
     @override
     void dispose() {
       _subscription?.cancel();  // ✅ Fixed
       super.dispose();
     }
   }
   ```

2. **Fix AnimationController:**
   ```dart
   // Before
   class AnimationWidget extends StatefulWidget {
     @override
     State<AnimationWidget> createState() => _AnimationWidgetState();
   }
   
   class _AnimationWidgetState extends State<AnimationWidget>
       with SingleTickerProviderStateMixin {
     late AnimationController _controller;
     
     @override
     void initState() {
       super.initState();
       _controller = AnimationController(vsync: this, duration: 1.seconds);
       // ❌ Missing dispose
     }
   }
   
   // After
   class AnimationWidget extends StatefulWidget {
     @override
     State<AnimationWidget> createState() => _AnimationWidgetState();
   }
   
   class _AnimationWidgetState extends State<AnimationWidget>
       with SingleTickerProviderStateMixin {
     late AnimationController _controller;
     
     @override
     void initState() {
       super.initState();
       _controller = AnimationController(vsync: this, duration: 1.seconds);
     }
     
     @override
     void dispose() {
       _controller.dispose();  // ✅ Fixed
       super.dispose();
     }
   }
   ```

### Questions

- Có bao nhiêu leak patterns đã fix?
- Làm sao detect leak trong future?

### Verification

- [ ] No StreamSubscription without cancel
- [ ] All AnimationController disposed
- [ ] All TextEditingController disposed

---

## Exercise 4: Implement Isolate Computation ⭐⭐

### Objective

Move heavy computation to background isolate.

### Steps

1. **Identify Heavy Computation:**
   
   Find in codebase:
   ```dart
   // ❌ Heavy computation on UI thread
   onPressed: () {
     final result = _heavyComputation(largeData);  // Blocks UI!
     setState(() => _result = result);
   }
   ```

2. **Create Isolate Function:**
   ```dart
   // lib/utils/isolate_utils.dart
   
   import 'package:flutter/foundation.dart';
   
   // Function must be top-level or static
   Future<List<int>> findPrimesInRange(int max) async {
     return compute(_findPrimes, max);
   }
   
   List<int> _findPrimes(int max) {
     return List.generate(max, (i) => i + 2)
         .where((n) => _isPrime(n))
         .toList();
   }
   
   bool _isPrime(int n) {
     if (n < 2) return false;
     for (int i = 2; i <= n / 2; i++) {
       if (n % i == 0) return false;
     }
     return true;
   }
   ```

3. **Implement with Loading State:**
   ```dart
   class HeavyComputationWidget extends StatefulWidget {
     @override
     State<HeavyComputationWidget> createState() =>
         _HeavyComputationWidgetState();
   }
   
   class _HeavyComputationWidgetState
       extends State<HeavyComputationWidget> {
     List<int> _primes = [];
     bool _loading = false;
   
     Future<void> _computeInBackground() async {
       setState(() => _loading = true);
       
       final result = await findPrimesInRange(100000);
       
       if (mounted) {
         setState(() {
           _primes = result;
           _loading = false;
         });
       }
     }
   
     @override
     Widget build(BuildContext context) {
       return Column(
         children: [
           ElevatedButton(
             onPressed: _loading ? null : _computeInBackground,
             child: const Text('Compute Primes'),
           ),
           if (_loading)
             const CircularProgressIndicator()
           else
             Text('Found ${_primes.length} primes'),
         ],
       );
     }
   }
   ```

4. **Test Performance:**
   ```bash
   # With isolate: ~500ms, UI responsive
   # Without isolate: ~2000ms, UI frozen
   ```

### Questions

- Sự khác nhau khi dùng isolate?
- compute() vs Isolate.spawn() khác nhau gì?

### Verification

- [ ] Heavy computation moved to isolate
- [ ] UI remains responsive during computation
- [ ] Loading state displayed

---

## Exercise 5: AI Prompt Dojo — Performance Review ⭐⭐⭐

### Objective

Viết prompt để AI review performance của Flutter app.

### Task

Tạo prompt để AI:

1. Review widget tree cho optimization opportunities
2. Identify memory leak patterns
3. Suggest ListView optimizations
4. Propose async optimization patterns

### Prompt Template

```markdown
# Flutter Performance Review Prompt

## Context
- Flutter version: [VERSION]
- App type: [TYPE - e.g., social media, e-commerce, etc.]
- Target: [iOS/Android/Both]
- Performance issues: [DESCRIBE any known issues]

## Code to Review
[PASTE relevant code files or describe patterns]

## Review Focus
1. Widget rebuild optimization
2. Memory leak prevention
3. ListView performance
4. Async operation handling
5. Image/network optimization

## Specific Questions
1. [Question 1]
2. [Question 2]
3. [Question 3]

## Expected Output Format
- Issues found (with file:line references)
- Severity (high/medium/low)
- Code examples for fixes
- Performance impact estimate
```

### Example Output Format

```yaml
issues:
  - severity: high
    category: memory_leak
    file: lib/widget/stream_widget.dart
    line: 25
    description: StreamSubscription not cancelled
    fix: |
      @override
      void dispose() {
        _subscription?.cancel();
        super.dispose();
      }
    impact: Memory grows indefinitely
  
  - severity: medium
    category: rebuild_optimization
    file: lib/widget/list_widget.dart
    line: 42
    description: Missing const constructor
    fix: |
      const MyWidget({super.key, ...})
    impact: Unnecessary rebuilds
```

### Verification

- [ ] Prompt captures all necessary context
- [ ] Output includes specific file:line references
- [ ] Code examples are actionable

---

## Self-Check Questions

Trả lời các câu hỏi sau trước khi qua module tiếp:

1. **DevTools:**
   - Làm thế nào để identify slow frames?
   - Timeline view hiển thị gì?

2. **Rebuild Optimization:**
   - Tại sao const constructor quan trọng?
   - Keys trong list dùng để làm gì?

3. **Memory Leaks:**
   - StreamSubscription cần làm gì trong dispose()?
   - AnimationController cần làm gì trong dispose()?

4. **Isolates:**
   - Khi nào cần dùng isolate?
   - compute() function hoạt động như thế nào?

5. **Lists:**
   - ListView.builder khác ListView thế nào?
   - itemExtent có tác dụng gì?

---

## Clean Up

```bash
# Remove debug prints
# git diff để verify changes
git diff --stat
```

---

## Next Steps

1. **Profile own app** với DevTools
2. **Apply optimizations** đã học
3. **Monitor performance** trong CI

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
