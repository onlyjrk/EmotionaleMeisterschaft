# Discord-Setup "Emotionale Meisterschaft"

Pragmatische Blaupause. 2-Stunden-Setup. Keine Floskeln.

---

## 1. Server-Name + Tagline (je 3 Optionen)

**Server-Name:**
1. `Emotionale Meisterschaft` (klarer Markenbezug, schlicht)
2. `Schatten & Kompass` (Nische, weniger Google-Konkurrenz)
3. `Die Tiefen-Männer` (Zielgruppen-Signal, aber etwas cringe)

**Empfehlung:** Option 1. Matched Brand, YouTube, Landing.

**Tagline:**
1. "Schattenarbeit für Männer, die keine Lust mehr auf Bypassing haben."
2. "Wo Männer 25–44 ihre Wut, Angst und Scham nicht wegatmen, sondern verstehen."
3. "Community für ehrliche Innenarbeit. Keine Gurus. Keine Bro-Energy."

**Empfehlung:** Option 1.

---

## 2. Channel-Struktur

### Kategorie: START HIER (public, alle sehen)
- `#willkommen` — Welcome-Bot-Text, Pinned Post mit Server-Map
- `#regeln` — 7 Regeln, Reaction-Role zum Bestätigen (öffnet Rest)
- `#vorstellung` — Jeder neue Member postet Intro (Template pinned)
- `#ankündigungen` — Nur Jurek/Mods posten, Announcement-Channel

### Kategorie: COMMUNITY (nach Verify sichtbar)
- `#allgemein` — Daily Talk, Smalltalk
- `#check-in` — Tages-/Wochen-Check-in (strukturiertes Teilen)
- `#wins-und-durchbrüche` — Erfolge, egal wie klein
- `#struggles` — wenn's gerade kracht, Support-Channel
- `#ressourcen` — kuratierte Links (Bücher, Videos, Podcasts)

### Kategorie: ARBEIT (Themen-Deep-Dives)
- `#schattenarbeit` — konkrete Schatten-Prozesse, Journal-Prompts
- `#wut-und-grenzen` — Trigger, Konflikt, Grenzziehung
- `#angst-und-scham` — Core-Themen
- `#beziehungen` — Partnerschaft, Sex, Eltern
- `#körper-und-nervensystem` — Somatic, CPPS, Polyvagal, Sport

### Kategorie: JUREK (teils rollen-gated)
- `#youtube-videos` — jedes neue Video wird hier gepostet + Diskussion
- `#jurek-fragt` — Jurek postet wöchentliche Frage
- `#ama-archiv` — Aufzeichnungen Q&A (rollen-gated für `Insider`)

### Kategorie: VOICE
- `Stammtisch` — offen, max 10
- `Stille-Session` — Pomodoro-Journaling, Video aus
- `Check-in-Call` — Sonntag 20:00 live
- `AFK` — Auto-Move nach 10min Inaktivität

### Kategorie: MOD (versteckt, nur Mods)
- `#mod-chat`, `#mod-logs`, `#bot-befehle`

**Public vs. Rollen-Gated:**
- Public vor Verify: nur `#willkommen`, `#regeln`
- Nach Reaktion auf Regeln (Rolle `Verified`): alles außer `#ama-archiv` und Mod
- `Insider` (zahlende Session-Kunden): zusätzlich `#ama-archiv`, exklusive Voice-Slots

---

## 3. Rollen-System

| Rolle | Farbe | Vergabe | Zugriff |
|---|---|---|---|
| `Gründer` | dunkelrot | manuell (Jurek) | alles |
| `Mod` | rot | manuell | alles außer Admin-Configs |
| `Insider` | gold | manuell nach Session-Kauf / via Stripe-Webhook | +AMA-Archiv |
| `Verified` | weiß | auto nach Regel-Reaktion | Community + Arbeit |
| `YouTuber` | blau | Reaction-Role Selbstauswahl | kosmetisch |
| `Newcomer` | grau | Default beim Join | nur Start-Kategorie |

**Interessen-Rollen (Self-Assign via Reaction-Role):**
- `Schattenarbeit`, `Körperarbeit`, `Beziehungen`, `Vaterthema`, `Sucht/Porno`, `CPPS/Körper`

Nur zum pingen für Themen-Threads. Keine Zugriffs-Logik.

**Wichtig:** Keine Level-Rollen via MEE6 XP. Das triggert Spam-Farming und passt nicht zur ernsten Nische.

---

## 4. Welcome-Flow (Bot-Text Copy-Paste)

### Carl-Bot Welcome-Message (DM beim Join):
```
Hey, schön dass du da bist.

Das hier ist kein Guru-Discord. Keine Hustle-Sprüche. 
Männer 25–44, die ehrlich an sich arbeiten. Schatten statt Bypass.

3 Schritte, dann bist du drin:
1) Lies #regeln und reagiere mit dem Haken
2) Stell dich in #vorstellung kurz vor (Template ist pinned)
3) Schnapp dir Interessen-Rollen in #rollen-wählen

Wenn's klemmt: @Jurek oder @Mod pingen.
```

### #regeln Channel-Text:
```
7 REGELN – bitte alle lesen.

1. Respekt. Keine Abwertung, kein Spott. Bei Konflikt PN oder Mod.
2. Keine ungefragten Ratschläge. Frag: "Willst du Input oder Raum?"
3. Vertraulichkeit. Was hier geteilt wird, bleibt hier.
4. Kein Guru-Selling. Wer eigene Dienste pusht ohne Absprache → Ban.
5. Keine Politik-Debatten, kein Covid-Krieg, keine Impf-Diskussionen.
6. Triggerwarnung bei Suizid/Missbrauch/Gewalt: || Spoiler-Tags ||.
7. Bei akuter Krise: Telefonseelsorge 0800-1110111. Discord ist kein Therapie-Ersatz.

Reagiere mit ✅ um Zugang zu bekommen.
```

### #vorstellung Template (pinned):
```
📍 Wer bist du?
- Vorname / Alter / grob wo du lebst
- Was dich hergebracht hat (1–2 Sätze)
- Was du gerade bearbeitest (Thema, nicht Story-Roman)
- Was du NICHT willst (z.B. "keine Ratschläge", "nur lesen erstmal")
- Ein Song / Buch / Satz, der dich gerade begleitet
```

### Reaction-Roles (Interessen, Channel `#rollen-wählen`):
```
Welche Themen ziehen dich? (mehrere möglich)

🌑 → Schattenarbeit
🔥 → Wut & Grenzen
💧 → Angst & Scham
❤️ → Beziehungen & Sex
🫁 → Körper & Nervensystem
👨 → Vaterthema
🎥 → YouTube-Notifications
```

---

## 5. Sicherheit + Moderation

**Bots:**
- **Carl-Bot** (Primary) — Reaction-Roles, Auto-Mod, Welcome, Embeds. Besser als MEE6 (kein Paywall-Gating).
- **Wick** (Anti-Raid) — Scammer-Welle-Abwehr, Phishing-Link-Scan.
- Optional: **Statbot** — Aktivitäts-Dashboard für Jurek.

**NICHT** MEE6 nehmen: Paywall für Basics, XP-Spam-Incentives passen nicht zur Nische.

**Auto-Mod-Regeln (Carl-Bot + Discord-Native):**
- Wort-Blacklist: Slurs, rassistische Begriffe, Onlyfans/Porn-Spam-Keywords
- Link-Filter: Whitelist nur für `youtube.com`, `emotionale-meisterschaft.de`, Jureks Domains
- Invite-Link-Filter: andere Discord-Invites automatisch gelöscht
- Spam: 5 identische Messages in 10s → Mute 10min
- Mention-Spam: >5 Mentions in einer Message → Warn
- Caps-Lock: >70% Großbuchstaben bei >10 Zeichen → Warn
- Neu-Account-Filter: Discord-Account <7 Tage alt → Auto-Kick + Log (Scammer-Standard)

**Member-Verification:**
- Discord Community-Feature aktivieren: "Verified Email" Pflicht
- Verification-Level auf "High" (verifizierte Telefonnummer bei Discord)
- Regel-Reaktion = Rolle `Verified` (Gating)
- Bei Verdacht: manuelle Frage im `#willkommen` vor Freischaltung

---

## 6. Engagement-Rituale

### Wöchentlich (fix getaktet):

- **Montag 08:00** — `#check-in`: "Worum geht's diese Woche innerlich? 1 Satz."
- **Mittwoch 19:00** — `#jurek-fragt`: wöchentliche Tiefen-Frage (siehe unten)
- **Freitag 17:00** — `#wins-und-durchbrüche`: "Was hat sich diese Woche bewegt?"
- **Sonntag 20:00** — Voice-Channel `Check-in-Call`: 60min moderierter Austausch, max 12 Leute

### 12-Wochen-Fragen-Plan (`#jurek-fragt`):
1. Welchen Teil von dir verbirgst du vor deiner Partnerin/Familie?
2. Was macht dein Vater in dir, wenn du ehrlich bist?
3. Wo performst du noch, statt zu fühlen?
4. Was ist dein teuerster Kontroll-Mechanismus?
5. Welche Wut hast du nie ausgedrückt — und an wen?
6. Was hast du "spirituell bypasst", was eigentlich Trauer war?
7. Wo wartest du auf Erlaubnis, die nie kommt?
8. Was willst du, traust dich aber nicht zu wollen?
9. Wann hast du zuletzt geweint — und was hat es ausgelöst?
10. Welches Nein schuldest du dir seit Jahren?
11. Wer wärst du ohne dein Leistungs-Narrativ?
12. Was würdest du tun, wenn keiner applaudiert?

### Monatlich:
- **Letzter Sonntag im Monat, 20:00** — Jurek Live-AMA Voice (90min). Aufzeichnung in `#ama-archiv` (Insider-only).

### Trigger-Events (optional, nach 100+ Member):
- Buch-Club: 1 Buch / Monat, Thread im `#ressourcen`
- 7-Tage-Journal-Sprints (Carl-Bot reminder)

---

## 7. Integration mit dem Funnel

### YouTube → Discord:
- Jedes Video: Pinned Comment + Description-Link: "Community → discord.gg/XXXX"
- End-Screen: "Wenn du tiefer gehen willst: Discord-Link unten"
- Discord-Invite permanent, nicht ablaufend, custom Vanity-URL sobald Level 3 Boost

### Discord → Quiz → Session:
- `#ankündigungen` Pinned Post: "Wenn du rausfinden willst, wo du stehst → Kompass-Quiz [Link]"
- Bot-Command `!kompass` → DM mit Quiz-Link
- Nach Quiz-Abschluss (Email capture) → Automation (ConvertKit/MailerLite) → Session-Angebot Tag 3
- Session-Käufer bekommen manuelle Insider-Rolle via Stripe-Webhook → Zapier → Carl-Bot

### Was NICHT über Discord verkauft wird:
- **Keine** 1:1-Session-Pitches im Chat. Niemals.
- **Keine** Affiliate-Links, auch nicht "kleine" Empfehlungen
- **Keine** Launch-Announcements mehr als 1x/Monat
- **Keine** DMs von Jurek mit Angeboten (zerstört Trust sofort)
- Regel: Discord ist Beziehung, Landing ist Verkauf. Trennung heilig.

---

## 8. Startplan – Erste 30 Tage

### Tag 0 (Setup-Tag, 2h Block):
- Server erstellen, Community-Feature an
- Kategorien + Channels anlegen (nach Struktur oben)
- Carl-Bot + Wick einladen, konfigurieren
- Welcome-Message, Regeln, Reaction-Roles testen
- 3 Test-Accounts (Handy, Zweit-Discord, Freund) durch Onboarding jagen → Bugs fixen
- Invite-Link generieren, noch NICHT posten

### Tag 1–7 (Seed-Phase, 20–50 Leute):
- Handverlesen einladen: 20–50 Leute aus bestehender Email-Liste / engste YouTube-Commenter / Kuschelparty-Bekannte
- Persönliche DM/Email: "Bau gerade Community auf, will dich als Gründungsmitglied, ehrliches Feedback"
- Jeden Tag selbst aktiv: 3 Posts, 5 Reaktionen, 2 Voice-Slots
- Keine Public-Ankündigung in YouTube-Videos
- Am Tag 7: 3 Gründungsmitglieder zu Mod machen (Vertrauen, nicht Follower)

### Tag 7–30 (kontrolliertes Wachstum):
- **Tag 8:** Erstes YouTube-Video mit Discord-Link in Pinned Comment (nicht im Video selbst)
- **Tag 14:** Erstes Voice-AMA (auch wenn nur 8 kommen)
- **Tag 15:** Discord-Link in Video-Outro einbauen
- **Tag 21:** Erste Kooperation (anderer YouTuber/Coach stellt sich im Server vor)
- **Tag 28:** Review-Runde: was funktioniert, was nicht? Channel mergen/löschen ohne Sentiment.
- **Tag 30:** Entscheidung: Vanity-URL? Boost-Campaign? Einladungs-System?

### KPIs Tag 30 (realistisch):
- 80–150 Member
- 20–30% Weekly-Active (nicht DAU jagen)
- 5+ Posts/Tag organisch ohne Jurek
- 1 Session-Sale direkt via Discord → Quiz → Landing

### Rote Flaggen:
- <5 Posts/Tag nach Tag 14 → zu viele passive Member, Onboarding nachschärfen
- Viel Volume aber kein Tiefgang → Regeln nachziehen, Kultur aktiv modellieren
- Jurek postet >50% aller Messages → Community ist fake, Gründungsmitglieder aktivieren

---

## Quick-Start-Checkliste (2h Timer)

- [ ] 0:00 Server erstellen, Community-Feature, Verify-Level High
- [ ] 0:15 Kategorien + Channels anlegen
- [ ] 0:40 Rollen anlegen + Permissions setzen
- [ ] 1:00 Carl-Bot einladen, Welcome/Regeln/Reaction-Roles konfigurieren
- [ ] 1:25 Wick Anti-Raid einrichten
- [ ] 1:40 2 Test-Accounts durch Onboarding → Bugs fixen
- [ ] 1:55 Invite-Link generieren, in Notion/Landing parken
- [ ] 2:00 Fertig. Morgen erste 5 Leute einladen.

---

Saved to discord-setup.md
