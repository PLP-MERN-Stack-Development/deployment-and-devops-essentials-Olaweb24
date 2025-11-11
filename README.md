# Deployment & DevOps Essentials — PLP-MERN-Stack-Development (Olaweb24)

A compact, practical guide and documentation for the deployment and DevOps deliverables for the MERN stack project in this repository.


## Table of contents
- Project overview
- Deployed applications
- Architecture & components
- Tech stack
- How to run locally
- Environment variables
- CI / CD pipeline (screenshots included)
- Monitoring & observability (documentation + sample configs)
- Logs, alerts, and incident runbook
- Troubleshooting
- Contributing
- License
- Contact

---

## Project overview
This repository contains the deployment, DevOps, and Ops-related assets used to deploy and monitor a MERN (MongoDB, Express, React, Node) stack application. It includes CI/CD workflows, infrastructure configuration examples, monitoring dashboards and alerting rules, and documentation to reproduce the environment.

Goals:
- Continuous integration and delivery for frontend and backend
- Containerized deployments (Docker)
- Simple orchestration and/or PaaS deployment examples
- Production-grade monitoring and alerting
- Clear documentation so the system is reproducible

---

## Deployed applications

- Frontend (React) — Deployed URL:
  - https://chat-app-frontend-c8on.vercel.app/

- Backend API (Node/Express) — Deployed URL:
  - https://mern-chat-backend-azxk.onrender.com/


---

## Architecture & components

High-level architecture:
- React frontend served either as static assets via CDN / Vercel / Netlify or as a container behind a load balancer.
- Node/Express backend exposing a REST API; runs in a container or in a PaaS.
- MongoDB (managed service or self-hosted) as the primary datastore.
- CI/CD: GitHub Actions that build, test, containerize, and deploy.
- Monitoring: Prometheus (metrics), Grafana (dashboards), Alertmanager (alerts), Loki / ELK (logs) or a cloud logging alternative.
- Container registry: Docker Hub / GitHub Container Registry / private registry.

Recommended directory layout in this repo:
- /.github/workflows/ — GitHub Actions workflows (CI/CD)
- /docker/ — Dockerfiles, docker-compose files
- /k8s/ or /helm/ — Kubernetes manifests or Helm charts (if using k8s)
- /infra/ — Terraform or cloud deployment scripts (optional)
- /monitoring/ — Prometheus/Grafana/Alertmanager configs
- /docs/screenshots/ — pipeline and monitoring screenshots

---

## Tech stack
- Frontend: React (create-react-app / Vite)
- Backend: Node.js + Express
- Database: MongoDB (Atlas or self-hosted)
- Containerization: Docker
- CI/CD: GitHub Actions
- Monitoring: Prometheus + Grafana + Alertmanager (or cloud alternatives)
- Logging: Loki / Elasticsearch / Papertrail / Cloud provider logs
- Registry: Docker Hub or GHCR
- Orchestration / hosting: Kubernetes / Render / Heroku / Vercel / AWS / DigitalOcean

---

## How to run locally

1. Clone the repository:
   git clone https://github.com/PLP-MERN-Stack-Development/deployment-and-devops-essentials-Olaweb24.git
   cd deployment-and-devops-essentials-Olaweb24

2. Start MongoDB (example with Docker):
   docker run -d --name local-mongo -p 27017:27017 mongo:6

3. Backend:
   cd backend
   cp .env.example .env
   # Edit .env to point to local MongoDB and set other secrets
   npm install
   npm run dev
   # or in Docker:
   docker build -t my-backend:local .
   docker run -p 5000:5000 --env-file .env my-backend:local

4. Frontend:
   cd frontend
   cp .env.example .env
   npm install
   npm start
   # or build:
   npm run build

5. Access:
   - Frontend: http://localhost:3000 (or configured port)
   - Backend API: http://localhost:5000/api

---

## Environment variables

Example minimal backend .env:
- MONGO_URI=mongodb://localhost:27017/mydb
- PORT=5000
- JWT_SECRET=__REPLACE_WITH_SECRET__
- NODE_ENV=development

Example minimal frontend .env:
- REACT_APP_API_URL=http://localhost:5000/api

Never commit production secrets to git. Use GitHub secrets, cloud secret managers, or environment variables injected by your host.

---

## CI / CD pipeline

This project uses GitHub Actions as an example CI/CD pipeline. A suggested set of workflows:
- .github/workflows/ci.yml — install, lint, test for frontend & backend
- .github/workflows/build-and-push.yml — build Docker images and push to registry
- .github/workflows/deploy-frontend.yml — deploy frontend to hosting (Vercel / Netlify / S3 + CloudFront / Kubernetes)
- .github/workflows/deploy-backend.yml — deploy backend (Kubernetes / Render / Heroku / Docker host)

Screenshots of CI/CD pipeline in action:
- Add your pipeline screenshots to `docs/screenshots/` and link here. Example Markdown:

![CI Run - build tests and lint](docs/screenshots/cicd-ci-run-1.png)
![CD Run - frontend deploy](docs/screenshots/cicd-deploy-frontend.png)
![CD Run - backend deploy](docs/screenshots/cicd-deploy-backend.png)

If you want these to render in GitHub, create the images at those paths. Example suggestions for capturing screenshots:
- GitHub Actions run page showing the workflow run list
- A workflow run showing job steps (build, test, docker build, push, deploy)
- Kubernetes rollout or PaaS deploy console showing a successful deployment

Sample GitHub Actions job (simplified):

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      - name: Install & Test Backend
        working-directory: backend
        run: |
          npm ci
          npm test --silent

  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      - name: Install & Test Frontend
        working-directory: frontend
        run: |
          npm ci
          npm test --silent
```

Deployment jobs will vary according to your target provider. Use runtime secrets stored in GitHub repository secrets (e.g., DOCKER_USERNAME, DOCKER_PASSWORD, KUBE_CONFIG, VERCEL_TOKEN).

---

## Monitoring & Observability

This section documents a recommended monitoring setup (Prometheus + Grafana + Alertmanager). Adjust according to your hosting provider.

High-level components:
- Prometheus: Scrapes metrics endpoints (Node exporter, application metrics /metrics, cAdvisor)
- Grafana: Dashboards to visualize metrics
- Alertmanager: Sends alerts via Slack / PagerDuty / Email
- Logging: Loki / Elasticsearch or cloud logs for application logs
- Uptime/availability: Blackbox exporter or external uptime monitoring service

Suggested metrics to collect:
- Application: request rate, error rate, latency (p50/p95), memory/CPU per process, GC pause times, active connections
- Infrastructure: CPU, memory, disk, network, container restarts
- Database: replica lag, query latency, connection count

Prometheus sample scrape config (monitoring/prometheus.yml):
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'backend'
    static_configs:
      - targets: ['backend:9090', 'backend:3000']  # Replace with actual host:port where /metrics is exposed

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

Example alert rules (monitoring/alerts.yml):
```yaml
groups:
- name: app.rules
  rules:
  - alert: HighErrorRate
    expr: increase(http_requests_total{status=~"5.."}[5m]) > 5
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High 5xx error rate"
      description: "Service {{ $labels.job }} has high error rate."
```

Grafana:
- Create dashboards for:
  - API latency and error rates
  - Throughput
  - Host/Container resource usage
  - MongoDB metrics
- You can store Grafana dashboard JSON in `monitoring/grafana/dashboards/`

Logging:
- Centralize logs with Loki or Elasticsearch.
- Configure application to log json to stdout; collector (promtail/filebeat/fluentd) will harvest logs and push to central store.

Alert delivery:
- Configure Alertmanager with receivers:
  - Slack webhook
  - PagerDuty
  - Email
- Example Alertmanager receiver config (monitoring/alertmanager.yml):
```yaml
route:
  receiver: 'slack-notifications'
receivers:
- name: 'slack-notifications'
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/XXXXXXXX'
    channel: '#alerts'
```

Grafana / Prometheus access:
- If running locally in Docker:
  docker run --network host prom/prometheus
  docker run -p 3000:3000 grafana/grafana

- For Kubernetes, use port-forwarding to view dashboards:
  kubectl port-forward svc/grafana 3000:3000 -n monitoring

Documentation of your monitoring setup (what to include here in repo):
- `monitoring/README.md` explaining how to spin up monitoring stack (docker-compose or helm)
- Prometheus config files (prometheus.yml, alert rules)
- Grafana dashboard JSONs
- Alertmanager config and receiver secrets (do not commit secrets — use templated files)
- Runbook (how to silence alerts, how to escalate)

Add screenshots of monitoring dashboards:
![Grafana - API Latency Dashboard](docs/screenshots/monitoring-grafana-latency.png)
![Prometheus - Targets](docs/screenshots/monitoring-prometheus-targets.png)
![Alertmanager - Alerts](docs/screenshots/monitoring-alertmanager.png)

---

## Logs, alerts, and incident runbook

Brief runbook:
- If new critical alert fires (severity=critical):
  1. Check Alertmanager for alert details.
  2. Inspect Grafana dashboard for metric spikes.
  3. Check logs in Loki/ELK for correlated errors (query by trace id / request id).
  4. If deployment related, check recent GitHub Actions runs and rollout history.
  5. If database related, check MongoDB status and replica set health.
  6. Post incident summary to incident channel and, if required, page on-call via PagerDuty.

Escalation policy:
- 0-15 minutes: on-call engineer investigates
- 15-60 minutes: escalate to team lead
- >60 minutes: executive notice

Store full runbook in `docs/runbooks/incident-response.md`.

---

## Troubleshooting

Common issues & quick checks:
- "500 errors in API": check backend logs; verify DB connectivity; roll back recent deployment
- "Frontend cannot reach backend": verify REACT_APP_API_URL env var and CORS configuration
- "High memory usage": inspect process memory snapshots, consider increasing pod resources, or add horizontal scaling
- "CI failing on Docker push": check registry credentials and GitHub Secrets

Useful commands:
- kubectl get pods -A
- kubectl logs -n namespace pod-name
- docker logs container-name
- docker-compose up --build

---

## Contributing

1. Fork the repository
2. Create a branch: git checkout -b feature/my-change
3. Make changes, add tests, run CI locally
4. Open a pull request with clear description and screenshots if appropriate

Coding standards:
- Linting with ESLint and Prettier for JS/TS
- Write tests for significant business logic

---

## License
This project is licensed under the MIT License. See LICENSE for details.

---

## Contact
Maintainer: Olaweb24
Repository: https://github.com/PLP-MERN-Stack-Development/deployment-and-devops-essentials-Olaweb24

---

Appendix — Checklist before final submission
- [ ] Set FRONTEND and BACKEND deployed URLs at the top of this README
- [ ] Add CI/CD screenshots to `docs/screenshots/` and commit them
- [ ] Add Grafana dashboards and Prometheus config under `monitoring/`
- [ ] Ensure GitHub secrets (registry credentials, API tokens) are configured and not committed
- [ ] Update runbook and alerting channels with actual Slack/PagerDuty endpoints
