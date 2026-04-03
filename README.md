# GloryLabs Developer Portal

Static API documentation portal for all GloryLabs applications, hosted at
[developer.glorylabs.nl](https://developer.glorylabs.nl).

Built with plain HTML, CSS, and [Redoc](https://github.com/Redocly/redoc). No build step.
OpenAPI specs are loaded live from each backend's `/api-docs` endpoint.

---

## APIs documented

| API | Spec URL | Page |
|-----|----------|------|
| AuditPic | `https://api.auditpic.com/api-docs` | `auditpic.html` |
| Mahmoud Consultancy | `https://api.mahmoud-consultancy.com/api-docs` | `consultancy.html` |
| europeLogin | `https://api.europelogin.com/api-docs` | `europelogin.html` |
| Claimio | `https://api.claimio.nl/api-docs` | `claimio.html` |
| ValideerLeeftijd | `https://api.valideerleeftijd.nl/api-docs` | `valideerleeftijd.html` |

---

## Run locally

Requires Docker.

```bash
docker build -t developer-portal .
docker run --rm -p 8080:80 developer-portal
```

Open [http://localhost:8080](http://localhost:8080).

---

## Add a new API

1. Copy an existing Redoc page (e.g. `auditpic.html`) to a new file named after your API slug:

   ```bash
   cp auditpic.html myapi.html
   ```

2. Edit `myapi.html` — update three things:
   - `<title>` — name of the API
   - `.nav-section` span — display name shown in the nav bar
   - `spec-url` attribute on `<redoc>` — full URL to the live `/api-docs` JSON

3. Add a card to `index.html` inside the `.card-grid` div. Copy an existing `<a class="card">` block
   and update: `href`, the icon character, `card-name`, description text, and `card-tag` colour class.

4. Commit on a `feature/` branch and open a PR into `main`. CI will build, push, and deploy.

---

## Deploy manually

Prerequisites: `kubectl` configured for the k3s cluster, `helm` 3.x.

```bash
# Build and push image
docker build -t ghcr.io/mahmoudholding/developer-portal:sha-<short> .
docker push ghcr.io/mahmoudholding/developer-portal:sha-<short>

# Deploy via Helm
helm upgrade --install developer-portal helm/developer-portal \
  --namespace developer-portal \
  --create-namespace \
  --set image.tag=sha-<short> \
  --wait
```

The Helm chart creates:
- `Deployment` — 1 replica, nginx:alpine, 100m CPU / 64Mi memory limit
- `Service` — ClusterIP on port 80
- `Ingress` — `developer.glorylabs.nl`, TLS via cert-manager (`letsencrypt-prod`)

---

## Project tracking

Issues: [mahmoudholding/auditPic#34](https://github.com/mahmoudholding/auditPic/issues/34)
