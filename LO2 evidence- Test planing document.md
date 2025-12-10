# LO2 Evidence — Test Planning Document

## 1. Purpose of This Test Plan

The purpose of this test planning document is to demonstrate how I satisfy LO2 (Analyse requirements to determine appropriate testing strategies).  
This document focuses specifically on five selected requirements from the LO1 Evidence file:

- **FR-S2 — Paths must not enter restricted areas**  
- **FR-S3 — Drone selection must respect capacity & capability**  
- **FR-I1 — ILP backend must be queried once per planning request**  
- **FR-U2 — nextPosition must move exactly one STEP**  
- **MR-1 — All requests must complete within 30 seconds**

These requirements were selected because they cover system, integration, unit, and measurable attributes, providing sufficient diversity for planning activities.

---

## 2. Selected Requirements Summary

| ID     | Level       | Type          | Priority | Reason |
|--------|-------------|---------------|----------|--------|
| FR-S2  | System      | Functional    | High     | Core safety requirement; constrained geometry central to pathfinding |
| FR-S3  | System      | Functional    | High     | Determines correctness of drone allocation |
| FR-I1  | Integration | Functional    | Medium   | Ensures correctness of backend interactions |
| FR-U2  | Unit        | Functional    | Medium   | Core geometric primitive used throughout pathfinding |
| MR-1   | Measurable  | Performance   | High     | Directly affects feasibility under ILP marking constraints |

---

## 3. Priority and Pre‑requisite Analysis

### 3.1 Priority Rationale
- **FR-S2** and **FR-S3** are *high priority* because they determine the correctness and safety of system behaviour.
- **MR-1** is *high priority* because performance violations lead to automarker timeouts, making the system unusable.
- **FR-I1** and **FR-U2** are *medium priority* because they support core behaviour but are not inherently safety-critical.

### 3.2 Pre‑requisites

- **FR-S2** depends on:
  - FR-U2 (STEP movement must be correct)
  - FR-U1/FR-U3 (geometry utilities)
- **FR-S3** depends on:
  - correctness of ILP backend data retrieval (FR-I1)
- **MR-1** depends on:
  - FR-S2 (pathfinding behaviour greatly impacts runtime)
  - MR-2 (expansion limits)
- **FR-I1** has no functional pre‑requisite other than configuration
- **FR-U2** is a base requirement with no dependencies

### 3.3 Dependency Summary

```
FR-U2 → FR-S2 → MR-1
FR-I1 → FR-S3
```

These dependencies shape the order in which tests must be executed in the test schedule.

---

## 4. Validation vs Verification (Chapter 2)

For each requirement, I considered whether validation, verification, or both are appropriate.

| Requirement | Validation? | Verification? | Justification |
|-------------|-------------|----------------|---------------|
| **FR-S2** | Yes | Yes | Requires realistic input scenarios (validation) and strict geometric correctness (verification) |
| **FR-S3** | Yes | Yes | Requires scenario-driven checking of attribute filtering plus rule-based verification |
| **FR-I1** | No | Yes | Purely structural requirement; must be verified using mocks/spies |
| **FR-U2** | No | Yes | Deterministic, mathematically defined behaviour |
| **MR-1** | Yes | Yes | Requires real-timing execution (validation) and measurable thresholds (verification) |

Key observation:  
**System-level requirements (FR-S2, FR-S3) require both validation and verification**, while low-level requirements mostly require verification.

---

## 5. Engineering Principles Applied (Chapter 3)

### **5.1 Partition**
Used for both FR-S2 and FR-S3.

- FR-S2 partitions the map into regions: *inside restricted area*, *on boundary*, *outside*, enabling representative scenario tests.
- FR-S3 partitions drone capabilities into categories (e.g., cooling only, heating only, both false, insufficient capacity).

### **5.2 Restriction**
Applied to FR-S2 and MR-1.

- Restricting pathfinding search depth or bounding-box region reduces state space and makes performance testing feasible.
- Restriction allows creation of "hard" maps that exercise A* under predictable stress conditions.

### **5.3 Visibility**
Applied to FR-I1.

- Mock ILP backend increases visibility by revealing exactly how many backend calls are made.
- Improving observability of internal interactions supports verification.

### **5.4 Sensitivity**
Applied to FR-S2 and MR-1.

- Construct “fragile” maps that produce failures with small deviations, exposing corner-case pathfinding bugs.
- MR-1 sensitivity tests reveal how performance degrades under load.

### **5.5 Redundancy**
Applied to FR-U2.

- Repeated checking of expected step length vs actual output ensures correctness.
- Redundant validation across multiple geometric scenarios increases confidence.

### **5.6 Feedback**
Applied to refinement of performance thresholds.

- Early benchmarking results guide refinement of test scenarios for MR-1.
- Helps prioritise test execution order.

---

## 6. Test & Analysis Approaches (A&T Methods)

For each requirement, the following A&T approaches are chosen:

### **FR-S2 — Path must not enter restricted areas**
- Scenario-based system testing  
- Boundary-value geometric tests  
- Model checking limited path sequences (lightweight)  
- Appropriateness: realistic ILP conditions require scenario tests  
- Weakness: geometric precision makes edge cases difficult

### **FR-S3 — Drone selection must respect capacity & capability**
- Combinatorial testing of attribute combinations  
- Integration tests using controlled backend responses  
- Appropriateness: decision logic depends on multi-attribute constraint checking  
- Weakness: backend variety is limited; may miss rare combinations

### **FR-I1 — Backend queried once per request**
- Mock-based verification  
- Interaction testing using spies  
- Appropriateness: prevents reliance on unstable backend  
- Weakness: real network issues are not captured

### **FR-U2 — nextPosition must move exactly one STEP**
- Oracle-based unit tests  
- Full 16-direction test suite  
- Appropriateness: deterministic function; ideal for unit-level verification  
- Weakness: does not reveal system-level movement problems

### **MR-1 — Request must complete within 30 seconds**
- Performance stress tests  
- Timing measurement under varying map complexity  
- Appropriateness: matches measurable system constraints  
- Weakness: timing varies by execution environment

---

## 6. Lifecycle Model Selection (V-Model)

I adopt the **V-Model lifecycle**, as described in Y&P Chapter 20. The V-Model links development stages with corresponding testing phases (unit → integration → system → acceptance). It fits the ILP microservice because requirements span multiple levels and require structured verification.

## 7. Mapping Selected Requirements to Lifecycle Testing Phases

| Requirement | Lifecycle Phase | Rationale |
|------------|-----------------|-----------|
| **FR-U2 — nextPosition must move exactly one STEP** | **Unit Testing** | Deterministic geometric primitive; best verified immediately after implementation. |
| **FR-I1 — Backend must be queried once per planning request** | **Integration Testing** | Requires correct interaction between service layer and ILP backend (mocks/spies). |
| **FR-S2 — Paths must not enter restricted areas** | **System Testing** | Emergent behaviour involving geometry, pathfinding, and filtering; only visible at system level. |
| **FR-S3 — Drone selection must respect capability & capacity** | **System Testing** | Requires system-level filtering and backend data. |
| **MR-1 — All requests must complete within 30 seconds** | **System + Acceptance Testing** | Performance constraints measurable only in integrated environment and validated during acceptance/performance evaluation. |

## 8. Resource Constraints & Trade‑offs

- Full pathfinding verification is computationally expensive; therefore **restriction** is used to target specific map sections rather than entire maps.
- ILP backend data may change; therefore mocks are required for deterministic integration tests.
- Performance tests cannot be exhaustive; representative stress cases are selected instead.
- Time limitations require prioritising high-priority requirements (FR-S2, FR-S3, MR-1).

---

## 8. Test Schedule (Timeline)

| Week | Activities |
|------|------------|
| **Week 1** | Finalise requirement selection; dependency and priority analysis |
| **Week 2** | Implement unit tests for FR-U2; construct geometry oracles |
| **Week 3** | Implement integration tests for FR-I1 and FR-S3 (mocks + spies) |
| **Week 4** | Implement system tests for FR-S2 (scenario maps + boundary cases) |
| **Week 5** | Implement performance tests for MR-1; refine based on feedback |
| **Week 6** | Consolidate results; perform coverage measurement; review test adequacy |

---

## 9. Summary

This test plan demonstrates how the selected requirements are analysed and mapped to appropriate A&T approaches, guided by:

- requirement priority  
- requirement dependency  
- validation vs verification distinction  
- engineering principles (Partition, Restriction, Visibility, Sensitivity, Redundancy, Feedback)  
- resource constraints and trade-offs  

This document fully satisfies LO2(a)–LO2(d) and provides a coherent foundation for LO3–LO5.

