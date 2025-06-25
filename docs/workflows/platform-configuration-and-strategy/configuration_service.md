# Configuration Service

## Purpose and Boundaries

### Purpose
The Configuration Service provides centralized configuration management for the entire QuantiVista platform with versioning, environment-specific settings, and dynamic updates. It serves as the single source of truth for all configuration parameters, ensuring consistency across services and environments.

### Strict Boundaries
- **Responsible for**: Configuration storage, retrieval, versioning, validation, environment-specific settings, feature flags, and configuration change notifications
- **Not responsible for**: Business logic, trading strategy definitions, user management, or application deployment
- **Interfaces with**: All platform services that require configuration, with special integration with the Strategy Management Service

## Place in Workflow

The Configuration Service is a foundational component in the Configuration and Strategy Management Workflow:

1. **Stores and manages** all platform configuration settings
2. **Provides environment-specific configurations** for development, testing, staging, and production
3. **Validates configuration changes** against schema definitions
4. **Maintains version history** of all configuration changes
5. **Supports rollback** to previous configuration versions
6. **Notifies services** of configuration changes via events
7. **Manages feature flags** for controlled feature rollouts
8. **Securely stores sensitive configuration** values

## API Design (API-First)

### REST API

#### Configuration Management Endpoints

```yaml
openapi: 3.0.3
info:
  title: Configuration Service API
  version: 1.0.0
  description: API for managing platform-wide configuration settings
paths:
  /configs:
    get:
      summary: List configurations
      operationId: listConfigurations
      tags: [Configurations]
      parameters:
        - name: application
          in: query
          schema:
            type: string
        - name: environment
          in: query
          schema:
            type: string
            enum: [DEV, TEST, STAGING, PROD]
        - name: page
          in: query
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of configurations
          content:
            application/json:
              schema:
                type: object
                properties:
                  content:
                    type: array
                    items:
                      $ref: '#/components/schemas/ConfigurationSummary'
                  page:
                    type: integer
                  size:
                    type: integer
                  totalElements:
                    type: integer
                  totalPages:
                    type: integer
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create a new configuration
      operationId: createConfiguration
      tags: [Configurations]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateConfigurationRequest'
      responses:
        '201':
          description: Configuration created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /configs/{configId}:
    get:
      summary: Get configuration by ID
      operationId: getConfiguration
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Configuration details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationResponse'
        '404':
          description: Configuration not found
        '401':
          $ref: '#/components/responses/Unauthorized'

    put:
      summary: Update configuration
      operationId: updateConfiguration
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateConfigurationRequest'
      responses:
        '200':
          description: Configuration updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          description: Configuration not found
        '401':
          $ref: '#/components/responses/Unauthorized'

    delete:
      summary: Delete configuration
      operationId: deleteConfiguration
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
      responses:
        '204':
          description: Configuration deleted successfully
        '404':
          description: Configuration not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /configs/{configId}/versions:
    get:
      summary: Get configuration version history
      operationId: getConfigurationVersions
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Configuration version history
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/ConfigurationVersion'
        '404':
          description: Configuration not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /configs/{configId}/versions/{versionId}:
    get:
      summary: Get specific configuration version
      operationId: getConfigurationVersion
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
        - name: versionId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Configuration version details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationVersion'
        '404':
          description: Configuration or version not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /configs/{configId}/rollback/{versionId}:
    post:
      summary: Rollback to specific configuration version
      operationId: rollbackConfiguration
      tags: [Configurations]
      parameters:
        - name: configId
          in: path
          required: true
          schema:
            type: string
        - name: versionId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Configuration rolled back successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationResponse'
        '404':
          description: Configuration or version not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /configs/lookup:
    get:
      summary: Look up configuration by key
      operationId: lookupConfiguration
      tags: [Configurations]
      parameters:
        - name: key
          in: query
          required: true
          schema:
            type: string
        - name: application
          in: query
          required: true
          schema:
            type: string
        - name: environment
          in: query
          required: true
          schema:
            type: string
            enum: [DEV, TEST, STAGING, PROD]
      responses:
        '200':
          description: Configuration value
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConfigurationValue'
        '404':
          description: Configuration not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /feature-flags:
    get:
      summary: List feature flags
      operationId: listFeatureFlags
      tags: [FeatureFlags]
      parameters:
        - name: application
          in: query
          schema:
            type: string
        - name: environment
          in: query
          schema:
            type: string
            enum: [DEV, TEST, STAGING, PROD]
        - name: page
          in: query
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of feature flags
          content:
            application/json:
              schema:
                type: object
                properties:
                  content:
                    type: array
                    items:
                      $ref: '#/components/schemas/FeatureFlag'
                  page:
                    type: integer
                  size:
                    type: integer
                  totalElements:
                    type: integer
                  totalPages:
                    type: integer
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create a new feature flag
      operationId: createFeatureFlag
      tags: [FeatureFlags]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateFeatureFlagRequest'
      responses:
        '201':
          description: Feature flag created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FeatureFlag'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /feature-flags/{flagId}:
    get:
      summary: Get feature flag by ID
      operationId: getFeatureFlag
      tags: [FeatureFlags]
      parameters:
        - name: flagId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Feature flag details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FeatureFlag'
        '404':
          description: Feature flag not found
        '401':
          $ref: '#/components/responses/Unauthorized'

    put:
      summary: Update feature flag
      operationId: updateFeatureFlag
      tags: [FeatureFlags]
      parameters:
        - name: flagId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateFeatureFlagRequest'
      responses:
        '200':
          description: Feature flag updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FeatureFlag'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          description: Feature flag not found
        '401':
          $ref: '#/components/responses/Unauthorized'

  /feature-flags/check:
    get:
      summary: Check if feature flag is enabled
      operationId: checkFeatureFlag
      tags: [FeatureFlags]
      parameters:
        - name: key
          in: query
          required: true
          schema:
            type: string
        - name: application
          in: query
          required: true
          schema:
            type: string
        - name: environment
          in: query
          required: true
          schema:
            type: string
            enum: [DEV, TEST, STAGING, PROD]
        - name: userId
          in: query
          schema:
            type: string
      responses:
        '200':
          description: Feature flag status
          content:
            application/json:
              schema:
                type: object
                properties:
                  key:
                    type: string
                  enabled:
                    type: boolean
                  value:
                    type: object
        '404':
          description: Feature flag not found
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    ConfigurationSummary:
      type: object
      properties:
        id:
          type: string
          description: Configuration ID
        key:
          type: string
          description: Configuration key
        application:
          type: string
          description: Application name
        environment:
          type: string
          enum: [DEV, TEST, STAGING, PROD]
          description: Environment
        description:
          type: string
          description: Configuration description
        valueType:
          type: string
          enum: [STRING, NUMBER, BOOLEAN, JSON, SECRET]
          description: Value type
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp

    CreateConfigurationRequest:
      type: object
      required: [key, application, environment, valueType, value]
      properties:
        key:
          type: string
          description: Configuration key
        application:
          type: string
          description: Application name
        environment:
          type: string
          enum: [DEV, TEST, STAGING, PROD]
          description: Environment
        description:
          type: string
          description: Configuration description
        valueType:
          type: string
          enum: [STRING, NUMBER, BOOLEAN, JSON, SECRET]
          description: Value type
        value:
          type: string
          description: Configuration value
        schema:
          type: string
          description: JSON Schema for validation (optional)
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization

    UpdateConfigurationRequest:
      type: object
      properties:
        description:
          type: string
          description: Configuration description
        value:
          type: string
          description: Configuration value
        schema:
          type: string
          description: JSON Schema for validation
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization

    ConfigurationResponse:
      type: object
      properties:
        id:
          type: string
          description: Configuration ID
        key:
          type: string
          description: Configuration key
        application:
          type: string
          description: Application name
        environment:
          type: string
          enum: [DEV, TEST, STAGING, PROD]
          description: Environment
        description:
          type: string
          description: Configuration description
        valueType:
          type: string
          enum: [STRING, NUMBER, BOOLEAN, JSON, SECRET]
          description: Value type
        value:
          type: string
          description: Configuration value (masked for SECRET type)
        schema:
          type: string
          description: JSON Schema for validation
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization
        version:
          type: integer
          description: Current version number
        createdBy:
          type: string
          description: User who created the configuration
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        updatedBy:
          type: string
          description: User who last updated the configuration
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp

    ConfigurationVersion:
      type: object
      properties:
        id:
          type: string
          description: Version ID
        configId:
          type: string
          description: Configuration ID
        version:
          type: integer
          description: Version number
        value:
          type: string
          description: Configuration value at this version
        createdBy:
          type: string
          description: User who created this version
        createdAt:
          type: string
          format: date-time
          description: Version creation timestamp
        changeDescription:
          type: string
          description: Description of changes in this version

    ConfigurationValue:
      type: object
      properties:
        key:
          type: string
          description: Configuration key
        value:
          type: string
          description: Configuration value
        valueType:
          type: string
          enum: [STRING, NUMBER, BOOLEAN, JSON, SECRET]
          description: Value type
        version:
          type: integer
          description: Current version number

    CreateFeatureFlagRequest:
      type: object
      required: [key, application, environment, description]
      properties:
        key:
          type: string
          description: Feature flag key
        application:
          type: string
          description: Application name
        environment:
          type: string
          enum: [DEV, TEST, STAGING, PROD]
          description: Environment
        description:
          type: string
          description: Feature flag description
        enabled:
          type: boolean
          default: false
          description: Whether the feature is enabled
        rules:
          type: array
          items:
            $ref: '#/components/schemas/FeatureFlagRule'
          description: Targeting rules
        value:
          type: object
          description: Additional values for the feature flag
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization

    UpdateFeatureFlagRequest:
      type: object
      properties:
        description:
          type: string
          description: Feature flag description
        enabled:
          type: boolean
          description: Whether the feature is enabled
        rules:
          type: array
          items:
            $ref: '#/components/schemas/FeatureFlagRule'
          description: Targeting rules
        value:
          type: object
          description: Additional values for the feature flag
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization

    FeatureFlag:
      type: object
      properties:
        id:
          type: string
          description: Feature flag ID
        key:
          type: string
          description: Feature flag key
        application:
          type: string
          description: Application name
        environment:
          type: string
          enum: [DEV, TEST, STAGING, PROD]
          description: Environment
        description:
          type: string
          description: Feature flag description
        enabled:
          type: boolean
          description: Whether the feature is enabled
        rules:
          type: array
          items:
            $ref: '#/components/schemas/FeatureFlagRule'
          description: Targeting rules
        value:
          type: object
          description: Additional values for the feature flag
        tags:
          type: array
          items:
            type: string
          description: Tags for categorization
        version:
          type: integer
          description: Current version number
        createdBy:
          type: string
          description: User who created the feature flag
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        updatedBy:
          type: string
          description: User who last updated the feature flag
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp

    FeatureFlagRule:
      type: object
      properties:
        attribute:
          type: string
          description: User attribute to match (e.g., userId, role, region)
        operator:
          type: string
          enum: [EQUALS, NOT_EQUALS, CONTAINS, NOT_CONTAINS, STARTS_WITH, ENDS_WITH, MATCHES, IN, NOT_IN]
          description: Comparison operator
        values:
          type: array
          items:
            type: string
          description: Values to compare against
        percentage:
          type: integer
          minimum: 0
          maximum: 100
          description: Percentage of users who should see the feature (for gradual rollout)

  responses:
    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
              errors:
                type: array
                items:
                  type: object
                  properties:
                    field:
                      type: string
                    message:
                      type: string

    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
                example: Authentication required
```

### gRPC API (Internal Services)

```protobuf
syntax = "proto3";

package config.v1;

import "google/protobuf/timestamp.proto";
import "google/protobuf/struct.proto";

service ConfigurationService {
  // Get configuration by key
  rpc GetConfiguration(GetConfigurationRequest) returns (ConfigurationResponse);

  // Get multiple configurations by keys
  rpc GetConfigurations(GetConfigurationsRequest) returns (GetConfigurationsResponse);

  // Check if feature flag is enabled
  rpc CheckFeatureFlag(CheckFeatureFlagRequest) returns (CheckFeatureFlagResponse);

  // Stream configuration updates
  rpc StreamConfigurationUpdates(StreamConfigurationUpdatesRequest) returns (stream ConfigurationUpdate);
}

message GetConfigurationRequest {
  string key = 1;
  string application = 2;
  string environment = 3;
}

message ConfigurationResponse {
  string key = 1;
  string value = 2;
  ValueType value_type = 3;
  int32 version = 4;
  google.protobuf.Timestamp updated_at = 5;
}

message GetConfigurationsRequest {
  repeated string keys = 1;
  string application = 2;
  string environment = 3;
}

message GetConfigurationsResponse {
  repeated ConfigurationResponse configurations = 1;
}

message CheckFeatureFlagRequest {
  string key = 1;
  string application = 2;
  string environment = 3;
  string user_id = 4;
  map<string, string> attributes = 5;
}

message CheckFeatureFlagResponse {
  string key = 1;
  bool enabled = 2;
  google.protobuf.Struct value = 3;
  string reason = 4;
}

message StreamConfigurationUpdatesRequest {
  string application = 1;
  string environment = 2;
  repeated string keys = 3;
}

message ConfigurationUpdate {
  string key = 1;
  string value = 2;
  ValueType value_type = 3;
  int32 version = 4;
  UpdateType update_type = 5;
  google.protobuf.Timestamp updated_at = 6;
}

enum ValueType {
  VALUE_TYPE_UNSPECIFIED = 0;
  VALUE_TYPE_STRING = 1;
  VALUE_TYPE_NUMBER = 2;
  VALUE_TYPE_BOOLEAN = 3;
  VALUE_TYPE_JSON = 4;
  VALUE_TYPE_SECRET = 5;
}

enum Environment {
  ENVIRONMENT_UNSPECIFIED = 0;
  ENVIRONMENT_DEV = 1;
  ENVIRONMENT_TEST = 2;
  ENVIRONMENT_STAGING = 3;
  ENVIRONMENT_PROD = 4;
}

enum UpdateType {
  UPDATE_TYPE_UNSPECIFIED = 0;
  UPDATE_TYPE_CREATED = 1;
  UPDATE_TYPE_UPDATED = 2;
  UPDATE_TYPE_DELETED = 3;
}
```

## Data Model

### Core Entities

#### Configuration
Represents a configuration setting with versioning.

```pseudo
// Enumerations
enum Environment {
    DEVELOPMENT,
    TESTING,
    STAGING,
    PRODUCTION
}

enum ValueType {
    STRING,
    INTEGER,
    FLOAT,
    BOOLEAN,
    JSON,
    ARRAY
}

// Data Models
struct Configuration {
    id: String
    key: String
    application: String
    environment: Environment
    description: String
    value_type: ValueType
    value: String
    schema: String
    tags: Set<String>
    version: Integer
    created_by: String
    created_at: DateTime
    updated_by: String
    updated_at: DateTime
    versions: List<ConfigurationVersion>
}
```

#### ConfigurationVersion
Represents a specific version of a configuration.

```pseudo
struct ConfigurationVersion {
    id: String
    configuration_id: String
    version: Integer
    value: String
    created_by: String
    created_at: DateTime
    change_description: String
}
```

#### FeatureFlag
Represents a feature flag for controlled feature rollout.

```pseudo
struct FeatureFlag {
    id: String
    key: String
    application: String
    environment: Environment
    description: String
    enabled: Boolean
    rules: List<FeatureFlagRule>
    value: String
    tags: Set<String>
    version: Integer
    created_by: String
    created_at: DateTime
    updated_by: String
    updated_at: DateTime
}
```

#### FeatureFlagRule
Represents a targeting rule for a feature flag.

```pseudo
enum RuleOperator {
    EQUALS,
    NOT_EQUALS,
    IN,
    NOT_IN,
    CONTAINS,
    STARTS_WITH,
    ENDS_WITH,
    GREATER_THAN,
    LESS_THAN
}

struct FeatureFlagRule {
    id: String
    feature_flag_id: String
    attribute: String
    operator: RuleOperator
    values: List<String>
    percentage: Integer
}
```

## Database Schema (CQRS Pattern)

### Write Model (Command Side)

#### configurations Table
```pseudo
Table configurations {
    id: String (primary key, max_length: 36)
    key: String (required, max_length: 255)
    application: String (required, max_length: 100)
    environment: String (required, max_length: 20)
    description: String
    value_type: String (required, max_length: 20)
    value: String (required)
    schema: String
    version: Integer (required, default: 1)
    created_by: String (max_length: 100)
    created_at: Timestamp (required)
    updated_by: String (max_length: 100)
    updated_at: Timestamp (required)

    // Constraints
    uk_config_key_app_env: (key, application, environment)
}

// Indexes
idx_configurations_app_env: (application, environment)
idx_configurations_key: (key)
```

#### configuration_tags Table
```pseudo
Table configuration_tags {
    configuration_id: String (required, max_length: 36, foreign_key: configurations.id)
    tag: String (required, max_length: 50)

    // Primary Key
    primary_key: (configuration_id, tag)

    // Foreign Key Constraints
    fk_config_tag: configuration_id -> configurations.id (on_delete: cascade)
}

// Indexes
idx_configuration_tags_tag: (tag)
```

#### configuration_versions Table
```pseudo
Table configuration_versions {
    id: String (primary key, max_length: 36)
    configuration_id: String (required, max_length: 36, foreign_key: configurations.id)
    version: Integer (required)
    value: String (required)
    created_by: String (max_length: 100)
    created_at: Timestamp (required)
    change_description: String

    // Constraints
    uk_config_version: (configuration_id, version)

    // Foreign Key Constraints
    fk_config_version: configuration_id -> configurations.id (on_delete: cascade)
}

// Indexes
idx_configuration_versions_config_id: (configuration_id)
```

#### feature_flags Table
```pseudo
Table feature_flags {
    id: String (primary key, max_length: 36)
    key: String (required, max_length: 255)
    application: String (required, max_length: 100)
    environment: String (required, max_length: 20)
    description: String
    enabled: Boolean (required, default: false)
    value: JSON
    version: Integer (required, default: 1)
    created_by: String (max_length: 100)
    created_at: Timestamp (required)
    updated_by: String (max_length: 100)
    updated_at: Timestamp (required)

    // Constraints
    uk_feature_flag_key_app_env: (key, application, environment)
}

// Indexes
idx_feature_flags_app_env: (application, environment)
idx_feature_flags_key: (key)
```

#### feature_flag_tags Table
```pseudo
Table feature_flag_tags {
    feature_flag_id: String (required, max_length: 36, foreign_key: feature_flags.id)
    tag: String (required, max_length: 50)

    // Primary Key
    primary_key: (feature_flag_id, tag)

    // Foreign Key Constraints
    fk_feature_flag_tag: feature_flag_id -> feature_flags.id (on_delete: cascade)
}

// Indexes
idx_feature_flag_tags_tag: (tag)
```

#### feature_flag_rules Table
```pseudo
Table feature_flag_rules {
    id: String (primary key, max_length: 36)
    feature_flag_id: String (required, max_length: 36, foreign_key: feature_flags.id)
    attribute: String (max_length: 100)
    operator: String (max_length: 20)
    percentage: Integer

    // Foreign Key Constraints
    fk_feature_flag_rule: feature_flag_id -> feature_flags.id (on_delete: cascade)
}

// Indexes
idx_feature_flag_rules_flag_id: (feature_flag_id)
```

#### feature_flag_rule_values Table
```pseudo
Table feature_flag_rule_values {
    rule_id: String (required, max_length: 36, foreign_key: feature_flag_rules.id)
    value: String (required, max_length: 255)

    // Primary Key
    primary_key: (rule_id, value)

    // Foreign Key Constraints
    fk_rule_value: rule_id -> feature_flag_rules.id (on_delete: cascade)
}
```

#### configuration_events Table
```pseudo
Table configuration_events {
    id: String (primary key, max_length: 36)
    configuration_id: String (required, max_length: 36, foreign_key: configurations.id)
    event_type: String (required, max_length: 20)
    version: Integer (required)
    user_id: String (max_length: 100)
    timestamp: Timestamp (required)

    // Foreign Key Constraints
    fk_config_event: configuration_id -> configurations.id
}

// Indexes
idx_configuration_events_config_id: (configuration_id)
idx_configuration_events_timestamp: (timestamp)
```

### Read Model (Query Side)

#### configuration_values View
```pseudo
View configuration_values {
    id: String
    key: String
    application: String
    environment: String
    value_type: String
    value: String
    version: Integer
    updated_at: Timestamp

    // Source Query
    SELECT id, key, application, environment, value_type, value, version, updated_at
    FROM configurations
}
```

#### feature_flag_status View
```pseudo
View feature_flag_status {
    id: String
    key: String
    application: String
    environment: String
    enabled: Boolean
    value: JSON
    version: Integer
    updated_at: Timestamp

    // Source Query
    SELECT id, key, application, environment, enabled, value, version, updated_at
    FROM feature_flags
}
```

#### configuration_audit_log View
```pseudo
View configuration_audit_log {
    event_id: String
    key: String
    application: String
    environment: String
    event_type: String
    version: Integer
    user_id: String
    timestamp: Timestamp
    value: String
    change_description: String

    // Source Query
    SELECT 
        e.id as event_id,
        c.key,
        c.application,
        c.environment,
        e.event_type,
        e.version,
        e.user_id,
        e.timestamp,
        cv.value,
        cv.change_description
    FROM 
        configuration_events e
    JOIN 
        configurations c ON e.configuration_id = c.id
    LEFT JOIN 
        configuration_versions cv ON e.configuration_id = cv.configuration_id AND e.version = cv.version
    ORDER BY 
        e.timestamp DESC
}
```

## Project Plan

### Phase 1: Foundation (Weeks 1-2)

#### Week 1: Setup and Core Domain Model
- Set up project structure and build system
- Implement core domain model (Configuration, FeatureFlag, etc.)
- Create database schema with migrations
- Implement basic repository layer
- Set up testing framework and write initial tests

#### Week 2: Basic API Implementation
- Implement REST API endpoints for configuration management
- Implement gRPC service definitions
- Create service layer with basic business logic
- Implement validation rules
- Write integration tests for API endpoints

### Phase 2: Core Functionality (Weeks 3-4)

#### Week 3: Configuration Management
- Implement configuration versioning
- Create configuration change events
- Implement rollback functionality
- Add environment-specific configuration
- Implement configuration validation against schemas

#### Week 4: Feature Flag System
- Implement feature flag management
- Create targeting rules engine
- Add percentage-based rollout
- Implement feature flag evaluation
- Write integration tests for feature flags

### Phase 3: Advanced Features (Weeks 5-6)

#### Week 5: Security and Integration
- Implement secure storage for sensitive configurations
- Add role-based access control
- Create integration with other services
- Implement configuration change notifications
- Add audit logging for all changes

#### Week 6: Performance and Caching
- Implement caching layer for configuration values
- Add distributed cache invalidation
- Optimize database queries
- Implement bulk configuration operations
- Conduct performance testing and optimization

### Phase 4: Finalization (Weeks 7-8)

#### Week 7: Monitoring and Resilience
- Add comprehensive logging and metrics
- Implement health check endpoints
- Create configuration validation tools
- Add configuration import/export functionality
- Implement rate limiting for API endpoints

#### Week 8: Documentation and Deployment
- Create comprehensive API documentation
- Write operational runbooks
- Prepare deployment configurations
- Conduct final testing and bug fixes
- Prepare for production deployment

### Milestones and Deliverables

1. **End of Week 2**: Basic configuration management API implemented
2. **End of Week 4**: Complete configuration versioning and feature flags
3. **End of Week 6**: Security, integration, and performance optimizations
4. **End of Week 8**: Production-ready service with documentation

### Risk Management

1. **Data Consistency**: Implement robust transaction management and versioning
2. **Performance Bottlenecks**: Implement effective caching strategies
3. **Security Vulnerabilities**: Secure storage of sensitive configuration values
4. **Integration Complexity**: Develop clear integration patterns for other services
5. **Backward Compatibility**: Ensure changes don't break existing configurations
