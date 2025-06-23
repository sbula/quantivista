# Metrics Collection Service

## Responsibility
High-performance metrics collection from all QuantiVista services and infrastructure components. Provides centralized metrics ingestion, processing, and storage with support for custom metrics, auto-discovery, and real-time streaming.

## Technology Stack
- **Language**: Go + Prometheus + OpenTelemetry + custom collectors
- **Libraries**: Prometheus client, OpenTelemetry, Kubernetes API, cloud provider SDKs
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by metric volume, sharded collection
- **NFRs**: P99 collection latency < 100ms, 1M+ metrics/sec throughput, 99.99% data integrity

## API Specification

### Core APIs
```pseudo
// Enumerations
enum MetricType {
    COUNTER,
    GAUGE,
    HISTOGRAM,
    SUMMARY,
    CUSTOM
}

enum CollectionMethod {
    PULL_SCRAPING,
    PUSH_GATEWAY,
    STREAMING,
    BATCH_UPLOAD
}

enum MetricSource {
    APPLICATION,
    INFRASTRUCTURE,
    KUBERNETES,
    CLOUD_PROVIDER,
    CUSTOM_EXPORTER
}

// Data Models
struct MetricCollectionRequest {
    source_id: String
    collection_method: CollectionMethod
    scrape_config: ScrapeConfig
    filters: List<MetricFilter>
    retention_policy: RetentionPolicy
}

struct ScrapeConfig {
    target_url: String
    scrape_interval: String
    scrape_timeout: String
    metrics_path: String
    scheme: String
    authentication: Optional<AuthConfig>
}

struct Metric {
    name: String
    metric_type: MetricType
    value: Float
    labels: Map<String, String>
    timestamp: DateTime
    source: MetricSource
    help_text: Optional<String>
}

struct MetricTarget {
    target_id: String
    target_url: String
    labels: Map<String, String>
    health_status: String
    last_scrape: DateTime
    scrape_duration_ms: Float
    samples_scraped: Integer
}

struct CollectionStats {
    total_targets: Integer
    healthy_targets: Integer
    total_metrics_collected: Integer
    collection_rate_per_second: Float
    average_scrape_duration_ms: Float
    error_rate: Float
}

// REST API Endpoints
POST /api/v1/metrics/targets
    Request: MetricCollectionRequest
    Response: MetricTarget

GET /api/v1/metrics/targets
    Response: List<MetricTarget>

POST /api/v1/metrics/custom
    Request: List<Metric>
    Response: IngestionResult

GET /api/v1/metrics/query
    Parameters: query, start, end, step
    Response: MetricQueryResult

GET /api/v1/metrics/stats
    Response: CollectionStats

DELETE /api/v1/metrics/targets/{target_id}
    Response: DeletionResult
```

### Event Output
```pseudo
Event metrics_collected {
    event_id: String
    timestamp: DateTime
    metrics_collected: MetricsCollectedData
}

struct MetricsCollectedData {
    collection_batch_id: String
    source_id: String
    metrics_count: Integer
    collection_duration_ms: Integer
    collection_method: String
    target_health: String
    metrics_summary: MetricsSummaryData
    error_count: Integer
    samples_per_second: Integer
}

struct MetricsSummaryData {
    counters: Integer
    gauges: Integer
    histograms: Integer
    summaries: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    metrics_collected: {
        collection_batch_id: "batch_20250621_001",
        source_id: "portfolio_management_service",
        metrics_count: 1250,
        collection_duration_ms: 85,
        collection_method: "PULL_SCRAPING",
        target_health: "healthy",
        metrics_summary: {
            counters: 450,
            gauges: 600,
            histograms: 150,
            summaries: 50
        },
        error_count: 0,
        samples_per_second: 14705
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table metric_targets {
    id: UUID (primary key, auto-generated)
    target_id: String (required, unique, max_length: 100)
    target_url: String (required, max_length: 500)
    source_type: String (required, max_length: 50)
    scrape_config: JSON (required)
    labels: JSON
    health_status: String (default: 'unknown', max_length: 20)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table collection_jobs {
    id: UUID (primary key, auto-generated)
    job_id: String (required, unique, max_length: 100)
    target_id: String (required, max_length: 100)
    collection_method: String (required, max_length: 50)
    status: String (required, max_length: 20)
    metrics_collected: Integer (default: 0)
    collection_duration_ms: Float
    error_details: String
    created_at: Timestamp (default: now)
}
```

### InfluxDB/TimescaleDB (Metrics Storage)
```pseudo
Table metrics_ts {
    timestamp: Timestamp (required, partition_key)
    metric_name: String (required, max_length: 200)
    metric_type: String (required, max_length: 20)
    value: Float (required)
    labels: JSON
    source: String (required, max_length: 100)
    target_id: String (required, max_length: 100)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension_1: metric_name (partitions: 16)
    partition_dimension_2: source (partitions: 8)
}
```

### Redis Caching
```pseudo
Cache metrics_cache {
    // Target health
    "target_health:{target_id}": HealthStatus (TTL: 5m)

    // Latest metrics
    "latest_metrics:{target_id}": List<Metric> (TTL: 2m)

    // Collection stats
    "collection_stats": CollectionStats (TTL: 1m)

    // Scrape schedule
    "scrape_schedule:{target_id}": NextScrapeTime (TTL: 10m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Foundation for all monitoring)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Collection Engine
- Go service setup with Prometheus integration
- Basic scraping and target management
- OpenTelemetry integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Collection Features
- Custom metrics ingestion
- Auto-discovery mechanisms
- Performance optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Integration & Scaling
- Integration with all QuantiVista services
- Horizontal scaling and sharding
- High-availability setup
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 monitoring specialist)
### Dependencies: Prometheus infrastructure, InfluxDB/TimescaleDB, all QuantiVista services

### Success Criteria:
- P99 collection latency < 100ms
- 1M+ metrics per second throughput
- 99.99% data integrity
- Auto-discovery of new services
- Support for custom metrics
- Horizontal scaling capability
