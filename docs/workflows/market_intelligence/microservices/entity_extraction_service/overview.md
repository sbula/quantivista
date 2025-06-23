# Entity Extraction Service

## Responsibility
Financial entity recognition and extraction from text content, including company names, ticker symbols, financial instruments, economic indicators, and key financial metrics.

## Technology Stack
- **Language**: Python + spaCy + Transformers + custom NER models
- **Models**: FinNER, custom financial entity models, fuzzy matching
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by content volume
- **NFRs**: P99 extraction latency < 150ms, 95% entity recognition accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum EntityType {
    COMPANY,
    TICKER_SYMBOL,
    FINANCIAL_METRIC,
    TECHNICAL_INDICATOR,
    PERSON,
    LOCATION,
    CURRENCY,
    DATE
}

enum Language {
    ENGLISH,
    SPANISH,
    GERMAN,
    FRENCH,
    CHINESE,
    JAPANESE
}

// Data Models
struct EntityExtractionRequest {
    content_id: String
    text: String
    language: Language
    extraction_types: List<EntityType>
}

struct ExtractedEntity {
    text: String
    entity_type: EntityType
    confidence: Float
    start_pos: Integer
    end_pos: Integer
    normalized_form: Optional<String>
    metadata: Map<String, Any>
}

struct EntityExtractionResponse {
    content_id: String
    entities: Map<EntityType, List<ExtractedEntity>>
    processing_time_ms: Float
}

// REST API Endpoints
POST /api/v1/entities/extract
    Request: EntityExtractionRequest
    Response: EntityExtractionResponse

GET /api/v1/entities/models
    Response: List<EntityModel>

POST /api/v1/entities/train
    Request: ModelTrainingRequest
    Response: TrainingJob
```

### Event Output
```pseudo
Event entity_extraction_completed {
    event_id: String
    timestamp: DateTime
    entity_extraction: EntityExtractionData
}

struct EntityExtractionData {
    content_id: String
    entities: EntitiesData
}

struct EntitiesData {
    companies: List<CompanyEntityData>
    metrics: List<MetricEntityData>
}

struct CompanyEntityData {
    text: String
    entity_type: String
    confidence: Float
    normalized_form: String
    metadata: CompanyMetadata
}

struct CompanyMetadata {
    sector: String
}

struct MetricEntityData {
    text: String
    entity_type: String
    confidence: Float
    metadata: MetricMetadata
}

struct MetricMetadata {
    metric_type: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    entity_extraction: {
        content_id: "news_article_001",
        entities: {
            companies: [
                {
                    text: "Apple Inc.",
                    entity_type: "company",
                    confidence: 0.98,
                    normalized_form: "AAPL",
                    metadata: {sector: "technology"}
                }
            ],
            metrics: [
                {
                    text: "revenue up 15%",
                    entity_type: "financial_metric",
                    confidence: 0.89,
                    metadata: {metric_type: "revenue_growth"}
                }
            ]
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table entity_models {
    id: UUID (primary key)
    model_name: String (required)
    entity_type: String (required)
    model_config: JSON (required)
    accuracy_score: Float
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table entity_mappings {
    id: UUID (primary key)
    entity_text: String (required)
    entity_type: String (required)
    normalized_form: String
    confidence: Float
    metadata: JSON
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Foundation for intelligence)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core NER Engine
- spaCy and Transformers integration
- Financial entity model training
- Multi-type entity extraction
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Entity normalization and mapping
- Fuzzy matching and disambiguation
- Custom financial entity types
- **Effort**: 1 ML engineer × 1 week = 1 dev-week

#### Week 4: Integration & Testing
- Integration with content services
- Performance optimization
- Accuracy validation
- **Effort**: 1 developer × 1 week = 1 dev-week

### Total Effort: **6 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: Pre-trained NER models, financial entity databases

### Success Criteria:
- 95% entity recognition accuracy
- P99 extraction latency < 150ms
- Support for 10K+ extractions per minute
- Multi-language entity support
