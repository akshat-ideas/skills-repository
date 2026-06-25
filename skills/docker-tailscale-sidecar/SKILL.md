---
name: docker-tailscale-sidecar
description: Use when adding a Tailscale sidecar to a Docker Compose project to expose one or more services over HTTPS via Tailscale Serve (private tailnet) or Funnel (public) — covers network namespace sharing, Docker DNS resolver compatibility, and the Tailscale Serve config that proxies traffic into the app.
---

# Docker Compose: Tailscale Sidecar

Join a Tailscale tailnet from inside Docker Compose, terminate TLS, and reverse-proxy HTTPS into one or more app containers via shared network namespace.

## Workflow

### 1. Discover services

Read the user's `docker-compose.yml`. Collect every `services:` entry with its exposed ports. If no compose exists yet, ask which service names + ports to add.

### 2. Ask the user — use the `question` tool

Call it **once** with all three questions:

1. **Which services should receive Tailscale traffic?** (multi-select)
   - One option per discovered service, label `name (port: NNN)`, description of role
   - Custom answer enabled so they can add services not yet in compose
2. **Serve (private) or Funnel (public)?** (single-select)
   - `Serve — accessible only to your tailnet (recommended for internal tools)`
   - `Funnel — publicly accessible HTTPS (recommended for demos)`
3. **Kernel mode or userspace?** (single-select)
   - `Kernel — needs NET_ADMIN + /dev/net/tun (recommended, faster)`
   - `Userspace — works on restricted hosts (Synology, locked-down CI)`

### 3. Generate the config

#### Sidecar service (always add)

```yaml
configs:
  ts-serve:
    content: |
      {ts-serve-json}

services:
  tailscale:
    image: tailscale/tailscale:latest
    container_name: tailscale-${PROJECT_NAME}
    hostname: ${PROJECT_NAME}
    environment:
      - TS_AUTHKEY=${TS_AUTHKEY}
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_SERVE_CONFIG=/config/serve.json
      - TS_USERSPACE=${TS_USERSPACE}
      - TS_AUTH_ONCE=true
      # Keep Docker's embedded DNS (127.0.0.11) so other services in the
      # shared netns can resolve sibling container names.
      - TS_ACCEPT_DNS=false
      - TS_CERT_DOMAIN=${TS_CERT_DOMAIN}
      - TS_ENABLE_HEALTH_CHECK=true
      - TS_LOCAL_ADDR_PORT=127.0.0.1:41234
    configs:
      - source: ts-serve
        target: /config/serve.json
    volumes:
      - ./ts/state:/var/lib/tailscale
    # Drop these two blocks when TS_USERSPACE=true:
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - net_admin
    ports:
      - "${CLIENT_PORT}:80"
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://127.0.0.1:41234/healthz"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 30s
    restart: unless-stopped
```

#### Per-selected-service: shared netns

Add to each picked service:

```yaml
  <service-name>:
    # ... existing config ...
    network_mode: service:tailscale
    depends_on:
      tailscale:
        condition: service_healthy
```

**Constraint:** only one container per shared netns can bind each port. If the user picks two services that both want `:80`, warn them and offer a second sidecar or distinct ports.

#### `ts-serve` JSON

Pick the matching shape; fill in `${TS_CERT_DOMAIN}` and the per-service ports.

**Single service, Serve:**
```json
{"Web":{"${TS_CERT_DOMAIN}:443":{"Handlers":{"/":{"Proxy":"http://127.0.0.1:<port>"}}}}}
```

**Single service, Funnel** — add `TCP` and `AllowFunnel`:
```json
{"TCP":{"443":{"HTTPS":true}},
"Web":{"${TS_CERT_DOMAIN}:443":{"Handlers":{"/":{"Proxy":"http://127.0.0.1:<port>"}}}},
"AllowFunnel":{"${TS_CERT_DOMAIN}:443":true}}
```

**Multiple services, path-based:**
```json
{
  "Web":{
    "${TS_CERT_DOMAIN}:443":{
      "Handlers":{
        "/":{"Proxy":"http://127.0.0.1:80"},
        "/<svc2>":{"Proxy":"http://127.0.0.1:<port2>"},
        "/<svc3>":{"Proxy":"http://127.0.0.1:<port3>"}
      }
    }
  }
}
```
Append `"AllowFunnel":{"${TS_CERT_DOMAIN}:443":true}` if Funnel.

The root `/` is the primary service. Other services get `/<name>` paths; the receiving container must reverse-proxy from those paths (e.g. nginx `location /api/ { proxy_pass http://api:3001/; }`).

#### `.env` additions

```env
TS_AUTHKEY=tskey-auth-...                 # https://login.tailscale.com/admin/settings/keys
TS_CERT_DOMAIN=my-app.tailf49db2.ts.net   # MagicDNS name, unique per tailnet
CLIENT_PORT=8841                          # host port for plain-HTTP local access
PROJECT_NAME=my-app                       # hostname shown in tailnet
TS_USERSPACE=false                        # true for userspace
```

Generate the auth key with **Reusable** + a tag (e.g. `tag:container`) to skip manual `tailscale up`. Enable **Funnel** on the key for public exposure.

#### `.dockerignore` — exclude the state directory

Add `ts/` so the persisted node state (and any other `ts/` files) never get shipped into the build context:

```gitignore
ts/
```

## Verification

```bash
docker compose up -d
docker compose logs tailscale --tail=50    # "tailscaled is healthy"
docker compose ps                          # tailscale = healthy
curl -I https://${TS_CERT_DOMAIN}          # from inside tailnet
# Funnel: repeat the curl from outside the tailnet.
```

## Common Mistakes

| Symptom | Fix |
|---|---|
| `not in /dev/net/tun` | Host blocks tun — switch to userspace |
| Restart loop on auth | Persist `./ts/state` volume |
| Service can't resolve siblings | `TS_ACCEPT_DNS=false` (overwrites Docker DNS otherwise) |
| Service starts before Tailscale | Add `tailscale: condition: service_healthy` |
| Healthcheck fails | Healthcheck URL must match `TS_LOCAL_ADDR_PORT` |
| Funnel rejected | Auth key needs Funnel permission + matching ACL tag |
| Port 80 conflict on host | Change `CLIENT_PORT`; in-netns `:80` is independent |
| New node every restart | Use a reusable key + persistent state dir |