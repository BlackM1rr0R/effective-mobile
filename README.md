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
├── .env             # Environment variables (NGINX_PORT=7070)
├── .gitignore
└── README.md
```

---

## How to Run

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/) ≥ 20
- [Docker Compose](https://docs.docker.com/compose/) v1 or v2

### Start

```bash
git clone <your-repo-url>
cd effective-mobile
docker-compose up --build -d
```

### Verify

```bash
curl http://localhost:7070
```

Expected response:

```
Hello from Effective Mobile!
```

You can also verify via the public server:

```bash
curl http://85.90.245.237:7070
```

### Stop

```bash
docker-compose down
```

---

## Configuration

The exposed host port is controlled via the `.env` file:

```env
NGINX_PORT=7070
```

Change this value if the port is already in use on your server.

---

## Architecture

```
 ┌──────────────────────────────────────────────────────────┐
 │                  Docker network: app-net                 │
 │                                                          │
 │  ┌─────────────────────┐              ┌───────────────┐  │
 │  │   nginx:80          │  proxy_pass  │ backend:8080  │  │
 │  │   (nginx:1.27-      │ ──────────►  │ (python:3.12- │  │
 │  │    alpine)          │              │  alpine)      │  │
 │  └──────────┬──────────┘              └───────────────┘  │
 └─────────────┼────────────────────────────────────────────┘
               │ port 7070 exposed to host
               │
        curl http://85.90.245.237:7070
```

**Flow:**
1. Client sends `GET /` to `85.90.245.237:7070`.
2. Nginx receives the request and forwards it to `backend:8080` via the internal Docker network (`app-net`).
3. Python HTTP server responds with `Hello from Effective Mobile!`.
4. Nginx returns the response to the client.

> The backend port **8080 is never exposed** to the host — it is accessible only within `app-net`.

---

## Technologies

| Component      | Image / Tool         |
|----------------|----------------------|
| Backend        | `python:3.12-alpine` |
| Reverse proxy  | `nginx:1.27-alpine`  |
| Orchestration  | Docker Compose       |

---

## Security Notes

- Backend runs as a **non-root** user (`appuser`).
- Only the Nginx port (`7070`) is published to the host — backend is not reachable from outside.
- No secrets are stored in the repository; host port is configured via `.env`.
- Both services have **healthchecks**; Nginx waits for backend to become healthy before starting.
