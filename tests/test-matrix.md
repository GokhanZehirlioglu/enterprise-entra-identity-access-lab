# Test Matrix

| ID | Scenario | Expected Result | Actual | Status |
|---|---|---|---|---|
| T01 | Standard researcher authentication registration | Microsoft Authenticator registration succeeds and the user becomes MFA capable | Mia Schneider registered Authenticator and became MFA capable | PASS |
| T02 | Workforce authentication repeatability | Authenticator baseline works across multiple non-privileged personas | Mia Schneider, Lisa Werner and Emilia Haas all became MFA capable | PASS |
| T03 | Authenticator number matching | Push validation requires number matching | Number matching challenge observed during workforce enrollment | PASS |
| T04 | Privileged TAP bootstrap | Privileged identity can use a short-lived TAP to bootstrap strong authentication | Sarah Klein (Admin) signed in with TAP and registered a Passkey | PASS |
| T05 | Privileged device-bound Passkey | Privileged identity can register a device-bound Passkey in Microsoft Authenticator | Sarah Klein (Admin) registered Passkey / Authenticator-iOS | PASS |
| T06 | Privileged passwordless sign-in | Fresh sign-in succeeds using Passkey without relying on the TAP | Sarah Klein (Admin) successfully signed in with Passkey | PASS |
| T07 | Privileged Passkey repeatability | Second privileged pilot can repeat the Passkey baseline | Jonas Becker (Admin) registered a device-bound Passkey and signed in successfully | PASS |
| T08 | TAP expiry lifecycle | Short-lived TAP becomes unusable after its configured lifetime | Sarah TAP appears under Non-usable authentication methods as `TAP expired` | PASS |
| T09 | Emergency administrator-assisted recovery | Emergency identity can authenticate with an administrator-issued temporary credential | Emergency Admin 01 successfully signed in with TAP | PASS |
| T10 | Independent break-glass validation | Emergency access remains available without dependency on normal admin authentication | Phase 3 validated CA exclusion, but an independent emergency credential / failure-domain test is still not implemented | DEFERRED |
| T11 | Expired TAP sign-in rejection | An expired TAP is rejected during a fresh sign-in attempt | Separate rejected sign-in attempt not captured; expiry state only was validated | NOT RUN |
| T12 | User outside application group | Access denied | TBD | NOT RUN |
| T13 | Finance user in approved group | Access allowed | TBD | NOT RUN |
| T14 | Standard user attempts privileged operation | Denied | TBD | NOT RUN |
| T15 | Privileged admin activates eligible role | Time-limited privilege where licensing permits | TBD | NOT RUN |
| T16 | Incorrect CA group scope | Failure visible in sign-in evidence | TBD | NOT RUN |
| T17 | Guest accesses assigned research app | Allowed under guest controls | TBD | NOT RUN |
| T18 | Guest attempts unassigned app | Denied | TBD | NOT RUN |
| T19 | Azure Reader attempts write | Denied | TBD | NOT RUN |
| T20 | Azure Contributor performs permitted change | Allowed | TBD | NOT RUN |
| T21 | Enterprise App / SSO misconfiguration | Failure captured and RCA completed | TBD | NOT RUN |
| T22 | Graph script lacks permission | Predictable failure and permission RCA | TBD | NOT RUN |
| T23 | Mover changes department | Old access removed, new access applied | TBD | NOT RUN |
| T24 | Leaver offboarding | Membership/access/session controls removed | TBD | NOT RUN |
| T25 | Workforce CA What If | Workforce MFA policy applies to the workforce pilot | `CA-WORKFORCE-REQUIRE-MFA` applied after pilot group membership was verified | PASS |
| T26 | Privileged CA What If | Privileged phishing-resistant policy applies to privileged admin scope | `CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` applied | PASS |
| T27 | Legacy authentication What If | Legacy authentication block policy applies to matching pilot scenario | `CA-PILOT-BLOCK-LEGACY-AUTH` applied | PASS |
| T28 | Device Code Flow What If | Device code block policy applies to matching pilot scenario | `CA-PILOT-BLOCK-DEVICE-CODE` applied | PASS |
| T29 | Emergency exclusion What If | Restrictive pilot policy does not apply to emergency identity | Emergency Admin 01 produced no matching restrictive pilot policy | PASS |
| T30 | Workforce report-only real sign-in | Workforce MFA policy evaluates successfully without enforcement | Mia Schneider → `Report-only: Success` | PASS |
| T31 | Privileged report-only real sign-in | Privileged Passkey satisfies phishing-resistant strength in report-only mode | Sarah Klein (Admin) → `Report-only: Success` | PASS |
| T32 | Workforce enforced sign-in | Workforce MFA policy succeeds after policy is enabled | Mia Schneider → `Success` | PASS |
| T33 | Privileged enforced sign-in | Privileged Passkey satisfies enforced phishing-resistant policy | Sarah Klein (Admin) → `Success` | PASS |
| T34 | Privileged user without phishing-resistant credential | Enforced privileged policy prevents completion of sign-in | Daniel Krüger (Admin) sign-in interrupted; CA policy result `Failure`, error `50072` | PASS |
| T35 | Privileged remediation | Strong credential is added without weakening the policy | Daniel used short-lived TAP, registered Passkey, then CA policy returned `Success` | PASS |
| T36 | Conditional Access rollback runbook | Recovery path is documented before phase closeout | Policy-state, scope, credential-readiness and Security Defaults fallback paths documented; full tenant-wide rollback not executed | DOCUMENTED |

## Notes

- Phase 2 validated authentication methods and registration.
- Phase 3 validated Conditional Access staging, report-only evaluation, enforcement, troubleshooting and remediation.
- `MFA capable` means the identity has an MFA-capable registered method. It does not by itself mean MFA is required on every sign-in.
- Emergency Admin 01 TAP testing proves administrator-assisted recovery/bootstrap, not fully independent break-glass authentication.
- Phase 3 validated the emergency Conditional Access exclusion, but an independent emergency credential remains intentionally unclaimed.
- A physical FIDO2 security key was not available in the lab; no hardware-key validation is claimed.
