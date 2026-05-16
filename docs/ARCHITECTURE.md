# Deployment & runtime architecture

> **TL;DR:** This is the apex waitlist site at `ediblefactor.com`, hosted on **Cloudflare Pages** (project `edible-factor-waitlist`). Auto-deploys from `main` via Pages' built-in Git integration. The rest of the fleet (operator dashboard, consumer app, backend) lives on Cloudflare Workers or a laptop tunnel — see "Topology" below. Vercel and Supabase have been removed across the fleet.

## Topology

```
              ediblefactor.com (Cloudflare DNS)
                          │
   ┌────────┬─────────────┼──────────────┬─────────────┐
   │        │             │              │             │
 plate.*  app.*    api.* / api-dev.*    apex       (others)
   │        │             │              │
 Worker   Worker      cloudflared      Pages
(plate) (web,OpenNext)    │           ← THIS REPO
                          ▼
                cloudflared on Abhi's laptop
                          │
                          ▼
              edible-factor-backend (Go @ :9091 prod / :9090 dev)
                  + MongoDB (local Docker)
```

## What runs where

| Component        | Repo                  | Runtime                                | Ship                              |
|------------------|-----------------------|----------------------------------------|-----------------------------------|
| Plate operator   | edible-factor-plate   | Worker `edible-factor-plate`           | Dashboard Git → main push (wired) |
| Web consumer     | edible-factor-web     | Worker `edible-factor-web` (OpenNext)  | Dashboard Git → main push (**TODO**) |
| Waitlist (apex)  | **THIS REPO**         | Pages `edible-factor-waitlist`         | Pages Git → main push (wired)     |
| Backend API      | edible-factor-backend | Local Go binary on Abhi's laptop       | `make prod` (laptop-side)         |
| MongoDB          | backend repo          | Local Docker on Abhi's laptop          | `docker compose up`               |

## This repo's domains

| Host                                  | Backed by                                          |
|---------------------------------------|----------------------------------------------------|
| `ediblefactor.com` (apex)             | Pages project `edible-factor-waitlist`              |
| `edible-factor-waitlist.pages.dev`    | Same Pages project (preview/default Pages URL)      |

The Pages project has the GitHub integration enabled — pushes to `main` build and deploy automatically. Confirmed via `npx wrangler pages project list` (last-modified timestamp keeps moving with merges).

## Implications for AI agents

- **Pages, not Workers.** This is the only repo in the fleet on Cloudflare *Pages*. The sibling UIs (`plate`, `web`) are on Cloudflare *Workers*. Don't conflate the two — the dashboard surfaces are different and so are the deploy command shapes (`wrangler pages deploy` vs `wrangler deploy`).
- **Auto-deploy is wired** via Pages' Git integration. Don't add a GitHub Actions deploy workflow — it would race.
- **No cloud backend.** The Go backend the rest of the fleet talks to runs on Abhi's laptop. Not relevant for this repo directly (waitlist is static), but useful context if you find yourself reasoning about why an API call from a sibling app is failing.
- **No Vercel, no Supabase.** Both removed across the fleet.

## Build / preview

| Command                           | Purpose                                       |
|-----------------------------------|-----------------------------------------------|
| `npm run dev`                     | Local dev server                              |
| `npm run build`                   | Production build                              |
| Push to `main`                    | Auto-deploy via Pages Git integration         |
