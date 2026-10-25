# Lab sheet 2: Diagnosis from the pipeline log

Team **SNAR**, branch `chaos/tp2`, PR #8, run 40181265640.
Written from the pipeline log only, before looking at the diff.

Gates: install ✅, lint ✅, tests ❌, secret scan ❌, dependency audit ❌.

| Defect     | Gate that failed | Name in the log | One-line cause |
|------------|------------------|-----------------|----------------|
| Test 1     | tests | `tests/test_scoring.py::test_last_decaying_solve_is_one_step_above_the_floor` | **Assertion**: `points_for(SPEC, 8)` returns 100 instead of 150, so the last decaying solve (solve 8 of 8) already falls to the floor: off-by-one in the decay boundary of `points_for` in `scoring.py`. |
| Test 2     | tests | `tests/test_scoring.py::test_average_of_an_empty_competition_is_zero` | **Exception**: `ZeroDivisionError` at `scoring.py:122` (`sum(totals) / len(totals)`): `average_score([])` no longer handles an empty list. |
| Credential | secret scan | gitleaks: `leaks found: 2` (rule `aws-access-token`) | An AWS access key ID is hard-coded in `src/scoreboard/config.py` (line 50). Found twice: in the source and in its compiled `__pycache__/config.cpython-312.pyc`, created by the tests step. |
| Dependency | dependency audit | pip-audit: `pyyaml 5.3.1  PYSEC-2021-142  fix 5.4` | `requirements.txt` pins PyYAML 5.3.1, which has a known vulnerability (arbitrary code execution when loading untrusted YAML). |

Notes:
- The log only prints `leaks found: 2` because the pipeline runs gitleaks without `-v`. The rule name and file come from the same command run locally with `-v`
  (`gitleaks dir --redact -v .`), not from the diff.
- Lint passed: the defects are valid, well-formatted code. Only behaviour, secrets and dependencies are wrong, which is why we need the other gates.
