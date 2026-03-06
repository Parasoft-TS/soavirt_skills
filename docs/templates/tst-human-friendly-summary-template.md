# TST Human-Friendly Summary Template

Use this template when explaining a `.tst` file to users.

## 0) Output Profile
- **Audience:** non-technical / mixed / technical
- **Depth:** brief / standard / deep
- **Voice:** operator handoff (default) / explanatory walkthrough

## 0.1) Default Detail Heuristic
- Determine size from YAML structure counts: number of suites and tests.
- Default behavior:
	- **Small TST** (<= 15 tests): include suites and tests in Section C.
	- **Medium TST** (16-50 tests): include suites, and include tests only for top-level or first 3 per suite.
	- **Large TST** (> 50 tests): include suites only by default; omit test lists unless user asks.
- Always allow explicit user override (for example: "include all tests" or "suite-only view").
- Heuristic bucket is internal guidance and should not be shown in the default user summary.

## A) Executive Summary
- **What this TST does (1-3 sentences):**
- **Application/service interface under test:**
- **Active environment used for testing:**
- **Primary business flow(s) covered:**
- **Flow model:** single-flow root suite / multiple suite-level flows / endpoint unit-test style suite

## B) Tested Environment (Active)
- **Active environment name:**
- **Active environment variables (table):**

| Variable | Value |
|---|---|
|  |  |

When Environment Reference is used:
- Download referenced `.env` or `.envs` file from server.
- Resolve active environment to a concrete environment node from that file.
- Populate the variable table from the resolved active environment.
- Include note of source file and whether it is `.env` (single env) or `.envs` (multiple envs).

## C) Structure at a Glance
- **File id:**
- **Root suite name:**
- **Active environment:**
- **Suite/test hierarchy (default):**
	- root suite
	- setup tests (always listed first within root suite when present)
	- child suites
	- tests contained in each suite
- **Optional detail (only for `deep`):** include data sources and chained tools under each test

## D) Business Flow Summary
- **`<suite name>`:**
	- step 1
	- step 2
	- step 3
- **`<suite name>`:**
	- step 1
	- step 2

Guidance:
- Name each flow by the suite that encapsulates it.
- If the root suite alone is the flow, explicitly state that.
