# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pre-sell advertorial landing pages for **MeisterTrim Der Zähmer 2.0** (Swiss body/intimate trimmer, CHF 59.95 Standard / CHF 69.95 Premium). Pages sit between paid ads and Shopify checkout. Deployed on Cloudflare Pages at `try.meistertrim.ch`. No build system — pure static HTML.

**Shopify store:** `meistertrim.ch` (existing, do not touch)

## File Structure

```
public/                             ← Cloudflare Pages build output directory (only this is served)
  maenner-insider-report/
    index.html                      ← live LP (try.meistertrim.ch/maenner-insider-report)
    images/kontrolle_nicht_mut.png  ← hero image (section 1)
  _redirects                        ← try.meistertrim.ch/ → meistertrim.ch/products/zaehmer (302)
  robots.txt                        ← Disallow: / (all LPs are noindex — paid traffic only)
CLAUDE.md                           ← not served (repo root, outside public/)
COPY-GUIDELINES.md                  ← not served (repo root, outside public/)
source-documents/                   ← not served (repo root, outside public/)
```

**Cloudflare Pages config:** Build output directory = `public`. DNS is an external CNAME (not proxied through Cloudflare), so there is no zone-level cache to purge — a new deploy is sufficient to invalidate.

New LP variants go into their own folder: `mkdir public/<slug> && cp -r public/maenner-insider-report/ public/<slug>/`

## Commands

```bash
npx serve public/                    # local preview (mirrors CF Pages serving from public/)
open public/maenner-insider-report/index.html  # quick single-file preview
npx lighthouse http://localhost:PORT/maenner-insider-report --view
```

Deployment: `git push origin main` → Cloudflare Pages auto-deploys within ~30s.

## Integrations (all live)

| What | Value |
|---|---|
| Meta Pixel | `532114799530840` |
| GA4 Measurement ID | `G-ZL6JB3MB4C` |
| Triple Whale store | `f04d56-3c.myshopify.com` |
| Shopify Standard Variant | `50296566088026` |
| Shopify Premium Variant | `50296566120794` |
| Cart URL format | `https://meistertrim.ch/cart/{VARIANT_ID}:1?{utm_params}` |

UTM pass-through uses `window.location.search` (full query string, not an allowlist) so all parameters including custom Triple Whale ones are forwarded. CTA IDs: `checkout-btn`, `final-cta`, `sticky-btn`, `pkg-standard`, `pkg-premium`.

All general CTAs (`checkout-btn`, `final-cta`, `sticky-btn`) scroll to `#kaufen` (price block). Only `pkg-standard` and `pkg-premium` go directly to Shopify checkout — this is intentional to maximise premium selection.

Cart URL JS (bottom of each LP):
```javascript
var STANDARD_ID = "50296566088026";
var PREMIUM_ID  = "50296566120794";
function buildShopifyURL(variantId) {
  var qs = window.location.search;
  var sep = qs ? "&" : "?";
  return "https://meistertrim.ch/cart/" + variantId + ":1" + qs + sep + "attributes[lp_source]=maenner-insider-report";
}
```
`attributes[lp_source]` is a Shopify cart attribute — appears in order details for attribution without needing UTM.

**Triple Whale:** official headless snippet v2.17 in `<head>`. Must register `try.meistertrim.ch` as a custom domain in TW Dashboard (one-time, not yet done). Product ID for `TriplePixel("AddToCart", ...)` calls: `9822808801626`.

**GA4 cross-domain still needs UI config** (one-time, no code required):
1. GA4 Admin → Data Streams → Web Stream → Configure tag settings → Configure your domains → add `meistertrim.ch` + `try.meistertrim.ch`
2. Same screen → List unwanted referrals → add `try.meistertrim.ch`

## Shipping Status (important)

Shipping currently from **supplier, not Swiss warehouse** — longer transit times, not yet disclosed on LP. All Swiss shipping claims are commented out until CH stock is available:
- Trust badges "Schnelle Lieferung" + "CH-Lager" → commented out
- "Lieferung in 2–5 Werktagen · Versand aus der Schweiz" in price block → commented out
- FAQ "Wie lange dauert die Lieferung in die Schweiz?" → commented out

When CH fulfillment is live: uncomment these blocks and update the FAQ with accurate transit times.

## Legal Decisions (resolved)

| Item | Decision |
|---|---|
| Countdown | Removed timer, replaced with "Nur für kurze Zeit verfügbar" |
| Streichpreis CHF 99.95 | Accepted risk — has been on Shopify for 1.5+ years |
| Author byline | "MeisterTrim Redaktion" (no fake persona) |
| ANZEIGE-Label | In sale-bar ✅ |
| Absolute health claims | Relativized ("minimiert das Risiko von...") ✅ |
| Neutral packaging claim | Removed — packaging is standard trimmer packaging |
| 2-Jahres-Garantie als USP | Removed — is legal minimum in CH (OR Art. 210), not a differentiator |
| Footer links | All pointing to `meistertrim.ch/policies/...` ✅ |

**Still open (non-blocking for launch):**
- **Reviews:** Real customer reviews are in the HTML (Fabio C., Doron K., Ben R., Stefan B.) but VERIFIZIERT-badges are placeholders — only activate when Shopify/Trustpilot purchase proof exists (TODO comment in HTML).
- **Cookie banner:** Needed for revDSG compliance (Meta Pixel, GA4, Triple Whale) — separate task.
- **GA4 cross-domain UI config:** One-time setup, no code needed: GA4 Admin → Data Streams → Web Stream → Configure tag settings → Configure your domains → add `meistertrim.ch` + `try.meistertrim.ch`. Same screen → List unwanted referrals → add `try.meistertrim.ch`.
- **Triple Whale custom domain:** Register `try.meistertrim.ch` in TW Dashboard.

## SEO / Crawling

All LPs are intentionally **noindex**. Each LP has `<meta name="robots" content="noindex, nofollow">` in `<head>`. `public/robots.txt` also disallows all crawlers. These pages are paid-traffic only — indexing would dilute the Shopify product page and expose the ad funnel to competitors.

## Copy Rules

Read `COPY-GUIDELINES.md` before touching any copy. Key constraints:

**Belief chain order is mandatory** — each section builds on the previous:
Schuld-Entlastung → LED-Mechanism → IPX7 → Grip/3-Säulen → Kontrolle > Glätte → Risk Reversal

**Never:** absolute claims ("100%", "garantiert", "keine Pickel"), brand names (Manscaped, Philips), fake urgency, sexual copy, "ß" (always "ss" in CH).

**Swiss-specific:** Thousands separator is `'` — write `CHF 1'490` in HTML, never inside single-quoted JS strings (breaks parsing). TWINT before PayPal. "Franken" not "Euro".

## Brand Reference

**Product:** Der Zähmer 2.0 — IPX7 waterproof, LED light, Anti-Rutsch-Grip, HautSchutzPro™ blade, 90 min battery. Standard includes guard attachments (3/4.5/6 mm). Premium adds Dual-Foil blade for smoother finish. Payment: TWINT, Klarna, Kreditkarte. Social proof: 5'000+ Kunden DACH.

**Mechanism (Big Idea):** Schnitte entstehen durch fehlende Kontrolle (schlechte Sicht + rutschiger Griff), nicht durch Ungeschicklichkeit. Lösung: LED + IPX7 + Grip = Kontroll-Stack.

**Target audience:** Men 25–40, CH/DE, pragmatic. "Ich mach das für Hygiene, nicht weil ich eitel bin." Skeptisch gegenüber DTC-Hype. Stage 4–5 sophistication — "No cuts" wird nicht geglaubt, nur Mechanism + Erwartungssteuerung + Risiko-Umkehr wirken.

**Design tokens:** `#1f3a2e` (Tannengrün) · `#c8462c` (Terracotta CTA) · `#f6f1ea` (Sand BG). Fonts: Fraunces (display) + Inter (body).
