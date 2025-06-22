# Pattern Recognition Service

## Responsibility
Chart pattern detection and candlestick pattern recognition using computer vision and machine learning. Identifies classic technical patterns, candlestick formations, and custom pattern definitions with confidence scoring.

## Technology Stack
- **Language**: Python + OpenCV + TensorFlow + scikit-image
- **Libraries**: OpenCV, TensorFlow, scikit-image, TA-Lib, custom pattern libraries
- **Scaling**: Horizontal by pattern complexity, GPU acceleration for ML models
- **NFRs**: P99 pattern detection < 1s, 90% pattern recognition accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum PatternType {
    // Chart Patterns
    HEAD_AND_SHOULDERS,
    DOUBLE_TOP,
    DOUBLE_BOTTOM,
    TRIANGLE_ASCENDING,
    CUP_AND_HANDLE,

    // Candlestick Patterns
    DOJI,
    HAMMER,
    ENGULFING_BULL,
    MORNING_STAR
}

enum CompletionStatus {
    FORMING,
    COMPLETED,
    BROKEN
}

// Data Models
struct PatternRecognitionRequest {
    instrument_id: String
    timeframe: String
    pattern_types: List<PatternType>
    lookback_periods: Integer
    min_confidence: Float
}

struct DetectedPattern {
    pattern_type: PatternType
    confidence: Float
    start_time: DateTime
    end_time: DateTime
    key_points: List<Map<String, Float>>
    pattern_target: Optional<Float>
    completion_status: CompletionStatus
}

struct PatternRecognitionResponse {
    instrument_id: String
    timeframe: String
    timestamp: DateTime
    detected_patterns: List<DetectedPattern>
    processing_time_ms: Float
}

// REST API Endpoints
POST /api/v1/patterns/recognize
    Request: PatternRecognitionRequest
    Response: PatternRecognitionResponse
```

### Event Output
```pseudo
Event pattern_detected {
    event_id: String
    timestamp: DateTime
    pattern_detection: PatternDetectionData
}

struct PatternDetectionData {
    instrument_id: String
    timeframe: String
    detected_patterns: List<DetectedPattern>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    pattern_detection: {
        instrument_id: "AAPL",
        timeframe: "1h",
        detected_patterns: [
            {
                pattern_type: "CUP_AND_HANDLE",
                confidence: 0.87,
                completion_status: "COMPLETED",
                pattern_target: 152.50
            }
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table pattern_definitions {
    id: UUID (primary key, auto-generated)
    pattern_name: String (required, unique, max_length: 100)
    pattern_type: String (required, max_length: 50)
    detection_rules: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table pattern_performance {
    id: UUID (primary key, auto-generated)
    pattern_type: String (required, max_length: 50)
    instrument_id: String (required, max_length: 20)
    detection_date: Date (required)
    predicted_target: Float
    actual_outcome: Float
    success: Boolean
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table pattern_detections_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    pattern_type: String (required, max_length: 50)
    confidence: Float (required)
    key_points: JSON
    pattern_target: Float
    completion_status: String (max_length: 20)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **MEDIUM** (Advanced analysis feature)
### Estimated Time: **6-7 weeks**

#### Week 1-2: Core Pattern Engine
- Python service setup with OpenCV and TensorFlow
- Basic chart pattern detection algorithms
- Candlestick pattern recognition with TA-Lib
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Pattern Recognition
- Computer vision-based pattern detection
- ML model training for pattern classification
- Custom pattern definition framework
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 5-7: Integration & Testing
- Pattern reliability scoring and validation
- Integration with market data services
- Performance optimization and GPU acceleration
- **Effort**: 2 developers × 3 weeks = 6 dev-weeks

### Total Effort: **14 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: Market data, technical indicators, GPU infrastructure

### Success Criteria:
- 90% pattern recognition accuracy
- P99 detection latency < 1 second
- Support for 20+ pattern types
- Real-time pattern formation tracking
