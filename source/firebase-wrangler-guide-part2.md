# Part 2 — Wrangler / Cloudflare

## 2.1 Install & Auth

```bash
npm install -g wrangler
# or per-project (usually better):
npm install -D wrangler
npx wrangler --version

wrangler login
```

Wrangler v4 has been current since March 2025. It supports the Current, Active, and Maintenance versions of Node.js. On Linux it needs a distro with glibc 2.35+ — fine on CachyOS.

## 2.2 Configuration

Configured via `wrangler.jsonc` (recommended), `wrangler.json`, or `wrangler.toml` in the project root. JSONC is the newer recommendation; TOML still works.

```jsonc
// wrangler.jsonc
{
  "name": "my-worker",
  "main": "src/index.js",
  "compatibility_date": "2026-09-01"
}
```

## 2.3 Core Commands

| Command | What it does |
|---|---|
| `wrangler dev` | Local dev server |
| `wrangler deploy` | Deploy a Worker |
| `wrangler pages deploy <dir>` | Deploy a Pages site |
| `wrangler tail` | Live-stream production logs |
| `wrangler secret put <NAME>` | Set an encrypted secret |
| `wrangler kv` | KV namespace operations |
| `wrangler r2` | R2 bucket operations |
| `wrangler d1` | D1 database operations |

## 2.4 Pages Deploys

```bash
wrangler pages deploy ./dist
wrangler pages deploy ./dist --project-name my-site
wrangler pages deploy ./dist --branch preview
```

### The production-branch gotcha

`wrangler pages deploy` has **no `--production` flag** — it never had one, in any version. Whether a deploy counts as production is decided by comparing your `--branch` value against the project's configured production branch in Cloudflare. Match the branch name and it's a production deploy; anything else is a preview.

### Deleting deployments
```bash
wrangler pages deployment delete <deployment-id> --project-name <name>
wrangler pages deployment delete <deployment-id> --project-name <name> --force
```
`--force` (or `-f`) skips the confirmation prompt — needed for CI.

### Recent behavior change worth knowing
As of Wrangler 4.108.0, when `wrangler pages deploy` or `pages project create` is run **by an AI coding agent** against a brand-new, purely static project, Wrangler delegates it to Workers static assets instead of Cloudflare Pages. Accounts with existing Pages projects, human (non-agent) sessions, and projects using Pages Functions, a `_worker.js`, or a `_routes.json` are unaffected. Pass `--force` to opt out and deploy to Pages directly.

Relevant if you use Claude Code on your Cloudflare projects — deploys you run yourself in the terminal behave normally.

## 2.5 Secrets & Environment

```bash
wrangler secret put API_KEY          # prompts for value, encrypted
wrangler secret list
wrangler secret delete API_KEY
```

Non-secret vars go in `wrangler.jsonc` under `vars`. Secrets never go in the config file.

## 2.6 GitHub Actions

```yaml
- name: Deploy
  id: deploy
  uses: cloudflare/wrangler-action@v4
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    command: pages deploy --project-name=example
```

Version notes for the action:
- `wrangler-action@v4` defaults to Wrangler CLI v4. No input changes were needed for existing `pages deploy` usage when upgrading from v3.
- Pin to v3 if you need it: `wranglerVersion: "3.90.0"`.
- The `v` prefix is now **required** — `cloudflare/wrangler-action@3.x.x` is no longer valid syntax; use `@v4`, `@v4.x`, or `@v4.x.x`.

### Useful outputs
```yaml
- name: print deployment URL
  env:
    DEPLOYMENT_URL: ${{ steps.deploy.outputs.deployment-url }}
  run: echo $DEPLOYMENT_URL
```

Also available: `command-output` and `command-stderr` for parsing full Wrangler output in later steps.
