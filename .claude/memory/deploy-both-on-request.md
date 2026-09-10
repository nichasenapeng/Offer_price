---
name: deploy-both-on-request
description: "Deploy to both GitHub Pages and Cloudflare, but only when the user asks — never automatically after an edit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 28d1bc27-8254-4081-8df7-47d4469f23ab
  modified: 2026-09-07T15:39:42.493Z
---

When the user asks to deploy, ship to **both** targets, not just one:
1. `git push` to `nichasenapeng/Offer_price` (GitHub Pages, auto-builds)
2. `npx wrangler deploy` to the Cloudflare Worker `sunfood-offer` on nicha's account

Do **not** deploy after every edit — wait for the user to say so. Editing and
verifying locally is the default; publishing is a separate, explicit step.

**Why:** the two sites drifted badly once — Cloudflare sat on the first-day
build for weeks while GitHub Pages was current, and the user only noticed when
a feature "didn't exist" on the link they were using. But the user also does not
want every small change published, so deployment stays on request.

**How to apply:** on "deploy" (or similar), push and run wrangler, then verify
both URLs serve the same file before reporting done. See [[cloudflare-deploy-setup]].
