# Safe(H)er

**A production-ready emergency safety companion for victims of domestic violence, stalking, and assault.**

Version 1.1.21 · Android + iOS + Web · IT / EN / ES / FR / DE

---

## What is it?

Safe(H)er is a mobile application designed with a single mission: to protect people at risk of gender-based violence with a discreet, fast, private, and offline-resilient tool. Every design decision — from the calculator disguise to the on-device voice recognition — is made through the lens of an abuse survivor whose device may be inspected at any moment.

## Core features

### 🚨 Emergency SOS
- **3-second cancellable countdown** with strong haptics — one tap fires the SOS.
- **Automatic GPS + reverse geocoding** — full address included in the outbound message.
- **Battery + network status** in the payload so responders know your device state.
- **Country-based emergency-number routing** — the correct national number (112, 911, 1522, etc.) is dialed automatically based on device locale.

### 🎙️ Hands-free activation (dual channel)
Two independent channels share the same audio session — **zero extra battery cost**:
1. **Keyword trigger** — up to **3 personalised safewords**, matched with a **fuzzy Levenshtein algorithm** (tolerates transcription errors typical of native STT engines). Works in **any language**, processed **100 % on-device** (iOS SFSpeechRecognizer / Android SpeechRecognizer). `maxAlternatives=5` on every result to raise recall.
   - **Guardian Angel escalation (Android)** — if the keyword is recognised while a Guardian Angel timer is currently active, the app places a real, automatic call to **112** directly from the native Android layer (`ACTION_CALL`, requires the `CALL_PHONE` permission) — no dialer screen, no extra tap. Outside an active Guardian Angel session (or on iOS, or if the call permission isn't granted), the keyword trigger keeps opening the normal message composer/WhatsApp queue, unchanged.
2. **Scream detection (L2 hybrid)** — analyses microphone volume peaks; requires **two peaks within 3 s** above a configurable threshold (Low / Medium / High sensitivity). Designed for panic screams when no codeword is possible. Triggers a **5-second cancellable countdown with strong vibration** — you can abort false positives.

### 📱 Sequential WhatsApp queue
When the SOS fires with WhatsApp as the alert channel, a queue modal opens WhatsApp for each trusted contact one at a time. **Auto-advances** when you return to the app after each send — fixes the classic "only first contact received it" bug.

### 👥 Trusted contacts & Circle
- **Trusted Contacts** — up to 500 contacts, imported from the phonebook or added manually. Each can be individually enabled/disabled for SOS notifications.
- **Circle (live-location)** — mutual real-time location sharing with people you trust. Requires a **user code + double opt-in**, so nobody can track you without your explicit acceptance.

### 🎭 Disguise & lock
- **Calculator masked shell** — the app opens as a fully working calculator. Typing your PIN + `=` unlocks the real interface; a long-press with biometrics enabled also unlocks. This single combination is now the **only** unlock step — a second, redundant app-lock PIN prompt right after no longer appears.
- **PIN or biometric lock** — configurable in Settings. Unlock persists for the whole JS session; a cold restart re-locks.
- **PIN is stored only on-device** (never synced to the server in a verifiable form — see [`PRIVACY_POLICY.md`](./PRIVACY_POLICY.md) §2.2). To prevent a permanent lockout after a reinstall, new device, or cleared app data — situations where the disguise would otherwise demand a PIN that no longer exists locally — the calculator disguise **automatically stays disabled** until a new PIN is configured on that device, even if it was previously turned on for the account.
- **Immersive mode** — hides system bars during emergencies.

### 🎬 Evidence Vault
- Audio recording via `expo-audio` (video coming in v1.1).
- Encrypted at rest on the server; accessible only to the account owner via authenticated API.

### ⏰ Guardian Angel Timer
Dead-man switch: set a check-in interval, get reminded, and if you fail to confirm safety within the window, the SOS auto-fires with your last known position — sent silently in the background on Android when configured, or via the normal composer flow otherwise.
- **Single-delivery guarantee** — the native background alarm/SMS path now closes out its own session the moment it delivers (or confirms a prior delivery) for an automatic timer expiry. This prevents the app from re-firing a second, redundant "SOS with confirmation" request the next time you bring it to the foreground after a background auto-send.
- **Voice-trigger escalation** — see "Hands-free activation" above: your safeword calls 112 directly while a Guardian Angel session is active.
- A temporary, read-only diagnostics panel (Guardian Angel screen → "Diagnostica timer") surfaces the native timer's last scheduled time, alarm mode, and last SMS result, for QA/support purposes only — it does not change any Guardian Angel/SOS behaviour.

### 📞 Fake incoming call
Simulates a phone call at a scheduled time — useful for exiting uncomfortable situations discreetly.

### 🌍 Full i18n
IT / EN / ES / FR / DE — auto-detected from device locale, manually overridable in Settings.

### 🔒 Privacy & data controls
- **Show / hide password toggle** on all authentication screens.
- **In-app account deletion** — Settings → Delete my account. Requires password re-entry, purges every collection in one click.
- **Public deletion page** at [`https://www.safeher.app/delete-account`](https://www.safeher.app/delete-account) for users who have uninstalled or lost access.
- **Auto-delete after inactivity** (opt-in) — Settings → Data retention. Configurable at 30 / 60 / 90 / 180 / 365 days or Never. A daily background sweeper purges accounts past their threshold.
- **Anti-enumeration** on `/api/deletion-request` and `/api/auth/forgot-password` — neutral responses, no info leak on whether an email is registered.

---

## Tech stack

| Layer | Choice |
|---|---|
| **Frontend** | Expo SDK 54 · React Native · Expo Router (file-based) · TypeScript |
| **Backend** | FastAPI · Python 3.11 · Motor (async MongoDB) |
| **Database** | MongoDB |
| **Email** | Resend (OTP, deletion acknowledgments) |
| **Auth** | JWT tokens · bcrypt password hashing · `expo-secure-store` client-side |
| **Speech** | `expo-speech-recognition` (native STT, on-device) |
| **Native safety module** | `modules/silent-sms` (Expo Module, Android/Kotlin) — Guardian Angel alarm scheduling, silent SMS send, and the voice-trigger automatic call to 112 |
| **Vibration** | React Native `Vibration` + `expo-haptics` |
| **Package name** | `com.safeherofficial.safeher` |

---

## Repository layout
