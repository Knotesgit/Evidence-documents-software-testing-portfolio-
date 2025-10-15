# Test Planning Document — ILP Drone Delivery Microservice

---

## 1. Introduction and Scope
This Test Planning Document supports **Learning Outcome 2** — “Design and implement comprehensive test plans with instrumented code.”  
It builds upon the *Requirements Document (LO1 evidence)* for the ILP Drone Delivery microservice.  
The scope covers a small but diverse subset of requirements to demonstrate planning, instrumentation, and lifecycle mapping.

**Selected Requirements:**
1. **FR-1 `/api/v1/distanceTo`** – Compute the Euclidean distance between two coordinates.  
2. **FR-4 `/api/v1/isInRegion`** – Determine whether a position lies inside a polygonal region.

These two functions represent different testing levels:
- `distanceTo` → deterministic mathematical logic (unit + integration)
- `isInRegion` → geometric reasoning with branching and edge cases (integration + system)

---

## 2. Requirement Prioritisation and Quality Goals
| Requirement | Priority | Rationale | Desired Quality |
|--------------|-----------|------------|----------------|
| FR-1 `/distanceTo` | **High** | Core computational building block used by multiple endpoints; must be precise and reliable. | Accuracy ≤ 1e-12, deterministic results, full boundary coverage. |
| FR-4 `/isInRegion` | **Medium–High** | Essential for validating drone route legality; more complex logic with polygon edge cases. | Robustness against malformed input; correctness on boundary inclusion. |

Quality attributes targeted:
- **Reliability:** consistent and repeatable outcomes  
- **Correctness:** mathematically sound calculations  
- **Performance:** each request completes ≤ 30 s in container  
- **Robustness:** graceful handling of invalid or missing fields  

---

## 3. Scaffolding and Instrumentation

### 3.1 Scaffolding
To enable isolated and repeatable testing:
- Implement a **`GeoService`** class exposing pure methods (`distanceBetween`, `isPointInRegion`).
- Use ** `Coordinate` and `Region` objects** in unit tests to avoid dependency on HTTP or JSON parsing.
- Apply **JUnit 5** and **MockMvc** for integration tests, enabling full endpoint coverage without manual deployment.
- Include **test data fixtures** for polygons and coordinates in `src/test/resources`.

### 3.2 Instrumentation
Instrumentation will make program state and metrics observable:
- **Logging (SLF4J)** at `DEBUG` level to trace validation and decision branches.
- **Execution timing hooks** to verify performance (measured via `System.nanoTime()`).
- **Custom assertion messages** in JUnit for traceability of failure points.
- **Docker health-check endpoint** (`/actuator/health`) ensures service availability in container context.

All instrumentation is non-intrusive and can be toggled via application configuration.

---

## 4. Test Process and Lifecycle Mapping

### Chosen Lifecycle: **V-Model (Verification and Validation)**
| Phase | Verification Activity | Validation/Test Level | Deliverables |
|--------|----------------------|----------------------|--------------|
| Requirements | Review of LO1 document for testability | — | Requirements checklist |
| Design | Identify test models (equivalence partitions, boundary sets) | Static review | Test model notes |
| Implementation | Develop unit tests for `GeoService` | **Unit Testing** | JUnit test classes |
| Integration | Use MockMvc to test controllers and JSON serialization | **Integration Testing** | Endpoint test reports |
| System | Deploy container and test via Postman or automated suite | **System Testing** | Test report logs, Docker run results |
| Acceptance | Ensure endpoints meet ILP specification and automarker constraints | **Acceptance Testing** | Evidence screenshots, timing data |

Testing occurs iteratively following each implementation increment, ensuring traceability between code, tests, and requirements.

---

## 5. Risks, Omissions, and Mitigation

| Risk ID | Description | Impact | Mitigation |
|:--|:--|:--|:--|
| R1 | Floating-point precision causes minor discrepancies in equality checks. | Medium | Use tolerance `1e-12` and avoid direct equality comparisons. |
| R2 | Invalid JSON structures not covered by current unit tests. | Medium | Expand MockMvc tests for null and malformed requests. |
| R3 | Polygon self-intersection undetected by simple point-in-polygon algorithm. | Low | Document limitation; future version may integrate geometry library. |
| R4 | Over-logging may clutter outputs in automarker. | Low | Limit logs via profile-based configuration. |

---

## 6. Evaluation of Plan and Instrumentation

**Plan Adequacy:**  
- Requirements chosen reflect both computational and geometric diversity.  
- Lifecycle mapping ensures full coverage from unit to system level.  
- Risk analysis identifies major vulnerabilities early.

**Instrumentation Adequacy:**  
- Logging and metrics provide observability without altering functional behaviour.  
- Scaffolding allows isolated testing and CI integration.  
- Improvement opportunity: add coverage collection (Jacoco) to quantify completeness.

## 7. References
- ISO/IEC/IEEE 29119-1 (2022): §4.2 Test plans and strategies  
- ISO/IEC/IEEE 29119-2 (2021): §7.2 Test strategy and planning process  
- ISO/IEC/IEEE 29119-3 (2021): §7.2 Test Plan template  
- Pezzè & Young (Ch. 20): Planning and Process  
- Software Testing Tutorial LO2 (2025/6)

