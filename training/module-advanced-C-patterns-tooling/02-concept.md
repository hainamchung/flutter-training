# Concepts — Advanced Patterns & Tooling

> 📌 **Module context:** Module này survey 6 concepts chính về advanced patterns và tooling trong Flutter, mapping từ code đã đọc ở [01-code-walk](./01-code-walk.md). Mỗi concept kèm FE bridge cho dev có background React/JavaScript.

---

## Concept 1: Bloc vs Riverpod — State Management Comparison 🔴 MUST-KNOW

**WHY:** Chọn đúng state management approach giúp maintainability và team productivity.

### Architecture Comparison

```
┌────────────────────────────────────────────────────────────────┐
│                         BLOC PATTERN                            │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Action ──▶ Event ──▶ Bloc ──▶ Emit ──▶ State ──▶ UI    │
│                          │                                     │
│                          ▼                                     │
│                    Repository ──▶ DataSource                   │
│                                                                 │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│                        RIVERPOD PATTERN                          │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Provider ──▶ (logic) ──▶ State ──▶ Consumer ──▶ UI          │
│       │                                                         │
│       └── Repository ──▶ DataSource                            │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Detailed Comparison

| Aspect | Bloc | Riverpod |
|--------|------|----------|
| **Philosophy** | Event-driven, strict | Provider-driven, flexible |
| **Boilerplate** | High (3+ files per feature) | Low (1-2 files) |
| **Learning curve** | Steeper | Gentler |
| **Testing** | Easy with `blocTest` | Easy with `ProviderScope` override |
| **DevTools** | BlocObserver built-in | Provider DevTools extension |
| **Code generation** | `bloc` package (optional) | `riverpod_generator` (optional) |
| **Flutter pub score** | 100% | 100% |

### Bloc Code Structure

```dart
// Traditional Bloc structure (5+ files)
lib/
├── bloc/
│   ├── auth/
│   │   ├── auth_bloc.dart       // Main bloc logic
│   │   ├── auth_event.dart     // Events
│   │   └── auth_state.dart     // States
│   └── ...
```

### Riverpod Code Structure

```dart
// Riverpod structure (1-2 files per feature)
lib/
├── providers/
│   ├── auth_provider.dart      // All auth-related providers
│   └── ...
```

### Migration: Riverpod to Bloc

```dart
// Riverpod Provider
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  Future<AuthState> build() async {
    return await _authRepository.getCurrentUser();
  }
  
  Future<void> login(String email, String password) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => 
      _authRepository.login(email, password)
    );
  }
}

// Equivalent Bloc
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc(): super(AuthInitial()) {
    on<AuthCheckRequested>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await _authRepository.getCurrentUser();
        emit(user != null ? AuthAuthenticated(user) : AuthUnauthenticated());
      } catch (e) {
        emit(AuthError(e.toString()));
      }
    });
    
    on<AuthLoginRequested>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await _authRepository.login(event.email, event.password);
        emit(AuthAuthenticated(user));
      } catch (e) {
        emit(AuthError(e.toString()));
      }
    });
  }
}
```

### When to Choose Which?

| Scenario | Recommended |
|----------|-------------|
| Small team, quick prototyping | Riverpod |
| Large team, strict patterns needed | Bloc |
| Team has Redux/Angular background | Bloc |
| Need minimal boilerplate | Riverpod |
| Complex state machines | Bloc |
| Simple CRUD apps | Riverpod |

> 💡 **FE Perspective**
> **Flutter:** Bloc ≈ Redux (event → action → reducer → state); Riverpod ≈ Context + hooks.
> **React/Vue tương đương:** Bloc = Redux Toolkit; Riverpod = Zustand / Jotai.
> **Khác biệt quan trọng:** Flutter Riverpod có better dependency injection built-in.

---

## Concept 2: GraphQL Architecture 🟡 SHOULD-KNOW

**WHY:** GraphQL provides flexibility trong data fetching so với REST.

### GraphQL vs REST

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Data fetching** | Multiple endpoints | Single endpoint |
| **Over-fetching** | Common | Avoided |
| **Under-fetching** | Multiple calls | Avoided |
| **Caching** | HTTP caching | Normalized cache |
| **Real-time** | Polling / WebSocket | Subscriptions |
| **Learning curve** | Gentle | Steeper |
| **Tooling** | Standard | GraphiQL/Playground |

### GraphQL Operations

```dart
// Query - Read data
const getUserQuery = '''
  query GetUser(\$id: ID!) {
    user(id: \$id) {
      id
      name
      email
    }
  }
''';

// Mutation - Write data
const createUserMutation = '''
  mutation CreateUser(\$input: CreateUserInput!) {
    createUser(input: \$input) {
      id
      name
    }
  }
''';

// Subscription - Real-time
const onMessageSubscription = '''
  subscription OnMessageCreated(\$conversationId: ID!) {
    messageCreated(conversationId: \$conversationId) {
      id
      content
      createdAt
    }
  }
''';
```

### Cache Strategies

```dart
// Fetch policies
enum FetchPolicy {
  cacheFirst,      // Try cache first, network if missing
  cacheAndNetwork, // Return cached, then update from network
  networkOnly,     // Always fetch from network
  cacheOnly,       // Only return from cache
}

// Normalized cache example
final cache = InMemoryCache(
  dataIdFromObject: (object) {
    if (object is User) return 'User:\${object.id}';
    if (object is Post) return 'Post:\${object.id}';
    return null;
  },
);
```

### Data Normalization

```dart
// Without normalization (nested)
{
  "user": {
    "name": "John",
    "posts": [
      {"title": "Post 1"},
      {"title": "Post 2"}
    ]
  }
}

// With normalization (flattened)
{
  "User:1": {"name": "John"},
  "Post:1": {"title": "Post 1", "authorId": "User:1"},
  "Post:2": {"title": "Post 2", "authorId": "User:1"}
}
```

> 💡 **FE Perspective**
> **Flutter:** `graphql_flutter` provides Apollo-like client.
> **React/Vue tương đương:** Apollo Client, urql.
> **Khác biệt quan trọng:** Flutter GraphQL client tương tự web clients.

---

## Concept 3: WebSocket Real-time Communication 🟡 SHOULD-KNOW

**WHY:** WebSocket provides full-duplex communication cho real-time features.

### WebSocket vs HTTP

| Aspect | HTTP | WebSocket |
|--------|------|-----------|
| **Connection** | Request-response | Persistent |
| **Bidirectional** | No (client initiates) | Yes |
| **Overhead** | Headers every request | Minimal after handshake |
| **Use case** | REST APIs | Real-time updates |
| **Reconnection** | Automatic | Manual handling |

### Message Types

```dart
// JSON message format
{
  "type": "message.created",
  "payload": {
    "id": "msg_123",
    "content": "Hello!",
    "conversationId": "conv_456",
    "senderId": "user_789",
    "createdAt": "2024-01-01T00:00:00Z"
  },
  "timestamp": 1704067200000
}

// System messages
{
  "type": "ping",
  "timestamp": 1704067200000
}

// Acknowledgment
{
  "type": "ack",
  "messageId": "msg_123",
  "status": "delivered"
}
```

### Reconnection Patterns

```dart
// Exponential backoff
class ReconnectionStrategy {
  static const maxAttempts = 10;
  static const baseDelay = Duration(seconds: 1);
  static const maxDelay = Duration(minutes: 5);
  
  Duration calculateDelay(int attempt) {
    if (attempt >= maxAttempts) return Duration.zero;
    
    final delay = baseDelay * (1 << attempt); // Exponential
    return delay > maxDelay ? maxDelay : delay;
  }
}
```

### State Synchronization

```dart
// Optimistic updates with rollback
class ChatSyncService {
  Future<void> sendMessage(Message message) async {
    // 1. Optimistic update
    _chatRepository.addMessage(message.copyWith(
      status: MessageStatus.sending,
    ));
    
    try {
      // 2. Send via WebSocket
      _wsService.send({
        'type': 'message.send',
        'payload': message.toJson(),
      });
      
      // 3. Update status on ACK
      _wsService.onAck(message.id, () {
        _chatRepository.updateMessageStatus(
          message.id,
          MessageStatus.sent,
        );
      });
    } catch (e) {
      // 4. Rollback on error
      _chatRepository.updateMessageStatus(
        message.id,
        MessageStatus.failed,
      );
    }
  }
}
```

> 💡 **FE Perspective**
> **Flutter:** `web_socket_channel` wraps native WebSocket.
> **React/Vue tương đương:** Native WebSocket API, Socket.io.
> **Khác biệt quan trọng:** Flutter needs manual reconnection; web Socket.io handles automatically.

---

## Concept 4: Melos Monorepo 🟡 SHOULD-KNOW

**WHY:** Melos helps manage multiple packages in single repository efficiently.

### Monorepo Benefits

| Benefit | Description |
|---------|-------------|
| **Code sharing** | Common packages shared across apps |
| **Consistent tooling** | Single CI/CD, lint, test config |
| **Atomic changes** | Change shared code in one PR |
| **Dependency management** | Melos handles pub get across packages |

### Package Types

```yaml
# Types of packages in monorepo

# Core package - shared utilities
packages/core:
  - Constants
  - Extensions
  - Utilities

# Shared UI - reusable widgets
packages/shared/ui:
  - Button components
  - Form fields
  - Layout widgets

# Feature packages - domain-specific
packages/features/auth:
  - Login/Register pages
  - Auth providers
  - Auth models

packages/features/profile:
  - Profile pages
  - Profile providers
```

### Workspace Dependencies

```yaml
# apps/app/pubspec.yaml
dependencies:
  core: ^1.0.0              # Package with version
  shared_ui:
    path: ../shared/ui      # Local path
  features_auth:
    path: ../features/auth
```

### CI/CD with Melos

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          
      - uses: dart-lang/setup-dart@v1
      
      - name: Setup Melos
        run: dart pub global activate melos
      
      - name: Bootstrap
        run: melos bootstrap
        
      - name: Analyze
        run: melos analyze
        
      - name: Test
        run: melos test
```

> 💡 **FE Perspective**
> **Flutter:** Melos là Nx cho Flutter.
> **React/Vue tương đương:** Nx, Turborepo, Lerna.
> **Khác biệt quan trọng:** Melos optimized cho Flutter/Dart ecosystem.

---

## Concept 5: Custom build_runner Generators 🟡 SHOULD-KNOW

**WHY:** Custom code generators reduce boilerplate và enforce consistency.

### Generator Use Cases

| Use Case | Example |
|----------|---------|
| **Annotation-based** | `@immutable`, `@copyWith` |
| **Schema-driven** | GraphQL → Dart types |
| **Template-based** | API client from OpenAPI spec |
| **Convention enforcement** | Check naming, structure |

### Generator Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                     GENERATOR ARCHITECTURE                      │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Source Files ──▶ LibraryReader ──▶ Generator ──▶ Output       │
│         │                                    │                  │
│         │                                    ▼                  │
│         │                            Generated Files            │
│         │                            (.g.dart files)            │
│         │                                    │                  │
│         └─────────────── BuildStep ◀────────┘                  │
│                              │                                 │
│                         .g.dart                                │
└────────────────────────────────────────────────────────────────┘
```

### Example: Annotation-Based Generator

```dart
// Define annotation
class JsonSerializable extends Annotation {
  const JsonSerializable();
}

// Create annotation reader
class JsonSerializableGenerator extends Generator {
  @override
  Future<String> generate(LibraryReader library, BuildStep step) async {
    final buffer = StringBuffer();
    
    for (final classElement in library.classes) {
      final annotation = classElement.getAnnotation(const JsonSerializable());
      if (annotation != null) {
        buffer.writeln(_generateFromJson(classElement));
        buffer.writeln(_generateToJson(classElement));
      }
    }
    
    return buffer.toString();
  }
}

// Configure build.yaml
targets:
  $default:
    builders:
      my_generator:
        enabled: true
```

> 💡 **FE Perspective**
> **Flutter:** build_runner = Babel plugins + webpack loaders combined.
> **React/Vue tương đương:** TypeScript generators, webpack plugins.
> **Khác biệt quan trọng:** Dart generators use Dart annotations, more type-safe.

---

## Concept 6: Mutation Testing 🟡 SHOULD-KNOW

**WHY:** Mutation testing validates test quality beyond code coverage.

### How Mutation Testing Works

```
┌────────────────────────────────────────────────────────────────┐
│                   MUTATION TESTING FLOW                         │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Original Code                                               │
│     if (user.isActive && user.age > 18)                        │
│                                                                 │
│  2. Generate Mutations                                          │
│     ├─ Changed operator: && → ||                               │
│     ├─ Changed value: 18 → 17                                  │
│     ├─ Removed condition: user.isActive                         │
│     └─ Changed comparison: > → <                               │
│                                                                 │
│  3. Run Tests Against Mutations                                 │
│     ├─ Mutation survives (test passes) → Test is WEAK           │
│     └─ Mutation killed (test fails) → Test is STRONG            │
│                                                                 │
│  4. Mutation Score = Killed / Total                             │
│     Score > 80% = Good test suite                               │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Mutation Testing in Dart

```bash
# Add mutation_testing package
flutter pub add --dev mutation_testing
```

```yaml
# mutation_testing.yaml
package:
  excluded_mutation_operators:
    - RemoveControlStructures
  excluded_lines:
    - "*.g.dart"
    - "lib/generated/*"
  report:
    terminal: true
    html: true
```

```dart
// mutation_testing_config.dart
// Note: mutation_test uses built-in transformers — no import needed
// Example configuration (all transformers are built-in):

const reporters = [
  ConsoleReporter(),
  HtmlReporter(),
];
```

### Interpreting Results

| Mutation Score | Rating | Action |
|---------------|--------|--------|
| > 90% | Excellent | Great test suite |
| 80-90% | Good | Minor improvements |
| 60-80% | Adequate | Review surviving mutations |
| < 60% | Poor | Improve tests significantly |

> 💡 **FE Perspective**
> **Flutter:** `mutation_testing` package.
> **React/Vue tương đương:** Stryker.NET, PITest (Java), Mutpy (Python).
> **Khác biệt quan trọng:** Dart mutation testing still evolving.

---

## Concept Map — How They Connect

```
Concept 1: State Management → Choose Bloc vs Riverpod
Concept 2: GraphQL          → API layer alternative
Concept 3: WebSocket       → Real-time communication
Concept 4: Melos           → Project structure
Concept 5: build_runner    → Code generation
Concept 6: Mutation Testing → Test quality validation
```

**Advanced Flutter Architecture:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADVANCED ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐     │
│  │   App       │      │   Features  │      │   Shared    │     │
│  │  (Melos)    │◀────▶│  (Melos)    │◀────▶│  (Melos)    │     │
│  └─────────────┘      └─────────────┘      └─────────────┘     │
│        │                    │                    │               │
│        │                    │                    │               │
│        ▼                    ▼                    ▼               │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐     │
│  │   Riverpod/ │      │    GraphQL  │      │  build_runner│     │
│  │    Bloc     │      │   / REST    │      │  (Generators)│     │
│  └─────────────┘      └─────────────┘      └─────────────┘     │
│        │                    │                    │               │
│        ▼                    ▼                    ▼               │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    WebSocket (Real-time)                    │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │               Mutation Testing (Test Quality)               │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

📖 [Glossary](../_meta/glossary.md)

<!-- AI_VERIFY: generation-complete -->
