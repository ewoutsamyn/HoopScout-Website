# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in the **HoopTrax Website** repo.

## What this is

The marketing / landing page for **HoopTrax**, a single-coach live basketball
game-tracking app (the app itself lives in the sibling `../HoopScout App/` folder —
an Expo/React Native project). This repo is **only the website**, not the app.

It is a **pre-launch** site: the app isn't on the stores yet, so there is **no
download link and no waitlist**. The call-to-action everywhere is a "Coming soon"
cue with non-clickable App Store / Google Play badges.

## Stack & how to run

Plain **static HTML/CSS/JS — no build step, no dependencies, no framework.**

- **Run:** open `index.html` directly (double-click), or serve the folder:
  `npx serve .` — there is no `python` on PATH in this environment, use Node.
- **Deploy:** upload the whole folder to any static host (Netlify, Vercel, GitHub
  Pages, Cloudflare Pages). No env vars, no server code.
- **Correctness check:** there is no test suite or linter. Verify by eye in a
  browser at mobile / tablet / desktop widths, plus a `prefers-reduced-motion` pass.

## File map

```
index.html        Entire landing page — one file, semantic <section>s. Edit copy/structure here.
privacy.html      GDPR/AdMob privacy policy (its own page). Shares nav/footer + styles.css.
css/styles.css    Design tokens (:root) + ALL styling. Dark default + light theme.
js/main.js        Vanilla JS: scroll reveals, count-ups, parallax, theme toggle, year.
                  Its null-guards mean it runs harmlessly on privacy.html too.
assets/
  logo.svg          Brand mark (basketball + scout magnifier).
  favicon.svg       Tab/touch icon (navy rounded square framing the mark).
  og-image.svg      Social share card.
  screenshots/      Real PNGs (hero.png, shot-zones.png) + SVG placeholders + README.txt.
README.md         Human-facing run/deploy/screenshot/launch instructions.
CNAME             GitHub Pages custom domain (`hoop-trax.com`). Do not delete.
```

**Hosting:** deployed to GitHub Pages (public repo, Actions workflow) and served
at **https://hoop-trax.com** — apex canonical, `www` redirects. All asset paths
are relative, so the site works at the domain root and at the `github.io` subpath
alike. Absolute URLs (canonical, `og:url`, `og:image`) point at `hoop-trax.com`.

**Changing the domain takes two steps, not one.** Because this repo builds via
GitHub Actions (`build_type: workflow`), the `CNAME` file is *not* read back into
the Pages custom-domain setting — that only happens on legacy branch-based builds.
So editing `CNAME` alone leaves the new domain answering GitHub's "Site not
found". You must also set it under Settings → Pages, or
`gh api -X PUT repos/ewoutsamyn/HoopTrax-Website/pages -f cname=<domain>`. Keep
the `CNAME` file regardless — deploying without it drops the custom domain.
Afterwards, **re-run the workflow**: right after a domain change the root path can
still 404 while `/index.html` serves fine, and a fresh deploy repairs the root
routing.

**Cloudflare is a proxy in front, NOT the host.** The site is served by **GitHub
Pages**; Cloudflare only does DNS and (orange-cloud) CDN proxying. **Cloudflare
Pages / "Workers & Pages" is not used and must not be** — trying to add
`hoop-trax.com` there fails with *"hostname already has externally managed DNS
records"*, which is correct: those records are the ones pointing at GitHub. The
predecessor domain hoop-scout.com had this exact same arrangement, which is easy
to misread as "hosted on Cloudflare" because every response says
`Server: cloudflare`. It isn't — a GitHub Actions deploy is what changes the
content.

**DNS is on Cloudflare, not the registrar** — the domain is *registered* at
Squarespace but its nameservers are delegated to Cloudflare, where four apex `A`
records point at GitHub's `185.199.108.153`–`185.199.111.153` and `www` is a
`CNAME` to `ewoutsamyn.github.io`. This repo's docs previously claimed DNS lived
at Squarespace directly; it does not, and that error sent one session looking for
records that were not there. Two Cloudflare-specific traps:

- **A deploy can succeed while the live page stays stale**, if a Cache Rule is
  doing "Cache Everything" — Cloudflare does not cache HTML by default, so this
  means a rule exists. Purge the cache. To check whether the deploy itself was
  fine, hit `https://ewoutsamyn.github.io/HoopScout-Website/`, which bypasses
  Cloudflare.
- **A proxied (orange-cloud) record blocks GitHub's TLS certificate issuance**,
  because Cloudflare intercepts the validation challenge — Pages then shows no
  certificate and "Enforce HTTPS" is unavailable. Set records to DNS-only until
  the certificate is issued, then re-proxy with SSL/TLS mode **Full (strict)**.
  Never **Flexible**: it talks plain HTTP to an origin that redirects to HTTPS,
  which loops.

## Brand system (must stay in sync with the app)

Colors, type and the logo are copied from the app's `../HoopScout App/CLAUDE.md`
§Theming and `../HoopScout App/assets/logo/brand-logo.svg`. If the app's brand
changes, update `:root` in `css/styles.css` and the inlined logo SVGs to match.

- **Dark palette (default):** `--bg #0B0D10`, `--surface #141821`, `--border #232833`,
  `--accent #F2790A` (orange), `--accent-2` muted teal, `--navy #0D1B2A`,
  `--accent-text #06171A`.
- **Light palette:** `:root[data-theme="light"]` block — light surfaces, navy text,
  same orange accent.
- **Type:** Barlow Semi Condensed (headings + numerals, `tabular-nums`), Poppins
  (wordmark). Loaded from Google Fonts with a system fallback stack.
- **Positioning (keep it distinct):** the solo sideline coach — courtside-fast
  one-thumb tracking, offline-first, pro-grade insight. Deliberately *not* the
  global/community/live-feed angle of statastic.info. Don't drift toward that.

## Conventions & gotchas (easy to break)

- **The logo SVG is inlined in multiple places** (nav, hero implicit, closing CTA,
  footer) so it can be CSS-animated and theme-tinted. If you change the mark, change
  **every** copy, plus `assets/logo.svg` and `assets/favicon.svg`. There is no single
  source include — it's intentional duplication for animation control.
- **Screenshots use an automatic PNG→SVG fallback.** Each `.phone__screen` `<img>`
  points at a real filename (e.g. `assets/screenshots/live-tracking.png`) with an
  inline `onerror` that swaps to the placeholder SVG. Dropping a correctly-named PNG
  into `assets/screenshots/` makes it appear with **no code edit**. Expected names:
  `hero.png`, `live-tracking.png`, `shot-zones.png` (see `assets/screenshots/README.txt`).
  Keep the `onerror` handler if you add a new screenshot slot. `hero.png`
  (Game Log) and `shot-zones.png` (Shot Zones heatmap) are real, already cleaned;
  `live-tracking.png` is still the placeholder.
- **Clean every real screenshot before using it.** Raw device captures show the
  OS status bar (clock, notification icons, battery) and, from an Expo Go dev
  build, a floating settings-gear button — **both must be removed.** The two
  current shots were processed with Windows PowerShell + `System.Drawing`: sample
  a flat background pixel, `FillRectangle` over the gear, then crop off the top
  status bar and bottom gesture pill and centre-crop to the phone-frame aspect
  **320:692** (so `object-fit: cover` doesn't clip content). Mirror that for any
  new screenshot.
- **Reveal-on-scroll:** elements with class `reveal` start at `opacity:0` and are
  shown when `main.js` adds `.is-visible` via IntersectionObserver. Stagger is set
  with `data-reveal-delay="N"` (× 90ms). If you add content that must animate in, give
  it `class="reveal"`; if it must show immediately (e.g. the nav) do **not**. A
  `<noscript>` block + a reduced-motion fallback both force `.reveal` visible, so
  content never disappears when JS/motion is off — preserve those.
- **Count-up numbers:** `<span data-count="16" data-suffix="%">`. Animated once on
  scroll into view. Purely presentational.
- **Parallax vs. float conflict:** both write `transform`. The hero phone uses the CSS
  `phone--float` keyframe animation and therefore has **no** `data-parallax` (an inline
  transform would kill the float). Static spotlight phones use `data-parallax`. Don't
  add `data-parallax` to a floating element.
- **Theme toggle** persists to `localStorage['hooptrax-theme']`; an inline script in
  `<head>` applies it before paint to avoid a flash. Dark is the default.
- **Store badges are `<span class="store-badge">`, not links, on purpose** (app is
  pre-launch). At launch, convert each to an `<a href>` and update the "Coming soon"
  copy — there are **two** pairs: the hero and the closing CTA. Also remove the
  `.chip--soon` "Coming soon" nav chip. Steps are in `README.md`.
- **No hardcoded hex in new UI** — use the `--token` CSS variables so both themes stay
  correct. Existing intentional exceptions: the brand hexes baked into the inline logo
  SVGs and the placeholder screenshot SVGs.
- **Self-contained:** everything is local except Google Fonts. Don't add CDN scripts,
  frameworks, or a build step — the whole value here is "double-click and it works."

## Scope

Website only. Do not implement any actual basketball-scouting/stat-tracking logic —
that's the app's job. Keep this repo to marketing content, styling, and light
presentational JS.
