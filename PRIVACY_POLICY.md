# Safe(H)er — Privacy Policy

**Effective date:** July 21, 2026 · **Version:** 1.0.4
**Data controller:** Safe(H)er Team · privacy@safeher.app · www.safeher.app

---

## 1. Our commitment

Safe(H)er is used by people at risk of gender-based violence. We treat every byte of your data as if a hostile third party could inspect the device at any moment. This policy explains **exactly** what we collect, why, how long we keep it, and how you can control or delete it.

We collect the **minimum** necessary to keep you safe. Everything that can be processed on-device is processed on-device.

---

## 2. Data we collect

### 2.1 Data you provide (account)

| Data | Purpose | Legal basis (GDPR) |
|---|---|---|
| **Email address** | Login, password reset, deletion confirmation | Art. 6(1)(b) contract performance |
| **Password (bcrypt-hashed)** | Authentication | Art. 6(1)(b) |
| **Full name** (optional) | Personalisation of alert messages | Art. 6(1)(a) consent |
| **Phone number** (optional) | Included in outbound SOS as the sender identifier | Art. 6(1)(a) |
| **Safe(H)er ID** (auto-generated 8-char code) | Circle invitations | Art. 6(1)(b) |

### 2.2 Data you provide (safety config)

| Data | Purpose | Legal basis |
|---|---|---|
| **Trusted contacts** (name, phone, email, relationship) | Recipients of your SOS alerts | Art. 6(1)(b) |
| **Voice trigger phrases** (up to 3, plaintext in DB — encrypted at rest by the DB provider) | On-device keyword matching by the STT engine | Art. 6(1)(a) |
| **PIN hash** (SHA-256, salted) | Unlock the disguised interface | Art. 6(1)(a) |
| **Custom SOS message templates** | Personalised alert body | Art. 6(1)(a) |
| **App settings** (language, notification channel, sensitivity, etc.) | UX personalisation | Art. 6(1)(a) |

### 2.3 Data generated during use

| Data | When | Retention |
|---|---|---|
| **GPS location** (latitude, longitude, accuracy, reverse-geocoded address) | Captured **only** at the moment of an SOS event and while the Circle sharing session is active | See §5 |
| **Battery level & network status** | Included in each SOS event payload | See §5 |
| **SOS event history** (type, timestamp, notified contact IDs) | Every time you fire the SOS (manual, keyword, scream, shake, guardian timer) | See §5 |
| **Audio evidence recordings** | Only if you manually start a recording in the Evidence Vault | See §5 |
| **Circle live-location breadcrumbs** | Only while an accepted Circle link is active — never for un-linked peers | Automatically overwritten every N seconds (default 30s) |
| **Circle invites** (sender + receiver code, status) | When you invite someone | Deleted when accepted, declined, or the sender cancels |
| **Password-reset codes** (hashed) | 15-minute validity | Auto-deleted on use, expiry, or after 5 wrong attempts |
| **Deletion-request records** | When you use the public form at `/delete-account` | 12 months (audit trail) |
| **`last_seen_at`** timestamp on the user record | Updated on every authenticated request | Used by the inactivity sweeper (§5.3), erased on account deletion |

### 2.4 What we DO NOT collect

- ❌ **No advertising IDs** — no IDFA, no GAID.
- ❌ **No analytics SDKs** — no Google Analytics, no Firebase, no Mixpanel.
- ❌ **No third-party trackers** — nothing on our servers reports back to marketing platforms.
- ❌ **No continuous audio upload** — the microphone stream **never** leaves your device. Voice trigger + scream detection run entirely on-device using the OS-native speech recognizer (Apple SFSpeechRecognizer / Android SpeechRecognizer). We only receive the transcribed text — and only when a keyword matches, we fire an SOS event which is transmitted **without any audio**.
- ❌ **No image content** — camera roll access is used only for the calculator disguise icon; no image is uploaded to our servers.
- ❌ **No third-party cloud storage** — audio evidence stays on the primary MongoDB instance under our control.
- ❌ **No sale of personal data**. Ever. Under no circumstances.

---

## 3. Permissions we request

Each permission is asked contextually, only when the feature that needs it is enabled.

| Permission | Feature | What we access |
|---|---|---|
| **Location (When-in-use)** | SOS + Guardian Angel | GPS coordinates at the moment of the emergency |
| **Location (Always / Background)** | Circle live-location sharing | GPS coordinates while a Circle link is active |
| **Microphone** | Voice trigger, scream detection, evidence recording | Live audio stream processed **on-device**; audio never uploaded |
| **Contacts (read)** | Import contacts from phonebook | Read-only, one-shot; nothing is uploaded until you tick specific contacts |
| **Camera** | Photo evidence (roadmap v1.1) | Not yet used |
| **Biometrics / Face ID** | App unlock | Handled entirely by the OS; we only receive success/failure |
| **Foreground service** (Android) | Keep background features alive | The OS displays a persistent notification while active |
| **Vibration** | SOS confirmation, scream countdown | — |

---

## 4. Where data is stored and processed

- **Backend**: hosted on Emergent infrastructure, EU-friendly region (see current DPO contact).
- **Database**: MongoDB, encrypted at rest.
- **Email**: Resend for transactional email delivery (password reset, deletion confirmation). Resend acts as a data processor under a signed DPA. Only your email address is transmitted to Resend.
- **STT engine**: Apple (iOS) or Google (Android). These are the OS built-in speech engines. On iOS we explicitly request `requiresOnDeviceRecognition: true`. On Android on-device availability depends on the device manufacturer.

**No transfer outside the EU** is performed on personal data except where Resend/Apple/Google inherently operate cross-border for their processing; those transfers rely on **Standard Contractual Clauses** or an equivalent adequacy mechanism.

---

## 5. Data retention

### 5.1 Standard retention

Data is kept **for as long as your account is active** — because you might need to review past SOS events, audio evidence, or contact history in a legal context.

### 5.2 Deletion on user request

You can delete your account and all associated data at any time. Three paths:

1. **In-app (fastest, immediate)**: `Settings → Delete my account`. Requires you to re-enter your password. Erases everything within seconds.
2. **Public form (no login needed)**: [`https://www.safeher.app/delete-account`](https://www.safeher.app/delete-account). Enter your email; the request is queued and executed within **30 days** per GDPR Art. 17.
3. **Email**: `support@safeher.app` — reply-quality within 5 business days.

### 5.3 Automatic deletion on inactivity (opt-in)

New in v1.0.4: `Settings → Data retention → Auto-delete after inactivity`. You choose a threshold (30, 60, 90, 180, 365 days, or Never). If you don't sign in within that window, the server automatically purges your account and all associated data. Default is **Never** — you must actively opt-in.

A daily background sweeper (`server.py::_inactivity_sweeper`) enforces this. A 7-day advance warning email is sent when possible.

### 5.4 Anonymised audit log

When an account is deleted, we retain a single anonymised record with only:
- The **date** of deletion
- The **reason** you entered (if any)

No user ID, no email, no identifiable data. Retained for 6 months for regulatory audit.

---

## 6. Your GDPR rights

You have the right to:

- **Access** your data — via the app or by emailing `privacy@safeher.app`
- **Rectify** inaccurate data — directly editable in the app
- **Erase** your data — see §5.2
- **Restrict** processing — turn off features in Settings; disable notifications
- **Data portability** — request a JSON export at `privacy@safeher.app`
- **Object** to processing — email us
- **Lodge a complaint** with your national supervisory authority (Italy: Garante Privacy, garanteprivacy.it)

We respond to all requests within **30 days**.

---

## 7. Data security

See [`SECURITY.md`](./SECURITY.md) for the full technical posture. In brief:

- **Bcrypt** for password hashing.
- **HTTPS/TLS** for all in-transit traffic.
- **JWT tokens** in `expo-secure-store` (iOS Keychain / Android Keystore) — never in AsyncStorage.
- **Anti-enumeration** on `/forgot-password` and `/deletion-request` — responses never reveal whether an account exists.
- **Rate limiting** on password-reset codes (5 wrong attempts → invalidated).
- **On-device speech processing** — no audio ever leaves the phone.

---

## 8. Children

Safe(H)er is not directed to children under 13. We do not knowingly collect data from children. If you become aware that a minor has provided us data, please contact us to delete it.

---

## 9. Changes to this policy

We may update this policy to reflect legal or functional changes. The current version is always available at `https://www.safeher.app/privacy` and inside the app (Settings → Privacy Policy). Significant changes will be announced by email to the address associated with your account.

---

## 10. Contact

- **Privacy inquiries**: privacy@safeher.app
- **Security disclosures**: security@safeher.app
- **General support**: support@safeher.app
- **Postal address**: on request

---

*This policy is written in plain English to be understandable by non-lawyers. A legally-precise Italian version is available on request.*
