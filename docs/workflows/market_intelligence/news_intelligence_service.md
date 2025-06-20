# News Intelligence Service

## Purpose
The News Intelligence Service is responsible for collecting and analyzing news, social media, and other text-based information sources with advanced Natural Language Processing (NLP) capabilities. It transforms unstructured text data into structured, actionable intelligence that can be used to inform trading decisions and risk assessments.

## Strict Boundaries

### Responsibilities
- Multi-source content aggregation from news, social media, and alternative data
- Content deduplication and filtering
- NLP processing and entity extraction
- Sentiment analysis with confidence scoring
- Impact assessment and categorization
- Distribution of processed intelligence

### Non-Responsibilities
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT perform market data analysis
- Does NOT manage user preferences or authentication
- Does NOT handle portfolio management

## API Design (API-First)

### REST API

#### GET /api/v1/news/articles
Retrieves news articles based on various filters.

**Query Parameters:**
- `entities` (string, optional): Comma-separated list of entity IDs (companies, people, etc.)
- `instruments` (string, optional): Comma-separated list of instrument IDs
- `sentiment` (string, optional): Filter by sentiment (positive, negative, neutral)
- `impact` (string, optional): Filter by impact level (high, medium, low)
- `timeframe` (string, optional): Filter by impact timeframe (immediate, short-term, long-term)
- `from` (string, optional): Start timestamp (ISO 8601)
- `to` (string, optional): End timestamp (ISO 8601)
- `limit` (integer, optional): Maximum number of results (default: 50)
- `offset` (integer, optional): Pagination offset (default: 0)

**Response:**
```json
{
  "articles": [
    {
      "id": "news-123456",
      "title": "Apple Reports Record Quarterly Revenue",
      "source": "Financial Times",
      "url": "https://ft.com/articles/apple-record-revenue",
      "published_at": "2025-06-19T16:30:00Z",
      "summary": "Apple Inc. reported record quarterly revenue of $98.5 billion, exceeding analyst expectations.",
      "sentiment": {
        "polarity": "positive",
        "score": 0.85,
        "confidence": 0.92
      },
      "entities": [
        {
          "id": "company-aapl",
          "name": "Apple Inc.",
          "type": "company",
          "sentiment": "positive"
        }
      ],
      "impact": {
        "level": "high",
        "timeframe": "short-term",
        "sectors": ["technology", "consumer electronics"],
        "regions": ["global", "us"]
      }
    }
  ],
  "pagination": {
    "total": 1250,
    "limit": 50,
    "offset": 0
  }
}
```

#### GET /api/v1/news/entities/{entity_id}
Retrieves news and sentiment information for a specific entity.

**Path Parameters:**
- `entity_id` (string, required): Entity identifier

**Query Parameters:**
- `from` (string, optional): Start timestamp (ISO 8601)
- `to` (string, optional): End timestamp (ISO 8601)
- `limit` (integer, optional): Maximum number of results (default: 50)

**Response:**
```json
{
  "entity": {
    "id": "company-aapl",
    "name": "Apple Inc.",
    "type": "company",
    "aliases": ["AAPL", "Apple"],
    "metadata": {
      "sector": "Technology",
      "industry": "Consumer Electronics",
      "founded": "1976-04-01"
    }
  },
  "sentiment_summary": {
    "current": {
      "polarity": "positive",
      "score": 0.75,
      "confidence": 0.88
    },
    "trend": [
      {
        "date": "2025-06-19",
        "polarity": "positive",
        "score": 0.75
      },
      {
        "date": "2025-06-18",
        "polarity": "neutral",
        "score": 0.15
      }
    ]
  },
  "recent_articles": [
    {
      "id": "news-123456",
      "title": "Apple Reports Record Quarterly Revenue",
      "source": "Financial Times",
      "published_at": "2025-06-19T16:30:00Z",
      "sentiment": {
        "polarity": "positive",
        "score": 0.85
      }
    }
  ]
}
```

#### GET /api/v1/news/sentiment/summary
Retrieves sentiment summary for multiple entities or instruments.

**Query Parameters:**
- `entities` (string, optional): Comma-separated list of entity IDs
- `instruments` (string, optional): Comma-separated list of instrument IDs
- `from` (string, optional): Start timestamp (ISO 8601)
- `to` (string, optional): End timestamp (ISO 8601)

**Response:**
```json
{
  "summaries": [
    {
      "id": "company-aapl",
      "name": "Apple Inc.",
      "type": "company",
      "sentiment": {
        "polarity": "positive",
        "score": 0.75,
        "confidence": 0.88,
        "article_count": 42
      }
    },
    {
      "id": "company-msft",
      "name": "Microsoft Corporation",
      "type": "company",
      "sentiment": {
        "polarity": "neutral",
        "score": 0.05,
        "confidence": 0.72,
        "article_count": 38
      }
    }
  ]
}
```

### gRPC API

```protobuf
syntax = "proto3";

package news_intelligence.v1;

import "google/protobuf/timestamp.proto";

service NewsIntelligenceService {
  // Stream real-time news updates
  rpc StreamNewsUpdates(StreamNewsRequest) returns (stream NewsUpdate);
  
  // Get sentiment analysis for entities
  rpc GetEntitySentiment(EntitySentimentRequest) returns (EntitySentimentResponse);
  
  // Get impact assessment for news events
  rpc GetImpactAssessment(ImpactAssessmentRequest) returns (ImpactAssessmentResponse);
}

message StreamNewsRequest {
  repeated string entity_ids = 1;
  repeated string instrument_ids = 2;
  string sentiment_filter = 3; // "positive", "negative", "neutral", "all"
  string impact_level = 4; // "high", "medium", "low", "all"
}

message NewsUpdate {
  string id = 1;
  string title = 2;
  string source = 3;
  string url = 4;
  google.protobuf.Timestamp published_at = 5;
  string summary = 6;
  SentimentAnalysis sentiment = 7;
  repeated EntityMention entities = 8;
  ImpactAssessment impact = 9;
}

message SentimentAnalysis {
  string polarity = 1; // "positive", "negative", "neutral"
  double score = 2; // -1.0 to 1.0
  double confidence = 3; // 0.0 to 1.0
}

message EntityMention {
  string id = 1;
  string name = 2;
  string type = 3; // "company", "person", "location", "product", etc.
  SentimentAnalysis sentiment = 4;
}

message ImpactAssessment {
  string level = 1; // "high", "medium", "low"
  string timeframe = 2; // "immediate", "short-term", "long-term"
  repeated string sectors = 3;
  repeated string regions = 4;
  repeated string instruments = 5;
}

message EntitySentimentRequest {
  repeated string entity_ids = 1;
  google.protobuf.Timestamp from = 2;
  google.protobuf.Timestamp to = 3;
  bool include_trend = 4;
}

message EntitySentimentResponse {
  repeated EntitySentiment entities = 1;
}

message EntitySentiment {
  string id = 1;
  string name = 2;
  string type = 3;
  SentimentAnalysis current_sentiment = 4;
  repeated SentimentTrendPoint trend = 5;
  int32 article_count = 6;
}

message SentimentTrendPoint {
  google.protobuf.Timestamp date = 1;
  string polarity = 2;
  double score = 3;
}

message ImpactAssessmentRequest {
  string news_id = 1;
  bool include_historical_comparison = 2;
}

message ImpactAssessmentResponse {
  ImpactAssessment impact = 1;
  repeated HistoricalComparison historical_comparisons = 2;
}

message HistoricalComparison {
  string similar_event_id = 1;
  string similar_event_title = 2;
  google.protobuf.Timestamp event_date = 3;
  double similarity_score = 4;
  repeated MarketReaction market_reactions = 5;
}

message MarketReaction {
  string instrument_id = 1;
  double price_change_percent = 2;
  string timeframe = 3; // "1d", "3d", "1w", "1m"
}
```

## Data Model

### Core Entities

#### NewsArticle
Represents a news article or social media post.

**Attributes:**
- `id` (string): Unique identifier for the article
- `title` (string): Article headline or title
- `source` (string): Source of the article (publication, website, social media platform)
- `author` (string): Author of the article
- `url` (string): URL to the original article
- `content` (text): Full content of the article
- `summary` (text): Summarized content
- `published_at` (datetime): Publication timestamp
- `collected_at` (datetime): When the article was collected by the system
- `language` (string): Language of the article
- `sentiment` (object): Sentiment analysis results
- `entities` (array): Entities mentioned in the article
- `impact` (object): Impact assessment

#### Entity
Represents a real-world entity mentioned in news articles.

**Attributes:**
- `id` (string): Unique identifier for the entity
- `name` (string): Primary name of the entity
- `type` (enum): Type of entity (company, person, location, product, etc.)
- `aliases` (array): Alternative names or abbreviations
- `metadata` (map): Entity-specific metadata
- `external_ids` (map): IDs in external systems (e.g., CIK, ISIN)

#### EntityMention
Represents a mention of an entity in a specific article.

**Attributes:**
- `article_id` (string): Reference to the article
- `entity_id` (string): Reference to the entity
- `mentions` (integer): Number of mentions in the article
- `sentiment` (object): Entity-specific sentiment in this article
- `relevance` (float): Relevance score of the entity to the article
- `context` (array): Contextual snippets around entity mentions

#### SentimentAnalysis
Represents sentiment analysis results.

**Attributes:**
- `polarity` (enum): Sentiment polarity (positive, negative, neutral)
- `score` (float): Sentiment score (-1.0 to 1.0)
- `confidence` (float): Confidence level of the analysis (0.0 to 1.0)
- `aspects` (map): Aspect-based sentiment (optional)

#### ImpactAssessment
Represents the assessed impact of a news event.

**Attributes:**
- `level` (enum): Impact level (high, medium, low)
- `timeframe` (enum): Impact timeframe (immediate, short-term, long-term)
- `sectors` (array): Affected industry sectors
- `regions` (array): Affected geographic regions
- `instruments` (array): Affected financial instruments
- `confidence` (float): Confidence level of the assessment

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### news_articles
```sql
CREATE TABLE news_articles (
    id VARCHAR(50) PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    source VARCHAR(100) NOT NULL,
    author VARCHAR(100),
    url VARCHAR(1000) NOT NULL,
    content TEXT,
    summary TEXT,
    published_at TIMESTAMP WITH TIME ZONE NOT NULL,
    collected_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    language VARCHAR(10) NOT NULL DEFAULT 'en',
    raw_content JSONB,
    processing_status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX news_articles_published_at_idx ON news_articles(published_at);
CREATE INDEX news_articles_source_idx ON news_articles(source);
CREATE INDEX news_articles_processing_status_idx ON news_articles(processing_status);
```

#### entities
```sql
CREATE TABLE entities (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    type VARCHAR(50) NOT NULL,
    aliases JSONB,
    metadata JSONB,
    external_ids JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX entities_type_idx ON entities(type);
CREATE INDEX entities_name_idx ON entities(name);
```

#### entity_mentions
```sql
CREATE TABLE entity_mentions (
    id SERIAL PRIMARY KEY,
    article_id VARCHAR(50) NOT NULL REFERENCES news_articles(id),
    entity_id VARCHAR(50) NOT NULL REFERENCES entities(id),
    mentions INTEGER NOT NULL DEFAULT 1,
    sentiment_polarity VARCHAR(10),
    sentiment_score DECIMAL(4,3),
    sentiment_confidence DECIMAL(4,3),
    relevance DECIMAL(4,3),
    context JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT entity_mentions_article_entity_idx UNIQUE (article_id, entity_id)
);

CREATE INDEX entity_mentions_article_id_idx ON entity_mentions(article_id);
CREATE INDEX entity_mentions_entity_id_idx ON entity_mentions(entity_id);
```

#### sentiment_analyses
```sql
CREATE TABLE sentiment_analyses (
    article_id VARCHAR(50) PRIMARY KEY REFERENCES news_articles(id),
    polarity VARCHAR(10) NOT NULL,
    score DECIMAL(4,3) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    aspects JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

#### impact_assessments
```sql
CREATE TABLE impact_assessments (
    article_id VARCHAR(50) PRIMARY KEY REFERENCES news_articles(id),
    level VARCHAR(10) NOT NULL,
    timeframe VARCHAR(20) NOT NULL,
    sectors JSONB,
    regions JSONB,
    instruments JSONB,
    confidence DECIMAL(4,3) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

### Read Schema (Query Side)

#### news_articles_view
```sql
CREATE TABLE news_articles_view (
    id VARCHAR(50) PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    source VARCHAR(100) NOT NULL,
    author VARCHAR(100),
    url VARCHAR(1000) NOT NULL,
    summary TEXT,
    published_at TIMESTAMP WITH TIME ZONE NOT NULL,
    sentiment_polarity VARCHAR(10),
    sentiment_score DECIMAL(4,3),
    sentiment_confidence DECIMAL(4,3),
    impact_level VARCHAR(10),
    impact_timeframe VARCHAR(20),
    impact_confidence DECIMAL(4,3),
    entities JSONB,
    sectors JSONB,
    regions JSONB,
    instruments JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX news_articles_view_published_at_idx ON news_articles_view(published_at);
CREATE INDEX news_articles_view_sentiment_polarity_idx ON news_articles_view(sentiment_polarity);
CREATE INDEX news_articles_view_impact_level_idx ON news_articles_view(impact_level);
```

#### entity_sentiment_summary
```sql
CREATE TABLE entity_sentiment_summary (
    entity_id VARCHAR(50) PRIMARY KEY REFERENCES entities(id),
    current_polarity VARCHAR(10) NOT NULL,
    current_score DECIMAL(4,3) NOT NULL,
    current_confidence DECIMAL(4,3) NOT NULL,
    article_count INTEGER NOT NULL DEFAULT 0,
    sentiment_trend JSONB,
    last_updated TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

#### entity_news_index
```sql
CREATE TABLE entity_news_index (
    entity_id VARCHAR(50) NOT NULL REFERENCES entities(id),
    article_id VARCHAR(50) NOT NULL REFERENCES news_articles_view(id),
    relevance DECIMAL(4,3) NOT NULL,
    sentiment_score DECIMAL(4,3),
    published_at TIMESTAMP WITH TIME ZONE NOT NULL,
    PRIMARY KEY (entity_id, article_id)
);

CREATE INDEX entity_news_index_entity_published_idx ON entity_news_index(entity_id, published_at DESC);
```

#### sector_sentiment_summary
```sql
CREATE TABLE sector_sentiment_summary (
    sector VARCHAR(50) PRIMARY KEY,
    current_polarity VARCHAR(10) NOT NULL,
    current_score DECIMAL(4,3) NOT NULL,
    current_confidence DECIMAL(4,3) NOT NULL,
    article_count INTEGER NOT NULL DEFAULT 0,
    sentiment_trend JSONB,
    last_updated TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

## Place in Workflow

The News Intelligence Service is a critical component of the Market Intelligence Workflow and serves as the primary processor of text-based market information. It:

1. **Initiates the workflow** by collecting news and social media content from multiple sources
2. **Processes the content** through deduplication, filtering, NLP analysis, and sentiment scoring
3. **Assesses impact** on industries, companies, instruments, and regions
4. **Distributes processed intelligence** to downstream services via event streams

The service interacts with:
- **External news providers** (input): Reuters, Bloomberg, Financial Times, etc.
- **Social media APIs** (input): Twitter, Reddit, Discord, etc.
- **ML Prediction Service** (output): Provides sentiment and impact data for prediction models
- **Trading Strategy Service** (output): Supplies market intelligence for strategy adjustments
- **Risk Analysis Service** (output): Delivers news impact assessments for risk calculations
- **Portfolio Optimization Service** (output): Provides intelligence for portfolio adjustments
- **Reporting Service** (output): Supplies news and sentiment data for reports and dashboards

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL for data storage
- Implement basic service skeleton with health checks

#### Week 2: Data Collection Framework
- Develop news source connectors (RSS, APIs)
- Implement social media connectors
- Create content normalization pipeline
- Build deduplication and filtering mechanisms

#### Week 3: NLP Processing Pipeline
- Implement entity extraction and linking
- Develop sentiment analysis engine
- Create text summarization capabilities
- Build language detection and translation

#### Week 4: Basic API Implementation
- Implement REST API endpoints for data retrieval
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Advanced NLP Features
- Implement aspect-based sentiment analysis
- Develop entity relationship extraction
- Create event detection and classification
- Build topic modeling and clustering

#### Week 6: Impact Assessment
- Implement impact level classification
- Develop timeframe prediction
- Create sector and region mapping
- Build historical correlation analysis

#### Week 7: Intelligence Distribution
- Implement event streaming for real-time updates
- Develop customizable intelligence feeds
- Create alert generation for significant events
- Build intelligence report generation

#### Week 8: Integration and Testing
- Integrate with downstream services
- Perform load and stress testing
- Conduct accuracy testing for NLP components
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Performance Optimization
- Optimize NLP processing pipeline
- Enhance database query performance
- Implement caching strategies
- Fine-tune resource utilization

#### Week 11: Multi-Language Support
- Add support for additional languages
- Implement language-specific NLP models
- Create cross-language entity linking
- Develop language detection improvements

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular model retraining and improvement
- Source connector updates and additions
- Performance monitoring and optimization
- NLP algorithm enhancements