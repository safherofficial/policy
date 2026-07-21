# Safe(H)er — Security Posture

**Version:** 1.0.4 · **Last review:** July 21, 2026 · **Contact:** security@safeher.app

This document is the security counterpart to [`PRIVACY_POLICY.md`](./PRIVACY_POLICY.md). It describes the concrete technical measures in place, the outstanding known risks, and how to report new vulnerabilities.

Safe(H)er is used by people at physical risk. We hold ourselves to a higher security standard than a generic consumer app.

---

## 1. Threat model

| Threat actor | Capability | Mitigations |
|---|---|---|
| **Curious abuser with physical access to unlocked phone** | Can open the app icon | Calculator disguise (MaskedShell) + PIN/biometric lock |
| **Curious abuser with brief physical access to locked phone** | Can inspect the app drawer | Launcher label is "Calculator" and the icon looks like a calculator; disguised on OS level |
| **Abuser who steals the phone briefly and unlocks it** | Can navigate the app once unlocked | PIN re-prompt on `Delete my account` action (defense-in-depth); no auto-unlock on background→foreground (session-only) |
| **Attacker who knows the victim's email** | Can attempt password reset / deletion | Reset code delivered **only** via email; anti-enumeration responses; 5-attempt lockout on the reset code; deletion form does not reveal whether the account exists |
| **Attacker with network eavesdropping** | Can inspect traffic | Full TLS on all API endpoints |
| **Attacker with server-side access** | Full DB access if backend is breached | Passwords bcrypt-hashed; audio evidence stored server-side but subject to your compromised backend — future roadmap: E2E encryption |
| **Attacker with source-code access to the git repo** | Can read `.env` if not gitignored | ⚠️ **SEC-004 open** — secrets currently in tracked `.env`; rotation + secret manager on roadmap |

---

## 2. Authentication

- **Passwords** are hashed with **bcrypt** (cost factor 12) via the `passlib` library.
- **JWT tokens** (HS256) with **7-day expiry**, signed by `JWT_SECRET`.
- **Token storage** on client: `expo-secure-store` — iOS Keychain / Android EncryptedSharedPreferences. **Never** in AsyncStorage or WebStorage.
- **Password reset**: 6-digit code, 15-minute validity, invalidated after **5 wrong attempts**.
- **Anti-enumeration**: `/forgot-password` and `/deletion-request` return `{sent: true}` / `{received: true}` regardless of whether the email exists.
- **Password-visibility toggle** on all auth screens (login, register, reset) — `autoCorrect={false}` to prevent Android keyboards from injecting characters when visible.

---

## 3. Session hardening

- **No auto-relock on background→foreground** — deliberate UX/safety trade-off. Once you unlock in a JS session you stay unlocked until the OS tears down the app; this prevents the emergency flow (opening WhatsApp, Maps, permission dialogs) from throwing you back to the calculator/PIN mid-SOS.
- **Sensitive re-authentication** — `DELETE /api/auth/me` requires **password re-entry** even with a valid bearer token, defense-in-depth against stolen/shoulder-surfed sessions.
- **Session lifetime**: 7 days from issuance; refresh requires full sign-in.

---

## 4. Data protection

### 4.1 At rest

- **MongoDB** encrypted at rest on the hosting layer.
- **Audio evidence** stored as base64 blobs in `evidence` collection. Access gated by `user_id` foreign key.
- **Voice trigger phrases** stored plaintext in the `settings` collection (needed for STT matching). Encrypted only by the DB-level encryption. Roadmap: per-user AES key so even a DB dump doesn't expose codewords.

### 4.2 In transit

- **HTTPS/TLS 1.2+** enforced by the ingress controller.
- **CORS** currently `allow_origins=["*"]` — acceptable for the current architecture (no cookie-based auth, only bearer tokens), but on the roadmap to restrict to known origins for defense-in-depth.

### 4.3 On-device

- **PIN hash** currently SHA-256 + static salt (**SEC-002 open** — see §7). Roadmap: PBKDF2 with per-user salt.
- **Audio for voice/scream detection** never leaves the device.
- **Emergency countdown state** kept in memory only.

---

## 5. Privacy-preserving audio pipeline

The voice trigger and scream detector share **one microphone session** owned by `@likeyoureyes/expo-speech-recognition`. This session:

1. Runs the OS-native STT engine (iOS SFSpeechRecognizer with `requiresOnDeviceRecognition: true`; Android SpeechRecognizer).
2. Emits **transcript strings** via `result` events — these are matched against your saved keywords using fuzzy Levenshtein.
3. Emits **normalised volume samples** via `volumechange` events — these feed the ScreamDetector which flags two peaks within 3 s above threshold.
4. **Never** returns raw audio to our JS layer. **Never** uploads audio to any server.

If a keyword or scream is matched, we emit a **synthetic event** — `POST /api/events` — containing only:
- Event type (`sos_keyword` / `sos_scream`)
- Timestamp
- GPS coordinates (if permission granted)
- Battery level
- Notified contact IDs

No audio.

---

## 6. Right to erasure (GDPR Art. 17)

Three routes, all implemented and audited:

1. **`DELETE /api/auth/me`** — authenticated, password-verified, purges every per-user document across 9 collections (`contacts`, `events`, `evidence`, `settings`, `locations`, `circle_invites`, `circle_links`, `password_resets`, and `users`). See `server.py::purge_user_and_data`.
2. **`POST /api/deletion-request`** — public, no-auth, queues a manual deletion (processed within 30 days). Anti-enumeration.
3. **Inactivity sweeper** — daily background task purges accounts whose `last_seen_at` is older than the user's opt-in threshold (`settings.auto_delete_after_days`, values 30/60/90/180/365).

After deletion we retain a single **anonymised audit record** — date + optional reason string only — for 6 months.

---

## 7. Known open items (roadmap)

### 🟡 SEC-002 — Weak PIN hashing
**Status:** open · **Priority:** P2

- Current: single-round SHA-256 + static salt.
- Target: PBKDF2-SHA256 (100 000 rounds) + per-user random salt.
- Also missing: **lockout counter** on wrong PIN attempts.

### 🟢 SEC-004 — Secrets in tracked `.env`
**Status:** open · **Priority:** P3

- `JWT_SECRET`, `RESEND_API_KEY`, and Mongo URL currently sit in `/app/backend/.env`.
- `.gitignore` excludes `*.env.local` but not `.env` itself.
- Roadmap: move to Emergent secret manager, rotate both keys.

### 🟢 CORS wildcard
**Status:** open · **Priority:** P3 (defense-in-depth)

- Restrict `allow_origins` to known frontends (`https://www.safeher.app`, Emergent preview domain).

### 🟢 File-upload size limits
**Status:** open · **Priority:** P3

- No explicit cap on audio evidence upload size — theoretical DoS via storage exhaustion.
- Roadmap: enforce 25 MB per recording, aggregate quota per user.

---

## 8. Recently closed (v1.0.4)

### ✅ SEC-001 — Reset code leaked in HTTP response
**Fixed 2026-07-20.**

- Old behaviour: `POST /auth/forgot-password` returned `{sent: true, code: "123456"}` when `EXPOSE_RESET_CODE_DEV=1` (which was accidentally set to `1` in production).
- Impact: any attacker knowing a victim's email could take over the account via a single API call + a reset call.
- Fix: the `code` field was removed from the endpoint entirely (dead code, not just gated). The `.env` flag was set to `0` for defense-in-depth.
- Additional hardening: brute-force protection on the reset endpoint (5-attempts lockout).
- Regression test in `tests/test_iteration12_account_deletion.py::TestSEC001Regression`.

### ✅ SEC-003 — Indefinite data retention
**Fixed 2026-07-21.**

- Added `DELETE /api/auth/me`, `POST /api/deletion-request`, and the inactivity sweeper.
- Full test coverage in `tests/test_iteration12_account_deletion.py`.

---

## 9. Testing

- **Backend**: pytest suite (`/app/backend/tests/`) — 10/10 passing on iteration 12.
- **Frontend**: Playwright-driven smoke tests via the internal testing agent.
- **Static analysis**: `ruff` + `mypy` (Python), `eslint` (TypeScript).
- **Manual pen-test checkpoints**: prior to every store release.

---

## 10. Responsible disclosure

If you find a vulnerability:

1. Email **security@safeher.app** with a proof-of-concept.
2. Please **do not** publicly disclose before we've had 90 days to fix (standard responsible-disclosure window).
3. We commit to acknowledging your report within 5 business days.

We do not currently run a bug-bounty programme but we're happy to publicly credit good-faith researchers.

---

## 11. Dependencies

Kept up-to-date monthly. Notable:

- `fastapi`, `motor`, `passlib[bcrypt]`, `python-jose`, `resend` — server
- `expo` SDK 54, `@likeyoureyes/expo-speech-recognition`, `expo-secure-store`, `expo-haptics`, `expo-location`, `expo-audio` — client
- Removed `@howincodes/expo-dynamic-app-icon` (was breaking EAS Android build)
- Removed `expo-av` (deprecated) — replaced with `expo-audio`

---

## 12. Physical device recommendations for users

We recommend users at high risk:

1. Use a **strong device passcode** (6+ digits) — nothing we do can protect data if the phone itself is unlocked and unattended.
2. **Enable biometric unlock** on Safe(H)er + configure a **PIN** that differs from the device passcode.
3. **Enable the calculator disguise** if you share your phone with the abuser.
4. **Configure the inactivity auto-delete** (Settings → Data retention → 90 days) — a fail-safe if you lose access to the device.
5. **Verify Circle invites** face-to-face before accepting.
6. **Test the voice trigger** in a safe environment before relying on it.

---

*This document is public. The internal incident-response playbook and the DPO contact are separate.*
