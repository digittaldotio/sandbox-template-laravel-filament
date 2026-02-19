# Sandbox Template

Deploy a Laravel + Filament app to `*.sandbox.digittal.mobi` by pushing to GitHub.

## Quick Start

1. Click **"Use this template"** (green button above) to create your repo
2. Add the GitHub topic `cco-sandbox` to your new repo (Settings > About > Topics)
3. Write your Laravel app
4. Push to `main`
5. Your app appears at `https://<your-repo-name>.sandbox.digittal.mobi`

## What's Included

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage build: Node assets → Composer deps → PHP 8.3-FPM + nginx |
| `Jenkinsfile` | Auto-builds, pushes to registry, deploys to Kubernetes |
| `docker/` | nginx, PHP, supervisor configs |
| `deploy/k8s/` | Kubernetes Deployment, Service, Ingress templates |

## You Don't Need To

- Access Docker registries
- Run kubectl commands
- Manage certificates or DNS
- SSH into any server
- Edit the Jenkinsfile, Dockerfile, or k8s manifests

## Database

By default, your app uses **SQLite** (zero config). To switch to MySQL, edit `deploy/k8s/deployment.yaml` — see the [sandbox setup guide](https://github.com/digittaldotio/sandbox-template/wiki) for details.

## Local Development

```bash
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm run dev
php artisan serve
```
