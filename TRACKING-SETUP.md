# TRACKING SETUP — Meta, Google, Triple Whale

> Ziel: 100% Conversion-Attribution von Ad-Click → LP → Shopify-Checkout, trotz Cross-Domain-Setup und iOS-14+-Limitierungen.

---

## Architektur-Übersicht

```
┌─────────────┐         ┌──────────────────────┐         ┌──────────────────┐
│  Meta /     │         │ try.meistertrim.ch   │         │ meistertrim.ch   │
│  Google Ad  │ ──────▶ │ (Landing Page)       │ ──────▶ │ (Shopify Checkout│
└─────────────┘         │                      │         │  & Thank You)    │
       │                │ - Meta Pixel         │         │                  │
       │                │ - GTM                │         │ - Meta CAPI      │
       │                │ - Triple Pixel       │         │ - GA4            │
       │                │ - GA4                │         │ - Triple Whale   │
       │                └──────────────────────┘         │   App            │
       │                          │                       └──────────────────┘
       │                          │
       │                          ▼ feuert beim Klick:
       │                   - InitiateCheckout (Meta)
       │                   - begin_checkout (GA4)
       │                   - +UTM/fbclid/gclid weitergereicht
       │
       └──────────────────────────────────────────▶ Click-IDs landen in URL
```

---

## 1. Meta Pixel (Browser)

### Code im `<head>` der LP

```html
<!-- Meta Pixel -->
<script>
!function(f,b,e,v,n,t,s)
{if(f.fbq)return;n=f.fbq=function(){n.callMethod?
n.callMethod.apply(n,arguments):n.queue.push(arguments)};
if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
n.queue=[];t=b.createElement(e);t.async=!0;
t.src=v;s=b.getElementsByTagName(e)[0];
s.parentNode.insertBefore(t,s)}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', 'YOUR_PIXEL_ID');
fbq('track', 'PageView');
</script>
<noscript><img height="1" width="1" style="display:none"
src="https://www.facebook.com/tr?id=YOUR_PIXEL_ID&ev=PageView&noscript=1"/></noscript>
```

### Custom Events bei CTA-Klicks

```html
<a href="..." class="btn" onclick="fbq('track', 'InitiateCheckout', {
  value: 59.95,
  currency: 'CHF',
  content_ids: ['zaehmer-2-standard'],
  content_name: 'Der Zähmer 2.0 Standard',
  content_type: 'product'
});">Jetzt risikofrei testen</a>
```

### Meta Conversions API (CAPI) — PFLICHT seit iOS 14

**Warum:** Browser-Pixel verliert ~50% der Conversions durch ATT/Adblocker.
**Wie:** Shopify-App "Facebook & Instagram" → CAPI in den Settings aktivieren.

**Wichtig:** `fbclid` aus LP-URL muss an Shopify weitergereicht werden:

```javascript
// Helper: alle URL-Parameter durchreichen zum Shopify-Cart
function buildShopifyURL(variantId, quantity = 1) {
  const params = new URLSearchParams(window.location.search);
  const shopifyParams = new URLSearchParams();

  // UTM-Parameter
  ['utm_source','utm_medium','utm_campaign','utm_content','utm_term'].forEach(p => {
    if (params.get(p)) shopifyParams.set(p, params.get(p));
  });

  // Click-IDs
  ['fbclid','gclid','ttclid','wbraid','gbraid'].forEach(p => {
    if (params.get(p)) shopifyParams.set(p, params.get(p));
  });

  const queryString = shopifyParams.toString();
  return `https://meistertrim.ch/cart/${variantId}:${quantity}` +
         (queryString ? `?${queryString}` : '');
}

// CTA-Buttons updaten
document.querySelectorAll('a[href*="#kaufen"], a[href*="#checkout"]').forEach(btn => {
  const variantId = btn.dataset.variantId || 'STANDARD_VARIANT_ID';
  btn.href = buildShopifyURL(variantId);
});
```

---

## 2. Google Tag Manager (GTM)

### Container-Code im `<head>`

```html
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
```

### Im `<body>` direkt nach `<body>`-Tag

```html
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
```

### dataLayer-Events (für GTM-Trigger)

```javascript
// Beim CTA-Klick:
dataLayer.push({
  event: 'cta_click',
  cta_position: 'hero',  // oder 'mid_page', 'price_block', 'final'
  product_variant: 'standard',
  product_price: 59.95,
  product_currency: 'CHF'
});
```

In GTM dann Trigger erstellen für:
- `PageView` → GA4 Tag
- `cta_click` → Google Ads Conversion + GA4 `begin_checkout`

### Cross-Domain in GA4

GA4 → Admin → Data Streams → Web Stream → Configure tag settings → **Configure your domains**:
- `try.meistertrim.ch`
- `meistertrim.ch`
- `*.meistertrim.ch`

---

## 3. Triple Whale Pixel

### Code im `<head>` (von TW-Dashboard kopieren)

```html
<!-- Triple Whale Pixel -->
<script>!function(){var TripleObject={trackEnable:true,deduplicate:true,
isUserId:!1};window.tripleWhale=TripleObject; ... }();</script>
<script src="https://conversions-config.triplewhale.com/pixel/YOUR_TW_ID.js"
async></script>
```

### Setup in Triple Whale Dashboard

1. **Custom Domain** registrieren: `try.meistertrim.ch`
2. **Linker** aktivieren (TW schreibt Click-IDs in Cookies und gibt sie an Shopify weiter)
3. Shopify-App "Triple Whale" muss installiert sein (zieht Bestellungen automatisch)

### Was TW automatisch macht

- Liest `fbclid`, `gclid`, `ttclid` aus LP-URL
- Schreibt in First-Party Cookies (umgeht ITP/iOS-Limitierungen)
- Server-Side Attribution zwischen LP-Visit und Shopify-Order
- Dashboard zeigt: Ad → LP → Order Funnel

---

## 4. UTM-Parameter Konvention

**Standard für jede Ad-URL:**

```
https://try.meistertrim.ch/zaehmer
  ?utm_source=facebook
  &utm_medium=cpc
  &utm_campaign=zaehmer_kontrolle_de
  &utm_content={{ad.id}}
  &utm_term={{adset.name}}
```

**Quellen-Mapping:**
| Source | Medium | Campaign-Format |
|---|---|---|
| `facebook` | `cpc` | `zaehmer_{angle}_{language}` |
| `instagram` | `cpc` | `zaehmer_{angle}_{language}` |
| `tiktok` | `cpc` | `zaehmer_{angle}_{language}` |
| `google` | `cpc` | `zaehmer_{kw_intent}` |
| `youtube` | `cpc` | `zaehmer_{angle}` |

**Angles** (siehe COPY-GUIDELINES.md):
- `safety_first` (Schnitt-Angst)
- `no_pull` (Ziehen)
- `no_itch` (Rasierpickel)
- `shower_ready` (Convenience)
- `swiss_trust` (TWINT/Support)

---

## 5. Consent-Management (revDSG / DSGVO)

**Wichtig:** Pixels dürfen erst NACH Consent feuern (DSGVO bei DACH-Traffic).

### Lösungs-Optionen:
1. **Cookiebot** (kostenpflichtig, ~CHF 9/Monat, Schweizer Server)
2. **Usercentrics** (kostenpflichtig, EU-fokussiert)
3. **Termly** (US, free tier)
4. **Open-Source:** [klaro!](https://klaro.org/) oder [cookieconsent.js](https://www.cookieconsent.com/)

### Beispiel mit Klaro

```html
<script defer src="https://cdn.kiprotect.com/klaro/v0.7/klaro.js"></script>
<script defer src="/static/klaro-config.js"></script>
```

```javascript
// klaro-config.js
var klaroConfig = {
  version: 1,
  elementID: 'klaro',
  styling: { theme: ['light', 'top', 'wide'] },
  noAutoLoad: false,
  htmlTexts: true,
  cookieExpiresAfterDays: 365,
  default: false,
  mustConsent: false,
  acceptAll: true,
  hideDeclineAll: false,
  translations: {
    de: {
      consentModal: {
        title: 'Wir verwenden Cookies',
        description: 'Wir nutzen Tracking, um unsere Werbung zu optimieren. Du kannst selbst entscheiden, welche Tools wir nutzen dürfen.'
      },
      acceptAll: 'Alle akzeptieren',
      decline: 'Ablehnen',
      ok: 'Speichern'
    }
  },
  services: [
    {
      name: 'meta-pixel',
      title: 'Meta Pixel',
      purposes: ['marketing'],
      cookies: [/^_fbp/, /^_fbc/]
    },
    {
      name: 'gtm',
      title: 'Google Tag Manager',
      purposes: ['analytics', 'marketing'],
      cookies: [/^_ga/, /^_gid/, /^_gat/]
    },
    {
      name: 'triplewhale',
      title: 'Triple Whale',
      purposes: ['analytics']
    }
  ]
};
```

---

## 6. Test-Checklist nach Setup

### Browser-Test
- [ ] Meta Pixel Helper (Chrome Extension) zeigt `PageView` ✅
- [ ] Bei CTA-Klick: `InitiateCheckout` ✅
- [ ] GTM Preview-Mode: alle Tags feuern ✅
- [ ] Triple Whale Debugger: Pixel aktiv ✅

### Conversion-Test
- [ ] Test-Kauf mit eindeutigem UTM (z.B. `utm_campaign=test_setup`)
- [ ] Nach 24h prüfen:
  - Meta Ads Manager: zeigt Test-Conversion
  - Google Ads: Test-Conversion in Conversion Tracking
  - GA4: Test-Conversion in Realtime + DebugView
  - Triple Whale: Order in Pixel-Dashboard
  - Shopify: Order mit korrekten UTM-Parametern in Note Attributes

### Cross-Domain-Test
- [ ] Klick auf LP-CTA → Shopify-Cart hat fbclid/gclid in URL
- [ ] GA4 zeigt Session als ZUSAMMENHÄNGEND (nicht 2 separate Sessions)
- [ ] Meta Conversions API empfängt Match-Quality > 7

---

## 7. Häufige Fehler

| Fehler | Symptom | Fix |
|---|---|---|
| Pixel feuert vor Consent | DSGVO-Verstoss | Consent-Manager vor Pixel laden |
| `fbclid` geht verloren | Conversion ohne Attribution | URL-Parameter durchreichen (siehe oben) |
| Cross-Domain nicht aktiviert | 2 GA4-Sessions pro User | Domains in GA4-Settings eintragen |
| Doppel-Tracking (Pixel + CAPI ohne Dedup) | Conversions doppelt gezählt | `eventID` in beiden Events identisch setzen |
| TW zeigt keine Daten | Pixel nicht auf Subdomain | Custom Domain in TW registrieren |

---

## 8. Variant-IDs holen (Shopify)

```bash
# In Shopify Admin: Product → Variants → Edit
# URL zeigt: /admin/products/PRODUCT_ID/variants/VARIANT_ID
# Diese VARIANT_ID braucht ihr für die Cart-Permalinks
```

Oder per API:
```bash
curl -X GET "https://meistertrim.myshopify.com/admin/api/2024-04/products.json" \
  -H "X-Shopify-Access-Token: YOUR_TOKEN"
```

**Format Cart-Permalink:**
```
https://meistertrim.ch/cart/{VARIANT_ID}:1?attributes[utm_source]=facebook
```

---

## 9. Empfehlung Reihenfolge Setup

1. ✅ Variant-IDs vom Kunden holen
2. ✅ CTA-Buttons mit Cart-Permalink + UTM-Pass-Through coden
3. ✅ GTM-Container aufsetzen (zentralisiert alles)
4. ✅ Meta Pixel über GTM einbauen
5. ✅ Triple Pixel direkt einbauen (TW empfiehlt das)
6. ✅ Shopify Conversions API aktivieren
7. ✅ Consent-Manager (Klaro/Cookiebot)
8. ✅ Test-Käufe → 24h warten → Attribution prüfen
9. ✅ Live für eine Audience, A/B vs. alte Shopify-Page
