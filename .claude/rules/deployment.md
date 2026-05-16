# Deployment & runtime

This fleet does **not** run on cloud-hosted backends or third-party platforms. Internalize this before proposing changes.

## What's actually wired

| Component                          | Runtime                                                            | Ship                                              |
|------------------------------------|--------------------------------------------------------------------|---------------------------------------------------|
| Plate (operator UI)                | Cloudflare Worker `edible-factor-plate`                            | Cloudflare dashboard Git → push to `main` (wired) |
| Web (consumer, OpenNext)           | Cloudflare Worker `edible-factor-web`                              | Cloudflare dashboard Git → push to `main` (TODO)  |
| Waitlist (this repo, apex)         | Cloudflare Pages `edible-factor-waitlist`                          | Pages Git → push to `main` (wired)                |
| Backend API                        | Local Go binary on Abhi's Mac/Windows laptop                       | `make prod` (laptop-side)                         |
| MongoDB                            | Local Docker on Abhi's laptop                                      | `docker compose up`                               |
| Tunnel                             | `cloudflared` named tunnel → `api.ediblefactor.com` / `api-dev.*`  | Started alongside `make prod`                     |

## Constraints

1. **Pages, not Workers.** This is the only repo in the fleet on Cloudflare **Pages**. Sibling UIs (`plate`, `web`) are on Cloudflare Workers. Don't conflate — `wrangler pages deploy` vs `wrangler deploy` are different shapes.
2. **Don't add a GitHub Actions deploy workflow.** The Pages Git integration handles it; a CI workflow would race.
3. **Don't restore Vercel or Supabase.** Both removed across the fleet.
4. **Don't propose cloud hosting for the backend** — the Go API runs on Abhi's laptop behind a tunnel. Not relevant for this repo directly (waitlist is static), but useful context if you're reasoning about why a sibling app's API call is failing.

## Full topology

See [`docs/ARCHITECTURE.md`](../../docs/ARCHITECTURE.md).
