# Artha Scheduling — white-label + gated auto-update kit

Artha Scheduling is a thin white-label fork of [Cal.com](https://github.com/calcom/cal.com)
that tracks upstream and ships under the Artha brand. This directory is the
**set-and-forget auto-update kit**, identical in spirit to the kits used by the
other Artha modules.

## Files

| File | Purpose |
|---|---|
| `update.sh` | Weekly orchestrator: fetch upstream → trial-merge into the `artha` branch → `verify-rebrand.sh` → merge. **Aborts before any deploy on conflict / branding drift — production is never touched on failure.** |
| `verify-rebrand.sh` | Universal gate. Reads `deploy.conf` and asserts the Artha brand strings are present and no upstream brand leaked back into the user-facing locale catalogue. Exit 1 stops the pipeline. |
| `deploy.conf` | Per-module config (module name, upstream/fork branches, Scalingo app, live-URL probe, rebrand assertions). |
| `UPSTREAM_URL` | Upstream git URL wired as the `upstream` remote by the workflow. |
| `VERSION` | The upstream Cal.com SHA currently tracked. Pinned automatically after each successful update. |

## Deploy model

Cal.com is the heaviest Next.js monorepo in the suite and does not build reliably
on GitHub's free 2-core/7GB runners. So this module is **gate-only**
(`DEPLOY_METHOD=none`): the free weekly workflow proves the next upstream is safe
(clean trial-merge + Artha rebrand intact) and pushes the updated `artha` branch.
The actual rebuild/redeploy ships via the **`bytepassperks/artha-cal-buildpack`**
on Scalingo (covered by the account's Scalingo credits, not Devin/GitHub minutes),
triggered after a green gate. This keeps the pipeline **$0 and Devin-free** while
never risking production on a bad merge.

## Brand layer (what `verify-rebrand.sh` guards)

- `packages/i18n/locales/*/common.json` — all user-visible UI strings rebranded to **Artha Scheduling**
- `apps/web/public/*.svg` — Cal logo assets → Artha mark
- `apps/web/modules/auth/login-view.tsx`, `signup-view.tsx` — auth screens → Artha
- HTML `<title>` / metadata → **Artha Scheduling**

## Running an update manually

```bash
GH_TOKEN=… deploy/artha/update.sh --dry-run    # merge + verify only, no deploy
```

Requires push rights to `bytepassperks/artha-scheduling`.
