# Intelligence Distribution Service

## Responsibility
Event streaming and API management for distributing processed intelligence data to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for intelligence access.

## Technology Stack
- **Language**: Go + Apache Kafka + gRPC for intelligence distribution
- **Protocols**: Apache Kafka, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 50ms, 99.99% delivery guarantee, 500K+ events/sec

## API Specification

### Streaming APIs
```pseudo
// Enumerations
enum IntelligenceDataType {
    SENTIMENT,
    IMPACT_ASSESSMENT,
    ENTITY_EXTRACTION,
    NLP_PROCESSING,
    CONTENT_QUALITY
}

// Data Models
struct IntelligenceDistribution {
    event_id: String
    timestamp: DateTime
    data_type: IntelligenceDataType
    data: Any
    routing: RoutingInfo
    metadata: DistributionMetadata
}

struct RoutingInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
}

struct DistributionMetadata {
    source_service: String
    processing_time_ms: Float
    confidence_score: Float
    data_quality_tier: String
}

// gRPC Service Interface
interface IntelligenceStream {
    method streamIntelligence(request: StreamRequest) -> Stream<IntelligenceMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/intelligence/{type}/latest
    Parameters: symbol, limit
    Response: LatestIntelligenceResponse

GET /api/v1/intelligence/sentiment/{symbol}
    Parameters: timeframe
    Response: SentimentResponse

GET /api/v1/intelligence/impact/{symbol}
    Parameters: start_date, end_date
    Response: ImpactResponse

WebSocket /api/v1/intelligence/stream
    Parameters: symbols, data_types
    Response: Stream<IntelligenceUpdate>
```

### Event Output
```pseudo
Event intelligence_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: IntelligenceData
    metadata: IntelligenceMetadata
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
}

struct IntelligenceData {
    symbol: String
    sentiment_score: Float
    impact_score: Float
    entities: List<String>
    quality_tier: String
    confidence: Float
}

struct IntelligenceMetadata {
    source_service: String
    processing_time_ms: Float
    data_quality_tier: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "intelligence/sentiment/processed",
        routing_key: "AAPL.sentiment.positive",
        target_workflows: ["instrument_analysis", "trading_decision"],
        priority: 1
    },
    data: {
        symbol: "AAPL",
        sentiment_score: 0.75,
        impact_score: 0.65,
        entities: ["AAPL", "earnings", "iPhone"],
        quality_tier: "TIER_1_PREMIUM",
        confidence: 0.89
    },
    metadata: {
        source_service: "sentiment_analysis_service",
        processing_time_ms: 125.4,
        data_quality_tier: "TIER_1_PREMIUM"
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table intelligence_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    retention_policy: String (default: '30d', max_length: 50)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table intelligence_subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    topic_name: String (required, max_length: 200)
    subscriber_workflow: String (required, max_length: 100)
    filters: JSON
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
}

Table distribution_stats {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    topic_name: String (required, max_length: 200)
    messages_distributed: Integer (default: 0)
    avg_latency_ms: Float
    error_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
struct IntelligenceCache {
    // Latest intelligence: "latest:{data_type}:{symbol}" -> IntelligenceData
    // Topic stats: "topic_stats:{topic}" -> TopicStats
    // Subscription status: "subscription:{id}" -> SubscriptionStatus
    // Distribution metrics: "dist_metrics:{topic}" -> DistributionMetrics
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical distribution layer)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Kafka integration
- Topic management and subscription handling
- Basic message routing and distribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 2-3: API Development
- gRPC streaming API implementation
- REST API for intelligence access
- WebSocket support for real-time clients
- **Effort**: 1 developer × 2 weeks = 2 dev-weeks

#### Week 4: Integration & Testing
- Integration with all intelligence services
- Performance testing and optimization
- Monitoring and alerting setup
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Kafka cluster, intelligence processing services

### Success Criteria:
- Distribute 500K+ events per second
- P99 distribution latency < 50ms
- 99.99% message delivery guarantee
- Support 500+ concurrent subscribers
- Zero message loss during normal operations
