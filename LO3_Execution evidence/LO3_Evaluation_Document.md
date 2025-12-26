# LO3 Evidence — Evaluation of Testing

## Scope and Purpose

This document evaluates the adequacy and outcomes of the testing activities carried out
for the ILP Drone Delivery Microservice.

The purpose is **not** to claim full system correctness, but to assess how effectively
the executed tests provide confidence that selected requirement properties hold within
the defined testing scope.

The evaluation covers the following requirements:

- **FR-U2 — STEP-based movement correctness**
- **FR-I1 — Backend queried exactly once per planning request**
- **FR-S2 — Restricted-area avoidance**
- **FR-S3 — Drone selection constraints**
- **MR-1 — Performance constraint**

All supporting evidence is provided in the `LO3_Execution evidence` directory as code
excerpts, behaviour tables, and test execution results.

---

## FR-U2 — STEP-based Movement Correctness

All unit-level tests evaluating STEP-based movement passed under tolerance-aware numeric
assertions.

The evaluation criterion checks that the displacement magnitude of a computed movement
equals the STEP constant, independent of direction. This directly reflects the semantic
content of the requirement, which specifies a deterministic numeric property.

**Supporting evidence:**

![Assertion-based verification of STEP displacement](FR-U2/FR-U2_code.png)

![Successful execution of unit tests](FR-U2/FR-U2_test_result.png)

![Method-level coverage of STEP movement primitive](FR-U2/FR-U2_JaCoCo_MethodCoverage_nextPosition.png)

The coverage evidence shows complete execution of the STEP movement primitive.
Coverage of unrelated geometric utilities is intentionally limited and consistent with
the restricted evaluation scope of FR-U2.

Residual risk is limited to extreme floating-point edge cases at unusual angles, which
are acceptable given the representative input coverage.

---

## FR-I1 — Backend Queried Exactly Once per Planning Request

All integration-level tests passed with backend interaction counts matching the
requirement.

The evaluation criterion directly observes the number of backend queries issued during a
planning request. A test passes if and only if the backend is queried exactly once per
request.

**Supporting evidence:**

![Instrumented verification of backend query count](FR-I1/FR-I1_code.png)

![Successful execution of backend interaction tests](FR-I1/FR-I1_test_result.png)

Returned data correctness and backend response semantics are intentionally excluded, as
FR-I1 defines an interaction property rather than a computational result.

---

## FR-S2 — Restricted-area Avoidance

All system-level scenario tests passed under a geometric safety oracle that evaluates
continuous movement segments against restricted polygons.

The evaluation explicitly checks **segment–polygon intersection**, rather than only
verifying discrete waypoints. This matches the operational interpretation of path
execution and avoids false confidence from endpoint-only checks.

The evaluated scenarios include:
- Feasible routing entirely outside restricted areas
- Rejection of paths intersecting restricted-area interiors
- Boundary-contact cases treated as blocked
- Near-boundary paths strictly outside restricted areas accepted

**Supporting evidence:**

![System-level path validation logic for restricted-area avoidance](FR-S2/FR-S2_code.png)

![Geometric oracle enforcing segment–polygon safety](FR-S2/FR-S2_oracle.png)

![Scenario-based system test execution results](FR-S2/FR-S2_test_result.png)

No generated path violated the geometric safety oracle.

Residual risk remains for highly irregular or adversarial polygon geometries and extreme
floating-point boundary cases, which are not exhaustively explored.

---

## FR-S3 — Drone Selection Constraints

All system-level tests evaluating drone selection passed under the defined constraint
scenarios.

The evaluation checks that selected drones satisfy capacity and capability constraints.
In scenarios where multiple drones are eligible, the oracle accepts **any eligible drone**
rather than enforcing an undocumented tie-breaking rule.

**Supporting evidence:**

![Drone selection logic under different constraint scenarios](FR-S3/FR-S3-4_code.png)

![Additional constraint combination scenarios](FR-S3/FR-S3-5_code.png)

![Multi-eligible drone scenario](FR-S3/FR-S3-6_code.png)

![Execution results for drone selection scenarios](FR-S3/FR-S3_test_result.png)

This evaluation aligns with the semantic intent of the requirement rather than imposing
implementation-specific ordering assumptions.

---

## MR-1 — Performance Constraint

All performance tests completed within the specified time threshold.

Repeated executions under controlled inputs produced stable execution times. Additional
sanity checks confirm that all measured runs involved non-trivial planning behaviour,
excluding empty-plan or early-exit cases that could distort performance results.

**Supporting evidence:**

![Instrumentation for measuring execution time in performance tests](MR-1/MR-1_code.png)

![Performance test execution](MR-1/MR-1_test_result.png)

![Reproducibility evidence across repeated runs](MR-1/MR-1_reproducible_evidence.png)

![Sanity check showing non-zero planning effort](MR-1/MR-1_totalmove_evidence.png)

The evaluation provides confidence in typical-case performance behaviour while
acknowledging that exhaustive performance guarantees under all workloads are infeasible
within the coursework scope.



