# Emergency Access

> **Phase:** 3 — Conditional Access & Emergency Access  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Conditional Access exclusion validated; independent emergency credential remains a documented limitation  
> **Date:** 2026-09-14

## Objective

Define emergency-access identities as a separate identity class and ensure normal Conditional Access restrictions do not accidentally remove the tenant's recovery path.

The Phase 3 objective is specifically to validate:

- emergency identities remain separate from workforce and privileged scopes,
- restrictive pilot policies explicitly exclude emergency identities,
- the exclusion behaves as expected in What If,
- the project does not falsely claim independent break-glass validation when the credential failure domain has not been proven.

---

# 1. Emergency Identity Model

Two emergency identities were created in Phase 1:

- Emergency Admin 01
- Emergency Admin 02

They are members of:

`GRP-CA-BREAKGLASS-EXCLUDED`

They are not normal workforce identities and are not members of the privileged admin scope used for day-to-day administrator controls.

---

# 2. Conditional Access Exclusion

The following restrictive pilot policies explicitly exclude:

`GRP-CA-BREAKGLASS-EXCLUDED`

Policies:

- `CA-PILOT-BLOCK-LEGACY-AUTH`
- `CA-PILOT-BLOCK-DEVICE-CODE`

The purpose of the exclusion is to prevent a baseline policy from unintentionally removing the emergency-access path.

---

# 3. What If Validation

Emergency Admin 01 was evaluated with a sign-in scenario that would otherwise match the pilot restriction.

Result:

> No restrictive pilot policy applied.

This validates the Conditional Access exclusion logic.

Evidence:

`images/03-conditional-access/08-whatif-breakglass-exclusion.png`

---

# 4. Relationship to Phase 2

Phase 2 validated that Emergency Admin 01 could use an administrator-issued Temporary Access Pass.

That test proved:

> administrator-assisted temporary bootstrap / recovery.

It did **not** prove:

> independent break-glass access when normal administrators or the normal authentication failure domain are unavailable.

The distinction remains intentional.

---

# 5. Current Limitation

A fully independent emergency credential was not implemented in this lab phase.

No claim is made that:

- a dedicated hardware FIDO2 key was validated,
- the emergency account can recover the tenant without any dependency on another administrator,
- the complete emergency process was tested against a simulated total administrator lockout.

The Phase 3 result is therefore:

```text
Emergency identities isolated
        ↓
CA exclusion implemented
        ↓
What If exclusion validated
        ↓
Independent emergency credential
        ↓
NOT YET VALIDATED
```

---

# 6. Emergency Access Operating Rules

For the WEZ lab design, emergency identities should follow these principles:

1. Remain separate from normal workforce use.
2. Remain separate from daily privileged administrator accounts.
3. Be excluded only from policies where exclusion is explicitly required for recovery.
4. Never be used for normal administration.
5. Be monitored for unexpected sign-in activity.
6. Be validated periodically.
7. Use an authentication dependency that is meaningfully independent from normal administrator authentication where possible.

---

# 7. Rollback Role

Emergency access is part of the Conditional Access rollback model.

If a new Conditional Access policy causes unexpected lockout:

```text
Validated emergency / unaffected operator access
↓
Policy moved to Report-only or Off
↓
Scope / exclusion corrected
↓
What If
↓
Real sign-in validation
↓
Re-enable only after evidence is clean
```

The emergency exclusion is therefore not a decorative exception. It is a control that protects the recovery path during policy changes.

---

# 8. Evidence

Primary public evidence:

- `08-whatif-breakglass-exclusion.png`

Related Phase 2 evidence:

- administrator-assisted TAP recovery/bootstrap test.

---

# 9. Status

## Completed

- [x] two emergency identities exist
- [x] dedicated exclusion group exists
- [x] emergency identities remain outside normal workforce policy scope
- [x] restrictive Phase 3 policies include explicit emergency exclusion
- [x] What If confirms the exclusion behaves as intended
- [x] rollback role documented

## Deferred / limitation

- [ ] independent emergency credential validated
- [ ] full lockout simulation performed
- [ ] periodic emergency-access operational test established

**EMERGENCY ACCESS STATUS: CA EXCLUSION VALIDATED — FULL INDEPENDENT BREAK-GLASS AUTHENTICATION NOT CLAIMED**
