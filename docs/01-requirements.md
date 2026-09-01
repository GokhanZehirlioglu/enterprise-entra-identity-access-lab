# Requirements

## 1. Business Scenario
WEZ is a fictional research institute with internal researchers, management, administration, finance, IT staff and external research partners.

## 2. Organizational Structure
1. Directorate & Strategy
2. Research
   - European Economy & Policy
   - Labour Markets & Social Policy
   - Digital Economy & Innovation
   - Public Finance & Regulation
3. IT & Digital Services
4. Administration & Finance
5. Communications & Knowledge Transfer
6. External Guests

The initial dataset contains 47 directory identities.

## 3. Identity Requirements
- **IR-01:** Every employee receives one named standard account.
- **IR-02:** IT personnel requiring admin roles use separate privileged accounts.
- **IR-03:** Access should use groups instead of direct user assignment wherever practical.
- **IR-04:** External partners use Guest identities with explicit access.
- **IR-05:** Department, job title, user type and persona attributes are stored for later lifecycle automation.

## 4. Authentication Requirements
- **AR-01:** Member accounts use MFA according to the lab baseline.
- **AR-02:** Privileged identities receive stronger controls than standard users.
- **AR-03:** Legacy authentication is blocked where the tenant and test scenario allow it.

## 5. Conditional Access Requirements
- **CA-01:** High-impact policies use staged rollout/report-only testing when possible.
- **CA-02:** Emergency access accounts are deliberately excluded from normal CA scope and tested separately.
- **CA-03:** Privileged identities receive stricter CA controls.
- **CA-04:** Guests have an explicit CA strategy.
- **CA-05:** Policy behavior is validated with What If/sign-in evidence.

## 6. Privileged Access Requirements
- **PA-01:** Administrative roles follow least privilege.
- **PA-02:** Where licensing permits, roles are eligible/time-limited through PIM/JIT.
- **PA-03:** Two independent emergency accounts reduce recovery risk.
- **PA-04:** Privileged assignment/activation is auditable.

## 7. Azure RBAC Requirements
- Group-based Azure access where practical.
- Reader vs Contributor scenario.
- At least one deliberate insufficient/excessive permission failure.

## 8. Application / SSO Requirements
- At least one Enterprise Application / SSO integration.
- Group-based assignment.
- At least one controlled SSO failure and RCA.

## 9. Monitoring & Audit Requirements
- Sign-in evidence for authentication/CA tests.
- Audit evidence for identity/role/policy changes.
- Troubleshooting must cite evidence before changing configuration.

## 10. Automation Requirements
- Structured source dataset in `sanitized-samples/users.csv`.
- Later provisioning/lifecycle via PowerShell and/or Microsoft Graph.
- At least one permission/error scenario with documented fix.

## 11. Public Data Safety
No real employer, internship organization, person, tenant, production domain, email address, device, ticket, IP, GUID, subscription ID, token or secret may appear in public output.

All WEZ identities are fictional. `wez-lab.example` is documentation-only.
