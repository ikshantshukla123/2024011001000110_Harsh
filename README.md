# FestPass — Docker & Container Security Assessment

## Student

- **Name:** Harsh Mishra
- **REGNO:** 2024011001000110
- **PORT:** 8010
- **Docker Hub username:** harshmishra_2024011001000110
- **Docker Hub repository:** https://hub.docker.com/r/harshmishra_2024011001000110/fest-backend

## AI tools used

- **opencode (DeepSeek V4)** — writing the Dockerfile, `compose.yaml`, `.dockerignore`, README, Trivy remediation guidance, and cosign commands.
- All commands were executed on this machine by me.

## Stack

| Service  | Image / build                  | Port        |
|----------|--------------------------------|-------------|
| frontend | `nginx:1.30-alpine`            | 8010 → 80   |
| backend  | `harshmishra_2024011001000110/fest-backend:1.0.0` | internal 5000 |
| db       | `postgres:17-alpine`           | internal 5432 |

Networks: `frontend-backend` (frontend + backend) and `backend-db` (backend + db).
The frontend cannot resolve or reach `db`. Database data lives in the named volume
`festpass_festpass-db`.

## Run

```bash
cp .env.example .env   # then set a password
docker compose up -d
open http://localhost:8010
```

## Stage 3 — Trivy before/after

| Image | Base | CRITICAL | HIGH | Size |
|-------|------|----------|------|------|
| `fest-backend:0.1` | `python:3.9` | 48 | 148 | 1.61 GB |
| `fest-backend:0.2` | `python:3.13-slim-bookworm`, non-root | 2 (0 fixable) | 20 | 237 MB |

Gate command (exit code 0 = no fixable CRITICAL):

```bash
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

Full reports: `reports/trivy-before.txt`, `reports/trivy-after.txt`.

## Stage 4 — Released image

- Tags pushed: `harshmishra_2024011001000110/fest-backend:1.0.0` and `:1.0`
- `1.0.0` digest: `PENDING`

## Stage 5 — cosign

- Public key: `cosign.pub`
- `1.0.0` is signed by digest and verifies with `cosign.pub`.
- `1.0.1` is pushed unsigned and is rejected by `cosign verify`.

```bash
cosign verify --key cosign.pub harshmishra_2024011001000110/fest-backend:1.0.0
```

## Steps not completed

- PENDING
