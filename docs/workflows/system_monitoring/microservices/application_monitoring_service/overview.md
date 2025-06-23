# Application Monitoring Service

## Responsibility
Application-level monitoring for all QuantiVista microservices including performance metrics, health checks, distributed tracing, and application-specific KPIs. Provides deep visibility into service behavior and performance.

## Technology Stack
- **Language**: Java + Spring Boot + Micrometer + OpenTelemetry
- **Libraries**: Micrometer, OpenTelemetry, Jaeger, Zipkin, Spring Boot Actuator
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by service count and trace volume
- **NFRs**: P99 trace processing < 200ms, distributed tracing coverage 100%, real-time health monitoring

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ServiceHealthStatus {
    UP,
    DOWN,
    DEGRADED,
    UNKNOWN,
    MAINTENANCE
}

enum TraceStatus {
    SUCCESS,
    ERROR,
    TIMEOUT,
    CANCELLED
}

enum MetricCategory {
    PERFORMANCE,
    BUSINESS,
    TECHNICAL,
    SECURITY,
    CUSTOM
}

// Data Models
struct ServiceHealth {
    service_name: String
    service_version: String
    health_status: ServiceHealthStatus
    health_checks: List<HealthCheck>
    uptime_seconds: Integer
    last_restart: DateTime
    dependencies: List<ServiceDependency>
}

struct HealthCheck {
    check_name: String
    status: String
    details: Map<String, Any>
    response_time_ms: Float
    last_checked: DateTime
}

struct DistributedTrace {
    trace_id: String
    span_id: String
    parent_span_id: Optional<String>
    service_name: String
    operation_name: String
    start_time: DateTime
    duration_ms: Float
    status: TraceStatus
    tags: Map<String, String>
    logs: List<TraceLog>
}

struct ApplicationMetric {
    service_name: String
    metric_name: String
    metric_category: MetricCategory
    value: Float
    tags: Map<String, String>
    timestamp: DateTime
}

struct ServiceDependency {
    dependency_name: String
    dependency_type: String
    health_status: ServiceHealthStatus
    response_time_ms: Float
    error_rate: Float
}

// REST API Endpoints
GET /api/v1/application/services
    Response: List<ServiceHealth>

GET /api/v1/application/services/{service_name}/health
    Response: ServiceHealth

GET /api/v1/application/traces
    Parameters: service_name, start_time, end_time, status
    Response: List<DistributedTrace>

GET /api/v1/application/traces/{trace_id}
    Response: DistributedTrace

GET /api/v1/application/metrics
    Parameters: service_name, metric_category, start_time, end_time
    Response: List<ApplicationMetric>

POST /api/v1/application/health-check/{service_name}
    Response: HealthCheckResult
```

### Event Output
```pseudo
Event application_status_updated {
    event_id: String
    timestamp: DateTime
    application_status: ApplicationStatusData
}

struct ApplicationStatusData {
    service_name: String
    service_version: String
    health_status: String
    health_checks: List<HealthCheckData>
    performance_metrics: PerformanceMetricsData
    dependencies: List<DependencyData>
}

struct HealthCheckData {
    check_name: String
    status: String
    response_time_ms: Float
}

struct PerformanceMetricsData {
    requests_per_second: Float
    average_response_time_ms: Float
    error_rate: Float
    cpu_usage_percent: Float
    memory_usage_mb: Integer
}

struct DependencyData {
    dependency_name: String
    health_status: String
    response_time_ms: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    application_status: {
        service_name: "portfolio_management_service",
        service_version: "1.2.3",
        health_status: "UP",
        health_checks: [
            {
                check_name: "database_connectivity",
                status: "UP",
                response_time_ms: 15.2
            },
            {
                check_name: "redis_connectivity",
                status: "UP",
                response_time_ms: 8.5
            }
        ],
        performance_metrics: {
            requests_per_second: 125.5,
            average_response_time_ms: 45.2,
            error_rate: 0.002,
            cpu_usage_percent: 35.8,
            memory_usage_mb: 512
        },
        dependencies: [
            {
                dependency_name: "postgresql_primary",
                health_status: "UP",
                response_time_ms: 12.1
            }
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table service_registry {
    id: UUID (primary key, auto-generated)
    service_name: String (required, unique, max_length: 100)
    service_version: String (required, max_length: 50)
    service_type: String (required, max_length: 50)
    health_endpoint: String (max_length: 500)
    metrics_endpoint: String (max_length: 500)
    dependencies: JSON
    monitoring_config: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table health_check_history {
    id: UUID (primary key, auto-generated)
    service_name: String (required, max_length: 100)
    check_name: String (required, max_length: 100)
    status: String (required, max_length: 20)
    response_time_ms: Float
    details: JSON
    checked_at: Timestamp (default: now)
}
```

### TimescaleDB (Traces and Metrics)
```pseudo
Table distributed_traces_ts {
    timestamp: Timestamp (required, partition_key)
    trace_id: String (required, max_length: 100)
    span_id: String (required, max_length: 100)
    parent_span_id: String (max_length: 100)
    service_name: String (required, max_length: 100)
    operation_name: String (required, max_length: 200)
    duration_ms: Float (required)
    status: String (required, max_length: 20)
    tags: JSON
    logs: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: service_name (partitions: 8)
}

Table application_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    service_name: String (required, max_length: 100)
    metric_name: String (required, max_length: 200)
    metric_category: String (required, max_length: 50)
    metric_value: Float (required)
    tags: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for service monitoring)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Application Monitoring
- Java service setup with Spring Boot
- Health check framework
- Basic metrics collection
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Distributed Tracing
- OpenTelemetry integration
- Trace collection and processing
- Service dependency mapping
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Integration & Advanced Features
- Integration with all QuantiVista services
- Performance optimization
- Advanced monitoring dashboards
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Java developer, 1 monitoring specialist)
### Dependencies: All QuantiVista microservices, OpenTelemetry infrastructure

### Success Criteria:
- P99 trace processing < 200ms
- 100% distributed tracing coverage
- Real-time health monitoring
- Comprehensive service dependency mapping
- Application performance insights
