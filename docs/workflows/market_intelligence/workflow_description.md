# Market Intelligence Workflow

## Overview
The Market Intelligence Workflow is responsible for collecting, analyzing, and distributing news, social media, and other text-based information sources to provide valuable market insights. Given the heavy reliance on free and social media data sources, this workflow emphasizes robust quality assurance, source reliability assessment, and noise filtering to extract actionable intelligence from potentially unreliable sources.

## Key Challenges Addressed
- **Free Data Source Quality**: Social media and free news sources often contain noise, misinformation, and low-quality content
- **Real-time Social Media Processing**: High-velocity, unstructured social media streams requiring immediate processing
- **Multi-language Global Sources**: Content in multiple languages requiring translation and cultural context
- **Source Reliability Assessment**: Dynamic scoring of source credibility based on historical accuracy
- **Spam and Bot Detection**: Filtering automated content and manipulation attempts
- **Scalable NLP Processing**: Handling millions of social media posts and news articles daily

## Refined Workflow Sequence

### 1. Multi-Source Content Ingestion
**Responsibility**: Content Ingestion Services (specialized by source type)

#### Social Media Monitoring Service
- **Twitter/X API**: Real-time tweet streams, trending topics, financial hashtags
- **Reddit API**: Subreddit monitoring (r/investing, r/stocks, r/wallstreetbets)
- **Discord/Telegram**: Financial community channels and groups
- **YouTube**: Financial influencer content and earnings call recordings
- **Rate limiting**: Respect API limits, implement exponential backoff

#### News Aggregation Service
- **Free RSS feeds**: Yahoo Finance, Google News, MarketWatch, Seeking Alpha
- **Financial blogs**: Zero Hedge, The Motley Fool, Benzinga (free tiers)
- **Economic calendars**: FRED, Trading Economics, Investing.com
- **Press releases**: Company websites, PR Newswire free feeds
- **Regulatory filings**: SEC EDGAR, company investor relations pages

#### Alternative Data Collection Service
- **Google Trends**: Search volume for financial terms and companies
- **Wikipedia**: Page view statistics for companies and financial topics
- **GitHub**: Repository activity for tech companies
- **Job postings**: Company hiring trends from free job boards
- **Patent filings**: USPTO database for innovation indicators

### 2. Content Quality Assurance and Filtering
**Responsibility**: Content Quality Service

#### Spam and Bot Detection
- **Account analysis**: Follower patterns, posting frequency, account age
- **Content patterns**: Repetitive messaging, coordinated posting
- **Engagement metrics**: Like/share ratios, comment quality
- **Network analysis**: Suspicious interaction patterns
- **Machine learning models**: Trained on known spam/bot datasets

#### Source Credibility Scoring
```python
def calculate_source_credibility(source):
    factors = {
        'historical_accuracy': check_prediction_accuracy(source),
        'verification_status': get_platform_verification(source),
        'follower_quality': analyze_follower_authenticity(source),
        'content_consistency': measure_posting_patterns(source),
        'external_validation': cross_reference_claims(source),
        'domain_authority': get_website_authority(source)
    }

    weights = {
        'historical_accuracy': 0.35,
        'verification_status': 0.15,
        'follower_quality': 0.20,
        'content_consistency': 0.10,
        'external_validation': 0.15,
        'domain_authority': 0.05
    }

    return sum(factor * weights[name] for name, factor in factors.items())
```

#### Content Deduplication
- **Fuzzy matching**: Near-duplicate detection using MinHash/LSH
- **Cross-platform deduplication**: Same story across multiple sources
- **Temporal clustering**: Related content within time windows
- **Canonical source identification**: Identify original vs. reposted content

### 3. Multi-Language NLP Processing
**Responsibility**: NLP Processing Service

#### Language Detection and Translation
- **Language identification**: FastText language detection
- **Translation services**: Google Translate API (free tier), LibreTranslate
- **Cultural context preservation**: Maintain sentiment nuances across languages
- **Quality assessment**: Translation confidence scoring

#### Entity Extraction and Linking
- **Named Entity Recognition**: spaCy, NLTK for companies, people, locations
- **Financial instrument mapping**: Ticker symbol extraction and validation
- **Entity disambiguation**: Link mentions to canonical entities
- **Relationship extraction**: Identify connections between entities
- **Temporal entity tracking**: Track entity mentions over time

### 4. Advanced Sentiment Analysis
**Responsibility**: Sentiment Analysis Service

#### Multi-Model Sentiment Analysis
- **General sentiment**: VADER, TextBlob for broad sentiment
- **Financial sentiment**: FinBERT, specialized financial language models
- **Aspect-based sentiment**: Sentiment toward specific aspects (earnings, products, management)
- **Emotion detection**: Fear, greed, uncertainty indicators
- **Sarcasm detection**: Identify ironic or sarcastic content

#### Confidence and Quality Scoring
```python
def calculate_sentiment_confidence(text, models_results):
    factors = {
        'model_agreement': calculate_model_consensus(models_results),
        'text_clarity': assess_text_ambiguity(text),
        'context_completeness': check_context_availability(text),
        'source_reliability': get_source_credibility_score(text.source),
        'language_confidence': get_translation_confidence(text)
    }

    return weighted_average(factors, confidence_weights)
```

### 5. Market Impact Assessment
**Responsibility**: Impact Assessment Service

#### Real-time Impact Prediction
- **Historical correlation analysis**: Compare with similar past events
- **Sector impact modeling**: Predict affected industries and companies
- **Geographic impact assessment**: Regional market implications
- **Timeframe classification**: Immediate (minutes), short-term (hours/days), long-term (weeks/months)
- **Volatility prediction**: Expected price movement magnitude

#### Feedback Loop Integration
- **Market reaction tracking**: Monitor actual price movements post-news
- **Model accuracy assessment**: Continuously evaluate prediction quality
- **Dynamic weight adjustment**: Update impact models based on performance
- **Anomaly detection**: Identify unexpected market reactions

### 6. Event-Driven Intelligence Distribution
**Responsibility**: Intelligence Distribution Service
- **Real-time streaming**: Apache Pulsar for immediate intelligence delivery
- **Batch processing**: Apache Kafka for historical analysis and reporting
- **Quality-based routing**: High-quality intelligence to real-time trading, lower quality to research
- **Personalized feeds**: User-specific intelligence based on portfolios and interests
- **Alert generation**: Threshold-based notifications for significant events

## Event Contracts

### Events Produced

#### `NewsAggregatedEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.123Z",
  "source": {
    "type": "twitter|reddit|rss|blog|filing",
    "name": "wallstreetbets|yahoo_finance|sec_edgar",
    "url": "https://reddit.com/r/wallstreetbets/comments/xyz",
    "credibility_score": 0.65,
    "verification_status": "verified|unverified|suspicious"
  },
  "content": {
    "id": "content-123456",
    "title": "AAPL earnings beat expectations",
    "text": "Apple just reported Q2 earnings...",
    "language": "en",
    "author": {
      "id": "user-789",
      "username": "financial_analyst_pro",
      "follower_count": 15000,
      "account_age_days": 1825
    },
    "published_at": "2025-06-21T10:25:00.000Z",
    "engagement": {
      "likes": 245,
      "shares": 67,
      "comments": 89,
      "engagement_rate": 0.027
    }
  },
  "quality_metrics": {
    "spam_probability": 0.05,
    "bot_probability": 0.12,
    "content_quality_score": 0.78,
    "duplicate_probability": 0.03
  }
}
```

#### `NewsSentimentAnalyzedEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.200Z",
  "content_id": "content-123456",
  "sentiment": {
    "overall": {
      "polarity": "positive|negative|neutral",
      "score": 0.75,
      "confidence": 0.88,
      "intensity": "strong|moderate|weak"
    },
    "aspects": [
      {
        "aspect": "earnings",
        "polarity": "positive",
        "score": 0.82,
        "confidence": 0.91
      },
      {
        "aspect": "guidance",
        "polarity": "neutral",
        "score": 0.05,
        "confidence": 0.67
      }
    ],
    "emotions": {
      "fear": 0.15,
      "greed": 0.72,
      "uncertainty": 0.23,
      "confidence": 0.68
    }
  },
  "entities": [
    {
      "id": "company-aapl",
      "name": "Apple Inc.",
      "type": "company",
      "mentions": 3,
      "sentiment": {
        "polarity": "positive",
        "score": 0.78,
        "confidence": 0.85
      },
      "relevance": 0.95
    }
  ],
  "processing_metadata": {
    "models_used": ["finbert", "vader", "textblob"],
    "model_agreement": 0.89,
    "processing_time_ms": 145,
    "language_detected": "en",
    "translation_confidence": 1.0
  }
}
```

#### `MarketImpactAssessmentEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.300Z",
  "content_id": "content-123456",
  "impact_assessment": {
    "overall_impact": {
      "level": "high|medium|low",
      "confidence": 0.82,
      "timeframe": "immediate|short_term|long_term",
      "duration_estimate": "2-4 hours"
    },
    "affected_entities": [
      {
        "entity_id": "company-aapl",
        "impact_type": "direct",
        "expected_direction": "positive",
        "magnitude": 0.75,
        "confidence": 0.88
      }
    ],
    "sector_impact": [
      {
        "sector": "technology",
        "impact_level": "high",
        "expected_direction": "positive",
        "confidence": 0.79
      }
    ],
    "geographic_impact": [
      {
        "region": "us_markets",
        "impact_level": "high",
        "confidence": 0.85
      }
    ]
  },
  "historical_correlation": {
    "similar_events": [
      {
        "event_id": "historical-event-456",
        "similarity_score": 0.87,
        "market_reaction": {
          "price_change_1h": 0.025,
          "price_change_1d": 0.045,
          "volatility_increase": 0.15
        }
      }
    ],
    "correlation_confidence": 0.73
  }
}
```

#### `ContentQualityAlertEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.400Z",
  "alert_type": "SPAM_DETECTED|BOT_ACTIVITY|MANIPULATION_SUSPECTED|SOURCE_DEGRADED",
  "severity": "LOW|MEDIUM|HIGH|CRITICAL",
  "source": {
    "type": "twitter",
    "name": "suspicious_account_123"
  },
  "details": {
    "description": "Coordinated posting pattern detected",
    "affected_content_count": 47,
    "confidence": 0.92,
    "recommended_action": "QUARANTINE|BLOCK|INVESTIGATE"
  },
  "metrics": {
    "spam_probability": 0.94,
    "bot_probability": 0.87,
    "manipulation_indicators": ["coordinated_timing", "identical_content", "fake_engagement"]
  }
}
```

## Microservices Architecture

### 1. Social Media Monitoring Service
**Purpose**: Real-time social media content ingestion with platform-specific optimizations
**Technology**: Python + Tweepy + PRAW (Reddit) + aiohttp
**Scaling**: Horizontal by platform, rate-limited by API quotas
**NFRs**: P99 ingestion latency < 2s, 99.5% uptime, handle 10K posts/minute

### 2. News Aggregation Service
**Purpose**: RSS feed monitoring and free news source aggregation
**Technology**: Python + feedparser + BeautifulSoup + Scrapy
**Scaling**: Horizontal by source groups
**NFRs**: P99 processing latency < 5s, 99.9% uptime, handle 1K articles/hour

### 3. Content Quality Service
**Purpose**: Spam detection, bot identification, and source credibility assessment
**Technology**: Python + scikit-learn + NetworkX + spaCy
**Scaling**: Horizontal by content volume
**NFRs**: P99 quality assessment < 500ms, 99.95% spam detection accuracy

### 4. NLP Processing Service
**Purpose**: Multi-language entity extraction, translation, and text preprocessing
**Technology**: Python + spaCy + Transformers + FastText
**Scaling**: Horizontal with GPU acceleration
**NFRs**: P99 processing latency < 1s, support 15+ languages, 95% entity accuracy

### 5. Sentiment Analysis Service
**Purpose**: Multi-model sentiment analysis with financial domain specialization
**Technology**: Python + FinBERT + VADER + Transformers
**Scaling**: Horizontal with GPU clusters
**NFRs**: P99 analysis latency < 800ms, 90% sentiment accuracy, 85% confidence calibration

### 6. Impact Assessment Service
**Purpose**: Market impact prediction and historical correlation analysis
**Technology**: Python + scikit-learn + pandas + NumPy
**Scaling**: Horizontal by entity groups
**NFRs**: P99 assessment latency < 1.5s, 75% impact prediction accuracy

### 7. Intelligence Distribution Service
**Purpose**: Event streaming, alert generation, and API management
**Technology**: Go + Apache Pulsar + Redis
**Scaling**: Horizontal by topic partitions
**NFRs**: P99 distribution latency < 100ms, exactly-once delivery guarantees

## Messaging Technology Strategy

### Apache Pulsar (Primary for Real-time Intelligence)
**Use Cases**:
- **Breaking news streams**: Ultra-low latency for market-moving events
- **Social media firehose**: High-throughput social media processing
- **Quality-based routing**: Route high-quality content to trading systems
- **Geographic distribution**: Multi-region intelligence distribution
- **Schema evolution**: Evolving sentiment and impact models

**Configuration**:
```yaml
pulsar:
  topics:
    - "intelligence/social-media/{platform}/{quality_tier}"
    - "intelligence/news/{source_type}/{impact_level}"
    - "intelligence/sentiment/{entity_type}/{timeframe}"
    - "intelligence/alerts/{severity}/{entity}"
  retention:
    real_time_intelligence: "24 hours"
    historical_sentiment: "1 year"
    quality_alerts: "30 days"
  replication:
    clusters: ["us-east", "us-west", "eu-central"]
```

### Apache Kafka (Batch Processing & Analytics)
**Use Cases**:
- **Historical analysis**: Long-term sentiment trend analysis
- **Model training**: ML model training data pipelines
- **Compliance reporting**: Audit trails for intelligence sources
- **Data lake integration**: Feed data warehouses for research

## Free Data Sources Strategy

### Social Media Sources (Primary Focus)
#### Twitter/X (Free Tier)
- **Rate limits**: 300 requests/15min, 10K tweets/month
- **Content focus**: Financial hashtags (#earnings, #stocks), verified accounts
- **Quality indicators**: Verification status, follower count, engagement rates
- **Monitoring strategy**: Track financial influencers, breaking news accounts

#### Reddit (Free API)
- **Subreddits**: r/investing, r/stocks, r/SecurityAnalysis, r/wallstreetbets
- **Rate limits**: 60 requests/minute
- **Quality indicators**: Upvote ratios, comment quality, user karma
- **Content filtering**: Focus on DD (Due Diligence) posts, earnings discussions

#### Discord/Telegram (Public Channels)
- **Financial communities**: Public investment discussion groups
- **Real-time monitoring**: WebSocket connections for live discussions
- **Quality challenges**: Higher noise ratio, requires aggressive filtering

### News Sources (Free Tiers)
#### RSS Feeds
- **Yahoo Finance**: Company news, earnings announcements
- **MarketWatch**: Market analysis, economic news
- **Seeking Alpha**: Free articles, earnings previews
- **Google News**: Aggregated financial news

#### Economic Data
- **FRED (Federal Reserve)**: Economic indicators, interest rates
- **Trading Economics**: Global economic calendar
- **Investing.com**: Economic events, earnings calendar

### Alternative Data (Free Sources)
#### Google Trends
- **Search volume**: Company names, financial terms
- **Geographic trends**: Regional interest patterns
- **Correlation analysis**: Search volume vs. stock performance

#### GitHub Activity (for Tech Companies)
- **Repository metrics**: Commits, stars, forks
- **Developer activity**: Hiring indicators, project momentum
- **Open source adoption**: Technology trend indicators

## Quality Assurance Framework for Free Sources

### Multi-Tier Quality Classification
```python
class ContentQualityTier:
    TIER_1_PREMIUM = {
        'sources': ['verified_twitter_accounts', 'established_news_sites'],
        'min_credibility': 0.8,
        'use_case': 'real_time_trading_decisions',
        'latency_target': '< 1s'
    }

    TIER_2_STANDARD = {
        'sources': ['reddit_high_karma', 'financial_blogs'],
        'min_credibility': 0.6,
        'use_case': 'sentiment_analysis',
        'latency_target': '< 5s'
    }

    TIER_3_RESEARCH = {
        'sources': ['general_social_media', 'unverified_sources'],
        'min_credibility': 0.4,
        'use_case': 'trend_analysis',
        'latency_target': '< 30s'
    }
```

### Source Reliability Tracking
```python
def update_source_reliability(source_id, prediction, actual_outcome):
    """Update source reliability based on prediction accuracy"""
    source = get_source(source_id)

    # Calculate prediction accuracy
    accuracy = calculate_prediction_accuracy(prediction, actual_outcome)

    # Update running average with decay factor
    decay_factor = 0.95
    source.reliability_score = (
        source.reliability_score * decay_factor +
        accuracy * (1 - decay_factor)
    )

    # Adjust content weighting
    if source.reliability_score < 0.3:
        source.status = 'QUARANTINED'
    elif source.reliability_score < 0.5:
        source.status = 'LOW_PRIORITY'
    else:
        source.status = 'ACTIVE'

    save_source(source)
```

### Spam and Manipulation Detection
#### Social Media Bot Detection
```python
def detect_bot_probability(account):
    features = {
        'account_age': account.created_days_ago,
        'follower_ratio': account.followers / max(account.following, 1),
        'posting_frequency': account.posts_per_day,
        'engagement_rate': account.avg_engagement / max(account.followers, 1),
        'profile_completeness': calculate_profile_completeness(account),
        'posting_pattern_regularity': analyze_posting_times(account)
    }

    return bot_detection_model.predict_proba(features)[1]
```

#### Coordinated Manipulation Detection
```python
def detect_coordinated_activity(content_batch):
    """Detect coordinated posting patterns"""
    similarities = []

    for content1, content2 in combinations(content_batch, 2):
        similarity = {
            'text_similarity': calculate_text_similarity(content1.text, content2.text),
            'timing_similarity': calculate_timing_similarity(content1.timestamp, content2.timestamp),
            'account_similarity': calculate_account_similarity(content1.author, content2.author),
            'engagement_similarity': calculate_engagement_similarity(content1.engagement, content2.engagement)
        }
        similarities.append(similarity)

    coordination_score = calculate_coordination_score(similarities)
    return coordination_score > COORDINATION_THRESHOLD
```

## Performance Optimizations for High-Volume Processing

### Social Media Stream Processing
```python
async def process_social_media_stream():
    """Optimized social media processing pipeline"""

    # Parallel processing with asyncio
    async with aiohttp.ClientSession() as session:
        tasks = []

        # Create processing tasks for each platform
        for platform in ['twitter', 'reddit', 'discord']:
            task = asyncio.create_task(
                process_platform_stream(platform, session)
            )
            tasks.append(task)

        # Process with backpressure handling
        await asyncio.gather(*tasks, return_exceptions=True)

async def process_platform_stream(platform, session):
    """Platform-specific stream processing"""
    rate_limiter = RateLimiter(platform.rate_limit)

    async for content in platform.stream():
        await rate_limiter.acquire()

        # Quick quality pre-filter
        if quick_quality_check(content):
            await process_content_async(content)
        else:
            log_filtered_content(content, reason='quality_prefilter')
```

### Caching Strategy for Free APIs
```python
class IntelligentCache:
    def __init__(self):
        self.redis_client = redis.Redis()
        self.cache_strategies = {
            'twitter_user_info': {'ttl': 3600, 'strategy': 'write_through'},
            'reddit_post_details': {'ttl': 1800, 'strategy': 'write_behind'},
            'sentiment_results': {'ttl': 300, 'strategy': 'write_around'},
            'entity_mappings': {'ttl': 86400, 'strategy': 'write_through'}
        }

    async def get_or_compute(self, key, compute_func, cache_type):
        """Intelligent caching with different strategies"""
        strategy = self.cache_strategies[cache_type]

        # Try cache first
        cached_value = await self.redis_client.get(key)
        if cached_value:
            return json.loads(cached_value)

        # Compute value
        value = await compute_func()

        # Cache based on strategy
        if strategy['strategy'] in ['write_through', 'write_behind']:
            await self.redis_client.setex(
                key, strategy['ttl'], json.dumps(value)
            )

        return value

## Data Storage Strategy

### Elasticsearch (Primary Text Search & Analytics)
- **Content indexing**: Full-text search across all collected content
- **Entity-based queries**: Find all content mentioning specific companies
- **Sentiment time-series**: Historical sentiment trends by entity
- **Real-time analytics**: Aggregations for trending topics and sentiment shifts

### PostgreSQL (Metadata & Relationships)
- **Source management**: Credibility scores, API configurations
- **Entity master data**: Company information, symbol mappings
- **User preferences**: Personalized intelligence feeds
- **Quality metrics**: Historical accuracy and reliability tracking

### Redis (Real-time Caching & Queues)
- **API rate limiting**: Track and enforce rate limits per source
- **Real-time sentiment**: Latest sentiment scores for quick access
- **Processing queues**: Async job queues for NLP processing
- **Session management**: User sessions and preferences

### TimescaleDB (Time-series Intelligence Metrics)
- **Sentiment trends**: Time-series sentiment data by entity
- **Impact correlations**: Historical impact vs. actual market movements
- **Source performance**: Accuracy metrics over time
- **Volume metrics**: Content volume and processing statistics

## Monitoring and Alerting

### Key Metrics for Free Sources
- **API quota utilization**: Track usage against free tier limits
- **Content quality scores**: Real-time quality assessment metrics
- **Source reliability trends**: Degradation in source accuracy
- **Processing latency**: End-to-end intelligence generation time
- **False positive rates**: Spam/bot detection accuracy

### Alert Conditions
- **API limit approaching**: 80% of quota used
- **Quality degradation**: Source reliability drops below 0.5
- **Spam surge detected**: Coordinated manipulation attempts
- **Processing backlog**: Queue depth exceeds 1000 items
- **Model drift**: Sentiment accuracy drops below 85%

## Usage by Downstream Services

### ML Prediction Service
- **Consumes**: `NewsSentimentAnalyzedEvent`, `MarketImpactAssessmentEvent`
- **Requirements**: High-quality sentiment features, impact predictions
- **SLA**: < 200ms for feature extraction, 90% sentiment accuracy

### Trading Strategy Service
- **Consumes**: `MarketImpactAssessmentEvent` for strategy adjustments
- **Requirements**: Real-time high-impact events, low false positive rate
- **SLA**: < 500ms for critical market-moving events

### Risk Analysis Service
- **Consumes**: `MarketImpactAssessmentEvent`, `ContentQualityAlertEvent`
- **Requirements**: Risk-relevant news, manipulation detection alerts
- **SLA**: < 1s for risk assessment updates

### Portfolio Optimization Service
- **Consumes**: `NewsSentimentAnalyzedEvent` for sentiment-based adjustments
- **Requirements**: Entity-specific sentiment, sector impact analysis
- **SLA**: < 2s for portfolio impact assessment

### Reporting Service
- **Consumes**: All events for intelligence dashboards and reports
- **Requirements**: Historical trends, source attribution, quality metrics
- **SLA**: < 10s for dashboard updates

## Implementation Roadmap

### Phase 1: Foundation & Social Media (Weeks 1-6)
- Set up Twitter and Reddit monitoring services
- Implement basic spam and bot detection
- Deploy content quality service with initial models
- Set up Elasticsearch for content indexing

### Phase 2: NLP & Sentiment Analysis (Weeks 7-12)
- Deploy multi-model sentiment analysis pipeline
- Implement entity extraction and linking
- Add multi-language support for major languages
- Integrate FinBERT for financial sentiment

### Phase 3: Quality & Reliability (Weeks 13-18)
- Implement advanced bot detection algorithms
- Deploy source reliability tracking system
- Add coordinated manipulation detection
- Implement feedback loops for model improvement

### Phase 4: Impact Assessment & Distribution (Weeks 19-24)
- Deploy market impact assessment service
- Implement historical correlation analysis
- Set up Apache Pulsar for real-time distribution
- Add personalized intelligence feeds

### Phase 5: Scale & Optimize (Weeks 25-30)
- Add remaining free data sources
- Implement advanced caching strategies
- Optimize for high-throughput processing
- Add predictive quality scoring

### Phase 6: Advanced Features (Weeks 31-36)
- Machine learning-based source discovery
- Automated model retraining pipelines
- Advanced manipulation detection
- Cross-platform correlation analysis

## Cost Optimization for Free Sources

### API Quota Management
```python
class QuotaManager:
    def __init__(self):
        self.quotas = {
            'twitter': {'limit': 300, 'window': 900, 'used': 0},
            'reddit': {'limit': 60, 'window': 60, 'used': 0},
            'google_trends': {'limit': 100, 'window': 3600, 'used': 0}
        }

    async def can_make_request(self, source):
        quota = self.quotas[source]
        if quota['used'] >= quota['limit']:
            return False
        return True

    async def record_request(self, source):
        self.quotas[source]['used'] += 1

    async def reset_quota(self, source):
        """Reset quota after time window"""
        self.quotas[source]['used'] = 0
```

### Intelligent Content Prioritization
```python
def prioritize_content_processing(content_batch):
    """Prioritize processing based on potential value"""

    priority_scores = []
    for content in content_batch:
        score = calculate_priority_score(content)
        priority_scores.append((score, content))

    # Sort by priority and process high-value content first
    priority_scores.sort(reverse=True)

    # Process within API limits
    processed = 0
    for score, content in priority_scores:
        if can_process_more():
            process_content(content)
            processed += 1
        else:
            queue_for_later(content)

    return processed

def calculate_priority_score(content):
    """Calculate content processing priority"""
    factors = {
        'source_credibility': content.source.credibility_score,
        'engagement_level': content.engagement.total / content.source.avg_engagement,
        'entity_relevance': max([e.relevance for e in content.entities]),
        'recency': calculate_recency_score(content.timestamp),
        'uniqueness': 1 - content.duplicate_probability
    }

    weights = {
        'source_credibility': 0.3,
        'engagement_level': 0.2,
        'entity_relevance': 0.25,
        'recency': 0.15,
        'uniqueness': 0.1
    }

    return sum(factor * weights[name] for name, factor in factors.items())
```

## Security Considerations for Social Media Data

### Data Privacy & Compliance
- **GDPR compliance**: Anonymize personal data, respect deletion requests
- **Platform ToS compliance**: Respect rate limits, attribution requirements
- **Data retention**: Automatic deletion of personal information
- **Access controls**: Role-based access to sensitive intelligence data

### API Security
- **Key rotation**: Regular rotation of API keys and tokens
- **Rate limit monitoring**: Prevent accidental quota exhaustion
- **Error handling**: Graceful degradation when APIs are unavailable
- **Audit logging**: Complete audit trail of all API interactions

This comprehensive refinement addresses your focus on free and social media sources while emphasizing the critical quality assurance needed for reliable intelligence extraction from potentially noisy sources.
```