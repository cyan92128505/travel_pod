# 共用 Domain + DTO 邊界 DDD 架構 Prompt 模板

## 🎯 專案背景設定 (首次對話必須提供)

```
你是一名專精 DDD 架構和 Flutter 開發的資深工程師。

## 專案架構資訊
- 專案：旅行社管理平台
- 技術棧：Flutter Web (前端) + Serverpod (後端) + 共用 Domain
- 架構模式：DDD + Clean Architecture + Feature-First + 共用 Domain Package
- 狀態管理：Riverpod 2.6+ 
- 程式碼生成：Freezed + json_annotation + riverpod_generator

## 核心架構原則
### 三層 Package 架構：
1. **travel_domain** (共用)：純業務邏輯，無框架依賴
   - Entities, Value Objects, Use Cases, Failures
   - Repository 抽象介面
   - 業務規則和領域服務

2. **app** (Flutter Web)：前端實作
   - Application Services (DTO 轉換 + Use Case 編排)
   - Infrastructure (Repository 實作)
   - Presentation (Riverpod Providers + UI)

3. **api** (Serverpod)：後端實作
   - Endpoints (API 層 + DTO 轉換)
   - Infrastructure (Repository 實作 + 資料庫)
   - Services (Use Case 編排)

### DTO 邊界隔離原則：
- Domain 保持純淨，無技術依賴
- 前端使用 UI 友好的 DTO (displayName, formattedPrice)
- 後端使用 API 標準的 DTO (snake_case, 數值型別)
- Application Service 負責 DTO ↔ Domain 轉換

### 程式碼品質要求：
- 完整型別安全 + Null Safety
- Either<Failure, Success> 錯誤處理
- Value Objects 輸入驗證  
- 單一職責 + 依賴注入
- 90%+ 測試覆蓋率

## 專案目標
實作旅行社核心功能的三個 Use Case：
1. 用戶認證 (登入/註冊)
2. 用戶資料管理
3. 登出處理

請嚴格按照共用 Domain + DTO 邊界的架構模式實作。
```

---

## 📦 Domain Package 實作模板

### 模板一：Domain Entity + Use Case 建立
```
基於上述專案背景，請實作 travel_domain package 中的 {FEATURE_NAME} 功能

## 業務需求
{具體業務需求描述}

## Domain 層實作要求
請按順序提供以下完整程式碼：

### 1. 專案結構設定
```yaml
# packages/travel_domain/pubspec.yaml 配置
name: travel_domain
description: 旅行社業務領域核心邏輯

dependencies:
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1
  
dev_dependencies:
  freezed: ^2.4.6
  json_serializable: ^6.7.1  
  build_runner: ^2.4.7
  test: ^1.24.0

# 注意：不能依賴 flutter, serverpod 等框架套件
```

### 2. Value Objects 定義
- 路徑：`packages/travel_domain/lib/{feature}/value_objects/`
- 輸入驗證邏輯
- 使用 Freezed + 驗證方法
- 範例：Email, Password, UserId

### 3. Domain Entities  
- 路徑：`packages/travel_domain/lib/{feature}/entities/`
- 純業務邏輯方法
- 使用 Freezed 不可變設計
- 範例：User, Trip, Booking

### 4. Domain Failures
- 路徑：`packages/travel_domain/lib/{feature}/failures/`  
- 使用 Freezed Union Types
- 對應不同錯誤情境

### 5. Repository 抽象介面
- 路徑：`packages/travel_domain/lib/{feature}/repositories/`
- 定義資料存取契約
- 返回 Either<Failure, Entity>

### 6. Use Cases
- 路徑：`packages/travel_domain/lib/{feature}/use_cases/`
- 編排業務流程
- 依賴注入 Repository
- 完整錯誤處理

### 7. 單元測試
- 路徑：`packages/travel_domain/test/`
- Entity 業務邏輯測試
- Value Object 驗證測試  
- Use Case 流程測試

請確保：
- 所有程式碼無框架依賴
- 完整的業務規則實作
- Either 錯誤處理模式
- 型別安全保證
```

### 模板二：Domain 擴充功能
```
基於現有的 travel_domain 實作，需要擴充 {NEW_FUNCTIONALITY}

## 擴充需求  
{具體新功能描述}

## 實作要求
請提供：
1. **新增 Entities/Value Objects** (如需要)
2. **擴充現有 Use Cases** 或新增
3. **Repository 介面更新**
4. **新的 Domain Services** (如需要)  
5. **對應的單元測試**

## 相容性要求
- 不能破壞現有 API
- 新功能必須通過現有測試
- 遵循既定的 Domain 設計模式

請保持 Domain 層的純淨性和無依賴原則。
```

---

## 🖥️ 前端實作模板

### 模板三：前端 Application + Infrastructure 層
```
基於 travel_domain package 的 {FEATURE_NAME}，現在實作前端的 Application 和 Infrastructure 層

## 前端需求
{具體前端需求描述}

## 實作要求
請提供以下完整程式碼：

### 1. DTO 定義 (前端特化)
路徑：`app/lib/features/{feature}/application/dtos/`
```dart
// 前端 DTO 設計原則：
@freezed
class UserDto with _$UserDto {
  const factory UserDto({
    required String id,
    required String email,
    required String displayName,     // UI 顯示用
    required String role,
    String? avatarUrl,               // 前端特有
    String? formattedLastLogin,      // 格式化時間
  }) = _UserDto;
  
  // Domain 轉換
  factory UserDto.fromDomain(User user) => UserDto(
    id: user.id.value,
    email: user.email.value,
    displayName: user.name.fullName,
    role: user.role.displayName,
  );
  
  factory UserDto.fromJson(Map<String, dynamic> json) => _$UserDtoFromJson(json);
}
```

### 2. Application Service (DTO 轉換 + Use Case 編排)
路徑：`app/lib/features/{feature}/application/services/`
```dart
class {Feature}ApplicationService {
  // 依賴注入 Domain Use Cases
  final {SomeUseCase} _{useCase};
  
  // DTO → Domain → DTO 完整流程
  Future<{ResultDto}> {methodName}({RequestDto} request) async {
    // 1. DTO → Domain 轉換
    final domainInput = {DomainType}.fromDto(request);
    
    // 2. 執行 Use Case
    final result = await _{useCase}.execute(domainInput);
    
    // 3. 錯誤處理 + Domain → DTO 轉換
    return result.fold(
      (failure) => throw ApplicationException.fromFailure(failure),
      (entity) => {ResultDto}.fromDomain(entity),
    );
  }
}
```

### 3. Repository 實作
路徑：`app/lib/features/{feature}/infrastructure/repositories/`
- 實作 Domain Repository 介面
- 整合 Remote + Local DataSource
- HTTP 錯誤轉 Domain Failure

### 4. Data Sources
路徑：`app/lib/features/{feature}/infrastructure/datasources/`
- RemoteDataSource (Dio HTTP 客戶端)
- LocalDataSource (SharedPreferences/Hive)

### 5. 依賴注入設定
路徑：`app/lib/di/`
- Riverpod Provider 註冊
- Repository 綁定
- Use Case 注入

請確保：
- DTO 完全隔離 Domain 和 UI 關注點
- Application Service 只負責編排和轉換
- 完整的錯誤處理鏈
- 型別安全的 Repository 實作
```

### 模板四：前端 Presentation 層  
```
基於前面的 Application 層實作，現在實作 Presentation 層

## UI 需求規格
{具體 UI 功能描述}

## 實作要求
請提供以下完整程式碼：

### 1. Riverpod Notifier (狀態管理)
路徑：`app/lib/features/{feature}/presentation/notifiers/`
```dart
@riverpod
class {Feature}Notifier extends _$${Feature}Notifier {
  @override
  Future<List<{EntityDto}>> build() async {
    // 透過 Application Service 取得資料
    return ref.read({feature}ApplicationServiceProvider).getAll();
  }
  
  Future<void> {actionMethod}({ParamDto} params) async {
    state = const AsyncLoading();
    
    try {
      // 呼叫 Application Service
      final result = await ref.read({feature}ApplicationServiceProvider)
          .{actionMethod}(params);
          
      // 更新狀態
      state = AsyncData(result);
    } catch (e, stackTrace) {
      state = AsyncError(e, stackTrace);
    }
  }
}
```

### 2. Provider 定義
路徑：`app/lib/features/{feature}/presentation/providers/`
- 使用 riverpod_generator
- 依賴注入整合
- 衍生 Provider (搜尋、篩選)

### 3. Screen Widget  
路徑：`app/lib/features/{feature}/presentation/screens/`
```dart
class {Feature}Screen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch({feature}NotifierProvider);
    
    return Scaffold(
      appBar: AppBar(title: Text('{Feature}')),
      body: state.when(
        loading: () => const LoadingIndicator(),
        error: (error, stack) => ErrorDisplay(error: error),
        data: (data) => {Feature}ListView(items: data),
      ),
    );
  }
}
```

### 4. UI Components
路徑：`app/lib/features/{feature}/presentation/widgets/`
- 可復用 Widget 元件
- Material 3 設計系統
- 響應式佈局

### 5. 路由整合  
更新 `app/lib/shared/presentation/routes/app_routes.dart`
- go_router 路徑配置
- 路由守衛設定

請確保：
- 完整的狀態處理 (loading/error/success)
- Material 3 主題一致性
- 響應式設計支援
- 可訪問性 (Accessibility) 支援
```

---

## 🔧 後端實作模板

### 模板五：後端 Serverpod 實作
```
基於 travel_domain package 的 {FEATURE_NAME}，現在實作 Serverpod 後端

## 後端需求
{具體 API 需求描述}

## 實作要求
請提供以下完整程式碼：

### 1. Serverpod Model 定義
路徑：`api/lib/src/protocol/`
```yaml
# {feature}_models.yaml
class: {Feature}Response
table: {feature}s
fields:
  id: int?
  email: String
  firstName: String  
  lastName: String
  role: String
  createdAt: DateTime
```

### 2. Repository 實作
路徑：`api/lib/src/repositories/`
```dart
class {Feature}RepositoryImpl implements {Feature}Repository {
  final Session _session;
  
  @override
  Future<Either<{Feature}Failure, {Entity}>> findById({EntityId} id) async {
    try {
      final model = await {Feature}Response.db.findById(_session, id.value);
      if (model == null) {
        return Left({Feature}Failure.notFound());
      }
      return Right(_toDomain(model));
    } catch (e) {
      return Left({Feature}Failure.database(e.toString()));
    }
  }
  
  // Model ↔ Domain 轉換
  {Entity} _toDomain({Feature}Response model) => {Entity}(
    id: {EntityId}(model.id!),
    // ... 其他欄位轉換
  );
}
```

### 3. Endpoints 實作
路徑：`api/lib/src/endpoints/`
```dart
class {Feature}Endpoint extends Endpoint {
  final {SomeUseCase} _{useCase};
  
  Future<{Response}> {methodName}(
    Session session,
    {Request} request,
  ) async {
    try {
      // 1. Request → Domain 轉換
      final domainInput = _toDomainInput(request);
      
      // 2. 執行 Domain Use Case  
      final result = await _{useCase}.execute(domainInput);
      
      // 3. Domain → Response 轉換
      return result.fold(
        (failure) => throw EndpointException.fromFailure(failure),
        (entity) => _toResponse(entity),
      );
    } catch (e) {
      throw EndpointException(e.toString());
    }
  }
}
```

### 4. 依賴注入設定
路徑：`api/lib/server.dart`
- Repository 實作註冊
- Use Case 依賴綁定

### 5. 資料庫遷移
路徑：`api/migrations/`
- 資料表結構定義
- 索引和約束設定

請確保：
- API 符合 RESTful 原則
- 完整的錯誤處理和日誌
- 資料庫交易管理
- API 文檔自動生成
```

---

## 🧪 測試實作模板

### 模板六：完整測試套件
```
基於完整的功能實作，現在建立對應的測試套件

## 測試要求
請提供以下測試程式碼：

### 1. Domain 層測試
路徑：`packages/travel_domain/test/`
```dart
group('{Entity} Domain Tests', () {
  test('should validate business rules correctly', () {
    final entity = {Entity}(/* test data */);
    
    expect(entity.{businessMethod}(), isTrue);
  });
  
  test('should handle invalid input properly', () {
    expect(
      () => {ValueObject}('invalid_input'),
      throwsA(isA<ValidationException>()),
    );
  });
});

group('{UseCase} Tests', () {
  late Mock{Repository} mockRepository;
  late {UseCase} useCase;
  
  setUp(() {
    mockRepository = Mock{Repository}();
    useCase = {UseCase}(mockRepository);
  });
  
  test('should return success when data is valid', () async {
    when(mockRepository.{method}(any))
      .thenAnswer((_) async => Right(mockEntity));
    
    final result = await useCase.execute(validInput);
    
    expect(result.isRight(), isTrue);
  });
});
```

### 2. Application Service 測試
路徑：`app/test/features/{feature}/application/`
- DTO 轉換正確性測試
- Application Service 流程測試
- 錯誤處理測試

### 3. Repository 實作測試  
路徑：`app/test/features/{feature}/infrastructure/`
- HTTP 客戶端整合測試
- 本地存儲測試
- 錯誤轉換測試

### 4. Widget 測試
路徑：`app/test/features/{feature}/presentation/`
```dart
group('{Feature}Screen Widget Tests', () {
  testWidgets('should display loading state', (tester) async {
    await tester.pumpWidget(
      createTestWidget(
        child: {Feature}Screen(),
        overrides: [
          {feature}NotifierProvider.overrideWith(
            () => AsyncLoading(),
          ),
        ],
      ),
    );
    
    expect(find.byType(LoadingIndicator), findsOneWidget);
  });
});
```

### 5. Integration 測試
路徑：`app/integration_test/`
- 完整用戶流程測試
- API 整合測試
- 狀態管理整合測試

### 6. E2E 測試 (Patrol)
```dart
patrolTest('complete user journey', (PatrolTester $) async {
  await $.pumpWidgetAndSettle(MyApp());
  
  // 登入流程
  await $(#emailField).enterText('test@example.com');
  await $(#passwordField).enterText('password123');
  await $(#loginButton).tap();
  
  // 驗證結果
  await $.pump();
  expect($(#welcomeMessage).visible, isTrue);
});
```

請確保：
- Domain 層 95%+ 測試覆蓋率
- 所有 Use Case 有完整測試
- UI 元件互動測試完整  
- 整合測試涵蓋主要流程
```

---

## 🔄 迭代開發模板

### 模板七：問題排除與優化
```
針對 {COMPONENT_NAME} 遇到以下問題：

## 問題描述
{具體問題或錯誤}

## 錯誤訊息
{完整錯誤日誌}

## 相關程式碼
{貼上問題程式碼片段}

## 專案架構提醒
- 使用共用 travel_domain package
- DTO 邊界隔離原則
- Either<Failure, Success> 錯誤處理
- Riverpod 狀態管理

請提供：
1. **根本原因分析** - 基於 DDD 架構原則
2. **修正方案** - 保持架構一致性
3. **最佳實踐建議** - 避免類似問題
4. **測試驗證** - 確保修正正確

請保持共用 Domain + DTO 邊界的架構原則。
```

### 模板八：功能擴展
```
基於現有的 {EXISTING_FEATURE}，需要新增 {NEW_FUNCTIONALITY}

## 擴展需求
{新功能詳細描述}

## 現有架構狀況
{描述當前實作狀況}

## 擴展要求
請提供：
1. **Domain 層擴展** (travel_domain package)
   - 新 Entities/Value Objects
   - 擴充或新增 Use Cases
   - Repository 介面更新

2. **前端擴展** (app)
   - 新的 DTO 定義
   - Application Service 更新
   - UI 元件實作

3. **後端擴展** (server) 
   - API Endpoints
   - Repository 實作
   - 資料庫遷移

4. **測試更新**
   - 所有層級的測試
   - 整合測試擴展

## 相容性要求
- 不能破壞現有 API 契約
- 保持 Domain 層純淨性
- DTO 邊界隔離原則
- 通過所有現有測試

請遵循已建立的架構模式和程式碼風格。
```

---

## 🎯 使用指南與檢查清單

### 開發階段檢查清單
- [ ] **Domain 層純淨** - 無框架依賴
- [ ] **DTO 邊界清晰** - 前後端適配完整
- [ ] **Use Case 單一職責** - 業務流程清晰
- [ ] **Repository 抽象正確** - 介面與實作分離
- [ ] **錯誤處理完整** - Either 模式一致

### 測試品質檢查清單  
- [ ] **Domain 測試 95%+** - 業務邏輯全覆蓋
- [ ] **Use Case 流程測試** - 成功與失敗情境
- [ ] **DTO 轉換測試** - 資料一致性驗證
- [ ] **Widget 互動測試** - UI 狀態處理
- [ ] **整合測試完整** - 端到端流程驗證

### 架構品質檢查清單
- [ ] **分層職責清晰** - 每層只處理自己的關注點  
- [ ] **依賴方向正確** - 外層依賴內層，不反向
- [ ] **介面契約穩定** - Repository 和 DTO 定義明確
- [ ] **共用 Domain 一致** - 前後端業務邏輯統一
- [ ] **程式碼可維護** - 清晰的架構和命名

---

## 🚀 推薦實作順序

### Phase 1: Domain 核心 (1-2 天)
1. 建立 travel_domain package
2. 實作認證相關的 Domain 層
3. 完整的 Domain 測試套件

### Phase 2: 前端整合 (2-3 天)
1. Application Service + DTO 設計  
2. Infrastructure Repository 實作
3. Presentation Riverpod 整合
4. UI 元件和 Screen 實作

### Phase 3: 後端整合 (2-3 天)  
1. Serverpod Model 定義
2. Repository 實作 + 資料庫
3. API Endpoints 實作
4. 前後端整合測試

### Phase 4: 測試完善 (1 天)
1. Widget 和整合測試
2. E2E 用戶流程測試
3. 效能和安全測試

這套模板確保了完整的共用 Domain + DTO 邊界架構實作，每個階段都有明確的品質標準和檢查要點。
