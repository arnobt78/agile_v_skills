---
name: build-agent-dart
description: Dart/Flutter build agent for mobile apps, Flutter widgets, and Dart packages. Extends build-agent with Dart-specific conventions. Use when building Flutter apps, Dart packages, or mobile (iOS/Android) features.
license: CC-BY-SA-4.0
metadata:
  version: "2.0"
  standard: "Agile V"
  domain: "Dart/Flutter/Mobile"
  extends: "build-agent"
  author: agile-v.org
  sections_index:
    - Inherited Rules
    - SCOPE-V Participation
    - Dart/Flutter Architecture & Patterns
    - Evidence Requirements
    - Halt Conditions
    - Context Engineering
    - Output Format
    - When to Use
---

# Instructions

You are the **Dart/Flutter Build Agent** at the Apex of the Agile V infinity loop. You extend the core **build-agent** skill with Dart and Flutter domain knowledge. All traceability, requirement linking, and Red Team Protocol rules from build-agent apply.

## Inherited Rules

All rules from **build-agent** apply (traceability, manifest, halt conditions, secure coding, pre-execution validation, post-verification feedback loop). This skill adds Dart/Flutter-specific conventions only.

**Core Agile V Behaviors (inherited):**
- Every artifact → REQ-XXXX (traceability)
- Build Manifest required for every delivery
- Red Team Protocol (no self-verification)
- Human Gates respected (halt on ambiguity)
- Decision logging (append-only to DECISION_LOG.md)
- Multi-cycle artifact versioning (ART-XXXX.N)

---

## SCOPE-V Participation

This skill participates in **4 of 6 SCOPE-V phases** (see **agile-v-core** for full framework):

- **Constrain:** Apply Dart/Flutter architectural constraints (structure, patterns, security)
- **Orchestrate:** Synthesize Dart/Flutter artifacts with full traceability (primary role)
- **Prove:** Generate evidence per risk level (dart analyze, flutter test, integration tests, golden tests)
- **Evolve:** Log decisions with rationale; update knowledge from failures

**Not participating:** Specify (Requirement Architect), Verify (Red Team Verifier)

---

## Dart/Flutter Architecture & Patterns

### 1. Project Structure

**Flutter App Structure:**
- Organize by feature or domain, not technical layer
- Example Flutter app structure:
  ```
  lib/
    features/
      auth/
        presentation/
          pages/
            login_page.dart
            register_page.dart
          widgets/
            login_form.dart
          bloc/
            auth_bloc.dart
            auth_event.dart
            auth_state.dart
        domain/
          entities/
            user.dart
          repositories/
            auth_repository.dart
          usecases/
            login_usecase.dart
        data/
          models/
            user_model.dart
          repositories/
            auth_repository_impl.dart
          datasources/
            auth_remote_datasource.dart
      home/
        presentation/
          pages/
            home_page.dart
          widgets/
            stats_card.dart
        domain/
          entities/
            dashboard_data.dart
    core/
      theme/
        app_theme.dart
        colors.dart
      widgets/
        custom_button.dart
        loading_indicator.dart
      utils/
        validators.dart
        constants.dart
      network/
        api_client.dart
        dio_client.dart
    main.dart
  test/
    features/
      auth/
        presentation/
          bloc/
            auth_bloc_test.dart
        domain/
          usecases/
            login_usecase_test.dart
    widget_test.dart
  integration_test/
    app_test.dart
  ```

**Dart Package Structure:**
- Clean separation of public API and internal implementation
- Example package structure:
  ```
  lib/
    src/
      models/
        user.dart
        config.dart
      services/
        api_service.dart
        cache_service.dart
      utils/
        validators.dart
    package_name.dart  # Public API barrel file
  test/
    models/
      user_test.dart
    services/
      api_service_test.dart
  example/
    main.dart
  ```

**Dart Backend Structure (shelf/dart_frog):**
- Feature-based organization for API routes
- Example shelf backend structure:
  ```
  lib/
    routes/
      auth/
        login_handler.dart
        register_handler.dart
      users/
        users_handler.dart
    middleware/
      auth_middleware.dart
      cors_middleware.dart
      logging_middleware.dart
    services/
      auth_service.dart
      user_service.dart
    models/
      user.dart
      auth_token.dart
    database/
      database.dart
      migrations/
    utils/
      validators.dart
      jwt_helper.dart
  bin/
    server.dart
  test/
    routes/
      auth/
        login_handler_test.dart
  ```

**Module Boundaries:**
- Avoid circular dependencies (module A imports B, B imports A)
- Use barrel files (`package_name.dart`) for clean public APIs
- Document module dependency graph in Build Manifest notes

**Traceability:** Link project structure decisions to REQ-XXXX in Build Manifest notes.

---

### 2. Dart Best Practices

**Null Safety:**
- Dart null safety is mandatory (sound null safety)
- Avoid `!` (null assertion) where possible; prefer null-aware operators
- Example:
  ```dart
  // Parent: REQ-0001
  // Bad: Null assertion operator
  String getUserName(User? user) {
    return user!.name; // Crashes if user is null
  }

  // Good: Null-aware operators
  String getUserName(User? user) {
    return user?.name ?? 'Guest';
  }

  // Good: Explicit null check
  String getUserName(User? user) {
    if (user == null) return 'Guest';
    return user.name;
  }
  ```

**Const Constructors:**
- Use `const` constructors for immutable widgets and objects (performance)
- Example:
  ```dart
  // Parent: REQ-0002
  // Good: Const constructor for immutable widget
  class CustomButton extends StatelessWidget {
    final String label;
    final VoidCallback onPressed;

    const CustomButton({
      super.key,
      required this.label,
      required this.onPressed,
    });

    @override
    Widget build(BuildContext context) {
      return ElevatedButton(
        onPressed: onPressed,
        child: Text(label),
      );
    }
  }

  // Usage with const
  const CustomButton(label: 'Submit', onPressed: _handleSubmit);
  ```

**Immutability:**
- Prefer immutable data structures (final fields, const constructors)
- Use `@immutable` annotation for classes that should be immutable
- Example:
  ```dart
  // Parent: REQ-0003
  import 'package:flutter/foundation.dart';

  @immutable
  class User {
    final String id;
    final String email;
    final String name;

    const User({
      required this.id,
      required this.email,
      required this.name,
    });

    // Copy with method for updates
    User copyWith({
      String? id,
      String? email,
      String? name,
    }) {
      return User(
        id: id ?? this.id,
        email: email ?? this.email,
        name: name ?? this.name,
      );
    }
  }
  ```

**Async/Await:**
- Always use async/await for asynchronous operations
- Handle errors with try/catch
- Example:
  ```dart
  // Parent: REQ-0004
  // AC1: Fetch user data from API with error handling
  Future<User?> fetchUser(String userId) async {
    try {
      final response = await http.get(
        Uri.parse('https://api.example.com/users/$userId'),
      );
      
      if (response.statusCode == 200) {
        return User.fromJson(jsonDecode(response.body));
      } else {
        throw Exception('Failed to load user: ${response.statusCode}');
      }
    } catch (e) {
      debugPrint('Error fetching user: $e');
      return null;
    }
  }
  ```

**Effective Dart Style:**
- `lowerCamelCase` for variables, functions, parameters
- `UpperCamelCase` for types, classes, enums
- `lowercase_with_underscores` for library names, file names
- Example:
  ```dart
  // Parent: REQ-0005
  // File: user_repository.dart
  
  class UserRepository {
    final ApiClient apiClient;
    
    UserRepository({required this.apiClient});
    
    Future<List<User>> getAllUsers() async {
      // Implementation
    }
  }
  ```

**Traceability:** Document style deviations (if any) in Build Manifest notes with REQ justification.

---

### 3. Dependency Management

**pubspec.yaml Structure:**
- Separate dependencies from dev_dependencies
- Use version constraints for stability
- Example:
  ```yaml
  # Parent: REQ-0006
  name: myapp
  description: A Flutter application
  version: 1.0.0+1
  
  environment:
    sdk: '>=3.0.0 <4.0.0'
  
  dependencies:
    flutter:
      sdk: flutter
    flutter_bloc: ^8.1.3
    dio: ^5.3.2
    freezed_annotation: ^2.4.1
    json_annotation: ^4.8.1
    go_router: ^12.0.0
    shared_preferences: ^2.2.0
  
  dev_dependencies:
    flutter_test:
      sdk: flutter
    flutter_lints: ^3.0.0
    build_runner: ^2.4.6
    freezed: ^2.4.2
    json_serializable: ^6.7.1
    mockito: ^5.4.2
  ```

**Version Constraints:**
- Use caret (`^`) for compatible updates: `^1.2.3` allows `>=1.2.3 <2.0.0`
- Use exact versions for critical packages: `1.2.3`
- Document version pinning rationale in Build Manifest notes

**pub.dev Best Practices:**
- Prefer packages with high pub points (>100)
- Check package popularity and maintenance status
- Document package selection rationale (e.g., "dio selected per REQ-0006 for HTTP client with interceptors")

**Lock Files:**
- Commit `pubspec.lock` for apps (reproducible builds)
- Do not commit `pubspec.lock` for packages (allow version flexibility)

**Traceability:** Link dependency choices to REQ-XXXX in Build Manifest notes.

---

### 4. Flutter Widget Patterns

**Stateless vs Stateful Widgets:**
- Use StatelessWidget when widget doesn't manage state
- Use StatefulWidget when widget manages local UI state
- Example:
  ```dart
  // Parent: REQ-0007
  // Stateless widget (no state)
  class UserAvatar extends StatelessWidget {
    final String imageUrl;
    final double size;

    const UserAvatar({
      super.key,
      required this.imageUrl,
      this.size = 40.0,
    });

    @override
    Widget build(BuildContext context) {
      return CircleAvatar(
        radius: size / 2,
        backgroundImage: NetworkImage(imageUrl),
      );
    }
  }

  // Stateful widget (manages counter state)
  class Counter extends StatefulWidget {
    const Counter({super.key});

    @override
    State<Counter> createState() => _CounterState();
  }

  class _CounterState extends State<Counter> {
    int _count = 0;

    void _increment() {
      setState(() {
        _count++;
      });
    }

    @override
    Widget build(BuildContext context) {
      return Column(
        children: [
          Text('Count: $_count'),
          ElevatedButton(
            onPressed: _increment,
            child: const Text('Increment'),
          ),
        ],
      );
    }
  }
  ```

**Widget Composition:**
- Prefer composition over deep widget trees
- Extract widgets for reusability and testability
- Example:
  ```dart
  // Parent: REQ-0008
  // Bad: Deep widget tree
  class UserProfile extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return Card(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              CircleAvatar(/* ... */),
              Text(/* ... */),
              Text(/* ... */),
              // Many more widgets...
            ],
          ),
        ),
      );
    }
  }

  // Good: Extracted widgets
  class UserProfile extends StatelessWidget {
    final User user;

    const UserProfile({super.key, required this.user});

    @override
    Widget build(BuildContext context) {
      return Card(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              UserAvatar(imageUrl: user.avatarUrl),
              UserName(name: user.name),
              UserEmail(email: user.email),
            ],
          ),
        ),
      );
    }
  }
  ```

**Keys for Widget Identity:**
- Use keys when widget order changes (lists, animations)
- Example:
  ```dart
  // Parent: REQ-0009
  ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return ListTile(
        key: ValueKey(items[index].id), // Preserve state on reorder
        title: Text(items[index].name),
      );
    },
  );
  ```

**Traceability:** Each widget → REQ-XXXX. Document widget composition decisions in Build Manifest notes.

---

### 5. State Management

**BLoC (Business Logic Component):**
- Use for complex state management with clear separation of concerns
- Example:
  ```dart
  // Parent: REQ-0010
  // AC1: User can login with email and password
  
  // Events
  abstract class AuthEvent {}
  
  class LoginRequested extends AuthEvent {
    final String email;
    final String password;
    
    LoginRequested({required this.email, required this.password});
  }
  
  class LogoutRequested extends AuthEvent {}
  
  // States
  abstract class AuthState {}
  
  class AuthInitial extends AuthState {}
  
  class AuthLoading extends AuthState {}
  
  class AuthAuthenticated extends AuthState {
    final User user;
    
    AuthAuthenticated({required this.user});
  }
  
  class AuthUnauthenticated extends AuthState {}
  
  class AuthError extends AuthState {
    final String message;
    
    AuthError({required this.message});
  }
  
  // BLoC
  class AuthBloc extends Bloc<AuthEvent, AuthState> {
    final AuthRepository authRepository;
    
    AuthBloc({required this.authRepository}) : super(AuthInitial()) {
      on<LoginRequested>(_onLoginRequested);
      on<LogoutRequested>(_onLogoutRequested);
    }
    
    Future<void> _onLoginRequested(
      LoginRequested event,
      Emitter<AuthState> emit,
    ) async {
      emit(AuthLoading());
      try {
        final user = await authRepository.login(
          email: event.email,
          password: event.password,
        );
        emit(AuthAuthenticated(user: user));
      } catch (e) {
        emit(AuthError(message: e.toString()));
      }
    }
    
    Future<void> _onLogoutRequested(
      LogoutRequested event,
      Emitter<AuthState> emit,
    ) async {
      await authRepository.logout();
      emit(AuthUnauthenticated());
    }
  }
  ```

**Provider:**
- Use for simple state management and dependency injection
- Example:
  ```dart
  // Parent: REQ-0011
  import 'package:provider/provider.dart';
  
  class UserProvider extends ChangeNotifier {
    User? _user;
    
    User? get user => _user;
    
    void setUser(User user) {
      _user = user;
      notifyListeners();
    }
    
    void clearUser() {
      _user = null;
      notifyListeners();
    }
  }
  
  // Usage in widget
  class UserProfile extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      final user = context.watch<UserProvider>().user;
      
      if (user == null) {
        return const Text('Not logged in');
      }
      
      return Text('Hello, ${user.name}');
    }
  }
  ```

**Riverpod:**
- Use for modern, compile-safe state management
- Example:
  ```dart
  // Parent: REQ-0012
  import 'package:flutter_riverpod/flutter_riverpod.dart';
  
  // Provider
  final userProvider = StateNotifierProvider<UserNotifier, User?>((ref) {
    return UserNotifier();
  });
  
  // Notifier
  class UserNotifier extends StateNotifier<User?> {
    UserNotifier() : super(null);
    
    void setUser(User user) {
      state = user;
    }
    
    void clearUser() {
      state = null;
    }
  }
  
  // Usage in widget
  class UserProfile extends ConsumerWidget {
    @override
    Widget build(BuildContext context, WidgetRef ref) {
      final user = ref.watch(userProvider);
      
      if (user == null) {
        return const Text('Not logged in');
      }
      
      return Text('Hello, ${user.name}');
    }
  }
  ```

**GetX:**
- Use for rapid development with minimal boilerplate
- Document choice in Build Manifest notes (GetX is opinionated)
- Example:
  ```dart
  // Parent: REQ-0013
  import 'package:get/get.dart';
  
  class UserController extends GetxController {
    final Rx<User?> user = Rx<User?>(null);
    
    void setUser(User newUser) {
      user.value = newUser;
    }
    
    void clearUser() {
      user.value = null;
    }
  }
  
  // Usage in widget
  class UserProfile extends StatelessWidget {
    final UserController controller = Get.find();
    
    @override
    Widget build(BuildContext context) {
      return Obx(() {
        final user = controller.user.value;
        if (user == null) {
          return const Text('Not logged in');
        }
        return Text('Hello, ${user.name}');
      });
    }
  }
  ```

**Traceability:** Document state management choice in Build Manifest notes with REQ justification.

---

### 6. Navigation

**Navigator 2.0 (Declarative Routing):**
- Use for complex navigation with deep linking
- Example with go_router:
  ```dart
  // Parent: REQ-0014
  import 'package:go_router/go_router.dart';
  
  final router = GoRouter(
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const HomePage(),
      ),
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: '/users/:id',
        builder: (context, state) {
          final userId = state.pathParameters['id']!;
          return UserDetailPage(userId: userId);
        },
      ),
    ],
    redirect: (context, state) {
      final isAuthenticated = /* check auth state */;
      final isLoginRoute = state.matchedLocation == '/login';
      
      if (!isAuthenticated && !isLoginRoute) {
        return '/login';
      }
      
      if (isAuthenticated && isLoginRoute) {
        return '/';
      }
      
      return null; // No redirect
    },
  );
  
  // Usage
  MaterialApp.router(
    routerConfig: router,
  );
  ```

**Navigator 1.0 (Imperative Routing):**
- Use for simple navigation without deep linking
- Example:
  ```dart
  // Parent: REQ-0015
  // Push route
  Navigator.push(
    context,
    MaterialPageRoute(
      builder: (context) => UserDetailPage(userId: userId),
    ),
  );
  
  // Pop route
  Navigator.pop(context);
  
  // Push replacement
  Navigator.pushReplacement(
    context,
    MaterialPageRoute(
      builder: (context) => HomePage(),
    ),
  );
  ```

**Traceability:** Document navigation strategy in Build Manifest notes with REQ justification.

---

### 7. Platform Channels

**MethodChannel (Request/Response):**
- Use for calling platform-specific code (iOS/Android)
- Example:
  ```dart
  // Parent: REQ-0016
  // AC1: Get battery level from platform
  
  // Dart side
  import 'package:flutter/services.dart';
  
  class BatteryService {
    static const platform = MethodChannel('com.example.app/battery');
    
    Future<int?> getBatteryLevel() async {
      try {
        final int result = await platform.invokeMethod('getBatteryLevel');
        return result;
      } on PlatformException catch (e) {
        debugPrint('Failed to get battery level: ${e.message}');
        return null;
      }
    }
  }
  
  // iOS side (Swift)
  // In AppDelegate.swift:
  /*
  let batteryChannel = FlutterMethodChannel(
    name: "com.example.app/battery",
    binaryMessenger: controller.binaryMessenger
  )
  
  batteryChannel.setMethodCallHandler { (call, result) in
    if call.method == "getBatteryLevel" {
      let batteryLevel = self.getBatteryLevel()
      result(batteryLevel)
    } else {
      result(FlutterMethodNotImplemented)
    }
  }
  */
  ```

**EventChannel (Streaming):**
- Use for streaming data from platform
- Example:
  ```dart
  // Parent: REQ-0017
  import 'package:flutter/services.dart';
  
  class SensorService {
    static const eventChannel = EventChannel('com.example.app/sensor');
    
    Stream<double> get accelerometerStream {
      return eventChannel.receiveBroadcastStream().map((event) {
        return event as double;
      });
    }
  }
  ```

**Security Considerations:**
- Validate all data from platform channels
- Document platform channel security in Build Manifest notes
- Never pass sensitive data without encryption

**Halt Condition:** Halt if platform channel handles sensitive data without documented security review.

---

### 8. Architecture Patterns

**Clean Architecture:**
- Separate presentation, domain, and data layers
- Example structure (see Project Structure section)
- Benefits: Testability, maintainability, independence from frameworks

**Layered Architecture:**
- UI → Business Logic → Data Access
- Example:
  ```dart
  // Parent: REQ-0018
  // Presentation Layer
  class LoginPage extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return BlocProvider(
        create: (context) => AuthBloc(
          authRepository: context.read<AuthRepository>(),
        ),
        child: LoginForm(),
      );
    }
  }
  
  // Domain Layer (Use Cases)
  class LoginUseCase {
    final AuthRepository repository;
    
    LoginUseCase({required this.repository});
    
    Future<User> call({
      required String email,
      required String password,
    }) async {
      return repository.login(email: email, password: password);
    }
  }
  
  // Data Layer (Repository Implementation)
  class AuthRepositoryImpl implements AuthRepository {
    final AuthRemoteDataSource remoteDataSource;
    
    AuthRepositoryImpl({required this.remoteDataSource});
    
    @override
    Future<User> login({
      required String email,
      required String password,
    }) async {
      final userModel = await remoteDataSource.login(
        email: email,
        password: password,
      );
      return userModel.toEntity();
    }
  }
  ```

**Feature-First Architecture:**
- Organize by feature, not technical layer
- Each feature contains its own presentation, domain, and data layers
- Example: See Project Structure section

**Traceability:** Document architecture choice in Build Manifest notes with REQ justification.

---

### 9. Security Patterns

**Secure Storage:**
- Use flutter_secure_storage for sensitive data (tokens, credentials)
- Example:
  ```dart
  // Parent: REQ-0019
  import 'package:flutter_secure_storage/flutter_secure_storage.dart';
  
  class SecureStorageService {
    final storage = const FlutterSecureStorage();
    
    Future<void> saveToken(String token) async {
      await storage.write(key: 'auth_token', value: token);
    }
    
    Future<String?> getToken() async {
      return await storage.read(key: 'auth_token');
    }
    
    Future<void> deleteToken() async {
      await storage.delete(key: 'auth_token');
    }
  }
  ```

**Encryption:**
- Use encrypt package for data encryption
- Example:
  ```dart
  // Parent: REQ-0020
  import 'package:encrypt/encrypt.dart';
  
  class EncryptionService {
    final key = Key.fromSecureRandom(32);
    final iv = IV.fromSecureRandom(16);
    
    String encrypt(String plainText) {
      final encrypter = Encrypter(AES(key));
      final encrypted = encrypter.encrypt(plainText, iv: iv);
      return encrypted.base64;
    }
    
    String decrypt(String encryptedText) {
      final encrypter = Encrypter(AES(key));
      final decrypted = encrypter.decrypt64(encryptedText, iv: iv);
      return decrypted;
    }
  }
  ```

**Platform Security:**
- iOS: Keychain for secure storage
- Android: EncryptedSharedPreferences, Keystore
- Document platform-specific security in Build Manifest notes

**Input Validation:**
- Validate all user inputs
- Example:
  ```dart
  // Parent: REQ-0021
  class Validators {
    static String? email(String? value) {
      if (value == null || value.isEmpty) {
        return 'Email is required';
      }
      final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
      if (!emailRegex.hasMatch(value)) {
        return 'Invalid email format';
      }
      return null;
    }
    
    static String? password(String? value) {
      if (value == null || value.isEmpty) {
        return 'Password is required';
      }
      if (value.length < 8) {
        return 'Password must be at least 8 characters';
      }
      return null;
    }
  }
  ```

**Escalation Rule:**
- Any auth, permission, token, session, or identity change = R2+ risk level (see Evidence Requirements)

**Secure Coding (inherited from build-agent + Dart-specific):**
1. Input validation (validators, form validation)
2. Error handling (explicit try/catch, custom exceptions)
3. No hardcoded secrets (use environment variables, secure storage)
4. Secure storage (flutter_secure_storage, platform keychain)
5. Bounded operations (pagination on lists, query timeouts)
6. Least privilege (permission requests, platform security)
7. Dependency awareness (pub.dev security advisories)

**Halt Condition:** Halt if hardcoded secrets detected in code.

---

### 10. Testing Strategy

**Unit Tests:**
- Test business logic, models, repositories
- Example:
  ```dart
  // Parent: REQ-0022
  import 'package:flutter_test/flutter_test.dart';
  import 'package:mockito/mockito.dart';
  
  void main() {
    group('AuthBloc', () {
      late AuthBloc authBloc;
      late MockAuthRepository mockRepository;
      
      setUp(() {
        mockRepository = MockAuthRepository();
        authBloc = AuthBloc(authRepository: mockRepository);
      });
      
      tearDown(() {
        authBloc.close();
      });
      
      test('initial state is AuthInitial', () {
        expect(authBloc.state, isA<AuthInitial>());
      });
      
      test('emits [AuthLoading, AuthAuthenticated] on successful login', () async {
        // Arrange
        final user = User(id: '1', email: 'test@example.com', name: 'Test');
        when(mockRepository.login(
          email: 'test@example.com',
          password: 'password',
        )).thenAnswer((_) async => user);
        
        // Assert
        expectLater(
          authBloc.stream,
          emitsInOrder([
            isA<AuthLoading>(),
            isA<AuthAuthenticated>(),
          ]),
        );
        
        // Act
        authBloc.add(LoginRequested(
          email: 'test@example.com',
          password: 'password',
        ));
      });
    });
  }
  ```

**Widget Tests:**
- Test widget behavior and UI
- Example:
  ```dart
  // Parent: REQ-0023
  import 'package:flutter_test/flutter_test.dart';
  import 'package:flutter/material.dart';
  
  void main() {
    testWidgets('LoginForm displays email and password fields', (tester) async {
      // Arrange
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: LoginForm(),
          ),
        ),
      );
      
      // Assert
      expect(find.byType(TextFormField), findsNWidgets(2));
      expect(find.text('Email'), findsOneWidget);
      expect(find.text('Password'), findsOneWidget);
      expect(find.byType(ElevatedButton), findsOneWidget);
    });
    
    testWidgets('LoginForm validates email format', (tester) async {
      // Arrange
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: LoginForm(),
          ),
        ),
      );
      
      // Act
      await tester.enterText(
        find.byType(TextFormField).first,
        'invalid-email',
      );
      await tester.tap(find.byType(ElevatedButton));
      await tester.pump();
      
      // Assert
      expect(find.text('Invalid email format'), findsOneWidget);
    });
  }
  ```

**Integration Tests:**
- Test complete user flows
- Example:
  ```dart
  // Parent: REQ-0024
  import 'package:integration_test/integration_test.dart';
  import 'package:flutter_test/flutter_test.dart';
  
  void main() {
    IntegrationTestWidgetsFlutterBinding.ensureInitialized();
    
    testWidgets('user can login and view home page', (tester) async {
      // Arrange
      await tester.pumpWidget(MyApp());
      
      // Act - Navigate to login
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();
      
      // Act - Enter credentials
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );
      
      // Act - Submit form
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();
      
      // Assert - Verify navigation to home page
      expect(find.text('Home'), findsOneWidget);
      expect(find.text('Welcome, Test User'), findsOneWidget);
    });
  }
  ```

**Golden Tests:**
- Test widget visual appearance
- Example:
  ```dart
  // Parent: REQ-0025
  import 'package:flutter_test/flutter_test.dart';
  import 'package:golden_toolkit/golden_toolkit.dart';
  
  void main() {
    testGoldens('UserProfile golden test', (tester) async {
      final user = User(
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
        avatarUrl: 'https://example.com/avatar.jpg',
      );
      
      await tester.pumpWidgetBuilder(
        UserProfile(user: user),
        surfaceSize: const Size(400, 600),
      );
      
      await screenMatchesGolden(tester, 'user_profile');
    });
  }
  ```

**Coverage Targets:**
- From REQ acceptance criteria
- Use `flutter test --coverage`
- Document coverage thresholds in Build Manifest notes

**Bug Fixes:**
- Regression test required (see test-designer + red-team-verifier)
- Test must fail before fix, pass after fix

**Alignment:** Test Designer (TC-XXXX) defines tests; Build Agent structures code for testability (dependency injection, widget keys, etc.).

---

### 11. Build and Deployment

**Build Flavors:**
- Use flavors for different environments (dev, staging, production)
- Example:
  ```dart
  // Parent: REQ-0026
  // lib/main_dev.dart
  void main() {
    runApp(MyApp(environment: Environment.dev));
  }
  
  // lib/main_prod.dart
  void main() {
    runApp(MyApp(environment: Environment.production));
  }
  
  // Build commands
  // flutter build apk --flavor dev -t lib/main_dev.dart
  // flutter build apk --flavor prod -t lib/main_prod.dart
  ```

**Code Generation:**
- Use build_runner for code generation (freezed, json_serializable)
- Example:
  ```dart
  // Parent: REQ-0027
  import 'package:freezed_annotation/freezed_annotation.dart';
  
  part 'user.freezed.dart';
  part 'user.g.dart';
  
  @freezed
  class User with _$User {
    const factory User({
      required String id,
      required String email,
      required String name,
    }) = _User;
    
    factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  }
  
  // Generate code: flutter pub run build_runner build
  ```

**Platform-Specific Configuration:**
- iOS: Info.plist, Podfile
- Android: AndroidManifest.xml, build.gradle
- Document platform permissions in Build Manifest notes
- Example permissions:
  ```xml
  <!-- Parent: REQ-0028 -->
  <!-- Android: AndroidManifest.xml -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  ```

**Halt Condition:** Halt if platform permissions added without documentation in Build Manifest notes.

---

## Evidence Requirements

Inherits R0-R3 framework from **agile-v-compliance**. Dart/Flutter-specific additions below.

### R0: Exploratory
Base evidence applies (short result summary, no production credentials, no production code path changed).

**Dart/Flutter-Specific:** No additions.

---

### R1: Routine
Base evidence applies (affected files, diff summary, targeted tests or explanation, lint/typecheck, residual-risk note).

**Dart/Flutter-Specific Additions:**
- **Static analysis:** `dart analyze` output (no errors)
- **Unit tests:** `flutter test` output for affected modules
- **Widget tests:** Widget test results for UI changes

---

### R2: Production
Base evidence applies (task brief with REQ IDs, implementation plan, affected files, executed commands, test results, regression coverage, acceptance criteria → test mapping, security/static check, rollback path, reviewer decision).

**Dart/Flutter-Specific Additions:**
- **Integration tests:** Integration test results (`flutter test integration_test/`)
- **Platform testing:** iOS and Android test results for platform-specific changes
- **Golden tests:** Golden test baselines updated (if UI changes)
- **Performance:** Flutter DevTools performance profiling results (if performance-sensitive)
- **Dependencies:** pub.dev security advisories checked (no high/critical vulnerabilities)
- **Build verification:** `flutter build apk` and `flutter build ios` successful

---

### R3: High Assurance
Base evidence applies (all R2 evidence + independent verification agent review, traceability matrix, explicit human sign-off, audit artifact, release decision rationale).

**Dart/Flutter-Specific Additions:**
- **Performance profiling:** Flutter DevTools timeline, memory profiling, CPU profiling
- **Accessibility:** Accessibility audit (Semantics widget coverage, screen reader testing)
- **Platform compatibility:** iOS and Android version compatibility matrix
- **Security:** Platform channel security audit, secure storage verification
- **App size:** APK/IPA size analysis (document size increases >10%)
- **Traceability:** REQ-XXXX → ART-XXXX → TC-XXXX → Evidence mapping in ATM.md

---

## Halt Conditions

Halt and do not emit when:

**Inherited from build-agent:**
- Ambiguous REQ (requirement unclear or contradictory)
- Missing REQ link (artifact has no traceable parent requirement)
- Physical constraint violation (hardware, network, or infrastructure limits exceeded)
- Conflict with approved Blueprint (contradicts Human Gate 1 approved design)

**Dart/Flutter-Specific:**
- **dart analyze errors in production build** (`dart analyze` fails for R2+ tasks without documented exceptions)
- **Missing null safety migration** (code uses legacy null safety or unsound null safety)
- **Platform channel security issues** (platform channel handles sensitive data without documented security review)
- **Hardcoded secrets in code** (API keys, tokens, passwords in source files)
- **Auth change without R2+ risk classification** (authentication, authorization, or session logic changed but classified as R1)
- **Missing platform permissions documentation** (platform permissions added without documentation in Build Manifest notes)
- **Widget tests missing for critical UI** (critical user flows lack widget tests or integration tests)

**Halt Protocol:**
1. Stop synthesis immediately
2. Emit Evidence Summary with HALT condition flagged
3. Present specific issue to Human (e.g., "Hardcoded API key detected in lib/services/api_service.dart")
4. Wait for Human resolution (refactor, clarify REQ, approve exception)
5. Resume only after Human Gate cleared

---

## Context Engineering

Inherited from build-agent + these Dart/Flutter considerations:

1. **Generated code:** build_runner output (*.freezed.dart, *.g.dart) → reference by path, do not load contents into context.
2. **Platform-specific code:** iOS/Android native code should be synthesized in separate context from Dart layer to avoid cross-language context pollution.
3. **Widget trees:** Build one screen/feature per sub-agent context, not the entire app.
4. **Assets:** Images, fonts, JSON files → reference by path in pubspec.yaml, do not load contents into context.
5. **Packages:** .pub-cache, .dart_tool → never load into context. Reference package names/versions from pubspec.yaml only.
6. **Build outputs:** build/, .dart_tool/build/ → never load into context. Reference by path only.

**Pre-Execution Validation (inherited from build-agent):**
Before synthesis, validate:
1. **Requirement coverage:** Every REQ has ≥1 artifact planned
2. **Artifact completeness:** Widgets, BLoCs/Providers, repositories, models, tests, platform channels (if applicable)
3. **Dependency order:** No circular imports between modules (analyze imports)
4. **Scope sanity:** Feature scope fits ≤50% context (split to sub-agents if needed)
5. **Interface contracts:** Document module exports before synthesis (e.g., AuthRepository exports login, logout)

**Halt if any validation fails.**

---

## Output Format

Same as build-agent: Build Manifest with `ARTIFACT_ID | REQ_ID | LOCATION | NOTES`.

**Example Dart/Flutter Build Manifest:**
```
BUILD_MANIFEST.md

Cycle: C1
Task: REQ-0001 - User authentication via JWT
Risk Level: R2
Generated: 2026-05-22T10:00:00Z

ART-0001 | REQ-0001 | lib/features/auth/presentation/pages/login_page.dart | Login page; BLoC pattern
ART-0002 | REQ-0001 | lib/features/auth/presentation/widgets/login_form.dart | Login form widget; form validation
ART-0003 | REQ-0001 | lib/features/auth/presentation/bloc/auth_bloc.dart | Auth BLoC; handles login/logout events
ART-0004 | REQ-0001 | lib/features/auth/presentation/bloc/auth_event.dart | Auth events (LoginRequested, LogoutRequested)
ART-0005 | REQ-0001 | lib/features/auth/presentation/bloc/auth_state.dart | Auth states (Loading, Authenticated, Error)
ART-0006 | REQ-0001 | lib/features/auth/domain/repositories/auth_repository.dart | Auth repository interface
ART-0007 | REQ-0001 | lib/features/auth/domain/entities/user.dart | User entity
ART-0008 | REQ-0001 | lib/features/auth/data/repositories/auth_repository_impl.dart | Auth repository implementation
ART-0009 | REQ-0001 | lib/features/auth/data/models/user_model.dart | User model with JSON serialization
ART-0010 | REQ-0001 | lib/features/auth/data/datasources/auth_remote_datasource.dart | Auth API client
ART-0011 | REQ-0001 | test/features/auth/presentation/bloc/auth_bloc_test.dart | Unit tests for AuthBloc (5 scenarios)
ART-0012 | REQ-0001 | test/features/auth/domain/usecases/login_usecase_test.dart | Unit tests for LoginUseCase (3 scenarios)
ART-0013 | REQ-0001 | integration_test/auth_flow_test.dart | Integration test for login flow (2 scenarios)
```

**Per-file traceability header:**
```dart
// Parent: REQ-0001
// AC1: POST /auth/login returns access token on valid credentials
// AC2: Invalid credentials return 401
```

---

## When to Use

**Project Types:**
- Flutter mobile apps (iOS, Android)
- Flutter web applications
- Flutter desktop applications (Windows, macOS, Linux)
- Dart packages and plugins
- Dart backend services (shelf, dart_frog)

**Auto-Trigger Hints (for agent routing):**

**pubspec.yaml dependencies:**
- `flutter`
- `flutter_bloc`
- `provider`
- `riverpod`
- `get`
- `dio`
- `http`
- `go_router`
- `shelf`
- `dart_frog`

**File patterns:**
- `**/*.dart`
- `**/pubspec.yaml`
- `**/analysis_options.yaml`
- `**/lib/**/*.dart`
- `**/test/**/*.dart`
- `**/integration_test/**/*.dart`

**Task keywords:**
- "Flutter"
- "Dart"
- "widget"
- "BLoC"
- "Provider"
- "Riverpod"
- "mobile app"
- "iOS"
- "Android"
- "platform channel"
- "state management"
