# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pre-sell advertorial landing page for **MeisterTrim Der Zähmer 2.0** (Swiss body/intimate trimmer). The page sits between paid ads and Shopify checkout at `try.meistertrim.ch`. Target market is German-speaking Switzerland (CHF, TWINT). No build system — pure static HTML deployed via Cloudflare Pages.

**Current state:** `landingpage-v1.html` is drafted but **not live-ready**. See the blockers in `README.md` and `LEGAL-CHECKLIST.md` before touching anything.

## Commands

No build system. To preview locally:
```bash
open landingpage-v1.html          # open in default browser
npx serve .                        # serve on localhost
npx lighthouse http://localhost:PORT --view   # Lighthouse audit
```

Deployment is `git push` to `main` → Cloudflare Pages auto-deploys.

## Architecture

Single-file HTML (`landingpage-v1.html`) with all CSS inline in `<head>` and JS at bottom of `<body>`. Planned future structure (from `DEPLOYMENT-GUIDE.md`):

```
index.html
/assets/js/tracking.js       ← Pixel + dataLayer events
/assets/js/shopify-cart.js   ← Cart permalink + UTM pass-through
/no-itch/index.html          ← A/B angle variants
/shower-ready/index.html
```

**Design tokens:** Deep forest green `#1f3a2e` + terracotta CTA `#c8462c` + sand `#f6f1ea`. Fonts: Fraunces (display/italic headlines) + Inter (body) — editorial magazine look.

## Key Integrations (pending client info)

- **Shopify checkout:** `https://meistertrim.ch/cart/{VARIANT_ID}:1?{utm_params}` — needs `STANDARD_VARIANT_ID` and `PREMIUM_VARIANT_ID` from client
- **Tracking stack:** Meta Pixel (browser) + CAPI via Shopify app + GTM container + Triple Whale Pixel — all IDs pending from client
- **UTM pass-through:** `fbclid`, `gclid`, `ttclid`, `wbraid`, `gbraid` must be forwarded from LP URL to Shopify cart URL

## Legal Blockers (must resolve before go-live)

All details in `LEGAL-CHECKLIST.md`. Short list:

1. **Countdown:** Currently rolling reset (UWG violation) → needs real end date or removal
2. **Streichpreis CHF 99.95:** Only legal if product was sold at that price for 30+ consecutive days (PBV Art. 16) → confirm with client
3. **Reviews + author byline:** Currently placeholder personas → must be real (with consent) or replaced with "MeisterTrim Redaktion"
4. **"ANZEIGE" label:** Required in sale-bar (Swiss press council Richtlinie 10.1)
5. **Absolute health claims:** "Keine Pickel" etc. → must be relativized ("minimiert das Risiko von...")
6. **Cookie banner:** Required for Meta Pixel / GTM under revDSG (since 1.9.2023)
7. **Footer links:** Currently `href="#"` → must point to `meistertrim.ch/policies/...`

## Copy Rules (see `COPY-GUIDELINES.md` for full library)

**Belief chain order is mandatory** — each section builds on the previous:
1. Schuld-Entlastung ("nicht deine Schuld") → 2. LED-Mechanism → 3. IPX7 → 4. Grip/3-Säulen → 5. Kontrolle > Glätte → 6. Risk Reversal

**Swiss-specific:**
- Thousands separator: `'` (apostrophe) — `CHF 1'490` not `CHF 1.490`. Store raw numbers; format at render time with `.replace(/\B(?=(\d{3})+(?!\d))/g, "'")`. Do NOT put Swiss-formatted numbers in single-quoted JS strings — the `'` breaks parsing.
- Always `ss` not `ß` (Strasse, nicht Straße)
- TWINT before PayPal; "Franken" not "Euro"; "Post" for shipping

**Never:** absolute claims ("100%", "garantiert"), brand comparisons (Manscaped, Philips), fake urgency, sexual copy.

## Open Questions (block go-live)

Client must answer before implementation (see `README.md` for full list):
- Streichpreis history (30-day rule)
- Real review pool (Trustpilot/Shopify)
- Pixel IDs (Meta, GTM container, Triple Whale)
- Shopify Variant IDs (Standard + Premium)
- Countdown end date
- Subdomain confirmation (`try.meistertrim.ch`)
