# Screenshot capture sheet

Set the prompt in every new terminal first (zsh):

```bash
export PS1='[2024011001000110] %~ %# '
```

Capture the whole window. Name each file with the evidence code.

## S1-2 — image built with size

```bash
docker build -t fest-backend:0.1 ./backend
docker image ls fest-backend
```

## S1-3 — container, health, logs

```bash
docker run -d --name backend -p 8010:5000 -e REGNO=2024011001000110 fest-backend:0.1
docker ps
curl -s http://localhost:8010/api/health
docker logs backend
docker rm -f backend
```

## S2-2 — frontend cannot reach db

```bash
docker compose exec frontend ping -c 1 -W 2 db
```

## S2-3 — page with REGNO and two registrations

Open <http://localhost:8010> in a browser; the address bar and the status line
`REGNO 2024011001000110 | backend 1.0.0 ... database connected` must be visible.

## S2-4 — persistence after down/up and the volume

```bash
docker compose down
docker compose up -d
curl -s http://localhost:8010/api/registrations
docker volume ls | grep festpass
```

## S3-1 — Trivy before

```bash
trivy image --severity CRITICAL,HIGH fest-backend:0.1
```

## S3-3 — Trivy after, gate exits 0

```bash
docker build -t fest-backend:0.2 ./backend
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
echo $?
```

## S4-1 — push output with digest

```bash
docker tag fest-backend:0.2 localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0
docker push localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0
docker push localhost:5005/harshmishra_2024011001000110/fest-backend:1.0
```

For the real submission, push to Docker Hub instead and show the Hub tags page
for S4-2.

## S4-3 — stack running from the registry

```bash
docker compose down
docker rmi -f fest-backend:0.1 fest-backend:0.2 localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0 localhost:5005/harshmishra_2024011001000110/fest-backend:1.0
docker compose up -d
curl -s http://localhost:8010/api/health
```

## S5-1 — only the public key is committed

```bash
cat .gitignore
git status --short
git ls-files | grep cosign
```

## S5-2 — sign by digest

```bash
cosign sign --key cosign.key --yes \
  localhost:5005/harshmishra_2024011001000110/fest-backend@sha256:72d0ecd294513e76ffe452991b583a995f9e83ba91038a058347d4c1d47766cd
```

## S5-3 — verify 1.0.0

```bash
cosign verify --key cosign.pub \
  localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.0
echo $?
```

## S5-4 — unsigned 1.0.1 is rejected

```bash
docker build --build-arg APP_VERSION=1.0.1 -t localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.1 ./backend
docker push localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.1
cosign verify --key cosign.pub \
  localhost:5005/harshmishra_2024011001000110/fest-backend:1.0.1
echo $?
```
