# Configuration Management Service

## Responsibility
Centralized configuration management for all QuantiVista services with dynamic updates, feature flags, environment-specific settings, and audit trails. Provides secure configuration distribution and real-time updates without service restarts.

## Technology Stack
- **Language**: Go + etcd + Vault integration
- **Libraries**: etcd client, Vault API, configuration validation libraries
- **Scaling**: Horizontal with etcd clustering
- **NFRs**: P99 config retrieval < 10ms, 99.99% availability, secure secret management

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ConfigType {
    APPLICATION_CONFIG,
    FEATURE_FLAG,
    SECRET,
    ENVIRONMENT_VARIABLE,
    BUSINESS_RULE
}

enum Environment {
    DEVELOPMENT,
    STAGING,
    PRODUCTION,
    TEST
}

// Data Models
struct Configuration {
    config_key: String
    config_value: Any
    config_type: ConfigType
    environment: Environment
    service_name: String
    description: String
    is_encrypted: Boolean
    version: Integer
    created_at: DateTime
    updated_at: DateTime
}

struct FeatureFlag {
    flag_name: String
    enabled: Boolean
    rollout_percentage: Float
    target_services: List<String>
    conditions: Map<String, Any>
    description: String
}

struct ConfigurationUpdate {
    config_key: String
    old_value: Any
    new_value: Any
    updated_by: String
    reason: String
    rollback_available: Boolean
}

// REST API Endpoints
GET /api/v1/config/{service_name}
    Parameters: environment
    Response: Map<String, Any>

GET /api/v1/config/{service_name}/{config_key}
    Parameters: environment
    Response: Configuration

PUT /api/v1/config/{service_name}/{config_key}
    Request: ConfigurationUpdate
    Response: Configuration

GET /api/v1/feature-flags/{service_name}
    Response: List<FeatureFlag>

PUT /api/v1/feature-flags/{flag_name}
    Request: FeatureFlagUpdate
    Response: FeatureFlag

POST /api/v1/config/rollback
    Request: RollbackRequest
    Response: RollbackResult

WebSocket /api/v1/config/watch/{service_name}
    Response: Stream<ConfigurationUpdate>
```

### Event Output
```pseudo
Event configuration_updated {
    event_id: String
    timestamp: DateTime
    configuration_updated: ConfigurationUpdatedData
}

struct ConfigurationUpdatedData {
    config_key: String
    service_name: String
    environment: String
    old_value: Float
    new_value: Float
    updated_by: String
    reason: String
    version: Integer
    rollback_available: Boolean
    affected_instances: List<String>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    configuration_updated: {
        config_key: "portfolio_management.risk_threshold",
        service_name: "portfolio_management_service",
        environment: "PRODUCTION",
        old_value: 0.05,
        new_value: 0.04,
        updated_by: "risk_manager_001",
        reason: "Reduce risk exposure due to market volatility",
        version: 15,
        rollback_available: true,
        affected_instances: [
            "portfolio-mgmt-pod-001",
            "portfolio-mgmt-pod-002",
            "portfolio-mgmt-pod-003"
        ]
    }
}
```

## Database Schema

### etcd (Primary Storage)
```pseudo
// Configuration stored as key-value pairs in etcd
// Key format: /quantivista/{environment}/{service_name}/{config_key}
// Example: /quantivista/production/portfolio_management/risk_threshold

struct EtcdConfigEntry {
    key: String
    value: String  // JSON serialized
    version: Integer
    create_revision: Integer
    mod_revision: Integer
    lease: Optional<Integer>
}
```

### PostgreSQL (Audit Trail)
```pseudo
Table configuration_audit {
    id: UUID (primary key, auto-generated)
    config_key: String (required, max_length: 200)
    service_name: String (required, max_length: 100)
    environment: String (required, max_length: 20)
    old_value: JSON
    new_value: JSON (required)
    updated_by: String (required, max_length: 100)
    reason: String
    version: Integer (required)
    created_at: Timestamp (default: now)
}

Table feature_flags {
    id: UUID (primary key, auto-generated)
    flag_name: String (required, unique, max_length: 100)
    enabled: Boolean (default: false)
    rollout_percentage: Decimal (default: 0.0, precision: 5, scale: 2)
    target_services: JSON
    conditions: JSON
    description: String
    created_by: String (max_length: 100)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}
```

## External Dependencies

### **Infrastructure Components:**
- **etcd Cluster**: Distributed key-value store for configuration data
- **HashiCorp Vault**: Secret management and encryption
- **Kubernetes ConfigMaps**: Integration with K8s native configuration
- **Environment Variables**: Fallback configuration source

### **Security Integration:**
- **RBAC System**: Role-based access control for configuration changes
- **Audit Logging**: Comprehensive audit trail for compliance
- **Encryption**: At-rest and in-transit encryption for sensitive data

## Implementation Estimation

### Priority: **HIGH** (Foundation for all services)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Configuration Engine
- Go service setup with etcd integration
- Basic configuration CRUD operations
- Vault integration for secrets
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Feature flag management
- Real-time configuration updates
- Rollback capabilities
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Integration & Security
- Integration with all QuantiVista services
- Security hardening and audit trails
- Performance optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 DevOps specialist)
### Dependencies: etcd cluster, Vault, Kubernetes, RBAC system

### Success Criteria:
- P99 config retrieval < 10ms
- 99.99% availability
- Real-time configuration updates
- Secure secret management
- Comprehensive audit trails
