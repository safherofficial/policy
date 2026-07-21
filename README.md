# Safe(H)er

**A production-ready emergency safety companion for victims of domestic violence, stalking, and assault.**

Version 1.0.4 · Android + iOS + Web · IT / EN / ES / FR / DE

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
2. **Scream detection (L2 hybrid)** — analyses microphone volume peaks; requires **two peaks within 3 s** above a configurable threshold (Low / Medium / High sensitivity). Designed for panic screams when no codeword is possible. Triggers a **5-second cancellable countdown with strong vibration** — you can abort false positives.

### 📱 Sequential WhatsApp queue
When the SOS fires with WhatsApp as the alert channel, a queue modal opens WhatsApp for each trusted contact one at a time. **Auto-advances** when you return to the app after each send — fixes the classic "only first contact received it" bug.

### 👥 Trusted contacts & Circle
- **Trusted Contacts** — up to 500 contacts, imported from the phonebook or added manually. Each can be individually enabled/disabled for SOS notifications.
- **Circle (live-location)** — mutual real-time location sharing with people you trust. Requires a **user code + double opt-in**, so nobody can track you without your explicit acceptance.

### 🎭 Disguise & lock
- **Calculator masked shell** — the app opens as a fully working calculator. A specific PIN + `=` sequence unlocks the real interface.
- **PIN or biometric lock** — configurable in Settings. Unlock persists for the whole JS session; a cold restart re-locks.
- **Immersive mode** — hides system bars during emergencies.

### 🎬 Evidence Vault
- Audio recording via `expo-audio` (video coming in v1.1).
- Encrypted at rest on the server; accessible only to the account owner via authenticated API.

### ⏰ Guardian Angel Timer
Dead-man switch: set a check-in interval, get reminded, and if you fail to confirm safety within the window, the SOS auto-fires with your last known position.

### 📞 Fake incoming call
Simulates a phone call at a scheduled time — useful for exiting uncomfortable situations discreetly.

### 🌍 Full i18n
IT / EN / ES / FR / DE — auto-detected from device locale, manually overridable in Settings.

### 🔒 Privacy & data controls (v1.0.4)
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
| **Speech** | `@likeyoureyes/expo-speech-recognition` (native STT, on-device) |
| **Vibration** | React Native `Vibration` + `expo-haptics` |
| **Package name** | `com.safeherofficial.safeher` |

---

## Repository layout

```
/app
├── backend/
│   ├── server.py              # FastAPI + all API endpoints + inactivity sweeper
│   ├── emails.py              # Resend integration (OTP, deletion ack)
│   ├── support_numbers.py     # National emergency number lookup
│   └── tests/                 # pytest suite (iterations 10, 11, 12)
├── frontend/
│   ├── app/
│   │   ├── (tabs)/            # index / contacts / circle / vault / settings
│   │   ├── auth/              # login, register, forgot
│   │   ├── delete-account.tsx # public deletion page
│   │   ├── guardian.tsx
│   │   ├── fake-call*.tsx
│   │   └── privacy.tsx
│   ├── src/
│   │   ├── components/        # AppLock, MaskedShell, VoiceTrigger,
│   │   │                        ScreamCountdown, WhatsAppQueue, PasswordInput
│   │   ├── utils/             # voice-detect, emergency-bus, sos-fire,
│   │   │                        guardian, messages, background, pin
│   │   ├── i18n/              # locales/{en,it,es,fr,de}.ts
│   │   └── api/client.ts      # typed API client
│   └── app.json
├── delete-account.html        # standalone HTML for external landing pages
├── DELETE_ACCOUNT_INTEGRATION.md
├── RELEASE_NOTES_v1.0.4.md
├── PRIVACY_POLICY.md
├── SECURITY.md
├── TERMS_OF_SERVICE.md
└── README.md
```

---

## API endpoints (public + authenticated)

### Auth
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`
- `POST /api/auth/forgot-password` — sends 6-digit code via email; **never** returns the code in the response
- `POST /api/auth/reset-password` — locks after 5 wrong attempts per code
- `DELETE /api/auth/me` — requires password re-entry, purges everything

### Emergency
- `POST /api/events` — records an SOS event
- `GET /api/events` — event history
- `POST /api/events/{id}/evidence` — attach evidence

### Contacts
- `GET /api/contacts` · `POST /api/contacts` · `PUT /api/contacts/{id}` · `DELETE /api/contacts/{id}`

### Circle
- `POST /api/circle/invite` · `POST /api/circle/accept` · `POST /api/circle/location`
- `GET /api/circle/links` · `GET /api/circle/invites`
- `DELETE /api/circle/link/{id}`

### Settings
- `GET /api/settings` · `PUT /api/settings`

### Compliance
- `POST /api/deletion-request` — **public**, no-auth, neutral response

---

## Getting started

### Development

```bash
# Backend
cd /app/backend
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8001 --reload

# Frontend
cd /app/frontend
yarn install
yarn expo start
```

Environment variables live in `/app/backend/.env` and `/app/frontend/.env`.

### Running tests

```bash
cd /app/backend
pytest tests/ -v
```

### Deployment

Deployed via Emergent hosting. Publish button in the Emergent dashboard generates:
- Web bundle at `https://www.safeher.app`
- Android APK/AAB (package `com.safeherofficial.safeher`, versionCode 5)
- iOS build (requires Apple Developer account)

Release notes for v1.0.4: [`RELEASE_NOTES_v1.0.4.md`](./RELEASE_NOTES_v1.0.4.md).

---

## Security posture

Read the full [`SECURITY.md`](./SECURITY.md). Highlights:

- ✅ **SEC-001 fixed** — reset codes never leaked in HTTP responses, brute-force protection (5 attempts).
- ⚠️ **SEC-002** — PIN uses SHA-256 + static salt (roadmap: PBKDF2 + per-user salt + lockout).
- ✅ **SEC-003 fixed** — GDPR Art. 17 in-app + public deletion + opt-in auto-delete after inactivity.
- ⚠️ **SEC-004** — Live secrets in tracked `.env` (roadmap: secret manager rotation).

Reporting: `security@safeher.app`.

---

## License

Proprietary — © 2026 Safe(H)er. Not for redistribution.

## Contact

- **Support**: support@safeher.app
- **Privacy**: privacy@safeher.app
- **Security**: security@safeher.app
- **Domain**: www.safeher.app

---

Made with love for the people who need it most. 💜
