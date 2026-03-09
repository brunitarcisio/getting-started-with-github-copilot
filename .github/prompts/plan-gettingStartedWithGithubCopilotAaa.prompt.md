## Plan: Backend FastAPI Test Suite

Add a dedicated pytest-based backend test suite under a separate tests directory, focused on endpoint behavior and regression safety for signup/unregister logic. The approach uses FastAPI TestClient plus fixtures that reset in-memory state between tests so tests stay deterministic without changing production behavior, and structures every test with the AAA pattern (Arrange, Act, Assert) for readability and consistency.

**Steps**
1. Phase 1 - Test foundation
2. Add `/workspaces/getting-started-with-github-copilot/tests/` with shared fixtures in `conftest.py` for `TestClient` and activity-state reset (*blocks all test files*).
3. Update `/workspaces/getting-started-with-github-copilot/requirements.txt` to include pytest runtime dependencies (`pytest`, and optionally `pytest-asyncio` if async tests are introduced later) so local and CI runs are reproducible.
4. Phase 2 - Endpoint coverage
5. Create `test_root.py` for redirect behavior of `GET /` (*parallel with step 6 and step 7*).
6. Create `test_activities.py` for `GET /activities` response status and schema expectations (*parallel with step 5 and step 7*).
7. Create `test_signup.py` for `POST /activities/{activity_name}/signup` covering happy path, duplicate signup, unknown activity, and missing email validation (*parallel with step 5 and step 6*).
8. Create `test_unregister.py` for `DELETE /activities/{activity_name}/signup` covering happy path, unknown activity, non-registered email, and missing email validation (*depends on step 2*).
9. Add an explicit AAA layout rule in each test module: arrange fixtures and inputs first, perform exactly one endpoint call in act, then assert status and payload/state mutations in assert (*depends on steps 5-8*).
10. Add concise Arrange/Act/Assert comments only where the phase boundary is not obvious, to keep tests readable without noise (*parallel with step 9*).
11. Phase 3 - Stability and polish
12. Add assertions that validate data mutation effects by re-fetching `/activities` after signup/unregister operations (*depends on step 7 and step 8*).
13. Ensure tests are independent by resetting activity data in fixture setup/teardown and avoiding ordering assumptions (*depends on step 2*).
14. Run full suite with pytest and fix flaky assertions or fixture leakage before merge (*depends on steps 5-13*).

**Relevant files**
- `/workspaces/getting-started-with-github-copilot/src/app.py` — source of API behavior under test (`root`, `get_activities`, `signup_for_activity`, `unregister_from_activity`).
- `/workspaces/getting-started-with-github-copilot/pytest.ini` — existing pytest config (`pythonpath = .`) that should allow importing `src.app`.
- `/workspaces/getting-started-with-github-copilot/requirements.txt` — add test dependencies.
- `/workspaces/getting-started-with-github-copilot/tests/conftest.py` — shared fixtures for `TestClient` and activity reset.
- `/workspaces/getting-started-with-github-copilot/tests/test_root.py` — redirect test(s).
- `/workspaces/getting-started-with-github-copilot/tests/test_activities.py` — list/read endpoint tests.
- `/workspaces/getting-started-with-github-copilot/tests/test_signup.py` — registration endpoint tests.
- `/workspaces/getting-started-with-github-copilot/tests/test_unregister.py` — unregister endpoint tests.

**Verification**
1. Install deps: `pip install -r requirements.txt`.
2. Execute suite: `pytest -q` from repository root.
3. Spot-check targeted files: `pytest -q tests/test_signup.py tests/test_unregister.py`.
4. Confirm isolation: run suite twice consecutively and verify identical pass/fail results.

**Decisions**
- Included scope: backend API tests only, stored in a separate top-level tests directory.
- Excluded scope: frontend/UI tests and backend endpoint redesign.
- Assumption: keep current query-parameter API style for email; tests should validate current contract.
- Assumption: use fixture-based in-memory reset instead of refactoring app architecture right now.

**Further Considerations**
1. Capacity rule: there is no max-participant enforcement yet; add a TODO/skipped test to document expected future behavior.
2. If async endpoints are added later, expand fixtures to support async client patterns.
