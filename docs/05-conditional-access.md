# Conditional Access

> **Phase:** 3 — Conditional Access & Emergency Access  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Implementation, staged validation, enforcement validation and troubleshooting complete  
> **Date:** 2026-09-14

## Objective

Replace the broad Security Defaults baseline with a deliberately scoped Conditional Access design, validate policy impact before enforcement, enforce stronger controls for workforce and privileged identities, block selected legacy authentication paths, and produce evidence through What If and Sign-in Logs.

The Phase 3 implementation follows this engineering sequence:

```text
Design
↓
Premium-licensed pilot scope
↓
Report-only
↓
What If
↓
Real sign-in validation
↓
Security Defaults transition
↓
Enforcement
↓
Controlled failure
↓
Root cause analysis
↓
Remediation
↓
Successful retest
```

---

# 1. Design Principles

## Staged deployment

Conditional Access policies were not enabled immediately.

All four policies were created first in **Report-only** mode. Their targeting and expected behavior were validated with:

- What If,
- interactive sign-in logs,
- report-only policy results.

Only after those checks succeeded was Security Defaults disabled and the policies moved to **On**.

## Licensing-aware pilot

The lab uses a deliberately limited premium-licensed validation cohort rather than applying premium controls to every synthetic identity.

This keeps the technical test compliant with the lab licensing model and demonstrates staged rollout rather than arbitrary tenant-wide enforcement.

## Separate policy purposes

Each policy has one primary security purpose.

The design avoids combining unrelated requirements into one large policy because separate policies make:

- testing easier,
- failure analysis clearer,
- rollback safer,
- future scope changes more controlled.

## Authentication strength instead of generic MFA only

The workforce baseline requires the built-in **Multifactor authentication** strength.

Privileged identities require the built-in **Phishing-resistant MFA** strength.

This preserves the Phase 2 distinction:

```text
Available
↓
Registered
↓
Preferred
↓
Required
```

Phase 3 implements the final **Required** layer.

---

# 2. Pilot Scope

A dedicated assigned Security group was created:

`GRP-CA-PILOT`

The validation cohort contains eight synthetic identities:

- three workforce pilot identities,
- three privileged administrator identities,
- two emergency identities.

Existing specialized groups remain the authoritative scope for workforce, privileged and emergency behavior:

- `GRP-AUTH-PILOT-WORKFORCE`
- `GRP-CA-ADMINS-STRICT`
- `GRP-CA-BREAKGLASS-EXCLUDED`

The general pilot group is used where a shared baseline control is required.

---

# 3. Conditional Access Policy Set

## 3.1 `CA-WORKFORCE-REQUIRE-MFA`

**Purpose:** require MFA for the workforce pilot.

| Setting | Value |
|---|---|
| Include | `GRP-AUTH-PILOT-WORKFORCE` |
| Target resources | All resources |
| Conditions | None |
| Grant | Grant access |
| Requirement | Authentication strength: Multifactor authentication |
| State | On |

Expected normal sign-in result:

> Workforce pilot users satisfy the MFA authentication-strength requirement.

---

## 3.2 `CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA`

**Purpose:** require a stronger phishing-resistant method for privileged-class identities.

| Setting | Value |
|---|---|
| Include | `GRP-CA-ADMINS-STRICT` |
| Target resources | All resources |
| Conditions | None |
| Grant | Grant access |
| Requirement | Authentication strength: Phishing-resistant MFA |
| State | On |

Sarah Klein (Admin) and Jonas Becker (Admin) already had device-bound Passkeys from Phase 2.

Daniel Krüger (Admin) was deliberately left without a compatible phishing-resistant credential until the failure / remediation test.

---

## 3.3 `CA-PILOT-BLOCK-LEGACY-AUTH`

**Purpose:** block selected legacy authentication clients for the pilot cohort.

| Setting | Value |
|---|---|
| Include | `GRP-CA-PILOT` |
| Exclude | `GRP-CA-BREAKGLASS-EXCLUDED` |
| Target resources | All resources |
| Client apps | Exchange ActiveSync clients + Other clients |
| Grant | Block access |
| State | On |

Emergency identities are explicitly excluded.

---

## 3.4 `CA-PILOT-BLOCK-DEVICE-CODE`

**Purpose:** block Device Code Flow for the pilot cohort.

| Setting | Value |
|---|---|
| Include | `GRP-CA-PILOT` |
| Exclude | `GRP-CA-BREAKGLASS-EXCLUDED` |
| Target resources | All resources |
| Authentication flow | Device code flow |
| Grant | Block access |
| State | On |

Emergency identities are explicitly excluded.

---

# 4. Report-only Validation

Before enforcement, all four policies were kept in **Report-only** mode.

## What If — privileged policy

A privileged administrator identity was evaluated against Azure Management.

Result:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → applies.

## What If — legacy authentication

A workforce pilot identity using an `Other clients` scenario caused:

- workforce MFA policy to apply,
- legacy authentication block policy to apply.

This also demonstrated that multiple Conditional Access policies can evaluate the same sign-in.

## What If — device code flow

A workforce pilot identity using Device Code Flow caused:

- workforce MFA policy to apply,
- legacy authentication block policy to apply where the selected client condition matched,
- device code block policy to apply.

## What If — emergency exclusion

Emergency Admin 01 was evaluated under a scenario that would normally match the restrictive pilot baseline.

Result:

> No restrictive pilot policy applied.

This validated the `GRP-CA-BREAKGLASS-EXCLUDED` exclusion logic.

---

# 5. Real Report-only Sign-in Validation

What If validates policy applicability.

Real sign-in logs validate actual sign-in evaluation.

## Workforce

Mia Schneider performed a fresh Azure Portal sign-in.

Report-only result:

`CA-WORKFORCE-REQUIRE-MFA` → **Report-only: Success**

The privileged and block policies were correctly reported as not applied for the normal browser sign-in.

## Privileged

Sarah Klein (Admin) performed a fresh Passkey sign-in.

Report-only result:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Report-only: Success**

This confirmed that the Phase 2 device-bound Passkey satisfied the privileged authentication-strength design before enforcement.

---

# 6. Security Defaults Transition

Security Defaults remained enabled during design and report-only validation.

After report-only evidence was collected:

1. Security Defaults was disabled.
2. The reason selected was that the organization is moving to Conditional Access.
3. The four validated Conditional Access policies were changed from Report-only to **On**.

This was a deliberate transition, not an immediate replacement without testing.

Important lab limitation:

The Conditional Access design is intentionally scoped to the licensed validation cohort. This is a portfolio lab, not a claim that every synthetic identity has a production-grade tenant-wide policy baseline.

---

# 7. Enforcement Validation

## Workforce success path

Mia Schneider performed a fresh Azure Portal sign-in after enforcement.

Conditional Access result:

`CA-WORKFORCE-REQUIRE-MFA` → **Success**

## Privileged success path

Sarah Klein (Admin) performed a fresh Passkey sign-in after enforcement.

Conditional Access result:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Success**

The same policy correctly did not apply to workforce identities.

---

# 8. Controlled Failure — Privileged Authentication Strength

Daniel Krüger (Admin) was intentionally used as the negative control.

At the time of the test:

- Daniel was a member of `GRP-CA-ADMINS-STRICT`,
- the privileged policy was enabled,
- Daniel did not have a compatible phishing-resistant credential.

## Symptom

The interactive sign-in could not be completed.

The sign-in log showed:

- Status: Interrupted
- Sign-in error code: `50072`
- Conditional Access:
  `CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Failure**

## Root cause

The policy targeting was correct.

The account belonged to the privileged scope, but the identity could not satisfy the required **Phishing-resistant MFA** authentication strength.

This was an authentication-readiness problem, not a policy-scope problem.

## Remediation

The policy requirement was not weakened.

Instead:

```text
Administrator issues short-lived TAP
↓
Daniel signs in with TAP
↓
Daniel registers a device-bound Passkey
↓
Fresh privileged sign-in
↓
Phishing-resistant authentication strength satisfied
```

## Retest

After Passkey registration, Daniel performed a new sign-in.

Result:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Success**

This completed the full engineering chain:

> Failure → evidence → root cause → remediation → successful retest.

---

# 9. Rollback / Recovery Runbook

Rollback belongs in Phase 3 because Conditional Access can create tenant-wide lockout risk when targeting, exclusions or authentication requirements are incorrect.

The project therefore documents rollback before considering the phase complete.

## Trigger conditions

Use rollback when one of the following occurs:

- expected users are unexpectedly blocked,
- break-glass exclusion does not behave as designed,
- a policy scope is broader than intended,
- the required authentication method is not operationally ready,
- multiple policies combine into an unintended effective control.

## Level 1 — policy state rollback

For the affected policy:

```text
On
↓
Report-only
```

or, when immediate removal of evaluation is required:

```text
On
↓
Off
```

This is preferred over deleting the policy because the configuration and evidence remain available for diagnosis.

## Level 2 — scope rollback

If targeting is the problem:

1. restore the previous group membership / exclusion state,
2. run What If again,
3. confirm the expected policy set,
4. repeat a real sign-in,
5. inspect Conditional Access details.

## Level 3 — authentication-readiness remediation

If the policy is correct but the user lacks a compatible credential, do **not** immediately weaken the policy.

The Daniel test demonstrates the preferred recovery model:

```text
Keep security requirement
↓
Bootstrap compatible strong credential
↓
Retest
```

## Level 4 — emergency transition fallback

If the Conditional Access transition itself causes broad access problems:

1. use an unaffected operator or validated emergency path,
2. move the problematic custom Conditional Access policies to Report-only / Off,
3. validate administrative access,
4. correct policy scope or credential readiness,
5. if a full return to Security Defaults is required, remove or disable conflicting Conditional Access configuration as required by the portal before re-enabling Security Defaults.

## Validation status

The rollback **procedure is documented**.

A destructive tenant-wide rollback was not executed because the staged rollout, What If tests, report-only sign-ins and enforced success tests all validated cleanly.

The credential-remediation branch of the recovery model was executed successfully with Daniel.

---

# 10. Evidence

Public sanitized evidence is stored in:

`images/03-conditional-access/`

Key evidence includes:

- authentication strengths,
- pilot group design,
- report-only policy set,
- Security Defaults transition,
- What If validation,
- break-glass exclusion,
- report-only sign-in success,
- enabled policy set,
- enforced workforce and privileged success,
- Daniel controlled failure,
- Conditional Access failure evidence,
- TAP bootstrap with passcode removed,
- successful remediation retest.

See:

[`images/03-conditional-access/README-EVIDENCE.md`](../images/03-conditional-access/README-EVIDENCE.md)

---

# 11. Lessons Learned

1. Conditional Access must be treated as a rollout process, not a collection of portal checkboxes.
2. Report-only plus What If significantly reduces lockout risk before enforcement.
3. Group membership is part of the policy design; a correct policy with an empty or wrong scope is still functionally wrong.
4. Multiple Conditional Access policies can evaluate the same sign-in.
5. Authentication-method registration and authentication-strength enforcement are separate layers.
6. A stronger privileged requirement is only useful when privileged identities are operationally ready to satisfy it.
7. A failed privileged sign-in should be fixed by restoring credential readiness where possible, not automatically by weakening the security policy.
8. Emergency accounts need explicit exclusion from restrictive policies.
9. Exclusion validation is not the same as proving a fully independent break-glass credential.
10. Sign-in Logs are the authoritative troubleshooting evidence after policy enforcement.

---

# 12. Phase 3 Quality Gate

## Technical

- [x] Conditional Access design frozen before enforcement
- [x] premium-licensed pilot cohort defined
- [x] workforce MFA policy created
- [x] privileged phishing-resistant MFA policy created
- [x] legacy authentication block policy created
- [x] device code flow block policy created
- [x] emergency exclusion configured
- [x] What If validation completed
- [x] Report-only real sign-in validation completed
- [x] Security Defaults transition completed
- [x] policies enforced
- [x] workforce enforcement success validated
- [x] privileged enforcement success validated
- [x] controlled privileged failure captured
- [x] Sign-in Log RCA completed
- [x] remediation performed without weakening the policy
- [x] successful retest captured
- [x] rollback / recovery procedure documented

## Limitation

- [ ] fully independent break-glass credential / failure-domain validation

The Conditional Access exclusion is validated, but a separate independent emergency credential is not claimed.

**PHASE 3 STATUS: COMPLETE WITH DOCUMENTED EMERGENCY-CREDENTIAL LIMITATION**
