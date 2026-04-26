# Pre-Frame-Mails — Calendly-Booking Show-up-Boost

*Erstellt: 2026-04-26. Ziel: Show-up-Rate von Erstgespräch von ~50% (Branchenstandard) auf 80-85% boosten.*

> **Setup**: In Calendly unter "Workflows" → "Send email" konfigurieren, getriggert durch "Event scheduled". Optional zweite Mail "Day before event" und SMS-Reminder "1 hour before".
> **Quelle Best-Practice**: Justin Welsh, Cole Gordon, BridgeMind Pre-Call-Frame-Patterns 2026. Show-up-Rate-Effekt empirisch ~30-40% Boost dokumentiert.

---

## Mail 1 — Sofort nach Buchung (Confirmation + Frame)

**Trigger**: Event scheduled
**Send-time**: Direkt nach Buchung
**Betreff**: Termin steht. Hier ist, was im Gespräch passiert (und was nicht).

**Body**:

{{invitee_first_name}},

dein Termin: **{{event_date_time}}** — {{event_type_duration}} via {{event_location}}.

Drei Sachen, damit du das Gespräch optimal nutzt:

**1. Was tatsächlich passiert in 20 Min:**

- 5 Min: Du erzählst mir kurz, wo du gerade stehst und was dich hierher gebracht hat. Keine Geschichte von Anfang an — was JETZT brennt.
- 10 Min: Ich frage 2-3 gezielte Sachen, die mir zeigen, wo der Hebel liegt. Du merkst direkt, dass das nicht "Therapie-Sprech" ist.
- 5 Min: Ich sag dir ehrlich, ob meine Arbeit zu deinem Anliegen passt. Wenn ja: Wie weiter. Wenn nicht: Wer für dich besser passen könnte.

Das ist es. Kein Sales-Pitch. Kein 90-Min-Verkaufsgespräch verkleidet als "Erstgespräch".

**2. Eine Sache, die du davor klären kannst:**

Such dir 5 Minuten und beantworte für dich selbst:

*"Was wäre in 6 Monaten anders, wenn das Thema, mit dem ich grade kämpfe, sich tatsächlich bewegt hätte?"*

Nicht abstrakt. Konkret. Eine Beziehung. Ein Schlafproblem. Ein wiederkehrender innerer Zustand. Wenn du das vor unserem Gespräch geklärt hast, sparen wir 10 Min.

**3. Was du NICHT tun musst:**

- Du musst keine Vorgeschichte aufbereiten.
- Du musst nichts über deine Kindheit erzählen.
- Du musst nicht "in Stimmung" sein.
- Du musst keine Frage haben, die "tief genug" klingt.

Komm einfach so, wie du an dem Tag bist.

**Zur Technik:**

- Falls Video: Stell sicher, dass Kamera + Mikro funktionieren. {{event_location}} öffnet sich im Browser, kein Download nötig.
- Falls Telefon: Ich rufe pünktlich auf der Nummer an, die du angegeben hast.
- Bei Last-Minute-Verschieben: einfach in der Calendly-Bestätigungs-Mail auf "Termin verschieben" klicken — null Problem.

Bis bald.

— Jurek
*Heilpraktiker für Psychotherapie, 4.800+ Sessions*

---

## Mail 2 — 24h vor Termin (Reminder + Frame-Refresh)

**Trigger**: 1 day before event
**Betreff**: Morgen, {{event_time}} — Erstgespräch

**Body**:

{{invitee_first_name}},

morgen, {{event_date}} um {{event_time}}.

Drei kurze Punkte zur Erinnerung:

→ **Format**: {{event_type_duration}} per {{event_location}}.
→ **Inhalt**: 5 Min Standortbestimmung, 10 Min Klärung, 5 Min Fit-Check. Kein Verkauf.
→ **Vorbereitung**: Falls noch nicht: Beantworte für dich kurz "*Was wäre in 6 Monaten anders, wenn das Thema sich bewegt?*"

Wenn dir was dazwischen kommt: Über den Calendly-Link in der Bestätigungs-Mail kannst du verschieben — kein Problem, kein Drama.

Bis morgen.

— Jurek

---

## Mail 3 — 1h vor Termin (Final Reminder)

**Trigger**: 1 hour before event
**Betreff**: In 1 Stunde — kurzer Reminder

**Body**:

{{invitee_first_name}},

in 1 Stunde sprechen wir.

Falls du noch nicht weißt, womit du reinkommen willst — egal. Wir finden's zusammen.

{{event_join_link}}

— Jurek

---

## Mail 4 — Nach Erstgespräch (Follow-up, je nach Outcome)

### 4A — Wenn du sagst "Session passt, lass uns weiter machen"

**Send-time**: Direkt nach dem Call (manuell oder via Workflow)
**Betreff**: Nächste Schritte — {{firstname}}

**Body**:

{{firstname}},

danke für das Gespräch eben.

Wir hatten besprochen: {{summary_of_path}}

**Nächste Schritte:**

1. **Termin für 1. Session**: {{calendly_full_session_link}}
2. **Vor der Session**: Du brauchst nichts vorbereiten. Falls dir spontan was kommt, was du anschauen willst — schreib's auf, mehr nicht.
3. **Falls du das Tiefenarbeit-Paket (6er) gewählt hast**: Ratenzahlung läuft über {{billing_method}}. Erste Rechnung kommt nach Session 1.

Wenn dir bis dahin etwas dazwischenkommt oder Fragen aufkommen — antworte einfach auf diese Mail. Geht direkt an mich.

— Jurek

### 4B — Wenn du sagst "Ich überleg's mir"

**Betreff**: {{firstname}} — kurzer Nachgang

**Body**:

{{firstname}},

danke für das Gespräch.

Falls du noch überlegst — kein Stress, das ist die richtige Reaktion. Eine Session ist eine Investition (Zeit + Geld + Bereitschaft). Diese Entscheidung gehört nicht in einen schnellen Moment.

Falls du dich entscheidest: Hier der Buchungs-Link {{calendly_full_session_link}}.

Falls du Fragen hast, die im Gespräch nicht aufkamen: Antworte auf diese Mail. Direkt zu mir.

Falls's für dich nicht passt: Auch das ist ein klares Ergebnis. Du musst mir nicht antworten.

— Jurek

P.S. Falls du in 3-6 Monaten merkst, dass die Sache wieder hochkommt — dann weißt du, wo du mich findest. Die Tür bleibt offen.

### 4C — Wenn fit nicht passt (du verweist an Kolleg:in)

**Betreff**: {{firstname}} — die Empfehlung, die wir besprochen hatten

**Body**:

{{firstname}},

wie eben besprochen — meine Arbeitsweise passt für dein Anliegen nicht optimal. Hier die Empfehlungen:

**Für {{their_concern}}:**
- {{recommendation_1_with_link}}
- {{recommendation_2_with_link}}

Falls eine davon dir hilft, würde mich eine kurze Rückmeldung freuen — aber keine Pflicht.

— Jurek

P.S. Solltest du in einer ganz anderen Lebensphase irgendwann doch das Gefühl haben, dass meine Art passend ist — die Tür bleibt offen.

---

## Setup-Checkliste in Calendly

- [ ] Workflow "Booking confirmation" → Mail 1 (sofort)
- [ ] Workflow "Day before event" → Mail 2
- [ ] Workflow "1 hour before event" → Mail 3 + optional SMS-Reminder (Calendly bietet das gegen Aufpreis)
- [ ] Bestätigungs-Mail in Calendly-Settings selbst (nicht Workflow) auf KURZ stellen — nur Termin + Join-Link, NICHT die Frame-Mail dort drin (sonst doppelt)
- [ ] Calendly-Eventtitel auf "Erstgespräch — Emotionale Meisterschaft" setzen (nicht "Discovery Call" — zu sales-y)
- [ ] Reschedule-Buffer auf 4 Stunden setzen (Last-Minute-Verschiebungen reduziert No-Shows)
- [ ] No-Show-Mail (für nicht erschienen): manueller Workflow nach 2h, freundlich, einmal Re-Booking-Option

---

## Erwartete Conversion-Kette

Mit Pre-Frame-Mails sollten typische Coach-Funnels folgende Quoten erreichen:

| Stufe | Industry-Standard | Mit Pre-Frame |
|---|---|---|
| Calendly-Booking → Show-up | 50-65% | 80-85% |
| Erstgespräch → Klient | 25-40% | 35-50% |
| Klient → Wiederbuchung | 40-60% | 55-70% |

→ Cumulative-Effekt: Aus 100 Bookings: 13-26 Klienten ohne Pre-Frame, **28-42 mit Pre-Frame**. Faktor 2-3x mehr Umsatz pro Lead.

*Quellen: Cole Gordon Closer-Frameworks, Justin Welsh Discovery-Call-Templates, Calendly-eigene Show-up-Studien 2025.*
