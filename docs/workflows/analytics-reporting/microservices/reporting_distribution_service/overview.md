# Reporting Distribution Service

## Responsibility
Event streaming and API management for distributing reports and analytics to downstream consumers. Manages report delivery, subscription management, and provides both streaming and REST APIs for report access and distribution.

## Technology Stack
- **Language**: Go + Apache Kafka + gRPC + email/notification services
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Protocols**: Apache Kafka, gRPC, SMTP, WebSocket, REST
- **Scaling**: Horizontal by distribution volume and subscriber count
- **NFRs**: P99 distribution latency < 100ms, 99.99% delivery guarantee, multi-channel delivery

## API Specification

### Streaming APIs
```pseudo
// Enumerations
enum ReportDataType {
    GENERATED_REPORT,
    SCHEDULED_REPORT,
    ALERT_REPORT,
    COMPLIANCE_REPORT,
    PERFORMANCE_REPORT,
    RISK_REPORT
}

enum DeliveryChannel {
    EMAIL,
    SFTP,
    API_WEBHOOK,
    DASHBOARD_NOTIFICATION,
    MOBILE_PUSH,
    SLACK_INTEGRATION
}

enum SubscriptionType {
    REAL_TIME,
    SCHEDULED,
    ON_DEMAND,
    TRIGGERED
}

// Data Models
struct ReportDistribution {
    event_id: String
    timestamp: DateTime
    data_type: ReportDataType
    report_data: ReportData
    routing: DistributionRouting
    metadata: DistributionMetadata
}

struct DistributionRouting {
    topic: String
    routing_key: String
    target_subscribers: List<String>
    delivery_channels: List<DeliveryChannel>
    priority: Integer
}

struct ReportSubscription {
    subscription_id: String
    subscriber_id: String
    report_types: List<ReportDataType>
    delivery_preferences: DeliveryPreferences
    filters: SubscriptionFilters
    status: SubscriptionStatus
}

struct DeliveryPreferences {
    channels: List<DeliveryChannel>
    schedule: Optional<String>
    format_preferences: List<String>
    notification_settings: NotificationSettings
}

struct DeliveryStatus {
    delivery_id: String
    subscription_id: String
    report_id: String
    channel: DeliveryChannel
    status: String
    delivery_time: DateTime
    retry_count: Integer
    error_details: Optional<String>
}

// gRPC Service Interface
interface ReportingStream {
    method streamReports(request: StreamRequest) -> Stream<ReportMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
    method getDeliveryStatus(request: DeliveryStatusRequest) -> DeliveryStatus
}

// REST API Endpoints
POST /api/v1/reporting/subscribe
    Request: ReportSubscription
    Response: SubscriptionResponse

GET /api/v1/reporting/subscriptions/{subscriber_id}
    Response: List<ReportSubscription>

POST /api/v1/reporting/deliver
    Request: DeliveryRequest
    Response: DeliveryResponse

GET /api/v1/reporting/delivery/{delivery_id}/status
    Response: DeliveryStatus

WebSocket /api/v1/reporting/stream
    Parameters: subscription_id, report_types
    Response: Stream<ReportUpdate>
```

### Event Output
```pseudo
Event report_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    report_data: ReportDataInfo
    metadata: ReportMetadata
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_subscribers: List<String>
    delivery_channels: List<String>
}

struct ReportDataInfo {
    report_id: String
    report_type: String
    portfolio_id: String
    report_period: String
    file_path: String
    file_size_bytes: Integer
}

struct ReportMetadata {
    source_service: String
    delivery_priority: Integer
    encryption_required: Boolean
    retention_days: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "reporting/performance/delivered",
        routing_key: "portfolio_001.performance.monthly",
        target_subscribers: ["client_001", "portfolio_manager_001"],
        delivery_channels: ["EMAIL", "DASHBOARD_NOTIFICATION"]
    },
    report_data: {
        report_id: "report_20250621_001",
        report_type: "PERFORMANCE_REPORT",
        portfolio_id: "portfolio_001",
        report_period: "2025-06",
        file_path: "/reports/portfolio_performance_202506.pdf",
        file_size_bytes: 2048576
    },
    metadata: {
        source_service: "report_generation_service",
        delivery_priority: 1,
        encryption_required: true,
        retention_days: 90
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table report_subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    subscriber_id: String (required, max_length: 100)
    subscriber_type: String (required, max_length: 50)
    report_types: JSON (required)
    delivery_preferences: JSON (required)
    filters: JSON
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
}

Table delivery_channels {
    id: UUID (primary key, auto-generated)
    channel_id: String (required, unique, max_length: 100)
    channel_type: String (required, max_length: 50)
    channel_config: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table delivery_history {
    id: UUID (primary key, auto-generated)
    delivery_id: String (required, unique, max_length: 100)
    subscription_id: String (required, max_length: 100)
    report_id: String (required, max_length: 100)
    channel: String (required, max_length: 50)
    status: String (required, max_length: 20)
    delivery_time: Timestamp
    retry_count: Integer (default: 0)
    error_details: String
    created_at: Timestamp (default: now)
}

Table distribution_stats {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    report_type: String (required, max_length: 50)
    channel: String (required, max_length: 50)
    deliveries_attempted: Integer (default: 0)
    deliveries_successful: Integer (default: 0)
    deliveries_failed: Integer (default: 0)
    avg_delivery_time_ms: Float
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache reporting_distribution_cache {
    // Active subscriptions
    "subscriptions:{subscriber_id}": List<Subscription> (TTL: 1h)

    // Delivery queue
    "delivery_queue:{priority}": List<DeliveryTask> (TTL: 30m)

    // Channel status
    "channel_status:{channel_id}": ChannelStatus (TTL: 5m)

    // Distribution metrics
    "dist_metrics:{report_type}": DistributionMetrics (TTL: 15m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for report delivery)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Kafka integration
- Subscription management system
- Basic delivery channel implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Multi-Channel Delivery
- Email delivery integration
- SFTP and webhook delivery
- Mobile push notifications
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Advanced Features & Integration
- Delivery retry and error handling
- Integration with all reporting services
- Performance monitoring and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 integration specialist)
### Dependencies: Apache Kafka cluster, reporting services, notification infrastructure

### Success Criteria:
- P99 distribution latency < 100ms
- 99.99% delivery guarantee
- Multi-channel delivery support
- Comprehensive subscription management
- Automated retry and error handling
- Real-time delivery status tracking
