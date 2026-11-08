# My Coworkers deployment

Private copy of [fayazara/coworkers](https://github.com/fayazara/coworkers).
Only `.github/workflows/` and this file differ from upstream.

## How updates work

| Workflow | When | What |
|---|---|---|
| `sync-upstream.yml` | Daily 10:17 UTC, or **Run workflow** | Merges upstream `main`. If anything changed, starts a deploy. Fails loudly on a merge conflict. |
| `deploy.yml` | Push to `main`, after a sync, or **Run workflow** | Installs, runs tests, deploys. Skipped until `DEPLOY_ENABLED=true`. A failed test blocks the deploy. |

## One-time setup

1. **Cloudflare:** Workers Paid plan on the account.
2. **First deploy, from your own machine** (creates the AI Gateway and Cloudflare Access):
   ```bash
   git clone https://github.com/<you>/coworkers && cd coworkers
   npm i -g cf && pnpm install
   cf auth login
   pnpm run setup
   ```
   This writes `coworkers.deploy.json` and `.dev.vars.production` (both git-ignored).
3. **API token:** Cloudflare dashboard → My Profile → API Tokens → Create.
   Permissions: *Account · Workers Scripts · Edit*, *Account · Workers AI · Read*,
   *Account · Browser Rendering · Edit*, *Account · AI Gateway · Read*.
   Scope it to this one account.
4. **GitHub → Settings → Environments → New environment `production`**, add secrets:

   | Secret | Value |
   |---|---|
   | `CLOUDFLARE_API_TOKEN` | the token from step 3 |
   | `CLOUDFLARE_ACCOUNT_ID` | your account ID |
   | `COWORKERS_DEPLOY_JSON` | full contents of `coworkers.deploy.json` |
   | `DEV_VARS_PRODUCTION` | full contents of `.dev.vars.production` |

5. **Settings → Secrets and variables → Actions → Variables:** `DEPLOY_ENABLED` = `true`.
6. **Actions → Deploy to Cloudflare → Run workflow** to confirm it works.

Changing who can sign in or the model: run `pnpm run setup` locally again,
then paste the new `coworkers.deploy.json` into the secret.

To pause auto-deploys: set `DEPLOY_ENABLED` to `false`. Syncs keep running.
