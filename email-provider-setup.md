# Email-Provider-Setup — Emotionale Meisterschaft Quiz-Funnel

*Erstellt: 2026-04-21. Preise Stand April 2026.*

---

## 1. Vergleichstabelle

| Kriterium | MailerLite | Brevo | Kit (ConvertKit) | Beehiiv |
|---|---|---|---|---|
| **Free-Tier** | 500 Subs, 12.000 Mails/Monat | 100.000 Kontakte, 300 Mails/Tag (~9.000/Mo) | 10.000 Subs, unbegrenzt Mails | 2.500 Subs, unbegrenzt Mails |
| **Automations (Free)** | Ja, aber keine Multi-Step | Ja, bis 2.000 Kontakte in Autos | **Nur 1 Automation**, keine Sequences | **Keine** (nur im Paid) |
| **GDPR / EU-Hosting** | Stark: EU-Segmentierung, GDPR-Form-Builder, DPA | Stark: EU-basiert (FR), DPO, 3-fach Backup | GDPR-compliant, aber US-hosted | GDPR-compliant, US-hosted |
| **Embedded Form** | HTML-Snippet + JS, fertige Forms | HTML + API-Endpoint | HTML + API | HTML, limitiert (Newsletter-Fokus) |
| **Tags / Segments** | Groups + Custom Fields (Free) | Listen + Attributes (Free) | Tags unbegrenzt (Free) | Tags + Segments (Free) |
| **Deutsche UI** | Ja, vollständig | Ja, vollständig (FR-Firma) | Nur Englisch | Nur Englisch |
| **Paid ab** | $10/Mo (500+ Subs) | $9/Mo (Starter) | **$39/Mo** (1.000 Subs) | $49/Mo (Scale) |

---

## 2. Empfehlung: **MailerLite**

**Warum für Jurek:**

1. **Deutsche UI** — Jurek arbeitet auf Deutsch, spart Friction.
2. **GDPR-nativ** — Automatische EU-Subscriber-Segmentierung, GDPR-Checkboxes vorgebaut. Kit und Beehiiv sind US-first.
3. **Automations im Free-Plan** — Das 5-Email-Funnel ist technisch 1 Automation (Trigger: Form-Submit → 5 verzögerte Mails). Funktioniert auf Free. Kit erlaubt nur 1 Automation OHNE Sequences (Knock-out). Beehiiv braucht Paid für jede Automation.
4. **500 Subs Free-Limit reicht bis Launch-Phase.** Paid-Tier ($10/Mo) erst fällig wenn das Quiz wirklich zieht — gutes Signal zum Upgrade.
5. **Tags/Custom Fields** — Perfekt für `shadow_type` als personalisierbare Variable in allen 5 Mails.

**Nicht Kit:** Free-Plan-Automation-Limit killt die Sequence. Paid $39/Mo ist 4x teurer als MailerLite.
**Nicht Brevo:** UI weniger intuitiv, Design-Templates schlechter, Tageslimit 300 Mails kann bei Funnel-Spikes nerven.
**Nicht Beehiiv:** Keine Automations im Free. $49/Mo ab Paid. Ist ein Newsletter-Tool, kein Funnel-Tool.

---

## 3. Integration-Guide für MailerLite

### Schritt 1: Account + Form anlegen

1. Registrieren auf https://www.mailerlite.com/ (Sprache Deutsch wählen)
2. Domain-Authentifizierung: SPF + DKIM für `emotionalemeisterschaft.com` einrichten (Einstellungen → Domains)
3. **Custom Field anlegen:** Subscribers → Fields → "Add field"
   - Name: `shadow_type` (Typ: Text)
   - Name: `firstname` existiert bereits als Standard
4. **Group anlegen:** Subscribers → Groups → `quiz-lead`

### Schritt 2: Embedded Form erstellen

1. Forms → Embedded forms → Create embedded form
2. Name: `Quiz Lead Form`
3. Fields hinzufügen: Email, First name, **shadow_type (hidden)**
4. Group zuweisen: `quiz-lead`
5. Double-Opt-In: **AUS** für Quiz (User hat Intent bereits bestätigt) ODER AN (GDPR-sicherer, empfohlen)
6. Save → "Embed form" → **Action-URL kopieren** (sieht aus wie: `https://assets.mailerlite.com/jsonp/XXXXXX/forms/YYYYYY/subscribe`)

### Schritt 3: quiz.html patchen

Ersetze in `quiz.html` (Zeile 233-247) die `captureLead`-Funktion:

```javascript
function captureLead(e){
  e.preventDefault();
  const email = document.getElementById('email').value;
  const firstname = document.getElementById('firstname').value;
  const dominant = Object.entries(state.counts).sort((a,b)=>b[1]-a[1])[0][0];

  // MailerLite JSONP-Endpoint (aus Form-Embed kopiert)
  const ML_ENDPOINT = 'https://assets.mailerlite.com/jsonp/XXXXXX/forms/YYYYYY/subscribe';

  const formData = new FormData();
  formData.append('fields[email]', email);
  formData.append('fields[name]', firstname);
  formData.append('fields[shadow_type]', dominant);
  formData.append('ml-submit', '1');
  formData.append('anticsrf', 'true');

  try {
    fetch(ML_ENDPOINT, { method: 'POST', body: formData, mode: 'no-cors' })
      .catch(()=>{});
    localStorage.setItem('em_lead_' + Date.now(), JSON.stringify({
      email, firstname, shadow_type: dominant, ts: new Date().toISOString()
    }));
  } catch(_) {}

  // UI: Ergebnis zeigen
  document.getElementById('lead-gate').style.display = 'none';
  document.getElementById('result').style.display = 'block';
  renderResult(dominant);
}
```

**`XXXXXX` und `YYYYYY` ersetzen durch die IDs aus der MailerLite-Action-URL.**

### Schritt 4: 5-Email-Sequence importieren

MailerLite hat **keinen Markdown-Import**. Manuell anlegen:

1. Automation → Create automation → Trigger: `When subscriber joins group "quiz-lead"`
2. **Step 1 (sofort):** Email → Content aus `funnel-content.md` Email 1 kopieren
   - Betreff: `Dein Schatten heißt {$fields.shadow_type}. Hier ist, was er dir kostet.`
   - MailerLite-Syntax: `{$fields.name}` für Firstname, `{$fields.shadow_type}` für Schatten
3. **Delay 1 Tag** → Email 2
4. **Delay 2 Tage** (= Tag 3 ab Start) → Email 3
5. **Delay 4 Tage** (= Tag 7) → Email 4
6. **Delay 7 Tage** (= Tag 14) → Email 5
7. Activate

**Platzhalter-Mapping funnel-content.md → MailerLite:**
- `{{firstname}}` → `{$fields.name}`
- `{{shadow_type_title}}` → `{$fields.shadow_type}` (ggf. via Dropdown-Mapping in einem zweiten Feld als Klartext: "Der Kontrolleur" statt `kontrolleur`)

**Tipp:** Leg ein zweites Custom Field `shadow_type_title` an und mappe im Quiz-JS:
```javascript
const titles = {
  kontrolleur: 'Der Kontrolleur',
  flüchter: 'Der Flüchter',
  // ... usw.
};
formData.append('fields[shadow_type_title]', titles[dominant]);
```

---

## 4. Kostenrahmen bis 5.000 Subscriber

| Subscriber | MailerLite-Plan | Preis/Monat |
|---|---|---|
| 0 – 500 | Free | **0 €** |
| 501 – 1.000 | Growing Business | ~$15 |
| 1.001 – 2.500 | Growing Business | ~$25 |
| 2.501 – 5.000 | Growing Business | ~$39 |
| 5.000+ | Growing Business/Advanced | ~$50-70 |

Jährliche Zahlung: 15% Rabatt. **Trigger-Upgrade**: Wenn Subs-Counter auf Dashboard 450+ zeigt.

---

## 5. Migration-Pfad (falls Wechsel später)

**MailerLite → Kit (wenn Liste 5k+ und Jurek US-Creator-Ökonomie will):**

1. MailerLite: Subscribers → Export → CSV (Email, firstname, shadow_type, groups, date_subscribed)
2. Kit: Grow → Subscribers → Import → CSV-Mapping
3. Tags in Kit neu anlegen (`shadow-kontrolleur`, `shadow-flüchter` etc.)
4. Automation manuell neu in Kit bauen (Visual Automations) — Sequences importieren ginge via Kit-API
5. `quiz.html` Endpoint-URL tauschen: `https://app.kit.com/forms/FORMID/subscriptions`
6. MailerLite-Account 30 Tage parallel laufen lassen, dann archivieren
7. DNS: SPF/DKIM für neue Domain-Auth in Kit umkonfigurieren

**Exportierbare Assets:** Subscribers (CSV), Email-HTML (copy/paste), Automation-Struktur (nicht automatisch, muss neu gebaut werden — das ist bei ALLEN ESPs so).

**Lock-in-Risiko MailerLite:** Niedrig. Standard-CSV + Standard-HTML. Keine proprietären Formate.

---

## Quick-Action-Checkliste

- [ ] MailerLite-Account erstellen (DE-Sprache)
- [ ] Domain authentifizieren (SPF + DKIM)
- [ ] Custom Fields: `shadow_type`, `shadow_type_title`
- [ ] Group: `quiz-lead`
- [ ] Embedded Form erstellen → IDs kopieren
- [ ] quiz.html patchen (captureLead-Funktion ersetzen)
- [ ] 5-Email-Automation aufbauen (aus funnel-content.md)
- [ ] Testlauf: eigene Email durch Quiz jagen, alle 5 Mails prüfen
- [ ] Impressum + Datenschutzerklärung linken (GDPR)

---

Saved to email-provider-setup.md
