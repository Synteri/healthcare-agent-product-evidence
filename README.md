# 🏥 Healthcare Agent Product Evidence — Jacob Schumacher

[![Target Role](https://img.shields.io/badge/Target_Role-Sr_AI_Product_Manager,_Healthcare_Agents_(Scale_AI)-blue.svg)](https://scale.com/careers/4709281005)
[![Domain](https://img.shields.io/badge/Domain-Outpatient_Ops_%7C_EHR/Claims_%7C_Agent_Environments-green.svg)](#)
[![Status](https://img.shields.io/badge/Status-Applied_%7C_Evidence_Bundle_Ready-success.svg)](#)

> **Core Focus**: Translating outpatient clinical operations, EHR/claims workflows, and PHI privacy boundaries into verifiable AI agent training & evaluation environments.

---

## 📌 Fast Skim for Recruiter / Hiring Manager

| Domain Area | Direct Operational & Technical Proof |
| :--- | :--- |
| **Outpatient Healthcare Operations** | **4 years** leading operations at a high-volume clinic (~130 visits/wk, ~100 insurance claims/wk including HUSKY/CT Medicaid, EHR administration, CPT/ICD-10 coding). |
| **Deployed Healthcare AI Product** | Designed & deployed **Patient Recall** (`recall-generator`): 4 months daily use, 1,000s of patient records, 20+ outreach calls/user/day, 7-tier PHI suppression logic. |
| **Scale AI Task Environment Design** | Authored concrete spec for **Outpatient Claims Readiness RL Environment** ($S, A, O, R$ spaces, rubrics, and Medicaid edge cases). |
| **Agent Harness Engineering** | 4-iteration progression (`Overseer` → `Lattice` → `Painter` → Harness Engineering) focusing on scoped context budgets, isolated worktrees, and verifier separation. |

---

## 1. Patient Recall Case Study

### The Problem
Outpatient clinic staff spend hours manually sorting unprioritized EHR exports to manage patient follow-ups and overdue care. This leads to inconsistent outreach, patient over-contacting, missed care adherence, and lost clinic revenue.

### The Product (`recall-generator`)
Built a multi-tenant SaaS prioritization engine with a 5-pillar architecture:
- 🛡️ **PHI Shield (`data_layer.py`)**: Row-level tenant data isolation & HIPAA compliance.
- ⚙️ **Smart Suppression Engine (`smart_suppression.py`)**: 7-tier suppression logic (opt-outs, recent contact windows, clinic rules).
- 💻 **Clinic Outreach CLI**: Fast administrative surface (`recall prioritize`, `recall send-outreach --dry-run`, `recall validate-phi`, `recall export-metrics`).

```
+-------------------+      +------------------------+      +-----------------------+
|  EHR Data Export  | ---> |  PHI Shield & 7-Tier   | ---> |  Prioritized Call     |
| (Raw Patient CSV) |      |   Suppression Engine   |      |  Queue & Telemetry    |
+-------------------+      +------------------------+      +-----------------------+
```

### Empirical Production Impact
* **Daily Deployment**: Used daily for ~4 months across 120–130 patient visits/week.
* **Volume**: Processed thousands of patient records; supported 20+ outreach calls per user per day.
* **Workflow Efficiency**: Reduced staff decision time on whom to call next from ~15 min/day to **effectively 0**.

> 💡 **Product Maturity Note**: *"While operational feedback was strongly positive, a future enterprise deployment must instrument contact rate, booking conversion, suppression accuracy, time-to-action, and incremental retained revenue to demonstrate validated causality."*

---

## 2. Example Scale AI Task Environment: Outpatient Claims Readiness

Demonstrating how domain judgment translates into a formal Reinforcement Learning (RL) agent environment:

### Environment Definition ($S, A, O, R$)

```typescript
// Candidate Agent Environment Contract for Outpatient Claims Readiness
interface ClaimsReadinessEnvironment {
  state: {
    patientChart: PatientDemographics & CoverageInfo;
    payerPolicy: PayerRules & AuthorizationRequirements;
    visitRecord: ClinicalNote & CPT_ICD10_Codes;
    historicalClaims: ClaimDenialLog[];
  };
  
  actionSpace: [
    'verify_eligibility',
    'check_prior_auth_requirement',
    'flag_missing_documentation',
    'apply_cpt_unbundling_rule',
    'approve_claim_readiness',
    'route_to_human_biller'
  ];
}
```

### Scoring Rubric ($R$)
| Agent Action | Reward / Penalty | Rationale |
| :--- | :---: | :--- |
| **Compliant Claim Approval** | `+100` | Claim passes coding, authorization, and payer rules cleanly. |
| **Appropriate Human Escalation** | `+50` | Correctly identifies ambiguous payer policy and routes to human biller. |
| **False Positive Submission** | `-50` | Submits claim lacking mandatory prior authorization. |
| **PHI / HIPAA Privacy Violation** | **`-500`** | **Catastrophic Failure**: Exposes patient data or routes to wrong clearinghouse. |

---

## 3. Agent Environment Lineage

```
Overseer (v1-v4)  ──►  Lattice Core  ──►  Painter Scaffold  ──►  Harness Engineering
(Task Dispatch)       (Work Isolation)   (Procedural Eval)     (Verifier Separation)
```

1. **Overseer (`Overseer-v4`)**: Learned that unbounded LLM loops stall; execution requires deterministic state machines and progress checkpoints.
2. **Lattice (`Lattice-Core`)**: Proved agents must operate in isolated worktrees with verification gates before merging code or data.
3. **Painter (`Painter`)**: Replaced free-form LLM generation with composable generator steps evaluated against explicit fitness criteria.
4. **Harness Engineering**: Demonstrated that scoping agent context tightly and separating the *worker agent* from the *verifier agent* drastically increases execution reliability.

---

## 4. Product Thesis: 3 Healthcare Workflow Families to Model First

1. **Revenue-Cycle Denial Exception Handling**: High economic ROI, governed by ANSI 835/837 rules, clear denial codes.
2. **Prior-Authorization Readiness**: High clinical impact, matches unstructured EHR notes against payer coverage guidelines.
3. **Outpatient Recall Recovery**: Direct patient impact, requires strict 7-tier PHI suppression boundaries.
