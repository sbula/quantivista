# Portfolio Distribution Service

## Responsibility
Event streaming and API management for distributing portfolio management data to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for portfolio access.

## Technology Stack
- **Language**: Go + Apache Pulsar + gRPC
- **Protocols**: Apache Pulsar, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 15ms, 99.99% delivery guarantee, 100K+ events/sec

## API Specification

### Streaming APIs
```pseudo
// Enumerations
enum PortfolioDataType {
    PORTFOLIO_STATE,
    PERFORMANCE_ATTRIBUTION,
    REBALANCING,
    RISK_BUDGET,
    CASH_MANAGEMENT
}

// Data Models
struct PortfolioDistribution {
    event_id: String
    timestamp: DateTime
    data_type: PortfolioDataType
    data: Any
    routing: RoutingInfo
}

struct RoutingInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
}

// gRPC Service Interface
interface PortfolioStream {
    method streamPortfolio(request: StreamRequest) -> Stream<PortfolioMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/portfolio/{portfolio_id}/state
    Response: PortfolioState

GET /api/v1/portfolio/{portfolio_id}/performance
    Response: PerformanceData
```

### Event Output
```pseudo
Event portfolio_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: PortfolioData
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
}

struct PortfolioData {
    portfolio_id: String
    total_value: Float
    performance_metrics: PerformanceMetricsData
}

struct PerformanceMetricsData {
    daily_return: Float
    ytd_return: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "portfolio/state/updated",
        routing_key: "portfolio_001.state.update",
        target_workflows: ["reporting_analytics", "user_interface"]
    },
    data: {
        portfolio_id: "portfolio_001",
        total_value: 1000000.00,
        performance_metrics: {
            daily_return: 0.012,
            ytd_return: 0.085
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table portfolio_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table portfolio_subscriptions {
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
- Integration with all portfolio services
- Performance testing and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **6 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Pulsar cluster, portfolio processing services

### Success Criteria:
- Distribute 100K+ events per second
- P99 distribution latency < 15ms
- 99.99% message delivery guarantee
- Support 100+ concurrent subscribers
- Zero message loss during normal operations
