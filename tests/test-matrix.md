# Test Matrix

| ID | Scenario | Expected Result | Actual | Status |
|---|---|---|---|---|
| T01 | Standard researcher sign-in | Baseline controls apply | TBD | NOT RUN |
| T02 | User outside application group | Access denied | TBD | NOT RUN |
| T03 | Finance user in approved group | Access allowed | TBD | NOT RUN |
| T04 | Standard user attempts privileged operation | Denied | TBD | NOT RUN |
| T05 | Privileged admin activates eligible role | Time-limited privilege where licensing permits | TBD | NOT RUN |
| T06 | Emergency account validation | Emergency access remains available | TBD | NOT RUN |
| T07 | Incorrect CA group scope | Failure visible in sign-in evidence | TBD | NOT RUN |
| T08 | Guest accesses assigned research app | Allowed under guest controls | TBD | NOT RUN |
| T09 | Guest attempts unassigned app | Denied | TBD | NOT RUN |
| T10 | Azure Reader attempts write | Denied | TBD | NOT RUN |
| T11 | Azure Contributor performs permitted change | Allowed | TBD | NOT RUN |
| T12 | Enterprise App / SSO misconfiguration | Failure captured and RCA completed | TBD | NOT RUN |
| T13 | Graph script lacks permission | Predictable failure and permission RCA | TBD | NOT RUN |
| T14 | Mover changes department | Old access removed, new access applied | TBD | NOT RUN |
| T15 | Leaver offboarding | Membership/access/session controls removed | TBD | NOT RUN |
