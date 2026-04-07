# Module 17 — Architecture & Dependency Injection

## Tổng quan

Module này survey toàn bộ **Dependency Injection (DI) architecture** của dự án Flutter — từ `get_it` service locator, `injectable` code generation, `@Injectable()` và `@module` annotations, đến architecture decisions về DI vs non-DI. Đây là **Advanced Survey**: nắm DI architecture + patterns, trace injection flow trong codebase.

**Depth**: Advanced Survey — đọc hiểu DI setup và patterns, trace injection flow.

**Cycle:** CODE (DI setup + injection patterns) → EXPLAIN (service locator vs DI) → PRACTICE (trace flows + add new injectable).

---

## ⏭️ Skip Path

Bạn có thể bỏ qua module này nếu trả lời **Yes** cho tất cả câu sau:

1. Giải thích được `get_it` service locator pattern — `getIt<Service>()` vs `final service = Service()`?
2. Trace được DI flow: `@Injectable()` class → `getIt.init()` → `getIt<Service>()` usage?
3. Phân biệt được `@Injectable()` vs `@module` — khi nào dùng cái nào?
4. Hiểu `@preResolve` annotation — async dependency initialization?
5. Giải thích được tại sao `getIt` là service locator, không phải DI container thuần?

→ Nếu **5/5 Yes** — chuyển thẳng [Module 18 — Testing](../module-18-testing/).
→ Nếu có bất kỳ **No** — hoàn thành module này.

---

### Key Files

| File | Purpose |
|------|---------|
| `01-code-walk.md` | Bootstrap flow, ServiceLocator, DI patterns |
| `02-concept.md` | DI concepts, getIt, Injectable, Bootstrap flow |
| `03-exercise.md` | Injectable migration exercises |

## Bạn sẽ học

1. **Service Locator Pattern** — `get_it` implementation, singleton vs factory registration
2. **injectable Code Generation** — `@Injectable()`, `@module`, `@preResolve` annotations
3. **DI Registration Flow** — `di.config.dart` generation, `configureInjection()`
4. **Architecture Decisions** — service locator vs DI container, trade-offs
5. **Tracing Injection Flow** — từ annotation → codegen → runtime resolution

**Phân bố:** 🔴 ~33% · 🟡 ~50% · 🟢 ~17%

---

## Kiến thức cần có

| Module | Nội dung | Vai trò trong M17 |
|--------|----------|-------------------|
| **M1** | `main.dart` bootstrap | DI initialization entry point |
| **M3** | Config, constants | Environment-dependent registrations |
| **M8** | Riverpod providers | Alternative to class-based DI |

---

## Cấu trúc files

| File | Nội dung | Thời gian |
|------|----------|-----------|
| [01-code-walk.md](./01-code-walk.md) | Walk-through: get_it setup, injectable, di.config.dart, injection usage | 30 min |
| [02-concept.md](./02-concept.md) | 6 concepts: service locator, injectable, module, preResolve, singleton/factory, architecture trade-offs | 25 min |
| [03-exercise.md](./03-exercise.md) | 5 exercises: trace flow → add injectable → resolve DI debate | 60 min |
| [04-verify.md](./04-verify.md) | Verification checklist | 10 min |

---

## 💡 FE Perspective

| Flutter DI | Frontend Equivalent |
|------------|-------------------|
| `get_it` + `injectable` | React Context + `useInjection()` hook (Angular DI pattern) |
| `@Injectable()` | TypeScript `@Injectable()` decorator (NestJS) |
| `@module` | Module/Provider array (Angular providers) |
| `@preResolve` | Async providers (Angular `APP_INITIALIZER`) |
| `getIt<Service>()` | React: `const svc = useInjection(Service)` |
| Singleton registration | React: Singleton Context hoặc `useMemo` cache |

---

## Key Files trong Codebase

```
lib/
├── di.dart                          ← get_it setup + injectable init
├── di.config.dart                   ← Auto-generated (DO NOT EDIT)
├── main.dart                        ← configureInjection() call
├── data_source/
│   ├── api/                         ← @Injectable() API services
│   └── preference/                   ← @Injectable() preference service
└── model/                           ← Freezed models (auto-registered)
```

---

## Forward Reference

→ **Module 18 — Testing:** `@Injectable()` classes cần mock trong tests — hiểu DI giúp viết mock tốt hơn.
→ **Module 19 — CI/CD:** `make gen_env` triggers injectable codegen trong CI pipeline.

<!-- AI_VERIFY: generation-complete -->
