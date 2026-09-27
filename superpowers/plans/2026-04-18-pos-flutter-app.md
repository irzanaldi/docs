# POS System Phase 1 — Flutter App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build Flutter mobile app for kasir (cashier) + kitchen display w/ real-time order flow.

**Architecture:** Single Flutter project, two flows: Kasir & Kitchen Display. BLoC state mgmt, Dio REST calls, socket_io_client WebSocket → NestJS backend.

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

### Task 1: Project Setup & Deps

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

- [ ] **Step 2: Replace pubspec.yaml w/ deps**

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

- [ ] **Step 3: Install deps**

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

Expected: Compiles (some missing refs expected until later tasks)

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

### Task 5: Auth Module — Login, BLoC, JWT Mgmt

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

- [ ] **Step 2-4: Create MenuEvent, MenuState, MenuBloc** — Same code blocks as original (BLoC event/state/bloc pattern for menu loading, category filter, search filter w/ `_applyFilters`)

- [ ] **Step 5: Write MenuBloc test** — Tests menu load, category filter → FOOD only, search filter → name match

- [ ] **Step 6: Run MenuBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/menu_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 7-10: Create CategoryFilter, SearchBarWidget, MenuItemCard, MenuGrid widgets** — UI components for menu browsing; MenuGrid wires search/category/cart-add together

Full code for Steps 2-5 and 7-10:

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

### Task 7: Kasir — Cart Mgmt

**Files:**
- Create: `mobile/lib/modules/kasir/bloc/cart_event.dart`
- Create: `mobile/lib/modules/kasir/bloc/cart_state.dart`
- Create: `mobile/lib/modules/kasir/bloc/cart_bloc.dart`
- Create: `mobile/lib/modules/kasir/widgets/cart_panel.dart`
- Create: `mobile/lib/modules/kasir/widgets/cart_item_tile.dart`
- Create: `mobile/test/modules/kasir/bloc/cart_bloc_test.dart`

- [ ] **Step 1-3: Create CartEvent, CartState, CartBloc** — Add/remove/update-qty/update-notes/clear; existing item → increment qty

- [ ] **Step 4-5: Write & run CartBloc test**

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/cart_bloc_test.dart
```

Expected: All tests PASS

- [ ] **Step 6-7: Create CartItemTile, CartPanel widgets** — CartItemTile w/ qty +/- buttons, notes dialog; CartPanel w/ header, list, promo section, footer w/ subtotal/discount/total/pay button

Full code for Tasks 7 steps (cart_event, cart_state, cart_bloc, cart_bloc_test, cart_item_tile, cart_panel) — all code blocks identical to original, omitted here for brevity since they are pure code.

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

### Task 8-13: Remaining Modules

Tasks 8-13 follow same pattern. All code blocks remain unchanged. Prose compressed below:

---

### Task 8: Kasir — Promo System

**Files:** `promo_repository.dart`, `promo_event.dart`, `promo_state.dart`, `promo_bloc.dart`, `promo_badge.dart`, `promo_bloc_test.dart`

- [ ] Steps 1-4: PromoRepository (GET `/promos/active`), PromoEvent (load/select/deselect), PromoState (activePromos, selectedPromo, isLoading), PromoBloc (filter valid promos, select/deselect)
- [ ] Steps 5-6: PromoBloc test → PERCENT/FIXED discount calc, select/deselect
- [ ] Step 7: PromoBadge widget — horizontal scrollable promo chips, tap to select/deselect, shows discount amount

All code blocks identical to original spec. Run:

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/promo_bloc_test.dart
```

---

### Task 9: Kasir — Checkout & Payment

**Files:** `transaction_repository.dart`, `checkout_event.dart`, `checkout_state.dart`, `checkout_bloc.dart`, `payment_method_selector.dart`, `checkout_screen.dart`, `receipt_screen.dart`, `checkout_bloc_test.dart`

- [ ] Steps 1-4: TransactionRepository (POST `/pos/transactions`, GET `/pos/transactions`), CheckoutEvent (payment method select, submit, reset), CheckoutState (Initial/Processing/Success/Failure), CheckoutBloc
- [ ] Steps 5-6: CheckoutBloc test → payment method update, success/failure flows
- [ ] Step 7: PaymentMethodSelector — CASH/QRIS/EDC cards
- [ ] Step 8: CheckoutScreen — payment method + order summary, on success → clear cart + navigate to receipt
- [ ] Step 9: ReceiptScreen — success icon, transaction details, print/new transaction buttons

All code blocks identical to original spec. Run:

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kasir/bloc/checkout_bloc_test.dart
```

---

### Task 10: Kasir — Order Status Tracking (Real-time WebSocket)

**Files:** `order_tracking_event.dart`, `order_tracking_state.dart`, `order_tracking_bloc.dart`, `order_tracking_screen.dart`

- [ ] Steps 1-3: OrderTrackingEvent (started/orderReceived/statusUpdated/served), OrderTrackingState (activeOrders, isConnected), OrderTrackingBloc — connects WebSocket, loads active orders from `/orders/active`, listens `order:new` + `order:status`, emits `order:update_status` for SERVED
- [ ] Step 4: OrderTrackingScreen — connection indicator, order list w/ status badges, "Served" button for READY orders, elapsed time display

All code blocks identical to original spec.

---

### Task 11: Kasir Main Screen (Wiring Everything)

**Files:** `kasir_main_screen.dart`

- [ ] Step 1: KasirMainScreen — MultiBlocProvider (MenuBloc, CartBloc, PromoBloc, CheckoutBloc, OrderTrackingBloc), side nav (Kasir/Order/Logout), IndexedStack for view switching, MenuGrid + CartPanel layout, checkout navigation w/ BlocProvider.value pass-through

All code blocks identical to original spec.

---

### Task 12: Kitchen Display — Order Board (WebSocket + Color-Coded Cards)

**Files:** `kitchen_repository.dart`, `kitchen_event.dart`, `kitchen_state.dart`, `kitchen_bloc.dart`, `order_timer.dart`, `order_item_tile.dart`, `order_card.dart`, `kitchen_display_screen.dart`, `kitchen_bloc_test.dart`

- [ ] Steps 1-4: KitchenRepository (GET `/orders/active`, PATCH `/orders/:id/status`, PATCH `/orders/items/:id/status`), KitchenEvent (started/newOrder/orderStatus/itemStatus/externalUpdate), KitchenState (orders, isLoading, isConnected + computed pendingOrders/preparingOrders/readyOrders), KitchenBloc — WebSocket listeners for `order:new` + `order:status`, API calls for status updates, emits via WebSocket for kasir sync, removes SERVED orders
- [ ] Steps 5-6: KitchenBloc test → new order add, no duplicate, status update → PREPARING, item status → DONE
- [ ] Step 7: OrderTimer — live elapsed timer, color: >15m red, >10m yellow
- [ ] Step 8: KitchenOrderItemTile — checkbox circle, tap → DONE, strikethrough when done, notes in italic/warning color
- [ ] Step 9: OrderCard — color-coded border/header per status, item list, action button (PENDING→"Mulai Proses", PREPARING→"Tandai Siap" when all items done, READY→"Siap Disajikan" badge)
- [ ] Step 10: KitchenDisplayScreen — 3-column Kanban layout (PENDING/SEDANG DIPROSES/SIAP), connection indicator, order counts in app bar, logout

All code blocks identical to original spec. Run:

```bash
cd ~/Downloads/PROJECT/mobile
flutter test test/modules/kitchen/bloc/kitchen_bloc_test.dart
```

---

### Task 13: Final Verification & Widget Test

**Files:** Modify: `mobile/test/widget_test.dart`

- [ ] **Step 1: Replace default widget test** — Tests for AppButton (render, loading, onPressed), StatusBadge (render, colors), CartItem model (subtotal, toTransactionItem, copyWith), Promo model (PERCENT/FIXED/MIN_PURCHASE discount, isValid checks)

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

- [ ] **Step 4: Verify app builds**

```bash
cd ~/Downloads/PROJECT/mobile
flutter build apk --debug
```

Expected: Build successful
