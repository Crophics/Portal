# Part 1 — Firebase CLI

## 1.1 Install & Auth

```bash
npm install -g firebase-tools
firebase --version
firebase login
```

For a headless/SSH machine where the browser flow won't work:
```bash
firebase login --no-localhost
```
That gives you a code to copy-paste instead of spinning up a local auth server.

Help is well-structured — use it:
```bash
firebase --help              # list all commands
firebase deploy --help       # details for one command
```

## 1.2 Project Init

```bash
firebase init
```

Walks you through which features to configure — Hosting, Functions, Firestore Rules, Realtime Database Rules, Storage Rules, Emulators — and whether to attach to an existing Firebase project or create one.

Produces two files:

| File | Purpose |
|---|---|
| `firebase.json` | Deployment config — what gets deployed and how |
| `.firebaserc` | Project aliases (staging/prod mapping) |

You can re-run `firebase init` later to add features to an existing project.

## 1.3 `firebase.json`

Controls the deploy. For Hosting it sets the public directory, rewrites, headers, and redirects; for Functions it points at the functions directory.

```json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  }
}
```

That `rewrites` block with `"source": "**"` is the standard SPA catch-all — every route serves `index.html` so client-side routing works on refresh.

## 1.4 Deploying

```bash
firebase deploy
```

Deploys everything in `firebase.json`.

### Partial deploys — use these
```bash
firebase deploy --only hosting
firebase deploy --only functions
firebase deploy --only firestore:rules
firebase deploy --only hosting,functions     # comma-separated, no space
firebase deploy --only functions:myFunction  # a single function
```

The `--only` flag is the one you'll use most — full deploys are slow and redeploy things you didn't touch.

Add a message for the deploy history:
```bash
firebase deploy -m "fix auth redirect"
```

### Preview channels
Temporary URLs for testing before going live:
```bash
firebase hosting:channel:deploy preview-name
firebase hosting:channel:list
firebase hosting:channel:delete preview-name
```

Note: for preview-channel commands you **don't** pass `hosting` in `--only` — those commands only ever deploy Hosting content and config.

## 1.5 Local Development

```bash
firebase serve                    # local web server using firebase.json
firebase serve --only hosting
firebase emulators:start          # full emulator suite
```

The emulator suite is the better option once you have Functions/Firestore — it runs them locally instead of hitting production.

## 1.6 Multiple Sites / Environments

Deploy targets are the recommended way to handle multiple Hosting sites:

```bash
firebase target:apply hosting TARGET_NAME SITE_ID
```

Then reference `TARGET_NAME` in `firebase.json` instead of hardcoding the site ID. If you previously wrote `firebase.json` referencing `SITE_ID` directly, Firebase's own docs recommend migrating to targets.

Project aliases live in `.firebaserc`:
```bash
firebase use --add            # add an alias interactively
firebase use staging          # switch active project
firebase use                  # show current
```

## 1.7 Other Useful Commands

| Command | What it does |
|---|---|
| `firebase projects:list` | List your projects |
| `firebase open` | Open the project in the console/browser |
| `firebase hosting:rollback` | Roll back to previous deploy |
| `firebase functions:log` | Tail function logs |
| `firebase auth:export users.json` | Export auth users |
| `firebase auth:import users.json` | Import auth users |

Rollback caveat: Hosting deploys roll back cleanly, but **rules do not roll back**. Redeploy the old rules explicitly if you need to revert them.
