# GitHub Actions Capstone - End-to-End CI/CD Pipeline

A simple Flask app used to demonstrate a full CI/CD pipeline built entirely with reusable GitHub Actions workflows - covering PR checks, main-branch build/push/deploy, and scheduled health monitoring, etc.

## App

A minimal Flask app with two endpoints:

| Endpoint  | Description                     |
|-----------|----------------------------------|
| `/`       | Returns a welcome message       |
| `/health` | Returns `{"status": "healthy"}` — used by the scheduled health check |

## Project Structure

```
.
├── app.py                  # Flask application
├── requirements.txt        # Python dependencies
├── test_app.py             # Basic endpoint tests (pytest)
├── Dockerfile               # Container build definition
├── .github/
│   └── workflows/
│       ├── reusable-build-test.yml   # Reusable: install deps + run tests
│       ├── reusable-docker.yml       # Reusable: build & push Docker image
│       ├── pr-pipeline.yml            # PR trigger: build & test only
│       ├── main-pipeline.yml          # Main trigger: test → build → deploy
│       └── health-check.yml           # Scheduled: pull image, curl /health
└── README.md
```

## Running Locally

```bash
pip install -r requirements.txt
python app.py
```

The app will be available at `http://localhost:5000`.

Run tests:

```bash
pytest
```

## Running with Docker

```bash
docker build -t gha-capstone .
docker run -p 5000:5000 gha-capstone
```

## CI/CD Pipeline

### On Pull Request (`pr-pipeline.yml`)
- Calls the reusable **build & test** workflow (tests only, no Docker build/push)
- Posts a summary confirming PR checks passed for the branch

### On Merge to `main` (`main-pipeline.yml`)
1. **Build & Test** — runs the reusable build/test workflow
2. **Docker Build & Push** — builds and pushes the image to Docker Hub, tagged `latest` and `sha-<short-commit-hash>`
3. **Deploy** — deploys to the `production` environment (manual approval required if environment protection rules are enabled)

### Scheduled Health Check (`health-check.yml`)
- Runs every 12 hours (`0 */12 * * *`) and can also be triggered manually
- Pulls the latest image, runs it in detached mode, curls `/health`, and reports pass/fail
- Publishes results to the GitHub Actions step summary

## Pipeline Architecture

```
PR opened/updated ──► build & test ──► PR checks pass

Merge to main ──► build & test ──► Docker build & push ──► deploy (production)

Every 12 hours ──► pull image ──► run container ──► curl /health ──► report
```
---

## Status Badges
 
Add these to the top of this file once workflows have run at least once (replace `<owner>`/`<repo>`):
 
```markdown
![PR Pipeline]([![PR Pipeline](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/pr-pipeline.yml/badge.svg)](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/pr-pipeline.yml))
![Main Pipeline]([![Main Pipeline](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/main-pipeline.yml/badge.svg)](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/main-pipeline.yml))
![Health Check]([![Health check schedule](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/health-check.yml/badge.svg)](https://github.com/jeenicj97/github-actions-capstone/actions/workflows/health-check.yml))
```

---
