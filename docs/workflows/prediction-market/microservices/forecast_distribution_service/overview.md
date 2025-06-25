# Forecast Distribution Service

## Responsibility
High-performance prediction caching and distribution with multi-timeframe prediction management, cache invalidation, prediction versioning, and low-latency prediction serving for real-time trading applications.

## Technology Stack
- **Language**: Go + Redis + Apache Pulsar + Prometheus
- **Libraries**: Go-Redis, Pulsar client, Prometheus client, JSON serialization
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by cache volume, prediction type partitioning
- **NFRs**: P99 cache latency < 10ms, 50K+ cache ops/sec throughput, 99.9% cache availability

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CacheOperation {
    GET,
    SET,
    DELETE,
    INVALIDATE,
    REFRESH
}

enum PredictionStatus {
    FRESH,
    STALE,
    EXPIRED,
    INVALID
}

enum CacheLevel {
    L1_MEMORY,
    L2_REDIS,
    L3_DATABASE
}

// Data Models
struct CacheRequest {
    operation: CacheOperation
    cache_key: String
    prediction_data: Optional<PredictionData>
    ttl_seconds: Optional<Integer>
    cache_level: CacheLevel
}

struct CachedPrediction {
    cache_key: String
    prediction_data: PredictionData
    cached_at: DateTime
    expires_at: DateTime
    version: String
    status: PredictionStatus
    hit_count: Integer
}

struct PredictionData {
    instrument_id: String
    timeframe: String
    prediction: MarketPrediction
    metadata: PredictionMetadata
}

struct PredictionMetadata {
    model_version: String
    confidence_score: Float
    generation_time: DateTime
    data_freshness: Float
}

struct CacheStats {
    total_requests: Integer
    cache_hits: Integer
    cache_misses: Integer
    hit_ratio: Float
    average_latency_ms: Float
    memory_usage_mb: Float
    eviction_count: Integer
}

// REST API Endpoints
GET /api/v1/cache/predictions/{instrument_id}/{timeframe}
    Response: CachedPrediction

POST /api/v1/cache/predictions
    Request: CacheRequest
    Response: CacheResult

DELETE /api/v1/cache/predictions/{cache_key}
    Response: DeletionResult

POST /api/v1/cache/invalidate
    Request: InvalidationRequest
    Response: InvalidationResult

GET /api/v1/cache/stats
    Response: CacheStats

POST /api/v1/cache/refresh/{instrument_id}
    Response: RefreshResult
```

### Event Output
```pseudo
Event prediction_cached {
    event_id: String
    timestamp: DateTime
    prediction_cached: PredictionCachedData
}

struct PredictionCachedData {
    cache_batch_id: String
    instrument_id: String
    timeframe: String
    cache_operation: String
    cache_level: String
    operation_duration_ms: Integer
    cache_size_bytes: Integer
    ttl_seconds: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:35:00.000Z",
    prediction_cached: {
        cache_batch_id: "cache_20250621_001",
        instrument_id: "AAPL",
        timeframe: "1d",
        cache_operation: "SET",
        cache_level: "L2_REDIS",
        operation_duration_ms: 8,
        cache_size_bytes: 2048,
        ttl_seconds: 600
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table cache_operations {
    id: UUID (primary key, auto-generated)
    operation_id: String (required, unique, max_length: 100)
    cache_key: String (required, max_length: 200)
    operation_type: String (required, max_length: 20)
    cache_level: String (required, max_length: 20)
    operation_duration_ms: Float
    cache_size_bytes: Integer
    success: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table cache_invalidations {
    id: UUID (primary key, auto-generated)
    invalidation_id: String (required, unique, max_length: 100)
    invalidation_pattern: String (required, max_length: 200)
    affected_keys_count: Integer
    invalidation_reason: String (max_length: 100)
    invalidated_at: Timestamp (default: now)
}

Table cache_performance {
    id: UUID (primary key, auto-generated)
    measurement_id: String (required, unique, max_length: 100)
    cache_level: String (required, max_length: 20)
    hit_ratio: Float (required)
    average_latency_ms: Float (required)
    throughput_ops_per_sec: Float (required)
    memory_usage_mb: Float
    measured_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache prediction_cache {
    // Cached predictions by instrument and timeframe
    "pred:{instrument_id}:{timeframe}": CachedPrediction (TTL: 10m)
    
    // Prediction metadata
    "pred_meta:{instrument_id}:{timeframe}": PredictionMetadata (TTL: 10m)
    
    // Cache statistics
    "cache_stats": CacheStats (TTL: 1m)
    
    // Invalidation patterns
    "invalidation_patterns": List<String> (TTL: 1h)
    
    // Cache health status
    "cache_health": CacheHealthStatus (TTL: 30s)
}
```

## Implementation Estimation

### Priority: **HIGH** (Performance critical caching layer)
### Estimated Time: **2-3 weeks**

#### Week 1: Core Caching Engine
- Go service setup with Redis integration
- Basic cache operations (GET, SET, DELETE)
- Multi-level caching implementation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 2: Advanced Features
- Cache invalidation and refresh mechanisms
- Prediction versioning and metadata
- Performance monitoring and metrics
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 3: Integration & Optimization
- Integration with prediction services
- Performance optimization and tuning
- High availability and failover
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **6 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 caching specialist)
### Dependencies: Redis infrastructure, Market Prediction Engine Service

### Success Criteria:
- P99 cache latency < 10ms
- 50K+ cache operations per second
- 99.9% cache availability
- Multi-level caching capability
- Intelligent cache invalidation
- Real-time performance monitoring
