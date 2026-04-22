# EventXGames Service Inventory

This repository documents the complete EventXGames service architecture for the Azure to AWS migration.

## Service Inventory Summary

```
EVENTXGAMES SERVICE INVENTORY
─────────────────────────────────────────
Application Microservices:     5  (EKS workloads)
AWS Infrastructure Services:  16  (managed services)
External SaaS Services:        3  (third-party)
AI Agent Tools:               20  (software tools)
─────────────────────────────────────────
Total Components:             44
```

> **Note:** Only the 5 Application Microservices are actual containerized workloads running in EKS. AWS Infrastructure Services are managed services, not application code.

## Contents

- [MICROSERVICES.md](./MICROSERVICES.md) - Detailed specifications for the 5 EKS workloads

---

## 1. Application Microservices (5 total)

These are the containerized workloads that run in Amazon EKS:

| ID | Service | Technology | Status | Scaling |
|----|---------|------------|--------|---------|
| W-01 | Frontend Service | Next.js 14 | Existing | CPU-based (60%) |
| W-02 | API Service | Fastify/Node.js | Existing | CPU-based (70%) |
| W-03 | Game Worker | Node.js | Existing | CPU + custom metrics |
| W-04 | Asset Worker | Node.js | Existing | CPU-based (spot) |
| W-05 | Chat Worker | Node.js | Existing | Memory + connections |

### Architecture Diagram

```
                    ┌─────────────┐
                    │     ALB     │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  API Worker   │  │Frontend Worker│  │  Game Worker  │
│  (3-20 pods)  │  │  (3-15 pods)  │  │  (5-50 pods)  │
└───────┬───────┘  └───────────────┘  └───────┬───────┘
        │                                      │
        └──────────────────┬───────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
    ┌─────────────────┐       ┌─────────────────┐
    │ Aurora PostgreSQL│       │ ElastiCache Redis│
    └─────────────────┘       └─────────────────┘
```

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
| A-15 | Amazon SQS | Message queuing |
| A-16 | SNS | Push notifications |

---

## 4. AI Agent Tools (20 total)

Software tools used by the AI orchestrator for game generation and platform operations. These are documented separately in the AI Platform repository.

See: [nclouds-ai-platform](https://github.com/eventxgames-nclouds/nclouds-ai-platform)

---

## Quick Links

- [Main Project Index](https://github.com/eventxgames-nclouds/nclouds-overview)
- [AWS Architecture](https://github.com/eventxgames-nclouds/nclouds-aws-architecture)
- [Security Documentation](https://github.com/eventxgames-nclouds/nclouds-security)
- [AI Platform](https://github.com/eventxgames-nclouds/nclouds-ai-platform)
