
# Docker Lab: Containerization & Orchestration


## Dockerized Components

- **Scratch image**: Minimal container using a statically compiled binary.
- **Java/OpenCV image**: Containerized Java application with native library support.
- **Lightweight image**: Multi-stage build to reduce image size.
- **Reverse proxy**: Nginx-based proxy for routing to multiple containers.


## Key Docker Files

- `step1.1/Dockerfile` : scratch image
- `step1.2/Dockerfile` : Java/OpenCV image
- `step1.2/Dockerfile.light` : multi-stage lightweight image

## How to Use

### Prerequisites

- Linux environment
- Docker and Docker Compose installed

### Start the Stack

Run all services in the background:

```bash
docker compose up -d
```

### Check Running Containers

```bash
docker compose ps
```

### Test Reverse Proxy

Test routing with curl:

```bash
curl -H "Host: web.local" http://127.0.0.1:8080
curl -H "Host: app.local" http://127.0.0.1:8080
```

### Browser Access

Add to `/etc/hosts`:

```
127.0.0.1 web.local app.local
```

Then open in your browser:

- http://web.local:8080
- http://app.local:8080

