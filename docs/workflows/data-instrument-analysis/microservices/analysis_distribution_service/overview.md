# Analysis Distribution Service

## Responsibility
Event streaming and API management for distributing processed instrument analysis data to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for analysis access.

## Technology Stack
- **Language**: Go + Apache Pulsar + gRPC for high-performance distribution
- **Protocols**: Apache Pulsar, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 30ms, 99.99% delivery guarantee, 1M+ events/sec

## API Specification

### Core APIs
```pseudo
// Enumerations
enum DataType {
    INDICATORS,
    CORRELATIONS,
    PATTERNS,
    RISK
}

// Data Models
struct AnalysisDistribution {
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
}

struct IndicatorsResponse {
    symbol: String
    timeframe: String
    indicators: Map<String, IndicatorValue>
    timestamp: DateTime
}

// gRPC Service Definition
service AnalysisStream {
    streamAnalysis(request: StreamRequest) -> Stream<AnalysisMessage>
    manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/analysis/indicators/{symbol}
    Response: IndicatorsResponse
```

### Event Output
```pseudo
Event analysis_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: AnalysisData
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
}

struct AnalysisData {
    instrument_id: String
    indicators: Map<String, Float>
    correlations: Map<String, Float>
    risk_metrics: Map<String, Float>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "analysis/indicators/processed",
        routing_key: "AAPL.indicators.5m",
        target_workflows: ["market_prediction", "trading_decision"]
    },
    data: {
        instrument_id: "AAPL",
        indicators: {"rsi_14": 65.4, "macd": 0.45},
        correlations: {"SPY": 0.85},
        risk_metrics: {"volatility": 0.28}
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table analysis_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table analysis_subscriptions {
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
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Pulsar integration
- Topic management and subscription handling
- Basic message routing and distribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: API & Integration
- gRPC streaming API implementation
- REST API for analysis access
- Integration with all analysis services
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Pulsar cluster, analysis processing services

### Success Criteria:
- Distribute 1M+ events per second
- P99 distribution latency < 30ms
- 99.99% message delivery guarantee
- Support 1000+ concurrent subscribers
