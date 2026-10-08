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

---

# Failure 3 — Enterprise application assignment enforcement

**Status:** Executed and resolved  
**Phase:** 6 — Enterprise App / SSO

## Requirement

Only identities with justified access to the WEZ Research Collaboration Portal should be able to obtain effective application access.

The enterprise application must keep:

`Assignment required = Yes`

and normal access must remain group-based through:

`GRP-APP-RESEARCH-PORTAL-USERS`

## Controlled Setup

Lisa Werner was the positive-control user and a direct member of the assigned application-access group.

Mia Schneider was intentionally left outside the group.

The same SAML Service Provider endpoint was used for both identities.

## Positive Control

Lisa completed the SP-initiated SAML flow successfully.

Entra Enterprise Application Sign-in Logs later recorded:

```text
Application: WEZ Research Collaboration Portal
Status: Success
Conditional Access: Success
```

This established that the SAML integration itself was functional before the negative test.

## Symptom

Mia then attempted the same SAML sign-in while unassigned.

The browser returned:

`AADSTS50105`

The error stated that the signed-in user was blocked because she was neither directly assigned nor a direct member of a group with application access.

Evidence:

- `images/06-enterprise-app-sso/07-mia-unassigned-aadsts50105.png`
- `images/06-enterprise-app-sso/08-mia-50105-signin-log.png`

## Investigation

The Sign-in Log showed:

```text
Application: WEZ Research Collaboration Portal
Status: Failure
Sign-in error code: 50105
Conditional Access: Success
```

The investigation separated the control layers:

1. the user identity was recognized,
2. Conditional Access was not the blocking control,
3. the enterprise application existed and SAML configuration had already worked for Lisa,
4. Mia did not have effective application assignment.

## Root Cause

> Mia was not a direct member of `GRP-APP-RESEARCH-PORTAL-USERS` and had no direct application assignment.

Classification:

**Enterprise application authorization / assignment mismatch**

The failure was not caused by:

- invalid credentials,
- Conditional Access,
- the SAML signing certificate,
- NameID configuration,
- Azure RBAC.

## Remediation

The application control was not weakened.

The following settings remained unchanged:

- `Assignment required = Yes`
- group-based application assignment
- SAML configuration

Mia was temporarily added to:

`GRP-APP-RESEARCH-PORTAL-USERS`

After membership propagation, Entra recorded successful application sign-in.

## Observed Service Provider Mapping Issue

The first post-remediation Service Provider attempt still returned a generic Toolkit application error even though Entra showed **Success**.

The SAML Toolkit sample application requires a local Toolkit test user whose email matches the Entra NameID.

Lisa already had a matching Toolkit account; Mia did not.

After the matching Mia Toolkit test account was created, the SAML flow completed successfully.

This demonstrated an important distinction:

```text
Identity Provider success
≠
Service Provider application success
```

The local-user dependency is specific to the sample Toolkit and is not presented as an Entra federation defect.

## Validation

The final retest succeeded:

```text
Mia
↓
Entra authentication
↓
Conditional Access success
↓
effective group-based app assignment
↓
SAML assertion
↓
matching Toolkit user
↓
application session success
```

Evidence:

- `images/06-enterprise-app-sso/09-mia-successful-retest.png`

Result:

**PASS**

## Rollback / Recovery

Mia's group membership was required only for the controlled remediation/retest.

After validation, Mia was removed from:

`GRP-APP-RESEARCH-PORTAL-USERS`

The final least-privilege application-access state therefore returned to the original authorized user scope.

Rollback model:

```text
Unexpected application access
↓
Check authentication / Conditional Access
↓
Check enterprise-app assignment requirement
↓
Check direct group membership
↓
Correct justified membership
↓
Retest
↓
Remove temporary test access
↓
Restore least privilege
```

## Lesson Learned

Authentication success does not imply application authorization.

For federated enterprise applications, troubleshoot the chain in order:

```text
Identity
→ Authentication
→ Conditional Access
→ Application assignment
→ SAML federation
→ Service Provider application state
```

The correct fix for an assignment failure was to align membership with the business requirement, not to disable assignment enforcement.
