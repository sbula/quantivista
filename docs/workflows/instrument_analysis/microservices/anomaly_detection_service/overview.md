# Anomaly Detection Service

## Responsibility
Advanced statistical and machine learning-based anomaly detection for market data, technical indicators, and trading patterns. Identifies outliers, unusual patterns, and potential market disruptions with 85% accuracy and sub-5-minute detection time.

## Technology Stack
- **Language**: Python + scikit-learn + scipy + TensorFlow
- **Libraries**: pandas, numpy, isolation-forest, LSTM, autoencoder
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by instrument groups, vertical for ML model complexity
- **NFRs**: 85% detection accuracy, <15% false positive rate, 95% detection within 5 minutes

## API Specification

### Core APIs
```pseudo
// Enumerations
enum AnomalyType {
    PRICE_SPIKE,
    VOLUME_ANOMALY,
    PATTERN_DEVIATION,
    CORRELATION_BREAKDOWN,
    STATISTICAL_OUTLIER,
    TIME_SERIES_ANOMALY
}

enum AnomalySeverity {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}

enum DetectionMethod {
    Z_SCORE,
    ISOLATION_FOREST,
    LSTM_AUTOENCODER,
    LOCAL_OUTLIER_FACTOR,
    ONE_CLASS_SVM
}

// Data Models
struct AnomalyRequest {
    instrument_id: String
    timeframe: String
    data_points: List<DataPoint>
    detection_methods: List<DetectionMethod>
    sensitivity: Float  // 0.0 to 1.0
}

struct AnomalyResponse {
    instrument_id: String
    timestamp: DateTime
    anomalies: List<AnomalyDetection>
    detection_time_ms: Float
    confidence_score: Float
}

struct AnomalyDetection {
    anomaly_type: AnomalyType
    severity: AnomalySeverity
    confidence: Float
    value: Float
    expected_range: Range
    detection_method: DetectionMethod
    context: AnomalyContext
}

struct AnomalyContext {
    market_condition: String
    related_instruments: List<String>
    potential_causes: List<String>
    impact_assessment: String
}

// REST API Endpoints
POST /api/v1/anomalies/detect
    Request: AnomalyRequest
    Response: AnomalyResponse

GET /api/v1/anomalies/{instrument_id}/latest
    Parameters: timeframe, severity_min
    Response: List<AnomalyDetection>

GET /api/v1/anomalies/summary
    Parameters: date_range, severity_min
    Response: AnomalySummary

POST /api/v1/anomalies/feedback
    Request: AnomalyFeedback
    Response: Success/Error
```

### Event Output
```pseudo
Event anomaly_detected {
    event_id: String
    timestamp: DateTime
    anomaly_data: AnomalyEventData
}

struct AnomalyEventData {
    instrument_id: String
    anomaly_type: AnomalyType
    severity: AnomalySeverity
    confidence: Float
    value: Float
    expected_range: Range
    detection_method: DetectionMethod
    context: AnomalyContext
    alert_required: Boolean
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    anomaly_data: {
        instrument_id: "AAPL",
        anomaly_type: "PRICE_SPIKE",
        severity: "HIGH",
        confidence: 0.92,
        value: 165.50,
        expected_range: {
            min: 148.20,
            max: 152.80
        },
        detection_method: "ISOLATION_FOREST",
        context: {
            market_condition: "HIGH_VOLATILITY",
            related_instruments: ["MSFT", "GOOGL"],
            potential_causes: ["EARNINGS_SURPRISE", "NEWS_EVENT"],
            impact_assessment: "SIGNIFICANT_PRICE_MOVEMENT"
        },
        alert_required: true
    }
}
```

## Data Model & Database Schema

### PostgreSQL (Command Side)
```pseudo
Table anomaly_configurations {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    anomaly_type: String (required, max_length: 50)
    detection_method: String (required, max_length: 50)
    sensitivity: Float (default: 0.5)
    enabled: Boolean (default: true)
    thresholds: JSON (required)
    created_at: Timestamp (default: now)
    
    // Constraints
    unique_instrument_type_method: (instrument_id, anomaly_type, detection_method)
}

Table anomaly_feedback {
    id: UUID (primary key, auto-generated)
    anomaly_id: UUID (required, foreign_key: anomaly_detections.id)
    feedback_type: String (required, max_length: 20) // TRUE_POSITIVE, FALSE_POSITIVE
    user_id: String (max_length: 50)
    comments: Text
    created_at: Timestamp (default: now)
}

Table model_performance {
    id: UUID (primary key, auto-generated)
    model_type: String (required, max_length: 50)
    accuracy: Float
    precision: Float
    recall: Float
    f1_score: Float
    false_positive_rate: Float
    evaluation_date: Date
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table anomaly_detections_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    anomaly_type: String (required, max_length: 50)
    severity: String (required, max_length: 20)
    confidence: Float (required)
    value: Float
    expected_min: Float
    expected_max: Float
    detection_method: String (max_length: 50)
    context: JSON
    alert_sent: Boolean (default: false)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 16)
}

Table anomaly_statistics_ts {
    timestamp: Timestamp (required, partition_key)
    anomaly_type: String (required, max_length: 50)
    total_detections: Integer
    high_severity_count: Integer
    avg_confidence: Float
    false_positive_rate: Float
    detection_latency_ms: Float
}
```

### Redis Caching
```pseudo
Cache anomaly_cache {
    // Recent anomalies
    "anomalies:{instrument_id}:latest": List<AnomalyDetection> (TTL: 5m)
    
    // Detection models
    "models:{anomaly_type}:{method}": SerializedModel (TTL: 1h)
    
    // Statistical baselines
    "baseline:{instrument_id}:{timeframe}": StatisticalBaseline (TTL: 30m)
    
    // Alert status
    "alerts:{instrument_id}:status": AlertStatus (TTL: 10m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Risk management critical)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Detection Engine
- Python service setup with scikit-learn and scipy
- Basic statistical anomaly detection (Z-score, IQR)
- Isolation Forest and LOF implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced ML Models
- LSTM autoencoder for time series anomalies
- One-Class SVM implementation
- Model training and validation pipeline
- **Effort**: 1 senior ML engineer × 1 week = 1 dev-week

#### Week 4: Real-Time Processing
- Real-time anomaly detection pipeline
- Event publishing and alerting
- Performance optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 5: Integration & Validation
- Integration with technical indicator service
- Accuracy validation and model tuning
- False positive reduction optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers (1 senior ML engineer + 1 Python developer)**
### Dependencies: Technical Indicator Service, Market Data, TimescaleDB

### Success Criteria:
- 85% minimum anomaly detection accuracy
- <15% false positive rate
- 95% of anomalies detected within 5 minutes
- Support for 6+ anomaly types
- Real-time processing capability
