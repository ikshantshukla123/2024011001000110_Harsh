# FestPass — Docker & Container Security Assessment

## Student

- **Name:** Harsh Mishra
- **REGNO:** 2024011001000110
- **PORT:** 8010
- **Docker Hub username:** harshmishra_2024011001000110 (mocked — no live Docker Hub account in this environment)
- **Docker Hub repository:** https://hub.docker.com/r/harshmishra_2024011001000110/fest-backend
- **Registry used for the Stage 4/5 run:** `localhost:5005/harshmishra_2024011001000110/fest-backend` (local `registry:2` standing in for Docker Hub)

## AI tools used

- **opencode (DeepSeek V4)** — wrote the Dockerfile, `compose.yaml`, `.dockerignore`, README, Trivy remediation guidance and cosign commands.
- Every command was run on this machine.

## Stack

| Service  | Image / build | Port |
|----------|---------------|------|
| frontend | `nginx:1.30-alpine` | 8010 → 80 |
| backend | `localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0` (Docker Hub path without the mock registry prefix: `harshmishra_2024011001000110/fest-backend:1.0.0`) | internal 5000 |
| db | `postgres:17-alpine` | internal 5432 |

Networks: `frontend-backend` (frontend + backend) and `backend-db` (backend + db).
The frontend cannot resolve or reach `db` (`ping: bad address 'db'`). Database data
lives in the named volume `festpass_festpass-db`.

## Run

```bash
cp .env.example .env   # then set a password
docker compose up -d
open http://localhost:8010
```

## Stage 1 — backend image

- `backend/Dockerfile` starts `FROM python:3.9`, installs `requirements.txt` before
  copying `app.py`, and starts the API with `CMD`.
- Built as `fest-backend:0.1` (1.61 GB).
- Run as container `backend` on port 8010 with `REGNO=2024011001000110`;
  `GET /api/health` returned `"database":false` as expected.

## Stage 2 — three-tier compose stack

- `compose.yaml` defines `frontend`, `backend` and `db`, two networks, a named
  volume and a `pg_isready` healthcheck.
- `.env` is ignored by Git; `.env.example` is committed.
- Two registrations (Asha / Hackathon, Rahul / Robo Race) survived
  `docker compose down` + `docker compose up -d`.
- `docker compose exec frontend ping -c 1 db` failed; pinging `backend` worked.

## Stage 3 — Trivy before/after

| Image | Base | CRITICAL | HIGH | Size |
|-------|------|----------|------|------|
| `fest-backend:0.1` | `python:3.9` | 48 | 148 | 1.61 GB |
| `fest-backend:0.2` | `python:3.13-slim-bookworm`, non-root `appuser` | 2 (0 fixable) | 20 | 237 MB |

Gate command (exit code 0 = no fixable CRITICAL):

```bash
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

Full reports: `reports/trivy-before.txt`, `reports/trivy-after.txt`.
Remediation: pinned `python:3.13-slim-bookworm`, upgraded Flask to 3.1.3,
Werkzeug to 3.1.9 and psycopg2-binary to 2.9.13, and run as UID 10001.

## Stage 4 — released image

- Built with `APP_VERSION=1.0.0`.
- Pushed as `:1.0.0` and `:1.0` (same digest).
- `1.0.0` digest: `sha256:72d0ecd294513e76ffe452991b583a995f9e83ba91038a058347d4c1d47766cd`
- Local backend images were deleted and the stack was pulled back from the
  registry; the page then reported backend `1.0.0` and still listed both
  registrations.

### Moving from the mock registry to Docker Hub

```bash
docker tag localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0 \
  harshmishra_2024011001000110/fest-backend:1.0.0
docker push harshmishra_2024011001000110/fest-backend:1.0.0
docker push harshmishra_2024011001000110/fest-backend:1.0
# then remove the "localhost:5005/" prefix in compose.yaml
```

## Stage 5 — cosign

- Generated a password-protected key pair; only `cosign.pub` is committed.
- `*.key` and the local password file are ignored by Git.
- `1.0.0` was signed **by digest**:

```bash
cosign sign --key cosign.key --yes \
  localhost:5005/harshmishra_2024011001000110/fest-backend@sha256:72d0ecd294513e76ffe452991b583a995f9e83ba91038a058347d4c1d47766cd
```

- Verification succeeded:

```bash
cosign verify --key cosign.pub \
  localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0
# exit 0 — "The signatures were verified against the specified public key"
```

- `1.0.1` was built and pushed **unsigned**. Verification rejects it:

```bash
cosign verify --key cosign.pub \
  localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.1
# exit 10 — "no signatures found"
```

## Steps not completed

- Docker Hub push: no Docker Hub account/credentials were available, so a local
  `registry:2` at `localhost:5005` was used as the mock registry for Stage 4/5.
- Screenshots: this environment has no GUI/screen-recording permission, so the
  `screenshots/` PNGs still have to be captured by hand; commands for every
  evidence code are in `screenshots/CAPTURE.md`.
