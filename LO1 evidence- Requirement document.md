# LO1 Evidence — Requirements Document

This document provides a *testing‑oriented* set of requirements for the ILP Drone Delivery Microservice.  
It is **not** a full system specification.  
Its purpose is to provide a **diverse, testable, multi‑level requirement set** that supports LO2–LO5, as required by the marking scheme.

The requirements below cover unit‑level, integration‑level, system‑level, measurable, and operational aspects.  
Each requirement includes a **rationale** and **testability notes**, because LO2 explicitly requires that.

---

## 1. Scope (High‑Level Overview)

The service is a Java Spring Boot microservice providing:

- Geometry utilities for drone movement (CW1).  
- Integration with the ILP backend to retrieve drones, service points, restricted areas, and availability (CW2).  
- Drone capability/availability queries.  
- Computation of delivery plans, including safe pathfinding and cost calculation (CW2).  
- Optional GeoJSON serialization of flight paths.

This LO1 evidence covers only the **requirements**, not the implementation or test results.

---

## 2. Functional Requirements (System‑Level)

### **FR‑S1 — Delivery requests must produce a valid plan or an empty plan**
**Description:**  
Given a list of dispatches, the service must either return a complete delivery plan or return an empty plan with cost=0 and moves=0.

**Rationale:**  
This behaviour is required by CW2 and enables predictable testing of failure cases.

**Testability Notes:**  
Test both success and failure cases:  
- valid plan → non‑empty results  
- infeasible dispatch list → empty plan  

---

### **FR‑S2 — All computed paths must avoid restricted areas**
**Description:**  
Pathfinding must not produce coordinates located inside restricted polygons.

**Rationale:**  
Safety requirement; fundamental to ILP rules.

**Testability Notes:**  
Use synthetic restricted areas to test edge cases:  
- point on edge  
- point just inside  
- point just outside  

---

### **FR‑S3 — Drone selection must respect cooling/heating/capacity**
**Description:**  
Candidate drones must match dispatch requirements:  
- capacity ≥ required  
- cooling/heating flags compatible  
- both flags cannot be true simultaneously.

**Rationale:**  
Core of availability/feasibility logic.

**Testability Notes:**  
Unit tests on the helper logic; integration tests via `/queryAvailableDrones`.

---

### **FR‑S4 — Delivery plans must include outbound + return paths**
**Description:**  
Each delivery’s flight path must include movement from service point to destination and a corresponding return segment (which uses `deliveryId = null`).

**Rationale:**  
Preserves predictability of cost and drone state.

**Testability Notes:**  
Check ordering: outbound first, return last.

---

## 3. Functional Requirements (Integration‑Level)

### **FR‑I1 — ILP backend must be queried once per planning request**
**Description:**  
`calcDeliveryPath` must fetch drones, service points, restricted areas, and availability records at request time.

**Rationale:**  
CW2 requires external data freshness.

**Testability Notes:**  
Mock ILP client → verify number of calls.

---

### **FR‑I2 — Geometry computation is used inside system‑level pathfinding**
**Description:**  
`distanceBetween`, `isInRegion`, and `nextPosition` must be correctly called in pathfinding logic.

**Rationale:**  
Integration between CW1 and CW2 functionality.

**Testability Notes:**  
Use spies/fakes; confirm geometry routines are exercised during planning.

---

### **FR‑I3 — Query endpoints must correctly filter drones**
**Description:**  
`/query`, `/queryAsPath`, and `/queryAvailableDrones` must apply filtering consistently with attribute/operator/value rules.

**Rationale:**  
Ensures predictable integration-level logic.

**Testability Notes:**  
Multi-attribute tests, operator boundary tests.

---

## 4. Functional Requirements (Unit‑Level)

### **FR‑U1 — distanceBetween must compute Euclidean distance accurately**
**Description:**  
Inputs: two valid coordinates. Output: precise distance (degree‑space).

**Rationale:**  
Fundamental building block for pathfinding and sorting by proximity.

**Testability Notes:**  
Exact expected values; symmetry checks; zero‑distance check.

---

### **FR‑U2 — nextPosition must move exactly one STEP in a valid angle**
**Description:**  
Angle must be a multiple of 22.5°, and output must be one step away.

**Rationale:**  
Ensures discretization grid correctness.

**Testability Notes:**  
Test all 16 directions; test invalid angles.

---

### **FR‑U3 — isInRegion must treat boundary points as inside**
**Description:**  
Points on edges/vertices return true.

**Rationale:**  
Required by point-in-polygon semantics.

**Testability Notes:**  
Test edge-on, vertex-on, and nearly-on cases.

---

## 5. Measurable Requirements (MR)

### **MR‑1 — Performance: Request must complete in ≤ 30 seconds**
**Description:**  
All endpoints must finish within 30 seconds inside the ILP marking container.

**Rationale:**  
Explicit CW2 constraint; prevents infinite A* expansion.

**Testability Notes:**  
Stress pathfinding with large restricted areas.

---

### **MR‑2 — Pathfinding node expansion must remain below a safe cap**
**Description:**  
A* must stop if node expansions exceed an internal configurable limit.

**Rationale:**  
Safety requirement; prevents runaway CPU use.

**Testability Notes:**  
Trigger expansion limit → verify empty plan.

---

### **MR‑3 — Cost calculation must follow fixed formula**
**Description:**  
Cost = initial + final + costPerMove × stepsUsed.

**Rationale:**  
Predictability; correctness of planning.

**Testability Notes:**  
Use known paths with known step counts.

---

## 6. Operational / Quality Requirements (QR)

### **QR‑1 — Service must remain functional even if ILP_ENDPOINT is missing**
**Description:**  
Fallback to default ILP URL if environment variable is unset.

**Rationale:**  
Operational robustness.

**Testability Notes:**  
Unset ILP_ENDPOINT → check system still runs.

---

### **QR‑2 — Invalid geometry input must return 400, not 500**
**Description:**  
Invalid coordinates or regions → controlled 400 response.

**Rationale:**  
Input robustness and testability.

**Testability Notes:**  
Malformed JSON, null fields, NaN.

---

### **QR‑3 — JSON responses must be stable and deterministic**
**Description:**  
Given identical ILP backend data and request body, output must be deterministic.

**Rationale:**  
Required for testing repeatability.

**Testability Notes:**  
Run planning twice → compare outputs.

---

## 7. Requirement Priorities

| Requirement | Priority | Reason |
|------------|----------|--------|
| FR‑S1, FR‑S2 | High | Core correctness & safety |
| FR‑S3 | High | Drone assignment feasibility |
| FR‑U1, FR‑U2 | Medium | Important but testable in isolation |
| FR‑I2, FR‑I3 | Medium | Integration routes |
| MR‑1, MR‑2 | High | Performance cap required by CW2 |
| QR‑1, QR‑2 | Medium | Robustness and stability |

Priorities will be used in LO2 when selecting which requirements to emphasise in the test plan.

---

## 8. Dependencies

- Pathfinding depends on all geometry functions (FR‑U1, FR‑U2, FR‑U3).  
- Drone selection depends on capability + availability (FR‑S3).  
- Planning (FR‑S1/FR‑S4) depends on both selection logic and pathfinding.  
- MR‑1 depends on QR‑1 and MR‑2 (performance + expansion limit).  

Dependencies will be referenced in LO2 Activity 1 & 2.

---

## 9. Summary

This LO1 document provides a **testing‑oriented, multi‑level requirement set** with priority, rationale, testability notes, and dependencies.  
It forms the foundation for LO2 (test planning), LO3 (test execution and coverage), LO4 (evaluation), and LO5 (review and automation).

