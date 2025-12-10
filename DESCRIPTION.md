````md
# Sentinel – Technical Architecture & Implementation Spec

This document describes the **Sentinel** backend architecture and monorepo layout so that another AI (or developer) can implement a working baseline.

Goal:

> “Drop-in middleware for instant API analytics.”

- Monorepo managed with **Bun workspaces**
- Backend API with **Bun + Elysia**
- **Tinybird** for analytics storage (with a clear abstraction to swap to ClickHouse later)
- **Redis** for ingestion queue
- **Fully typed** TypeScript throughout
- SDKs for server frameworks to send batched events
- **Next.js + Clerk + shadcn** frontend will exist but is **out of scope for the implementation AI** (only structure stubbed here)

---

## 1. High-Level Architecture

**Data flow:**

1. Customer app uses **Sentinel SDK** (Node/Python/etc.) as middleware.
2. SDK intercepts each HTTP request/response and builds an **Event** object.
3. SDK batches events and sends them to **Sentinel Ingestion API** (`/v1/events/batch`).
4. The Ingestion API authenticates the project using an **Ingest API Key**, enqueues events to **Redis**.
5. A **Worker process** consumes from Redis, transforms/validates events, and writes them to **Tinybird**.
6. The **Dashboard Backend API** (same Elysia app or separate module) exposes summarized metrics using **Tinybird Pipes**.
7. The **Next.js frontend** (not implemented here) calls these backend endpoints.

---

## 2. Monorepo Structure (Bun Workspaces)

```txt
sentinel/
  packages/
    shared/
      types/                # Shared TypeScript types
      config/               # Env config helpers, constants, etc.

    api/                    # Bun + Elysia HTTP API + worker

    sdks/


    web/                    # Next.js app (frontend, no AI code generation yet)

  .env.example
  README.md                 # top-level; this file can be referenced or linked
```
````

---

## 3. Summary

This spec defines:

- Monorepo structure using **Bun workspaces**
- Elysia-based API service with:

  - Ingestion route
  - Redis queue
  - Tinybird ingestion worker
  - App DB schema for projects and API keys

- Fully-typed event schema and shared types
- Node SDK with batching and Express middleware
- Basic testing strategies
