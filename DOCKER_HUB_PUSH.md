# Docker Hub Push — Anleitung

## Voraussetzung
- Docker Hub Account (`rays23`) — erstellen unter https://hub.docker.com/signup
- Lokale Images `kaspanodemon_server:latest` und `kaspanodemon_client:latest` auf dem Unraid Server

## Schritt 1: Docker Hub Login (auf Unraid via SSH)

```bash
docker login
# Username: rays23
# Password: <dein Docker Hub Passwort oder Access Token>
```

**Empfehlung:** Erstelle einen Access Token unter https://hub.docker.com/settings/security statt das Passwort direkt zu verwenden.

## Schritt 2: Images taggen

```bash
# Monitor Server
docker tag kaspanodemon_server:latest rays23/kaspa-monitor-server:latest

# Monitor Client
docker tag kaspanodemon_client:latest rays23/kaspa-monitor-client:latest
```

## Schritt 3: Images pushen

```bash
# Monitor Server
docker push rays23/kaspa-monitor-server:latest

# Monitor Client
docker push rays23/kaspa-monitor-client:latest
```

## Schritt 4: Verifizieren

```bash
# Auf einem anderen Rechner oder nach docker rmi:
docker pull rays23/kaspa-monitor-server:latest
docker pull rays23/kaspa-monitor-client:latest
```

Alternativ unter https://hub.docker.com/u/rays23 prüfen, ob beide Repos sichtbar sind.

## Schritt 5: Docker Hub Repo-Beschreibung

Unter https://hub.docker.com/repository/docker/rays23/kaspa-monitor-server/general und .../kaspa-monitor-client/general jeweils eine kurze Beschreibung eintragen:

**kaspa-monitor-server:**
> Backend server for Kaspa Node Monitor. Connects via gRPC to rusty-kaspad and provides real-time data via WebSocket API. Part of the Kaspa Unraid monitoring stack.

**kaspa-monitor-client:**
> Web dashboard for Kaspa Node Monitor. Shows real-time block count, peers, hashrate, and node status. Part of the Kaspa Unraid monitoring stack.

## Optional: GitHub Actions CI/CD

Für automatische Rebuilds bei neuen Releases kannst du eine `.github/workflows/docker-publish.yml` im KaspaNodeMonitor-Fork erstellen. Siehe: https://docs.github.com/en/actions/publishing-packages/publishing-docker-images