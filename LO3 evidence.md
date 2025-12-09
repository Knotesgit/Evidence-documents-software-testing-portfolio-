# LO3 Evidence (Preliminary) — Application of Testing Techniques and Coverage

> *This document presents preliminary evidence for Learning Outcome 3 based on the CW1 test suite of the ILP Drone Delivery microservice. It will be extended in CW2 to include additional endpoints, coverage analysis, and system-level evaluation.*

---

## 1. Range of Techniques Used

The CW1 test suite applies a range of testing techniques at different levels of abstraction:

| Level | Technique | Evidence | Description |
|-------|------------|-----------|-------------|
| **Unit** | White-box testing (structural) | `GeoServiceTests.java` | Exercises all geometric functions — distance, angle-based movement, and polygon inclusion. Includes symmetry, boundary, and determinism checks. |
| **Integration** | Black-box testing (functional) | `DistanceEndpointsTests.java`, `IsInRegionEndpointTests.java`, `NextPositionEndpointTests.java` | Uses **MockMvc** to validate controller behaviour, JSON (de)serialization, and error handling for valid and invalid input. |
| **System / API sanity** | TBC in cw2 | TBC in cw2 | TBC in cw2 |
| **Cross-cutting validation** | Boundary-value and negative testing | Across all endpoint tests | Covers malformed JSON, missing fields, out-of-range coordinates, and closed-polygon enforcement. |

These techniques demonstrate practical application of both **functional** and **structural** approaches consistent with ISO 29119-2/-3.

---

## 2. Evaluation Criteria and Adequacy Measures

Testing adequacy was evaluated using three complementary measures:

1. **Coverage metrics** — collected via IntelliJ IDEA’s built-in coverage tool  
   - Line coverage ≈ **97 %** overall  
   - Branch coverage ≈ **75 %**
2. **Boundary adequacy** — each equivalence partition tested at and around its threshold:  
   - Distance threshold = 0.00015°  
   - Angle multiples = 22.5°  
   - Polygon closure and vertex inclusion.
3. **Yield** — all ≈ 34 tests executed successfully (**0 failures**) across five test classes.

These metrics indicate that the CW1 test suite exercises nearly all executable code and validates both normal and exceptional behaviours.

---

## 3. Results of Testing

| Test Class | Focus | Tests | Result | Notes |
|-------------|--------|--------|--------|-------|
| `GeoServiceTests` | Core geometry (distance, movement, polygon) | 7 |  Pass | Achieved 100 % line, ~76 % branch coverage. |
| `DistanceEndpointsTests` | `/distanceTo`, `/isCloseTo` endpoints | 9 |  Pass | Valid and invalid JSON requests. |
| `IsInRegionEndpointTests` | Polygon inclusion logic | 8 |  Pass | Includes border and vertex tests; invalid polygons → 400. |
| `NextPositionEndpointTests` | Directional step calculation | 8 |  Pass | Tests valid/invalid angles and malformed JSON. |
| `GetEndpointsTests` | Service health and UID | 2 |  Pass | Confirms actuator and identifier endpoints. |

No functional failures were observed; results match the ILP specification for CW1.

---

## 4. Evaluation and Reflection

**Strengths**
- Very high **line coverage** and solid **branch coverage** on computational logic.  
- Integration tests systematically verify controller behaviour, including error paths.  
- Functional correctness of all four major endpoints has been established.

**Branch coverage rationale**  
The overall branch coverage (~75 %) is slightly lower because several controller endpoints reuse shared validation methods. These were thoroughly tested once and intentionally not re-tested in every endpoint to avoid redundant cases that would not improve fault detection.

**Known gaps and limitations**
- Some defensive controller branches remain untested due to validation reuse.  
- Non-functional and performance aspects are not yet evaluated.  
- No mutation or fault-based adequacy analysis at this stage.

**Planned improvements (for CW2)**
- Extend coverage to new CW2 endpoints and integration flows.  
- Add targeted negative tests to hit remaining defensive branches without duplication.  
- Incorporate mutation-testing metrics and summarise combined coverage across CW1 + CW2.  
- Maintain a concise test log linking discovered issues → fixes → re-runs.


