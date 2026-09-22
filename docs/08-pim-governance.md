# PIM & Governance

> **Phase:** 5 — PIM / Governance  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Complete  
> **Scope:** Just-in-time privileged access using Microsoft Entra Privileged Identity Management (PIM) for Groups while preserving the existing group-based Azure RBAC model.

## Objective

Replace standing Contributor access with an eligible, approval-based and time-limited privileged access flow without changing the Azure RBAC architecture created in Phase 4.

The core engineering question was:

> Can an operator receive Azure Contributor capability only when needed, under MFA, justification and approval controls, and lose that capability again after deactivation?

---

## 1. Starting State

Phase 4 had already established:

```text
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
→ Contributor
→ rg-wez-rbac-lab
```

The Storage Account inherited that authorization from the Resource Group.

At the start of Phase 5, Emilia Haas was a permanent member of the Contributor group. That created standing effective Contributor access.

Phase 5 intentionally preserved the Azure RBAC assignment and changed only the lifecycle of group membership.

---

## 2. Design Decision

Instead of assigning Azure Contributor directly to a user through PIM, the existing group-based authorization model was retained.

```text
Emilia Haas
↓
Eligible membership
↓
PIM activation
↓
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
↓
Contributor
↓
rg-wez-rbac-lab
```

This separates:

- **Authorization design** — Azure RBAC remains group-based.
- **Privilege lifecycle** — PIM controls when the operator becomes an active group member.

This avoids switching from group-based RBAC back to direct user-role assignments.

---

## 3. PIM Member Policy

The Contributor group was brought under PIM for Groups management.

Final Member activation policy:

| Control | Configuration |
|---|---|
| Activation maximum duration | 1 hour |
| MFA on activation | Required |
| Justification | Required |
| Ticket information | Not required |
| Approval | Required |
| Approvers | Two designated privileged approvers |
| Permanent eligible assignment | Not allowed by policy |
| Permanent active assignment | Not allowed by policy |

Evidence:

- `images/05-pim-governance/01-pim-member-policy.png`

---

## 4. Standing Membership → Eligible Membership

The permanent active membership was converted to an **Eligible** PIM assignment.

After the conversion:

```text
Emilia Haas
→ Eligible for GRP-RBAC-WEZ-LAB-CONTRIBUTORS
→ Not active
```

Before activation, the identity no longer had effective Contributor access to the Phase 4 Azure resource.

Evidence:

- `images/05-pim-governance/02-emilia-eligible-assignment.png`
- `images/05-pim-governance/10-preactivation-access-denied.png`

Result:

**PASS**

This proves that eligibility is not equivalent to active privilege.

---

## 5. JIT Activation Workflow

The eligible member requested activation for one hour.

The request included a business justification for a temporary management-plane change.

The activation workflow required:

```text
Eligible membership
↓
Activation request
↓
MFA requirement
↓
Justification
↓
Approval
↓
Temporary active membership
```

Evidence:

- `images/05-pim-governance/03-activation-request.png`
- `images/05-pim-governance/04-approval-request.png`

After approval, the PIM assignment moved to the Active state.

Evidence:

- `images/05-pim-governance/05-active-jit-membership.png`

---

## 6. Positive Authorization Test

After PIM activation, Emilia again became an active member of:

`GRP-RBAC-WEZ-LAB-CONTRIBUTORS`

No Azure RBAC role assignment was modified.

The same existing authorization path became effective again:

```text
Emilia Haas
↓
Temporary active group membership
↓
GRP-RBAC-WEZ-LAB-CONTRIBUTORS
↓
Contributor
↓
rg-wez-rbac-lab
↓
storageaccountwez01
```

The Storage Account became accessible again.

A management-plane tag change was then performed using:

```text
PIM-JIT-Test = Validated
```

Result:

**PASS**

Evidence:

- `images/05-pim-governance/06-jit-contributor-write.png`

The tag was used only as a safe management-plane validation and was cleaned up after the test.

---

## 7. Auditability

The user's PIM audit history recorded the privileged-access lifecycle, including:

- eligible membership changes,
- activation request,
- approval request,
- approver action,
- completed activation / membership addition.

Evidence:

- `images/05-pim-governance/07-pim-my-audit.png`

The resource-wide audit view was not used as final public evidence because the current test identity did not expose a populated resource-audit view. The user-specific audit history was sufficient to demonstrate that the activation and approval events were recorded.

No claim is made that every tenant-wide audit surface was validated.

---

## 8. Deactivation and Privilege Removal

The temporary PIM membership was manually deactivated before the one-hour activation window expired.

Evidence:

- `images/05-pim-governance/08-deactivation-complete.png`

After deactivation:

```text
Active membership
→ removed

Eligible assignment
→ retained
```

The Contributor group no longer had an active member for the tested identity, while the eligibility remained available for a future approved activation.

---

## 9. Post-Deactivation Negative Test

After deactivation, the Azure Storage resource was no longer accessible through the tested Contributor path.

Result:

**PASS**

Evidence:

- `images/05-pim-governance/09-post-deactivation-access-denied.png`

This completes the JIT lifecycle validation:

```text
Eligible
↓
Not active
↓
Access denied

Eligible
↓
Activate + MFA + justification + approval
↓
Temporary active membership
↓
Contributor access
↓
Management-plane write succeeds

Deactivate
↓
Active membership removed
↓
Access denied again
```

---

## 10. Governance Model

The lab validated the technical privileged-access lifecycle with PIM for Groups.

A periodic recertification model is documented for the eligible privileged membership:

```text
Privileged eligibility
↓
Periodic review
↓
Does the operator still require Contributor eligibility?
├── Yes → retain eligibility
└── No  → remove eligible assignment
```

A dedicated PIM-for-Groups Access Review was not implemented in this phase. The governance control is therefore documented as a design concept rather than presented as executed evidence.

This limitation is explicit; the project does not claim an Access Review execution that did not occur.

---

## 11. Rollback / Recovery

If the PIM design creates an unexpected authorization state:

```text
Unexpected privileged-access result
↓
Check eligible assignment
↓
Check activation state
↓
Check approval state
↓
Check active group membership
↓
Check existing Azure RBAC assignment and scope
↓
Deactivate or remove the incorrect PIM assignment
↓
Re-evaluate effective access
↓
Retest
```

The deactivation branch was executed during this phase and successfully removed effective Contributor access.

---

## 12. Security Decisions

1. **Existing group-based RBAC was preserved.**
2. **Standing Contributor membership was removed.**
3. **Eligibility alone does not grant resource access.**
4. **Activation is time-limited.**
5. **MFA, justification and approval are required.**
6. **No Owner or subscription-wide privilege was introduced.**
7. **Emergency identities were not made dependent on PIM.**
8. **No direct user Azure RBAC assignment was added for the JIT scenario.**
9. **Access Review execution is not claimed where it was not implemented.**

---

## 13. Evidence Index

| Evidence | What it proves |
|---|---|
| `01-pim-member-policy.png` | 1-hour activation, MFA, justification and approval controls |
| `02-emilia-eligible-assignment.png` | Contributor-group membership is Eligible rather than standing active |
| `03-activation-request.png` | Time-limited activation request and justification |
| `04-approval-request.png` | Human approval step exists |
| `05-active-jit-membership.png` | Eligible membership became temporarily active |
| `06-jit-contributor-write.png` | JIT activation restored Contributor management-plane capability |
| `07-pim-my-audit.png` | PIM request / approval / activation events were recorded |
| `08-deactivation-complete.png` | Temporary privilege was explicitly deactivated |
| `09-post-deactivation-access-denied.png` | Effective Contributor access disappeared after deactivation |
| `10-preactivation-access-denied.png` | Eligible-but-inactive state did not provide resource access |

---

## 14. Lessons Learned

1. **Eligibility is not authorization.** The user must activate before the group-based Contributor path becomes effective.
2. **PIM can preserve an existing RBAC architecture.** The Azure role assignment did not need to change during activation or deactivation.
3. **JIT privilege is a lifecycle, not a checkbox.** A strong demonstration requires before-state denial, controlled activation, positive validation and post-deactivation denial.
4. **Approval and auditability matter as much as temporary access.** Privilege should have a reason, an approver and an observable history.
5. **Do not fake governance evidence.** A documented Access Review model is preferable to claiming a review that was not executed.

---

## 15. Phase 5 Status

```text
PIM for Groups onboarding                 ✅
Standing Contributor membership removed   ✅
Eligible membership                       ✅
1-hour activation limit                   ✅
MFA on activation                         ✅
Justification required                    ✅
Approval required                         ✅
Two approvers configured                  ✅
Pre-activation access denial              ✅
Activation request                        ✅
Approval workflow                         ✅
Temporary active membership               ✅
Contributor resource access restored      ✅
Management-plane write validated          ✅
PIM user audit evidence                    ✅
Manual deactivation                       ✅
Post-deactivation access denial           ✅
Governance / review model documented      ✅
Access Review execution                   NOT CLAIMED
```

**PHASE 5 STATUS: COMPLETE**

Next:

**Phase 6 — Enterprise App / SSO**
