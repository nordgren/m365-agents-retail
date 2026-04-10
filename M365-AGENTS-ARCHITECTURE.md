# Enterprise Architecture for M365 Agents in Multi-Tenant Retail Environments

**Version:** 3.1  
**Date:** 2026-04-10  
**Classification:** Enterprise Architecture Proposal  
**Scope:** Company-Agnostic, Enterprise-Grade  
**Updated:** QA completion—Operational runbooks, worked cost examples, retail DLP policies; Deep research (RAG, MCP, Multi-Agent); OWASP Top 10; Agent 365

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Microsoft Agent 365: The Native Control Plane](#3-microsoft-agent-365-the-native-control-plane) *(NEW)*
4. [Reference Architectures](#4-reference-architectures)
5. [Detailed Component Descriptions](#5-detailed-component-descriptions)
   - 5.3.2 [RAG Deep Dive](#532-rag-retrieval-augmented-generation) *(EXPANDED)*
   - 5.4.2 [MCP Servers](#542-model-context-protocol-mcp) *(EXPANDED)*
   - 5.5 [Multi-Agent Orchestration](#55-multi-agent-orchestration) *(NEW)*
   - 5.7 [OWASP Top 10 for Agentic AI Mapping](#57-owasp-top-10-for-agentic-ai-mapping)
6. [Security Architecture & Risks](#6-security-architecture--risks)
7. [Operational Model](#7-operational-model)
8. [Team Structure & RACI Matrix](#8-team-structure--raci-matrix)
9. [Anti-Patterns and Common Risks](#9-anti-patterns-and-common-risks)
10. [Glossary & Terminology](#10-glossary--terminology)

---

## 1. Executive Summary

### Strategic Context

Microsoft 365 Copilot and its extensibility framework—including declarative agents, custom engine agents, and Copilot Studio—represent a fundamental shift in how enterprises operationalize AI within productivity and business workflows. For retail organizations operating across multiple Entra ID tenants (typically an **Enterprise/Corporate tenant** and a **Retail/Store Operations tenant**), this creates both opportunity and architectural complexity.

This document provides enterprise architects, security teams, and transformation leaders with a company-agnostic framework for designing, deploying, operating, and securing M365 agents across multi-tenant retail environments. The guidance addresses identity boundaries, data residency, least-privilege access, cross-tenant orchestration, and DevSecOps practices specific to agent lifecycle management.

### Key Recommendations

1. **Adopt Microsoft Agent 365 as the native control plane** for agent governance, leveraging its Observe/Govern/Secure pillars to replace custom governance infrastructure where possible. (GA May 1, 2026; $15/user/month standalone or included in M365 E7 at $99/user/month)

2. **Implement Entra Agent ID for all production agents** to enable first-class identity management, Conditional Access policies, and Identity Protection for agents—treating agents with the same rigor as users and workloads.

3. **Adopt a tiered agent governance model** differentiating personal productivity agents, departmental agents, and mission-critical enterprise agents—each with appropriate security, compliance, and oversight controls.

4. **Establish explicit cross-tenant trust relationships** using Microsoft Entra cross-tenant access settings, B2B collaboration, and Agent Identity Blueprints for multi-tenant agents—avoiding implicit trust assumptions.

5. **Design for zero trust from day one** with agent-specific Conditional Access policies, risk-based access controls, network segmentation, secrets management via Azure Key Vault, and continuous monitoring integrated into CI/CD pipelines.

6. **Create a dedicated Agent Governance CoE** that leverages Agent 365's native capabilities while adding organization-specific policies, manages the agent registry, coordinates cross-functional reviews, and provides shared services (templates, testing frameworks, observability) to agent development teams.

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

## 3. Microsoft Agent 365: The Native Control Plane

> **General Availability:** May 1, 2026  
> **Pricing:** $15/user/month standalone | Included in M365 E7 ($99/user/month)

### 3.1 Overview

Microsoft Agent 365 is the **native control plane for AI agents** in the Microsoft 365 ecosystem. It provides enterprise-grade capabilities for observing, governing, and securing agents at scale—replacing or augmenting many custom governance mechanisms previously required.

Agent 365 introduces three core pillars:

| Pillar | Capabilities | Key Features |
|--------|-------------|--------------|
| **Observe** | Visibility into agent activity across the organization | Agent registry (including shadow agent discovery), real-time monitoring, agent-data-people visualization, performance analytics |
| **Govern** | Lifecycle management and policy enforcement | Agent onboarding workflows, access control, SDK/API standards, interoperability with Work IQ |
| **Secure** | Threat protection and data security | Runtime threat detection, risk-based access, data protection integration with Purview, agent compromise remediation |

### 3.2 Entra Agent ID: First-Class Agent Identities

Agent 365 introduces **Microsoft Entra Agent ID**, which treats AI agents as first-class identity principals alongside users and workload identities.

#### Identity Constructs

| Construct | Description | Purpose |
|-----------|-------------|---------|
| **Agent Blueprint** | Logical definition of an agent type | Template for creating agent instances |
| **Agent Identity Blueprint Principal** | Service principal representing the blueprint in a tenant | Creates agent identities and agent users |
| **Agent Identity** | Instantiated agent identity | Performs token acquisitions to access resources |
| **Agent User** | Non-human user identity for agent experiences | Used when agent requires user account context |
| **Agent Resource** | Blueprint or identity acting as target resource | Enables agent-to-agent (A2A) flows |

#### Impact on Identity Architecture

**Before Agent 365:**
- Agents used standard app registrations and service principals
- No native distinction between agents and other applications
- Custom governance required to track agent-specific behavior
- Cross-tenant agents required multi-tenant app registrations with manual consent

**With Agent 365:**
- Agents get dedicated identity type with agent-specific attributes
- Agent Registry provides complete inventory (including shadow agents)
- Entra Agent ID enables agent-specific Conditional Access policies
- Cross-tenant agents use **Agent Identity Blueprints** that can be brought into target tenants

### 3.3 Conditional Access for Agents

Agent 365 extends Conditional Access to agent identities, applying Zero Trust principles to agent access requests.

#### How Conditional Access Applies

| Flow | Conditional Access? | Notes |
|------|---------------------|-------|
| Agent Identity → Resource | ✅ Yes | Governed by agent identity policies |
| Agent User → Resource | ✅ Yes | Governed by agent user policies |
| Agent Blueprint → Graph (create agent) | ❌ No | Blueprint creation is privileged |
| Agent → Token Exchange Endpoint | ❌ No | Intermediate exchange, not resource access |

#### Policy Configuration Options

**Assignments:**
- All agent identities in tenant
- Specific agents by object ID
- Agents with specific custom security attributes
- Agents grouped by blueprint

**Conditions:**
- Agent risk level (High/Medium/Low) from ID Protection
- Custom security attributes on agents

**Controls:**
- Block risky agents
- Require specific compliance posture
- Report-only mode for simulation

#### Replacing Custom Governance Gates

| Previous Custom Gate | Agent 365 Native Replacement |
|---------------------|------------------------------|
| Manual agent registry | Agent 365 Agent Registry (auto-discovery) |
| Custom approval workflows | Onboarding workflows with blueprint-based provisioning |
| Ad hoc security reviews | Conditional Access policies + ID Protection for agents |
| Custom monitoring dashboards | Agent 365 visualization and analytics |
| Manual shadow agent discovery | Automatic shadow agent detection |

### 3.4 Agent 365 Mapping to Architecture Patterns

#### Pattern A: Corporate-Only Agents

| Agent 365 Capability | Application in Pattern A |
|---------------------|-------------------------|
| **Agent Registry** | Single-tenant inventory; shadow agent discovery within corporate tenant |
| **Conditional Access** | Block high-risk agents; require agents from approved blueprints only |
| **Purview Integration** | Apply sensitivity labels to agent-accessed content; DLP on agent outputs |
| **Visualization** | Map agent ↔ data ↔ user relationships within corporate boundary |

**Configuration:**
```
Tenant: Enterprise
├── Agent 365 enabled
├── Agent Registry: Active (discovers all agents)
├── Conditional Access Policies:
│   ├── Block high-risk agent identities
│   ├── Require approved blueprints for production agents
│   └── Report-only for Tier 1-2 agents
└── Purview: DLP policies apply to agent data access
```

#### Pattern B: Dual-Tenant Agents

| Agent 365 Capability | Application in Pattern B |
|---------------------|-------------------------|
| **Agent Identity Blueprint** | Create blueprint in Enterprise Tenant; provision blueprint principal in Retail Tenant |
| **Cross-Tenant Conditional Access** | Define policies in both tenants; coordinate trust settings |
| **Federated Monitoring** | Agent 365 in each tenant; correlate via Sentinel for cross-tenant view |
| **Shared Governance** | Common blueprints enable consistent agent behavior across tenants |

**Cross-Tenant Agent Identity Flow:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE TENANT                            │
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │ Agent Blueprint │───►│ Agent Identity  │                    │
│  │   (Template)    │    │  (Instance)     │                    │
│  └─────────────────┘    └────────┬────────┘                    │
│                                  │                              │
└──────────────────────────────────┼──────────────────────────────┘
                                   │
              Cross-Tenant Access via Agent Identity Blueprint
                                   │
┌──────────────────────────────────┼──────────────────────────────┐
│                                  ▼         RETAIL TENANT        │
│                                                                 │
│  ┌─────────────────────┐    ┌─────────────────┐                │
│  │ Blueprint Principal │───►│  Agent Identity │                │
│  │  (Consented Copy)   │    │ (Local Instance)│                │
│  └─────────────────────┘    └────────┬────────┘                │
│                                      │                          │
│                            ┌─────────▼─────────┐               │
│                            │  Retail Resources │               │
│                            │  (POS, Inventory) │               │
│                            └───────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

**Conditional Access Configuration (Both Tenants):**
```
Enterprise Tenant:
├── CA Policy: "Cross-Tenant Agent Access"
│   ├── Applies to: Agent identities accessing Retail Tenant resources
│   ├── Conditions: Agent risk = High → Block
│   └── Grant: Require approved blueprint membership

Retail Tenant:
├── CA Policy: "Inbound Agent Validation"
│   ├── Applies to: Agent identities from Enterprise Tenant
│   ├── Conditions: Custom attribute "approved_for_retail" = true
│   └── Grant: Allow with logging
```

#### Pattern C: Federated Agent Mesh

| Agent 365 Capability | Application in Pattern C |
|---------------------|-------------------------|
| **Centralized Registry** | Agent 365 instances in each tenant; aggregate via API/Sentinel |
| **Blueprint Distribution** | Define canonical blueprints; distribute principals to all tenants |
| **Unified Conditional Access** | Standard CA policies deployed across all tenants via automation |
| **Cross-Tenant Visualization** | Centralized dashboard aggregating Agent 365 data from all tenants |

**Federated Governance Architecture:**
```
┌─────────────────────────────────────────────────────────────────┐
│              GOVERNANCE HUB (Central Management)                │
│                                                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │  Canonical  │  │   Policy     │  │  Centralized │           │
│  │ Blueprints  │  │   Templates  │  │  Monitoring  │           │
│  └──────┬──────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                │                  │                   │
│         │    Distribute Blueprints & Policies                   │
│         │                │                  │                   │
└─────────┼────────────────┼──────────────────┼───────────────────┘
          │                │                  │
    ┌─────┴─────┬──────────┴──────┬───────────┴─────┐
    │           │                 │                 │
    ▼           ▼                 ▼                 ▼
┌────────┐  ┌────────┐      ┌────────┐       ┌────────┐
│ Tenant │  │ Tenant │      │ Tenant │       │ Tenant │
│   A    │  │   B    │      │   C    │       │   N    │
│        │  │        │      │        │       │        │
│Agent365│  │Agent365│      │Agent365│       │Agent365│
│Enabled │  │Enabled │      │Enabled │       │Enabled │
└────────┘  └────────┘      └────────┘       └────────┘
```

### 3.5 Licensing & Cost Model

#### Licensing Options

| License | Price | Includes | Best For |
|---------|-------|----------|----------|
| **Agent 365 Standalone** | $15/user/month | Agent 365 control plane only | Organizations adding agent governance to existing M365 |
| **Microsoft 365 E7** | $99/user/month | E5 + Copilot + Entra Suite + Agent 365 | Organizations committed to AI-first productivity |

#### What's Included vs. Consumption-Based

| Component | Included in License | Consumption-Based |
|-----------|--------------------|--------------------|
| Agent Registry | ✅ | — |
| Conditional Access for Agents | ✅ | — |
| Agent 365 Admin Center | ✅ | — |
| Monitoring & Visualization | ✅ | — |
| Purview Integration | ✅ | — |
| Copilot Credits (agent interactions) | ❌ | Pay-as-you-go or prepaid |
| Azure Infrastructure (Functions, etc.) | ❌ | Azure consumption |

#### M365 E7 Bundle Components

```
Microsoft 365 E7 ($99/user/month)
├── Microsoft 365 E5 (productivity, security, compliance)
├── Microsoft 365 Copilot ($30 value)
├── Microsoft Entra Suite
│   ├── Entra ID P2
│   ├── Entra Internet Access
│   ├── Entra Private Access
│   └── Entra Suite Governance
└── Microsoft Agent 365 ($15 value)

Savings vs. à la carte: ~15%
```

### 3.6 Migration Considerations

#### Transitioning Custom Governance to Agent 365

| Phase | Activities | Timeline |
|-------|-----------|----------|
| **Assessment** | Inventory current agents; identify custom governance components; map to Agent 365 capabilities | Week 1-2 |
| **Pilot** | Enable Agent 365 in non-production; migrate subset of agents to Agent ID; test Conditional Access | Week 3-6 |
| **Parallel Run** | Run Agent 365 alongside custom governance; validate feature parity; train teams | Week 7-12 |
| **Cutover** | Migrate production agents to Agent ID; deprecate custom registry; enable native policies | Week 13-16 |
| **Optimization** | Retire custom components; leverage advanced Agent 365 features; continuous improvement | Ongoing |

#### What to Keep Custom

Even with Agent 365, organizations may retain custom components for:

- **Organization-specific approval workflows** beyond Agent 365's onboarding
- **Integration with non-Microsoft systems** (ITSM, legacy GRC tools)
- **Industry-specific compliance requirements** not covered by Purview
- **Advanced analytics** beyond Agent 365's native visualization

### 3.7 Worked Cost Examples

The following models estimate total cost of ownership for three retail deployment scenarios. All figures are illustrative and should be validated with current Microsoft pricing.

---

#### Scenario 1: 500-User Corporate Pilot (Pattern A)

**Profile:**
- Single tenant (corporate only)
- 500 knowledge workers
- 3 agents: Store Ops FAQ, HR Policy, IT Help Desk
- 6-month pilot duration

**Monthly Cost Breakdown:**

| Cost Component | Quantity | Unit Cost | Monthly Cost |
|----------------|----------|-----------|--------------|
| **Licensing** | | | |
| M365 E5 (existing) | 500 | $0 (sunk) | $0 |
| Microsoft 365 Copilot | 500 | $30 | $15,000 |
| Agent 365 Standalone | 500 | $15 | $7,500 |
| **Copilot Credits** | | | |
| Classic answers (est. 2,000/month) | 2,000 | 1 credit | 2,000 credits |
| Generative answers (est. 3,000/month) | 3,000 | 2 credits | 6,000 credits |
| Agent actions (est. 500/month) | 500 | 5 credits | 2,500 credits |
| **Subtotal credits** | | | 10,500 credits |
| Credit cost (pay-as-you-go) | 10,500 | $0.01 | $105 |
| **Azure Infrastructure** | | | |
| Application Insights | 1 | ~$50 | $50 |
| Key Vault | 1 | ~$5 | $5 |
| **CoE Staffing** | | | |
| Shared (0.5 FTE pilot support) | 0.5 | $12,000 | $6,000 |
| | | | |
| **MONTHLY TOTAL** | | | **$28,660** |
| **ANNUAL PROJECTION** | | | **$343,920** |

**Optimization Levers:**
- Route 50% of queries to classic answers → Save ~$30/month on credits
- Negotiate M365 E7 bundle (saves ~15% on Copilot + Agent 365)
- Use prepaid Copilot Credit packs for volume discount (~10-15% savings)

---

#### Scenario 2: 5,000-User Dual-Tenant Deployment (Pattern B)

**Profile:**
- Two tenants: Enterprise (1,500 users) + Retail (3,500 store associates)
- 8 agents: Store Ops, Inventory, WFM, Customer Service (Tier 2-3)
- Cross-tenant inventory and workforce queries
- Full production deployment

**Monthly Cost Breakdown:**

| Cost Component | Quantity | Unit Cost | Monthly Cost |
|----------------|----------|-----------|--------------|
| **Licensing - Enterprise Tenant** | | | |
| M365 E5 (existing) | 1,500 | $0 (sunk) | $0 |
| Microsoft 365 Copilot | 1,500 | $30 | $45,000 |
| Agent 365 | 1,500 | $15 | $22,500 |
| **Licensing - Retail Tenant** | | | |
| M365 F3 (existing) | 3,500 | $0 (sunk) | $0 |
| Agent 365 (cross-tenant access) | 500 subset | $15 | $7,500 |
| **Copilot Credits - Enterprise** | | | |
| Generative answers | 15,000 | 2 credits | 30,000 |
| Tenant Graph Grounding | 5,000 | 10 credits | 50,000 |
| Agent actions | 8,000 | 5 credits | 40,000 |
| **Subtotal Enterprise credits** | | | 120,000 |
| Credit cost (prepaid pack) | 120,000 | $0.008 | $960 |
| **Copilot Credits - Retail** | | | |
| Generative answers | 25,000 | 2 credits | 50,000 |
| Agent actions | 10,000 | 5 credits | 50,000 |
| **Subtotal Retail credits** | | | 100,000 |
| Credit cost (prepaid pack) | 100,000 | $0.008 | $800 |
| **Azure Infrastructure** | | | |
| API Management (cross-tenant) | 1 | ~$300 | $300 |
| Application Insights (2 tenants) | 2 | ~$100 | $200 |
| Key Vault (2 tenants) | 2 | ~$10 | $20 |
| Azure Functions (custom agents) | 1 | ~$200 | $200 |
| **CoE Staffing** | | | |
| CoE Team (10 FTEs) | 10 | $12,000 | $120,000 |
| | | | |
| **MONTHLY TOTAL** | | | **$197,480** |
| **ANNUAL TOTAL** | | | **$2,369,760** |

**Cost Allocation Model (Chargeback):**
| Tenant | Copilot Credits | Infrastructure | Support | Total/Month |
|--------|-----------------|----------------|---------|-------------|
| Enterprise | $960 | $360 | $60,000 | $61,320 |
| Retail | $800 | $160 | $60,000 | $60,960 |
| Licensing (pooled) | — | — | — | $75,000 |
| **Total** | | | | **$197,280** |

**Optimization Levers:**
- Disable Tenant Graph Grounding for FAQ agents → Save $400/month
- Negotiate EA with Microsoft for volume discounts (15-20%)
- Batch agent actions using agent flows (13 credits/100 vs. 5 each)
- Cache frequent queries → Reduce generative answers 20%

---

#### Scenario 3: 15,000-User Federated Mesh (Pattern C)

**Profile:**
- Multiple tenants: 1 Enterprise + 5 Regional/Franchise tenants
- 15,000 total users (2,000 enterprise, 13,000 franchise)
- 20+ agents including cross-tenant orchestrators
- Multi-agent orchestration, A2A communication
- Centralized governance layer

**Monthly Cost Breakdown:**

| Cost Component | Quantity | Unit Cost | Monthly Cost |
|----------------|----------|-----------|--------------|
| **Licensing - Enterprise Tenant** | | | |
| Microsoft 365 E7 (bundled) | 2,000 | $99 | $198,000 |
| *(includes E5 + Copilot + Agent 365)* | | | |
| **Licensing - Franchise Tenants** | | | |
| M365 F3 (frontline, existing) | 10,000 | $0 (sunk) | $0 |
| Microsoft 365 Copilot (subset) | 3,000 | $30 | $90,000 |
| Agent 365 Standalone | 3,000 | $15 | $45,000 |
| **Copilot Credits - All Tenants** | | | |
| Generative answers | 150,000 | 2 credits | 300,000 |
| Tenant Graph Grounding | 30,000 | 10 credits | 300,000 |
| Agent actions | 80,000 | 5 credits | 400,000 |
| Agent flows (per 100 actions) | 20,000 | 13 credits | 260,000 |
| **Total credits** | | | 1,260,000 |
| Credit cost (EA prepaid) | 1,260,000 | $0.007 | $8,820 |
| **Azure Infrastructure** | | | |
| API Management (governance layer) | 1 | ~$1,500 | $1,500 |
| Azure Functions (orchestration) | 1 | ~$800 | $800 |
| Azure Sentinel (cross-tenant SIEM) | 1 | ~$2,000 | $2,000 |
| Application Insights (6 tenants) | 6 | ~$150 | $900 |
| Key Vault (6 tenants) | 6 | ~$15 | $90 |
| Service Bus (mesh messaging) | 1 | ~$200 | $200 |
| **CoE Staffing** | | | |
| CoE Team (20 FTEs) | 20 | $12,000 | $240,000 |
| Regional Champions (5 FTEs) | 5 | $8,000 | $40,000 |
| | | | |
| **MONTHLY TOTAL** | | | **$627,310** |
| **ANNUAL TOTAL** | | | **$7,527,720** |

**Cost Allocation by Franchise:**

| Franchise Region | Users | Agents | Credits Used | Monthly Allocation |
|------------------|-------|--------|--------------|-------------------|
| North America | 4,000 | 5 | 350,000 | $175,000 |
| Europe | 3,000 | 4 | 280,000 | $145,000 |
| Asia Pacific | 3,500 | 4 | 320,000 | $160,000 |
| Latin America | 1,500 | 3 | 180,000 | $80,000 |
| Middle East | 1,000 | 2 | 130,000 | $67,310 |
| **Total** | 13,000 | 18 | 1,260,000 | **$627,310** |

**Optimization Levers:**
- Negotiate Microsoft Strategic Partnership discount (20-25%)
- Centralize Tenant Graph Grounding to enterprise tenant only
- Implement aggressive caching and query deduplication (30% credit reduction)
- Use classic answers for 80% of frontline queries
- Optimize CoE: reduce to 15 FTEs with automation ($60K/month savings)

---

#### Cost Optimization Summary

| Optimization | Pattern A Savings | Pattern B Savings | Pattern C Savings |
|--------------|-------------------|-------------------|-------------------|
| Classic over generative answers | 5-10% | 10-15% | 15-20% |
| Disable Tenant Graph (where possible) | N/A | 20% credits | 30% credits |
| Prepaid credit packs | 10-15% | 15-20% | 20-25% |
| EA / strategic partnership | 5-10% | 10-15% | 20-25% |
| Response caching | 5% | 10% | 15% |
| CoE automation | N/A | 10% | 15% |

---

## 4. Reference Architectures

### 4.1 Architecture A: Corporate-Only Agents (Baseline)

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

### 4.2 Architecture B: Dual-Tenant Agents (Corporate + Retail)

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

*Option 4: Copilot Studio Native Multi-Tenant Mode (Public Preview)*

> **Status:** Public Preview (February 2026)  
> **Reference:** [Overview of multitenant mode in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/multi-tenant-overview)

Copilot Studio now supports **native multi-tenant mode**, allowing agents hosted in one tenant to be used by users in different Microsoft Entra ID tenants without complex B2B or cross-tenant app setup.

**How It Works:**
- Agent is built and hosted in the **host tenant** (e.g., Enterprise Tenant)
- Users in **client tenants** (e.g., Retail Tenant) access the agent via Teams or M365 Copilot
- Host tenant owns billing, governance, and agent lifecycle
- Client tenant users authenticate via their own Entra ID; the agent receives authenticated context

**Supported Channels:**
- Microsoft Teams (1:1 conversations only; group chats not supported)
- Microsoft 365 Copilot

**Supported Features in Preview:**

| Category | Supported | Not Supported |
|----------|-----------|---------------|
| **Knowledge** | Dataverse Upload, Public Websites | SharePoint (cross-tenant), OneDrive |
| **Tools** | Prompts, REST API connectors, Limited standard connectors (maker auth) | Custom connectors, Computer use, Agent flows, MCP tools |
| **Authentication** | Maker authentication (service principal) | End-user authentication, OAuth, Graph/M365 connectors, Guest users |
| **Analytics** | Channel metrics (Teams-provided) | Aggregated analytics, Transcript data, Session downloads, Satisfaction/effectiveness |
| **Other** | Single-geo deployment | Multi-geo, GCC/GCC High, Group conversations |

**Current Limitations (Public Preview):**

| Limitation | Impact on Retail Scenarios | Workaround |
|------------|---------------------------|------------|
| **Analytics disabled** | Cannot track agent performance across franchise tenants | Use Teams-provided channel metrics; build custom logging via REST API actions |
| **Dataverse transcripts unavailable** | Cannot review conversation history for compliance/training | Export via custom logging endpoint in agent flow; store in host tenant |
| **No end-user authentication** | Cannot access resources requiring delegated user permissions | Use maker authentication for data access; limit to non-personalized scenarios |
| **No Graph/M365 connectors** | Cannot query calendar, email, or M365 data in client tenant | Agent queries host tenant only; use REST API for client tenant integration |
| **No custom connectors** | Cannot integrate with proprietary retail systems via custom connectors | Use REST API actions with public/authenticated endpoints |
| **Single-geo only** | Data residency compliance remains partner responsibility | Verify host tenant region meets client tenant regulatory requirements |

**Admin Controls:**

*Host Tenant (Agent Owner):*
- Power Platform Admin Center → Tenant Isolation controls multi-tenant availability
- If tenant isolation is ON, multi-tenant mode is blocked (unless specific tenants allow-listed)
- DLP policies apply to host tenant
- Billing responsibility remains with host tenant

*Client Tenant (Agent Consumer):*
- Microsoft Admin Center controls app installation
- Standard 3P app governance applies
- Purview integration shows host tenant agent usage (users appear as "anonymous" with GUID)

---

#### Comparison: Native Multi-Tenant Mode vs. Custom Identity Patterns

| Criterion | Native Multi-Tenant Mode | B2B Collaboration | Cross-Tenant Sync | Multi-Tenant App Reg |
|-----------|--------------------------|-------------------|-------------------|---------------------|
| **Setup Complexity** | ✅ Low (toggle + Teams publish) | Medium (invite flow) | High (sync config) | High (app consent) |
| **Time to Deploy** | Days | 1-2 weeks | 2-4 weeks | 1-2 weeks |
| **End-User Auth** | ❌ Not supported | ✅ Full delegated | ✅ Full delegated | ✅ Via OBO flow |
| **Graph/M365 Access** | ❌ Host tenant only | ✅ Both tenants | ✅ Both tenants | ✅ Consented tenant |
| **Custom Connectors** | ❌ Not supported | ✅ Full support | ✅ Full support | ✅ Full support |
| **Analytics** | ❌ Disabled | ✅ Full | ✅ Full | ✅ Full |
| **Transcripts** | ❌ Disabled | ✅ Full | ✅ Full | ✅ Full |
| **Billing Model** | Host pays all | Per-tenant | Per-tenant | Varies |
| **Compliance/Audit** | Limited (no transcripts) | Full | Full | Full |
| **Production Readiness** | ⚠️ Preview | ✅ GA | ✅ GA | ✅ GA |

---

#### Decision Guidance: When to Use Each Pattern

**Use Native Multi-Tenant Mode When:**
- ✅ Building ISV/partner agents for Teams Store distribution
- ✅ FAQ-style, informational agents without personalized data access
- ✅ Rapid prototyping of cross-tenant scenarios
- ✅ Host tenant can own all billing and governance
- ✅ No requirement for conversation transcripts or detailed analytics
- ✅ Agent functionality limited to public websites, REST APIs, and Dataverse knowledge

**Use B2B Collaboration When:**
- ✅ Need end-user authentication and delegated permissions
- ✅ Require access to Graph/M365 data in both tenants
- ✅ Full analytics and transcript compliance requirements
- ✅ Limited number of cross-tenant users (manual invite manageable)
- ✅ Production workloads requiring GA-supported features

**Use Cross-Tenant Synchronization When:**
- ✅ Large-scale user base requiring cross-tenant access
- ✅ Seamless SSO experience without explicit invitations
- ✅ Ongoing identity synchronization requirements
- ✅ Already invested in cross-tenant sync for other workloads

**Use Multi-Tenant App Registration When:**
- ✅ Agent operates as a service (app identity, not user identity)
- ✅ Same agent deployed across many tenants (SaaS model)
- ✅ Client credentials flow sufficient for data access
- ✅ Need full Copilot Studio feature set with GA support

---

#### Migration Path: Custom Patterns → Native Multi-Tenant Mode

For organizations currently using Pattern B workarounds who want to evaluate native multi-tenant mode:

**Phase 1: Assessment (1-2 weeks)**
```
☐ Inventory current cross-tenant agents and their capabilities
☐ Map feature requirements against native mode limitations
☐ Identify agents suitable for migration (FAQ, informational, non-personalized)
☐ Verify host tenant region meets data residency requirements
☐ Review billing implications (host pays all)
```

**Phase 2: Pilot (2-4 weeks)**
```
☐ Select 1-2 low-risk agents for pilot
☐ Enable multi-tenant mode in Copilot Studio (verify tenant isolation is OFF or allow-listed)
☐ Publish to Teams channel with multi-tenant flag
☐ Sideload .ZIP manifest in test client tenant
☐ Validate functionality with users from client tenant
☐ Document gaps vs. current B2B/app registration approach
```

**Phase 3: Parallel Run (4-8 weeks)**
```
☐ Run native mode agent alongside existing cross-tenant agent
☐ Compare user experience and capabilities
☐ Monitor for feature gaps or blockers
☐ Collect user feedback from client tenant
☐ Evaluate Teams Store publication (if applicable)
```

**Phase 4: Decision (1 week)**
```
☐ If native mode meets requirements: Deprecate B2B/app registration approach
☐ If gaps remain: Continue with custom pattern; re-evaluate at native mode GA
☐ Document lessons learned for future migrations
```

**Rollback Plan:**
Native multi-tenant mode can be disabled at any time in Copilot Studio. The agent reverts to single-tenant behavior. Existing B2B/app registration patterns remain available as fallback.

---

**Hybrid Architecture: Combining Native + Custom Patterns**

For retail organizations with diverse requirements, a hybrid approach may be optimal:

| Agent Type | Recommended Pattern | Rationale |
|------------|--------------------|-----------| 
| Store FAQ Bot | Native Multi-Tenant | Informational only; no auth required; rapid deployment |
| Inventory Query Agent | Multi-Tenant App Reg | Requires Graph access to retail tenant Dataverse |
| HR Policy Agent | B2B Collaboration | Requires delegated access to user's HR profile |
| POS Support Agent | Native Multi-Tenant | Public website knowledge; REST API for tickets |
| Customer Service Agent | Cross-Tenant Sync | Needs seamless SSO for customer data access |

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

### 4.3 Architecture C: Federated Agent Mesh

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

### 4.4 Architecture Decision Criteria

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

### 4.5 Failure Modes & Remediation

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

### 4.6 Cost Model & Implications

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

### 4.7 Operational Runbooks

> **Note:** These runbooks provide step-by-step operational procedures with specific Azure and M365 admin actions. All runbooks include decision trees, rollback procedures, and escalation triggers.

---

#### Runbook 1: Agent Deployment (All Patterns)

**Runbook ID:** RB-DEPLOY-001  
**Version:** 2.0  
**Last Updated:** 2026-04-07  
**Owner:** CoE Engineering Lead

---

##### Trigger Conditions
- New agent approved for production deployment
- Major version upgrade of existing agent
- Agent migration between environments or tenants

##### Prerequisites Checklist

| Prerequisite | Pattern A | Pattern B | Pattern C | Verification |
|--------------|-----------|-----------|-----------|--------------|
| Agent design approved | ✅ Required | ✅ Required | ✅ Required | Check ITSM ticket |
| Tier 3+: CoE design review | ✅ Required | ✅ Required | ✅ Required | Review board minutes |
| Security assessment passed | ✅ Required | ✅ Required | ✅ Required | Security scan report |
| Cross-tenant access configured | ❌ N/A | ✅ Required | ✅ Required | Entra admin verification |
| B2B/app consent completed | ❌ N/A | ✅ Required | ✅ Required | Service principal check |
| Agent 365 registry entry | ✅ Required | ✅ Required | ✅ Required | Agent 365 Admin Center |
| CI/CD pipeline configured | ✅ Required | ✅ Required | ✅ Required | Azure DevOps validation |
| Monitoring/alerting active | ✅ Required | ✅ Required | ✅ Required | Application Insights check |
| Rollback plan documented | ✅ Required | ✅ Required | ✅ Required | Runbook attachment |
| CAB approval (if required) | Per policy | ✅ Required | ✅ Required | CAB ticket closed |

---

##### Decision Tree: Deployment Path Selection

```
                    ┌─────────────────────────┐
                    │ Agent deployment type?  │
                    └───────────┬─────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Pattern A │         │ Pattern B │         │ Pattern C │
    │ Single    │         │ Dual      │         │ Federated │
    │ Tenant    │         │ Tenant    │         │ Mesh      │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Deploy to │         │ Deploy to │         │ Deploy    │
    │ single    │         │ Enterprise│         │ governance│
    │ Copilot   │         │ tenant    │         │ layer     │
    │ Studio    │         │ first     │         │ first     │
    │ env       │         │           │         │           │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Standard  │         │ Configure │         │ Deploy to │
    │ validation│         │ cross-    │         │ each      │
    │ & publish │         │ tenant    │         │ tenant    │
    │           │         │ trust     │         │ runtime   │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Go-live   │         │ Deploy to │         │ Configure │
    │           │         │ Retail    │         │ mesh      │
    │           │         │ tenant    │         │ routing   │
    └───────────┘         └─────┬─────┘         └─────┬─────┘
                                │                     │
                                ▼                     ▼
                          ┌───────────┐         ┌───────────┐
                          │ Validate  │         │ End-to-end│
                          │ cross-    │         │ mesh      │
                          │ tenant    │         │ validation│
                          └───────────┘         └───────────┘
```

---

##### Step-by-Step Procedure: Pattern A (Single Tenant)

**Phase 1: Pre-Deployment (T-1 day)**

| Step | Action | Azure/M365 Location | Expected Outcome |
|------|--------|---------------------|------------------|
| 1.1 | Verify Copilot Studio environment health | Power Platform Admin Center → Environments | Status: Ready |
| 1.2 | Confirm agent solution in staging | Copilot Studio → Solutions | Version matches release |
| 1.3 | Validate Application Insights connection | Azure Portal → App Insights → Live Metrics | Telemetry flowing |
| 1.4 | Check Agent 365 registry | Agent 365 Admin Center → Agent Registry | Entry exists, status: Staging |
| 1.5 | Notify stakeholders via Teams | Teams Channel: #agent-deployments | Acknowledgment received |

**Phase 2: Deployment Execution (Change Window)**

| Step | Action | Azure/M365 Location | Expected Outcome |
|------|--------|---------------------|------------------|
| 2.1 | Export solution from staging | Copilot Studio → Solutions → Export | .zip file downloaded |
| 2.2 | Import solution to production | Copilot Studio (Prod) → Solutions → Import | Import successful |
| 2.3 | Publish agent | Copilot Studio → Agent → Publish | Status: Published |
| 2.4 | Verify Teams channel deployment | Teams Admin Center → Manage apps | Agent visible |
| 2.5 | Update Agent 365 status | Agent 365 Admin Center → Registry → Edit | Status: Production |

**Phase 3: Validation (T+30 min)**

| Step | Action | Expected Outcome | Rollback Trigger |
|------|--------|------------------|------------------|
| 3.1 | Execute smoke test suite | All tests pass | Any failure |
| 3.2 | Verify agent responds | Correct responses | Incorrect/no response |
| 3.3 | Check Application Insights | No errors, latency <2s | Error rate >1% |
| 3.4 | Confirm Conditional Access | Policy applies correctly | Policy not enforced |
| 3.5 | Test user permissions | ACL trimming works | Unauthorized access |

**Phase 4: Go-Live Confirmation**

| Step | Action | Azure/M365 Location |
|------|--------|---------------------|
| 4.1 | Update ITSM ticket | ServiceNow/ITSM → Close with success |
| 4.2 | Notify stakeholders | Teams Channel: #agent-deployments |
| 4.3 | Archive deployment artifacts | Azure DevOps → Releases → Tag |
| 4.4 | Update runbook evidence | SharePoint → Agent Documentation |

---

##### Step-by-Step Procedure: Pattern B (Dual-Tenant)

**Additional Prerequisites:**
- Cross-tenant access settings configured in both tenants
- B2B collaboration or multi-tenant app registration consented
- Certificate-based auth configured in Azure Key Vault

**Phase 1: Enterprise Tenant Deployment (T-1 day)**

| Step | Action | Azure/M365 Location | Notes |
|------|--------|---------------------|-------|
| 1.1 | Deploy agent to Enterprise Copilot Studio | Copilot Studio (Enterprise) | Standard deployment |
| 1.2 | Verify app registration | Entra ID → App registrations | Multi-tenant: Yes |
| 1.3 | Confirm certificate in Key Vault | Azure Portal → Key Vault → Certificates | Not expiring <30 days |
| 1.4 | Test enterprise-only functionality | Agent test pane | Passes |

**Phase 2: Cross-Tenant Configuration (Change Window)**

| Step | Action | Azure/M365 Location | Verification |
|------|--------|---------------------|--------------|
| 2.1 | Verify Retail Tenant inbound settings | Entra ID (Retail) → External Identities → Cross-tenant access | Enterprise Tenant allowed |
| 2.2 | Confirm service principal in Retail | Entra ID (Retail) → Enterprise applications | App ID matches |
| 2.3 | Grant API permissions in Retail | Entra ID (Retail) → Enterprise apps → Permissions | Admin consent granted |
| 2.4 | Test cross-tenant token acquisition | Postman/Graph Explorer | Token issued successfully |

**Phase 3: Retail Tenant Deployment**

| Step | Action | Azure/M365 Location | Notes |
|------|--------|---------------------|-------|
| 3.1 | Deploy retail-facing agent components | Copilot Studio (Retail) or API plugin | Depends on architecture |
| 3.2 | Configure agent to use Enterprise service | Agent settings → Connections | Certificate auth |
| 3.3 | Test end-to-end cross-tenant flow | Agent test with Retail user | Data flows correctly |
| 3.4 | Verify Agent 365 in both tenants | Agent 365 Admin Center (both) | Registered in both |

**Phase 4: Cross-Tenant Validation**

| Test | Method | Pass Criteria | Rollback Trigger |
|------|--------|---------------|------------------|
| User from Retail accesses Enterprise data | End-user test | Correct data, ACL enforced | Wrong data or ACL failure |
| Conditional Access enforced | Sign-in logs | CA policy applied | Policy bypassed |
| Cross-tenant audit logging | Sentinel query | Events in both tenants | Missing audit trail |
| Certificate auth working | Monitor auth logs | No client secret usage | Secret auth detected |

---

##### Step-by-Step Procedure: Pattern C (Federated Mesh)

**Additional Prerequisites:**
- Governance layer deployed (Azure Functions, API Management, Sentinel)
- All tenant runtimes configured
- Mesh routing rules defined
- Multi-tenant agent identity blueprints created

**Phase 1: Governance Layer Deployment**

| Step | Action | Azure Location |
|------|--------|----------------|
| 1.1 | Deploy/update policy engine | Azure Functions → Deploy slot |
| 1.2 | Update API Management routing | APIM → APIs → Agent routing |
| 1.3 | Configure Sentinel workbooks | Sentinel → Workbooks → Agent mesh |
| 1.4 | Test governance layer health | Azure Monitor → Dashboard |

**Phase 2: Tenant Runtime Deployments (Rolling)**

| Step | Action | Notes |
|------|--------|-------|
| 2.1 | Deploy to Tenant 1 (Enterprise) | Primary tenant, full validation |
| 2.2 | Validate Tenant 1 | Run test suite |
| 2.3 | Deploy to Tenant 2 (Retail) | Secondary tenant |
| 2.4 | Validate cross-tenant Tenant 1↔2 | Cross-tenant test suite |
| 2.5 | Deploy to Tenants 3-N | Regional/franchise tenants |
| 2.6 | Validate mesh routing | End-to-end mesh test |

**Phase 3: Mesh Validation**

| Test | Description | Tool |
|------|-------------|------|
| Policy propagation | Verify governance rules in all tenants | Azure Functions logs |
| Cross-mesh routing | Request routes correctly | APIM analytics |
| Federated identity | WIF tokens accepted across mesh | Entra sign-in logs |
| Centralized monitoring | All tenants report to Sentinel | Sentinel queries |
| Failover | Remove one tenant, verify graceful degradation | Chaos engineering |

---

##### Rollback Procedures

**Rollback Decision Tree:**
```
                    ┌─────────────────────────┐
                    │ Issue detected during   │
                    │ deployment/validation?  │
                    └───────────┬─────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Critical  │         │ Non-      │         │ Minor     │
    │ (P1/P2)   │         │ Critical  │         │ (P4)      │
    │ Data/     │         │ (P3)      │         │ Cosmetic  │
    │ Security  │         │ Degraded  │         │           │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ IMMEDIATE │         │ ROLLBACK  │         │ PROCEED   │
    │ ROLLBACK  │         │ within    │         │ with      │
    │ + disable │         │ change    │         │ post-     │
    │ agent     │         │ window    │         │ deploy    │
    └───────────┘         └───────────┘         │ fix       │
                                                └───────────┘
```

**Pattern A Rollback Steps:**

| Step | Action | Time |
|------|--------|------|
| R1 | Announce rollback decision in Teams | T+0 |
| R2 | Unpublish agent: Copilot Studio → Agent → Unpublish | T+2 min |
| R3 | Import previous solution version | T+5 min |
| R4 | Publish previous version | T+8 min |
| R5 | Validate rollback successful | T+15 min |
| R6 | Notify stakeholders | T+20 min |
| R7 | Create incident ticket | T+25 min |

**Pattern B Rollback Steps:**

| Step | Action | Tenant | Time |
|------|--------|--------|------|
| R1 | Disable agent in both tenants | Both | T+0 |
| R2 | Revoke Retail tenant consent (if needed) | Retail | T+5 min |
| R3 | Restore Enterprise tenant agent | Enterprise | T+10 min |
| R4 | Restore Retail tenant agent | Retail | T+15 min |
| R5 | Re-establish cross-tenant trust | Both | T+20 min |
| R6 | Validate rollback | Both | T+30 min |

**Pattern C Rollback Steps:**

| Step | Action | Scope | Time |
|------|--------|-------|------|
| R1 | Activate mesh circuit breaker | Governance | T+0 |
| R2 | Route traffic to previous version (blue-green) | APIM | T+2 min |
| R3 | Rollback governance layer | Azure Functions | T+10 min |
| R4 | Rollback tenant runtimes (parallel) | All tenants | T+20 min |
| R5 | Restore mesh routing | APIM | T+30 min |
| R6 | Validate mesh health | All tenants | T+45 min |

---

##### Escalation Triggers

| Condition | Escalation Level | Contact | SLA |
|-----------|-----------------|---------|-----|
| Rollback fails | L2: CoE Director | On-call rotation | 15 min |
| Data exposure suspected | L3: CISO + Legal | Security hotline | Immediate |
| Cross-tenant auth failure | L2: IAM Lead | IAM on-call | 30 min |
| Multiple tenants affected | L3: Executive Steering | CIO notification | 1 hour |
| Vendor/platform issue | L2: Microsoft Support | Premier support | Per contract |

---

##### Post-Deployment Checklist

| Item | Owner | Due |
|------|-------|-----|
| Update agent documentation | CoE Enablement | T+1 day |
| Archive deployment evidence | CoE Engineering | T+1 day |
| Send deployment report | CoE Director | T+2 days |
| Close ITSM ticket | Deployer | T+1 day |
| Schedule 7-day health review | AI Operations | T+7 days |

---

#### Runbook 2: Cross-Tenant Certificate Rotation

**Runbook ID:** RB-CERT-001  
**Version:** 2.0  
**Last Updated:** 2026-04-07  
**Owner:** IAM Team Lead

---

##### Trigger Conditions
- Certificate expiration within 30 days (automated alert)
- Security policy requires rotation (annual)
- Certificate compromise suspected (immediate)

##### Prerequisites

| Prerequisite | Verification | Responsible |
|--------------|--------------|-------------|
| Azure Key Vault access | Can create certificates | IAM Team |
| Enterprise Tenant admin | Global Admin or App Admin | IAM Team |
| Retail Tenant admin | Global Admin or App Admin | IAM Team (or Franchise IT) |
| Change approval | CAB ticket approved | Change Manager |
| Rollback certificate | Previous cert still valid | IAM Team |

---

##### Decision Tree: Rotation Type

```
                    ┌─────────────────────────┐
                    │ Rotation trigger?       │
                    └───────────┬─────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Scheduled │         │ Policy    │         │ Emergency │
    │ (30-day   │         │ (Annual)  │         │ (Compro-  │
    │ warning)  │         │           │         │ mise)     │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌───────────┐         ┌───────────┐         ┌───────────┐
    │ Standard  │         │ Standard  │         │ EXPEDITED │
    │ rotation  │         │ rotation  │         │ rotation  │
    │ (2 weeks) │         │ (2 weeks) │         │ (4 hours) │
    └───────────┘         └───────────┘         └───────────┘
```

---

##### Step-by-Step Procedure: Standard Rotation

**Week 1: Certificate Generation & Preparation**

| Step | Action | Azure/M365 Location | CLI/PowerShell |
|------|--------|---------------------|----------------|
| 1.1 | Generate new certificate | Azure Portal → Key Vault → Certificates → Generate | `az keyvault certificate create --vault-name <vault> --name <cert-name> --policy @policy.json` |
| 1.2 | Download public key (.cer) | Key Vault → Certificate → Download in CER format | `az keyvault certificate download --vault-name <vault> --name <cert> --file cert.cer` |
| 1.3 | Create change ticket | ITSM | N/A |
| 1.4 | Notify affected teams | Teams: #agent-ops | N/A |
| 1.5 | Schedule change window | Outlook/Teams | N/A |

**Week 2: Enterprise Tenant Update (Change Window)**

| Step | Action | Azure/M365 Location | CLI/PowerShell |
|------|--------|---------------------|----------------|
| 2.1 | Navigate to app registration | Entra ID → App registrations → <Agent App> | N/A |
| 2.2 | Add new certificate | Certificates & secrets → Upload certificate | `az ad app credential reset --id <app-id> --cert @cert.cer --append` |
| 2.3 | Verify certificate added | Certificates list shows both old and new | `az ad app credential list --id <app-id>` |
| 2.4 | Update agent config to use new cert | Copilot Studio → Connections or Key Vault reference | Update Key Vault reference version |
| 2.5 | Test Enterprise-only auth | Agent test with Enterprise user | N/A |

**Week 2: Retail Tenant Update (Change Window)**

| Step | Action | Azure/M365 Location | Notes |
|------|--------|---------------------|-------|
| 3.1 | Navigate to service principal | Entra ID (Retail) → Enterprise applications → <Agent App> | N/A |
| 3.2 | Update certificate | Properties → Manage → Certificates | May require re-consent |
| 3.3 | Test cross-tenant auth | Retail user → Enterprise agent | Token acquisition works |
| 3.4 | Validate all agents functional | Run smoke tests | All pass |

**Week 3: Cleanup (Post-Validation)**

| Step | Action | Timeline |
|------|--------|----------|
| 4.1 | Monitor with both certs active | 7 days |
| 4.2 | Verify no auth failures | Daily check |
| 4.3 | Remove old certificate from Enterprise app reg | Day 8 |
| 4.4 | Remove old certificate from Key Vault | Day 15 (retention) |
| 4.5 | Update documentation | Day 8 |
| 4.6 | Schedule next rotation | Day 8 |

---

##### Automation via Azure Automation

**Automated Certificate Rotation Runbook (PowerShell):**

```powershell
# Azure Automation Runbook: Rotate-AgentCertificate
# Schedule: Monthly check, rotate if expiring within 30 days

param(
    [string]$KeyVaultName = "agent-keyvault-prod",
    [string]$CertificateName = "agent-cross-tenant-cert",
    [string]$AppRegistrationId = "00000000-0000-0000-0000-000000000000",
    [int]$ExpirationThresholdDays = 30
)

# Connect using Managed Identity
Connect-AzAccount -Identity

# Check certificate expiration
$cert = Get-AzKeyVaultCertificate -VaultName $KeyVaultName -Name $CertificateName
$expiresIn = ($cert.Expires - (Get-Date)).Days

if ($expiresIn -le $ExpirationThresholdDays) {
    Write-Output "Certificate expires in $expiresIn days. Initiating rotation..."
    
    # Generate new certificate
    $policy = Get-AzKeyVaultCertificatePolicy -VaultName $KeyVaultName -Name $CertificateName
    $newCertOp = Add-AzKeyVaultCertificate -VaultName $KeyVaultName -Name "$CertificateName-new" -CertificatePolicy $policy
    
    # Wait for certificate creation
    while ($newCertOp.Status -ne "completed") {
        Start-Sleep -Seconds 10
        $newCertOp = Get-AzKeyVaultCertificateOperation -VaultName $KeyVaultName -Name "$CertificateName-new"
    }
    
    # Get new certificate
    $newCert = Get-AzKeyVaultCertificate -VaultName $KeyVaultName -Name "$CertificateName-new"
    
    # Add to app registration (append, don't replace)
    $certValue = [System.Convert]::ToBase64String($newCert.Certificate.GetRawCertData())
    New-AzADAppCredential -ApplicationId $AppRegistrationId -CertValue $certValue -EndDate $newCert.Expires
    
    # Send notification
    $webhookUri = Get-AutomationVariable -Name "TeamsWebhookUri"
    $body = @{
        "@type" = "MessageCard"
        "summary" = "Certificate Rotation Initiated"
        "sections" = @(
            @{
                "activityTitle" = "Agent Certificate Rotation"
                "facts" = @(
                    @{ "name" = "Certificate"; "value" = $CertificateName }
                    @{ "name" = "Old Expiry"; "value" = $cert.Expires.ToString() }
                    @{ "name" = "New Expiry"; "value" = $newCert.Expires.ToString() }
                    @{ "name" = "Action Required"; "value" = "Update Retail Tenant within 7 days" }
                )
            }
        )
    }
    Invoke-RestMethod -Uri $webhookUri -Method Post -Body ($body | ConvertTo-Json -Depth 10) -ContentType "application/json"
    
    Write-Output "Certificate rotation initiated. Manual steps required for Retail Tenant."
} else {
    Write-Output "Certificate valid for $expiresIn days. No action needed."
}
```

**Automation Schedule:**
- **Frequency:** Weekly (Sunday 02:00 UTC)
- **Alert:** If expiration <30 days, sends Teams notification
- **Action:** Auto-generates new cert, adds to Enterprise app reg
- **Manual Step:** Retail Tenant update still requires manual action (cross-tenant admin)

---

##### Rollback Procedure

| Step | Action | Timeline |
|------|--------|----------|
| R1 | Identify authentication failures | T+0 |
| R2 | Revert Enterprise app reg to old cert | T+5 min |
| R3 | Revert Retail service principal | T+10 min |
| R4 | Disable new cert in Key Vault | T+15 min |
| R5 | Validate auth restored | T+20 min |
| R6 | Create incident ticket | T+25 min |

---

##### Verification Checklist

| Verification | Method | Expected Result |
|--------------|--------|-----------------|
| Token acquisition (Enterprise) | Graph Explorer with app auth | Token issued |
| Token acquisition (Cross-tenant) | API call from Retail to Enterprise | Token issued |
| Agent functionality | Smoke test suite | All pass |
| Audit logging | Entra sign-in logs | Auth events logged |
| No client secret usage | Sign-in logs filter | 0 secret-based auths |

---

#### Runbook 3: Agent Incident Response (OWASP-Aligned)

**Runbook ID:** RB-INCIDENT-001  
**Version:** 2.0  
**Last Updated:** 2026-04-07  
**Owner:** Security Operations Lead

---

##### Trigger Conditions
- Agent producing harmful/incorrect outputs (ASI01: Goal Hijack)
- Unexpected data access or exfiltration (ASI03: Privilege Abuse)
- Agent unavailable affecting business operations
- Security alert from Agent 365 or Defender
- User/stakeholder report of suspicious behavior

##### Severity Classification (OWASP-Aligned)

| Severity | Definition | OWASP Risk Alignment | Response Time | Escalation |
|----------|------------|---------------------|---------------|------------|
| **P1** | Data exposure, harmful outputs, security breach | ASI01, ASI03, ASI09, ASI10 | 15 min | CISO + CoE Director |
| **P2** | Agent unavailable, business impact | ASI08 (Cascading) | 30 min | AI Operations Lead |
| **P3** | Degraded performance, intermittent issues | ASI02, ASI06 | 4 hours | AI Operations |
| **P4** | Minor issues, cosmetic, workaround available | — | 24 hours | AI Operations |

---

##### Decision Tree: Incident Classification

```
                    ┌─────────────────────────────────┐
                    │ What type of incident?          │
                    └───────────────┬─────────────────┘
                                    │
    ┌───────────────┬───────────────┼───────────────┬───────────────┐
    ▼               ▼               ▼               ▼               ▼
┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
│ Data   │    │ Agent  │    │ Unauth-│    │ Perform│    │ Suspi- │
│ Expo-  │    │ produc-│    │ orized │    │ ance/  │    │ cious  │
│ sure   │    │ ing    │    │ access │    │ Availa-│    │ behav- │
│        │    │ wrong  │    │ attemp-│    │ bility │    │ ior    │
│        │    │ outputs│    │ ted    │    │        │    │        │
└───┬────┘    └───┬────┘    └───┬────┘    └───┬────┘    └───┬────┘
    │             │             │             │             │
    ▼             ▼             ▼             ▼             ▼
┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
│ P1     │    │ P1/P2  │    │ P1     │    │ P2/P3  │    │ P1/P2  │
│ IMMED. │    │ Assess │    │ IMMED. │    │ Assess │    │ Investi│
│ DISABLE│    │ impact │    │ DISABLE│    │ scope  │    │ gate   │
└────────┘    └────────┘    └────────┘    └────────┘    └────────┘
```

---

##### OWASP Risk-Specific Response Procedures

**ASI01: Agent Goal Hijack (Prompt Injection)**

| Phase | Action | Azure/M365 Location | Time |
|-------|--------|---------------------|------|
| Detect | Alert from Agent 365 Threat Protection or user report | Agent 365 Admin Center → Threats | T+0 |
| Contain | Disable agent | Copilot Studio → Agent → Unpublish | T+5 min |
| Investigate | Review conversation transcripts (if available) | Copilot Studio → Analytics or Dataverse | T+15 min |
| Investigate | Query Defender for injection patterns | Defender XDR → Advanced Hunting: `CopilotAgentActivity \| where PromptContent contains_any("ignore previous", "system prompt")` | T+20 min |
| Remediate | Update system prompt with additional guardrails | Copilot Studio → Agent → Instructions | T+1 hour |
| Remediate | Enable/strengthen content filtering | Copilot Studio → Settings → Moderation | T+1 hour |
| Recover | Staged re-enablement with monitoring | Publish → 10% traffic → monitor → 100% | T+2 hours |

**ASI03: Identity & Privilege Abuse**

| Phase | Action | Azure/M365 Location | Time |
|-------|--------|---------------------|------|
| Detect | Anomalous access pattern in Entra sign-in logs | Entra ID → Sign-in logs → Filter by app | T+0 |
| Contain | Revoke agent identity tokens | Entra ID → Enterprise apps → Revoke sessions | T+5 min |
| Contain | Block Conditional Access | Entra ID → CA → Create blocking policy | T+10 min |
| Investigate | Query permissions used | Graph Explorer: `GET /servicePrincipals/{id}/oauth2PermissionGrants` | T+20 min |
| Investigate | Check for over-permissioned scopes | Compare granted vs. required permissions | T+30 min |
| Remediate | Remove excessive permissions | Entra ID → App registrations → API permissions | T+1 hour |
| Remediate | Implement least privilege | Update agent to request minimum scopes | T+2 hours |
| Recover | Re-enable with new permissions | Update CA policy, monitor | T+3 hours |

**ASI08: Cascading Failures**

| Phase | Action | Azure/M365 Location | Time |
|-------|--------|---------------------|------|
| Detect | Multiple agents failing simultaneously | Agent 365 Dashboard → Health | T+0 |
| Contain | Activate circuit breakers | Azure APIM → Policies → Circuit breaker | T+2 min |
| Contain | Isolate affected agents | Copilot Studio → Disable downstream agents | T+5 min |
| Investigate | Identify root cause (upstream failure) | Application Insights → Dependency failures | T+15 min |
| Investigate | Check M365 Service Health | admin.microsoft.com → Service Health | T+5 min |
| Remediate | Fix root cause or wait for service restoration | Depends on cause | Variable |
| Recover | Staged re-enablement, upstream first | Enable in order of dependency chain | T+1 hour |

**ASI10: Rogue Agent**

| Phase | Action | Azure/M365 Location | Time |
|-------|--------|---------------------|------|
| Detect | Unregistered agent activity or behavioral anomaly | Agent 365 → Shadow Agent Detection | T+0 |
| Contain | Disable agent immediately | Copilot Studio → Unpublish | T+2 min |
| Contain | Revoke all agent credentials | Entra ID → App reg → Certificates & secrets → Delete all | T+5 min |
| Contain | Block at network level (if custom engine) | Azure NSG / Firewall rules | T+10 min |
| Investigate | Forensic analysis of agent history | Defender XDR → Timeline | T+30 min |
| Investigate | Determine if configuration drift or compromise | Compare manifest to approved version | T+1 hour |
| Remediate | Rebuild agent from known-good state | CoE Engineering | T+4 hours |
| Recover | Deploy fresh agent with enhanced monitoring | Standard deployment + extra logging | T+8 hours |

---

##### Communication Templates

**Initial Notification (P1/P2):**
```
AGENT INCIDENT ALERT - [SEVERITY]
Agent: [Agent Name]
Tenant(s): [Affected Tenants]
Time Detected: [Timestamp]
Impact: [Brief description]
Current Status: [Investigating/Contained/Mitigating]
Incident Commander: [Name]
Next Update: [Time]
Bridge: [Teams/Zoom link]
```

**Status Update:**
```
AGENT INCIDENT UPDATE - [SEVERITY] - [Status]
Incident ID: [ID]
Agent: [Agent Name]
Summary: [What happened, what we know]
Actions Taken: [List]
Current Status: [Investigating/Contained/Resolved]
ETA to Resolution: [Time or Unknown]
Next Update: [Time]
```

---

##### Escalation Matrix

| Condition | Escalate To | Method | SLA |
|-----------|-------------|--------|-----|
| P1 not contained in 15 min | CoE Director | Phone + Teams | Immediate |
| Data breach confirmed | CISO + Legal + DPO | Security hotline | Immediate |
| Cross-tenant impact | Both Tenant Admins | Phone + Teams | 30 min |
| Multiple agents affected | Executive Steering | Email + Phone | 1 hour |
| Media/PR risk | Communications Lead | Phone | 1 hour |
| Vendor platform issue | Microsoft Premier Support | Support portal | Per contract |

---

##### Post-Incident Checklist

| Item | Owner | Due | Documentation |
|------|-------|-----|---------------|
| Incident report (5 Whys) | Incident Commander | 72 hours | SharePoint → Incidents |
| Root cause analysis | CoE Engineering | 1 week | Technical appendix |
| Update runbooks | CoE Standards | 2 weeks | This document |
| Implement preventive measures | Varies | 4 weeks | ITSM ticket |
| Stakeholder communication | CoE Director | 1 week | Email summary |
| Lessons learned session | All involved | 2 weeks | Meeting recording |

---

#### Runbook 4: Cost Anomaly Response

**Runbook ID:** RB-COST-001  
**Version:** 2.0  
**Last Updated:** 2026-04-07  
**Owner:** FinOps Lead

---

##### Trigger Conditions
- Cost monitoring alert: >20% deviation from 7-day baseline
- Copilot Credit consumption approaching 80% of allocation
- Unexpected Azure infrastructure cost spike
- Franchise chargeback dispute

##### Decision Tree: Anomaly Classification

```
                    ┌─────────────────────────────────┐
                    │ What type of cost anomaly?      │
                    └───────────────┬─────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          ▼                         ▼                         ▼
    ┌───────────┐           ┌───────────┐           ┌───────────┐
    │ Copilot   │           │ Azure     │           │ Licensing │
    │ Credits   │           │ Infra     │           │ (Copilot/ │
    │ Spike     │           │ Cost      │           │ Agent 365)│
    └─────┬─────┘           └─────┬─────┘           └─────┬─────┘
          │                       │                       │
          ▼                       ▼                       ▼
    ┌───────────┐           ┌───────────┐           ┌───────────┐
    │ Check:    │           │ Check:    │           │ Check:    │
    │ - Which   │           │ - Which   │           │ - License │
    │   agents  │           │   resource│           │   count   │
    │ - Feature │           │ - Scale   │           │ - Assign- │
    │   usage   │           │   events  │           │   ments   │
    └─────┬─────┘           └─────┬─────┘           └─────┬─────┘
          │                       │                       │
    ┌─────┴─────┐           ┌─────┴─────┐           ┌─────┴─────┐
    │Legitimate │           │ Normal    │           │ Expected  │
    │  spike?   │           │ scaling?  │           │ growth?   │
    └─────┬─────┘           └─────┬─────┘           └─────┬─────┘
      Yes │ No                Yes │ No                Yes │ No
          ▼                       ▼                       ▼
    ┌───────────┐           ┌───────────┐           ┌───────────┐
    │ Update    │           │ Review    │           │ Review    │
    │ baseline  │           │ auto-     │           │ allocation│
    │ OR        │           │ scaling   │           │ OR        │
    │ Investigate│          │ policies  │           │ Investigate│
    └───────────┘           └───────────┘           └───────────┘
```

---

##### Step-by-Step Procedure: Copilot Credit Spike

**Phase 1: Detection & Triage (0-30 min)**

| Step | Action | Location | Expected Outcome |
|------|--------|----------|------------------|
| 1.1 | Review PPAC alert details | Power Platform Admin Center → Licensing → Copilot Studio | Identify environment, time range |
| 1.2 | Identify top consuming agents | PPAC → Copilot Studio → Agent consumption | List of agents by credit usage |
| 1.3 | Compare to baseline | Historical usage (7-day, 30-day) | Deviation % |
| 1.4 | Classify: Legitimate vs. Anomaly | See decision criteria below | Classification |

**Classification Criteria:**

| Factor | Legitimate Spike | Anomaly |
|--------|-----------------|---------|
| New agent launched | Expected increase | N/A |
| Marketing campaign | Expected increase | N/A |
| Seasonal event (Black Friday) | Expected increase | N/A |
| No known trigger | Investigate | Likely anomaly |
| Single agent, extreme spike | Investigate | Likely runaway |
| Multiple tenants simultaneously | Platform issue? | Check M365 health |

**Phase 2: Investigation (30 min - 2 hours)**

| Step | Action | What to Look For |
|------|--------|------------------|
| 2.1 | Drill into top agent | PPAC → Agent → Usage details | Feature breakdown (classic, generative, graph, actions) |
| 2.2 | Check for recent deployments | ITSM → Recent changes | New version, config change |
| 2.3 | Review agent logic | Copilot Studio → Topics/Flows | Infinite loops, excessive tool calls |
| 2.4 | Check for runaway automation | Azure Monitor → Function logs | Autonomous triggers firing repeatedly |
| 2.5 | Contact franchise (if applicable) | Teams/Email | Unexpected usage pattern |

**Phase 3: Mitigation**

| Scenario | Action | PPAC Location |
|----------|--------|---------------|
| Runaway agent (infinite loop) | Disable agent immediately | Copilot Studio → Unpublish |
| Over-consumption (legitimate) | Add credits or throttle | PPAC → Licensing → Allocate |
| Inefficient agent design | Optimize (cache, reduce graph calls) | CoE Engineering review |
| Franchise over-budget | Notify, adjust allocation | Franchise Council escalation |

**Phase 4: Resolution & Documentation**

| Step | Action | Due |
|------|--------|-----|
| 4.1 | Implement fix | Immediate |
| 4.2 | Update baseline if legitimate growth | 1 day |
| 4.3 | Adjust alerts/thresholds | 1 day |
| 4.4 | Document in cost tracking | 1 day |
| 4.5 | Report to Governance Board (if >10% budget impact) | Monthly meeting |

---

##### Cost Optimization Quick Reference

| Cost Driver | Optimization Action | Potential Savings |
|-------------|---------------------|-------------------|
| Tenant Graph Grounding (10 credits) | Disable for FAQ agents; use standard SharePoint | 80% per interaction |
| Excessive generative answers | Route simple queries to classic answers | 50% per query |
| Agent flow actions (13/100) | Batch actions; optimize flow logic | 20-40% |
| High-frequency triggers | Add debouncing; increase polling interval | 30-50% |
| Premium AI tools (100 credits) | Use standard tools unless reasoning required | 85% per use |

---

##### Escalation Triggers

| Condition | Escalate To | Action |
|-----------|-------------|--------|
| >50% budget consumed by mid-month | CoE Director | Review allocation |
| Single agent >20% of total budget | Technical Review Board | Efficiency review |
| Franchise exceeds allocation | Franchise Council | Cost discussion |
| Projected overage >$10K/month | Executive Steering | Budget approval |
| Suspected abuse/fraud | Security Operations | Investigation |

---

##### Verification & Monitoring

| Metric | Target | Alert Threshold | Dashboard |
|--------|--------|-----------------|-----------|
| Daily credit consumption | Within 5% of baseline | >20% deviation | PPAC → Copilot Studio |
| Agent efficiency (credits/conversation) | <15 credits avg | >25 credits | Custom Power BI |
| Environment utilization | <80% allocated | >90% | PPAC |
| Month-to-date spend | On budget | >110% projected | Azure Cost Management + PPAC |

---

## 5. Detailed Component Descriptions
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

## 5. Detailed Component Descriptions

### 5.1 Identity & Access Components

#### 5.1.1 Entra ID (Azure AD)

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

#### 5.1.2 App Registrations

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

#### 5.1.3 Managed Identities

**Role:** Eliminate secrets for Azure-hosted agent components.

**Types:**
- **System-Assigned:** Tied to specific Azure resource lifecycle
- **User-Assigned:** Shared across resources, independent lifecycle

**Cross-Tenant Pattern:**
- Use Workload Identity Federation for cross-tenant scenarios
- Federate user-assigned managed identity with multi-tenant app registration
- Eliminates client secret management for cross-tenant auth

### 5.2 Agent Runtime Components

#### 5.2.1 Declarative Agents

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

#### 5.2.2 Custom Engine Agents

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

#### 5.2.3 Copilot Studio Agents

**Architecture:**
- Low-code visual builder
- Power Platform foundation (Dataverse, Power Automate)
- Microsoft-managed orchestration

**Governance:**
- Environment-based isolation
- DLP policies on connectors
- Tenant-level and environment-level controls

### 5.3 Integration Components

#### 5.3.1 API Plugins

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

#### 5.3.2 RAG (Retrieval-Augmented Generation)

> **Reference:** [Enhance AI responses with Retrieval Augmented Generation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/retrieval-augmented-generation)

**Role:** Ground agent responses in enterprise knowledge, reducing hallucination and ensuring accuracy through retrieval from trusted organizational content.

**RAG Architecture in Copilot Studio:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        USER QUERY                                    │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   QUERY OPTIMIZATION ENGINE                          │
│  • Clarifies meaning, adds context (last 10 turns)                  │
│  • Generates search-friendly queries                                │
└─────────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   SharePoint  │    │   Dataverse   │    │   Azure AI    │
│   / OneDrive  │    │   Upload/     │    │    Search     │
│               │    │   Tables      │    │               │
└───────────────┘    └───────────────┘    └───────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   SUMMARIZATION ENGINE                               │
│  • Synthesizes top 3 results per source                             │
│  • Applies custom instructions, generates citations                 │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   GROUNDED RESPONSE                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

##### Knowledge Source Comparison for Retail

| Source | Best For | Authentication | Cost (Credits) | Retail Use Case |
|--------|----------|----------------|----------------|-----------------|
| **SharePoint / OneDrive** | Internal docs, policies, SOPs | Entra ID delegated | Standard (free with M365 Copilot) | Store operations manuals, HR policies |
| **Tenant Graph Grounding** | Cross-M365 semantic search | Entra ID delegated | **10 credits/interaction** | Enterprise-wide knowledge, Graph connectors |
| **Dataverse Upload** | Static reference docs | None | Standard | Product catalogs, training materials |
| **Dataverse Tables** | Structured business records | Entra ID delegated | Standard | Inventory data, pricing tables |
| **Azure AI Search** | Custom vector indexes | Configured endpoint | Standard + Azure costs | Product search, recommendation engines |
| **Public Websites** | External FAQ, support | None | Standard | Vendor documentation, industry standards |
| **Graph Connectors** | Enterprise apps (ServiceNow, Confluence) | Entra ID delegated | 10 credits (enhanced) | IT tickets, knowledge bases |

---

##### Tenant Graph Grounding Configuration

> **Cost:** 10 Copilot Credits per graph-grounded interaction

Tenant Graph Grounding provides **semantic search across the entire Microsoft Graph**, including data indexed via Graph Connectors. This significantly improves retrieval quality for SharePoint-grounded agents.

**When to Enable:**
- ✅ Agent needs access to content across multiple SharePoint sites
- ✅ Graph Connectors are configured (ServiceNow, Salesforce, etc.)
- ✅ Response quality is critical (customer-facing agents)
- ✅ Cost of 10 credits/interaction is acceptable for the use case

**When to Avoid:**
- ❌ Agent only needs a single, well-scoped knowledge source
- ❌ High-volume, low-value interactions (simple FAQ)
- ❌ Cost optimization is a priority

**Configuration:**
```
Agent Settings → Knowledge → Tenant Graph Grounding
├── Enable: Yes
├── Enhanced search results: On
└── Maximum file size: 200 MB (vs. 15 MB standard)
```

---

##### Chunking Strategies for Retail Documents

| Document Type | Recommended Chunk Size | Overlap | Rationale |
|--------------|------------------------|---------|-----------|
| **Product Catalogs** | 500-800 tokens | 100 tokens | Product descriptions are self-contained |
| **SOPs / Procedures** | 1000-1500 tokens | 200 tokens | Steps should stay together for context |
| **Policy Manuals** | 800-1200 tokens | 150 tokens | Policy clauses need surrounding context |
| **Training Materials** | 1200-1500 tokens | 200 tokens | Learning objectives span multiple paragraphs |
| **FAQ Documents** | 300-500 tokens | 50 tokens | Q&A pairs are atomic units |

**Retail-Specific Chunking Considerations:**
- **SKU references:** Ensure product codes stay with descriptions
- **Price tables:** Keep pricing rows with product context
- **Seasonal content:** Tag chunks with validity dates for filtering
- **Store-specific content:** Include store/region metadata in chunk

---

##### Multi-Language Considerations for Global Retail

| Scenario | Approach | Trade-offs |
|----------|----------|------------|
| **Single-language index, multi-language queries** | Use Azure AI Search with language analyzers | Simple; may miss nuanced translations |
| **Per-language indexes** | Separate vector indexes per language | High quality; higher cost and maintenance |
| **Real-time translation** | Translate query → English index → translate response | Flexible; adds latency |
| **Multilingual embeddings** | Use multilingual embedding models (e.g., E5-multilingual) | Best quality; requires custom index |

**Recommendation for Global Retail:**
- Use **per-language SharePoint sites** with tenant graph grounding for common languages
- For long-tail languages, implement **real-time translation** via Azure AI Translator in custom flows

---

##### RAG Performance Optimization

| Optimization | Implementation | Impact |
|-------------|----------------|--------|
| **Response caching** | Cache frequent query results (TTL: 1-24h) | Reduce credits 40-60% |
| **Query classification** | Route simple queries to classic answers | Avoid RAG overhead for FAQ |
| **Source pre-filtering** | Limit knowledge sources per topic | Reduce retrieval time |
| **Chunk count tuning** | Reduce from default 3 to 1-2 for simple queries | Lower token usage |
| **Index optimization** | Remove stale content; deduplicate | Improve retrieval precision |

---

##### Decision Criteria: Which RAG Approach for Retail?

| Scenario | Recommended Approach | Cost Profile |
|----------|---------------------|--------------|
| **Store FAQ agent (single site)** | SharePoint + standard search | Low (2 credits/response) |
| **Enterprise knowledge agent** | Tenant Graph Grounding | Medium-High (12 credits/response) |
| **Product recommendation agent** | Azure AI Search (custom vectors) | Medium (2 credits + Azure costs) |
| **Customer service agent (multi-source)** | Hybrid: SharePoint + Graph Connectors + Tenant Graph | High (12+ credits/response) |
| **Franchise operations agent** | Dataverse Upload + per-region SharePoint | Medium (2 credits/response) |

---

##### Security Considerations

- **ACL Trimming:** SharePoint/OneDrive results respect user permissions; agent only retrieves what the user can access
- **Sensitivity Labels:** RAG inherits classification from source documents
- **Cross-Tenant Isolation:** Tenant Graph Grounding is tenant-scoped; no cross-tenant retrieval in native mode
- **No Training Data:** Customer data is never used to train language models

### 5.4 Retail-Specific Components

#### 5.4.1 Digital Retail Stack Integration

| System | Agent Use Cases | Integration Pattern |
|--------|-----------------|---------------------|
| **POS Systems** | Transaction queries, returns processing | API plugin with read-only access |
| **Inventory Management** | Stock levels, allocation, transfers | Real-time API with event triggers |
| **Workforce Management** | Schedule queries, shift swaps, time-off | Bidirectional API with approval workflows |
| **Customer Data Platform** | Customer profiles, preferences, history | Read-only with PII masking |
| **Digital Retail Offers** | Promotion rules, pricing, campaigns | Read-only with caching |

#### 5.4.2 Model Context Protocol (MCP)

> **Reference:** [Model Context Protocol Specification](https://modelcontextprotocol.io/specification/draft/basic/authorization)  
> **Status:** Enterprise auth (OAuth 2.1) shipping Q2 2026; MCP Registry planned Q4 2026

**Role:** Provides agents with a standardized protocol for connecting to external data sources, tools, and services—enabling consistent, secure integration across the agent ecosystem.

---

##### MCP Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         COPILOT STUDIO AGENT                         │
│                    (MCP Client / Host Application)                   │
└─────────────────────────────────────────────────────────────────────┘
                              │
                    MCP Protocol (JSON-RPC 2.0)
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  MCP Server   │    │  MCP Server   │    │  MCP Server   │
│  (Inventory)  │    │    (POS)      │    │  (Workforce)  │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ • Resources   │    │ • Resources   │    │ • Resources   │
│ • Tools       │    │ • Tools       │    │ • Tools       │
│ • Prompts     │    │ • Prompts     │    │ • Prompts     │
└───────────────┘    └───────────────┘    └───────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   Retail      │    │     POS       │    │   Workforce   │
│   Inventory   │    │    System     │    │   Management  │
│    System     │    │               │    │    System     │
└───────────────┘    └───────────────┘    └───────────────┘
```

**MCP Server Capabilities:**
- **Resources:** Expose data (inventory levels, product info, schedules) for agent consumption
- **Tools:** Provide actions the agent can invoke (place order, update schedule, process return)
- **Prompts:** Pre-configured prompt templates for domain-specific tasks

---

##### 2026 MCP Roadmap: Enterprise Readiness

| Feature | Status | Timeline | Impact on Retail |
|---------|--------|----------|------------------|
| **OAuth 2.1 with PKCE** | Specification draft | GA Q2 2026 | Secure token-based auth for MCP servers |
| **Enterprise IdP Integration** | Roadmap | Q2-Q3 2026 | Entra ID, Okta integration for SSO |
| **MCP Registry** | Roadmap | Q4 2026 | Verified server directory with security audits |
| **Audit Trails** | Roadmap | Q3 2026 | Compliance logging for all MCP interactions |
| **Multi-tenant Support** | Roadmap | Q4 2026 | Tenant isolation for MCP server resources |

**Current Limitations (Pre-Q2 2026):**
- Authentication relies on static API keys or pre-shared secrets
- No standardized enterprise identity integration
- Audit logging must be implemented per-server
- Cross-tenant MCP requires custom trust configuration

---

##### Retail-Specific MCP Server Patterns

| MCP Server | Resources Exposed | Tools Provided | Use Cases |
|------------|-------------------|----------------|-----------|
| **Inventory Management** | Stock levels by SKU/location, allocation rules, reorder points | Check stock, reserve inventory, trigger reorder | Store associates checking availability, automated replenishment |
| **POS Systems** | Transaction history, register status, tender types | Lookup transaction, process return, void item | Customer service agents, loss prevention queries |
| **Supplier Analytics** | Vendor performance, lead times, order history | Query supplier metrics, flag delays | Merchandising agents, supply chain optimization |
| **Workforce Management** | Schedules, shift coverage, time-off requests | Query schedule, swap shifts, approve time-off | Store manager agents, employee self-service |
| **Pricing & Promotions** | Price rules, active promotions, markdown schedules | Check price, apply discount, validate promo code | Customer service, pricing agents |
| **Customer Data Platform** | Customer profiles, purchase history, preferences | Lookup customer, update preferences | Personalization agents, loyalty programs |

---

##### MCP Server Implementation Example: Inventory Management

```python
# MCP Server: Inventory Management (Python SDK example)
from mcp import MCPServer, Resource, Tool

server = MCPServer("retail-inventory")

@server.resource("inventory/{sku}")
async def get_inventory(sku: str, location: str = None):
    """Get current inventory levels for a SKU."""
    # Query inventory system
    levels = await inventory_db.query(sku, location)
    return {
        "sku": sku,
        "available": levels.available,
        "reserved": levels.reserved,
        "in_transit": levels.in_transit,
        "locations": levels.by_store
    }

@server.tool("reserve_inventory")
async def reserve_inventory(sku: str, quantity: int, store_id: str):
    """Reserve inventory for a customer order."""
    result = await inventory_db.reserve(sku, quantity, store_id)
    return {"reservation_id": result.id, "expires_at": result.expires}

server.run(port=8080, auth="oauth2")  # Q2 2026: OAuth 2.1 support
```

---

##### MCP Integration with Copilot Studio Agents

| Integration Method | Complexity | When to Use |
|-------------------|------------|-------------|
| **Native MCP Tool** | Low | Copilot Studio GA MCP support (preview in Apps SDK) |
| **HTTP Action → MCP Server** | Medium | Wrap MCP calls in REST API for current compatibility |
| **Custom Engine Agent** | High | Full MCP client control, complex orchestration |
| **Power Automate Flow** | Medium | Workflow-based MCP invocation with approvals |

**Current Copilot Studio Integration (Pre-GA):**
```
Agent Topic
├── HTTP Request Action
│   ├── URL: https://mcp-inventory.retailcorp.com/v1/rpc
│   ├── Method: POST
│   ├── Headers: Authorization: Bearer {api_key}
│   └── Body: {"jsonrpc":"2.0","method":"resource/read","params":{"uri":"inventory/SKU123"}}
└── Parse JSON Response
    └── Return formatted inventory data
```

---

##### MCP vs. Custom API Plugins: Comparison for Retail Integration

| Criterion | MCP Servers | Custom API Plugins |
|-----------|-------------|-------------------|
| **Standardization** | ✅ Open protocol; portable across AI platforms | ❌ Proprietary; tied to Copilot Studio |
| **Discovery** | ✅ Self-describing (capabilities, schemas) | ❌ Manual configuration |
| **Ecosystem** | ✅ Growing library of pre-built servers | ❌ Build or buy each connector |
| **Enterprise Auth** | ⚠️ Q2 2026 (OAuth 2.1) | ✅ OAuth/Entra ID today |
| **Audit Logging** | ⚠️ Q3 2026 (standardized) | ✅ Implement per connector |
| **Multi-Tenant** | ⚠️ Q4 2026 | ✅ Cross-tenant app reg works today |
| **Real-Time Context** | ✅ Persistent connection; live updates | ❌ Request/response only |
| **Cost** | Medium (build/host servers) | Low-Medium (connectors exist) |

**Recommendation for Retail:**
- **Use MCP** for net-new integrations where you control the backend (inventory, WFM, proprietary systems)
- **Use Custom API Plugins** for integrations requiring enterprise auth today (Graph, Dynamics 365, ServiceNow)
- **Hybrid approach:** Build MCP server as abstraction layer; expose via API plugin until native MCP GA

---

##### Security Considerations for MCP in Multi-Tenant Retail

| Risk | Mitigation |
|------|------------|
| **Credential exposure** | Store API keys in Azure Key Vault; rotate regularly; migrate to OAuth 2.1 Q2 2026 |
| **Cross-tenant data leakage** | Implement tenant isolation at MCP server layer; validate tenant context in every request |
| **Tool misuse** | Define explicit tool allowlists per agent; log all tool invocations |
| **Resource over-exposure** | Scope MCP server resources to specific datasets; implement RBAC within server |
| **Supply chain risk** | Vet third-party MCP servers; prefer self-hosted or verified registry servers (Q4 2026) |

**Multi-Tenant MCP Architecture:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                     ENTERPRISE TENANT                                │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              MCP Gateway (API Management)                      │  │
│  │  • Tenant context validation                                   │  │
│  │  • Request/response logging                                    │  │
│  │  • Rate limiting per tenant                                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│         ┌────────────────────┼────────────────────┐                 │
│         ▼                    ▼                    ▼                 │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐           │
│  │ MCP Server  │     │ MCP Server  │     │ MCP Server  │           │
│  │ (Inventory) │     │   (WFM)     │     │  (Pricing)  │           │
│  └─────────────┘     └─────────────┘     └─────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
                              │
            Cross-Tenant Trust (B2B / App Consent)
                              │
┌─────────────────────────────┼───────────────────────────────────────┐
│                     RETAIL TENANT                                    │
│                              ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                   Copilot Studio Agent                         │  │
│  │  • Accesses MCP via API plugin (pre-GA)                        │  │
│  │  • Native MCP support (GA Q2-Q3 2026)                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

### 5.5 Multi-Agent Orchestration

> **Reference:** [Explore multi-agent orchestration patterns](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns)  
> **Status:** Generally Available (April 2026)

**Role:** Enable complex workflows where multiple specialized agents collaborate, delegate tasks, and coordinate responses—essential for retail scenarios spanning inventory, pricing, workforce, and customer service.

---

##### Multi-Agent Architecture Patterns

| Pattern | Description | Use Case | Complexity |
|---------|-------------|----------|------------|
| **Inline (Child) Agents** | Small, reusable workflows within the same agent (topics as subroutines) | Shared utilities: translate text, format currency, validate inputs | Low |
| **Connected Agents** | Separate agents with own orchestration, tools, knowledge; parent delegates to child | Domain specialists: inventory agent ↔ pricing agent ↔ WFM agent | Medium |
| **Agent-to-Agent (A2A)** | Open protocol for cross-platform agent communication (1P, 2P, 3P) | Third-party integrations: supplier agents, logistics partners | High |
| **Fabric Data Agents** | Connect to Microsoft Fabric for enterprise data/analytics reasoning | Business intelligence: sales analysis, demand forecasting | Medium |

---

##### Inline vs. Connected Agents: Decision Criteria

**Use Inline (Child) Agents When:**
- ✅ Task is simple and well-scoped (single responsibility)
- ✅ Shared context with parent agent is sufficient
- ✅ No separate governance or access control needed
- ✅ Reusable across multiple topics in the same agent

**Use Connected Agents When:**
- ✅ Task requires its own knowledge sources or tools
- ✅ Different governance rules or access controls apply
- ✅ Capability should be reusable across many parent agents
- ✅ Domain expertise warrants dedicated agent (e.g., pricing specialist)

---

##### Retail Multi-Agent Scenario: Store Operations Orchestrator

**Business Need:** A single store manager agent that coordinates inventory, pricing, and workforce questions without requiring the manager to know which system to query.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STORE OPERATIONS ORCHESTRATOR                     │
│                    (Main Copilot Studio Agent)                       │
│                                                                      │
│  User: "Can we run a flash sale on summer inventory this weekend   │
│         and do we have enough staff to cover the rush?"             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                GENERATIVE ORCHESTRATION                      │    │
│  │  1. Parse intent: pricing + inventory + workforce            │    │
│  │  2. Plan: Query inventory → Check pricing rules → Check WFM  │    │
│  │  3. Execute: Delegate to connected agents                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  INVENTORY      │  │    PRICING      │  │   WORKFORCE     │
│     AGENT       │  │     AGENT       │  │     AGENT       │
│                 │  │                 │  │                 │
│ Knowledge:      │  │ Knowledge:      │  │ Knowledge:      │
│ - Stock levels  │  │ - Promo rules   │  │ - Schedules     │
│ - Allocations   │  │ - Margin floor  │  │ - Availability  │
│                 │  │                 │  │                 │
│ Tools:          │  │ Tools:          │  │ Tools:          │
│ - Check stock   │  │ - Validate sale │  │ - Check shifts  │
│ - Reserve units │  │ - Set discount  │  │ - Request cover │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AGGREGATED RESPONSE                               │
│                                                                      │
│  "You have 342 summer items in stock (valued at $18,400). A 25%     │
│   markdown is within margin guidelines. Saturday coverage is at     │
│   85%; you may want to request 2 additional associates for the      │
│   afternoon shift. Shall I draft the shift request?"                │
└─────────────────────────────────────────────────────────────────────┘
```

---

##### Connected Agent Configuration

**Parent Agent Setup:**
```
Agent: Store Operations Orchestrator
├── Connected Agents
│   ├── Inventory Agent
│   │   ├── Description: "Answers inventory questions; can check stock and reserve units"
│   │   ├── When to invoke: "User asks about stock, inventory, availability, or allocation"
│   │   └── Data handoff: Pass store_id, sku_list from conversation context
│   │
│   ├── Pricing Agent
│   │   ├── Description: "Validates promotions and discounts against margin rules"
│   │   ├── When to invoke: "User asks about pricing, discounts, promotions, or margins"
│   │   └── Data handoff: Pass product_category, proposed_discount
│   │
│   └── Workforce Agent
│       ├── Description: "Checks schedules and manages shift coverage"
│       ├── When to invoke: "User asks about staffing, schedules, or shift coverage"
│       └── Data handoff: Pass store_id, date_range
```

---

##### Cross-Tenant Multi-Agent Trust

In multi-tenant retail (enterprise + franchise tenants), connected agents may span tenant boundaries.

| Scenario | Trust Model | Configuration |
|----------|-------------|---------------|
| **Same tenant** | Implicit (shared environment) | Standard connected agent setup |
| **Cross-tenant (B2B)** | Explicit (app consent) | Register agent as multi-tenant app; consent in target tenant |
| **Cross-tenant (A2A)** | Protocol-based | Use Agent-to-Agent protocol with authentication |
| **Third-party agents** | A2A with verification | Verify agent identity; define scope limits |

**Cross-Tenant Connected Agent Pattern:**
```
┌──────────────────────────────┐    ┌──────────────────────────────┐
│     ENTERPRISE TENANT        │    │       RETAIL TENANT          │
│                              │    │                              │
│  ┌────────────────────────┐  │    │  ┌────────────────────────┐  │
│  │   Pricing Agent        │  │    │  │  Store Ops Orchestrator│  │
│  │   (Multi-Tenant App)   │◄─┼────┼──│  (Calls Pricing Agent) │  │
│  │                        │  │    │  │                        │  │
│  │   App ID: abc-123      │  │    │  │   Uses: B2B consent    │  │
│  │   Consent: Retail Ten. │  │    │  │   or A2A protocol      │  │
│  └────────────────────────┘  │    │  └────────────────────────┘  │
│                              │    │                              │
└──────────────────────────────┘    └──────────────────────────────┘
```

---

##### Monitoring & Observability for Multi-Agent Systems

| Layer | What to Monitor | Tools |
|-------|-----------------|-------|
| **Orchestrator** | Request routing, plan execution, latency | Agent 365 Observe, Application Insights |
| **Connected Agents** | Invocation count, response time, errors | Per-agent analytics, Sentinel correlation |
| **Data Handoff** | Payload size, data quality, missing fields | Custom telemetry in handoff topics |
| **End-to-End** | Total response time, user satisfaction | Agent 365 dashboards, Copilot Studio analytics |

**Latency Budget (Recommended):**
| Component | Target | Alert Threshold |
|-----------|--------|-----------------|
| Orchestrator planning | <500ms | >1s |
| Connected agent call | <2s | >4s |
| Total end-to-end | <5s | >8s |

---

##### Failure Isolation & Circuit Breakers

**Problem:** A failing connected agent can cascade failures to the orchestrator and other agents.

**Mitigation Strategies:**
| Strategy | Implementation | Impact |
|----------|----------------|--------|
| **Timeout per agent** | Set max wait time (e.g., 4s) for each connected agent | Prevent slow agents from blocking |
| **Graceful degradation** | If agent unavailable, return partial response with disclaimer | Maintain user experience |
| **Circuit breaker** | After N failures, skip agent for M minutes | Reduce retry storm |
| **Fallback topics** | Define fallback behavior if connected agent fails | Predictable error handling |

**Example: Graceful Degradation Response**
```
"I found that we have 342 summer items in stock, and a 25% markdown 
is within guidelines. However, I couldn't reach the workforce system 
to check staffing. Please check the WFM dashboard or try again in 
a few minutes."
```

---

### 5.7 OWASP Top 10 for Agentic AI Mapping

> **Reference:** [OWASP Top 10 for Agentic Applications (2026)](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)  
> **Microsoft Guidance:** [Addressing the OWASP Top 10 Risks in Agentic AI with Microsoft Copilot Studio](https://www.microsoft.com/en-us/security/blog/2026/03/30/addressing-the-owasp-top-10-risks-in-agentic-ai-with-microsoft-copilot-studio/) (March 2026)

The OWASP Top 10 for Agentic Applications identifies critical security risks specific to autonomous AI systems that plan, act, and make decisions across complex workflows. For multi-tenant retail environments, these risks are amplified by cross-tenant data flows, franchise identity boundaries, and high-volume consumer interactions.

#### 5.7.1 Risk Mapping Overview

| OWASP ID | Risk Name | Severity in Multi-Tenant Retail | Primary Copilot Studio/Agent 365 Mitigation |
|----------|-----------|--------------------------------|---------------------------------------------|
| ASI01 | Agent Goal Hijack | **Critical** | Content filtering, grounding separation, prompt shields |
| ASI02 | Tool Misuse & Exploitation | **Critical** | Predefined actions, connector DLP, constrained tool invocation |
| ASI03 | Identity & Privilege Abuse | **Critical** | Entra Agent ID, Conditional Access, least privilege |
| ASI04 | Agentic Supply Chain Vulnerabilities | **High** | Plugin vetting, signed connectors, secure registry |
| ASI05 | Unexpected Code Execution | **High** | Sandboxed execution, no arbitrary code generation |
| ASI06 | Memory & Context Poisoning | **Medium** | Grounding data governance, RAG access controls |
| ASI07 | Insecure Inter-Agent Communication | **High** | A2A authentication via Agent ID, message integrity |
| ASI08 | Cascading Failures | **High** | Isolated environments, circuit breakers, disable/restrict controls |
| ASI09 | Human–Agent Trust Exploitation | **Medium** | Output validation, authority bias training, approval workflows |
| ASI10 | Rogue Agents | **Critical** | Agent registry, behavioral monitoring, republishing controls |

---

#### 5.7.2 ASI01: Agent Goal Hijack

**OWASP Definition:** Redirecting an agent's goals or plans through injected instructions or poisoned content.

**Multi-Tenant Retail Threat Scenario:**
An attacker embeds malicious instructions in a product description or customer message that causes a cross-tenant inventory agent to disclose pricing strategies or redirect stock allocation decisions. In franchise models, a compromised knowledge source in one franchisee's environment could manipulate an enterprise agent's behavior when queried.

**Copilot Studio / Agent 365 Mitigations:**
- **Content Filtering:** Built-in Azure AI Content Safety screens inputs for prompt injection patterns
- **Grounding Separation:** Knowledge sources are segmented from instruction context
- **Prompt Shields:** System prompt protection prevents user content from overriding agent instructions
- **Agent 365 Threat Protection:** Runtime detection of goal hijacking attempts with alerting

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Anomaly in agent decision patterns (e.g., unusual inventory transfers) | Auto-pause agent; trigger investigation | AI Operations |
| Prompt injection patterns in customer service inputs | Block input; log for review; notify security | Security Operations |
| Cross-tenant query attempting instruction override | Block at tenant boundary; alert CoE | IAM Team |

**KQL Hunt Query (Defender):**
```kusto
CopilotAgentActivity
| where ActivityType == "PromptProcessing"
| where PromptContent contains_any("ignore previous", "disregard instructions", "system prompt")
| where TenantId != HomeTenantId  // Cross-tenant context
| project Timestamp, AgentId, UserId, PromptContent, TenantId
```

---

#### 5.7.3 ASI02: Tool Misuse & Exploitation

**OWASP Definition:** Misusing legitimate tools through unsafe chaining, ambiguous instructions, or manipulated tool outputs.

**Multi-Tenant Retail Threat Scenario:**
A store operations agent legitimately connected to inventory, pricing, and customer systems could be manipulated to chain tools in unintended ways—e.g., using price lookup + inventory update + customer notification to execute an unauthorized promotional discount across all stores. Cross-tenant connector abuse could expose enterprise pricing models to franchise agents.

**Copilot Studio / Agent 365 Mitigations:**
- **Predefined Actions & Connectors:** Agents can only invoke explicitly configured tools
- **Connector DLP Policies:** Block or restrict dangerous connector combinations per environment
- **Tool Output Validation:** Responses are sanitized before further processing
- **Agent 365 Govern Pillar:** Tool usage monitoring and policy enforcement

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Unusual tool chaining sequence (e.g., price→inventory→notify) | Flag for human review before execution | Business Process Owner |
| High-frequency API calls to pricing/inventory systems | Rate limit; investigate for automation abuse | Platform Engineering |
| Cross-tenant connector invocation outside approved patterns | Block; review cross-tenant access settings | IAM Team |

**Recommended DLP Configuration:**
```
Environment: Production-Retail
Blocked Connector Combinations:
├── Pricing API + Mass Notification (without approval)
├── Inventory Write + External Email
└── Customer PII Access + Any External Connector
```

---

#### 5.7.4 ASI03: Identity & Privilege Abuse

**OWASP Definition:** Exploiting delegated trust, inherited credentials, or role chains to gain unauthorized access or actions.

**Multi-Tenant Retail Threat Scenario:**
An agent operating under delegated user permissions in the retail tenant accesses enterprise-tenant resources through over-permissioned cross-tenant trust. Alternatively, a franchise manager's agent inherits corporate pricing privileges through poorly scoped B2B collaboration, enabling unauthorized margin changes.

**Copilot Studio / Agent 365 Mitigations:**
- **Entra Agent ID:** First-class agent identities with dedicated Conditional Access policies
- **Agent-Specific Conditional Access:** Block high-risk agents; require specific compliance posture
- **Least Privilege by Design:** Agents request only necessary Graph/API scopes
- **Cross-Tenant Access Settings:** Explicit trust with granular permission scoping

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Agent identity accessing resources outside defined scope | Block access; review agent permissions | IAM Team |
| Elevated privilege usage during non-business hours | Alert; require re-authentication | Security Operations |
| Cross-tenant token acquisition for unregistered resources | Deny token; investigate app registration | IAM Team |

**Conditional Access Policy Template:**
```
Policy: "Retail Agent Privilege Boundaries"
├── Applies to: All agent identities with blueprint="RetailOps"
├── Target resources: Enterprise Tenant apps (Pricing, Inventory Master)
├── Conditions:
│   ├── Agent risk: High → Block
│   └── Custom attribute: cross_tenant_approved ≠ true → Block
└── Grant: Allow with audit logging
```

---

#### 5.7.5 ASI04: Agentic Supply Chain Vulnerabilities

**OWASP Definition:** Compromised or tampered third-party agents, tools, plugins, registries, or update channels.

**Multi-Tenant Retail Threat Scenario:**
A third-party retail analytics connector used by franchise agents is compromised, exfiltrating sales data to an external endpoint. Alternatively, a malicious update to a shared Power Platform component introduces a backdoor affecting agents across both enterprise and retail tenants.

**Copilot Studio / Agent 365 Mitigations:**
- **Connector Certification:** Use Microsoft-certified connectors where possible
- **Plugin Signing:** Custom connectors require signature validation
- **Agent 365 Secure Pillar:** Vulnerability scanning for agent dependencies
- **Environment Isolation:** Production environments separate from development/test

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Connector update outside change window | Block deployment; require CAB approval | Platform Engineering |
| New external data egress from agent | Investigate immediately; disable agent | Security Operations |
| Dependency vulnerability alert (Defender for Cloud) | Patch within SLA; isolate if critical | AI Engineering |

**Supply Chain Governance Checklist:**
- [ ] All custom connectors signed and stored in secure registry
- [ ] Third-party connectors reviewed quarterly for security posture
- [ ] Plugin updates require security scan before deployment
- [ ] Separate connector approval for production vs. non-production

---

#### 5.7.6 ASI05: Unexpected Code Execution

**OWASP Definition:** Turning agent-generated or agent-invoked code into unintended execution, compromise, or escape.

**Multi-Tenant Retail Threat Scenario:**
An analytics agent generates a query or script that, when executed against a retail data warehouse, performs destructive operations or exfiltrates data. A prompt injection causes an agent to generate and execute code that escapes its sandbox.

**Copilot Studio / Agent 365 Mitigations:**
- **No Arbitrary Code Generation:** Copilot Studio agents use predefined actions, not dynamic code
- **Sandboxed Execution:** Power Platform provides isolated runtime environments
- **Code-First Agent Isolation:** Custom engine agents run in containerized/serverless environments
- **Output Filtering:** Generated content is validated before execution

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Agent attempting to generate SQL/scripts outside approved templates | Block execution; log attempt | AI Engineering |
| Sandbox escape attempt detected | Disable agent immediately; forensic analysis | Security Operations |
| Unusual compute resource consumption | Throttle; investigate for cryptomining/abuse | Platform Engineering |

**Architecture Constraint:**
For retail agents requiring data manipulation, use parameterized stored procedures or pre-approved Power Automate flows—never dynamic SQL or script generation.

---

#### 5.7.7 ASI06: Memory & Context Poisoning

**OWASP Definition:** Corrupting stored context (memory, embeddings, RAG stores) to bias future reasoning and actions.

**Multi-Tenant Retail Threat Scenario:**
An attacker injects misleading information into a SharePoint knowledge base used for RAG grounding, causing store associates' agents to provide incorrect return policies. Embedding poisoning in a shared product catalog could bias inventory recommendations across all franchises.

**Copilot Studio / Agent 365 Mitigations:**
- **Knowledge Source Access Controls:** Respect SharePoint/OneDrive permissions (ACL trimming)
- **Sensitivity Label Inheritance:** RAG sources carry classification to agent responses
- **Embedding Integrity:** Azure AI Search provides versioning and audit trails
- **Grounding Data Governance:** Microsoft Purview tracks data lineage

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Unusual edits to high-value knowledge sources | Alert content owner; review changes | Knowledge Management |
| Agent responses inconsistent with source data | Investigate grounding pipeline; check for poisoning | AI Engineering |
| Embedding index rebuild outside schedule | Verify authorization; check for tampering | Platform Engineering |

**Knowledge Source Governance:**
| Source Type | Write Access | Audit Frequency | Validation |
|-------------|--------------|-----------------|------------|
| Corporate Policy Docs | Restricted (Policy team only) | Real-time | Approval workflow |
| Product Catalog | Automated (PIM system) | Daily | Reconciliation |
| Store Operations KB | Regional managers | Weekly | Spot checks |

---

#### 5.7.8 ASI07: Insecure Inter-Agent Communication

**OWASP Definition:** Spoofing, intercepting, or manipulating agent-to-agent messages due to weak authentication or integrity checks.

**Multi-Tenant Retail Threat Scenario:**
A rogue agent in a franchisee tenant impersonates an enterprise inventory agent, sending fraudulent stock transfer requests to distribution centers. Cross-tenant agent-to-agent flows without proper authentication enable unauthorized data exchange.

**Copilot Studio / Agent 365 Mitigations:**
- **Entra Agent ID for A2A:** Agent-to-agent authentication using first-class identities
- **Token Binding:** Agent tokens bound to specific agent instances
- **Message Signing:** Integrity verification for inter-agent payloads
- **Agent 365 Registry:** Authoritative inventory of legitimate agents per tenant

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| A2A communication from unregistered agent | Block; alert security | IAM Team |
| Message signature validation failure | Reject message; log for investigation | Platform Engineering |
| Cross-tenant A2A without explicit trust relationship | Block at tenant boundary | IAM Team |

**A2A Authentication Pattern:**
```
┌───────────────────┐        ┌───────────────────┐
│ Enterprise Agent  │        │   Retail Agent    │
│ (Agent ID: A1)    │        │   (Agent ID: A2)  │
└─────────┬─────────┘        └─────────┬─────────┘
          │                            │
          │  1. Request token for A2   │
          ├───────────────────────────►│
          │     (signed with A1 cert)  │
          │                            │
          │  2. Validate A1 identity   │
          │     via Agent Registry     │
          │                            │
          │  3. Return scoped response │
          │◄────────────────────────────
          │     (signed with A2 cert)  │
```

---

#### 5.7.9 ASI08: Cascading Failures

**OWASP Definition:** A single fault propagating across agents, tools, and workflows into system-wide impact.

**Multi-Tenant Retail Threat Scenario:**
An error in an enterprise pricing agent cascades to retail store agents, which then propagate incorrect prices to POS systems and customer-facing apps across hundreds of locations. A failed authentication in one tenant's agent triggers retry storms that impact cross-tenant services.

**Copilot Studio / Agent 365 Mitigations:**
- **Isolated Environments:** Agents run in isolated Power Platform environments
- **Disable/Restrict Controls:** Agents can be disabled immediately without code deployment
- **Circuit Breakers:** Built-in retry limits and timeout controls
- **Agent 365 Observe Pillar:** Real-time monitoring of agent health and dependencies

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Spike in agent error rates (>2σ from baseline) | Trigger circuit breaker; isolate affected agents | AI Operations |
| Downstream system degradation correlated with agent activity | Disable upstream agent; rollback if needed | Platform Engineering |
| Cross-tenant impact from single-tenant agent failure | Activate tenant isolation; notify affected franchises | CoE Director |

**Blast Radius Mitigation Architecture:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE TENANT                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ Pricing     │    │ Inventory   │    │ Analytics   │         │
│  │ Agent       │    │ Agent       │    │ Agent       │         │
│  │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │         │
│  │ │Circuit  │ │    │ │Circuit  │ │    │ │Circuit  │ │         │
│  │ │Breaker  │ │    │ │Breaker  │ │    │ │Breaker  │ │         │
│  │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         └──────────────────┼──────────────────┘                 │
│                            │                                    │
│                    ┌───────▼───────┐                            │
│                    │ Tenant        │                            │
│                    │ Isolation     │                            │
│                    │ Boundary      │                            │
│                    └───────┬───────┘                            │
└────────────────────────────┼────────────────────────────────────┘
                             │
┌────────────────────────────┼────────────────────────────────────┐
│                    RETAIL TENANT                                │
│                    ┌───────▼───────┐                            │
│                    │ Cross-Tenant  │                            │
│                    │ Gateway       │                            │
│                    └───────┬───────┘                            │
│                            │                                    │
│  ┌─────────────┐    ┌──────▼──────┐    ┌─────────────┐         │
│  │ Store A     │    │ Store B     │    │ Store C     │         │
│  │ Agent       │    │ Agent       │    │ Agent       │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

---

#### 5.7.10 ASI09: Human–Agent Trust Exploitation

**OWASP Definition:** Abusing user trust and authority bias to get unsafe approvals or extract sensitive information.

**Multi-Tenant Retail Threat Scenario:**
A malicious actor manipulates an agent to present a fraudulent refund request with artificial urgency, exploiting store managers' trust in "the system" to approve large transactions. Alternatively, an agent is coaxed into presenting data access requests that seem routine but actually exfiltrate customer PII.

**Copilot Studio / Agent 365 Mitigations:**
- **Output Validation:** Agent responses reviewed for authority manipulation patterns
- **Approval Workflows:** High-impact actions require explicit human confirmation
- **Agent Transparency:** Users can see what data sources and tools agents are using
- **Agent 365 Governance:** Policies for what agents can request from users

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Agent requesting approval for action outside normal patterns | Add friction (second approver, delay) | Business Process Owner |
| Sudden increase in high-value transaction approvals via agents | Audit sample; investigate anomalies | Loss Prevention |
| User reports agent exhibiting pressure tactics | Disable agent; review prompt design | AI Engineering |

**Human-in-the-Loop Thresholds:**
| Action Type | Threshold | Approval Requirement |
|-------------|-----------|---------------------|
| Customer refund | >$500 | Manager approval + reason code |
| Inventory transfer | >1000 units | Regional manager approval |
| Price override | Any | Store manager + LP alert |
| PII export | Any | Data steward approval + audit log |

---

#### 5.7.11 ASI10: Rogue Agents

**OWASP Definition:** Agents drifting or being compromised in ways that cause harmful behavior beyond intended scope.

**Multi-Tenant Retail Threat Scenario:**
A Copilot Studio agent initially scoped to answer store policy questions gradually expands its tool usage and begins accessing customer records, inventory systems, and pricing—either through misconfiguration drift or deliberate tampering. In multi-tenant environments, a rogue agent in one franchise could attempt to propagate to enterprise systems.

**Copilot Studio / Agent 365 Mitigations:**
- **Agent Registry:** Complete inventory of all agents, their capabilities, and status
- **Immutable Configuration:** Agents cannot modify their own logic; changes require republishing
- **Behavioral Monitoring:** Agent 365 Observe pillar tracks deviations from expected behavior
- **Disable/Restrict Controls:** Immediate containment without code deployment

**Retail-Specific Detection & Response:**
| Detection Method | Response Action | Owner |
|-----------------|-----------------|-------|
| Agent accessing resources not in original manifest | Disable agent; investigate configuration drift | AI Engineering |
| Shadow agent discovered (not in registry) | Block immediately; escalate to security | CoE / Security |
| Agent behavior deviation from baseline (ML anomaly detection) | Flag for review; consider containment | AI Operations |

**Agent Drift Prevention Checklist:**
- [ ] All production agents registered in Agent 365 registry
- [ ] Weekly comparison of agent capabilities vs. approved manifest
- [ ] Automated alerts for any agent republishing
- [ ] Quarterly access reviews for all enterprise agents
- [ ] Cross-tenant agent trust relationships reviewed monthly

**Shadow Agent Hunt Query:**
```kusto
let RegisteredAgents = AgentRegistry | distinct AgentId;
CopilotAgentActivity
| where AgentId !in (RegisteredAgents)
| summarize FirstSeen=min(Timestamp), LastSeen=max(Timestamp), 
            ActivityCount=count() by AgentId, TenantId
| order by ActivityCount desc
```

---

#### 5.7.12 Implementation Roadmap

**Phase 1: Foundation (Months 1-3)**
- [ ] Enable Agent 365 Observe pillar across all tenants
- [ ] Register all existing agents in Agent 365 registry
- [ ] Implement Entra Agent ID for Tier 3+ agents
- [ ] Deploy baseline Conditional Access policies for agents
- [ ] Establish KQL hunt queries for top 5 risks (ASI01, ASI02, ASI03, ASI08, ASI10)

**Phase 2: Hardening (Months 4-6)**
- [ ] Enable Agent 365 Govern pillar with lifecycle policies
- [ ] Implement connector DLP policies aligned to OWASP risks
- [ ] Deploy human-in-the-loop thresholds for retail operations
- [ ] Establish A2A authentication for cross-tenant agent flows
- [ ] Train SOC on agent-specific incident response

**Phase 3: Maturity (Months 7-12)**
- [ ] Enable Agent 365 Secure pillar with runtime threat protection
- [ ] Integrate agent telemetry into Microsoft Sentinel
- [ ] Implement ML-based behavioral anomaly detection
- [ ] Conduct tabletop exercises for agent compromise scenarios
- [ ] Achieve OWASP compliance attestation

---

## 6. Security Architecture & Risks

### 6.1 Zero Trust Principles for Agents

| Principle | Application to Agents |
|-----------|----------------------|
| **Verify Explicitly** | Authenticate every agent request; validate user and app identity; verify permissions on each resource access |
| **Least Privilege** | Request minimum API scopes; use delegated permissions where possible; scope data access to job function |
| **Assume Breach** | Segment agent networks; encrypt data in transit and at rest; monitor for anomalies; plan for incident response |

### 6.2 Identity Security

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

### 6.3 Data Security

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

#### 5.3.3 Retail-Specific DLP Policy Templates

> **Reference:** [Microsoft Purview Data Loss Prevention](https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp)

The following DLP policies are designed specifically for multi-tenant retail environments where agents handle customer data, payment information, and cross-franchise operations.

---

##### Sensitivity Label Taxonomy for Multi-Tenant Retail

| Label | Sublabel | Description | Agent Behavior |
|-------|----------|-------------|----------------|
| **Public** | — | Marketing materials, public FAQs | Full access |
| **Internal** | General | Internal communications, SOPs | Auth required |
| **Internal** | Franchise-Restricted | Franchise-specific data | Tenant isolation |
| **Confidential** | Customer PII | Names, emails, addresses, phone | Masked in responses |
| **Confidential** | Employee Data | HR records, payroll, performance | HR agent only |
| **Confidential** | Business Sensitive | Pricing strategies, margins, forecasts | Restricted agents |
| **Highly Confidential** | Payment Data (PCI) | Card numbers, CVV, auth codes | Blocked from agents |
| **Highly Confidential** | Health Data (if applicable) | PHI for pharmacy/health retail | Blocked from agents |

**Purview Label Configuration:**
```
Sensitivity Labels → Create label
├── Name: "Confidential - Customer PII"
├── Description: "Customer personally identifiable information"
├── Scope: Files, Emails, Meetings, Sites, Groups
├── Content marking: "CONFIDENTIAL - CUSTOMER PII" watermark
├── Encryption: None (handled by DLP)
└── Auto-labeling: 
    ├── Sensitive info types: Person's name, Email, Phone, Address
    ├── Confidence: High
    └── Scope: SharePoint, OneDrive, Exchange
```

---

##### Policy 1: Customer PII Protection in Agent Responses

**Objective:** Prevent agents from exposing customer PII in responses while allowing lookup and verification.

**Purview DLP Policy Configuration:**

| Setting | Value |
|---------|-------|
| **Policy name** | `DLP-Agent-CustomerPII-Protection` |
| **Mode** | Enforce |
| **Locations** | Exchange, SharePoint, OneDrive, Teams, Devices |
| **Conditions** | Content contains: Customer PII (Name + Email + Phone combo) |
| **Actions** | Block sharing externally; Notify user; Log to audit |
| **Exceptions** | Customer service agents with PII-approved role |

**Copilot Studio Implementation:**

```
Agent Settings → Content Moderation → Custom Rules
├── Rule: "Mask Customer PII"
├── Pattern: 
│   ├── Email: [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
│   ├── Phone: \b\d{3}[-.]?\d{3}[-.]?\d{4}\b
│   ├── SSN: \b\d{3}-\d{2}-\d{4}\b
├── Action: Replace with [REDACTED]
└── Log: Yes
```

**Agent Response Behavior:**
| Query | Without Policy | With Policy |
|-------|---------------|-------------|
| "What's John Smith's email?" | john.smith@email.com | Customer email on file. For verification, please confirm last 4 digits of phone. |
| "Show me customer 12345's details" | Full profile displayed | Name: John S. Phone: ***-***-1234. Address: [CITY, STATE only] |

---

##### Policy 2: PCI-DSS Compliance for Payment Data

**Objective:** Ensure agents never store, process, or transmit cardholder data in violation of PCI-DSS.

**PCI-DSS Requirements Mapping:**

| PCI-DSS Req | Agent Implementation |
|-------------|---------------------|
| 3.2 - Don't store sensitive auth data | Block card data in agent memory/transcripts |
| 3.4 - Render PAN unreadable | Mask all but last 4 digits |
| 7.1 - Limit access to cardholder data | No agent access to payment systems |
| 10.2 - Audit trail | Log all payment-related queries |

**Purview DLP Policy Configuration:**

| Setting | Value |
|---------|-------|
| **Policy name** | `DLP-Agent-PCI-Block` |
| **Mode** | Enforce (Block) |
| **Conditions** | Credit Card Number (Luhn validated), CVV pattern, Expiration date pattern |
| **Actions** | Block content; Generate incident report; Alert Security team |
| **Exceptions** | None (no exceptions for PCI data) |

**Sensitive Information Type - Custom:**

```json
{
  "name": "Payment Card Full Number",
  "description": "Full credit/debit card number (PCI violation risk)",
  "pattern": {
    "regex": "\\b(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13}|6(?:011|5[0-9]{2})[0-9]{12})\\b",
    "validator": "LuhnCheck"
  },
  "confidence": "High"
}
```

**Agent Guardrails:**

```
Agent Instructions (System Prompt):
├── "NEVER request, display, or store full credit card numbers"
├── "NEVER request CVV, PIN, or full expiration dates"
├── "For payment issues, direct to secure payment portal or phone support"
└── "If user provides card info, respond: 'For security, please do not share card details here. Use our secure payment portal.'"
```

---

##### Policy 3: Cross-Tenant Data Boundary Enforcement

**Objective:** Prevent data from Enterprise Tenant being accessed/shared to Retail Tenant (and vice versa) except through approved integration points.

**Architecture:**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     ENTERPRISE TENANT                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ DLP Policy: Block external sharing except to Retail Tenant       │   │
│  │ Sensitivity Labels: Enterprise-Confidential                      │   │
│  │ Cross-tenant access: Approved apps only                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                               │                                         │
│                    Approved Integration Points Only                     │
│                    (Inventory API, WFM API, Pricing API)               │
│                               │                                         │
└───────────────────────────────┼─────────────────────────────────────────┘
                                │
                    ◄──────────────────────►
                      Cross-Tenant Trust
                    (B2B / App Registration)
                                │
┌───────────────────────────────┼─────────────────────────────────────────┐
│                     RETAIL TENANT                                        │
│                               │                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ DLP Policy: Block sharing to Enterprise except via approved APIs │   │
│  │ Sensitivity Labels: Retail-Confidential                          │   │
│  │ Agent access: Retail data only (unless cross-tenant agent)       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

**Purview DLP Policy - Enterprise Tenant:**

| Setting | Value |
|---------|-------|
| **Policy name** | `DLP-CrossTenant-Enterprise-Boundary` |
| **Mode** | Enforce |
| **Conditions** | Content labeled "Enterprise-Confidential" AND shared outside organization |
| **Actions** | Block; Notify compliance team |
| **Exceptions** | Approved cross-tenant apps (list by App ID) |

**Power Platform DLP - Connector Groups:**

```
Environment: Production-Enterprise
├── Business Data Group (Allowed)
│   ├── SharePoint
│   ├── Microsoft Graph
│   ├── Dataverse
│   └── Approved cross-tenant API connector
├── Non-Business Data Group (Limited)
│   ├── HTTP connector (internal endpoints only)
│   └── Azure Key Vault
└── Blocked Group
    ├── All social media connectors
    ├── All personal storage connectors
    └── Unapproved external APIs
```

---

##### Policy 4: Franchise-Specific Data Isolation

**Objective:** Ensure each franchise can only access their own data through agents; prevent cross-franchise data leakage.

**Data Isolation Model:**

| Data Type | Isolation Method | Agent Enforcement |
|-----------|------------------|-------------------|
| Store sales data | Row-level security (Dataverse) | User's store affiliation filter |
| Inventory levels | Store ID column filter | Agent includes store_id in all queries |
| Customer data | Geographic/store assignment | Agent restricts to assigned region |
| Employee data | Manager hierarchy | Agent checks reporting chain |

**Dataverse Row-Level Security:**

```
Security Role: Franchise-Store-User
├── Entity: Store_Sales
│   ├── Read: Organization-owned filtered by Store_ID = User.Store_ID
│   ├── Write: None
│   └── Create: None
├── Entity: Inventory
│   ├── Read: Organization-owned filtered by Store_ID = User.Store_ID
│   └── Write: Adjustment requests only
└── Entity: Customer
    └── Read: Filtered by AssignedStore = User.Store_ID OR Region = User.Region
```

**Agent Query Injection Pattern:**

```
Agent Topic: Inventory Query
├── Trigger: User asks about inventory
├── Action: Get User Profile
│   └── Store_ID = System.User.Store_ID
├── Action: Query Dataverse
│   └── Filter: Store_ID eq '{Store_ID}'
└── Response: "Your store ({Store_ID}) has {qty} units of {product}."

# Prevents: User asking about competitor franchise's inventory
```

**Purview DLP for Franchise Isolation:**

| Setting | Value |
|---------|-------|
| **Policy name** | `DLP-Franchise-DataIsolation` |
| **Mode** | Audit (then Enforce after validation) |
| **Conditions** | Content contains franchise identifier AND accessed by user from different franchise |
| **Actions** | Block; Alert franchise compliance officer |

---

##### Policy 5: Employee Data Protection (HR/Payroll)

**Objective:** Restrict agent access to employee data to authorized HR agents and ensure compliance with employment privacy regulations.

**Protected Data Categories:**

| Category | Examples | Protection Level |
|----------|----------|------------------|
| Compensation | Salary, bonuses, equity | Highly Confidential |
| Performance | Reviews, PIPs, ratings | Confidential |
| Personal | SSN, bank accounts, benefits | Highly Confidential |
| Health | Leave records, accommodations | Highly Confidential (PHI) |
| Disciplinary | Warnings, terminations | Confidential |

**Purview DLP Policy:**

| Setting | Value |
|---------|-------|
| **Policy name** | `DLP-Agent-EmployeeData-Protection` |
| **Mode** | Enforce |
| **Conditions** | Content labeled "Employee Data - HR" OR contains SSN/bank account patterns |
| **Actions** | Block unless accessed by HR-approved agent; Log all access |
| **Exceptions** | Self-service (employee viewing own data) via authenticated HR portal |

**Agent Access Matrix:**

| Agent | Employee Self-Data | Team Data | All Employee Data |
|-------|-------------------|-----------|-------------------|
| HR Policy FAQ | ❌ | ❌ | ❌ |
| Employee Self-Service | ✅ Own only | ❌ | ❌ |
| Manager Agent | ✅ Own | ✅ Direct reports | ❌ |
| HR Operations Agent | ✅ | ✅ | ✅ (with audit) |
| Payroll Agent | ✅ Own | ❌ | ✅ (with audit) |

**Conditional Access for HR Agents:**

```
Policy: HR-Agent-Access-Control
├── Applies to: HR Operations Agent, Payroll Agent
├── Conditions:
│   ├── User risk: Low only
│   ├── Device: Compliant, corporate-managed
│   ├── Location: Corporate network OR VPN
│   └── Session: Max 8 hours
├── Grant: Allow with MFA
└── Session controls: Sign-in frequency every 4 hours
```

---

##### DLP Policy Monitoring & Enforcement

**Purview Compliance Dashboard Alerts:**

| Alert | Trigger | Response |
|-------|---------|----------|
| `Agent-PII-Exposure-Attempt` | Agent query returned PII to non-authorized user | Investigate; review agent permissions |
| `Agent-PCI-Violation` | Payment card data detected in agent context | Immediate disable; incident response |
| `Cross-Tenant-Leak-Attempt` | Data labeled Enterprise-Confidential sent to Retail | Block; review cross-tenant config |
| `Franchise-Boundary-Violation` | User accessed another franchise's data | Block; audit user permissions |
| `HR-Data-Unauthorized-Access` | Non-HR agent accessed employee records | Block; escalate to HR |

**KQL Query for Agent DLP Violations:**

```kusto
DLPPolicyMatch
| where Application == "Copilot Studio" or Application == "Microsoft Copilot"
| where TimeGenerated > ago(7d)
| summarize 
    Violations = count(),
    UniqueUsers = dcount(UserId),
    UniqueAgents = dcount(ResourceId)
  by PolicyName, Severity
| order by Violations desc
```

---

### 6.4 Network Security

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

### 6.5 Secrets Management

| Secret Type | Storage | Rotation | Access |
|-------------|---------|----------|--------|
| API Keys | Azure Key Vault | Automated (90 days) | Managed Identity |
| Certificates | Azure Key Vault | Automated (annual) | Managed Identity |
| Connection Strings | Key Vault Reference | Per deployment | App Configuration |
| OAuth Tokens | Runtime memory only | Per request | Not persisted |

### 6.6 Security Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Cross-Prompt Injection (XPIA)** | Malicious input manipulates agent behavior | Content filtering, input validation, grounding separation |
| **Data Exfiltration** | Sensitive data leaked via agent responses | DLP policies, output filtering, audit logging |
| **Privilege Escalation** | Agent gains unintended access | Least privilege, regular permission reviews |
| **Cross-Tenant Data Leakage** | Data from one tenant exposed to another | Strict tenant isolation, permission scoping, ACL enforcement |
| **Prompt Injection** | Adversarial prompts override instructions | System prompt protection, input sanitization |
| **Supply Chain Attack** | Malicious API plugin or connector | Plugin vetting, signed packages, secure registry |

---

## 7. Operational Model

### 7.1 Agent Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Design  │───►│  Build   │───►│   Test   │───►│  Deploy  │───►│  Operate │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                                      │
┌──────────┐    ┌──────────┐                                          │
│  Retire  │◄───│  Update  │◄─────────────────────────────────────────┘
└──────────┘    └──────────┘
```

### 7.2 Environment Strategy (Zoned Governance)

| Zone | Purpose | Governance | Data Access | Approval |
|------|---------|------------|-------------|----------|
| **Safe Zone** | Experimentation, POC | Minimal | Sample/test data only | Self-service |
| **Supported Zone** | Departmental use | Standard | Departmental data | Team approval |
| **IT-Managed Zone** | Production | Full | Enterprise data | CAB approval |

#### Agent 365 Integration with Zoned Governance

With Agent 365 GA (May 1, 2026), Conditional Access for agents can **technically enforce** zone boundaries:

| Zone | Agent 365 Conditional Access Policy | Enforcement |
|------|-------------------------------------|-------------|
| **Safe Zone** | `Report-only` mode; no blocking | Visibility without disruption |
| **Supported Zone** | Require approved blueprint; block high-risk agents | Automated enforcement |
| **IT-Managed Zone** | Require approved blueprint + custom attribute `zone=production` + low agent risk | Full policy enforcement |

**Configuration Example:**
```
Conditional Access Policy: "IT-Managed Zone Enforcement"
├── Applies to: All agent identities
├── Target resources: Production apps (tagged with "env:production")
├── Conditions:
│   ├── Agent risk: High or Medium → Block
│   └── Custom attribute: zone ≠ production → Block
└── Grant: Allow if conditions pass
```

**Benefits over Custom Governance:**
- **Automatic enforcement** at token acquisition (vs. manual approval gates)
- **Risk-based blocking** using ID Protection signals
- **Audit trail** in Entra sign-in logs (vs. custom logging)
- **Consistent policy** across all agents (vs. per-agent custom rules)

### 7.3 DevSecOps Pipeline

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

### 7.4 Monitoring & Observability

| Layer | Tool | Metrics |
|-------|------|---------|
| **Agent Performance** | Copilot Studio Analytics | Response time, completion rate, user satisfaction |
| **Application** | Azure Application Insights | Errors, latency, dependencies |
| **Security** | Microsoft Defender for Cloud | Threats, vulnerabilities, compliance |
| **Audit** | Microsoft Purview | Data access, admin actions, policy violations |
| **Cross-Tenant** | Azure Sentinel (centralized) | Correlated security events across tenants |

### 7.5 Incident Response

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

## 8. Team Structure & RACI Matrix

### 8.1 Recommended Teams

#### 8.1.1 AI Engineering Team (Agent Builders)

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

#### 8.1.2 M365 Platform Engineering

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

#### 8.1.3 Identity & Access Management (IAM)

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

#### 8.1.4 Security / SecOps

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

#### 8.1.5 Retail IT / Store Digital

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

#### 8.1.6 Enterprise Architecture

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

#### 8.1.7 Data Platforms

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

#### 8.1.8 AI Operations (AIOps)

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

### 8.2 RACI Matrices

#### 8.2.1 Agent Creation

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Gather requirements | R | C | C | C | C | C | C | I |
| Design agent architecture | R | C | C | A | C | A | C | I |
| Build agent | R | I | I | I | I | I | C | I |
| Security review | C | I | C | A | I | C | I | I |
| Documentation | R | I | I | C | I | I | I | I |

#### 8.2.2 Agent Deployment

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Environment provisioning | C | R | C | C | C | I | I | I |
| CI/CD pipeline execution | R | C | I | C | I | I | I | C |
| Pre-prod testing | R | C | I | C | C | I | C | C |
| Production approval | C | C | C | A | C | C | I | C |
| Production deployment | R | C | I | I | I | I | I | A |
| Post-deployment validation | R | C | I | C | C | I | I | R |

#### 8.2.3 Identity Setup

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| App registration | C | C | R | C | I | I | I | I |
| API permissions | C | C | R | A | I | I | C | I |
| Cross-tenant access | I | C | R | A | C | I | I | I |
| Conditional Access | I | C | R | A | I | I | I | I |
| Service principal mgmt | C | C | R | C | I | I | I | I |

#### 8.2.4 Secrets Handling

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Key Vault setup | I | R | C | A | I | I | I | I |
| Secret creation | R | C | C | C | I | I | I | I |
| Access policy config | I | C | R | A | I | I | I | I |
| Rotation automation | C | R | C | A | I | I | I | C |
| Audit and monitoring | I | C | C | R | I | I | I | C |

#### 8.2.5 Security Validation

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Threat modeling | C | C | C | R | C | C | C | I |
| Security scanning | C | I | I | R | I | I | I | I |
| Penetration testing | C | C | C | R | C | I | C | I |
| Compliance review | C | C | C | R | C | C | C | I |
| Risk acceptance | I | I | I | A | I | C | I | I |

#### 8.2.6 Monitoring & Incident Handling

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Monitoring setup | C | C | I | C | I | I | I | R |
| Alert configuration | C | C | I | C | I | I | I | R |
| Incident detection | I | I | I | C | I | I | I | R |
| Incident triage | C | C | C | C | C | I | I | R |
| Root cause analysis | R | C | C | C | C | I | C | C |
| Incident resolution | R | C | C | C | C | I | C | A |

#### 8.2.7 Upgrades and Breaking Changes

| Activity | AI Eng | M365 Platform | IAM | Security | Retail IT | EA | Data | AIOps |
|----------|--------|---------------|-----|----------|-----------|----|----- |-------|
| Impact assessment | R | C | C | C | C | A | C | C |
| Communication plan | R | C | C | C | C | I | I | C |
| Regression testing | R | C | I | C | C | I | C | C |
| Rollback planning | R | C | I | C | I | I | I | C |
| Change approval | C | C | C | C | C | A | I | C |
| Deployment execution | R | C | I | I | I | I | I | A |

#### 8.2.8 Decommissioning Agents

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

## 9. Anti-Patterns and Common Risks

### 9.1 Governance Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **No inventory, no ownership** | Agents proliferate without accountability; incidents have no owner | Maintain agent registry; require owner for every agent |
| **Controls as "guidance only"** | Policies exist but aren't enforced technically | Implement technical controls (DLP, Conditional Access) |
| **Missing environment strategy** | Production and dev mixed; accidental exposure | Implement zoned governance (Safe, Supported, IT-Managed) |
| **One-size-fits-all governance** | Over-restricts simple agents or under-governs critical ones | Tier agents by risk; differentiated controls |
| **Shadow agents** | Teams bypass governance to ship faster | Make governance enabling, not blocking; provide templates |

### 9.2 Security Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Overprivileged app permissions** | Agent can access more than needed; blast radius | Minimum necessary permissions; regular reviews |
| **Shared secrets across tenants** | Compromise in one tenant affects others | Per-tenant secrets; workload identity federation |
| **No input validation** | Prompt injection attacks succeed | Input sanitization; content filtering; grounding separation |
| **Trust without verification** | Cross-tenant access assumed safe | Zero trust; explicit trust configuration; continuous monitoring |
| **Audit logging as afterthought** | Can't investigate incidents or prove compliance | Design logging from start; centralize across tenants |

### 9.3 Operational Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Manual deployments** | Inconsistent, error-prone, slow | CI/CD automation; infrastructure as code |
| **No testing strategy** | Agents fail in production; regressions | Unit, integration, adversarial, load testing |
| **Monitoring blindspots** | Issues discovered by users, not ops | Comprehensive observability; proactive alerting |
| **Cost governance ignored** | Runaway token/API costs | Usage monitoring; alerts on thresholds; chargebacks |
| **Knowledge silos** | One person knows how agent works | Documentation requirements; cross-training |

### 9.4 Multi-Tenant Anti-Patterns

| Anti-Pattern | Risk | Mitigation |
|--------------|------|------------|
| **Implicit trust between tenants** | Assume security based on ownership | Explicit cross-tenant access settings; defense in depth |
| **Shared infrastructure without isolation** | Noisy neighbor; data leakage | Tenant-specific resources; logical/physical isolation |
| **Inconsistent governance across tenants** | Weakest tenant becomes entry point | Federated governance with centralized standards |
| **Cross-tenant admin sprawl** | Too many people with cross-tenant access | PIM for cross-tenant roles; regular access reviews |

---

### 9.5 Root Cause Analysis & Remediation Playbooks

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

## 10. Glossary & Terminology

| Term | Definition |
|------|------------|
| **Agent** | An AI-powered application that can understand context, take actions, and interact with users or systems autonomously within defined boundaries |
| **Agent 365** | Microsoft's native control plane for AI agents (GA May 1, 2026) providing observe, govern, and secure capabilities at enterprise scale |
| **Agent Blueprint** | Logical definition of an agent type in Entra Agent ID; template for creating agent instances across tenants |
| **Agent Identity** | First-class identity construct in Entra Agent ID representing an instantiated agent; performs token acquisitions |
| **Agent Identity Blueprint Principal** | Service principal representing an agent blueprint in a tenant; creates agent identities and agent users |
| **Agent Registry** | Agent 365 inventory of all agents in an organization, including shadow agent discovery |
| **Agent User** | Non-human user identity in Entra Agent ID for agent experiences requiring user account context |
| **B2B Collaboration** | Microsoft Entra feature enabling users from one tenant to access resources in another as guest users |
| **CAB** | Change Advisory Board — governance body approving production changes |
| **Conditional Access** | Policy-based access control in Entra ID based on signals like user, device, location, risk |
| **Conditional Access for Agents** | Agent 365 capability extending Conditional Access evaluation and enforcement to agent identities |
| **CoE** | Center of Excellence — centralized team providing standards, guidance, and shared services |
| **Copilot Studio** | Microsoft's low-code platform for building AI agents |
| **Cross-Tenant Access Settings** | Entra ID configuration controlling B2B collaboration between specific tenants |
| **Cross-Tenant Synchronization** | Automated provisioning of B2B users between trusted tenants |
| **Declarative Agent** | Configuration-based agent running on M365 Copilot infrastructure |
| **DLP** | Data Loss Prevention — policies preventing sensitive data exfiltration |
| **Entra Agent ID** | Microsoft Entra capability providing first-class identity management for AI agents |
| **Entra ID** | Microsoft's cloud identity and access management service (formerly Azure AD) |
| **Enterprise Tenant** | The Entra ID tenant serving corporate/HQ functions |
| **Federated Agent Mesh** | Architecture pattern with centralized governance and distributed agent execution |
| **Grounding** | Process of augmenting agent responses with enterprise knowledge |
| **ID Protection for Agents** | Agent 365 capability detecting risky agent behavior and enabling risk-based Conditional Access |
| **M365 E7** | Microsoft 365 E7 bundle ($99/user/month) including E5, Copilot, Entra Suite, and Agent 365 |
| **MCP** | Model Context Protocol — standardized protocol for agent-to-data communication |
| **Managed Identity** | Azure feature providing automatic identity for Azure resources without credentials |
| **MTO** | Multi-Tenant Organization — Entra ID feature simplifying collaboration across owned tenants |
| **OBO** | On-Behalf-Of flow — OAuth pattern where service acts on behalf of user |
| **PIM** | Privileged Identity Management — just-in-time access for administrative roles |
| **RACI** | Responsible, Accountable, Consulted, Informed — decision rights matrix |
| **RAG** | Retrieval-Augmented Generation — grounding AI responses in retrieved documents |
| **Retail Tenant** | The Entra ID tenant serving store operations and frontline workers |
| **Service Principal** | Identity object in Entra ID representing an application |
| **Shadow Agent** | Undiscovered or unregistered agent operating in an organization; detected by Agent 365 |
| **Workload Identity Federation** | Technique enabling Azure resources to access external systems without secrets |
| **XPIA** | Cross-Prompt Injection Attack — security vulnerability in conversational AI |
| **Zero Trust** | Security model assuming no implicit trust; verify explicitly, least privilege, assume breach |
| **Zoned Governance** | Environment strategy with progressive controls (Safe, Supported, IT-Managed) |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 2.1 | 2026-04-02 | Enterprise Architecture | Added Section 3: Microsoft Agent 365 as native control plane; Entra Agent ID identity architecture; Conditional Access for agents; updated section 7.2 with Agent 365 zone enforcement; pricing ($15/user, E7 $99/user) |
| 2.0 | 2026-04-02 | Enterprise Architecture | Added failure modes, cost model, operational runbooks, root cause playbooks |
| 1.0 | 2026-03-24 | Enterprise Architecture | Initial release |

---

*End of Architecture Document*
