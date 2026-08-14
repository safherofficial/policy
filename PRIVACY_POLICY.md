# Safe(H)er — Privacy Policy

**Effective date:** August 14, 2026 · **Version:** 1.1.21
**Data controller:** Safe(H)er Team · [safherofficial@gmail.com](mailto:safherofficial@gmail.com) · [www.safeherapp.site](http://www.safeherapp.site)

---

# 1. Our commitment

Safe(H)er is used by people at risk of gender-based violence. We treat every byte of your data as if a hostile third party could inspect the device at any moment.

This policy explains **exactly** what we collect, why we collect it, how long we keep it, and how you can control or delete it.

We collect the **minimum** necessary data required to provide safety features.

Everything that can be processed on-device is processed on-device.

---

# 2. Data we collect

## 2.1 Data you provide (account)

| Data                                               | Purpose                                      | Legal basis (GDPR)                |
| -------------------------------------------------- | -------------------------------------------- | ---------------------------------- |
| **Email address**                                  | Login, password reset, deletion confirmation | Art. 6(1)(b) contract performance |
| **Password (bcrypt-hashed)**                       | Authentication                               | Art. 6(1)(b)                      |
| **Full name** (optional)                           | Personalisation of emergency messages        | Art. 6(1)(a) consent              |
| **Phone number** (optional)                        | Included in outbound SOS communication       | Art. 6(1)(a) consent              |
| **Safe(H)er ID** (auto-generated 8-character code) | Circle invitations                           | Art. 6(1)(b)                      |

---

## 2.2 Data you provide (safety configuration)

| Data                                                    | Purpose                              | Legal basis  |
| ------------------------------------------------------- | ------------------------------------ | ------------ |
| **Trusted contacts** (name, phone, email, relationship) | Recipients of SOS alerts             | Art. 6(1)(b) |
| **Voice trigger phrases**                               | On-device keyword matching           | Art. 6(1)(a) |
| **PIN** — verification hash stored **only on the device**¹ | Unlock protection / disguise unlock | Art. 6(1)(a) |
| **Custom SOS messages**                                 | Personalised emergency communication | Art. 6(1)(a) |
| **Application settings**                                | User experience personalisation      | Art. 6(1)(a) |

¹ The PIN's actual, verifiable hash (SHA-256, locally salted) is generated and checked **entirely on-device** and is **never transmitted to our servers**. The server only stores a non-verifying flag indicating that "a PIN has been configured", so that your PIN-related settings (e.g. whether the calculator disguise is enabled) can sync across the app. Because of this, the verifiable PIN itself does not survive an app reinstall, a new device, or clearing the app's local data — see the calculator disguise note below.

---

## 2.3 Data generated during use

| Data                        | When collected                                    | Retention                 |
| --------------------------- | ------------------------------------------------- | ------------------------- |
| GPS location                | Only during SOS events and active Circle sessions | See Section 5             |
| Battery percentage          | Included in emergency messages                    | See Section 5             |
| Network status              | Included in emergency payload                     | See Section 5             |
| SOS history                 | Every emergency activation                        | See Section 5             |
| Audio evidence              | Only when manually recorded by the user           | See Section 5             |
| Circle location breadcrumbs | Only during active Circle sharing                 | Automatically overwritten |
| Circle invitations          | During invitation workflow                        | Deleted after completion  |
| Password reset codes        | During password recovery                          | Deleted after expiry      |
| Account deletion requests   | During deletion process                           | 12 months audit retention |
| last_seen_at timestamp      | Authentication activity                           | Deleted with account      |
# 2.4 What we DO NOT collect

Safe(H)er follows a privacy-first approach.

We do **not** collect:

* ❌ Advertising identifiers (IDFA, GAID)
* ❌ Analytics identifiers
* ❌ Marketing trackers
* ❌ Continuous microphone recordings
* ❌ Uploaded camera content
* ❌ Behavioural advertising data
* ❌ Sale or sharing of personal data
* ❌ Your PIN, in any verifiable form — it never leaves your device (see §2.2)

The microphone stream never leaves the device.

Voice trigger detection and scream detection are processed locally using the operating system speech recognition capabilities whenever possible.

Safe(H)er only receives the information required to activate an emergency workflow.

---

# 3. Permissions we request

Permissions are requested only when a user activates the related functionality.

| Permission                     | Feature                                             | Usage                                                      |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| Location (When-in-use)         | SOS + Guardian Angel                                | GPS position during emergency activation                   |
| Location (Always / Background) | Circle live location                                | Position sharing while an active Circle session exists     |
| Microphone                     | Voice trigger, scream detection, evidence recording | Local audio processing; audio is not continuously uploaded |
| Contacts                       | Trusted contacts import                             | One-time contact selection                                 |
| Camera                         | Future evidence features                            | Not currently used                                         |
| Biometrics / Face ID           | App protection                                      | Managed by device operating system                         |
| Foreground service             | Android background safety features                  | Keeps active safety services running                       |
| Vibration                      | Emergency confirmation                              | Feedback during SOS activation                             |
| SEND_SMS (Android optional)    | Guardian Angel silent SMS                           | Emergency SMS only after explicit activation and consent   |
| CALL_PHONE (Android optional)  | Voice-trigger automatic emergency call              | Places a call to 112 only when your safeword is recognised while Guardian Angel is active; see below |

---

# 3a. Guardian Angel — safety timer system

Guardian Angel is an optional safety feature designed to provide additional protection during potentially risky situations.

The feature requires explicit consent before activation.

Before enabling Guardian Angel:

* The user receives a clear explanation of the feature.
* The user confirms consent.
* At least one trusted contact must be configured.

Guardian Angel uses:

* GPS location
* Battery percentage
* Selected trusted contacts

Only when:

* The user starts a Guardian Angel timer.
* The timer expires without confirmation.
* An emergency event is triggered.

There is no continuous monitoring when Guardian Angel is inactive.

---

## Automatic silent SMS (Android)

Automatic silent SMS is an optional Android-only feature.

By default, Guardian Angel uses the normal emergency communication flow requiring user interaction.

If enabled:

* The user explicitly grants permission.
* The feature works only when a Guardian Angel timer expires.
* It sends the emergency message directly from the device.
* It is never used for advertising, tracking, or other purposes.
* It is not available on iOS due to platform restrictions.

The user can disable Guardian Angel and silent SMS at any time from application settings.

---

## Automatic emergency call via voice trigger (Android)

This is an optional, Android-only escalation of the hands-free voice trigger feature described in Section 3.

If your recognised safeword is heard **while a Guardian Angel session is currently active**:

* The app places a real, automatic call to the single European emergency number (**112**) directly from the device, without opening the dialer or requiring a further tap.
* This requires the separate `CALL_PHONE` permission, requested only the first time this situation occurs.
* If the permission is not granted, or you are on iOS, or no Guardian Angel session is active, the voice trigger keeps behaving as normal: it opens the message composer / WhatsApp queue to your trusted contacts, exactly as when Guardian Angel is off.
* This call is placed only to the emergency number — it is never used to call any other number, and never for advertising, tracking, or any other purpose.
* You can avoid this behaviour at any time by not enabling Guardian Angel, by not granting the `CALL_PHONE` permission, or by disabling voice trigger in Settings.

---

# 4. Where data is stored and processed

Safe(H)er infrastructure is designed according to privacy-by-design principles.

## Backend

Backend services are hosted on secure infrastructure.

## Database

User information is stored in MongoDB with encryption at rest.

## Email delivery

Transactional emails are used only for:

* Password recovery
* Account deletion confirmation
* Security notifications

Only the minimum required email information is processed.

## Speech recognition

Speech processing uses:

* Apple speech recognition services on iOS
* Android speech recognition services

Whenever supported, on-device recognition is preferred.

---

# Website and privacy resources

Official website:

https://www.safeherapp.site

Privacy policy:

https://www.safeherapp.site/privacy

Account deletion:

https://www.safeherapp.site/delete-account
# 5. Data retention

## 5.1 Standard retention

Safe(H)er keeps personal data only for the period necessary to provide the requested safety services.

Account data is maintained while the account remains active because users may need access to:

* Previous SOS events
* Safety configurations
* Trusted contacts
* Emergency history
* Evidence records

---

## 5.2 Deletion on user request

Users can request deletion of their account and associated data at any time.

Available methods:

### 1. In-app deletion

Path:

```
Settings → Delete my account
```

The user must confirm identity before deletion.

The deletion process removes associated personal data according to applicable GDPR requirements.

---

### 2. Website deletion request

Users can submit a deletion request through:

https://www.safeherapp.site/delete-account

Requests are processed according to GDPR Article 17 requirements.

---

### 3. Email request

Users can contact:

[safherofficial@gmail.com](mailto:safherofficial@gmail.com)

for account deletion or privacy-related requests.

---

## 5.3 Automatic deletion after inactivity

Safe(H)er may provide optional inactivity-based deletion controls.

Users can select their preferred retention period:

* 30 days
* 60 days
* 90 days
* 180 days
* 365 days
* Never

The default option is:

**Never**

No automatic deletion occurs unless explicitly activated by the user.

---

# 6. Your GDPR rights

Under GDPR regulations, users have the right to:

## Access

Request information about stored personal data.

## Rectification

Correct inaccurate information.

## Erasure

Request deletion of personal data.

## Restriction

Limit processing through account settings or privacy controls.

## Data portability

Request a structured export of personal information.

## Objection

Object to specific processing activities.

---

Privacy requests can be sent to:

**[safherofficial@gmail.com](mailto:safherofficial@gmail.com)**

We aim to respond within the applicable GDPR timeframe.

---

# 7. Data security

Safe(H)er applies technical and organisational measures designed to protect personal information.

Security measures include:

* Bcrypt password hashing
* HTTPS/TLS encrypted communication
* Secure token storage
* Protection against account enumeration
* Rate limiting for authentication workflows
* Minimal data collection principles
* Local processing whenever possible
* PIN verification kept entirely on-device (see §2.2) — our servers never receive a verifiable copy of your PIN
* Calculator disguise mode automatically stays disabled on any device that does not have a locally configured PIN (e.g. after a reinstall), so you can never be permanently locked out of your own account by the disguise

Sensitive emergency data is transmitted only when required to provide safety functionality.

---

# 8. Children

Safe(H)er is not intended for children under 13 years old.

We do not knowingly collect personal information from children.

If you believe that a minor has provided personal data, contact:

[safherofficial@gmail.com](mailto:safherofficial@gmail.com)

and we will take appropriate action.

---

# 9. Changes to this Privacy Policy

Safe(H)er may update this Privacy Policy when:

* New features are introduced
* Legal requirements change
* Security improvements are implemented

The latest version is always available at:

https://www.safeherapp.site/privacy

Significant changes may be communicated through available channels.

---

# 10. Contact

For privacy, security and support requests:

**Safe(H)er Team**

Email:

[safherofficial@gmail.com](mailto:safherofficial@gmail.com)

Website:

https://www.safeherapp.site

Account deletion:

https://www.safeherapp.site/delete-account

---

This Privacy Policy is written to provide clear information about Safe(H)er's privacy practices and explain how personal data is handled.

A legally precise Italian version can be provided when required.
