# Identity Model

## Design Goal
User count alone does not create technical depth. Useful complexity comes from different personas, group relationships, policy scopes, lifecycle states and privilege boundaries.

For that reason, WEZ v1 uses **47 directory identities** rather than hundreds of meaningless test users.

## Identity Population

| Identity class | Count | Purpose |
|---|---:|---|
| Standard member accounts | 38 | normal work identities |
| Separate privileged admin accounts | 3 | administrative operations |
| Emergency / break-glass accounts | 2 | tenant recovery |
| Guest identities | 4 | external collaboration |
| **Total** | **47** | |

## Organizational Structure
- Directorate & Strategy: 3
- Research: 20 across four research groups
- IT & Digital Services: 5 standard accounts + 3 separate privileged accounts
- Administration & Finance: 6
- Communications & Knowledge Transfer: 4
- External Guests: 4
- Emergency identities: 2

## Account Separation Example

```text
Sarah Klein
├── sarah.klein@wez-lab.example
│   └── normal daily work account
└── admin.sarah.klein@wez-lab.example
    └── privileged administrative account
```

## Group Model

### Department Groups
- GRP-DEP-DIR
- GRP-DEP-RES-EU
- GRP-DEP-RES-LAB
- GRP-DEP-RES-DIG
- GRP-DEP-RES-PFR
- GRP-DEP-IT
- GRP-DEP-AF
- GRP-DEP-COMMS

### Business Access Groups
- GRP-APP-RESEARCH-PORTAL-USERS
- GRP-APP-FINANCE-USERS
- GRP-APP-COMMS-CMS-EDITORS
- GRP-AZURE-READERS
- GRP-AZURE-LAB-CONTRIBUTORS

### Conditional Access Scope Groups
- GRP-CA-MFA-ALL-MEMBERS
- GRP-CA-ADMINS-STRICT
- GRP-CA-GUESTS
- GRP-CA-BREAKGLASS-EXCLUDED

### Privileged Role Groups
- GRP-ROLE-HELPDESK-ELIGIBLE
- GRP-ROLE-USERADMIN-ELIGIBLE
- GRP-ROLE-SECURITYREADER-ELIGIBLE
- GRP-ROLE-PRIVILEGEDROLEADMIN-ELIGIBLE

## Planned Lifecycle Tests
- Joiner: new researcher enters Digital Economy & Innovation.
- Mover: researcher becomes Research Group Lead.
- Mover: employee transfers from Research to Administration & Finance.
- Leaver: standard access removed and sessions revoked.
- Privilege change: IT staff gains temporary eligible administrative role.
- Guest review: external researcher access expires or is renewed.
- Emergency validation: break-glass account remains usable during policy failure.
