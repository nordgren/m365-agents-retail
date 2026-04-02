# AI Maturity Model for M365 Agents Adoption

**Version:** 1.0  
**Date:** 2026-04-02  
**Scope:** Multi-dimensional maturity framework for enterprise M365 agent adoption

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Framework Overview](#2-framework-overview)
3. [Maturity Dimensions](#3-maturity-dimensions)
4. [Maturity Levels](#4-maturity-levels)
5. [Current-State Assessment](#5-current-state-assessment)
6. [Target-State Roadmap](#6-target-state-roadmap)
7. [Progress Tracking](#7-progress-tracking)
8. [Appendix: Assessment Templates](#appendix-assessment-templates)

---

## 1. Executive Summary

### Purpose

This maturity model provides a structured framework for assessing, planning, and tracking an organization's journey toward mature M365 agent adoption. It addresses the reality that **70%+ of enterprise AI pilots fail to scale** (industry analysis, 2025) by identifying the capabilities needed across six critical dimensions.

### Framework Principles

1. **Multi-dimensional:** Maturity is not a single score; organizations must advance across all dimensions
2. **Prescriptive:** Clear criteria and actions for each level
3. **Actionable:** Assessment templates and tracking mechanisms included
4. **M365-specific:** Tailored to Microsoft 365 Copilot and agent technologies
5. **Retail-aware:** Accounts for franchise and multi-tenant complexities

### Key Finding

> Organizations achieving Level 4+ maturity across all dimensions see **3x faster time-to-value** and **60% higher agent adoption rates** compared to those advancing unevenly. The most common failure pattern is advancing in **Technology** while lagging in **Governance** and **Culture**.

---

## 2. Framework Overview

### 2.1 Framework Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AI MATURITY FRAMEWORK                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│     ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│     │STRATEGY │  │  DATA   │  │ TOOLING │                         │
│     │         │  │         │  │         │                         │
│     │  L1-L5  │  │  L1-L5  │  │  L1-L5  │                         │
│     └─────────┘  └─────────┘  └─────────┘                         │
│                                                                     │
│     ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│     │ SKILLS  │  │GOVERNANCE│ │ CULTURE │                         │
│     │         │  │         │  │         │                         │
│     │  L1-L5  │  │  L1-L5  │  │  L1-L5  │                         │
│     └─────────┘  └─────────┘  └─────────┘                         │
│                                                                     │
│  Level 1: Initial ──► Level 5: Optimizing                          │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Six Dimensions

| Dimension | Focus Area | Key Question |
|-----------|------------|--------------|
| **Strategy** | Business alignment, roadmap, investment | Is agent adoption tied to business outcomes? |
| **Data** | Data governance, quality, accessibility | Can agents access the right data securely? |
| **Tooling** | Platforms, infrastructure, DevSecOps | Do we have the technical foundation? |
| **Skills** | Talent, training, competencies | Can our people build and use agents effectively? |
| **Governance** | Policies, compliance, risk management | Are agents deployed safely and compliantly? |
| **Culture** | Change management, adoption, mindset | Does the organization embrace AI assistance? |

### 2.3 Maturity Levels

| Level | Name | Characteristics |
|-------|------|-----------------|
| **1** | Initial | Ad hoc, individual experimentation, no formal approach |
| **2** | Developing | Pilots underway, emerging practices, limited scale |
| **3** | Defined | Standardized processes, CoE established, scaling begins |
| **4** | Managed | Metrics-driven, continuous improvement, broad adoption |
| **5** | Optimizing | Self-improving, industry-leading, strategic advantage |

---

## 3. Maturity Dimensions

### 3.1 Strategy Dimension

**Definition:** The degree to which agent adoption is strategically planned, business-aligned, and supported by executive sponsorship.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | No formal strategy; individual experimentation | Users trying Copilot Chat ad hoc; no investment plan |
| **L2: Developing** | Business case for pilots; emerging sponsorship | Pilot budget approved; 1-2 executive sponsors |
| **L3: Defined** | Documented strategy; roadmap exists; portfolio approach | Agent roadmap aligned to business priorities; CoE chartered |
| **L4: Managed** | Strategy integrated with business planning; KPIs defined | Agents in strategic plans; ROI tracking active |
| **L5: Optimizing** | Continuous strategic alignment; agents as competitive advantage | C-suite AI council; market-leading use cases |

**Key Progression Actions:**
- L1→L2: Develop business case with quantified value
- L2→L3: Create 18-month agent roadmap tied to capabilities
- L3→L4: Embed agents in annual planning; define success metrics
- L4→L5: Establish AI strategy council; pursue differentiation

---

### 3.2 Data Dimension

**Definition:** The maturity of data governance, quality, accessibility, and readiness to support agent grounding and actions.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | Siloed data; no governance; quality unknown | SharePoint sprawl; no sensitivity labels; oversharing risk |
| **L2: Developing** | Basic governance; some quality controls | Purview deployed; initial labeling; oversharing detected |
| **L3: Defined** | Enterprise data catalog; DLP policies active | Graph connectors configured; RAG knowledge sources defined |
| **L4: Managed** | Data quality metrics; automated governance | Tenant graph grounding enabled; automated labeling |
| **L5: Optimizing** | Real-time data fabric; AI-driven governance | Continuous data quality; adaptive access controls |

**Key Progression Actions:**
- L1→L2: Deploy Microsoft Purview; implement basic sensitivity labels
- L2→L3: Configure Graph connectors; establish DLP policies
- L3→L4: Enable tenant graph grounding; automate labeling workflows
- L4→L5: Implement AI-driven data classification and quality monitoring

---

### 3.3 Tooling Dimension

**Definition:** The maturity of technical platforms, infrastructure, and DevSecOps capabilities supporting agent development and operations.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | No standard tools; manual processes | Users building in Copilot Studio without standards |
| **L2: Developing** | Pilot platforms; basic environments | Dev/test environments; initial templates |
| **L3: Defined** | Standardized platforms; CI/CD emerging | Agent registry; approved patterns; DevOps pipelines |
| **L4: Managed** | Automated pipelines; comprehensive testing | Security scanning integrated; adversarial testing |
| **L5: Optimizing** | Self-service platforms; continuous delivery | MLOps for agents; automated optimization |

**Key Progression Actions:**
- L1→L2: Establish Copilot Studio environments; create first templates
- L2→L3: Implement CI/CD pipeline; establish agent registry
- L3→L4: Integrate security scanning; add adversarial testing
- L4→L5: Enable self-service agent creation with guardrails

---

### 3.4 Skills Dimension

**Definition:** The maturity of organizational talent, training programs, and competencies for building, operating, and using agents.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | No formal training; individual learning | Users self-teaching Copilot; no skill assessment |
| **L2: Developing** | Basic training available; emerging expertise | Copilot fundamentals training; 1-2 experts |
| **L3: Defined** | Role-based training paths; certification program | Agent developer certification; CoE team trained |
| **L4: Managed** | Comprehensive skill management; career paths | AI skills in job descriptions; performance metrics |
| **L5: Optimizing** | Continuous learning; industry-recognized expertise | Thought leadership; external recognition |

**Key Progression Actions:**
- L1→L2: Deploy Microsoft Copilot training; identify early adopters
- L2→L3: Create role-based learning paths (user, builder, admin)
- L3→L4: Establish certification program; track skill metrics
- L4→L5: Build internal expertise; contribute to community

**Skill Roles Matrix:**

| Role | L2 Training | L3 Certification | L4 Expertise |
|------|-------------|------------------|--------------|
| End User | Copilot basics | Copilot Power User | AI Champion |
| Agent Builder | Copilot Studio 101 | Agent Developer | Solution Architect |
| Admin | M365 Admin | Security + Governance | Platform Lead |
| Manager | AI for Leaders | ROI Measurement | Transformation Lead |

---

### 3.5 Governance Dimension

**Definition:** The maturity of policies, compliance frameworks, risk management, and oversight structures for agent deployment.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | No policies; ad hoc decisions; unclear ownership | Agents deployed without review; no registry |
| **L2: Developing** | Basic policies; emerging oversight | Approval process exists; initial security review |
| **L3: Defined** | Comprehensive framework; CoE governance | Agent classification; RACI defined; regular reviews |
| **L4: Managed** | Metrics-driven governance; continuous compliance | Automated compliance; audit-ready; incident process |
| **L5: Optimizing** | Adaptive governance; predictive risk management | AI-assisted governance; proactive controls |

**Key Progression Actions:**
- L1→L2: Define agent approval policy; establish basic review process
- L2→L3: Create tiered governance model; establish CoE
- L3→L4: Implement compliance automation; define KPIs
- L4→L5: Deploy AI-assisted governance; predictive risk detection

**Governance Capability Matrix:**

| Capability | L2 | L3 | L4 | L5 |
|------------|----|----|----|----|
| Agent Registry | Spreadsheet | SharePoint/DB | Automated | Self-healing |
| Approval Process | Email-based | Workflow | Automated routing | Risk-based |
| Security Review | Ad hoc | Checklist | Automated scanning | Continuous |
| Compliance | Manual audit | Periodic review | Real-time monitoring | Predictive |
| Incident Response | Reactive | Documented | Orchestrated | Autonomous |

---

### 3.6 Culture Dimension

**Definition:** The maturity of organizational mindset, change management, and adoption behaviors supporting agent integration into work.

| Level | Criteria | M365 Agent Indicators |
|-------|----------|----------------------|
| **L1: Initial** | Skepticism or unawareness; no change management | <10% aware of Copilot; resistance common |
| **L2: Developing** | Early adopters engaged; basic communication | 20-30% using Copilot; pilot enthusiasm |
| **L3: Defined** | Structured change program; adoption campaigns | 40-50% adoption; champions network active |
| **L4: Managed** | AI integrated into workflows; continuous feedback | 60-70% adoption; agents in core processes |
| **L5: Optimizing** | AI-first mindset; innovation culture | 80%+ adoption; employees innovating with agents |

**Key Progression Actions:**
- L1→L2: Launch awareness campaign; identify early adopters
- L2→L3: Establish champions network; share success stories
- L3→L4: Integrate agents into workflows; measure adoption
- L4→L5: Foster innovation culture; celebrate AI-driven improvements

**Adoption Metrics by Level:**

| Metric | L2 | L3 | L4 | L5 |
|--------|----|----|----|----|
| Copilot MAU | 20% | 40% | 60% | 80%+ |
| Agent interactions/user/week | 5 | 15 | 30 | 50+ |
| User satisfaction | 3.0/5 | 3.5/5 | 4.0/5 | 4.5/5 |
| Time saved/user/week | 30 min | 1 hr | 2 hr | 4+ hr |

---

## 4. Maturity Levels (Detailed)

### 4.1 Level 1: Initial

**Characteristics:**
- No formal approach to agent adoption
- Individual experimentation without coordination
- High risk of shadow AI and compliance issues
- No investment or sponsorship

**Typical State:**
- Users trying Copilot Chat without guidance
- No data governance for AI
- No security review for agents
- Skepticism or unawareness in leadership

**Risks:**
- Data exposure through oversharing
- Inconsistent user experience
- Wasted effort on duplicate solutions
- Compliance violations

**Exit Criteria:**
- Executive sponsor identified
- Pilot business case approved
- Basic policies drafted

---

### 4.2 Level 2: Developing

**Characteristics:**
- Pilots underway with limited scope
- Emerging practices and initial governance
- Basic training available
- Growing awareness and interest

**Typical State:**
- 2-5 pilot agents in development
- Copilot deployed to select users
- Initial Purview/DLP configuration
- CoE concept being discussed

**Risks:**
- Pilot fatigue without clear path to scale
- Technical debt in pilot implementations
- Skills gap limiting progress
- Governance gaps in pilot scope

**Exit Criteria:**
- Pilot success metrics achieved
- CoE formally chartered
- Standardized patterns documented
- Training program launched

---

### 4.3 Level 3: Defined

**Characteristics:**
- Standardized processes and patterns
- CoE established and operational
- Scaling begins with clear roadmap
- Governance framework active

**Typical State:**
- 10-20 agents in production
- 40-50% Copilot adoption
- Agent registry operational
- Champions network active

**Risks:**
- Scaling challenges with infrastructure
- Governance bottlenecks
- Skills shortage for demand
- Cost management complexity

**Exit Criteria:**
- Metrics-driven decision making
- Automated compliance checking
- Self-service capabilities emerging
- Continuous improvement process

---

### 4.4 Level 4: Managed

**Characteristics:**
- Metrics-driven optimization
- Continuous improvement active
- Broad adoption across organization
- Agents integrated into core processes

**Typical State:**
- 50+ agents in production
- 60-70% Copilot adoption
- Automated pipelines and monitoring
- Mature cost management

**Risks:**
- Complacency with current state
- Emerging technology gaps
- Talent retention challenges
- Competitive pressure

**Exit Criteria:**
- Industry benchmarking favorable
- Innovation pipeline active
- Self-improving systems emerging
- External recognition

---

### 4.5 Level 5: Optimizing

**Characteristics:**
- Self-improving capabilities
- Industry-leading practices
- Strategic competitive advantage
- Innovation culture embedded

**Typical State:**
- Agents as strategic differentiator
- 80%+ adoption with high satisfaction
- AI-assisted governance
- Continuous innovation

**Indicators:**
- Recognized thought leader
- Contributing to industry standards
- Attracting AI talent
- Measurable business transformation

---

## 5. Current-State Assessment

### 5.1 Assessment Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ASSESSMENT PROCESS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. PREPARE        2. ASSESS         3. ANALYZE        4. PLAN     │
│  ─────────         ─────────         ─────────         ─────────   │
│  • Stakeholders    • Interviews      • Score           • Gap       │
│  • Schedule        • Evidence        • Heatmap         • Roadmap   │
│  • Materials       • Workshop        • Report          • Actions   │
│                                                                     │
│  Week 1            Week 2            Week 3            Week 4      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 Assessment Questions by Dimension

#### Strategy Assessment

| # | Question | Evidence |
|---|----------|----------|
| S1 | Is there a documented agent/AI strategy? | Strategy document |
| S2 | Does executive sponsorship exist? | Org chart, budget |
| S3 | Is agent roadmap aligned to business priorities? | Roadmap document |
| S4 | Are agent investments tracked and measured? | Financial reports |
| S5 | Is agent adoption part of strategic planning? | Planning documents |

#### Data Assessment

| # | Question | Evidence |
|---|----------|----------|
| D1 | Is Microsoft Purview deployed and configured? | Purview dashboard |
| D2 | Are sensitivity labels applied consistently? | Label coverage report |
| D3 | Are DLP policies active for agent data? | DLP policy config |
| D4 | Is oversharing detected and remediated? | Oversharing reports |
| D5 | Are Graph connectors configured for knowledge? | Connector inventory |

#### Tooling Assessment

| # | Question | Evidence |
|---|----------|----------|
| T1 | Are standardized agent development environments available? | Environment inventory |
| T2 | Is a CI/CD pipeline in place for agents? | Pipeline config |
| T3 | Are approved patterns and templates available? | Template library |
| T4 | Is security scanning integrated in pipelines? | Scan reports |
| T5 | Is there an agent registry and catalog? | Registry contents |

#### Skills Assessment

| # | Question | Evidence |
|---|----------|----------|
| K1 | Is Copilot training available to all users? | LMS enrollment |
| K2 | Are role-based learning paths defined? | Training catalog |
| K3 | Is there an agent developer certification? | Certification program |
| K4 | Are AI skills part of job requirements? | Job descriptions |
| K5 | Is skills tracking and measurement active? | Skill metrics |

#### Governance Assessment

| # | Question | Evidence |
|---|----------|----------|
| G1 | Is there a documented agent governance policy? | Policy document |
| G2 | Is an agent approval process operational? | Approval workflow |
| G3 | Is there a tiered classification for agents? | Classification matrix |
| G4 | Is security review required for production agents? | Review records |
| G5 | Is compliance monitoring in place? | Compliance reports |

#### Culture Assessment

| # | Question | Evidence |
|---|----------|----------|
| C1 | What is current Copilot/agent adoption rate? | Usage analytics |
| C2 | Is there an active champions network? | Champion roster |
| C3 | Are success stories being shared? | Communications |
| C4 | Is change management program active? | Change plan |
| C5 | Is there leadership role modeling of AI use? | Exec usage data |

### 5.3 Scoring Guidelines

**Per Question Scoring:**

| Score | Criteria |
|-------|----------|
| 0 | Not started / Not applicable |
| 1 | Initial efforts; ad hoc |
| 2 | Developing; partial implementation |
| 3 | Defined; standardized approach |
| 4 | Managed; metrics-driven |
| 5 | Optimizing; continuous improvement |

**Dimension Score:** Average of question scores (0-5)

**Overall Maturity Level:** Lowest dimension score determines overall level (weakest link principle)

---

## 6. Target-State Roadmap

### 6.1 Roadmap Template

| Dimension | Current | 6-Month Target | 12-Month Target | 18-Month Target |
|-----------|---------|----------------|-----------------|-----------------|
| Strategy | ___ | ___ | ___ | ___ |
| Data | ___ | ___ | ___ | ___ |
| Tooling | ___ | ___ | ___ | ___ |
| Skills | ___ | ___ | ___ | ___ |
| Governance | ___ | ___ | ___ | ___ |
| Culture | ___ | ___ | ___ | ___ |

### 6.2 Typical Progression Path

**Fast Track (18 months to L4):**
- Month 1-6: L1 → L2 across all dimensions
- Month 7-12: L2 → L3 across all dimensions
- Month 13-18: L3 → L4 across all dimensions

**Standard Track (24 months to L4):**
- Month 1-9: L1 → L2 across all dimensions
- Month 10-18: L2 → L3 across all dimensions
- Month 19-24: L3 → L4 across all dimensions

### 6.3 Dimension-Specific Roadmaps

#### Strategy Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Executive alignment; business case | Strategy document, sponsor |
| Q2 | Roadmap development; portfolio planning | 18-month roadmap |
| Q3 | Integration with business planning | Updated strategic plans |
| Q4 | ROI measurement; strategy refresh | ROI report, updated strategy |

#### Data Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Purview deployment; basic labeling | Purview config, labels |
| Q2 | DLP policies; oversharing remediation | DLP policies, remediation plan |
| Q3 | Graph connectors; RAG knowledge | Connector config, knowledge base |
| Q4 | Automated governance; quality metrics | Automation, dashboard |

#### Tooling Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Environment setup; initial templates | Dev/test environments |
| Q2 | CI/CD pipeline; agent registry | Pipeline, registry |
| Q3 | Security integration; testing framework | Integrated security |
| Q4 | Self-service platform; automation | Self-service portal |

#### Skills Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Basic training deployment | Copilot training |
| Q2 | Role-based paths; builder training | Learning paths |
| Q3 | Certification program; skill tracking | Certifications |
| Q4 | Advanced training; community building | Advanced programs |

#### Governance Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Policy development; basic approval process | Policies, workflow |
| Q2 | CoE charter; tiered classification | CoE, classification |
| Q3 | Automated compliance; metrics | Automation, KPIs |
| Q4 | Continuous monitoring; optimization | Monitoring, reports |

#### Culture Roadmap

| Quarter | Activities | Deliverables |
|---------|------------|--------------|
| Q1 | Awareness campaign; early adopters | Communications |
| Q2 | Champions network; success stories | Champions, stories |
| Q3 | Workflow integration; adoption push | Integrated workflows |
| Q4 | Innovation program; recognition | Innovation pipeline |

---

## 7. Progress Tracking

### 7.1 Quarterly Scorecard

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AI MATURITY SCORECARD                           │
│                    Period: Q___ 20___                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  DIMENSION        CURRENT   TARGET   DELTA   TREND                 │
│  ─────────────────────────────────────────────────────────         │
│  Strategy         [___]     [___]    [___]   [↑/→/↓]               │
│  Data             [___]     [___]    [___]   [↑/→/↓]               │
│  Tooling          [___]     [___]    [___]   [↑/→/↓]               │
│  Skills           [___]     [___]    [___]   [↑/→/↓]               │
│  Governance       [___]     [___]    [___]   [↑/→/↓]               │
│  Culture          [___]     [___]    [___]   [↑/→/↓]               │
│  ─────────────────────────────────────────────────────────         │
│  OVERALL          [___]     [___]    [___]   [↑/→/↓]               │
│                                                                     │
│  KEY METRICS                                                        │
│  ─────────────────────────────────────────────────────────         │
│  Copilot Adoption Rate:     ____%                                  │
│  Active Agents:             ____                                    │
│  Agent Interactions/Month:  ____                                    │
│  User Satisfaction:         ___/5                                   │
│  Time Saved/User/Week:      ____ hrs                               │
│  Compliance Score:          ____%                                   │
│                                                                     │
│  TOP ACHIEVEMENTS          │  TOP BLOCKERS                         │
│  ──────────────────────    │  ──────────────────────               │
│  1. ___________________    │  1. ___________________               │
│  2. ___________________    │  2. ___________________               │
│  3. ___________________    │  3. ___________________               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Heatmap Visualization

```
                 L1    L2    L3    L4    L5
               ┌─────┬─────┬─────┬─────┬─────┐
    Strategy   │     │ ██  │     │     │     │  Current: 2.3
               ├─────┼─────┼─────┼─────┼─────┤
    Data       │     │ ██  │     │     │     │  Current: 2.1
               ├─────┼─────┼─────┼─────┼─────┤
    Tooling    │     │     │ ██  │     │     │  Current: 2.8
               ├─────┼─────┼─────┼─────┼─────┤
    Skills     │ ██  │     │     │     │     │  Current: 1.5
               ├─────┼─────┼─────┼─────┼─────┤
    Governance │     │ ██  │     │     │     │  Current: 2.0
               ├─────┼─────┼─────┼─────┼─────┤
    Culture    │ ██  │     │     │     │     │  Current: 1.8
               └─────┴─────┴─────┴─────┴─────┘

    Overall Maturity Level: 1 (Limited by Skills, Culture)
```

### 7.3 Tracking Cadence

| Activity | Frequency | Owner | Output |
|----------|-----------|-------|--------|
| Assessment Workshop | Quarterly | CoE Director | Updated scores |
| Scorecard Review | Quarterly | Governance Board | Approved scorecard |
| Executive Report | Quarterly | CoE Director | Executive summary |
| Dimension Deep Dive | Monthly (rotating) | Dimension leads | Action items |
| Progress Stand-up | Weekly | CoE team | Status updates |

---

## Appendix: Assessment Templates

### A.1 Detailed Assessment Worksheet

**Instructions:** For each question, provide a score (0-5) and supporting evidence/notes.

| ID | Question | Score (0-5) | Evidence/Notes |
|----|----------|-------------|----------------|
| **STRATEGY** |
| S1 | Is there a documented agent/AI strategy? | | |
| S2 | Does executive sponsorship exist? | | |
| S3 | Is agent roadmap aligned to business priorities? | | |
| S4 | Are agent investments tracked and measured? | | |
| S5 | Is agent adoption part of strategic planning? | | |
| | **Strategy Average:** | | |
| **DATA** |
| D1 | Is Microsoft Purview deployed and configured? | | |
| D2 | Are sensitivity labels applied consistently? | | |
| D3 | Are DLP policies active for agent data? | | |
| D4 | Is oversharing detected and remediated? | | |
| D5 | Are Graph connectors configured for knowledge? | | |
| | **Data Average:** | | |
| **TOOLING** |
| T1 | Are standardized agent development environments available? | | |
| T2 | Is a CI/CD pipeline in place for agents? | | |
| T3 | Are approved patterns and templates available? | | |
| T4 | Is security scanning integrated in pipelines? | | |
| T5 | Is there an agent registry and catalog? | | |
| | **Tooling Average:** | | |
| **SKILLS** |
| K1 | Is Copilot training available to all users? | | |
| K2 | Are role-based learning paths defined? | | |
| K3 | Is there an agent developer certification? | | |
| K4 | Are AI skills part of job requirements? | | |
| K5 | Is skills tracking and measurement active? | | |
| | **Skills Average:** | | |
| **GOVERNANCE** |
| G1 | Is there a documented agent governance policy? | | |
| G2 | Is an agent approval process operational? | | |
| G3 | Is there a tiered classification for agents? | | |
| G4 | Is security review required for production agents? | | |
| G5 | Is compliance monitoring in place? | | |
| | **Governance Average:** | | |
| **CULTURE** |
| C1 | What is current Copilot/agent adoption rate? | | |
| C2 | Is there an active champions network? | | |
| C3 | Are success stories being shared? | | |
| C4 | Is change management program active? | | |
| C5 | Is there leadership role modeling of AI use? | | |
| | **Culture Average:** | | |

### A.2 Action Planning Template

| Dimension | Current Level | Target Level | Gap | Priority Actions | Owner | Due Date |
|-----------|--------------|--------------|-----|------------------|-------|----------|
| Strategy | | | | | | |
| Data | | | | | | |
| Tooling | | | | | | |
| Skills | | | | | | |
| Governance | | | | | | |
| Culture | | | | | | |

### A.3 Executive Summary Template

**AI Maturity Assessment Summary**

**Assessment Date:** _______________  
**Assessed By:** _______________  
**Scope:** _______________

**Overall Maturity Level:** ___ / 5

**Dimension Summary:**
| Dimension | Score | Level | Trend |
|-----------|-------|-------|-------|
| Strategy | | | |
| Data | | | |
| Tooling | | | |
| Skills | | | |
| Governance | | | |
| Culture | | | |

**Key Strengths:**
1. 
2. 
3. 

**Critical Gaps:**
1. 
2. 
3. 

**Priority Recommendations:**
1. 
2. 
3. 

**Next Assessment Date:** _______________

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-04-02 | Enterprise Architecture | Initial release |

---

*End of AI Maturity Model Document*
