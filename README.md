# Marketing Automation System — Use-Case Flow Views

## Homepage Recommendations for a Returning User

**Use Case:** Personalized Homepage Recommendations  
**Inspired by:** Amazon / Noon  
**Architecture:** Real-time Vector Retrieval  
**Latency Target:** P95 < 100 ms

---

## 1. Business Scenario

An e-commerce platform wants to provide personalized product recommendations whenever a returning customer visits its homepage.

The recommendation system uses the customer's historical interactions, purchase history, browsing behavior, and product preferences to retrieve relevant products dynamically.

For example, if a customer frequently purchases electronics and recently viewed headphones, the platform should recommend relevant headphones, accessories, and other electronic products.

The system must satisfy the following requirements:

- Generate personalized recommendations for returning customers.
- Retrieve relevant products dynamically using vector similarity search.
- Rank candidate products according to customer preferences.
- Maintain a P95 response latency below 100 milliseconds.
- Avoid executing expensive analytical queries against the operational database during homepage requests.
- Continuously update customer features and product embeddings using background processing.

## 2. Actors and Systems

| Component | Responsibility |
|---|---|
| Customer / Web Channel | Requests the personalized homepage. |
| API Gateway | Receives requests and forwards them to the appropriate service. |
| E-commerce Service | Handles homepage requests and integrates recommendation results. |
| Operational Database | Stores transactional data, customer interactions, orders, and product information. |
| CDC | Captures database changes. |
| Kafka | Transports customer and product events. |
| Flink | Processes events and updates near-real-time customer features. |
| Airflow | Schedules and orchestrates batch processing workflows. |
| CDP | Consolidates customer profiles and historical behavioral information. |
| Feature Store | Stores customer features for online retrieval. |
| Vector DB | Stores product embeddings and performs similarity searches. |
| Recommendation Engine | Retrieves candidate products, scores them, and returns ranked recommendations. |
| WhatsApp | Not required for this use case. |

## 3. Real-Time Request Path

When a returning customer opens the homepage:

1. The customer sends a homepage request through the web channel.
2. The API Gateway receives the request and forwards it to the E-commerce Service.
3. The E-commerce Service sends a recommendation request containing the customer identifier.
4. The Recommendation Engine retrieves customer features from the Feature Store.
5. The engine uses customer preferences and behavioral features to construct a query vector.
6. The Recommendation Engine queries the Vector DB using approximate nearest-neighbor search.
7. The Vector DB returns relevant product candidates.
8. The Recommendation Engine scores and ranks the candidates using customer features and relevance signals.
9. The E-commerce Service prepares the personalized homepage response.
10. The API Gateway returns the response to the customer.

### Real-Time Optimization

The architecture uses dynamic vector retrieval to discover relevant products during the customer's homepage request.

To satisfy the latency requirement:

- Use approximate nearest-neighbor indexing.
- Limit the number of retrieved candidates.
- Apply lightweight ranking.
- Retrieve customer features from the online Feature Store.
- Avoid analytical queries against the Operational Database.
- Keep the online request path independent of batch processing.

## 4. Background Ingestion and Batch Path

The background path continuously prepares the data required by the real-time recommendation engine.

### Event Processing

1. Customers interact with the e-commerce platform through browsing, cart additions, and purchases.
2. Interactions are persisted in the Operational Database.
3. CDC captures relevant database changes.
4. Kafka transports the captured events.
5. Flink processes events and calculates updated customer behavioral features.
6. Updated features are synchronized with the Feature Store.
7. Customer events and historical information are consolidated in the CDP.

### Batch Processing

Airflow orchestrates scheduled workflows:

1. Extract historical customer and product information.
2. Refresh customer profiles and segmentation in the CDP.
3. Generate or update product embeddings.
4. Update the Vector DB index.
5. Ensure the recommendation engine has access to fresh product representations.

This separation allows the system to process customer activity continuously while performing expensive historical processing asynchronously.

## 5. Proposed P95 Latency Budget

The target is below 100 milliseconds for the complete homepage recommendation request.

| Processing stage | Budget |
|---|---:|
| API Gateway | 8 ms |
| E-commerce Service request handling | 8 ms |
| Feature Store lookup | 8 ms |
| Vector DB similarity search | 20 ms |
| Recommendation scoring and ranking | 25 ms |
| Response preparation | 11 ms |
| Network and serialization overhead | 10 ms |
| **Total proposed budget** | **90 ms** |

The remaining 10 ms provides a safety margin.

These are proposed engineering allocations, not measured benchmark results. P95 must be validated under realistic concurrent traffic.

## 6. Over-Engineering Check

The architecture follows the provided high-level system design.

### Required Components

- Channels and API Gateway.
- E-commerce Service and Operational Database.
- CDC, Kafka, and Flink.
- Airflow and CDP.
- Feature Store and Vector DB.
- Recommendation Engine.

### Unnecessary Components

**WhatsApp:** Not required because the selected use case involves homepage recommendations rather than outbound notifications.

### Missing Components

**Authentication / Customer Identity Resolution**

The system needs a reliable way to identify the returning customer and associate the request with the correct customer profile.

This responsibility can be handled by the existing API Gateway and E-commerce Service. A separate identity microservice is not required for this project.

**Product Availability Validation**

The system should avoid recommending unavailable products. This can be handled by the E-commerce Service using its existing product information.

No additional microservices, message brokers, or databases are introduced beyond the high-level architecture.

## 7. Conclusion

The proposed architecture provides a personalized homepage recommendation solution using real-time vector retrieval and background data processing.

CDC, Kafka, and Flink keep customer behavioral features updated, while Airflow orchestrates historical processing and product embedding refreshes.

During homepage requests, the Recommendation Engine retrieves customer features, queries the Vector DB dynamically, and ranks relevant products.

This design separates online serving from background processing, minimizes operational database load, and targets a P95 latency below 100 milliseconds.

The final architecture balances dynamic personalization, scalability, and operational efficiency while remaining aligned with the provided high-level system design.