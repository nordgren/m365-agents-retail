# Enterprise Architecture for M365 Agents in Multi-Tenant Retail Environments

**Version:** 2.0  
**Date:** 2026-04-02  
**Classification:** Enterprise Architecture Proposal  
**Scope:** Company-Agnostic, Enterprise-Grade

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Reference Architectures](#3-reference-architectures)
4. [Detailed Component Descriptions](#4-detailed-component-descriptions)
5. [Security Architecture & Risks](#5-security-architecture--risks)
6. [Operational Model](#6-operational-model)
7. [Team Structure & RACI Matrix](#7-team-structure--raci-matrix)
8. [Anti-Patterns and Common Risks](#8-anti-patterns-and-common-risks)
9. [Glossary & Terminology](#9-glossary--terminology)

---

## 1. Executive Summary

### Strategic Context

Microsoft 365 Copilot and its extensibility framework—including declarative agents, custom engine agents, and Copilot Studio—represent a fundamental shift in how enterprises operationalize AI within productivity and business workflows. For retail organizations operating across multiple Entra ID tenants (typically an **Enterprise/Corporate tenant** and a **Retail/Store Operations tenant**), this creates both opportunity and architectural complexity.

This document provides enterprise architects, security teams, and transformation leaders with a company-agnostic framework for designing, deploying, operating, and securing M365 agents across multi-tenant retail environments. The guidance addresses identity boundaries, data residency, least-privilege access, cross-tenant orchestration, and DevSecOps practices specific to agent lifecycle management.

### Key Recommendations

1. **Adopt a tiered agent governance model** differentiating personal productivity agents, departmental agents, and mission-critical enterprise agents—each with appropriate security, compliance, and oversight controls.

2. **Implement a zoned governance strategy** with distinct environments (Safe Zone for experimentation, Supported Zone for departmental use, IT-Managed Zone for production) that enforce progressive security and approval gates.

3. **Establish explicit cross-tenant trust relationships** using Microsoft Entra cross-tenant access settings, B2B collaboration, and cross-tenant synchronization where appropriate—avoiding implicit trust assumptions.

4. **Design for zero trust from day one** with identity-based access controls, network segmentation, secrets management via Azure Key Vault, and continuous monitoring integrated into CI/CD pipelines.

5. **Create a dedicated Agent Governance CoE** that sets standards, manages the agent registry, coordinates cross-functional reviews, and provides shared services (templates, testing frameworks, observability) to agent development teams.

### Business Value

- **Operational Efficiency:** Agents automate high-volume, time-sensitive retail workflows (inventory reconciliation, workforce scheduling, customer service) at machine speed with human oversight.
- **Risk Isolation:** Multi-tenant architecture prevents corporate data exposure to store operations and vice versa while enabling controlled collaboration.
- **Scalability:** Federated governance allows rapid agent deployment within guardrails, avoiding central bottlenecks.
- **Compliance:** Built-in controls for data residency, GDPR, and industry-specific regulations.

---

## 2. Architecture Overview

### 2.1 Multi-Tenant Context

Retail enterprises commonly operate multiple Entra ID tenants for valid business, compliance, or historical reasons:

| Tenant Type | Purpose | Typical Users |
|-------------|---------|---------------|
| **Enterprise Tenant** | Corporate functions (finance, HR, merchandising, supply chain, IT) | HQ employees, knowledge workers |
| **Retail Tenant** | Store operations (POS, inventory, workforce management, customer service) | Store associates, district managers, frontline workers |

These tenants form **identity and access management boundaries** that are securely isolated by default. Cross-tenant collaboration requires explicit configuration.

### 2.2 Agent Architecture Layers

M365 agents operate across five architectural layers:

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                             │
│  Teams, Outlook, SharePoint, M365 Apps, Web, Custom Frontends   │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION LAYER                          │
│  M365 Copilot Orchestrator, Copilot Studio, Custom Engine       │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                       AGENT LAYER                               │
│  Declarative Agents, Custom Agents, API Plugins, Connectors     │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    INTEGRATION LAYER                            │
│  Microsoft Graph, Power Platform, Custom APIs, RAG Endpoints    │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                │
│  SharePoint, OneDrive, Dataverse, SQL, Retail Systems (POS,     │
│  Inventory, WFM), Data Lakes, External SaaS                     │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Agent Types

| Agent Type | Description | Build Approach | Orchestration |
|------------|-------------|----------------|---------------|
| **Declarative Agent** | Configuration-based agents using M365 Copilot infrastructure | Instructions + Knowledge + Actions manifest | Microsoft-managed |
| **Custom Engine Agent** | Code-first agents with custom orchestration | Azure Functions, Bot Framework, custom runtime | Developer-managed |
| **Copilot Studio Agent** | Low-code/no-code agents built in Copilot Studio | Visual builder, Power Automate flows | Microsoft-managed |

### 2.4 Cross-Tenant Patterns

Three primary patterns for multi-tenant agent architecture:

1. **Pattern A: Tenant-Isolated Agents** — Agents operate entirely within their home tenant with no cross-tenant access.

2. **Pattern B: Cross-Tenant Integration** — Agents in one tenant access resources in another through explicit B2B collaboration or cross-tenant app registrations.

3. **Pattern C: Federated Agent Mesh** — A governance layer orchestrates agents across multiple tenants with centralized policy enforcement and distributed execution.

---

## 3. Reference Architectures

### 3.1 Architecture A: Corporate-Only Agents (Baseline)

**Scenario:** Agents serve only corporate users within the Enterprise Tenant. No cross-tenant requirements.

**Characteristics:**
- Single Entra ID boundary
- Standard M365 Copilot licensing and governance
- Agents access corporate SharePoint, OneDrive, Microsoft Graph
- Integration with corporate LOB systems via API plugins

**Identity Model:**
- Users authenticate via Enterprise Tenant Entra ID
- Agents execute with user's delegated permissions (OBO flow)
- Service principals for headless/scheduled agent operations

**Data Flow:**
```
User (Enterprise Tenant)
    │
    ▼
M365 Copilot Client
    │
    ▼
M365 Orchestrator ──► Declarative Agent
    │                      │
    ▼                      ▼
Microsoft Graph ◄───► API Plugin ──► Corporate LOB System
    │
    ▼
SharePoint / OneDrive / Dataverse
```

**Security Controls:**
- Conditional Access policies for Copilot access
- Data Loss Prevention (DLP) policies on connectors
- Sensitivity labels on grounded content
- Audit logging to Microsoft Purview

**When to Use:**
- Pilot/POC phase
- Corporate-only use cases (HR, Finance, IT)
- No requirement to integrate with retail operations

---

### 3.2 Architecture B: Dual-Tenant Agents (Corporate + Retail)

**Scenario:** Agents operate across Enterprise and Retail tenants, enabling use cases like:
- Corporate merchandising agents querying store inventory
- Retail workforce agents accessing corporate HR policies
- Cross-tenant analytics and reporting

**Characteristics:**
- Two Entra ID tenants with explicit trust relationship
- Cross-tenant access settings configured bidirectionally
- B2B collaboration users or cross-tenant sync for identity
- Careful data classification and access controls

**Identity Model:**

*Option 1: B2B Collaboration*
- Corporate users invited as B2B guests to Retail Tenant
- Retail users invited as B2B guests to Enterprise Tenant
- Agents use delegated permissions of the signed-in user

*Option 2: Cross-Tenant Synchronization*
- Automated sync of user objects between tenants
- Users exist as B2B collaboration objects in target tenant
- Enables seamless collaboration with consistent identity

*Option 3: Multi-Tenant App Registration*
- Application registered in Enterprise Tenant as multi-tenant app
- Consented in Retail Tenant via admin consent flow
- Agents authenticate as app (client credentials) for cross-tenant access

**Data Flow:**
```
┌──────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE TENANT                              │
│                                                                   │
│  User ──► M365 Copilot ──► Corporate Agent                       │
│                │                  │                               │
│                ▼                  ▼                               │
│         Microsoft Graph    API Plugin ──► Corporate LOB          │
│                │                                                  │
└────────────────┼──────────────────────────────────────────────────┘
                 │
    Cross-Tenant Access (B2B / App Consent)
                 │
┌────────────────┼──────────────────────────────────────────────────┐
│                ▼                    RETAIL TENANT                  │
│                                                                    │
│         Retail Agent ◄──── API Plugin                             │
│              │                  │                                  │
│              ▼                  ▼                                  │
│       Retail Graph API    Retail Systems (POS, Inventory, WFM)    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Security Controls:**
- Cross-tenant access policies (inbound/outbound)
- Conditional Access requiring compliant devices
- Token lifetime policies limiting cross-tenant sessions
- Network segmentation between tenant workloads
- Separate Key Vault instances per tenant for secrets
- Cross-tenant audit log correlation

**Implementation Requirements:**

1. **Cross-Tenant Access Settings:**
   - Enterprise Tenant: Configure outbound settings to trust Retail Tenant
   - Retail Tenant: Configure inbound settings to allow Enterprise Tenant access
   - Define which applications and users can collaborate

2. **Application Registration:**
   - Register agent application in primary (Enterprise) tenant
   - Set application to "Accounts in any organizational directory" (multi-tenant)
   - Request admin consent in Retail Tenant
   - Use certificate-based authentication (no client secrets)

3. **API Permissions:**
   - Request minimum necessary Microsoft Graph scopes
   - Use delegated permissions where user context required
   - Use application permissions only for service-to-service

**When to Use:**
- Corporate-retail integration scenarios
- Centralized governance with distributed execution
- Controlled data sharing between tenants

---

### 3.3 Architecture C: Federated Agent Mesh

**Scenario:** Large enterprise with multiple tenants (regional, M&A legacy, regulatory separation) requiring unified agent governance with distributed execution.

**Characteristics:**
- Three or more Entra ID tenants
- Centralized Agent Governance Layer
- Federated execution in each tenant
- Policy-driven orchestration

**Components:**

1. **Agent Governance Layer (Centralized)**
   - Agent Registry (inventory of all agents across tenants)
   - Policy Engine (governance rules, approval workflows)
   - Monitoring Hub (aggregated telemetry, security events)
   - Template Library (approved agent patterns)

2. **Tenant Agent Runtimes (Distributed)**
   - Local Copilot Studio environments
   - Tenant-specific API plugins
   - Local data access within tenant boundary

3. **Cross-Tenant Orchestration**
   - Event-driven messaging (Azure Service Bus / Event Grid)
   - API Gateway with tenant routing
   - Federated identity (managed identities + workload identity federation)

**Architecture Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                   AGENT GOVERNANCE LAYER                            │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │   Agent     │  │   Policy     │  │  Monitoring  │               │
│  │  Registry   │  │   Engine     │  │     Hub      │               │
│  └─────────────┘  └──────────────┘  └──────────────┘               │
│                         │                                           │
│              Policy Distribution / Event Routing                    │
└─────────────────────────┼───────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  ENTERPRISE   │  │    RETAIL     │  │   REGIONAL    │
│    TENANT     │  │    TENANT     │  │    TENANT     │
│               │  │               │  │               │
│  ┌─────────┐  │  │  ┌─────────┐  │  │  ┌─────────┐  │
│  │ Agents  │  │  │  │ Agents  │  │  │  │ Agents  │  │
│  └────┬────┘  │  │  └────┬────┘  │  │  └────┬────┘  │
│       │       │  │       │       │  │       │       │
│  ┌────▼────┐  │  │  ┌────▼────┐  │  │  ┌────▼────┐  │
│  │  Data   │  │  │  │  Data   │  │  │  │  Data   │  │
│  │ Sources │  │  │  │ Sources │  │  │  │ Sources │  │
│  └─────────┘  │  │  └─────────┘  │  │  └─────────┘  │
└───────────────┘  └───────────────┘  └───────────────┘
```

**Identity Model:**
- Workload Identity Federation for cross-tenant service auth
- Managed Identities in each tenant for local execution
- No shared secrets across tenant boundaries
- Centralized identity governance with federated execution

**Security Controls:**
- Zero trust network architecture
- Private endpoints for cross-tenant APIs
- mTLS for service-to-service communication
- Centralized SIEM/SOAR with tenant-specific data residency
- Automated compliance scanning across tenants

**When to Use:**
- Large enterprises with 3+ tenants
- M&A scenarios with tenant consolidation roadmap
- Regulatory requirements for data localization
- Global retail operations with regional autonomy

---

### 3.4 Architecture Decision Criteria

Use this decision matrix to select the appropriate reference architecture:

| Factor | Pattern A (Corporate-Only) | Pattern B (Dual-Tenant) | Pattern C (Federated Mesh) |
|--------|---------------------------|------------------------|---------------------------|
| **Tenant Count** | 1 | 2 | 3+ |
| **Complexity** | Low | Medium | High |
| **Time to Deploy** | 2-4 weeks | 2-3 months | 6-12 months |
| **Cost (Implementation)** | $ | $$ | $$$ |
| **Cost (Operations)** | $ | $$ | $$$ |
| **Skills Required** | M365 Admin | M365 + Identity Specialist | EA + Identity + Platform |
| **Governance Overhead** | Low | Medium | High |
| **Cross-Tenant Data** | N/A | Controlled | Orchestrated |
| **Best For** | Pilots, corporate-only | Corporate-retail integration | Global enterprises |

#### Decision Tree

```
                    ┌─────────────────────────────┐
                    │ How many Entra ID tenants?  │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
           1 tenant            2 tenants            3+ tenants
              │                    │                    │
              ▼                    ▼                    ▼
      ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
      │   Pattern A   │    │ Need cross-   │    │   Pattern C   │
      │ Corporate-Only│    │ tenant data?  │    │Federated Mesh │
      └───────────────┘    └───────┬───────┘    └───────────────┘
                                   │
                        ┌──────────┴──────────┐
                        │                     │
                       Yes                   No
                        │                     │
                        ▼                     ▼
                ┌───────────────┐     ┌───────────────┐
                │   Pattern B   │     │   Pattern A   │
                │  Dual-Tenant  │     │  per tenant   │
                └───────────────┘     └───────────────┘
```

---

### 3.5 Failure Modes & Remediation

#### Pattern A: Corporate-Only Failure Modes

| Failure Mode | Root Cause | Impact | Detection | Remediation |
|--------------|------------|--------|-----------|-------------|
| **Oversharing exposure** | Inadequate SharePoint permissions | Sensitive data in Copilot responses | Purview alerts; user reports | Implement DLP; remediate permissions |
| **Agent unavailable** | Copilot service outage | Business process disruption | M365 Service Health; synthetic monitoring | Failover to manual process; incident comms |
| **Performance degradation** | Token limit exceeded; slow API plugins | Poor user experience | Response time monitoring | Optimize grounding; cache API responses |
| **Data staleness** | Knowledge source not updated | Incorrect agent responses | User feedback; accuracy metrics | Automate refresh; alert on staleness |
| **Permission creep** | Over-provisioned API permissions | Security risk; compliance violation | Quarterly access review | Implement least privilege; PIM |

#### Pattern B: Dual-Tenant Failure Modes

| Failure Mode | Root Cause | Impact | Detection | Remediation |
|--------------|------------|--------|-----------|-------------|
| **Cross-tenant auth failure** | Expired certificates; consent revoked | Cross-tenant agents non-functional | Auth error monitoring; synthetic tests | Rotate certificates; re-consent |
| **Tenant trust drift** | Cross-tenant settings changed | Unexpected access denial/grant | Config change alerts; periodic audit | Version control settings; change process |
| **Data leakage** | Cross-tenant oversharing | Compliance violation; data breach | DLP alerts; audit log correlation | Immediate access revocation; incident response |
| **Latency spikes** | Cross-tenant network issues | Poor user experience | End-to-end latency monitoring | Private endpoints; geographic optimization |
| **Consent sprawl** | Unmanaged multi-tenant app consents | Security risk | Consent inventory review | Consent governance process |

#### Pattern C: Federated Mesh Failure Modes

| Failure Mode | Root Cause | Impact | Detection | Remediation |
|--------------|------------|--------|-----------|-------------|
| **Governance layer outage** | Central infrastructure failure | No policy enforcement | Health monitoring; redundancy | Active-passive failover; DR plan |
| **Policy sync failure** | Distribution mechanism broken | Stale policies in tenants | Policy version tracking | Manual policy push; sync repair |
| **Cross-tenant cascade** | Failure propagates across tenants | Multi-tenant outage | Correlation analysis | Circuit breakers; tenant isolation |
| **Registry inconsistency** | Agent registry out of sync | Ghost agents; missing agents | Reconciliation checks | Automated sync; manual cleanup |
| **Cost explosion** | Uncontrolled agent proliferation | Budget overrun | Cost monitoring; alerts | Quotas; chargeback; cleanup |

---

### 3.6 Cost Model & Implications

#### Cost Categories

| Category | Components | Billing Model |
|----------|------------|---------------|
| **M365 Copilot Licensing** | Per-user license ($30/user/month typical) | Per user, annual |
| **Copilot Studio Consumption** | Copilot Credits for agent interactions | Pay-as-you-go or prepaid packs |
| **Azure Infrastructure** | Key Vault, API Management, Functions | Consumption-based |
| **Development** | CoE team, development resources | FTE / contractor |
| **Operations** | Monitoring, support, maintenance | FTE / MSP |

#### Copilot Credit Consumption Reference

| Agent Feature | Credits/Interaction | Typical Volume | Monthly Credits |
|---------------|--------------------|--------------------|-----------------|
| Classic answer | 1 | 10,000 | 10,000 |
| Generative answer | 2 | 5,000 | 10,000 |
| Agent action | 5 | 3,000 | 15,000 |
| Tenant graph grounding | 10 | 2,000 | 20,000 |
| Generative + graph | 12 | 2,000 | 24,000 |

*Note: Copilot Credits for M365 Copilot-licensed users in B2E scenarios are included in license (fair use limits apply).*

#### Cost by Architecture Pattern

| Cost Element | Pattern A | Pattern B | Pattern C |
|--------------|-----------|-----------|-----------|
| **Implementation (one-time)** | $50-150K | $200-500K | $500K-2M |
| **CoE Team (annual)** | 3-5 FTEs | 8-12 FTEs | 15-25 FTEs |
| **Infrastructure (annual)** | $20-50K | $100-300K | $300K-1M |
| **Copilot Licenses** | Per user | Per user both tenants | Per user all tenants |
| **Copilot Credits** | Moderate | Higher (cross-tenant) | Highest (mesh overhead) |

#### Cost Optimization Strategies

1. **Right-size agent features:** Use classic answers where generative not needed
2. **Optimize grounding:** Limit knowledge sources; tune retrieval
3. **Cache API responses:** Reduce external API calls
4. **Chargeback model:** Allocate costs to consuming business units
5. **Consumption monitoring:** Alert on anomalies; enforce quotas
6. **License optimization:** Review Copilot license utilization quarterly

---

### 3.7 Operational Runbooks

#### Runbook 1: Agent Deployment

**Trigger:** New agent ready for production deployment

**Prerequisites:**
- [ ] Agent design approved (Tier 3+: CoE review complete)
- [ ] Security assessment passed
- [ ] CI/CD pipeline configured
- [ ] Monitoring and alerting configured
- [ ] Documentation complete
- [ ] Rollback plan documented

**Steps:**

1. **Pre-Deployment Validation**
   ```
   ☐ Verify all prerequisites complete
   ☐ Confirm deployment window with stakeholders
   ☐ Validate target environment health
   ☐ Backup current state (if upgrade)
   ```

2. **Deployment Execution**
   ```
   ☐ Execute CI/CD pipeline (Stage → Production)
   ☐ Monitor pipeline execution
   ☐ Validate deployment artifacts
   ```

3. **Post-Deployment Validation**
   ```
   ☐ Execute smoke tests
   ☐ Verify agent responds correctly
   ☐ Confirm monitoring/alerting active
   ☐ Test rollback capability (dry run)
   ```

4. **Go-Live Confirmation**
   ```
   ☐ Notify stakeholders of successful deployment
   ☐ Update agent registry status
   ☐ Archive deployment artifacts
   ☐ Close deployment ticket
   ```

**Rollback Trigger:** Any critical failure in smoke tests

**Rollback Steps:**
```
☐ Announce rollback decision
☐ Execute rollback pipeline
☐ Verify previous version restored
☐ Notify stakeholders
☐ Document rollback reason
☐ Schedule post-mortem
```

---

#### Runbook 2: Cross-Tenant Certificate Rotation

**Trigger:** Certificate expiration approaching (30 days before)

**Frequency:** Annual (or as defined by security policy)

**Steps:**

1. **Preparation (Week 1)**
   ```
   ☐ Generate new certificate in Key Vault (Enterprise Tenant)
   ☐ Export public key for Retail Tenant registration
   ☐ Schedule change window with both tenant admins
   ☐ Notify affected teams
   ```

2. **Enterprise Tenant Update (Change Window)**
   ```
   ☐ Add new certificate to app registration (do not remove old)
   ☐ Validate new certificate is available
   ☐ Update agent configurations to use new certificate
   ```

3. **Retail Tenant Update (Change Window)**
   ```
   ☐ Update service principal with new certificate
   ☐ Test cross-tenant authentication with new cert
   ☐ Validate all agents functional
   ```

4. **Cleanup (Post-Validation)**
   ```
   ☐ Monitor for 7 days with both certificates active
   ☐ Remove old certificate from app registration
   ☐ Remove old certificate from Key Vault (after retention)
   ☐ Update documentation with new expiration date
   ☐ Schedule next rotation
   ```

---

#### Runbook 3: Agent Incident Response

**Trigger:** Agent incident reported or detected

**Severity Levels:**

| Severity | Definition | Response Time | Escalation |
|----------|------------|---------------|------------|
| **P1** | Agent producing harmful/incorrect critical outputs | 15 min | Security + CoE Director |
| **P2** | Agent unavailable affecting business operations | 30 min | AI Operations Lead |
| **P3** | Agent degraded performance or intermittent issues | 4 hours | AI Operations |
| **P4** | Minor issues; cosmetic; workaround available | 24 hours | AI Operations |

**P1/P2 Response Steps:**

1. **Immediate Response (0-15 min)**
   ```
   ☐ Acknowledge incident
   ☐ Assess severity and impact
   ☐ If data exposure: Disable agent immediately
   ☐ If availability: Check M365 Service Health
   ☐ Notify incident commander
   ```

2. **Investigation (15-60 min)**
   ```
   ☐ Gather evidence (logs, user reports, screenshots)
   ☐ Identify root cause category
   ☐ Determine affected scope (users, data, tenants)
   ☐ Engage SMEs as needed
   ```

3. **Containment (Concurrent)**
   ```
   ☐ Disable agent if risk continues
   ☐ Revoke cross-tenant access if breach suspected
   ☐ Preserve evidence for forensics
   ☐ Communicate status to stakeholders
   ```

4. **Resolution**
   ```
   ☐ Implement fix or workaround
   ☐ Test fix in non-production
   ☐ Deploy fix with approval
   ☐ Verify resolution
   ☐ Re-enable agent (staged rollout)
   ```

5. **Post-Incident**
   ```
   ☐ Conduct post-mortem (within 72 hours)
   ☐ Document root cause and remediation
   ☐ Update runbooks if needed
   ☐ Implement preventive measures
   ☐ Close incident
   ```

---

#### Runbook 4: Cost Anomaly Response

**Trigger:** Cost monitoring alert (>20% deviation from baseline)

**Steps:**

1. **Detection & Triage**
   ```
   ☐ Review alert details (which agents, tenants, time period)
   ☐ Identify consumption spike source
   ☐ Determine if legitimate usage or anomaly
   ```

2. **Investigation**
   ```
   ☐ Analyze agent usage patterns
   ☐ Review recent deployments/changes
   ☐ Check for runaway processes or infinite loops
   ☐ Identify responsible business unit
   ```

3. **Mitigation**
   ```
   ☐ If malicious/errant: Disable agent immediately
   ☐ If legitimate spike: Communicate to budget owner
   ☐ If optimization needed: Engage CoE engineering
   ```

4. **Resolution**
   ```
   ☐ Implement fix (code optimization, caching, limits)
   ☐ Update cost baseline if legitimate growth
   ☐ Adjust alerts as needed
   ☐ Document lessons learned
   ```

---

## 4. Detailed Component Descriptions

### 4.1 Identity & Access Components

#### 4.1.1 Entra ID (Azure AD)

**Role:** Primary identity provider for all M365 agent authentication and authorization.

**Configuration Requirements:**
- Conditional Access policies for Copilot/agent access
- Named locations for network-based access control
- Risk-based access policies (sign-in risk, user risk)
- Privileged Identity Management (PIM) for admin roles

**Multi-Tenant Considerations:**
- Cross-tenant access settings for B2B collaboration
- Cross-tenant synchronization for seamless user experience
- Multi-tenant organization (MTO) for simplified collaboration

#### 4.1.2 App Registrations

**Role:** Define applications (agents) that can authenticate to Entra ID and access resources.

**Best Practices:**
- Use certificate-based credentials (no client secrets in production)
- Define minimum necessary API permissions
- Implement application-level Conditional Access
- Regular credential rotation via automation

**Multi-Tenant App Pattern:**
```
Enterprise Tenant                    Retail Tenant
┌─────────────────────┐              ┌─────────────────────┐
│  App Registration   │              │  Service Principal  │
│  (Home Tenant)      │              │  (Consented Copy)   │
│                     │   Consent    │                     │
│  - Client ID        │ ──────────►  │  - Same Client ID   │
│  - Certificates     │   Flow       │  - Local Perms      │
│  - API Permissions  │              │  - Audit Logs       │
└─────────────────────┘              └─────────────────────┘
```

#### 4.1.3 Managed Identities

**Role:** Eliminate secrets for Azure-hosted agent components.

**Types:**
- **System-Assigned:** Tied to specific Azure resource lifecycle
- **User-Assigned:** Shared across resources, independent lifecycle

**Cross-Tenant Pattern:**
- Use Workload Identity Federation for cross-tenant scenarios
- Federate user-assigned managed identity with multi-tenant app registration
- Eliminates client secret management for cross-tenant auth

### 4.2 Agent Runtime Components

#### 4.2.1 Declarative Agents

**Architecture:**
- Defined via JSON manifest (instructions, knowledge sources, actions)
- Executed by M365 Copilot orchestrator
- Inherits M365 security, compliance, governance

**Limits:**
| Limit | Value |
|-------|-------|
| Grounding records | 50 items |
| Plugin response | 25 items |
| Token limit | 4,096 |
| Timeout | 45 seconds |

**Best Fit:**
- Information retrieval and summarization
- Simple, single-step workflows
- M365-centric data access

**Poor Fit:**
- Complex multi-step orchestration
- Large data processing
- Custom AI models

#### 4.2.2 Custom Engine Agents

**Architecture:**
- Code-first implementation (C#, Python, JavaScript)
- Developer-managed orchestration and hosting
- Can use any AI model (Azure OpenAI, OpenAI, custom)

**Hosting Options:**
- Azure Functions (serverless)
- Azure Container Apps (containers)
- Azure Kubernetes Service (enterprise scale)

**When to Use:**
- Complex reasoning loops
- Integration with non-M365 systems
- Custom security or compliance requirements

#### 4.2.3 Copilot Studio Agents

**Architecture:**
- Low-code visual builder
- Power Platform foundation (Dataverse, Power Automate)
- Microsoft-managed orchestration

**Governance:**
- Environment-based isolation
- DLP policies on connectors
- Tenant-level and environment-level controls

### 4.3 Integration Components

#### 4.3.1 API Plugins

**Role:** Connect agents to external APIs and LOB systems.

**Types:**
- **OpenAPI-based plugins:** Standard REST API integration
- **Power Platform connectors:** Pre-built and custom connectors
- **Microsoft Graph connectors:** Index external content in M365

**Security:**
- OAuth 2.0 for authentication
- API key management via Azure Key Vault
- Rate limiting and throttling
- Response validation and sanitization

#### 4.3.2 RAG (Retrieval-Augmented Generation)

**Role:** Ground agent responses in enterprise knowledge.

**Components:**
- **Knowledge Sources:** SharePoint, OneDrive, Dataverse, custom indexes
- **Vector Stores:** Azure AI Search, Cosmos DB (vCore)
- **Embedding Models:** Azure OpenAI embeddings

**Security Considerations:**
- Respect source document permissions (ACL trimming)
- Data classification inheritance
- Prompt injection protection

### 4.4 Retail-Specific Components

#### 4.4.1 Digital Retail Stack Integration

| System | Agent Use Cases | Integration Pattern |
|--------|-----------------|---------------------|
| **POS Systems** | Transaction queries, returns processing | API plugin with read-only access |
| **Inventory Management** | Stock levels, allocation, transfers | Real-time API with event triggers |
| **Workforce Management** | Schedule queries, shift swaps, time-off | Bidirectional API with approval workflows |
| **Customer Data Platform** | Customer profiles, preferences, history | Read-only with PII masking |
| **Digital Retail Offers** | Promotion rules, pricing, campaigns | Read-only with caching |

#### 4.4.2 Model Context Protocol (MCP)

**Role:** Provides agents with shared, enterprise-grade understanding of products, inventory, pricing, policies, and customer intent.

**Architecture:**
- Standardized protocol for agent-to-data communication
- Supports Microsoft and third-party MCP servers
- Enables consistent context across agent ecosystem

---

## 5. Security Architecture & Risks

### 5.1 Zero Trust Principles for Agents

| Principle | Application to Agents |
|-----------|----------------------|
| **Verify Explicitly** | Authenticate every agent request; validate user and app identity; verify permissions on each resource access |
| **Least Privilege** | Request minimum API scopes; use delegated permissions where possible; scope data access to job function |
| **Assume Breach** | Segment agent networks; encrypt data in transit and at rest; monitor for anomalies; plan for incident response |

### 5.2 Identity Security

#### 5.2.1 Authentication Flows

| Scenario | Recommended Flow | Security Considerations |
|----------|------------------|------------------------|
| User-interactive agent | OAuth 2.0 Authorization Code + PKCE | MFA enforcement via Conditional Access |
| Service-to-service | Client Credentials with certificate | Managed Identity preferred; rotate certificates |
| Cross-tenant | Workload Identity Federation | No secrets cross boundary; short-lived tokens |
| Delegated access | On-Behalf-Of (OBO) | Validate downstream permissions; token binding |

#### 5.2.2 Cross-Tenant Access Controls

**Inbound Settings (Retail Tenant accepting Enterprise Tenant):**
- Specify trusted organizations (Enterprise Tenant ID)
- Define which users can be invited
- Require MFA / compliant device
- Limit accessible applications

**Outbound Settings (Enterprise Tenant accessing Retail Tenant):**
- Control which users can access external tenants
- Restrict accessible applications
- Enforce security requirements

### 5.3 Data Security

#### 5.3.1 Data Classification

| Classification | Agent Access | Controls |
|---------------|--------------|----------|
| Public | Allowed | Standard logging |
| Internal | Allowed with auth | Access logging, DLP |
| Confidential | Restricted | Sensitivity labels, encryption, auditing |
| Highly Confidential | Prohibited | Block via DLP, alerts |

#### 5.3.2 Data Loss Prevention

- **Connector DLP:** Block or allow specific connectors per environment
- **Sensitivity Labels:** Agents inherit label restrictions from source content
- **Prompt Guardrails:** Content filtering on agent inputs/outputs

### 5.4 Network Security

#### 5.4.1 Architecture Pattern

```
Internet
    │
    ▼
┌─────────────────────────────────────┐
│  Azure Front Door / App Gateway     │
│  (WAF, DDoS Protection)             │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│  API Management (APIM)              │
│  (Rate Limiting, Auth, Routing)     │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│  Private Endpoints                  │
│  (Agent Runtime, Data Sources)      │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│  Virtual Network                    │
│  (Segmented Subnets, NSGs)          │
└─────────────────────────────────────┘
```

#### 5.4.2 Cross-Tenant Network Isolation

- Use Private Link for cross-tenant API access
- Implement network peering only where required
- Maintain separate VNets per tenant
- Use Azure Firewall for egress control

### 5.5 Secrets Management

| Secret Type | Storage | Rotation | Access |
|-------------|---------|----------|--------|
| API Keys | Azure Key Vault | Automated (90 days) | Managed Identity |
| Certificates | Azure Key Vault | Automated (annual) | Managed Identity |
| Connection Strings | Key Vault Reference | Per deployment | App Configuration |
| OAuth Tokens | Runtime memory only | Per request | Not persisted |

### 5.6 Security Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Cross-Prompt Injection (XPIA)** | Malicious input manipulates agent behavior | Content filtering, input validation, grounding separation |
| **Data Exfiltration** | Sensitive data leaked via agent responses | DLP policies, output filtering, audit logging |
| **Privilege Escalation** | Agent gains unintended access | Least privilege, regular permission reviews |
| **Cross-Tenant Data Leakage** | Data from one tenant exposed to another | Strict tenant isolation, permission scoping, ACL enforcement |
| **Prompt Injection** | Adversarial prompts override instructions | System prompt protection, input sanitization |
| **Supply Chain Attack** | Malicious API plugin or connector | Plugin vetting, signed packages, secure registry |

---

## 6. Operational Model

### 6.1 Agent Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Design  │───►│  Build   │───►│   Test   │───►│  Deploy  │───►│  Operate │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                                      │
┌──────────┐    ┌──────────┐                                          │
│  Retire  │◄───│  Update  │◄─────────────────────────────────────────┘
└──────────┘    └──────────┘
```

### 6.2 Environment Strategy (Zoned Governance)

| Zone | Purpose | Governance | Data Access | Approval |
|------|---------|------------|-------------|----------|
| **Safe Zone** | Experimentation, POC | Minimal | Sample/test data only | Self-service |
| **Supported Zone** | Departmental use | Standard | Departmental data | Team approval |
| **IT-Managed Zone** | Production | Full | Enterprise data | CAB approval |

### 6.3 DevSecOps Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CI/CD PIPELINE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Source ──► Build ──► Security ──► Test ──► Stage ──► Production   │
│    │          │         Scan        │         │           │        │
│    │          │          │          │         │           │        │
│  Git Repo  Compile    SAST/DAST   Unit/     UAT        Canary/     │
│  + Lint    + Package  + Secrets   Integration          Blue-Green  │
│            + Sign     + Deps                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Pipeline Stages:**

1. **Source:** Git-based version control with branch protection
2. **Build:** Compile agent manifest, package plugins, generate artifacts
3. **Security Scan:** 
   - SAST: Static analysis of agent code
   - Dependency scanning for vulnerabilities
   - Secrets detection
   - AI Red Team Agent scanning (where available)
4. **Test:**
   - Unit tests for API plugins
   - Integration tests for agent flows
   - Performance/load testing
   - Adversarial testing for prompt injection
5. **Stage:** Deploy to staging environment, UAT validation
6. **Production:** Canary or blue-green deployment, feature flags

### 6.4 Monitoring & Observability

| Layer | Tool | Metrics |
|-------|------|---------|
| **Agent Performance** | Copilot Studio Analytics | Response time, completion rate, user satisfaction |
| **Application** | Azure Application Insights | Errors, latency, dependencies |
| **Security** | Microsoft Defender for Cloud | Threats, vulnerabilities, compliance |
| **Audit** | Microsoft Purview | Data access, admin actions, policy violations |
| **Cross-Tenant** | Azure Sentinel (centralized) | Correlated security events across tenants |

### 6.5 Incident Response

**Agent-Specific Incident Types:**
- Agent producing incorrect/harmful outputs
- Data leakage via agent responses
- Agent unavailability impacting business
- Cross-tenant access violation
- Prompt injection attack detected

**Response Workflow:**
1. **Detect:** Alert from monitoring/user report
2. **Triage:** Assess severity, identify affected agents
3. **Contain:** Disable agent, revoke access if needed
4. **Investigate:** Review logs, identify root cause
5. **Remediate:** Fix issue, deploy patch
6. **Review:** Post-incident analysis, update controls

---

## 7. Team Structure & RACI Matrix

### 7.1 Recommended Teams

#### 7.1.1 AI Engineering Team (Agent Builders)

**Mission:** Design, build, and maintain M365 agents that deliver business value while adhering to enterprise standards.

**Key Responsibilities:**
- Develop declarative agents, custom engine agents, Copilot Studio agents
- Implement API plugins and connectors
- Write and maintain agent test suites
- Optimize agent performance and reliability
- Collaborate with business stakeholders on requirements

**Required Competencies:**
- Microsoft 365 Copilot extensibility
- Copilot Studio, Power Platform
- Azure AI services (OpenAI, AI Search)
- API design and development
- Prompt engineering

**Daily/Weekly Workflows:**
- Daily standups with sprint team
- Weekly agent performance reviews
- Bi-weekly security scanning
- Monthly backlog refinement

**Governance Participation:**
- Agent Design Reviews
- Security Review Board (presenting)

**Handover Processes:**
- Deploy to production via CI/CD pipeline
- Handoff to AI Operations for monitoring
- Documentation in agent registry

---

#### 7.1.2 M365 Platform Engineering

**Mission:** Provide and maintain the M365 platform foundation that enables secure, compliant agent deployment.

**Key Responsibilities:**
- Manage M365 tenant configuration
- Configure Copilot licensing and access
- Maintain SharePoint/OneDrive infrastructure
- Implement M365 security policies
- Enable cross-tenant collaboration settings

**Required Competencies:**
- Microsoft 365 administration
- SharePoint Online, OneDrive
- Microsoft Graph API
- M365 security and compliance
- PowerShell automation

**Daily/Weekly Workflows:**
- Daily platform health monitoring
- Weekly configuration reviews
- Monthly security posture assessments
- Quarterly license optimization

**Governance Participation:**
- Platform Governance Board
- Security Review Board
- Change Advisory Board

**Handover Processes:**
- Environment provisioning to AI Engineering
- Cross-tenant settings to IAM team
- Incident escalation from Operations

---

#### 7.1.3 Identity & Access Management (IAM)

**Mission:** Ensure secure, compliant identity and access controls for all agents and their users across tenants.

**Key Responsibilities:**
- Manage Entra ID configuration
- Configure Conditional Access policies
- Administer cross-tenant access settings
- Manage app registrations and service principals
- Implement Privileged Identity Management

**Required Competencies:**
- Microsoft Entra ID
- OAuth 2.0, OIDC, SAML
- Conditional Access design
- Multi-tenant identity patterns
- Zero trust architecture

**Daily/Weekly Workflows:**
- Daily access review alerts
- Weekly cross-tenant access audits
- Monthly Conditional Access policy reviews
- Quarterly identity governance assessments

**Governance Participation:**
- Identity Governance Board
- Security Review Board
- Cross-Tenant Access Council

**Handover Processes:**
- App registrations to AI Engineering
- Cross-tenant config to M365 Platform
- Incident response with Security/SecOps

---

#### 7.1.4 Security / SecOps

**Mission:** Protect the organization's agents, data, and users from security threats through proactive defense and rapid response.

**Key Responsibilities:**
- Security architecture review for agents
- Penetration testing and red teaming
- Security monitoring and alerting
- Incident response and forensics
- Compliance validation

**Required Competencies:**
- Cloud security (Azure, M365)
- Threat modeling and risk assessment
- SIEM/SOAR (Microsoft Sentinel)
- Incident response
- AI/ML security considerations

**Daily/Weekly Workflows:**
- Continuous security monitoring
- Daily threat intelligence review
- Weekly vulnerability scanning
- Monthly penetration testing (rotating agents)

**Governance Participation:**
- Security Review Board (leading)
- Incident Response Team
- Risk Committee

**Handover Processes:**
- Security clearance to AI Engineering for deployment
- Threat intel to IAM for policy updates
- Incident handoff to all affected teams

---

#### 7.1.5 Retail IT / Store Digital

**Mission:** Enable store operations through technology while ensuring reliability, security, and alignment with retail business needs.

**Key Responsibilities:**
- Manage retail-specific systems (POS, inventory, WFM)
- Configure Retail Tenant M365 environment
- Support store associate technology needs
- Integrate retail systems with agents
- Ensure store network connectivity

**Required Competencies:**
- Retail technology systems
- Store operations understanding
- Microsoft 365 (Frontline Workers)
- Retail data and analytics
- Field support and deployment

**Daily/Weekly Workflows:**
- Daily store operations support
- Weekly retail system health checks
- Monthly agent usage reviews with stores
- Quarterly retail technology roadmap planning

**Governance Participation:**
- Retail Technology Council
- Cross-Tenant Access Council
- Agent Design Reviews (retail agents)

**Handover Processes:**
- Retail integration requirements to AI Engineering
- Store feedback to Product Management
- Escalations to Enterprise IT

---

#### 7.1.6 Enterprise Architecture

**Mission:** Define and govern the enterprise architecture that ensures agents align with business strategy and technology standards.

**Key Responsibilities:**
- Define agent architecture standards
- Maintain enterprise architecture repository
- Review and approve significant agent designs
- Ensure alignment with business capabilities
- Guide technology investment decisions

**Required Competencies:**
- Enterprise architecture frameworks (TOGAF, etc.)
- Business capability modeling
- Technology portfolio management
- Cloud architecture (Azure, M365)
- AI/ML architecture patterns

**Daily/Weekly Workflows:**
- Weekly architecture review sessions
- Monthly technology radar updates
- Quarterly architecture governance reviews
- Annual technology strategy planning

**Governance Participation:**
- Architecture Review Board (leading)
- Technology Investment Committee
- Agent Governance CoE

**Handover Processes:**
- Architecture decisions to all teams
- Standards to Platform Engineering
- Strategy input to Executive leadership

---

#### 7.1.7 Data Platforms

**Mission:** Provide the data infrastructure and governance that enables agents to access, process, and protect enterprise data.

**Key Responsibilities:**
- Manage data lakes, warehouses, and marts
- Implement data governance and quality
- Enable RAG and knowledge retrieval
- Ensure data security and privacy
- Support analytics and AI workloads

**Required Competencies:**
- Data engineering (Azure Data Factory, Databricks, Synapse)
- Data governance (Microsoft Purview)
- Vector databases and AI Search
- Data security and privacy
- SQL and NoSQL databases

**Daily/Weekly Workflows:**
- Daily data pipeline monitoring
- Weekly data quality reviews
- Monthly data governance assessments
- Quarterly data strategy reviews

**Governance Participation:**
- Data Governance Council
- Architecture Review Board
- Agent Design Reviews (data-intensive agents)

**Handover Processes:**
- Data access provisioning to AI Engineering
- Data quality reports to business stakeholders
- Security incidents to SecOps

---

#### 7.1.8 AI Operations (AIOps)

**Mission:** Ensure reliable, efficient, and compliant operation of all production agents through monitoring, optimization, and support.

**Key Responsibilities:**
- Monitor agent health and performance
- Manage agent capacity and scaling
- Handle production incidents
- Optimize agent costs and efficiency
- Maintain production environment stability

**Required Competencies:**
- SRE/DevOps practices
- Azure monitoring (Application Insights, Monitor)
- Incident management
- Performance optimization
- AI/ML operations (MLOps)

**Daily/Weekly Workflows:**
- 24/7 production monitoring
- Daily performance reviews
- Weekly capacity planning
- Monthly cost optimization reviews

**Governance Participation:**
- Operations Review Board
- Incident Response Team
- Change Advisory Board

**Handover Processes:**
- Deployment acceptance from AI Engineering
- Incident escalation to appropriate teams
- Performance insights to Architecture

---

### 7.2 RACI Matrices

#### 7.2.1 Agent Creation

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Gather requirements | R | C | C | C | C | C | C | I |
| Design agent architecture | R | C | C | A | C | A | C | I |
| Build agent | R | I | I | I | I | I | C | I |
| Security review | C | I | C | A | I | C | I | I |
| Documentation | R | I | I | C | I | I | I | I |

#### 7.2.2 Agent Deployment

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Environment provisioning | C | R | C | C | C | I | I | I |
| CI/CD pipeline execution | R | C | I | C | I | I | I | C |
| Pre-prod testing | R | C | I | C | C | I | C | C |
| Production approval | C | C | C | A | C | C | I | C |
| Production deployment | R | C | I | I | I | I | I | A |
| Post-deployment validation | R | C | I | C | C | I | I | R |

#### 7.2.3 Identity Setup

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| App registration | C | C | R | C | I | I | I | I |
| API permissions | C | C | R | A | I | I | C | I |
| Cross-tenant access | I | C | R | A | C | I | I | I |
| Conditional Access | I | C | R | A | I | I | I | I |
| Service principal mgmt | C | C | R | C | I | I | I | I |

#### 7.2.4 Secrets Handling

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Key Vault setup | I | R | C | A | I | I | I | I |
| Secret creation | R | C | C | C | I | I | I | I |
| Access policy config | I | C | R | A | I | I | I | I |
| Rotation automation | C | R | C | A | I | I | I | C |
| Audit and monitoring | I | C | C | R | I | I | I | C |

#### 7.2.5 Security Validation

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Threat modeling | C | C | C | R | C | C | C | I |
| Security scanning | C | I | I | R | I | I | I | I |
| Penetration testing | C | C | C | R | C | I | C | I |
| Compliance review | C | C | C | R | C | C | C | I |
| Risk acceptance | I | I | I | A | I | C | I | I |

#### 7.2.6 Monitoring & Incident Handling

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Monitoring setup | C | C | I | C | I | I | I | R |
| Alert configuration | C | C | I | C | I | I | I | R |
| Incident detection | I | I | I | C | I | I | I | R |
| Incident triage | C | C | C | C | C | I | I | R |
| Root cause analysis | R | C | C | C | C | I | C | C |
| Incident resolution | R | C | C | C | C | I | C | A |

#### 7.2.7 Upgrades and Breaking Changes

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Impact assessment | R | C | C | C | C | A | C | C |
| Communication plan | R | C | C | C | C | I | I | C |
| Regression testing | R | C | I | C | C | I | C | C |
| Rollback planning | R | C | I | C | I | I | I | C |
| Change approval | C | C | C | C | C | A | I | C |
| Deployment execution | R | C | I | I | I | I | I | A |

#### 7.2.8 Decommissioning Agents

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Sunset decision | C | C | C | C | C | A | C | C |
| User notification | R | C | I | I | C | I | I | C |
| Data retention review | C | C | I | R | C | I | A | I |
| Access revocation | C | C | R | C | I | I | I | C |
| Resource cleanup | R | R | R | C | I | I | R | R |
| Registry update | R | I | I | I | I | I | I | I |

**Legend:** R = Responsible, A = Accountable, C = Consulted, I = Informed

---

## 8. Anti-Patterns and Common Risks

### 8.1 Governance Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **No inventory, no ownership** | Agents proliferate without accountability; incidents have no owner | Maintain agent registry; require owner for every agent |
| **Controls as "guidance only"** | Policies exist but aren't enforced technically | Implement technical controls (DLP, Conditional Access) |
| **Missing environment strategy** | Production and dev mixed; accidental exposure | Implement zoned governance (Safe, Supported, IT-Managed) |
| **One-size-fits-all governance** | Over-restricts simple agents or under-governs critical ones | Tier agents by risk; differentiated controls |
| **Shadow agents** | Teams bypass governance to ship faster | Make governance enabling, not blocking; provide templates |

### 8.2 Security Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Overprivileged app permissions** | Agent can access more than needed; blast radius | Minimum necessary permissions; regular reviews |
| **Shared secrets across tenants** | Compromise in one tenant affects others | Per-tenant secrets; workload identity federation |
| **No input validation** | Prompt injection attacks succeed | Input sanitization; content filtering; grounding separation |
| **Trust without verification** | Cross-tenant access assumed safe | Zero trust; explicit trust configuration; continuous monitoring |
| **Audit logging as afterthought** | Can't investigate incidents or prove compliance | Design logging from start; centralize across tenants |

### 8.3 Operational Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Manual deployments** | Inconsistent, error-prone, slow | CI/CD automation; infrastructure as code |
| **No testing strategy** | Agents fail in production; regressions | Unit, integration, adversarial, load testing |
| **Monitoring blindspots** | Issues discovered by users, not ops | Comprehensive observability; proactive alerting |
| **Cost governance ignored** | Runaway token/API costs | Usage monitoring; alerts on thresholds; chargebacks |
| **Knowledge silos** | One person knows how agent works | Documentation requirements; cross-training |

### 8.4 Multi-Tenant Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Implicit trust between tenants** | Assume security based on ownership | Explicit cross-tenant access settings; defense in depth |
| **Shared infrastructure without isolation** | Noisy neighbor; data leakage | Tenant-specific resources; logical/physical isolation |
| **Inconsistent governance across tenants** | Weakest tenant becomes entry point | Federated governance with centralized standards |
| **Cross-tenant admin sprawl** | Too many people with cross-tenant access | PIM for cross-tenant roles; regular access reviews |

---

### 8.5 Root Cause Analysis & Remediation Playbooks

#### Anti-Pattern: Oversharing Exposure in Copilot

**Scenario:** Users report Copilot is surfacing confidential documents they shouldn't see.

**Root Cause Analysis:**
1. **Immediate:** SharePoint permissions too broad (Everyone, All Company)
2. **Contributing:** No sensitivity labeling on confidential documents
3. **Systemic:** Self-service site creation without default permissions restrictions
4. **Cultural:** Users unaware of Copilot's data access scope

**Remediation Playbook:**

| Phase | Actions | Timeline | Owner |
|-------|---------|----------|-------|
| **Contain** | Run Purview oversharing report; identify exposed sites | Day 1 | Security |
| **Remediate** | Restrict broad permissions; apply sensitivity labels | Week 1-2 | M365 Platform |
| **Prevent** | Enable default labeling; restrict sharing links | Week 2-4 | IAM + Security |
| **Monitor** | Deploy continuous oversharing detection | Ongoing | Security |
| **Educate** | Copilot data awareness training for all users | Week 4-8 | Enablement |

---

#### Anti-Pattern: Cross-Tenant Authentication Failure

**Scenario:** Cross-tenant agents suddenly stop working; authentication errors in logs.

**Root Cause Analysis:**
1. **Immediate:** Certificate expired on multi-tenant app registration
2. **Contributing:** No monitoring for certificate expiration
3. **Systemic:** Manual certificate rotation process
4. **Cultural:** Certificate management not treated as critical path

**Remediation Playbook:**

| Phase | Actions | Timeline | Owner |
|-------|---------|----------|-------|
| **Contain** | Emergency certificate generation and deployment | Hours | IAM + M365 Platform |
| **Remediate** | Update both tenant configurations; validate auth | Day 1 | IAM |
| **Prevent** | Implement automated cert monitoring (30-day alerts) | Week 1 | AI Operations |
| **Automate** | Deploy Azure Key Vault auto-rotation | Month 1 | M365 Platform |
| **Document** | Update runbook; add to operational checklist | Week 2 | CoE |

---

#### Anti-Pattern: Shadow Agents

**Scenario:** Audit discovers 50+ undocumented agents in Copilot Studio environments.

**Root Cause Analysis:**
1. **Immediate:** Users created agents without going through approval
2. **Contributing:** Copilot Studio access too permissive
3. **Systemic:** No agent discovery or inventory process
4. **Cultural:** Governance perceived as blocker; users go around it

**Remediation Playbook:**

| Phase | Actions | Timeline | Owner |
|-------|---------|----------|-------|
| **Discover** | Run environment scan; inventory all agents | Week 1 | M365 Platform |
| **Assess** | Classify discovered agents (keep, remediate, retire) | Week 2 | CoE + Security |
| **Register** | Add legitimate agents to registry with owners | Week 3 | CoE |
| **Restrict** | Implement environment controls (DLP, maker restrictions) | Week 4 | M365 Platform |
| **Enable** | Streamline approval process; provide templates | Month 2 | CoE |
| **Communicate** | Share "why governance" messaging; highlight benefits | Ongoing | Enablement |

---

#### Anti-Pattern: Cost Explosion

**Scenario:** Monthly Copilot Credits consumption 400% over budget.

**Root Cause Analysis:**
1. **Immediate:** New agent using tenant graph grounding for all queries
2. **Contributing:** No cost estimation during agent design
3. **Systemic:** No consumption limits or quotas on environments
4. **Cultural:** "Cloud is free" mindset; no chargeback model

**Remediation Playbook:**

| Phase | Actions | Timeline | Owner |
|-------|---------|----------|-------|
| **Contain** | Identify high-consumption agents; optimize or disable | Day 1-3 | CoE + AI Ops |
| **Analyze** | Run cost attribution report; identify patterns | Week 1 | Finance + CoE |
| **Optimize** | Implement caching, reduce grounding scope, use classic answers | Week 2-4 | AI Engineering |
| **Govern** | Set environment quotas; implement cost alerts | Month 1 | M365 Platform |
| **Chargeback** | Implement cost allocation model to business units | Month 2-3 | Finance + CoE |
| **Educate** | Add cost estimation to agent design review | Ongoing | CoE |

---

#### Anti-Pattern: Prompt Injection Attack

**Scenario:** Agent produces unauthorized output or performs unintended actions after receiving adversarial input.

**Root Cause Analysis:**
1. **Immediate:** Agent processed malicious input without validation
2. **Contributing:** No adversarial testing in CI/CD
3. **Systemic:** Agent instructions not hardened against injection
4. **Cultural:** Security viewed as "nice to have" not "must have"

**Remediation Playbook:**

| Phase | Actions | Timeline | Owner |
|-------|---------|----------|-------|
| **Contain** | Disable affected agent immediately | Hours | AI Ops + Security |
| **Investigate** | Analyze attack vector; assess data exposure | Day 1-2 | Security |
| **Fix** | Harden agent instructions; add input validation | Week 1 | AI Engineering |
| **Test** | Run adversarial testing suite before re-deployment | Week 1-2 | QA + Security |
| **Prevent** | Add adversarial testing to CI/CD pipeline | Month 1 | CoE Engineering |
| **Monitor** | Deploy prompt anomaly detection | Month 1-2 | Security |
| **Educate** | Prompt engineering security training | Ongoing | CoE |

---

## 9. Glossary & Terminology

| Term | Definition |
|------|------------|
| **Agent** | An AI-powered application that can understand context, take actions, and interact with users or systems autonomously within defined boundaries |
| **B2B Collaboration** | Microsoft Entra feature enabling users from one tenant to access resources in another as guest users |
| **CAB** | Change Advisory Board — governance body approving production changes |
| **Conditional Access** | Policy-based access control in Entra ID based on signals like user, device, location, risk |
| **CoE** | Center of Excellence — centralized team providing standards, guidance, and shared services |
| **Copilot Studio** | Microsoft's low-code platform for building AI agents |
| **Cross-Tenant Access Settings** | Entra ID configuration controlling B2B collaboration between specific tenants |
| **Cross-Tenant Synchronization** | Automated provisioning of B2B users between trusted tenants |
| **Declarative Agent** | Configuration-based agent running on M365 Copilot infrastructure |
| **DLP** | Data Loss Prevention — policies preventing sensitive data exfiltration |
| **Entra ID** | Microsoft's cloud identity and access management service (formerly Azure AD) |
| **Enterprise Tenant** | The Entra ID tenant serving corporate/HQ functions |
| **Federated Agent Mesh** | Architecture pattern with centralized governance and distributed agent execution |
| **Grounding** | Process of augmenting agent responses with enterprise knowledge |
| **MCP** | Model Context Protocol — standardized protocol for agent-to-data communication |
| **Managed Identity** | Azure feature providing automatic identity for Azure resources without credentials |
| **MTO** | Multi-Tenant Organization — Entra ID feature simplifying collaboration across owned tenants |
| **OBO** | On-Behalf-Of flow — OAuth pattern where service acts on behalf of user |
| **PIM** | Privileged Identity Management — just-in-time access for administrative roles |
| **RACI** | Responsible, Accountable, Consulted, Informed — decision rights matrix |
| **RAG** | Retrieval-Augmented Generation — grounding AI responses in retrieved documents |
| **Retail Tenant** | The Entra ID tenant serving store operations and frontline workers |
| **Service Principal** | Identity object in Entra ID representing an application |
| **Workload Identity Federation** | Technique enabling Azure resources to access external systems without secrets |
| **XPIA** | Cross-Prompt Injection Attack — security vulnerability in conversational AI |
| **Zero Trust** | Security model assuming no implicit trust; verify explicitly, least privilege, assume breach |
| **Zoned Governance** | Environment strategy with progressive controls (Safe, Supported, IT-Managed) |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-24 | Enterprise Architecture | Initial release |

---

*End of Architecture Document*
