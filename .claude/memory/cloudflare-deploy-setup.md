---
name: cloudflare-deploy-setup
description: "How to deploy the SunFood offer app to Cloudflare — account, command, and the git identity switch needed for GitHub"
metadata: 
  node_type: memory
  type: project
  originSessionId: 28d1bc27-8254-4081-8df7-47d4469f23ab
  modified: 2026-09-07T15:39:57.260Z
---

Deploying this app touches two accounts that are **not** the machine's default.

**Cloudflare** — the live Worker is `sunfood-offer` on nicha's account
(`186b05df927d0192ed92ec799dc1b2a0`), serving
https://sunfood-offer.nichasenapeng13.workers.dev

Deploy by copying `index.html` into a scratch folder as `public/index.html` with a
`wrangler.toml` declaring `[assets] directory = "./public"`, then
`CLOUDFLARE_ACCOUNT_ID=186b05df927d0192ed92ec799dc1b2a0 npx wrangler deploy`.

Where the OAuth token sits depends on the machine: `~/.wrangler/config/default.toml`
on the original one, `~/Library/Preferences/.wrangler/config/default.toml` where Node
came from the official macOS installer. Confirm with `npx wrangler whoami` before
deploying — it must print account `186b05df...`, not ai.eng.sunfood.

If the token is missing or expired, `npx wrangler login` — the user approves it in the
browser within **two minutes**, and the browser must already be signed in as nicha,
otherwise the token lands on ai.eng.sunfood, which cannot reach nicha's worker. Do
**not** pipe that command through `tail`/`head`: they buffer until the process exits,
so the authorize link never appears while it is still clickable.

A machine may have no Node at all. Installing it needs the user's password, so ask
them for the macOS LTS installer from nodejs.org rather than trying to do it.

**GitHub** — the repo is `nichasenapeng/Offer_price`. Where a `gh` CLI exists and is
active as `aiengsunfood`, it cannot push: `gh auth switch -u nichasenapeng` → push with
`git -c credential.helper='!gh auth git-credential' push origin main` → switch back.
Where there is no `gh` but the macOS keychain already holds nichasenapeng's credential,
a plain `git push origin main` works and no switching is needed.

**The shell sandbox hides `/usr/local/bin` and blocks the network**, so `gh`, `node`,
`npx` and `curl` all look absent until the sandbox is disabled for those commands.
Check that before concluding a tool is not installed.

Related: [[deploy-both-on-request]]
