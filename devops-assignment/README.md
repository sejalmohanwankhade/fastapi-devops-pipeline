# DevOps Assignment – FastAPI + Docker + Jenkins + Prometheus + Grafana

## Project Structure

```
devops-assignment/
├── app/
│   ├── __init__.py
│   ├── main.py            # FastAPI app (with /metrics endpoint)
│   ├── schema.py
│   ├── services.py
│   └── data/              # Persistent JSON storage (mounted as Docker volume)
├── prometheus/
│   └── prometheus.yml     # Prometheus scrape config
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   └── prometheus.yml
│       └── dashboards/
│           ├── dashboard.yml
│           └── fastapi_dashboard.json
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
└── requirements.txt
```

---

## How to Run

### 1. Start all services

```bash
docker-compose up -d --build
```

### 2. Test the API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET    | http://localhost:8000/          | Hello message |
| GET    | http://localhost:8000/users     | List users |
| POST   | http://localhost:8000/users     | Add a user |
| GET    | http://localhost:8000/docs      | Swagger UI |
| GET    | http://localhost:8000/metrics   | Prometheus metrics |

Example POST:
```bash
curl -X POST http://localhost:8000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}'
```

---

## Data Persistence Test

```bash
# Add a user
curl -X POST http://localhost:8000/users -H "Content-Type: application/json" -d '{"name": "Test User", "email": "test@test.com"}'

# Destroy containers
docker-compose down

# Recreate containers (data volume is preserved)
docker-compose up -d --build

# Verify data still exists
curl http://localhost:8000/users
```

Data is stored in the `user_data` named Docker volume, so it persists across container restarts/recreations.

---

## Monitoring

| Service    | URL                           | Credentials   |
|------------|-------------------------------|---------------|
| Prometheus | http://localhost:9090         | —             |
| Grafana    | http://localhost:3000         | admin / admin |

- Prometheus scrapes `/metrics` from the FastAPI app every 15 seconds.
- Grafana is pre-configured with the Prometheus datasource and a FastAPI dashboard.

---

## Jenkins Pipeline

The `Jenkinsfile` defines these stages:
1. **Clone Repository** – pulls the source code
2. **Build Docker Image** – builds the FastAPI image
3. **Stop Existing Containers** – tears down old deployment
4. **Deploy Application** – brings up all services via docker-compose
5. **Health Check** – verifies the app is responding
6. **Verify Data Persistence** – confirms data volume is intact

### Jenkins Setup Steps
1. Install Jenkins on your server.
2. Install plugins: **Docker**, **Docker Pipeline**, **Git**.
3. Create a new **Pipeline** job.
4. Point it to this repo / Jenkinsfile.
5. Run the pipeline.

---

## Notes

- `users.json` is stored inside the `user_data` Docker named volume (not inside the container), so data is preserved even if containers are destroyed and recreated.
- The `/metrics` endpoint is automatically exposed by `prometheus-fastapi-instrumentator`.

## Output
# Docker Desktop
<img width="949" height="499" alt="Screenshot 2026-04-16 191101" src="https://github.com/user-attachments/assets/d1dad1e3-1a86-4ecb-9b28-353cb5f22cad" />

# 1. fastapi_app
<img width="959" height="479" alt="Screenshot 2026-04-16 185503" src="https://github.com/user-attachments/assets/cd92e5cc-cf6c-48e6-aacd-822e1e8d9a09" />
# 2. prometheus
<img width="959" height="418" alt="Screenshot 2026-04-16 190127" src="https://github.com/user-attachments/assets/5af00d2e-b101-4f77-b368-2a48b3177ac6" />
# 3. grafana
<img width="959" height="419" alt="Screenshot 2026-04-16 185717" src="https://github.com/user-attachments/assets/031184f2-ddae-48c9-925f-a8709cd62830" />


