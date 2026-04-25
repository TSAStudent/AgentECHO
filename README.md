# Agent ECHO

> Your ears, voice, and executive assistant — an always-on autonomous AI
> companion for the Deaf and hard-of-hearing community.

Agent ECHO runs passively in the background and acts on your behalf across
every domain of daily life: home, work, school, healthcare, travel,
relationships, and emergencies. You should never have to "open the app."
ECHO hears what you miss, interprets it in context, decides what matters,
and does the right thing — whether that's silently logging a reminder,
flashing an urgent alert, booking a ride, translating ASL to a clerk, or
calling your emergency contacts.

```
.
├── mobile/        Expo React Native app (iOS / Android)
└── backend/       Node.js + Express API (OpenAI, Twilio, summaries, vision)
```

---

## 1 · Quick start (5 minutes)

### 1.1  Clone & install
```bash
git clone https://github.com/TSAStudent/AgentECHO.git
cd AgentECHO
npm install               # installs mobile + backend via workspaces
```

### 1.2  Configure environment variables

Copy the template and fill in the keys you have. **Every variable is
optional** — any integration without a key runs in graceful demo mode so the
UI always has something to show.

```bash
cp .env.example .env.local
cp .env.example backend/.env
```

Edit `backend/.env` (the backend reads this file). The mobile app picks up
`EXPO_PUBLIC_*` variables from `.env.local` at the repo root automatically.

### 1.3  Run it

```bash
npm run dev
```

This starts:
- **Backend** at `http://localhost:4000`
- **Expo** at `http://localhost:8081` with a QR code you can scan with Expo Go

```bash
# Or run them individually
npm run dev:backend
npm run dev:mobile
```

---

## 2 · Complete environment variable reference

Paste these into **both** `.env.local` (repo root) **and** `backend/.env`.
Any integration without a key automatically falls back to demo mode, so the
app is always runnable.

### 2.1  OpenAI — REQUIRED for real AI features
```bash
OPENAI_API_KEY=sk-REPLACE_ME
```
**How to get it:** https://platform.openai.com/api-keys → *Create new secret
key*. The key must have access to `whisper-1`, `gpt-4o`, `gpt-4o-mini`,
`tts-1`, and GPT-4o vision (enabled by default on paid accounts).

Powers:
- `/api/transcribe` (Whisper)
- `/api/extract-actions` (GPT-4o-mini smart action engine)
- `/api/summarize` (lecture notes, flashcards, meeting follow-ups)
- `/api/asl-recognize` (GPT-4o Vision sign recognizer)
- `/api/vibe-report` (post-meeting emotional dynamics)
- `/api/tts` (drive-thru / "please read this" mode)

### 2.2  Twilio — REQUIRED for Trusted Circle SMS
```bash
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_FROM_NUMBER=+15555550123
```
**How to get them:**
1. Sign up at https://www.twilio.com/try-twilio (trial credits are free).
2. In the Twilio Console home (https://console.twilio.com/) copy your
   **Account SID** and **Auth Token**.
3. Under *Phone Numbers → Buy a number* grab one SMS-capable number (you
   can use the free trial number). Paste it as `TWILIO_FROM_NUMBER` in E.164
   format (leading `+`, no spaces).

Powers the `/api/emergency` tiered SMS + live-location share.

### 2.3  Google Maps — REQUIRED for spatial direction + evacuation routing
```bash
GOOGLE_MAPS_API_KEY=AIzaXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
EXPO_PUBLIC_GOOGLE_MAPS_API_KEY=AIzaXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX   # same value
```
**How to get it:**
1. Go to https://console.cloud.google.com/ → create or pick a project.
2. Enable these APIs under *APIs & Services → Library*:
   - Maps SDK for Android
   - Maps SDK for iOS
   - Geocoding API
   - Directions API
3. Go to *APIs & Services → Credentials → Create credentials → API key*.
4. Restrict the key by app bundle id (`ai.agentecho.app`) in production.

> **Important:** Expo only exposes env vars whose name starts with
> `EXPO_PUBLIC_` to the mobile bundle. That's why you must set both.

### 2.4  Hume AI — OPTIONAL (vocal emotion / prosody)
```bash
HUME_API_KEY=
```
Get it at https://beta.hume.ai/settings/keys. Leave blank to use the built-in
heuristic emotion tagger.

### 2.5  Anthropic Claude — OPTIONAL (fallback planner)
```bash
ANTHROPIC_API_KEY=
```
https://console.anthropic.com/settings/keys

### 2.6  Firebase Cloud Messaging — OPTIONAL (Night Mode push)
```bash
FIREBASE_PROJECT_ID=
FIREBASE_SERVER_KEY=
```
Create a Firebase project at https://console.firebase.google.com/, then
*Project settings → Cloud Messaging* to find the IDs.

### 2.7  Backend ↔ Mobile wiring
```bash
BACKEND_PORT=4000
EXPO_PUBLIC_API_URL=http://localhost:4000
```
- **Android emulator:** use `http://10.0.2.2:4000`
- **iOS simulator:** `http://localhost:4000` works
- **Physical device (Expo Go):** use your machine's LAN IP, e.g.
  `http://192.168.1.42:4000` (run `ipconfig getifaddr en0` on macOS or
  `hostname -I` on Linux).

### 2.8  Postgres — OPTIONAL (otherwise in-memory)
```bash
DATABASE_URL=
```

### 2.9  Misc
```bash
NODE_ENV=development
LOG_LEVEL=info
```

---

## 3 · Feature map

| Capability                                | Where it lives                                  |
|------------------------------------------ |------------------------------------------------ |
| Ambient sound-type classification         | `backend/src/services/soundClassifier.js` + `AmbientScreen` |
| Name-triggered smart action capture       | `backend/src/services/smartActions.js` + `HomeScreen`        |
| Live transcription + speaker diarization  | `backend/src/services/transcribe.js` + `ConversationScreen`  |
| Two-way ASL translator (GPT-4o Vision)    | `backend/src/services/aslVision.js` + `AslScreen`            |
| Tiered emergency SMS + location share     | `backend/src/services/emergency.js` + `EmergencyScreen`      |
| Lecture/Meeting auto-notes & flashcards   | `backend/src/services/summarize.js` + `ClassroomScreen`      |
| Vibe report (post-meeting emotional read) | `backend/src/services/vibeReport.js`                         |
| Medical companion + med reminders         | `MedicalScreen`                                              |
| Privacy controls + retention              | `SettingsScreen`                                             |

---

## 4 · Project structure

```
AgentECHO/
├── .env.example                   template for .env.local and backend/.env
├── package.json                   workspaces root
├── backend/
│   └── src/
│       ├── index.js               Express entry point
│       └── services/
│           ├── openaiClient.js
│           ├── transcribe.js        Whisper + heuristic diarization
│           ├── smartActions.js      GPT-4o-mini action extraction
│           ├── soundClassifier.js   YAMNet fallback
│           ├── summarize.js         Lecture + meeting notes
│           ├── aslVision.js         GPT-4o vision sign recognizer
│           ├── vibeReport.js        Post-meeting vibe
│           ├── tts.js               OpenAI TTS
│           └── emergency.js         Twilio Trusted Circle
└── mobile/
    └── src/
        ├── theme/                 design system (colors, type, radii)
        ├── components/            GlassCard, PulseRing, WaveformBars, …
        ├── context/EchoContext    global state (listening, events, circle)
        ├── navigation/            stack + tabs, blurred tab bar
        └── screens/
            ├── OnboardingScreen
            ├── HomeScreen            ambient status + captured actions
            ├── AmbientScreen         live sound event log + direction radar
            ├── ConversationScreen    live captions w/ diarization + emotion
            ├── AslScreen             two-way sign ↔ speech translator
            ├── EmergencyScreen       hold-for-SOS + Trusted Circle
            ├── ClassroomScreen       lecture + meeting auto-notes
            ├── MedicalScreen         appointment companion
            └── SettingsScreen        privacy, retention, profile
```

---

## 5 · Privacy posture

- Raw audio is **never** stored off-device by default.
- Cloud offload (Whisper / GPT-4o) is **opt-in** per screen and can be
  disabled globally in *Settings → Privacy*.
- Transcripts auto-delete after a user-chosen window (default 7 days).
- Twilio messages never contain transcript content unless the user types
  one manually.
- HIPAA-ready architecture for medical features (TLS in transit, encrypted
  at rest, configurable BAA support).

---

## 6 · Roadmap (what's next)

- Native foreground service (`expo-modules-core`) for true 24/7 listening.
- Fine-tuned WLASL / How2Sign transformer replacing the GPT-4o Vision
  substitute.
- ReSpeaker-based multi-mic room hub with real direction-of-arrival.
- Baby-monitor profile with per-child voice enrollment.
- Partner/caregiver sync (scoped permissions).

---

Built for a national-level competition. If this helps, star the repo and
tell a judge a story from it. 💜

Main features
Always-on assistant concept: App is designed to run in the background and surface important events/actions without requiring constant manual interaction.
Ambient sound awareness: Classifies nearby sounds and highlights important sound events (with direction/radar-style UI).
Live conversation captions: Real-time transcription with speaker separation cues and emotion/context tagging.
Smart action capture: Detects intent from speech/text and turns it into actionable reminders/tasks.
ASL two-way translator: Sign-to-speech and speech/text-to-ASL companion flow (vision-assisted ASL recognition path in backend).
Emergency mode (Trusted Circle): Hold-to-SOS flow, tiered emergency alerts, and location sharing via SMS integrations.
Classroom/meeting assistant: Auto-generated lecture/meeting notes, summaries, and follow-up style outputs (including flashcard-like study support).
Medical companion tools: Appointment/medical support workflows plus medication-related reminders/tracking.
Post-meeting vibe report: Emotional/social dynamic summary after conversations/meetings.
Text-to-speech helper: Speak generated or typed text in accessibility scenarios.
Privacy controls: Retention controls and settings for data handling behavior.
User profile + preferences: Personalization and app behavior settings (contacts, state, preferences endpoints on backend).
