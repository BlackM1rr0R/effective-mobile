# Effective Mobile — Docker Test Assignment

A minimal web stack: **Python HTTP backend** proxied through **Nginx**, orchestrated with **Docker Compose**.

---

## Project Structure

```
.
├── backend/
│   ├── Dockerfile   # Python 3.12-alpine image, runs as non-root user
│   └── app.py       # Simple HTTP server on port 8080
├── nginx/
│   └── nginx.conf   # Reverse-proxy config (upstream → backend:8080)
├── docker-compose.yml
├── .env             # Environment variables (NGINX_PORT)
├── .gitignore
└── README.md
```

---

## How to Run

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/) ≥ 24
- [Docker Compose](https://docs.docker.com/compose/) v2 (bundled with Docker Desktop)

### Start

```bash
git clone <your-repo-url>
cd <repo-folder>
docker compose up --build -d
```

### Verify

```bash
curl http://localhost
```

Expected response:

```
Hello from Effective Mobile!
```

### Stop

```bash
docker compose down
```

---

## Architecture

```
 ┌──────────────────────────────────────┐
 │           Docker network: app-net    │
 │                                      │
 │  ┌─────────────────┐                 │
 │  │  nginx:80       │                 │
 │  │  (nginx:1.27-   │  proxy_pass     │  ┌─────────────────────┐
 │  │   alpine)       │ ─────────────►  │  │  backend:8080       │
 │  └────────┬────────┘                 │  │  (python:3.12-      │
 │           │                          │  │   alpine)           │
 └───────────┼──────────────────────────┘  └─────────────────────┘
             │
     exposed on host
       port 80 only
             │
         curl http://localhost
```

**Flow:**
1. Client sends `GET /` to `localhost:80`.
2. Nginx receives the request and forwards it to `backend:8080` via the internal Docker network.
3. Python server responds with `Hello from Effective Mobile!`.
4. Nginx returns the response to the client.

> The backend port **8080 is never exposed** to the host — it is reachable only within `app-net`.

---

## Technologies

| Component | Image / Tool |
|-----------|-------------|
| Backend   | `python:3.12-alpine` |
| Reverse proxy | `nginx:1.27-alpine` |
| Orchestration | Docker Compose v2 |

---

## Security Notes

- Backend runs as a **non-root** user (`appuser`).
- Only port **80** is published to the host.
- No secrets are stored in the repository; use `.env` for configuration.
- Both services have **healthchecks**; nginx waits for backend to be healthy before starting.
