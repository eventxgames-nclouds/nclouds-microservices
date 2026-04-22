# EventXGames Microservices Specifications

Detailed specifications for all 16 application microservices running in Amazon EKS.

---

## Core Services

### W-01: Frontend Service

**Purpose:** Web application UI serving the EventXGames platform

**Technology:** Next.js 14

**Current (Azure):**
- Docker container serving React/Next.js application

**Target (AWS):**
```yaml
Deployment: frontend-service
Namespace: eventxgames
Replicas: 3 (min) - 15 (max)
Resources:
  Requests:
    CPU: 250m
    Memory: 512Mi
  Limits:
    CPU: 1000m
    Memory: 2Gi
HPA:
  Target: 60% CPU utilization
Health Checks:
  Readiness: GET /health (port 3000)
  Liveness: GET /health (port 3000)
```

---

### W-02: API Service

**Purpose:** Core API backend handling all business logic and data operations

**Technology:** Fastify/Node.js

**Current (Azure):**
- Docker container on Azure VPS
- Connected to PostgreSQL and Redis

**Target (AWS):**
```yaml
Deployment: api-service
Namespace: eventxgames
Replicas: 3 (min) - 20 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Health Checks:
  Readiness: GET /health (port 8080)
  Liveness: GET /health (port 8080)
```

**Topology:**
- Spread across 3 availability zones
- Anti-affinity rules prevent co-location

---

## Game Generation Workers

### W-03: Game Generation Orchestrator

**Purpose:** Coordinates game creation workflow across multiple services

**Technology:** Node.js

**Target (AWS):**
```yaml
Deployment: game-orchestrator
Namespace: eventxgames
Replicas: 3 (min) - 30 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - BEDROCK_MODEL: anthropic.claude-3-sonnet
  - SQS_QUEUE_URL: from ConfigMap
```

**Responsibilities:**
- Receive game generation requests
- Coordinate with Content, Asset, and Audio services
- Manage generation state machine
- Handle retries and error recovery

---

### W-04: Asset Worker

**Purpose:** Processes and transforms game assets (images, graphics)

**Technology:** Node.js + Sharp/Canvas

**Target (AWS):**
```yaml
Deployment: asset-worker
Namespace: eventxgames
Replicas: 2 (min) - 10 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Node Selection:
  role: spot-workloads
Tolerations:
  - key: spot-instance
    effect: NoSchedule
```

**Cost Optimization:**
- Runs on Spot instances for cost savings
- Non-critical workload tolerates interruptions

---

### W-05: Chat Worker

**Purpose:** Real-time chat and WebSocket connections

**Technology:** Node.js + Socket.io

**Target (AWS):**
```yaml
Deployment: chat-worker
Namespace: eventxgames
Replicas: 3 (min) - 20 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
Ports:
  - 8080 (HTTP)
  - 8081 (WebSocket)
HPA:
  Metrics:
    - Memory: 75% utilization
    - Custom: websocket_connections (avg 1000 per pod)
```

**Special Considerations:**
- Memory-based scaling (WebSocket connections consume memory)
- Session stickiness for WebSocket connections
- Graceful shutdown for connection migration

---

## Content & Templates

### W-06: Content Generation Service

**Purpose:** Generates trivia questions, stories, and game narratives using AI

**Technology:** Node.js + AWS Bedrock (Claude)

**Target (AWS):**
```yaml
Deployment: content-generator
Namespace: eventxgames
Replicas: 2 (min) - 15 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - BEDROCK_MODEL: anthropic.claude-3-sonnet
  - CONTENT_CACHE_TTL: 3600
```

**Capabilities:**
- Trivia question generation (multiple categories)
- Story/narrative generation
- Quiz content creation
- Content validation and filtering

---

### W-07: Template Service

**Purpose:** Manages game templates and configurations

**Technology:** Node.js

**Target (AWS):**
```yaml
Deployment: template-service
Namespace: eventxgames
Replicas: 2 (min) - 8 (max)
Resources:
  Requests:
    CPU: 250m
    Memory: 512Mi
  Limits:
    CPU: 1000m
    Memory: 2Gi
HPA:
  Target: 70% CPU utilization
```

**Game Categories Supported:**
- Trivia
- Quiz
- Arcade
- Puzzle
- Memory
- Spin-wheel

---

### W-08: Audio Generation Service

**Purpose:** Generates and processes sound effects and music

**Technology:** Node.js

**Target (AWS):**
```yaml
Deployment: audio-generator
Namespace: eventxgames
Replicas: 1 (min) - 5 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Node Selection:
  role: spot-workloads
```

**Capabilities:**
- Sound effect selection
- Background music management
- Audio format conversion
- Volume normalization

---

## Runtime Services (100K Players)

### W-09: Leaderboard Service

**Purpose:** Manages real-time player rankings and scores

**Technology:** Node.js + DynamoDB

**Target (AWS):**
```yaml
Deployment: leaderboard-service
Namespace: eventxgames
Replicas: 3 (min) - 20 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Metrics:
    - CPU: 70% utilization
    - Custom: requests_per_second (avg 500 per pod)
```

**Features:**
- Real-time score updates
- Global and per-game leaderboards
- Time-based rankings (daily, weekly, all-time)
- DynamoDB for low-latency reads

---

### W-10: Player Session Service

**Purpose:** Manages 100K concurrent player sessions

**Technology:** Node.js + ElastiCache Redis

**Target (AWS):**
```yaml
Deployment: player-session-service
Namespace: eventxgames
Replicas: 5 (min) - 50 (max)
Resources:
  Requests:
    CPU: 1000m
    Memory: 2Gi
  Limits:
    CPU: 4000m
    Memory: 8Gi
HPA:
  Metrics:
    - CPU: 65% utilization
    - Custom: active_sessions (avg 2000 per pod)
Environment:
  - REDIS_CLUSTER_URL: from Secrets Manager
```

**Scale Targets:**
- 100K concurrent players
- Sub-100ms session lookup
- Session affinity for active games

---

### W-11: Analytics Service

**Purpose:** Collects and processes player behavior and game metrics

**Technology:** Node.js + SQS

**Target (AWS):**
```yaml
Deployment: analytics-service
Namespace: eventxgames
Replicas: 2 (min) - 10 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - SQS_QUEUE: eventx-analytics
  - KINESIS_STREAM: eventx-events
```

**Metrics Collected:**
- Player engagement
- Game completion rates
- Lead qualification data
- Post-event analytics

---

## Support Services

### W-12: Localization Service

**Purpose:** Provides multi-language support using AI translation

**Technology:** Node.js + AWS Bedrock (Claude Sonnet)

**Target (AWS):**
```yaml
Deployment: localization-service
Namespace: eventxgames
Replicas: 2 (min) - 8 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - BEDROCK_MODEL: anthropic.claude-3-sonnet
  - TRANSLATION_CACHE_TTL: 86400
```

**Supported Languages:**
- Dynamic translation via Claude Sonnet
- Translation caching for performance
- RTL language support

---

### W-13: Notification Service

**Purpose:** Handles email and push notifications

**Technology:** Node.js + SNS

**Target (AWS):**
```yaml
Deployment: notification-service
Namespace: eventxgames
Replicas: 2 (min) - 8 (max)
Resources:
  Requests:
    CPU: 250m
    Memory: 512Mi
  Limits:
    CPU: 1000m
    Memory: 2Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - SNS_TOPIC: eventx-notifications
  - SES_SENDER: noreply@eventxgames.com
```

**Channels:**
- Email (via SES/ZeptoMail)
- Push notifications (via SNS)
- In-app notifications

---

### W-14: Preview Service

**Purpose:** Provides game preview and QA functionality

**Technology:** Node.js

**Target (AWS):**
```yaml
Deployment: preview-service
Namespace: eventxgames
Replicas: 2 (min) - 6 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
```

**Features:**
- Game preview rendering
- QA testing tools
- Screenshot generation
- Preview link generation

---

### W-15: Export Service

**Purpose:** Handles game export for self-hosting (ZIP download, embed code)

**Technology:** Node.js + S3

**Target (AWS):**
```yaml
Deployment: export-service
Namespace: eventxgames
Replicas: 2 (min) - 8 (max)
Resources:
  Requests:
    CPU: 500m
    Memory: 1Gi
  Limits:
    CPU: 2000m
    Memory: 4Gi
HPA:
  Target: 70% CPU utilization
Environment:
  - S3_EXPORT_BUCKET: eventxgames-exports
```

**Export Formats:**
- ZIP package for self-hosting
- Embed code (iframe)
- Standalone HTML

---

### W-16: Webhook Service

**Purpose:** Manages third-party integrations and callbacks

**Technology:** Node.js

**Target (AWS):**
```yaml
Deployment: webhook-service
Namespace: eventxgames
Replicas: 2 (min) - 8 (max)
Resources:
  Requests:
    CPU: 250m
    Memory: 512Mi
  Limits:
    CPU: 1000m
    Memory: 2Gi
HPA:
  Target: 70% CPU utilization
```

**Features:**
- Outbound webhooks for events
- Retry logic with exponential backoff
- Webhook signature verification
- Rate limiting per client

---

## Service Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│                         External Traffic                              │
└───────────────────────────────────┬─────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    CloudFront + ALB                                   │
└───────────────────────────────────┬─────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────┐
        │                           │                       │
        ▼                           ▼                       ▼
┌───────────────┐           ┌───────────────┐       ┌───────────────┐
│   Frontend    │           │  API Service  │       │  Chat Worker  │
│   Service     │           │               │       │  (WebSocket)  │
└───────────────┘           └───────┬───────┘       └───────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────┐
        │                           │                       │
        ▼                           ▼                       ▼
┌───────────────┐           ┌───────────────┐       ┌───────────────┐
│     Game      │           │   Content     │       │  Leaderboard  │
│  Orchestrator │           │  Generator    │       │   Service     │
└───────┬───────┘           └───────────────┘       └───────────────┘
        │
        ├───────────────────────────┬───────────────────────┐
        │                           │                       │
        ▼                           ▼                       ▼
┌───────────────┐           ┌───────────────┐       ┌───────────────┐
│ Asset Worker  │           │   Template    │       │    Audio      │
│               │           │   Service     │       │   Generator   │
└───────────────┘           └───────────────┘       └───────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────┐
        │                           │                       │
        ▼                           ▼                       ▼
┌───────────────┐           ┌───────────────┐       ┌───────────────┐
│    Aurora     │           │  ElastiCache  │       │   DynamoDB    │
│  PostgreSQL   │           │    Redis      │       │               │
└───────────────┘           └───────────────┘       └───────────────┘
```

---

## Database Access Patterns

| Service | Aurora (Read) | Aurora (Write) | Redis | DynamoDB |
|---------|--------------|----------------|-------|----------|
| Frontend Service | No | No | Yes (cache) | No |
| API Service | Yes | Yes | Yes | No |
| Game Orchestrator | Yes | Yes | Yes | No |
| Asset Worker | Yes | Yes | No | No |
| Chat Worker | Limited | No | Yes | No |
| Content Generator | Yes | Yes | Yes | No |
| Template Service | Yes | Yes | Yes | No |
| Audio Generator | Yes | Yes | No | No |
| Leaderboard Service | No | No | Yes | Yes |
| Player Session Service | Limited | No | Yes | No |
| Analytics Service | Yes | Yes | No | No |
| Localization Service | Yes | Yes | Yes | No |
| Notification Service | Yes | No | No | No |
| Preview Service | Yes | No | Yes | No |
| Export Service | Yes | No | No | No |
| Webhook Service | Yes | Yes | No | No |

---

## Redis Key Prefixes by Service

| Service | Key Prefix | Purpose |
|---------|------------|---------|
| API Service | `session:`, `cache:` | User sessions, data cache |
| Game Orchestrator | `game:`, `job:` | Game state, job tracking |
| Chat Worker | `pubsub:`, `ws:` | Real-time messaging |
| Leaderboard Service | `lb:` | Leaderboard caches |
| Player Session Service | `player:` | Player session data |
| All Services | `rate:` | Rate limiting |

---

## Migration Strategy

All services will be migrated using the **Replatform** strategy:

1. Containerize with EKS-compatible manifests
2. Configure IRSA for AWS service access
3. Update connection strings for Aurora/ElastiCache/DynamoDB
4. Deploy to staging EKS cluster
5. Run integration tests
6. Cutover with traffic shifting

### Migration Phases

| Phase | Services | Timeline |
|-------|----------|----------|
| Phase 1 | Frontend, API Service | Week 1-2 |
| Phase 2 | Game Orchestrator, Asset Worker, Chat Worker | Week 3-4 |
| Phase 3 | Content Generator, Template, Audio | Week 5-6 |
| Phase 4 | Leaderboard, Player Session, Analytics | Week 7-8 |
| Phase 5 | Localization, Notification, Preview, Export, Webhook | Week 9-10 |
