# CLAUDE.md — developer-portal

These rules apply to every session. Follow them without being asked.

---

## Project Overview

Static developer portal hosted at `developer.glorylabs.nl`. No build step, no framework —
pure HTML, CSS, and nginx. Redoc loads OpenAPI specs from live backend URLs at page load.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Site | Static HTML + CSS (no framework, no build step) |
| API viewer | Redoc (loaded from CDN via `<script>` tag) |
| Server | nginx:alpine |
| Container | Docker |
| Infra | k3s (TransIP VPS), Helm chart at `helm/developer-portal/` |
| Namespace | `developer-portal` |
| Domain | `developer.glorylabs.nl` |
| TLS | cert-manager + letsencrypt-prod |

---

## Port Configuration

| Service | Port | Notes |
|---------|------|-------|
| nginx (container) | 80 | HTTP only inside cluster |
| Ingress | 443 | TLS termination via cert-manager |

---

## Files

```
developer-portal/
├── index.html               # Landing page — card grid of all APIs
├── auditpic.html            # Redoc viewer — AuditPic API
├── consultancy.html         # Redoc viewer — Mahmoud Consultancy API
├── europelogin.html         # Redoc viewer — europeLogin API
├── claimio.html             # Redoc viewer — Claimio API
├── valideerleeftijd.html    # Redoc viewer — ValideerLeeftijd API
├── style.css                # Shared styles (dark nav, card grid, responsive)
├── nginx.conf               # nginx server block (gzip, cache headers, security)
├── Dockerfile               # nginx:alpine — copies everything into html root
├── .github/workflows/
│   └── deploy.yml           # Build image → push GHCR → helm upgrade
└── helm/developer-portal/   # Helm chart
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── _helpers.tpl
        ├── deployment.yaml
        ├── service.yaml
        └── ingress.yaml
```

---

## Adding a new API

1. Create a new `<slug>.html` by copying any existing Redoc page and updating:
   - `<title>` tag
   - `.nav-section` text
   - `spec-url` attribute on the `<redoc>` element
2. Add a card to `index.html` with the correct `href`, name, description, and tag colour.
3. Commit on a `feature/` branch, open a PR into `main`.

---

## Local development

```bash
# Build and run locally
docker build -t developer-portal .
docker run --rm -p 8080:80 developer-portal
# Open http://localhost:8080
```

No hot-reload. Edit the files, rebuild, restart.

---

## Deployment

Push to `main` triggers `.github/workflows/deploy.yml`:
1. Builds `ghcr.io/mahmoudholding/developer-portal:sha-<short>`
2. Pushes to GHCR with `sha-<short>`, `main`, and `latest` tags
3. Runs `helm upgrade --install` against the k3s cluster

Manual deploy:
```bash
helm upgrade --install developer-portal helm/developer-portal \
  --namespace developer-portal \
  --create-namespace \
  --set image.tag=sha-<short>
```

---

## GitHub Issues

Issues are tracked in the auditPic repo (portal is tracked there as **#34**):
https://github.com/mahmoudholding/auditPic/issues

---

## Rules

- No npm, no Webpack, no Angular/React/Vue — this is intentionally framework-free
- All styles live in `style.css` — no inline styles except inside `<style>` blocks for one-offs
- Redoc is loaded from CDN (`cdn.jsdelivr.net`) — no local copy needed
- Keep HTML semantic and accessible (`lang`, `alt`, `aria` where relevant)
- Never commit secrets — there are none in this project; keep it that way
