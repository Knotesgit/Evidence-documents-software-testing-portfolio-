# LO3 Evaluation of Testing Results



This document records the application of the evaluation criteria (oracles) defined

for LO3 and summarises the results of those evaluations. The purpose is to document

what conclusions can be drawn from the executed tests, and what limitations remain,

rather than to describe test design or implementation.



---



## FR-U2 — STEP-based movement correctness



### Evaluation criterion (oracle)

A numeric oracle asserting that the displacement magnitude between the input and

output coordinates equals `STEP` within a defined tolerance.



### Application

The oracle is applied to unit tests executing the `nextPosition` primitive across a

representative set of movement directions.



### Evaluation result

No violations of the numeric constraint were observed in the executed tests.



### Limitations

The evaluation does not explore extreme floating-point edge cases beyond the tested

direction set.



### Evidence

- `LO3 Evidence/FR-U2/FR-U2\_code.png`



---



## FR-I1 — Backend queried exactly once per request



### Evaluation criterion (oracle)

An interaction oracle verifying that each backend query method is invoked exactly

once per request, and that no additional backend interactions occur.



### Application

The oracle is applied in integration tests using a mocked backend client to observe

call behaviour during request handling.



### Evaluation result

All executed tests satisfied the exact-once interaction constraint.



### Limitations

The evaluation does not assess the semantic correctness of backend responses, which

is outside the scope of the requirement.



### Evidence

- `LO3 Evidence/FR-I1/FR-I1\_code.png`



---



## FR-S2 — Path must not enter restricted areas



### Evaluation criterion (oracle)

A geometric safety oracle asserting that:

- no path endpoint lies inside or on the boundary of any restricted area, and

- no path segment intersects or touches any restricted-area boundary.



Boundary contact is treated as a violation.



\### Application

The oracle is applied to every returned path segment in system-level scenario tests.



### Evaluation result

No restricted-area violations were detected in any executed scenario.



### Limitations

Scenario-based system testing cannot exhaustively cover all possible geometric

configurations.



### Evidence

- `LO3 Evidence/FR-S2/FR-S2\_oracle.png`

- `LO3 Evidence/FR-S2/Behaviour table.md`



---



## FR-S3 — Constraint-based drone selection



### Evaluation criterion (oracle)

A constraint-satisfaction oracle asserting that the selected drone satisfies all

stated capability constraints. Where multiple drones are eligible, any valid

candidate is accepted.



### Application

The oracle is applied to system tests constructed with controlled backend datasets

to isolate individual and combined constraint interactions.



### Evaluation result

In all executed tests, the selected drone satisfied the required constraints.



### Limitations

The evaluation does not assess optimisation or preference policies not specified by

the requirement.



### Evidence

- `LO3 Evidence/FR-S3/FR-S3-4\_code.png`

- `LO3 Evidence/FR-S3/FR-S3-5\_code.png`

- `LO3 Evidence/FR-S3/FR-S3-6\_code.png`



---



## MR-1 — End-to-end performance constraint



### Evaluation criterion (oracle)

A statistical performance oracle asserting that summary execution-time statistics

(median and 95th percentile) remain below the specified threshold.



### Application

The oracle is applied to repeated executions under representative and controlled

stress workloads.



### Evaluation result

Observed summary statistics satisfied the performance threshold.



### Limitations

Results are environment-dependent and may not generalise to all deployment

conditions.



### Evidence
- `LO3 Evidence/MR-1/MR-1\_code.png`



---





