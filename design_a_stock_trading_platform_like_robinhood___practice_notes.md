# Design a Stock Trading Platform Like Robinhood - Practice Notes


![alt text](https://github.com/akash-shindhe86/next-level/blob/master/robinhood.png)

### Design a Stock Trading Platform Like Robinhood Guided Practice - November 29, 2025

<!-- EXCALIDRAW_PRACTICE practiceId="cmil1s1e101ty08ad9c1jny75" -->

#### Requirements

* Trading systems have two distinct latency requirements: order processing latency (how fast trades execute) and price update latency (how fast users see market data). Both need to be in the low 100s of milliseconds - traders need real-time price feeds to make informed decisions, otherwise they're trading on stale data.

* When discussing consistency in trading systems, be specific about *where* it's critical. Consistency is most important for order management (ensuring users see accurate order status and account balances) rather than for all data uniformly - this prevents issues like double-spending or incorrect position tracking.

* Non-functional requirements should include quantified numbers (e.g., '100ms latency' not 'low latency') to make them actionable for engineering teams. Vague requirements lead to misaligned implementations and make it impossible to validate if the system meets its goals.

#### Core Entities

#### API

* RESTful APIs for trading systems need a collection endpoint (GET /orders) in addition to single-resource endpoints (GET /orders/{orderId}). Users need to view their entire order history to manage their portfolio, not just look up individual orders by ID.

* Order retrieval endpoints should return complete order details including symbol, quantity, order type (market/limit), status, timestamp, and execution prices. Returning only 'bought price' and 'new price' is insufficient for users to track their trading activity.

* Pagination is essential for endpoints that return collections of user data like order history. Without pagination, the response payload can become too large as users accumulate orders over time, causing performance issues and poor user experience.

#### High Level Design

* For real-time price data, maintain a persistent connection (like WebSocket) to the exchange rather than making API calls per user request. This avoids rate limits and reduces latency, since exchanges typically limit requests per IP and concurrent connections.

* Orders must be persisted in your own database before sending to the exchange. This allows you to track order status, handle failures gracefully, maintain order history without repeatedly querying the exchange, and recover from system crashes without losing order data.

* Use a cache (like Redis) to store frequently accessed data like stock prices that are updated via a background processor. This separates the concern of fetching data from the exchange (Symbol Processor) from serving user requests (Symbol Service), allowing you to serve many concurrent user requests without hitting the exchange API.

* Server-Sent Events (SSE) is appropriate for pushing real-time price updates from server to client in a stock trading app, as it maintains a persistent HTTP connection for one-way server-to-client streaming of data updates.

#### Deep Dives

* Server-Sent Events (SSE) require sticky sessions at the load balancer level to maintain long-lived HTTP connections between clients and the same server instance. Without sticky sessions, clients would lose their connection stream when routed to different servers.

* When using Redis Pub/Sub to distribute updates across service instances, partition user subscriptions intelligently to avoid hotspots where one server handles all subscriptions for popular stocks while others sit idle. Balance load by routing users to servers already subscribed to their symbols or with lower load.

* Database partitioning by userId ensures all orders for a user live on the same partition, enabling transactional consistency when querying order states. This prevents scenarios where concurrent updates could show users inconsistent order information during reads.

* Order management systems need explicit intermediate states (like 'pending', 'submitted', 'partially_filled') to reliably track order lifecycle, handle failures gracefully, and provide accurate status queries. Without defined states, you can't distinguish between orders awaiting submission versus those stuck in error conditions.

​
