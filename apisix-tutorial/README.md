# 🧪 APISIX Hands-on Tutorial (Unofficial)

This repository provides a hands-on tutorial for [Apache APISIX](https://apisix.apache.org/), an open-source dynamic API gateway. It focuses on step-by-step learning via `curl` commands and covers real-world use cases like load balancing, rate limiting, authentication, proxy caching, and gRPC integration.

> 📘 **Note:** This is an **unofficial** tutorial, created for learning purposes. It is not affiliated with or endorsed by the Apache APISIX project.

---

## 📚 Contents

- Web service routing (`web1`, `web2`)
- Load balancer config
- Plugins: rate limit, key-auth, jwt-auth
- gRPC setup and test
- Proxy cache configuration
- External Go plugin
- Docker Compose environment
- Route testing and inspection

👉 See the full tutorial in [docs/tutorial.md](./docs/tutorial.md)

---

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- `curl`
- Basic knowledge of REST API and reverse proxy concepts

### Run APISIX via Docker Compose

```bash
docker compose -f example/docker-compose.yml up -d
```
