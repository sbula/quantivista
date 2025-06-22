# NLP Processing Service

## Responsibility
Advanced natural language processing for financial text analysis. Performs text preprocessing, tokenization, named entity recognition, topic modeling, and semantic analysis for market intelligence content.

## Technology Stack
- **Language**: Python + spaCy + Transformers + NLTK
- **Libraries**: spaCy, Transformers, NLTK, scikit-learn, Gensim
- **Scaling**: Horizontal by text processing volume, GPU acceleration
- **NFRs**: P99 processing < 500ms, 95% NLP accuracy, multi-language support

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ProcessingTask {
    TOKENIZATION,
    NAMED_ENTITY_RECOGNITION,
    TOPIC_MODELING,
    SEMANTIC_ANALYSIS,
    KEYWORD_EXTRACTION,
    LANGUAGE_DETECTION,
    TEXT_CLASSIFICATION
}

enum LanguageCode {
    EN,  // English
    ES,  // Spanish
    DE,  // German
    FR,  // French
    ZH,  // Chinese
    JA   // Japanese
}

// Data Models
struct NLPProcessingRequest {
    content_id: String
    text: String
    tasks: List<ProcessingTask>
    language: Optional<LanguageCode>
    domain_context: String  // "financial", "general", "technical"
}

struct NLPProcessingResponse {
    content_id: String
    language_detected: LanguageCode
    processing_results: Map<ProcessingTask, ProcessingResult>
    processing_time_ms: Float
    confidence_scores: Map<ProcessingTask, Float>
}

struct ProcessingResult {
    task_type: ProcessingTask
    result_data: Any
    confidence: Float
    metadata: Map<String, Any>
}

struct NamedEntity {
    text: String
    label: String  // PERSON, ORG, MONEY, PERCENT, etc.
    start_pos: Integer
    end_pos: Integer
    confidence: Float
}

struct Topic {
    topic_id: String
    keywords: List<String>
    probability: Float
    coherence_score: Float
}

struct SemanticAnalysis {
    main_themes: List<String>
    sentiment_polarity: Float
    subjectivity: Float
    readability_score: Float
    complexity_score: Float
}

// REST API Endpoints
POST /api/v1/nlp/process
    Request: NLPProcessingRequest
    Response: NLPProcessingResponse

POST /api/v1/nlp/batch-process
    Request: List<NLPProcessingRequest>
    Response: List<NLPProcessingResponse>

GET /api/v1/nlp/models/status
    Response: ModelStatus

POST /api/v1/nlp/extract-entities
    Request: EntityExtractionRequest
    Response: List<NamedEntity>
```

### Event Output
```pseudo
Event nlp_processing_completed {
    event_id: String
    timestamp: DateTime
    nlp_processed: NLPProcessedData
}

struct NLPProcessedData {
    content_id: String
    language_detected: String
    processing_results: ProcessingResultsData
    processing_time_ms: Float
}

struct ProcessingResultsData {
    named_entity_recognition: NamedEntityRecognitionData
    topic_modeling: TopicModelingData
    semantic_analysis: SemanticAnalysisData
}

struct NamedEntityRecognitionData {
    entities: List<EntityData>
}

struct EntityData {
    text: String
    label: String
    start_pos: Integer
    end_pos: Integer
    confidence: Float
}

struct TopicModelingData {
    topics: List<TopicData>
}

struct TopicData {
    topic_id: String
    keywords: List<String>
    probability: Float
}

struct SemanticAnalysisData {
    main_themes: List<String>
    sentiment_polarity: Float
    complexity_score: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    nlp_processed: {
        content_id: "news_article_001",
        language_detected: "EN",
        processing_results: {
            named_entity_recognition: {
                entities: [
                    {
                        text: "Apple Inc.",
                        label: "ORG",
                        start_pos: 15,
                        end_pos: 25,
                        confidence: 0.98
                    },
                    {
                        text: "$150.25",
                        label: "MONEY",
                        start_pos: 45,
                        end_pos: 52,
                        confidence: 0.95
                    }
                ]
            },
            topic_modeling: {
                topics: [
                    {
                        topic_id: "earnings_results",
                        keywords: ["earnings", "revenue", "profit", "guidance"],
                        probability: 0.85
                    }
                ]
            },
            semantic_analysis: {
                main_themes: ["quarterly_earnings", "financial_performance"],
                sentiment_polarity: 0.65,
                complexity_score: 0.72
            }
        },
        processing_time_ms: 245.8
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table nlp_models {
    id: UUID (primary key, auto-generated)
    model_name: String (required, max_length: 100)
    model_type: String (required, max_length: 50)
    language: String (required, max_length: 10)
    domain: String (required, max_length: 50)
    model_config: JSON (required)
    accuracy_metrics: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table processing_tasks {
    id: UUID (primary key, auto-generated)
    content_id: String (required, max_length: 100)
    task_type: String (required, max_length: 50)
    processing_status: String (default: 'pending', max_length: 20)
    result_data: JSON
    confidence_score: Float
    processing_time_ms: Float
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table nlp_processing_ts {
    timestamp: Timestamp (required, partition_key)
    content_id: String (required, max_length: 100)
    language: String (required, max_length: 10)
    task_type: String (required, max_length: 50)
    confidence_score: Float
    processing_time_ms: Float
    result_summary: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: task_type (partitions: 4)
}
```

### Redis Caching
```pseudo
struct NLPCache {
    // Model cache: "nlp_model:{model_name}" -> ModelData
    // Processing cache: "nlp_result:{content_hash}" -> ProcessingResult
    // Entity cache: "entities:{content_id}" -> List<NamedEntity>
    // Topic cache: "topics:{content_id}" -> List<Topic>
}
```

## Implementation Estimation

### Priority: **HIGH** (Core intelligence processing)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core NLP Engine
- Python service setup with spaCy and Transformers
- Basic text processing pipeline
- Named entity recognition implementation
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced NLP Features
- Topic modeling with Gensim
- Semantic analysis and text classification
- Multi-language support
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 5-6: Optimization & Integration
- GPU acceleration and performance optimization
- Integration with content collection services
- Model training and validation pipeline
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: Content collection services, GPU infrastructure, pre-trained models

### Success Criteria:
- P99 processing latency < 500ms
- 95% NLP accuracy across tasks
- Multi-language support (6+ languages)
- Support for 10K+ documents per hour
- Real-time processing capability
