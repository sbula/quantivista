# Decision Distribution Service

## Responsibility
Event streaming and API management for distributing trading signals and decisions to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for decision access.

## Technology Stack
- **Language**: Go + Apache Pulsar + gRPC
- **Protocols**: Apache Pulsar, gRPC, WebSocket, REST
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 25ms, 99.99% delivery guarantee, 500K+ events/sec

## API Specification

### Streaming APIs
```pseudo
// Enumerations
enum DataType {
    SIGNAL,
    SYNTHESIS,
    QUALITY
}

// Data Models
struct DecisionDistribution {
    event_id: String
    timestamp: DateTime
    data_type: DataType
    data: Any
    routing: RoutingInfo
}

struct RoutingInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
}

struct StreamRequest {
    symbols: List<String>
    data_types: List<DataType>
    filters: Optional<Map<String, Any>>
}

struct SignalsResponse {
    symbol: String
    signals: List<TradingSignal>
    timestamp: DateTime
}

// gRPC Service Interface
interface DecisionStream {
    method streamDecisions(request: StreamRequest) -> Stream<DecisionMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/decisions/signals/{symbol}
    Response: SignalsResponse
```

### Event Output
```pseudo
Event decision_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: DecisionData
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
}

struct DecisionData {
    signal_id: String
    instrument_id: String
    direction: String
    confidence: Float
    quality_score: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "decisions/signals/processed",
        routing_key: "AAPL.signal.buy",
        target_workflows: ["portfolio_trading_coordination", "trade_execution"]
    },
    data: {
        signal_id: "signal_20250621_001",
        instrument_id: "AAPL",
        direction: "BUY",
        confidence: 0.85,
        quality_score: 0.88
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table decision_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table decision_subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    topic_name: String (required, max_length: 200)
    subscriber_workflow: String (required, max_length: 100)
    filters: JSON
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical distribution layer)
### Estimated Time: **2-3 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Pulsar integration
- Topic management and subscription handling
- Message routing and distribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Integration & Testing
- gRPC and REST API implementation
- Integration with all decision services
- Performance testing and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **6 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Pulsar cluster, decision processing services

### Success Criteria:
- Distribute 500K+ events per second
- P99 distribution latency < 25ms
- 99.99% message delivery guarantee
- Support 500+ concurrent subscribers
- Zero message loss during normal operations
