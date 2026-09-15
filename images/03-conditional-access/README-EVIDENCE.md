# Phase 3 Evidence Index

All screenshots in this folder are sanitized for public portfolio use.

| File | Evidence |
|---|---|
| `01-authentication-strengths.png` | Built-in MFA, Passwordless MFA and Phishing-resistant MFA authentication strengths available |
| `02-ca-pilot-group.png` | Assigned Security group used as the Phase 3 Conditional Access validation cohort |
| `03-ca-policies-report-only.png` | Four Conditional Access policies staged in Report-only mode |
| `04-security-defaults-disabled.png` | Security Defaults transition to Conditional Access |
| `05-whatif-privileged-phishing-resistant.png` | What If evaluation of the privileged phishing-resistant MFA policy |
| `06-whatif-legacy-auth-block.png` | Legacy authentication block policy applies to the pilot scope |
| `07-whatif-device-code-block.png` | Device code flow block policy applies to the pilot scope |
| `08-whatif-breakglass-exclusion.png` | Emergency account exclusion prevents restrictive pilot policies from applying |
| `09-mia-report-only-success.png` | Workforce MFA policy evaluated successfully in Report-only mode |
| `10-sarah-report-only-success.png` | Privileged phishing-resistant MFA policy evaluated successfully in Report-only mode |
| `11-ca-policies-enforced.png` | Four validated Conditional Access policies enabled |
| `12-mia-enforced-success.png` | Workforce MFA policy enforced successfully |
| `13-sarah-enforced-success.png` | Privileged phishing-resistant MFA policy enforced successfully |
| `14-daniel-controlled-failure.png` | Privileged identity sign-in blocked before compatible phishing-resistant credential existed |
| `15-daniel-ca-failure.png` | Conditional Access log shows privileged authentication-strength policy failure |
| `16-daniel-tap-bootstrap-redacted.png` | Temporary Access Pass used as a controlled bootstrap credential; passcode redacted |
| `17-daniel-remediation-success.png` | After Passkey remediation, the same privileged Conditional Access policy succeeds |

## Sanitization

The public evidence package removes or crops:

- real tenant/operator account details,
- actual tenant domain where visible,
- TAP passcodes,
- request IDs,
- IP addresses,
- subscription/billing identifiers,
- secrets, tokens and private GUIDs.

Synthetic WEZ persona names are intentionally retained because they are fictional project data.
