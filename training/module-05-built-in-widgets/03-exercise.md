# Exercises — Thực hành Built-in Widgets

> ⚠️ Tất cả bài tập thực hiện **trên codebase `base_flutter`** — không tạo project mới.
> Prerequisite: Đã đọc xong [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md).

---

## ⭐ Exercise 1: Widget Catalog Explorer

**Mục tiêu:** Explore Flutter widget catalog — tìm và identify widgets trong codebase.

### Hướng dẫn

1. Search codebase cho các widget patterns:

```bash
# Search trong lib/ folder
grep -rn "ListView\." base_flutter/lib/
grep -rn "GridView\." base_flutter/lib/
grep -rn "Stack(" base_flutter/lib/
grep -rn "GestureDetector" base_flutter/lib/
```

2. Với mỗi widget type, tìm ví dụ sử dụng:

| Widget Type | File | Line | Usage Pattern |
|------------|------|------|---------------|
| Column | ? | ? | ? |
| Row | ? | ? | ? |
| Stack | ? | ? | ? |
| Expanded | ? | ? | ? |
| Container | ? | ? | ? |
| TextField | ? | ? | ? |
| ElevatedButton | ? | ? | ? |
| ListView | ? | ? | ? |
| GridView | ? | ? | ? |

3. Document each usage:
   - Widget type
   - Parent widget
   - Purpose (layout, display, input)
   - Key props

### Câu hỏi

- Widget nào được dùng nhiều nhất?
- Layout widget nào thường chứa các layout widget khác?
- Input widgets dùng pattern gì để handle user input?

### ✅ Checklist hoàn thành

- [ ] Tìm ≥ 10 unique widgets trong codebase
- [ ] Document usage pattern cho mỗi widget
- [ ] Trả lời 3 câu hỏi

---

## ⭐⭐ Exercise 2: Build a Dashboard Layout

**Mục tiêu:** Build dashboard page với multiple widget types.

### Hướng dẫn

1. Tạo file `lib/ui/page/dashboard/dashboard_page.dart`:

```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

import '../../../../index.dart';

@RoutePage()
class DashboardPage extends StatelessWidget {
  const DashboardPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        elevation: 0,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Section 1: Stats Cards Row
            _buildStatsRow(),
            const SizedBox(height: 24),
            
            // Section 2: Featured Item Card
            _buildFeaturedCard(),
            const SizedBox(height: 24),
            
            // Section 3: Action Buttons
            _buildActionButtons(),
            const SizedBox(height: 24),
            
            // Section 4: Recent Items List
            _buildRecentList(),
          ],
        ),
      ),
    );
  }

  Widget _buildStatsRow() {
    // TODO: Implement stats row with 3 cards
    // Use Row with Expanded children
    // Each card: Container with Card, Icon, Text
    return Row(
      children: [
        Expanded(child: _buildStatCard(Icons.people, 'Users', '1,234')),
        const SizedBox(width: 8),
        Expanded(child: _buildStatCard(Icons.shopping_cart, 'Orders', '567')),
        const SizedBox(width: 8),
        Expanded(child: _buildStatCard(Icons.attach_money, 'Revenue', '\$12K')),
      ],
    );
  }

  Widget _buildStatCard(IconData icon, String label, String value) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Icon(icon, size: 32, color: Colors.blue),
            const SizedBox(height: 8),
            Text(value, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Text(label, style: TextStyle(color: Colors.grey[600])),
          ],
        ),
      ),
    );
  }

  Widget _buildFeaturedCard() {
    // TODO: Implement featured card with Stack
    // Image background + overlay text
    return Card(
      clipBehavior: Clip.antiAlias,
      child: Stack(
        children: [
          Container(
            height: 150,
            width: double.infinity,
            color: Colors.blue[100],
          ),
          Positioned(
            bottom: 16,
            left: 16,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Featured', style: TextStyle(color: Colors.white, fontSize: 24, fontWeight: FontWeight.bold)),
                Text('Check out our latest products', style: TextStyle(color: Colors.white70)),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildActionButtons() {
    // TODO: Implement action buttons row
    // Use Row with ElevatedButton and OutlinedButton
    return Row(
      children: [
        Expanded(
          child: ElevatedButton.icon(
            onPressed: () {},
            icon: Icon(Icons.add),
            label: Text('Create'),
          ),
        ),
        const SizedBox(width: 16),
        Expanded(
          child: OutlinedButton.icon(
            onPressed: () {},
            icon: Icon(Icons.search),
            label: Text('Search'),
          ),
        ),
      ],
    );
  }

  Widget _buildRecentList() {
    // TODO: Implement recent items list
    // Use ListView.separated with ListTile
    final items = ['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5'];
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Recent Items', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        ListView.separated(
          shrinkWrap: true,
          physics: NeverScrollableScrollPhysics(),
          itemCount: items.length,
          separatorBuilder: (_, __) => Divider(),
          itemBuilder: (context, index) => ListTile(
            leading: CircleAvatar(child: Text('${index + 1}')),
            title: Text(items[index]),
            trailing: Icon(Icons.chevron_right),
            onTap: () {},
          ),
        ),
      ],
    );
  }
}
```

2. Thêm route vào `app_router.dart`:

```dart
AutoRoute(page: DashboardRoute.page),
```

3. Chạy `dart run build_runner build --delete-conflicting-outputs`.

4. Navigate từ Login page để test.

### Câu hỏi

- `shrinkWrap: true` và `physics: NeverScrollableScrollPhysics()` cần thiết không? Nếu không có thì sao?
- `Stack` với `Positioned` hoạt động thế nào?
- `ListView.separated` vs `ListView.builder` — khác nhau gì?

### ✅ Checklist hoàn thành

- [ ] Tạo DashboardPage với 4 sections
- [ ] Stats row với 3 cards (Row + Expanded)
- [ ] Featured card với Stack + Positioned
- [ ] Action buttons với ElevatedButton + OutlinedButton
- [ ] Recent list với ListView.separated
- [ ] Chạy build_runner thành công
- [ ] Test navigation
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes** sau khi verify

---

## ⭐⭐ Exercise 3: Implement ListView with Actions

**Mục tiêu:** Build list với swipe-to-delete và pull-to-refresh.

### Hướng dẫn

1. Tạo file `lib/ui/page/todo_list/todo_list_page.dart`:

```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

import '../../../../index.dart';

@RoutePage()
class TodoListPage extends StatefulWidget {
  const TodoListPage({super.key});

  @override
  State<TodoListPage> createState() => _TodoListPageState();
}

class _TodoListPageState extends State<TodoListPage> {
  final List<String> _items = List.generate(20, (i) => 'Task ${i + 1}');
  final GlobalKey<RefreshIndicatorState> _refreshKey = GlobalKey();

  Future<void> _onRefresh() async {
    await Future.delayed(Duration(seconds: 1));
    setState(() {
      _items.shuffle();
    });
  }

  void _onDismissed(int index) {
    setState(() {
      _items.removeAt(index);
    });
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Item deleted'),
        action: SnackBarAction(
          label: 'Undo',
          onPressed: () {
            setState(() {
              _items.insert(index, 'Task ${index + 1}');
            });
          },
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List'),
        actions: [
          IconButton(
            icon: const Icon(Icons.add),
            onPressed: () => _showAddDialog(context),
          ),
        ],
      ),
      body: _items.isEmpty
          ? Center(child: Text('No items'))
          : RefreshIndicator(
              key: _refreshKey,
              onRefresh: _onRefresh,
              child: ListView.builder(
                itemCount: _items.length,
                itemBuilder: (context, index) {
                  return Dismissible(
                    key: Key(_items[index]),
                    direction: DismissDirection.endToStart,
                    background: Container(
                      color: Colors.red,
                      alignment: Alignment.centerRight,
                      padding: EdgeInsets.only(right: 16),
                      child: Icon(Icons.delete, color: Colors.white),
                    ),
                    onDismissed: (_) => _onDismissed(index),
                    child: ListTile(
                      leading: Checkbox(
                        value: false,
                        onChanged: (value) {},
                      ),
                      title: Text(_items[index]),
                      trailing: Icon(Icons.drag_handle),
                    ),
                  );
                },
              ),
            ),
    );
  }

  void _showAddDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Add Task'),
        content: TextField(
          autofocus: true,
          decoration: InputDecoration(hintText: 'Task name'),
          onSubmitted: (value) {
            if (value.isNotEmpty) {
              setState(() => _items.add(value));
              Navigator.pop(context);
            }
          },
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
        ],
      ),
    );
  }
}
```

2. Thêm route và test.

### Câu hỏi

- `Dismissible` direction `endToStart` vs `horizontal` — khác nhau gì?
- `RefreshIndicator` cần `GlobalKey` không? Tại sao?
- `ListView.builder` vs `ListView.separated` — performance khác gì?

### ✅ Checklist hoàn thành

- [ ] Tạo TodoListPage với swipe-to-delete
- [ ] Pull-to-refresh với RefreshIndicator
- [ ] Add task dialog với AlertDialog
- [ ] SnackBar với undo action
- [ ] Test swipe → delete → undo
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes**

---

## ⭐⭐ Exercise 4: Create Modal Bottom Sheet

**Mục tiêu:** Build filter bottom sheet với multiple selections.

### Hướng dẫn

1. Tạo file `lib/ui/page/filter/filter_bottom_sheet.dart`:

```dart
import 'package:flutter/material.dart';

class FilterBottomSheet extends StatefulWidget {
  const FilterBottomSheet({super.key});

  static Future<Map<String, List<String>>> show(BuildContext context) {
    return showModalBottomSheet<Map<String, List<String>>>(
      context: context,
      isScrollControlled: true,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.6,
        minChildSize: 0.3,
        maxChildSize: 0.9,
        expand: false,
        builder: (context, scrollController) => FilterBottomSheet(
          scrollController: scrollController,
        ),
      ),
    );
  }

  const FilterBottomSheet({super.key, this.scrollController});

  final ScrollController? scrollController;

  @override
  State<FilterBottomSheet> createState() => _FilterBottomSheetState();
}

class _FilterBottomSheetState extends State<FilterBottomSheet> {
  final Set<String> _selectedCategories = {};
  final Set<String> _selectedTags = {};
  String _sortBy = 'newest';

  final List<String> _categories = ['Electronics', 'Clothing', 'Books', 'Food'];
  final List<String> _tags = ['Sale', 'New', 'Popular', 'Featured'];

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Handle
        Container(
          margin: EdgeInsets.only(top: 12),
          width: 40,
          height: 4,
          decoration: BoxDecoration(
            color: Colors.grey[300],
            borderRadius: BorderRadius.circular(2),
          ),
        ),
        
        // Header
        Padding(
          padding: EdgeInsets.all(16),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text('Filters', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
              TextButton(
                onPressed: () {
                  setState(() {
                    _selectedCategories.clear();
                    _selectedTags.clear();
                    _sortBy = 'newest';
                  });
                },
                child: Text('Reset'),
              ),
            ],
          ),
        ),
        
        Divider(height: 1),
        
        // Content
        Expanded(
          child: ListView(
            controller: widget.scrollController,
            padding: EdgeInsets.all(16),
            children: [
              // Sort By
              Text('Sort By', style: TextStyle(fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              Wrap(
                spacing: 8,
                children: [
                  ChoiceChip(
                    label: Text('Newest'),
                    selected: _sortBy == 'newest',
                    onSelected: (selected) {
                      setState(() => _sortBy = 'newest');
                    },
                  ),
                  ChoiceChip(
                    label: Text('Oldest'),
                    selected: _sortBy == 'oldest',
                    onSelected: (selected) {
                      setState(() => _sortBy = 'oldest');
                    },
                  ),
                  ChoiceChip(
                    label: Text('Price Low'),
                    selected: _sortBy == 'price_low',
                    onSelected: (selected) {
                      setState(() => _sortBy = 'price_low');
                    },
                  ),
                  ChoiceChip(
                    label: Text('Price High'),
                    selected: _sortBy == 'price_high',
                    onSelected: (selected) {
                      setState(() => _sortBy = 'price_high');
                    },
                  ),
                ],
              ),
              
              const SizedBox(height: 24),
              
              // Categories
              Text('Categories', style: TextStyle(fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              Wrap(
                spacing: 8,
                runSpacing: 8,
                children: _categories.map((category) {
                  final isSelected = _selectedCategories.contains(category);
                  return FilterChip(
                    label: Text(category),
                    selected: isSelected,
                    onSelected: (selected) {
                      setState(() {
                        if (selected) {
                          _selectedCategories.add(category);
                        } else {
                          _selectedCategories.remove(category);
                        }
                      });
                    },
                  );
                }).toList(),
              ),
              
              const SizedBox(height: 24),
              
              // Tags
              Text('Tags', style: TextStyle(fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              Wrap(
                spacing: 8,
                runSpacing: 8,
                children: _tags.map((tag) {
                  final isSelected = _selectedTags.contains(tag);
                  return FilterChip(
                    label: Text(tag),
                    selected: isSelected,
                    onSelected: (selected) {
                      setState(() {
                        if (selected) {
                          _selectedTags.add(tag);
                        } else {
                          _selectedTags.remove(tag);
                        }
                      });
                    },
                  );
                }).toList(),
              ),
            ],
          ),
        ),
        
        Divider(height: 1),
        
        // Apply Button
        Padding(
          padding: EdgeInsets.all(16),
          child: ElevatedButton(
            onPressed: () {
              Navigator.pop(context, {
                'categories': _selectedCategories.toList(),
                'tags': _selectedTags.toList(),
                'sortBy': _sortBy,
              });
            },
            style: ElevatedButton.styleFrom(
              minimumSize: Size(double.infinity, 48),
            ),
            child: Text('Apply Filters'),
          ),
        ),
      ],
    );
  }
}
```

2. Tích hợp vào một page:

```dart
// Trong một page, gọi:
final result = await FilterBottomSheet.show(context);
if (result != null) {
  // Handle filter result
  print('Categories: ${result['categories']}');
  print('Tags: ${result['tags']}');
  print('Sort: ${result['sortBy']}');
}
```

3. Test bottom sheet behavior.

### Câu hỏi

- `DraggableScrollableSheet` vs `showModalBottomSheet` thuần — khác nhau gì?
- `isScrollControlled: true` có tác dụng gì?
- `FilterChip` vs `ChoiceChip` — khác nhau gì?

### ✅ Checklist hoàn thành

- [ ] Tạo FilterBottomSheet với categories
- [ ] Sort options với ChoiceChip
- [ ] Filter options với FilterChip
- [ ] Reset button
- [ ] Apply button với result
- [ ] Test: drag to expand, select filters, apply
- [ ] Trả lời 3 câu hỏi
- [ ] **Revert changes**

---

## ⭐⭐⭐ Exercise 5: AI Prompt Dojo — Widget Selection

### 🤖 AI Dojo — Widget Selection cho Complex UI

**Mục tiêu:** Dùng AI để design widget selection cho complex UI scenario.

**Bước thực hiện:**

1. Gửi prompt sau cho AI:

```
Bạn là Flutter UI architect. Design widget selection cho scenario sau:

**Scenario:** E-commerce product detail page với:
- Collapsible header image gallery (swipeable)
- Product info: name, price, rating (stars)
- Size selector (chips, single select)
- Color selector (circles, single select)
- Quantity selector (stepper: -, count, +)
- Add to cart button (full width, prominent)
- Description expandable section
- Reviews section (lazy loaded list)
- Floating bottom bar với price + add to cart

Yêu cầu:
1. Vẽ widget tree structure (ASCII art)
2. List các widget cần dùng cho mỗi section
3. Đề xuất state management approach cho:
   - Image gallery position
   - Selected size/color
   - Quantity
4. Nêu potential performance issues

Code structure (Dart):
[PASTE any relevant base_flutter widgets nếu cần]
```

2. Với mỗi AI suggestion:
   - Verify bằng cách đọc docs
   - Check xem có pattern nào trong base_flutter có thể reuse
   - Evaluate performance implications

3. Hỏi follow-up: "Với approach tốt nhất, viết skeleton code cụ thể."

**✅ Tiêu chí đánh giá:**

- [ ] AI cung cấp complete widget tree
- [ ] AI đề xuất appropriate widgets cho mỗi section
- [ ] AI address performance considerations
- [ ] Bạn verify được ≥ 2 suggestions của AI
- [ ] Bạn viết 2-3 câu đánh giá: "AI tốt ở..., miss ở..., sai ở..."

---

## Exercise Summary

| # | Bài tập | Độ khó | Concept chính | Output |
|---|---------|--------|--------------|--------|
| 1 | Widget Catalog Explorer | ⭐ | Widget identification | Bảng documentation |
| 2 | Build Dashboard Layout | ⭐⭐ | Layout widgets composition | Working dashboard page |
| 3 | ListView with Actions | ⭐⭐ | ListView, Dismissible, Refresh | Working todo list |
| 4 | Modal Bottom Sheet | ⭐⭐ | BottomSheet, Chips, Draggable | Working filter sheet |
| 5 | AI Dojo — Widget Selection | ⭐⭐⭐ | Widget architecture | AI design + evaluation |

**Tiếp theo:** [04-verify.md](./04-verify.md) — checklist tự đánh giá.

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
