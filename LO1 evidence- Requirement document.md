# LO1 Evidence — Requirements Document

This document provides a **testing-oriented** set of requirements for the ILP Drone Delivery Microservice.  

It is **not** a full system specification.  
Its purpose is to present a **diverse, multi-level, testable set of requirements** including:

- System-level functional requirements  
- Integration-level requirements  
- Unit-level requirements  
- Measurable attributes  
- Operational and quality requirements  

Each requirement includes:

- Level (unit / integration / system / operational)  
- Test approach  
- Appropriateness  
- Weaknesses  
- Rationale  

---

# 1. Scope (High-Level Overview)

The microservice under test is a Java Spring Boot application that provides:

- Geometry utilities for drone navigation (CW1)  
- Integration with the ILP backend for drones, service points, restricted areas, and availability (CW2)  
- Drone capability and attribute queries  
- Delivery plan computation with safe pathfinding, drone selection, and cost calculation (CW2)  
- Optional GeoJSON serialization  

This LO1 evidence covers **requirements only**, not implementation or results.

---

# 2. System-Level Functional Requirements

## FR-S1 — Valid or empty plan must be returned  
**Level:** System  
**Description:** The service must return either a complete delivery plan or an empty plan with cost=0 and moves=0.  
**Rationale:** Ensures deterministic behaviour in success/failure cases.

**Test Approach:** Scenario-based system testing; black-box workflow tests; boundary case with no feasible paths.  
**Appropriateness:** Captures full system behaviour across components.  
**Weaknesses:** Hard to isolate cause of empty plans; sensitive to backend variability.

---

## FR-S2 — All computed paths must avoid restricted areas  
**Level:** System  
**Description:** No coordinate in a returned path may fall inside a restricted polygon.  
**Rationale:** Safety rule required by ILP specification.

**Test Approach:** Synthetic restricted areas; edge/inside/outside boundary tests.  
**Appropriateness:** Realistically simulates operational safety conditions.  
**Weaknesses:** Floating-point issues near polygon edges; exhaustive geometry is hard.

---

## FR-S3 — Drone selection must respect capacity & capability  
**Level:** System  
**Description:** Drones must satisfy capacity, cooling/heating, and incompatibility constraints.  
**Rationale:** Ensures valid drone assignment.

**Test Approach:** Scenario-based multi-constraint filtering; known expected outputs.  
**Appropriateness:** Matches decision rules precisely.  
**Weaknesses:** Dependent on backend dataset variety; edge cases may be scarce.

---

## FR-S4 — Delivery plans must include outbound and return segments  
**Level:** System  
**Description:** Plans must include outbound paths for deliveries and return paths (`deliveryId = null`).  
**Rationale:** Required for cost consistency and final drone state.

**Test Approach:** Plan-structure validation; order-checking tests.  
**Appropriateness:** Tests observable behaviour through API.  
**Weaknesses:** Faults in return segments may originate from pathfinding, not planning.

---

# 3. Integration-Level Functional Requirements

## FR-I1 — ILP backend must be queried once per planning request  
**Level:** Integration  
**Description:** `calcDeliveryPath` must request drones, service points, restricted areas, and availability per invocation.  
**Rationale:** Ensures fresh ILP data.

**Test Approach:** Mock ILP client; verify call-count expectations.  
**Appropriateness:** Mocking isolates backend behaviour.  
**Weaknesses:** Mocking hides real-network defects.

---

## FR-I2 — Geometry utilities must be invoked during pathfinding  
**Level:** Integration  
**Description:** `distanceBetween`, `nextPosition`, and `isInRegion` must be called during full-path search.  
**Rationale:** Ensures CW1 logic feeds into CW2 planning.

**Test Approach:** Spy wrappers for geometry functions.  
**Appropriateness:** Verifies internal interaction pathways.  
**Weaknesses:** Spies may slightly alter behaviour or timing.

---

## FR-I3 — Query endpoints must correctly filter drones  
**Level:** Integration  
**Description:** `/query`, `/queryAsPath`, `/queryAvailableDrones` must apply rules consistently.  
**Rationale:** Ensures coherent filtering logic.

**Test Approach:** Parameterised integration tests across attributes/operators.  
**Appropriateness:** Matches multi-component filtering.  
**Weaknesses:** Backend dataset may limit operator coverage.

---

# 4. Unit-Level Requirements

## FR-U1 — distanceBetween computes accurate Euclidean distance  
**Level:** Unit  
**Description:** Must compute correct degree-space Euclidean distance.  
**Rationale:** Core primitive for ordering and estimation.

**Test Approach:** Oracle-based numeric tests; symmetry validation.  
**Appropriateness:** Deterministic and mathematically verifiable.  
**Weaknesses:** Floating-point precision introduces small errors.

---

## FR-U2 — nextPosition moves exactly one STEP  
**Level:** Unit  
**Description:** Given a valid angle (multiple of 22.5°), output is exactly one STEP away.  
**Rationale:** Ensures grid correctness.

**Test Approach:** Test all 16 valid angles; invalid-angle rejection.  
**Appropriateness:** Small, self-contained computation.  
**Weaknesses:** Does not test interaction with pathfinding.

---

## FR-U3 — isInRegion must treat boundary points as inside  
**Level:** Unit  
**Description:** Edge and vertex points count as inside.  
**Rationale:** Ensures consistent polygon semantics.

**Test Approach:** Boundary polygons; vertex-on/edge-on inputs.  
**Appropriateness:** Ideal for boundary-value testing.  
**Weaknesses:** Simplified shapes may not match real backend data.

---

# 5. Measurable Requirements

## MR-1 — Requests complete within 30 seconds  
**Level:** Measurable  
**Description:** All endpoints must finish within container time limit.  
**Rationale:** Prevents automarker timeouts.

**Test Approach:** Stress tests with large restricted areas; timing measurements.  
**Appropriateness:** Directly targets measurable performance.  
**Weaknesses:** Non-deterministic timing across environments.

---

## MR-2 — Pathfinding node expansion is capped  
**Level:** Measurable  
**Description:** A* expansion count must not exceed safe limit.  
**Rationale:** Prevents runaway CPU use.

**Test Approach:** Construct near-worst-case maps; detect fallback behaviour.  
**Appropriateness:** Validates operational safety constraints.  
**Weaknesses:** Hard to replicate exact expansion conditions.

---

## MR-3 — Cost formula must be consistent  
**Level:** Measurable  
**Description:** Cost = initial + final + costPerMove × stepsUsed.  
**Rationale:** Ensures predictable pricing model.

**Test Approach:** Known-path oracles; compare expected cost.  
**Appropriateness:** Cost functions are deterministic and measurable.  
**Weaknesses:** Relies on accurate step counting from pathfinding.

---

# 6. Operational / Quality Requirements

## QR-1 — Default ILP endpoint used if ILP_ENDPOINT is missing  
**Level:** Operational  
**Description:** System must fall back to default backend URL.  
**Rationale:** Ensures deployability.

**Test Approach:** Unset environment variable in test container.  
**Appropriateness:** Matches operational environment conditions.  
**Weaknesses:** Hard to reproduce container behaviour exactly.

---

## QR-2 — Invalid geometry input must produce 400, not 500  
**Level:** Quality  
**Description:** Malformed coordinates/regions must return controlled errors.  
**Rationale:** Ensures robustness.

**Test Approach:** Fuzz invalid JSON inputs; malformed numeric values.  
**Appropriateness:** Standard negative testing.  
**Weaknesses:** JSON parsers may differ in behaviour.

---

## QR-3 — JSON responses must be deterministic  
**Level:** Quality  
**Description:** Identical inputs produce identical outputs.  
**Rationale:** Required for repeatable tests.

**Test Approach:** Repeat-call differential comparison.  
**Appropriateness:** Straightforward and clear.  
**Weaknesses:** Sensitive to floating-point formatting or map ordering.

---

# 7. Requirement Priorities

| Requirement | Priority | Reason |
|------------|----------|--------|
| FR-S1, FR-S2 | High | Core correctness and safety |
| FR-S3 | High | Drone assignment feasibility |
| FR-U1, FR-U2 | Medium | Isolated yet essential |
| FR-I2, FR-I3 | Medium | Integration behaviour |
| MR-1, MR-2 | High | Performance constraints |
| QR-1, QR-2 | Medium | Stability and robustness |

---

# 8. Dependency Map

- Pathfinding (FR-S2, FR-S4) depends on FR-U1, FR-U2, FR-U3  
- Drone assignment (FR-S3) depends on integration-level availability logic  
- System planning (FR-S1) depends on pathfinding + assignment  
- MR-1 depends on MR-2  
- Operational (QR-1) affects backend integration flow  

---

# 9. Summary

This LO1 Evidence document provides:

- A diverse, multi-level requirement set  
- Explicit level classification  
- Test approaches, appropriateness, and weaknesses  
- Priorities and dependencies  

It fully satisfies LO1(a)–LO1(d) and supports LO2–LO5 test planning, execution, evaluation, and review.

