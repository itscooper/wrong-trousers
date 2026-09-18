---
name: threat-modelling
description: Identify credible threats and appropriate controls.
---

# Threat Modelling Skill

## Purpose

This skill describes how to conduct a threat model: understand the system under review, identify meaningful threats, and propose proportionate controls.

Threat modelling is a proactive, lightweight, and repeatable process for identifying security risks during the design or planning phase of development — where they are cheapest and easiest to fix. The output is a living artefact: a set of security requirements and a risk record that can be updated as the product evolves.

---

## The Four Questions

Every threat model is built around four questions (from the Threat Modeling Manifesto):

1. **What are we working on?** — Understand the system.
2. **What can go wrong?** — Identify threats.
3. **What are we going to do about it?** — Define controls.
4. **Did we do a good enough job?** — Review and iterate.

These questions structure the three core phases below.

---

## Phase 1: Understand the Solution

### 1.1 Define Scope and Purpose

Before identifying threats, you must understand what you are modelling. Establish:

- **What is being built or changed?** Is this a new system, an integration with a third party, a change to an existing component, or a new security-sensitive feature (e.g. payments, authentication)?
- **What is the purpose of the system?** What user problems does it solve? What data does it handle?
- **What are the boundaries of the model?** What is in scope, and what is out of scope for this session?

Threat modelling is most valuable for: new products, third-party integrations, changes to security controls, payment or identity functionality, and any change perceived to carry meaningful security risk.

### 1.2 Gather and Annotate Diagrams

The goal is to build a shared, accurate picture of how the system works — technically and from a user perspective. Use **whatever artefacts already exist** before creating new ones:

- Existing architectural diagrams
- Data flow diagrams (DFDs)
- User journey maps
- API documentation
- Infrastructure diagrams

If no diagrams exist, create a **high-level data flow diagram** that covers:

- **External entities** — users, third-party systems, other services that interact with the system
- **Processes** — components that receive, transform, or route data
- **Data stores** — databases, caches, file systems, queues
- **Data flows** — how data moves between entities, processes, and stores (annotate with data types where useful)

Annotate diagrams to show **trust boundaries** (see 1.3). You do not need a perfect diagram — a good-enough diagram focused on the right level is more valuable than a precise but overly detailed one.

### 1.3 Identify Trust Boundaries

Trust boundaries are the most important analytical tool for scoping threat identification. A trust boundary exists wherever **the level of trust changes** between two communicating components — i.e., where data crosses from one trust domain into another.

**Common examples:**
- Internet user → public-facing API (large trust gap — high priority)
- System → third-party vendor API (large trust gap — high priority)
- API → internal database (smaller gap — lower priority but still relevant)
- Between microservices in the same VPC (often a smaller gap, depending on auth model)

**Prioritise** trust boundaries with the **largest trust gaps** first. A user-facing API endpoint typically warrants more attention than an internal service-to-service call secured within a private network, unless the internal call presents unusual risk.

Mark trust boundaries clearly on your diagram. Every trust boundary is a potential area for threat identification.

### 1.4 Understand Data Sensitivity

Consider what data the system handles and at what points:

- What is the most sensitive data in the system? (PII, payment data, credentials, health data, etc.)
- Where does this data enter the system, and where does it flow to?
- Where is it stored, and in what form? (encrypted at rest? in transit?)
- Who has access to it, and under what conditions?

Data sensitivity informs the **inherent risk** rating of threats and which threats are worth prioritising.

---

## Phase 2: Identify Threats

### 2.1 Focus on Trust Boundaries

For each identified trust boundary, brainstorm threats by asking: *What could an adversary do at this crossing point?*

**Think like an attacker.** Consider what a motivated, capable adversary would try to achieve against this system. Their goals typically include:
- Gaining unauthorised access to data or functionality
- Disrupting availability for legitimate users
- Covering their tracks or avoiding accountability
- Escalating their privileges within the system

Avoid documenting "noise" — threats that are already inherently and universally mitigated (e.g. transport encryption between internal services behind a VPC, if that is standard practice and not being deviated from). Focus on threats that will generate meaningful, constructive output.

### 2.2 Use STRIDE as a Prompt

STRIDE is a structured mnemonic to ensure comprehensive coverage across all threat categories. For each component or trust boundary, systematically ask each STRIDE question:

| Letter | Threat Category | Security Property Violated | Prompt Question |
|--------|----------------|--------------------------|-----------------|
| **S** | **Spoofing** | Authentication | Can an adversary pretend to be another user or system? |
| **T** | **Tampering** | Integrity | Can an adversary alter data or system components without authorisation? |
| **R** | **Repudiation** | Accountability / Non-repudiation | Can an adversary perform actions and deny responsibility without being detected? |
| **I** | **Information Disclosure** | Confidentiality | Can an adversary access sensitive information they are not authorised to see? |
| **D** | **Denial of Service** | Availability | Can an adversary make the system unavailable or significantly degrade performance? |
| **E** | **Elevation of Privilege** | Authorisation | Can an adversary gain higher privileges than they are entitled to? |

STRIDE is not the only framework, but it is proven, widely understood, and well-suited to structured brainstorming. It pairs naturally with data flow diagrams. Apply it systematically — don't skip categories because they "seem unlikely"; the value of the framework is in its completeness.

### 2.3 Develop Each Threat

For each threat identified, capture:

- **A description** — be specific about the mechanism, not just the category. Not "information disclosure" but "an attacker can enumerate user accounts via the password reset endpoint's differential response behaviour."
- **The component(s) affected** — which process, data store, or data flow is the locus of the threat?
- **The trust boundary involved** — at which crossing does this threat materialise?
- **Inherent risk** — how severe would the impact be if this threat materialised, in the absence of any controls? Use a consistent scale relative to other threats in the model: `Critical / High / Medium / Low / Very Low`.

Inherent risk is assessed on a combination of:
- **Impact** — how much harm would result (data loss, financial damage, reputational harm, user safety)?
- **Likelihood** — how easy is this to exploit, and how motivated would an attacker be?

### 2.4 Spend Time Proportionately

Allocate discussion time according to:

- **Higher risk** threats warrant more time
- Threats **specific to this application's design** are more valuable than generic threats applicable to any web application
- Threats **not already addressed** by existing standards, platform defaults, or security baselines are worth more focus

Avoid analysis paralysis. A threat model that covers eight meaningful threats well is more valuable than one that catalogues forty threats superficially.

### 2.5 Retrospective Modelling Considerations

When modelling **existing systems** (rather than designs):

- Identify existing controls first — do they actually work?
- Where possible, **generate test cases** and validate threats through penetration testing before proposing costly remediations
- Recommend controls that **align with the current architecture** unless the severity justifies re-architecture
- Focus on what changes are practical given the current state

---

## Phase 3: Define Controls

### 3.1 Determine a Response for Every Threat

Every identified threat must have an explicit response. The four options are:

| Response | Meaning |
|----------|---------|
| **Mitigate** | Take action to reduce likelihood or impact — implement a control |
| **Eliminate** | Remove the feature or component that introduces the threat |
| **Transfer** | Shift responsibility to another party (e.g. a third-party provider, contractual obligation, insurance) |
| **Accept** | Consciously accept the risk — document the decision and rationale |

"Accept" is a valid response but must be an **explicit, documented decision** — not an implicit omission. It is appropriate when the cost of mitigation outweighs the risk, or when a business constraint makes mitigation impractical.

### 3.2 Define Specific, Actionable Controls

For each threat being mitigated, define a control that:

- **Is specific** — not "improve authentication" but "enforce multi-factor authentication for all admin console access"
- **Is implementable** — can be built into the system as a concrete engineering story
- **Addresses the root threat mechanism** — not just its symptoms
- **Aligns with existing architecture** — avoid proposing re-architecture unless the risk clearly justifies it

Useful references for control selection:
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) — verification standards mapped to security requirements
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) — practical control guidance by topic
- Existing security standards, RFCs, and platform security baselines

### 3.3 Assess Residual Risk

For each control, estimate the **residual risk** — the risk that remains after the control is implemented. Use the same scale as inherent risk: `Critical / High / Medium / Low / Very Low`.

The gap between inherent and residual risk expresses the value of the control. If residual risk is still high after proposed controls, consider whether additional controls or architectural changes are warranted.

### 3.4 Record Control Status

Track the implementation status of each control:

- `Not Implemented` — control is agreed but not yet built
- `Partially Implemented` — control exists but has gaps
- `Implemented` — control is fully in place

Link each control to an issue in the project's tracking system so that implementation is tracked and the threat model stays up to date with delivery.

---

## The Threat Model Artefact

The output of a threat model is a structured document — a living artefact — that records:

### Section 1: What is this thing?
Diagrams, user journeys, descriptions, and/or references that allow any reader to understand the solution from both application and architectural perspectives. Must be sufficient for someone unfamiliar with the system to understand it.

### Section 2 & 3: Threats and Controls

A structured log table with the following columns:

| Field | Description |
|-------|-------------|
| **Threat ID** | Unique identifier (e.g. `T.A01`, `T.B01`) — use sections to group related threats |
| **Threat** | Description of the threat |
| **Inherent Risk** | `Critical / High / Medium / Low / Very Low` — risk before controls |
| **Control** | The agreed mitigation or response |
| **Control Status** | `Not Implemented / Partially Implemented / Implemented` |
| **Issue** | Link to implementation story/ticket |
| **Residual Risk** | `Critical / High / Medium / Low / Very Low` — risk after controls |

Threats can be grouped into named sections (e.g. `SECTION A: Authentication`, `SECTION B: Data Storage`) to improve readability.

---

## Key Principles

### Systematic, Not Exhaustive
Use STRIDE and trust boundaries as a systematic structure, not a bureaucratic checklist. The goal is thorough coverage of the threat landscape, not an exhaustive catalogue of every conceivable risk. Prioritise constructive threats over noise.

### Proportional Effort
The depth and duration of a threat model should be proportional to the risk and complexity of the change. A small feature change may be modelled quickly and informally; a new payment system warrants significantly more rigour.

### Diverse Perspectives
Threat modelling is most effective when it draws on multiple viewpoints: security, architecture, engineering, product. No single "hero" threat modeller or agent has full visibility. Different backgrounds surface different threat vectors. Expect your output to be reviewed, discussed and challenged by people and agents - bear this in mind whilst writing.

### Avoid Overcomplicating
The most common failure mode is over-engineering the process. Heavy frameworks, exhaustive documentation, and lengthy sessions reduce participation and kill momentum. Favour simplicity, speed, and actionability.

### Living Document
A threat model is not a one-time snapshot. It should be updated as the product evolves, as new threats emerge, and as controls are implemented. The artefact grows with the product.

### Shift Left
Threat modelling is most effective — and most cost-efficient — at the design and planning phase, before code is written. Retrospective modelling is valid and useful, but prospective modelling is the ideal.

### From Model to Penetration Test
A well-structured threat model generates focused test cases for penetration testers. When the system reaches a feature-complete state in a lower environment, share the threat model with the penetration testing team so they can test against the specific threats identified. Request their feedback on which threats were practical and which controls were effective.

---

## STRIDE Quick Reference

### Spoofing
**Violates:** Authentication  
**Ask:** Can an attacker impersonate a user, system, or service?  
**Common controls:** Strong authentication (MFA where appropriate), token validation, mutual TLS for service-to-service, signed requests, certificate pinning.

### Tampering
**Violates:** Integrity  
**Ask:** Can an attacker modify data in transit, at rest, or within a process?  
**Common controls:** Input validation, parameterised queries, HMAC/signing for sensitive data, integrity checks, write access controls, audit trails.

### Repudiation
**Violates:** Accountability / Non-repudiation  
**Ask:** Can an attacker perform actions and then plausibly deny them?  
**Common controls:** Comprehensive and tamper-evident audit logging, log centralisation, correlation of actions to authenticated identities, digital signatures for high-value transactions.

### Information Disclosure
**Violates:** Confidentiality  
**Ask:** Can an attacker access data they are not authorised to see?  
**Common controls:** Encryption at rest and in transit, fine-grained authorisation (need-to-know), proper error handling (no stack traces in production), secrets management, masking of sensitive fields in logs.

### Denial of Service
**Violates:** Availability  
**Ask:** Can an attacker degrade or eliminate service for legitimate users?  
**Common controls:** Rate limiting, throttling, CAPTCHA, account lockout policies (balanced against lockout abuse), CDN/WAF protections, graceful degradation, resource limits.

### Elevation of Privilege
**Violates:** Authorisation  
**Ask:** Can an attacker gain capabilities beyond those they were granted?  
**Common controls:** Principle of least privilege, role-based access control, validation of authorisation on every request (not just at login), JWT/token integrity, avoiding client-side trust for privilege decisions.

---

## Trust Boundary Quick Reference

| Boundary Type | Typical Risk Level | Notes |
|--------------|-------------------|-------|
| Internet user → public API | High | Largest trust gap; always prioritise |
| System → third-party vendor | High | External party; limited control over their side |
| Public API → internal service | Medium | Depends on auth model between tiers |
| Internal service → database | Medium | Depends on whether DB is directly accessible |
| Service → service (same VPC) | Lower | Often network-controlled; check auth model |
| Admin console → privileged functions | High | Small user base but high blast radius |

---

## What Makes a Good Threat Model

- The system description is sufficient for any reader to understand the solution
- All major trust boundaries have been considered
- STRIDE has been applied systematically — no category skipped without reason
- Every threat has an inherent risk rating
- Every threat has an explicit response (mitigate / eliminate / transfer / accept)
- Every mitigated threat has a specific, implementable control
- Every control has a residual risk rating
- Controls are linked to implementation tickets
- The document is stored where it can be found, updated, and referenced
- Penetration testers are provided the model when testing begins

---

## Sources

This skill is informed by the following public sources:

- [Threat Modeling Manifesto](https://www.threatmodelingmanifesto.org/)
- [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- Microsoft STRIDE framework
- SAFECode threat modeling guidance
- Adam Shostack threat modeling resources
