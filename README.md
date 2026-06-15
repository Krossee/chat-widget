# Krossee Live Chat Widget

Static HTML shell that embeds the HubSpot Conversations widget for the Krossee
mobile app. The iOS app opens this URL in `SFSafariViewController` because
WKWebView can't focus inputs inside HubSpot's cross-origin iframe.

## What this is

A single file (`index.html`) that:

1. Reads `?email=…` and `?nag_id=…` (legacy) / `?user_id=…` (forward-looking)
   from the URL.
2. Identifies the visitor to HubSpot Conversations via `_hsq.push(['identify', …])`.
3. Loads HubSpot's tracking script for Krossee portal `343359100`.
4. Shows a "Connecting…" placeholder until the widget initializes, then hides it.

No build step, no framework, no dependencies. Just upload and serve.

## Where this needs to live

iOS calls this URL (see `ios/NAG/iNAG/LiveChatView.swift`):

```
https://chat.krossee.com/?email=…&nag_id=…
```

So `chat.krossee.com` must serve `index.html` at its root.

## Deployment (one-time setup)

### Option A — GitHub Pages (simplest, free)

1. Create a public repo: `krossee/chat-widget` (or whatever org you settle on).
2. Push only `index.html` to the repo's default branch.
3. Settings → Pages → Source = Deploy from a branch → Branch = `main`, Folder = `/ (root)`.
4. Settings → Pages → Custom domain → enter `chat.krossee.com` → Save.
5. Add a DNS record at the krossee.com registrar:
   - Type: `CNAME`
   - Name: `chat`
   - Value: `<github-username-or-org>.github.io`
   - TTL: 300
6. Wait for DNS to propagate (~5-30 min), then check "Enforce HTTPS" in GitHub Pages settings.

### Option B — Cloudflare Pages (also free, faster CDN)

1. Push `index.html` to any GitHub repo (public or private).
2. Cloudflare dashboard → Pages → Create a project → connect the repo.
3. Build settings: framework `None`, build command empty, output dir `/`.
4. Custom domains → add `chat.krossee.com` → Cloudflare auto-creates the DNS
   record if `krossee.com` is on Cloudflare DNS; otherwise it gives you a CNAME
   target to add at your registrar.

### Option C — Vercel

Same shape as Cloudflare Pages — connect repo, configure custom domain.

## Verification

After deployment, open in a browser:

```
https://chat.krossee.com/?email=test@example.com&nag_id=TEST-001
```

Should see the "Connecting you to Krossee Support…" loader briefly, then the
HubSpot Conversations widget panel sliding in from the bottom-right (or filling
the page on mobile).

Inspect the page source — must contain `js.hs-scripts.com/343359100.js`. If you
see `343305473`, the build is stale.

## Portal ID rotation

If the HubSpot portal ID ever changes again, only this file needs updating:
swap the digits in the `<script src="https://js.hs-scripts.com/{portal}.js">`
line and redeploy.

The iOS app does NOT have the portal ID hardcoded for the chat path — it just
opens `https://chat.krossee.com/`, so portal swaps are zero-iOS-change events.
