# RBAC & Least Privilege

> **Phase:** 4 — RBAC & Least Privilege  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Complete  
> **Scope:** Azure resource authorization using group-based Azure RBAC

## Objective

Turn the identity model created in earlier phases into real Azure resource authorization.

Phase 4 answers a different question from authentication and Conditional Access:

> After an identity successfully signs in, **what is it allowed to do on Azure resources, and at what scope?**

The implementation intentionally focuses on group-based Azure RBAC, least privilege, scope inheritance, positive/negative validation and a controlled authorization failure.

---

## 1. Requirement

The WEZ lab needs three distinct authorization outcomes:

1. A read-only persona can inspect the Phase 4 Azure resource but cannot modify it.
2. An operator persona can perform a safe management-plane change.
3. A control identity without an RBAC assignment cannot access the Phase 4 resource.

Direct user assignments are avoided. Access is granted through security groups.

---

## 2. Resource Access Model

The Phase 4 Azure hierarchy is:

```text
Azure subscription
└── rg-wez-rbac-lab
    └── storageaccountwez01
```

The Storage Account uses a minimal lab configuration:

- Standard performance
- Locally redundant storage (LRS)
- StorageV2
- Germany West Central
- no additional workload deployed for the RBAC test

Lab tags:

```text
Environment = Lab
Project = Enterprise-Entra-IAM-Lab
Phase = 4
Purpose = RBAC-Least-Privilege
```

The resource exists to test authorization, not Storage workload design.

---

## 3. Group-Based RBAC Model

Two assigned Microsoft Entra security groups were created:

```text
GRP-RBAC-WEZ-LAB-READERS
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
```

Final membership:

```text
GRP-RBAC-WEZ-LAB-READERS
└── Mia Schneider

GRP-RBAC-WEZ-LAB-CONTRIBUTORS
└── Emilia Haas

Lisa Werner
└── no Phase 4 RBAC group
```

Azure RBAC assignments were applied to the **resource-group scope**:

| Group | Azure RBAC role | Scope | Assignment |
|---|---|---|---|
| `GRP-RBAC-WEZ-LAB-READERS` | Reader | `rg-wez-rbac-lab` | Active / Permanent |
| `GRP-RBAC-WEZ-LAB-CONTRIBUTORS` | Contributor | `rg-wez-rbac-lab` | Active / Permanent |

No direct user RBAC assignment was required.

---

## 4. Why Resource-Group Scope?

The role assignments were made at:

```text
rg-wez-rbac-lab
```

rather than directly on the Storage Account.

This demonstrates Azure RBAC scope inheritance:

```text
Resource Group assignment
↓
Storage Account inherits access
```

The Storage Account IAM view confirms that both Reader and Contributor assignments are inherited from the Resource Group.

The scope is deliberately narrower than the full subscription and broader than a single-resource-only assignment.

---

## 5. Why Reader and Contributor?

### Reader

Reader is used for the read-only persona.

Expected behavior:

```text
Inspect resource configuration   ✅
Modify resource configuration    ❌
Delete resource                  ❌
```

### Contributor

Contributor is used for the operator persona.

Expected behavior:

```text
Inspect resource configuration   ✅
Modify resource configuration    ✅
Manage normal Azure resources    ✅
Assign Azure RBAC roles          ❌
```

### Why not Owner?

Owner is not required by the business scenario.

Giving Owner only to make a failed operation succeed would violate least privilege because it also grants access-management capability.

### Why not PIM here?

Phase 4 validates the static RBAC model.

Eligible / time-bound privileged access, activation and governance are intentionally deferred to **Phase 5 — PIM / Governance**.

---

## 6. Management Plane vs Data Plane

This phase validates **management-plane RBAC**.

The write test modifies an Azure Resource Manager property: a resource tag.

It does not claim that Reader or Contributor automatically provides access to Blob data.

Storage data-plane roles such as:

- Storage Blob Data Reader
- Storage Blob Data Contributor

are separate authorization concepts and are outside the core Phase 4 test.

---

## 7. Validation

### Test A — Reader can inspect the resource

Identity:

`Mia Schneider`

Effective access path:

```text
Mia Schneider
↓
GRP-RBAC-WEZ-LAB-READERS
↓
Reader
↓
rg-wez-rbac-lab
↓
storageaccountwez01
```

Result:

**PASS**

Mia could open and inspect the Storage Account.

Evidence:

- `images/04-rbac-least-privilege/07-mia-reader-access-success.png`

---

### Test B — Reader cannot perform a write

Identity:

`Mia Schneider`

Action:

Attempt to add/modify a Storage Account tag.

Expected:

**Denied**

Actual:

**Denied**

Result:

**PASS**

Evidence:

- `images/04-rbac-least-privilege/08-reader-write-denied.png`

This demonstrates that successful authentication and read access do not imply write authorization.

---

### Test C — Identity with no Phase 4 RBAC assignment

Identity:

`Lisa Werner`

Phase 4 RBAC assignment:

None

Expected:

The Phase 4 resource should not be available through the intended RBAC model.

Actual:

The Storage resource was not available to the identity.

Result:

**PASS**

Evidence:

- `images/04-rbac-least-privilege/09-lisa-no-rbac-access.png`

---

### Test D — Contributor can perform the permitted change

Identity:

`Emilia Haas`

Action:

Modify a Storage Account tag.

After remediation, access path:

```text
Emilia Haas
↓
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
↓
Contributor
↓
rg-wez-rbac-lab
↓
storageaccountwez01
```

Actual:

Tag assignment succeeded.

Result:

**PASS**

Evidence:

- `images/04-rbac-least-privilege/13-emilia-contributor-write-success.png`
- `images/04-rbac-least-privilege/17-contributor-write-success-toast.png`

---

## 8. Controlled Failure — RBAC Group Membership Mismatch

### Requirement

Emilia Haas represents an operator persona that needs to perform a safe management-plane change on the Phase 4 lab resource.

### Controlled initial state

Emilia was intentionally placed in:

`GRP-RBAC-WEZ-LAB-READERS`

This created the following effective access:

```text
Emilia
↓
Readers group
↓
Reader
↓
Resource Group
↓
Storage Account
```

### Symptom

A tag write was attempted.

Result:

**Denied**

Evidence:

- `images/04-rbac-least-privilege/05-reader-initial-membership-controlled-failure.png`
- `images/04-rbac-least-privilege/10-emilia-reader-write-denied.png`

### Investigation

The troubleshooting sequence was:

```text
Requirement
↓
Resource exists
↓
RBAC scope checked
↓
Role assignment checked
↓
Group membership checked
↓
Effective authorization identified
```

The resource and scope were correct.

The Reader assignment was also technically correct.

The mismatch was between the persona requirement and the identity's group membership.

### Root Cause

> Emilia required Contributor-level resource management, but her effective access came from the Reader group.

This was an **authorization / RBAC membership mismatch**, not:

- an authentication failure,
- a Conditional Access failure,
- a Storage Account deployment failure.

### Remediation

Emilia was removed from:

`GRP-RBAC-WEZ-LAB-READERS`

and added to:

`GRP-RBAC-WEZ-LAB-CONTRIBUTORS`

Evidence:

- `images/04-rbac-least-privilege/11-emilia-reader-membership-removed.png`
- `images/04-rbac-least-privilege/12-emilia-contributor-membership-added.png`

### Retest

The same class of management-plane write operation was repeated.

Result:

**Success**

Evidence:

- `images/04-rbac-least-privilege/13-emilia-contributor-write-success.png`

This completed the engineering chain:

```text
Denied operation
↓
Role / scope / membership analysis
↓
Root cause
↓
Least-privilege remediation
↓
Retest
↓
Success
```

---

## 9. Rollback / Recovery

A destructive rollback test was not required for this phase.

The documented RBAC rollback model is:

```text
Unexpected authorization state
↓
Identify affected role + scope + group
↓
Remove incorrect role assignment
or
restore correct group membership
↓
Re-evaluate effective access
↓
Retest
```

The group-membership remediation branch was executed during the controlled failure scenario.

No broad privilege escalation was used as a shortcut.

---

## 10. Evidence Index

| Evidence | What it proves |
|---|---|
| `01-rbac-final-role-assignments.png` | Reader and Contributor are assigned group-based at Resource Group scope |
| `02-storage-account-lab-tags.png` | Phase 4 lab resource and final classification |
| `03-reader-group-definition.png` | Reader access group exists as an assigned Security group |
| `04-contributor-group-definition.png` | Contributor access group exists as an assigned Security group |
| `05-reader-initial-membership-controlled-failure.png` | Initial controlled-failure membership: Mia + Emilia as Readers |
| `06-reader-role-assignment-group-based.png` | Reader role is assigned to a group, not directly to a user |
| `07-mia-reader-access-success.png` | Reader can inspect the Storage Account |
| `08-reader-write-denied.png` | Reader management-plane write is denied |
| `09-lisa-no-rbac-access.png` | No-assignment control identity cannot access the intended Azure resource |
| `10-emilia-reader-write-denied.png` | Operator persona fails while effective access is Reader |
| `11-emilia-reader-membership-removed.png` | Remediation removes incorrect Reader membership |
| `12-emilia-contributor-membership-added.png` | Remediation adds Contributor membership |
| `13-emilia-contributor-write-success.png` | Contributor write succeeds after remediation |
| `14-final-reader-membership.png` | Final Reader group contains Mia |
| `15-final-contributor-membership.png` | Final Contributor group contains Emilia |
| `16-storage-inherited-rbac.png` | Storage Account inherits Reader/Contributor from Resource Group |
| `17-contributor-write-success-toast.png` | Explicit successful write confirmation |

---

## 11. Security / Sanitization

Public evidence is sanitized.

The repository must not expose:

- real tenant primary domain
- real UPNs / administrator account information
- subscription ID
- object IDs / GUIDs
- tokens, passwords or secrets
- private identifiers

WEZ identities and organization data are fictional.

---

## 12. Lessons Learned

1. **Authentication and authorization are separate controls.**  
   Successful sign-in does not imply permission to modify an Azure resource.

2. **Effective access is the combination of role, scope and membership.**  
   A correct role assignment with the wrong group membership can still produce the wrong operational result.

3. **A denied operation is not a reason to assign Owner.**  
   Requirement, role, scope and membership should be investigated before increasing privilege.

4. **Group-based RBAC is easier to operate than direct user assignment.**  
   Access changes can be implemented by changing membership while keeping the resource authorization model stable.

5. **Scope is part of least privilege.**  
   Resource-group scope provided a meaningful authorization boundary without granting subscription-wide access.

---

## 13. Phase 4 Status

```text
Resource access model             ✅
Group-based RBAC                  ✅
Reader role                       ✅
Contributor role                  ✅
Resource-group scope              ✅
Scope inheritance                 ✅
Reader read success               ✅
Reader write denial               ✅
No-assignment control             ✅
Controlled RBAC failure           ✅
Root cause analysis               ✅
Least-privilege remediation       ✅
Successful retest                 ✅
Rollback model documented         ✅
Sanitized evidence package        ✅
```

**PHASE 4 STATUS: COMPLETE**

Next:

**Phase 5 — PIM / Governance**
