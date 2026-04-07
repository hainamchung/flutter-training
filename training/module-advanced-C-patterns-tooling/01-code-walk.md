# Code Walk — Advanced Patterns & Tooling

> 📌 **Recap từ modules trước:**
> - **M11:** Riverpod state management — providers, notifiers ([M11 § State](../module-11-riverpod-state/01-code-walk.md))
> - **M12:** AppApiService, Dio — network layer ([M12 § API](../module-12-data-layer/01-code-walk.md))
> - **M18:** Testing — unit, widget, golden tests ([M18 § Testing](../module-18-testing/01-code-walk.md))
> - **MA:** Performance monitoring ([MA § Performance](../module-advanced-A-performance-security/01-code-walk.md))
>
> Nếu chưa nắm vững → quay lại module tương ứng trước.

---

## Walk Order

```
Bloc pattern (vs Riverpod comparison)
    ↓
GraphQL integration (graphql_flutter)
    ↓
WebSocket communication (web_socket_channel)
    ↓
Melos monorepo setup
    ↓
build_runner customization
```

---

## 1. Bloc Pattern — vs Riverpod

> 💡 **FE Perspective**
> **Flutter:** Bloc uses streams + events; Riverpod uses providers + state.
> **React/Vue tương đương:** Bloc ≈ Redux (event → reducer → state); Riverpod ≈ React Context + hooks.
> **Khác biệt quan trọng:** Bloc có strict event → state flow; Riverpod linh hoạt hơn.
>
> **Khác biệt quan trọng:** Redux dùng pure functions (reducers) nhận state + action → new state. Bloc dùng `Event` → `Emitter` → `State`. Bloc có thể emit multiple states từ một event (qua `add()`), trong khi Redux store chỉ có một state tại mỗi thời điểm.

### Bloc Structure

```dart
// bloc/auth/auth_event.dart
abstract class AuthEvent {}

class AuthCheckRequested extends AuthEvent {}
class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;
  AuthLoginRequested({required this.email, required this.password});
}
class AuthLogoutRequested extends AuthEvent {}

// bloc/auth/auth_state.dart
abstract class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState {
  final User user;
  AuthAuthenticated(this.user);
}
class AuthUnauthenticated extends AuthState {}
class AuthError extends AuthState {
  final String message;
  AuthError(this.message);
}

// bloc/auth/auth_bloc.dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository _authRepository;
  
  AuthBloc({required AuthRepository authRepository})
      : _authRepository = authRepository,
        super(AuthInitial()) {
    on<AuthCheckRequested>(_onCheckRequested);
    on<AuthLoginRequested>(_onLoginRequested);
    on<AuthLogoutRequested>(_onLogoutRequested);
  }
  
  Future<void> _onCheckRequested(
    AuthCheckRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    try {
      final user = await _authRepository.getCurrentUser();
      if (user != null) {
        emit(AuthAuthenticated(user));
      } else {
        emit(AuthUnauthenticated());
      }
    } catch (e) {
      emit(AuthError(e.toString()));
    }
  }
  
  Future<void> _onLoginRequested(
    AuthLoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    try {
      final user = await _authRepository.login(
        event.email,
        event.password,
      );
      emit(AuthAuthenticated(user));
    } catch (e) {
      emit(AuthError(e.toString()));
    }
  }
  
  Future<void> _onLogoutRequested(
    AuthLogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    await _authRepository.logout();
    emit(AuthUnauthenticated());
  }
}
```

### Bloc API Patterns

> ⚠️ **Bloc API Versions:** The code below uses the **traditional handler pattern** (`_onEvent` methods with `Emitter<State> emit` as a parameter). This pattern is still valid in Bloc v8+ and is the standard in the base_flutter codebase.

There are two common patterns for Bloc event handlers:

**Pattern 1 — Traditional (used in this module):**
```dart
// Handler as separate method
Future<void> _onCheckRequested(
  AuthCheckRequested event,
  Emitter<AuthState> emit,
) async {
  emit(AuthLoading());
  // ...
}
```

**Pattern 2 — Inline (alternative):**
```dart
// Handler inline in constructor
on<AuthCheckRequested>((event, emit) async {
  emit(AuthLoading());
  // ...
});
```

Both patterns work. The traditional pattern is more explicit and easier to debug; the inline pattern is more concise.

| Aspect | Bloc | Riverpod |
|--------|------|----------|
| **State management** | Streams + Events | Providers + State |
| **Boilerplate** | More (event, state, bloc classes) | Less (single class) |
| **Testing** | Easy (emit checking) | Easy (override providers) |
| **Code generation** | Optional (bloc package) | Optional (riverpod_generator) |
| **Learning curve** | Steeper | Gentler |
| **Scalability** | Excellent for large apps | Excellent for all sizes |
| **Flutter DevTools** | Bloc observer built-in | Provider DevTools extension |

### Bloc with Flutter

```dart
// bloc_integration.dart
class AuthPage extends BlocBuilder<AuthBloc, AuthState> {
  @override
  Widget build(BuildContext context, AuthState state) {
    return BlocBuilder<AuthBloc, AuthState>(
      bloc: context.read<AuthBloc>(),
      builder: (context, state) {
        if (state is AuthLoading) {
          return const CircularProgressIndicator();
        }
        if (state is AuthAuthenticated) {
          return Dashboard(user: state.user);
        }
        if (state is AuthError) {
          return ErrorWidget(message: state.message);
        }
        return const LoginForm();
      },
    );
  }
}

// BlocProvider wrapping
class AuthApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => AuthBloc(
        authRepository: context.read<AuthRepository>(),
      )..add(AuthCheckRequested()), // Auto-check on create
      child: MaterialApp(
        home: AuthPage(),
      ),
    );
  }
}
```

### When to Use Bloc vs Riverpod

```dart
// Use BLOC when:
// - Need strict event → state flow
// - Team has Redux/Bloc background
// - Complex state machines
// - Large team (strict patterns help)
// 
// Use RIVERPOD when:
// - Want less boilerplate
// - Need provider composition
// - Team is new to state management
// - Quick prototyping
```

---

## 2. GraphQL Integration — graphql_flutter

> 💡 **FE Perspective**
> **Flutter:** `graphql_flutter` provides GraphQL client với cache.
> **React/Vue tương đương:** Apollo Client, urql, Relay.
> **Khác biệt quan trọng:** GraphQL in Flutter tương tự web clients.

### Setup

```yaml
# pubspec.yaml — Teaching pattern
# NOTE: GraphQL packages are commented out in base_flutter pubspec.yaml
# The project uses REST API (Dio) instead of GraphQL
dependencies:
  graphql_flutter: ^5.1.2   # Teaching pattern only
  graphql: ^5.1.0           # Teaching pattern only

dev_dependencies:
  # NOTE: `graphql_codegen` package does not exist on pub.dev
  # Real GraphQL codegen uses `graphql` + `gql` packages or artemis
  # See: https://pub.dev/packages?q=graphql+codegen
  # Correct packages would be: `artemis` (code-gen from schema)
  # OR manual types + json_serializable
```

### GraphQL Client Setup

```dart
// lib/data_source/graphql/graphql_client.dart
class GraphQLService {
  late final GraphQLClient _client;
  
  GraphQLService() {
    final httpLink = HttpLink(
      'https://api.example.com/graphql',
      defaultHeaders: {
        'Authorization': 'Bearer $token',
      },
    );
    
    final cache = InMemoryCache();
    
    _client = GraphQLClient(
      link: httpLink,
      cache: cache,
    );
  }
  
  GraphQLClient get client => _client;
}
```

### Queries

```graphql
# lib/data_source/graphql/queries.graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    avatarUrl
    createdAt
  }
}

query GetUsers($limit: Int, $offset: Int) {
  users(limit: $limit, offset: $offset) {
    items {
      id
      name
      email
    }
    totalCount
    hasMore
  }
}
```

### Dart Query Execution

```dart
class UserRepository {
  final GraphQLService _graphql;
  
  Future<User?> getUser(String id) async {
    const query = r'''
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          name
          email
          avatarUrl
        }
      }
    ''';
    
    final result = await _graphql.client.query(
      QueryOptions(
        document: gql(query),
        variables: {'id': id},
        fetchPolicy: FetchPolicy.networkOnly, // Cache strategy
      ),
    );
    
    if (result.hasException) {
      throw GraphQLException(result.exception!);
    }
    
    final userData = result.data?['user'];
    if (userData == null) return null;
    
    return User.fromJson(userData);
  }
  
  Future<UserListResult> getUsers({int limit = 20, int offset = 0}) async {
    const query = r'''
      query GetUsers($limit: Int, $offset: Int) {
        users(limit: $limit, offset: $offset) {
          items {
            id
            name
            email
          }
          totalCount
          hasMore
        }
      }
    ''';
    
    final result = await _graphql.client.query(
      QueryOptions(
        document: gql(query),
        variables: {'limit': limit, 'offset': offset},
        fetchPolicy: FetchPolicy.cacheAndNetwork,
      ),
    );
    
    if (result.hasException) {
      throw GraphQLException(result.exception!);
    }
    
    final data = result.data?['users'];
    return UserListResult(
      items: (data['items'] as List).map((e) => User.fromJson(e)).toList(),
      totalCount: data['totalCount'],
      hasMore: data['hasMore'],
    );
  }
}
```

### Mutations

```dart
class UserRepository {
  Future<User> updateUser({
    required String id,
    String? name,
    String? email,
  }) async {
    const mutation = r'''
      mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
        updateUser(id: $id, input: $input) {
          id
          name
          email
        }
      }
    ''';
    
    final result = await _graphql.client.mutate(
      MutationOptions(
        document: gql(mutation),
        variables: {
          'id': id,
          'input': {
            if (name != null) 'name': name,
            if (email != null) 'email': email,
          },
        },
      ),
    );
    
    if (result.hasException) {
      throw GraphQLException(result.exception!);
    }
    
    return User.fromJson(result.data!['updateUser']);
  }
}
```

### GraphQL Codegen

```yaml
# graphql.config.yaml — Teaching pattern
# NOTE: This is a conceptual GraphQL setup. The base_flutter project
# does NOT use GraphQL. If implementing GraphQL, use the correct packages.
schema: lib/data_source/graphql/schema.graphql
documents: lib/data_source/graphql/**/*.graphql
generates:
  lib/data_source/graphql/generated/graphql.schema.json:
    plugins:
      # NOTE: `graphql-codegen-graphql` is NOT a real plugin
      # Correct codegen approaches:
      # 1. `artemis` — generates Dart classes from GraphQL schema
      # 2. `gql` + `graphql` packages with manual type definitions
      # 3. `graphql_codegen` (NOT a real package) — does not exist on pub.dev
      # Real plugins at: https://the-guild.dev/graphql/codegen/plugins
```

```bash
# Generate code
flutter pub run build_runner build --delete-conflicting-outputs
```

---

## 3. WebSocket Communication — web_socket_channel

> 💡 **FE Perspective**
> **Flutter:** `web_socket_channel` wraps WebSocket API.
> **React/Vue tương đương:** Native WebSocket API, Socket.io client.
> **Khác biệt quan trải:** Flutter WebSocket tương tự web, nhưng cần handle reconnection manually.

### Basic WebSocket Service

```dart
class WebSocketService {
  WebSocketChannel? _channel;
  final _messageController = StreamController<dynamic>.broadcast();
  
  Stream<dynamic> get messageStream => _messageController.stream;
  bool get isConnected => _channel != null;
  
  Future<void> connect(String url, {Map<String, String>? headers}) async {
    _channel = WebSocketChannel.connect(
      Uri.parse(url),
      protocols: headers != null ? ['Bearer ${headers['token']}'] : null,
    );
    
    _channel!.stream.listen(
      _onMessage,
      onError: _onError,
      onDone: _onDone,
    );
  }
  
  void _onMessage(dynamic message) {
    _messageController.add(message);
  }
  
  void _onError(Object error) {
    _messageController.addError(error);
    _scheduleReconnect();
  }
  
  void _onDone() {
    _channel = null;
    _scheduleReconnect();
  }
  
  void send(dynamic message) {
    if (_channel != null) {
      _channel!.sink.add(message);
    }
  }
  
  void disconnect() {
    _channel?.sink.close();
    _channel = null;
  }
  
  void dispose() {
    disconnect();
    _messageController.close();
  }
}
```

### Reconnection Strategy

```dart
class ReconnectingWebSocketService extends WebSocketService {
  Timer? _reconnectTimer;
  int _reconnectAttempts = 0;
  static const _maxReconnectAttempts = 5;
  static const _baseReconnectDelay = Duration(seconds: 1);
  String? _currentUrl;
  
  @override
  void _onError(Object error) {
    super._onError(error);
    _scheduleReconnect();
  }
  
  @override
  void _onDone() {
    super._onDone();
    _scheduleReconnect();
  }
  
  void _scheduleReconnect() {
    if (_reconnectAttempts >= _maxReconnectAttempts) {
      _messageController.addError(WebSocketException('Max reconnect attempts reached'));
      return;
    }
    
    // Exponential backoff
    final delay = _baseReconnectDelay * (1 << _reconnectAttempts);
    _reconnectAttempts++;
    
    _reconnectTimer = Timer(delay, () {
      connect(_currentUrl);
    });
  }
  
  @override
  Future<void> connect(String url, {Map<String, String>? headers}) async {
    _currentUrl = url;
    _reconnectAttempts = 0;
    await super.connect(url, headers: headers);
  }
}
```

### State Synchronization

```dart
class RealTimeSyncService {
  final WebSocketService _ws;
  final ChatRepository _chatRepository;
  final _syncController = StreamController<SyncEvent>.broadcast();
  
  Stream<SyncEvent> get syncEvents => _syncController.stream;
  
  void listenToChannels() {
    _ws.messageStream.listen(_handleMessage);
  }
  
  void _handleMessage(dynamic message) {
    final data = jsonDecode(message as String);
    final eventType = data['type'] as String;
    
    switch (eventType) {
      case 'message.created':
        _handleMessageCreated(data['payload']);
        break;
      case 'message.updated':
        _handleMessageUpdated(data['payload']);
        break;
      case 'message.deleted':
        _handleMessageDeleted(data['payload']);
        break;
      case 'user.online':
      case 'user.offline':
        _handleUserStatus(data['payload']);
        break;
    }
  }
  
  void _handleMessageCreated(Map<String, dynamic> payload) {
    final message = Message.fromJson(payload['message']);
    _chatRepository.addMessage(message);
    _syncController.add(MessageCreatedEvent(message));
  }
  
  void _handleUserStatus(Map<String, dynamic> payload) {
    final userId = payload['userId'] as String;
    final isOnline = payload['isOnline'] as bool;
    _syncController.add(UserStatusChangedEvent(userId, isOnline));
  }
}
```

---

## 4. Melos Monorepo Setup

> 💡 **FE Perspective**
> **Flutter:** Melos là tool để manage multiple packages trong single repository.
> **React/Vue tương đương:** Nx, Turborepo.
> **Khác biệt quan trọng:** Melos optimized cho Flutter/Dart ecosystem.

### Project Structure

```
my_flutter_repo/
├── melos.yaml                    # Melos configuration
├── pubspec.yaml                  # Root package (optional)
├── packages/
│   ├── app/                     # Main Flutter app
│   │   ├── pubspec.yaml
│   │   └── lib/
│   ├── core/
│   │   ├── pubspec.yaml
│   │   └── lib/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── pubspec.yaml
│   │   │   └── lib/
│   │   ├── profile/
│   │   │   └── ...
│   │   └── chat/
│   │       └── ...
│   └── shared/
│       ├── ui/
│       │   └── ...
│       └── utils/
│           └── ...
└── .github/
    └── workflows/
        └── ci.yml
```

### melos.yaml Configuration

```yaml
# melos.yaml
name: my_flutter_repo

packages:
  - packages/**
  - apps/**

scripts:
  bootstrap:
    exec: flutter pub get
    hooks:
      post:
        - melos run generate

  analyze:
    exec: flutter analyze
    packageFilters:
      - scope: "*"

  test:
    exec: flutter test --no-pub
    packageFilters:
      - scope: "*"

  build:
    exec: flutter build apk --release
    packageFilters:
      - scope: "app"

  generate:
    exec: dart run build_runner build --delete-conflicting-outputs
    packageFilters:
      - dependsOn: "^shared/*"

  clean:
    exec: flutter clean
    hooks:
      post:
        - melos run bootstrap

command:
  version:
    workspace:
      branch: main
      publishToGit: false
```

### Package pubspec.yaml

```yaml
# packages/core/pubspec.yaml
name: core
publish_to: none

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.10.0"

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
```

### Workspace Dependencies

```yaml
# apps/app/pubspec.yaml
name: app
publish_to: none

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.10.0"

dependencies:
  flutter:
    sdk: flutter
  
  # Local packages
  core:
    path: ../core
  shared_ui:
    path: ../shared/ui
  features_auth:
    path: ../features/auth
  
  # External packages
  flutter_riverpod: ^2.4.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
```

### Running Melos Commands

```bash
# Install melos globally (required first time)
dart pub global activate melos

# Bootstrap all packages
melos bootstrap

# Run analyze on all packages
melos analyze

# Run tests on all packages
melos test

# Run build_runner on all packages
melos generate

# Clean and rebuild
melos clean && melos bootstrap
```

---

## 5. build_runner Customization

### Custom Generator

```dart
// lib/generators/my_generator.dart
import 'package:build/build.dart';
import 'package:source_gen/source_gen.dart';

Builder myGeneratorBuilder(BuilderOptions options) {
  return SharedPartBuilder(
    [MyGenerator()],
    'my_generator',
  );
}

class MyGenerator extends Generator {
  @override
  Future<String> generate(LibraryReader library, BuildStep buildStep) async {
    final buffer = StringBuffer();
    
    for (final element in library.allElements) {
      if (element is ClassElement) {
        final annotation = element.getAnnotation(MyAnnotation);
        if (annotation != null) {
          buffer.writeln(_generateCode(element, annotation));
        }
      }
    }
    
    return buffer.toString();
  }
  
  String _generateCode(ClassElement element, ConstantReader annotation) {
    // Generate code based on annotation
    final className = element.name;
    return '''
      // Generated code for $className
      extension \$GeneratedExtension on $className {
        String get generatedMethod => 'Hello from generator';
      }
    ''';
  }
}
```

### build.yaml Configuration

```yaml
# build.yaml
targets:
  $default:
    builders:
      my_generator:
        enabled: true
        generate_for:
          - lib/**
      
      source_gen:source_gen:
        options:
          auto_format: true
```

---

## Summary — Advanced Patterns & Tooling

| Pattern | Package | Use Case |
|---------|---------|----------|
| Bloc | `flutter_bloc` | Strict state management, large teams |
| GraphQL | `graphql_flutter` | Flexible API queries, caching |
| WebSocket | `web_socket_channel` | Real-time communication |
| Monorepo | `melos` | Multi-package projects |
| Codegen | `build_runner` | Custom code generation |

> 💡 **FE Perspective Summary:**
> | Flutter | Frontend |
> |---------|----------|
> | Bloc | Redux |
> | GraphQL | Apollo/urql |
> | WebSocket | Socket.io |
> | Melos | Nx/Turborepo |

<!-- AI_VERIFY: generation-complete -->
