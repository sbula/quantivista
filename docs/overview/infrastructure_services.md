# Infrastructure Services

This document provides detailed information about the shared infrastructure services in the QuantVista platform. These services provide the foundation for communication, data storage, and system management.


## Messaging Infrastructure

### Apache Pulsar

#### Purpose
Apache Pulsar serves as the primary messaging system for the platform, providing reliable, high-throughput message streaming with multi-tenancy support. It enables event-driven architecture and asynchronous communication between services.

#### Key Features
- Multi-tenancy for isolation between different components
- Geo-replication for disaster recovery
- Tiered storage for cost-effective message retention
- Schema registry for message validation
- Exactly-once processing semantics
- Pulsar Functions for lightweight stream processing

#### Used By
- All microservices for event-driven communication
- CQRS implementation for event sourcing
- Data pipelines for streaming analytics
- Integration with external systems

### Aeron

#### Purpose
Aeron provides ultra-low latency messaging for performance-critical paths in the trading system, particularly between the Market Data Service and Trading Engine.

#### Key Features
- Sub-50µs latency messaging
- Lock-free, wait-free algorithms
- Reliable UDP transport
- Efficient memory usage with zero-copy operations
- High throughput (millions of messages per second)
- Clustered deployment for fault tolerance

#### Used By
- Market Data Service to Trading Engine communication
- Trading Engine to Risk Management critical paths
- Any component requiring ultra-low latency messaging



## Database Infrastructure

### QuestDB

#### Purpose
QuestDB serves as the primary time-series database for storing market data, trading performance metrics, and other time-series information with high-performance query capabilities.

#### Key Features
- Column-oriented storage optimized for time-series data
- SQL query language with time-series extensions
- High-throughput ingestion (millions of rows per second)
- Low-latency queries for real-time analytics
- Native support for financial time-series operations
- Efficient downsampling and aggregation

#### Used By
- Market Data Service for historical data storage
- Trading Engine for performance metrics
- Analytics Service for time-series analysis
- ML Prediction Service for feature storage

### PostgreSQL

#### Purpose
PostgreSQL serves as the relational database for structured data, user information, order history, and other transactional data requiring ACID compliance.

#### Key Features
- ACID-compliant transactions
- Advanced indexing capabilities
- JSON/JSONB support for flexible schemas
- Strong data integrity constraints
- Mature ecosystem and tooling
- Event triggers for change data capture

#### Used By
- User Service for account information
- Order Service for order history
- Event Store for CQRS implementation
- Configuration management
- Any service requiring transactional data storage



## Observability Infrastructure

### OpenTelemetry

#### Purpose
OpenTelemetry provides a unified framework for collecting traces, metrics, and logs from all services, enabling comprehensive observability across the platform.

#### Key Features
- Vendor-neutral instrumentation APIs
- Support for distributed tracing
- Metrics collection and aggregation
- Context propagation across service boundaries
- Integration with multiple backends (Jaeger, Prometheus)
- Auto-instrumentation for common frameworks

#### Used By
- All microservices for instrumentation
- API Gateway for request tracing
- Performance monitoring systems
- SLA compliance tracking

### Jaeger

#### Purpose
Jaeger provides distributed tracing capabilities, allowing visualization and analysis of request flows across multiple services.

#### Key Features
- End-to-end distributed tracing
- Trace visualization and exploration
- Root cause analysis tools
- Service dependency diagrams
- Performance bottleneck identification
- Sampling strategies for high-volume systems

#### Used By
- Operations teams for troubleshooting
- Developers for performance analysis
- Automated alerting systems
- SLA monitoring

### Prometheus & Grafana

#### Purpose
Prometheus collects and stores metrics from all services, while Grafana provides visualization and dashboarding capabilities for monitoring and alerting.

#### Key Features
- Time-series metric collection
- Powerful query language (PromQL)
- Alerting rules and notifications
- Service discovery integration
- Long-term storage with Thanos
- Rich visualization with Grafana

#### Used By
- System monitoring and alerting
- Performance dashboards
- Capacity planning
- SLA tracking
- Business metrics visualization
