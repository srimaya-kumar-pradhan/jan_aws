# design.md

## 1. High-Level Architecture Overview

JanSetu AI utilizes a **Hybrid Edge-Cloud Architecture** to balance the heavy compute requirements of LLMs with the low-latency, offline-first needs of rural users.

- **Client Layer**: A customized Progressive Web App (PWA) that caches UI shells and critical data locally. Serves as the primary touchpoint.
- **Edge Layer**: Content Delivery Network (CDN) for static assets and a lightweight "SMS Gateway" for text-based fallback.
- **API Gateway**: The secure entry point managing authentication, rate limiting, and request routing.
- **Intelligence Layer (Cloud)**: Hosts the heavy-lifting components—LLM Inference, Vector Search, and Orchestration.
- **Data Layer**: Polyglot persistence with Relational, Vector, and Document databases.

---

## 2. Component-Level Breakdown

### 2.1 Front-End (The "Setu" Client)
- **Framework**: Next.js (React) optimized for static generation (SSG).
- **State Management**: Redux Toolkit with `redux-persist` for offline state retention.
- **Voice Module**: Web Speech API for basic ASR (browser-native) fallback to AI4Bharat API for higher accuracy in dialects.

### 2.2 Backend Services
- **Orchestrator Service**: Python (FastAPI). Manages the dialogue flow, calls disparate AI services, and aggregates results.
- **Ingestion Service**: Evaluates and scrapes government portals; creates embeddings.
- **Notification Service**: Manages WhatsApp API (Meta), SMS (Twilio/Gov Gateway), and Push Notifications.

### 2.3 AI Services
- **Translation Engine**:  Translation/Transliteration layer (e.g., Bhashini / NLLB).
- **Retrieval Engine**: Vector database for semantic search of schemes.
- **Reasoning Engine**: Fine-tuned LLM (e.g., Llama-3-8B-Indic or similar) for synthesis and eligibility checking.

---

## 3. Detailed Data Flow

1.  **User Input**: User speaks a query/need in Bhojpuri: *"Hamara khet me paani na aawata, kono madad mili?"* (No water in my field, any help?)
2.  **Voice Processing**: Audio blob sent to **ASR Service**. Transcribed to Bhojpuri text -> Translated to English (Pivot Language).
    *   *Result*: "Water supply issue in farm, is there any subsidiary/help?"
3.  **Intent Classification**: Orchestrator determines intent: `SCHEME_DISCOVERY` + `DOMAIN:AGRICULTURE`.
4.  **Retrieval (RAG)**:
    *   Query Vector DB with embeddings of the English query.
    *   Retrieve top 5 relevant chunks (e.g., "Pradhan Mantri Krishi Sinchayee Yojana").
5.  **Synthesis**:
    *   LLM constructs a response using retrieved chunks.
    *   *Instruction*: "Summarize benefits and eligibility in simple terms."
6.  **Response Delivery**:
    *   English response translated back to Bhojpuri.
    *   TTS Service generates audio.
    *   Response sent to client with "Confidence Score" and metadata.

---

## 4. AI Processing Pipeline

### 4.1 Ingestion Pipeline
`Gov Portal/PDF` -> `Unstructured.io Parser` -> `Semantic Chunker` -> `Embedding Model` -> `Vector DB (Milvus)`

### 4.2 Query Pipeline
`Query` -> `HyDE (Hypothetical Document Embeddings)` -> `Hybrid Search (Keyword + Vector)` -> `Re-ranking (Cross-Encoder)` -> `Context Window Stuffing` -> `LLM Generation`

### 4.3 Innovation: Impact Prediction Engine
A specialized regression model trained on scheme payout data estimates potential benefits:
- *Input*: [User: Farmer, Land: 2 Acres, State: UP]
- *Logic*: Matches against historic disbursement data for PM-KISAN.
- *Output*: "Potential Benefit: ₹6,000 / year."

---

## 5. Eligibility Matching Engine Architecture

To avoid the non-deterministic nature of LLMs for strict rules, we use a **Hybrid Rule-Neuro Approach**:

1.  **Rule Extraction**: LLM extracts structured rules from scheme PDFs into a JSON Logic format during ingestion.
    *   *Example*: `{"and": [{"var": "age"} > 18, {"var": "income"} < 200000]}`
2.  **Deterministic Evaluation**: When a user applies, their profile JSON is run against this Logic JSON using a standard rules engine (e.g., *json-logic-py*).
3.  **Soft Matching**: If the rule fails (e.g., income depends on "household definition"), the LLM is invoked to interpret the ambiguity based on user context.

---

## 6. Multilingual & Voice System Design

- **Architecture**: ASR -> Machine Translation (MT) -> Core Logic -> MT -> TTS.
- **Dialect Handling**: Utilizing "Bhashini" or similar open models fine-tuned on dialect pairs.
- **Voice Activity Detection (VAD)**: Implementation of strict client-side VAD to stop recording when the user pauses, saving bandwidth.
- **Streaming**: Audio responses are streamed (chunked transfer encoding) to play immediately, reducing perceived latency.

---

## 7. Offline / Low-Bandwidth Optimization Strategy

- **Service Workers**: Aggressive caching of the App Shell, CSS, and Fonts.
- **"Just-in-Time" Assets**: Images are loaded only when in viewport (Lazy Load). High-res images for banners are replaced with CSS gradients on 2G.
- **Innovation: SMS Query Protocol**:
    *   User sends SMS to Shortcode `55055`.
    *   Body: `SCHEME <Space> <Keyword>`.
    *   Server parses SMS, runs a lightweight search, and replies with a text-only summary (160 chars).
- **Data Compression**: All API JSON responses are Brotli compressed.

---

## 8. Database Schema Strategy

### PostgreSQL (Primary Relational DB)
- `Users`: ID, Phone_Hash, Demographics, Preferences (Language).
- `Schemes`: ID, Title, Ministry, Valid_From, Valid_To, Benefits_Structured_Data.
- `Applications`: ID, User_ID, Scheme_ID, Status, Timeline.

### Vector Database (Milvus/Qdrant)
- `Scheme_Embeddings`: Vector_768d, Chunk_Text, Source_URL, Metadata (Filters: State, Category).

### Redis (Cache)
- `Session_Store`: User conversation history (TTL 1 hour).
- `Hot_Schemes`: Top 10 most accessed schemes in the district.

---

## 9. Security Architecture

- **Auth**: Passwordless (Mobile OTP). OAuth2 standard.
- **Encryption**:
    - **Field-Level Encryption**: Sensitive fields (Aadhaar Number, Income) are encrypted using a specific key in the DB.
    - **Ephemeral Storage**: Uploaded documents for verification are processed in memory and deleted immediately; only the boolean validation result is stored.
- **Rate Limiting**: "Leaky Bucket" algorithm on the API Gateway to prevent abuse, with higher limits for trusted Partners (CSCs).

---

## 10. Scalability & Performance Strategy

- **Horizontal Scaling**: The Stateless Orchestrator and API Gateway scale automatically on Kubernetes (HPA) based on CPU/Memory usage.
- **Read Replicas**: PostgreSQL Read Replicas for the heavy "Scheme Feed" read operations.
- **Asynchronous Processing**: "Application Generation" and "Document Validation" tasks are offloaded to a message queue (RabbitMQ/Celery) to keep the HTTP response fast.

---

## 11. DevOps & Deployment Architecture

- **Containerization**: Docker for all services.
- **Orchestration**: Kubernetes (EKS or GKE).
- **CI/CD**: GitHub Actions.
    - *Pipeline*: Lint -> Unit Test -> Build Container -> Scan Vulnerabilities -> Deploy to Staging -> Integration Test -> Deploy to Prod.
- **Infrastructure as Code (IaC)**: Terraform to provision VPCs, Load Balancers, and Database clusters.

---

## 12. Monitoring & Observability Plan

- **APM**: OpenTelemetry instrumentation to trace a request (Voice Input -> Response) across all microservices.
- **Logs**: Centralized logging (ELK Stack: Elasticsearch, Logstash, Kibana).
- **Dashboard**: Grafana dashboard visualizing:
    - *Business Metrics*: Schemes Discovered, Applications Started.
    - *System Metrics*: ASR Latency, LLM Token/Sec, API Error Rates.
- **Alerting**: PagerDuty alerts for >1% Error Rate or Latency spikes >5s.

---

## 13. Failure Handling & Fallback Mechanisms

- **ASR Failure**: If Voice Input API fails (timeout/error), UI auto-prompts to "Type your query" or "Select from Options."
- **LLM Degradation**: If the primary LLM is slow/down, the system falls back to a smaller, faster model (e.g., 8B param -> 3B param) or a purely Keyword-based search.
- **Network Drop**: Re-try logic with exponential backoff for API calls. Use `BackgroundSync` API to submit forms when the internet returns.
