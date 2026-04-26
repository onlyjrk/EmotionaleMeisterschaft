# Branded Short URLs — Setup Guide

*Erstellt: 2026-04-26 (Iteration 2)*
*Erwarteter Effekt: +39% CTR vs lange URL (Replug-Aggregat). Bei 5.000 YouTube-Views → das ist netto 50-200 zusätzliche Quiz-Klicks/Monat.*

---

## Domains zu registrieren

| Domain | Ziel | Use-Case | Kosten |
|---|---|---|---|
| **em-test.de** | `https://neujahr.emotionalemeisterschaft.com/quiz.html` | Quiz-CTA in YouTube-Description, Pinned-Comment, Community-Posts | ~10 €/Jahr |
| **em-call.de** | `https://calendly.com/jurekwit/session-mit-jurek` | Calendly-CTA in YouTube-Description, Email-Signaturen | ~10 €/Jahr |
| **em-session.de** (optional) | `https://neujahr.emotionalemeisterschaft.com/sessions.html` | Sessions-Detail-Page (für Visitenkarten, Bio-Links) | ~10 €/Jahr |

**Total**: 20-30 €/Jahr. ROI nach erstem zusätzlichen Klienten amortisiert.

**Domain-Registrar-Empfehlung**:
- **INWX** (deutsch, .de = 4,90 €/Jahr) — https://www.inwx.de/
- **Hetzner** (deutsch, einfach, .de = 5,99 €/Jahr) — https://www.hetzner.com/domainregistration/
- **Cloudflare Registrar** (englisch, .de funktioniert, at-cost-Pricing) — https://www.cloudflare.com/products/registrar/

→ Empfehlung: alle 3 Domains bei **Cloudflare Registrar** registrieren, weil dann auch die Cloudflare-Worker-Redirects nahtlos auf der gleichen Plattform laufen.

---

## Setup-Variante 1: Cloudflare Workers (empfohlen)

**Vorteile**: Schnellste 301-Redirects (Edge-Network, <50ms), kostenlos bis 100.000 Requests/Tag, einfacher als DNS-CNAME.

### Schritt-für-Schritt

1. **Cloudflare-Account anlegen** (falls noch nicht): https://dash.cloudflare.com/sign-up
2. **Domain einbinden** (Cloudflare DNS für em-test.de aktivieren — Nameserver bei Registrar umstellen)
3. **Worker erstellen** (Cloudflare Dashboard → Workers & Pages → Create)

```javascript
// Worker für em-test.de → quiz.html
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const target = 'https://neujahr.emotionalemeisterschaft.com/quiz.html';

    // Optional: UTM-Parameter durchreichen für Plausible-Tracking
    const utm = url.searchParams.toString();
    const finalTarget = utm ? `${target}?${utm}&src=branded_short_url` : `${target}?src=branded_short_url`;

    return Response.redirect(finalTarget, 301);
  }
};
```

4. **Worker-Route binden**: Cloudflare Dashboard → Workers & Pages → dein Worker → Triggers → Add Custom Domain → `em-test.de`
5. **Test**: `curl -I https://em-test.de` sollte `HTTP/2 301` und `Location: https://neujahr.emotionalemeisterschaft.com/quiz.html?src=branded_short_url` zurückgeben.

### Worker für em-call.de → Calendly

```javascript
// Worker für em-call.de → Calendly
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const utm = url.searchParams.toString();
    const target = 'https://calendly.com/jurekwit/session-mit-jurek';
    const finalTarget = utm ? `${target}?${utm}&src=branded_short_url` : target;
    return Response.redirect(finalTarget, 301);
  }
};
```

### Worker für em-session.de → sessions.html

```javascript
export default {
  async fetch(request) {
    return Response.redirect('https://neujahr.emotionalemeisterschaft.com/sessions.html?src=branded_short_url', 301);
  }
};
```

---

## Setup-Variante 2: DNS-Redirect via Hosting-Provider

Falls du Cloudflare nicht nutzen willst, geht's auch via DNS-Redirect bei vielen Registraren (INWX, Hetzner, all-inkl).

**Bei INWX**:
- Domain-Verwaltung → DNS-Einstellungen → Web-Forwarding aktivieren
- Type: **Permanent (301)**
- Ziel: `https://neujahr.emotionalemeisterschaft.com/quiz.html`

**Nachteil**: Etwas langsamer als Worker, kein UTM-Pass-through, oft nur einfache Redirects ohne Tracking-Logik.

---

## Setup-Variante 3: Privater Bit.ly / Rebrandly Account

Falls du keine eigene Domain willst:
- **Bit.ly Custom Branded Short Domain**: $35/Monat — overkill für dich
- **Rebrandly**: Free-Tier 250 Links/Monat mit eigener Domain — geht für deinen Volumen

→ **NICHT empfohlen** — eigene Domain ist günstiger UND wirkt seriöser.

---

## Plausible-Tracking via UTM

Bei jedem Branded-Short-URL-Hit fügt der Worker `?src=branded_short_url` an. Damit kannst du in Plausible (`Sources` Filter) sehen:
- `branded_short_url` → kam über em-test.de / em-call.de
- `youtube` → kam über YouTube-Description (organisch)
- `pinned_comment` → kam über Pinned Comment (wenn du `?src=pinned_a|b|c` setzt)

**Plausible-URL-Pattern für YouTube-Description**:
```
em-test.de?src=ytdesc
em-call.de?src=ytdesc
```

**Plausible-URL-Pattern für Pinned-Comment-A/B-Test**:
```
em-test.de?src=pinned_a   (Variante A in Pinned Comment)
em-test.de?src=pinned_b   (Variante B)
em-test.de?src=pinned_c   (Variante C)
```

---

## Migration auf alle Templates

Nachdem Domains live sind, in folgenden Files Suchen-Ersetzen:

```bash
# In funnel-content.md (Description-Template)
neujahr.emotionalemeisterschaft.com/quiz.html → em-test.de
calendly.com/jurekwit/session-mit-jurek → em-call.de

# In email-sequences-per-shadow-type.md
[calendly-link] → em-call.de
[quiz-ergebnis-link] → em-test.de?type={{shadow_type}}

# In pre-frame-emails.md (post-Calendly-Frame)
calendly.com/jurekwit/session-mit-jurek → em-call.de
```

Top-10 existierende YouTube-Videos manuell updaten (Description + Pinned Comment).

---

## Erwartete Auswirkung

**Vor**:
- 5.000 Views × 4% CTR auf URL = 200 Klicks
- Quiz-Start-Rate 40% → 80 Quiz-Starts
- Email-Capture 70% → 56 Leads/Video

**Nach Branded-Short-URL** (+39% CTR auf Pinned + Description):
- 5.000 Views × 5,56% CTR = 278 Klicks
- 40% → 111 Quiz-Starts
- 70% → 78 Leads/Video

→ **Netto +22 Leads/Video** ohne mehr Aufwand. Bei 4 Videos/Monat = +88 Leads/Monat. Bei 12% Lead→Klient = +10 Klienten/Monat. Bei 130 €/Session × 3 Sessions/Klient (3er-Paket) = **+3.900 €/Monat zusätzlicher Umsatz**.

ROI der Domain-Registrierung (20 €/Jahr): **erste 1 Klient amortisiert es 200x**.

---

*Setup-Aufwand: ~30 Min einmalig. Domain-Wartung: 0. Update der existierenden 30-50 alten YouTube-Descriptions: ~1 Tag manuelle Arbeit (kann auch in Tranches).*
