# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pre-sell advertorial landing pages for **MeisterTrim Der Zähmer 2.0** (Swiss body/intimate trimmer, CHF 59.95 Standard / CHF 69.95 Premium). Pages sit between paid ads and Shopify checkout. Deployed on Cloudflare Pages at `try.meistertrim.ch`. No build system — pure static HTML.

**Shopify store:** `meistertrim.ch` (existing, do not touch)

## File Structure

```
maenner-insider-report/index.html   ← live LP (try.meistertrim.ch/maenner-insider-report)
_redirects                          ← try.meistertrim.ch/ → meistertrim.ch (302)
COPY-GUIDELINES.md                  ← tone, belief chain, angle library (read before any copy changes)
source-documents/                   ← original PDFs from client (Avatar Sheet, Offer Brief, Research Dossier)
```

New LP variants go into their own folder: `mkdir <slug> && cp maenner-insider-report/index.html <slug>/index.html`

## Commands

```bash
npx serve .                          # local preview (all paths work)
open maenner-insider-report/index.html  # quick single-file preview
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

UTM pass-through (fbclid, gclid, ttclid, wbraid, gbraid) is implemented in the JS at the bottom of each LP. CTA IDs: `checkout-btn`, `final-cta`, `sticky-btn`, `pkg-standard`, `pkg-premium`.

**GA4 cross-domain still needs UI config** (one-time, no code required):
1. GA4 Admin → Data Streams → Web Stream → Configure tag settings → Configure your domains → add `meistertrim.ch` + `try.meistertrim.ch`
2. Same screen → List unwanted referrals → add `try.meistertrim.ch`

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
- **Reviews:** Placeholders remain (TODO comment in HTML) — real customer reviews with consent pending from client. VERIFIZIERT-Badge only when Shopify/Trustpilot purchase proof exists.
- **Cookie banner:** Needed for revDSG compliance (Meta Pixel, GA4, Triple Whale) — separate task.

## Copy Rules

Read `COPY-GUIDELINES.md` before touching any copy. Key constraints:

**Belief chain order is mandatory** — each section builds on the previous:
Schuld-Entlastung → LED-Mechanism → IPX7 → Grip/3-Säulen → Kontrolle > Glätte → Risk Reversal

**Never:** absolute claims ("100%", "garantiert", "keine Pickel"), brand names (Manscaped, Philips), fake urgency, sexual copy, "ß" (always "ss" in CH).

**Swiss-specific:** Thousands separator is `'` — write `CHF 1'490` in HTML, never inside single-quoted JS strings (breaks parsing). TWINT before PayPal. "Franken" not "Euro".

## Brand Reference

**Product:** Der Zähmer 2.0 — IPX7 waterproof, LED light, Anti-Rutsch-Grip, HautSchutzPro™ blade, 90 min battery. Standard includes guard attachments (3/4.5/6 mm). Premium adds Dual-Foil blade for smoother finish.

**Mechanism (Big Idea):** Schnitte entstehen durch fehlende Kontrolle (schlechte Sicht + rutschiger Griff), nicht durch Ungeschicklichkeit. Lösung: LED + IPX7 + Grip = Kontroll-Stack.

**Target audience:** Men 25–40, CH/DE, pragmatic. "Ich mach das für Hygiene, nicht weil ich eitel bin." Skeptisch gegenüber DTC-Hype. Stage 4–5 sophistication — "No cuts" wird nicht geglaubt, nur Mechanism + Erwartungssteuerung + Risiko-Umkehr wirken.

**Design tokens:** `#1f3a2e` (Tannengrün) · `#c8462c` (Terracotta CTA) · `#f6f1ea` (Sand BG). Fonts: Fraunces (display) + Inter (body).
