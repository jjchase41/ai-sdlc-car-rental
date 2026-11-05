---
applyTo: "docs/test-scenario/**"
---

Use this folder only to store the test scenarios and related support files.
When generating a specific artifact, you must follow this standard.

The general filename rule is:
  - `[prd-title]/test-scenario-[feature].md` for the particular feature to be tested.
  - For supporting files, no specific rule

One PRD can contain multiple features.
You should create one test scenario document for each feature. Ensure you group related features (based on PRD) on a dedicated subfolder under `docs/test-scenario/[prd-title]`. 

Each test scenario must contain the following items (in the given sequence):

[Title: Test Scenario - Feature Name]

- Document Information
  + Author: The assignee's username
  + Date: blank
  + Version: blank

- Table of Contents
  Includes items up to the second-level header

- Test Scenario
  + Feature to test: brief description of this scenario (must refer to the PRD). For example: _test an addition operation feature for a calculator_.
  + Preconditions: List of preconditions, if any. For example: _calculator application is running_.

- Test Case 1
  + Description: brief description of this test case goal. For example: _testing addition with result less than 100_.
  + Steps: numerical step-by-step on how to execute this test case. For example: 
    1. _Input number 16_
    2. _Input number 37_
    3. _Click calculate_
  + Expected result: the expected result. For example: _the calculator must display 53_.

- Test Case X
  + Description: ....
  + Steps: ...
  + Expected result: ...
