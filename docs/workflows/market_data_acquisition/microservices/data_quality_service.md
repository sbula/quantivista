# Data Quality Service

## Responsibility
Centralized quality assurance and validation for all ingested market data. Performs multi-level validation, cross-source verification, anomaly detection, and quality scoring. Ensures data integrity and reliability for downstream trading systems.

## Technology Stack
- **Language**: Python + asyncio for concurrent processing
- **Libraries**: Pandas, NumPy, scikit-learn for anomaly detection
- **Scaling**: Horizontal by instrument groups
- **NFRs**: P99 validation latency < 100ms, 99.99% accuracy in anomaly detection

## API Specification

### Internal APIs

#### Quality Validation API
```pseudo
// Data Models
struct QualityValidationRequest {
    raw_data: JsonObject
    provider: String
    symbol: String
    timestamp: DateTime
    validation_level: String  // "basic", "standard", "full"
}

struct QualityValidationResponse {
    is_valid: Boolean
    quality_score: Float  // 0.0 to 1.0
    validation_results: Map<String, Boolean>
    anomalies_detected: List<String>
    confidence_level: Float
    processing_time_ms: Float
}

struct QualityScore {
    symbol: String
    timeframe: String
    score: Float
    timestamp: DateTime
}

// REST API Endpoints
POST /api/v1/validate
    Request: QualityValidationRequest
    Response: QualityValidationResponse

GET /api/v1/quality/score/{symbol}
    Parameters: timeframe (optional, default: "1h")
    Response: QualityScore

GET /api/v1/quality/report/{symbol}
    Parameters: start_date, end_date
    Response: QualityReport
```

#### Quality Metrics API
```pseudo
// Enumerations
enum AlertType {
    QUALITY_DEGRADATION,
    ANOMALY_DETECTED,
    DATA_GAP,
    FORMAT_ERROR,
    RANGE_VIOLATION
}

enum AlertSeverity {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}

// Data Models
struct QualityMetrics {
    symbol: String
    provider: String
    timeframe: String
    completeness: Float
    accuracy: Float
    timeliness: Float
    consistency: Float
    overall_score: Float
    sample_size: Integer
    last_updated: DateTime
}

struct QualityAlert {
    alert_id: String
    symbol: String
    provider: String
    alert_type: AlertType
    severity: AlertSeverity
    description: String
    quality_score: Float
    threshold: Float
    created_at: DateTime
}
```

### Event Output

#### DataQualityValidatedEvent
```pseudo
Event data_quality_validated {
    event_id: String
    timestamp: DateTime
    validation: ValidationData
    quality_metrics: QualityMetricsData
    anomalies: List<String>
    processing_time_ms: Integer
}

struct ValidationData {
    symbol: String
    provider: String
    is_valid: Boolean
    quality_score: Float
    validation_results: ValidationResultsData
}

struct ValidationResultsData {
    format_valid: Boolean
    range_valid: Boolean
    sequence_valid: Boolean
    cross_source_valid: Boolean
    anomaly_free: Boolean
}

struct QualityMetricsData {
    completeness: Float
    accuracy: Float
    timeliness: Float
    consistency: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T09:30:00.150Z",
    validation: {
        symbol: "AAPL",
        provider: "bloomberg",
        is_valid: true,
        quality_score: 0.95,
        validation_results: {
            format_valid: true,
            range_valid: true,
            sequence_valid: true,
            cross_source_valid: true,
            anomaly_free: true
        }
    },
    quality_metrics: {
        completeness: 0.98,
        accuracy: 0.96,
        timeliness: 0.92,
        consistency: 0.94
    },
    anomalies: [],
    processing_time_ms: 45
}
```

#### DataQualityAlertEvent
```pseudo
Event data_quality_alert_generated {
    event_id: String
    timestamp: DateTime
    alert: QualityAlertData
    context: AlertContextData
}

struct QualityAlertData {
    alert_id: String
    symbol: String
    provider: String
    alert_type: String
    severity: String
    description: String
    current_score: Float
    threshold: Float
    trend: String
}

struct AlertContextData {
    recent_scores: List<Float>
    provider_status: String
    market_conditions: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T09:30:00.200Z",
    alert: {
        alert_id: "alert-12345",
        symbol: "AAPL",
        provider: "reuters",
        alert_type: "quality_degradation",
        severity: "high",
        description: "Quality score dropped below threshold",
        current_score: 0.65,
        threshold: 0.8,
        trend: "declining"
    },
    context: {
        recent_scores: [0.85, 0.78, 0.72, 0.65],
        provider_status: "connected",
        market_conditions: "high_volatility"
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct QualityValidation {
    symbol: String
    provider: String
    timestamp: DateTime
    quality_score: Float
    validation_results: Map<String, Boolean>
    anomalies: List<String>
    processing_time_ms: Float
}

struct QualityMetrics {
    symbol: String
    provider: String
    timeframe: String
    completeness: Float
    accuracy: Float
    timeliness: Float
    consistency: Float
    sample_size: Integer
    calculated_at: DateTime
}

struct AnomalyDetection {
    symbol: String
    provider: String
    anomaly_type: String
    severity: Float
    description: String
    detected_at: DateTime
    context: Map<String, Any>
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Quality validation rules and configuration
Table quality_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    rule_type: String (required, max_length: 50) // 'format', 'range', 'sequence', 'cross_source'
    rule_config: JSON (required)
    enabled: Boolean (default: true)
    priority: Integer (default: 1)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Quality thresholds by symbol and provider
Table quality_thresholds {
    id: UUID (primary key, auto-generated)
    symbol: String (required, max_length: 20)
    provider: String (max_length: 50)
    metric_type: String (required, max_length: 50) // 'completeness', 'accuracy', 'timeliness', 'consistency'
    threshold_value: Float (required)
    alert_enabled: Boolean (default: true)
    created_at: Timestamp (default: now)

    // Constraints
    unique_symbol_provider_metric: (symbol, provider, metric_type)
}

// Quality validation results (command side)
Table quality_validations {
    id: UUID (primary key, auto-generated)
    symbol: String (required, max_length: 20)
    provider: String (required, max_length: 50)
    timestamp: Timestamp (required)
    quality_score: Float (required)
    validation_results: JSON (required)
    anomalies: JSON
    processing_time_ms: Float
    created_at: Timestamp (default: now)
}

// Quality alerts
Table quality_alerts {
    id: UUID (primary key, auto-generated)
    alert_id: String (required, unique, max_length: 100)
    symbol: String (required, max_length: 20)
    provider: String (required, max_length: 50)
    alert_type: String (required, max_length: 50)
    severity: String (required, max_length: 20)
    description: String
    quality_score: Float
    threshold_value: Float
    status: String (default: 'active', max_length: 20) // 'active', 'acknowledged', 'resolved'
    created_at: Timestamp (default: now)
    resolved_at: Timestamp
}

// Indexes
idx_quality_validations_symbol_time: (symbol, timestamp DESC)
idx_quality_validations_provider_time: (provider, timestamp DESC)
idx_quality_alerts_symbol_status: (symbol, status)
idx_quality_alerts_severity_created: (severity, created_at DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Quality metrics time series
Table quality_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    symbol: String (required, max_length: 20)
    provider: String (required, max_length: 50)
    timeframe: String (required, max_length: 10) // '1m', '5m', '1h', '1d'
    completeness: Float
    accuracy: Float
    timeliness: Float
    consistency: Float
    overall_score: Float
    sample_size: Integer
    calculated_at: Timestamp (default: now)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Anomaly detection results
Table anomaly_detections {
    timestamp: Timestamp (required, partition_key)
    symbol: String (required, max_length: 20)
    provider: String (required, max_length: 50)
    anomaly_type: String (required, max_length: 50)
    severity: Float (required)
    description: String
    context: JSON
    detected_at: Timestamp (default: now)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Indexes for fast queries
idx_quality_metrics_symbol_time: (symbol, timestamp DESC)
idx_anomaly_detections_symbol_time: (symbol, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache quality_cache {
    // Current quality scores
    "quality:{symbol}:{provider}": QualityScore (TTL: 5m)

    // Quality trends
    "quality_trend:{symbol}:{timeframe}": List<QualityScore> (TTL: 1h)

    // Alert status
    "alert_status:{symbol}:{provider}": AlertStatus (TTL: 10m)

    // Validation cache
    "validation:{symbol}:{timestamp_hash}": ValidationResult (TTL: 30m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for data integrity)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Validation Framework
- Basic Python service setup with asyncio
- Quality validation rule engine
- Multi-level validation implementation (format, range, sequence)
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Quality Features
- Cross-source validation and consensus building
- Anomaly detection using statistical methods
- Quality scoring algorithm implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Machine Learning Integration
- ML-based anomaly detection models
- Quality prediction and trend analysis
- Model training and validation pipeline
- **Effort**: 1 ML engineer × 1 week = 1 dev-week

#### Week 6: Integration & Testing
- Integration with Data Ingestion Service
- Performance testing and optimization
- Alert system integration
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers + 1 ML engineer**
### Dependencies:
- Data Ingestion Service operational
- TimescaleDB and PostgreSQL setup
- Apache Pulsar for event streaming

### Risk Factors:
- **Medium**: ML model accuracy for anomaly detection
- **Low**: Performance requirements for real-time validation
- **Low**: Integration complexity

### Success Criteria:
- Validate 10,000+ data points per second
- Achieve 99.99% accuracy in anomaly detection
- P99 validation latency < 100ms
- Quality score accuracy > 95%
- Alert generation within 1 second of quality degradation
