# Analysis Cache Service

## Responsibility
High-performance multi-tier caching service for analysis results, technical indicators, and computed data. Provides sub-10ms cache response times with intelligent cache warming and memory optimization for the entire instrument analysis workflow.

## Technology Stack
- **Language**: Go + Redis + InfluxDB + in-memory caching
- **Libraries**: go-redis, influxdb-client-go, sync.Map, groupcache
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal with Redis Cluster, vertical for memory optimization
- **NFRs**: P99 cache response < 10ms, 95% hit ratio, 99.99% availability, 80% memory efficiency

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CacheType {
    TECHNICAL_INDICATORS,
    CORRELATION_MATRICES,
    PATTERN_RESULTS,
    ANOMALY_DETECTIONS,
    RISK_METRICS,
    ANALYSIS_SUMMARIES
}

enum CacheTier {
    L1_MEMORY,      // In-memory hot cache
    L2_REDIS,       // Redis warm cache
    L3_INFLUXDB     // InfluxDB cold storage
}

// Data Models
struct CacheRequest {
    key: String
    cache_type: CacheType
    ttl_seconds: Optional<Integer>
    tier_preference: Optional<CacheTier>
}

struct CacheResponse {
    key: String
    value: JSON
    hit_tier: CacheTier
    response_time_ms: Float
    expiry: DateTime
}

struct CacheStats {
    total_requests: Integer
    hit_ratio_l1: Float
    hit_ratio_l2: Float
    hit_ratio_l3: Float
    avg_response_time_ms: Float
    memory_usage_mb: Float
}

// REST API Endpoints
GET /api/v1/cache/{key}
    Parameters: cache_type, tier_preference
    Response: CacheResponse

PUT /api/v1/cache/{key}
    Request: CacheRequest + value
    Response: Success/Error

DELETE /api/v1/cache/{key}
    Response: Success/Error

GET /api/v1/cache/stats
    Response: CacheStats

POST /api/v1/cache/warm
    Request: List<String> (keys to warm)
    Response: WarmingStatus
```

### Event Input/Output
```pseudo
// Input Events (Cache Invalidation)
Event cache_invalidation_requested {
    event_id: String
    timestamp: DateTime
    invalidation_data: InvalidationData
}

struct InvalidationData {
    keys: List<String>
    pattern: Optional<String>
    cache_type: CacheType
    reason: String
}

// Output Events (Cache Performance)
Event cache_performance_metrics {
    event_id: String
    timestamp: DateTime
    performance_data: CachePerformanceData
}

struct CachePerformanceData {
    hit_ratio: Float
    avg_response_time_ms: Float
    memory_usage_mb: Float
    requests_per_second: Float
    cache_efficiency: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    performance_data: {
        hit_ratio: 0.95,
        avg_response_time_ms: 5.2,
        memory_usage_mb: 2048.5,
        requests_per_second: 15000,
        cache_efficiency: 0.88
    }
}
```

## Data Model & Database Schema

### Redis (L2 Cache)
```pseudo
Cache technical_indicators {
    // Latest indicators
    "indicators:{instrument_id}:{timeframe}": IndicatorData (TTL: 1m)
    
    // Correlation matrices
    "correlation:{date}:{cluster_id}": CorrelationMatrix (TTL: 1h)
    
    // Pattern results
    "patterns:{instrument_id}:{timeframe}": PatternResults (TTL: 5m)
    
    // Risk metrics
    "risk:{instrument_id}:{date}": RiskMetrics (TTL: 30m)
}

Cache metadata {
    // Cache statistics
    "stats:cache_performance": CacheStats (TTL: 1m)
    
    // Cache warming queue
    "warm:queue": List<String> (TTL: 5m)
    
    // Cache invalidation log
    "invalidation:log": List<InvalidationEvent> (TTL: 1h)
}
```

### InfluxDB (L3 Storage)
```pseudo
Table cache_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    cache_type: String (required, max_length: 50)
    operation: String (required, max_length: 20) // GET, PUT, DELETE
    hit_tier: String (max_length: 20)
    response_time_ms: Float
    cache_size_mb: Float
    hit_ratio: Float
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: cache_type (partitions: 8)
}

Table cache_performance_ts {
    timestamp: Timestamp (required, partition_key)
    total_requests: Integer
    hit_ratio_l1: Float
    hit_ratio_l2: Float
    hit_ratio_l3: Float
    memory_usage_mb: Float
    cpu_usage_percent: Float
    network_io_mbps: Float
}
```

### In-Memory (L1 Cache)
```pseudo
Cache hot_cache {
    // Most frequently accessed data
    sync.Map[string]CachedItem
    
    // LRU eviction policy
    max_size: 1GB
    max_items: 100000
    ttl_default: 30s
}

struct CachedItem {
    value: []byte
    expiry: time.Time
    access_count: int64
    last_access: time.Time
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Performance foundation)
### Estimated Time: **3-4 weeks**

#### Week 1: Core Cache Infrastructure
- Go service setup with Redis and InfluxDB clients
- Multi-tier cache implementation (L1, L2, L3)
- Basic cache operations (get, set, delete)
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 2: Intelligent Caching
- Cache warming strategies and predictive preloading
- LRU eviction policies and memory optimization
- Cache hit ratio optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 3: Performance Optimization
- Memory-efficient data structures
- Parallel cache operations
- Connection pooling and optimization
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 4: Integration & Monitoring
- Integration with analysis services
- Prometheus metrics and monitoring
- Performance testing and validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **7 dev-weeks**
### Team Size: **2 developers (1 senior Go developer + 1 developer)**
### Dependencies: Redis Cluster, InfluxDB, Prometheus

### Success Criteria:
- P99 cache response time < 10ms
- 95% cache hit ratio for hot data
- 80% memory utilization efficiency
- 99.99% cache availability
- 100K+ cache operations per second
