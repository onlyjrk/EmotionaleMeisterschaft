# Emotionale Meisterschaft App — Performance Audit

Date: 2026-04-21
Scope: `C:\Users\Jurek\emotionale-meisterschaft-app\src` (Expo SDK 54, RN 0.81, Legacy Arch)
Sampled: 46 screens, 18 components, App.js, TabNavigator

## Executive Summary

Code quality is ok — `useRef(new Animated.Value)` is used correctly almost everywhere, FlatLists mostly have `keyExtractor`/`initialNumToRender`/`maxToRenderPerBatch`. Three big concerns:

1. **N+1 onSnapshot listeners in VoiceChannelScreen** (lines 1011–1028): for every voice room, another listener is opened — scales linearly with room count.
2. **Huge single-file screens** (ChatScreen 2.6k lines, ProfileScreen 5.3k lines, MeditationsScreen 4.3k lines) bundled in initial JS, plus imports of `expo-av`, Firebase, Three.js from multiple screens with no lazy loading.
3. **561 inline `onPress={() => …}`** arrow functions and **529 inline `style={{…}}`** objects across 58/63 files. Most are inside map callbacks over static data so the impact is bounded, but several sit inside FlatList renderItem paths and break memoization.

No `<Image>` components use `resizeMode` correctly (only 4 occurrences total, and most images are icons via `@expo/vector-icons` which is fine). No `Image.prefetch` anywhere. No `getItemLayout` on any FlatList.

---

## 1. FlatList / ScrollList Performance

### 1.1 No `getItemLayout` anywhere
- File: all 13 FlatList usages
- Severity: **medium**
- Problem: Without `getItemLayout`, `scrollToIndex` is unreliable (see ChatScreen.js:2193–2199 which has a `onScrollToIndexFailed` workaround) and scroll perf degrades on long lists.
- Fix for ChatScreen (fixed row height if message cells are bounded, else skip):
  ```js
  const getItemLayout = useCallback((_, index) => (
    { length: ESTIMATED_MESSAGE_HEIGHT, offset: ESTIMATED_MESSAGE_HEIGHT * index, index }
  ), []);
  // <FlatList getItemLayout={getItemLayout} ... />
  ```

### 1.2 Inline `renderItem` creating a new function every render
- Files/lines: `AffirmationScreen.js:871`, `AffirmationScreen.js:953`, `InnerChildScreen.js:288`, `PartsWorkScreen.js:742`, `VoiceSessionScreen.js:497`
- Severity: **medium**
- Problem: `renderItem={({ item }) => (…)}` re-creates the function every render, triggering full FlatList re-render.
- Fix:
  ```js
  const renderAffirmation = useCallback(({ item }) => (<AffirmationCard item={item} />), [/* stable deps */]);
  // renderItem={renderAffirmation}
  ```

### 1.3 Large static lists rendered with `.map()` inside `<ScrollView>`
- File: `BreathworkScreen.js:1220–1225` (TECHNIQUES.map), `CommunityChallengeScreen.js:727–728`, `ChatScreen.js:2101–2106` (CHANNELS — fine, small), `ChatScreen.js:2314–2319` (emoji categories)
- Severity: **low** for BreathworkScreen (few items), **low** for ChatScreen channel list
- Problem: Acceptable when N is small and static, but the Breathwork technique cards all construct animated `interpolate` output inline during render (line 1230). This rebuilds Animated interpolation nodes each render.
- Fix: Pre-compute `cardAnims[index].interpolate(...)` once via `useMemo` keyed by `cardAnims`, or extract `<TechCard>` memo child.

### 1.4 Inline `renderItem` in ScrollView+map that uses `index` as key
- Files: `CommunityChallengeScreen.js:658, 1054, 1076, 1306, 1356`; `BodyScanScreen.js:1286, 1305`; `CoachingHubClassicScreen.js:105, 221, 270, 327, 368, 393`; `AIRecommendationScreen.js:769`; `ChatScreen.js:1923`; `BreathworkScreen.js:1155, 1883, 1992`; `BookingScreen.js:358, 435`; `ChatPoll.js:256, 474`; `SkeletonLoader.js:234, 249`; `PaywallModal.js:102, 665`; `AIConsentModal.js:328`; `AffirmationCard.js:235`; `MysteryBoxModal.js:173, 262`
- Severity: **low** (lists are mostly static and short)
- Problem: `key={i}` is an anti-pattern if the list reorders or items get inserted; causes unnecessary re-renders / wrong component instance reuse.
- Fix: Use stable IDs: `key={item.id ?? i}`.

---

## 2. Re-Render Hotspots

### 2.1 Inline onPress handlers everywhere
- 561 occurrences across 66 files
- Severity: **medium** (cumulative)
- Hotspots in performance-critical paths:
  - `ChatScreen.js` — 33 inline handlers inside message rows
  - `MeditationsScreen.js` — 38, including inside TrackItem map
  - `ProfileScreen.js` — 67
- Fix: In any row component rendered inside FlatList, wrap the row in `React.memo` and hoist handlers with `useCallback` that takes `item.id`:
  ```js
  const handlePressMessage = useCallback((id) => { /* … */ }, []);
  // <MessageRow onPress={handlePressMessage} id={item.id} />
  ```

### 2.2 Inline `style={{…}}` objects
- 529 occurrences across 63 files
- Severity: **medium** — biggest offenders are `ProfileScreen.js` (67), `ChatScreen.js` (12), `CourseScreen.js` (13), `BreathworkScreen.js` (10), `SessionNotesScreen.js` (58!)
- Fix: Extract to `StyleSheet.create` at the bottom. For theme-dependent styles use `useMemo`:
  ```js
  const dynamicStyles = useMemo(() => StyleSheet.create({
    root: { backgroundColor: isDarkMode ? colors.black : lightMode.background },
  }), [isDarkMode]);
  ```

### 2.3 `useEffect` dependency on derived boolean mutation
- File: `ChatScreen.js:335`
  ```js
  useEffect(() => { … }, [activeChannel, messages.length > 0]);
  ```
- Severity: **low**
- Problem: React warns on non-primitive/computed dep arrays in StrictMode; `messages.length > 0` is fine but fragile.
- Fix: `const hasMessages = messages.length > 0; useEffect(..., [activeChannel, hasMessages]);`

### 2.4 `useWindowDimensions` in 5 screens — OK, but
- Files: `ProfileScreen.js:680, 1150`, `OnboardingScreen.js:213`, `MeditationsScreen.js:233`, `AIRecommendationScreen.js:51`, `TabNavigator.js:75`
- Severity: **low**
- Problem: `useWindowDimensions` re-renders the whole screen on any orientation/keyboard event. `ProfileScreen` uses it in 2 places → two full re-renders per dimension change on a 5.3k-line screen.
- Fix: Consolidate to a single parent hook call, pass width via prop/context; or use `Dimensions.get('window')` when dynamic resize is not needed.

### 2.5 Missing memoization on derived message arrays
- File: `ChatScreen.js:392, 394-405` — `topLevelMessages`/`replyGroups` memoized correctly. Good.
- But `chatParticipants` iterates `messages` on every change (line 374) — OK with `useMemo([messages])`, but the parent also computes `userStats` in a `useEffect` that writes state (line 360–370), causing an extra render cycle per new message.
- Severity: **medium**
- Fix: Move `userStats` into a `useMemo([messages])` so it's derived, not state.

---

## 3. Heavy Component Issues

### 3.1 `new Animated.Value` inside `.map()` call (not useRef)
- File: `DreamJournalScreen.js:216, 275` — `new Animated.Value(Math.random())` inside a star/particle generator
- File: `BreathworkScreen.js:312-315` — `x/y/opacity/scale` created in an array mapped on every render
- Severity: **high** (on BreathworkScreen, this is the animated particle dots array)
- Problem: Every re-render creates fresh Animated.Value objects; animations attached to the old ones keep running → leaks + jitter.
- Fix:
  ```js
  const dots = useRef(
    Array.from({ length: N }, () => ({
      x: new Animated.Value(0), y: new Animated.Value(0),
      opacity: new Animated.Value(0), scale: new Animated.Value(0),
    }))
  ).current;
  ```

### 3.2 Animated.multiply / subtract creating transient Animated.Value in render
- File: `BreathworkScreen.js:1556, 1557, 1829, 1839`
  ```js
  { translateX: Animated.subtract(dotX, new Animated.Value(dotSize / 2)) }
  { scale: Animated.multiply(flameScale, new Animated.Value(0.85)) }
  ```
- Severity: **high** — allocation + graph rebuild each render
- Fix: Hoist constants with `useRef(new Animated.Value(dotSize/2)).current` once, or just use `Animated.subtract(dotX, dotSize/2)` — RN supports number second arg via static compose if wrapped as `useMemo`.

### 3.3 `DocumentsScreen.js:60` — push Animated.Value in a loop (inside render)
- Severity: **high**
- Context: `cardAnims.push(new Animated.Value(0))` — reading the surrounding block shows it's inside a `useMemo` or top-level, need verification. If inside component body/useEffect without ref, same issue as 3.1.

### 3.4 Synchronous heavy list building in render
- File: `MeditationsScreen.js:63-80` — `getAllPodcastsSorted()` called at module scope — fine.
- But this file is 4367 lines, imports Audio + AsyncStorage + Haptics + Firebase-derived XP service on module load. Tree-shaking in Metro is weak; importing `MeditationsScreen` into `MainNavigator` at top-level loads all of it at app start.
- Severity: **high**
- Fix: Use `React.lazy` + `<Suspense>` on navigation tabs that aren't the default tab, or split by using `getComponent: () => require('./MeditationsScreen').default` in stack config.

### 3.5 Three.js imported for a single screen
- File: `GameScreen.js:18` — `import * as THREE from 'three'`
- Severity: **high** — three.js is ~600KB minified, pulled into the app bundle for all users
- Fix: Namespace-import only what's used (`import { Scene, PerspectiveCamera, … } from 'three'`) or lazy-load the screen.

---

## 4. Asset Loading

### 4.1 Zero real PNG/JPG images in `src/assets` or `require()`'d from screens
- Severity: **low**
- Finding: `require('…/assets/…png')` returns 0 matches in `src/`. App uses vector icons (`@expo/vector-icons`) and emoji. Icon files at top-level `assets/` (icon.png 385KB, adaptive-icon.png 385KB) are launcher-only; fine.
- 3 screens use remote images via `source={{ uri: … }}` (ChatScreen, CourseScreen, VideoPlayerScreen).
- Fix: Add `resizeMode="cover"` on all `<Image source={{ uri }}>` (currently only 4 uses of `resizeMode` across the whole codebase). Consider `Image.prefetch(url)` for recurring avatars (`ChatScreen`, `CoachesScreen`).

### 4.2 `social-preview.png` (257KB), `grok-image-*.png` (196KB) unused at runtime
- Severity: **low**
- Fix: Move to `/dist`-only or landing-page folder; excluding from Expo bundle saves ~450KB install size.

---

## 5. Firebase Listener Leaks

### 5.1 N+1 listener fan-out in VoiceChannelScreen
- File: `VoiceChannelScreen.js:1011-1034`
- Severity: **critical**
- Problem: For every voice room the outer listener emits, a new `onSnapshot` on `participants` is opened. On snapshot reemission, old participant listeners are unsubscribed, but if rooms count = R, you always hold R+1 active listeners. If R grows, cost grows linearly. The `roomListUnsubs.current.forEach((u) => u())` cleanup is fine — but called inside the snapshot callback itself (line 1019), which means between old-cleanup and new-subscribe the participant counts briefly flash to 0 → visible flicker and extra Firestore reads.
- Fix: Use a single collectionGroup query against `participants` with a `where('roomId','==', …)` index, OR only subscribe on visible rooms (virtualize). Minimum: move subscription setup into per-Room children that manage their own lifecycle.

### 5.2 Participant sub-listener cleanup on re-emission
- Same file 1024: the nested `participantUnsub` is only cleaned up when the **outer** `roomsRef` emits again or on unmount. If the outer doesn't change, old per-room listeners remain forever even if the room doc is deleted.
- Severity: **high**
- Fix: Diff current rooms vs. previous; unsubscribe deleted rooms only.

### 5.3 ChatScreen: 5 concurrent listeners
- File: `ChatScreen.js:318, 345, 578, 602` + auth listener
- Severity: **medium**
- The unread-check listener at 344–356 opens `limit(1)` per channel (6 channels = 6 listeners) just to detect "new message since last seen". All have proper unsubs.
- Fix (cost): Replace with a single Firestore document at `user/{uid}/chatSeen` with per-channel timestamps, read on mount. Saves 5× listener overhead.

### 5.4 All other onSnapshot usages have cleanup
- Verified: `CoachesScreen:170`, `ChatScreen` block, `FreeCoachingScreen:193`, `HealingCircleScreen:211`, `VoiceSessionScreen` (uses refs with `.current = onSnapshot(…)`), `ChatPoll:290`. All return `() => unsub()`. Good.

### 5.5 Missing `where()` on queries
- File: `ChatScreen.js:316` — `collection(db, 'community-${activeChannel}')` + `limit(100)` — fine, naturally scoped by channel name.
- File: `VoiceChannelScreen.js:1010` — `collection(db, 'voice_rooms')` — no filter. If inactive rooms accumulate, this reads them all.
- Severity: **medium**
- Fix: `query(roomsRef, where('active', '==', true))` or prune rooms via Cloud Function.

---

## 6. Bundle Size

### 6.1 `three` (~600KB) + `@shopify/react-native-skia` (~4MB native) for one screen each
- Files: `GameScreen.js` uses Three; Skia used… let me check.
- Severity: **high**
- Fix: `React.lazy` GameScreen; keep Skia only if actually needed (Skia is a great perf tool but massive).

### 6.2 `expo-av` (legacy) still present alongside RN 0.81
- `voiceService.js:6`, `MeditationsScreen.js:23`, `PersonalQuestionScreen.js:17`, `VideoPlayerScreen.js:19`
- Severity: **medium**
- Expo SDK 54 is deprecating `expo-av` in favor of `expo-audio` + `expo-video`. Migrating lets you drop `expo-av` package size.
- Fix: Swap to `expo-audio`/`expo-video` package-by-package; test each screen.

### 6.3 `@xenova/transformers` in devDependencies only
- Good — not in app bundle.

### 6.4 No moment.js/lodash — only one reference to `date-fns` in `SessionNotesScreen.js`
- Severity: **low**. Check that `SessionNotesScreen.js:1` imports are named (not default) to allow tree-shake.

### 6.5 `firebase` v12 full import
- Severity: **medium** — ensure all imports use modular SDK (`firebase/firestore`, not `firebase/compat/firestore`). Confirmed: `App.js:71` uses modular API. Good.

---

## 7. Startup Performance

### 7.1 `App.js` initApp blocks splash until AsyncStorage read
- Lines 300–361
- Sequence: `setupGlobalErrorHandler` → `auth.authStateReady()` (awaited on native) → optional `fetch('/version.json')` (web only) → `AsyncStorage.getItem('@has_seen_onboarding')` → `setShowOnboarding(...)`.
- Splash (line 368) shows until `showOnboarding !== null && fontsLoaded`.
- Severity: **medium**
- Issues:
  - `auth.authStateReady()` blocks boot on native. If Firebase fails (bad network), user sits on splash.
  - Fonts (6 weights × 2 families) all loaded synchronously via `useFonts`.
- Fix:
  - Wrap `auth.authStateReady()` in `Promise.race([ready, timeout(2000)])` so boot never stalls >2s.
  - Use `expo-splash-screen` (already installed) to `preventAutoHideAsync()` + `hideAsync()` explicitly once ready; splash pixel-match is better than JS splash.
  - Preload only the 2 most-used weights (Regular, SemiBold); lazy-register others when a screen needs them.

### 7.2 `NavigationContainer` mounts *all* stack screens
- File: `MainNavigator` (ref from App.js:485)
- Severity: **high** — 46 stack screens, React Navigation mounts all definitions at boot (components themselves lazy-load only when focused, but their *imports* at the top of each file run immediately — see 3.4 above).
- Fix: Use `getComponent` in `Stack.Screen`:
  ```js
  <Stack.Screen name="Meditationen"
    getComponent={() => require('../screens/MeditationsScreen').default} />
  ```
  This defers the `require()` until navigation; saves several hundred KB on initial JS parse.

### 7.3 `AdminSessionRequestNotifier` runs for every user
- File: `App.js:210-284`
- Severity: **low**
- The effect `return undefined` path early-exits for non-admins (line 220), but the component still mounts inside the `NavigationContainer` tree (line 483). OK.

---

## Top 10 Quick Wins (<30 min each, high impact)

| # | File:line | Fix | Impact |
|---|-----------|-----|--------|
| 1 | `VoiceChannelScreen.js:1022-1028` | Diff-based participant sub-listener mgmt: only (un)sub on room add/remove, not every emission | **Critical** — stops listener flicker, avoids stale subscribers |
| 2 | `BreathworkScreen.js:312-315, 1556-1557, 1829, 1839` | Hoist `new Animated.Value(dotSize/2)` etc. into `useRef` once; stop re-creating each render | **High** — breathwork is a flagship screen, animation jitter |
| 3 | `App.js:300-361` | Wrap `auth.authStateReady()` in 2s race; use `expo-splash-screen` API for native splash | **High** — cuts cold start on flaky network |
| 4 | `MainNavigator.js` (all 46 screens) | Convert `Stack.Screen component={…}` to `getComponent={() => require('…').default}` for non-default routes | **High** — biggest single bundle-parse win |
| 5 | `ChatScreen.js:338-357` | Replace 5 per-channel listeners with a single doc `users/{uid}/chatSeen` read on mount | **High** — saves 5× Firestore listener overhead |
| 6 | `ChatScreen.js:360-371` | Convert `userStats` from `useState` + effect to `useMemo([messages])` | **Medium** — removes extra render per incoming message |
| 7 | `AffirmationScreen.js:871, 953`, `InnerChildScreen.js:288`, `PartsWorkScreen.js:742` | Replace inline `renderItem={({item}) => …}` with memoized `renderItem` via `useCallback` | **Medium** — smoother scrolling |
| 8 | `GameScreen.js:18` | Replace `import * as THREE from 'three'` with named imports + `React.lazy` the whole screen | **Medium** — ~600KB off initial bundle |
| 9 | `DreamJournalScreen.js:216, 275` + `DocumentsScreen.js:60` | Wrap Animated.Value arrays in `useRef(...).current` (init once) | **Medium** — fixes animation leaks |
| 10 | `ProfileScreen.js:680, 1150` | Consolidate `useWindowDimensions` to a single top-level hook; remove second call | **Medium** — halves re-renders on orientation/keyboard on a 5.3k-line screen |

### Honorable mentions (not quite quick wins, worth scheduling)

- Extract `<MessageRow>` from `ChatScreen` as `React.memo` with hoisted handlers
- Migrate from `expo-av` to `expo-audio`/`expo-video` per SDK 54 deprecation path
- Delete `social-preview.png`, `grok-image-*.png`, `adaptive-icon-backup.png`, `icon-backup.png` from `/assets` (save ~1.2MB install)
- Add `getItemLayout` on `ChatScreen` FlatList once message-row height is deterministic
- Add `where('active','==',true)` on `VoiceChannelScreen.js:1010` rooms query

---

## Quick self-check

No blocker bugs found in the listener-cleanup paths (except N+1 fan-out). Animated.Value usage is 95% correct — just BreathworkScreen + DreamJournal + Documents need ref-wrapping. Bundle can shrink ~1MB+ with 3 steps: Three.js lazy, expo-av migration, asset trim.

Saved to app-performance-audit.md
