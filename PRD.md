# Product Requirements Document (PRD)

## Project Title: AI-Driven Microservices E-Commerce Platform

**Author:** Tanimowo David (Systems Architect)  
**Date:** June 23, 2026  
**Status:** Approved / Ready for Implementation

---

## 1. Executive Summary & Overview

This document specifies the technical and product requirements for an event-driven, microservices-based e-commerce platform built using **FastAPI**, **Apache Kafka**, and **Docker**.

The core differentiation of this platform is the native integration of vector-based Artificial Intelligence (AI) directly into the shopping cycle. Unlike traditional systems that rely on strict relational constraints, this platform combines high-speed transactional commerce with real-time semantic **Smart Search**, an intelligent **Personalized Product Feed**, and a context-aware **AI Chat Concierge**.

To maximize operational speed and isolation, every microservice controls its own data layer, eliminating direct database coupling and synchronizing state entirely through asynchronous event streaming.

---

## 2. System Goals & Objectives

- **Microservices Excellence:** Demonstrate an absolute "Database-per-Service" pattern across four distinct microservices.
- **High Performance Authentication:** Ensure the User Service provides sub-15ms stateless JWT verification to avoid system-wide performance bottlenecks.
- **Semantic Intent Mapping:** Replace primitive SQL/substring keywords with vector space comparisons, ensuring unstructured user search phrases always return conceptually accurate matching items.
- **Contextual Agentic Experience:** Deploy a Retrieval-Augmented Generation (RAG) assistant capable of reading active user metadata to answer administrative, technical, and recommendation queries on the fly.

---

## 3. Microservices Domain Architectures & Boundaries

The application is strictly separated into four distributed modules running inside isolated Docker environments:

| Microservice                | Data Storage Layer        | Core Strategic Domain Responsibilities                                                                                                             |
| :-------------------------- | :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. User Service**         | PostgreSQL / SQLite       | High-speed registration, secure login hash matching, issuance of stateless JWT tokens, and user profile management.                                |
| **2. Catalog Service**      | MongoDB / PostgreSQL      | Standard source of truth for items. Allows structural updates exclusively from Verified Admin credentials. Emits catalog mutation states to Kafka. |
| **3. Order & Cart Service** | PostgreSQL                | Handles live cart sessions, mutations, and mock checkout state tracking.                                                                           |
| **4. AI Service**           | ChromaDB (Vector) + Redis | Vectors and indexes incoming items. Manages clickstream telemetry weights to dynamically construct personalized feeds. Exposes RAG chat hooks.     |

---

## 4. User Personas & Experience Flow

### 4.1 Logged-In (Authenticated) User

- Full access to product catalogs, carts, and checkout routines.
- The system Home Feed dynamically adapts to display personalized offerings based on their implicit behavior (clicks) and explicit indicators (wishlists/cart items).
- The AI Chat Concierge recognizes them by name, tracks their current session tokens, and delivers highly relevant, personalized answers.

### 4.2 Guest (Unauthenticated) User

- Can freely browse all items, utilize the Smart Search engine, and chat with the AI.
- The Home Feed displays a generic, default assortment without individualized relevance.
- The AI Assistant defaults to a friendly, generic posture, politely prompting them to create an account or log in to unlock personalized options.

### 4.3 System Administrator (Admin)

- Special user accounts flags verified explicitly inside the JWT claims.
- Possess exclusive write privileges to create, edit, or archive products within the platform inventory.
- Do not interact with customer-facing personalized feeds.

---

## 5. Detailed Functional Specifications

### 5.1 Core E-Commerce Mechanics

- **Admin-Only Inventory Controls:** Simple backend API endpoints exposed to pass explicit item structures (`Title`, `Description`, `SKU`, `Price`, `Stock`, `Category`). No merchant dashboard or third-party seller onboarding functionality will be built.
- **Cart Ledger:** Real-time transactional tracking of cart records. Persists item counts and references pricing instantly.
- **Mock Checkout Pipeline:** A streamlined workflow that reads active cart configurations, cross-checks catalog stock parameters, freezes values, and instantly routes the workflow to `Order_Status: Success` via internal payload mocks.

### 5.2 Semantic Smart Search System

- **Vector Processing Pipeline:** When an item is added or modified by an admin, the Catalog Service broadcasts a `PRODUCT_UPSERTED` event over Kafka. The AI Service intercepts this, reads the description strings, generates embeddings via a lightweight transformer engine, and stores them in ChromaDB.
- **Intent Mapping Evaluation:** When a user enters text into the search bar (e.g., typing _"hiking in rainy weather"_), the system converts the input into a temporary vector and performs a Cosine Similarity match against ChromaDB:
  $$\text{Similarity} = \frac{A \cdot B}{\|A\| \|B\|}$$
- **Zero Empty States:** Because search calculates vector proximities, the user is never faced with an empty "0 results found" error page. The system will always display the closest semantic matches available in inventory.

### 5.3 Predictive Intelligent Product Feed

- **Implicit Telemetry Capturing:** Every time a logged-in user views an item, a lightweight event is published asynchronously to track their interests.
- **Algorithmic Synthesis:** The AI Service continuously maps three specific criteria to compute item relevancy:
  1. Items actively saved inside their Wishlist.
  2. Items resting within their active Shopping Cart.
  3. Historical product IDs extracted from their clickstream logs.
- **Dynamic Landing Matrix:** The landing page combines traditional catalog indexes with localized vector proximity mappings, tailoring the displayed items to match the user's inferred intent.

### 5.4 Context-Aware AI Chat Concierge

An interface that accepts conversational text strings and executes smart RAG handling across three distinct scopes:

1.  **Operational Helpdesk:** Answers logistical questions regarding ordering rules, mock checkout workflows, or supported checkout methods using embedded system markdown files as context.
2.  **Conversational Discovery:** Converts natural language prompts (e.g., _"Find me a comfortable water-resistant jacket under \$100"_) into real-time queries against ChromaDB, returning structured recommendations.
3.  **Session-Aware Responses:** Extracts identity claims from incoming request headers to smoothly transition between customized personal support and guest assistance.

---

## 6. Critical System Constraints (Explicitly Out of Scope)

To maximize development velocity and prevent scope creep, **the following components are strictly out of scope**:

- **No Ratings or Review Modules:** No star points, textual reviews, or product upvoting interfaces.
- **No Refund Workflows:** All checkout operations are final. Order state parameters only move forward.
- **No Shipment Tracking Systems:** No integrations with external logistics APIs (e.g., FedEx/UPS) or shipment tracking maps. The system's responsibility ends upon successful mock order generation.

---

## 7. Kafka Event-Streaming Schema Specifications

### 7.1 Topic: `user.events.registration`

```json
{
  "event_id": "evt_998811",
  "timestamp": "2026-06-23T19:43:00Z",
  "event_type": "USER_REGISTERED",
  "payload": {
    "user_id": "usr_7736152",
    "email": "dev.trainee@example.com",
    "first_name": "Alex"
  }
}
```

### 7.2 Topic: `catalog.events.product`

```json
{
  "event_id": "evt_443322",
  "timestamp": "2026-06-23T19:43:05Z",
  "event_type": "PRODUCT_UPSERTED",
  "payload": {
    "product_id": "prod_88291",
    "title": "Waterproof Technical Trail Shell",
    "description": "Heavy duty windproof breathable jacket designed for torrential mountain storms.",
    "price": 149.99,
    "inventory_count": 45
  }
}
```

## 8. Technical Non-Functional Benchmarks

- **Latency:** The User Service must complete token evaluation cycles in < 15ms using optimized Python async/await patterns.

- **Resilience & Durability:** Kafka message partitions must be written directly to persistent Docker volumes. If the AI service drops or restarts during heavy processing, it will pick up exactly where its committed log offset left off without losing data.

- **Portability:** The entire network infrastructure—including the microservices, Kafka brokers, and databases—must launch using a single docker-compose up --build command.
