# Cloud-Based Distributed Task Scheduler (MCA College Project)

A high-visibility distributed task scheduling platform engineered for MCA final evaluation (20 marks) that demonstrates scheduled job dispatching, RabbitMQ priority queues, worker concurrency, and in-memory token bucket rate limiting with real-time UI telemetry and an exportable Spring Boot 3.2 Java 17 production codebase.

### User Review & Critical Decisions

> [!IMPORTANT]
> The project runs as an end-to-end interactive system in the AI Studio environment with an active backend engine powering real REST endpoints (`/api/tasks`, `/api/events`, `/api/stats`, `/api/tasks/rate-limit-status`), while providing the complete, downloadable Spring Boot 3.2 / Java 17 / MySQL / RabbitMQ Maven project for college viva submission.

- **Confirmed Delivery Model**: Live full-stack execution engine in Express/Node matching the exact Spring Boot REST contracts and behavior, paired with an integrated Spring Boot Project Inspector & ZIP/code exporter containing the exact Java 17 classes (`@Entity`, `@RabbitListener`, `@Scheduled`, TokenBucket, JUnit tests).
- **Confirmed Viva Presets**: One-click viva demonstration scenarios included in the top bar (e.g., "Burst 6 Tasks (429 Rate Limit Demo)", "Schedule Task in 15s", "Concurrent 3-Worker Stress", "Trigger Failure Recovery").
- **Confirmed Visual Aesthetic**: Node.js official website theme with deep charcoal canvas (`#0d1117`), dark hex-grid motif, emerald green accents (`#22c55e` / `#10b981`), tabbed developer code & event viewers, and high-contrast tabular telemetry.

---

### 1. Overview & Core Concept

- **What It Does**: Simulates and operates a cloud-distributed task execution system where tasks with different types (`EMAIL`, `HTTP_CALL`, `COMPUTE`) and priorities (`HIGH`, `MEDIUM`, `LOW`) are scheduled for future or immediate execution. A central scheduler discovers due tasks every 5 seconds, dispatches them through a prioritized queue, and dispatches them across a pool of 3 concurrent workers. An in-memory token bucket limits submissions to 5 tokens with a 12-second refill interval.
- **Target Audience / Persona**: College examiners, MCA students, and evaluators reviewing distributed systems concepts without needing an external terminal, MySQL Workbench, or RabbitMQ Management UI.
- **Key Value**: 100% visible proof of work on-screen. Every scheduler tick, RabbitMQ message publication, worker thread sleep, status transition (`PENDING` → `RUNNING` → `COMPLETED` / `FAILED`), and rate-limiting decision is streamed into an active Live Events panel and reflected across interactive dashboard metrics.

---

### 2. User Experience & Visual Design

- **Layout Structure**:
  - **Top Navigation Bar**: Brand identity (`DistriSched // Cloud Task Engine`), live Token Bucket meter (`Tokens: 4/5` with refill countdown bar), Queue Depth counter, and one-click **Viva Demo Presets** dropdown.
  - **Main Working Area (2-Column Grid on Desktop, Stacked on Mobile/Tablet)**:
    - **Left Column (Tabs & Workspaces)**:
      - **Tab 1: Create Task**: Interactive form with task name, description, type, priority selector, scheduled datetime offset (`Now`, `+15s`, `+1m`, custom), and instant submit with 429 rate-limit feedback toast.
      - **Tab 2: Tasks Explorer**: High-density tabular grid with search, status filters, UUID truncation, task type badges, duration meters, and cancel action for pending tasks.
      - **Tab 3: System Dashboard**: Large metric cards (Total, Pending, Running, Completed, Failed), Recharts status distribution chart, worker concurrency meters, and scheduler heartbeat timing.
      - **Tab 4: Spring Boot Java 3.2 Codebase**: Built-in interactive code viewer showcasing the complete Java 17 files (`Task.java`, `RabbitMQConfig.java`, `SchedulerService.java`, `WorkerService.java`, `TokenBucketService.java`, `TaskServiceTest.java`) with syntax highlighting and 1-click Download Project ZIP button.
    - **Right Column (Fixed Live Events Stream)**:
      - Always-visible scrolling telemetry panel auto-polling `/api/events` every 1s.
      - Source badges: `[SCHEDULER]`, `[WORKER]`, `[QUEUE]`, `[RATELIMIT]`, `[API]`.
      - Color-coded levels (INFO in muted cyan/gray, WARN in amber, ERROR in coral, SUCCESS in emerald).
      - Quick actions: Pause stream, Clear view, Filter by source, and Export logs.

- **Visual Identity & Color Tokens**:
  - *Canvas Neutral (60%)*: `#0a0e14` / `#0d1117` with subtle hexagonal geometric background pattern.
  - *Structural Surfaces (30%)*: `#161b22` cards, `#21262d` hairline borders, `#8b949e` muted text.
  - *Accent Budget (10%)*: Neon emerald `#22c55e` / `#10b981` for active tabs, primary CTAs, successful workers, and healthy tokens.
  - *Typography*: Clean neo-grotesque sans (`Plus Jakarta Sans` / `Outfit`) paired with `JetBrains Mono` with `tabular-nums` for all IDs, timestamps, token counts, and queue depth.
  - *Mobile & Tablet Responsiveness*: Drawer-toggleable Live Events panel on small viewports so the primary task form and task list remain effortlessly usable on tablets and phones.

---

### 3. Key Product Decisions & Trade-Offs

- **Decision 1: Live Engine + Full Spring Boot Java Project Mirror**
  - *Chosen Approach*: The running web application uses an Express backend that implements the exact REST API schema and state machine requested, while embedding and serving the full, production-ready Spring Boot 3.2 Java 17 Maven project files (`pom.xml`, `application.yml`, complete Java classes, JUnit tests) for download and viva demonstration.
  - *Why*: The browser environment runs in a Node/Linux container without pre-installed Java/MySQL/RabbitMQ daemons. This dual architecture ensures an instantly working, zero-friction live demo for the user while delivering 100% of the Java/Spring Boot code required for their college viva submission.
  - *Alternatives Considered*: Stubbing the UI with fake static data (rejected: breaks the requirement that the engine really runs and produces live ticks/events) or trying to compile Maven inside the container (slow and fragile in web previews).

- **Decision 2: 1-Second Polling Over WebSockets**
  - *Chosen Approach*: React `usePolling` hook querying `/api/tasks`, `/api/stats`, `/api/tasks/rate-limit-status`, and incremental `/api/events?since={lastId}` every 1000ms.
  - *Why*: Matches the user brief explicitly ("NO WebSocket needed. Simpler to implement, works fine for demo") and avoids socket reconnect drops, proxy buffering, or firewall issues during presentations.

- **Decision 3: Token Bucket Rate Limiter with Visible Refill Dynamics**
  - *Chosen Approach*: In-memory token bucket with capacity 5 and refill rate of 1 token per 12 seconds. The API returns tokens remaining, capacity, and `retryAfterSeconds`.
  - *Why*: Allows rapid creation of 5 tasks to demonstrate immediate processing, while the 6th triggers a crisp 429 response accompanied by a visible `RATELIMIT DENY` log event.

---

### 4. Technical Architecture & Data Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       REACT FRONTEND (Vite + Tailwind CSS)                  │
│                                                                             │
│  ┌───────────────────────┐ ┌────────────────────────┐ ┌──────────────────┐  │
│  │   Top Bar (Tokens,    │ │   Main View Tabs       │ │   Live Events    │  │
│  │  Queue, Viva Presets) │ │ (Create/Tasks/Dash/Java│ │  Panel (1s Poll) │  │
│  └───────────┬───────────┘ └───────────┬────────────┘ └────────┬─────────┘  │
└──────────────┼─────────────────────────┼───────────────────────┼────────────┘
               │ 1s Polling / REST API   │                       │
               ▼                         ▼                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FULL-STACK BACKEND ENGINE (server.ts)                    │
│                                                                             │
│  ┌─────────────────────────┐  ┌──────────────────────────────────────────┐  │
│  │ In-Memory Token Bucket  │  │   EventService (Concurrent Ring Buffer)  │  │
│  │ capacity=5, refill=12s  │  │   Cap 200 events, GET /api/events?since  │  │
│  └───────────┬─────────────┘  └────────────────────▲─────────────────────┘  │
│              │                                     │                        │
│  ┌───────────▼─────────────┐  ┌────────────────────┴─────────────────────┐  │
│  │   Task Engine & DB      │  │   Scheduler & 3-Worker Pool Executor     │  │
│  │ (PENDING/RUNNING/DONE)  │  │   5s interval tick, simulated latency    │  │
│  └───────────┬─────────────┘  └────────────────────▲─────────────────────┘  │
│              │                                     │                        │
│              └────────────► [ RabbitMQ Queue ] ────┘                        │
│                             (Priority dispatch)                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                       ▲
                                       │ (Includes exportable)
┌──────────────────────────────────────┴──────────────────────────────────────┐
│         COMPLETE SPRING BOOT 3.2 + JAVA 17 + MYSQL MAVEN PROJECT             │
│   ├── pom.xml (Spring Boot 3.2, spring-boot-starter-amqp, data-jpa, mysql)  │
│   ├── src/main/java/com/scheduler/model/Task.java (JPA Entity)              │
│   ├── src/main/java/com/scheduler/service/TokenBucketService.java           │
│   ├── src/main/java/com/scheduler/service/SchedulerService.java (@Scheduled)│
│   ├── src/main/java/com/scheduler/service/WorkerService.java                │
│   └── src/test/java/com/scheduler/TokenBucketServiceTest.java (JUnit 5)     │
└─────────────────────────────────────────────────────────────────────────────┘
```

- **Endpoints Contract**:
  - `POST /api/tasks`: Validates token bucket; if 0 tokens, returns `429 Too Many Requests` with `{ error, retryAfterSeconds }` and emits `WARN: RATELIMIT DENY`; if allowed, saves task and emits `INFO: API Task created`.
  - `GET /api/tasks`: Returns list of all tasks sorted by `createdAt DESC`.
  - `GET /api/tasks/:id`: Returns single task details.
  - `DELETE /api/tasks/:id`: Cancels pending task.
  - `GET /api/tasks/rate-limit-status`: Returns `{ tokens, capacity, refillSeconds, nextRefillIn }`.
  - `GET /api/events?since=:id`: Returns events occurred since last ID (ring buffer up to 200 items).
  - `GET /api/stats`: Returns `{ total, pending, running, completed, failed, tokensRemaining, queueDepth, schedulerLastRun, workersActive, concurrency: 3 }`.
  - `GET /api/download-java-project`: Generates or streams the complete Spring Boot 3.2 project archive.
  - `POST /api/viva/preset`: Executes automated viva scenarios (burst 6 tasks, rapid test, clear state).
