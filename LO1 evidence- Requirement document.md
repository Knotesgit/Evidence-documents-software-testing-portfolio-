# Requirements Document — ILP Drone Delivery Microservice

---

## 1. Terminology and Data Conventions
- **Coordinate**: `{ "lng": number, "lat": number }`  
  Longitude ∈ [-180, 180], Latitude ∈ [-90, 90], both double-precision floats.  
- **Distance**: Euclidean distance in degrees (planar approximation).  
- **STEP**: Fixed movement step = `0.00015 degrees`.  
- **HTTP Format**: All endpoints use `application/json`.  
  Invalid input → `400 Bad Request`.  
- **Performance limit**: Each request must complete ≤ 30 seconds (in automarker container).  

---

## 2. Functional Requirements (FR)

### FR-1 `/api/v1/distanceTo` – Compute distance between two points
- **Input**  
  ```json
  {
    "position1": { "lng": -3.19, "lat": 55.94 },
    "position2": { "lng": -3.19, "lat": 55.95 }
  }
  ```
- **Coordinate constraints**
  1. Both "lng" and "lat" should not be NaN, null or missing
- **Behaviour**  If both coordinates are valid, return their Euclidean distance in degrees.  
- **Output**  `200 OK` with numeric body.  
- **Error**  Invalid coordinate → `400 Bad Request`.  
- **Precision**  Absolute error ≤ `1e-12`.  
- **Level**  Unit (core computation) / Integration (controller binding).

---

### FR-2 `/api/v1/isCloseTo` – Proximity threshold check
- **Input**  Same as FR-1.  
- **Behaviour**  Return `true` if distance < `0.00015`, else `false`.  
- **Output**  `200 OK` with boolean body.  
- **Error**  Invalid coordinate → `400 Bad Request`.  
- **Boundary rule**  Equality to threshold counts as `false`.  
- **Level**  Unit (threshold) / Integration (controller).

---

### FR-3 `/api/v1/nextPosition` – Step forward by angle
- **Input**  
  ```json
  {
    "start": { "lng": -3.19, "lat": 55.94 },
    "angleDegrees": 90.0
  }
  ```
- **Angle constraints**
  1. Must ∈ [0,360] 
  2. Be a multiple of 22.5° 
- **Angle definition**  East = 0°, North = 90°, West = 180°, South = 270°.  
- **Behaviour**  Return new coordinate after moving one STEP in given direction.  
- **Output**  `200 OK` with new coordinate object.  
- **Error**  Invalid start or out-of-range result → `400 Bad Request`.  
- **Level**  Unit (vector math) / Integration.


---

### FR-4 `/api/v1/isInRegion` – Point-in-polygon test (inclusive)
- **Input**  
  ```json
  {
    "position": { "lng": -3.19, "lat": 55.94 },
    "region": {
      "vertices": [
        { "lng": -3.20, "lat": 55.94 },
        { "lng": -3.18, "lat": 55.94 },
        { "lng": -3.18, "lat": 55.95 },
        { "lng": -3.20, "lat": 55.95 },
        { "lng": -3.20, "lat": 55.94 }
      ]
    }
  }
  ```
- **Region constraints**
  1. Closed (last vertex = first).  
  2. ≥ 4 vertices (including closure).  
  3. Simple (non-self-intersecting).  
  4. All coordinates valid.  
- **Behaviour**  Return `true` if the point lies inside or on the border; otherwise `false`.  
- **Output**  `200 OK` boolean.  
- **Error**  Invalid or open/self-intersecting polygon → `400 Bad Request`.  
- **Level**  Integration / System.

---

## 3. Quality Requirements (QR)

| ID | Description | Measurement | Level |
|:--|:--|:--|:--|
| QR-1 | Response time ≤ 30 s for any endpoint under normal load | End-to-end timing | System |
| QR-2 | Invalid inputs handled without uncaught exceptions | No 5xx responses or stack traces | Integration / System |
---

## 4. Reliability and Robustness (RR)

| ID | Description | Expected Behaviour | Level |
|:--|:--|:--|:--|
| RR-1 | Determinism | Same input → same output ( ± 1e-12 ) | Unit / Integration |
| RR-2 | Boundary consistency | isCloseTo strict `<` threshold; isInRegion inclusive border | Unit / Integration |

---

## 5. Security and Maintainability (SR / MR)

| ID | Category | Requirement | Level |
|:--|:--|:--|:--|
| SR-1 | Security | No hard-coded URLs or credentials in source code | Static analysis |
| MR-1 | Maintainability | Follow Java naming and layered structure (Controller / Service / Model) | Code review |

---

## 6. Requirement Level Mapping

| Requirement | Level |
|:--|:--|
| FR-1 | Unit / Integration |
| FR-2 | Unit / Integration |
| FR-3 | Unit / Integration |
| FR-4 | Integration / System |
| QR-1 | System |
| QR-2 | Integration / System |
| RR-1, RR-2 | Unit / Integration |
| SR-1, MR-1 | Static / Review |

---

## 7. Out of Scope
- Spherical distance calculation (not required by spec).  
- Self-intersecting or degenerate polygons (counted invalid → 400).  
- Detailed error objects or exception messages (basic 400 sufficient).  

---

## 8. Traceability
- This file serves as the **LO1 evidence document**, demonstrating requirement diversity and coverage across levels (Unit, Integration, System).  
- It is referenced by the portfolio’s LO1 section for analysis of testing strategy appropriateness.  
