# trustno.si

One-page site whose only job is to sell the domain **trustno.si**. The name
reads two ways — `trustno.si` and **Trust No .SI** (`.si` read as "Super
Intelligence"). The scroll animation exists to make a buyer see the second
reading.

- Repo: https://github.com/noustelos/trustnosi
- Live: https://trustno.si/ (Cloudflare Pages, git-connected to `main`)

## Deploy & infrastructure

> ⚠️ **A push to `main` is instantly live on trustno.si.** There is no staging
> step and no build to fail first. Work on a branch, merge deliberately, then
> open the live site and check it.

**Repo** — https://github.com/noustelos/trustnosi, branch `main`.

**Cloudflare Pages** — project `trustnosi`, on Avatar68@gmail.com's Account.
Git-connected to `main`. Framework preset **None**, no build command, output
directory `/`.

**Domains** — all three serve the same Pages project:

| | |
|---|---|
| https://trustno.si | primary |
| https://www.trustno.si | custom domain, redirected to the root |
| https://trustnosi.pages.dev | Pages default |

**Redirect** — a Cloudflare Single Redirect rule, "Redirect from WWW to root":
`https://www.*` → `https://${1}`, **301**. The `<link rel="canonical">` in
`<head>` is `https://trustno.si/`. If the redirect target ever changes, change
the canonical with it — a canonical pointing at a URL that 301s elsewhere is a
bad signal.

**DNS** — zone `trustno.si` on Cloudflare (nameservers delegated from
Namecheap):

| type | name | value | |
|---|---|---|---|
| CNAME | `@` | `trustnosi.pages.dev` | proxied |
| CNAME | `www` | `trustnosi.pages.dev` | proxied |
| TXT | `@` | marketplace ownership verification | Sedo |
| TXT | `@` | marketplace ownership verification | Afternic |

> ⚠️ **Never delete the two TXT records while the domain is listed.** They are
> what proves ownership to Sedo and Afternic; removing one can delist the
> domain. They look like clutter at `@`. They are not.

**Registrar** — Namecheap. Domain bought November 2026.

**Marketplace listings** — both live, Buy Now **$1,490 USD**:

- Afternic — https://www.afternic.com/domain/trustno.si
- Sedo — https://sedo.com/search/details/?domain=trustno.si

The price on the page must match both listings. If it changes on the
marketplaces, update the page in the same session — see **Links** for the
three files that carry it.

**Contact** — `info@noustelos.gr`, and only that. Never an `@trustno.si`
address: that is the domain being sold, and a buyer may take it over.

> ⚠️ **Cloudflare Email Address Obfuscation is OFF for this zone on purpose —
> the Offer CTA must work without JS. Don't re-enable.** Scrape Shield rewrites
> `mailto:` links to `/cdn-cgi/l/email-protection#…` and relies on an injected
> script to decode them, which leaves the Offer CTA dead for anyone without JS
> and keeps the address out of the served HTML entirely. Every anchor on the
> page carries a real `href` so the page works without JS; this is the one CTA
> that is ours rather than a marketplace's. Verify with `curl -s https://trustno.si/ |
> grep -o 'mailto:[^"]*'` — it must return a plain `mailto:`.

**Rollback** — either revert the commit and push, or Cloudflare dashboard →
Pages → `trustnosi` → Deployments → roll back to a previous deployment. The
dashboard route is faster when the live site is broken; the revert is the one
that keeps git honest, so do that too.

## Stack

Single `index.html` with inline CSS and JS. No framework, no build step, no
dependencies. One webfont (Archivo Black, Google Fonts); everything else is a
system stack. Transforms and opacity only — no `backdrop-filter`, no blur
filters, no canvas/WebGL.

Other files: `robots.txt`, `sitemap.xml` (1 URL), `llms.txt`, `og-image.png`
(1200×630). The favicon is an inline SVG data URI.

## The scroll mechanic

Adapted from "Type Takeover" by Matt Cannon (CodePen, MIT) — credit comment
sits at the top of the CSS. Keep it.

A 450vh `.scroll-scene` holds a sticky 100svh `.stage`. A registered
`@property --p` (0→1) is the single clock: every other movement on the page is
a `calc()` off it. Nothing animates on its own.

**JS drives `--p` on every browser**, because a scene wired straight to scroll
position inherits the scroll's steppiness — one mouse-wheel notch is one
discrete jump. Instead `--p` *chases* the scroll: each frame it closes
`SMOOTHING` (0.10) of the remaining distance, which turns those steps into a
glide. The rAF loop parks itself once within `SETTLED` (0.0005) of the target
and is restarted by the next scroll/resize, so idle cost is zero. The
smoothing is applied per millisecond rather than per frame, so a 120Hz display
glides at the same rate as a 60Hz one.

A head script sets `.js-driven` on `<html>` **before first paint**; that class
cancels the `--p` and reveal-gate CSS animations, which would otherwise beat
the inline `--p` in the cascade (and, with no timeline support at all, snap it
to 1). The CSS `view-timeline` path is now only the no-JS fallback — it still
works, and is worth re-checking after any change to the head script.

Each phase has a raw local progress `--t-*` and a smoothstepped `--e-*`
(`t*t*(3-2t)`); **transforms and opacity read the `--e-*` values**, so each
phase leaves and arrives at zero velocity. The phases deliberately overlap, so
nothing hands over at a single point. All of them are registered properties,
so they resolve to a number once per frame instead of being re-substituted as
tokens everywhere they are read.

`--split-gap` is the one that must stay **unregistered**: it has to remain a
token so its `em` resolves against the big word's font-size rather than the
stage's.

Nothing in the scene touches layout. The split is the two outer spans
translating, not a `gap` or `letter-spacing` animation, so the word's box never
changes size — transforms and opacity only. `will-change: transform` sits
permanently on `.word` and its three spans (`opacity` too on `.word`), and
must stay that way. It used to be toggled through a `.stage.moving` class
while `0 < --p < 1`; that was a bug. `will-change` promotes an element to a
compositor layer, and a layer created part-way through the scene is
rasterised against whatever transform it is wearing at that moment — so the
same `--p` rendered differently depending on which direction you had arrived
from (8.4% of the viewport at `--p` 0.65, every letterform edge off by a
subpixel below that). Three layers held for the visit is the cheaper trade.

`--si-half` is measured in JS as half the width of `.si` in `em`, so the
takeover's vanishing point lands on `.si` and stays there as the split opens
and on resize. Without JS it falls back to `0.46em`.

### Sequence (all driven by `--p`)

Ranges overlap on purpose — read them as crossfades, not as cuts.

| range | what happens |
|---|---|
| 0.00–0.14 | Intro: mono label + "One name. Two readings.", fading out. Big word `trustno.si` centred, `.si` already red. |
| 0.10–0.40 | The split: `trust` travels left and `.si` right; caption "trust · no · SI" fades in (0.14–0.26) and out (0.28–0.40). |
| 0.32–0.46 | The word slides so `.si` sits dead centre, overlapping the start of the takeover. |
| 0.36–0.78 | Takeover: the word scales 1→14× around the centre of `.si`, fading out over 0.54–0.76. |
| 0.70–1.00 | Reveal: "TRUST NO .SI", sub-line, then the price line, three CTAs and the escrow note (the `.offer` block, on its own `--t-cta` ramp from 0.80). |

The reveal becomes visible at 0.70 and clickable at 0.88 — in the JS path from
`apply()`, in the no-JS path from the `reveal-gate` keyframes. Keep the two in
step if you move the reveal.

A 4px progress line (`scaleX(--p)`) sits at the bottom; header and footer are
pinned inside the stage.

## Palette rule

```
--bg #0B0B0C · --ink #EDEDE6 · --dim #8A8A85
--signal #00E5FF · --signal-hi #33EBFF (hover) · --on-signal #04191C (text on cyan)
```

**`--signal` has exactly three uses: the `.si` of the big word, the `.SI` of
the reveal headline, and the Afternic button. Nowhere else** — the Sedo and
offer buttons are `--ink` outlines — not on the sub-line, not on the progress
bar, and **not on focus rings**, which stay `--ink` so the accent keeps its
exclusivity. The restraint is the design.

(The reveal headline was `--ink` originally; Nick moved the accent onto its
`.SI` when the headline became "TRUST NO .SI", so the cyan now lands on the
same two letters in both the big word and the payoff.)

Never put off-white on the cyan: `--ink` on `--signal` is about 1.1:1. Text on
the cyan fill is `--on-signal` — 11.8:1 on `--signal`, 12.5:1 on
`--signal-hi`. The cyan itself is 12.8:1 on `--bg`.

Changing the accent again means touching all of: `--signal`, `--signal-hi`,
`--on-signal`, the inline SVG favicon data URI in `<head>`, and
`og-image.png`.

Display type: Archivo Black, tracking ≈ -0.06em. Labels: monospace, 11px,
+1.5px tracking, uppercase.

No X-Files references, logos or fonts. The phrase is generic; the franchise is
not.

## Honesty rules

- No traffic, revenue, valuation or "appraised at" claims.
- No fake scarcity, countdowns or "X people are viewing this".
- No market statistics on the page. If one is ever added it must link its
  primary source.
- The price shown is the real marketplace price (**$1,490** Buy Now). See the
  Links section — it must match both listings.
- Checkout and escrow are the marketplace's, not ours. The note under the
  buttons says so; don't imply this page takes payment.

## Links

The marketplace URLs themselves are in **Deploy & infrastructure**. What
matters here is that each one lives in *two* places in `index.html`: as an
`AFTERNIC_URL` / `SEDO_URL` constant at the top of the script, and as a real
`href` on the anchor so the page still works without JS. Change both.

The three CTAs in the reveal:

- **Buy on Afternic** — primary, the only `--signal` fill on the page.
- **Buy on Sedo** — `--ink` outline.
- **Make an offer** — `--ink` outline, `mailto:info@noustelos.gr`.

Both marketplace links open in a new tab (`target="_blank" rel="noopener"`).
Bottom-left credit → https://noustelos.gr.

**The price appears in three files** — the `.price` line in `index.html`,
`llms.txt`, and this one. If a listing's price changes, change all three in
the same commit, and never let the two listings disagree with each other.

## Accessibility

- `prefers-reduced-motion: reduce` → no scroll scene (100svh), `--p: 1`, the
  reveal and CTAs show immediately, the big word and progress bar are hidden.
- The big word is decorative (`aria-hidden`). The stage has an `aria-label`,
  and the real text for screen readers is the visually-hidden `h1`:
  "trustno.si — Trust No .SI. Domain for sale."
- Focus-visible outlines on every link and button.
- ≤600px: the header mono label is hidden and the CTAs stack; the word stays
  on one line at every width.

## og-image.png

1200×630, regenerated rather than hand-edited, so it always matches the
palette:

```
chrome --headless --disable-gpu --screenshot=og-image.png \
  --window-size=1200,630 --virtual-time-budget=4000 og.html
```

`og.html` is a throwaway page (black bg, Archivo Black from Google Fonts,
"TRUST NO .SI" with ".SI" in `--signal`, "trustno.si — for sale" beneath).
Keep it in sync with the accent colour.

## Checking a change

```
python3 -m http.server 8000
```

Headless Chrome's `--virtual-time-budget` mode **freezes scroll-driven
timelines** (`--p` reads 0 at every scroll position), so screenshots taken
that way cannot verify the no-JS scene. Drive a real browser over CDP
(`--headless=new --remote-debugging-port=9222`) and scroll it, or check in a
normal browser. Always check desktop, 390px, and reduced motion.

Because `--p` now glides, give it ~2s to settle before screenshotting, or you
will capture a frame mid-catch-up.

Worth re-measuring after any change to the scene:

- **Steppiness.** Dispatch discrete `mouseWheel` notches over CDP and record
  `--p` per frame. The largest per-frame change should be several times
  smaller than one raw notch (one notch ≈ 0.025 of the scene).
- **Long frames.** Sample `requestAnimationFrame` timestamps while scrolling
  through the takeover under `Emulation.setCPUThrottlingRate: 6`. There should
  be no frame over 50ms.
- **The no-JS path.** `Emulation.setScriptExecutionDisabled` — `--p` must
  still track scroll off the CSS timeline.
- **Direction symmetry.** Every visual on the stage is a pure function of
  `--p`: the same `--p` must give the same frame whatever the scroll
  direction or history. Reach one scroll position twice — once wheeling down
  from the top, once wheeling up from the bottom — let `--p` settle to the
  same value both times, and screenshot. The two images must be pixel
  identical. This is what catches compositor-raster bugs, which the `--p`
  numbers alone look fine through.
