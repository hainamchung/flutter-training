# Advanced Module MC — Patterns & Tooling

> **Depth:** Advanced Survey — optional module, lighter scaffolding

---

## Mục tiêu

Sau module này, bạn sẽ:
- Hiểu advanced state management: Bloc pattern, Riverpod comparison, migration strategies
- Nắm GraphQL integration: queries, mutations, subscriptions, caching
- Implement WebSocket real-time communication: reconnection, state sync
- Master advanced tooling: Melos monorepo, custom build_runner, mutation testing

---

## Prerequisites

| Module | Cần nắm | Relevance cho MC |
|--------|---------|------------------|
| **M8** | Riverpod state management — đã dùng Riverpod | Bloc vs Riverpod comparison |
| **M12** | AppApiService, Dio — network layer | GraphQL, WebSocket APIs |
| **M18** | Testing — unit, widget, golden tests | Mutation testing |
| **MA** | Performance monitoring | Custom tooling |

---

## Nội dung

| File | Nội dung | Thời lượng |
|------|----------|------------|
| [01-code-walk.md](./01-code-walk.md) | Bloc, GraphQL, WebSocket, Melos code patterns | ~45 min |
| [02-concept.md](./02-concept.md) | 6 concepts: state management comparison, GraphQL, WebSocket, build_runner, monorepo, testing | ~30 min |
| [03-exercise.md](./03-exercise.md) | 5 exercises: Bloc migration → GraphQL → WebSocket → Melos → mutation testing | ~3-5 hrs |
| [04-verify.md](./04-verify.md) | Checklist xác nhận hoàn thành | ~10 min |

**Phân bố:** 🔴 ~25% · 🟡 ~50% · 🟢 ~25%

---

## Anchor Files

```
pubspec.yaml                              — GraphQL, WebSocket, Melos dependencies
lib/data_source/api/                     — REST API (base_flutter)
lib/ui/page/                             — State management patterns
test/                                    — Testing patterns
```

---

## 💡 FE Perspective Summary

| Flutter | Frontend Equivalent |
|---------|---------------------|
| Bloc pattern | Redux pattern |
| Riverpod | React Context + useState |
| GraphQL | Apollo Client / urql |
| WebSocket | Socket.io client |
| Melos monorepo | Nx / Turborepo |
| build_runner | Babel plugins / webpack loaders |

---

## Forward Reference

→ **Capstone:** Áp dụng patterns từ MC để build advanced features.

<!-- AI_VERIFY: generation-complete -->
