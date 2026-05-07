# DEPLOYMENT GUIDE — Hosting & Go-Live

## TL;DR — empfohlenes Setup

**Cloudflare Pages auf `try.meistertrim.ch`** + Shopify-Cart-Permalinks für Checkout.
Gratis, schnell, einfach. Ideal für statische LP.

---

## Option-Vergleich

| Option | Kosten | Speed | Flexibilität | Wartung | Empfehlung |
|---|---|---|---|---|---|
| **Cloudflare Pages** | Gratis | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | gering | ✅ **EMPFOHLEN** |
| **Vercel** | Gratis | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | gering | ✅ Top |
| **Netlify** | Gratis | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | gering | ✅ Top |
| **Shopify Page** | inkl. | ⭐⭐⭐ | ⭐⭐ | mittel | ⚠️ nur für Quick-Test |
| **Replo** (Shopify-Builder) | $99/Mo | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | gering | ✅ wenn ihr A/B-Tests visuell wollt |
| **Funnelish** | $99+/Mo | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | mittel | ⚠️ eigener Checkout, weg von Shopify |

---

## 🚀 Setup: Cloudflare Pages (empfohlen)

### Vorbereitung

1. **GitHub-Repo anlegen:**
```bash
mkdir meistertrim-lp && cd meistertrim-lp
git init
# landingpage-v1.html als index.html ablegen
mv landingpage-v1.html index.html
git add .
git commit -m "initial: V1 landing page"
git remote add origin git@github.com:YOUR_ORG/meistertrim-lp.git
git push -u origin main
```

2. **Cloudflare Account** anlegen (falls noch nicht vorhanden) — gratis.

### Deploy-Schritte

1. Cloudflare Dashboard → **Workers & Pages** → **Create Application** → **Pages** → **Connect to Git**
2. GitHub-Repo auswählen
3. Build-Config:
   - **Framework preset:** None
   - **Build command:** *(leer lassen)*
   - **Build output directory:** `/`
4. **Save and Deploy**
5. Cloudflare gibt dir eine URL: `meistertrim-lp.pages.dev`

### Custom Domain `try.meistertrim.ch`

1. Im Pages-Projekt → **Custom domains** → **Set up a custom domain**
2. Eingabe: `try.meistertrim.ch`
3. Cloudflare zeigt CNAME-Eintrag:
   ```
   try.meistertrim.ch  CNAME  meistertrim-lp.pages.dev
   ```
4. Diesen Eintrag bei deinem DNS-Provider von `meistertrim.ch` setzen
   - Falls Domain bei Cloudflare ist → automatisch
   - Falls bei anderem Anbieter (z.B. Hostpoint, Infomaniak) → manuell setzen
5. SSL ist automatisch (Cloudflare).

### Auto-Deploy

Jeder `git push` auf `main` triggert automatisch ein neues Deployment. Vorschau-URLs für andere Branches.

---

## 🛒 Shopify-Anbindung: Cart-Permalinks

### Konzept

LP-Buttons leiten direkt zum Shopify-Cart mit Variant-ID und durchgereichten UTMs.
Kein API-Call nötig — Shopify rendert den Cart und User klickt "Checkout".

### Variant-IDs holen

```
Shopify Admin → Products → "Der Zähmer 2.0" → Variants
URL-Format: /admin/products/{PRODUCT_ID}/variants/{VARIANT_ID}
```

Notieren:
- `STANDARD_VARIANT_ID = 12345678901234`
- `PREMIUM_VARIANT_ID = 12345678901235`

### Cart-Permalink-Format

```
https://meistertrim.ch/cart/{VARIANT_ID}:{QUANTITY}
```

Mit Pre-Fill von Email/Discount:
```
https://meistertrim.ch/cart/12345:1?discount=ZAEHMER40&checkout[email]=...
```

### CTA-Buttons in der LP updaten

Aktuell: `<a href="#kaufen">`
Neu:

```html
<!-- Standard-Button -->
<a href="#" class="btn btn-arrow"
   data-variant="STANDARD_VARIANT_ID"
   data-product="zaehmer-standard"
   data-price="59.95">
  Jetzt risikofrei testen
</a>

<!-- Im JS -->
<script>
const STANDARD_ID = '12345678901234';
const PREMIUM_ID  = '12345678901235';

function buildShopifyURL(variantId) {
  const params = new URLSearchParams(window.location.search);
  const passThrough = new URLSearchParams();

  ['utm_source','utm_medium','utm_campaign','utm_content','utm_term',
   'fbclid','gclid','ttclid','wbraid','gbraid'].forEach(p => {
    if (params.get(p)) passThrough.set(p, params.get(p));
  });

  const qs = passThrough.toString();
  return `https://meistertrim.ch/cart/${variantId}:1` + (qs ? `?${qs}` : '');
}

document.querySelectorAll('a.btn').forEach(btn => {
  const variantId = btn.dataset.variant === 'PREMIUM_VARIANT_ID'
    ? PREMIUM_ID
    : STANDARD_ID;
  btn.href = buildShopifyURL(variantId);

  btn.addEventListener('click', e => {
    // Tracking-Events
    if (typeof fbq !== 'undefined') {
      fbq('track', 'InitiateCheckout', {
        value: parseFloat(btn.dataset.price),
        currency: 'CHF',
        content_ids: [btn.dataset.product]
      });
    }
    if (typeof dataLayer !== 'undefined') {
      dataLayer.push({
        event: 'cta_click',
        product_variant: btn.dataset.product,
        product_price: parseFloat(btn.dataset.price)
      });
    }
  });
});
</script>
```

### Discount-Code Integration

Falls die Aktion einen Code nutzt (z.B. `ZAEHMER40` für 40% Rabatt):
```javascript
return `https://meistertrim.ch/cart/${variantId}:1?discount=ZAEHMER40` +
       (qs ? `&${qs}` : '');
```

---

## 🌐 DNS-Setup (für try.meistertrim.ch)

### Wenn Domain bei Cloudflare gehostet ist
Automatisch — nichts zu tun nach Custom Domain Setup.

### Wenn Domain bei anderem Provider (z.B. Hostpoint, Infomaniak, GoDaddy)

**DNS-Record hinzufügen:**
```
Type: CNAME
Name: try
Target: meistertrim-lp.pages.dev
TTL: Auto (oder 3600)
```

DNS-Propagation dauert 5 Min – 24h. Prüfen mit:
```bash
dig try.meistertrim.ch
```

---

## ⚡ Performance-Optimierung

### Lighthouse-Ziele
- Performance: > 90
- Accessibility: > 95
- Best Practices: > 95
- SEO: > 90

### Quick Wins

1. **Fonts preload:**
```html
<link rel="preload" href="https://fonts.gstatic.com/s/fraunces/..."
      as="font" type="font/woff2" crossorigin>
```

2. **Image lazy-loading** (falls echte Fotos hinzukommen):
```html
<img src="..." loading="lazy" alt="...">
```

3. **Critical CSS inline** — bereits gemacht (alle Styles im `<head>`).

4. **GZIP/Brotli** — Cloudflare macht das automatisch.

5. **Cache-Headers** — Cloudflare default ist gut. Bei Bedarf:
   ```
   Cache-Control: public, max-age=3600
   ```

---

## 🧪 Testing vor Live-Schaltung

### Smoke-Test
- [ ] LP lädt unter `try.meistertrim.ch`
- [ ] Mobile-Ansicht funktioniert (iPhone Safari, Android Chrome)
- [ ] Alle CTAs gehen zu Shopify-Cart mit korrekter Variant
- [ ] Countdown läuft korrekt
- [ ] Sticky-CTA erscheint ab 600px Scroll
- [ ] FAQ-Accordions funktionieren
- [ ] Footer-Links zeigen auf echte Shopify-Policies

### Tracking-Test
- [ ] Test-URL: `try.meistertrim.ch/?utm_source=test&utm_campaign=launch_test`
- [ ] Meta Pixel Helper zeigt PageView ✅
- [ ] Bei CTA-Klick: Cart-URL hat alle UTMs ✅
- [ ] Test-Order in Shopify hat UTMs in Order Note Attributes ✅
- [ ] Triple Whale zeigt Pixel-Event ✅
- [ ] GA4 Realtime zeigt Visit ✅

### Browser-Kompatibilität
- [ ] Chrome (Desktop + Mobile)
- [ ] Safari (Desktop + iOS)
- [ ] Firefox
- [ ] Edge
- [ ] Samsung Internet (Android)

### Lighthouse-Test
```bash
npx lighthouse https://try.meistertrim.ch --view
```

---

## 🎯 Launch-Plan

### Soft Launch (Tag 1-3)
- LP live, aber nur kleine Audience (z.B. CHF 50/Tag Budget)
- Nur 1 Audience, 1 Ad-Set, 2-3 Creatives
- Tracking-Sanity-Check: Conversions kommen sauber an?

### A/B-Test (Tag 4-14)
- 50% Traffic → neue LP (`try.meistertrim.ch`)
- 50% Traffic → alte Shopify-Produktseite (`meistertrim.ch/products/zaehmer`)
- Gleiche Audience, gleiche Ads, gleiches Budget
- KPIs:
  - CTR (Ads → LP/PDP)
  - LP/PDP Conversion Rate
  - CPA
  - AOV (Average Order Value)

### Auswertung
- Nach 7 Tagen: Frühe Indikatoren prüfen
- Nach 14 Tagen: Statistisch signifikante Aussage möglich
- Erwartung: LP sollte 25–60% Lift bringen (typisch für Pre-Sell-Funnel)

### Scale-Up
- Wenn LP gewinnt → 100% Traffic auf LP
- Mehr Ad-Variations testen (Angles aus COPY-GUIDELINES.md)
- Premium-Upsell auf LP optimieren

---

## 📂 File-Struktur (für Repo)

```
meistertrim-lp/
├── index.html                  ← Hauptseite
├── /assets/
│   ├── /img/
│   │   └── og-image.jpg        ← Open Graph (1200x630)
│   ├── /js/
│   │   ├── tracking.js         ← Pixel + dataLayer
│   │   └── shopify-cart.js     ← Cart-Permalink-Logik
│   └── /css/
│       └── styles.css          ← optional: separater CSS-File
├── /thank-you/
│   └── index.html              ← optional: Custom TY-Page
├── robots.txt
├── sitemap.xml
└── README.md
```

**Hinweis:** Aktuell ist alles in einer Datei. Falls ihr A/B-Varianten oder mehrere Angle-LPs braucht, lohnt sich Aufsplittung.

---

## 🔄 Multi-LP-Setup (für verschiedene Angles)

Falls ihr verschiedene Hooks testen wollt:

```
try.meistertrim.ch/
├── /                          ← Default (Safety/Kontrolle)
├── /no-itch                   ← Angle: Rasierpickel-Lösung
├── /shower-ready              ← Angle: Convenience
└── /swiss-trust               ← Angle: TWINT/CH-Support
```

Jede URL = eigenes HTML-File mit angepasster Headline + Hero-Punkt 1, Rest identisch.
Cloudflare Pages unterstützt das nativ über Ordnerstruktur.

---

## ⚠️ Häufige Probleme

| Problem | Ursache | Fix |
|---|---|---|
| Cart-Permalink leitet zu falscher Variant | Variant-ID falsch | Shopify Admin → Variant → ID kopieren |
| TWINT erscheint nicht im Checkout | Shopify Payments nicht aktiv | In Shopify → Payments → TWINT aktivieren |
| LP lädt langsam | Custom-Fonts nicht preloaded | `<link rel="preload">` hinzufügen |
| Tracking zeigt 2 Sessions pro User | Cross-Domain nicht aktiv | GA4 + TW Cross-Domain Setup prüfen |
| 404 bei try.meistertrim.ch | DNS noch nicht propagiert | 1-24h warten, mit `dig` prüfen |

---

## 📞 Support-Kontakte

- **Cloudflare:** Dashboard → Help → Live Chat (Pro-Plan), sonst Community Forum
- **Shopify:** 24/7 Live Chat im Admin
- **Triple Whale:** support@triplewhale.com (response < 24h)
- **Meta:** Business Help Center (oft langsam, daher CAPI-Setup gut testen)
