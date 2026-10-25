# Lab sheet 2: Continuous Integration (answers)

**Team members:**
- Amir Elcharif Mohammedi
- Koraichi Sondous Hiba
- Khelloufi Nour
- Benmecheri Ahmed Ryad

**Repository URL:** https://github.com/NourKH2245/DevOps-Lab

---

## Exercise 1: The pipeline

The pipeline is in `.github/workflows/ci.yml` (PR #6). It was green on the first run.

**(a)** `push` only checks code that is already on the branch. `pull_request` checks the change **before** it goes to `main`: it tests the result of the merge with `main`, it shows the result on the pull request, and with a required check it can block the merge. A broken change never reaches `main`.

**(b)** Lint is the cheapest gate (a few seconds, no need to run the code). Small mistakes like an unused import or bad formatting are found first, so we get feedback fast.

**(c)** With 3 seconds the order does not matter much. With 20 minutes of tests, a typo found by lint after 5 seconds saves 20 minutes of waiting (and 20 minutes of CI time) for each push. Cheap gates first = fast feedback.

---

## Exercise 2: Nobody can skip the pipeline

Branch protection on `main`: *Require status checks to pass*, with `verify` selected.

**(a)** `git commit --no-verify` (or `git commit -n`). The hook runs on our own machine, so everyone can skip it.

**(b)** Only someone who can bypass the branch protection: a repository admin/owner (the rule is not enforced for admins unless "Do not allow bypassing the above settings" is ticked). Everyone else must go through a pull request with a green `verify`.

---

## Exercise 3: Why the install step is important

We deleted `pip install -e .` on a temporary branch and opened PR #7 (closed, not merged, branch deleted). Only the `tests` gate failed; lint, secret scan and dependency audit passed.

**(a)**
```
E   ModuleNotFoundError: No module named 'scoreboard'
```
(from `ImportError while loading conftest 'tests/conftest.py'`, at `from scoreboard.app import create_app`, exit code 4.)

**(b)** The project uses a **src layout**: the package is in `src/scoreboard/`, not at the root of the repo. Python looks for modules in the current directory and in `site-packages`, and `src/` is in neither. So `import scoreboard` only works after the package is installed. `pip install -e .` installs it in editable mode (it adds `src/` to the import path), so the tests can import it. The src layout is on purpose: the tests run against the installed package, like a user would, and not against random files in the folder.

**(c)** `make install`. Its last line is `$(BIN)/pip install -q -e .`.

---

## Exercise 4: Diagnosis

Team **SNAR**, PR #8. The table is in `docs/TP2_diagnosis.md`, committed (`9a03709`) before any fix.

- Test 1: `test_last_decaying_solve_is_one_step_above_the_floor`, failed by **assertion** (`assert 100 == 150`).
- Test 2: `test_average_of_an_empty_competition_is_zero`, failed by **exception** (`ZeroDivisionError`).
- Credential: gitleaks, rule `aws-access-token` in `src/scoreboard/config.py`.
- Dependency: pip-audit, `pyyaml 5.3.1`, `PYSEC-2021-142`, fixed in 5.4.

---

## Exercise 5: Repairing the defects

The diff confirmed the diagnosis. One commit per defect, only application code changed (no test was edited):

| Defect | Fix | Commit |
|--------|-----|--------|
| Test 1 | `scoring.py`: `if n >= spec.decay_solves` back to `if n - 1 >= spec.decay_solves` (off-by-one) | `aeaa816` |
| Test 2 | `scoring.py`: put back the guard `if not totals: return 0.0` in `average_score` | `69a6321` |
| Credential | `config.py`: removed `backup_access_key_id = "AKIA..."` | `44c4dd7` |
| Dependency | `requirements.txt`: `pyyaml==5.3.1` back to `pyyaml==6.0.3` | `a43d793` |

After the fixes the pipeline is green on PR #8.

**(a)** **Direct.** `pyyaml` is listed by name in our own `requirements.txt` (and in `pyproject.toml`). We did not get it through another package.

**(b)** `grep -in yaml requirements.txt` shows the line `pyyaml==...` in our own manifest. To confirm: `pip show pyyaml` shows who requires it (`Required-by:`), and `pipdeptree -r -p pyyaml` prints the reverse dependency tree. A transitive dependency would not be in `requirements.txt` and would only appear under another package in that tree.

Note: the key is removed from the code, but it is still in the history of the branch (commit `b478920`). In a real project the key must be revoked at the provider first (see lab sheet 1, Exercise 4).
