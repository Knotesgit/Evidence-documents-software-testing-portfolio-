# LO3 — Test Execution and Evidence

This section presents evidence that the planned tests were implemented and executed, and that they provide meaningful pass/fail decisions for the selected requirements. The focus of LO3 is not test design, which is addressed in LO2, but the execution of those tests and the evaluation of their adequacy.

Tests were implemented using JUnit 5, with Mockito used where isolation of external dependencies was required. Execution evidence is provided in the accompanying `LO3_screenshots` directory, which contains screenshots of test oracles, executed test results, and supporting coverage or scenario visualisations where appropriate.

The executed tests span multiple test levels, including unit tests for geometric primitives, integration tests for backend interaction constraints, system-level scenario-based tests for end-to-end path planning behaviour, and measurable tests for performance constraints under controlled workloads.


## 1. Overview of Executed Tests

Tests were implemented and executed for the requirements selected in LO2, demonstrating a concrete range of testing techniques applied across multiple test levels. The objective was to ensure that each requirement is associated with an executable test and an explicit, programmatic pass/fail oracle.

Assertion-based testing was used at the unit level to verify deterministic geometric computations. These tests rely on direct numeric assertions and boolean predicates, such as equality within tolerance, empty versus non-empty results, and exact step-based movement behaviour.

Mock-based interaction testing was applied at the integration level to validate constraints on backend usage. External dependencies were isolated using Mockito, and interaction properties were verified through explicit call-count and no-extra-interaction assertions.

At the system level, scenario-based testing was employed to exercise end-to-end path-planning behaviour. Carefully constructed geometric configurations were used to drive acceptance and rejection outcomes, including interior violations, boundary conditions, forced detours, and multi-area navigation. System-level oracles verify both the existence of a valid path and the absence of restricted-area intersections along all path segments.

For measurable requirements, performance measurement techniques were used. Execution time was collected across repeated runs under controlled workloads, and statistical summaries were evaluated against defined thresholds. Additional assertions were included to exclude trivial early-exit behaviour and ensure that measured runs represent genuine path-planning execution.

---

## 2. Requirement-level Evidence 

### FR-U2 — STEP-based movement correctness

FR-U2 defines a deterministic geometric primitive and was planned in LO2 to be tested at unit level using assertion-based verification. The planned approach focused on verifying exact STEP displacement and directional correctness independently of higher-level path-planning or obstacle-avoidance logic.

The implemented tests follow this plan without deviation. Assertion-based unit tests directly verify that the computed next position advances exactly one STEP in the intended direction. Numeric assertions are applied to the resulting coordinates, using tolerance-aware comparisons to account for floating-point representation.

The pass/fail criterion for FR-U2 is fully programmatic: a test passes if and only if the displacement magnitude equals STEP (within a defined epsilon), irrespective of direction. Because FR-U2’s contract is fully defined by deterministic numeric constraints (fixed step length and direction), a direct numeric assertion is a sufficient evaluation criterion: any defect in STEP magnitude or directional calculation manifests immediately as an assertion failure. The oracle is intentionally independent of other geometric utilities and computes distance directly using elementary arithmetic, avoiding shared logic that could mask faults through common-cause errors.

![FR-U2 oracle: assertion-based verification of exact STEP displacement](LO3_screenshots/FR-U2_code.png)
*Figure — FR-U2 oracle: STEP represents the fixed movement length specified by FR-U2, while EPS is a deliberately small tolerance introduced solely to account for floating-point representation error. EPS does not relax the behavioural contract of the requirement.*

This testing approach is considered thorough for FR-U2 because the main uncertainty for this requirement is whether the implementation consistently produces the correct STEP-length displacement for the requested direction. Assertion-based unit tests reduce this uncertainty by checking the contract directly at the point it is computed, without interference from path-planning heuristics, restricted-area logic, or backend data. Exercising FR-U2 in isolation therefore builds confidence that any higher-level behaviour that depends on step-based movement is not compensating for, or hiding, a defect in the movement primitive. No additional scenario-based or integration testing is required for this requirement, as such tests would not increase confidence beyond what direct numeric verification already provides.

###
