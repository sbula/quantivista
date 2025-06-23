# Data Distribution Service

## Responsibility
High-performance event streaming and API management for distributing normalized market data to downstream workflows. Manages topic routing, subscription management, backpressure handling, and provides both streaming and REST APIs for data access.

## Technology Stack
- **Language**: Go + Apache Pulsar + gRPC for high-performance streaming
- **Protocols**: Apache Pulsar, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 25ms, 99.99% delivery guarantee, 2M+ events/sec throughput

## API Specification

### Streaming APIs

#### Pulsar Topic Management
```pseudo
// Enumerations
enum SubscriptionType {
    EXCLUSIVE,
    SHARED,
    FAILOVER,
    KEY_SHARED
}

enum MessagePosition {
    LATEST,
    EARLIEST,
    MESSAGE_ID
}

// Data Models
struct SubscriptionRequest {
    topic: String
    subscription: String
    type: SubscriptionType
    filters: List<MessageFilter>
    start_position: MessagePosition
}

// Interface Definitions
interface TopicManager {
    createTopic(topic: String, partitions: Integer) -> Result
    deleteTopic(topic: String) -> Result
    getTopicStats(topic: String) -> TopicStats
    listTopics() -> List<String>
}

interface SubscriptionManager {
    createSubscription(request: SubscriptionRequest) -> Result
    deleteSubscription(topic: String, subscription: String) -> Result
    getSubscriptionStats(topic: String, subscription: String) -> SubscriptionStats
}
```

#### gRPC Streaming API
```pseudo
// gRPC Service Definition
service MarketDataStream {
    // Real-time market data streaming
    streamMarketData(request: StreamRequest) -> Stream<MarketDataMessage>

    // Historical data streaming
    streamHistoricalData(request: HistoricalRequest) -> Stream<MarketDataMessage>

    // Subscription management
    manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// Message Definitions
struct StreamRequest {
    symbols: List<String>
    data_types: List<String>
    subscription_name: String
    filters: FilterCriteria
}

struct MarketDataMessage {
    event_id: String
    timestamp: Integer
    data: NormalizedMarketData
    metadata: MessageMetadata
}

struct FilterCriteria {
    exchanges: List<String>
    currencies: List<String>
    min_price: Float
    max_price: Float
    min_volume: Integer
}
```

### REST APIs

#### Data Access API
```pseudo
// Data Models
struct LatestDataResponse {
    symbol: String
    data: NormalizedMarketData
    timestamp: DateTime
    source: String
}

struct HistoricalDataRequest {
    symbol: String
    start_time: DateTime
    end_time: DateTime
    interval: String  // 1m, 5m, 1h, 1d
    limit: Integer
}

struct HistoricalDataResponse {
    symbol: String
    interval: String
    data: List<NormalizedMarketData>
    count: Integer
    has_more: Boolean
}

struct WebSocketMessage {
    type: String  // subscribe, unsubscribe, data, error
    payload: Any
    timestamp: DateTime
}

// REST API Endpoints
GET /api/v1/market-data/{symbol}/latest
    Response: LatestDataResponse

GET /api/v1/market-data/{symbol}/history
    Parameters: start_time, end_time, interval, limit
    Response: HistoricalDataResponse

WebSocket /api/v1/market-data/stream
    Message: WebSocketMessage
```

### Event Output

#### MarketDataDistributedEvent
```pseudo
Event market_data_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: MarketDataPayload
    routing: RoutingInfo
}

struct DistributionInfo {
    topic: String
    partition: Integer
    offset: Integer
    message_id: String
    subscribers_notified: Integer
    distribution_latency_ms: Integer
}

struct MarketDataPayload {
    symbol: String
    exchange: String
    timestamp: DateTime
    price: Float
    volume: Integer
}

struct RoutingInfo {
    routing_key: String
    target_workflows: List<String>
    delivery_guarantees: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T09:30:00.225Z",
    distribution: {
        topic: "market-data/normalized/equities",
        partition: 5,
        offset: 12345678,
        message_id: "msg-abc123",
        subscribers_notified: 15,
        distribution_latency_ms: 8
    },
    data: {
        symbol: "AAPL",
        exchange: "NASDAQ",
        timestamp: "2025-06-21T09:30:00.200Z",
        price: 150.25,
        volume: 1000
    },
    routing: {
        routing_key: "AAPL.NASDAQ.trade",
        target_workflows: ["market_intelligence", "instrument_analysis"],
        delivery_guarantees: "at_least_once"
    }
}
```

## Data Model

### Core Entities
```pseudo
// Enumerations
enum DeliveryMode {
    AT_MOST_ONCE,
    AT_LEAST_ONCE,
    EXACTLY_ONCE
}

enum SubscriptionStatus {
    ACTIVE,
    PAUSED,
    ERROR,
    STOPPED
}

// Data Models
struct MarketDataDistribution {
    event_id: String
    timestamp: DateTime
    data: NormalizedMarketData
    routing: RoutingInfo
    metadata: DistributionMetadata
}

struct RoutingInfo {
    topic: String
    routing_key: String
    partition: Integer
    target_workflows: List<String>
    delivery_mode: DeliveryMode
}

struct DistributionMetadata {
    message_id: String
    offset: Integer
    subscribers_notified: Integer
    distribution_latency: Float  // milliseconds
    retry_count: Integer
}

struct Subscription {
    id: String
    topic: String
    name: String
    type: SubscriptionType
    filters: List<MessageFilter>
    status: SubscriptionStatus
    created_at: DateTime
    last_activity: DateTime
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Topic configuration and management
Table topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    partitions: Integer (required, default: 1)
    retention_policy: String (default: '7d', max_length: 50)
    compression_type: String (default: 'LZ4', max_length: 20)
    schema_version: String (max_length: 10)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Subscription management
Table subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    topic_name: String (required, max_length: 200, foreign_key: topics.topic_name)
    subscription_name: String (required, max_length: 100)
    subscription_type: String (required, max_length: 20) // 'Exclusive', 'Shared', 'Failover', 'KeyShared'
    consumer_group: String (max_length: 100)
    filters: JSON
    start_position: String (default: 'Latest', max_length: 20) // 'Latest', 'Earliest', 'MessageId'
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
    last_activity: Timestamp (default: now)
}

// Distribution statistics
Table distribution_stats {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    topic_name: String (required, max_length: 200)
    messages_distributed: Integer (default: 0)
    bytes_distributed: Integer (default: 0)
    avg_latency_ms: Float
    error_count: Integer (default: 0)
    active_subscriptions: Integer (default: 0)
    created_at: Timestamp (default: now)
}

// Message routing rules
Table routing_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    source_pattern: String (required, max_length: 200)
    target_topic: String (required, max_length: 200)
    routing_key_template: String (max_length: 200)
    conditions: JSON
    priority: Integer (default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Indexes
idx_topics_enabled: (enabled)
idx_subscriptions_topic_status: (topic_name, status)
idx_distribution_stats_topic_time: (topic_name, timestamp DESC)
idx_routing_rules_priority: (priority, enabled)
```

### Query Side (TimescaleDB + Redis)
```pseudo
// Distribution metrics time series
Table distribution_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    topic_name: String (required, max_length: 200)
    partition_id: Integer
    throughput_per_second: Float
    latency_p50_ms: Float
    latency_p99_ms: Float
    error_rate: Float
    backlog_size: Integer
    consumer_count: Integer
    producer_count: Integer

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Message delivery tracking
Table message_delivery_log {
    timestamp: Timestamp (required, partition_key)
    message_id: String (required, max_length: 100)
    topic_name: String (required, max_length: 200)
    partition_id: Integer
    offset_position: Integer
    delivery_latency_ms: Float
    retry_count: Integer (default: 0)
    status: String (required, max_length: 20) // 'delivered', 'failed', 'retrying'
    error_message: String

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Indexes
idx_distribution_metrics_topic_time: (topic_name, timestamp DESC)
idx_delivery_log_message_time: (message_id, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache distribution_cache {
    // Latest data cache
    "latest:{symbol}": NormalizedMarketData (TTL: 1m)

    // Topic statistics
    "topic_stats:{topic}": TopicStats (TTL: 30s)

    // Subscription status
    "subscription:{id}": SubscriptionStatus (TTL: 5m)

    // Routing cache
    "routing:{pattern}": RoutingRule (TTL: 15m)

    // Backpressure status
    "backpressure:{topic}:{partition}": BackpressureInfo (TTL: 10s)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core distribution infrastructure)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Apache Pulsar integration
- Topic management and subscription handling
- Basic message routing and distribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: API Development
- gRPC streaming API implementation
- REST API for data access and management
- WebSocket support for real-time clients
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Performance & Reliability
- High-throughput optimization (2M+ events/sec)
- Backpressure handling and flow control
- Error handling and retry mechanisms
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 5: Advanced Features
- Message filtering and routing rules
- Subscription management and monitoring
- Historical data access optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 6: Integration & Testing
- Integration with Data Processing Service
- Load testing and performance validation
- Monitoring and alerting setup
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies:
- Apache Pulsar cluster operational
- Data Processing Service providing normalized data
- TimescaleDB and Redis setup

### Risk Factors:
- **High**: High-throughput performance requirements (2M+ events/sec)
- **Medium**: Apache Pulsar operational complexity
- **Low**: API design and implementation

### Success Criteria:
- Distribute 2M+ events per second
- P99 distribution latency < 25ms
- 99.99% message delivery guarantee
- Support 1000+ concurrent subscribers
- Zero message loss during normal operations
