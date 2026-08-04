# Remission — Product Plan & Architecture

An AI-assisted mobile app for people living with IBD (Crohn's disease and Ulcerative Colitis) that helps users manage flare-ups, recognize symptom patterns, stay organized with medications and appointments, and communicate better with their healthcare team.

> **Non-negotiable:** The app never diagnoses, never replaces medical advice, and never recommends medication changes. It organizes the user's own data into meaningful trends and prepares them for conversations with their care team.

---

## 1. Guiding Principles

1. **Non-diagnostic, always** — insights are framed as "patterns in your data," never conclusions or recommendations.
2. **Flare-friendly UX** — the app must work for someone in pain, fatigued, or brain-fogged. Flare Mode is a dedicated, low-effort logging path.
3. **1–2 taps to anything important.**
4. **Personalization by age / diagnosis / treatment** — a teen on Humira sees a different experience than an adult on infusions.
5. **Family-aware** — parents managing a child's IBD are a first-class use case.
6. **Calm, modern design** — soft blues, greens, neutral tones, rounded cards, large buttons, uncluttered layouts.

## 2. Target Audience

- Children with IBD (with parent assistance)
- Teenagers with Crohn's or UC (dedicated Teen Dashboard, ages 13–18)
- Adults with IBD
- Parents managing a child's IBD
- Newly diagnosed patients

---

## 3. Phased Roadmap

Each phase is shippable and usable on its own.

### Phase 1 — MVP (core daily value)
- Onboarding & profile (name, age, gender, diagnosis, year diagnosed, meds, treatment type, optional GI & emergency contact)
- Home Dashboard — today's status, pain, bathroom visits, mood, energy, sleep, water, medication status, upcoming appointments, streak, quick actions, and the **"I'm Having a Flare"** button
- Flare Mode quick-log (essential questions only, minimal effort)
- Daily Symptom Tracker (15 trackable symptoms)
- Bathroom Tracker — one-tap logging, Bristol scale, weekly/monthly trend graphs
- Medication Manager + basic local reminders (pills, liquid, biologics, infusions, injectables)
- Calendar (good day / mild flare / severe flare color-coding)
- Dark mode + large text (accessibility from day 1)

### Phase 2 — Treatment depth & family use
- Smart Treatment Reminders (infusion prep/hydration; injection numbing cream, ice pack, fridge removal, site rotation, needle disposal)
- Medication Response Tracker (post-dose observations & long-term trends)
- Injection Site Tracker (location, reactions, photos)
- Food & Hydration Journal (meals, snacks, drinks, water, portions, photos)
- Mood & Mental Wellness tracker (mood + influences, trend comparisons)
- Sleep Tracker (bedtime, wake, quality, awakenings, hours)
- Emergency Information storage
- Parent/child account linking (Firebase Auth + Firestore Security Rules)

### Phase 3 — Insights & clinical communication
- Pattern & Insights Dashboard (deterministic statistical engine)
- Lab Results tracking + trend graphs (CRP, ESR, fecal calprotectin, hemoglobin, iron, ferritin, vitamin D, albumin, WBC)
- Doctor Reports — printable/PDF export (symptoms, bathroom trends, pain charts, mood, adherence, injections, labs, food, sleep, weight, notes)
- Appointment Manager (colonoscopies, MRIs, blood work, infusions, questions for doctor, med changes)
- Teen Dashboard (school, family, friends, activities life-impact tracking)
- Menstrual cycle tracker (optional opt-in, compare hormonal changes with symptoms)

### Phase 4 — AI polish & accessibility depth
- LLM-powered weekly/monthly health summaries, personalized trend explanations, suggested doctor questions
- Voice input, high-contrast mode, one-handed navigation refinements
- Refill reminders tied to pharmacy timing, smarter notification bundling

---

## 4. Tech Stack

| Layer | Choice |
|---|---|
| Client | Expo SDK 54, React Native 0.81, TypeScript, React Navigation (bottom tabs + stacks + modal) |
| Backend | **Google Firebase** |
| Auth | Firebase Authentication (email/password; optional anonymous auth upgraded to full account later) |
| Database | Firestore **with offline persistence enabled** — the local cache is the local-first store; writes work instantly offline and sync automatically |
| Photos | Firebase Storage (injection sites, meals) |
| Server logic | Cloud Functions 2nd gen (TypeScript) — AI proxy, PDF report generation, notification orchestration |
| Push | Firebase Cloud Messaging via Expo push service (Phase 2+); `expo-notifications` local scheduling in Phase 1 |
| Charts | Victory Native (or react-native-svg-charts) |
| PDF export | Cloud Function (or `expo-print` client-side fallback) |

### Why Firestore as the local-first store
Flare Mode must work instantly, offline, anywhere. Firestore's SDK caches writes locally and syncs when connectivity returns — no custom SQLite + sync engine to build or keep consistent. Tradeoff: no SQL joins/aggregation; the pattern engine pulls date-range documents and computes statistics client-side (fine at personal-log scale).

## 5. System Shape

```
┌─────────────────────────────┐
│         Expo App            │
│  ┌────────────┐             │
│  │ UI Screens  │             │
│  │ (Navigation)│             │
│  └──────┬─────┘              │
│  ┌──────▼─────────────┐      │
│  │ Firestore SDK        │      │  ← offline cache = local-first store
│  │ (offline persistence)│      │
│  └──────┬─────────────┘      │
└─────────┼────────────────────┘
          │
   ┌──────▼───────────┐
   │ Firestore + Auth   │
   │ + Storage          │
   │ + Security Rules   │
   └──────┬─────────────┘
          │
   ┌──────▼───────────┐
   │ Cloud Functions    │
   │ (AI proxy, PDF,    │
   │  notifications)    │
   └──────┬─────────────┘
          │
   ┌──────▼───────────┐
   │   Claude API       │  ← server-side only, guardrailed,
   │ (summarize, never  │     never called from the client
   │  diagnose)         │
   └────────────────────┘
```

## 6. Data Model (Firestore)

```
users/{userId}
  profile (doc)            — name, age, gender, diagnosis, yearDiagnosed,
                             treatmentType, gastroenterologist?, accountRole
  emergencyInfo (doc)      — contacts, GI, PCP, hospital, insurance, allergies
  dailySymptomLogs/{id}    — pain, fatigue, nausea, appetite, energy, sleep,
                             temperature, weight, jointPain, mouthSores,
                             skinProblems, bloating, gas, headaches, overall
  flareLogs/{id}           — painLevel, painLocation, painType, bathroomFreq,
                             blood, mucus, urgency, fever, nausea, vomiting,
                             fatigue, appetite, stress, recentFood, medsTaken, notes
  bathroomLogs/{id}        — time, bristolScale, blood, mucus, urgency, pain,
                             nighttime, notes
  foodLogs/{id}            — type (meal/snack/drink), items, portion, waterMl,
                             mealTime, photoRef?
  medications/{medId}      — name, type (pill/liquid/biologic/infusion/injectable),
                             dose, frequency, schedule, refillThreshold
    doses/{doseId}         — scheduledTime, takenTime? (null → missed/late)
    responses/{id}         — painImprovement, fatigue, energy, nausea, appetite,
                             sideEffects, overallEffectiveness
  injectionSiteLogs/{id}   — location, redness, swelling, bruising, pain, itching,
                             warmth, bleeding, lump, duration, photoRef?, notes
  moodLogs/{id}            — mood (happy/calm/neutral/anxious/sad/frustrated/
                             overwhelmed), influences[]
  teenLifeLogs/{id}        — school, family, friends, activities metrics (age-gated)
  sleepLogs/{id}           — bedtime, wakeTime, quality, awakenings, totalHours
  labResults/{id}          — date, type (CRP/ESR/calprotectin/…), value, unit
  appointments/{id}        — date, type, provider, questionsForDoctor[], notes
  cycleLogs/{id}           — optional opt-in menstrual tracking

families/{familyId}
  members: [parentUserId, childUserId, …]
```

**Security Rules:** a user reads/writes only their own `users/{userId}` subtree; a parent can access a linked child's subtree only if listed in `families/{familyId}.members`. Photos inherit the same access controls and are never sent to the AI layer.

## 7. Navigation Structure

- **Bottom tabs:** Home · Track (symptom/bathroom/food hub) · Calendar · Insights · More (meds, labs, appointments, emergency info, settings)
- **Full-screen modal:** Flare Mode — an emergency action, not a navigation destination
- **Nested stacks per tab** for detail screens

## 8. AI / Insights Architecture (guardrails-first)

1. **Pattern engine (deterministic, no LLM).** Correlation/lag analysis over the user's own logs (e.g., pain vs. sleep with 1–2 day lag), output phrased from a fixed template library: *"On days following poor sleep, your logged pain was higher in X% of entries this month."* Runs client-side or in a Cloud Function; works even offline/local-only.
2. **LLM summarization layer (Phase 4, optional).** A Cloud Function sends only the pattern engine's structured output (never raw free text or photos) to Claude, with a system prompt hard-constrained to organizing/summarizing language — diagnostic or treatment-change phrasing is forbidden and filtered. Every output carries a persistent "based on your data, not medical advice" footer.
3. Disclaimers are recorded in the data model (`insight.disclaimerShown`) so guardrails are auditable, not just cosmetic.

## 9. Notifications

- **Phase 1:** all local via `expo-notifications` — medication times, water intake, daily check-ins.
- **Phase 2+:** treatment-aware reminder chains keyed off `Medication.type` (infusion: hydrate before/after, pack essentials; injectable: numbing cream, ice pack, fridge removal, site rotation, disposal). Push via FCM for cross-device/parent alerts.

## 10. Security & Privacy

- Firestore Security Rules scope all access; parent↔child access is explicit and auditable.
- Sensitive health data for minors — follow platform health-data guidelines; no third-party analytics on health content.
- Full data export and full data deletion as first-class settings.

## 11. Accessibility

- Dark mode, large text, and high-contrast are **theme tokens from day 1** — every screen respects them automatically.
- Flare Mode designed against the "high pain / brain fog" persona: large tap targets, minimal required fields, calm colors.
- Voice input and one-handed refinements in Phase 4.

## 12. Design Language

- Soft blues (#4A90D9-family), greens (sage/mint), warm neutrals
- Rounded cards (16–24px radii), generous spacing, large friendly buttons
- Clean, minimal charts; no alarming reds except where clinically meaningful (blood flag, severe flare)
- Tone: calm, professional, friendly — suitable for teens and adults
