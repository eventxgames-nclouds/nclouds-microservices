# EventXGames Microservices Mapping

## Service Specifications

### 1. API Worker

**Purpose:** Core API backend handling all business logic and data operations

**Current (Azure):**
- Docker container on Azure VPS
- Connected to PostgreSQL and Redis

**Target (AWS):**
```yaml
Deployment: api-worker
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

### 2. Frontend Worker

**Purpose:** Serves web application and static assets

**Current (Azure):**
- Docker container serving React/Next.js application

**Target (AWS):**
```yaml
Deployment: frontend-worker
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
```

---

### 3. Game Worker

**Purpose:** Handles game session management, real-time game logic, and player state

**Current (Azure):**
- Stateful Docker containers with Redis for session state

**Target (AWS):**
```yaml
Deployment: game-worker
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
    - Custom: active_game_sessions (avg 100 per pod)
Environment:
  - REDIS_URL: from Secrets Manager
```

**Special Considerations:**
- High memory for game state caching
- Custom metrics for session-based scaling
- Session affinity for active games

---

### 4. Asset Worker

**Purpose:** Processes and transforms game assets (images, audio, video)

**Current (Azure):**
- Background processing container

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

### 5. Chat Worker

**Purpose:** Real-time chat and WebSocket connections

**Current (Azure):**
- WebSocket-enabled container

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

## Service Dependencies

```
┌─────────────────────────────────────────────────────────────┐
│                      External Traffic                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Load Balancer                 │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  API Worker   │────▶│  Game Worker  │────▶│  Chat Worker  │
└───────────────┘     └───────────────┘     └───────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│    Aurora     │     │  ElastiCache  │     │   Bedrock     │
│  PostgreSQL   │     │    Redis      │     │   (Claude)    │
└───────────────┘     └───────────────┘     └───────────────┘
```

## Database Access Patterns

| Service | Aurora (Read) | Aurora (Write) | Redis |
|---------|--------------|----------------|-------|
| API Worker | Yes | Yes | Yes |
| Frontend Worker | No | No | Yes (cache) |
| Game Worker | Yes | Limited | Yes |
| Asset Worker | Yes | Yes | No |
| Chat Worker | Limited | No | Yes |

## Redis Key Prefixes by Service

| Service | Key Prefix | Purpose |
|---------|------------|---------|
| API Worker | `session:`, `cache:` | User sessions, data cache |
| Game Worker | `game:`, `lb:` | Game state, leaderboards |
| Chat Worker | `pubsub:` | Real-time messaging |
| All Services | `rate:` | Rate limiting |

## Migration Strategy

All services will be migrated using the **Replatform** strategy:

1. Containerize with EKS-compatible manifests
2. Configure IRSA for AWS service access
3. Update connection strings for Aurora/ElastiCache
4. Deploy to staging EKS cluster
5. Run integration tests
6. Cutover with traffic shifting
