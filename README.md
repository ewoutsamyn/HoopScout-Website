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

**Push to `main`.** The site is hosted on a Cloudflare Worker that is connected
to this repo and rebuilds itself on every push (see [Hosting](#hosting) below).
There's nothing to run and no environment variables.

Locally it's still a plain static site, so any static server (or double-clicking
`index.html`) previews it exactly as it ships.

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

## Hosting

The site runs on a **Cloudflare Worker** (`hoopscoutwebsite` — an account-legacy
name, not the product name) with **Cloudflare Workers Builds** connected to this
GitHub repo. Every push to `main` makes Cloudflare clone, build and deploy — the
repo is static HTML with no build config, so the build settings live in the
Cloudflare dashboard. Served at **https://hoop-trax.com** (apex canonical; `www`
serves the same site). hoop-scout.com is a second Cloudflare zone pointing at the
same Worker.

There is **no GitHub Pages, no GitHub Actions workflow, and no `CNAME` file** —
they were removed when the site moved fully onto the Worker. Don't add them back;
a second deploy pipeline building the same repo is the clutter that removal
cleared.

**To attach or change a domain**, use the Worker's Domains tab — *not* a `CNAME`
file or hand-edited DNS records: Cloudflare dashboard → Workers & Pages →
`hoopscoutwebsite` → **Domains** → Add Domain. Cloudflare creates the routing
records itself. If it errors with *"hostname already has externally managed DNS
records"*, delete that hostname's existing A/CNAME records from the zone's DNS
first, then add it here.

**DNS** is on Cloudflare — the domain is registered at Squarespace but its
nameservers are delegated to Cloudflare (`dante` / `stephane.ns.cloudflare.com`).
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
