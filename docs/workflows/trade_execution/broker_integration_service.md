# Broker Integration Service

## Purpose and Boundaries

### Purpose
The Broker Integration Service provides unified access to multiple brokers with intelligent routing and execution optimization. It serves as the abstraction layer between the QuantiVista platform and various external brokerage APIs, ensuring reliable order execution and standardized communication.

### Strict Boundaries
- **Responsible for**: Broker connectivity, API management, intelligent order routing, execution quality monitoring, and transaction cost analysis
- **Not responsible for**: Order lifecycle management, trading decisions, portfolio management, or risk assessment
- **Interfaces with**: Order Management Service (receives orders for execution), Market Data Service (for real-time pricing), Risk Analysis Service (for execution risk assessment)

## Place in Workflow

The Broker Integration Service is a critical component in the Trade Execution Workflow:

1. **Receives orders** from the Order Management Service
2. **Selects optimal broker** based on execution requirements, costs, and market conditions
3. **Translates orders** into broker-specific formats and protocols
4. **Routes orders** to selected brokers
5. **Monitors execution** in real-time
6. **Receives execution reports** from brokers
7. **Forwards execution reports** to the Order Management Service
8. **Performs transaction cost analysis** to evaluate execution quality
9. **Maintains broker connectivity** and handles failover scenarios

## API Design (API-First)

### REST API

#### Broker Management Endpoints

```yaml
openapi: 3.0.3
info:
  title: Broker Integration Service API
  version: 1.0.0
  description: API for managing broker connections and order execution
paths:
  /brokers:
    get:
      summary: List available brokers
      operationId: listBrokers
      tags: [Brokers]
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [ACTIVE, INACTIVE, MAINTENANCE]
        - name: assetClass
          in: query
          schema:
            type: string
            enum: [EQUITY, FOREX, CRYPTO, FUTURES, OPTIONS]
      responses:
        '200':
          description: List of brokers
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/BrokerInfo'
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /brokers/{brokerId}/status:
    get:
      summary: Get broker connection status
      operationId: getBrokerStatus
      tags: [Brokers]
      parameters:
        - name: brokerId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Broker status
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BrokerStatus'
        '404':
          description: Broker not found
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /execution/route:
    post:
      summary: Route order to optimal broker
      operationId: routeOrder
      tags: [Execution]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderRoutingRequest'
      responses:
        '202':
          description: Order accepted for routing
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderRoutingResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /execution/{executionId}:
    get:
      summary: Get execution details
      operationId: getExecution
      tags: [Execution]
      parameters:
        - name: executionId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Execution details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ExecutionDetails'
        '404':
          description: Execution not found
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /execution/cancel:
    post:
      summary: Cancel order at broker
      operationId: cancelOrder
      tags: [Execution]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CancelOrderRequest'
      responses:
        '202':
          description: Cancellation request accepted
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CancelOrderResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /analysis/tca:
    post:
      summary: Get transaction cost analysis
      operationId: getTransactionCostAnalysis
      tags: [Analysis]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/TCARequest'
      responses:
        '200':
          description: Transaction cost analysis
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TCAResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    BrokerInfo:
      type: object
      properties:
        id:
          type: string
          description: Broker ID
        name:
          type: string
          description: Broker name
        status:
          type: string
          enum: [ACTIVE, INACTIVE, MAINTENANCE]
          description: Current broker status
        supportedAssetClasses:
          type: array
          items:
            type: string
            enum: [EQUITY, FOREX, CRYPTO, FUTURES, OPTIONS]
          description: Asset classes supported by this broker
        features:
          type: array
          items:
            type: string
            enum: [MARKET_ORDERS, LIMIT_ORDERS, STOP_ORDERS, STOP_LIMIT_ORDERS, TRAILING_STOPS, OCO_ORDERS, ALGO_ORDERS]
          description: Features supported by this broker
        commissionStructure:
          type: object
          properties:
            baseRate:
              type: number
              format: double
              description: Base commission rate
            minCommission:
              type: number
              format: double
              description: Minimum commission per trade
            tieredRates:
              type: array
              items:
                type: object
                properties:
                  volumeThreshold:
                    type: number
                    format: double
                  rate:
                    type: number
                    format: double
        latencyStats:
          type: object
          properties:
            averageMs:
              type: number
              format: double
              description: Average latency in milliseconds
            p95Ms:
              type: number
              format: double
              description: 95th percentile latency in milliseconds
            p99Ms:
              type: number
              format: double
              description: 99th percentile latency in milliseconds
    
    BrokerStatus:
      type: object
      properties:
        brokerId:
          type: string
          description: Broker ID
        status:
          type: string
          enum: [CONNECTED, DISCONNECTED, DEGRADED, MAINTENANCE]
          description: Connection status
        uptime:
          type: number
          format: double
          description: Uptime percentage in the last 24 hours
        lastConnected:
          type: string
          format: date-time
          description: Last successful connection timestamp
        issues:
          type: array
          items:
            type: object
            properties:
              type:
                type: string
                enum: [CONNECTIVITY, AUTHENTICATION, RATE_LIMIT, API_ERROR]
              message:
                type: string
              timestamp:
                type: string
                format: date-time
        currentLoad:
          type: number
          format: double
          description: Current load percentage
    
    OrderRoutingRequest:
      type: object
      required: [orderId, accountId, symbol, side, type, quantity]
      properties:
        orderId:
          type: string
          description: Order ID from Order Management Service
        accountId:
          type: string
          description: Account identifier
        symbol:
          type: string
          description: Instrument symbol
        side:
          type: string
          enum: [BUY, SELL]
          description: Order side
        type:
          type: string
          enum: [MARKET, LIMIT, STOP, STOP_LIMIT]
          description: Order type
        quantity:
          type: number
          format: double
          description: Order quantity
        price:
          type: number
          format: double
          description: Limit price (required for LIMIT and STOP_LIMIT orders)
        stopPrice:
          type: number
          format: double
          description: Stop price (required for STOP and STOP_LIMIT orders)
        timeInForce:
          type: string
          enum: [DAY, GTC, IOC, FOK]
          default: DAY
          description: Time in force
        preferredBrokers:
          type: array
          items:
            type: string
          description: Preferred brokers in order of preference (optional)
        executionStrategy:
          type: string
          enum: [LOWEST_COST, FASTEST_EXECUTION, HIGHEST_FILL_PROBABILITY, MINIMAL_MARKET_IMPACT, SMART]
          default: SMART
          description: Execution strategy preference
        urgency:
          type: string
          enum: [LOW, MEDIUM, HIGH, CRITICAL]
          default: MEDIUM
          description: Order urgency
    
    OrderRoutingResponse:
      type: object
      properties:
        routingId:
          type: string
          description: Routing request ID
        orderId:
          type: string
          description: Original order ID
        selectedBroker:
          type: string
          description: ID of the selected broker
        brokerOrderId:
          type: string
          description: Order ID assigned by the broker
        status:
          type: string
          enum: [ACCEPTED, REJECTED, PENDING]
          description: Routing status
        rejectionReason:
          type: string
          description: Reason for rejection (if status is REJECTED)
        estimatedExecutionTime:
          type: number
          format: double
          description: Estimated execution time in milliseconds
        timestamp:
          type: string
          format: date-time
          description: Routing timestamp
    
    ExecutionDetails:
      type: object
      properties:
        executionId:
          type: string
          description: Execution ID
        orderId:
          type: string
          description: Original order ID
        brokerOrderId:
          type: string
          description: Broker's order ID
        brokerId:
          type: string
          description: Broker ID
        status:
          type: string
          enum: [PENDING, EXECUTING, COMPLETED, PARTIALLY_FILLED, CANCELED, REJECTED]
          description: Execution status
        fills:
          type: array
          items:
            type: object
            properties:
              fillId:
                type: string
              quantity:
                type: number
                format: double
              price:
                type: number
                format: double
              timestamp:
                type: string
                format: date-time
              venue:
                type: string
        averagePrice:
          type: number
          format: double
          description: Average execution price
        totalQuantity:
          type: number
          format: double
          description: Total order quantity
        filledQuantity:
          type: number
          format: double
          description: Filled quantity
        fees:
          type: number
          format: double
          description: Total fees
        startTime:
          type: string
          format: date-time
          description: Execution start time
        endTime:
          type: string
          format: date-time
          description: Execution end time
        marketImpact:
          type: number
          format: double
          description: Estimated market impact in basis points
    
    CancelOrderRequest:
      type: object
      required: [orderId, brokerId]
      properties:
        orderId:
          type: string
          description: Original order ID
        brokerId:
          type: string
          description: Broker ID
        brokerOrderId:
          type: string
          description: Broker's order ID
        reason:
          type: string
          description: Cancellation reason
    
    CancelOrderResponse:
      type: object
      properties:
        requestId:
          type: string
          description: Cancellation request ID
        orderId:
          type: string
          description: Original order ID
        status:
          type: string
          enum: [ACCEPTED, REJECTED, PENDING]
          description: Cancellation status
        rejectionReason:
          type: string
          description: Reason for rejection (if status is REJECTED)
        timestamp:
          type: string
          format: date-time
          description: Cancellation request timestamp
    
    TCARequest:
      type: object
      required: [orderId]
      properties:
        orderId:
          type: string
          description: Order ID
        executionId:
          type: string
          description: Execution ID (if available)
        benchmark:
          type: string
          enum: [ARRIVAL_PRICE, VWAP, TWAP, CLOSE_PRICE, IMPLEMENTATION_SHORTFALL]
          default: ARRIVAL_PRICE
          description: TCA benchmark
    
    TCAResponse:
      type: object
      properties:
        orderId:
          type: string
          description: Order ID
        executionId:
          type: string
          description: Execution ID
        benchmark:
          type: string
          enum: [ARRIVAL_PRICE, VWAP, TWAP, CLOSE_PRICE, IMPLEMENTATION_SHORTFALL]
          description: TCA benchmark
        benchmarkPrice:
          type: number
          format: double
          description: Benchmark price
        averageExecutionPrice:
          type: number
          format: double
          description: Average execution price
        slippageBps:
          type: number
          format: double
          description: Slippage in basis points
        marketImpactBps:
          type: number
          format: double
          description: Market impact in basis points
        totalCostBps:
          type: number
          format: double
          description: Total execution cost in basis points
        savingsOpportunityBps:
          type: number
          format: double
          description: Potential savings opportunity in basis points
        executionQualityScore:
          type: number
          format: double
          description: Execution quality score (0-100)
        brokerPerformanceRank:
          type: integer
          description: Broker performance rank for this asset class
        recommendations:
          type: array
          items:
            type: object
            properties:
              type:
                type: string
                enum: [BROKER_SELECTION, EXECUTION_STRATEGY, TIMING, ORDER_TYPE]
              description:
                type: string
              estimatedImprovementBps:
                type: number
                format: double
  
  responses:
    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
              errors:
                type: array
                items:
                  type: object
                  properties:
                    field:
                      type: string
                    message:
                      type: string
    
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
                example: Authentication required
```

### gRPC API (Internal Services)

```protobuf
syntax = "proto3";

package trading.broker.v1;

import "google/protobuf/timestamp.proto";

service BrokerService {
  // Route order to optimal broker
  rpc RouteOrder(OrderRoutingRequest) returns (OrderRoutingResponse);
  
  // Get execution details
  rpc GetExecution(GetExecutionRequest) returns (ExecutionDetails);
  
  // Cancel order at broker
  rpc CancelOrder(CancelOrderRequest) returns (CancelOrderResponse);
  
  // Stream execution updates
  rpc StreamExecutionUpdates(StreamExecutionUpdatesRequest) returns (stream ExecutionUpdate);
  
  // Get broker status
  rpc GetBrokerStatus(GetBrokerStatusRequest) returns (BrokerStatus);
  
  // Get transaction cost analysis
  rpc GetTransactionCostAnalysis(TCARequest) returns (TCAResponse);
}

message OrderRoutingRequest {
  string order_id = 1;
  string account_id = 2;
  string symbol = 3;
  OrderSide side = 4;
  OrderType type = 5;
  double quantity = 6;
  double price = 7;
  double stop_price = 8;
  TimeInForce time_in_force = 9;
  repeated string preferred_brokers = 10;
  ExecutionStrategy execution_strategy = 11;
  OrderUrgency urgency = 12;
}

message OrderRoutingResponse {
  string routing_id = 1;
  string order_id = 2;
  string selected_broker = 3;
  string broker_order_id = 4;
  RoutingStatus status = 5;
  string rejection_reason = 6;
  double estimated_execution_time = 7;
  google.protobuf.Timestamp timestamp = 8;
}

message GetExecutionRequest {
  string execution_id = 1;
}

message ExecutionDetails {
  string execution_id = 1;
  string order_id = 2;
  string broker_order_id = 3;
  string broker_id = 4;
  ExecutionStatus status = 5;
  repeated Fill fills = 6;
  double average_price = 7;
  double total_quantity = 8;
  double filled_quantity = 9;
  double fees = 10;
  google.protobuf.Timestamp start_time = 11;
  google.protobuf.Timestamp end_time = 12;
  double market_impact = 13;
}

message Fill {
  string fill_id = 1;
  double quantity = 2;
  double price = 3;
  google.protobuf.Timestamp timestamp = 4;
  string venue = 5;
}

message CancelOrderRequest {
  string order_id = 1;
  string broker_id = 2;
  string broker_order_id = 3;
  string reason = 4;
}

message CancelOrderResponse {
  string request_id = 1;
  string order_id = 2;
  CancellationStatus status = 3;
  string rejection_reason = 4;
  google.protobuf.Timestamp timestamp = 5;
}

message StreamExecutionUpdatesRequest {
  repeated string order_ids = 1;
  repeated string broker_ids = 2;
  repeated ExecutionStatus statuses = 3;
}

message ExecutionUpdate {
  string execution_id = 1;
  string order_id = 2;
  ExecutionStatus status = 3;
  Fill latest_fill = 4;
  double filled_quantity = 5;
  double average_price = 6;
  google.protobuf.Timestamp timestamp = 7;
}

message GetBrokerStatusRequest {
  string broker_id = 1;
}

message BrokerStatus {
  string broker_id = 1;
  ConnectionStatus status = 2;
  double uptime = 3;
  google.protobuf.Timestamp last_connected = 4;
  repeated BrokerIssue issues = 5;
  double current_load = 6;
}

message BrokerIssue {
  IssueType type = 1;
  string message = 2;
  google.protobuf.Timestamp timestamp = 3;
}

message TCARequest {
  string order_id = 1;
  string execution_id = 2;
  TCABenchmark benchmark = 3;
}

message TCAResponse {
  string order_id = 1;
  string execution_id = 2;
  TCABenchmark benchmark = 3;
  double benchmark_price = 4;
  double average_execution_price = 5;
  double slippage_bps = 6;
  double market_impact_bps = 7;
  double total_cost_bps = 8;
  double savings_opportunity_bps = 9;
  double execution_quality_score = 10;
  int32 broker_performance_rank = 11;
  repeated TCARecommendation recommendations = 12;
}

message TCARecommendation {
  RecommendationType type = 1;
  string description = 2;
  double estimated_improvement_bps = 3;
}

enum OrderSide {
  ORDER_SIDE_UNSPECIFIED = 0;
  ORDER_SIDE_BUY = 1;
  ORDER_SIDE_SELL = 2;
}

enum OrderType {
  ORDER_TYPE_UNSPECIFIED = 0;
  ORDER_TYPE_MARKET = 1;
  ORDER_TYPE_LIMIT = 2;
  ORDER_TYPE_STOP = 3;
  ORDER_TYPE_STOP_LIMIT = 4;
}

enum TimeInForce {
  TIME_IN_FORCE_UNSPECIFIED = 0;
  TIME_IN_FORCE_DAY = 1;
  TIME_IN_FORCE_GTC = 2;
  TIME_IN_FORCE_IOC = 3;
  TIME_IN_FORCE_FOK = 4;
}

enum ExecutionStrategy {
  EXECUTION_STRATEGY_UNSPECIFIED = 0;
  EXECUTION_STRATEGY_LOWEST_COST = 1;
  EXECUTION_STRATEGY_FASTEST_EXECUTION = 2;
  EXECUTION_STRATEGY_HIGHEST_FILL_PROBABILITY = 3;
  EXECUTION_STRATEGY_MINIMAL_MARKET_IMPACT = 4;
  EXECUTION_STRATEGY_SMART = 5;
}

enum OrderUrgency {
  ORDER_URGENCY_UNSPECIFIED = 0;
  ORDER_URGENCY_LOW = 1;
  ORDER_URGENCY_MEDIUM = 2;
  ORDER_URGENCY_HIGH = 3;
  ORDER_URGENCY_CRITICAL = 4;
}

enum RoutingStatus {
  ROUTING_STATUS_UNSPECIFIED = 0;
  ROUTING_STATUS_ACCEPTED = 1;
  ROUTING_STATUS_REJECTED = 2;
  ROUTING_STATUS_PENDING = 3;
}

enum ExecutionStatus {
  EXECUTION_STATUS_UNSPECIFIED = 0;
  EXECUTION_STATUS_PENDING = 1;
  EXECUTION_STATUS_EXECUTING = 2;
  EXECUTION_STATUS_COMPLETED = 3;
  EXECUTION_STATUS_PARTIALLY_FILLED = 4;
  EXECUTION_STATUS_CANCELED = 5;
  EXECUTION_STATUS_REJECTED = 6;
}

enum CancellationStatus {
  CANCELLATION_STATUS_UNSPECIFIED = 0;
  CANCELLATION_STATUS_ACCEPTED = 1;
  CANCELLATION_STATUS_REJECTED = 2;
  CANCELLATION_STATUS_PENDING = 3;
}

enum ConnectionStatus {
  CONNECTION_STATUS_UNSPECIFIED = 0;
  CONNECTION_STATUS_CONNECTED = 1;
  CONNECTION_STATUS_DISCONNECTED = 2;
  CONNECTION_STATUS_DEGRADED = 3;
  CONNECTION_STATUS_MAINTENANCE = 4;
}

enum IssueType {
  ISSUE_TYPE_UNSPECIFIED = 0;
  ISSUE_TYPE_CONNECTIVITY = 1;
  ISSUE_TYPE_AUTHENTICATION = 2;
  ISSUE_TYPE_RATE_LIMIT = 3;
  ISSUE_TYPE_API_ERROR = 4;
}

enum TCABenchmark {
  TCA_BENCHMARK_UNSPECIFIED = 0;
  TCA_BENCHMARK_ARRIVAL_PRICE = 1;
  TCA_BENCHMARK_VWAP = 2;
  TCA_BENCHMARK_TWAP = 3;
  TCA_BENCHMARK_CLOSE_PRICE = 4;
  TCA_BENCHMARK_IMPLEMENTATION_SHORTFALL = 5;
}

enum RecommendationType {
  RECOMMENDATION_TYPE_UNSPECIFIED = 0;
  RECOMMENDATION_TYPE_BROKER_SELECTION = 1;
  RECOMMENDATION_TYPE_EXECUTION_STRATEGY = 2;
  RECOMMENDATION_TYPE_TIMING = 3;
  RECOMMENDATION_TYPE_ORDER_TYPE = 4;
}
```

## Data Model

### Core Entities

#### BrokerConnection
Represents a connection to a specific broker.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BrokerConnection {
    pub id: String,
    pub broker_id: String,
    pub name: String,
    pub status: ConnectionStatus,
    pub supported_asset_classes: Vec<AssetClass>,
    pub features: Vec<BrokerFeature>,
    pub commission_structure: CommissionStructure,
    pub credentials: BrokerCredentials,
    pub connection_params: HashMap<String, String>,
    pub rate_limits: RateLimits,
    pub latency_stats: LatencyStats,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub last_connected_at: Option<DateTime<Utc>>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ConnectionStatus {
    Connected,
    Disconnected,
    Degraded,
    Maintenance,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum AssetClass {
    Equity,
    Forex,
    Crypto,
    Futures,
    Options,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum BrokerFeature {
    MarketOrders,
    LimitOrders,
    StopOrders,
    StopLimitOrders,
    TrailingStops,
    OcoOrders,
    AlgoOrders,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CommissionStructure {
    pub base_rate: Decimal,
    pub min_commission: Decimal,
    pub tiered_rates: Vec<TieredRate>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct TieredRate {
    pub volume_threshold: Decimal,
    pub rate: Decimal,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BrokerCredentials {
    pub api_key: String,
    pub api_secret: String,
    pub passphrase: Option<String>,
    pub access_token: Option<String>,
    pub refresh_token: Option<String>,
    pub expiry: Option<DateTime<Utc>>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct RateLimits {
    pub requests_per_second: u32,
    pub requests_per_minute: u32,
    pub requests_per_hour: u32,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct LatencyStats {
    pub average_ms: f64,
    pub p95_ms: f64,
    pub p99_ms: f64,
    pub last_updated: DateTime<Utc>,
}
```

#### BrokerOrder
Represents an order sent to a broker.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BrokerOrder {
    pub id: String,
    pub order_id: String,
    pub broker_id: String,
    pub broker_order_id: Option<String>,
    pub account_id: String,
    pub symbol: String,
    pub side: OrderSide,
    pub order_type: OrderType,
    pub quantity: Decimal,
    pub price: Option<Decimal>,
    pub stop_price: Option<Decimal>,
    pub time_in_force: TimeInForce,
    pub status: BrokerOrderStatus,
    pub filled_quantity: Decimal,
    pub average_price: Option<Decimal>,
    pub fees: Option<Decimal>,
    pub rejection_reason: Option<String>,
    pub execution_strategy: ExecutionStrategy,
    pub urgency: OrderUrgency,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub fills: Vec<OrderFill>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum OrderSide {
    Buy,
    Sell,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum OrderType {
    Market,
    Limit,
    Stop,
    StopLimit,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum TimeInForce {
    Day,
    GoodTillCanceled,
    ImmediateOrCancel,
    FillOrKill,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum BrokerOrderStatus {
    Pending,
    Accepted,
    Rejected,
    PartiallyFilled,
    Filled,
    Canceled,
    Expired,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ExecutionStrategy {
    LowestCost,
    FastestExecution,
    HighestFillProbability,
    MinimalMarketImpact,
    Smart,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum OrderUrgency {
    Low,
    Medium,
    High,
    Critical,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OrderFill {
    pub id: String,
    pub broker_fill_id: String,
    pub quantity: Decimal,
    pub price: Decimal,
    pub fees: Decimal,
    pub timestamp: DateTime<Utc>,
    pub venue: String,
}
```

#### ExecutionReport
Represents an execution report for transaction cost analysis.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ExecutionReport {
    pub id: String,
    pub order_id: String,
    pub broker_order_id: String,
    pub broker_id: String,
    pub symbol: String,
    pub side: OrderSide,
    pub quantity: Decimal,
    pub filled_quantity: Decimal,
    pub average_price: Decimal,
    pub benchmark_price: Option<Decimal>,
    pub benchmark_type: BenchmarkType,
    pub slippage_bps: Option<f64>,
    pub market_impact_bps: Option<f64>,
    pub total_cost_bps: Option<f64>,
    pub execution_start: DateTime<Utc>,
    pub execution_end: DateTime<Utc>,
    pub execution_duration_ms: u64,
    pub fills: Vec<OrderFill>,
    pub execution_quality_score: Option<f64>,
    pub broker_performance_rank: Option<u32>,
    pub recommendations: Vec<ExecutionRecommendation>,
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum BenchmarkType {
    ArrivalPrice,
    Vwap,
    Twap,
    ClosePrice,
    ImplementationShortfall,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ExecutionRecommendation {
    pub recommendation_type: RecommendationType,
    pub description: String,
    pub estimated_improvement_bps: f64,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum RecommendationType {
    BrokerSelection,
    ExecutionStrategy,
    Timing,
    OrderType,
}
```

## Database Schema (CQRS Pattern)

### Write Model (Command Side)

#### broker_connections Table
```sql
CREATE TABLE broker_connections (
    id VARCHAR(36) PRIMARY KEY,
    broker_id VARCHAR(36) NOT NULL,
    name VARCHAR(100) NOT NULL,
    status VARCHAR(20) NOT NULL,
    supported_asset_classes JSONB NOT NULL,
    features JSONB NOT NULL,
    commission_structure JSONB NOT NULL,
    credentials JSONB NOT NULL,
    connection_params JSONB NOT NULL,
    rate_limits JSONB NOT NULL,
    latency_stats JSONB NOT NULL,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    last_connected_at TIMESTAMP
);

CREATE INDEX idx_broker_connections_broker_id ON broker_connections(broker_id);
CREATE INDEX idx_broker_connections_status ON broker_connections(status);
```

#### broker_orders Table
```sql
CREATE TABLE broker_orders (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL,
    broker_id VARCHAR(36) NOT NULL,
    broker_order_id VARCHAR(100),
    account_id VARCHAR(36) NOT NULL,
    symbol VARCHAR(20) NOT NULL,
    side VARCHAR(10) NOT NULL,
    order_type VARCHAR(15) NOT NULL,
    quantity DECIMAL(20, 8) NOT NULL,
    price DECIMAL(20, 8),
    stop_price DECIMAL(20, 8),
    time_in_force VARCHAR(10) NOT NULL,
    status VARCHAR(20) NOT NULL,
    filled_quantity DECIMAL(20, 8) NOT NULL DEFAULT 0,
    average_price DECIMAL(20, 8),
    fees DECIMAL(20, 8),
    rejection_reason TEXT,
    execution_strategy VARCHAR(30) NOT NULL,
    urgency VARCHAR(10) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_broker FOREIGN KEY (broker_id) REFERENCES broker_connections(broker_id)
);

CREATE INDEX idx_broker_orders_order_id ON broker_orders(order_id);
CREATE INDEX idx_broker_orders_broker_id ON broker_orders(broker_id);
CREATE INDEX idx_broker_orders_account_id ON broker_orders(account_id);
CREATE INDEX idx_broker_orders_symbol ON broker_orders(symbol);
CREATE INDEX idx_broker_orders_status ON broker_orders(status);
CREATE INDEX idx_broker_orders_created_at ON broker_orders(created_at);
```

#### order_fills Table
```sql
CREATE TABLE order_fills (
    id VARCHAR(36) PRIMARY KEY,
    broker_order_id VARCHAR(36) NOT NULL,
    broker_fill_id VARCHAR(100) NOT NULL,
    quantity DECIMAL(20, 8) NOT NULL,
    price DECIMAL(20, 8) NOT NULL,
    fees DECIMAL(20, 8) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    venue VARCHAR(50) NOT NULL,
    
    CONSTRAINT fk_broker_order FOREIGN KEY (broker_order_id) REFERENCES broker_orders(id)
);

CREATE INDEX idx_order_fills_broker_order_id ON order_fills(broker_order_id);
CREATE INDEX idx_order_fills_timestamp ON order_fills(timestamp);
```

#### execution_reports Table
```sql
CREATE TABLE execution_reports (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL,
    broker_order_id VARCHAR(36) NOT NULL,
    broker_id VARCHAR(36) NOT NULL,
    symbol VARCHAR(20) NOT NULL,
    side VARCHAR(10) NOT NULL,
    quantity DECIMAL(20, 8) NOT NULL,
    filled_quantity DECIMAL(20, 8) NOT NULL,
    average_price DECIMAL(20, 8) NOT NULL,
    benchmark_price DECIMAL(20, 8),
    benchmark_type VARCHAR(30) NOT NULL,
    slippage_bps FLOAT,
    market_impact_bps FLOAT,
    total_cost_bps FLOAT,
    execution_start TIMESTAMP NOT NULL,
    execution_end TIMESTAMP NOT NULL,
    execution_duration_ms BIGINT NOT NULL,
    execution_quality_score FLOAT,
    broker_performance_rank INTEGER,
    recommendations JSONB,
    created_at TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_broker_order FOREIGN KEY (broker_order_id) REFERENCES broker_orders(id),
    CONSTRAINT fk_broker FOREIGN KEY (broker_id) REFERENCES broker_connections(broker_id)
);

CREATE INDEX idx_execution_reports_order_id ON execution_reports(order_id);
CREATE INDEX idx_execution_reports_broker_id ON execution_reports(broker_id);
CREATE INDEX idx_execution_reports_symbol ON execution_reports(symbol);
CREATE INDEX idx_execution_reports_execution_start ON execution_reports(execution_start);
```

### Read Model (Query Side)

#### broker_performance View
```sql
CREATE VIEW broker_performance AS
SELECT 
    b.broker_id,
    b.name,
    COUNT(e.id) as total_executions,
    AVG(e.execution_quality_score) as avg_quality_score,
    AVG(e.slippage_bps) as avg_slippage_bps,
    AVG(e.market_impact_bps) as avg_market_impact_bps,
    AVG(e.total_cost_bps) as avg_total_cost_bps,
    AVG(e.execution_duration_ms) as avg_execution_duration_ms,
    SUM(CASE WHEN bo.status = 'FILLED' THEN 1 ELSE 0 END) * 100.0 / COUNT(bo.id) as fill_rate_percent
FROM 
    broker_connections b
LEFT JOIN 
    broker_orders bo ON b.broker_id = bo.broker_id
LEFT JOIN 
    execution_reports e ON bo.id = e.broker_order_id
GROUP BY 
    b.broker_id, b.name;
```

#### execution_quality_by_asset_class Materialized View
```sql
CREATE MATERIALIZED VIEW execution_quality_by_asset_class AS
SELECT 
    b.broker_id,
    b.name,
    SUBSTRING(bo.symbol, 1, 3) as asset_class,
    COUNT(e.id) as total_executions,
    AVG(e.execution_quality_score) as avg_quality_score,
    AVG(e.slippage_bps) as avg_slippage_bps,
    AVG(e.market_impact_bps) as avg_market_impact_bps,
    AVG(e.total_cost_bps) as avg_total_cost_bps,
    AVG(e.execution_duration_ms) as avg_execution_duration_ms,
    SUM(CASE WHEN bo.status = 'FILLED' THEN 1 ELSE 0 END) * 100.0 / COUNT(bo.id) as fill_rate_percent
FROM 
    broker_connections b
JOIN 
    broker_orders bo ON b.broker_id = bo.broker_id
JOIN 
    execution_reports e ON bo.id = e.broker_order_id
GROUP BY 
    b.broker_id, b.name, SUBSTRING(bo.symbol, 1, 3);

CREATE UNIQUE INDEX idx_execution_quality_broker_asset ON execution_quality_by_asset_class(broker_id, asset_class);
```

#### broker_latency_stats Materialized View
```sql
CREATE MATERIALIZED VIEW broker_latency_stats AS
SELECT 
    b.broker_id,
    b.name,
    AVG(EXTRACT(EPOCH FROM (bo.updated_at - bo.created_at)) * 1000) as avg_order_processing_ms,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY EXTRACT(EPOCH FROM (bo.updated_at - bo.created_at)) * 1000) as p95_order_processing_ms,
    PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY EXTRACT(EPOCH FROM (bo.updated_at - bo.created_at)) * 1000) as p99_order_processing_ms,
    COUNT(bo.id) as total_orders,
    SUM(CASE WHEN bo.status = 'REJECTED' THEN 1 ELSE 0 END) as rejected_orders,
    SUM(CASE WHEN bo.status = 'REJECTED' THEN 1 ELSE 0 END) * 100.0 / COUNT(bo.id) as rejection_rate_percent
FROM 
    broker_connections b
JOIN 
    broker_orders bo ON b.broker_id = bo.broker_id
WHERE 
    bo.created_at > NOW() - INTERVAL '7 days'
GROUP BY 
    b.broker_id, b.name;

CREATE UNIQUE INDEX idx_broker_latency_stats_broker_id ON broker_latency_stats(broker_id);
```

## Project Plan

### Phase 1: Foundation (Weeks 1-2)

#### Week 1: Setup and Core Domain Model
- Set up project structure and build system
- Implement core domain model (BrokerConnection, BrokerOrder, etc.)
- Create database schema with migrations
- Implement basic repository layer
- Set up testing framework and write initial tests

#### Week 2: Broker Connectivity
- Implement broker adapter interface
- Create adapters for initial set of brokers (Interactive Brokers, Alpaca)
- Implement connection management and health checks
- Create broker selection algorithm
- Write integration tests for broker connectivity

### Phase 2: Core Functionality (Weeks 3-4)

#### Week 3: Order Routing and Execution
- Implement order routing logic
- Create execution monitoring system
- Implement fill processing
- Add order cancellation functionality
- Create execution report generation

#### Week 4: API Implementation
- Implement REST API endpoints
- Implement gRPC service definitions
- Create service layer with business logic
- Implement validation rules
- Write integration tests for API endpoints

### Phase 3: Advanced Features (Weeks 5-6)

#### Week 5: Transaction Cost Analysis
- Implement TCA calculation engine
- Create benchmark price collection
- Add execution quality scoring
- Implement broker performance ranking
- Create recommendation engine

#### Week 6: Smart Order Routing
- Implement advanced routing algorithms
- Add market impact modeling
- Create execution strategy optimization
- Implement adaptive routing based on market conditions
- Conduct performance testing and optimization

### Phase 4: Finalization (Weeks 7-8)

#### Week 7: Monitoring and Resilience
- Add comprehensive logging and metrics
- Implement circuit breakers for broker connections
- Create health check endpoints
- Add retry mechanisms for transient failures
- Implement rate limiting for broker APIs

#### Week 8: Documentation and Deployment
- Create comprehensive API documentation
- Write operational runbooks
- Prepare deployment configurations
- Conduct final testing and bug fixes
- Prepare for production deployment

### Milestones and Deliverables

1. **End of Week 2**: Basic broker connectivity implemented
2. **End of Week 4**: Complete order routing and execution with API
3. **End of Week 6**: Advanced TCA and smart routing implemented
4. **End of Week 8**: Production-ready service with documentation

### Risk Management

1. **Broker API Changes**: Implement adapter pattern for isolation from API changes
2. **Connectivity Issues**: Implement robust error handling and circuit breakers
3. **Performance Bottlenecks**: Regular performance testing with high-volume scenarios
4. **Data Consistency**: Implement transaction management and event sourcing
5. **Security Concerns**: Secure credential storage and encryption for broker APIs