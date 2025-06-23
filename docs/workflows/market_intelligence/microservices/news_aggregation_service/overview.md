# News Aggregation Service

## Responsibility
RSS feed monitoring and free news source aggregation for financial news, earnings announcements, economic data, and market analysis. Handles content extraction, deduplication, and source reliability tracking while optimizing for free tier limitations.

## Technology Stack
- **Language**: Python + asyncio for concurrent processing
- **Libraries**: feedparser, BeautifulSoup, Scrapy, aiohttp, newspaper3k
- **Scaling**: Horizontal by source groups
- **NFRs**: P99 processing latency < 5s, 99.9% uptime, handle 1K articles/hour

## API Specification

### Internal APIs

#### News Source Management API
```pseudo
// Enumerations
enum SourceType {
    RSS_FEED,
    WEB_SCRAPER,
    API_ENDPOINT,
    ECONOMIC_DATA
}

// Data Models
struct NewsSource {
    source_id: String
    name: String
    source_type: SourceType
    url: String
    category: String  // "earnings", "economic", "market_news", "analysis"
    credibility_score: Float
    update_frequency: Integer  // minutes
    rate_limit: Optional<Integer>
    enabled: Boolean
    last_updated: Optional<DateTime>
}

struct SourceConfig {
    source_id: String
    extraction_rules: Map<String, Any>
    content_selectors: Map<String, Any>
    quality_filters: Map<String, Any>
    rate_limiting: Map<String, Any>
}

struct NewsArticle {
    article_id: String
    source_id: String
    title: String
    content: String
    summary: String
    author: Optional<String>
    published_at: DateTime
    url: String
    category: String
    entities_mentioned: List<String>
    quality_score: Float
    credibility_score: Float
}

// REST API Endpoints
POST /api/v1/sources
    Request: NewsSource
    Response: OperationResult

GET /api/v1/sources
    Parameters: category
    Response: List<NewsSource>

PUT /api/v1/sources/{source_id}
    Request: NewsSource
    Response: OperationResult

DELETE /api/v1/sources/{source_id}
    Response: OperationResult

POST /api/v1/sources/{source_id}/crawl
    Response: CrawlResult
```

#### Content Extraction API
```pseudo
// Data Models
struct ExtractionRequest {
    url: String
    source_id: String
    extraction_type: String  // "rss", "html", "api"
    custom_selectors: Optional<Map<String, Any>>
}

struct ExtractionResult {
    success: Boolean
    articles: List<NewsArticle>
    errors: List<String>
    extraction_time_ms: Float
    articles_found: Integer
    articles_filtered: Integer
}

struct ContentQuality {
    readability_score: Float
    content_length: Integer
    has_financial_keywords: Boolean
    duplicate_probability: Float
    spam_probability: Float
    overall_quality: Float
}

// REST API Endpoints
POST /api/v1/extract
    Request: ExtractionRequest
    Response: ExtractionResult

GET /api/v1/articles/recent
    Parameters: hours, category, min_quality
    Response: List<NewsArticle>

GET /api/v1/articles/{article_id}
    Response: NewsArticle
```

### Event Output

#### NewsArticleExtractedEvent
```pseudo
Event news_article_extracted {
    event_id: String
    timestamp: DateTime
    article: NewsArticleData
    extraction_metadata: ExtractionMetadata
}

struct NewsArticleData {
    article_id: String
    source_id: String
    title: String
    content: String
    summary: String
    author: String
    published_at: DateTime
    url: String
    category: String
    entities_mentioned: List<String>
    quality_score: Float
    credibility_score: Float
}

struct ExtractionMetadata {
    source_type: String
    extraction_time_ms: Integer
    content_length: Integer
    duplicate_check: Boolean
    quality_filters_passed: List<String>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    article: {
        article_id: "yahoo_finance_20250621_001",
        source_id: "yahoo_finance_rss",
        title: "Apple Reports Record Q4 Earnings, Beats Wall Street Expectations",
        content: "Apple Inc. (NASDAQ: AAPL) reported record fourth-quarter earnings...",
        summary: "Apple beats Q4 earnings expectations with strong iPhone sales and services revenue growth.",
        author: "Financial Reporter",
        published_at: "2025-06-21T09:30:00.000Z",
        url: "https://finance.yahoo.com/news/apple-earnings-q4-2025",
        category: "earnings",
        entities_mentioned: ["AAPL", "iPhone", "services", "Q4"],
        quality_score: 0.92,
        credibility_score: 0.88
    },
    extraction_metadata: {
        source_type: "rss_feed",
        extraction_time_ms: 2400,
        content_length: 1250,
        duplicate_check: false,
        quality_filters_passed: ["length", "financial_keywords", "readability"]
    }
}
```

#### NewsSourceStatusEvent
```pseudo
Event news_source_status_updated {
    event_id: String
    timestamp: DateTime
    source_status: NewsSourceStatusData
}

struct NewsSourceStatusData {
    source_id: String
    status: String
    last_successful_crawl: DateTime
    error_message: String
    consecutive_failures: Integer
    credibility_impact: Float
    next_retry: DateTime
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:05:00.000Z",
    source_status: {
        source_id: "marketwatch_rss",
        status: "error",
        last_successful_crawl: "2025-06-21T09:45:00.000Z",
        error_message: "RSS feed temporarily unavailable",
        consecutive_failures: 3,
        credibility_impact: -0.05,
        next_retry: "2025-06-21T10:15:00.000Z"
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct NewsSourceMetadata {
    source_id: String
    name: String
    base_url: String
    source_type: SourceType
    category: String
    credibility_score: Float
    reliability_history: List<Float>
    last_crawl: DateTime
    crawl_frequency: Integer
    success_rate: Float
}

struct ExtractionRule {
    rule_id: String
    source_id: String
    selector_type: String  // "css", "xpath", "regex"
    selector: String
    field_name: String
    required: Boolean
    validation_regex: Optional<String>
}

struct ArticleMetrics {
    article_id: String
    view_count: Integer
    social_shares: Integer
    comment_count: Integer
    engagement_score: Float
    viral_potential: Float
    market_impact_score: Float
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// News sources configuration
Table news_sources {
    id: UUID (primary key, auto-generated)
    source_id: String (required, unique, max_length: 100)
    name: String (required, max_length: 200)
    source_type: String (required, max_length: 20)
    base_url: String (required)
    rss_url: String
    category: String (required, max_length: 50)
    credibility_score: Float (default: 0.5)
    update_frequency: Integer (default: 60) // minutes
    rate_limit: Integer
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
    last_crawled: Timestamp
}

// Content extraction rules
Table extraction_rules {
    id: UUID (primary key, auto-generated)
    source_id: String (required, max_length: 100, foreign_key: news_sources.source_id)
    rule_name: String (required, max_length: 100)
    selector_type: String (required, max_length: 20) // 'css', 'xpath', 'regex'
    selector: String (required)
    field_name: String (required, max_length: 50)
    required: Boolean (default: false)
    validation_regex: String
    priority: Integer (default: 1)
    created_at: Timestamp (default: now)
}

// Source reliability tracking
Table source_reliability {
    id: UUID (primary key, auto-generated)
    source_id: String (required, max_length: 100, foreign_key: news_sources.source_id)
    timestamp: Timestamp (required)
    crawl_success: Boolean (required)
    articles_extracted: Integer (default: 0)
    articles_quality_passed: Integer (default: 0)
    response_time_ms: Float
    error_message: String
    credibility_adjustment: Float (default: 0.0)
    created_at: Timestamp (default: now)
}

// Article processing statistics
Table article_stats {
    id: UUID (primary key, auto-generated)
    source_id: String (required, max_length: 100, foreign_key: news_sources.source_id)
    timestamp: Timestamp (required)
    articles_processed: Integer (default: 0)
    articles_published: Integer (default: 0)
    articles_filtered: Integer (default: 0)
    avg_quality_score: Float
    avg_processing_time_ms: Float
    duplicate_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}

// Indexes
idx_news_sources_category_enabled: (category, enabled)
idx_extraction_rules_source: (source_id, priority)
idx_source_reliability_source_time: (source_id, timestamp DESC)
idx_article_stats_source_time: (source_id, timestamp DESC)
```

### Query Side (Elasticsearch + TimescaleDB)
```pseudo
// Elasticsearch Index Schema
Index news_articles {
    article_id: Keyword
    source_id: Keyword
    title: Text (analyzer: financial_analyzer, fields: {raw: Keyword})
    content: Text (analyzer: financial_analyzer)
    summary: Text
    author: Keyword
    published_at: Date
    url: Keyword
    category: Keyword
    entities_mentioned: Keyword
    quality_score: Float
    credibility_score: Float
    sentiment_score: Float
    impact_score: Float
    social_metrics: {
        shares: Integer
        comments: Integer
        engagement_rate: Float
    }
}
```

### TimescaleDB for Time-Series Metrics
```pseudo
// News source performance metrics
Table news_source_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    source_id: String (required, max_length: 100)
    articles_per_hour: Float
    avg_quality_score: Float
    credibility_score: Float
    success_rate: Float
    response_time_p99_ms: Float
    duplicate_rate: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Article publication trends
Table article_trends_ts {
    timestamp: Timestamp (required, partition_key)
    category: String (required, max_length: 50)
    article_count: Integer
    avg_quality_score: Float
    avg_sentiment: Float
    trending_entities: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Indexes
idx_news_metrics_source_time: (source_id, timestamp DESC)
idx_article_trends_category_time: (category, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache news_cache {
    // Article deduplication
    "article_hash:{hash}": String (TTL: 24h)

    // Source status
    "source_status:{source_id}": SourceStatus (TTL: 5m)

    // Recent articles
    "recent_articles:{category}:{hours}": List<String> (TTL: 1h)

    // Extraction cache
    "extraction:{url_hash}": ExtractionResult (TTL: 6h)

    // Rate limiting
    "rate_limit:{source_id}:{window}": Integer (TTL: window_duration)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core intelligence source)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Aggregation Framework
- Python service setup with asyncio and feedparser
- RSS feed parsing and content extraction
- Basic web scraping with BeautifulSoup and Scrapy
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Content Processing
- Content quality assessment and filtering
- Article deduplication and uniqueness detection
- Entity extraction and financial keyword detection
- **Effort**: 1 developer × 1 week = 1 dev-week

#### Week 4: Source Management
- Dynamic source configuration and management
- Source reliability tracking and credibility scoring
- Error handling and retry mechanisms
- **Effort**: 1 developer × 1 week = 1 dev-week

#### Week 5: Free Source Integration
- Yahoo Finance, MarketWatch, Seeking Alpha integration
- Google News and economic data sources
- Rate limiting and quota management for free tiers
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 6: Integration & Testing
- Integration with Content Quality Service
- Performance testing and optimization
- Monitoring and alerting setup
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 mid-level developer)
### Dependencies:
- Elasticsearch cluster for article storage
- TimescaleDB for metrics storage
- Redis for caching and deduplication
- Apache Pulsar for event streaming

### Risk Factors:
- **Medium**: Website structure changes affecting scraping
- **Medium**: Free tier rate limits and access restrictions
- **Low**: Content quality and deduplication accuracy
- **Low**: Technology stack complexity

### Success Criteria:
- Aggregate 1K+ articles per hour from 20+ sources
- Achieve 95% deduplication accuracy
- Maintain 99.9% uptime for RSS feeds
- P99 processing latency < 5 seconds
- Source credibility tracking with 90% accuracy
