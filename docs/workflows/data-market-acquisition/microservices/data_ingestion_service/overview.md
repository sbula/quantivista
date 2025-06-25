# Data Ingestion Service

## Responsibility
Provider-specific data collection with optimized protocols for real-time market data ingestion from multiple sources (Bloomberg, Reuters, IEX, Alpha Vantage, Polygon). Handles connection management, rate limiting, and provider-specific protocol optimization.

## Technology Stack
- **Language**: Rust + Tokio for async I/O
- **Protocols**: WebSocket, REST, FIX, provider-specific SDKs
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal by provider, vertical by throughput
- **NFRs**: P99 ingestion latency < 50ms, 99.9% uptime per provider

## API Specification

### Internal APIs

#### Provider Management API
```pseudo
// Provider Interface
interface DataProvider {
    method connect() -> Result<Success, ProviderError>
    method subscribe(instruments: List<String>) -> Result<Success, ProviderError>
    method getStream() -> Stream<RawMarketData>
    method disconnect() -> Result<Success, ProviderError>
}

// Data Models
struct ProviderStatus {
    provider_id: String
    status: ConnectionStatus
    last_heartbeat: DateTime
    messages_per_second: Float
    error_rate: Float
}

struct SubscriptionRequest {
    provider_id: String
    instruments: List<String>
    data_types: List<DataType>
}

// REST API Endpoints
POST /api/v1/providers/{provider_id}/subscribe
    Request: SubscriptionRequest
    Response: SubscriptionResponse

GET /api/v1/providers/{provider_id}/status
    Response: ProviderStatus

PUT /api/v1/providers/{provider_id}/connect
    Response: ConnectionResult
```

#### Health Check API
```pseudo
// Health Monitoring
struct HealthResponse {
    status: ServiceStatus
    providers: Map<String, ProviderHealth>
    total_throughput: Integer
    uptime_seconds: Integer
}

struct MetricsResponse {
    ingestion_rate: Float
    latency_p99: Duration
    error_rate: Float
    active_connections: Integer
}

// REST API Endpoints
GET /health
    Response: HealthResponse

GET /metrics
    Response: MetricsResponse
```

### Event Output

#### RawMarketDataEvent
```pseudo
Event raw_market_data_ingested {
    event_id: String
    timestamp: DateTime
    provider: String
    raw_data: RawMarketDataPayload
    metadata: IngestionMetadata
}

struct RawMarketDataPayload {
    symbol: String
    price: Float
    volume: Integer
    timestamp: DateTime
    bid: Float
    ask: Float
    provider_specific: JSON
}

struct IngestionMetadata {
    ingestion_latency_ms: Integer
    provider_sequence: Integer
    quality_flags: List<String>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T09:30:00.123Z",
    provider: "bloomberg",
    raw_data: {
        symbol: "AAPL",
        price: 150.25,
        volume: 1000,
        timestamp: "2025-06-21T09:30:00.120Z",
        bid: 150.24,
        ask: 150.26,
        provider_specific: {}
    },
    metadata: {
        ingestion_latency_ms: 3,
        provider_sequence: 12345,
        quality_flags: []
    }
}
```

## Data Model

### Core Entities
```pseudo
// Core Data Structures
struct RawMarketData {
    symbol: String
    provider: String
    timestamp: DateTime
    data_type: DataType
    price: Optional<Float>
    volume: Optional<Integer>
    bid: Optional<Float>
    ask: Optional<Float>
    provider_specific: JsonObject
}

enum DataType {
    TRADE,
    QUOTE,
    ORDER_BOOK,
    NEWS,
    CORPORATE_ACTION
}

struct ProviderConnection {
    provider_id: String
    connection_type: ConnectionType
    status: ConnectionStatus
    last_heartbeat: DateTime
    subscriptions: Set<String>
    metrics: ConnectionMetrics
}

enum ConnectionStatus {
    CONNECTED,
    DISCONNECTED,
    CONNECTING,
    ERROR,
    RATE_LIMITED
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Provider configuration and management
Table providers {
    id: UUID (primary key, auto-generated)
    name: String (required, unique, max_length: 50)
    provider_type: String (required, max_length: 20)
    connection_config: JSON (required)
    rate_limits: JSON
    priority: Integer (default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Subscription management
Table subscriptions {
    id: UUID (primary key, auto-generated)
    provider_id: UUID (required, foreign_key: providers.id)
    instrument_symbol: String (required, max_length: 20)
    data_types: List<String> (required)
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)

    // Constraints
    unique_provider_symbol: (provider_id, instrument_symbol)
}

// Connection status tracking
Table provider_connections {
    id: UUID (primary key, auto-generated)
    provider_id: UUID (required, foreign_key: providers.id)
    status: String (required, max_length: 20)
    last_heartbeat: Timestamp
    error_message: String
    connection_metadata: JSON
    created_at: Timestamp (default: now)
}

// Ingestion metrics (command side for writes)
Table ingestion_metrics {
    id: UUID (primary key, auto-generated)
    provider_id: UUID (required, foreign_key: providers.id)
    timestamp: Timestamp (required)
    messages_ingested: Integer (default: 0)
    bytes_ingested: Integer (default: 0)
    errors_count: Integer (default: 0)
    avg_latency_ms: Float
    created_at: Timestamp (default: now)
}

// Indexes for performance
idx_providers_enabled: (enabled)
idx_subscriptions_provider: (provider_id)
idx_connections_provider_status: (provider_id, status)
idx_metrics_provider_timestamp: (provider_id, timestamp)
```

### Query Side (TimescaleDB + Redis)
```pseudo
// Raw market data storage (TimescaleDB)
Table raw_market_data {
    timestamp: Timestamp (required, partition_key)
    symbol: String (required, max_length: 20)
    provider: String (required, max_length: 50)
    data_type: String (required, max_length: 20)
    price: Decimal (precision: 15, scale: 6)
    volume: Integer
    bid: Decimal (precision: 15, scale: 6)
    ask: Decimal (precision: 15, scale: 6)
    provider_specific: JSON
    ingestion_timestamp: Timestamp (default: now)
    quality_score: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: symbol (partitions: 16)
}

// Indexes for fast queries
idx_raw_data_symbol_time: (symbol, timestamp DESC)
idx_raw_data_provider_time: (provider, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache redis_cache {
    // Latest price cache
    "latest:{symbol}": MarketData (TTL: 5s)

    // Provider status
    "provider:{id}:status": ProviderStatus (TTL: 30s)

    // Rate limiting
    "ratelimit:{provider}:{minute}": Integer (TTL: 60s)

    // Circuit breaker state
    "circuit:{provider}": CircuitBreakerState (TTL: 300s)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Foundation service)
### Estimated Time: **6-8 weeks**

#### Week 1-2: Core Infrastructure
- Basic Rust service setup with Tokio runtime
- Provider abstraction trait and connection management
- Basic WebSocket and REST client implementations
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Provider Integrations
- Bloomberg API integration (if available)
- IEX Cloud integration
- Alpha Vantage integration
- Polygon.io integration
- **Effort**: 3 developers × 2 weeks = 6 dev-weeks

#### Week 5-6: Quality & Reliability
- Circuit breaker implementation
- Rate limiting and backpressure handling
- Error handling and retry mechanisms
- Connection pooling and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 7-8: Monitoring & Testing
- Metrics collection and health checks
- Integration testing with mock providers
- Performance testing and optimization
- Documentation and deployment scripts
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **18 dev-weeks**
### Team Size: **3 developers** (1 senior Rust developer, 2 mid-level developers)
### Dependencies: 
- Apache Pulsar cluster setup
- TimescaleDB deployment
- Provider API access and credentials

### Risk Factors:
- **High**: Provider API availability and rate limits
- **Medium**: Real-time performance requirements
- **Low**: Technology stack complexity

### Success Criteria:
- Ingest data from at least 3 providers simultaneously
- Achieve P99 latency < 50ms
- Handle 10,000+ messages per second
- 99.9% uptime during market hours
- Automatic failover between providers
