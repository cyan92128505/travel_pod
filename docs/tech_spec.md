# 完整系統架構設計文件

## 技術決策總結

### 核心架構模式
- **Clean Architecture + DDD** (Domain-Driven Design)
- **Repository Pattern** + **Dependency Injection**
- **Multi-tenant Architecture** (Row-level isolation)

### 技術棧選擇
**第一階段 (快速上線)**:
- Frontend: Next.js + Flutter Web
- Backend: Firebase Cloud Functions (REST API)
- Database: Firestore
- Auth: Firebase Authentication

**保留遷移彈性**:
- Backend: Express.js/TypeScript 或 Serverpod
- Database: PostgreSQL (Supabase)
- Auth: JWT + Redis
- Cache: Redis

## 系統架構圖

```
┌─────────────────┐    ┌─────────────────┐
│   Next.js       │    │  Flutter Web    │
│ (靜態展示網站)    │    │  (應用功能)      │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          └──────────┬───────────┘
                     │ REST API calls
         ┌───────────▼───────────┐
         │ Firebase Cloud        │
         │ Functions (REST)      │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │   UseCase Layer       │
         │ (Business Logic)      │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │  Repository Abstract  │
         │     Interface        │
         └───────────┬───────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼────┐   ┌───────▼───────┐   ┌────▼────┐
│Firebase│   │   Supabase    │   │Serverpod│
│ Impl   │   │    Impl       │   │  Impl   │
└───┬────┘   └───────┬───────┘   └────┬────┘
    │                │                │
┌───▼────┐   ┌───────▼───────┐   ┌────▼────┐
│Firestore│  │  PostgreSQL   │   │PostgreSQL│
└────────┘   └───────────────┘   └─────────┘
```

## Clean Architecture 分層詳述

### 1. Domain Layer (核心業務邏輯)
```dart
// Entities (聚合根)
class Order {
  final String id;
  final String tenantId;    // 多租戶隔離
  final String userId;
  final String serviceType; // 'immigration' | 'passport'
  final OrderStatus status;
  final Map<String, dynamic> formData;
  final List<Document> documents;
  final DateTime createdAt;
}

class User {
  final String id;
  final String tenantId;    // 多租戶隔離
  final String name;
  final String email;
  final String phone;
  final UserRole role;      // 'customer' | 'admin' | 'super_admin'
}

// Repository Interfaces
abstract class OrderRepository {
  Future<List<Order>> getOrdersByUser(String userId, String tenantId);
  Future<Order> createOrder(Order order);
  Future<Order> updateOrderStatus(String orderId, OrderStatus status);
  Future<List<Order>> getOrdersByTenant(String tenantId, {OrderFilter? filter});
}

abstract class UserRepository {
  Future<User?> getUserById(String userId);
  Future<User> createUser(User user);
  Future<List<User>> getUsersByTenant(String tenantId);
}
```

### 2. Application Layer (UseCase)
```dart
// Auth Context 抽象
abstract class AuthContext {
  String get currentUserId;
  String get tenantId;
  List<String> get permissions;
  bool hasPermission(String permission);
}

// Use Cases
class GetOrdersUseCase {
  final OrderRepository _orderRepository;
  
  GetOrdersUseCase(this._orderRepository);

  Future<List<Order>> execute(AuthContext auth, {OrderFilter? filter}) async {
    // 業務邏輯：只能查看自己租戶的訂單
    return _orderRepository.getOrdersByUser(
      auth.currentUserId, 
      auth.tenantId
    );
  }
}

class CreateOrderUseCase {
  final OrderRepository _orderRepository;
  final NotificationService _notificationService;
  
  Future<Order> execute(CreateOrderRequest request, AuthContext auth) async {
    // 業務邏輯驗證
    final order = Order.create(
      userId: auth.currentUserId,
      tenantId: auth.tenantId,
      serviceType: request.serviceType,
      formData: request.formData,
    );
    
    // 儲存
    final savedOrder = await _orderRepository.createOrder(order);
    
    // 發送通知
    await _notificationService.sendOrderCreatedNotification(savedOrder);
    
    return savedOrder;
  }
}
```

### 3. Infrastructure Layer (實作細節)

#### Firebase 實作
```dart
// Firebase Auth Context
class FirebaseAuthContext implements AuthContext {
  final User _firebaseUser;
  
  @override
  String get currentUserId => _firebaseUser.uid;
  
  @override
  String get tenantId => _firebaseUser.customClaims?['tenant_id'] ?? '';
}

// Firebase Repository 實作
class FirebaseOrderRepository implements OrderRepository {
  final FirebaseFirestore _db;

  @override
  Future<List<Order>> getOrdersByUser(String userId, String tenantId) async {
    final snapshot = await _db
        .collection('orders')
        .where('tenant_id', isEqualTo: tenantId)
        .where('user_id', isEqualTo: userId)
        .orderBy('created_at', descending: true)
        .get();
    
    return snapshot.docs.map((doc) => Order.fromFirestore(doc)).toList();
  }

  @override
  Future<Order> createOrder(Order order) async {
    final docRef = await _db.collection('orders').add(order.toFirestore());
    return order.copyWith(id: docRef.id);
  }
}
```

### 4. Presentation Layer (API)

#### Firebase Cloud Functions (REST)
```javascript
// functions/src/index.js
const functions = require('firebase-functions');
const admin = require('firebase-admin');

// 認證中間件
const authenticate = async (context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User not authenticated');
  }
  
  return {
    currentUserId: context.auth.uid,
    tenantId: context.auth.token.tenant_id || 'default',
    permissions: context.auth.token.permissions || []
  };
};

// REST Endpoints
exports.getMyOrders = functions.https.onCall(async (data, context) => {
  const auth = await authenticate(context);
  const useCase = container.get('GetOrdersUseCase');
  return useCase.execute(auth, data);
});

exports.createOrder = functions.https.onCall(async (data, context) => {
  const auth = await authenticate(context);
  const useCase = container.get('CreateOrderUseCase');
  return useCase.execute(data, auth);
});

exports.updateOrderStatus = functions.https.onCall(async (data, context) => {
  const auth = await authenticate(context);
  // 檢查管理員權限
  if (!auth.permissions.includes('admin')) {
    throw new functions.https.HttpsError('permission-denied', 'Admin access required');
  }
  const useCase = container.get('UpdateOrderStatusUseCase');
  return useCase.execute(data, auth);
});
```

## 多租戶架構設計

### Row-Level Isolation
```javascript
// Firestore 資料結構
{
  "orders": {
    "order_001": {
      "tenant_id": "chen_agency",     // 多租戶隔離
      "brand_type": "immigration",    // 品牌區分
      "user_id": "user123",
      "service_type": "vietnam_immigration",
      "status": "processing",
      // ...
    }
  },
  "users": {
    "user123": {
      "tenant_id": "chen_agency",     // 多租戶隔離
      "name": "王小明",
      "email": "wang@example.com",
      // ...
    }
  }
}
```

### 權限控制
```dart
class TenantGuard {
  static bool canAccess(AuthContext auth, String resourceTenantId) {
    return auth.tenantId == resourceTenantId || 
           auth.permissions.contains('super_admin');
  }
}
```

## 依賴注入配置

```dart
// DI Container
class DIContainer {
  static void configure() {
    // 根據環境選擇實作
    if (useFirebase) {
      GetIt.instance.registerLazySingleton<OrderRepository>(
        () => FirebaseOrderRepository(FirebaseFirestore.instance)
      );
      GetIt.instance.registerLazySingleton<AuthContext>(
        () => FirebaseAuthContext(FirebaseAuth.instance.currentUser!)
      );
    } else if (useSupabase) {
      GetIt.instance.registerLazySingleton<OrderRepository>(
        () => SupabaseOrderRepository(supabaseClient)
      );
      GetIt.instance.registerLazySingleton<AuthContext>(
        () => JWTAuthContext(jwtToken)
      );
    }
    
    // Use Cases
    GetIt.instance.registerFactory(
      () => GetOrdersUseCase(GetIt.instance<OrderRepository>())
    );
  }
}
```

## API 端點設計

### 客戶端 API
```
GET  /getMyOrders           - 取得我的訂單列表
POST /createOrder           - 創建新訂單
GET  /getOrderDetail        - 取得訂單詳細資料
POST /uploadDocument        - 上傳文件
POST /updateProfile         - 更新個人資料
```

### 管理端 API
```
GET  /getAllOrders          - 取得所有訂單 (管理員)
POST /updateOrderStatus     - 更新訂單狀態 (管理員)
GET  /getOrdersSummary      - 取得訂單統計報表 (管理員)
GET  /getAllUsers           - 取得所有用戶 (管理員)
POST /sendNotification      - 發送通知 (管理員)
```

## 前端整合

### Flutter Web
```dart
class ApiService {
  final FirebaseFunctions _functions;
  
  Future<List<Order>> getMyOrders() async {
    final callable = _functions.httpsCallable('getMyOrders');
    final result = await callable.call();
    return (result.data as List).map((e) => Order.fromJson(e)).toList();
  }
  
  Future<Order> createOrder(CreateOrderRequest request) async {
    final callable = _functions.httpsCallable('createOrder');
    final result = await callable.call(request.toJson());
    return Order.fromJson(result.data);
  }
}
```

### Next.js
```javascript
// API 呼叫封裝
class ApiClient {
  async getMyOrders() {
    const callable = httpsCallable(functions, 'getMyOrders');
    const result = await callable();
    return result.data;
  }
  
  async createOrder(orderData) {
    const callable = httpsCallable(functions, 'createOrder');
    const result = await callable(orderData);
    return result.data;
  }
}
```

## 遷移計劃

### 階段 1: Firebase 快速上線 (1-3 個月)
- ✅ Firebase Functions REST API
- ✅ Firebase Auth + Firestore
- ✅ 基本 Clean Architecture
- ✅ 多租戶 Row-level isolation

### 階段 2: 架構強化 (平行進行)
- ✅ 完整 Repository 抽象層
- ✅ 依賴注入容器
- ✅ 單元測試覆蓋
- ✅ 效能監控

### 階段 3: 可選技術遷移
**選項 A: 保持 Firebase**
- 如果流量和成本可控
- 繼續享受 Firebase 生態系

**選項 B: 遷移至 Express.js + PostgreSQL**
- 更多控制權
- 降低長期成本
- 更複雜的查詢能力

**選項 C: 遷移至 Serverpod**
- 統一 Dart 技術棧
- 學習新技術
- 完全自主控制

## 關鍵優勢

1. **快速上線**: Firebase 托管服務，專注業務開發
2. **架構彈性**: Clean Architecture 支援後續技術遷移
3. **多租戶就緒**: 設計時即考慮未來擴展
4. **技術債控制**: 分層設計易於維護和重構
5. **測試友善**: 依賴注入支援完整單元測試
