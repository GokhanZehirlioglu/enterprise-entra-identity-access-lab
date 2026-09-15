# Failure Scenarios

## Failure 1 — Privileged authentication-strength mismatch

**Status:** Executed and resolved  
**Phase:** 3 — Conditional Access & Emergency Access

### Requirement

Privileged identities in `GRP-CA-ADMINS-STRICT` must satisfy the built-in **Phishing-resistant MFA** authentication strength.

The control must not be weakened simply because one privileged identity is not ready.

### Controlled setup

Daniel Krüger (Admin) was intentionally retained as the privileged negative-control identity.

At the time of enforcement:

- Daniel was in the strict privileged Conditional Access scope.
- `CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` was enabled.
- Daniel did not yet have a compatible phishing-resistant credential.

### Symptom

Daniel could not complete interactive sign-in.

User-facing result:

> Sign-in could not be completed.

Sign-in Logs showed:

- Status: `Interrupted`
- Sign-in error code: `50072`

Conditional Access showed:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Failure**

### Evidence

- `images/03-conditional-access/14-daniel-controlled-failure.png`
- `images/03-conditional-access/15-daniel-ca-failure.png`

### Hypothesis

Either:

1. the policy target was incorrect, or
2. the policy was correct but Daniel lacked a method that could satisfy the required authentication strength.

### Root Cause

Policy targeting was correct.

The identity was a member of the strict privileged group, but did not have a registered phishing-resistant credential.

Therefore the identity could not satisfy:

> Authentication strength: Phishing-resistant MFA.

### Fix

The Conditional Access policy was **not weakened**.

A short-lived Temporary Access Pass was issued to bootstrap a compatible long-term credential.

Flow:

```text
Short-lived TAP
↓
Security Info
↓
Device-bound Passkey registration
↓
Fresh sign-in
↓
Phishing-resistant MFA satisfied
```

### Validation

After remediation, Daniel completed a fresh sign-in.

Conditional Access result:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Success**

Evidence:

- `images/03-conditional-access/16-daniel-tap-bootstrap-redacted.png`
- `images/03-conditional-access/17-daniel-remediation-success.png`

### Rollback

If the policy had blocked a legitimate administrator and credential remediation was not immediately possible:

1. use an unaffected operator or validated emergency path,
2. move the affected policy from `On` to `Report-only` or `Off`,
3. verify administrative access,
4. correct credential readiness / scope,
5. validate with What If,
6. perform a real sign-in,
7. return the policy to `On` only after evidence is clean.

The preferred recovery path is credential remediation rather than weakening the privileged authentication requirement.

### Lesson Learned

A correct security control can still cause an operational failure when identity readiness is incomplete.

For privileged access, enforcement readiness must include both:

- correct policy targeting, and
- a compatible registered authentication method.

The failure also demonstrated why **Report-only, What If, emergency exclusions and a rollback plan must exist before broad enforcement**.

---

## Observed validation issue — Pilot group membership

**Status:** Resolved during validation

During early What If testing, the workforce policy did not apply even though the policy configuration itself was correct.

The issue was not the authentication-strength setting.

The workforce pilot group's effective membership was incomplete.

After membership was corrected, the same What If scenario caused the expected workforce policy to apply.

### Lesson

Conditional Access scope objects are part of the security control.

A policy with correct logic but incorrect group membership is still functionally incorrect.

This issue was resolved before enforcement and was not used as the main controlled failure scenario.
