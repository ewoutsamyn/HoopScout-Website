# HoopTrax — Website

The marketing / landing page for **HoopTrax**, the courtside basketball
stats app. Static HTML/CSS/JS — no build step, no dependencies.

## Run it

Just open `index.html` in any browser (double-click it), or serve the
folder with any static server, e.g.:

```bash
npx serve .
# or
python -m http.server 8000
```

## Deploy it

It's a plain static site — drop the whole folder on any static host:

- **Netlify / Vercel:** drag-and-drop the folder, or point it at the repo.
- **GitHub Pages:** push the folder and enable Pages.
- **Cloudflare Pages / any web host:** upload as-is.

No environment variables, no server code.

## Add your real screenshots

The page ships with built-in placeholder graphics so it looks complete
right now. To swap in real app screenshots, drop PNGs into
`assets/screenshots/` using these exact names:

| File | Where it shows |
|------|----------------|
| `hero.png` | Hero phone at the top |
| `live-tracking.png` | "Live tracking" spotlight section |
| `shot-zones.png` | "Shot zones" spotlight section |

No code editing needed — if the PNG exists it's used automatically;
if not, the matching placeholder SVG is shown. See
`assets/screenshots/README.txt` for sizes and capture tips.

## Wire up the store links at launch

Right now the App Store / Google Play badges are non-clickable
"Coming soon" markers (`<span class="store-badge">`). When the app is
live, turn each into a link:

1. In `index.html`, change the badge `<span class="store-badge" …>` to
   `<a class="store-badge" href="YOUR_STORE_URL" target="_blank" rel="noopener">`.
2. Update the `aria-label` / small text from "Coming soon" to "Download
   on the".
3. Optionally remove the `.chip--soon` "Coming soon" chip in the nav and
   the closing CTA copy.

There are two badge pairs to update: the **hero** and the **closing CTA**.

## Privacy policy, contact & Instagram

- **Privacy policy:** `privacy.html` is a GDPR / Google AdMob-ready policy,
  linked from the footer. Its public URL (needed in the AdMob console) is
  `https://hoop-trax.com/privacy.html`. It's a tailored draft — review it
  before relying on it legally. **This exact URL is registered in the Google
  AdMob console** — if the domain changes again, update it there too, or the
  link AdMob points at goes dead with no warning from anywhere in this repo.

## Custom domain

The site is served at **https://hoop-trax.com** (GitHub Pages custom domain).
The apex `hoop-trax.com` is canonical; `www` redirects to it. Keep the root
`CNAME` file — deploying without it drops the custom domain.

**The `CNAME` file alone does NOT set the domain here.** This repo deploys via
GitHub Actions (`build_type: workflow`), and that path does *not* read `CNAME`
back into the Pages custom-domain setting — only legacy branch-based builds do.
Changing the domain therefore takes two steps, and doing just the first leaves
the new domain serving GitHub's "Site not found":

1. commit the new value to `CNAME`, and
2. set it in **Settings → Pages → Custom domain** (or
   `gh api -X PUT repos/ewoutsamyn/HoopTrax-Website/pages -f cname=<domain>`).

Then **re-run the deploy workflow**. Immediately after a domain change the root
path can keep returning "Site not found" while `/index.html` and `/privacy.html`
serve correctly — a fresh deploy fixes the root routing. That symptom is a
routing artifact, not a broken deploy.

**The host is GitHub Pages. Cloudflare is only a proxy in front of it** — DNS
plus CDN. Don't reach for Cloudflare Pages / "Workers & Pages"; adding the domain
there fails with *"hostname already has externally managed DNS records"*, because
the records already point at GitHub. Every response header says
`Server: cloudflare`, which makes it look Cloudflare-hosted; it isn't, and a
GitHub Actions deploy is what publishes changes.

**DNS runs through Cloudflare, not the registrar.** The domain is *registered*
at Squarespace, but its nameservers are delegated to Cloudflare
(`dante.ns.cloudflare.com` / `stephane.ns.cloudflare.com`), and the DNS records
live in the Cloudflare dashboard:

- four apex `A` records → GitHub's `185.199.108.153`, `185.199.109.153`,
  `185.199.110.153`, `185.199.111.153`
- `www` `CNAME` → `ewoutsamyn.github.io` (account-scoped, so it is the same
  value for any domain on this account)

Two consequences of the Cloudflare layer that cost real debugging time on the
old hoop-scout.com domain:

1. **While a record is proxied (orange cloud), GitHub cannot issue its own TLS
   certificate** — Cloudflare intercepts the validation challenge, so Pages
   reports no certificate and "Enforce HTTPS" stays greyed out. To get a real
   GitHub certificate, set the records to **DNS only** (grey cloud) until Pages
   issues one and Enforce HTTPS is on, *then* switch to proxied with the
   Cloudflare SSL/TLS mode set to **Full (strict)**. Never use **Flexible** — it
   sends plain HTTP to an origin that redirects to HTTPS, producing a redirect
   loop.
2. **A proxied Cloudflare zone can serve stale HTML after a deploy.** Cloudflare
   does not cache HTML by default, so if a deploy succeeds but the live page is
   unchanged, look for a Cache Rule or Page Rule doing "Cache Everything" and
   purge the cache. Confirm the deploy itself worked by checking
   `https://ewoutsamyn.github.io/HoopScout-Website/`, which bypasses Cloudflare
   entirely.
- **Contact:** the footer "Contact" link and the policy use
  `bballsidelineiq@gmail.com`. Change it in `index.html`, `privacy.html` and the
  policy body if that address changes.
- **Instagram:** the footer button links to
  `https://www.instagram.com/hooptraxofficial/` (update in both `index.html`
  and `privacy.html` if the handle changes).

## Structure

```
index.html            The landing page (single file, semantic sections)
privacy.html          Privacy policy page (shares nav/footer + styles.css)
css/styles.css        Design tokens + all styling (dark default + light theme)
js/main.js            Scroll reveals, count-ups, parallax, theme toggle
assets/
  logo.svg            Brand mark
  favicon.svg         Tab / touch icon
  og-image.svg        Social share image
  screenshots/        hero.png + shot-zones.png (real) + SVG placeholders
```

## Design notes

- Colours, type and the logo mirror the HoopTrax app's theme
  (`constants/theme.ts`): near-black dark palette, orange accent
  `#F2790A`, Barlow Semi Condensed headings, Poppins wordmark.
- A **dark / light theme toggle** sits in the nav (dark is default); the
  choice is remembered in `localStorage`.
- Animations are CSS + a little vanilla JS, and fully disable under
  `prefers-reduced-motion`.
- Fonts load from Google Fonts; everything else is local, so the site
  works offline once cached.
