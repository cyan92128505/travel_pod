# 旅行社管理平台 - 目錄結構設計

## 專案根目錄結構

```
travel_platform/
├── packages/
│   └── travel_domain/                 # 共用 Domain Package
├── travel_app/                        # Flutter Web 前端
├── travel_api/                        # Serverpod 後端
├── melos.yaml                         # 多包管理配置
└── README.md
```

## 1. travel_domain/ (共用 Domain Package)

```
packages/travel_domain/
├── lib/
│   ├── src/
│   │   ├── core/                      # 核心抽象
│   │   │   ├── failures/
│   │   │   │   ├── auth_failure.dart
│   │   │   │   ├── user_failure.dart
│   │   │   │   └── failure.dart
│   │   │   ├── value_objects/
│   │   │   │   ├── email.dart
│   │   │   │   ├── password.dart
│   │   │   │   ├── user_name.dart
│   │   │   │   └── value_object.dart
│   │   │   └── either.dart            # Result 型別
│   │   │
│   │   ├── features/
│   │   │   └── auth/                  # 認證領域
│   │   │       ├── entities/
│   │   │       │   ├── user.dart
│   │   │       │   └── auth_session.dart
│   │   │       ├── repositories/
│   │   │       │   ├── auth_repository.dart
│   │   │       │   └── user_repository.dart
│   │   │       ├── use_cases/
│   │   │       │   ├── sign_in_use_case.dart
│   │   │       │   ├── sign_up_use_case.dart
│   │   │       │   ├── sign_out_use_case.dart
│   │   │       │   ├── get_user_profile_use_case.dart
│   │   │       │   └── update_user_profile_use_case.dart
│   │   │       └── services/
│   │   │           └── auth_domain_service.dart
│   │   │
│   │   └── shared/                    # 共用領域邏輯
│   │       ├── constants/
│   │       │   └── domain_constants.dart
│   │       └── extensions/
│   │           └── string_extensions.dart
│   │
│   └── travel_domain.dart             # 公開 API
├── test/                              # Domain 單元測試
├── pubspec.yaml
└── README.md
```

## 2. travel_app/ (Flutter Web 前端)

```
travel_app/
├── lib/
│   ├── src/
│   │   ├── core/                      # 核心基礎設施
│   │   │   ├── di/
│   │   │   │   └── injection.dart     # 依賴注入配置
│   │   │   ├── router/
│   │   │   │   ├── app_router.dart
│   │   │   │   └── route_paths.dart
│   │   │   ├── theme/
│   │   │   │   └── app_theme.dart
│   │   │   └── utils/
│   │   │       ├── constants.dart
│   │   │       └── extensions.dart
│   │   │
│   │   ├── features/
│   │   │   └── auth/                  # 認證功能模組
│   │   │       ├── application/       # Application Services
│   │   │       │   ├── dtos/
│   │   │       │   │   ├── sign_in_dto.dart
│   │   │       │   │   ├── sign_up_dto.dart
│   │   │       │   │   └── user_profile_dto.dart
│   │   │       │   └── services/
│   │   │       │       ├── auth_application_service.dart
│   │   │       │       └── user_application_service.dart
│   │   │       │
│   │   │       ├── infrastructure/    # Infrastructure 實作
│   │   │       │   ├── datasources/
│   │   │       │   │   ├── auth_remote_datasource.dart
│   │   │       │   │   └── user_remote_datasource.dart
│   │   │       │   ├── repositories/
│   │   │       │   │   ├── auth_repository_impl.dart
│   │   │       │   │   └── user_repository_impl.dart
│   │   │       │   └── mappers/
│   │   │       │       ├── user_mapper.dart
│   │   │       │       └── auth_mapper.dart
│   │   │       │
│   │   │       └── presentation/      # UI 層
│   │   │           ├── providers/
│   │   │           │   ├── auth_providers.dart
│   │   │           │   └── user_providers.dart
│   │   │           ├── pages/
│   │   │           │   ├── sign_in_page.dart
│   │   │           │   ├── sign_up_page.dart
│   │   │           │   └── profile_page.dart
│   │   │           ├── widgets/
│   │   │           │   ├── auth_form.dart
│   │   │           │   ├── user_avatar.dart
│   │   │           │   └── loading_overlay.dart
│   │   │           └── states/
│   │   │               ├── auth_state.dart
│   │   │               └── user_state.dart
│   │   │
│   │   └── shared/                    # 共用前端組件
│   │       ├── widgets/
│   │       │   ├── app_button.dart
│   │       │   ├── app_text_field.dart
│   │       │   └── error_widget.dart
│   │       └── utils/
│   │           └── ui_helpers.dart
│   │
│   ├── app.dart                       # App 根組件
│   └── main.dart                      # 應用入口
├── web/                               # Web 配置
├── test/                              # 測試
├── pubspec.yaml
└── README.md
```

## 3. travel_api/ (Serverpod 後端)

```
travel_api/
├── lib/
│   ├── src/
│   │   ├── endpoints/                 # API Endpoints
│   │   │   ├── auth_endpoint.dart
│   │   │   └── user_endpoint.dart
│   │   │
│   │   ├── features/
│   │   │   └── auth/                  # 認證功能模組
│   │   │       ├── application/       # Application Services  
│   │   │       │   ├── dtos/
│   │   │       │   │   ├── sign_in_request_dto.dart
│   │   │       │   │   ├── sign_in_response_dto.dart
│   │   │       │   │   ├── sign_up_request_dto.dart
│   │   │       │   │   ├── user_response_dto.dart
│   │   │       │   │   └── update_user_request_dto.dart
│   │   │       │   └── services/
│   │   │       │       ├── auth_application_service.dart
│   │   │       │       └── user_application_service.dart
│   │   │       │
│   │   │       └── infrastructure/    # Infrastructure 實作
│   │   │           ├── datasources/
│   │   │           │   ├── auth_database_datasource.dart
│   │   │           │   └── user_database_datasource.dart
│   │   │           ├── repositories/
│   │   │           │   ├── auth_repository_impl.dart
│   │   │           │   └── user_repository_impl.dart
│   │   │           ├── models/
│   │   │           │   ├── user_model.dart
│   │   │           │   └── auth_session_model.dart
│   │   │           └── mappers/
│   │   │               ├── user_mapper.dart
│   │   │               └── auth_mapper.dart
│   │   │
│   │   ├── core/                      # 核心基礎設施
│   │   │   ├── di/
│   │   │   │   └── injection.dart
│   │   │   ├── database/
│   │   │   │   └── connection.dart
│   │   │   └── middleware/
│   │   │       └── auth_middleware.dart
│   │   │
│   │   └── generated/                 # Serverpod 生成檔案
│   │       └── protocol.dart
│   │
│   └── server.dart                    # Server 入口
├── config/                            # 配置檔案
├── migrations/                        # 資料庫遷移
├── test/                             # 測試
├── pubspec.yaml
└── README.md
```

## 關鍵設計原則

### 1. 共用 Domain Package
- 純業務邏輯，無技術框架依賴
- 定義 Repository 抽象介面
- 包含所有 Use Cases 和 Domain Services

### 2. DTO 邊界隔離
- **前端 DTO**: UI 友好格式 (displayName, formattedDate)
- **後端 DTO**: API 標準格式 (snake_case, 原始數值)
- Application Service 負責轉換

### 3. Feature-First 組織
- 每個功能模組包含完整的三層架構
- Application → Infrastructure → Presentation
- 清楚的職責分離

### 4. 依賴方向
```
Presentation → Application → Infrastructure
     ↓              ↓              ↓
          Domain (共用核心)
```

這個架構設計確保了：
- 🎯 **業務邏輯純淨**：Domain 層無技術依賴
- 🔄 **程式碼重用**：前後端共用業務邏輯  
- 🧪 **測試友好**：每層都可獨立測試
- 🚀 **可擴展性**：新功能遵循相同模式
- 🛡️ **型別安全**：完整的 Freezed + Riverpod Generator

您覺得這個目錄結構設計如何？有需要調整的地方嗎？