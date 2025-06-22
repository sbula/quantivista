# Sentiment Analysis Service

## Responsibility
Financial sentiment analysis using domain-specific models, multi-language support, and context-aware sentiment scoring for market-relevant content.

## Technology Stack
- **Language**: Python + Transformers + spaCy + VADER
- **Models**: FinBERT, custom financial sentiment models
- **Scaling**: Horizontal by content volume, GPU acceleration
- **NFRs**: P99 analysis latency < 200ms, 95% sentiment accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum Language {
    ENGLISH,
    SPANISH,
    GERMAN,
    FRENCH,
    CHINESE,
    JAPANESE
}

// Data Models
struct SentimentAnalysisRequest {
    content_id: String
    text: String
    language: Language
    context: Optional<Map<String, Any>>
    entities: Optional<List<String>>
}

struct SentimentAnalysisResponse {
    content_id: String
    overall_sentiment: Float  // -1.0 to 1.0
    confidence: Float
    entity_sentiments: Map<String, Float>
    sentiment_breakdown: Map<String, Float>
    processing_time_ms: Float
}

// REST API Endpoints
POST /api/v1/sentiment/analyze
    Request: SentimentAnalysisRequest
    Response: SentimentAnalysisResponse

POST /api/v1/sentiment/batch-analyze
    Request: List<SentimentAnalysisRequest>
    Response: List<SentimentAnalysisResponse>
```

### Event Output
```pseudo
Event sentiment_analysis_completed {
    event_id: String
    timestamp: DateTime
    sentiment_analysis: SentimentAnalysisData
}

struct SentimentAnalysisData {
    content_id: String
    overall_sentiment: Float
    confidence: Float
    entity_sentiments: EntitySentimentsData
    sentiment_breakdown: SentimentBreakdownData
}

struct EntitySentimentsData {
    AAPL: Float
    iPhone: Float
    earnings: Float
}

struct SentimentBreakdownData {
    positive: Float
    neutral: Float
    negative: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    sentiment_analysis: {
        content_id: "news_article_001",
        overall_sentiment: 0.75,
        confidence: 0.89,
        entity_sentiments: {
            AAPL: 0.82,
            iPhone: 0.68,
            earnings: 0.71
        },
        sentiment_breakdown: {
            positive: 0.78,
            neutral: 0.15,
            negative: 0.07
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table sentiment_models {
    id: UUID (primary key, auto-generated)
    model_name: String (required, max_length: 100)
    model_type: String (required, max_length: 50) // 'transformer', 'lexicon', 'hybrid'
    model_config: JSON (required)
    accuracy_score: Float
    language: String (default: 'en', max_length: 10)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table sentiment_calibration {
    id: UUID (primary key, auto-generated)
    content_type: String (required, max_length: 50)
    model_name: String (required, max_length: 100)
    calibration_data: JSON (required)
    accuracy_metrics: JSON
    last_updated: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table sentiment_results_ts {
    timestamp: Timestamp (required, partition_key)
    content_id: String (required, max_length: 100)
    content_type: String (required, max_length: 50)
    overall_sentiment: Float (required)
    confidence: Float (required)
    entity_sentiments: JSON
    processing_time_ms: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core intelligence capability)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Sentiment Engine
- FinBERT integration and fine-tuning
- Multi-language sentiment analysis
- Entity-specific sentiment extraction
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Context-aware sentiment analysis
- Financial domain adaptation
- Sentiment calibration and validation
- **Effort**: 1 ML engineer × 1 week = 1 dev-week

#### Week 4-5: Integration & Optimization
- GPU optimization and batching
- Integration with content services
- Performance testing and monitoring
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: GPU infrastructure, pre-trained models

### Success Criteria:
- 95% sentiment accuracy on financial content
- P99 analysis latency < 200ms
- Support for 5K+ analyses per minute
- Multi-language support (EN, ES, DE, FR)
