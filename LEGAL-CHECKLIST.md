# LEGAL CHECKLIST — Schweizer Compliance vor Live-Schaltung

> ⚠️ **Hinweis:** Dies ist KEINE Rechtsberatung. Bei kritischen Entscheidungen → Schweizer Wirtschaftsanwalt konsultieren (~CHF 300–500 für 60 Min).
>
> Die folgenden Punkte basieren auf SECO-Quellen, dem UWG und der PBV (Stand: Mai 2026).

---

## 🔴 BLOCKER — vor Live-Schaltung zwingend fixen

### 1. Countdown-Timer (UWG-Risiko)

**Aktueller Zustand:** Timer startet bei jedem Page-Load neu bei 1h 45min.

**Problem:** Wenn ein User die Seite mehrfach besucht und denselben "Aktion endet"-Timer sieht, ist das nach **UWG Art. 3 Abs. 1 lit. b** eine irreführende Geschäftspraxis ("rolling fake urgency").

**Lösungs-Optionen:**

**Option A — echtes Aktionsende (empfohlen):**
```javascript
// Festes Datum statt rolling reset
const ACTION_END = new Date('2026-05-15T23:59:00+02:00');
```
- Frühlings-Aktion endet z.B. am 15.05.2026 um 23:59 Uhr
- Nach dem Datum: Aktion ist wirklich vorbei (Preis steigt oder Box verschwindet)
- Aktion kann später als neue Kampagne (z.B. "Sommer-Aktion") wiederkommen

**Option B — Countdown ganz entfernen:**
- Sale-Bar bleibt, aber ohne Timer
- Reduziert Conversion etwas, eliminiert Risiko komplett

**Frage an Kunden:** Welche Option? Wenn A → konkretes Enddatum nötig.

---

### 2. Streichpreis CHF 99.95 → CHF 59.95 (PBV-Pflicht)

**Aktueller Zustand:** Durchgestrichener "Normalpreis" CHF 99.95, Aktionspreis CHF 59.95.

**Schweizer Recht (PBV Art. 16):** Selbstvergleich (Streichpreis) ist nur in 2 Varianten zulässig:

#### Variante A — "Halbierungsregel"
- Du musst den Trimmer **unmittelbar vorher** zum Vergleichspreis (CHF 99.95) angeboten haben
- Streichpreis darf **nur halb so lange** angezeigt werden wie der Original-Preis galt
- **Maximal 2 Monate** lang

**Beispiel:** Trimmer wurde 30 Tage zu CHF 99.95 verkauft → darf danach 15 Tage als "CHF 99.95 statt CHF 59.95" beworben werden.

#### Variante B — "30-Tage-Regel" (seit 2024 NEU, empfohlen für E-Commerce)
- Vergleichspreis muss mindestens **30 aufeinanderfolgende Tage** real gegolten haben
- Danach: **zeitlich unbegrenzter** Streichpreis erlaubt
- Funktioniert auch wenn Produkt zwischenzeitlich aus Sortiment war

**Beispiel:** Trimmer wird 30 Tage zu CHF 99.95 verkauft → ab Tag 31 darf "CHF 59.95 statt CHF 99.95" zeitlich unbegrenzt beworben werden.

#### ❓ Frage an Kunden (KRITISCH):

> **War der Zähmer 2.0 jemals 30+ Tage am Stück real für CHF 99.95 im Verkauf?**

| Antwort | Konsequenz |
|---|---|
| **Ja, dokumentiert** | ✅ Streichpreis bleibt — PBV-konform |
| **Ja, aber nicht dokumentiert** | ⚠️ Screenshots aus Wayback Machine / Shopify-History sammeln |
| **Nein** | ❌ Streichpreis muss raus oder durch andere Anker ersetzt werden |

**Wenn "Nein" — Alternativen:**
- "Marktpreis-Vergleich" mit Konkurrenz-Trimmern (PBV erlaubt Konkurrenzvergleich)
- "Wert"-Statt-Preis-Anker ("im Wert von CHF 99.95")
- Bundle-Logik ("Sparen Sie CHF 40 gegenüber Einzelkauf")

---

### 3. "Verifizierte" Reviews & Author-Byline (UWG-Risiko)

**Aktueller Zustand:**
- Autor "Marco Schneider" mit Verified-Badge
- 4 Reviews mit Vor-/Nachname, Stadt, "VERIFIZIERT"-Badge

**Problem:** Wenn diese Personen erfunden sind → **UWG Art. 23 Strafrecht** (Geldstrafe bis CHF 540'000 oder Freiheitsstrafe bis 3 Jahre).

**Konkurrenten und der Konsumentenschutz reichen aktiv solche Fälle weiter.**

**Lösungs-Optionen:**

**Option A — echte Reviews verwenden:**
- Vor- + Anfangsbuchstabe des Nachnamens ist okay (Datenschutz)
- "Verifiziert"-Badge nur, wenn ihr wirklich Kaufbeleg habt
- Datum der Review angeben (Transparenz)
- Quelle: Trustpilot / Shopify Reviews / Google Reviews

**Option B — Autor zu MeisterTrim-Team ändern:**
```html
<div class="byline">
  <div class="avatar">MT</div>
  <div>
    <div class="name">MeisterTrim Redaktion</div>
    <div class="role">Schweizer Männerpflege · Stand: Mai 2026</div>
  </div>
</div>
```
- Kein Fake-Person-Risiko mehr
- "Werbung"-Charakter wird transparenter (was sowieso Pflicht ist, siehe Punkt 4)

**Frage an Kunden:**
1. Wie viele echte Trustpilot/Shopify-Reviews gibt es?
2. Habt ihr Erlaubnis, Kundennamen + Stadt zu zitieren?

---

### 4. "Werbung"-Kennzeichnung (Trennungsgebot)

**Aktueller Zustand:** Seite sieht aus wie Magazin-Artikel ("MeisterTrim Report"), keine explizite Werbe-Kennzeichnung.

**Schweizer Recht:** Der Schweizer Presserat (Richtlinie 10.1) verlangt eine klare Trennung redaktionell/Werbung. Bei Direct-Response-Ads, die direkt auf die LP führen, ist die Werbeintention durch das Ad-Format selbst klar — **aber sobald die URL anders geteilt wird (Influencer, Newsletter, Forum-Posts), greift die Kennzeichnungspflicht.**

**Lösung:** Dezentes "WERBUNG" oder "ANZEIGE" oben rechts in der Sale-Bar.

```html
<div class="sale-bar">
  <span class="ad-label">ANZEIGE</span>
  <strong>FRÜHLINGS-AKTION</strong> | BIS ZU 40% RABATT...
</div>
```

```css
.ad-label {
  background: #fff;
  color: #1a1410;
  padding: 2px 8px;
  font-size: 10px;
  border-radius: 3px;
  margin-right: 12px;
  letter-spacing: .12em;
}
```

---

### 5. Absolute Health-Claims abmildern

**Aktueller Zustand:**
- "Keine eingewachsenen Haare"
- "Keine juckenden Pickel"
- "Kein Ziehen oder Reissen"

**Problem:** Absolute Aussagen sind nach UWG irreführend, sobald ein einziger Kunde nach Benutzung trotzdem ein Problem hat.

**Lösung — relativieren:**

| Vorher | Nachher |
|---|---|
| "Keine eingewachsenen Haare" | "Deutlich weniger eingewachsene Haare" |
| "Keine juckenden Pickel" | "Minimiert das Risiko von Rasierpickeln" |
| "Kein Ziehen oder Reissen" | "Schneidet sauber statt zu ziehen" |
| "100% Kontrolle" (LED-Section) | "Volle Sicht auf jeden Winkel" |

---

## 🟡 KRITISCH — sollte gefixt werden

### 6. Vergleichstabelle (vergleichende Werbung)

**Aktueller Zustand:** Tabelle vergleicht "Zähmer 2.0" mit "Standard-Trimmer" und "DTC-Premium (US)" — ohne Markennamen.

**Status:** Keine Markennamen → grundsätzlich okay nach UWG.

**ABER prüfen:**
- Stimmt "DTC-Premium: 1 Jahr Garantie" für Manscaped wirklich?
- Stimmt "Abo aktiv" — bietet Manscaped Lawn Mower 5.0 zwingend ein Abo an oder optional?
- Sind die Preise akkurat?

**Lösung:** Faktisch falsche Vergleiche entfernen oder mit Quellenstand versehen ("Stand: Mai 2026").

---

### 7. Garantie-Aussagen vertraglich absichern

**Aktueller Zustand:**
- "30 Tage testen, ohne Diskussion"
- "2 Jahre Garantie auf die Qualität"
- "Kein Abo-Trick. Punkt."

**To-Do:**
- AGB von MeisterTrim prüfen → stimmen die Angaben überein?
- Refund-Policy auf der LP verlinken (Footer)
- Garantie-Bedingungen klar dokumentiert auf Shopify?

---

### 8. Diskreter Versand — Faktencheck

**Aktueller Zustand:** "Komplett neutrale Verpackung — niemand sieht von aussen, was drin ist."

**To-Do:**
- Stimmt das wirklich zu 100%?
- Steht "MeisterTrim" als Absender oder neutral (z.B. "MT Versand AG")?
- Karton-Branding?

Wenn Aussage nicht zu 100% stimmt → entweder fixen oder weicher formulieren ("diskret verpackt, ohne Produkt-Branding aussen").

---

### 9. Cookie-Banner / revDSG-Compliance

**Aktueller Zustand:** Kein Cookie-Banner.

**Schweizer Recht (revDSG seit 1.9.2023):**
- Tracking-Cookies (Meta Pixel, GTM, etc.) brauchen **informierten Consent**
- Pflichtangaben: Was wird getrackt, von wem, wofür, wie lange
- Opt-out muss möglich sein

**Lösung:** Cookie-Banner einbauen (z.B. **Cookiebot, Usercentrics, Termly, oder Open-Source Cookieconsent.js**).

Bei reinem Schweiz-Traffic: revDSG reicht. Bei DACH-Traffic (auch DE/AT): DSGVO-konform → strenger.

---

### 10. Impressum & Datenschutzerklärung

**Aktueller Zustand:** Footer-Links auf "#" (Platzhalter).

**Pflicht in der Schweiz:** Impressum nur für Medien, **aber für E-Commerce empfohlen** (Vertrauen + UWG-Konformität).

**Pflicht-Links im Footer:**
- Impressum (Firmenname, Adresse, UID-Nummer, Kontakt)
- Datenschutzerklärung
- AGB
- Widerrufs-/Rückgaberecht

Diese sollten alle auf `meistertrim.ch/policies/...` zeigen (existiert bereits).

---

## ✅ Quick-Check vor Go-Live

- [ ] Countdown-Logik gefixt (echtes Datum oder entfernt)
- [ ] Streichpreis-Frage mit Kunden geklärt + dokumentiert
- [ ] Reviews echt oder neutralisiert
- [ ] Author-Byline geklärt (real oder "MeisterTrim Team")
- [ ] "ANZEIGE"-Label in Sale-Bar
- [ ] Health-Claims abgemildert
- [ ] Vergleichstabelle faktisch geprüft
- [ ] Garantie-Aussagen mit AGB abgeglichen
- [ ] Diskreter-Versand-Aussage geprüft
- [ ] Cookie-Banner integriert
- [ ] Footer-Links auf echte Policies
- [ ] **Optional:** Anwalt-Review (60 Min, ~CHF 400)

---

## Worst-Case-Szenarien (für Risiko-Einschätzung)

| Szenario | Wahrscheinlichkeit | Folgen |
|---|---|---|
| Konsument meldet bei Konsumentenschutz | mittel | Mahnschreiben, ggf. öffentliche Listung |
| Mitbewerber-Abmahnung (Anwalt) | hoch (bei Volumen) | CHF 1'500–5'000 Anwaltskosten + LP anpassen |
| SECO/Gewerbepolizei Untersuchung | niedrig | Verfügung, Bussgeld bis CHF 100'000 |
| Lauterkeitskommission Beschwerde | mittel | Empfehlung (nicht bindend), Reputation |
| UWG-Strafanzeige (Art. 23) | sehr niedrig | Theoretisch bis CHF 540'000 / 3J Haft |

**Realistisch:** Bei sauberer Umsetzung der obigen Punkte ist das Risiko überschaubar. Bei aktuellem V1-Stand wäre eine Abmahnung in den ersten 3 Monaten wahrscheinlich.

---

## Quellen

- SECO UWG-Grundlagen: https://www.seco.admin.ch/seco/de/home/Werbe_Geschaeftsmethoden/Unlauterer_Wettbewerb/grundlagen.html
- SECO PBV: https://www.seco.admin.ch/seco/de/home/Werbe_Geschaeftsmethoden/Preisbekanntgabe/generalites.html
- PBV-Wegleitung 2025 (PDF): SECO-Website
- Schweizer Presserat (Richtlinie 10.1): https://presserat.ch/
- Lauterkeitskommission: https://www.faire-werbung.ch/
- Konsumentenschutz Meldeplattform: https://www.konsumentenschutz.ch/
