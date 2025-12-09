# LO1 Evidence — Requirements Document

## 1. Scope

This document defines the functional, measurable, and quality requirements for the ILP Drone Delivery Microservice implemented in CW1 and CW2.  
The service provides:

- Geometry utilities for drone navigation (CW1).
- Integration with the ILP backend for drones, service points, restricted areas, and availability records (CW2).
- Drone capability and availability queries.
- Delivery plan computation including safe pathfinding, cost calculation, and GeoJSON output (CW2).

The requirements here form the foundation for the later Learning Outcomes (LO2, LO3, LO4, LO5).

---

## 2. Terminology and Conventions

### 2.1 Coordinate
A coordinate is a JSON object:
```
{ "lng": number, "lat": number }
```
Constraints:
- Longitude ∈ [-180, 180]  
- Latitude ∈ [-90, 90]  
- Values must be finite, non-null, non-NaN.

### 2.2 STEP
A fixed movement step used for drone motion:
```
STEP = 0.00015 degrees
```

### 2.3 Direction Grid
Supported drone movement angles:
```
ANGLES = { 0, 22.5, 45, …, 337.5 }   // 16 directions
```
0° = East, 90° = North, 180° = West, 270° = South.

### 2.4 Region (Polygon)
JSON object:
```
{ "vertices": [ Coordinate, ..., Coordinate ] }
```
Requirements:
- At least 4 vertices (closed polygon).
- First and last vertex must be equal within tolerance.
- No null or invalid vertices.

### 2.5 MedDispatchRec
The structure for a medical dispatch request:
```
{
  "id": 123,
  "date": "YYYY-MM-DD",
  "time": "HH:MM:SS",
  "delivery": Coordinate,
  "requirements": {
    "capacity": 0.75,
    "cooling": false,
    "heating": true,
    "maxCost": 13.5
  }
}
```
Rules:
- `id`, `delivery`, and `capacity` are required.
- If `time` is provided, `date` must also be provided.
- `cooling` and `heating` cannot both be `true`.

### 2.6 ILP Endpoint
External backend base URL:
```
ILP_ENDPOINT = System.getenv("ILP_ENDPOINT") or default URL
```

---

## 3. Functional Requirements (FR)

### 3.1 General Endpoints

#### FR-G0: GET `/api/v1/`
Returns an HTML page describing available endpoints.

#### FR-G1: GET `/api/v1/uid`
Returns the student UID as plain text.

---

### 3.2 Geometry Endpoints (CW1)

#### FR-G2: POST `/api/v1/distanceTo`
Input:
```
{
  "position1": Coordinate,
  "position2": Coordinate
}
```
Output: JSON number representing Euclidean distance.  
Error: 400 for invalid coordinates.

#### FR-G3: POST `/api/v1/isCloseTo`
Returns true if the distance between coordinates is < STEP.

#### FR-G4: POST `/api/v1/nextPosition`
Moves one STEP along a valid 16-direction angle.

#### FR-G5: POST `/api/v1/isInRegion`
Validates polygon and returns whether the point lies inside or on the boundary.

---

### 3.3 Drone Query Endpoints (CW2)

#### FR-D1: GET `/api/v1/dronesWithCooling/{state}`
Returns list of drone IDs whose cooling flag matches `{state}`.

#### FR-D2: GET `/api/v1/droneDetails/{id}`
Returns the Drone object or 404 if not found.

#### FR-D3: GET `/api/v1/queryAsPath/{attribute}/{value}`
Single-attribute drone filtering.

#### FR-D4: POST `/api/v1/query`
Multi-condition filtering based on attribute/operator/value constraints.

#### FR-D5: POST `/api/v1/queryAvailableDrones`
Given a list of MedDispatchRec objects, returns drones capable of serving all deliveries based on:
- capacity,
- cooling/heating requirement,
- availability windows,
- conservative maxCost feasibility.

---

### 3.4 Delivery Planning (CW2)

#### FR-P1: POST `/api/v1/calcDeliveryPath`
Input: list of MedDispatchRec.  
Behaviour:
- Validates all dispatches.
- Fetches ILP backend data each request.
- Performs A* pathfinding with 16-direction grid and restricted-area avoidance.
- Groups deliveries by date/time.
- Selects feasible drones based on capability, availability, maxMoves, and maxCost.
- Builds flights and computes total cost and moves.

Output on success:
```
{
  "dronePaths": [...],
  "totalCost": number,
  "totalMoves": number
}
```

Output on failure/infeasible planning:
```
{
  "dronePaths": [],
  "totalCost": 0.0,
  "totalMoves": 0
}
```

#### FR-P2: POST `/api/v1/calcDeliveryPathAsGeoJson`
Same planning logic as FR-P1, output serialized as GeoJSON.

---

## 4. Measurable Requirements (MR)

### MR-1 Performance
Each endpoint must complete within 30 seconds in the marking environment.

### MR-2 Path Safety
No coordinate in any path may lie inside a restricted area.

### MR-3 Cost Correctness
Cost must follow:
```
flightCost = costInitial + costFinal + costPerMove * stepsUsed
```
Total cost = sum over flights.

### MR-4 Determinism
Same input and backend state must produce identical output.

### MR-5 Geometry Correctness
Geometry utilities must be unit-testable and achieve high line and branch coverage.

---

## 5. System Quality Requirements (QR)

### QR-1 Robustness
- Geometry endpoints return 400 on invalid input.
- Drone and planning endpoints never throw unhandled exceptions; semantic invalidity is expressed by empty results.

### QR-2 External Consistency
Must remain compatible with ILP backend schemas.

### QR-3 Configuration Flexibility
Uses ILP_ENDPOINT environment variable, falling back to the default URL.

### QR-4 Testability
Geometry, pathfinding, drone selection, and planning logic must be separated into pure helper classes to support unit/integration testing.

---

## 6. Assumptions

- ILP backend is reachable and provides valid JSON.
- Planar Euclidean geometry is sufficient for ILP.
- Time zone interpretation follows backend semantics.
- No caching; all ILP data is fetched per request.

---

## 7. Risks and Limitations

- Pathfinding may fail in heavily obstructed maps and return an empty plan.
- Conservative maxCost estimation may discard some feasible plans.
- Dependency on ILP backend availability.
- Euclidean geometry is an approximation; not suitable for large real-world distances.

---

## 8. Traceability

- Geometry requirements map to CW1 controllers and services.
- Drone query requirements map to drone-related controllers and ILP client integration.
- Delivery planning requirements map to DeliveryPlanner, DeliveryPlanHelper, QueryDroneHelper, and pathfinding utilities.

This document provides the requirements foundation for subsequent LO2–LO5 analysis.
