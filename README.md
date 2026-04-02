# m6-scope-1
Superstructure GTM for the Post Fiat network - M6 Scope
# Post Fiat Superstructure Validator Platform — M6 Scope and Handoff Plan

**Version:** 1.0
**Network:** Post Fiat Mainnet
**Depends On:** M0–M5 (shipped)
**Owner:** Superstructure Operations
**Classification:** Public
**Last Updated:** 2026-04-01

---

## Table of Contents

1. [M6 Objective and Success Metrics](#1-m6-objective-and-success-metrics)
2. [M5 Inheritance](#2-m5-inheritance)
3. [Module Inventory and Dependency Map](#3-module-inventory-and-dependency-map)
4. [Authorization and Adjudication Architecture](#4-authorization-and-adjudication-architecture)
5. [Phased Milestone Plan with Acceptance Criteria](#5-phased-milestone-plan-with-acceptance-criteria)
6. [Contributor Handoff Matrix](#6-contributor-handoff-matrix)
7. [Threshold Reference](#7-threshold-reference)
8. [Named Event Registry](#8-named-event-registry)
9. [Error Code Registry](#9-error-code-registry)
10. [Assumption Registry](#10-assumption-registry)
11. [Completion Verification Table](#11-completion-verification-table)
12. [Change Log](#12-change-log)

---

## 1. M6 Objective and Success Metrics

### Objective

M6 moves the Post Fiat Superstructure validator platform from a coordinated pilot fleet into a production-grade, multi-operator network with cryptographically enforced privacy for inference verification. The defining deliverable is a ZK Privacy Layer for Verified Inference: a protocol extension that allows validators to prove inference quality without revealing proprietary model weights, inference inputs, or operator identity to peer validators.

M6 is complete when: (a) a ZK proof construction and verification pipeline is integrated into the Orchestration Control Plane, (b) the UNL Selection Engine applies verified ZK attestations as a primary input alongside Composite Routing Score, (c) the authorization and adjudication layer classifies operators by reliability tier using accumulated ZK proof history, and (d) at least three independent contributor packages have been implemented and accepted against the acceptance criteria in this document.

### Success Metrics

| Metric | Target | Measurement Method |
|---|---|---|
| ZK proof generation time per inference batch | Below 800ms p95 | Telemetry field `zkProofGenerationMs` sampled per batch |
| ZK verification latency at Middleware | Below 150ms p95 | OCP internal log `zkVerificationMs` |
| Proof rejection rate (malformed or expired) | Below 0.5% per 24-hour window | `ERR_ZK_PROOF_INVALID` count / total proofs received |
| Operator reliability tier assignment lag | Below 60 seconds from triggering event | `RELIABILITY_TIER_ASSIGNED` event timestamp delta |
| Single-operator allocation ceiling enforced | No operator exceeds 60% routing weight | CRS weight audit log |
| Signing ceremony latency under ZK load | Below 5 minutes end-to-end | `CEREMONY_COMPLETED` timestamp delta |
| Contributor work package acceptance rate | 100% pass acceptance gate before merge | Adjudicator sign-off log |

M6 is not complete if any of the following persist: ZK proof pipeline integrated but not connected to UNL Selection Engine; reliability tier assignments not persisted to audit log; contributor packages partially implemented without adjudicator sign-off.

---

## 2. M5 Inheritance

### What M5 Shipped

M5 delivered six production-ready modules. Each has inherited interfaces M6 must preserve and open gaps M6 must resolve.

**Validator Fleet Readiness Audit Framework** establishes audit cadence, scoring bands, and escalation triggers. Canonical events: `FLEET_AUDIT_INITIATED` and `FLEET_AUDIT_PASSED`. Open gap: audit scores are computed but not classified into reliability tiers that feed routing decisions.

**Telemetry and Health Scoring Framework** defines the telemetry payload schema and the four-input Composite Routing Score: Verified Inference score (40%), uptime rate 24h (30%), p95 latency (20%), telemetry freshness (10%). Open gap: `verifiedInferenceScore` in the telemetry payload is a placeholder pending ZK integration. CRS does not yet accept ZK-attested scores as a distinct input class.

**Cross-Operator Coordination Protocol** routes all inter-operator coordination through Middleware. No peer-to-peer operator channels. Open gap: no protocol exists for ZK proof broadcast, deduplication, or sequencing when multiple operators submit proofs for overlapping inference batches.

**Governance Proposal Lifecycle Module** covers vote initiation, cohort notification, institutional review, and signing ceremony flow. Maker/Checker/Reviewer separation is non-waivable for all root-gated governance actions. No open gaps inherited into M6.

**Protocol Upgrade Orchestration Module** defines staged rollout, per-operator opt-in window, and rollback triggers. Open gap: upgrade compatibility matrix does not include ZK circuit version dependencies. A protocol upgrade that changes proof construction must handle proofs generated under the prior circuit version.

**UNL Selection Engine** selects the active validator set from enrolled operators using CRS as its sole dynamic input. Open gap: ZK attestation status is not a gating input. An operator without a valid recent ZK proof should not advance to ACTIVE routing regardless of CRS.

### Inherited Interface Constraints

Every M6 module must preserve these without modification:

- **Signature scheme:** Ed25519 over canonical JSON (RFC 8785). ZK proof envelopes must be signed before submission using the same scheme.
- **Replay protection:** 120-second freshness window, 30-day nonce retention, `ERR_NONCE_REPLAY` on duplicate nonce.
- **Role model:** Maker / Checker / Reviewer. Cannot collapse to two roles for high-value actions. ZK circuit parameter updates are high-value actions.
- **Custody boundary:** Keys never leave Layer 1. ZK proof generation occurs inside operator infrastructure. Middleware receives the proof, not the inference data.
- **Routing ceiling:** No single operator exceeds 60% allocation weight.
- **Telemetry staleness threshold:** 90 seconds.
- **Failover SLA:** 30 seconds from trigger to weight update broadcast.

### Open Gap Registry

| Gap ID | Source Module | Description | M6 Resolution |
|---|---|---|---|
| GAP-M5-01 | Telemetry Framework | `verifiedInferenceScore` is a placeholder | Define ZK-attested score schema and CRS integration |
| GAP-M5-02 | Fleet Readiness Audit | Audit scores not classified into reliability tiers | Define tier classification logic and persistence |
| GAP-M5-03 | Cross-Operator Coordination | No ZK proof broadcast or deduplication protocol | Define proof sequencing and overlap resolution |
| GAP-M5-04 | UNL Selection Engine | ZK attestation not a gating input | Add ZK status gate before ACTIVE routing assignment |
| GAP-M5-05 | Protocol Upgrade Module | Circuit version dependencies absent from compatibility matrix | Add circuit version field to upgrade compatibility matrix |

---

## 3. Module Inventory and Dependency Map

### M6 Modules

| Module ID | Module Name | Closes Gaps | Parallelizable |
|---|---|---|---|
| M6-MOD-01 | ZK Proof Construction Specification | GAP-M5-01, GAP-M5-03 | Root — must complete first |
| M6-MOD-02 | ZK Verification Integration for OCP | GAP-M5-01 | After M6-MOD-01 |
| M6-MOD-03 | Reliability Classification Engine | GAP-M5-02 | After M6-MOD-02 |
| M6-MOD-04 | UNL Selection Engine — ZK Gate Extension | GAP-M5-04 | After M6-MOD-02 and M6-MOD-03 |
| M6-MOD-05 | Circuit Version Compatibility Layer | GAP-M5-05 | Parallel with M6-MOD-02 after M6-MOD-01 |

### Dependency Chain

```
M6-MOD-01  (ZK Proof Construction Specification)
    |
    +---> M6-MOD-02  (ZK Verification Integration for OCP)
    |         |
    |         +---> M6-MOD-03  (Reliability Classification Engine)
    |                   |
    |                   +---> M6-MOD-04  (UNL Selection Engine — ZK Gate)
    |
    +---> M6-MOD-05  (Circuit Version Compatibility Layer)
              [parallel with M6-MOD-02 and M6-MOD-03]
```

M6-MOD-01 is the root. No other module can begin until it is accepted. M6-MOD-04 is the integration gate and cannot be accepted until M6-MOD-02 and M6-MOD-03 are both complete. M6-MOD-05 can run in parallel with M6-MOD-02 and M6-MOD-03 once M6-MOD-01 is signed off.

### Design Decision — ZK Proof Generation Location

**Rejected:** Generating ZK proofs inside Middleware (Layer 3). This would require Middleware to receive inference inputs or model parameters, violating the custody boundary for operators who treat inference configurations as proprietary. It also concentrates proof computation in a single layer, making the 800ms p95 target unreachable under multi-operator load.

**Chosen:** Proofs generated inside operator infrastructure (Layer 2). Middleware receives the signed proof envelope and verifies the proof without accessing underlying inputs.

**Cost:** Middleware cannot independently audit whether inference inputs were well-formed. It can only verify the mathematical proof. Acceptable because the ZK circuit design enforces structural constraints on what a valid proof can represent.

---

## 4. Authorization and Adjudication Architecture

### Contributor Approval Boundaries

M6 introduces a formal contributor authorization model because multiple operators and external contributors are implementing parallel work packages. Overlapping implementations risk conflicting interface definitions if approval boundaries are not explicit.

| Tier | Scope | Approval Required From | Cannot Do Without Escalation |
|---|---|---|---|
| Tier 1 — Implementation | Code and schema work within a bounded work package | Work package lead sign-off | Modify interface contracts defined in M6-MOD-01 |
| Tier 2 — Interface Change | Payload schema changes, event name changes, error code additions | Superstructure operations lead plus one additional Tier 2 reviewer | Deploy to any shared integration environment |
| Tier 3 — Root Action | ZK circuit parameter updates, trust anchor rotation, reliability tier threshold changes | Full Maker/Checker/Reviewer signing ceremony | Any Tier 3 action without ceremony is an unauthorized change |

No contributor operates above their authorization tier without explicit escalation and approval through the adjudication log.

### Adjudication Flow

Every contributor work package submission goes through four steps:

**Step 1 — Submission.** Contributor submits implementation artifact and a completed self-checklist against the work package acceptance criteria. Triggers `WORK_PACKAGE_SUBMITTED`.

**Step 2 — Interface Review.** Work package lead verifies the implementation respects schemas, event names, and error codes defined in M6-MOD-01 without modification. Approval triggers `INTERFACE_REVIEW_PASSED`. Rejection triggers `ERR_INTERFACE_CONTRACT_VIOLATION` and returns the package with a specific written finding.

**Step 3 — Integration Test.** Implementation is tested against the shared integration environment. All named events must fire. All error codes must be reachable. Passing triggers `INTEGRATION_TEST_PASSED`.

**Step 4 — Adjudicator Sign-Off.** Superstructure operations lead reviews interface review result and integration test output and signs off. Sign-off triggers `WORK_PACKAGE_ACCEPTED`. Rejection requires a written finding logged to the audit trail before the package is returned.

### Audit Trail Requirements

Every adjudication event must be logged with: contributor identifier, work package ID, event name, timestamp (ISO 8601 UTC), reviewer identifier, and finding text if the event is a rejection. Audit log entries are append-only. No entry may be modified after creation.

### Reliability Classification

The Reliability Classification Engine (M6-MOD-03) assigns each operator to one of four reliability tiers based on accumulated ZK proof history and fleet audit scores.

| Tier | Criteria | Routing Weight Ceiling |
|---|---|---|
| TIER_ALPHA | ZK proof acceptance rate above 99% over trailing 7 days; fleet audit score above 90; no `ERR_ZK_PROOF_INVALID` in trailing 48 hours | 60% |
| TIER_BETA | ZK proof acceptance rate 95–99% over trailing 7 days; fleet audit score 75–90 | 40% |
| TIER_WATCH | ZK proof acceptance rate 85–95% over trailing 7 days, or fleet audit score 60–75, or one `ERR_ZK_PROOF_INVALID` in trailing 48 hours | 20%; `OPERATOR_WATCH_INITIATED` fires |
| TIER_SUSPENDED | ZK proof acceptance rate below 85% over trailing 7 days, or fleet audit score below 60 | 0%; `OPERATOR_ROUTING_SUSPENDED` fires |

Tier assignment recalculates every 60 seconds. A tier change in either direction triggers `RELIABILITY_TIER_ASSIGNED` with the previous tier, new tier, and triggering metric. An operator moving from TIER_ALPHA to TIER_WATCH triggers weight redistribution within policy bounds and operator notification, not automatic failover. An operator moving to TIER_SUSPENDED triggers `OPERATOR_ROUTING_SUSPENDED` and autonomous weight redistribution to remaining eligible operators within 30 seconds.

Reliability tier feeds live operations through the UNL Selection Engine (M6-MOD-04), which applies tier as a hard filter before CRS weighting. A TIER_SUSPENDED operator is excluded from the candidate set regardless of CRS.

---

## 5. Phased Milestone Plan with Acceptance Criteria

### Phase 1 — ZK Foundation (M6-MOD-01 and M6-MOD-05)

**Duration target:** 3 weeks from M6 kickoff.

**Deliverables:** M6-MOD-01 (ZK Proof Construction Specification) and M6-MOD-05 (Circuit Version Compatibility Layer).

**Acceptance Criteria:**

- [ ] M6-MOD-01 proof envelope schema defines every field with name, type, and description. No placeholder fields.
- [ ] Circuit input constraints specify what structural properties a valid proof attests to, without referencing operator-specific model parameters.
- [ ] Deduplication protocol specifies exactly how Middleware resolves overlapping proofs from multiple operators for the same inference batch. Includes named event `ZK_PROOF_DEDUPLICATION_RESOLVED`.
- [ ] M6-MOD-05 compatibility matrix includes a `circuitVersion` field in the upgrade payload.
- [ ] Migration path for pre-upgrade proofs specifies a grace period in seconds, the error code returned after the grace period (`ERR_ZK_CIRCUIT_VERSION_MISMATCH`), and the rollback path if the circuit upgrade is rejected by any operator.
- [ ] Both modules reviewed and signed off by Tier 2 reviewers before Phase 2 begins.

**Rollback:**
- Success: `PHASE_1_ACCEPTED` logged. Phase 2 begins.
- Failure: `ERR_PHASE_1_ACCEPTANCE_FAILED` logged with specific finding. M6-MOD-01 returned for revision. Phase 2 does not begin until Phase 1 is re-accepted.

---

### Phase 2 — OCP Integration and Reliability Classification (M6-MOD-02 and M6-MOD-03)

**Duration target:** 4 weeks. M6-MOD-02 and M6-MOD-03 run in parallel.

**Deliverables:** M6-MOD-02 (ZK Verification Integration for OCP) and M6-MOD-03 (Reliability Classification Engine).

**Acceptance Criteria:**

- [ ] OCP receives ZK proof envelope, verifies signature (Ed25519 / RFC 8785), and verifies the mathematical proof before accepting `verifiedInferenceScore` as a CRS input.
- [ ] A proof failing signature verification returns `ERR_ZK_PROOF_SIGNATURE_INVALID`. A proof failing mathematical verification returns `ERR_ZK_PROOF_INVALID`. Both errors logged to audit trail with operator ID and proof nonce.
- [ ] A proof older than 120 seconds returns `ERR_ZK_PROOF_EXPIRED`. A duplicate nonce returns `ERR_NONCE_REPLAY`. Existing replay protection applies unchanged.
- [ ] CRS recalculates within 60 seconds of a new accepted ZK proof. `CRS_UPDATED` fires with the new score and ZK attestation timestamp.
- [ ] Reliability tier recalculates every 60 seconds. `RELIABILITY_TIER_ASSIGNED` fires on any tier change with previous tier, new tier, triggering metric, and operator ID.
- [ ] TIER_SUSPENDED triggers `OPERATOR_ROUTING_SUSPENDED` and weight redistribution completes within 30 seconds. `ROUTING_WEIGHT_UPDATED` fires within 30 seconds. If redistribution fails, `ERR_WEIGHT_REDISTRIBUTION_FAILED` fires and previous weights are retained pending manual review.
- [ ] Integration tests demonstrate all named events firing under normal operation and all error codes reachable under specified failure conditions.

**Rollback:**
- Success: `PHASE_2_ACCEPTED` logged. Phase 3 begins.
- Failure on M6-MOD-02: `ERR_OCP_INTEGRATION_FAILED` logged. M6-MOD-02 returned for revision without blocking M6-MOD-03 acceptance.
- Failure on M6-MOD-03: `ERR_RELIABILITY_ENGINE_FAILED` logged. M6-MOD-03 returned for revision. Phase 3 requires both modules accepted.

---

### Phase 3 — UNL Gate and Full Integration (M6-MOD-04)

**Duration target:** 2 weeks.

**Deliverables:** M6-MOD-04 (UNL Selection Engine — ZK Gate Extension).

**Acceptance Criteria:**

- [ ] UNL Selection Engine rejects any operator without a ZK proof accepted within the trailing 300 seconds. Rejection fires `UNL_CANDIDATE_ZK_REJECTED` with operator ID and last proof timestamp.
- [ ] A TIER_SUSPENDED operator is excluded from the candidate set before CRS is evaluated. Exclusion fires `UNL_CANDIDATE_TIER_EXCLUDED`.
- [ ] Weight ceiling per tier enforced: TIER_ALPHA up to 60%, TIER_BETA up to 40%, TIER_WATCH up to 20%.
- [ ] Single-operator 60% ceiling enforced regardless of tier.
- [ ] Minimum two active operators enforced at all times. If the candidate set falls below two, `ERR_INSUFFICIENT_ACTIVE_OPERATORS` fires and the last known valid weight set is retained pending operator recovery or manual intervention.
- [ ] End-to-end integration test demonstrates proof generation at Layer 2, verification at OCP, tier classification, and UNL selection completing within latency targets: ZK verification below 150ms p95, tier assignment within 60 seconds, UNL update within 30 seconds of a tier change that affects routing.

**Rollback:**
- Success: `PHASE_3_ACCEPTED` and `M6_COMPLETE` logged.
- Failure: `ERR_UNL_INTEGRATION_FAILED` logged with specific finding. M6-MOD-04 returned for revision. `M6_COMPLETE` does not fire until Phase 3 is accepted.

---

## 6. Contributor Handoff Matrix

Three work packages are available for parallel implementation by independent contributors after M6-MOD-01 is accepted. Each has a defined scope boundary, explicit interface constraints, an acceptance gate, and a named adjudicator.

---

### Work Package A — ZK Verification Engine

**Package ID:** M6-WP-A
**Implements:** M6-MOD-02
**Prerequisite:** M6-MOD-01 accepted and interface contracts published
**Parallelizable With:** M6-WP-B, M6-WP-C (specification phase)

**Scope:** Implement ZK verification logic inside the Orchestration Control Plane. The OCP must receive signed ZK proof envelopes from operators, verify the Ed25519 signature over canonical JSON (RFC 8785), verify the mathematical proof against the circuit definition published in M6-MOD-01, and update the `verifiedInferenceScore` CRS input.

**Out of Scope:** Reliability tier assignment (M6-WP-B); UNL selection logic (M6-WP-C); changes to the proof envelope schema (frozen in M6-MOD-01).

**Interface Constraints:**
- Proof envelope schema: defined in M6-MOD-01. No field additions or removals without Tier 2 approval.
- Signature scheme: Ed25519 / RFC 8785. No substitution.
- Replay protection: 120-second freshness window, 30-day nonce retention, `ERR_NONCE_REPLAY` on duplicate nonce. Existing implementation must not be modified.
- CRS recalculation must fire within 60 seconds of a new accepted proof.

**Named Events This Package Must Emit:** `ZK_PROOF_RECEIVED`, `ZK_PROOF_VERIFIED`, `CRS_UPDATED`

**Error Codes This Package Must Return:** `ERR_ZK_PROOF_SIGNATURE_INVALID`, `ERR_ZK_PROOF_INVALID`, `ERR_ZK_PROOF_EXPIRED`, `ERR_NONCE_REPLAY`

**Acceptance Gate:** All four error codes reachable in integration test. All three named events fire under normal operation. CRS recalculates within 60 seconds of accepted proof. Adjudicator sign-off: Superstructure operations lead.

**Self-Checklist:**
- [ ] Proof envelope schema matches M6-MOD-01 exactly — no field additions or removals
- [ ] Signature verification uses Ed25519 / RFC 8785
- [ ] Nonce log retains entries for 30 days
- [ ] `ERR_ZK_PROOF_EXPIRED` fires on proofs older than 120 seconds
- [ ] All named events fire with required fields: operatorId, proofNonce, timestamp
- [ ] Integration test output attached to submission

---

### Work Package B — Reliability Classification Engine

**Package ID:** M6-WP-B
**Implements:** M6-MOD-03
**Prerequisite:** M6-MOD-01 accepted; M6-WP-A interface contracts available
**Parallelizable With:** M6-WP-A, M6-WP-C (specification phase)

**Scope:** Implement the Reliability Classification Engine that ingests ZK proof history and fleet audit scores, assigns each operator to a reliability tier, and persists tier assignments to the audit log. Tier classification runs every 60 seconds. Tier changes trigger named events and routing weight adjustments.

**Out of Scope:** UNL candidate set selection (M6-WP-C); ZK proof verification logic (M6-WP-A); fleet audit scoring (inherited from M5, consumed as input here).

**Interface Constraints:**
- Tier definitions and thresholds: defined in Section 4. These are Tier 2 changes — do not modify without approval.
- Tier recalculation cadence: exactly 60 seconds. Not configurable per operator.
- Routing weight ceilings per tier: TIER_ALPHA 60%, TIER_BETA 40%, TIER_WATCH 20%, TIER_SUSPENDED 0%.
- TIER_SUSPENDED triggers weight redistribution with a 30-second SLA.

**Named Events This Package Must Emit:** `RELIABILITY_TIER_ASSIGNED`, `OPERATOR_WATCH_INITIATED`, `OPERATOR_ROUTING_SUSPENDED`, `ROUTING_WEIGHT_UPDATED`

**Error Codes This Package Must Return:** `ERR_WEIGHT_REDISTRIBUTION_FAILED`, `ERR_INSUFFICIENT_ACTIVE_OPERATORS`

**Acceptance Gate:** Tier recalculation fires at exactly 60-second intervals. All named events fire with required fields. TIER_SUSPENDED triggers redistribution within 30 seconds. `ERR_INSUFFICIENT_ACTIVE_OPERATORS` fires when candidate set falls below 2. Audit log entries are append-only. Adjudicator sign-off: Superstructure operations lead.

**Self-Checklist:**
- [ ] Tier thresholds match Section 4 exactly — not parameterized locally
- [ ] Recalculation fires every 60 seconds regardless of operator count
- [ ] TIER_SUSPENDED redistribution completes within 30 seconds or `ERR_WEIGHT_REDISTRIBUTION_FAILED` fires
- [ ] Audit log entries are append-only — no update or delete operations
- [ ] `RELIABILITY_TIER_ASSIGNED` includes previousTier, newTier, and triggeringMetric
- [ ] Integration test output attached to submission

---

### Work Package C — UNL ZK Gate Extension

**Package ID:** M6-WP-C
**Implements:** M6-MOD-04
**Prerequisite:** M6-WP-A accepted, M6-WP-B accepted
**Parallelizable With:** M6-WP-A and M6-WP-B (specification work only; integration requires both accepted)

**Scope:** Extend the UNL Selection Engine with two hard filters before CRS-based weighting: (1) ZK attestation gate — operators without a ZK proof accepted within the trailing 300 seconds are excluded from the candidate set; (2) reliability tier gate — TIER_SUSPENDED operators are excluded from the candidate set. Apply tier-based weight ceilings to the remaining candidates. Enforce the 60% single-operator ceiling and the minimum two active operators floor.

**Out of Scope:** ZK proof verification (M6-WP-A); tier assignment logic (M6-WP-B); CRS computation (inherited from M5).

**Interface Constraints:**
- ZK attestation validity window: 300 seconds. Not configurable per operator.
- Reliability tier input: consumed from M6-WP-B output. Not recomputed here.
- Single-operator ceiling: 60%. Hard limit with no override.
- Minimum active operators: 2. No override under any operational scenario.
- UNL update after a qualifying tier change must complete within 30 seconds.

**Named Events This Package Must Emit:** `UNL_CANDIDATE_ZK_REJECTED`, `UNL_CANDIDATE_TIER_EXCLUDED`, `UNL_SELECTION_COMPLETE`

**Error Codes This Package Must Return:** `ERR_INSUFFICIENT_ACTIVE_OPERATORS`, `ERR_CONCENTRATION_CAP_BREACH`

**Acceptance Gate:** End-to-end integration test demonstrates proof generated at Layer 2, verified by M6-WP-A, tier assigned by M6-WP-B, UNL selection completing within 30 seconds. All named events fire. Weight ceilings enforced. `ERR_INSUFFICIENT_ACTIVE_OPERATORS` fires when candidate set falls below 2. Adjudicator sign-off: Superstructure operations lead plus one additional Tier 2 reviewer (integration gate requires dual sign-off).

**Self-Checklist:**
- [ ] ZK attestation window is exactly 300 seconds — not configurable
- [ ] TIER_SUSPENDED exclusion runs before CRS weighting — not after
- [ ] 60% ceiling enforced — `ERR_CONCENTRATION_CAP_BREACH` logged and corrected, not propagated to routing
- [ ] Minimum 2 active operators enforced — `ERR_INSUFFICIENT_ACTIVE_OPERATORS` fires and last valid weights retained
- [ ] End-to-end integration test output demonstrates all three modules operating together
- [ ] Integration test output attached to submission

---

## 7. Threshold Reference

| Threshold | Value | Sustain Window | Source |
|---|---|---|---|
| ZK proof generation p95 | 800ms | 5-minute trailing window | M6-MOD-01 |
| ZK verification p95 at OCP | 150ms | 5-minute trailing window | M6-MOD-02 |
| ZK proof validity (replay protection) | 120 seconds | Single proof age | M6-WP-A (inherited from M3) |
| ZK UNL attestation window | 300 seconds | No proof within 300s = excluded | M6-WP-C |
| Nonce retention | 30 days | Rolling window | Inherited from M3 |
| Reliability tier recalculation cadence | 60 seconds | Fixed cadence | M6-MOD-03 |
| ZK proof history window for tier | 7 days | Trailing window | M6-MOD-03 |
| TIER_WATCH proof acceptance rate trigger | Below 95% | 7-day trailing window, not point-in-time | M6-MOD-03 |
| TIER_SUSPENDED proof acceptance rate trigger | Below 85% | 7-day trailing window, not point-in-time | M6-MOD-03 |
| TIER_SUSPENDED fleet audit score trigger | Below 60 | Single audit score | M6-MOD-03 |
| Reliability tier assignment lag | 60 seconds | From triggering event to `RELIABILITY_TIER_ASSIGNED` | M6-MOD-03 |
| TIER_SUSPENDED redistribution SLA | 30 seconds | From `OPERATOR_ROUTING_SUSPENDED` to `ROUTING_WEIGHT_UPDATED` | M6-MOD-03 |
| UNL update after tier change | 30 seconds | From `RELIABILITY_TIER_ASSIGNED` to `UNL_SELECTION_COMPLETE` | M6-MOD-04 |
| CRS recalculation after accepted proof | 60 seconds | From `ZK_PROOF_VERIFIED` to `CRS_UPDATED` | M6-MOD-02 |
| Single-operator routing ceiling | 60% | Hard ceiling; no override | Inherited from M3 |
| Minimum active operators | 2 | Hard floor; no override | Inherited from M3 |
| Telemetry staleness threshold | 90 seconds | From last telemetry receipt | Inherited from M5 |
| Signing ceremony latency under ZK load | 5 minutes | End-to-end | M6 overall |
| p95 latency failover trigger | 400ms sustained for 60 seconds | 60-second sustain window | Inherited from M5 |

---

## 8. Named Event Registry

| Event Name | Fired By | Trigger |
|---|---|---|
| `ZK_PROOF_RECEIVED` | OCP (M6-WP-A) | Proof envelope arrives before verification |
| `ZK_PROOF_VERIFIED` | OCP (M6-WP-A) | Signature and mathematical verification pass |
| `ZK_PROOF_DEDUPLICATION_RESOLVED` | OCP (M6-MOD-01) | Overlapping proof from multiple operators resolved |
| `CRS_UPDATED` | OCP (M6-WP-A) | CRS recalculates after accepted ZK proof |
| `RELIABILITY_TIER_ASSIGNED` | Classification Engine (M6-WP-B) | Any tier change; fields: operatorId, previousTier, newTier, triggeringMetric, assignedAt |
| `OPERATOR_WATCH_INITIATED` | Classification Engine (M6-WP-B) | Transition to TIER_WATCH |
| `OPERATOR_ROUTING_SUSPENDED` | Classification Engine (M6-WP-B) | Transition to TIER_SUSPENDED |
| `ROUTING_WEIGHT_UPDATED` | Classification Engine (M6-WP-B) | Weight redistribution after tier change |
| `UNL_CANDIDATE_ZK_REJECTED` | UNL Engine (M6-WP-C) | Operator excluded due to ZK attestation gap |
| `UNL_CANDIDATE_TIER_EXCLUDED` | UNL Engine (M6-WP-C) | Operator excluded due to TIER_SUSPENDED |
| `UNL_SELECTION_COMPLETE` | UNL Engine (M6-WP-C) | UNL update cycle completes |
| `WORK_PACKAGE_SUBMITTED` | Adjudication layer | Contributor submits package |
| `INTERFACE_REVIEW_PASSED` | Adjudication layer | Work package lead approves interface compliance |
| `INTEGRATION_TEST_PASSED` | Adjudication layer | Integration test completes without failures |
| `WORK_PACKAGE_ACCEPTED` | Adjudication layer | Adjudicator sign-off complete |
| `PHASE_1_ACCEPTED` | Superstructure operations | Phase 1 acceptance gate cleared |
| `PHASE_2_ACCEPTED` | Superstructure operations | Phase 2 acceptance gate cleared |
| `PHASE_3_ACCEPTED` | Superstructure operations | Phase 3 acceptance gate cleared |
| `M6_COMPLETE` | Superstructure operations | All phases accepted |
| `FLEET_AUDIT_INITIATED` | Audit Framework (M5 inherited) | Audit cycle begins |
| `FLEET_AUDIT_PASSED` | Audit Framework (M5 inherited) | Operator passes audit |
| `CEREMONY_COMPLETED` | Middleware (M3 inherited) | Signing ceremony completes |

---

## 9. Error Code Registry

| Error Code | Returned By | Trigger | System Response |
|---|---|---|---|
| `ERR_ZK_PROOF_SIGNATURE_INVALID` | OCP (M6-WP-A) | Ed25519 signature verification fails | Reject proof; log to audit trail with operatorId and nonce |
| `ERR_ZK_PROOF_INVALID` | OCP (M6-WP-A) | Mathematical proof verification fails | Reject proof; log; increment rejection counter for tier calculation |
| `ERR_ZK_PROOF_EXPIRED` | OCP (M6-WP-A) | Proof older than 120 seconds | Reject proof; log; do not increment mathematical rejection counter |
| `ERR_ZK_CIRCUIT_VERSION_MISMATCH` | OCP (M6-MOD-05) | Proof generated under prior circuit version after grace period | Reject proof; notify operator with grace period expiry timestamp |
| `ERR_NONCE_REPLAY` | OCP (M6-WP-A) | Duplicate nonce within 30-day retention window | Reject proof; log as potential replay attack; escalate to operations |
| `ERR_WEIGHT_REDISTRIBUTION_FAILED` | Classification Engine (M6-WP-B) | Redistribution cannot complete within 30 seconds | Retain previous weights; escalate to Superstructure operations; notify all operators |
| `ERR_INSUFFICIENT_ACTIVE_OPERATORS` | Classification Engine (M6-WP-B) or UNL Engine (M6-WP-C) | Active operator count falls below 2 | Retain last valid weights; escalate immediately; block further automated redistribution |
| `ERR_CONCENTRATION_CAP_BREACH` | UNL Engine (M6-WP-C) | Weight computation would exceed 60% for any operator | Log breach; apply correction to cap at 60%; redistribute remainder; do not propagate breach weight to routing |
| `ERR_INTERFACE_CONTRACT_VIOLATION` | Adjudication layer | Work package modifies frozen interface contracts | Return package to contributor with specific finding; do not merge |
| `ERR_OCP_INTEGRATION_FAILED` | Adjudication layer | M6-WP-A acceptance gate not cleared | Log finding; return package; Phase 2 not accepted until resolved |
| `ERR_RELIABILITY_ENGINE_FAILED` | Adjudication layer | M6-WP-B acceptance gate not cleared | Log finding; return package; Phase 3 requires both modules accepted |
| `ERR_UNL_INTEGRATION_FAILED` | Adjudication layer | M6-WP-C acceptance gate not cleared | Log finding; return package; `M6_COMPLETE` does not fire |
| `ERR_PHASE_1_ACCEPTANCE_FAILED` | Superstructure operations | Phase 1 acceptance gate not cleared | Log finding; Phase 2 does not begin |
| `ERR_LATENCY_THRESHOLD_BREACH` | OCP (M5 inherited) | p95 latency above 400ms sustained for 60 seconds | Failover trigger per M5 routing logic |
| `ERR_TELEMETRY_GAP` | OCP (M5 inherited) | Telemetry gap exceeds 90 seconds | Failover trigger per M5 routing logic |

---

## 10. Assumption Registry

### Confirmed (Inherited from M0–M5, Enforced in M6)

| ID | Assumption |
|---|---|
| CONF-01 | Signature scheme is Ed25519 over canonical JSON (RFC 8785). No substitution. |
| CONF-02 | Keys never leave Layer 1 under any operational scenario. |
| CONF-03 | Maker/Checker/Reviewer separation is non-waivable for high-value actions. |
| CONF-04 | Maximum single-operator routing weight is 60%. Hard ceiling. |
| CONF-05 | Minimum active operators is 2. Hard floor. No override. |
| CONF-06 | Routing is a middleware-layer function. Not on-chain. Not root-gated. |
| CONF-07 | Cloud HSM is rejected. Approved custody: institution-owned HSM (FIPS 140-2 L3), air-gapped system, or institutional multisig with institution as policy owner. |

### Open (Require Confirmation from Post Fiat Core Before Implementation)

| ID | Assumption | Impact If Wrong |
|---|---|---|
| OPEN-01 | ZK circuit definition and proof format will be specified by Post Fiat core before M6-MOD-01 begins. M6-MOD-01 is designing around a confirmed circuit, not specifying the circuit itself. | If unconfirmed, M6-MOD-01 must treat circuit parameters as placeholders and flag them as UNKNOWN. This is blocking for Phase 1. |
| OPEN-02 | The `verifiedInferenceScore` field in the M5 telemetry payload maps directly to a ZK-attested score field. No schema migration required beyond populating the field from verified proof output. | If the field schema changes, M6-WP-A must include a migration step and notify all operators. |
| OPEN-03 | ZK proof deduplication for overlapping batch proofs can be resolved by Middleware using proof submission timestamp as a tiebreaker. | If Post Fiat core requires deterministic consensus on deduplication, the protocol must be redesigned. Flag as blocking for M6-MOD-01 if unresolved at Phase 1 kickoff. |
| OPEN-04 | Circuit version field can be added to the upgrade compatibility matrix as a simple string field without a signing ceremony for the field addition itself. | If circuit version is a root-gated parameter, adding it requires a signing ceremony, which changes the Phase 1 timeline. |

---

## 11. Completion Verification Table

| Task Step | Section | Status |
|---|---|---|
| 1. Review M5 architecture and extract inherited interfaces, assumptions, and gaps | Section 2 — M5 Inheritance | Complete |
| 2. Define M6 objective, named modules, dependency order, and measurable success criteria | Sections 1 and 3 | Complete |
| 3. Specify authorization and adjudication layer including contributor approval boundaries, audit trail expectations, and reliability classification feeding live operations | Section 4 | Complete |
| 4. Add phased milestone plan with acceptance criteria and handoff matrix for at least 3 parallelizable contributor work packages | Sections 5 and 6 | Complete — three work packages: M6-WP-A, M6-WP-B, M6-WP-C |
| 5. Publish full M6 scope as one public document reviewable without login | This document | Complete — upload to GitHub Pages as index.md or index.html |

---

## 12. Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.0 | 2026-04-01 | Superstructure Operations | Initial M6 scope document. All five sections complete. Three parallelizable contributor work packages defined. |
