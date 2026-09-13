# 04 - Architecture

**Category:** Architecture  
**Purpose:** Technical architecture, layers, components, patterns

---

## Contents

- `architecture-overview.md` - High-level architecture
- `technology-stack.md` - Complete tech stack (100% custom, NO Medusa.js)
- `layered-architecture.md` - Presentation, business logic, data layers
- `component-diagram.md` - Major system components
- `design-patterns.md` - Applied design patterns
- `scalability-strategy.md` - Horizontal and vertical scaling approaches

---

## Technology Stack

### Backend
- **Runtime:** Node.js 20+ LTS
- **Language:** TypeScript 5+
- **Framework:** Express/Fastify (custom, NO Medusa.js)
- **ORM:** Prisma (decided via D-13)
- **Database:** PostgreSQL 16+
- **Cache:** Redis 7+
- **Search:** Elasticsearch 8+

### Frontend
- **Admin Panel:** React 18 + Vite + Tailwind CSS (Port 7001)
- **Vendor Panel:** React 18 + Vite + Tailwind CSS (Port 7002)
- **Customer Web:** Next.js 15 + React 19 (Port 3000)
- **Mobile Apps:** React Native (iOS/Android)

### Infrastructure
- **Deployment:** Docker + Kubernetes
- **CI/CD:** GitHub Actions / GitLab CI
- **Monitoring:** Prometheus + Grafana
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)

---

## Architecture Principles

1. **Custom Build** - NO third-party e-commerce frameworks
2. **Microservices-Ready** - Modular monolith with clear boundaries
3. **API-First** - REST API as primary interface
4. **Event-Driven** - Async processing for heavy operations
5. **Stateless** - Session state in Redis, not in-memory
6. **Scalable** - Horizontal scaling capability

---

## Related Categories
- `03-system-analysis` - System design
- `06-backend` - Backend implementation
- `05-frontend` - Frontend implementation

---

*Source: Architecture from analayesev2/02-ARCHITECTURE and technology decisions*
