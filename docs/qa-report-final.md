# QA Report: Final Sprint, DevSecOps and IT Project Management

| | |
|---|---|
| **Section** | Shared VM (team ___) / Local VM |
| **Test author(s)** | ______________________ (Shared VM: QA) |
| **Tester** | ______________________ (Local VM: you are author and tester) |
| **Commit tested** | `git rev-parse --short HEAD` = ________ |
| **Dates** | Cases written: ________ Cases run: ________ |

## How to use this report

1. **Write the test cases first,** in Week 11, before the work they test exists. A test written after the fact tends to test what was built instead of what was asked for.
2. **Write the expected result before you run anything.** If the actual result surprises you, that is a finding, not a typo to fix.
3. **Paste real output or point to an evidence file.** "Works" is not a result. A command and its output is.
4. **Cover every success criterion with at least one case,** and include at least one negative test (something that must fail or be blocked) and the rebuild proof of life.
5. **Re-run every case in Week 13** on the rebuilt VM and update the status. A case that passed only on the old VM is a rebuild gap.

If your team replaced a criterion through [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md), test the replacement instead, and link the approved design brief.

---

## Test Cases

Copy the block for each new case. Six slots are a starting point, not a limit.

<details>
<summary>Example of a filled-in case (delete when you start)</summary>

**TC-00: Hadolint gate catches a bad Dockerfile**

| Field | Value |
|---|---|
| Criterion | 2 |
| Steps | Open a pull request that adds `USER root` and an unpinned base image to the Dockerfile; run `gh pr checks <number>` |
| Expected (written before running) | The Dockerfile lint check is red; the PR cannot merge if the check is required; no other check is affected |
| Actual | See `capstone/devsecops-pm/evidence/<file>`; summary: ... |
| Status | Pass |
| Run on | Original VM, YYYY-MM-DD; rebuilt VM, YYYY-MM-DD |

</details>

### TC-01: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-02: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-03: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-04: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-05: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-06: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

---

## Success Criteria Verification

Evidence paths are relative to `capstone/devsecops-pm/` and come from [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md). Change a path if your file is named differently.

| Criterion | Evidence | Test case(s) | Met? |
|---|---|---|---|
| 1. Four new checks (Dockerfile lint, shell lint, security scan, k6 threshold) on five real pull requests | `evidence/gate-runs.txt` | | [ ] Yes [ ] No |
| 2. Three bad pull requests red, one good pull request merged, required checks intact before and after | `evidence/gate-runs.txt`, `evidence/branch-protection.txt` | | [ ] Yes [ ] No |
| 3. Pre-commit rejects a bad commit and passes on a clean branch | `evidence/pre-commit-reject.txt`, `evidence/pre-commit-clean.txt` | | [ ] Yes [ ] No |
| 4. Two annotated tags, CHANGELOG entries, rollback timed under 5 minutes | `CHANGELOG.md`, `evidence/rollback-rehearsal.txt` | | [ ] Yes [ ] No |
| 5. Roadmap, RACI (one Accountable per row), risk register, status report of 400 words or less | `ROADMAP.md`, `RACI.md`, `RISK-REGISTER.md`, `STATUS-REPORT.md` | | [ ] Yes [ ] No |

---

## Rebuild Verification (Week 13)

| Check | Result |
|---|---|
| Rebuild date and VM (new VirtualBox VM, or Shared VM queue slot) | |
| Started from commit | |
| Proof of life, Shared VM: `pre-commit run --all-files` exits 0; both annotated tags exist; the rollback command from your RUNBOOK works | [ ] Pass [ ] Fail |
| Proof of life, Local VM: Same as the Shared VM, and `gh api repos/{owner}/{repo}/rulesets` still lists `protect-master` | [ ] Pass [ ] Fail |
| Every app change came back (schema, routes, services, images) | [ ] Yes [ ] No |
| Gaps logged in `capstone/devsecops-pm/REBUILD-GAPS.md` and fixed by pull request | ___ gaps found, ___ fixed |
| All test cases above re-run on the rebuilt VM | [ ] Yes [ ] No |

---

## Deliverables Verification

- [ ] Implementation committed under `capstone/devsecops-pm/` (merged by pull request)
- [ ] `capstone/devsecops-pm/RUNBOOK.md`: operate, check, and recover
- [ ] `capstone/devsecops-pm/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages)
- [ ] `capstone/devsecops-pm/ARCHITECTURE.md` with a diagram and a "What we changed and why" section
- [ ] "How to run this" section in the top-level README
- [ ] `capstone/devsecops-pm/REBUILD-GAPS.md` from the Week 13 rebuild
- [ ] This QA report, with every case run and signed
- [ ] Shared VM: `docs/final-sprint-retrospective.md`. Local VM: the Individual Reflection section
- [ ] `capstone/devsecops-pm/DESIGN-BRIEF.md`, only if a criterion was replaced, with the approval noted
- [ ] Discussion answers in the Google Doc
- [ ] Final presentation and Week 14 rebuild presentation prepared (live on the Shared VM, recorded on the Local VM)
- [ ] Track-specific: The pull-request workflow(s) with the new checks
- [ ] Track-specific: `.pre-commit-config.yaml`
- [ ] Track-specific: `CHANGELOG.md` and both pushed tags
- [ ] Track-specific: `ROADMAP.md`, `RACI.md`, `RISK-REGISTER.md`, `STATUS-REPORT.md`

---

## Rework Required

| # | Failed case or criterion | What is wrong | Fix and owner | Re-test date | Re-test result |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

---

## Sign-Off

**Overall status:** [ ] All criteria met [ ] Met with documented exceptions [ ] Rework required

**Exceptions** (criteria not met, and why):

```

```

| Role | Name | Date |
|---|---|---|
| Test author | | |
| Tester (Local VM: same person) | | |

**Notes:** observations about technical quality and, on the Shared VM, team coordination.
