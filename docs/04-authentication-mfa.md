# Authentication & MFA

> **Phase:** 2 — Authentication & MFA  
> **Current status:** Step 2.1 — Authentication & MFA Design Freeze ✅ Complete  
> **Scenario:** WEZ — fictional research organization  
> **Design freeze date:** 2026-09-07

## Objective

Design, implement and validate a modern Microsoft Entra authentication and MFA baseline for the WEZ lab without mixing authentication-method configuration with Conditional Access enforcement.

The goal is not simply to "turn on MFA." The goal is to define which authentication methods are appropriate for each identity class, implement them safely through a pilot-first rollout, validate expected behavior, capture evidence, and document the reasoning.

---

## Phase Boundary

Phase 2 is responsible for:

- authentication-method design
- authentication-method policy
- MFA-capable user registration
- pilot onboarding
- passwordless / stronger-authentication preparation where appropriate
- positive and controlled negative validation
- evidence and test documentation

Phase 2 does **not** implement the full Conditional Access policy architecture.

The following are intentionally deferred to **Phase 3 — Conditional Access & Emergency Access**:

- Require MFA Conditional Access policies
- authentication-strength enforcement
- report-only Conditional Access rollout
- What If validation
- location / device / application conditions
- break-glass Conditional Access exclusions
- Conditional Access failure and rollback scenarios

**Design principle:** Authentication methods define *how an identity can authenticate*. Conditional Access later defines *when a particular authentication requirement must be enforced*.

---

## Identity Classes

Phase 2 inherits the identity boundaries implemented in Phase 1:

| Identity class | Count | Authentication objective |
|---|---:|---|
| Standard workforce | 38 | Establish a modern MFA-capable workforce baseline |
| Separate privileged identities | 3 | Prepare for stronger, phishing-resistant authentication |
| Emergency / break-glass identities | 2 | Preserve an independent recovery authentication path |
| B2B guest identities | 4 | Keep guest authentication separate from internal workforce assumptions |

No identity redesign is required in Phase 2.

---

# Step 2.1 — Authentication & MFA Design Freeze

## Design Decision 1 — Microsoft Authenticator Is the Workforce Baseline

Microsoft Authenticator is selected as the primary modern MFA method for standard WEZ workforce identities.

The lab will not attempt to register all 38 synthetic users manually. Instead, a small controlled pilot population will be used to prove the design and implementation.

**Why:**

- Microsoft-native authentication method
- appropriate for Entra-based workforce authentication
- supports MFA registration and modern authentication flows
- produces realistic registration and sign-in evidence
- avoids meaningless bulk enrollment of synthetic users

---

## Design Decision 2 — Pilot First, Then Expand the Scope

Authentication changes will be validated with a small pilot population before any broader rollout.

Initial pilot categories should include:

- one standard workforce identity
- one dedicated privileged-class identity
- an emergency identity only when the specific recovery scenario is ready to be tested

The full 38-user workforce population does not need physical MFA enrollment for the lab to prove the design.

**Why:** A staged rollout reduces configuration risk and creates clearer troubleshooting evidence.

---

## Design Decision 3 — Privileged Identities Require a Stronger Authentication Posture

The three dedicated privileged-class identities created in Phase 1 remain separate from daily-work identities.

Phase 2 will prepare these identities for stronger authentication than the standard workforce baseline.

Preferred direction:

- Microsoft Authenticator as an available modern method
- Passkey / FIDO2 as the stronger phishing-resistant candidate

Important limitation:

The three privileged identities do not yet have privileged Entra roles assigned. PIM and actual privileged-role governance belong to a later phase.

Therefore the project must not claim that privileged-role access has already been protected by MFA or FIDO2.

The accurate portfolio statement is:

> Dedicated privileged-class identities are prepared for stronger authentication controls and later privileged-access enforcement.

---

## Design Decision 4 — Emergency Access Uses an Independent Recovery Model

The two emergency / break-glass identities must not simply copy the normal workforce authentication dependency.

Their purpose is tenant recovery.

Phase 2 will document and prepare a recovery-oriented authentication strategy. Actual Conditional Access exclusion, break-glass validation and monitoring are deferred to Phase 3.

Possible strong-authentication / recovery mechanisms may include:

- independently controlled Passkey / FIDO2 credentials
- Temporary Access Pass where appropriate for controlled bootstrap or recovery

No emergency authentication mechanism will be treated as a normal daily sign-in method.

---

## Design Decision 5 — B2B Guests Remain a Separate Authentication Scenario

The four Phase 1 B2B guest objects are synthetic identities.

The lab will not fake invitation redemption or external sign-in merely to create screenshots.

Guest-specific authentication behavior will be documented in Phase 2, while real controlled guest sign-in testing can be performed later with an appropriate external test identity when the Enterprise Application / SSO scenario requires it.

---

## Design Decision 6 — Temporary Access Pass Is a Bootstrap / Recovery Tool

Temporary Access Pass (TAP) is not selected as a normal everyday authentication method.

Its intended role in the lab is:

- bootstrap of passwordless authentication
- controlled recovery
- registration of stronger methods where appropriate

If TAP is tested, its lifetime, one-time-use behavior and target scope must be explicitly documented.

---

## Design Decision 7 — SMS and Voice Are Not the Target Baseline

SMS and voice-call authentication are not selected as the preferred WEZ workforce MFA baseline.

The project prioritizes stronger modern methods rather than enabling every method merely because Entra supports it.

Any weaker or legacy-compatible method must have a concrete requirement before it is introduced.

---

## Design Decision 8 — Legacy Per-User MFA Is Not the Target Architecture

Legacy per-user MFA will not be used as the main MFA enforcement model for this project.

The project separates:

1. authentication-method availability and registration in Phase 2
2. access-policy enforcement through Conditional Access in Phase 3

This keeps the architecture aligned with the later Conditional Access design and prevents competing enforcement models from being mixed unnecessarily.

---

## Design Decision 9 — Security Defaults Is Not the Final Enforcement Architecture

Security Defaults may exist as the tenant's current baseline protection.

It will be inventoried before changes are made.

The project does not treat Security Defaults as the final WEZ authentication / access-control architecture because the later enterprise scenario requires granular Conditional Access controls.

No Security Defaults change is made as part of Step 2.1.

Its current state and the migration point toward Conditional Access will be documented separately.

---

## Design Decision 10 — Evidence Must Prove a Specific Claim

Screenshots are evidence, not decoration.

Every captured image must answer:

> What does this prove?

Planned Phase 2 evidence categories:

- current Security Defaults state
- current Authentication Methods policy
- pilot targeting / scope
- successful authentication-method registration
- successful MFA-capable authentication
- stronger-authentication / passwordless evidence if implemented
- controlled negative result where appropriate
- final test-matrix result

All public evidence must be sanitized before it is committed.

Never expose:

- real tenant domain
- tenant ID
- personal or operator email addresses
- Object IDs / GUIDs
- subscription IDs
- passwords
- secrets
- tokens
- private identifiers

---

# Phase 2 Implementation Sequence

The frozen execution sequence is:

1. **Step 2.1 — Authentication & MFA Design Freeze** ✅
2. **Step 2.2 — Current Authentication Baseline / Inventory**
3. **Step 2.3 — Pilot Identity & Licensing Preparation**
4. **Step 2.4 — Authentication Methods Policy Implementation**
5. **Step 2.5 — MFA Method Registration / Enrollment**
6. **Step 2.6 — Positive Validation Tests**
7. **Step 2.7 — Controlled Negative / Failure Test**
8. **Step 2.8 — Evidence + Test Matrix**
9. **Step 2.9 — GitHub Documentation**
10. **Step 2.10 — Phase 2 Quality Gate**

---

# Step 2.1 Quality Gate

Step 2.1 is complete because the following decisions are frozen:

- [x] workforce baseline method defined
- [x] pilot-first rollout accepted
- [x] privileged authentication direction defined
- [x] emergency authentication treated separately
- [x] B2B guest behavior kept separate
- [x] TAP purpose defined
- [x] SMS / voice excluded from the target baseline
- [x] legacy per-user MFA excluded from the target architecture
- [x] Security Defaults identified as a current-state control, not the final architecture
- [x] Conditional Access explicitly deferred to Phase 3
- [x] evidence standard defined
- [x] public sanitization requirements confirmed
- [x] GitHub documentation path confirmed

**STEP 2.1 STATUS: COMPLETE**

---

# Next Step

## Step 2.2 — Current Authentication Baseline / Inventory

Before changing authentication settings, record the tenant's current state.

Inventory:

- Security Defaults
- Authentication Methods policy
- Microsoft Authenticator
- Passkey / FIDO2
- Temporary Access Pass
- Software OATH
- SMS
- Voice call
- Email OTP
- other enabled or disabled authentication methods
- relevant registration / system-preferred authentication settings

The result of Step 2.2 becomes the documented **before-state** for Phase 2.
