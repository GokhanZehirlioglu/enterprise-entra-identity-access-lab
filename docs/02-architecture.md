# Architecture

## Version
`architecture-v1`

## Objective
Model a realistic identity control plane for the fictional **WEZ – Wirtschafts- und Europaforschungszentrum**.

## Overall Identity Architecture

```mermaid
flowchart LR
    subgraph PEOPLE["WEZ People & Personas"]
        EMP["Employees / Researchers"]
        IT["IT Staff"]
        ADM["Privileged Admin Accounts"]
        GST["External Research Guests"]
        BG["Emergency Accounts"]
    end

    subgraph GROUPS["Group Model"]
        DEP["Department Groups"]
        ACC["Application / Azure Access Groups"]
        CAS["Conditional Access Scope Groups"]
        ROLE["Privileged Role Groups"]
    end

    ENTRA["Microsoft Entra ID"]
    AUTH["Authentication / MFA"]
    CA["Conditional Access"]
    PRIV["Privileged Access / PIM / RBAC"]
    APPS["Enterprise Apps / SSO"]
    AZ["Azure Resources"]
    LOGS["Sign-in & Audit Logs"]
    AUTO["PowerShell / Microsoft Graph"]

    EMP --> DEP
    IT --> DEP
    GST --> ACC
    ADM --> ROLE
    BG --> CAS
    DEP --> ACC
    DEP --> CAS
    ACC --> ENTRA
    CAS --> ENTRA
    ROLE --> ENTRA
    ENTRA --> AUTH
    AUTH --> CA
    ENTRA --> PRIV
    CA --> APPS
    CA --> AZ
    PRIV --> APPS
    PRIV --> AZ
    ENTRA --> LOGS
    CA --> LOGS
    PRIV --> LOGS
    AUTO <--> ENTRA
```

## Design Decisions
1. Business access is group-driven rather than assigned directly to users.
2. Standard and privileged accounts are separated.
3. Conditional Access is treated as a staged control plane, not a checkbox.
4. Sign-in and Audit Logs are part of the architecture for validation and RCA.
5. The identity dataset is structured so later Graph/PowerShell automation can consume it.

## Draw.io Source
`diagrams/architecture-v1.drawio`

GitHub shows Draw.io XML as text when opening the raw source. That is normal. The Mermaid diagram above is the GitHub-native visual representation.

## Next Revision
Create `architecture-v2` after users, groups and the first role model are actually deployed and tested. Keep v1 for evidence.
