# Anomalous Collectibles — AI-Powered Microservices E-Commerce Platform

Welcome to **Anomalous Collectibles**, an advanced, event-driven e-commerce platform specializing in rare action figures, comic statues, and multiverse collectibles.

This repository implements a production-grade **Database-per-Service** microservices architecture using **FastAPI**, **Apache Kafka**, and **Docker Compose**. It showcases how traditional checkout pipelines can be combined with modern **AI Vector Search (ChromaDB)** and dynamic user behavioral tracking to build a personalized storefront.

---

## 1. System Architecture & Component Mapping

The application splits its responsibilities across 4 isolated microservices. Databases are completely decoupled to prevent single points of failure and maximize throughput.

```text
   ┌──────────────────┐      ┌─────────────────────┐      ┌────────────────────┐
   │   User Service   │      │   Catalog Service   │      │Cart & Order Service│
   │ (PostgreSQL/JWT) │      │  (MongoDB/Admins)   │      │    (PostgreSQL)    │
   └────────┬─────────┘      └──────────┬──────────┘      └──────────┬─────────┘
            │                           │                            │
            │     Asynchronous Events   │                            │
            └─────────────────► ┌───────▼───────┐ ◄──────────────────┘
                                │ APACHE KAFKA  │
                                └───────┬───────┘
                                        │
                                        ▼
                             ┌────────────────────┐
                             │     AI Service     │
                             │ (ChromaDB Vector)  │
                             └────────────────────┘
```

### Domain Breakdowns

1. **User Service (Port 8001):** Manages user account states, secure registration workflows, and signs stateless, high-speed **JWT authentication tokens** to establish session identities (Guest vs. Logged-In User).
2. **Catalog Service (Port 8002):** The canonical database for stock items. Restricts inventory write privileges exclusively to verified **Admin accounts** (e.g., adding a rare "Dark Knight Premium Statuette").
3. **Cart & Order Service (Port 8003):** Tracks state modifications for real-time shopping cart additions and handles a fast, deterministic mock checkout processing layout.
4. **AI Service (Port 8004):** Consumes product modifications from Kafka to maintain dense text embeddings via **ChromaDB**. It hosts semantic smart search, parses conversational chat intents, and evaluates real-time clickstreams to re-weight user landing pages.

---

## 2. Project Directory Blueprint

```text
anomalous-collectibles/
├── docker-compose.yml # Orchestrates the Kafka brokers, DBs, and application containers
├── README.md # System Documentation & Blueprint (This file)
├── PRD.md # Core Product & Functional Requirements Matrix
├── user_service/
│ ├── app/
│ │ ├── main.py # FastAPI entry point & Authentication Routes
│ │ ├── models.py # SQLAlchemy Models (User records)
│ │ └── database.py # Database engine setup
│ └── Dockerfile
├── catalog_service/
│ ├── app/
│ │ ├── main.py # Inventory management endpoints
│ │ └── kafka_producer.py # Dispatches inventory state shifts to Kafka
│ └── Dockerfile
├── order_service/
│ ├── app/
│ │ └── main.py # Cart allocations & mock checkouts
│ └── Dockerfile
└── ai_service/
├── app/
│ ├── main.py # RAG Chat, Search, & Personalization Engines
│ ├── vector_core.py # ChromaDB interactions and embedding calculations
│ └── kafka_consumer.py # Syncs catalog updates into vector spaces asynchronously
└── Dockerfile
```

---

## 3. Thematic Test Cases (Proving the AI Vector Logic)

Because action figures carry rich descriptive lore, you can verify your semantic smart search using descriptive search queries rather than strict character names:

- **Query Input:** `"something for hiking in rainy weather"`
  - **AI Output Recommendation:** _Retro Space Bounty Hunter_ (Matches descriptions of weathered armor and jetpacks).
- **Query Input:** `"purple alien with a golden glove"`
  - **AI Output Recommendation:** _Intergalactic Titan_ (Successfully matches lore attributes without the word "Thanos" being in the title).
- **Query Input:** `"dark vigilante standing on a roof"`
  - **AI Output Recommendation:** _The Dark Knight Premium Statuette_ (Resolves semantic proximity to Gotham descriptions).

---

## 4. Local Quick Start Guide

### Prerequisites

- Docker Desktop installed locally.
- Python 3.10+ (for isolated local script execution if running outside containers).

### Step 1: Fire Up the Infrastructure Cluster

From the root directory of the project, spin up your microservices along with Apache Kafka and your database engines:

```bash
docker-compose up --build -d
The -d flag runs the containers in detached mode, keeping your terminal clear.
```

### Step 2: Validate API Interconnectivity

Each microservice spins up its own automated OpenAPI swagger UI. You can verify routing and trigger requests using your browser:

- **User Endpoint Control:** http://localhost:8001/docs
- **Catalog Inventory Engine:** http://localhost:8002/docs
- **Cart Processing Engine:** http://localhost:8003/docs
- **AI Engine & Vector Hub:** http://localhost:8004/docs

## 5. Async Event Broker Design Flow

Data integrity across separate services is managed completely through event streams via Kafka topics.

### Data Flow Example: Admin Adds an Action Figure

1. An Admin posts a new action figure JSON body to the **Catalog Service**.
2. The Catalog Service records it in its database and dispatches a message payload containing the item's details to the `catalog.events.product` Kafka topic.
3. The **AI Service** listens to this topic, reads the text fields, generates vector representations, and appends the new embedding node to **ChromaDB**.
4. The item is immediately discoverable via semantic search queries without requiring manual database migrations or sync scripts.

## 6. Technical Benchmarks & Guidelines

- **Stateless Tokens:** Avoid query overheads inside authorization loops. Ensure standard route verifications check cryptographic JWT signatures on the fly.
- **Non-Blocking IO:** Use FastAPI's native `async def` handling patterns for operations involving Kafka event dispatches or database reads.
- **Message Durability:** If the AI service is paused or undergoing an update, Kafka retains message offsets safely within its log queues. Upon coming back online, the service will process the backlogged catalog events without missing a beat.
