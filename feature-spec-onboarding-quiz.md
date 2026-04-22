# Feature-Spec: In-App Shadow-Quiz Onboarding

**Version**: 1.0
**Stand**: 2026-04-21
**Owner**: Jurek
**Zielplattform**: React Native / Expo 54 (`emotionale-meisterschaft-app`)
**Abhängigkeit**: Web-Quiz `quiz.html` (4-Shadow-Typen-Algorithmus, live getestet)

---

## 0. TL;DR

Nach den bestehenden 5 Onboarding-Slides wird ein 8-Fragen-Quiz eingeschoben, das den User einem von 4 Shadow-Typen zuordnet (**Krieger / Abwesender / Ungenügender / Kontrolleur**). Ergebnis landet in Firestore (`users/{uid}.shadowType`) und wird ab sofort in Guide, AI-Recommendations, Journal-Prompts und Meditationen als Personalisierungs-Signal benutzt. Ein Skip-Pfad ist erlaubt, aber visuell abgeschwächt. Re-Take aus dem Profil heraus ist jederzeit möglich.

---

## 1. User-Flow (Schritt für Schritt)

### 1.1 Bereits existierende Screens

| Screen | Pfad | Funktion |
|---|---|---|
| `OnboardingScreen.js` | `src/screens/OnboardingScreen.js` | 5-Page-Slideshow mit Particles, GlowIcon, Counter; endet mit `handleComplete()` → setzt `@has_seen_onboarding = true` und ruft `onComplete` |
| `AuthScreen.js` | `src/screens/AuthScreen.js` | Login/Signup; schreibt `users/{uid}` initial |
| `TabNavigator` / Guide-Tab | `src/navigation/TabNavigator.js` | Haupt-Dashboard nach Login |
| `SelbsttestScreen.js` | `src/screens/SelbsttestScreen.js` | Existierender Self-Test (anderes Format) |

### 1.2 Neue Screens

1. **`ShadowQuizScreen.js`** — Intro → 8 Fragen → Result → Weiter. Ersetzt den heutigen `handleComplete()`-Exit nicht, sondern wird davor eingehängt.
2. **`ShadowResultScreen.js`** (optional als separater Screen, im MVP als Sub-View innerhalb des Quiz-Screens via State — einfacher, keine Navigation-Tricks nötig).

### 1.3 Integrations-Flow

```
App-Start
  └─ AsyncStorage: @has_seen_onboarding ?
       ├─ nein → OnboardingScreen (5 Slides)
       │         └─ "JETZT STARTEN" → setItem @has_seen_onboarding
       │                              → Navigation zu ShadowQuiz (neuer Gate)
       │         └─ "Anmelden" (Login-Shortcut) → AuthScreen (skippt Quiz, kann später triggered werden)
       │
       ├─ ja + kein shadowType in Firestore → Bei nächstem App-Start "Quiz nachholen"-Prompt im Guide
       └─ ja + shadowType gesetzt → normale App
```

**Entscheidung**: Das Quiz läuft **vor** der Auth (anonym), damit User zuerst Value kriegt, bevor Email gefordert wird. Resultat wird in AsyncStorage gepuffert (`@pending_shadow_type`) und beim ersten Login auf `users/{uid}` geflusht.

### 1.4 Schritt-für-Schritt Flow

| # | Screen | Aktion | Daten-Event |
|---|---|---|---|
| 1 | Onboarding Slide 1–5 | Swipen | — |
| 2 | Onboarding Slide 5 | "JETZT STARTEN" Tap | `@has_seen_onboarding = true` |
| 3 | **Quiz Intro** | "Test starten" | `logEvent('shadow_quiz_start')` |
| 4 | **Quiz Q1–Q8** | Antwort tippen | `logEvent('shadow_quiz_answer', {q, type})` |
| 5 | **Quiz Result** | "Ergebnis ansehen" | `logEvent('shadow_quiz_complete', {type})`; `@pending_shadow_type = wut\|leere\|angst\|kontrolle` |
| 6 | Result → "Weiter zu deinem Guide" | Navigation zu Tabs | — |
| 7 | AuthScreen (wenn User sich später registriert) | — | `users/{uid}.shadowType = pending`, AsyncStorage clearen |

---

## 2. Daten-Modell

### 2.1 Firestore — `users/{uid}`

Neue Felder (additiv, keine Breaking Changes):

```js
{
  // ...bestehende Felder...
  shadowType: "wut" | "leere" | "angst" | "kontrolle" | null,
  shadowTypeLabel: "Der Krieger" | "Der Abwesende" | "Der Ungenügende" | "Der Kontrolleur",
  shadowTypeCompletedAt: Timestamp,
  shadowTypeVersion: 1,  // bump on algorithm change — erlaubt Re-Takes zu erzwingen
  shadowTypeAnswers: [
    { q: 0, a: 2, type: "angst" },  // Frage-Index, Antwort-Index, resultierender Typ
    // ... 8 Einträge
  ],
  shadowTypeScores: { wut: 2, leere: 1, angst: 4, kontrolle: 1 },  // Raw counts
  shadowTypeHistory: [                    // Nur bei Re-Takes angehängt
    { type: "wut", at: Timestamp, scores: {...} }
  ]
}
```

### 2.2 Persistenz-Schichten

| Schicht | Zweck | TTL |
|---|---|---|
| AsyncStorage `@pending_shadow_type` | Zwischenspeicher bis Login | Bis Login |
| AsyncStorage `@shadow_quiz_answers` | Antworten lokal (für Re-Analyse offline) | Permanent, bis Re-Take |
| Firestore `users/{uid}.shadowType` | Single source of truth | Permanent |
| In-Memory (AuthContext) | Runtime-Personalisierung | Session |

### 2.3 Re-Evaluation

Der Shadow-Type wird **nicht automatisch** re-evaluated. Re-Take passiert explizit durch User:

- **Profil-Tab → "Shadow-Quiz wiederholen"** → Reset, neuer Run, altes Ergebnis landet in `shadowTypeHistory[]`
- **App-Version-Bump** mit `shadowTypeVersion++` → sanfter Prompt beim nächsten Start "Wir haben das Quiz überarbeitet, 2min?"

### 2.4 Firestore Security Rule (additiv)

```
match /users/{uid} {
  allow update: if request.auth.uid == uid
    && request.resource.data.shadowType in ['wut','leere','angst','kontrolle', null];
}
```

---

## 3. UX-Details

### 3.1 Animations-Stil

Wiederverwendung der bestehenden Design-Tokens und Patterns aus `OnboardingScreen.js`:

- **Particle-Field** (NUM_PARTICLES=28, gold) als Hintergrund
- **GlowIcon** für Intro + Result (Icon passend zum Shadow-Type: `flame` / `moon` / `heart-dislike` / `settings`)
- **Card-Container** mit `PANEL_BG` / `PANEL_BORDER` (gleicher Glassmorph-Stil)
- **Fade+Scale Transition zwischen Fragen**: 400ms ease-out, wie `OnboardingPage`-Animation (`fadeAnim`, `scaleAnim`, `slideAnim` in parallel)
- **Progress-Bar oben**: Gold-Gradient wie im Web-Quiz, animiert über `Animated.timing` auf `width`

### 3.2 Back-Button Verhalten

| Position | Verhalten |
|---|---|
| Intro-Screen | Zurück zum letzten Onboarding-Slide (swipe-right-Geste) |
| Fragen 1–8 | Zurück zur **vorigen** Frage (State rollback: `counts[prevType]--`, `qi--`) |
| Result-Screen | Kein Back (ist Ende) — "Weiter" oder "Wiederholen" |

Android-Hardware-Back-Button: via `useFocusEffect` + `BackHandler.addEventListener` auf denselben Handler.

### 3.3 Skip-Button

**Ja, erlauben — aber visuell abgeschwächt** (ein einziger unauffälliger Text-Link, keine Button-Box):

- **Intro-Screen**: "Überspringen" klein unten, `WHITE_DIM` Color
- **Fragen-Screen**: Kein Skip mehr — nur Back und nächste Antwort wählbar (User hat sich committed)

Risiko-Mitigation: Wer Intro skippt, kriegt **nach 3 App-Öffnungen** einen sanften Reminder im Guide-Dashboard: "Lern deinen Shadow kennen — 2 Min." mit einer Dismiss-Option.

### 3.4 Result-Screen — Inhalt

```
[GlowIcon passend zum Typ — flame/moon/heart-broken/cog]

DEIN DOMINANTER SCHATTEN          ← Label, gold, uppercase, letter-spacing
Der Krieger                        ← Title, 38px, fontWeight 900

"{firstName}, Wut ist dein         ← Personalized desc (firstName kann leer sein)
 Hauptkanal. Sie schützt dich..."

────────────────────────────────

Was das für dich bedeutet          ← H2, 22px
{meaning-Text, 16px, line-height 26}

Die nächsten 3 Schritte            ← H2
 1. Entkoppeln: 3 Atemzüge...
 2. Ursprung finden: Trigger...
 3. Arbeit darunter: Die Wut...

────────────────────────────────

[  Weiter zum Guide  ]             ← Primary Gold-Gradient Button
  Quiz wiederholen                  ← Secondary outline
```

Result-Screen **scrollbar**, da Content länger ist. CTA am Ende plus zusätzlich als sticky-Footer während des Scrolls — damit User nicht suchen muss.

---

## 4. Technical Implementation

### 4.1 Neue Datei-Struktur

```
src/
  screens/
    ShadowQuizScreen.js         ← Haupt-Screen (Intro/Quiz/Result)
  data/
    shadowQuizContent.js        ← Fragen + Results (portiert aus quiz.html)
  services/
    shadowTypeService.js        ← Firestore-Read/Write + AsyncStorage Sync
  utils/
    shadowPersonalization.js    ← Helper: getShadowContent(type, context)
```

### 4.2 Navigation-Integration

- **App.js**: Nach Onboarding-Completion + wenn noch kein `shadowType` → ShadowQuiz mounten, bevor Tabs rendern. Einfachster Weg: State in `App.js` ergänzen um `hasCompletedShadowQuiz`.
- **MainNavigator.js**: Neue Stack-Screen `ShadowQuiz` hinzufügen (falls User Re-Take aus Profil triggert).

### 4.3 Firestore-Write Sequence

```js
async function saveShadowType(uid, type, scores, answers) {
  const ref = doc(db, 'users', uid);
  const prev = await getDoc(ref);
  const prevType = prev.exists() ? prev.data().shadowType : null;

  const history = prev.exists() && prev.data().shadowTypeHistory || [];
  if (prevType && prevType !== type) {
    history.push({ type: prevType, at: prev.data().shadowTypeCompletedAt, scores: prev.data().shadowTypeScores });
  }

  await setDoc(ref, {
    shadowType: type,
    shadowTypeLabel: LABELS[type],
    shadowTypeCompletedAt: serverTimestamp(),
    shadowTypeVersion: 1,
    shadowTypeScores: scores,
    shadowTypeAnswers: answers,
    shadowTypeHistory: history
  }, { merge: true });

  await AsyncStorage.removeItem('@pending_shadow_type');
}
```

### 4.4 Offline-Fallback

Wenn `saveShadowType` fehlschlägt (offline / Firestore unreachable): Resultat bleibt in AsyncStorage. Beim nächsten App-Start prüft `AuthContext`, ob `@pending_shadow_type` existiert und ob `users/{uid}.shadowType` leer ist — falls ja, sync'en.

---

## 5. Kerncode — Production-Ready

### 5.1 `src/data/shadowQuizContent.js`

```js
// Portiert aus emotionale-meisterschaft-landing/quiz.html
// Single source of truth für Fragen + Results

export const SHADOW_QUESTIONS = [
  {
    q: "Wie reagierst du, wenn jemand dich kritisiert oder angreift?",
    a: [
      { t: "Ich werde innerlich sofort heiß. Es kocht.", s: "wut" },
      { t: "Ich schalte ab. Nichts mehr.", s: "leere" },
      { t: "Ich überlege sofort, was ICH falsch gemacht habe.", s: "angst" },
      { t: "Ich analysiere, wie ich die Situation wieder unter Kontrolle kriege.", s: "kontrolle" },
    ],
  },
  {
    q: "Was passiert nachts um 3, wenn du nicht schlafen kannst?",
    a: [
      { t: "Wut über jemanden oder etwas Bestimmtes — es kreist.", s: "wut" },
      { t: "Ein großes Nichts. Als wäre ich nicht wirklich da.", s: "leere" },
      { t: "Angst. Sorgen. Was wenn... was wenn... was wenn.", s: "angst" },
      { t: "Pläne, To-Do-Listen, Szenarien durchspielen.", s: "kontrolle" },
    ],
  },
  {
    q: "Wenn du ehrlich bist — was vermeidest du in Beziehungen?",
    a: [
      { t: "Dass jemand mir zu nahe kommt und mich provoziert.", s: "wut" },
      { t: "Ernsthafte Nähe überhaupt. Die strengt mich an.", s: "leere" },
      { t: "Verlassen zu werden. Lieber gehe ich zuerst.", s: "angst" },
      { t: "Kontrolle abzugeben. Mich wirklich fallen zu lassen.", s: "kontrolle" },
    ],
  },
  {
    q: "Als Kind warst du vermutlich der Junge, der...",
    a: [
      { t: "...öfter in Streit geraten ist oder sich gewehrt hat.", s: "wut" },
      { t: "...sich irgendwann in seine Welt zurückgezogen hat.", s: "leere" },
      { t: "...immer versucht hat, es allen recht zu machen.", s: "angst" },
      { t: "...der vernünftige, erwachsene unter den Kindern war.", s: "kontrolle" },
    ],
  },
  {
    q: "Welchen Satz kennst du innerlich am besten?",
    a: [
      { t: "\"Lass mich in Ruhe.\"", s: "wut" },
      { t: "\"Ist eh egal.\"", s: "leere" },
      { t: "\"Ich bin nicht genug.\"", s: "angst" },
      { t: "\"Ich muss das schaffen.\"", s: "kontrolle" },
    ],
  },
  {
    q: "Wenn du dich betäuben musst — womit?",
    a: [
      { t: "Adrenalin, Sport bis zur Erschöpfung, Konfrontation.", s: "wut" },
      { t: "Netflix, Scrollen, Gaming — bis es dunkel wird.", s: "leere" },
      { t: "Essen, Alkohol, alles womit ich mich kleinmachen kann.", s: "angst" },
      { t: "Arbeit. Mehr arbeiten. Noch mehr arbeiten.", s: "kontrolle" },
    ],
  },
  {
    q: "Was denkst du über dein inneres Kind?",
    a: [
      { t: "Das wurde übergangen. Das hat eine Wut.", s: "wut" },
      { t: "Das ist irgendwann gegangen. Ich weiß nicht wohin.", s: "leere" },
      { t: "Das hatte dauernd Angst, nicht geliebt zu werden.", s: "angst" },
      { t: "Das musste zu früh funktionieren. Erwachsen sein.", s: "kontrolle" },
    ],
  },
  {
    q: "Was wäre die größte Veränderung, wenn du wirklich dahinter kommst?",
    a: [
      { t: "Ich müsste nicht mehr kämpfen. Einfach da sein.", s: "wut" },
      { t: "Ich würde wieder etwas fühlen. Egal was, aber fühlen.", s: "leere" },
      { t: "Ich würde aufhören, ständig zu beweisen, dass ich okay bin.", s: "angst" },
      { t: "Ich könnte loslassen. Vertrauen, dass es auch ohne mich läuft.", s: "kontrolle" },
    ],
  },
];

export const SHADOW_RESULTS = {
  wut: {
    icon: "flame",
    title: "Der Krieger",
    desc: "Wut ist dein Hauptkanal. Sie schützt dich vor Verletzung — aber sie schneidet dich gleichzeitig von Nähe, Ruhe und echter Stärke ab.",
    meaning: "Ein wütender Schatten entsteht fast immer dort, wo dich früher jemand nicht geschützt hat. Dein System hat gelernt: \"Wenn ich nicht kämpfe, verliere ich.\" Das war damals richtig. Heute kostet es dich Beziehungen, Gesundheit und die Fähigkeit, weich zu sein, wenn es dran wäre.",
    steps: [
      { bold: "Entkoppeln:", text: "Wenn Wut hochkommt — 3 tiefe Atemzüge BEVOR du reagierst. Nicht die Wut wegdrücken, nur nicht sofort handeln." },
      { bold: "Ursprung finden:", text: "Notiere eine Woche lang jeden Wut-Trigger. 90% werden auf dieselben 2-3 Muster zeigen." },
      { bold: "Arbeit darunter:", text: "Die Wut ist das Symptom. Darunter liegt meistens Ohnmacht, Enttäuschung oder unverarbeiteter Schmerz." },
    ],
  },
  leere: {
    icon: "moon",
    title: "Der Abwesende",
    desc: "Leere ist dein Hauptkanal. Du funktionierst nach außen, aber innerlich ist ein großer Raum, in dem nichts mehr richtig ankommt.",
    meaning: "Leere ist fast immer ein Schutzmechanismus: Wenn früh zu viel passiert ist — emotional, familiär, chaotisch — lernt dein System, alles runterzufahren. Das hat dich damals geschützt. Heute sitzt du im Job, in Beziehungen, im Leben und fragst dich, warum du nichts mehr spürst.\n\nDas ist keine Depression (notwendigerweise). Das ist ein abgeschalteter Nerv.",
    steps: [
      { bold: "Klein starten:", text: "1x am Tag 60 Sekunden still. Keine App, kein Scroll. Nur: Was ist gerade in meinem Körper?" },
      { bold: "Nicht reparieren, wahrnehmen:", text: "Du musst die Leere nicht füllen. Du musst sie kennen." },
      { bold: "Integration mit Anleitung:", text: "Allein läuft man in Leere-Mustern im Kreis. Struktur hilft." },
    ],
  },
  angst: {
    icon: "heart-dislike",
    title: "Der Ungenügende",
    desc: "Angst vor Ablehnung, Verlassensein, Nicht-genug-Sein ist dein Hauptkanal. Du funktionierst oft — aber immer für andere, selten für dich.",
    meaning: "Dieser Schatten entsteht dort, wo die Liebe als Kind an Bedingungen geknüpft war. Als Erwachsener hast du dieses Muster verfeinert — du spürst dauernd die Stimmungen der anderen und passt dich an. Nur: Es gibt keinen Moment, in dem DU einfach sein darfst.",
    steps: [
      { bold: "Erkennen, nicht kämpfen:", text: "Wenn die Angst kommt — benenne sie. \"Jetzt ist der alte Angst-Junge da.\" Mehr nicht." },
      { bold: "Bedürfnisse trainieren:", text: "Täglich EINEN Satz sagen, der nur dir dient. \"Ich brauche gerade Ruhe.\"" },
      { bold: "Arbeit an der Wurzel:", text: "Dieses Muster löst sich nicht durch Selbstoptimierung — sondern durch Arbeit mit der Ursprungssituation." },
    ],
  },
  kontrolle: {
    icon: "settings",
    title: "Der Kontrolleur",
    desc: "Kontrolle ist dein Hauptkanal. Du funktionierst gut, performst, planst — aber du kommst nie wirklich zur Ruhe.",
    meaning: "Dieser Schatten entsteht dort, wo du zu früh erwachsen werden musstest. Familie, die nicht getragen hat. Chaos, das du strukturiert hast. Dein Nervensystem hat gelernt: \"Nur wenn ICH das steuere, ist es sicher.\"\n\nDas hat dich erfolgreich gemacht. Es hindert dich aber auch daran, echte Nähe und echtes Fallenlassen zu erleben.",
    steps: [
      { bold: "Mikro-Loslassen:", text: "1x pro Tag bewusst eine kleine Sache NICHT kontrollieren. Jemand anderer entscheidet." },
      { bold: "Körper vor Kopf:", text: "10 Min Körperwahrnehmung täglich (Spüren, nicht Yoga) bringt dich zurück." },
      { bold: "Den Grund anschauen:", text: "Die Kontrolle ist kein Charakterzug. Sie ist eine Antwort auf etwas, das damals passiert ist." },
    ],
  },
};

export const SHADOW_LABELS = {
  wut: "Der Krieger",
  leere: "Der Abwesende",
  angst: "Der Ungenügende",
  kontrolle: "Der Kontrolleur",
};
```

### 5.2 `src/services/shadowTypeService.js`

```js
import { doc, getDoc, setDoc, serverTimestamp } from 'firebase/firestore';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { db } from '../config/firebase';
import { SHADOW_LABELS } from '../data/shadowQuizContent';

const PENDING_KEY = '@pending_shadow_type';
const ANSWERS_KEY = '@shadow_quiz_answers';

export async function computeDominantType(scores) {
  return Object.entries(scores).sort((a, b) => b[1] - a[1])[0][0];
}

export async function saveShadowTypeLocal(type, scores, answers) {
  await AsyncStorage.setItem(PENDING_KEY, type);
  await AsyncStorage.setItem(ANSWERS_KEY, JSON.stringify({ type, scores, answers, ts: Date.now() }));
}

export async function flushShadowTypeToFirestore(uid) {
  if (!uid) return null;
  const stored = await AsyncStorage.getItem(ANSWERS_KEY);
  if (!stored) return null;
  const { type, scores, answers } = JSON.parse(stored);

  const ref = doc(db, 'users', uid);
  const prev = await getDoc(ref);
  const prevData = prev.exists() ? prev.data() : {};
  const history = prevData.shadowTypeHistory || [];
  if (prevData.shadowType && prevData.shadowType !== type) {
    history.push({
      type: prevData.shadowType,
      at: prevData.shadowTypeCompletedAt || null,
      scores: prevData.shadowTypeScores || null,
    });
  }

  await setDoc(ref, {
    shadowType: type,
    shadowTypeLabel: SHADOW_LABELS[type],
    shadowTypeCompletedAt: serverTimestamp(),
    shadowTypeVersion: 1,
    shadowTypeScores: scores,
    shadowTypeAnswers: answers,
    shadowTypeHistory: history,
  }, { merge: true });

  await AsyncStorage.removeItem(PENDING_KEY);
  return type;
}

export async function getPendingShadowType() {
  return AsyncStorage.getItem(PENDING_KEY);
}

export async function clearShadowType() {
  await AsyncStorage.removeItem(PENDING_KEY);
  await AsyncStorage.removeItem(ANSWERS_KEY);
}
```

### 5.3 `src/screens/ShadowQuizScreen.js` (Kern, gekürzt auf essentials)

```js
import React, { useState, useRef, useEffect, useCallback } from 'react';
import {
  View, Text, StyleSheet, TouchableOpacity, ScrollView, Animated, Platform,
  useWindowDimensions, BackHandler,
} from 'react-native';
import { LinearGradient } from 'expo-linear-gradient';
import { Ionicons } from '@expo/vector-icons';
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { useFocusEffect } from '@react-navigation/native';
import { getAnalytics, logEvent } from 'firebase/analytics';

import { SHADOW_QUESTIONS, SHADOW_RESULTS } from '../data/shadowQuizContent';
import {
  computeDominantType, saveShadowTypeLocal, flushShadowTypeToFirestore,
} from '../services/shadowTypeService';
import { useAuth } from '../context/AuthContext';

// ====== Tokens (shared mit OnboardingScreen) ======
const GOLD = '#d4a843';
const GOLD_DARK = '#a67c2e';
const BG_DARK = '#040410';
const BG_MID = '#0a0820';
const BG_LIGHT = '#151528';
const WHITE = '#f5f5f7';
const WHITE_DIM = 'rgba(245, 245, 247, 0.35)';
const WHITE_MUTED = 'rgba(245,245,247,0.7)';
const GOLD_GLOW = 'rgba(212, 168, 67, 0.35)';
const PANEL_BG = 'rgba(7, 8, 23, 0.72)';
const PANEL_BORDER = 'rgba(212, 168, 67, 0.16)';

const STEP_INTRO = 'intro';
const STEP_QUESTION = 'question';
const STEP_RESULT = 'result';

function safeLog(name, params) {
  try {
    if (Platform.OS === 'web') {
      const analytics = getAnalytics();
      logEvent(analytics, name, params || {});
    }
  } catch (_) {}
}

export default function ShadowQuizScreen({ navigation, route, onComplete }) {
  const insets = useSafeAreaInsets();
  const { width: windowWidth } = useWindowDimensions();
  const { user } = useAuth();
  const allowSkip = route?.params?.allowSkip ?? true;

  const [step, setStep] = useState(STEP_INTRO);
  const [qi, setQi] = useState(0);
  const [counts, setCounts] = useState({ wut: 0, leere: 0, angst: 0, kontrolle: 0 });
  const [answers, setAnswers] = useState([]);
  const [resultType, setResultType] = useState(null);

  const fadeAnim = useRef(new Animated.Value(1)).current;
  const progressAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    const pct = step === STEP_RESULT ? 1 : step === STEP_QUESTION ? qi / SHADOW_QUESTIONS.length : 0;
    Animated.timing(progressAnim, { toValue: pct, duration: 300, useNativeDriver: false }).start();
  }, [step, qi]);

  useFocusEffect(useCallback(() => {
    const handler = () => {
      if (step === STEP_QUESTION && qi > 0) { handleBack(); return true; }
      if (step === STEP_QUESTION && qi === 0) { setStep(STEP_INTRO); return true; }
      return false;
    };
    const sub = BackHandler.addEventListener('hardwareBackPress', handler);
    return () => sub.remove();
  }, [step, qi]));

  const animateTransition = useCallback((after) => {
    Animated.timing(fadeAnim, { toValue: 0, duration: 180, useNativeDriver: true }).start(() => {
      after();
      Animated.timing(fadeAnim, { toValue: 1, duration: 280, useNativeDriver: true }).start();
    });
  }, [fadeAnim]);

  const handleStart = () => {
    safeLog('shadow_quiz_start');
    animateTransition(() => { setStep(STEP_QUESTION); setQi(0); });
  };

  const handleAnswer = (answerIndex, type) => {
    safeLog('shadow_quiz_answer', { q: qi, type });
    const newAnswers = [...answers, { q: qi, a: answerIndex, type }];
    const newCounts = { ...counts, [type]: counts[type] + 1 };
    setAnswers(newAnswers);
    setCounts(newCounts);

    if (qi + 1 >= SHADOW_QUESTIONS.length) {
      computeDominantType(newCounts).then(async (dominant) => {
        setResultType(dominant);
        await saveShadowTypeLocal(dominant, newCounts, newAnswers);
        if (user?.uid) {
          flushShadowTypeToFirestore(user.uid).catch(() => {});
        }
        safeLog('shadow_quiz_complete', { type: dominant });
        animateTransition(() => setStep(STEP_RESULT));
      });
    } else {
      animateTransition(() => setQi(qi + 1));
    }
  };

  const handleBack = () => {
    if (qi === 0) return;
    const last = answers[answers.length - 1];
    setAnswers(answers.slice(0, -1));
    setCounts({ ...counts, [last.type]: counts[last.type] - 1 });
    animateTransition(() => setQi(qi - 1));
  };

  const handleSkip = () => {
    safeLog('shadow_quiz_skip', { step, qi });
    onComplete?.();
  };

  const handleResultContinue = () => {
    safeLog('shadow_quiz_continue_to_guide', { type: resultType });
    onComplete?.();
  };

  const handleRetake = () => {
    safeLog('shadow_quiz_retake');
    setCounts({ wut: 0, leere: 0, angst: 0, kontrolle: 0 });
    setAnswers([]);
    setQi(0);
    setResultType(null);
    animateTransition(() => setStep(STEP_INTRO));
  };

  return (
    <LinearGradient colors={[BG_DARK, BG_MID, BG_LIGHT]} style={styles.container}>
      <View style={[styles.header, { paddingTop: Math.max(insets.top, 16) + 8 }]}>
        {step === STEP_QUESTION && qi > 0 && (
          <TouchableOpacity onPress={handleBack} style={styles.backBtn}>
            <Ionicons name="chevron-back" size={22} color={GOLD} />
          </TouchableOpacity>
        )}
        <View style={styles.progressTrack}>
          <Animated.View style={[styles.progressFill, {
            width: progressAnim.interpolate({ inputRange: [0, 1], outputRange: ['0%', '100%'] }),
          }]} />
        </View>
        {allowSkip && step !== STEP_RESULT && (
          <TouchableOpacity onPress={handleSkip} style={styles.skipBtn}>
            <Text style={styles.skipText}>Überspringen</Text>
          </TouchableOpacity>
        )}
      </View>

      <Animated.View style={[styles.content, { opacity: fadeAnim }]}>
        {step === STEP_INTRO && (
          <ScrollView contentContainerStyle={styles.scrollContent}>
            <View style={styles.card}>
              <Text style={styles.eyebrow}>KOSTENLOSER SELBSTTEST</Text>
              <Text style={styles.title}>Welcher Schatten steuert dich gerade?</Text>
              <Text style={styles.body}>
                8 Fragen. 2 Minuten. Am Ende weißt du, welcher innere Anteil gerade im Vordergrund deines Lebens arbeitet — und was der erste klare Schritt raus ist.
              </Text>
              <TouchableOpacity onPress={handleStart} style={styles.primaryBtn} activeOpacity={0.85}>
                <LinearGradient colors={[GOLD, GOLD_DARK]} start={{x:0,y:0}} end={{x:1,y:1}} style={styles.primaryBtnGradient}>
                  <Ionicons name="play" size={18} color="#0a0a0f" style={{ marginRight: 10 }} />
                  <Text style={styles.primaryBtnText}>Test starten</Text>
                </LinearGradient>
              </TouchableOpacity>
              <Text style={styles.fineprint}>Ehrlich antworten. Niemand sieht deine Antworten außer dir.</Text>
            </View>
          </ScrollView>
        )}

        {step === STEP_QUESTION && (
          <ScrollView contentContainerStyle={styles.scrollContent}>
            <View style={styles.card}>
              <Text style={styles.qCounter}>FRAGE {qi + 1} VON {SHADOW_QUESTIONS.length}</Text>
              <Text style={styles.question}>{SHADOW_QUESTIONS[qi].q}</Text>
              <View style={styles.answers}>
                {SHADOW_QUESTIONS[qi].a.map((opt, idx) => (
                  <TouchableOpacity
                    key={idx}
                    style={styles.answer}
                    onPress={() => handleAnswer(idx, opt.s)}
                    activeOpacity={0.75}
                    testID={`shadow-answer-${qi}-${idx}`}
                  >
                    <Text style={styles.answerText}>{opt.t}</Text>
                  </TouchableOpacity>
                ))}
              </View>
            </View>
          </ScrollView>
        )}

        {step === STEP_RESULT && resultType && (
          <ScrollView contentContainerStyle={styles.scrollContent}>
            <View style={styles.card}>
              <View style={styles.resultIconWrap}>
                <Ionicons name={SHADOW_RESULTS[resultType].icon} size={64} color={GOLD} />
              </View>
              <Text style={[styles.eyebrow, { textAlign: 'center' }]}>DEIN DOMINANTER SCHATTEN</Text>
              <Text style={[styles.title, { textAlign: 'center' }]}>{SHADOW_RESULTS[resultType].title}</Text>
              <Text style={[styles.body, { textAlign: 'center' }]}>{SHADOW_RESULTS[resultType].desc}</Text>

              <Text style={styles.h2}>Was das für dich bedeutet</Text>
              <Text style={styles.body}>{SHADOW_RESULTS[resultType].meaning}</Text>

              <Text style={styles.h2}>Die nächsten 3 Schritte</Text>
              {SHADOW_RESULTS[resultType].steps.map((s, i) => (
                <View key={i} style={styles.step}>
                  <Text style={styles.stepNum}>{i + 1}</Text>
                  <Text style={styles.stepText}>
                    <Text style={styles.stepBold}>{s.bold} </Text>{s.text}
                  </Text>
                </View>
              ))}

              <TouchableOpacity onPress={handleResultContinue} style={[styles.primaryBtn, { marginTop: 32 }]} activeOpacity={0.85}>
                <LinearGradient colors={[GOLD, GOLD_DARK]} start={{x:0,y:0}} end={{x:1,y:1}} style={styles.primaryBtnGradient}>
                  <Text style={styles.primaryBtnText}>Weiter zu deinem Guide</Text>
                  <Ionicons name="arrow-forward" size={18} color="#0a0a0f" style={{ marginLeft: 10 }} />
                </LinearGradient>
              </TouchableOpacity>

              <TouchableOpacity onPress={handleRetake} style={styles.secondaryBtn}>
                <Text style={styles.secondaryBtnText}>Quiz wiederholen</Text>
              </TouchableOpacity>
            </View>
          </ScrollView>
        )}
      </Animated.View>
    </LinearGradient>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  header: { flexDirection: 'row', alignItems: 'center', paddingHorizontal: 20, paddingBottom: 12, gap: 12 },
  backBtn: { padding: 8 },
  skipBtn: { paddingHorizontal: 10, paddingVertical: 8 },
  skipText: { color: WHITE_DIM, fontSize: 14, fontWeight: '500' },
  progressTrack: { flex: 1, height: 3, backgroundColor: 'rgba(212,168,67,0.1)', borderRadius: 2, overflow: 'hidden' },
  progressFill: { height: '100%', backgroundColor: GOLD },
  content: { flex: 1 },
  scrollContent: { padding: 20, paddingBottom: 60 },
  card: {
    backgroundColor: PANEL_BG, borderRadius: 20, padding: 28, borderWidth: 1, borderColor: PANEL_BORDER,
    ...Platform.select({
      web: { boxShadow: '0 28px 90px rgba(0,0,0,0.4)', backdropFilter: 'blur(16px)' },
      default: { shadowColor: '#000', shadowOffset: { width: 0, height: 20 }, shadowOpacity: 0.22, shadowRadius: 32, elevation: 10 },
    }),
  },
  eyebrow: { fontSize: 11, fontWeight: '700', letterSpacing: 3, color: GOLD, marginBottom: 14 },
  title: { fontSize: 30, fontWeight: '900', color: WHITE, marginBottom: 16, lineHeight: 38 },
  body: { fontSize: 16, color: WHITE_MUTED, lineHeight: 25, marginBottom: 12 },
  fineprint: { fontSize: 13, color: WHITE_DIM, textAlign: 'center', marginTop: 18 },
  qCounter: { fontSize: 11, letterSpacing: 2.5, color: GOLD, fontWeight: '700', marginBottom: 14 },
  question: { fontSize: 22, color: WHITE, fontWeight: '600', lineHeight: 30, marginBottom: 24 },
  answers: { gap: 10 },
  answer: {
    padding: 18, backgroundColor: 'rgba(255,255,255,0.04)', borderWidth: 1, borderColor: 'rgba(212,168,67,0.14)', borderRadius: 12,
  },
  answerText: { color: WHITE, fontSize: 15, lineHeight: 22 },
  h2: { fontSize: 20, color: WHITE, fontWeight: '700', marginTop: 24, marginBottom: 10 },
  step: { flexDirection: 'row', marginBottom: 14, gap: 12 },
  stepNum: { color: GOLD, fontSize: 20, fontWeight: '800', width: 22 },
  stepText: { flex: 1, color: WHITE_MUTED, fontSize: 15, lineHeight: 23 },
  stepBold: { color: WHITE, fontWeight: '700' },
  resultIconWrap: {
    alignSelf: 'center', width: 110, height: 110, borderRadius: 55, backgroundColor: GOLD_GLOW,
    alignItems: 'center', justifyContent: 'center', marginBottom: 20,
  },
  primaryBtn: { marginTop: 24, borderRadius: 28, overflow: 'hidden' },
  primaryBtnGradient: {
    flexDirection: 'row', height: 58, justifyContent: 'center', alignItems: 'center', paddingHorizontal: 28,
  },
  primaryBtnText: { color: '#0a0a0f', fontSize: 16, fontWeight: '800', letterSpacing: 0.5 },
  secondaryBtn: {
    marginTop: 14, paddingVertical: 14, borderRadius: 28, borderWidth: 1.5, borderColor: GOLD, alignItems: 'center',
    backgroundColor: 'rgba(212,168,67,0.08)',
  },
  secondaryBtnText: { color: GOLD, fontSize: 15, fontWeight: '600' },
});
```

### 5.4 Integration-Patch für `App.js`

Direkt nach dem bestehenden `OnboardingScreen`-Block. Suche diesen Abschnitt in `App.js`:

```js
// Alt (ungefähr)
if (!hasSeenOnboarding) {
  return <OnboardingScreen onComplete={...} onLogin={...} />;
}
return <MainNavigator ... />;
```

Ersetze mit:

```js
// Neu
import ShadowQuizScreen from './src/screens/ShadowQuizScreen';

// innerhalb der Root-Component:
const [hasCompletedShadowQuiz, setHasCompletedShadowQuiz] = useState(null);
useEffect(() => {
  AsyncStorage.getItem('@has_completed_shadow_quiz').then(v => setHasCompletedShadowQuiz(v === 'true'));
}, []);

// ...

if (!hasSeenOnboarding) {
  return (
    <OnboardingScreen
      onComplete={() => {
        AsyncStorage.setItem('@has_seen_onboarding', 'true');
        setHasSeenOnboarding(true);
      }}
      onLogin={() => {
        AsyncStorage.setItem('@has_seen_onboarding', 'true');
        setHasSeenOnboarding(true);
        setSkipShadowQuiz(true);  // Login-Pfad skippt Quiz
      }}
    />
  );
}

if (hasSeenOnboarding && !hasCompletedShadowQuiz && !skipShadowQuiz) {
  return (
    <ShadowQuizScreen
      onComplete={() => {
        AsyncStorage.setItem('@has_completed_shadow_quiz', 'true');
        setHasCompletedShadowQuiz(true);
      }}
    />
  );
}

return <MainNavigator ... />;
```

### 5.5 Integration-Patch für `MainNavigator.js`

Damit User das Quiz auch später aus Profil heraus starten können:

```js
import ShadowQuizScreen from '../screens/ShadowQuizScreen';

const SafeShadowQuiz = withErrorBoundary(ShadowQuizScreen, 'Das Quiz konnte nicht geladen werden.');

// innerhalb Stack.Navigator:
<Stack.Screen
  name="ShadowQuiz"
  component={SafeShadowQuiz}
  options={{ presentation: 'modal', gestureEnabled: true }}
/>
```

Und in `App.js` linking-config:
```js
ShadowQuiz: 'shadow-quiz',
```

---

## 6. Personalisierung-Hooks — 5 konkrete Stellen

Sobald `user.shadowType` in `AuthContext` verfügbar ist, bauen wir folgende Stellen aus:

### 6.1 Guide-Tab Dashboard (`GuideScreen.js` bzw. Guide-Tab im TabNavigator)

- **Hero-Greeting**: "Hey Jurek — für deinen **Krieger**-Anteil heute: 3-Min-Atemübung"
- **Empfohlene Videos**: Videos mit Tag `wut` / `kontrolle` / etc. erscheinen oben im Grid
- **Tageszitat**: Zufällig aus shadowType-spezifischer Pool (z.B. für Krieger: Rumi, für Kontrolleur: Tao Te Ching)

### 6.2 AI-Recommendation (`AIRecommendationScreen.js`)

System-Prompt erweitern: `Der User hat im Shadow-Quiz den Typ "{shadowTypeLabel}" gezogen. Gib Empfehlungen, die mit diesem Anteil arbeiten.` — In der Praxis erhöht das die Relevanz der Empfehlungen massiv.

### 6.3 Journal-Prompts (`JournalScreen.js`)

Statt generischer Prompts `Was war heute?` erscheint shadowType-spezifisch:
- **Krieger**: "Wo kam heute Wut hoch? Was lag darunter?"
- **Abwesender**: "Was hast du heute tatsächlich gefühlt? Auch 1 Sekunde zählt."
- **Ungenügender**: "Wofür hast du dich heute zu klein gemacht?"
- **Kontrolleur**: "Was hast du heute nicht kontrolliert — und was ist passiert?"

### 6.4 Meditationen (`SoundscapeScreen.js` / `BreathworkScreen.js`)

Featured-Medi je nach Typ:
- **Krieger**: Kühlende Atmung (Shitali), Erdungsmeditation
- **Abwesender**: Body-Scan, Fühl-Aktivierungs-Meditation
- **Ungenügender**: Self-Compassion-Meditation
- **Kontrolleur**: Loslass-Meditation, Yin

### 6.5 Membership / CTAs

1:1-Session-Booking (`BookingScreen.js`) zeigt shadowType-spezifischen Vorab-Text:
- "Session-Fokus für den **Kontrolleur**: Wir arbeiten an der Ursprungssituation, aus der die Kontrolle entstanden ist."

**Helper**: `src/utils/shadowPersonalization.js` stellt zentrale Content-Maps bereit, damit Personalisierung nicht 5× dupliziert wird.

```js
export function getShadowContent(type, context) {
  // context: 'journal_prompt' | 'greeting' | 'booking_intro' | 'meditation_pick' | 'daily_tip'
  const map = {
    wut: { journal_prompt: "Wo kam heute Wut hoch? Was lag darunter?", /* ... */ },
    leere: { journal_prompt: "Was hast du heute tatsächlich gefühlt?", /* ... */ },
    angst: { journal_prompt: "Wofür hast du dich heute zu klein gemacht?", /* ... */ },
    kontrolle: { journal_prompt: "Was hast du heute nicht kontrolliert?", /* ... */ },
  };
  return map[type]?.[context] || null;
}
```

---

## 7. Tracking — Analytics Events

Alle Events über Firebase Analytics (`logEvent(analytics, name, params)`) + parallel über Plausible auf der Landing-Page (`quiz.html`, bereits live).

| Event-Name | Wann | Properties |
|---|---|---|
| `shadow_quiz_start` | Intro-Screen "Test starten" | `{ source: 'onboarding' \| 'profile_retake' }` |
| `shadow_quiz_answer` | Jede Antwort | `{ q: 0-7, type: 'wut'\|..., answer_index: 0-3 }` |
| `shadow_quiz_back` | Back-Button gedrückt | `{ from_q: N }` |
| `shadow_quiz_skip` | Skip gedrückt | `{ step: 'intro'\|'question', q: N }` |
| `shadow_quiz_complete` | Result gezeigt | `{ type, scores_json, duration_ms }` |
| `shadow_quiz_continue_to_guide` | Result "Weiter" | `{ type }` |
| `shadow_quiz_retake` | Retake-Button | `{ previous_type }` |
| `shadow_type_personalization_shown` | Jedes Mal Personalisierung greift | `{ surface: 'guide_hero' \| 'journal_prompt' \| ..., type }` |

**Zusätzlich — User-Property** setzen: `setUserProperties(analytics, { shadow_type: type })` — ermöglicht Funnel-Analysen über alle Events hinweg.

---

## 8. Test-Plan

### 8.1 Happy Paths (3)

1. **Neuer User, komplett durch**
   - App fresh installieren → 5 Onboarding-Slides → "Jetzt starten" → Quiz-Intro → 8× Antwort → Result zeigt "Der Krieger" (wenn dominante Antworten Typ `wut`) → "Weiter zum Guide" → Tabs-View
   - **Assert**: AsyncStorage hat `@has_completed_shadow_quiz=true`, `@pending_shadow_type='wut'`

2. **User registriert sich nach Quiz**
   - Happy Path 1 → Profil → Account erstellen → Firestore `users/{uid}` geschrieben
   - **Assert**: `users/{uid}.shadowType === 'wut'`, `shadowTypeAnswers.length === 8`, AsyncStorage `@pending_shadow_type` ist geleert

3. **Re-Take aus Profil**
   - Eingeloggter User mit `shadowType='kontrolle'` → Profil → "Quiz wiederholen" → Anderer Typ (z.B. `leere`) → Firestore update
   - **Assert**: `users/{uid}.shadowType === 'leere'`, `shadowTypeHistory[0].type === 'kontrolle'`, `shadowTypeCompletedAt` aktualisiert

### 8.2 Edge Cases (3)

1. **Offline während Quiz-Complete**
   - Flugmodus an → Quiz durchlaufen → Result sichtbar
   - **Assert**: `@pending_shadow_type` in AsyncStorage gesetzt; Firestore-Write wirft Error aber crasht nicht; beim nächsten Online-Event (`AuthContext`-Init mit network) flushed er nach Firestore

2. **User mit bestehendem `shadowType` öffnet App neu**
   - Firestore `users/{uid}.shadowType='angst'`, AsyncStorage clear
   - App-Start → soll **nicht** zum Quiz leiten (Gate prüft Firestore bei eingeloggtem User über `AuthContext`)
   - **Assert**: Direkt ins Tabs-View; `@has_completed_shadow_quiz=true` wird gesetzt

3. **Skip-Pfad**
   - User skippt Quiz direkt im Intro → Tabs-View
   - **Assert**: `@has_completed_shadow_quiz=true` (damit Prompt nicht jedes Mal kommt), aber `users/{uid}.shadowType===null`
   - Nach 3 App-Starts: Guide-Hero zeigt sanften Reminder "Lern deinen Shadow kennen — 2 Min"
   - Tap auf Reminder → ShadowQuizScreen mit `allowSkip=true`

### 8.3 Zusätzliche Regression

- **Back-Button (Android Hardware)**: In Frage 4 zurück → Count rollback korrekt → Zurück zu Q3
- **Schnelle Double-Taps**: Kein Doppel-Zählen (via `disabled`-State während animateTransition)
- **Web-Variante**: `logEvent` wrappt in try/catch — auf iOS/Android wo kein Web-Analytics, kein Crash
- **Light/Dark Mode**: Quiz nutzt fixed BG_DARK, da Onboarding-Kontext immer dark — keine Theme-Inkonsistenz

---

## 9. Rollout-Plan

| Phase | Scope | Zeit |
|---|---|---|
| **Phase 1 — MVP** | ShadowQuizScreen + Firestore-Write + Skip | 2 Tage |
| **Phase 2 — Personalisierung** | Guide-Hero + Journal-Prompts (Hooks 6.1, 6.3) | 1 Tag |
| **Phase 3 — AI + Medi** | AI-Prompt-Erweiterung + Meditation-Picks (Hooks 6.2, 6.4) | 1 Tag |
| **Phase 4 — Booking** | Session-Booking Personalisierung (Hook 6.5) | 0.5 Tage |
| **Phase 5 — Analytics** | Event-Tracking + User-Property + Funnel im Analytics-Dashboard | 0.5 Tage |

**Gesamt: 5 Arbeitstage** bis Full-Rollout. MVP (Phase 1) ist alleine schon wertvoll.

---

## 10. Offene Fragen / Follow-ups

1. Soll der **Name** vor dem Quiz erfragt werden (für "Jurek, Wut ist dein...")? → MVP nein, nutze `user.displayName` wenn vorhanden, sonst leer. Im Onboarding-Slide 1 könnte ein Name-Input eingeschoben werden.
2. **Mehrere dominante Typen**: Wenn zwei Typen gleich viele Punkte haben — aktuell first-one-wins. Evtl. "Primär + Sekundär" anzeigen? → v2.
3. **Push-Notifications** shadowType-spezifisch (morgens "Dein Krieger-Impuls für heute")? → v2, nach erstem Telemetrie-Review.
4. **ShadowQuizScreen in SelbsttestScreen einbetten** als "offizieller Onboarding-Selbsttest"? → v1.5, wenn Selbsttest-Screen eh überarbeitet wird.

---

Saved to feature-spec-onboarding-quiz.md
