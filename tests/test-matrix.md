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
| T10 | Independent break-glass validation | Emergency access remains available without dependency on normal admin authentication | Not tested; independent hardware credential not available in Phase 2 | DEFERRED |
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

## Notes

- Phase 2 validates authentication methods and registration. Full Conditional Access enforcement belongs to Phase 3.
- `MFA capable` means the identity has an MFA-capable registered method. It does not by itself mean MFA is required on every sign-in.
- The Emergency Admin 01 TAP test is an administrator-assisted recovery/bootstrap test, not full independent break-glass validation.
- A physical FIDO2 security key was not available in the lab; no hardware-key validation is claimed.
