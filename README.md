# Healthcare Agent Product Evidence

This portfolio brings together a clinic workflow I put into use and a separate design proposal for evaluating agents on outpatient claims review.

## 1. Patient Recall Case Study

At Fairfield Acupuncture Center, staff tracked missed appointments and patients due for follow-up in Trello. I replaced that process with a prioritized call queue based on visit history and clinic rules. Staff could exclude patients, temporarily defer contact, log outcomes, and review metrics. Staff still decided when to reach out and made the calls themselves.

I observed that it became easier for staff to decide whom to call next. In the original clinic deployment, I reported roughly four months of daily use, thousands of patient records processed, and more than 20 outreach calls per user per day. These are reported usage figures, not audited measurements of time saved or improvements in patient outcomes.

For a future rollout, I would measure time to identify the next eligible patient, whether the rules blocked inappropriate outreach, and what happened after staff made contact.

---

## 2. Proposed Agent Evaluation: Outpatient Claims Readiness

This example is a design proposal for a reinforcement learning (RL) environment, not a deployed claims system.

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

## 3. Separate Agent Infrastructure Projects

- **Overseer** explored task dispatch and execution checkpoints.
- **Lattice** is a separate project that explored isolated workspaces and verification gates.
- **Painter** explored breaking generation into steps that can be checked against explicit criteria.

---

## 4. Product Thesis: 3 Healthcare Workflow Families to Model First

1. **Revenue-Cycle Denial Exception Handling**: High economic ROI, governed by ANSI 835/837 rules, clear denial codes.
2. **Prior-Authorization Readiness**: High clinical impact, matches unstructured EHR notes against payer coverage guidelines.
3. **Outpatient Recall Recovery**: A staff-led workflow where eligibility, contact preferences, and appropriate suppression matter.
