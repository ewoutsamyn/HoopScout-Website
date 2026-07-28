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
  `https://hoop-scout.com/privacy.html`. It's a tailored draft — review it
  before relying on it legally.

## Custom domain

The site is served at **https://hoop-scout.com** (GitHub Pages custom domain).
The apex `hoop-scout.com` is canonical; `www` redirects to it. The domain is set
by the root `CNAME` file (`hoop-scout.com`) plus the Pages setting; DNS lives at
the Squarespace registrar (four A records → GitHub's `185.199.108–111.153`, and
`www` CNAME → `ewoutsamyn.github.io`). Don't delete the `CNAME` file or the
custom domain drops on the next deploy.
- **Contact:** the footer "Contact" link and the policy use
  `info.hooptrax@gmail.com`. Change it in `index.html`, `privacy.html` and the
  policy body if that address changes.
- **Instagram:** the footer button links to
  `https://www.instagram.com/hooptrax/` (update in both `index.html`
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
