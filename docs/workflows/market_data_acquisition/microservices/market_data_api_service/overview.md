# Market Data API Service

## Responsibility
External-facing API gateway providing secure, rate-limited access to market data for internal services and external clients. Handles authentication, authorization, caching, and API versioning while maintaining high performance and reliability.

## Technology Stack
- **Language**: Go + Gin + Redis for high-performance API serving
- **Security**: JWT authentication, OAuth2, API key management
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal with load balancing and API gateway patterns
- **NFRs**: P99 API response < 100ms, 99.9% uptime, 10K+ concurrent requests

## API Specification

### Public REST APIs

#### Market Data Endpoints
```pseudo
// Enumerations
enum DataInterval {
    ONE_MINUTE,
    FIVE_MINUTES,
    FIFTEEN_MINUTES,
    ONE_HOUR,
    ONE_DAY
}

enum ResponseFormat {
    JSON,
    CSV
}

// Data Models
struct LatestDataRequest {
    symbol: String
    fields: String  // price,volume,bid,ask
    currency: String
}

struct LatestDataResponse {
    symbol: String
    exchange: String
    currency: String
    data: MarketDataSnapshot
    timestamp: DateTime
    source: String
    cache_hit: Boolean
}

struct HistoricalDataRequest {
    symbol: String
    start_time: DateTime
    end_time: DateTime
    interval: DataInterval
    limit: Integer
    offset: Integer
    fields: String
    format: ResponseFormat
}

struct HistoricalDataResponse {
    symbol: String
    interval: String
    time_range: TimeRange
    data: List<MarketDataPoint>
    pagination: PaginationInfo
    metadata: ResponseMetadata
}

struct SymbolsResponse {
    symbols: List<SymbolInfo>
    count: Integer
    last_update: DateTime
}

struct BulkDataRequest {
    symbols: List<String>
    fields: List<String>
    currency: String
    as_of: Optional<DateTime>
}

struct BulkDataResponse {
    data: Map<String, MarketDataSnapshot>
    errors: Map<String, String>
    timestamp: DateTime
    count: Integer
}

// REST API Endpoints
GET /api/v1/market-data/{symbol}/latest
    Parameters: fields, currency
    Response: LatestDataResponse

GET /api/v1/market-data/{symbol}/history
    Parameters: start_time, end_time, interval, limit, offset, fields, format
    Response: HistoricalDataResponse

GET /api/v1/market-data/symbols
    Response: SymbolsResponse

POST /api/v1/market-data/bulk
    Request: BulkDataRequest
    Response: BulkDataResponse
```

#### WebSocket Streaming API
```pseudo
// Enumerations
enum WebSocketMessageType {
    SUBSCRIBE,
    UNSUBSCRIBE,
    DATA,
    ERROR,
    HEARTBEAT
}

enum MarketDataType {
    TRADE,
    QUOTE,
    ORDERBOOK,
    OHLC
}

// Data Models
struct WebSocketConnection {
    id: String
    subscriptions: Map<String, Subscription>
    rate_limit: RateLimitConfig
    last_activity: DateTime
}

struct WebSocketMessage {
    type: WebSocketMessageType
    request_id: Optional<String>
    payload: Any
    timestamp: DateTime
}

struct SubscribeMessage {
    symbols: List<String>
    data_types: List<MarketDataType>
    filters: Optional<Filters>
}

struct DataMessage {
    symbol: String
    data_type: MarketDataType
    data: MarketDataSnapshot
    timestamp: DateTime
}

// WebSocket API
WebSocket /api/v1/market-data/stream
    Message: WebSocketMessage
```

#### Authentication & Authorization
```pseudo
// Enumerations
enum GrantType {
    API_KEY,
    PASSWORD,
    REFRESH_TOKEN
}

enum TokenType {
    BEARER,
    API_KEY
}

// Data Models
struct AuthenticationRequest {
    api_key: Optional<String>
    username: Optional<String>
    password: Optional<String>
    grant_type: GrantType
}

struct AuthenticationResponse {
    access_token: String
    token_type: TokenType
    expires_in: Integer
    refresh_token: Optional<String>
    scope: List<String>
    issued_at: DateTime
}

struct APIKeyInfo {
    key_id: String
    name: String
    permissions: List<String>
    rate_limit: RateLimit
    expires_at: Optional<DateTime>
    created_at: DateTime
}

// REST API Endpoints
POST /api/v1/auth/token
    Request: AuthenticationRequest
    Response: AuthenticationResponse

GET /api/v1/auth/keys
    Response: List<APIKeyInfo>
```

### Internal APIs

#### Cache Management
```pseudo
// Interface Definitions
interface CacheManager {
    get(key: String) -> Any
    set(key: String, value: Any, ttl: Integer) -> Result
    delete(key: String) -> Result
    invalidate(pattern: String) -> Result
}

// Data Models
struct CacheStats {
    hit_rate: Float
    miss_rate: Float
    eviction_rate: Float
    memory_usage_bytes: Integer
    key_count: Integer
}

// REST API Endpoints
GET /api/v1/cache/stats
    Response: CacheStats

DELETE /api/v1/cache/{pattern}
    Response: OperationResult
```

## Data Model

### Core Entities
```pseudo
// Enumerations
enum ClientType {
    INTERNAL,
    EXTERNAL,
    PARTNER
}

enum ClientStatus {
    ACTIVE,
    SUSPENDED,
    INACTIVE
}

// Data Models
struct MarketDataSnapshot {
    symbol: String
    exchange: String
    currency: String
    timestamp: DateTime

    // Price data
    price: Optional<Float>
    volume: Optional<Integer>
    bid: Optional<Float>
    ask: Optional<Float>

    // Derived fields
    spread: Optional<Float>
    mid_price: Optional<Float>
    vwap: Optional<Float>

    // Metadata
    data_type: String
    source: String
    quality_score: Float
}

struct APIClient {
    client_id: String
    name: String
    type: ClientType
    permissions: List<String>
    rate_limit: RateLimit
    status: ClientStatus
    created_at: DateTime
    last_activity: DateTime
}

struct RateLimit {
    requests_per_second: Integer
    requests_per_minute: Integer
    requests_per_hour: Integer
    requests_per_day: Integer
    burst_limit: Integer
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// API clients and authentication
Table api_clients {
    id: UUID (primary key, auto-generated)
    client_id: String (required, unique, max_length: 100)
    client_secret_hash: String (required, max_length: 255)
    name: String (required, max_length: 200)
    client_type: String (required, max_length: 20) // 'internal', 'external', 'partner'
    permissions: JSON (required, default: [])
    rate_limit: JSON (required)
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
    last_activity: Timestamp
}

// API keys
Table api_keys {
    id: UUID (primary key, auto-generated)
    key_id: String (required, unique, max_length: 100)
    key_hash: String (required, max_length: 255)
    client_id: UUID (required, foreign_key: api_clients.id)
    name: String (required, max_length: 200)
    permissions: JSON (required, default: [])
    rate_limit: JSON
    expires_at: Timestamp
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
    last_used: Timestamp
}

// API usage tracking
Table api_usage {
    id: UUID (primary key, auto-generated)
    client_id: UUID (required, foreign_key: api_clients.id)
    api_key_id: UUID (required, foreign_key: api_keys.id)
    endpoint: String (required, max_length: 200)
    method: String (required, max_length: 10)
    status_code: Integer (required)
    response_time_ms: Float
    request_size_bytes: Integer
    response_size_bytes: Integer
    timestamp: Timestamp (default: now)
    user_agent: String
    ip_address: String
}

// Rate limiting buckets
Table rate_limit_buckets {
    id: UUID (primary key, auto-generated)
    client_id: UUID (required, foreign_key: api_clients.id)
    bucket_type: String (required, max_length: 20) // 'second', 'minute', 'hour', 'day'
    bucket_key: String (required, max_length: 100)
    current_count: Integer (default: 0)
    limit_value: Integer (required)
    window_start: Timestamp (required)
    window_end: Timestamp (required)
    created_at: Timestamp (default: now)

    // Constraints
    unique_client_bucket_window: (client_id, bucket_type, bucket_key, window_start)
}

// Indexes
idx_api_clients_status: (status)
idx_api_keys_client_status: (client_id, status)
idx_api_usage_client_timestamp: (client_id, timestamp DESC)
idx_rate_limit_client_type_window: (client_id, bucket_type, window_end)
```

### Query Side (TimescaleDB + Redis)
```pseudo
// API metrics time series
Table api_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    endpoint: String (required, max_length: 200)
    method: String (required, max_length: 10)
    status_code: Integer (required)
    request_count: Integer (default: 1)
    avg_response_time_ms: Float
    p99_response_time_ms: Float
    error_count: Integer (default: 0)
    total_bytes_transferred: Integer (default: 0)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Client usage analytics
Table client_usage_analytics {
    timestamp: Timestamp (required, partition_key)
    client_id: UUID (required)
    total_requests: Integer (default: 0)
    successful_requests: Integer (default: 0)
    failed_requests: Integer (default: 0)
    avg_response_time_ms: Float
    data_transferred_mb: Float
    rate_limit_hits: Integer (default: 0)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Indexes
idx_api_metrics_endpoint_time: (endpoint, timestamp DESC)
idx_client_analytics_client_time: (client_id, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache api_cache {
    // Market data cache
    "market_data:{symbol}:latest": MarketDataSnapshot (TTL: 5s)

    // Historical data cache
    "historical:{symbol}:{interval}:{start}:{end}": HistoricalData (TTL: 1h)

    // Rate limit counters
    "rate_limit:{client_id}:{bucket_type}:{window}": Integer (TTL: window_duration)

    // Authentication cache
    "auth:{token_hash}": ClientInfo (TTL: 15m)

    // API key cache
    "api_key:{key_hash}": APIKeyInfo (TTL: 1h)
}
```

## Implementation Estimation

### Priority: **HIGH** (External interface for market data)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core API Framework
- Go service setup with Gin framework
- Authentication and authorization system
- Basic REST endpoints for market data access
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- WebSocket streaming implementation
- Rate limiting and throttling
- Caching layer with Redis
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Security & Performance
- API security hardening
- Performance optimization and caching
- Bulk data endpoints
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 5: Integration & Testing
- Integration with Data Distribution Service
- Load testing and performance validation
- API documentation and client SDKs
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies:
- Data Distribution Service operational
- Redis cluster for caching
- Authentication infrastructure

### Risk Factors:
- **Medium**: High-concurrency performance requirements
- **Medium**: Security and authentication complexity
- **Low**: API design and implementation

### Success Criteria:
- Handle 10K+ concurrent API requests
- P99 API response time < 100ms
- 99.9% API uptime
- Comprehensive rate limiting and security
- Support for both REST and WebSocket clients
