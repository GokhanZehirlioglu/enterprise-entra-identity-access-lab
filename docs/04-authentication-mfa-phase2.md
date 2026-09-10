# Authentication & MFA

> **Phase:** 2 — Authentication & MFA  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Technical implementation and validation complete; repository closeout in progress  
> **Date:** 2026-09-10

## Objective

Design, implement and validate a modern Microsoft Entra authentication and MFA baseline for the WEZ lab while keeping authentication-method configuration separate from Conditional Access enforcement.

Phase 2 focuses on authentication methods, registration, passwordless onboarding, validation, evidence and documentation.

Full Conditional Access enforcement remains Phase 3.

---

# 1. Design Principles

The Phase 2 design was frozen before implementation.

## Workforce

Microsoft Authenticator is the workforce baseline.

The goal is not to enroll all 38 synthetic workforce identities. A controlled pilot proves the design and repeatability.

## Privileged identities

Dedicated privileged-class identities use a stronger authentication posture.

Device-bound Passkeys in Microsoft Authenticator were selected as the phishing-resistant passwordless method for the privileged pilot.

Important: these identities are privileged-class accounts by design, but privileged Entra roles / PIM are implemented in a later phase.

## Emergency identities

Emergency identities remain separate from workforce and privileged authentication assumptions.

A Temporary Access Pass was tested as an administrator-assisted recovery/bootstrap path.

This does **not** count as full break-glass validation because TAP issuance itself depends on an already-authorized administrator.

Independent emergency-access authentication and Conditional Access exclusions remain Phase 3 work.

## Guests

Synthetic B2B guest objects were not forced through fake redemption or sign-in flows.

Real guest authentication testing remains deferred until the Enterprise Application / SSO scenario.

## Enforcement boundary

Authentication Methods answers:

> Which methods can an identity register and use?

Conditional Access later answers:

> Under which conditions must a particular authentication requirement be satisfied?

Legacy per-user MFA is not used as the target architecture.

Security Defaults remains enabled during Phase 2 and is not replaced until the Phase 3 Conditional Access transition is designed and validated.

---

# 2. Step 2.2 — Current-State Authentication Inventory

The tenant was inventoried before settings were changed.

## Security Defaults

| Setting | Before-state |
|---|---|
| Security Defaults | Enabled |

## Authentication Methods Policy

| Method | Target | Enabled |
|---|---|---|
| Passkey (FIDO2) | All users | Yes |
| Microsoft Authenticator | All users | Yes |
| SMS | — | No |
| Temporary Access Pass | All users | Yes |
| Hardware OATH tokens (Preview) | — | No |
| Software OATH tokens | All users | Yes |
| Voice call | — | No |
| Email OTP | All users | Yes |
| Certificate-based authentication | — | No |
| Verified ID | — | No |
| QR code | — | No |

## Microsoft Authenticator before-state

| Setting | Value |
|---|---|
| Enabled | Yes |
| Target | All users |
| Registration | Optional |
| Authentication mode | Any |
| Authenticator OTP | Allowed |
| Number matching | Enabled / All users |
| Application name in notifications | Microsoft managed |
| Geographic location in notifications | Microsoft managed |

## Passkey (FIDO2) before-state

| Setting | Value |
|---|---|
| Enabled | Yes |
| Target | All users |
| Profile | Default passkey profile |
| Self-service setup | Allowed |
| Passkey types | Device-bound + Synced |
| Attestation | Not enforced |
| Key restrictions | None |

## Temporary Access Pass before-state

| Setting | Value |
|---|---|
| Enabled | Yes |
| Target | All users |
| Minimum lifetime | 1 hour |
| Maximum lifetime | 8 hours |
| Default lifetime | 1 hour |
| One-time use | No |
| Length | 8 characters |

## Registration Campaign before-state

| Setting | Value |
|---|---|
| State | Microsoft managed |
| Method | Passkey (FIDO2) |
| Include | All users |
| Exclude | None |

## Authentication Settings before-state

| Setting | Value |
|---|---|
| Report suspicious activity | Microsoft managed / All users |
| System-preferred authentication | Microsoft managed / All users |

## User registration before-state

The modeled WEZ identities had no registered authentication methods.

Observed state:

- MFA capable: Not Capable
- Passwordless capable: Not Capable
- SSPR capable: Not Capable
- no registered methods visible

This established a clean before-state for the Phase 2 rollout.

---

# 3. Step 2.3 — Pilot Cohort

A six-identity pilot / validation cohort was selected.

## Standard workforce pilots

| Identity | Department / Persona | Purpose |
|---|---|---|
| Mia Schneider | Researcher | Normal workforce authentication |
| Lisa Werner | Finance Controller | Business-sensitive but non-privileged workforce |
| Emilia Haas | Service Desk Specialist | IT workforce but non-privileged |

## Privileged-class pilots

| Identity | Persona | Purpose |
|---|---|---|
| Sarah Klein (Admin) | Cloud & Identity Administrator — Privileged | Primary TAP → Passkey bootstrap |
| Jonas Becker (Admin) | Head of IT — Privileged | Repeat privileged Passkey validation |

## Emergency pilot

| Identity | Purpose |
|---|---|
| Emergency Admin 01 | Administrator-assisted temporary recovery/bootstrap validation |

## Control identities kept untouched

- Daniel Krüger (Admin)
- Emergency Admin 02

These remain useful as future control / expected-failure identities.

---

# 4. Pilot Scope Groups

Two assigned Security groups were created.

## `GRP-AUTH-PILOT-WORKFORCE`

Members:

- Mia Schneider
- Lisa Werner
- Emilia Haas

Purpose:

> Phase 2 pilot scope for standard workforce authentication and MFA validation.

## `GRP-AUTH-PILOT-PRIVILEGED`

Members:

- Sarah Klein (Admin)
- Jonas Becker (Admin)

Purpose:

> Phase 2 pilot scope for stronger authentication and Passkey/FIDO2 validation of privileged-class identities.

Emergency Admin 01 was intentionally not added to either pilot group.

---

# 5. Step 2.4 — Authentication Policy Implementation

## Microsoft Authenticator

Microsoft Authenticator remained enabled for All users.

Reason:

- Security Defaults remains active.
- method availability is different from pilot enrollment scope.
- Phase 2 validates a controlled cohort without creating an artificial tenant-wide method restriction.

## Passkey (FIDO2)

Passkey remained available for All users.

Reason:

> Availability is not enforcement.

The active Phase 2 Passkey pilot is privileged-class, while later Conditional Access authentication-strength enforcement belongs to Phase 3.

## Registration Campaign

The Passkey registration campaign was narrowed from All users to:

`GRP-AUTH-PILOT-PRIVILEGED`

The campaign remains Microsoft managed.

This prevents the whole synthetic tenant from being actively nudged toward Passkey registration while the privileged pilot is validated.

## Temporary Access Pass

TAP remained available at policy level.

TAP issuance was performed per user only when a bootstrap/recovery requirement existed.

---

# 6. Step 2.5-A — Workforce Microsoft Authenticator Enrollment

Three workforce identities were enrolled:

- Mia Schneider
- Lisa Werner
- Emilia Haas

Observed user flow:

```text
No registered MFA method
        ↓
Security Defaults registration requirement
        ↓
Microsoft Authenticator enrollment
        ↓
QR/device registration
        ↓
Number matching validation
        ↓
Authenticator added
        ↓
Successful account access
```

## Validation result

All three workforce pilots became:

- MFA capable: Capable
- default MFA method: Microsoft Authenticator app (push notification)

Registered methods included:

- Microsoft Authenticator app (push notification)
- Software OATH token

The three-user result demonstrates repeatability across Research, Finance and IT Service Desk personas.

---

# 7. Step 2.5-B — Privileged Passkey / FIDO2 Enrollment

The privileged pilot used a stronger passwordless authentication workflow.

## Sarah Klein (Admin)

Flow:

```text
No usable strong authentication method
        ↓
Administrator issues 1-hour Temporary Access Pass
        ↓
Sarah signs in with TAP
        ↓
Passkey in Microsoft Authenticator registered
        ↓
Fresh browser session
        ↓
Cross-device Passkey sign-in
        ↓
Phone / Authenticator completes FIDO2 authentication
        ↓
Successful passwordless access
```

Validation showed:

- Passkey registered
- Passkey detail: Authenticator — iOS
- system-preferred MFA method: FIDO2
- successful fresh Passkey sign-in
- TAP later moved to Non-usable authentication methods with status `TAP expired`

This proves the TAP lifecycle was time-limited and that access continued through the long-term Passkey credential rather than the bootstrap credential.

## Jonas Becker (Admin)

Jonas repeated the privileged baseline:

- Temporary Access Pass bootstrap
- Passkey in Microsoft Authenticator
- device-bound Passkey
- successful fresh Passkey sign-in
- backend registration validation

Central User Registration Details showed both Sarah and Jonas as MFA capable with Passkey (Microsoft Authenticator) registered.

## Privileged design result

The privileged-class pilot is standardized on:

> Device-bound Passkey in Microsoft Authenticator

A physical FIDO2 security key was not available in the lab.

The lab therefore does not claim hardware-key validation.

A hardware FIDO2 security key remains a valid production alternative for elevated-privilege and independent emergency-access scenarios.

---

# 8. Temporary Access Pass Lesson Learned

Two TAP usage models were considered:

- one-time TAP
- time-limited multi-use TAP

During the lab, one-time TAP proved operationally fragile when the registration workflow was interrupted or another authentication step was requested.

The final lab approach used a short-lived multi-use TAP for controlled bootstrap.

This is documented as an operational lab decision, not as a universal production rule.

Core lifecycle:

```text
Issue
↓
Bootstrap
↓
Register long-term credential
↓
Validate long-term credential
↓
TAP expires
```

TAP is not a normal daily authentication method.

---

# 9. Emergency Admin 01 — Scope and Limitation

Emergency Admin 01 successfully authenticated using an administrator-issued Temporary Access Pass.

What this proves:

> An administrator-assisted temporary recovery/bootstrap path works.

What this does **not** prove:

> Independent break-glass access works when all normal administrators or authentication dependencies are unavailable.

No Authenticator or Passkey was intentionally registered for Emergency Admin 01 during this Phase 2 test.

Reason:

Reusing the same Authenticator device used by privileged admins would recreate the same dependency and would not provide a meaningful independent emergency-access design.

Independent break-glass credentials, Conditional Access exclusion, monitoring and periodic emergency-access validation remain Phase 3 work.

---

# 10. Validation Summary

| Scenario | Expected result | Actual result | Status |
|---|---|---|---|
| Workforce pilot enrollment | Authenticator registration succeeds | 3/3 pilots registered successfully | PASS |
| Workforce number matching | Push validation uses number matching | Number matching observed | PASS |
| Workforce backend validation | Pilots become MFA capable | Mia, Lisa and Emilia show Capable | PASS |
| Sarah TAP bootstrap | Temporary bootstrap sign-in works | Successful | PASS |
| Sarah Passkey registration | Device-bound Passkey registers | Passkey / Authenticator-iOS registered | PASS |
| Sarah fresh passwordless sign-in | Passkey authenticates without TAP/password | Successful | PASS |
| Sarah TAP lifecycle | TAP becomes unusable after lifetime | `TAP expired` observed | PASS |
| Jonas privileged baseline | Privileged Passkey flow is repeatable | Registration and sign-in successful | PASS |
| Emergency temporary recovery | Emergency identity can use admin-issued TAP | Successful | PASS |
| Independent break-glass access | Independent credential / failure-domain test | Not implemented in Phase 2 | DEFERRED |

A separate expired-TAP sign-in rejection was not captured. The tenant did, however, report the expired TAP as a non-usable authentication method. No stronger claim is made.

---

# 11. Evidence Plan

Public evidence should be sanitized before commit.

Recommended evidence set:

| File | What it proves |
|---|---|
| `01-security-defaults-before.png` | Security Defaults was enabled before implementation |
| `02-authentication-methods-before.png` | Authentication-method baseline before rollout |
| `03-authentication-pilot-groups.png` | Staged workforce / privileged pilot scopes |
| `04-workforce-registration-required.png` | Unregistered workforce user was required to set up authentication |
| `05-workforce-number-matching.png` | Authenticator number matching validation |
| `06-workforce-authenticator-added.png` | Successful Authenticator enrollment |
| `07-workforce-registration-validation.png` | Three workforce pilots became MFA capable |
| `08-privileged-passkey-device-bound.png` | Device-bound Passkey exists for privileged identity |
| `09-privileged-passkey-validation.png` | Privileged Passkey registered / system preferred FIDO2 |
| `10-privileged-passwordless-signin.png` | Fresh Passkey authentication succeeded |
| `11-tap-expired.png` | TAP lifecycle ended as designed |
| `12-emergency-tap-recovery.png` | Administrator-assisted temporary recovery/bootstrap succeeded |

Do not publish:

- QR registration secrets
- TAP passcodes
- temporary passwords
- real tenant ID
- real tenant domain
- operator email
- Object IDs / GUIDs
- secrets or tokens

---

# 12. Phase 2 Lessons Learned

1. Authentication-method availability is not the same as authentication enforcement.
2. MFA capable does not mean MFA is required on every sign-in.
3. Registration Campaign is a nudge / enrollment mechanism, not a Conditional Access control.
4. System-preferred authentication can prefer a stronger registered method without replacing Conditional Access enforcement.
5. Microsoft Authenticator push and Passkey in Microsoft Authenticator are different credential models even though both use the same mobile application.
6. TAP is useful as a short-lived bootstrap credential but should not become a standing authentication dependency.
7. A working TAP path for an emergency account is not equivalent to independent break-glass validation.
8. Realistic lab documentation should expose limitations rather than simulate unavailable hardware or claim untested controls.
9. Repeating the privileged Passkey baseline across two accounts is more meaningful than using different technologies only for feature variety.
10. Pilot cohorts create controlled rollout evidence without meaningless bulk enrollment of synthetic identities.

---

# 13. Deferred to Phase 3

Phase 3 will implement the access-policy layer:

- Conditional Access policy design
- staged / report-only rollout
- MFA enforcement
- phishing-resistant authentication strength for privileged access
- break-glass exclusions
- independent emergency-access design
- What If validation
- sign-in log validation
- controlled Conditional Access failure scenario
- rollback path

Daniel Krüger (Admin) remains useful as an unregistered privileged control identity for later policy validation.

Emergency Admin 02 remains untouched as a second emergency identity.

---

# 14. Phase 2 Quality Gate

## Technical

- [x] authentication design frozen
- [x] before-state inventoried
- [x] pilot cohort defined
- [x] pilot scope groups created
- [x] workforce Authenticator enrollment validated
- [x] number matching validated
- [x] privileged Passkey enrollment validated
- [x] privileged fresh passwordless sign-in validated
- [x] TAP lifecycle / expiry observed
- [x] administrator-assisted emergency TAP path validated
- [x] limitations documented
- [x] Conditional Access kept out of Phase 2

## Repository closeout

- [ ] screenshots sanitized
- [ ] selected evidence copied to `images/02-authentication-mfa/`
- [ ] this file committed as `docs/04-authentication-mfa.md`
- [ ] `tests/test-matrix.md` updated
- [ ] README status changed to Phase 2 Complete / Phase 3 Next
- [ ] final GitHub commit / push verified

**PHASE 2 STATUS: TECHNICALLY COMPLETE — REPOSITORY CLOSEOUT PENDING**
