# Verify — Advanced Patterns & Tooling

> 📌 **Hoàn thành [03-exercise.md](./03-exercise.md) trước khi làm verify.**

---

## Verify Checklist

### Phần 1: Bloc Pattern 🔴

| # | Criteria | Check |
|---|----------|-------|
| 1.1 | Giải thích được event → state flow trong Bloc | [ ] |
| 1.2 | Implement được Bloc với event, state, bloc classes | [ ] |
| 1.3 | Phân biệt được Bloc vs Riverpod structures | [ ] |
| 1.4 | Migrate được existing Riverpod provider sang Bloc | [ ] |
| 1.5 | Test được Bloc với `blocTest` | [ ] |

### Phần 2: GraphQL 🟡

| # | Criteria | Check |
|---|----------|-------|
| 2.1 | Setup được GraphQL client với HttpLink và cache | [ ] |
| 2.2 | Implement được GraphQL query với variables | [ ] |
| 2.3 | Implement được GraphQL mutation | [ ] |
| 2.4 | Xử lý được GraphQL errors | [ ] |
| 2.5 | Hiểu được GraphQL caching strategies | [ ] |

### Phần 3: WebSocket 🟡

| # | Criteria | Check |
|---|----------|-------|
| 3.1 | Implement được WebSocket service với connect/disconnect | [ ] |
| 3.2 | Handle được reconnection với exponential backoff | [ ] |
| 3.3 | Implement được message send/receive | [ ] |
| 3.4 | Handle được WebSocket errors gracefully | [ ] |
| 3.5 | Hiểu được state synchronization patterns | [ ] |

### Phần 4: Melos Monorepo 🟡

| # | Criteria | Check |
|---|----------|-------|
| 4.1 | Tạo được melos.yaml configuration | [ ] |
| 4.2 | Setup được package structure với pubspec.yaml | [ ] |
| 4.3 | Configured được workspace dependencies | [ ] |
| 4.4 | Chạy được melos bootstrap thành công | [ ] |
| 4.5 | Chạy được melos analyze/test | [ ] |

### Phần 5: Testing & Tooling 🟢

| # | Criteria | Check |
|---|----------|-------|
| 5.1 | Setup được mutation testing configuration | [ ] |
| 5.2 | Interpret được mutation testing report | [ ] |
| 5.3 | Hiểu được mutation score calculation | [ ] |
| 5.4 | Identify được weak tests từ mutation report | [ ] |
| 5.5 | Created được custom build_runner generator (bonus) | [ ] |

---

## Quick Quiz

### Question 1: Bloc vs Riverpod

**Sự khác biệt chính giữa Bloc và Riverpod?**

A) Bloc dùng streams, Riverpod dùng providers
B) Bloc nhanh hơn Riverpod
C) Riverpod hỗ trợ code generation, Bloc không
D) Không có khác biệt

<details>
<summary>Answer</summary>

**A) Bloc dùng streams, Riverpod dùng providers**

Bloc sử dụng Dart Streams cho event → state flow. Riverpod sử dụng providers cho dependency injection và state management.

</details>

### Question 2: GraphQL

**Khi nào nên dùng GraphQL thay vì REST?**

A) Khi cần flexible data fetching với single endpoint
B) Khi backend đơn giản
C) Khi không cần caching
D) Khi API không thay đổi

<details>
<summary>Answer</summary>

**A) Khi cần flexible data fetching với single endpoint**

GraphQL tốt khi cần request exactly what you need, tránh over-fetching/under-fetching.

</details>

### Question 3: WebSocket

**Exponential backoff trong reconnection strategy là gì?**

A) Retry với delay tăng dần theo exponential
B) Retry immediately sau mỗi failure
C) Retry sau fixed delay
D) Không retry khi fail

<details>
<summary>Answer</summary>

**A) Retry với delay tăng dần theo exponential**

VD: 1s → 2s → 4s → 8s → maxDelay, tránh overwhelming server.

</details>

### Question 4: Melos

**Melos là gì?**

A) Package manager cho multi-package Flutter projects
B) State management library
C) Testing framework
D) Code generator

<details>
<summary>Answer</summary>

**A) Package manager cho multi-package Flutter projects**

Melos giúp manage và bootstrap multiple packages trong single repository.

</details>

---

## Practical Demonstration

### Task 1: Bloc Migration (5 min)

1. Show existing Riverpod provider
2. Show equivalent Bloc implementation
3. Compare code structure và lines of code
4. Run Bloc tests

### Task 2: GraphQL (5 min)

1. Show GraphQL client setup
2. Run query → show results
3. Run mutation → show data update
4. Explain caching behavior

### Task 3: WebSocket (5 min)

1. Show WebSocket service implementation
2. Connect → send test message
3. Simulate disconnect → show reconnection
4. Show message sync flow

### Task 4: Melos (5 min)

1. Show monorepo structure
2. Run melos bootstrap
3. Run melos analyze
4. Show dependency graph

### Task 5: Mutation Testing (3 min)

1. Run mutation tests
2. Show report
3. Explain mutation score
4. Identify weak tests

---

## Completion Criteria

Để hoàn thành module này, bạn cần:

- [ ] ✅ Hoàn thành **ít nhất 3/5 exercises**
- [ ] ✅ Pass **ít nhất 3/4 quiz questions** (75%)
- [ ] ✅ Demonstrate được **tất cả 5 patterns/tools** hoạt động
- [ ] ✅ Pass **15/20 practical criteria** hoặc hơn

**Points breakdown:**

| Section | Max Points | Passing |
|---------|------------|---------|
| Bloc Pattern | 25 | 18 |
| GraphQL | 20 | 14 |
| WebSocket | 20 | 14 |
| Melos Monorepo | 20 | 14 |
| Testing & Tooling | 15 | 11 |
| **Total** | **100** | **70** |

---

## Next Steps

✅ **Hoàn thành module MC** → Chuyển sang:
- [Capstone Full Project](../module-capstone-full/) — Apply all patterns
- Review tất cả optional modules (MA, MB, MC)

❌ **Chưa đạt yêu cầu** → Review lại:
- Đọc kỹ concepts chưa nắm vững
- Làm lại exercises
- Hỏi facilitator

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
