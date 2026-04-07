# Exercise — Advanced Patterns & Tooling

> 📌 **Prerequisites:**
> - Hoàn thành [01-code-walk.md](./01-code-walk.md) và [02-concept.md](./02-concept.md)
> - Có `base_flutter` project đã setup

---

## Exercise Overview

| # | Exercise | Focus | Difficulty | Thời gian |
|---|----------|-------|------------|-----------|
| 1 | Bloc Migration | Migrate existing Riverpod provider to Bloc | 🟡 Medium | ~45 min |
| 2 | GraphQL Integration | Setup GraphQL client và queries | 🟡 Medium | ~45 min |
| 3 | WebSocket Service | Implement real-time chat service | 🔴 Hard | ~60 min |
| 4 | Melos Monorepo | Setup monorepo structure | 🔴 Hard | ~60 min |
| 5 | Mutation Testing | Add mutation testing to project | 🟡 Medium | ~30 min |

**Tổng thời gian:** ~3-4 hours

---

## Exercise 1: Bloc Migration 🔄

**Mục tiêu:** Migrate một Riverpod provider sang Bloc pattern.

### Task 1.1: Analyze Existing Riverpod Code

Tìm một Riverpod provider trong codebase:

```bash
# Find Riverpod providers
grep -r "StateNotifierProvider\|AsyncNotifierProvider" lib/ --include="*.dart" | head -20
```

### Task 1.2: Create Bloc Equivalent

Chọn một provider để migrate:

```dart
// TODO: Tạo equivalent Bloc structure
// 1. Tạo auth_event.dart - Define all events
// 2. Tạo auth_state.dart - Define all states
// 3. Tạo auth_bloc.dart - Implement event handlers
// 4. Tạo auth_bloc_test.dart - Migrate existing tests
```

### Task 1.3: Create BlocProvider Integration

Update app để sử dụng Bloc:

```dart
// TODO: Integrate Bloc với app
// 1. Add BlocProvider vào widget tree
// 2. Replace ConsumerWidget → BlocBuilder
// 3. Update ref.read() calls → context.read()
// 4. Update ref.watch() calls → BlocBuilder
```

### Task 1.4: Compare Implementation

So sánh hai implementations:

```markdown
## Migration Report

### Lines of Code
- Riverpod implementation: XXX lines
- Bloc implementation: XXX lines

### Pros/Cons
- Riverpod: ...
- Bloc: ...

### Recommendation
```

---

## Exercise 2: GraphQL Integration 📊

**Mục tiêu:** Setup GraphQL client và implement queries.

### Setup

```bash
flutter pub add graphql graphql_flutter
```

### Task 2.1: Setup GraphQL Client

Tạo `lib/data_source/graphql/graphql_service.dart`:

```dart
// TODO: Setup GraphQL client với:
// - HttpLink to GraphQL endpoint
// - InMemoryCache
// - Error handling
```

### Task 2.2: Create GraphQL Schema

Tạo mock GraphQL schema:

```graphql
# lib/data_source/graphql/schema.graphql
type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): UserList!
}

type Mutation {
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

type User {
  id: ID!
  name: String!
  email: String!
  avatarUrl: String
  createdAt: String!
}

type UserList {
  items: [User!]!
  totalCount: Int!
  hasMore: Boolean!
}

input UpdateUserInput {
  name: String
  email: String
}
```

### Task 2.3: Implement Queries

Tạo `lib/data_source/graphql/user_repository.dart`:

```dart
// TODO: Implement GraphQL operations:
// 1. getUser(id) - query
// 2. getUsers(limit, offset) - paginated query
// 3. updateUser(id, input) - mutation
// 4. Error handling
```

### Task 2.4: Integrate với UI

Update user profile page:

```dart
// TODO: Replace REST API calls với GraphQL queries
// - Use GraphQL client
// - Handle loading/error states
// - Implement optimistic updates for mutations
```

---

## Exercise 3: WebSocket Real-time Service 🔌

**Mục tiêu:** Implement WebSocket service cho real-time chat.

### Setup

```bash
flutter pub add web_socket_channel
```

### Task 3.1: Create WebSocket Service

Tạo `lib/data_source/websocket/websocket_service.dart`:

```dart
// TODO: Implement WebSocket service với:
// 1. connect(url) - establish connection
// 2. disconnect() - close connection
// 3. send(message) - send JSON message
// 4. onMessage - stream of incoming messages
// 5. Reconnection logic với exponential backoff
```

### Task 3.2: Create Chat Sync Service

Tạo `lib/data_source/websocket/chat_sync_service.dart`:

```dart
// TODO: Implement chat synchronization:
// 1. Listen to message.created events
// 2. Listen to message.updated events
// 3. Listen to message.deleted events
// 4. Update local repository
// 5. Emit sync events
```

### Task 3.3: Implement Optimistic Updates

```dart
// TODO: Implement optimistic UI:
// 1. Show message immediately when sent
// 2. Update status: sending → sent → delivered
// 3. Rollback on failure
// 4. Handle reconnection sync
```

### Task 3.4: Test WebSocket

```dart
// TODO: Create WebSocket tests:
// 1. Connection test
// 2. Reconnection test
// 3. Message delivery test
// 4. Error handling test
```

---

## Exercise 4: Melos Monorepo Setup 🏗️

**Mục tiêu:** Setup Melos monorepo cho multi-package project.

### Setup

```bash
# Install Melos globally
dart pub global activate melos

# Create project structure
mkdir -p my_monorepo/{packages,apps}
```

### Task 4.1: Create melos.yaml

Tạo `melos.yaml`:

```yaml
# TODO: Configure melos.yaml với:
# - name: my_monorepo
# - packages: packages/**
# - scripts:
#   - bootstrap
#   - analyze
#   - test
#   - build
#   - clean
# - command hooks
```

### Task 4.2: Create Core Package

Tạo `packages/core/` structure:

```
packages/core/
├── pubspec.yaml
├── lib/
│   ├── core.dart
│   ├── constants/
│   ├── extensions/
│   └── utils/
└── test/
```

### Task 4.3: Create Feature Package

Tạo `packages/features/auth/` structure:

```
packages/features/auth/
├── pubspec.yaml
├── lib/
│   ├── auth.dart
│   ├── models/
│   ├── repositories/
│   └── providers/
└── test/
```

### Task 4.4: Create App Package

Tạo `apps/my_app/` structure:

```
apps/my_app/
├── pubspec.yaml
├── lib/
│   └── main.dart
├── android/
├── ios/
└── test/
```

### Task 4.5: Test Melos Commands

```bash
# Install melos globally (required first time)
dart pub global activate melos

# Bootstrap all packages
melos bootstrap

# Run analyze on all
melos analyze

# Run tests
melos test

# Clean and rebuild
melos clean && melos bootstrap
```

---

## Exercise 5: Mutation Testing 🧪

**Mục tiêu:** Add mutation testing để validate test quality.

### Setup

```bash
flutter pub add --dev mutation_testing mutation_testing_commons
```

### Task 5.1: Configure Mutation Testing

Tạo `mutation_testing.yaml`:

```yaml
# TODO: Configure mutation testing:
# - package: (project name)
# - timeout: 60 seconds
# - excluded_mutation_operators:
# - excluded_lines:
# - report format
```

### Task 5.2: Run Mutation Tests

```bash
# Run mutation testing
dart run mutation_testing mutation_testing.yaml
```

### Task 5.3: Analyze Results

Review mutation testing report:

```markdown
## Mutation Testing Report

### Coverage
- Line coverage: XX%
- Mutation score: XX%

### Survived Mutations (needs attention)
1. ...

### Killed Mutations
1. ...

### Recommendations
```

### Task 5.4: Improve Weak Tests

Fix tests that let mutations survive:

```dart
// TODO: Review và improve tests:
// 1. Add missing assertions
// 2. Cover edge cases
// 3. Test error paths
// 4. Re-run mutation testing
```

---

## Bonus Challenges ⭐

### Bonus 1: GraphQL Subscriptions

Implement real-time subscriptions:

```dart
// TODO: Add GraphQL subscription:
// - messageCreated subscription
// - messageUpdated subscription
// - Connect với WebSocket link
```

### Bonus 2: Melos Version Management

Setup automatic versioning:

```yaml
# TODO: Configure melos version:
// - branch: main
// - publishToGit: false
// - workspace versioning
```

### Bonus 3: Custom build_runner Generator

Create custom code generator:

```dart
// TODO: Create generator cho:
// - Custom annotations
// - Generator class
// - build.yaml configuration
// - build_runner integration
```

---

## Submission

1. **Tạo PR** với title: `feat(patterns): Bloc migration, GraphQL, WebSocket, Melos`
2. **Kiểm tra:**
   - [ ] Bloc migration complete (Riverpod → Bloc)
   - [ ] GraphQL client setup và queries working
   - [ ] WebSocket service với reconnection logic
   - [ ] Melos monorepo bootstrap successful
   - [ ] Mutation testing report generated
3. **Demo:** Show each pattern/tool working

---

## Hints

### Hint 1: Bloc vs Riverpod

```
Để migrate:
1. Events = Actions (what happened)
2. States = State (what to show)
3. Bloc methods = Notifier methods

Main difference: Bloc dùng emit(state), Riverpod dùng state = ...
```

### Hint 2: GraphQL vs REST

```
GraphQL advantages:
- Single endpoint
- Request exactly what you need
- Strong typing với schema

Choose GraphQL when:
- Multiple platforms (web, mobile)
- Complex data relationships
- Need real-time subscriptions
```

### Hint 3: WebSocket Reconnection

```
Exponential backoff formula:
delay = min(baseDelay * 2^attempt, maxDelay)

VD: baseDelay = 1s
Attempt 0: 1s
Attempt 1: 2s
Attempt 2: 4s
Attempt 3: 8s
...
```

### Hint 4: Melos Bootstrap

```
Nếu bootstrap fails:
1. Kiểm tra pubspec.yaml syntax
2. Verify dependency paths
3. Run melos clean → melos bootstrap
```

---

→ Tiếp theo: [04-verify.md](./04-verify.md)

<!-- AI_VERIFY: generation-complete -->
