# EventXGames Service Inventory

This repository documents the complete EventXGames service architecture for the Azure to AWS migration.

## Service Inventory Summary

```
EVENTXGAMES SERVICE INVENTORY
─────────────────────────────────────────
Application Microservices:    16  (EKS workloads)
AWS Infrastructure Services:  16  (managed services)
External SaaS Services:        3  (third-party)
AI Agent Tools:               20  (software tools)
─────────────────────────────────────────
Total Components:             55
```

> **Note:** Application Microservices are containerized workloads running in EKS. AWS Infrastructure Services are managed services, not application code.

## Contents

- [MICROSERVICES.md](./MICROSERVICES.md) - Detailed specifications for all 16 EKS workloads

---

## 1. Application Microservices (16 total)

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EKS APPLICATION MICROSERVICES                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CORE SERVICES (Existing)                                           │
│  ┌──────────────┐  ┌──────────────┐                                │
│  │   Frontend   │  │  API Service │                                │
│  │   (Next.js)  │  │  (Fastify)   │                                │
│  └──────────────┘  └──────────────┘                                │
│                                                                      │
│  GAME GENERATION WORKERS                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │    Game      │  │    Asset     │  │    Chat      │             │
│  │ Orchestrator │  │   Worker     │  │   Worker     │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                      │
│  CONTENT & TEMPLATES                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Content    │  │   Template   │  │   Audio      │             │
│  │  Generator   │  │   Service    │  │  Generator   │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                      │
│  RUNTIME SERVICES (100K Players)                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Leaderboard  │  │   Player     │  │  Analytics   │             │
│  │   Service    │  │  Sessions    │  │   Service    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                      │
│  SUPPORT SERVICES                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Localization │  │ Notification │  │   Preview    │             │
│  │   Service    │  │   Service    │  │   Service    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐                                │
│  │   Export     │  │   Webhook    │                                │
│  │   Service    │  │   Service    │                                │
│  └──────────────┘  └──────────────┘                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Complete Microservices List

#### Core Services (Existing)

| ID | Service | Technology | Status | Purpose |
|----|---------|------------|--------|---------|
| W-01 | Frontend Service | Next.js 14 | Existing | Web application UI |
| W-02 | API Service | Fastify/Node.js | Existing | Core API backend |

#### Game Generation Workers (Planned)

| ID | Service | Technology | Status | Purpose |
|----|---------|------------|--------|---------|
| W-03 | Game Orchestrator | Node.js | Planned | Game generation coordination |
| W-04 | Asset Worker | Node.js | Planned | Image/asset processing |
| W-05 | Chat Worker | Node.js | Planned | Real-time chat/WebSocket |

#### Content & Templates (Planned)

| ID | Service | Technology | Status | Purpose |
|----|---------|------------|--------|---------|
| W-06 | Content Generator | Node.js + Bedrock | Planned | Trivia, stories, narratives |
| W-07 | Template Service | Node.js | Planned | Game template management |
| W-08 | Audio Generator | Node.js | Planned | Sound effects, music |

#### Runtime Services - 100K Players (Planned)

| ID | Service | Technology | Status | Purpose |
|----|---------|------------|--------|---------|
| W-09 | Leaderboard Service | Node.js + DynamoDB | Planned | Real-time player rankings |
| W-10 | Player Session Service | Node.js + Redis | Planned | 100K concurrent player management |
| W-11 | Analytics Service | Node.js + SQS | Planned | Player behavior, game metrics |

#### Support Services (Planned)

| ID | Service | Technology | Status | Purpose |
|----|---------|------------|--------|---------|
| W-12 | Localization Service | Node.js + Bedrock | Planned | Multi-language support |
| W-13 | Notification Service | Node.js + SNS | Planned | Email/push notifications |
| W-14 | Preview Service | Node.js | Planned | Game preview/QA |
| W-15 | Export Service | Node.js + S3 | Planned | Download ZIP, embed code |
| W-16 | Webhook Service | Node.js | Planned | Third-party integrations |

---

## 2. External SaaS Services (3 total)

Third-party services that remain unchanged during migration:

| ID | Service | Provider | Purpose | Migration Impact |
|----|---------|----------|---------|------------------|
| E-01 | Firebase Auth | Google Cloud | User authentication | No change |
| E-02 | ZeptoMail | Zoho | Transactional email | No change |
| E-03 | Cloudflare | Cloudflare | DNS (partial) | Route 53 for AWS |

---

## 3. AWS Infrastructure Services (16 total)

These are AWS managed services, NOT application microservices:

### Compute
| ID | Service | Purpose |
|----|---------|---------|
| A-01 | Amazon EKS | Kubernetes container orchestration |
| A-02 | EC2 (EKS nodes) | Worker nodes for EKS cluster |

### Database
| ID | Service | Purpose |
|----|---------|---------|
| A-03 | Aurora PostgreSQL | Primary relational database |
| A-04 | ElastiCache Redis | Caching and session storage |
| A-05 | DynamoDB | Game state and leaderboards |

### Storage & CDN
| ID | Service | Purpose |
|----|---------|---------|
| A-06 | Amazon S3 | Asset storage |
| A-07 | CloudFront | Global content delivery |

### Networking
| ID | Service | Purpose |
|----|---------|---------|
| A-08 | Route 53 | DNS management |
| A-09 | ALB | Application load balancing |
| A-10 | API Gateway | API management |

### AI/ML
| ID | Service | Purpose |
|----|---------|---------|
| A-11 | AWS Bedrock | AI model inference (Claude) |

### Security
| ID | Service | Purpose |
|----|---------|---------|
| A-12 | Secrets Manager | Credential management |
| A-13 | WAF | Web application firewall |

### Monitoring & Messaging
| ID | Service | Purpose |
|----|---------|---------|
| A-14 | CloudWatch | Logging and monitoring |
| A-15 | Amazon SQS | Message queuing (eventx-analytics) |
| A-16 | SNS | Push notifications (eventx-notifications) |

---

## 4. AI Agent Tools (20 total)

Software tools used by the AI orchestrator for game generation and platform operations.

See: [nclouds-ai-platform](https://github.com/eventxgames-nclouds/nclouds-ai-platform)

---

## Service Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│                         External Traffic                              │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    CloudFront + ALB                                   │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   Frontend    │       │  API Service  │       │  Chat Worker  │
│   Service     │       │               │       │  (WebSocket)  │
└───────────────┘       └───────┬───────┘       └───────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│     Game      │       │   Content     │       │  Leaderboard  │
│  Orchestrator │       │  Generator    │       │   Service     │
└───────┬───────┘       └───────────────┘       └───────────────┘
        │
        ├───────────────────────┬───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│ Asset Worker  │       │   Template    │       │    Audio      │
│               │       │   Service     │       │   Generator   │
└───────────────┘       └───────────────┘       └───────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│    Aurora     │       │  ElastiCache  │       │   DynamoDB    │
│  PostgreSQL   │       │    Redis      │       │               │
└───────────────┘       └───────────────┘       └───────────────┘
```

---

## Quick Links

- [Main Project Index](https://github.com/eventxgames-nclouds/nclouds-overview)
- [AWS Architecture](https://github.com/eventxgames-nclouds/nclouds-aws-architecture)
- [Security Documentation](https://github.com/eventxgames-nclouds/nclouds-security)
- [AI Platform](https://github.com/eventxgames-nclouds/nclouds-ai-platform)
