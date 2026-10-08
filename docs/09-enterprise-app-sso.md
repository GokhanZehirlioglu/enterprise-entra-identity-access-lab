# Enterprise Application & SSO

> **Phase:** 6 — Enterprise App / SSO  
> **Scenario:** WEZ — fictional research organization  
> **Status:** Complete  
> **Scope:** SAML 2.0 enterprise application integration, group-based application assignment, controlled assignment failure, sign-in-log RCA, remediation/retest, My Apps launch validation, and business-attribute claim configuration.

## Objective

Integrate an enterprise-style SaaS application with Microsoft Entra ID as the Identity Provider (IdP), restrict access through a dedicated security group, validate successful SAML SSO, deliberately test an unassigned user, diagnose the failure from Entra sign-in evidence, remediate the access path without weakening the control, and restore least privilege after the test.

The core engineering question was:

> Can WEZ provide group-controlled SAML SSO to an enterprise application while keeping assignment enforcement enabled and producing a complete failure → RCA → remediation → retest chain?

---

## 1. Starting State

Phase 1 intentionally deferred application-access groups until a real application requirement existed.

Phase 6 introduced that requirement and created:

`GRP-APP-RESEARCH-PORTAL-USERS`

The application scenario is:

**WEZ Research Collaboration Portal**

The technical implementation uses the Microsoft Entra SAML Toolkit as the Service Provider (SP) for a controlled lab integration.

---

## 2. Design Freeze

| Decision | Final design |
|---|---|
| Enterprise application | Microsoft Entra SAML Toolkit |
| WEZ application name | WEZ Research Collaboration Portal |
| Federation protocol | SAML 2.0 |
| Identity Provider | Microsoft Entra ID |
| Access model | Group-based assignment |
| Access group | `GRP-APP-RESEARCH-PORTAL-USERS` |
| Assignment required | Yes |
| Positive-control user | Lisa Werner |
| Negative-control user | Mia Schneider |
| NameID format | Email address |
| NameID source | `user.userprincipalname` |
| Business claims configured | `department`, `jobTitle` |
| Main controlled failure | Unassigned user blocked by application assignment enforcement |
| RCA source | Enterprise Application Sign-in Logs |
| Rollback | Remove temporary remediation membership after successful retest |

SCIM provisioning and PowerShell / Microsoft Graph automation are intentionally out of scope for Phase 6.

---

## 3. Enterprise Application

A gallery-based SAML application was created and renamed:

`WEZ Research Collaboration Portal`

Important application properties:

- user sign-in enabled,
- `Assignment required = Yes`,
- application visible to assigned users.

The assignment requirement is a security boundary: successful Entra authentication alone is not sufficient for application access.

Evidence:

- `images/06-enterprise-app-sso/01-enterprise-app-properties.png`

---

## 4. Application Access Group

A dedicated assigned Security group was created:

`GRP-APP-RESEARCH-PORTAL-USERS`

Purpose:

> Controls access to the WEZ Research Collaboration Portal enterprise application.

Initial authorized member:

- Lisa Werner — Finance Controller, Administration & Finance

The group was assigned to the enterprise application with **Default Access**.

No direct per-user application assignment was used for the normal access model.

Evidence:

- `images/06-enterprise-app-sso/02-app-access-group.png`
- `images/06-enterprise-app-sso/03-group-application-assignment.png`

---

## 5. SAML Federation Configuration

The SAML Toolkit was configured as the Service Provider and Microsoft Entra ID as the Identity Provider.

Final Service Provider values followed the configuration-specific Toolkit endpoints:

```text
Entity ID
https://samltoolkit.azurewebsites.net

SP-initiated Sign-on URL
https://samltoolkit.azurewebsites.net/SAML/Login/<configuration-id>

Assertion Consumer Service (ACS) URL
https://samltoolkit.azurewebsites.net/SAML/Consume/<configuration-id>
```

The Entra-side SAML configuration was aligned to the Toolkit-generated Entity ID, sign-on URL and ACS endpoint.

The Microsoft Entra SAML signing certificate was used by the Toolkit to validate signed assertions.

Evidence:

- `images/06-enterprise-app-sso/04-saml-configuration.png`

No certificate private key, password, token or other credential is stored in the repository.

---

## 6. NameID and Claims

The required NameID mapping was configured as:

```text
Name identifier format: Email address
Source: Attribute
Source attribute: user.userprincipalname
```

Two business-context claims were also configured:

```text
department → user.department
jobTitle   → user.jobtitle
```

For Lisa Werner the directory values are:

```text
Department: Administration & Finance
Job title: Finance Controller
```

The project claims **configuration evidence** for these business attributes. The raw SAML assertion was not independently decoded and therefore the project does not claim separate assertion-level verification of the two custom claim values.

Evidence:

- `images/06-enterprise-app-sso/10-saml-claims-configuration.png`

---

## 7. Positive SSO Test — Lisa Werner

Lisa Werner was already a direct member of the assigned application-access group.

The SP-initiated flow was tested in a clean private browser session:

```text
Lisa Werner
↓
Microsoft Entra authentication
↓
Conditional Access evaluation
↓
Application assignment evaluation
↓
SAML assertion
↓
SAML Toolkit
↓
Successful application session
```

The Toolkit displayed Lisa as the signed-in user.

Entra Enterprise Application Sign-in Logs subsequently recorded the application sign-in as **Success**.

A prior credential typo produced error `50126`; it was retained as normal sign-in history and is not treated as a SAML failure. A later successful event is the authoritative positive result.

Evidence:

- `images/06-enterprise-app-sso/05-lisa-sso-success.png`
- `images/06-enterprise-app-sso/06-lisa-signin-log-success.png`

Result:

**PASS**

---

## 8. Controlled Failure — Mia Schneider

Mia Schneider was intentionally kept outside:

`GRP-APP-RESEARCH-PORTAL-USERS`

while:

`Assignment required = Yes`

remained enabled.

Mia then attempted the same SP-initiated SAML sign-in.

Expected result:

**Access denied**

Actual result:

`AADSTS50105`

The error explicitly stated that the user was blocked because she was not directly assigned and was not a direct member of a group with application access.

Evidence:

- `images/06-enterprise-app-sso/07-mia-unassigned-aadsts50105.png`

---

## 9. Sign-in Log RCA

The Entra Enterprise Application Sign-in Log for Mia showed:

```text
Application: WEZ Research Collaboration Portal
Status: Failure
Sign-in error code: 50105
Conditional Access: Success
```

This separated the failure domains:

```text
Authentication / CA
→ successful

Enterprise application assignment
→ denied
```

Root cause:

> Mia did not have effective application assignment because she was not a member of the assigned application-access group.

Classification:

**Enterprise application authorization / assignment mismatch**

It was not:

- a password failure,
- a Conditional Access failure,
- a SAML certificate failure,
- a NameID failure,
- an Azure RBAC failure.

Evidence:

- `images/06-enterprise-app-sso/08-mia-50105-signin-log.png`

---

## 10. Remediation and Retest

The application security control was not weakened.

The following settings were deliberately preserved:

- `Assignment required = Yes`
- group-based application assignment
- existing SAML configuration

Remediation:

Mia was temporarily added to:

`GRP-APP-RESEARCH-PORTAL-USERS`

After membership propagation, Entra Sign-in Logs showed successful application sign-in.

The first Service Provider-side retest still produced a generic Toolkit application error even though Entra had recorded **Success**.

Investigation showed that the SAML Toolkit requires a local Toolkit user whose email matches the Entra user NameID. Lisa already had such a Toolkit account; Mia did not.

After creating the matching Toolkit test account for Mia, the same SAML flow completed successfully and the Toolkit displayed Mia as signed in.

Evidence:

- `images/06-enterprise-app-sso/09-mia-successful-retest.png`

Result:

**PASS**

---

## 11. Observed SP-side Mapping Issue

This lab produced an additional useful troubleshooting distinction:

```text
Entra sign-in = Success
does not necessarily mean
Service Provider application session = Success
```

In Mia's first post-remediation retest:

- Entra authentication succeeded,
- Conditional Access succeeded,
- application assignment succeeded,
- the Service Provider still returned a generic application error.

The remaining dependency was application-local user matching inside the sample Toolkit.

Lesson:

> Identity Provider success and Service Provider application success are separate checkpoints.

This is documented as an observed Toolkit behavior, not as a failure of Entra SAML federation.

---

## 12. My Apps / IdP-Initiated Launch

Lisa signed in to Microsoft My Apps.

The assigned **WEZ Research Collaboration Portal** tile was visible and launched the SAML Toolkit successfully without a new application-assignment error.

This validated the assigned-user application-launch path in addition to the direct SP-initiated URL.

Evidence:

- `images/06-enterprise-app-sso/11-myapps-application-visible.png`

Result:

**PASS**

---

## 13. Rollback and Final Least-Privilege State

Mia's application-group membership existed only to prove the remediation and retest path.

After successful validation, Mia was removed from:

`GRP-APP-RESEARCH-PORTAL-USERS`

Final access model:

```text
GRP-APP-RESEARCH-PORTAL-USERS
└── Lisa Werner
```

Therefore the controlled remediation did not become permanent access.

Rollback principle:

```text
Unexpected application-access state
↓
Check authentication / CA
↓
Check enterprise-app assignment
↓
Check direct group membership
↓
Correct membership if justified
↓
Retest
↓
Remove temporary test access
↓
Restore least privilege
```

---

## 14. Security Decisions

1. **Application assignment enforcement remains enabled.**
2. **Normal access is group-based rather than direct-user assignment.**
3. **An unassigned user is deliberately denied.**
4. **The security control was not disabled to fix the failure.**
5. **The remediation changed group membership, not application policy.**
6. **Temporary remediation access was removed after validation.**
7. **NameID uses the user's Entra UPN in email-address format.**
8. **Business claims are configured, but decoded-assertion validation is not claimed.**
9. **No SCIM provisioning is claimed.**
10. **No Graph / PowerShell automation is claimed in this phase.**

---

## 15. Evidence Index

| Evidence | What it proves |
|---|---|
| `01-enterprise-app-properties.png` | Enterprise application exists and assignment enforcement is enabled |
| `02-app-access-group.png` | Dedicated application-access Security group design |
| `03-group-application-assignment.png` | Group-based assignment to the enterprise application |
| `04-saml-configuration.png` | Entra SAML configuration and federation endpoints |
| `05-lisa-sso-success.png` | Lisa reached the Service Provider through SSO |
| `06-lisa-signin-log-success.png` | Entra recorded Lisa's enterprise-app sign-in as Success |
| `07-mia-unassigned-aadsts50105.png` | Unassigned user was blocked by assignment enforcement |
| `08-mia-50105-signin-log.png` | Sign-in Log RCA evidence for error 50105 |
| `09-mia-successful-retest.png` | Mia completed SSO after justified remediation and SP-side user matching |
| `10-saml-claims-configuration.png` | Department and jobTitle SAML claim mappings are configured |
| `11-myapps-application-visible.png` | Assigned enterprise application is visible in Lisa's My Apps |

---

## 16. Lessons Learned

1. **Authentication is not application authorization.** A user can authenticate successfully and still be denied because no application assignment exists.
2. **Assignment enforcement is a real security boundary.** `Assignment required = Yes` produced the expected AADSTS50105 denial.
3. **Group membership is part of the access-control system.** Correct app configuration with incorrect membership still produces the wrong effective access.
4. **Do not weaken the policy to fix an assignment problem.** The correct remediation was justified group membership.
5. **IdP success is not identical to SP success.** The Toolkit's local user mapping created a separate application-side dependency.
6. **Evidence from Sign-in Logs is more reliable than browser appearance alone.**
7. **Temporary test access should be removed after validation.**
8. **Configured claims should not be described as assertion-validated unless the assertion was actually inspected.**

---

## 17. Phase 6 Status

```text
Enterprise application created                 ✅
SAML 2.0 federation configured                 ✅
Group-based application assignment             ✅
Assignment required                            ✅
Lisa positive SSO                              ✅
Lisa sign-in-log success                       ✅
Mia unassigned negative test                   ✅
AADSTS50105 captured                           ✅
Sign-in-log RCA                                ✅
Group-membership remediation                   ✅
Successful Mia retest                          ✅
SP-side local-user mapping issue understood    ✅
My Apps launch                                 ✅
department / jobTitle claims configured        ✅
Raw assertion claim decoding                   NOT CLAIMED
Temporary Mia access removed                   ✅
Least-privilege final state restored           ✅
SCIM provisioning                              OUT OF SCOPE
PowerShell / Graph                             DEFERRED TO PHASE 7
```

**PHASE 6 STATUS: COMPLETE**

Next:

**Phase 7 — Project-1 PowerShell / Microsoft Graph layer**
