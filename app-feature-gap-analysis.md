# EM-App — Feature-Gap-Analyse gegen App-Store-Success-Patterns

**Datum:** 2026-04-21
**Version:** EM-App 1.0.1 (Expo 54 / RN 0.81 / Firebase 12.8)
**Zielgruppe:** Maenner 25-44, Schattenarbeit, 300 EUR Exklusives Paket

---

## 1. Aktueller Feature-Stand

### Tab-Struktur (Bottom Tabs)
1. **Guide** (AIRecommendationScreen) — AI-gesteuerte Empfehlung (sparkles icon)
2. **Kurs** (CourseScreen) — 750+ Videos Hauptinhalt
3. **Medi** (MeditationsScreen) — 12 Meditationen
4. **Chat** (ChatScreen) — Community
5. **Mehr** (ProfileScreen) — Alles andere

### Stack-Screens (55+ Screens!) — Auszug
Core Therapy: Journal, WeeklyReview, Selbsttest, EmotionWheel, EmotionalCompass, TriggerTracker, InnerChild, PartsWork, Somatic, BodyScan, BodyEmotionMap, Breathwork, HealingCircle

Coaching: Coaches, FreeCoaching, Booking, CoachingHub, CoachingHubClassic, Supervision, PersonalQuestion, ProjectRequest, SessionNotes, VoiceSession

Content & Tools: VideoPlayer, Soundscape, RainGenerator, MediMixer, Affirmation, StoryMode, DreamJournal, VisionBoard, MoonRitual, Transcriber

Gamification/Tracking: Tracker, HabitTracker, Journey, GrowthDashboard, Game, Analytics, HealthTracker, CommunityChallenge, WeeklyDigest

Community: VoiceChannel, Events, CommunityChallenge

### Services (vorhanden aber unklar ob voll genutzt)
`gamificationService.js` (XP + 50 Levels), `xpService.js`, `achievementsService.js`, `streakService` (in notification), `karmaService.js`, `aiService.js`, `recommendationService.js`, `reviewPromptService.js`, `stripeService.js` + RevenueCat

### Onboarding
5 Screens (TOTAL_PAGES = 5), KEIN Quiz/Assessment/Goal-Setting — reine Slideshow mit Partikel-Eyecandy.

---

## 2. Meta-Patterns aus erfolgreichen Apps

| Pattern | Calm | Finch | Duolingo | Headspace |
|---|---|---|---|---|
| Onboarding | 60-90s, 8-12 Fragen | 2-3 Min, Name+Bird+Goals | 90s, Goal+Level+Reminder | 45-60s, 3 Ziele |
| Day-1 Hook | Guided 3min Intro | First check-in | First lesson + owl reminder | 3-min Basics |
| Day-7 Hook | Streak + Series | Streak + Bird energy | Streak shield + leagues | Streak + stat cards |
| Paywall | Nach Onboarding + Content-Gate | Soft (Energie-Limit) | Nach Level 5 + Hearts | Nach Day 1 Session |
| Notifications | Smart Reminder, 1-2x/day | Bird needs you, 2-3x/day | Streak danger, 1-3x/day | Bedtime + Morning |
| Social Proof | 4.8 Sterne in Paywall | Freunde-Bird-System | Leaderboards | Testimonials in Paywall |
| Gamification | Streaks, Mindful-Minutes | Rainbows, Items, Outfits | Streaks, XP, Leagues, Gems | Streaks, Pro Badge |

---

## 3. Gap-Report

| Pattern | EM hat? | Impact | Kommentar |
|---|---|---|---|
| **Onboarding Personalisierung** | NEIN (nur Slideshow) | **HIGH** | Kein Quiz, kein Name-Input, kein Ziel-Setting. User kommt rein und sieht generische Tab-Bar. |
| **Day-1 Retention Mechanik** | TEILWEISE | **HIGH** | Es gibt Guide-Tab mit AI, aber kein expliziter 3-Min "First Win" Flow. |
| **Day-7 Retention (Streak)** | TEILWEISE | **MED** | `streakService` existiert, `gamificationService` auch — aber nicht prominent in Home/Guide sichtbar. |
| **Paywall-Trigger** | UNKLAR | **HIGH** | MembershipScreen existiert, aber kein klarer Content-Gate-Trigger nach X Videos/Sessions. |
| **Notification-Strategie** | JA | **MED** | Sehr gut: Morning Check-in, Evening Journal, Streak Warning, max 3/Tag. Nur: default-off fuer die wirksamsten (streakWarning, eveningJournal). |
| **Social Proof in-App** | NEIN | **HIGH** | Keine Testimonials, keine Ratings-Prompt auf Paywall, keine "X Maenner nutzen das heute". |
| **Streaks sichtbar** | TEILWEISE | **MED** | Service da, aber nicht als Hero-Element auf Home/Guide. |
| **Badges/Achievements** | JA (Service) | **MED** | `achievementsService` existiert — Frage ob UI-sichtbar + emotional belohnend. |
| **Levels/XP** | JA (50 Levels!) | **LOW** | Existiert. Risk: Zu komplex fuer Zielgruppe Maenner 25-44 die Schattenarbeit machen (nicht Duolingo-Gaming). |
| **Progress Sharing (Viral)** | NEIN | **MED** | Keine "Share Progress Card", keine Referral. |
| **Leaderboards** | NEIN | **LOW** | Fuer Schattenarbeit sogar kontraproduktiv (scham-triggernd). |
| **Review Prompt** | JA | **LOW** | `reviewPromptService` existiert. |
| **Deep Links** | JA | - | Vorbildlich umgesetzt. |
| **Offline** | JA | - | OfflineBanner + offlineService. |
| **Error Boundaries** | JA | - | Vorbildlich — jeder Screen einzeln gewrapt. |

---

## 4. Top 5 Feature-Vorschlaege (Impact/Effort)

### #1 — Onboarding-Quiz "Dein Schatten-Profil" (HIGH Impact, M Effort)
**Was:** 6-8 Fragen direkt nach Splash. Ersetzt/erweitert das aktuelle 5-Page-Slideshow-Onboarding.
- "Was bringt dich her?" (Wut / Leere / Beziehung / Sucht)
- "Wie oft triggert es dich?" (taeglich / woechentlich / selten)
- "Was hast du schon probiert?"
- "Was ist dein Ziel in 90 Tagen?"

**Warum:** Finch + Noom zeigen: personalisiertes Onboarding steigert D7-Retention um 30-60%. Zielgruppe Maenner brauchen "diagnostisches" Einstieg — bestaetigt Problem, schafft Commitment.

**Wie:**
- Neue Screens in `OnboardingScreen.js` oder neue `OnboardingQuizScreen.js`.
- Ergebnis als `user_profile` in Firestore speichern.
- `recommendationService.js` nutzt Profil um Guide-Tab mit personalisiertem ersten Content zu oeffnen.
- Endscreen zeigt "Dein Schatten-Profil: Der Verwundete Krieger — 73% der Maenner mit deinem Profil haben nach 14 Tagen X erreicht".

**Aufwand:** M (2-3 Tage). **Impact:** Aktivierung + Retention.

---

### #2 — "Heute" Dashboard statt Guide-Tab als Landing (HIGH Impact, S Effort)
**Was:** Guide-Tab zeigt beim Oeffnen: Streak-Counter (gross), Daily Check-in CTA, 1 empfohlenes Video/Meditation, Progress zum Wochenziel.

**Warum:** Calm/Headspace machen das genau so. User oeffnet App -> sieht sofort WAS tun + Fortschritt. Aktuell: AI-Recommendation ist gut, aber fehlender Streak-Hero + Day-Number ("Tag 12 deiner Reise") killt Momentum-Feeling.

**Wie:**
- In `AIRecommendationScreen.js` (=Guide-Tab) oben fix:
  - Streak-Card: "12-Tage-Flamme" mit goldenem Glow
  - "Tag 12 / 90 deiner Reise"
  - Today's Check-in Button
- Daten aus `gamificationService` + `streakService` nutzen — existiert schon.

**Aufwand:** S (1 Tag). **Impact:** Retention D3-D14.

---

### #3 — Content-Gated Paywall nach 3 Videos (HIGH Impact, M Effort)
**Was:** Nach 3 konsumierten Lektionen / 1 Meditation FULL -> Paywall-Screen mit:
- Testimonials (3 Maenner, Namen + Foto + 2 Zeilen)
- "750 Videos + 12 Meditationen + 3 Sessions mit Jurek" klare Value Stack
- Einmalig 300 EUR vs. monatlich X (Anchoring)
- 7-Tage Geld-zurueck-Garantie

**Warum:** Aktuell unklar WANN Paywall triggert. Best practice: nach Aha-Moment, nicht vorher. Duolingo + Calm warten bis User Wert gespuert hat.

**Wie:**
- In `VideoPlayerScreen.js` + `MeditationsScreen.js`: Counter in AsyncStorage (`@videos_watched_free`).
- Bei >=3: Navigation-Intercept -> `Membership` screen.
- `MembershipScreen.js` um Testimonials + Value-Stack erweitern.
- Social Proof: `img-testimonial-1/2/3.png` aus Landing wiederverwenden.

**Aufwand:** M (2 Tage). **Impact:** Revenue (direkt).

---

### #4 — Daily Check-in Widget mit 30-Sekunden-Commitment (MED Impact, S Effort)
**Was:** Auf Home-Tab prominenter Button "Wie geht's dir gerade? (30 Sek)". Oeffnet Mini-Flow: Emotion-Wheel-Pick -> Intensitaet 1-10 -> optional 1 Satz -> Done (mit XP + Streak +1).

**Warum:** Finch macht das mit "Daily Reflection" — der KRITISCHE Hook. User muss einmal am Tag die App oeffnen MIT Reward. 30 Sek = low-friction, aber genug fuer Streak-Ausloesung.

**Wie:**
- `EmotionWheelScreen.js` existiert schon — Mini-Version als Bottom-Sheet auf Home.
- Ergebnis -> `journal` collection + `streakService.checkin()` + XP-Toast.
- Notification 08:00 bereits konfiguriert — nur einschalten by default.

**Aufwand:** S (1 Tag). **Impact:** Retention (die wichtigste Metrik).

---

### #5 — Share-Progress-Card (MED Impact, S Effort)
**Was:** Nach Meilenstein (Tag 7, 30, 90, Level-Up) Screen mit schoener goldener Card -> "Als Bild teilen". WhatsApp/Insta-Share triggern.

**Warum:** Peloton + Strava's Wachstum kam von Share-Cards. Pro geteiltes Screenshot = potenziell 1-2 neue User. Maenner 25-44 = Whatsapp-heavy.

**Wie:**
- `react-native-view-shot` + `expo-sharing` (letzteres schon installiert).
- Nach `gamificationService.levelUp()` oder `streakService` Meilensteinen -> Modal.
- Design: Dunkelblau+Gold, "Tag 30 | Jurek's Emotionale Meisterschaft" + eigener Name.

**Aufwand:** S (1-2 Tage). **Impact:** Virality + Retention (Pride-Loop).

---

## 5. Die 3 Features die JETZT weghauen sollten

### WEG #1 — Uebermaessige Tool-Fragmentierung (BodyEmotionMap, PartsWork, InnerChild, Somatic, HealingCircle, MoonRitual)
**Problem:** 6+ ueberlappende "Tiefenarbeit"-Screens die alle aehnliche Uebungen machen. BridgeMind-Lehre "Stop Overbuilding" exakt hier getroffen. User findet nichts, Entscheidungsmuedigkeit, keiner weiss welche Uebung wann.

**Aktion:** 
- Konsolidieren in EINEN Screen "Tiefenarbeit" mit Tabs oder Filter (Koerper / Innere Anteile / Inneres Kind / Ritual).
- Oder: NUR die Top-2 behalten (was Analytics zeigt) — Rest loeschen oder hinter Feature-Flag verstecken.
- `featureFlagService.js` existiert schon — nutzen.

---

### WEG #2 — GameScreen + react-native-game-engine + three.js + @shopify/react-native-skia
**Problem:** Das sind 3 schwere Dependencies fuer EIN Spiel (GameScreen.js). three.js allein = ~600 KB. Fuer eine Therapie-App fuer Maenner 25-44 mit Schattenarbeit ist ein GAME nicht Kernprodukt — Duolingo-Gamification passt nicht zu CPPS/Schatten-Zielgruppe.

**Aktion:**
- GameScreen + Game-Route rausnehmen.
- `react-native-game-engine` + `three` + evtl. `@shopify/react-native-skia` rausschmeissen.
- Bundle-Size-Ersparnis: ~1-2 MB. Performance-Gewinn auf Low-End-Android spuerbar.

---

### WEG #3 — CoachingHubClassic (Legacy-Duplikat) + TranscriberScreen + Supervision (wenn nicht fuer Admin)
**Problem:** 
- `CoachingHubClassic` ist offensichtlich alte Version neben `CoachingHub`. Klassisches "ich hab neues gebaut aber altes nicht geloescht" — verwirrt User und bringt Maintenance-Schuld.
- `TranscriberScreen` ist ein internes Tool — User brauchen das nicht.
- `SupervisionScreen` ist Admin-Feature (wird auch nur via `AdminSessionRequestNotifier` navigiert) — sollte aus normaler User-Nav komplett weg und nur fuer `isAdminEmail` sichtbar sein.

**Aktion:**
- `CoachingHubClassic` loeschen, Route raus, Datei raus.
- `TranscriberScreen` raus (oder Admin-only).
- `SupervisionScreen` bleibt, aber Entry-Points nur fuer Admin-User rendern.

---

## Zusammenfassung (TL;DR)

**Starke Seite:** Solide Technik-Fundament (Error Boundaries, Offline, Deep Links, Push, RevenueCat, Firebase). 55+ Features = beeindruckend aber ueberkomplex.

**Kritische Gaps:** (1) Kein Personalisierungs-Onboarding, (2) kein prominentes Streak/Daily-Dashboard, (3) kein klar definierter Paywall-Trigger nach Aha-Moment, (4) keine Testimonials/Social Proof in-App, (5) keine Share-Mechanik.

**Strategische Empfehlung:** Drei Wochen Fokus auf Feature #1-3 (Quiz, Today-Dashboard, Content-Paywall) + parallel die 3 "Weg"-Punkte loeschen. Das ist der klassische BridgeMind-Pattern: **Cut 30%, Polish the Core, Add the Hook**. Bundle wird kleiner, Retention geht hoch, Conversion zum 300 EUR Paket steigt.

**Erwartete Effekte (Annahmen):**
- D1 Retention: +15-25% durch Quiz + Today-Dashboard
- D7 Retention: +20-30% durch Daily Check-in + Streak Hero
- Conversion auf 300 EUR: +30-50% durch Content-Gated Paywall mit Social Proof
- App-Bundle: -2 MB durch Dead-Code-Elimination

Saved to C:/Users/Jurek/emotionale-meisterschaft-landing/app-feature-gap-analysis.md
