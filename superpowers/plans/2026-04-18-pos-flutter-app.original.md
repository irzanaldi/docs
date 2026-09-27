# POS System Phase 1 — Flutter App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the Flutter mobile app for kasir (cashier) and kitchen display with real-time order flow.

**Architecture:** Single Flutter project with two main flows: Kasir and Kitchen Display. Uses BLoC for state management, Dio for REST API calls, and socket_io_client for real-time WebSocket communication with the NestJS backend.

**Tech Stack:** Flutter, Dart, BLoC, Dio, socket_io_client, shared_preferences (token storage)

**Spec:** `docs/superpowers/specs/2026-04-18-pos-system-design.md`
**Backend API Plan:** `docs/superpowers/plans/2026-04-18-pos-backend-api.md`

---

## File Structure

```
mobile/
├── pubspec.yaml
├── analysis_options.yaml
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── config/
│   │   ├── app_config.dart
│   │   └── app_theme.dart
│   ├── core/
│   │   ├── api/
│   │   │   ├── api_client.dart
│   │   │   ├── api_interceptor.dart
│   │   │   └── api_response.dart
│   │   ├── socket/
│   │   │   └── socket_service.dart
│   │   ├── storage/
│   │   │   └── token_storage.dart
│   │   └── error/
│   │       └── app_exception.dart
│   ├── models/
│   │   ├── user.dart
│   │   ├── menu_item.dart
│   │   ├── retail_item.dart
│   │   ├── cart_item.dart
│   │   ├── transaction.dart
│   │   ├── order.dart
│   │   ├── order_item.dart
│   │   └── promo.dart
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── bloc/
│   │   │   │   ├── auth_bloc.dart
│   │   │   │   ├── auth_event.dart
│   │   │   │   └── auth_state.dart
│   │   │   ├── repository/
│   │   │   │   └── auth_repository.dart
│   │   │   └── screens/
│   │   │       └── login_screen.dart
│   │   ├── kasir/
│   │   │   ├── bloc/
│   │   │   │   ├── menu_bloc.dart
│   │   │   │   ├── menu_event.dart
│   │   │   │   ├── menu_state.dart
│   │   │   │   ├── cart_bloc.dart
│   │   │   │   ├── cart_event.dart
│   │   │   │   ├── cart_state.dart
│   │   │   │   ├── promo_bloc.dart
│   │   │   │   ├── promo_event.dart
│   │   │   │   ├── promo_state.dart
│   │   │   │   ├── checkout_bloc.dart
│   │   │   │   ├── checkout_event.dart
│   │   │   │   ├── checkout_state.dart
│   │   │   │   ├── order_tracking_bloc.dart
│   │   │   │   ├── order_tracking_event.dart
│   │   │   │   └── order_tracking_state.dart
│   │   │   ├── repository/
│   │   │   │   ├── menu_repository.dart
│   │   │   │   ├── promo_repository.dart
│   │   │   │   └── transaction_repository.dart
│   │   │   ├── screens/
│   │   │   │   ├── kasir_main_screen.dart
│   │   │   │   ├── checkout_screen.dart
│   │   │   │   ├── receipt_screen.dart
│   │   │   │   └── order_tracking_screen.dart
│   │   │   └── widgets/
│   │   │       ├── menu_grid.dart
│   │   │       ├── menu_item_card.dart
│   │   │       ├── category_filter.dart
│   │   │       ├── search_bar_widget.dart
│   │   │       ├── cart_panel.dart
│   │   │       ├── cart_item_tile.dart
│   │   │       ├── promo_badge.dart
│   │   │       └── payment_method_selector.dart
│   │   └── kitchen/
│   │       ├── bloc/
│   │       │   ├── kitchen_bloc.dart
│   │       │   ├── kitchen_event.dart
│   │       │   └── kitchen_state.dart
│   │       ├── repository/
│   │       │   └── kitchen_repository.dart
│   │       ├── screens/
│   │       │   └── kitchen_display_screen.dart
│   │       └── widgets/
│   │           ├── order_card.dart
│   │           ├── order_item_tile.dart
│   │           └── order_timer.dart
│   └── shared/
│       └── widgets/
│           ├── app_button.dart
│           ├── app_text_field.dart
│           ├── loading_overlay.dart
│           └── status_badge.dart
├── test/
│   ├── modules/
│   │   ├── auth/
│   │   │   └── bloc/
│   │   │       └── auth_bloc_test.dart
│   │   ├── kasir/
│   │   │   └── bloc/
│   │   │       ├── menu_bloc_test.dart
│   │   │       ├── cart_bloc_test.dart
│   │   │       ├── promo_bloc_test.dart
│   │   │       └── checkout_bloc_test.dart
│   │   └── kitchen/
│   │       └── bloc/
│   │           └── kitchen_bloc_test.dart
│   └── widget_test.dart
```

---

### Task 1: Project Setup & Dependencies

**Files:**
- Create: `mobile/pubspec.yaml`
- Create: `mobile/analysis_options.yaml`
- Create: `mobile/lib/main.dart`
- Create: `mobile/lib/app.dart`
- Create: `mobile/lib/config/app_config.dart`

- [ ] **Step 1: Create Flutter project**

```bash
cd ~/Downloads/PROJECT
flutter create --org com.possystem --project-name pos_mobile mobile
```

- [ ] **Step 2: Replace pubspec.yaml with dependencies**

Replace `mobile/pubspec.yaml`:

```yaml
name: pos_mobile
description: POS System - Kasir & Kitchen Display
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.6
  bloc: ^8.1.4
  dio: ^5.4.3+1
  socket_io_client: ^2.0.3+1
  shared_preferences: ^2.2.3
  equatable: ^2.0.5
  intl: ^0.19.0
  google_fonts: ^6.2.1
  cached_network_image: ^3.3.1
  flutter_staggered_grid_view: ^0.7.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
  bloc_test: ^9.1.7
  mocktail: ^1.0.4

flutter:
  uses-material-design: true
```

- [ ] **Step 3: Install dependencies**

```bash
cd ~/Downloads/PROJECT/mobile
flutter pub get
```

- [ ] **Step 4: Create app config**

```dart
// mobile/lib/config/app_config.dart
class AppConfig {
  static const String baseUrl = 'http://10.0.2.2:3000';
  static const String socketUrl = 'http://10.0.2.2:3000';
  static const String socketNamespace = '/orders';

  static const Duration apiTimeout = Duration(seconds: 30);
  static const Duration socketReconnectDelay = Duration(seconds: 3);
}
```

- [ ] **Step 5: Create main.dart entry point**

```dart
// mobile/lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'app.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight,
  ]);
  runApp(const PosApp());
}
```

- [ ] **Step 6: Create app.dart shell**

```dart
// mobile/lib/app.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'config/app_theme.dart';
import 'core/api/api_client.dart';
import 'core/storage/token_storage.dart';
import 'core/socket/socket_service.dart';
import 'modules/auth/bloc/auth_bloc.dart';
import 'modules/auth/bloc/auth_event.dart';
import 'modules/auth/bloc/auth_state.dart';
import 'modules/auth/repository/auth_repository.dart';
import 'modules/auth/screens/login_screen.dart';
import 'modules/kasir/screens/kasir_main_screen.dart';
import 'modules/kitchen/screens/kitchen_display_screen.dart';

class PosApp extends StatefulWidget {
  const PosApp({super.key});

  @override
  State<PosApp> createState() => _PosAppState();
}

class _PosAppState extends State<PosApp> {
  late final ApiClient _apiClient;
  late final TokenStorage _tokenStorage;
  late final SocketService _socketService;
  late final AuthRepository _authRepository;

  @override
  void initState() {
    super.initState();
    _tokenStorage = TokenStorage();
    _apiClient = ApiClient(tokenStorage: _tokenStorage);
    _socketService = SocketService(tokenStorage: _tokenStorage);
    _authRepository = AuthRepository(apiClient: _apiClient);
  }

  @override
  void dispose() {
    _socketService.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MultiRepositoryProvider(
      providers: [
        RepositoryProvider.value(value: _apiClient),
        RepositoryProvider.value(value: _tokenStorage),
        RepositoryProvider.value(value: _socketService),
        RepositoryProvider.value(value: _authRepository),
      ],
      child: BlocProvider(
        create: (_) => AuthBloc(
          authRepository: _authRepository,
          tokenStorage: _tokenStorage,
        )..add(AuthCheckRequested()),
        child: MaterialApp(
          title: 'POS System',
          debugShowCheckedModeBanner: false,
          theme: AppTheme.lightTheme,
          home: BlocBuilder<AuthBloc, AuthState>(
            builder: (context, state) {
              if (state is AuthAuthenticated) {
                if (state.user.roleName == 'KITCHEN') {
                  return const KitchenDisplayScreen();
                }
                return const KasirMainScreen();
              }
              return const LoginScreen();
            },
          ),
        ),
      ),
    );
  }
}
```

- [ ] **Step 7: Verify project compiles**

```bash
cd ~/Downloads/PROJECT/mobile
flutter analyze --no-fatal-infos --no-fatal-warnings
```

Expected: Project compiles (some missing references expected until later tasks)

---

### Task 2: Core Layer — API Client, WebSocket, Token Storage

**Files:**
- Create: `mobile/lib/core/api/api_response.dart`
- Create: `mobile/lib/core/api/api_interceptor.dart`
- Create: `mobile/lib/core/api/api_client.dart`
- Create: `mobile/lib/core/socket/socket_service.dart`
- Create: `mobile/lib/core/storage/token_storage.dart`
- Create: `mobile/lib/core/error/app_exception.dart`

- [ ] **Step 1: Create AppException**

```dart
// mobile/lib/core/error/app_exception.dart
class AppException implements Exception {
  final String message;
  final int? statusCode;
  final dynamic data;

  AppException({
    required this.message,
    this.statusCode,
    this.data,
  });

  @override
  String toString() => 'AppException($statusCode): $message';
}

class UnauthorizedException extends AppException {
  UnauthorizedException({String message = 'Unauthorized'})
      : super(message: message, statusCode: 401);
}

class NotFoundException extends AppException {
  NotFoundException({String message = 'Not found'})
      : super(message: message, statusCode: 404);
}

class ServerException extends AppException {
  ServerException({String message = 'Internal server error'})
      : super(message: message, statusCode: 500);
}

class NetworkException extends AppException {
  NetworkException({String message = 'No internet connection'})
      : super(message: message);
}
```

- [ ] **Step 2: Create TokenStorage**

```dart
// mobile/lib/core/storage/token_storage.dart
import 'package:shared_preferences/shared_preferences.dart';

class TokenStorage {
  static const String _tokenKey = 'access_token';
  static const String _userKey = 'user_data';

  Future<void> saveToken(String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_tokenKey, token);
  }

  Future<String?> getToken() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_tokenKey);
  }

  Future<void> saveUserData(String userData) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_userKey, userData);
  }

  Future<String?> getUserData() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_userKey);
  }

  Future<void> clearAll() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_tokenKey);
    await prefs.remove(_userKey);
  }

  Future<bool> hasToken() async {
    final token = await getToken();
    return token != null && token.isNotEmpty;
  }
}
```

- [ ] **Step 3: Create ApiResponse model**

```dart
// mobile/lib/core/api/api_response.dart
class ApiResponse<T> {
  final int statusCode;
  final T data;
  final String message;

  ApiResponse({
    required this.statusCode,
    required this.data,
    required this.message,
  });

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(dynamic) fromJsonT,
  ) {
    return ApiResponse(
      statusCode: json['statusCode'] as int,
      data: fromJsonT(json['data']),
      message: json['message'] as String? ?? 'Success',
    );
  }
}

class PaginatedResponse<T> {
  final List<T> data;
  final int total;
  final int page;
  final int limit;

  PaginatedResponse({
    required this.data,
    required this.total,
    required this.page,
    required this.limit,
  });

  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    return PaginatedResponse(
      data: (json['data'] as List).map((e) => fromJsonT(e as Map<String, dynamic>)).toList(),
      total: json['total'] as int,
      page: json['page'] as int,
      limit: json['limit'] as int,
    );
  }
}
```

- [ ] **Step 4: Create ApiInterceptor**

```dart
// mobile/lib/core/api/api_interceptor.dart
import 'package:dio/dio.dart';
import '../storage/token_storage.dart';

class AuthInterceptor extends Interceptor {
  final TokenStorage _tokenStorage;

  AuthInterceptor({required TokenStorage tokenStorage})
      : _tokenStorage = tokenStorage;

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await _tokenStorage.getToken();
    if (token != null && token.isNotEmpty) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (err.response?.statusCode == 401) {
      _tokenStorage.clearAll();
    }
    handler.next(err);
  }
}
```

- [ ] **Step 5: Create ApiClient**

```dart
// mobile/lib/core/api/api_client.dart
import 'package:dio/dio.dart';
import '../../config/app_config.dart';
import '../storage/token_storage.dart';
import '../error/app_exception.dart';
import 'api_interceptor.dart';

class ApiClient {
  late final Dio _dio;
  final TokenStorage tokenStorage;

  ApiClient({required this.tokenStorage}) {
    _dio = Dio(
      BaseOptions(
        baseUrl: AppConfig.baseUrl,
        connectTimeout: AppConfig.apiTimeout,
        receiveTimeout: AppConfig.apiTimeout,
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );
    _dio.interceptors.add(AuthInterceptor(tokenStorage: tokenStorage));
  }

  Future<Response> get(
    String path, {
    Map<String, dynamic>? queryParameters,
  }) async {
    try {
      return await _dio.get(path, queryParameters: queryParameters);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Future<Response> post(
    String path, {
    dynamic data,
  }) async {
    try {
      return await _dio.post(path, data: data);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Future<Response> patch(
    String path, {
    dynamic data,
  }) async {
    try {
      return await _dio.patch(path, data: data);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Future<Response> delete(String path) async {
    try {
      return await _dio.delete(path);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  AppException _handleError(DioException e) {
    if (e.type == DioExceptionType.connectionTimeout ||
        e.type == DioExceptionType.receiveTimeout ||
        e.type == DioExceptionType.sendTimeout) {
      return NetworkException(message: 'Connection timeout');
    }

    if (e.type == DioExceptionType.connectionError) {
      return NetworkException(message: 'No internet connection');
    }

    final statusCode = e.response?.statusCode;
    final responseData = e.response?.data;
    String message = 'An unexpected error occurred';

    if (responseData is Map<String, dynamic>) {
      final msg = responseData['message'];
      if (msg is String) {
        message = msg;
      } else if (msg is List) {
        message = msg.join(', ');
      }
    }

    switch (statusCode) {
      case 401:
        return UnauthorizedException(message: message);
      case 404:
        return NotFoundException(message: message);
      case 500:
        return ServerException(message: message);
      default:
        return AppException(message: message, statusCode: statusCode);
    }
  }
}
```

- [ ] **Step 6: Create SocketService**

```dart
// mobile/lib/core/socket/socket_service.dart
import 'dart:async';
import 'package:socket_io_client/socket_io_client.dart' as io;
import '../../config/app_config.dart';
import '../storage/token_storage.dart';

class SocketService {
  final TokenStorage _tokenStorage;
  io.Socket? _socket;
  bool _isConnected = false;

  final _connectionController = StreamController<bool>.broadcast();
  Stream<bool> get connectionStream => _connectionController.stream;
  bool get isConnected => _isConnected;

  final Map<String, StreamController<dynamic>> _eventControllers = {};

  SocketService({required TokenStorage tokenStorage})
      : _tokenStorage = tokenStorage;

  Future<void> connect() async {
    if (_isConnected) return;

    final token = await _tokenStorage.getToken();
    if (token == null) return;

    _socket = io.io(
      '${AppConfig.socketUrl}${AppConfig.socketNamespace}',
      io.OptionBuilder()
          .setTransports(['websocket'])
          .disableAutoConnect()
          .setExtraHeaders({'Authorization': 'Bearer $token'})
          .setQuery({'token': token})
          .build(),
    );

    _socket!.onConnect((_) {
      _isConnected = true;
      _connectionController.add(true);
    });

    _socket!.onDisconnect((_) {
      _isConnected = false;
      _connectionController.add(false);
    });

    _socket!.onConnectError((error) {
      _isConnected = false;
      _connectionController.add(false);
    });

    _socket!.connect();
  }

  Stream<T> on<T>(String event) {
    if (!_eventControllers.containsKey(event)) {
      _eventControllers[event] = StreamController<dynamic>.broadcast();
      _socket?.on(event, (data) {
        _eventControllers[event]?.add(data);
      });
    }
    return _eventControllers[event]!.stream.cast<T>();
  }

  void emit(String event, dynamic data) {
    _socket?.emit(event, data);
  }

  void disconnect() {
    _socket?.disconnect();
    _isConnected = false;
    _connectionController.add(false);
  }

  void dispose() {
    disconnect();
    _connectionController.close();
    for (final controller in _eventControllers.values) {
      controller.close();
    }
    _eventControllers.clear();
    _socket?.dispose();
  }
}
```

- [ ] **Step 7: Verify project compiles**

```bash
cd ~/Downloads/PROJECT/mobile
flutter analyze --no-fatal-infos --no-fatal-warnings
```

---

### Task 3: Data Models

**Files:**
- Create: `mobile/lib/models/user.dart`
- Create: `mobile/lib/models/menu_item.dart`
- Create: `mobile/lib/models/retail_item.dart`
- Create: `mobile/lib/models/cart_item.dart`
- Create: `mobile/lib/models/transaction.dart`
- Create: `mobile/lib/models/order.dart`
- Create: `mobile/lib/models/order_item.dart`
- Create: `mobile/lib/models/promo.dart`

- [ ] **Step 1: Create User model**

```dart
// mobile/lib/models/user.dart
import 'package:equatable/equatable.dart';

class User extends Equatable {
  final String id;
  final String name;
  final String email;
  final String roleName;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.roleName,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      roleName: json['role'] as String,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'role': roleName,
    };
  }

  @override
  List<Object?> get props => [id, name, email, roleName];
}

class AuthResponse {
  final String accessToken;
  final User user;

  AuthResponse({required this.accessToken, required this.user});

  factory AuthResponse.fromJson(Map<String, dynamic> json) {
    return AuthResponse(
      accessToken: json['accessToken'] as String,
      user: User.fromJson(json['user'] as Map<String, dynamic>),
    );
  }
}
```

- [ ] **Step 2: Create MenuItem model**

```dart
// mobile/lib/models/menu_item.dart
import 'package:equatable/equatable.dart';

class MenuItem extends Equatable {
  final String id;
  final String name;
  final String category;
  final double sellingPrice;
  final String? imageUrl;
  final bool isActive;
  final double hpp;
  final double profit;
  final double margin;

  const MenuItem({
    required this.id,
    required this.name,
    required this.category,
    required this.sellingPrice,
    this.imageUrl,
    this.isActive = true,
    this.hpp = 0,
    this.profit = 0,
    this.margin = 0,
  });

  factory MenuItem.fromJson(Map<String, dynamic> json) {
    return MenuItem(
      id: json['id'] as String,
      name: json['name'] as String,
      category: json['category'] as String,
      sellingPrice: (json['sellingPrice'] as num).toDouble(),
      imageUrl: json['imageUrl'] as String?,
      isActive: json['isActive'] as bool? ?? true,
      hpp: (json['hpp'] as num?)?.toDouble() ?? 0,
      profit: (json['profit'] as num?)?.toDouble() ?? 0,
      margin: (json['margin'] as num?)?.toDouble() ?? 0,
    );
  }

  @override
  List<Object?> get props => [id, name, category, sellingPrice, isActive];
}
```

- [ ] **Step 3: Create RetailItem model**

```dart
// mobile/lib/models/retail_item.dart
import 'package:equatable/equatable.dart';

class RetailItem extends Equatable {
  final String id;
  final String name;
  final String category;
  final double buyPrice;
  final double sellingPrice;
  final double stockQty;
  final String? imageUrl;
  final double profit;
  final double margin;

  const RetailItem({
    required this.id,
    required this.name,
    required this.category,
    required this.buyPrice,
    required this.sellingPrice,
    required this.stockQty,
    this.imageUrl,
    this.profit = 0,
    this.margin = 0,
  });

  factory RetailItem.fromJson(Map<String, dynamic> json) {
    return RetailItem(
      id: json['id'] as String,
      name: json['name'] as String,
      category: json['category'] as String,
      buyPrice: (json['buyPrice'] as num).toDouble(),
      sellingPrice: (json['sellingPrice'] as num).toDouble(),
      stockQty: (json['stockQty'] as num).toDouble(),
      imageUrl: json['imageUrl'] as String?,
      profit: (json['profit'] as num?)?.toDouble() ?? 0,
      margin: (json['margin'] as num?)?.toDouble() ?? 0,
    );
  }

  @override
  List<Object?> get props => [id, name, category, sellingPrice];
}
```

- [ ] **Step 4: Create CartItem model**

```dart
// mobile/lib/models/cart_item.dart
import 'package:equatable/equatable.dart';

enum CartItemType { menu, retail }

class CartItem extends Equatable {
  final String itemId;
  final String name;
  final CartItemType type;
  final double unitPrice;
  final int quantity;
  final String? notes;
  final String? imageUrl;

  const CartItem({
    required this.itemId,
    required this.name,
    required this.type,
    required this.unitPrice,
    this.quantity = 1,
    this.notes,
    this.imageUrl,
  });

  double get subtotal => unitPrice * quantity;

  CartItem copyWith({
    String? itemId,
    String? name,
    CartItemType? type,
    double? unitPrice,
    int? quantity,
    String? notes,
    String? imageUrl,
  }) {
    return CartItem(
      itemId: itemId ?? this.itemId,
      name: name ?? this.name,
      type: type ?? this.type,
      unitPrice: unitPrice ?? this.unitPrice,
      quantity: quantity ?? this.quantity,
      notes: notes ?? this.notes,
      imageUrl: imageUrl ?? this.imageUrl,
    );
  }

  Map<String, dynamic> toTransactionItem() {
    return {
      'itemType': type == CartItemType.menu ? 'MENU' : 'RETAIL',
      'itemId': itemId,
      'qty': quantity,
    };
  }

  @override
  List<Object?> get props => [itemId, type, quantity, notes];
}
```

- [ ] **Step 5: Create Promo model**

```dart
// mobile/lib/models/promo.dart
import 'package:equatable/equatable.dart';

class Promo extends Equatable {
  final String id;
  final String name;
  final String type;
  final double value;
  final double? minPurchaseAmount;
  final DateTime startDate;
  final DateTime endDate;
  final bool isActive;
  final int? maxUsage;
  final int usedCount;
  final List<PromoItem> items;

  const Promo({
    required this.id,
    required this.name,
    required this.type,
    required this.value,
    this.minPurchaseAmount,
    required this.startDate,
    required this.endDate,
    this.isActive = true,
    this.maxUsage,
    this.usedCount = 0,
    this.items = const [],
  });

  factory Promo.fromJson(Map<String, dynamic> json) {
    return Promo(
      id: json['id'] as String,
      name: json['name'] as String,
      type: json['type'] as String,
      value: (json['value'] as num).toDouble(),
      minPurchaseAmount: (json['minPurchaseAmount'] as num?)?.toDouble(),
      startDate: DateTime.parse(json['startDate'] as String),
      endDate: DateTime.parse(json['endDate'] as String),
      isActive: json['isActive'] as bool? ?? true,
      maxUsage: json['maxUsage'] as int?,
      usedCount: json['usedCount'] as int? ?? 0,
      items: (json['items'] as List?)
              ?.map((e) => PromoItem.fromJson(e as Map<String, dynamic>))
              .toList() ??
          [],
    );
  }

  bool get isValid {
    final now = DateTime.now();
    if (!isActive) return false;
    if (now.isBefore(startDate) || now.isAfter(endDate)) return false;
    if (maxUsage != null && usedCount >= maxUsage!) return false;
    return true;
  }

  double calculateDiscount(double totalAmount) {
    switch (type) {
      case 'PERCENT':
        return (value / 100) * totalAmount;
      case 'FIXED':
        return value > totalAmount ? totalAmount : value;
      case 'MIN_PURCHASE':
        if (minPurchaseAmount != null && totalAmount >= minPurchaseAmount!) {
          return (value / 100) * totalAmount;
        }
        return 0;
      default:
        return 0;
    }
  }

  String get displayLabel {
    switch (type) {
      case 'PERCENT':
        return '${value.toInt()}% OFF';
      case 'FIXED':
        return 'Rp ${value.toInt()} OFF';
      case 'MIN_PURCHASE':
        return '${value.toInt()}% (min Rp ${minPurchaseAmount?.toInt()})';
      case 'BUY_X_GET_Y':
        return 'Buy X Get Y';
      case 'BUNDLE':
        return 'Bundle Deal';
      default:
        return name;
    }
  }

  @override
  List<Object?> get props => [id, name, type, value, isActive];
}

class PromoItem extends Equatable {
  final String id;
  final String itemType;
  final String itemId;
  final String role;

  const PromoItem({
    required this.id,
    required this.itemType,
    required this.itemId,
    required this.role,
  });

  factory PromoItem.fromJson(Map<String, dynamic> json) {
    return PromoItem(
      id: json['id'] as String,
      itemType: json['itemType'] as String,
      itemId: json['itemId'] as String,
      role: json['role'] as String,
    );
  }

  @override
  List<Object?> get props => [id, itemType, itemId, role];
}
```

- [ ] **Step 6: Create Transaction model**

```dart
// mobile/lib/models/transaction.dart
import 'package:equatable/equatable.dart';

class Transaction extends Equatable {
  final String id;
  final String transactionCode;
  final String paymentMethod;
  final double totalAmount;
  final double totalHpp;
  final double totalProfit;
  final double promoDiscount;
  final DateTime createdAt;
  final TransactionCashier? cashier;
  final List<TransactionItem> items;
  final List<TransactionPromo> promos;

  const Transaction({
    required this.id,
    required this.transactionCode,
    required this.paymentMethod,
    required this.totalAmount,
    required this.totalHpp,
    required this.totalProfit,
    this.promoDiscount = 0,
    required this.createdAt,
    this.cashier,
    this.items = const [],
    this.promos = const [],
  });

  factory Transaction.fromJson(Map<String, dynamic> json) {
    return Transaction(
      id: json['id'] as String,
      transactionCode: json['transactionCode'] as String,
      paymentMethod: json['paymentMethod'] as String,
      totalAmount: (json['totalAmount'] as num).toDouble(),
      totalHpp: (json['totalHpp'] as num).toDouble(),
      totalProfit: (json['totalProfit'] as num).toDouble(),
      promoDiscount: (json['promoDiscount'] as num?)?.toDouble() ?? 0,
      createdAt: DateTime.parse(json['createdAt'] as String),
      cashier: json['cashier'] != null
          ? TransactionCashier.fromJson(json['cashier'] as Map<String, dynamic>)
          : null,
      items: (json['items'] as List?)
              ?.map((e) => TransactionItem.fromJson(e as Map<String, dynamic>))
              .toList() ??
          [],
      promos: (json['promos'] as List?)
              ?.map((e) => TransactionPromo.fromJson(e as Map<String, dynamic>))
              .toList() ??
          [],
    );
  }

  @override
  List<Object?> get props => [id, transactionCode];
}

class TransactionItem extends Equatable {
  final String id;
  final String itemType;
  final String itemId;
  final int qty;
  final double unitPrice;
  final double unitHpp;
  final double subtotal;
  final double subHpp;

  const TransactionItem({
    required this.id,
    required this.itemType,
    required this.itemId,
    required this.qty,
    required this.unitPrice,
    required this.unitHpp,
    required this.subtotal,
    required this.subHpp,
  });

  factory TransactionItem.fromJson(Map<String, dynamic> json) {
    return TransactionItem(
      id: json['id'] as String,
      itemType: json['itemType'] as String,
      itemId: json['itemId'] as String,
      qty: json['qty'] as int,
      unitPrice: (json['unitPrice'] as num).toDouble(),
      unitHpp: (json['unitHpp'] as num).toDouble(),
      subtotal: (json['subtotal'] as num).toDouble(),
      subHpp: (json['subHpp'] as num).toDouble(),
    );
  }

  @override
  List<Object?> get props => [id, itemType, itemId, qty];
}

class TransactionCashier extends Equatable {
  final String id;
  final String name;

  const TransactionCashier({required this.id, required this.name});

  factory TransactionCashier.fromJson(Map<String, dynamic> json) {
    return TransactionCashier(
      id: json['id'] as String,
      name: json['name'] as String,
    );
  }

  @override
  List<Object?> get props => [id, name];
}

class TransactionPromo extends Equatable {
  final String id;
  final double discountAmount;

  const TransactionPromo({required this.id, required this.discountAmount});

  factory TransactionPromo.fromJson(Map<String, dynamic> json) {
    return TransactionPromo(
      id: json['id'] as String,
      discountAmount: (json['discountAmount'] as num).toDouble(),
    );
  }

  @override
  List<Object?> get props => [id, discountAmount];
}
```

- [ ] **Step 7: Create Order model**

```dart
// mobile/lib/models/order.dart
import 'package:equatable/equatable.dart';
import 'order_item.dart';

class Order extends Equatable {
  final String id;
  final String transactionId;
  final int orderNumber;
  final String status;
  final DateTime createdAt;
  final DateTime updatedAt;
  final List<OrderItem> items;

  const Order({
    required this.id,
    required this.transactionId,
    required this.orderNumber,
    required this.status,
    required this.createdAt,
    required this.updatedAt,
    this.items = const [],
  });

  factory Order.fromJson(Map<String, dynamic> json) {
    return Order(
      id: json['id'] as String,
      transactionId: json['transactionId'] as String,
      orderNumber: json['orderNumber'] as int,
      status: json['status'] as String,
      createdAt: DateTime.parse(json['createdAt'] as String),
      updatedAt: DateTime.parse(json['updatedAt'] as String),
      items: (json['items'] as List?)
              ?.map((e) => OrderItem.fromJson(e as Map<String, dynamic>))
              .toList() ??
          [],
    );
  }

  Order copyWith({
    String? status,
    DateTime? updatedAt,
    List<OrderItem>? items,
  }) {
    return Order(
      id: id,
      transactionId: transactionId,
      orderNumber: orderNumber,
      status: status ?? this.status,
      createdAt: createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
      items: items ?? this.items,
    );
  }

  Duration get elapsedTime => DateTime.now().difference(createdAt);

  bool get allItemsDone => items.every((item) => item.status == 'DONE');

  @override
  List<Object?> get props => [id, orderNumber, status, items];
}
```

- [ ] **Step 8: Create OrderItem model**

```dart
// mobile/lib/models/order_item.dart
import 'package:equatable/equatable.dart';

class OrderItem extends Equatable {
  final String id;
  final String orderId;
  final String menuItemId;
  final String menuItemName;
  final int qty;
  final String? notes;
  final String status;

  const OrderItem({
    required this.id,
    required this.orderId,
    required this.menuItemId,
    required this.menuItemName,
    required this.qty,
    this.notes,
    required this.status,
  });

  factory OrderItem.fromJson(Map<String, dynamic> json) {
    final menuItem = json['menuItem'] as Map<String, dynamic>?;
    return OrderItem(
      id: json['id'] as String,
      orderId: json['orderId'] as String,
      menuItemId: json['menuItemId'] as String,
      menuItemName: menuItem?['name'] as String? ?? 'Unknown',
      qty: json['qty'] as int,
      notes: json['notes'] as String?,
      status: json['status'] as String,
    );
  }

  OrderItem copyWith({String? status}) {
    return OrderItem(
      id: id,
      orderId: orderId,
      menuItemId: menuItemId,
      menuItemName: menuItemName,
      qty: qty,
      notes: notes,
      status: status ?? this.status,
    );
  }

  @override
  List<Object?> get props => [id, orderId, menuItemId, qty, status];
}
```

---

### Task 4: Shared UI Components & App Theme

**Files:**
- Create: `mobile/lib/config/app_theme.dart`
- Create: `mobile/lib/shared/widgets/app_button.dart`
- Create: `mobile/lib/shared/widgets/app_text_field.dart`
- Create: `mobile/lib/shared/widgets/loading_overlay.dart`
- Create: `mobile/lib/shared/widgets/status_badge.dart`

- [ ] **Step 1: Create AppTheme**

```dart
// mobile/lib/config/app_theme.dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class AppColors {
  static const Color primary = Color(0xFF2563EB);
  static const Color primaryDark = Color(0xFF1D4ED8);
  static const Color secondary = Color(0xFF10B981);
  static const Color error = Color(0xFFEF4444);
  static const Color warning = Color(0xFFF59E0B);
  static const Color info = Color(0xFF3B82F6);

  static const Color background = Color(0xFFF8FAFC);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color surfaceVariant = Color(0xFFF1F5F9);

  static const Color textPrimary = Color(0xFF1E293B);
  static const Color textSecondary = Color(0xFF64748B);
  static const Color textLight = Color(0xFF94A3B8);

  static const Color border = Color(0xFFE2E8F0);
  static const Color divider = Color(0xFFF1F5F9);

  // Order status colors
  static const Color statusPending = Color(0xFFFBBF24);
  static const Color statusPreparing = Color(0xFF3B82F6);
  static const Color statusReady = Color(0xFF10B981);
  static const Color statusServed = Color(0xFF6B7280);

  static Color statusColor(String status) {
    switch (status) {
      case 'PENDING':
        return statusPending;
      case 'PREPARING':
        return statusPreparing;
      case 'READY':
        return statusReady;
      case 'SERVED':
        return statusServed;
      case 'DONE':
        return statusReady;
      default:
        return statusPending;
    }
  }

  static Color statusBackground(String status) {
    switch (status) {
      case 'PENDING':
        return const Color(0xFFFEF3C7);
      case 'PREPARING':
        return const Color(0xFFDBEAFE);
      case 'READY':
        return const Color(0xFFD1FAE5);
      case 'SERVED':
        return const Color(0xFFF3F4F6);
      case 'DONE':
        return const Color(0xFFD1FAE5);
      default:
        return const Color(0xFFFEF3C7);
    }
  }
}

class AppTheme {
  static ThemeData get lightTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: const ColorScheme.light(
        primary: AppColors.primary,
        secondary: AppColors.secondary,
        error: AppColors.error,
        surface: AppColors.surface,
      ),
      scaffoldBackgroundColor: AppColors.background,
      textTheme: GoogleFonts.interTextTheme(),
      appBarTheme: AppBarTheme(
        backgroundColor: AppColors.surface,
        foregroundColor: AppColors.textPrimary,
        elevation: 0,
        centerTitle: false,
        titleTextStyle: GoogleFonts.inter(
          fontSize: 20,
          fontWeight: FontWeight.w600,
          color: AppColors.textPrimary,
        ),
      ),
      cardTheme: CardTheme(
        color: AppColors.surface,
        elevation: 1,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          backgroundColor: AppColors.primary,
          foregroundColor: Colors.white,
          elevation: 0,
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10),
          ),
          textStyle: GoogleFonts.inter(
            fontSize: 14,
            fontWeight: FontWeight.w600,
          ),
        ),
      ),
      inputDecorationTheme: InputDecorationTheme(
        filled: true,
        fillColor: AppColors.surfaceVariant,
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(10),
          borderSide: BorderSide.none,
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(10),
          borderSide: const BorderSide(color: AppColors.primary, width: 2),
        ),
        errorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(10),
          borderSide: const BorderSide(color: AppColors.error, width: 1),
        ),
        contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
        hintStyle: GoogleFonts.inter(
          color: AppColors.textLight,
          fontSize: 14,
        ),
      ),
      dividerTheme: const DividerThemeData(
        color: AppColors.divider,
        thickness: 1,
      ),
    );
  }
}
```

- [ ] **Step 2: Create AppButton**

```dart
// mobile/lib/shared/widgets/app_button.dart
import 'package:flutter/material.dart';
import '../../config/app_theme.dart';

class AppButton extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final bool isLoading;
  final bool isOutlined;
  final Color? color;
  final IconData? icon;
  final double? width;

  const AppButton({
    super.key,
    required this.label,
    this.onPressed,
    this.isLoading = false,
    this.isOutlined = false,
    this.color,
    this.icon,
    this.width,
  });

  @override
  Widget build(BuildContext context) {
    final buttonColor = color ?? AppColors.primary;

    if (isOutlined) {
      return SizedBox(
        width: width,
        child: OutlinedButton(
          onPressed: isLoading ? null : onPressed,
          style: OutlinedButton.styleFrom(
            foregroundColor: buttonColor,
            side: BorderSide(color: buttonColor),
            padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(10),
            ),
          ),
          child: _buildChild(buttonColor),
        ),
      );
    }

    return SizedBox(
      width: width,
      child: ElevatedButton(
        onPressed: isLoading ? null : onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: buttonColor,
          foregroundColor: Colors.white,
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10),
          ),
        ),
        child: _buildChild(Colors.white),
      ),
    );
  }

  Widget _buildChild(Color contentColor) {
    if (isLoading) {
      return SizedBox(
        height: 20,
        width: 20,
        child: CircularProgressIndicator(
          strokeWidth: 2,
          valueColor: AlwaysStoppedAnimation<Color>(contentColor),
        ),
      );
    }

    if (icon != null) {
      return Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, size: 18),
          const SizedBox(width: 8),
          Text(label),
        ],
      );
    }

    return Text(label);
  }
}
```

- [ ] **Step 3: Create AppTextField**

```dart
// mobile/lib/shared/widgets/app_text_field.dart
import 'package:flutter/material.dart';

class AppTextField extends StatelessWidget {
  final String? label;
  final String? hint;
  final TextEditingController? controller;
  final bool obscureText;
  final TextInputType? keyboardType;
  final String? Function(String?)? validator;
  final ValueChanged<String>? onChanged;
  final Widget? prefixIcon;
  final Widget? suffixIcon;
  final int? maxLines;

  const AppTextField({
    super.key,
    this.label,
    this.hint,
    this.controller,
    this.obscureText = false,
    this.keyboardType,
    this.validator,
    this.onChanged,
    this.prefixIcon,
    this.suffixIcon,
    this.maxLines = 1,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      mainAxisSize: MainAxisSize.min,
      children: [
        if (label != null) ...[
          Text(
            label!,
            style: const TextStyle(
              fontSize: 13,
              fontWeight: FontWeight.w600,
              color: Color(0xFF475569),
            ),
          ),
          const SizedBox(height: 6),
        ],
        TextFormField(
          controller: controller,
          obscureText: obscureText,
          keyboardType: keyboardType,
          validator: validator,
          onChanged: onChanged,
          maxLines: maxLines,
          decoration: InputDecoration(
            hintText: hint,
            prefixIcon: prefixIcon,
            suffixIcon: suffixIcon,
          ),
        ),
      ],
    );
  }
}
```

- [ ] **Step 4: Create LoadingOverlay**

```dart
// mobile/lib/shared/widgets/loading_overlay.dart
import 'package:flutter/material.dart';
import '../../config/app_theme.dart';

class LoadingOverlay extends StatelessWidget {
  final bool isLoading;
  final Widget child;
  final String? message;

  const LoadingOverlay({
    super.key,
    required this.isLoading,
    required this.child,
    this.message,
  });

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        child,
        if (isLoading)
          Container(
            color: Colors.black.withOpacity(0.3),
            child: Center(
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(24),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      const CircularProgressIndicator(
                        color: AppColors.primary,
                      ),
                      if (message != null) ...[
                        const SizedBox(height: 16),
                        Text(
                          message!,
                          style: const TextStyle(
                            fontSize: 14,
                            color: AppColors.textSecondary,
                          ),
                        ),
                      ],
                    ],
                  ),
                ),
              ),
            ),
          ),
      ],
    );
  }
}
```

- [ ] **Step 5: Create StatusBadge**

```dart
// mobile/lib/shared/widgets/status_badge.dart
import 'package:flutter/material.dart';
import '../../config/app_theme.dart';

class StatusBadge extends StatelessWidget {
  final String status;
  final double fontSize;

  const StatusBadge({
    super.key,
    required this.status,
    this.fontSize = 11,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
      decoration: BoxDecoration(
        color: AppColors.statusBackground(status),
        borderRadius: BorderRadius.circular(20),
        border: Border.all(
          color: AppColors.statusColor(status).withOpacity(0.4),
        ),
      ),
      child: Text(
        status,
        style: TextStyle(
          fontSize: fontSize,
          fontWeight: FontWeight.w600,
          color: AppColors.statusColor(status),
        ),
      ),
    );
  }
}
```

---

### Task 5: Auth Module — Login, BLoC, JWT Management

**Files:**
- Create: `mobile/lib/modules/auth/repository/auth_repository.dart`
- Create: `mobile/lib/modules/auth/bloc/auth_event.dart`
- Create: `mobile/lib/modules/auth/bloc/auth_state.dart`
- Create: `mobile/lib/modules/auth/bloc/auth_bloc.dart`
- Create: `mobile/lib/modules/auth/screens/login_screen.dart`
- Create: `mobile/test/modules/auth/bloc/auth_bloc_test.dart`

- [ ] **Step 1: Create AuthRepository**

```dart
// mobile/lib/modules/auth/repository/auth_repository.dart
import 'dart:convert';
import '../../../core/api/api_client.dart';
import '../../../models/user.dart';

class AuthRepository {
  final ApiClient _apiClient;

  AuthRepository({required ApiClient apiClient}) : _apiClient = apiClient;

  Future<AuthResponse> login(String email, String password) async {
    final response = await _apiClient.post(
      '/auth/login',
      data: {'email': email, 'password': password},
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data'] as Map<String, dynamic>
        : responseData as Map<String, dynamic>;

    return AuthResponse.fromJson(data);
  }

  Future<User> getProfile() async {
    final response = await _apiClient.get('/auth/me');

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data'] as Map<String, dynamic>
        : responseData as Map<String, dynamic>;

    return User(
      id: data['id'] as String,
      name: data['name'] as String,
      email: data['email'] as String,
      roleName: data['roleName'] as String,
    );
  }
}
```

- [ ] **Step 2: Create AuthEvent**

```dart
// mobile/lib/modules/auth/bloc/auth_event.dart
import 'package:equatable/equatable.dart';

abstract class AuthEvent extends Equatable {
  const AuthEvent();

  @override
  List<Object?> get props => [];
}

class AuthCheckRequested extends AuthEvent {}

class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;

  const AuthLoginRequested({required this.email, required this.password});

  @override
  List<Object?> get props => [email, password];
}

class AuthLogoutRequested extends AuthEvent {}
```

- [ ] **Step 3: Create AuthState**

```dart
// mobile/lib/modules/auth/bloc/auth_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/user.dart';

abstract class AuthState extends Equatable {
  const AuthState();

  @override
  List<Object?> get props => [];
}

class AuthInitial extends AuthState {}

class AuthLoading extends AuthState {}

class AuthAuthenticated extends AuthState {
  final User user;

  const AuthAuthenticated({required this.user});

  @override
  List<Object?> get props => [user];
}

class AuthUnauthenticated extends AuthState {}

class AuthError extends AuthState {
  final String message;

  const AuthError({required this.message});

  @override
  List<Object?> get props => [message];
}
```

- [ ] **Step 4: Create AuthBloc**

```dart
// mobile/lib/modules/auth/bloc/auth_bloc.dart
import 'dart:convert';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/storage/token_storage.dart';
import '../../../core/error/app_exception.dart';
import '../../../models/user.dart';
import '../repository/auth_repository.dart';
import 'auth_event.dart';
import 'auth_state.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository _authRepository;
  final TokenStorage _tokenStorage;

  AuthBloc({
    required AuthRepository authRepository,
    required TokenStorage tokenStorage,
  })  : _authRepository = authRepository,
        _tokenStorage = tokenStorage,
        super(AuthInitial()) {
    on<AuthCheckRequested>(_onAuthCheck);
    on<AuthLoginRequested>(_onLogin);
    on<AuthLogoutRequested>(_onLogout);
  }

  Future<void> _onAuthCheck(
    AuthCheckRequested event,
    Emitter<AuthState> emit,
  ) async {
    final hasToken = await _tokenStorage.hasToken();
    if (!hasToken) {
      emit(AuthUnauthenticated());
      return;
    }

    try {
      final user = await _authRepository.getProfile();
      emit(AuthAuthenticated(user: user));
    } catch (e) {
      await _tokenStorage.clearAll();
      emit(AuthUnauthenticated());
    }
  }

  Future<void> _onLogin(
    AuthLoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());

    try {
      final authResponse = await _authRepository.login(
        event.email,
        event.password,
      );

      await _tokenStorage.saveToken(authResponse.accessToken);
      await _tokenStorage.saveUserData(jsonEncode(authResponse.user.toJson()));

      emit(AuthAuthenticated(user: authResponse.user));
    } on AppException catch (e) {
      emit(AuthError(message: e.message));
      emit(AuthUnauthenticated());
    } catch (e) {
      emit(const AuthError(message: 'Login failed. Please try again.'));
      emit(AuthUnauthenticated());
    }
  }

  Future<void> _onLogout(
    AuthLogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    await _tokenStorage.clearAll();
    emit(AuthUnauthenticated());
  }
}
```

- [ ] **Step 5: Write AuthBloc test**

```dart
// mobile/test/modules/auth/bloc/auth_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:pos_mobile/modules/auth/bloc/auth_bloc.dart';
import 'package:pos_mobile/modules/auth/bloc/auth_event.dart';
import 'package:pos_mobile/modules/auth/bloc/auth_state.dart';
import 'package:pos_mobile/modules/auth/repository/auth_repository.dart';
import 'package:pos_mobile/core/storage/token_storage.dart';
import 'package:pos_mobile/core/error/app_exception.dart';
import 'package:pos_mobile/models/user.dart';

class MockAuthRepository extends Mock implements AuthRepository {}
class MockTokenStorage extends Mock implements TokenStorage {}

void main() {
  late AuthBloc authBloc;
  late MockAuthRepository mockAuthRepository;
  late MockTokenStorage mockTokenStorage;

  const testUser = User(
    id: 'user-1',
    name: 'Kasir Demo',
    email: 'kasir@pos.com',
    roleName: 'KASIR',
  );

  final testAuthResponse = AuthResponse(
    accessToken: 'mock-token',
    user: testUser,
  );

  setUp(() {
    mockAuthRepository = MockAuthRepository();
    mockTokenStorage = MockTokenStorage();

    when(() => mockTokenStorage.saveToken(any())).thenAnswer((_) async {});
    when(() => mockTokenStorage.saveUserData(any())).thenAnswer((_) async {});
    when(() => mockTokenStorage.clearAll()).thenAnswer((_) async {});

    authBloc = AuthBloc(
      authRepository: mockAuthRepository,
      tokenStorage: mockTokenStorage,
    );
  });

  tearDown(() {
    authBloc.close();
  });

  group('AuthCheckRequested', () {
    blocTest<AuthBloc, AuthState>(
      'emits [AuthUnauthenticated] when no token stored',
      build: () {
        when(() => mockTokenStorage.hasToken())
            .thenAnswer((_) async => false);
        return authBloc;
      },
      act: (bloc) => bloc.add(AuthCheckRequested()),
      expect: () => [AuthUnauthenticated()],
    );

    blocTest<AuthBloc, AuthState>(
      'emits [AuthAuthenticated] when valid token exists',
      build: () {
        when(() => mockTokenStorage.hasToken())
            .thenAnswer((_) async => true);
        when(() => mockAuthRepository.getProfile())
            .thenAnswer((_) async => testUser);
        return authBloc;
      },
      act: (bloc) => bloc.add(AuthCheckRequested()),
      expect: () => [const AuthAuthenticated(user: testUser)],
    );

    blocTest<AuthBloc, AuthState>(
      'emits [AuthUnauthenticated] when token is invalid',
      build: () {
        when(() => mockTokenStorage.hasToken())
            .thenAnswer((_) async => true);
        when(() => mockAuthRepository.getProfile())
            .thenThrow(UnauthorizedException());
        return authBloc;
      },
      act: (bloc) => bloc.add(AuthCheckRequested()),
      expect: () => [AuthUnauthenticated()],
    );
  });

  group('AuthLoginRequested', () {
    blocTest<AuthBloc, AuthState>(
      'emits [AuthLoading, AuthAuthenticated] on successful login',
      build: () {
        when(() => mockAuthRepository.login('kasir@pos.com', 'password123'))
            .thenAnswer((_) async => testAuthResponse);
        return authBloc;
      },
      act: (bloc) => bloc.add(const AuthLoginRequested(
        email: 'kasir@pos.com',
        password: 'password123',
      )),
      expect: () => [
        AuthLoading(),
        const AuthAuthenticated(user: testUser),
      ],
      verify: (_) {
        verify(() => mockTokenStorage.saveToken('mock-token')).called(1);
      },
    );

    blocTest<AuthBloc, AuthState>(
      'emits [AuthLoading, AuthError, AuthUnauthenticated] on failed login',
      build: () {
        when(() => mockAuthRepository.login('wrong@pos.com', 'wrong'))
            .thenThrow(AppException(message: 'Invalid credentials', statusCode: 401));
        return authBloc;
      },
      act: (bloc) => bloc.add(const AuthLoginRequested(
        email: 'wrong@pos.com',
        password: 'wrong',
      )),
      expect: () => [
        AuthLoading(),
        const AuthError(message: 'Invalid credentials'),
        AuthUnauthenticated(),
      ],
    );
  });

  group('AuthLogoutRequested', () {
    blocTest<AuthBloc, AuthState>(
      'emits [AuthUnauthenticated] and clears token',
      build: () => authBloc,
      act: (bloc) => bloc.add(AuthLogoutRequested()),
      expect: () => [AuthUnauthenticated()],
      verify: (_) {
        verify(() => mockTokenStorage.clearAll()).called(1);
      },
    );
  });
}
```

- [ ] **Step 6: Run AuthBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/auth/bloc/auth_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7: Create LoginScreen**

```dart
// mobile/lib/modules/auth/screens/login_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/app_button.dart';
import '../../../shared/widgets/app_text_field.dart';
import '../bloc/auth_bloc.dart';
import '../bloc/auth_event.dart';
import '../bloc/auth_state.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _onLogin() {
    if (_formKey.currentState?.validate() ?? false) {
      context.read<AuthBloc>().add(
            AuthLoginRequested(
              email: _emailController.text.trim(),
              password: _passwordController.text,
            ),
          );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColors.background,
      body: BlocListener<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(state.message),
                backgroundColor: AppColors.error,
              ),
            );
          }
        },
        child: Center(
          child: SingleChildScrollView(
            child: Container(
              constraints: const BoxConstraints(maxWidth: 420),
              padding: const EdgeInsets.all(32),
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(40),
                  child: Form(
                    key: _formKey,
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      children: [
                        const Icon(
                          Icons.point_of_sale_rounded,
                          size: 56,
                          color: AppColors.primary,
                        ),
                        const SizedBox(height: 16),
                        Text(
                          'POS System',
                          textAlign: TextAlign.center,
                          style: Theme.of(context)
                              .textTheme
                              .headlineSmall
                              ?.copyWith(fontWeight: FontWeight.w700),
                        ),
                        const SizedBox(height: 8),
                        const Text(
                          'Login to continue',
                          textAlign: TextAlign.center,
                          style: TextStyle(
                            color: AppColors.textSecondary,
                            fontSize: 14,
                          ),
                        ),
                        const SizedBox(height: 32),
                        AppTextField(
                          label: 'Email',
                          hint: 'Enter your email',
                          controller: _emailController,
                          keyboardType: TextInputType.emailAddress,
                          prefixIcon: const Icon(Icons.email_outlined, size: 20),
                          validator: (value) {
                            if (value == null || value.isEmpty) {
                              return 'Email is required';
                            }
                            if (!value.contains('@')) {
                              return 'Enter a valid email';
                            }
                            return null;
                          },
                        ),
                        const SizedBox(height: 20),
                        AppTextField(
                          label: 'Password',
                          hint: 'Enter your password',
                          controller: _passwordController,
                          obscureText: _obscurePassword,
                          prefixIcon: const Icon(Icons.lock_outline, size: 20),
                          suffixIcon: IconButton(
                            icon: Icon(
                              _obscurePassword
                                  ? Icons.visibility_off_outlined
                                  : Icons.visibility_outlined,
                              size: 20,
                            ),
                            onPressed: () {
                              setState(() {
                                _obscurePassword = !_obscurePassword;
                              });
                            },
                          ),
                          validator: (value) {
                            if (value == null || value.isEmpty) {
                              return 'Password is required';
                            }
                            if (value.length < 6) {
                              return 'Password must be at least 6 characters';
                            }
                            return null;
                          },
                        ),
                        const SizedBox(height: 32),
                        BlocBuilder<AuthBloc, AuthState>(
                          builder: (context, state) {
                            return AppButton(
                              label: 'Login',
                              isLoading: state is AuthLoading,
                              onPressed: _onLogin,
                              width: double.infinity,
                            );
                          },
                        ),
                      ],
                    ),
                  ),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

### Task 6: Kasir — Menu Browsing (Grid + Categories + Search)

**Files:**
- Create: `mobile/lib/modules/kasir/repository/menu_repository.dart`
- Create: `mobile/lib/modules/kasir/bloc/menu_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/menu_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/menu_bloc.dart`
- Create: `mobile/lib/modules/kasir/widgets/category_filter.dart`
- Create: `mobile/lib/modules/kasir/widgets/search_bar_widget.dart`
- Create: `mobile/lib/modules/kasir/widgets/menu_item_card.dart`
- Create: `mobile/lib/modules/kasir/widgets/menu_grid.dart`
- Create: `mobile/test/modules/kasir/bloc/menu_bloc_test.dart`

- [ ] **Step 1: Create MenuRepository**

```dart
// mobile/lib/modules/kasir/repository/menu_repository.dart
import '../../../core/api/api_client.dart';
import '../../../models/menu_item.dart';
import '../../../models/retail_item.dart';

class MenuRepository {
  final ApiClient _apiClient;

  MenuRepository({required ApiClient apiClient}) : _apiClient = apiClient;

  Future<List<MenuItem>> getMenuItems({int page = 1, int limit = 100}) async {
    final response = await _apiClient.get(
      '/menu',
      queryParameters: {'page': page, 'limit': limit},
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data']
        : responseData;

    if (data is Map<String, dynamic> && data.containsKey('data')) {
      return (data['data'] as List)
          .map((e) => MenuItem.fromJson(e as Map<String, dynamic>))
          .toList();
    }

    return (data as List)
        .map((e) => MenuItem.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  Future<List<RetailItem>> getRetailItems({int page = 1, int limit = 100}) async {
    final response = await _apiClient.get(
      '/retail',
      queryParameters: {'page': page, 'limit': limit},
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data']
        : responseData;

    if (data is Map<String, dynamic> && data.containsKey('data')) {
      return (data['data'] as List)
          .map((e) => RetailItem.fromJson(e as Map<String, dynamic>))
          .toList();
    }

    return (data as List)
        .map((e) => RetailItem.fromJson(e as Map<String, dynamic>))
        .toList();
  }
}
```

- [ ] **Step 2: Create MenuEvent**

```dart
// mobile/lib/modules/kasir/bloc/menu_event.dart
import 'package:equatable/equatable.dart';

abstract class MenuEvent extends Equatable {
  const MenuEvent();

  @override
  List<Object?> get props => [];
}

class MenuLoadRequested extends MenuEvent {}

class MenuCategoryChanged extends MenuEvent {
  final String category;

  const MenuCategoryChanged({required this.category});

  @override
  List<Object?> get props => [category];
}

class MenuSearchChanged extends MenuEvent {
  final String query;

  const MenuSearchChanged({required this.query});

  @override
  List<Object?> get props => [query];
}
```

- [ ] **Step 3: Create MenuState**

```dart
// mobile/lib/modules/kasir/bloc/menu_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/menu_item.dart';
import '../../../models/retail_item.dart';

class MenuState extends Equatable {
  final List<MenuItem> menuItems;
  final List<RetailItem> retailItems;
  final List<MenuItem> filteredMenuItems;
  final List<RetailItem> filteredRetailItems;
  final String selectedCategory;
  final String searchQuery;
  final bool isLoading;
  final String? error;

  static const String categoryAll = 'ALL';
  static const String categoryFood = 'FOOD';
  static const String categoryBeverage = 'BEVERAGE';
  static const String categoryRetail = 'RETAIL';

  const MenuState({
    this.menuItems = const [],
    this.retailItems = const [],
    this.filteredMenuItems = const [],
    this.filteredRetailItems = const [],
    this.selectedCategory = categoryAll,
    this.searchQuery = '',
    this.isLoading = false,
    this.error,
  });

  MenuState copyWith({
    List<MenuItem>? menuItems,
    List<RetailItem>? retailItems,
    List<MenuItem>? filteredMenuItems,
    List<RetailItem>? filteredRetailItems,
    String? selectedCategory,
    String? searchQuery,
    bool? isLoading,
    String? error,
  }) {
    return MenuState(
      menuItems: menuItems ?? this.menuItems,
      retailItems: retailItems ?? this.retailItems,
      filteredMenuItems: filteredMenuItems ?? this.filteredMenuItems,
      filteredRetailItems: filteredRetailItems ?? this.filteredRetailItems,
      selectedCategory: selectedCategory ?? this.selectedCategory,
      searchQuery: searchQuery ?? this.searchQuery,
      isLoading: isLoading ?? this.isLoading,
      error: error,
    );
  }

  @override
  List<Object?> get props => [
        menuItems,
        retailItems,
        filteredMenuItems,
        filteredRetailItems,
        selectedCategory,
        searchQuery,
        isLoading,
        error,
      ];
}
```

- [ ] **Step 4: Create MenuBloc**

```dart
// mobile/lib/modules/kasir/bloc/menu_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/error/app_exception.dart';
import '../../../models/menu_item.dart';
import '../../../models/retail_item.dart';
import '../repository/menu_repository.dart';
import 'menu_event.dart';
import 'menu_state.dart';

class MenuBloc extends Bloc<MenuEvent, MenuState> {
  final MenuRepository _menuRepository;

  MenuBloc({required MenuRepository menuRepository})
      : _menuRepository = menuRepository,
        super(const MenuState()) {
    on<MenuLoadRequested>(_onLoadMenu);
    on<MenuCategoryChanged>(_onCategoryChanged);
    on<MenuSearchChanged>(_onSearchChanged);
  }

  Future<void> _onLoadMenu(
    MenuLoadRequested event,
    Emitter<MenuState> emit,
  ) async {
    emit(state.copyWith(isLoading: true, error: null));

    try {
      final results = await Future.wait([
        _menuRepository.getMenuItems(),
        _menuRepository.getRetailItems(),
      ]);

      final menuItems = results[0] as List<MenuItem>;
      final retailItems = results[1] as List<RetailItem>;

      emit(state.copyWith(
        menuItems: menuItems,
        retailItems: retailItems,
        filteredMenuItems: menuItems,
        filteredRetailItems: retailItems,
        isLoading: false,
      ));
    } on AppException catch (e) {
      emit(state.copyWith(isLoading: false, error: e.message));
    } catch (e) {
      emit(state.copyWith(isLoading: false, error: 'Failed to load menu'));
    }
  }

  void _onCategoryChanged(
    MenuCategoryChanged event,
    Emitter<MenuState> emit,
  ) {
    emit(state.copyWith(selectedCategory: event.category));
    _applyFilters(emit);
  }

  void _onSearchChanged(
    MenuSearchChanged event,
    Emitter<MenuState> emit,
  ) {
    emit(state.copyWith(searchQuery: event.query));
    _applyFilters(emit);
  }

  void _applyFilters(Emitter<MenuState> emit) {
    final query = state.searchQuery.toLowerCase();
    final category = state.selectedCategory;

    List<MenuItem> filteredMenu = state.menuItems;
    List<RetailItem> filteredRetail = state.retailItems;

    // Filter by category
    if (category == MenuState.categoryFood) {
      filteredMenu = filteredMenu
          .where((item) => item.category == 'FOOD')
          .toList();
      filteredRetail = [];
    } else if (category == MenuState.categoryBeverage) {
      filteredMenu = filteredMenu
          .where((item) => item.category == 'BEVERAGE')
          .toList();
      filteredRetail = [];
    } else if (category == MenuState.categoryRetail) {
      filteredMenu = [];
    }

    // Filter by search query
    if (query.isNotEmpty) {
      filteredMenu = filteredMenu
          .where((item) => item.name.toLowerCase().contains(query))
          .toList();
      filteredRetail = filteredRetail
          .where((item) => item.name.toLowerCase().contains(query))
          .toList();
    }

    // Filter only active menu items
    filteredMenu = filteredMenu.where((item) => item.isActive).toList();

    emit(state.copyWith(
      filteredMenuItems: filteredMenu,
      filteredRetailItems: filteredRetail,
    ));
  }
}
```

- [ ] **Step 5: Write MenuBloc test**

```dart
// mobile/test/modules/kasir/bloc/menu_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:pos_mobile/modules/kasir/bloc/menu_bloc.dart';
import 'package:pos_mobile/modules/kasir/bloc/menu_event.dart';
import 'package:pos_mobile/modules/kasir/bloc/menu_state.dart';
import 'package:pos_mobile/modules/kasir/repository/menu_repository.dart';
import 'package:pos_mobile/models/menu_item.dart';
import 'package:pos_mobile/models/retail_item.dart';

class MockMenuRepository extends Mock implements MenuRepository {}

void main() {
  late MenuBloc menuBloc;
  late MockMenuRepository mockMenuRepository;

  final testMenuItems = [
    const MenuItem(
      id: 'm1',
      name: 'Nasi Goreng',
      category: 'FOOD',
      sellingPrice: 25000,
      isActive: true,
    ),
    const MenuItem(
      id: 'm2',
      name: 'Kopi Susu',
      category: 'BEVERAGE',
      sellingPrice: 18000,
      isActive: true,
    ),
  ];

  final testRetailItems = [
    const RetailItem(
      id: 'r1',
      name: 'Saus Sambal',
      category: 'Condiment',
      buyPrice: 15000,
      sellingPrice: 25000,
      stockQty: 20,
    ),
  ];

  setUp(() {
    mockMenuRepository = MockMenuRepository();
    menuBloc = MenuBloc(menuRepository: mockMenuRepository);
  });

  tearDown(() {
    menuBloc.close();
  });

  group('MenuLoadRequested', () {
    blocTest<MenuBloc, MenuState>(
      'emits loading then loaded state with menu and retail items',
      build: () {
        when(() => mockMenuRepository.getMenuItems())
            .thenAnswer((_) async => testMenuItems);
        when(() => mockMenuRepository.getRetailItems())
            .thenAnswer((_) async => testRetailItems);
        return menuBloc;
      },
      act: (bloc) => bloc.add(MenuLoadRequested()),
      expect: () => [
        const MenuState(isLoading: true),
        MenuState(
          menuItems: testMenuItems,
          retailItems: testRetailItems,
          filteredMenuItems: testMenuItems,
          filteredRetailItems: testRetailItems,
          isLoading: false,
        ),
      ],
    );
  });

  group('MenuCategoryChanged', () {
    blocTest<MenuBloc, MenuState>(
      'filters to show only FOOD items when FOOD selected',
      build: () {
        when(() => mockMenuRepository.getMenuItems())
            .thenAnswer((_) async => testMenuItems);
        when(() => mockMenuRepository.getRetailItems())
            .thenAnswer((_) async => testRetailItems);
        return menuBloc;
      },
      seed: () => MenuState(
        menuItems: testMenuItems,
        retailItems: testRetailItems,
        filteredMenuItems: testMenuItems,
        filteredRetailItems: testRetailItems,
      ),
      act: (bloc) => bloc.add(const MenuCategoryChanged(category: 'FOOD')),
      expect: () => [
        MenuState(
          menuItems: testMenuItems,
          retailItems: testRetailItems,
          filteredMenuItems: [testMenuItems[0]],
          filteredRetailItems: const [],
          selectedCategory: 'FOOD',
        ),
      ],
    );
  });

  group('MenuSearchChanged', () {
    blocTest<MenuBloc, MenuState>(
      'filters items by search query',
      build: () => menuBloc,
      seed: () => MenuState(
        menuItems: testMenuItems,
        retailItems: testRetailItems,
        filteredMenuItems: testMenuItems,
        filteredRetailItems: testRetailItems,
      ),
      act: (bloc) => bloc.add(const MenuSearchChanged(query: 'nasi')),
      expect: () => [
        MenuState(
          menuItems: testMenuItems,
          retailItems: testRetailItems,
          filteredMenuItems: [testMenuItems[0]],
          filteredRetailItems: const [],
          searchQuery: 'nasi',
        ),
      ],
    );
  });
}
```

- [ ] **Step 6: Run MenuBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/menu_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7: Create CategoryFilter widget**

```dart
// mobile/lib/modules/kasir/widgets/category_filter.dart
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';
import '../bloc/menu_state.dart';

class CategoryFilter extends StatelessWidget {
  final String selectedCategory;
  final ValueChanged<String> onCategoryChanged;

  const CategoryFilter({
    super.key,
    required this.selectedCategory,
    required this.onCategoryChanged,
  });

  static const _categories = [
    {'key': MenuState.categoryAll, 'label': 'Semua', 'icon': Icons.grid_view_rounded},
    {'key': MenuState.categoryFood, 'label': 'Makanan', 'icon': Icons.restaurant},
    {'key': MenuState.categoryBeverage, 'label': 'Minuman', 'icon': Icons.local_cafe},
    {'key': MenuState.categoryRetail, 'label': 'Retail', 'icon': Icons.shopping_bag},
  ];

  @override
  Widget build(BuildContext context) {
    return Row(
      children: _categories.map((cat) {
        final isSelected = selectedCategory == cat['key'];
        return Padding(
          padding: const EdgeInsets.only(right: 8),
          child: FilterChip(
            label: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                Icon(
                  cat['icon'] as IconData,
                  size: 16,
                  color: isSelected ? Colors.white : AppColors.textSecondary,
                ),
                const SizedBox(width: 6),
                Text(cat['label'] as String),
              ],
            ),
            selected: isSelected,
            onSelected: (_) => onCategoryChanged(cat['key'] as String),
            selectedColor: AppColors.primary,
            backgroundColor: AppColors.surface,
            labelStyle: TextStyle(
              color: isSelected ? Colors.white : AppColors.textPrimary,
              fontWeight: FontWeight.w500,
              fontSize: 13,
            ),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(20),
              side: BorderSide(
                color: isSelected ? AppColors.primary : AppColors.border,
              ),
            ),
            showCheckmark: false,
            padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 6),
          ),
        );
      }).toList(),
    );
  }
}
```

- [ ] **Step 8: Create SearchBarWidget**

```dart
// mobile/lib/modules/kasir/widgets/search_bar_widget.dart
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';

class SearchBarWidget extends StatelessWidget {
  final ValueChanged<String> onChanged;
  final String hintText;

  const SearchBarWidget({
    super.key,
    required this.onChanged,
    this.hintText = 'Cari menu atau item...',
  });

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 42,
      child: TextField(
        onChanged: onChanged,
        decoration: InputDecoration(
          hintText: hintText,
          hintStyle: const TextStyle(fontSize: 13, color: AppColors.textLight),
          prefixIcon: const Icon(Icons.search, size: 20, color: AppColors.textLight),
          filled: true,
          fillColor: AppColors.surfaceVariant,
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(10),
            borderSide: BorderSide.none,
          ),
          contentPadding: const EdgeInsets.symmetric(horizontal: 12, vertical: 0),
        ),
      ),
    );
  }
}
```

- [ ] **Step 9: Create MenuItemCard widget**

```dart
// mobile/lib/modules/kasir/widgets/menu_item_card.dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

class MenuItemCard extends StatelessWidget {
  final String name;
  final double price;
  final String? imageUrl;
  final String category;
  final bool isRetail;
  final VoidCallback onTap;

  const MenuItemCard({
    super.key,
    required this.name,
    required this.price,
    this.imageUrl,
    required this.category,
    this.isRetail = false,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Material(
      color: AppColors.surface,
      borderRadius: BorderRadius.circular(12),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(12),
            border: Border.all(color: AppColors.border),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Expanded(
                flex: 3,
                child: Container(
                  width: double.infinity,
                  decoration: BoxDecoration(
                    color: AppColors.surfaceVariant,
                    borderRadius: const BorderRadius.vertical(
                      top: Radius.circular(12),
                    ),
                  ),
                  child: imageUrl != null && imageUrl!.isNotEmpty
                      ? ClipRRect(
                          borderRadius: const BorderRadius.vertical(
                            top: Radius.circular(12),
                          ),
                          child: Image.network(
                            imageUrl!,
                            fit: BoxFit.cover,
                            errorBuilder: (_, __, ___) => _buildPlaceholder(),
                          ),
                        )
                      : _buildPlaceholder(),
                ),
              ),
              Expanded(
                flex: 2,
                child: Padding(
                  padding: const EdgeInsets.all(10),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        name,
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                        style: const TextStyle(
                          fontSize: 13,
                          fontWeight: FontWeight.w600,
                          color: AppColors.textPrimary,
                        ),
                      ),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Flexible(
                            child: Text(
                              _currencyFormat.format(price),
                              style: const TextStyle(
                                fontSize: 13,
                                fontWeight: FontWeight.w700,
                                color: AppColors.primary,
                              ),
                            ),
                          ),
                          Container(
                            padding: const EdgeInsets.symmetric(
                              horizontal: 6,
                              vertical: 2,
                            ),
                            decoration: BoxDecoration(
                              color: isRetail
                                  ? AppColors.warning.withOpacity(0.1)
                                  : category == 'FOOD'
                                      ? AppColors.error.withOpacity(0.1)
                                      : AppColors.info.withOpacity(0.1),
                              borderRadius: BorderRadius.circular(4),
                            ),
                            child: Text(
                              isRetail ? 'Retail' : category,
                              style: TextStyle(
                                fontSize: 9,
                                fontWeight: FontWeight.w600,
                                color: isRetail
                                    ? AppColors.warning
                                    : category == 'FOOD'
                                        ? AppColors.error
                                        : AppColors.info,
                              ),
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildPlaceholder() {
    return Center(
      child: Icon(
        isRetail
            ? Icons.shopping_bag_outlined
            : category == 'FOOD'
                ? Icons.restaurant
                : Icons.local_cafe,
        size: 36,
        color: AppColors.textLight,
      ),
    );
  }
}
```

- [ ] **Step 10: Create MenuGrid widget**

```dart
// mobile/lib/modules/kasir/widgets/menu_grid.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../models/menu_item.dart' as models;
import '../../../models/retail_item.dart';
import '../../../models/cart_item.dart';
import '../bloc/menu_bloc.dart';
import '../bloc/menu_event.dart';
import '../bloc/menu_state.dart';
import '../bloc/cart_bloc.dart';
import '../bloc/cart_event.dart';
import 'category_filter.dart';
import 'search_bar_widget.dart';
import 'menu_item_card.dart';

class MenuGrid extends StatelessWidget {
  const MenuGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<MenuBloc, MenuState>(
      builder: (context, state) {
        return Column(
          children: [
            Padding(
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  SearchBarWidget(
                    onChanged: (query) {
                      context
                          .read<MenuBloc>()
                          .add(MenuSearchChanged(query: query));
                    },
                  ),
                  const SizedBox(height: 12),
                  CategoryFilter(
                    selectedCategory: state.selectedCategory,
                    onCategoryChanged: (category) {
                      context
                          .read<MenuBloc>()
                          .add(MenuCategoryChanged(category: category));
                    },
                  ),
                ],
              ),
            ),
            Expanded(
              child: state.isLoading
                  ? const Center(child: CircularProgressIndicator())
                  : state.error != null
                      ? Center(child: Text(state.error!))
                      : _buildGrid(context, state),
            ),
          ],
        );
      },
    );
  }

  Widget _buildGrid(BuildContext context, MenuState state) {
    final menuCards = state.filteredMenuItems.map((item) => _MenuCardData(
          id: item.id,
          name: item.name,
          price: item.sellingPrice,
          imageUrl: item.imageUrl,
          category: item.category,
          isRetail: false,
        ));

    final retailCards = (state.selectedCategory != MenuState.categoryFood &&
            state.selectedCategory != MenuState.categoryBeverage)
        ? state.filteredRetailItems.map((item) => _MenuCardData(
              id: item.id,
              name: item.name,
              price: item.sellingPrice,
              imageUrl: item.imageUrl,
              category: item.category,
              isRetail: true,
            ))
        : <_MenuCardData>[];

    final allCards = [...menuCards, ...retailCards];

    if (allCards.isEmpty) {
      return const Center(
        child: Text(
          'Tidak ada item ditemukan',
          style: TextStyle(color: Color(0xFF94A3B8)),
        ),
      );
    }

    return GridView.builder(
      padding: const EdgeInsets.fromLTRB(16, 0, 16, 16),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 4,
        childAspectRatio: 0.78,
        mainAxisSpacing: 12,
        crossAxisSpacing: 12,
      ),
      itemCount: allCards.length,
      itemBuilder: (context, index) {
        final card = allCards.elementAt(index);
        return MenuItemCard(
          name: card.name,
          price: card.price,
          imageUrl: card.imageUrl,
          category: card.category,
          isRetail: card.isRetail,
          onTap: () {
            context.read<CartBloc>().add(CartItemAdded(
                  item: CartItem(
                    itemId: card.id,
                    name: card.name,
                    type: card.isRetail ? CartItemType.retail : CartItemType.menu,
                    unitPrice: card.price,
                    imageUrl: card.imageUrl,
                  ),
                ));
          },
        );
      },
    );
  }
}

class _MenuCardData {
  final String id;
  final String name;
  final double price;
  final String? imageUrl;
  final String category;
  final bool isRetail;

  _MenuCardData({
    required this.id,
    required this.name,
    required this.price,
    this.imageUrl,
    required this.category,
    required this.isRetail,
  });
}
```

---

### Task 7: Kasir — Cart Management

**Files:**
- Create: `mobile/lib/modules/kasir/bloc/cart_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/cart_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/cart_bloc.dart`
- Create: `mobile/lib/modules/kasir/widgets/cart_panel.dart`
- Create: `mobile/lib/modules/kasir/widgets/cart_item_tile.dart`
- Create: `mobile/test/modules/kasir/bloc/cart_bloc_test.dart`

- [ ] **Step 1: Create CartEvent**

```dart
// mobile/lib/modules/kasir/bloc/cart_event.dart
import 'package:equatable/equatable.dart';
import '../../../models/cart_item.dart';

abstract class CartEvent extends Equatable {
  const CartEvent();

  @override
  List<Object?> get props => [];
}

class CartItemAdded extends CartEvent {
  final CartItem item;

  const CartItemAdded({required this.item});

  @override
  List<Object?> get props => [item];
}

class CartItemRemoved extends CartEvent {
  final String itemId;

  const CartItemRemoved({required this.itemId});

  @override
  List<Object?> get props => [itemId];
}

class CartItemQuantityUpdated extends CartEvent {
  final String itemId;
  final int quantity;

  const CartItemQuantityUpdated({required this.itemId, required this.quantity});

  @override
  List<Object?> get props => [itemId, quantity];
}

class CartItemNotesUpdated extends CartEvent {
  final String itemId;
  final String notes;

  const CartItemNotesUpdated({required this.itemId, required this.notes});

  @override
  List<Object?> get props => [itemId, notes];
}

class CartCleared extends CartEvent {}
```

- [ ] **Step 2: Create CartState**

```dart
// mobile/lib/modules/kasir/bloc/cart_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/cart_item.dart';

class CartState extends Equatable {
  final List<CartItem> items;

  const CartState({this.items = const []});

  double get subtotal => items.fold(0, (sum, item) => sum + item.subtotal);

  int get totalItems => items.fold(0, (sum, item) => sum + item.quantity);

  bool get isEmpty => items.isEmpty;

  CartState copyWith({List<CartItem>? items}) {
    return CartState(items: items ?? this.items);
  }

  @override
  List<Object?> get props => [items];
}
```

- [ ] **Step 3: Create CartBloc**

```dart
// mobile/lib/modules/kasir/bloc/cart_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../models/cart_item.dart';
import 'cart_event.dart';
import 'cart_state.dart';

class CartBloc extends Bloc<CartEvent, CartState> {
  CartBloc() : super(const CartState()) {
    on<CartItemAdded>(_onItemAdded);
    on<CartItemRemoved>(_onItemRemoved);
    on<CartItemQuantityUpdated>(_onQuantityUpdated);
    on<CartItemNotesUpdated>(_onNotesUpdated);
    on<CartCleared>(_onCleared);
  }

  void _onItemAdded(CartItemAdded event, Emitter<CartState> emit) {
    final existingIndex = state.items.indexWhere(
      (item) => item.itemId == event.item.itemId && item.type == event.item.type,
    );

    if (existingIndex >= 0) {
      final updatedItems = List<CartItem>.from(state.items);
      final existing = updatedItems[existingIndex];
      updatedItems[existingIndex] = existing.copyWith(
        quantity: existing.quantity + 1,
      );
      emit(state.copyWith(items: updatedItems));
    } else {
      emit(state.copyWith(items: [...state.items, event.item]));
    }
  }

  void _onItemRemoved(CartItemRemoved event, Emitter<CartState> emit) {
    final updatedItems = state.items
        .where((item) => item.itemId != event.itemId)
        .toList();
    emit(state.copyWith(items: updatedItems));
  }

  void _onQuantityUpdated(
    CartItemQuantityUpdated event,
    Emitter<CartState> emit,
  ) {
    if (event.quantity <= 0) {
      add(CartItemRemoved(itemId: event.itemId));
      return;
    }

    final updatedItems = state.items.map((item) {
      if (item.itemId == event.itemId) {
        return item.copyWith(quantity: event.quantity);
      }
      return item;
    }).toList();

    emit(state.copyWith(items: updatedItems));
  }

  void _onNotesUpdated(CartItemNotesUpdated event, Emitter<CartState> emit) {
    final updatedItems = state.items.map((item) {
      if (item.itemId == event.itemId) {
        return item.copyWith(notes: event.notes);
      }
      return item;
    }).toList();

    emit(state.copyWith(items: updatedItems));
  }

  void _onCleared(CartCleared event, Emitter<CartState> emit) {
    emit(const CartState());
  }
}
```

- [ ] **Step 4: Write CartBloc test**

```dart
// mobile/test/modules/kasir/bloc/cart_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:pos_mobile/modules/kasir/bloc/cart_bloc.dart';
import 'package:pos_mobile/modules/kasir/bloc/cart_event.dart';
import 'package:pos_mobile/modules/kasir/bloc/cart_state.dart';
import 'package:pos_mobile/models/cart_item.dart';

void main() {
  late CartBloc cartBloc;

  const testItem = CartItem(
    itemId: 'm1',
    name: 'Nasi Goreng',
    type: CartItemType.menu,
    unitPrice: 25000,
  );

  const testRetailItem = CartItem(
    itemId: 'r1',
    name: 'Saus Sambal',
    type: CartItemType.retail,
    unitPrice: 25000,
  );

  setUp(() {
    cartBloc = CartBloc();
  });

  tearDown(() {
    cartBloc.close();
  });

  group('CartItemAdded', () {
    blocTest<CartBloc, CartState>(
      'adds new item to cart',
      build: () => cartBloc,
      act: (bloc) => bloc.add(const CartItemAdded(item: testItem)),
      expect: () => [
        const CartState(items: [testItem]),
      ],
    );

    blocTest<CartBloc, CartState>(
      'increments quantity when same item added again',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem]),
      act: (bloc) => bloc.add(const CartItemAdded(item: testItem)),
      expect: () => [
        CartState(items: [testItem.copyWith(quantity: 2)]),
      ],
    );

    blocTest<CartBloc, CartState>(
      'adds separate entry for different item types',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem]),
      act: (bloc) => bloc.add(const CartItemAdded(item: testRetailItem)),
      expect: () => [
        const CartState(items: [testItem, testRetailItem]),
      ],
    );
  });

  group('CartItemRemoved', () {
    blocTest<CartBloc, CartState>(
      'removes item from cart',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem, testRetailItem]),
      act: (bloc) => bloc.add(const CartItemRemoved(itemId: 'm1')),
      expect: () => [
        const CartState(items: [testRetailItem]),
      ],
    );
  });

  group('CartItemQuantityUpdated', () {
    blocTest<CartBloc, CartState>(
      'updates item quantity',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem]),
      act: (bloc) => bloc.add(
          const CartItemQuantityUpdated(itemId: 'm1', quantity: 3)),
      expect: () => [
        CartState(items: [testItem.copyWith(quantity: 3)]),
      ],
    );
  });

  group('CartItemNotesUpdated', () {
    blocTest<CartBloc, CartState>(
      'updates item notes',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem]),
      act: (bloc) => bloc.add(
          const CartItemNotesUpdated(itemId: 'm1', notes: 'Extra pedas')),
      expect: () => [
        CartState(items: [testItem.copyWith(notes: 'Extra pedas')]),
      ],
    );
  });

  group('CartCleared', () {
    blocTest<CartBloc, CartState>(
      'clears all items',
      build: () => cartBloc,
      seed: () => const CartState(items: [testItem, testRetailItem]),
      act: (bloc) => bloc.add(CartCleared()),
      expect: () => [const CartState()],
    );
  });

  group('CartState computed properties', () {
    test('subtotal calculates correctly', () {
      final state = CartState(items: [
        testItem.copyWith(quantity: 2),
        testRetailItem,
      ]);
      expect(state.subtotal, 75000);
    });

    test('totalItems counts correctly', () {
      final state = CartState(items: [
        testItem.copyWith(quantity: 2),
        testRetailItem,
      ]);
      expect(state.totalItems, 3);
    });
  });
}
```

- [ ] **Step 5: Run CartBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/cart_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 6: Create CartItemTile widget**

```dart
// mobile/lib/modules/kasir/widgets/cart_item_tile.dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';
import '../../../models/cart_item.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

class CartItemTile extends StatelessWidget {
  final CartItem item;
  final ValueChanged<int> onQuantityChanged;
  final VoidCallback onRemove;
  final ValueChanged<String> onNotesChanged;

  const CartItemTile({
    super.key,
    required this.item,
    required this.onQuantityChanged,
    required this.onRemove,
    required this.onNotesChanged,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
      decoration: const BoxDecoration(
        border: Border(
          bottom: BorderSide(color: AppColors.divider),
        ),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      item.name,
                      style: const TextStyle(
                        fontSize: 13,
                        fontWeight: FontWeight.w600,
                        color: AppColors.textPrimary,
                      ),
                    ),
                    const SizedBox(height: 2),
                    Text(
                      _currencyFormat.format(item.unitPrice),
                      style: const TextStyle(
                        fontSize: 12,
                        color: AppColors.textSecondary,
                      ),
                    ),
                  ],
                ),
              ),
              IconButton(
                icon: const Icon(Icons.close, size: 16),
                onPressed: onRemove,
                padding: EdgeInsets.zero,
                constraints: const BoxConstraints(
                  minWidth: 24,
                  minHeight: 24,
                ),
                color: AppColors.textLight,
              ),
            ],
          ),
          const SizedBox(height: 6),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Row(
                children: [
                  _QuantityButton(
                    icon: Icons.remove,
                    onTap: () => onQuantityChanged(item.quantity - 1),
                  ),
                  Padding(
                    padding: const EdgeInsets.symmetric(horizontal: 12),
                    child: Text(
                      '${item.quantity}',
                      style: const TextStyle(
                        fontSize: 14,
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                  ),
                  _QuantityButton(
                    icon: Icons.add,
                    onTap: () => onQuantityChanged(item.quantity + 1),
                  ),
                ],
              ),
              Text(
                _currencyFormat.format(item.subtotal),
                style: const TextStyle(
                  fontSize: 13,
                  fontWeight: FontWeight.w700,
                  color: AppColors.primary,
                ),
              ),
            ],
          ),
          if (item.type == CartItemType.menu) ...[
            const SizedBox(height: 6),
            GestureDetector(
              onTap: () => _showNotesDialog(context),
              child: Container(
                width: double.infinity,
                padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                decoration: BoxDecoration(
                  color: AppColors.surfaceVariant,
                  borderRadius: BorderRadius.circular(4),
                ),
                child: Text(
                  item.notes?.isNotEmpty == true ? item.notes! : 'Tambah catatan...',
                  style: TextStyle(
                    fontSize: 11,
                    color: item.notes?.isNotEmpty == true
                        ? AppColors.textPrimary
                        : AppColors.textLight,
                    fontStyle: item.notes?.isNotEmpty == true
                        ? FontStyle.normal
                        : FontStyle.italic,
                  ),
                ),
              ),
            ),
          ],
        ],
      ),
    );
  }

  void _showNotesDialog(BuildContext context) {
    final controller = TextEditingController(text: item.notes);

    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: Text('Catatan - ${item.name}',
            style: const TextStyle(fontSize: 16)),
        content: TextField(
          controller: controller,
          autofocus: true,
          maxLines: 3,
          decoration: const InputDecoration(
            hintText: 'Contoh: Extra pedas, tanpa sayur',
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: const Text('Batal'),
          ),
          ElevatedButton(
            onPressed: () {
              onNotesChanged(controller.text);
              Navigator.pop(ctx);
            },
            child: const Text('Simpan'),
          ),
        ],
      ),
    );
  }
}

class _QuantityButton extends StatelessWidget {
  final IconData icon;
  final VoidCallback onTap;

  const _QuantityButton({required this.icon, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return Material(
      color: AppColors.surfaceVariant,
      borderRadius: BorderRadius.circular(6),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(6),
        child: Container(
          width: 28,
          height: 28,
          alignment: Alignment.center,
          child: Icon(icon, size: 16, color: AppColors.textPrimary),
        ),
      ),
    );
  }
}
```

- [ ] **Step 7: Create CartPanel widget**

```dart
// mobile/lib/modules/kasir/widgets/cart_panel.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/app_button.dart';
import '../bloc/cart_bloc.dart';
import '../bloc/cart_event.dart';
import '../bloc/cart_state.dart';
import '../bloc/promo_bloc.dart';
import '../bloc/promo_state.dart';
import 'cart_item_tile.dart';
import 'promo_badge.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

class CartPanel extends StatelessWidget {
  final VoidCallback onCheckout;

  const CartPanel({super.key, required this.onCheckout});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 340,
      decoration: BoxDecoration(
        color: AppColors.surface,
        border: Border(
          left: BorderSide(color: AppColors.border),
        ),
      ),
      child: Column(
        children: [
          _buildHeader(context),
          Expanded(
            child: BlocBuilder<CartBloc, CartState>(
              builder: (context, state) {
                if (state.isEmpty) {
                  return const Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Icon(Icons.shopping_cart_outlined,
                            size: 48, color: AppColors.textLight),
                        SizedBox(height: 12),
                        Text(
                          'Keranjang kosong',
                          style: TextStyle(
                            color: AppColors.textLight,
                            fontSize: 14,
                          ),
                        ),
                        SizedBox(height: 4),
                        Text(
                          'Tap item menu untuk menambahkan',
                          style: TextStyle(
                            color: AppColors.textLight,
                            fontSize: 12,
                          ),
                        ),
                      ],
                    ),
                  );
                }

                return ListView.builder(
                  itemCount: state.items.length,
                  itemBuilder: (context, index) {
                    final item = state.items[index];
                    return CartItemTile(
                      item: item,
                      onQuantityChanged: (qty) {
                        context.read<CartBloc>().add(
                              CartItemQuantityUpdated(
                                itemId: item.itemId,
                                quantity: qty,
                              ),
                            );
                      },
                      onRemove: () {
                        context.read<CartBloc>().add(
                              CartItemRemoved(itemId: item.itemId),
                            );
                      },
                      onNotesChanged: (notes) {
                        context.read<CartBloc>().add(
                              CartItemNotesUpdated(
                                itemId: item.itemId,
                                notes: notes,
                              ),
                            );
                      },
                    );
                  },
                );
              },
            ),
          ),
          _buildPromoSection(context),
          _buildFooter(context),
        ],
      ),
    );
  }

  Widget _buildHeader(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
      decoration: const BoxDecoration(
        border: Border(bottom: BorderSide(color: AppColors.divider)),
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          BlocBuilder<CartBloc, CartState>(
            builder: (context, state) {
              return Row(
                children: [
                  const Icon(Icons.shopping_cart, size: 20, color: AppColors.primary),
                  const SizedBox(width: 8),
                  Text(
                    'Keranjang (${state.totalItems})',
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                ],
              );
            },
          ),
          BlocBuilder<CartBloc, CartState>(
            builder: (context, state) {
              if (state.isEmpty) return const SizedBox.shrink();
              return TextButton(
                onPressed: () {
                  context.read<CartBloc>().add(CartCleared());
                },
                style: TextButton.styleFrom(
                  foregroundColor: AppColors.error,
                  padding: const EdgeInsets.symmetric(horizontal: 8),
                ),
                child: const Text('Hapus Semua', style: TextStyle(fontSize: 12)),
              );
            },
          ),
        ],
      ),
    );
  }

  Widget _buildPromoSection(BuildContext context) {
    return BlocBuilder<PromoBloc, PromoState>(
      builder: (context, promoState) {
        return BlocBuilder<CartBloc, CartState>(
          builder: (context, cartState) {
            if (cartState.isEmpty) return const SizedBox.shrink();

            return PromoBadge(
              promoState: promoState,
              cartSubtotal: cartState.subtotal,
            );
          },
        );
      },
    );
  }

  Widget _buildFooter(BuildContext context) {
    return BlocBuilder<CartBloc, CartState>(
      builder: (context, cartState) {
        return BlocBuilder<PromoBloc, PromoState>(
          builder: (context, promoState) {
            final subtotal = cartState.subtotal;
            final discount = promoState.selectedPromo?.calculateDiscount(subtotal) ?? 0;
            final total = subtotal - discount;

            return Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: AppColors.surface,
                border: const Border(
                  top: BorderSide(color: AppColors.divider),
                ),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.05),
                    blurRadius: 10,
                    offset: const Offset(0, -2),
                  ),
                ],
              ),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Subtotal',
                          style: TextStyle(
                              fontSize: 13, color: AppColors.textSecondary)),
                      Text(_currencyFormat.format(subtotal),
                          style: const TextStyle(
                              fontSize: 13, fontWeight: FontWeight.w500)),
                    ],
                  ),
                  if (discount > 0) ...[
                    const SizedBox(height: 4),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Diskon Promo',
                            style: TextStyle(
                                fontSize: 13, color: AppColors.secondary)),
                        Text('-${_currencyFormat.format(discount)}',
                            style: const TextStyle(
                                fontSize: 13,
                                fontWeight: FontWeight.w500,
                                color: AppColors.secondary)),
                      ],
                    ),
                  ],
                  const SizedBox(height: 8),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Total',
                          style: TextStyle(
                              fontSize: 16, fontWeight: FontWeight.w700)),
                      Text(_currencyFormat.format(total),
                          style: const TextStyle(
                              fontSize: 16,
                              fontWeight: FontWeight.w700,
                              color: AppColors.primary)),
                    ],
                  ),
                  const SizedBox(height: 12),
                  AppButton(
                    label: 'Bayar',
                    icon: Icons.payment,
                    onPressed: cartState.isEmpty ? null : onCheckout,
                    width: double.infinity,
                  ),
                ],
              ),
            );
          },
        );
      },
    );
  }
}
```

---

### Task 8: Kasir — Promo System

**Files:**
- Create: `mobile/lib/modules/kasir/repository/promo_repository.dart`
- Create: `mobile/lib/modules/kasir/bloc/promo_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/promo_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/promo_bloc.dart`
- Create: `mobile/lib/modules/kasir/widgets/promo_badge.dart`
- Create: `mobile/test/modules/kasir/bloc/promo_bloc_test.dart`

- [ ] **Step 1: Create PromoRepository**

```dart
// mobile/lib/modules/kasir/repository/promo_repository.dart
import '../../../core/api/api_client.dart';
import '../../../models/promo.dart';

class PromoRepository {
  final ApiClient _apiClient;

  PromoRepository({required ApiClient apiClient}) : _apiClient = apiClient;

  Future<List<Promo>> getActivePromos() async {
    final response = await _apiClient.get('/promos/active');

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data']
        : responseData;

    return (data as List)
        .map((e) => Promo.fromJson(e as Map<String, dynamic>))
        .toList();
  }
}
```

- [ ] **Step 2: Create PromoEvent**

```dart
// mobile/lib/modules/kasir/bloc/promo_event.dart
import 'package:equatable/equatable.dart';
import '../../../models/promo.dart';

abstract class PromoEvent extends Equatable {
  const PromoEvent();

  @override
  List<Object?> get props => [];
}

class PromoLoadRequested extends PromoEvent {}

class PromoSelected extends PromoEvent {
  final Promo promo;

  const PromoSelected({required this.promo});

  @override
  List<Object?> get props => [promo];
}

class PromoDeselected extends PromoEvent {}
```

- [ ] **Step 3: Create PromoState**

```dart
// mobile/lib/modules/kasir/bloc/promo_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/promo.dart';

class PromoState extends Equatable {
  final List<Promo> activePromos;
  final Promo? selectedPromo;
  final bool isLoading;
  final String? error;

  const PromoState({
    this.activePromos = const [],
    this.selectedPromo,
    this.isLoading = false,
    this.error,
  });

  PromoState copyWith({
    List<Promo>? activePromos,
    Promo? selectedPromo,
    bool? clearSelection,
    bool? isLoading,
    String? error,
  }) {
    return PromoState(
      activePromos: activePromos ?? this.activePromos,
      selectedPromo: clearSelection == true ? null : (selectedPromo ?? this.selectedPromo),
      isLoading: isLoading ?? this.isLoading,
      error: error,
    );
  }

  @override
  List<Object?> get props => [activePromos, selectedPromo, isLoading, error];
}
```

- [ ] **Step 4: Create PromoBloc**

```dart
// mobile/lib/modules/kasir/bloc/promo_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/error/app_exception.dart';
import '../repository/promo_repository.dart';
import 'promo_event.dart';
import 'promo_state.dart';

class PromoBloc extends Bloc<PromoEvent, PromoState> {
  final PromoRepository _promoRepository;

  PromoBloc({required PromoRepository promoRepository})
      : _promoRepository = promoRepository,
        super(const PromoState()) {
    on<PromoLoadRequested>(_onLoadPromos);
    on<PromoSelected>(_onPromoSelected);
    on<PromoDeselected>(_onPromoDeselected);
  }

  Future<void> _onLoadPromos(
    PromoLoadRequested event,
    Emitter<PromoState> emit,
  ) async {
    emit(state.copyWith(isLoading: true, error: null));

    try {
      final promos = await _promoRepository.getActivePromos();
      final validPromos = promos.where((p) => p.isValid).toList();
      emit(state.copyWith(activePromos: validPromos, isLoading: false));
    } on AppException catch (e) {
      emit(state.copyWith(isLoading: false, error: e.message));
    } catch (e) {
      emit(state.copyWith(isLoading: false, error: 'Failed to load promos'));
    }
  }

  void _onPromoSelected(PromoSelected event, Emitter<PromoState> emit) {
    emit(state.copyWith(selectedPromo: event.promo));
  }

  void _onPromoDeselected(PromoDeselected event, Emitter<PromoState> emit) {
    emit(state.copyWith(clearSelection: true));
  }
}
```

- [ ] **Step 5: Write PromoBloc test**

```dart
// mobile/test/modules/kasir/bloc/promo_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:pos_mobile/modules/kasir/bloc/promo_bloc.dart';
import 'package:pos_mobile/modules/kasir/bloc/promo_event.dart';
import 'package:pos_mobile/modules/kasir/bloc/promo_state.dart';
import 'package:pos_mobile/modules/kasir/repository/promo_repository.dart';
import 'package:pos_mobile/models/promo.dart';

class MockPromoRepository extends Mock implements PromoRepository {}

void main() {
  late PromoBloc promoBloc;
  late MockPromoRepository mockPromoRepository;

  final testPromos = [
    Promo(
      id: 'p1',
      name: 'Diskon 20%',
      type: 'PERCENT',
      value: 20,
      startDate: DateTime(2026, 1, 1),
      endDate: DateTime(2026, 12, 31),
      isActive: true,
    ),
    Promo(
      id: 'p2',
      name: 'Potongan 5K',
      type: 'FIXED',
      value: 5000,
      startDate: DateTime(2026, 1, 1),
      endDate: DateTime(2026, 12, 31),
      isActive: true,
    ),
  ];

  setUp(() {
    mockPromoRepository = MockPromoRepository();
    promoBloc = PromoBloc(promoRepository: mockPromoRepository);
  });

  tearDown(() {
    promoBloc.close();
  });

  group('PromoLoadRequested', () {
    blocTest<PromoBloc, PromoState>(
      'emits loading then loaded state with active promos',
      build: () {
        when(() => mockPromoRepository.getActivePromos())
            .thenAnswer((_) async => testPromos);
        return promoBloc;
      },
      act: (bloc) => bloc.add(PromoLoadRequested()),
      expect: () => [
        const PromoState(isLoading: true),
        PromoState(activePromos: testPromos, isLoading: false),
      ],
    );
  });

  group('PromoSelected', () {
    blocTest<PromoBloc, PromoState>(
      'selects a promo',
      build: () => promoBloc,
      seed: () => PromoState(activePromos: testPromos),
      act: (bloc) => bloc.add(PromoSelected(promo: testPromos[0])),
      expect: () => [
        PromoState(activePromos: testPromos, selectedPromo: testPromos[0]),
      ],
    );
  });

  group('PromoDeselected', () {
    blocTest<PromoBloc, PromoState>(
      'deselects the current promo',
      build: () => promoBloc,
      seed: () => PromoState(
        activePromos: testPromos,
        selectedPromo: testPromos[0],
      ),
      act: (bloc) => bloc.add(PromoDeselected()),
      expect: () => [
        PromoState(activePromos: testPromos),
      ],
    );
  });

  group('Promo.calculateDiscount', () {
    test('PERCENT promo calculates correctly', () {
      expect(testPromos[0].calculateDiscount(100000), 20000);
    });

    test('FIXED promo calculates correctly', () {
      expect(testPromos[1].calculateDiscount(100000), 5000);
    });

    test('FIXED promo caps at total amount', () {
      expect(testPromos[1].calculateDiscount(3000), 3000);
    });
  });
}
```

- [ ] **Step 6: Run PromoBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/promo_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7: Create PromoBadge widget**

```dart
// mobile/lib/modules/kasir/widgets/promo_badge.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';
import '../../../models/promo.dart';
import '../bloc/promo_bloc.dart';
import '../bloc/promo_event.dart';
import '../bloc/promo_state.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

class PromoBadge extends StatelessWidget {
  final PromoState promoState;
  final double cartSubtotal;

  const PromoBadge({
    super.key,
    required this.promoState,
    required this.cartSubtotal,
  });

  @override
  Widget build(BuildContext context) {
    if (promoState.activePromos.isEmpty) return const SizedBox.shrink();

    final selectedPromo = promoState.selectedPromo;

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: const BoxDecoration(
        border: Border(top: BorderSide(color: AppColors.divider)),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Row(
                children: [
                  Icon(Icons.local_offer, size: 16, color: AppColors.secondary),
                  SizedBox(width: 6),
                  Text(
                    'Promo Tersedia',
                    style: TextStyle(
                      fontSize: 12,
                      fontWeight: FontWeight.w600,
                      color: AppColors.secondary,
                    ),
                  ),
                ],
              ),
              if (selectedPromo != null)
                GestureDetector(
                  onTap: () {
                    context.read<PromoBloc>().add(PromoDeselected());
                  },
                  child: const Text(
                    'Hapus',
                    style: TextStyle(
                      fontSize: 11,
                      color: AppColors.error,
                      fontWeight: FontWeight.w500,
                    ),
                  ),
                ),
            ],
          ),
          const SizedBox(height: 6),
          SizedBox(
            height: 36,
            child: ListView.separated(
              scrollDirection: Axis.horizontal,
              itemCount: promoState.activePromos.length,
              separatorBuilder: (_, __) => const SizedBox(width: 8),
              itemBuilder: (context, index) {
                final promo = promoState.activePromos[index];
                final isSelected = selectedPromo?.id == promo.id;
                final discount = promo.calculateDiscount(cartSubtotal);

                return GestureDetector(
                  onTap: () {
                    if (isSelected) {
                      context.read<PromoBloc>().add(PromoDeselected());
                    } else {
                      context.read<PromoBloc>().add(PromoSelected(promo: promo));
                    }
                  },
                  child: Container(
                    padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 6),
                    decoration: BoxDecoration(
                      color: isSelected
                          ? AppColors.secondary.withOpacity(0.1)
                          : AppColors.surfaceVariant,
                      borderRadius: BorderRadius.circular(8),
                      border: Border.all(
                        color: isSelected ? AppColors.secondary : AppColors.border,
                        width: isSelected ? 2 : 1,
                      ),
                    ),
                    child: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(
                          promo.displayLabel,
                          style: TextStyle(
                            fontSize: 11,
                            fontWeight: FontWeight.w600,
                            color: isSelected
                                ? AppColors.secondary
                                : AppColors.textPrimary,
                          ),
                        ),
                        if (discount > 0) ...[
                          const SizedBox(width: 4),
                          Text(
                            '(-${_currencyFormat.format(discount)})',
                            style: const TextStyle(
                              fontSize: 10,
                              color: AppColors.secondary,
                              fontWeight: FontWeight.w500,
                            ),
                          ),
                        ],
                      ],
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### Task 9: Kasir — Checkout & Payment

**Files:**
- Create: `mobile/lib/modules/kasir/repository/transaction_repository.dart`
- Create: `mobile/lib/modules/kasir/bloc/checkout_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/checkout_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/checkout_bloc.dart`
- Create: `mobile/lib/modules/kasir/widgets/payment_method_selector.dart`
- Create: `mobile/lib/modules/kasir/screens/checkout_screen.dart`
- Create: `mobile/lib/modules/kasir/screens/receipt_screen.dart`
- Create: `mobile/test/modules/kasir/bloc/checkout_bloc_test.dart`

- [ ] **Step 1: Create TransactionRepository**

```dart
// mobile/lib/modules/kasir/repository/transaction_repository.dart
import '../../../core/api/api_client.dart';
import '../../../models/transaction.dart';
import '../../../models/cart_item.dart';

class TransactionRepository {
  final ApiClient _apiClient;

  TransactionRepository({required ApiClient apiClient}) : _apiClient = apiClient;

  Future<Transaction> createTransaction({
    required String paymentMethod,
    required List<CartItem> items,
    String? promoId,
  }) async {
    final response = await _apiClient.post(
      '/pos/transactions',
      data: {
        'paymentMethod': paymentMethod,
        'items': items.map((item) => item.toTransactionItem()).toList(),
        if (promoId != null) 'promoId': promoId,
      },
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data'] as Map<String, dynamic>
        : responseData as Map<String, dynamic>;

    return Transaction.fromJson(data);
  }

  Future<List<Transaction>> getTransactions({int page = 1, int limit = 20}) async {
    final response = await _apiClient.get(
      '/pos/transactions',
      queryParameters: {'page': page, 'limit': limit},
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data']
        : responseData;

    if (data is Map<String, dynamic> && data.containsKey('data')) {
      return (data['data'] as List)
          .map((e) => Transaction.fromJson(e as Map<String, dynamic>))
          .toList();
    }

    return (data as List)
        .map((e) => Transaction.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  Future<Transaction> getTransaction(String id) async {
    final response = await _apiClient.get('/pos/transactions/$id');

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data'] as Map<String, dynamic>
        : responseData as Map<String, dynamic>;

    return Transaction.fromJson(data);
  }
}
```

- [ ] **Step 2: Create CheckoutEvent**

```dart
// mobile/lib/modules/kasir/bloc/checkout_event.dart
import 'package:equatable/equatable.dart';
import '../../../models/cart_item.dart';

abstract class CheckoutEvent extends Equatable {
  const CheckoutEvent();

  @override
  List<Object?> get props => [];
}

class CheckoutPaymentMethodSelected extends CheckoutEvent {
  final String paymentMethod;

  const CheckoutPaymentMethodSelected({required this.paymentMethod});

  @override
  List<Object?> get props => [paymentMethod];
}

class CheckoutSubmitted extends CheckoutEvent {
  final List<CartItem> items;
  final String paymentMethod;
  final String? promoId;

  const CheckoutSubmitted({
    required this.items,
    required this.paymentMethod,
    this.promoId,
  });

  @override
  List<Object?> get props => [items, paymentMethod, promoId];
}

class CheckoutReset extends CheckoutEvent {}
```

- [ ] **Step 3: Create CheckoutState**

```dart
// mobile/lib/modules/kasir/bloc/checkout_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/transaction.dart';

abstract class CheckoutState extends Equatable {
  const CheckoutState();

  @override
  List<Object?> get props => [];
}

class CheckoutInitial extends CheckoutState {
  final String paymentMethod;

  const CheckoutInitial({this.paymentMethod = 'CASH'});

  @override
  List<Object?> get props => [paymentMethod];
}

class CheckoutProcessing extends CheckoutState {}

class CheckoutSuccess extends CheckoutState {
  final Transaction transaction;

  const CheckoutSuccess({required this.transaction});

  @override
  List<Object?> get props => [transaction];
}

class CheckoutFailure extends CheckoutState {
  final String message;

  const CheckoutFailure({required this.message});

  @override
  List<Object?> get props => [message];
}
```

- [ ] **Step 4: Create CheckoutBloc**

```dart
// mobile/lib/modules/kasir/bloc/checkout_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/error/app_exception.dart';
import '../repository/transaction_repository.dart';
import 'checkout_event.dart';
import 'checkout_state.dart';

class CheckoutBloc extends Bloc<CheckoutEvent, CheckoutState> {
  final TransactionRepository _transactionRepository;

  CheckoutBloc({required TransactionRepository transactionRepository})
      : _transactionRepository = transactionRepository,
        super(const CheckoutInitial()) {
    on<CheckoutPaymentMethodSelected>(_onPaymentMethodSelected);
    on<CheckoutSubmitted>(_onCheckoutSubmitted);
    on<CheckoutReset>(_onReset);
  }

  void _onPaymentMethodSelected(
    CheckoutPaymentMethodSelected event,
    Emitter<CheckoutState> emit,
  ) {
    emit(CheckoutInitial(paymentMethod: event.paymentMethod));
  }

  Future<void> _onCheckoutSubmitted(
    CheckoutSubmitted event,
    Emitter<CheckoutState> emit,
  ) async {
    emit(CheckoutProcessing());

    try {
      final transaction = await _transactionRepository.createTransaction(
        paymentMethod: event.paymentMethod,
        items: event.items,
        promoId: event.promoId,
      );

      emit(CheckoutSuccess(transaction: transaction));
    } on AppException catch (e) {
      emit(CheckoutFailure(message: e.message));
    } catch (e) {
      emit(const CheckoutFailure(message: 'Gagal memproses transaksi'));
    }
  }

  void _onReset(CheckoutReset event, Emitter<CheckoutState> emit) {
    emit(const CheckoutInitial());
  }
}
```

- [ ] **Step 5: Write CheckoutBloc test**

```dart
// mobile/test/modules/kasir/bloc/checkout_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:pos_mobile/modules/kasir/bloc/checkout_bloc.dart';
import 'package:pos_mobile/modules/kasir/bloc/checkout_event.dart';
import 'package:pos_mobile/modules/kasir/bloc/checkout_state.dart';
import 'package:pos_mobile/modules/kasir/repository/transaction_repository.dart';
import 'package:pos_mobile/models/cart_item.dart';
import 'package:pos_mobile/models/transaction.dart';
import 'package:pos_mobile/core/error/app_exception.dart';

class MockTransactionRepository extends Mock implements TransactionRepository {}

void main() {
  late CheckoutBloc checkoutBloc;
  late MockTransactionRepository mockTransactionRepository;

  const testItems = [
    CartItem(
      itemId: 'm1',
      name: 'Nasi Goreng',
      type: CartItemType.menu,
      unitPrice: 25000,
      quantity: 2,
    ),
  ];

  final testTransaction = Transaction(
    id: 't1',
    transactionCode: 'TRX-20260418-0001',
    paymentMethod: 'CASH',
    totalAmount: 50000,
    totalHpp: 20000,
    totalProfit: 30000,
    createdAt: DateTime(2026, 4, 18),
  );

  setUp(() {
    mockTransactionRepository = MockTransactionRepository();
    checkoutBloc = CheckoutBloc(
      transactionRepository: mockTransactionRepository,
    );
  });

  tearDown(() {
    checkoutBloc.close();
  });

  group('CheckoutPaymentMethodSelected', () {
    blocTest<CheckoutBloc, CheckoutState>(
      'updates payment method',
      build: () => checkoutBloc,
      act: (bloc) => bloc.add(
        const CheckoutPaymentMethodSelected(paymentMethod: 'QRIS'),
      ),
      expect: () => [const CheckoutInitial(paymentMethod: 'QRIS')],
    );
  });

  group('CheckoutSubmitted', () {
    blocTest<CheckoutBloc, CheckoutState>(
      'emits [Processing, Success] on successful transaction',
      build: () {
        when(() => mockTransactionRepository.createTransaction(
              paymentMethod: 'CASH',
              items: testItems,
              promoId: null,
            )).thenAnswer((_) async => testTransaction);
        return checkoutBloc;
      },
      act: (bloc) => bloc.add(const CheckoutSubmitted(
        items: testItems,
        paymentMethod: 'CASH',
      )),
      expect: () => [
        CheckoutProcessing(),
        CheckoutSuccess(transaction: testTransaction),
      ],
    );

    blocTest<CheckoutBloc, CheckoutState>(
      'emits [Processing, Failure] on failed transaction',
      build: () {
        when(() => mockTransactionRepository.createTransaction(
              paymentMethod: 'CASH',
              items: testItems,
              promoId: null,
            )).thenThrow(AppException(message: 'Server error', statusCode: 500));
        return checkoutBloc;
      },
      act: (bloc) => bloc.add(const CheckoutSubmitted(
        items: testItems,
        paymentMethod: 'CASH',
      )),
      expect: () => [
        CheckoutProcessing(),
        const CheckoutFailure(message: 'Server error'),
      ],
    );
  });
}
```

- [ ] **Step 6: Run CheckoutBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/checkout_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7: Create PaymentMethodSelector widget**

```dart
// mobile/lib/modules/kasir/widgets/payment_method_selector.dart
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';

class PaymentMethodSelector extends StatelessWidget {
  final String selectedMethod;
  final ValueChanged<String> onMethodSelected;

  const PaymentMethodSelector({
    super.key,
    required this.selectedMethod,
    required this.onMethodSelected,
  });

  static const _methods = [
    {'key': 'CASH', 'label': 'Tunai', 'icon': Icons.money},
    {'key': 'QRIS', 'label': 'QRIS', 'icon': Icons.qr_code},
    {'key': 'EDC', 'label': 'EDC', 'icon': Icons.credit_card},
  ];

  @override
  Widget build(BuildContext context) {
    return Row(
      children: _methods.map((method) {
        final isSelected = selectedMethod == method['key'];
        return Expanded(
          child: Padding(
            padding: EdgeInsets.only(
              right: method != _methods.last ? 12 : 0,
            ),
            child: Material(
              color: isSelected
                  ? AppColors.primary.withOpacity(0.1)
                  : AppColors.surfaceVariant,
              borderRadius: BorderRadius.circular(12),
              child: InkWell(
                onTap: () => onMethodSelected(method['key'] as String),
                borderRadius: BorderRadius.circular(12),
                child: Container(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(12),
                    border: Border.all(
                      color: isSelected ? AppColors.primary : AppColors.border,
                      width: isSelected ? 2 : 1,
                    ),
                  ),
                  child: Column(
                    children: [
                      Icon(
                        method['icon'] as IconData,
                        size: 32,
                        color: isSelected
                            ? AppColors.primary
                            : AppColors.textSecondary,
                      ),
                      const SizedBox(height: 8),
                      Text(
                        method['label'] as String,
                        style: TextStyle(
                          fontSize: 14,
                          fontWeight: FontWeight.w600,
                          color: isSelected
                              ? AppColors.primary
                              : AppColors.textSecondary,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
        );
      }).toList(),
    );
  }
}
```

- [ ] **Step 8: Create CheckoutScreen**

```dart
// mobile/lib/modules/kasir/screens/checkout_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/app_button.dart';
import '../../../shared/widgets/loading_overlay.dart';
import '../bloc/cart_bloc.dart';
import '../bloc/cart_event.dart';
import '../bloc/cart_state.dart';
import '../bloc/promo_bloc.dart';
import '../bloc/promo_event.dart';
import '../bloc/promo_state.dart';
import '../bloc/checkout_bloc.dart';
import '../bloc/checkout_event.dart';
import '../bloc/checkout_state.dart';
import '../widgets/payment_method_selector.dart';
import 'receipt_screen.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

class CheckoutScreen extends StatelessWidget {
  const CheckoutScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocListener<CheckoutBloc, CheckoutState>(
      listener: (context, state) {
        if (state is CheckoutSuccess) {
          context.read<CartBloc>().add(CartCleared());
          context.read<PromoBloc>().add(PromoDeselected());

          Navigator.pushReplacement(
            context,
            MaterialPageRoute(
              builder: (_) => ReceiptScreen(transaction: state.transaction),
            ),
          );
        } else if (state is CheckoutFailure) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(state.message),
              backgroundColor: AppColors.error,
            ),
          );
        }
      },
      child: BlocBuilder<CheckoutBloc, CheckoutState>(
        builder: (context, checkoutState) {
          final isProcessing = checkoutState is CheckoutProcessing;
          final paymentMethod = checkoutState is CheckoutInitial
              ? checkoutState.paymentMethod
              : 'CASH';

          return LoadingOverlay(
            isLoading: isProcessing,
            message: 'Memproses transaksi...',
            child: Scaffold(
              appBar: AppBar(
                title: const Text('Checkout'),
                leading: IconButton(
                  icon: const Icon(Icons.arrow_back),
                  onPressed: () => Navigator.pop(context),
                ),
              ),
              body: Padding(
                padding: const EdgeInsets.all(24),
                child: Row(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Expanded(
                      flex: 3,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          const Text(
                            'Metode Pembayaran',
                            style: TextStyle(
                              fontSize: 18,
                              fontWeight: FontWeight.w600,
                            ),
                          ),
                          const SizedBox(height: 16),
                          PaymentMethodSelector(
                            selectedMethod: paymentMethod,
                            onMethodSelected: (method) {
                              context.read<CheckoutBloc>().add(
                                    CheckoutPaymentMethodSelected(
                                        paymentMethod: method),
                                  );
                            },
                          ),
                        ],
                      ),
                    ),
                    const SizedBox(width: 24),
                    Expanded(
                      flex: 2,
                      child: _buildOrderSummary(context, paymentMethod),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }

  Widget _buildOrderSummary(BuildContext context, String paymentMethod) {
    return BlocBuilder<CartBloc, CartState>(
      builder: (context, cartState) {
        return BlocBuilder<PromoBloc, PromoState>(
          builder: (context, promoState) {
            final subtotal = cartState.subtotal;
            final discount =
                promoState.selectedPromo?.calculateDiscount(subtotal) ?? 0;
            final total = subtotal - discount;

            return Card(
              child: Padding(
                padding: const EdgeInsets.all(20),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Ringkasan Pesanan',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                    const SizedBox(height: 16),
                    ...cartState.items.map((item) => Padding(
                          padding: const EdgeInsets.only(bottom: 8),
                          child: Row(
                            mainAxisAlignment: MainAxisAlignment.spaceBetween,
                            children: [
                              Expanded(
                                child: Text(
                                  '${item.name} x${item.quantity}',
                                  style: const TextStyle(fontSize: 13),
                                ),
                              ),
                              Text(
                                _currencyFormat.format(item.subtotal),
                                style: const TextStyle(fontSize: 13),
                              ),
                            ],
                          ),
                        )),
                    const Divider(height: 24),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Subtotal',
                            style: TextStyle(fontSize: 13)),
                        Text(_currencyFormat.format(subtotal),
                            style: const TextStyle(fontSize: 13)),
                      ],
                    ),
                    if (discount > 0) ...[
                      const SizedBox(height: 4),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            'Promo: ${promoState.selectedPromo?.name ?? ""}',
                            style: const TextStyle(
                                fontSize: 13, color: AppColors.secondary),
                          ),
                          Text(
                            '-${_currencyFormat.format(discount)}',
                            style: const TextStyle(
                                fontSize: 13, color: AppColors.secondary),
                          ),
                        ],
                      ),
                    ],
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Total',
                            style: TextStyle(
                                fontSize: 18, fontWeight: FontWeight.w700)),
                        Text(
                          _currencyFormat.format(total),
                          style: const TextStyle(
                            fontSize: 18,
                            fontWeight: FontWeight.w700,
                            color: AppColors.primary,
                          ),
                        ),
                      ],
                    ),
                    const Spacer(),
                    AppButton(
                      label: 'Proses Pembayaran',
                      icon: Icons.check_circle,
                      width: double.infinity,
                      onPressed: () {
                        context.read<CheckoutBloc>().add(CheckoutSubmitted(
                              items: cartState.items,
                              paymentMethod: paymentMethod,
                              promoId: promoState.selectedPromo?.id,
                            ));
                      },
                    ),
                  ],
                ),
              ),
            );
          },
        );
      },
    );
  }
}
```

- [ ] **Step 9: Create ReceiptScreen**

```dart
// mobile/lib/modules/kasir/screens/receipt_screen.dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/app_button.dart';
import '../../../models/transaction.dart';

final _currencyFormat = NumberFormat.currency(
  locale: 'id_ID',
  symbol: 'Rp ',
  decimalDigits: 0,
);

final _dateFormat = DateFormat('dd MMM yyyy, HH:mm');

class ReceiptScreen extends StatelessWidget {
  final Transaction transaction;

  const ReceiptScreen({super.key, required this.transaction});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColors.background,
      body: Center(
        child: Container(
          constraints: const BoxConstraints(maxWidth: 400),
          margin: const EdgeInsets.all(24),
          child: Card(
            child: Padding(
              padding: const EdgeInsets.all(32),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Icon(
                    Icons.check_circle,
                    color: AppColors.secondary,
                    size: 64,
                  ),
                  const SizedBox(height: 16),
                  const Text(
                    'Transaksi Berhasil!',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.w700,
                    ),
                  ),
                  const SizedBox(height: 24),
                  _buildReceiptLine(
                    'Kode Transaksi',
                    transaction.transactionCode,
                  ),
                  _buildReceiptLine(
                    'Tanggal',
                    _dateFormat.format(transaction.createdAt),
                  ),
                  _buildReceiptLine(
                    'Kasir',
                    transaction.cashier?.name ?? '-',
                  ),
                  _buildReceiptLine(
                    'Pembayaran',
                    transaction.paymentMethod,
                  ),
                  const Divider(height: 24),
                  ...transaction.items.map((item) => Padding(
                        padding: const EdgeInsets.only(bottom: 4),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Expanded(
                              child: Text(
                                '${item.itemType == "MENU" ? "[M]" : "[R]"} x${item.qty}',
                                style: const TextStyle(fontSize: 13),
                              ),
                            ),
                            Text(
                              _currencyFormat.format(item.subtotal),
                              style: const TextStyle(fontSize: 13),
                            ),
                          ],
                        ),
                      )),
                  const Divider(height: 24),
                  if (transaction.promoDiscount > 0)
                    _buildReceiptLine(
                      'Diskon Promo',
                      '-${_currencyFormat.format(transaction.promoDiscount)}',
                      valueColor: AppColors.secondary,
                    ),
                  _buildReceiptLine(
                    'Total',
                    _currencyFormat.format(transaction.totalAmount),
                    isBold: true,
                    valueColor: AppColors.primary,
                  ),
                  const SizedBox(height: 24),
                  Row(
                    children: [
                      Expanded(
                        child: AppButton(
                          label: 'Cetak Struk',
                          icon: Icons.print,
                          isOutlined: true,
                          onPressed: () {
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(
                                content: Text('Mencetak struk...'),
                              ),
                            );
                          },
                        ),
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: AppButton(
                          label: 'Transaksi Baru',
                          icon: Icons.add,
                          onPressed: () {
                            Navigator.popUntil(context, (route) => route.isFirst);
                          },
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  Widget _buildReceiptLine(
    String label,
    String value, {
    bool isBold = false,
    Color? valueColor,
  }) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 6),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: TextStyle(
              fontSize: isBold ? 15 : 13,
              fontWeight: isBold ? FontWeight.w700 : FontWeight.w400,
              color: AppColors.textSecondary,
            ),
          ),
          Text(
            value,
            style: TextStyle(
              fontSize: isBold ? 15 : 13,
              fontWeight: isBold ? FontWeight.w700 : FontWeight.w500,
              color: valueColor ?? AppColors.textPrimary,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### Task 10: Kasir — Order Status Tracking (Real-time via WebSocket)

**Files:**
- Create: `mobile/lib/modules/kasir/bloc/order_tracking_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/order_tracking_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/order_tracking_bloc.dart`
- Create: `mobile/lib/modules/kasir/screens/order_tracking_screen.dart`

- [ ] **Step 1: Create OrderTrackingEvent**

```dart
// mobile/lib/modules/kasir/bloc/order_tracking_event.dart
import 'package:equatable/equatable.dart';
import '../../../models/order.dart';

abstract class OrderTrackingEvent extends Equatable {
  const OrderTrackingEvent();

  @override
  List<Object?> get props => [];
}

class OrderTrackingStarted extends OrderTrackingEvent {}

class OrderTrackingOrderReceived extends OrderTrackingEvent {
  final Order order;

  const OrderTrackingOrderReceived({required this.order});

  @override
  List<Object?> get props => [order];
}

class OrderTrackingStatusUpdated extends OrderTrackingEvent {
  final Order updatedOrder;

  const OrderTrackingStatusUpdated({required this.updatedOrder});

  @override
  List<Object?> get props => [updatedOrder];
}

class OrderTrackingServed extends OrderTrackingEvent {
  final String orderId;

  const OrderTrackingServed({required this.orderId});

  @override
  List<Object?> get props => [orderId];
}
```

- [ ] **Step 2: Create OrderTrackingState**

```dart
// mobile/lib/modules/kasir/bloc/order_tracking_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/order.dart';

class OrderTrackingState extends Equatable {
  final List<Order> activeOrders;
  final bool isConnected;

  const OrderTrackingState({
    this.activeOrders = const [],
    this.isConnected = false,
  });

  OrderTrackingState copyWith({
    List<Order>? activeOrders,
    bool? isConnected,
  }) {
    return OrderTrackingState(
      activeOrders: activeOrders ?? this.activeOrders,
      isConnected: isConnected ?? this.isConnected,
    );
  }

  @override
  List<Object?> get props => [activeOrders, isConnected];
}
```

- [ ] **Step 3: Create OrderTrackingBloc**

```dart
// mobile/lib/modules/kasir/bloc/order_tracking_bloc.dart
import 'dart:async';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/api/api_client.dart';
import '../../../core/socket/socket_service.dart';
import '../../../models/order.dart';
import 'order_tracking_event.dart';
import 'order_tracking_state.dart';

class OrderTrackingBloc extends Bloc<OrderTrackingEvent, OrderTrackingState> {
  final SocketService _socketService;
  final ApiClient _apiClient;
  StreamSubscription? _statusSub;
  StreamSubscription? _newOrderSub;
  StreamSubscription? _connectionSub;

  OrderTrackingBloc({
    required SocketService socketService,
    required ApiClient apiClient,
  })  : _socketService = socketService,
        _apiClient = apiClient,
        super(const OrderTrackingState()) {
    on<OrderTrackingStarted>(_onStarted);
    on<OrderTrackingOrderReceived>(_onOrderReceived);
    on<OrderTrackingStatusUpdated>(_onStatusUpdated);
    on<OrderTrackingServed>(_onServed);
  }

  Future<void> _onStarted(
    OrderTrackingStarted event,
    Emitter<OrderTrackingState> emit,
  ) async {
    await _socketService.connect();

    // Load active orders from API
    try {
      final response = await _apiClient.get('/orders/active');
      final responseData = response.data;
      final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
          ? responseData['data']
          : responseData;

      final orders = (data as List)
          .map((e) => Order.fromJson(e as Map<String, dynamic>))
          .toList();

      emit(state.copyWith(activeOrders: orders, isConnected: true));
    } catch (e) {
      emit(state.copyWith(isConnected: true));
    }

    // Listen for new orders
    _newOrderSub = _socketService.on<dynamic>('order:new').listen((data) {
      if (data is Map<String, dynamic>) {
        final order = Order.fromJson(data);
        add(OrderTrackingOrderReceived(order: order));
      }
    });

    // Listen for status updates
    _statusSub = _socketService.on<dynamic>('order:status').listen((data) {
      if (data is Map<String, dynamic>) {
        final order = Order.fromJson(data);
        add(OrderTrackingStatusUpdated(updatedOrder: order));
      }
    });

    // Listen for connection changes
    _connectionSub = _socketService.connectionStream.listen((connected) {
      if (!isClosed) {
        emit(state.copyWith(isConnected: connected));
      }
    });
  }

  void _onOrderReceived(
    OrderTrackingOrderReceived event,
    Emitter<OrderTrackingState> emit,
  ) {
    final exists = state.activeOrders.any((o) => o.id == event.order.id);
    if (!exists) {
      emit(state.copyWith(
        activeOrders: [...state.activeOrders, event.order],
      ));
    }
  }

  void _onStatusUpdated(
    OrderTrackingStatusUpdated event,
    Emitter<OrderTrackingState> emit,
  ) {
    final updatedOrders = state.activeOrders.map((order) {
      if (order.id == event.updatedOrder.id) {
        return event.updatedOrder;
      }
      return order;
    }).toList();

    // Remove served orders
    updatedOrders.removeWhere((o) => o.status == 'SERVED');

    emit(state.copyWith(activeOrders: updatedOrders));
  }

  void _onServed(
    OrderTrackingServed event,
    Emitter<OrderTrackingState> emit,
  ) {
    _socketService.emit('order:update_status', {
      'orderId': event.orderId,
      'status': 'SERVED',
    });

    final updatedOrders = state.activeOrders
        .where((o) => o.id != event.orderId)
        .toList();
    emit(state.copyWith(activeOrders: updatedOrders));
  }

  @override
  Future<void> close() {
    _statusSub?.cancel();
    _newOrderSub?.cancel();
    _connectionSub?.cancel();
    return super.close();
  }
}
```

- [ ] **Step 4: Create OrderTrackingScreen**

```dart
// mobile/lib/modules/kasir/screens/order_tracking_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/status_badge.dart';
import '../../../shared/widgets/app_button.dart';
import '../../../models/order.dart';
import '../bloc/order_tracking_bloc.dart';
import '../bloc/order_tracking_event.dart';
import '../bloc/order_tracking_state.dart';

class OrderTrackingScreen extends StatelessWidget {
  const OrderTrackingScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Pesanan Aktif'),
        actions: [
          BlocBuilder<OrderTrackingBloc, OrderTrackingState>(
            builder: (context, state) {
              return Padding(
                padding: const EdgeInsets.only(right: 16),
                child: Row(
                  children: [
                    Container(
                      width: 8,
                      height: 8,
                      decoration: BoxDecoration(
                        shape: BoxShape.circle,
                        color: state.isConnected
                            ? AppColors.secondary
                            : AppColors.error,
                      ),
                    ),
                    const SizedBox(width: 6),
                    Text(
                      state.isConnected ? 'Terhubung' : 'Terputus',
                      style: TextStyle(
                        fontSize: 12,
                        color: state.isConnected
                            ? AppColors.secondary
                            : AppColors.error,
                      ),
                    ),
                  ],
                ),
              );
            },
          ),
        ],
      ),
      body: BlocBuilder<OrderTrackingBloc, OrderTrackingState>(
        builder: (context, state) {
          if (state.activeOrders.isEmpty) {
            return const Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Icon(Icons.receipt_long, size: 64, color: AppColors.textLight),
                  SizedBox(height: 16),
                  Text(
                    'Belum ada pesanan aktif',
                    style: TextStyle(
                      fontSize: 16,
                      color: AppColors.textSecondary,
                    ),
                  ),
                ],
              ),
            );
          }

          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: state.activeOrders.length,
            itemBuilder: (context, index) {
              final order = state.activeOrders[index];
              return _OrderTrackingCard(order: order);
            },
          );
        },
      ),
    );
  }
}

class _OrderTrackingCard extends StatelessWidget {
  final Order order;

  const _OrderTrackingCard({required this.order});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Container(
              width: 60,
              height: 60,
              decoration: BoxDecoration(
                color: AppColors.statusBackground(order.status),
                borderRadius: BorderRadius.circular(12),
              ),
              child: Center(
                child: Text(
                  '#${order.orderNumber}',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.w700,
                    color: AppColors.statusColor(order.status),
                  ),
                ),
              ),
            ),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        'Order #${order.orderNumber}',
                        style: const TextStyle(
                          fontSize: 15,
                          fontWeight: FontWeight.w600,
                        ),
                      ),
                      StatusBadge(status: order.status),
                    ],
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '${order.items.length} item - ${_formatDuration(order.elapsedTime)}',
                    style: const TextStyle(
                      fontSize: 13,
                      color: AppColors.textSecondary,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    order.items.map((i) => i.menuItemName).join(', '),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                    style: const TextStyle(
                      fontSize: 12,
                      color: AppColors.textLight,
                    ),
                  ),
                ],
              ),
            ),
            if (order.status == 'READY')
              Padding(
                padding: const EdgeInsets.only(left: 12),
                child: AppButton(
                  label: 'Served',
                  color: AppColors.secondary,
                  onPressed: () {
                    context.read<OrderTrackingBloc>().add(
                          OrderTrackingServed(orderId: order.id),
                        );
                  },
                ),
              ),
          ],
        ),
      ),
    );
  }

  String _formatDuration(Duration d) {
    final minutes = d.inMinutes;
    if (minutes < 1) return 'Baru saja';
    if (minutes < 60) return '${minutes}m lalu';
    return '${d.inHours}h ${minutes % 60}m lalu';
  }
}
```

---

### Task 11: Kasir Main Screen (Wiring Everything Together)

**Files:**
- Create: `mobile/lib/modules/kasir/screens/kasir_main_screen.dart`

- [ ] **Step 1: Create KasirMainScreen**

```dart
// mobile/lib/modules/kasir/screens/kasir_main_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../config/app_theme.dart';
import '../../../core/api/api_client.dart';
import '../../../core/socket/socket_service.dart';
import '../../auth/bloc/auth_bloc.dart';
import '../../auth/bloc/auth_event.dart';
import '../bloc/menu_bloc.dart';
import '../bloc/menu_event.dart';
import '../bloc/cart_bloc.dart';
import '../bloc/promo_bloc.dart';
import '../bloc/promo_event.dart';
import '../bloc/checkout_bloc.dart';
import '../bloc/order_tracking_bloc.dart';
import '../bloc/order_tracking_event.dart';
import '../repository/menu_repository.dart';
import '../repository/promo_repository.dart';
import '../repository/transaction_repository.dart';
import '../widgets/menu_grid.dart';
import '../widgets/cart_panel.dart';
import 'checkout_screen.dart';
import 'order_tracking_screen.dart';

class KasirMainScreen extends StatefulWidget {
  const KasirMainScreen({super.key});

  @override
  State<KasirMainScreen> createState() => _KasirMainScreenState();
}

class _KasirMainScreenState extends State<KasirMainScreen> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    final apiClient = context.read<ApiClient>();
    final socketService = context.read<SocketService>();

    final menuRepository = MenuRepository(apiClient: apiClient);
    final promoRepository = PromoRepository(apiClient: apiClient);
    final transactionRepository = TransactionRepository(apiClient: apiClient);

    return MultiBlocProvider(
      providers: [
        BlocProvider(
          create: (_) => MenuBloc(menuRepository: menuRepository)
            ..add(MenuLoadRequested()),
        ),
        BlocProvider(create: (_) => CartBloc()),
        BlocProvider(
          create: (_) => PromoBloc(promoRepository: promoRepository)
            ..add(PromoLoadRequested()),
        ),
        BlocProvider(
          create: (_) => CheckoutBloc(
            transactionRepository: transactionRepository,
          ),
        ),
        BlocProvider(
          create: (_) => OrderTrackingBloc(
            socketService: socketService,
            apiClient: apiClient,
          )..add(OrderTrackingStarted()),
        ),
      ],
      child: Scaffold(
        body: Row(
          children: [
            _buildSideNav(context),
            Expanded(
              child: IndexedStack(
                index: _currentIndex,
                children: [
                  _buildKasirView(),
                  const OrderTrackingScreen(),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSideNav(BuildContext context) {
    return Container(
      width: 72,
      decoration: BoxDecoration(
        color: AppColors.surface,
        border: Border(
          right: BorderSide(color: AppColors.border),
        ),
      ),
      child: Column(
        children: [
          const SizedBox(height: 16),
          const Icon(Icons.point_of_sale, color: AppColors.primary, size: 28),
          const SizedBox(height: 24),
          _NavItem(
            icon: Icons.restaurant_menu,
            label: 'Kasir',
            isActive: _currentIndex == 0,
            onTap: () => setState(() => _currentIndex = 0),
          ),
          _NavItem(
            icon: Icons.receipt_long,
            label: 'Order',
            isActive: _currentIndex == 1,
            onTap: () => setState(() => _currentIndex = 1),
          ),
          const Spacer(),
          _NavItem(
            icon: Icons.logout,
            label: 'Logout',
            isActive: false,
            onTap: () {
              context.read<AuthBloc>().add(AuthLogoutRequested());
            },
          ),
          const SizedBox(height: 16),
        ],
      ),
    );
  }

  Widget _buildKasirView() {
    return Builder(
      builder: (context) {
        return Row(
          children: [
            Expanded(
              child: MenuGrid(),
            ),
            CartPanel(
              onCheckout: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => MultiBlocProvider(
                      providers: [
                        BlocProvider.value(
                          value: context.read<CartBloc>(),
                        ),
                        BlocProvider.value(
                          value: context.read<PromoBloc>(),
                        ),
                        BlocProvider.value(
                          value: context.read<CheckoutBloc>(),
                        ),
                      ],
                      child: const CheckoutScreen(),
                    ),
                  ),
                );
              },
            ),
          ],
        );
      },
    );
  }
}

class _NavItem extends StatelessWidget {
  final IconData icon;
  final String label;
  final bool isActive;
  final VoidCallback onTap;

  const _NavItem({
    required this.icon,
    required this.label,
    required this.isActive,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: InkWell(
        onTap: onTap,
        child: Container(
          width: 56,
          padding: const EdgeInsets.symmetric(vertical: 8),
          decoration: BoxDecoration(
            color: isActive
                ? AppColors.primary.withOpacity(0.1)
                : Colors.transparent,
            borderRadius: BorderRadius.circular(12),
          ),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(
                icon,
                size: 22,
                color: isActive ? AppColors.primary : AppColors.textLight,
              ),
              const SizedBox(height: 4),
              Text(
                label,
                style: TextStyle(
                  fontSize: 10,
                  fontWeight: FontWeight.w500,
                  color: isActive ? AppColors.primary : AppColors.textLight,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### Task 12: Kitchen Display — Order Board (WebSocket + Color-Coded Cards)

**Files:**
- Create: `mobile/lib/modules/kitchen/repository/kitchen_repository.dart`
- Create: `mobile/lib/modules/kitchen/bloc/kitchen_event.dart`
- Create: `mobile/lib/modules/kitchen/bloc/kitchen_state.dart`
- Create: `mobile/lib/modules/kitchen/bloc/kitchen_bloc.dart`
- Create: `mobile/lib/modules/kitchen/widgets/order_timer.dart`
- Create: `mobile/lib/modules/kitchen/widgets/order_item_tile.dart`
- Create: `mobile/lib/modules/kitchen/widgets/order_card.dart`
- Create: `mobile/lib/modules/kitchen/screens/kitchen_display_screen.dart`
- Create: `mobile/test/modules/kitchen/bloc/kitchen_bloc_test.dart`

- [ ] **Step 1: Create KitchenRepository**

```dart
// mobile/lib/modules/kitchen/repository/kitchen_repository.dart
import '../../../core/api/api_client.dart';
import '../../../models/order.dart';

class KitchenRepository {
  final ApiClient _apiClient;

  KitchenRepository({required ApiClient apiClient}) : _apiClient = apiClient;

  Future<List<Order>> getActiveOrders() async {
    final response = await _apiClient.get('/orders/active');

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data']
        : responseData;

    return (data as List)
        .map((e) => Order.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  Future<Order> updateOrderStatus(String orderId, String status) async {
    final response = await _apiClient.patch(
      '/orders/$orderId/status',
      data: {'status': status},
    );

    final responseData = response.data;
    final data = responseData is Map<String, dynamic> && responseData.containsKey('data')
        ? responseData['data'] as Map<String, dynamic>
        : responseData as Map<String, dynamic>;

    return Order.fromJson(data);
  }

  Future<void> updateOrderItemStatus(String itemId, String status) async {
    await _apiClient.patch(
      '/orders/items/$itemId/status',
      data: {'status': status},
    );
  }
}
```

- [ ] **Step 2: Create KitchenEvent**

```dart
// mobile/lib/modules/kitchen/bloc/kitchen_event.dart
import 'package:equatable/equatable.dart';
import '../../../models/order.dart';

abstract class KitchenEvent extends Equatable {
  const KitchenEvent();

  @override
  List<Object?> get props => [];
}

class KitchenStarted extends KitchenEvent {}

class KitchenNewOrderReceived extends KitchenEvent {
  final Order order;

  const KitchenNewOrderReceived({required this.order});

  @override
  List<Object?> get props => [order];
}

class KitchenOrderStatusUpdated extends KitchenEvent {
  final String orderId;
  final String status;

  const KitchenOrderStatusUpdated({
    required this.orderId,
    required this.status,
  });

  @override
  List<Object?> get props => [orderId, status];
}

class KitchenItemStatusUpdated extends KitchenEvent {
  final String orderId;
  final String itemId;
  final String status;

  const KitchenItemStatusUpdated({
    required this.orderId,
    required this.itemId,
    required this.status,
  });

  @override
  List<Object?> get props => [orderId, itemId, status];
}

class KitchenExternalStatusUpdate extends KitchenEvent {
  final Order updatedOrder;

  const KitchenExternalStatusUpdate({required this.updatedOrder});

  @override
  List<Object?> get props => [updatedOrder];
}
```

- [ ] **Step 3: Create KitchenState**

```dart
// mobile/lib/modules/kitchen/bloc/kitchen_state.dart
import 'package:equatable/equatable.dart';
import '../../../models/order.dart';

class KitchenState extends Equatable {
  final List<Order> orders;
  final bool isLoading;
  final bool isConnected;
  final String? error;

  const KitchenState({
    this.orders = const [],
    this.isLoading = false,
    this.isConnected = false,
    this.error,
  });

  List<Order> get pendingOrders =>
      orders.where((o) => o.status == 'PENDING').toList();

  List<Order> get preparingOrders =>
      orders.where((o) => o.status == 'PREPARING').toList();

  List<Order> get readyOrders =>
      orders.where((o) => o.status == 'READY').toList();

  KitchenState copyWith({
    List<Order>? orders,
    bool? isLoading,
    bool? isConnected,
    String? error,
  }) {
    return KitchenState(
      orders: orders ?? this.orders,
      isLoading: isLoading ?? this.isLoading,
      isConnected: isConnected ?? this.isConnected,
      error: error,
    );
  }

  @override
  List<Object?> get props => [orders, isLoading, isConnected, error];
}
```

- [ ] **Step 4: Create KitchenBloc**

```dart
// mobile/lib/modules/kitchen/bloc/kitchen_bloc.dart
import 'dart:async';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../core/socket/socket_service.dart';
import '../../../models/order.dart';
import '../../../models/order_item.dart';
import '../repository/kitchen_repository.dart';
import 'kitchen_event.dart';
import 'kitchen_state.dart';

class KitchenBloc extends Bloc<KitchenEvent, KitchenState> {
  final KitchenRepository _kitchenRepository;
  final SocketService _socketService;
  StreamSubscription? _newOrderSub;
  StreamSubscription? _statusSub;
  StreamSubscription? _itemDoneSub;
  StreamSubscription? _connectionSub;

  KitchenBloc({
    required KitchenRepository kitchenRepository,
    required SocketService socketService,
  })  : _kitchenRepository = kitchenRepository,
        _socketService = socketService,
        super(const KitchenState()) {
    on<KitchenStarted>(_onStarted);
    on<KitchenNewOrderReceived>(_onNewOrderReceived);
    on<KitchenOrderStatusUpdated>(_onOrderStatusUpdated);
    on<KitchenItemStatusUpdated>(_onItemStatusUpdated);
    on<KitchenExternalStatusUpdate>(_onExternalStatusUpdate);
  }

  Future<void> _onStarted(
    KitchenStarted event,
    Emitter<KitchenState> emit,
  ) async {
    emit(state.copyWith(isLoading: true));

    // Connect to WebSocket
    await _socketService.connect();

    // Load active orders from API
    try {
      final orders = await _kitchenRepository.getActiveOrders();
      emit(state.copyWith(
        orders: orders,
        isLoading: false,
        isConnected: true,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Failed to load orders',
        isConnected: _socketService.isConnected,
      ));
    }

    // Listen for new orders via WebSocket
    _newOrderSub = _socketService.on<dynamic>('order:new').listen((data) {
      if (data is Map<String, dynamic>) {
        add(KitchenNewOrderReceived(order: Order.fromJson(data)));
      }
    });

    // Listen for status updates from kasir (e.g., SERVED)
    _statusSub = _socketService.on<dynamic>('order:status').listen((data) {
      if (data is Map<String, dynamic>) {
        add(KitchenExternalStatusUpdate(updatedOrder: Order.fromJson(data)));
      }
    });

    // Listen for connection changes
    _connectionSub = _socketService.connectionStream.listen((connected) {
      if (!isClosed) {
        emit(state.copyWith(isConnected: connected));
      }
    });
  }

  void _onNewOrderReceived(
    KitchenNewOrderReceived event,
    Emitter<KitchenState> emit,
  ) {
    final exists = state.orders.any((o) => o.id == event.order.id);
    if (!exists) {
      emit(state.copyWith(orders: [...state.orders, event.order]));
    }
  }

  Future<void> _onOrderStatusUpdated(
    KitchenOrderStatusUpdated event,
    Emitter<KitchenState> emit,
  ) async {
    try {
      await _kitchenRepository.updateOrderStatus(event.orderId, event.status);

      // Also emit via WebSocket for kasir to receive
      _socketService.emit('order:update_status', {
        'orderId': event.orderId,
        'status': event.status,
      });

      final updatedOrders = state.orders.map((order) {
        if (order.id == event.orderId) {
          return order.copyWith(
            status: event.status,
            updatedAt: DateTime.now(),
          );
        }
        return order;
      }).toList();

      // Remove SERVED orders from display
      updatedOrders.removeWhere((o) => o.status == 'SERVED');

      emit(state.copyWith(orders: updatedOrders));
    } catch (e) {
      emit(state.copyWith(error: 'Failed to update order status'));
    }
  }

  Future<void> _onItemStatusUpdated(
    KitchenItemStatusUpdated event,
    Emitter<KitchenState> emit,
  ) async {
    try {
      await _kitchenRepository.updateOrderItemStatus(event.itemId, event.status);

      // Emit via WebSocket
      _socketService.emit('order:update_item_status', {
        'itemId': event.itemId,
        'status': event.status,
      });

      final updatedOrders = state.orders.map((order) {
        if (order.id == event.orderId) {
          final updatedItems = order.items.map((item) {
            if (item.id == event.itemId) {
              return item.copyWith(status: event.status);
            }
            return item;
          }).toList();
          return order.copyWith(items: updatedItems);
        }
        return order;
      }).toList();

      emit(state.copyWith(orders: updatedOrders));
    } catch (e) {
      emit(state.copyWith(error: 'Failed to update item status'));
    }
  }

  void _onExternalStatusUpdate(
    KitchenExternalStatusUpdate event,
    Emitter<KitchenState> emit,
  ) {
    final updatedOrders = state.orders.map((order) {
      if (order.id == event.updatedOrder.id) {
        return event.updatedOrder;
      }
      return order;
    }).toList();

    // Remove SERVED orders
    updatedOrders.removeWhere((o) => o.status == 'SERVED');

    emit(state.copyWith(orders: updatedOrders));
  }

  @override
  Future<void> close() {
    _newOrderSub?.cancel();
    _statusSub?.cancel();
    _itemDoneSub?.cancel();
    _connectionSub?.cancel();
    return super.close();
  }
}
```

- [ ] **Step 5: Write KitchenBloc test**

```dart
// mobile/test/modules/kitchen/bloc/kitchen_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:pos_mobile/modules/kitchen/bloc/kitchen_bloc.dart';
import 'package:pos_mobile/modules/kitchen/bloc/kitchen_event.dart';
import 'package:pos_mobile/modules/kitchen/bloc/kitchen_state.dart';
import 'package:pos_mobile/modules/kitchen/repository/kitchen_repository.dart';
import 'package:pos_mobile/core/socket/socket_service.dart';
import 'package:pos_mobile/core/storage/token_storage.dart';
import 'package:pos_mobile/models/order.dart';
import 'package:pos_mobile/models/order_item.dart';

class MockKitchenRepository extends Mock implements KitchenRepository {}
class MockSocketService extends Mock implements SocketService {}

void main() {
  late KitchenBloc kitchenBloc;
  late MockKitchenRepository mockKitchenRepository;
  late MockSocketService mockSocketService;

  final testOrder = Order(
    id: 'o1',
    transactionId: 't1',
    orderNumber: 1,
    status: 'PENDING',
    createdAt: DateTime(2026, 4, 18, 10, 0),
    updatedAt: DateTime(2026, 4, 18, 10, 0),
    items: [
      const OrderItem(
        id: 'oi1',
        orderId: 'o1',
        menuItemId: 'm1',
        menuItemName: 'Nasi Goreng',
        qty: 2,
        status: 'PENDING',
      ),
    ],
  );

  setUp(() {
    mockKitchenRepository = MockKitchenRepository();
    mockSocketService = MockSocketService();

    when(() => mockSocketService.connect()).thenAnswer((_) async {});
    when(() => mockSocketService.isConnected).thenReturn(true);
    when(() => mockSocketService.connectionStream)
        .thenAnswer((_) => Stream.value(true));
    when(() => mockSocketService.on<dynamic>(any()))
        .thenAnswer((_) => const Stream.empty());
    when(() => mockSocketService.emit(any(), any())).thenReturn(null);

    kitchenBloc = KitchenBloc(
      kitchenRepository: mockKitchenRepository,
      socketService: mockSocketService,
    );
  });

  tearDown(() {
    kitchenBloc.close();
  });

  group('KitchenNewOrderReceived', () {
    blocTest<KitchenBloc, KitchenState>(
      'adds new order to the list',
      build: () => kitchenBloc,
      act: (bloc) => bloc.add(KitchenNewOrderReceived(order: testOrder)),
      expect: () => [
        KitchenState(orders: [testOrder]),
      ],
    );

    blocTest<KitchenBloc, KitchenState>(
      'does not duplicate existing order',
      build: () => kitchenBloc,
      seed: () => KitchenState(orders: [testOrder]),
      act: (bloc) => bloc.add(KitchenNewOrderReceived(order: testOrder)),
      expect: () => [],
    );
  });

  group('KitchenOrderStatusUpdated', () {
    blocTest<KitchenBloc, KitchenState>(
      'updates order status to PREPARING',
      build: () {
        when(() => mockKitchenRepository.updateOrderStatus('o1', 'PREPARING'))
            .thenAnswer((_) async => testOrder.copyWith(status: 'PREPARING'));
        return kitchenBloc;
      },
      seed: () => KitchenState(orders: [testOrder]),
      act: (bloc) => bloc.add(const KitchenOrderStatusUpdated(
        orderId: 'o1',
        status: 'PREPARING',
      )),
      expect: () => [
        KitchenState(orders: [testOrder.copyWith(status: 'PREPARING')]),
      ],
      verify: (_) {
        verify(() => mockSocketService.emit('order:update_status', {
              'orderId': 'o1',
              'status': 'PREPARING',
            })).called(1);
      },
    );
  });

  group('KitchenItemStatusUpdated', () {
    blocTest<KitchenBloc, KitchenState>(
      'updates individual item status to DONE',
      build: () {
        when(() => mockKitchenRepository.updateOrderItemStatus('oi1', 'DONE'))
            .thenAnswer((_) async {});
        return kitchenBloc;
      },
      seed: () => KitchenState(orders: [testOrder]),
      act: (bloc) => bloc.add(const KitchenItemStatusUpdated(
        orderId: 'o1',
        itemId: 'oi1',
        status: 'DONE',
      )),
      expect: () => [
        KitchenState(orders: [
          testOrder.copyWith(
            items: [testOrder.items[0].copyWith(status: 'DONE')],
          ),
        ]),
      ],
    );
  });

  group('KitchenState computed properties', () {
    test('pendingOrders filters correctly', () {
      final state = KitchenState(orders: [
        testOrder,
        testOrder.copyWith(status: 'PREPARING'),
      ]);
      expect(state.pendingOrders.length, 1);
      expect(state.preparingOrders.length, 1);
    });
  });
}
```

- [ ] **Step 6: Run KitchenBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kitchen/bloc/kitchen_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7: Create OrderTimer widget**

```dart
// mobile/lib/modules/kitchen/widgets/order_timer.dart
import 'dart:async';
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';

class OrderTimer extends StatefulWidget {
  final DateTime createdAt;

  const OrderTimer({super.key, required this.createdAt});

  @override
  State<OrderTimer> createState() => _OrderTimerState();
}

class _OrderTimerState extends State<OrderTimer> {
  late Timer _timer;
  Duration _elapsed = Duration.zero;

  @override
  void initState() {
    super.initState();
    _updateElapsed();
    _timer = Timer.periodic(const Duration(seconds: 1), (_) {
      _updateElapsed();
    });
  }

  void _updateElapsed() {
    setState(() {
      _elapsed = DateTime.now().difference(widget.createdAt);
    });
  }

  @override
  void dispose() {
    _timer.cancel();
    super.dispose();
  }

  Color _getTimerColor() {
    if (_elapsed.inMinutes >= 15) return AppColors.error;
    if (_elapsed.inMinutes >= 10) return AppColors.warning;
    return AppColors.textSecondary;
  }

  @override
  Widget build(BuildContext context) {
    final minutes = _elapsed.inMinutes;
    final seconds = _elapsed.inSeconds % 60;

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
      decoration: BoxDecoration(
        color: _getTimerColor().withOpacity(0.1),
        borderRadius: BorderRadius.circular(6),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(
            Icons.timer,
            size: 14,
            color: _getTimerColor(),
          ),
          const SizedBox(width: 4),
          Text(
            '${minutes.toString().padLeft(2, '0')}:${seconds.toString().padLeft(2, '0')}',
            style: TextStyle(
              fontSize: 13,
              fontWeight: FontWeight.w700,
              fontFeatures: const [FontFeature.tabularFigures()],
              color: _getTimerColor(),
            ),
          ),
        ],
      ),
    );
  }
}
```

- [ ] **Step 8: Create OrderItemTile widget (Kitchen)**

```dart
// mobile/lib/modules/kitchen/widgets/order_item_tile.dart
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';
import '../../../models/order_item.dart';

class KitchenOrderItemTile extends StatelessWidget {
  final OrderItem item;
  final ValueChanged<String> onStatusChanged;

  const KitchenOrderItemTile({
    super.key,
    required this.item,
    required this.onStatusChanged,
  });

  @override
  Widget build(BuildContext context) {
    final isDone = item.status == 'DONE';

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: isDone
            ? AppColors.secondary.withOpacity(0.05)
            : Colors.transparent,
        border: const Border(
          bottom: BorderSide(color: AppColors.divider, width: 0.5),
        ),
      ),
      child: Row(
        children: [
          GestureDetector(
            onTap: () {
              if (!isDone) {
                onStatusChanged('DONE');
              }
            },
            child: Container(
              width: 24,
              height: 24,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: isDone ? AppColors.secondary : Colors.transparent,
                border: Border.all(
                  color: isDone ? AppColors.secondary : AppColors.border,
                  width: 2,
                ),
              ),
              child: isDone
                  ? const Icon(Icons.check, size: 14, color: Colors.white)
                  : null,
            ),
          ),
          const SizedBox(width: 10),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '${item.menuItemName} x${item.qty}',
                  style: TextStyle(
                    fontSize: 13,
                    fontWeight: FontWeight.w600,
                    decoration: isDone ? TextDecoration.lineThrough : null,
                    color: isDone ? AppColors.textLight : AppColors.textPrimary,
                  ),
                ),
                if (item.notes != null && item.notes!.isNotEmpty)
                  Text(
                    item.notes!,
                    style: const TextStyle(
                      fontSize: 11,
                      fontStyle: FontStyle.italic,
                      color: AppColors.warning,
                    ),
                  ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

- [ ] **Step 9: Create OrderCard widget (Kitchen)**

```dart
// mobile/lib/modules/kitchen/widgets/order_card.dart
import 'package:flutter/material.dart';
import '../../../config/app_theme.dart';
import '../../../shared/widgets/status_badge.dart';
import '../../../shared/widgets/app_button.dart';
import '../../../models/order.dart';
import 'order_timer.dart';
import 'order_item_tile.dart';

class OrderCard extends StatelessWidget {
  final Order order;
  final void Function(String orderId, String status) onOrderStatusChanged;
  final void Function(String orderId, String itemId, String status)
      onItemStatusChanged;

  const OrderCard({
    super.key,
    required this.order,
    required this.onOrderStatusChanged,
    required this.onItemStatusChanged,
  });

  Color get _borderColor => AppColors.statusColor(order.status);
  Color get _headerColor => AppColors.statusBackground(order.status);

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 280,
      margin: const EdgeInsets.only(right: 16, bottom: 16),
      decoration: BoxDecoration(
        color: AppColors.surface,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: _borderColor, width: 2),
        boxShadow: [
          BoxShadow(
            color: _borderColor.withOpacity(0.15),
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        mainAxisSize: MainAxisSize.min,
        children: [
          // Header
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 10),
            decoration: BoxDecoration(
              color: _headerColor,
              borderRadius: const BorderRadius.vertical(
                top: Radius.circular(10),
              ),
            ),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Row(
                  children: [
                    Text(
                      '#${order.orderNumber}',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.w800,
                        color: _borderColor,
                      ),
                    ),
                    const SizedBox(width: 8),
                    StatusBadge(status: order.status),
                  ],
                ),
                OrderTimer(createdAt: order.createdAt),
              ],
            ),
          ),

          // Items
          Flexible(
            child: ListView.builder(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              itemCount: order.items.length,
              itemBuilder: (context, index) {
                final item = order.items[index];
                return KitchenOrderItemTile(
                  item: item,
                  onStatusChanged: (status) {
                    onItemStatusChanged(order.id, item.id, status);
                  },
                );
              },
            ),
          ),

          // Action buttons
          Padding(
            padding: const EdgeInsets.all(12),
            child: _buildActionButton(),
          ),
        ],
      ),
    );
  }

  Widget _buildActionButton() {
    switch (order.status) {
      case 'PENDING':
        return AppButton(
          label: 'Mulai Proses',
          icon: Icons.play_arrow,
          color: AppColors.statusPreparing,
          width: double.infinity,
          onPressed: () => onOrderStatusChanged(order.id, 'PREPARING'),
        );
      case 'PREPARING':
        final allDone = order.allItemsDone;
        return AppButton(
          label: allDone ? 'Tandai Siap' : 'Selesaikan Item Dulu',
          icon: allDone ? Icons.check_circle : Icons.hourglass_top,
          color: allDone ? AppColors.statusReady : AppColors.textLight,
          width: double.infinity,
          onPressed: allDone
              ? () => onOrderStatusChanged(order.id, 'READY')
              : null,
        );
      case 'READY':
        return Container(
          width: double.infinity,
          padding: const EdgeInsets.symmetric(vertical: 10),
          decoration: BoxDecoration(
            color: AppColors.statusReady.withOpacity(0.1),
            borderRadius: BorderRadius.circular(10),
          ),
          child: const Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.check_circle, color: AppColors.statusReady, size: 20),
              SizedBox(width: 8),
              Text(
                'Siap Disajikan',
                style: TextStyle(
                  fontSize: 14,
                  fontWeight: FontWeight.w600,
                  color: AppColors.statusReady,
                ),
              ),
            ],
          ),
        );
      default:
        return const SizedBox.shrink();
    }
  }
}
```

- [ ] **Step 10: Create KitchenDisplayScreen**

```dart
// mobile/lib/modules/kitchen/screens/kitchen_display_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../config/app_theme.dart';
import '../../../core/api/api_client.dart';
import '../../../core/socket/socket_service.dart';
import '../../auth/bloc/auth_bloc.dart';
import '../../auth/bloc/auth_event.dart';
import '../bloc/kitchen_bloc.dart';
import '../bloc/kitchen_event.dart';
import '../bloc/kitchen_state.dart';
import '../repository/kitchen_repository.dart';
import '../widgets/order_card.dart';

class KitchenDisplayScreen extends StatelessWidget {
  const KitchenDisplayScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final apiClient = context.read<ApiClient>();
    final socketService = context.read<SocketService>();
    final kitchenRepository = KitchenRepository(apiClient: apiClient);

    return BlocProvider(
      create: (_) => KitchenBloc(
        kitchenRepository: kitchenRepository,
        socketService: socketService,
      )..add(KitchenStarted()),
      child: const _KitchenDisplayView(),
    );
  }
}

class _KitchenDisplayView extends StatelessWidget {
  const _KitchenDisplayView();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Row(
          children: [
            Icon(Icons.restaurant, color: AppColors.primary),
            SizedBox(width: 10),
            Text('Kitchen Display'),
          ],
        ),
        actions: [
          BlocBuilder<KitchenBloc, KitchenState>(
            builder: (context, state) {
              return Padding(
                padding: const EdgeInsets.only(right: 12),
                child: Row(
                  children: [
                    Container(
                      width: 10,
                      height: 10,
                      decoration: BoxDecoration(
                        shape: BoxShape.circle,
                        color: state.isConnected
                            ? AppColors.secondary
                            : AppColors.error,
                      ),
                    ),
                    const SizedBox(width: 6),
                    Text(
                      state.isConnected ? 'Live' : 'Disconnected',
                      style: TextStyle(
                        fontSize: 13,
                        fontWeight: FontWeight.w600,
                        color: state.isConnected
                            ? AppColors.secondary
                            : AppColors.error,
                      ),
                    ),
                    const SizedBox(width: 16),
                    _buildOrderCount(
                      'Pending',
                      state.pendingOrders.length,
                      AppColors.statusPending,
                    ),
                    const SizedBox(width: 12),
                    _buildOrderCount(
                      'Proses',
                      state.preparingOrders.length,
                      AppColors.statusPreparing,
                    ),
                    const SizedBox(width: 12),
                    _buildOrderCount(
                      'Siap',
                      state.readyOrders.length,
                      AppColors.statusReady,
                    ),
                  ],
                ),
              );
            },
          ),
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: () {
              context.read<AuthBloc>().add(AuthLogoutRequested());
            },
          ),
        ],
      ),
      body: BlocBuilder<KitchenBloc, KitchenState>(
        builder: (context, state) {
          if (state.isLoading) {
            return const Center(child: CircularProgressIndicator());
          }

          if (state.orders.isEmpty) {
            return const Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Icon(
                    Icons.restaurant_menu,
                    size: 80,
                    color: AppColors.textLight,
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Belum ada pesanan masuk',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.w500,
                      color: AppColors.textSecondary,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Pesanan baru akan muncul otomatis',
                    style: TextStyle(
                      fontSize: 14,
                      color: AppColors.textLight,
                    ),
                  ),
                ],
              ),
            );
          }

          return SingleChildScrollView(
            padding: const EdgeInsets.all(16),
            scrollDirection: Axis.horizontal,
            child: Row(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // PENDING column
                _buildColumn(
                  context,
                  'PENDING',
                  state.pendingOrders,
                  AppColors.statusPending,
                ),
                const SizedBox(width: 8),
                // PREPARING column
                _buildColumn(
                  context,
                  'SEDANG DIPROSES',
                  state.preparingOrders,
                  AppColors.statusPreparing,
                ),
                const SizedBox(width: 8),
                // READY column
                _buildColumn(
                  context,
                  'SIAP',
                  state.readyOrders,
                  AppColors.statusReady,
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _buildColumn(
    BuildContext context,
    String title,
    List orders,
    Color color,
  ) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
          margin: const EdgeInsets.only(bottom: 12),
          decoration: BoxDecoration(
            color: color.withOpacity(0.1),
            borderRadius: BorderRadius.circular(8),
          ),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Container(
                width: 8,
                height: 8,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: color,
                ),
              ),
              const SizedBox(width: 8),
              Text(
                '$title (${orders.length})',
                style: TextStyle(
                  fontSize: 14,
                  fontWeight: FontWeight.w700,
                  color: color,
                ),
              ),
            ],
          ),
        ),
        Wrap(
          spacing: 0,
          runSpacing: 0,
          children: orders.map<Widget>((order) {
            return OrderCard(
              order: order,
              onOrderStatusChanged: (orderId, status) {
                context.read<KitchenBloc>().add(
                      KitchenOrderStatusUpdated(
                        orderId: orderId,
                        status: status,
                      ),
                    );
              },
              onItemStatusChanged: (orderId, itemId, status) {
                context.read<KitchenBloc>().add(
                      KitchenItemStatusUpdated(
                        orderId: orderId,
                        itemId: itemId,
                        status: status,
                      ),
                    );
              },
            );
          }).toList(),
        ),
      ],
    );
  }

  Widget _buildOrderCount(String label, int count, Color color) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(16),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Text(
            '$count',
            style: TextStyle(
              fontSize: 14,
              fontWeight: FontWeight.w700,
              color: color,
            ),
          ),
          const SizedBox(width: 4),
          Text(
            label,
            style: TextStyle(
              fontSize: 11,
              color: color,
              fontWeight: FontWeight.w500,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### Task 13: Final Verification & Widget Test

**Files:**
- Modify: `mobile/test/widget_test.dart`

- [ ] **Step 1: Replace default widget test**

```dart
// mobile/test/widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:pos_mobile/config/app_theme.dart';
import 'package:pos_mobile/shared/widgets/app_button.dart';
import 'package:pos_mobile/shared/widgets/status_badge.dart';
import 'package:pos_mobile/models/cart_item.dart';
import 'package:pos_mobile/models/promo.dart';

void main() {
  group('AppButton', () {
    testWidgets('renders label text', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: AppButton(label: 'Test Button'),
          ),
        ),
      );

      expect(find.text('Test Button'), findsOneWidget);
    });

    testWidgets('shows loading indicator when isLoading', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: AppButton(label: 'Loading', isLoading: true),
          ),
        ),
      );

      expect(find.byType(CircularProgressIndicator), findsOneWidget);
      expect(find.text('Loading'), findsNothing);
    });

    testWidgets('calls onPressed when tapped', (tester) async {
      var pressed = false;
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: AppButton(
              label: 'Tap Me',
              onPressed: () => pressed = true,
            ),
          ),
        ),
      );

      await tester.tap(find.text('Tap Me'));
      expect(pressed, isTrue);
    });
  });

  group('StatusBadge', () {
    testWidgets('renders status text', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: StatusBadge(status: 'PENDING'),
          ),
        ),
      );

      expect(find.text('PENDING'), findsOneWidget);
    });

    testWidgets('applies correct color for each status', (tester) async {
      for (final status in ['PENDING', 'PREPARING', 'READY', 'SERVED']) {
        await tester.pumpWidget(
          MaterialApp(
            home: Scaffold(
              body: StatusBadge(status: status),
            ),
          ),
        );
        expect(find.text(status), findsOneWidget);
      }
    });
  });

  group('CartItem model', () {
    test('subtotal calculates correctly', () {
      const item = CartItem(
        itemId: '1',
        name: 'Test',
        type: CartItemType.menu,
        unitPrice: 25000,
        quantity: 3,
      );
      expect(item.subtotal, 75000);
    });

    test('toTransactionItem generates correct format', () {
      const item = CartItem(
        itemId: 'm1',
        name: 'Nasi Goreng',
        type: CartItemType.menu,
        unitPrice: 25000,
        quantity: 2,
      );

      final result = item.toTransactionItem();
      expect(result['itemType'], 'MENU');
      expect(result['itemId'], 'm1');
      expect(result['qty'], 2);
    });

    test('copyWith preserves unchanged fields', () {
      const original = CartItem(
        itemId: '1',
        name: 'Test',
        type: CartItemType.menu,
        unitPrice: 25000,
        quantity: 1,
      );

      final updated = original.copyWith(quantity: 5, notes: 'Extra');
      expect(updated.itemId, '1');
      expect(updated.name, 'Test');
      expect(updated.quantity, 5);
      expect(updated.notes, 'Extra');
    });
  });

  group('Promo model', () {
    test('PERCENT discount calculates correctly', () {
      final promo = Promo(
        id: '1',
        name: 'Diskon 20%',
        type: 'PERCENT',
        value: 20,
        startDate: DateTime(2026, 1, 1),
        endDate: DateTime(2026, 12, 31),
      );
      expect(promo.calculateDiscount(100000), 20000);
    });

    test('FIXED discount caps at total amount', () {
      final promo = Promo(
        id: '1',
        name: 'Potongan 50K',
        type: 'FIXED',
        value: 50000,
        startDate: DateTime(2026, 1, 1),
        endDate: DateTime(2026, 12, 31),
      );
      expect(promo.calculateDiscount(30000), 30000);
      expect(promo.calculateDiscount(100000), 50000);
    });

    test('MIN_PURCHASE discount applies only when threshold met', () {
      final promo = Promo(
        id: '1',
        name: 'Min 100K',
        type: 'MIN_PURCHASE',
        value: 10,
        minPurchaseAmount: 100000,
        startDate: DateTime(2026, 1, 1),
        endDate: DateTime(2026, 12, 31),
      );
      expect(promo.calculateDiscount(50000), 0);
      expect(promo.calculateDiscount(150000), 15000);
    });

    test('isValid checks active period and usage', () {
      final validPromo = Promo(
        id: '1',
        name: 'Valid',
        type: 'PERCENT',
        value: 10,
        startDate: DateTime(2025, 1, 1),
        endDate: DateTime(2027, 12, 31),
        isActive: true,
      );
      expect(validPromo.isValid, isTrue);

      final expiredPromo = Promo(
        id: '2',
        name: 'Expired',
        type: 'PERCENT',
        value: 10,
        startDate: DateTime(2020, 1, 1),
        endDate: DateTime(2020, 12, 31),
        isActive: true,
      );
      expect(expiredPromo.isValid, isFalse);

      final maxedPromo = Promo(
        id: '3',
        name: 'Maxed',
        type: 'PERCENT',
        value: 10,
        startDate: DateTime(2025, 1, 1),
        endDate: DateTime(2027, 12, 31),
        isActive: true,
        maxUsage: 10,
        usedCount: 10,
      );
      expect(maxedPromo.isValid, isFalse);
    });
  });
}
```

- [ ] **Step 2: Run all tests**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test
```

Expected: All tests PASS

- [ ] **Step 3: Run flutter analyze**

```bash
cd ~/Downloads/PROJECT/mobile
flutter analyze
```

Expected: No errors

- [ ] **Step 4: Verify the app builds**

```bash
cd ~/Downloads/PROJECT/mobile
flutter build apk --debug
```

Expected: Build successful
