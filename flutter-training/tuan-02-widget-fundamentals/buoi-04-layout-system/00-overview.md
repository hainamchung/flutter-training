# Buổi 04: Layout System — Cách Flutter sắp xếp UI

> 🟡 **Mức độ ưu tiên: SHOULD — Cần hiểu concept**
> Hiểu concept trước, AI hỗ trợ viết boilerplate. Tập trung vào WHY hơn HOW.

## 🗺️ Vị trí trong lộ trình

```
Tuần 1: Dart cơ bản          ✅ Hoàn thành
├── Buổi 01: Giới thiệu Dart & Flutter
├── Buổi 02: Dart nâng cao

Tuần 2: Widget Fundamentals   🔵 Đang học
├── Buổi 03: Widget Tree cơ bản ✅
├── Buổi 04: Layout System    👈 BẠN ĐANG Ở ĐÂY
├── Buổi 05: ...
├── Buổi 06: ...

Tuần 3: State Management
Tuần 4: Architecture & Project
```

## 📅 Tiến độ chương trình

> Buổi 4/16 — Hoàn thành 25% lộ trình
> ████░░░░░░░░░░░░

| Giai đoạn | Tuần | Buổi | Nội dung | Trạng thái |
|-----------|------|------|----------|------------|
| Dart & Flutter Foundation | 1 | Buổi 01-02 | Giới thiệu Dart & Flutter, Dart nâng cao | ✅ Hoàn thành |
| Widget & Layout | 2 | Buổi 03-04 | Widget Tree cơ bản, Layout System | 🔵 Đang học |
| Navigation & State cơ bản | 3 | Buổi 05-06 | Navigation & Routing, State Management cơ bản | ⬜ Chưa bắt đầu |
| State Management nâng cao | 4 | Buổi 07-08 | Riverpod, BLoC Pattern | ⬜ Chưa bắt đầu |
| Architecture & DI | 5 | Buổi 09-10 | Clean Architecture, DI & Testing | ⬜ Chưa bắt đầu |
| Networking & Data | 6 | Buổi 11-12 | Networking, Local Storage | ⬜ Chưa bắt đầu |
| Performance & Animation | 7 | Buổi 13-14 | Performance Optimization, Animation | ⬜ Chưa bắt đầu |
| Platform & Production | 8 | Buổi 15-16 | Platform Integration, CI/CD & Production | ⬜ Chưa bắt đầu |

---

## 🌍 Vai trò trong hệ sinh thái Flutter

Row, Column, Flex → responsive design cho mọi screen size. Layout system là cách Flutter sắp xếp widget trên màn hình — khác hoàn toàn CSS. Trong production app, layout code chiếm phần lớn UI code, và hiểu constraints model giúp bạn xây dựng UI responsive trên mọi thiết bị.

## 💼 Đóng góp vào dự án thực tế

- **Responsive layouts** — app chạy đẹp trên phone, tablet, và nhiều screen sizes
- **Fix layout errors** — RenderFlex overflow, unbounded height — lỗi phổ biến nhất
- **Scrollable content** — ListView.builder cho danh sách dài, tối ưu performance
- **Pixel-perfect UI** — implement design từ Figma sang Flutter code

---

## 🎯 Mục tiêu buổi học

### ✅ Hiểu được
- **Constraints model** — "Constraints go down, Sizes go up, Parent sets position"
- Tight vs Loose constraints và cách chúng ảnh hưởng đến layout
- Tại sao Flutter không dùng CSS và khác biệt cốt lõi

### ✅ Làm được
- Sử dụng **single-child layout widgets** — Container, Padding, Center, SizedBox, Align
- Sử dụng **multi-child layout widgets** — Row, Column, Stack, Wrap, ListView, GridView
- Nắm vững **Flex system** — Expanded, Flexible, Spacer và cách phân bổ không gian
- Xây dựng **scrollable layouts** — ListView.builder, GridView, CustomScrollView + Slivers
- Thiết kế **responsive** — MediaQuery, LayoutBuilder, OrientationBuilder
- Xử lý **layout errors** — RenderFlex overflow, unbounded height

### 🚫 Chưa cần biết
- CustomMultiChildLayout, CustomSingleChildLayout (nâng cao)
- Sliver protocol chi tiết (chỉ cần biết SliverList, SliverGrid)
- Platform-specific responsive breakpoints (Buổi 15)

---

## 🎯 Đánh giá mức độ ưu tiên

> Đánh giá dựa trên ngữ cảnh: dev Frontend (React/Vue) chuyển sang Flutter, **làm việc cùng AI** trong dự án thực tế.

### Lý thuyết

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| **Constraints Model — 3 quy tắc vàng** | 🔴 | **Nguyên nhân 90% layout bug.** AI không debug layout được |
| Single-child Widgets | 🟡 | Container, Padding, Center — AI viết tốt |
| Multi-child Widgets | 🟡 | Row, Column, Stack — cần hiểu concept |
| Row & Column — axis alignment | 🔴 | mainAxis/crossAxis — dùng hàng ngày |
| **Flex: Expanded, Flexible** | 🔴 | Flex distribution — AI hay sai ở đây |
| Scrollable Widgets | 🟡 | ListView, GridView — API AI viết tốt |
| CustomScrollView + Slivers | 🟢 | Nâng cao, tra cứu khi dùng |
| Responsive Design | 🟡 | MediaQuery, LayoutBuilder — AI viết được |
| **Common Layout Errors** | 🔴 | **Bắt buộc.** "Unbounded height", "Overflow" — phải tự debug |

### Ví dụ & Bài tập

| Mục | Priority | Ghi chú |
|-----|:---:|---------|
| VD1: Constraints Demo | 🔴 | Chạy để hiểu constraints flow |
| VD2: Row + Column | 🟡 | Layout practice, AI assist OK |
| VD3: Expanded/Flexible | 🔴 | Flex distribution — phải hiểu |
| VD4: ListView.builder | 🟡 | Lazy rendering concept |
| VD5: Layout Errors | 🔴 | **Xem lỗi, hiểu lỗi, tự fix** |
| BT1 ⭐ Login Form | 🟡 | Layout practice, AI scaffold được |
| BT2 ⭐⭐ Dashboard | 🟡 | Complex layout, AI assist OK |
| BT3 ⭐⭐⭐ Responsive | 🟢 | AI viết responsive tốt, cần review |

---

## 📋 Nội dung chi tiết

| Phần | Chủ đề | Thời lượng | Độ quan trọng |
|------|--------|------------|---------------|
| 1 | Constraints model — nguyên tắc cốt lõi | ~30 phút | 🔴 Critical |
| 2 | Single-child layout widgets | ~20 phút | 🔴 Critical |
| 3 | Multi-child layout widgets | ~30 phút | 🔴 Critical |
| 4 | Flex system (Expanded, Flexible, Spacer) | ~20 phút | 🔴 Critical |
| 5 | Scrollable widgets | ~20 phút | 🟡 Quan trọng |
| 6 | Responsive design | ~20 phút | 🟡 Quan trọng |
| 7 | Common layout errors & fixes | ~20 phút | 🔴 Critical |

**Tổng thời gian ước tính: ~2.5 giờ**

---

## 📝 Bài tập

| Bài | Tên | Độ khó | Loại | Chạy | Mô tả |
|-----|-----|--------|------|------|--------|
| BT1 | Login Form UI | ⭐ | Flutter app | `flutter run` | Column, TextField, ElevatedButton, padding |
| BT2 | Dashboard Layout | ⭐⭐ | Flutter app | `flutter run` | AppBar, GridView cards, ListView recent items |
| BT3 | Responsive App | ⭐⭐⭐ | Flutter app | `flutter run` | LayoutBuilder — thay đổi layout phone/tablet |

---

## 🔑 Keyword quan trọng

`BoxConstraints` · `tight constraints` · `loose constraints` · `Container` · `Padding` · `Center` · `SizedBox` · `Row` · `Column` · `Stack` · `Expanded` · `Flexible` · `Spacer` · `ListView.builder` · `GridView` · `CustomScrollView` · `Sliver` · `MediaQuery` · `LayoutBuilder` · `RenderFlex overflow` · `Unbounded height`

---

## 💡 Tại sao Layout System quan trọng?

> Flutter **không dùng CSS**. Thay vì `display: flex`, `margin: auto`, `grid-template-columns`, Flutter sử dụng **Constraints model** — một hệ thống hoàn toàn khác. Nếu bạn từ web development chuyển sang, đây là phần **bắt buộc phải hiểu** để không bị "mắc kẹt" với layout errors.

```
CSS (Web)                    Flutter
─────────────                ───────────────────
display: flex          →     Row / Column
flex: 1                →     Expanded(flex: 1)
grid                   →     GridView
overflow: scroll       →     ListView / SingleChildScrollView
margin/padding         →     Container / Padding
position: absolute     →     Stack + Positioned
@media (max-width)     →     LayoutBuilder / MediaQuery
```

---

## 📁 Cấu trúc files

```
buoi-04-layout-system/
├── 00-overview.md              ← Bạn đang ở đây
├── 01-ly-thuyet.md             ← Lý thuyết chi tiết (7 phần + best practices)
├── 02-vi-du.md                 ← 5 ví dụ code thực tế
├── 03-thuc-hanh.md             ← 3 bài tập + câu hỏi thảo luận
└── 04-tai-lieu-tham-khao.md    ← Tài liệu đọc thêm
```
