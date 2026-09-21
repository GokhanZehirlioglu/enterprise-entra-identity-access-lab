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

Sign-in Logs showed:

- Status: `Interrupted`
- Sign-in error code: `50072`

Conditional Access showed:

`CA-ADMINS-REQUIRE-PHISHING-RESISTANT-MFA` → **Failure**

### Root Cause

Policy targeting was correct.

Daniel did not have a registered phishing-resistant credential and therefore could not satisfy the authentication strength.

### Fix

The Conditional Access policy was not weakened.

A short-lived Temporary Access Pass was used to bootstrap a device-bound Passkey.

### Validation

A fresh sign-in then satisfied the privileged Conditional Access policy.

### Lesson Learned

A correct security control can still create an operational failure when identity readiness is incomplete.

---

## Observed validation issue — Pilot group membership

**Status:** Resolved during Phase 3 validation

During early What If testing, the workforce policy did not apply even though the policy configuration itself was correct.

The workforce pilot group's effective membership was incomplete.

After membership was corrected, the expected policy applied.

### Lesson

A security rule with correct logic but incorrect scope membership is still functionally incorrect.

---

# Failure 2 — Azure RBAC group-membership mismatch

**Status:** Executed and resolved  
**Phase:** 4 — RBAC & Least Privilege

## Requirement

Emilia Haas represents an operator persona.

The identity must be able to perform a safe management-plane change on the Phase 4 Azure lab resource without receiving Owner or subscription-wide privilege.

## Controlled setup

The Azure resource model was:

```text
Azure subscription
└── rg-wez-rbac-lab
    └── storageaccountwez01
```

The RBAC groups were:

```text
GRP-RBAC-WEZ-LAB-READERS
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
```

Reader and Contributor were assigned at Resource Group scope.

For the controlled failure, Emilia was intentionally placed in the Reader group.

Effective access:

```text
Emilia Haas
↓
GRP-RBAC-WEZ-LAB-READERS
↓
Reader
↓
rg-wez-rbac-lab
↓
storageaccountwez01
```

## Symptom

Emilia could inspect the Storage Account but could not assign a resource tag.

The management-plane write was denied.

Evidence:

- `images/04-rbac-least-privilege/05-reader-initial-membership-controlled-failure.png`
- `images/04-rbac-least-privilege/10-emilia-reader-write-denied.png`

## Hypothesis

Possible causes included:

1. incorrect role assignment,
2. incorrect scope,
3. missing group membership,
4. unexpected authentication / Conditional Access interference,
5. a resource-side problem.

## Investigation

The investigation checked:

- the resource existed and was healthy,
- the role assignment existed,
- the role scope was the intended Resource Group,
- Reader/Contributor groups were present,
- the identity's effective group membership,
- the failed action was a management-plane write.

## Root Cause

The RBAC configuration was functioning as designed.

The failure was caused by a mismatch between the business requirement and the identity's group membership:

> Emilia needed Contributor-level management access but was a member of the Reader group.

This was an **authorization / RBAC membership mismatch**.

It was not:

- an authentication failure,
- a Conditional Access failure,
- a Storage Account deployment failure.

## Remediation

The security requirement was not weakened and no Owner role was added.

Instead:

1. Emilia was removed from `GRP-RBAC-WEZ-LAB-READERS`.
2. Emilia was added to `GRP-RBAC-WEZ-LAB-CONTRIBUTORS`.
3. The effective access path was allowed to update.
4. The same class of tag write was repeated.

Evidence:

- `images/04-rbac-least-privilege/11-emilia-reader-membership-removed.png`
- `images/04-rbac-least-privilege/12-emilia-contributor-membership-added.png`

## Validation

The repeated tag assignment succeeded.

Evidence:

- `images/04-rbac-least-privilege/13-emilia-contributor-write-success.png`
- `images/04-rbac-least-privilege/17-contributor-write-success-toast.png`

Result:

**PASS**

## Rollback / Recovery

A destructive rollback was not executed.

The documented model is:

```text
Unexpected RBAC outcome
↓
Inspect role + scope + membership
↓
Remove incorrect assignment
or
restore correct membership
↓
Re-evaluate access
↓
Retest
```

The membership-remediation branch was executed as part of this failure scenario.

## Lesson Learned

A denied Azure operation is not automatically evidence that the user needs a broader role.

Effective access should be analyzed as:

```text
Identity
+ Group membership
+ Role
+ Scope
= Effective authorization
```

The correct fix was to align membership with the business requirement, not to grant Owner.
