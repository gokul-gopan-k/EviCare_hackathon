# Clinical Copilot - Design Document

## 1. System Overview

Clinical Copilot is an AI-powered clinical decision-support system that helps healthcare providers make evidence-based treatment decisions by automatically analyzing patient data and matching it to relevant clinical guidelines. The system addresses the challenge of information overload in healthcare, where providers must synthesize complex patient records while staying current with thousands of clinical guidelines and medical literature.

The system ingests synthetic patient records containing diagnoses, laboratory results, medications, and vital signs. It uses Natural Language Processing (NLP) to extract and normalize clinical entities, then employs Retrieval-Augmented Generation (RAG) to semantically match patient conditions against a vector database of publicly available clinical guidelines from sources like the American Diabetes Association (ADA), World Health Organization (WHO), and Centers for Disease Control (CDC).

Using Large Language Models (LLMs) as a reasoning layer, Clinical Copilot generates evidence-backed recommendations with direct citations to source guidelines. Each recommendation includes a confidence score (0-100), detailed explanation of the reasoning process, and transparency into which patient data points influenced the decision. The system emphasizes responsible AI principles through comprehensive audit logging, bias mitigation strategies, and prominent disclaimers that position it as decision support rather than autonomous decision-making.

For the hackathon MVP, the system focuses on common chronic conditions (diabetes, hypertension, hyperlipidemia) using synthetic patient data and publicly available guidelines. The architecture is designed to be modular and scalable, with clear separation between data ingestion, RAG pipeline, LLM reasoning, and presentation layers.

## 2. High-Level Architecture


```
┌─────────────────────────────────────────────────────────────────────┐
│                          PRESENTATION LAYER                         │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  React Frontend (TypeScript + Tailwind CSS)               │    │
│  │  • Patient Input Forms  • Summary Display                 │    │
│  │  • Recommendation Viewer  • Explainability Dashboard      │    │
│  └────────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ REST API (HTTPS)
┌──────────────────────────────┴──────────────────────────────────────┐
│                          API GATEWAY LAYER                          │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  FastAPI Backend (Python 3.10+)                            │    │
│  │  • Request Validation  • Rate Limiting  • Error Handling   │    │
│  └────────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
┌──────────────────────────────┴──────────────────────────────────────┐
│                       CORE PROCESSING LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │
│  │   Patient    │  │  Clinical    │  │   Recommendation     │     │
│  │  Processor   │→ │  Extractor   │→ │     Generator        │     │
│  │              │  │              │  │                      │     │
│  │ • Validate   │  │ • Extract Dx │  │ • LLM Integration    │     │
│  │ • Normalize  │  │ • Extract    │  │ • Confidence Scoring │     │
│  │ • Summarize  │  │   Labs/Meds  │  │ • Citation Formatter │     │
│  └──────────────┘  └──────────────┘  └──────────────────────┘     │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
┌──────────────────────────────┴──────────────────────────────────────┐
│                          RAG PIPELINE LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │
│  │  Embedding   │  │    Vector    │  │    Retrieval         │     │
│  │   Service    │→ │   Database   │→ │     Ranker           │     │
│  │              │  │              │  │                      │     │
│  │ • Query      │  │ • ChromaDB/  │  │ • Semantic Search    │     │
│  │   Embedding  │  │   Pinecone   │  │ • Relevance Scoring  │     │
│  │ • Document   │  │ • Guideline  │  │ • Context Formation  │     │
│  │   Embedding  │  │   Vectors    │  │                      │     │
│  └──────────────┘  └──────────────┘  └──────────────────────┘     │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
┌──────────────────────────────┴──────────────────────────────────────┐
│                      EXTERNAL SERVICES LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │
│  │   LLM API    │  │  Guideline   │  │   Audit & Logging    │     │
│  │ (GPT-4 or    │  │   Sources    │  │                      │     │
│  │  Claude)     │  │ (ADA, WHO,   │  │ • PostgreSQL/SQLite  │     │
│  │              │  │  CDC, NIH)   │  │ • Structured Logs    │     │
│  └──────────────┘  └──────────────┘  └──────────────────────┘     │
└─────────────────────────────────────────────────────────────────────┘
```

The architecture follows a layered design pattern with clear separation of concerns. The Presentation Layer handles user interaction through a React-based web interface. The API Gateway Layer provides RESTful endpoints with FastAPI, handling authentication, rate limiting, and request validation. The Core Processing Layer contains the business logic for patient data processing, clinical entity extraction, and recommendation generation. The RAG Pipeline Layer manages semantic search over clinical guidelines using vector embeddings. Finally, the External Services Layer integrates with LLM APIs, guideline sources, and audit logging systems. This modular design enables independent scaling, testing, and updates of each component.

## 3. Data Flow


```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER INPUT                                                   │
│    Patient Record (JSON): Demographics, Diagnoses, Labs, Meds  │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. VALIDATION & NORMALIZATION                                   │
│    • Schema validation (Pydantic models)                        │
│    • ICD-10 code validation                                     │
│    • LOINC code normalization for labs                          │
│    • Data completeness checks                                   │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. CLINICAL EXTRACTION                                          │
│    • Extract diagnoses → ["Type 2 Diabetes", "Hypertension"]   │
│    • Extract labs → [{test: "HbA1c", value: 8.2, unit: "%"}]   │
│    • Extract medications → [{name: "Metformin", dose: "500mg"}] │
│    • Flag critical values (HbA1c > 7%, BP > 140/90)            │
│    • Generate patient summary (LLM-based)                       │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. QUERY CONSTRUCTION                                           │
│    Build semantic query from extracted data:                    │
│    "Type 2 Diabetes management HbA1c 8.2% Metformin therapy"   │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. EMBEDDING GENERATION                                         │
│    • Generate query embedding (768-1536 dimensions)             │
│    • Model: text-embedding-ada-002 or all-MiniLM-L6-v2         │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. VECTOR SEARCH                                                │
│    • Query ChromaDB with embedding                              │
│    • Retrieve top-k guidelines (k=5-10)                         │
│    • Return: guideline text + metadata + relevance scores       │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. RETRIEVAL RANKING                                            │
│    Re-rank results based on:                                    │
│    • Cosine similarity score (base)                             │
│    • Guideline recency (boost recent publications)              │
│    • Recommendation strength (Strong > Moderate > Weak)         │
│    • Evidence level (A > B > C)                                 │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. CONTEXT FORMATION                                            │
│    Format prompt for LLM:                                       │
│    • Patient summary                                            │
│    • Extracted clinical data                                    │
│    • Top-5 retrieved guidelines with metadata                   │
│    • Instructions for recommendation generation                 │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 9. LLM RECOMMENDATION GENERATION                                │
│    • Send context to GPT-4/Claude                               │
│    • Generate 2-4 evidence-based recommendations                │
│    • Include reasoning and citations                            │
│    • Assess recommendation strength                             │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 10. POST-PROCESSING                                             │
│     • Calculate confidence scores (multi-factor algorithm)      │
│     • Format citations with source links                        │
│     • Generate explainability data                              │
│     • Apply safety filters and disclaimers                      │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 11. AUDIT LOGGING                                               │
│     • Log request metadata                                      │
│     • Log retrieved guidelines and scores                       │
│     • Log generated recommendations                             │
│     • Store for compliance and debugging                        │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 12. RESPONSE DELIVERY                                           │
│     JSON Response: summary + recommendations + confidence +     │
│     explanations + citations                                    │
└─────────────────────────────────────────────────────────────────┘
```

## 4. RAG & AI Workflow


The RAG pipeline is the technical core of Clinical Copilot, bridging patient data with clinical knowledge through semantic search and LLM reasoning.

**Guideline Ingestion Pipeline:**
Clinical guidelines are sourced from publicly available repositories (ADA, WHO, CDC, NIH). Documents are parsed from PDF/HTML formats and preprocessed to extract clean text. The chunking strategy splits documents by logical sections (e.g., "Glycemic Targets", "Medication Management") with chunk sizes of 512-1024 tokens and 50-token overlap to preserve context. Each chunk is enriched with metadata including source organization, publication date, section title, recommendation strength (Strong/Moderate/Weak), and evidence level (A/B/C). Embeddings are generated using OpenAI's text-embedding-ada-002 (1536 dimensions) or the open-source all-MiniLM-L6-v2 model (384 dimensions). The resulting vectors and metadata are stored in ChromaDB for the MVP, with Pinecone as the production alternative for better scalability.

**Query Processing:**
When a patient record is processed, the system constructs a semantic query by combining primary diagnoses with relevant clinical context. For example, a patient with Type 2 Diabetes and HbA1c of 8.2% on Metformin generates the query: "Type 2 Diabetes management HbA1c 8.2% inadequate glycemic control Metformin monotherapy". This query is embedded using the same model used for guideline ingestion to ensure vector space compatibility.

**Semantic Retrieval:**
The query embedding is used to search the vector database using cosine similarity. The system retrieves the top-k most similar guideline chunks (typically k=5-10). Initial results are ranked purely by vector similarity, but a re-ranking algorithm applies additional heuristics: recent guidelines receive a recency boost (decay factor over 10 years), strong recommendations are boosted over weak ones, and high evidence levels (A > B > C) receive priority. This multi-factor ranking ensures the most relevant and authoritative guidelines surface to the top.

**Context Formation:**
The top-ranked guidelines are formatted into a structured context for the LLM. The prompt includes: (1) a concise patient summary, (2) extracted clinical data (diagnoses, labs, medications), (3) the retrieved guideline excerpts with full metadata, and (4) explicit instructions to generate evidence-based recommendations with citations. The prompt emphasizes grounding all recommendations in the provided guidelines to minimize hallucination risk.

**LLM Reasoning Layer:**
The formatted context is sent to GPT-4 or Claude with a temperature of 0.3 to balance creativity with consistency. The LLM generates 2-4 clinical recommendations, each with a clear title, detailed description, reasoning explanation, and citation to the source guideline. The model is instructed to assess recommendation strength (Strong/Moderate/Weak) and acknowledge uncertainty when evidence is limited or conflicting.

**Confidence Scoring Algorithm:**
A multi-factor algorithm calculates confidence scores (0-100) for each recommendation:
- Retrieval Quality (30 points): Average relevance score of cited guidelines
- Evidence Strength (25 points): Evidence level A=25, B=18, C=10
- Recommendation Strength (20 points): Strong=20, Moderate=12, Weak=5
- Guideline Recency (15 points): Decay 2 points per year since publication
- LLM Certainty (10 points): Penalize uncertainty markers ("may", "possibly")

This transparent scoring provides users with quantitative assessment of recommendation reliability.

**Explainability Generation:**
For each recommendation, the system generates structured explanations including: (1) patient factors that influenced the decision (e.g., "HbA1c 8.2% above target"), (2) guideline basis with relevance scores, and (3) a reasoning chain showing step-by-step logic. This explainability data is surfaced in the UI through expandable panels, confidence breakdowns, and citation links.

## 5. Database & Storage


The system uses a hybrid storage architecture combining vector and relational databases.

**Vector Database (ChromaDB/Pinecone):**
Stores clinical guideline embeddings with rich metadata. Each entry contains a 384 or 1536-dimensional vector, the original guideline text chunk, and metadata fields: source organization, publication date, section title, recommendation strength, evidence level, condition tags, and source URL. ChromaDB is used for the MVP due to its simplicity and local deployment capability. For production, Pinecone offers better performance, managed infrastructure, and horizontal scalability. The vector database supports semantic search with sub-second query latency for collections up to 100,000 guideline chunks.

**Relational Database (SQLite/PostgreSQL):**
Stores structured application data. The patient_requests table logs all incoming requests with patient_id, request_data (JSONB), timestamp, and processing_time_ms. The recommendations table stores generated recommendations with foreign key references to requests, including title, description, confidence_score, reasoning, and citations (JSONB). The audit_log table provides comprehensive tracking of all system events (retrieval, generation, feedback) with event_type, event_data (JSONB), and timestamp. The feedback table captures clinician feedback on recommendations for continuous improvement. SQLite is used for MVP simplicity, while PostgreSQL provides production-grade reliability, JSONB indexing, and connection pooling.

**Caching Layer:**
Redis is used in production to cache frequently accessed data: query embeddings (1-hour TTL), guideline retrieval results (30-minute TTL), and LLM responses for identical queries (24-hour TTL). This reduces API costs and improves response times for common queries.

## 6. Explainability & Responsible AI


Explainability and responsible AI are core design principles, not afterthoughts.

**Transparency Mechanisms:**
Every recommendation includes a detailed explanation showing: (1) which patient data points influenced the decision (e.g., "HbA1c 8.2% is 1.2% above target"), (2) which guidelines were retrieved and their relevance scores, (3) a step-by-step reasoning chain from patient data to recommendation, and (4) direct citations with links to source guidelines. The UI visualizes confidence scores with color-coded gauges (green >75, yellow 50-75, red <50) and provides expandable panels showing the confidence factor breakdown. Users can click citations to view the full guideline text, ensuring complete traceability.

**Confidence Scoring:**
The multi-factor confidence algorithm provides quantitative assessment of recommendation reliability. Each factor (retrieval quality, evidence strength, recommendation strength, guideline recency, LLM certainty) contributes a weighted score, and the breakdown is visible to users. Low-confidence recommendations (<50) trigger prominent warnings: "Low confidence - requires clinical validation". This prevents over-reliance on uncertain recommendations.

**Safety Filters:**
All recommendations pass through safety filters before delivery. High-risk keywords (surgery, discontinue, emergency) trigger mandatory review flags. Recommendations without guideline citations are flagged with warnings. Low-confidence recommendations receive prominent disclaimers. Every recommendation includes a standard disclaimer: "This is AI-generated decision support. All recommendations must be validated by a qualified healthcare provider."

**Bias Mitigation:**
The system addresses bias through diverse guideline sources (ADA, WHO, CDC, NIH, specialty societies) to avoid single-source bias. Demographic awareness flags when guidelines may not apply to specific populations (e.g., pediatric vs. adult guidelines). The audit log tracks recommendation patterns across demographic groups to enable post-hoc bias analysis. Future versions will implement fairness metrics to quantify and mitigate demographic disparities.

**Audit Trail:**
Comprehensive logging captures every system action: input data (anonymized), retrieved guidelines with relevance scores, generated recommendations, confidence scores, and user feedback. Logs are structured (JSON) and timestamped for compliance and debugging. The audit trail enables retrospective analysis of system behavior, identification of failure modes, and continuous improvement through feedback loops.

**Human-in-the-Loop Design:**
The system is explicitly positioned as decision support, not autonomous decision-making. The UI emphasizes that recommendations require clinical validation. Feedback mechanisms allow clinicians to rate recommendations (helpful/not helpful/incorrect) and provide comments. This feedback is logged for future model improvement and guideline curation. The system never claims to replace clinical judgment, only to augment it with evidence-based insights.

**Limitations Disclosure:**
The system prominently discloses limitations: not FDA approved, for demonstration purposes only, trained on synthetic data, limited to guideline-based care, cannot handle cases requiring clinical experience beyond guidelines, and may not capture nuanced patient-specific factors. These limitations are displayed in the UI and included in all documentation.

## 7. Deployment & Scalability


**MVP Deployment (Hackathon):**
The MVP runs locally with Docker Compose orchestrating three containers: React frontend (port 3000), FastAPI backend (port 8000), and ChromaDB (port 8001). SQLite provides local relational storage. Environment variables store API keys for OpenAI. The entire stack can be launched with `docker-compose up` for demo purposes. This setup requires no cloud infrastructure and runs on a standard laptop.

**Production Deployment:**
Production deployment uses cloud infrastructure (AWS/Azure/GCP) with the following architecture: Frontend is built as static assets and served via CDN (CloudFront/Azure CDN) for global low-latency access. Backend API runs on containerized instances (ECS/AKS/Cloud Run) behind an Application Load Balancer with auto-scaling based on CPU and request rate. Pinecone provides managed vector database with automatic scaling. PostgreSQL runs as a managed service (RDS/Azure Database/Cloud SQL) with read replicas for query scaling. Redis provides distributed caching. All services communicate over private networks with TLS encryption.

**Scalability Strategies:**
The stateless API design enables horizontal scaling by adding backend instances. Database connection pooling (SQLAlchemy) efficiently manages PostgreSQL connections. Caching (Redis) reduces LLM API calls and vector searches for common queries. Asynchronous processing (FastAPI async endpoints) handles concurrent requests efficiently. Rate limiting (10 requests/minute per IP) prevents abuse. Batch processing endpoints allow processing multiple patients in parallel using asyncio.gather(). Vector database sharding (Pinecone namespaces) enables partitioning guidelines by specialty or condition for faster retrieval.

**Monitoring:**
Production monitoring tracks: request latency (p50, p95, p99), throughput (requests/second), error rates (4xx, 5xx), LLM API costs (token usage), vector DB query performance, and confidence score distribution. Alerts trigger on high error rates, elevated latency, or API cost spikes. Structured logging (JSON) enables log aggregation and analysis via ELK stack or CloudWatch.

**Cost Optimization:**
LLM API costs are the primary expense. Caching reduces redundant API calls. Prompt optimization minimizes token usage. Batch processing amortizes fixed costs. Vector database costs scale with collection size; guideline pruning removes outdated content. Serverless deployment (Cloud Run, Lambda) reduces idle costs for low-traffic periods.

## 8. Limitations & Future Work


**Current Limitations:**
LLM hallucination risk remains despite RAG grounding; all recommendations require human validation. Guideline coverage is limited to publicly available sources and may not include latest research or specialty guidelines. Context window limits (8k-128k tokens) constrain the number of guidelines that can be included in prompts. Latency of 5-10 seconds per patient may be too slow for real-time clinical workflows. The system is not FDA approved and cannot be used for actual clinical decision-making. Synthetic data training may not reflect real-world complexity and edge cases.

**Future Enhancements:**
Short-term improvements include expanding guideline sources (specialty societies, international guidelines), implementing user authentication and session management, and adding support for more clinical conditions. Medium-term enhancements include multi-modal support (lab trends, vital sign graphs, medical images), FHIR integration for EHR connectivity, real-time guideline updates with versioning, and advanced explainability with attention visualization. Long-term vision includes fine-tuned medical LLMs for better clinical reasoning, active learning from clinician feedback, predictive analytics for patient outcomes, multi-language support for global deployment, and mobile applications for point-of-care use.


