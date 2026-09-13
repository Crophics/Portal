# Firebase CLI + Wrangler (Cloudflare) Deploys

Covers both, since you're running Firebase for TaskPlus and Wrangler/Cloudflare Pages for deployment.

```{toctree}
:maxdepth: 1

firebase-wrangler-guide-part1
firebase-wrangler-guide-part2
```

## Quick Comparison

| | Firebase | Cloudflare |
|---|---|---|
| CLI | `firebase-tools` | `wrangler` |
| Config | `firebase.json` + `.firebaserc` | `wrangler.jsonc` |
| Deploy | `firebase deploy --only hosting` | `wrangler pages deploy ./dist` |
| Preview | `firebase hosting:channel:deploy <name>` | `wrangler pages deploy ./dist --branch <name>` |
| Local dev | `firebase emulators:start` | `wrangler dev` |
| Logs | `firebase functions:log` | `wrangler tail` |
| Rollback | `firebase hosting:rollback` | Via dashboard / redeploy |

## Reference

- Firebase CLI reference: https://firebase.google.com/docs/cli
- Firebase Hosting quickstart: https://firebase.google.com/docs/hosting/quickstart
- Firebase multi-site deploy targets: https://firebase.google.com/docs/hosting/multisites
- Wrangler on npm: https://www.npmjs.com/package/wrangler
- wrangler-action: https://github.com/cloudflare/wrangler-action
