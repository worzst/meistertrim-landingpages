# MeisterTrim — Landingpage Projekt · Handover für Claude Code

> **Hand-off von Claude (Web-Chat) an Claude Code.**
> Ziel: Eine produktionsfertige, rechtskonforme Direct-Response-Landingpage für den **MeisterTrim Zähmer 2.0** deployen, mit vollständigem Tracking und Shopify-Checkout-Anbindung.

---

## 📦 Was ist in diesem Paket?

```
meistertrim-handover/
├── README.md                          ← du bist hier
├── BRIEFING.md                        ← komplettes Projekt-Briefing (lies das zuerst!)
├── LEGAL-CHECKLIST.md                 ← Schweizer Rechts-Compliance (PBV, UWG)
├── TRACKING-SETUP.md                  ← Meta Pixel, GTM, Triple Whale, CAPI
├── DEPLOYMENT-GUIDE.md                ← Hosting-Optionen + Schritt-für-Schritt
├── COPY-GUIDELINES.md                 ← Tone, Belief Chain, Do's & Don'ts
└── landingpage-v1.html                ← aktuelle LP (Stand: V1, noch NICHT live-tauglich)
```

---

## 🎯 Aktueller Stand & nächste Schritte

**Was ist fertig:**
- ✅ Komplette Landingpage als Single-File-HTML (Advertorial-Style nach Drivse-Vorbild)
- ✅ Belief-Chain umgesetzt (siehe `COPY-GUIDELINES.md`)
- ✅ Mobile-responsive, Sticky-CTA, Countdown, FAQ, Reviews, Vergleichstabelle
- ✅ Custom-SVG-Visuals (kein Stock-Foto-Bedarf)

**Was Claude Code noch tun muss (Priorität):**

### 🔴 BLOCKER — vor Live-Schaltung zwingend
1. **Rechts-Anpassungen** umsetzen (siehe `LEGAL-CHECKLIST.md`)
   - Countdown-Logik fixen (echtes Enddatum statt rolling reset)
   - Streichpreis-Frage mit Kunden klären (PBV-konform)
   - "Verifizierte" Reviews entweder durch echte ersetzen oder entfernen
   - "Werbung"-Kennzeichnung einbauen
   - Health-Claims abmildern

### 🟠 KRITISCH — für Funktionalität
2. **Tracking-Setup** (siehe `TRACKING-SETUP.md`)
   - Meta Pixel + Conversions API
   - Google Tag Manager
   - Triple Whale Pixel
   - Cross-Domain-Tracking konfigurieren

3. **Shopify-Anbindung** (siehe `DEPLOYMENT-GUIDE.md`)
   - CTA-Buttons → Cart-Permalinks mit Variant-IDs
   - UTM-Parameter durchreichen
   - TWINT als Payment sichtbar im Checkout

4. **Hosting deployen**
   - Empfehlung: Cloudflare Pages auf `try.meistertrim.ch`
   - DNS-Setup, SSL, Cache-Headers

### 🟡 NICE-TO-HAVE
5. **A/B-Testing Setup** (z.B. via GTM oder Google Optimize Alternative)
6. **Performance-Optimierung** (Lighthouse Score > 90)
7. **Cookie-Banner** für DSGVO/revDSG-Compliance

---

## ⚠️ WICHTIG: Lies bevor du anfängst

1. **`BRIEFING.md`** — komplettes Projekt-Verständnis (Zielgruppe, Mechanism, Belief Chain)
2. **`LEGAL-CHECKLIST.md`** — du musst diese Punkte mit dem Kunden besprechen, **bevor** du sie umsetzt. Nicht raten — nachfragen.
3. **`COPY-GUIDELINES.md`** — falls der Kunde Copy-Änderungen will, halte dich an diese Tonalität

---

## 👤 Über den Kunden

- **Brand:** MeisterTrim (Schweizer DTC-Marke)
- **Produkt:** Der Zähmer 2.0 — Body- & Intimtrimmer für Männer
- **Markt:** primär Schweiz (CHF, TWINT), sekundär DACH
- **Bestehende Site:** https://meistertrim.ch (Shopify)
- **Aktuelle Strategie:** Ads → direkte Shopify-Produktseite
- **Neue Strategie:** Ads → Pre-Sell-LP → Shopify Checkout

---

## 🛠 Tech-Stack-Empfehlung

| Layer | Empfehlung | Alternative |
|---|---|---|
| Hosting LP | Cloudflare Pages | Vercel, Netlify |
| Domain | `try.meistertrim.ch` | `offer.meistertrim.ch` |
| Checkout | Shopify (bestehend) | — |
| Analytics | GTM + GA4 + Triple Whale | Plausible (privacy-first) |
| Ads-Pixel | Meta Pixel + CAPI, Google Ads, TikTok | — |
| A/B-Test | Google Optimize-Nachfolger / GrowthBook | Convert.com |

---

## ❓ Offene Fragen an den Kunden (vor Deployment klären)

Diese Punkte stehen zwischen "fertig" und "live":

1. **Streichpreis CHF 99.95:** Wurde dieser Preis jemals 30+ Tage am Stück real verlangt? (PBV-Pflicht, siehe `LEGAL-CHECKLIST.md`)
2. **Reviews:** Hast du echte Kundenstimmen mit Vor-/Nachname + Stadt + Datum, die wir verwenden dürfen? (4 Reviews aktuell)
3. **Pixel-IDs:** Welche Meta Pixel ID, GTM-Container ID, Triple Whale Account?
4. **Shopify Variant IDs:** Standard-Variant + Premium-Variant
5. **Subdomain:** Reicht `try.meistertrim.ch` oder eigene Domain (z.B. `sichertrimmen.ch`)?
6. **Aktion-Enddatum:** Echtes Datum für Countdown (z.B. "Aktion endet am 15. Mai 2026, 23:59 Uhr")?
7. **Garantie-Bedingungen:** Sind die "30 Tage testen" + "2 Jahre Garantie" exakt so vertraglich abgesichert?
8. **Diskreter Versand:** Stimmt die Aussage zu 100%? (Verpackung, Absenderfeld)

---

## 🗣 Tonalität für Kommunikation mit dem Kunden

Der Kunde:
- Kennt sein Produkt sehr gut (siehe Research-Dossier mit 17 Seiten)
- Denkt in Direct-Response-Mechanik (Belief Chains, Awareness-Stufen, Mechanism)
- Will pragmatische Antworten, keine Theorie
- Schweizer Markt-Verständnis ist Pflicht (TWINT, CHF, "kein Drama")

Sprich auf Augenhöhe. Wenn etwas rechtlich heikel ist — sag es klar (siehe Legal Checklist).
Wenn etwas technisch besser geht — schlag es vor.

---

**Viel Erfolg. Bei Fragen zum Belief-Chain-Ansatz oder Direct-Response-Mechanik:
Lies zuerst die Original-PDFs des Kunden (Avatar Sheet, Offer Brief, Research Dossier, 6 Kauf-Überzeugungen).**
