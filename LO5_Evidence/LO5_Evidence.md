# LO5 Evidence — Test process improvement and automation

## Scope
This document records evidence for Learning Outcome 5, focusing on review/inspection
activities and the integration of automated testing into a continuous integration (CI)
process to address limitations identified in earlier testing.

## 1. Review / inspection evidence

### Purpose
The purpose of this review is to identify limitations of the implemented testing process
that are not exposed by executed test results, and to assess risks arising from test
architecture, observability, and implicit assumptions.

---

### Review approach
A lightweight inspection of the testing process was performed, focusing on:
- trustworthiness of system-level test oracles,
- visibility of internal decision behaviour,
- reliance on assumptions not independently validated by tests.

---

### Findings

**Finding 1 — Oracle dependence**
System-level test outcomes depend on checking logic that is shared between the system
under test and the test oracle. This introduces a risk that defects in the shared logic
will be consistently masked, leading to false confidence even when all system-level
tests pass. This limitation reflects a test architecture issue rather than a lack of
test cases.

---

**Finding 2 — Limited observability of internal decision state**
System-level tests primarily observe final outcomes and do not expose intermediate
decision states. As a result, certain failure modes may remain undetected if they still
produce acceptable end results. These masked failures are difficult to reveal through
black-box or scenario-based testing alone.

---

**Finding 3 — Implicit preconditions not independently validated**
Several correctness properties rely on assumed preconditions, such as well-formed inputs
or consistent internal representations. These assumptions are not explicitly validated
by tests, making resulting failures harder to diagnose and potentially misleading when
they occur.

---

### Implications
These findings indicate that the main limitations of the current testing process arise
from oracle trust, observability boundaries, and unverified assumptions. Review and
inspection therefore provide complementary assurance by exposing risks that are
structurally invisible to executed tests.

---

## 2. CI pipeline design

### Purpose
The CI pipeline is designed to integrate automated testing into the development workflow,
ensuring that test execution is repeatable, non-interactive, and consistently triggered
by code changes.

---

### Pipeline structure
The pipeline follows a minimal and deterministic structure:
1. Source checkout
2. Environment setup (JDK and build tooling)
3. Automated test execution
4. Pass/fail reporting

The pipeline is triggered on each push and pull request, ensuring that all changes are
subject to the same automated checks before integration.

---

### Design rationale
The pipeline deliberately prioritises reliability and clarity over complexity. By
limiting the pipeline to essential build and test stages, failures can be directly
attributed to code or test regressions rather than pipeline-specific behaviour. This
supports efficient regression detection and reduces the risk of false alarms caused by
overly complex automation.

---

## 3. Automated checks integrated into the pipeline

### Scope of automation
The CI pipeline integrates automated execution of the existing test suite as a
regression-checking mechanism. On each pipeline run, all implemented unit,
integration, and system tests are executed automatically, providing immediate
feedback on behavioural regressions introduced by code changes.

---

### Rationale for selected checks
Automated test execution is prioritised because it is repeatable, deterministic,
and directly aligned with the testing objectives defined in earlier learning
outcomes. Integrating these tests into CI ensures that previously validated
behaviour is continuously re-checked without manual intervention.

Other quality activities, such as exploratory testing or review-based inspection,
are not automated, as their effectiveness depends on human judgement rather than
mechanical repetition. These activities are therefore treated as complementary
techniques rather than candidates for automation.

---

### Limitations of automation
While CI automation improves consistency and regression detection, it does not
address the structural limitations identified through review, such as oracle
dependence or limited observability of internal decision states. Automated checks
therefore strengthen discipline and repeatability but do not eliminate the need
for inspection and higher-level reasoning.

---

## 4. Evidence that the pipeline operates as intended

A CI run triggered by a repository push demonstrates that automated test execution is
successfully integrated into the development workflow. The pipeline executes the
configured test suite without manual intervention and produces a clear pass/fail
outcome, providing immediate feedback on regressions introduced by code changes.

![CI execution triggered by repository push showing automated test run](CI_test_execution.png)

The recorded CI execution confirms that testing is consistently and repeatably applied
to all changes, supporting disciplined regression checking rather than ad hoc or
developer-dependent test execution.
