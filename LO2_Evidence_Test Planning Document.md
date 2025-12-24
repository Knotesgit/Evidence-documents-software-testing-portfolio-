# LO2 Evidence — Test Planning Document

## 1. Purpose of This Test Plan

The purpose of this test planning document is to demonstrate how LO2
(*Analyse requirements to determine appropriate testing strategies*)
is satisfied for the ILP Drone Delivery Microservice.

This document focuses specifically on five selected requirements from the LO1 Evidence file:

- **FR-S2 — Paths must not enter restricted areas**
- **FR-S3 — Drone selection must respect capacity & capability**
- **FR-I1 — ILP backend must be queried once per planning request**
- **FR-U2 — nextPosition must move exactly one STEP**
- **MR-1 — All requests must complete within 30 seconds**

These requirements were selected because they span **system, integration, unit, and measurable**
levels, and because each introduces distinct testability challenges that directly influence
test planning, instrumentation, and scaffolding decisions.

This document is a **planning artefact**, not a report of executed tests.
Its role is to justify *what will be tested, how it will be tested,
and what additional mechanisms are required to make such testing feasible*.

---

## 2. Selected Requirements Summary

| ID     | Level       | Type        | Priority | Reason |
|--------|-------------|-------------|----------|--------|
| FR-S2  | System      | Functional  | High     | Core safety requirement; constrained geometry central to pathfinding |
| FR-S3  | System      | Functional  | High     | Determines correctness of drone allocation |
| FR-I1  | Integration | Functional  | Medium   | Ensures correctness of backend interactions |
| FR-U2  | Unit        | Functional  | Medium   | Core geometric primitive used throughout pathfinding |
| MR-1   | System      | Performance | High     | Directly affects feasibility under ILP marking constraints |

---

## 3. Priority and Pre-requisite Analysis

### 3.1 Priority Rationale

- **FR-S2** and **FR-S3** are high priority because violations result in unsafe or invalid system behaviour.
- **MR-1** is high priority because performance failures lead to automarker timeouts.
- **FR-I1** and **FR-U2** are medium priority as they support core functionality but are not safety-critical in isolation.

### 3.2 Pre-requisites

- **FR-S2** depends on:
  - FR-U2 (correct STEP movement)
  - geometry utilities (FR-U1 / FR-U3)
- **FR-S3** depends on:
  - correct backend data retrieval (FR-I1)
- **MR-1** depends on:
  - FR-S2 (pathfinding complexity dominates runtime)
  - MR-2 (expansion limits)
- **FR-I1** has no functional pre-requisites beyond configuration
- **FR-U2** is a base requirement with no dependencies

### 3.3 Dependency Summary

```
FR-U2 → FR-S2 → MR-1
FR-I1 → FR-S3
```

---

## 4. Validation vs Verification

For each requirement, the appropriate balance of validation and verification was considered.

| Requirement | Validation | Verification | Justification |
|------------|------------|--------------|---------------|
| FR-S2 | Yes | Yes | Requires realistic scenarios and strict geometric correctness |
| FR-S3 | Yes | Yes | Scenario-driven rule enforcement plus deterministic verification |
| FR-I1 | No  | Yes | Structural integration requirement |
| FR-U2 | No  | Yes | Deterministic mathematical behaviour |
| MR-1  | Yes | Yes | Real execution timing and measurable thresholds |

---

## 5. Test and Analysis Approaches (A&T)

| Requirement | Test Level | Rationale |
|------------|------------|-----------|
| FR-U2 | Unit Testing | Pure geometric primitive, isolated and deterministic |
| FR-I1 | Integration Testing | Requires observing interactions between service and backend |
| FR-S2 | System Testing | Emergent behaviour from pathfinding and geometry |
| FR-S3 | System Testing | Multi-constraint filtering across components |
| MR-1  | System / Acceptance | Performance measurable only in integrated context |

---

## 6. Instrumentation and Scaffolding Strategy

Instrumentation is used to improve **observability, controllability, and measurability**
of requirement-relevant behaviour.  
Scaffolding provides the supporting infrastructure that enables controlled and repeatable
use of that instrumentation.

### FR-U2 — STEP-based movement

- **Instrumentation**
  - Direct access to returned coordinates
  - Numeric distance computation against the STEP constant
- **Scaffolding**
  - Pure unit-test harness with no external dependencies

---

### FR-I1 — Backend queried once per request

- **Instrumentation**
  - Call-count tracking for backend queries
  - Distinction between different backend endpoints
- **Scaffolding**
  - Mock ILP backend client
  - Deterministic mock responses

---

### FR-S3 — Drone selection constraints

- **Instrumentation**
  - Visibility of filtering outcomes (accepted vs rejected drones)
- **Scaffolding**
  - Synthetic drone datasets exercising individual constraints
  - Backend data isolation via mocks

---

### FR-S2 — Restricted-area avoidance

- **Instrumentation**
  - Expose complete path segments, enabling post-hoc segment–polygon intersection checks in system tests.
- **Scaffolding**
  - Synthetic restricted polygons
  - Reduced, controlled map sections

---

### MR-1 — Performance constraint

- **Instrumentation**
  - Wall-clock timing around complete request execution
  - Clear request boundary definition
- **Scaffolding**
  - Fixed datasets
  - Bounded search regions
  - Mocked backend interactions

---

## 7. Evaluation of Instrumentation Adequacy

The proposed instrumentation is **adequate for LO2 purposes**:

- Hidden internal behaviour is made observable (FR-I1, FR-S2).
- Rule-based decision logic can be controlled and analysed (FR-S3).
- Deterministic and measurable behaviour is supported (FR-U2, MR-1).

### Limitations

- Mocked backends do not capture real network latency or concurrency effects (FR-I1).
- Geometry testing cannot exhaustively cover all floating-point boundary cases (FR-S2).
- Performance measurements reflect controlled rather than worst-case conditions (MR-1).

### Possible Improvements

- Limited testing against a real backend for FR-I1.
- Statistical sampling over larger map regions for MR-1.
- Mutation-based sensitivity checks for FR-S2 and FR-S3.

These improvements are acknowledged but excluded due to time and resource constraints.

---

## 8. Resource Constraints and Trade-offs

- Full pathfinding verification is computationally expensive; restriction is applied.
- Backend data variability necessitates mocking for repeatability.
- Performance testing cannot be exhaustive; representative cases are selected.
- High-priority requirements are tested first due to limited time.

---

## 9. Lifecycle Model Selection (V-Model)

I adopt the **V-Model lifecycle**, as described in Y&P Chapter 20. The V-Model links development stages with corresponding testing phases (unit → integration → system → acceptance). It fits the ILP microservice because requirements span multiple levels and require structured verification.

## 10. Mapping Selected Requirements to Lifecycle Testing Phases

| Requirement | Lifecycle Phase | Rationale |
|------------|-----------------|-----------|
| **FR-U2 — nextPosition must move exactly one STEP** | **Unit Testing** | Deterministic geometric primitive; best verified immediately after implementation. |
| **FR-I1 — Backend must be queried once per planning request** | **Integration Testing** | Requires correct interaction between service layer and ILP backend (mocks/spies). |
| **FR-S2 — Paths must not enter restricted areas** | **System Testing** | Emergent behaviour involving geometry, pathfinding, and filtering; only visible at system level. |
| **FR-S3 — Drone selection must respect capability & capacity** | **System Testing** | Requires system-level filtering and backend data. |
| **MR-1 — All requests must complete within 30 seconds** | **System + Acceptance Testing** | Performance constraints measurable only in integrated environment and validated during acceptance/performance evaluation. |

## 11. Test Schedule

| Week | Activities |
|------|-----------|
| Week 1 | Finalise requirement selection and dependencies |
| Week 2 | Implement unit tests for FR-U2 |
| Week 3 | Integration tests for FR-I1 and FR-S3 |
| Week 4 | System tests for FR-S2 |
| Week 5 | Performance tests for MR-1 |
| Week 6 | Review adequacy and coverage |

---

## 12. Evaluation of the Quality of the Test Plan 
The quality of the test plan was evaluated by considering whether the selected requirements and testing levels provide adequate coverage of the system’s key risks under realistic resource constraints. The plan deliberately focuses on a small but diverse set of requirements spanning unit, integration, system, and measurable levels, rather than attempting exhaustive coverage of all functional behaviour.

A potential omission of the plan is that not all system requirements are tested directly, and some behaviours (e.g. secondary path optimality or rare constraint combinations) are not exhaustively explored. However, this risk is mitigated by prioritising requirements whose failure would result in unsafe behaviour (FR-S2), invalid outputs (FR-S3), or infeasible execution (MR-1). 

In particular, FR-S2 represents the highest residual risk in the plan. Although unit-level geometric predicates can be verified in isolation, they cannot establish whole-path safety. System-level testing was therefore chosen deliberately, despite higher cost, because it is the lowest level at which segment–polygon interactions and search behaviour can be observed together. 

This introduces a trade-off: exhaustive geometric coverage is sacrificed in favour of representative scenario coverage. The plan accepts this limitation explicitly, as exhaustive verification would be computationally infeasible and disproportionate to the risks addressed by LO2. Overall, the test plan is considered adequate for LO2 purposes because it targets the dominant failure modes of the system while explicitly acknowledging and controlling for the limitations imposed by time, computational cost, and backend variability.

Overall, the test plan is considered adequate for LO2 purposes because it targets the dominant failure modes of the system while explicitly acknowledging and controlling for the limitations imposed by time, computational cost, and backend variability.

---

## 13. Summary

This test planning document demonstrates a coherent LO2-level analysis by:

- selecting diverse, multi-level requirements
- mapping them to appropriate test levels and A&T approaches
- identifying necessary instrumentation and scaffolding
- evaluating the adequacy and limitations of those choices

It provides a sound foundation for the execution (LO3), architectural support (LO4),
and review (LO5) activities that follow.
