# Analytics-Setup — Emotionale Meisterschaft Funnel

**Stand:** 2026-04-21
**Domain:** `neujahr.emotionalemeisterschaft.com`

---

## 1. Empfehlung: **Plausible Analytics**

**Warum Plausible, nicht die anderen?**

| Tool | Verdict | Grund |
|------|---------|-------|
| **Plausible** | ✅ **GEWINNER** | Cookieless, GDPR-by-default, Goals + Funnels eingebaut, 1 Script, EU-hosted (Frankfurt) |
| Umami | Zweitplatz | Self-hosted kostet Zeit (VPS, Updates, Downtime-Risk) — kein Funnel-Feature im Free-Tier |
| PostHog | Overkill | Stark aber komplex, braucht Cookie-Consent bei EU-User-Identification, Free-Tier 1M Events aber UX too heavy für Solo |
| GA4 | ❌ Nein | Cookie-Banner zwingend, EuGH-Urteil zu US-Datentransfer, Schrems-II-Problem |
| Simple Analytics | Knapp dahinter | Kein Funnel-Builder im Basic-Plan, teurer |

**Konkrete Plausible-Vorteile für dich:**
- Kein Cookie-Banner → keine 30% Bounce durch Consent-Dialog
- Eingebauter **Funnel-Builder** (UI) — genau was du brauchst: Landing → Quiz-Start → Quiz-Complete → Email → Session
- Custom Events mit Props (Shadow-Type als Property)
- Script nur **1 KB** (vs. GA4 ~50 KB) → schneller Landing-Speed (gut für SEO)
- EU-hosted (Hetzner Frankfurt) → DSGVO-safe ohne SCCs
- **Kosten:** 9 €/Monat Growth-Plan (10k Pageviews) ODER **30 Tage free trial**, danach self-host-option auf eigenem VPS kostenlos

**Alternative falls Budget = 0:** Umami self-hosted auf Vercel/Railway (kostenlos bis ~5k events/Tag). Code-Snippets unten sind für Plausible, Umami-Syntax ist fast identisch (`umami.track()` statt `plausible()`).

---

## 2. Code-Snippets

### 2.1 Tracking-Script (in `<head>` aller 3 Files)

```html
<!-- Plausible Analytics — cookieless, GDPR-safe -->
<script defer data-domain="neujahr.emotionalemeisterschaft.com"
        src="https://plausible.io/js/script.tagged-events.outbound-links.js"></script>
<script>window.plausible = window.plausible || function() { (window.plausible.q = window.plausible.q || []).push(arguments) }</script>
```

Das `tagged-events`-Script erlaubt Event-Tracking via CSS-Klasse. `outbound-links` trackt automatisch externe Links (z.B. Teachable, Calendly).

---

### 2.2 `index.html` — Kurs-Landing

**Teachable-Checkout-Buttons** (3 Stück) — füge Klasse hinzu:

```html
<a href="https://teachable.com/..."
   class="plausible-event-name=Checkout+Click plausible-event-tier=Basis">
   Kurs buchen — Basis
</a>
<a href="..." class="plausible-event-name=Checkout+Click plausible-event-tier=Plus">Plus</a>
<a href="..." class="plausible-event-name=Checkout+Click plausible-event-tier=Premium">Premium</a>
```

**Quiz-CTA-Button auf Landing:**
```html
<a href="/quiz.html" class="plausible-event-name=Quiz+Start+From+Landing">
   Mach den Shadow-Type-Test
</a>
```

---

### 2.3 `quiz.html` — Lead-Magnet Quiz

**Im `<head>` zusätzlich zum Plausible-Script:**

```html
<script>
  // Quiz-Start: beim Klick auf erste Frage
  function trackQuizStart() {
    plausible('Quiz Start');
  }

  // Quiz-Complete: wenn letzte Frage beantwortet, vor Email-Capture
  function trackQuizComplete(shadowType) {
    plausible('Quiz Complete', { props: { shadow_type: shadowType } });
  }

  // Email-Submit: beim Form-Submit
  function trackEmailSubmit(shadowType) {
    plausible('Email Submit', { props: { shadow_type: shadowType } });
  }

  // Session-CTA-Click im Quiz-Ergebnis
  function trackSessionCTAFromQuiz(shadowType) {
    plausible('Session CTA Click', { props: { source: 'quiz', shadow_type: shadowType } });
  }
</script>
```

**Einbau in existierenden Quiz-Code** (passe an deine Funktionsnamen an):

```javascript
// Im startQuiz()-Handler:
document.querySelector('#quiz-start-btn').addEventListener('click', () => {
  trackQuizStart();
  // ... bestehende Logik
});

// Am Ende der letzten Frage, sobald shadowType berechnet:
function showResult(shadowType) {
  trackQuizComplete(shadowType);
  // ... Ergebnis anzeigen + Email-Form
}

// Beim Email-Form-Submit:
document.querySelector('#email-form').addEventListener('submit', (e) => {
  const st = window.currentShadowType; // aus Quiz-State
  trackEmailSubmit(st);
  // ... Email an ConvertKit/Mailjet
});

// Beim "Jetzt 1:1 Session buchen"-Link im Ergebnis:
document.querySelector('#session-cta').addEventListener('click', () => {
  trackSessionCTAFromQuiz(window.currentShadowType);
});
```

---

### 2.4 `sessions.html` — 1:1 Session-Landing

**Im `<head>` zusätzlich:**

```html
<script>
  function trackCalendlyClick(sessionType) {
    plausible('Calendly Click', { props: { session_type: sessionType } });
  }
</script>
```

**Calendly-Button:**

```html
<a href="https://calendly.com/jurek/discovery-call"
   onclick="trackCalendlyClick('discovery')"
   class="plausible-event-name=Calendly+Click plausible-event-type=discovery">
   Kostenloses Kennenlern-Gespräch buchen
</a>

<a href="https://calendly.com/jurek/1h-session"
   onclick="trackCalendlyClick('1h_paid')"
   class="plausible-event-name=Calendly+Click plausible-event-type=1h_paid">
   1h Session (150 €)
</a>
```

---

## 3. Dashboard-Setup nach Signup

Nach Account-Erstellung bei plausible.io → Site hinzufügen → `neujahr.emotionalemeisterschaft.com` → **Goals** + **Funnels** anlegen:

### 3.1 Goals anlegen (Settings → Goals → Add Goal)

| Goal-Name | Typ | Beschreibung |
|-----------|-----|-------------|
| `Quiz Start From Landing` | Custom Event | CTA-Klick auf Landing |
| `Quiz Start` | Custom Event | Quiz betreten |
| `Quiz Complete` | Custom Event | Alle Fragen durch |
| `Email Submit` | Custom Event | Lead erfasst |
| `Session CTA Click` | Custom Event | Quiz → Session-Interest |
| `Calendly Click` | Custom Event | Hot Lead |
| `Checkout Click` | Custom Event | Kurs-Kauf-Intent |
| `Visit /quiz.html` | Pageview | Quiz-Seiten-Aufruf |
| `Visit /sessions.html` | Pageview | Session-Seiten-Aufruf |

### 3.2 Funnel #1 — Main Funnel (Lead-Flow)

**Settings → Funnels → New Funnel:** Name: `YouTube → Email Lead`
1. Pageview `/`
2. Custom Event `Quiz Start`
3. Custom Event `Quiz Complete`
4. Custom Event `Email Submit`

→ Du siehst pro Step % Drop-off.

### 3.3 Funnel #2 — Revenue-Funnel

Name: `Quiz Lead → Session Booking`
1. `Email Submit`
2. Pageview `/sessions.html`
3. `Calendly Click`

### 3.4 Funnel #3 — Direct Course-Sale

Name: `Landing → Checkout`
1. Pageview `/`
2. `Checkout Click`

### 3.5 Shadow-Type-Analyse

Nicht als Funnel — als **Custom Property Breakdown**:
- Dashboard → Goal `Email Submit` → Klick „Properties" → `shadow_type`
- Zeigt Ranking: welcher Shadow-Type wird am häufigsten gesubmittet

### 3.6 YouTube-Traffic-Quelle

Plausible erkennt `utm_source` automatisch. In YouTube-Beschreibungen:
```
https://neujahr.emotionalemeisterschaft.com/?utm_source=youtube&utm_medium=video&utm_campaign=shadow_quiz
```
→ Dashboard „Top Sources" zeigt YouTube-Conversions.

---

## 4. Kosten & Limits

**Plausible Growth Plan:**
- **9 €/Monat** (jährlich 90 €) für bis 10.000 Pageviews/Monat
- 30 Tage Free Trial ohne Kreditkarte
- Unlimited Goals, Funnels, Custom Events
- 1 Site im Starter, bis 50 Sites im Plus (19 €)

**Wann reicht Free-Trial-Ende + Self-Host?**
Wenn du > 10k/Monat erreichst und 9 € nicht stemmen willst → Umami auf Railway self-hosten (kostenlos bis 500 h/Monat Compute).

**Realistische Skalierung für dich:**
- Monat 1–3: < 2k Pageviews → locker im 9 €-Plan
- Monat 6+: bei 5k Leads ggf. 19 €-Plan

**Hidden Costs:** Keine. Keine Consent-Management-Tool-Lizenz (CookieBot = 9 €/Monat) gespart.

---

## 5. Privacy-Text für Footer/Impressum

**Kurzform für Footer** (Link „Datenschutz"):
```
Diese Website nutzt Plausible Analytics — ein cookieloses, datenschutzfreundliches
Analyse-Tool mit Servern in der EU (Frankfurt, Deutschland). Es werden keine
personenbezogenen Daten oder Cookies gespeichert. Keine IPs, keine Fingerprints.
Mehr: plausible.io/data-policy
```

**Ausführlich für `/datenschutz.html`:**
```
Reichweitenmessung mit Plausible Analytics

Zur statistischen Auswertung der Besucherströme nutzen wir Plausible Analytics
(Anbieter: Plausible Insights OÜ, Västriku tn 2, 50403 Tartu, Estland). Plausible
verwendet KEINE Cookies und speichert KEINE personenbezogenen Daten. Es werden
ausschließlich aggregierte Daten erhoben (Seitenaufrufe, Referrer, Browsertyp,
Gerätetyp, Land basierend auf anonymisierter IP). Die IP-Adresse wird nie
gespeichert und nur zur Länderzuordnung in Echtzeit verarbeitet und verworfen.

Rechtsgrundlage: Art. 6 Abs. 1 lit. f DSGVO (berechtigtes Interesse an
statistischer Auswertung der Website-Nutzung). Da keine personenbezogenen Daten
verarbeitet werden, ist keine Einwilligung erforderlich (kein Cookie-Banner nötig).

Datenschutzerklärung Plausible: https://plausible.io/privacy
Server-Standort: Frankfurt am Main, Deutschland (Hetzner Online GmbH)
```

---

## 6. Next Steps Checklist

- [ ] plausible.io → Signup (30 Tage Trial)
- [ ] Site hinzufügen: `neujahr.emotionalemeisterschaft.com`
- [ ] Tracking-Script in `<head>` von `index.html`, `quiz.html`, `sessions.html` einbauen
- [ ] Custom-Event-JS in `quiz.html` + `sessions.html`
- [ ] CSS-Klassen `plausible-event-name=...` an CTAs
- [ ] Goals anlegen (9 Stück, Liste oben)
- [ ] 3 Funnels anlegen
- [ ] UTM-Parameter an alle YouTube-Links hängen
- [ ] Footer-Link „Datenschutz" → Privacy-Text einfügen
- [ ] 7 Tage beobachten, dann Funnel-Drop-offs analysieren und CTAs iterieren

---

**App-Tracking (em://*):** Plausible trackt nur Web. Für App-Events nutze weiterhin RevenueCat (Subscriptions) + Firebase Analytics (Events). Web ↔ App via User-ID beim Email-Submit verknüpfen (Plausible-Prop `user_id_hash` setzen wenn Login vorhanden).

Saved to analytics-setup.md
