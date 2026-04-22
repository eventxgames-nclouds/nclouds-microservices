# EventXGames Microservices Documentation

This repository contains documentation for the EventXGames microservices architecture and their migration from Azure to AWS.

## Contents

- [MICROSERVICES.md](./MICROSERVICES.md) - Complete service mapping and specifications

## Service Overview

| Service | Purpose | Scaling Strategy |
|---------|---------|-----------------|
| API Worker | Core API backend | CPU-based (70% threshold) |
| Frontend Worker | Web application serving | CPU-based (60% threshold) |
| Game Worker | Game session management | CPU + custom metrics |
| Asset Worker | Asset processing | CPU-based on spot instances |
| Chat Worker | Real-time chat/WebSocket | Memory + connection-based |

## Architecture Diagram

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

## Quick Links

- [Main Project Index](https://github.com/eventxgames-nclouds/nclouds-overview)
- [AWS Architecture](https://github.com/eventxgames-nclouds/nclouds-aws-architecture)
