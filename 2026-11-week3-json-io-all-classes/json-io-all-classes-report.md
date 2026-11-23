# Do LLM-generated tests survive longer than developer tests but cover less code? json-io, all classes, 300 commits, four regeneration policies

## Key findings

1. **The direction seen in the earlier runs of this design holds on json-io: under one-time generation the LLM sets keep a larger share of their tests alive and execute fewer lines.** At the last commit 42.8% of Dev is alive against 65.3% of Ouédraogo and 69.1% of KTester. Dev executes 53.0% of the changed lines against 26.2% and 35.1%, and 83.1% of the project's lines at *t* against 59.0% and 66.0%, although the KTester set has twice as many methods as Dev (Tables 1 and 3).
2. **On json-io the survival gap comes from one commit, a defect in the production code that the project repaired 31 minutes later.** Up to #102 (cc6bf54b) the three shares are within four points (Dev 70.0%, Ouédraogo 69.2%, KTester 72.8%). At #118 (22dde9cf) Dev loses 276 methods, KTester 2 and Ouédraogo 0. The 276 methods pass again from #120 (3266953f) on, but a method that fails once is not run again, so they stay counted as lost.
3. **Regeneration raises survival for every group and leaves the coverage gap in place.** With regeneration at the two releases, 98.2%, 89.6% and 93.9% of the last sets are alive at the last commit. The changed lines executed rise by 300 for Dev, 106 for Ouédraogo and 115 for KTester out of 4,675. At #299 (7ca09df5) the re-established Dev set executes 82.0% of the project's lines and the regenerated LLM sets 41.9% and 49.9%.

## Question

A test set generated at a starting commit *t* breaks as the code moves on. On json-io, with tests generated for every class of the project: (1) how do developer tests and LLM-generated tests compare in survival and in the code they execute, and is the direction the same as on the projects measured before; (2) what do the regeneration policies change?

## Setup

- **Project.** json-io (`jdereg/json-io`), *t* = 4.30.0 (a6f9b6bb, 2024-10-13), 300 first-parent commits to 2025-07-14. Commit #*n* is the *n*-th first-parent commit after *t*. 120 commits change neither production code nor a build file and are skipped. Of the other 180, 150 build and are evaluated; 30 do not build and carry the previous result. The last evaluated commit is #299 (7ca09df5).
- **Classes given to the generators.** All 60 production classes that have a method to test (367 methods).
- **Groups** (test methods that passed one run at *t*; the same model, gpt-oss-120b, for both generators):
  - Dev, 1,015: all developer tests of the project.
  - Ouédraogo, 1,114: Ouédraogo et al., EMSE 2026, the five prompt templates, one generation round, with three adjustments (signatures of the project types the class refers to are added to the prompt, the model is told to use only the API shown and no mocks, missing imports are inserted after generation).
  - KTester, 2,105: Li et al., ICSE 2026, the released tool with three repair rounds.
- **Regeneration policies.**

| Policy | When | What |
|---|---|---|
| One-time generation | never | the set generated at *t* |
| Regenerate at each release | 4.51.0, first buildable commit #102 (cc6bf54b); 4.55.0, #231 (32b011eb) | the group is discarded and generated again; Dev is re-established from the developer tests of that commit |
| Reset when 50% broken | the first commit where half of the group has broken: Dev at #118 (22dde9cf); never reached by Ouédraogo and KTester | the same |
| Fill when 50% broken | the same commit | new tests are added for the broken ones; Dev stays the set of *t*, so no group changes in this run |

- **Measurements.** Survival: a method survives at a commit if it passed at every evaluated commit up to it. Changed-line coverage: the share of the executable lines changed at each commit that the surviving methods execute (4,675 lines over the range). Whole-project line coverage of the surviving methods at *t* (4,236 executable lines) and at #299 (6,395).

## Results

The per-commit series (survival and changed-line coverage under each policy) are in the shared spreadsheet: https://docs.google.com/spreadsheets/d/1dKA_jz-KgP-7aLH_s2xyuXT1RgrzxqaSAqjXlXO6FrY/edit. There, a survival row is shown when a group lost three or more methods, and a coverage row when the commit changed ten or more executable lines or a group lost methods; the remaining commits are aggregated.

**Table 1. Survival and coverage by regeneration policy** (alive / size of the last segment; lines executed and share). At *t* the three sets execute 3,518 (83.1%), 2,501 (59.0%) and 2,797 (66.0%) of the 4,236 executable lines. Under the two regeneration policies Dev is the developer tests of the regeneration commit (1,015 → 1,213 → 1,384 at the releases; 1,015 → 815 at #118), so its share alive is not comparable with the one-time-generation row.

| Policy | Group | Alive at the last commit | Changed lines executed (of 4,675) | Project lines executed at #299 (of 6,395) |
|---|---|---:|---:|---:|
| One-time generation | Dev | 434/1,015 (42.8%) | 2,478 (53.0%) | 3,562 (55.7%) |
| | Ouédraogo | 727/1,114 (65.3%) | 1,225 (26.2%) | 2,227 (34.8%) |
| | KTester | 1,454/2,105 (69.1%) | 1,641 (35.1%) | 2,930 (45.8%) |
| Regenerate at each release | Dev | 1,359/1,384 (98.2%) | 2,778 (59.4%) | 5,242 (82.0%) |
| | Ouédraogo | 1,037/1,158 (89.6%) | 1,331 (28.5%) | 2,681 (41.9%) |
| | KTester | 1,854/1,975 (93.9%) | 1,756 (37.6%) | 3,188 (49.9%) |
| Reset when 50% broken | Dev | 802/815 (98.4%) | 2,622 (56.1%) | 4,275 (66.8%) |
| | Ouédraogo, KTester | not triggered | same as one-time generation | same as one-time generation |

**Table 2. How the methods were lost, one-time generation** (first failure of each lost method)

| Group | Lost | Test stopped compiling | Test failed when run | Lost at #118 (22dde9cf) |
|---|---:|---:|---:|---:|
| Dev | 581 | 247 | 334 | 276 |
| Ouédraogo | 387 | 333 | 54 | 0 |
| KTester | 651 | 486 | 165 | 2 |

**Table 3. The same two measurements in the runs of this design, one-time generation** (share alive at the last commit; share of the changed lines executed). In the five runs with scope classes the tests are generated for one to seven classes per project and Dev is the developer tests that execute those classes. The ranges and totals differ between rows, so the table is read row by row.

| Run | Alive: Dev | Alive: Ouédraogo | Alive: KTester | Changed lines: Dev | Changed lines: Ouédraogo | Changed lines: KTester |
|---|---:|---:|---:|---:|---:|---:|
| ojAlgo, scope classes | 0.8% | 0.0% | 91.9% | 50.7% | 0.0% | 23.3% |
| smallrye-config, scope classes | 19.6% | 0.0% | 10.3% | 74.9% | 1.4% | 1.8% |
| jinjava, scope classes | 85.3% | 61.8% | 100.0% | 96.4% | 0.0% | 0.0% |
| java-util, scope classes | 52.3% | 64.3% | 78.9% | 58.2% | 29.1% | 38.1% |
| logback, scope classes | 22.2% | 27.8% | 91.2% | 72.4% | 34.5% | 24.1% |
| java-util, all classes | 50.5% | 62.1% | 69.2% | 58.0% | 41.3% | 34.5% |
| json-io, all classes | 42.8% | 65.3% | 69.1% | 53.0% | 26.2% | 35.1% |

## Observations

1. **Across the seven runs the coverage direction has no exception and the survival direction has some.** Dev executes a larger share of the changed lines than both LLM groups in all seven rows of Table 3. KTester has a larger share alive than Dev in six of seven (not on smallrye-config) and Ouédraogo in four of seven (both java-util runs, logback, json-io). The two runs with all classes give close values: on java-util 50.5%, 62.1% and 69.2% alive and 58.0%, 41.3% and 34.5% of the changed lines; on json-io 42.8%, 65.3% and 69.1% and 53.0%, 26.2% and 35.1%.
2. **The LLM tests are lost mostly when the code they call changes; the developer tests on json-io are lost mostly at one defect.** 333 of the 387 lost Ouédraogo methods and 486 of the 651 lost KTester methods stopped compiling. 334 of the 581 lost Dev methods failed when run, 276 of them at #118 (22dde9cf), a change to the parser merged as upstream pull request #316. Run again outside the experiment, the 276 pass at #117 (d52b8185), all fail at #118, 189 fail at #119 (9c5cef3f), and all pass at #120 (3266953f), where upstream pull request #318 (titled as fixing the root cause of 174 test errors) was merged, and at each of the 31 later commits checked up to #231 (32b011eb). The same commit takes 399 of the 1,212 methods of the Dev set re-established at #102 (cc6bf54b) and is where Dev reaches the 50% threshold.
3. **Regeneration does not close the coverage gap: the regenerated LLM sets execute only a few hundred more lines than the sets generated at t, although the project has grown by 2,159 executable lines.** The executable lines grow from 4,236 at *t* to 6,395 at #299 (7ca09df5). The sets generated at #231 (32b011eb) execute 2,681 (Ouédraogo) and 3,188 (KTester) lines at #299, against 2,501 and 2,797 at *t*; the Dev set re-established at #231 executes 5,242 against 3,518. The gap between Dev and KTester in whole-project coverage is 17 points at *t* and 32 points at #299 when both sets are regenerated.

## Limitations

- A method that fails once is not run again, so a method lost at a defect that is later repaired stays lost. Run again at #299 (7ca09df5), 290 of the 581 lost Dev methods pass (276 of them lost at #118), 44 of the 651 lost KTester methods and 1 of the 387 lost Ouédraogo methods. This was checked at one commit and for one-time generation only.
- 30 commits do not build and have no coverage (#29 to #38, #72, #74 to #77, #90 to #95, #111, #136, #182, #183, #241, #248 to #251). json-io depends on java-util of the same maintainer and the required version changes 18 times in the range.
- One project, one model and one generation run per generation point. The 50% policies were exercised only by Dev and only at the defect commit, where the developer tests of that commit that pass number 815 (1,213 at #102, 1,384 at #231).

## Repository

Report: https://github.com/Hamjoon/test-survival-reports, `2026-11-week3-json-io-all-classes/json-io-all-classes-report.md` (this file). Experiment branch: https://github.com/Hamjoon/json-io/tree/experiment/2026-11-week3-json-io-all-classes, with the one-time-generation run in `experiments/commit-survival/` and the policy run in `experiments/regen-policies/`.

- Table 1: `experiments/regen-policies/results/policy/summary.csv`; whole-project coverage in `experiments/regen-policies/results/policy/<policy>/coverage-total.md` and `experiments/commit-survival/results/coverage-total.md`; segment sizes in `experiments/regen-policies/results/policy/release-reset/table1-survival-by-event.md` and `half-reset/table1-survival-by-event.md`.
- Table 2 and losses per commit: `experiments/commit-survival/results/first-failures.csv`, `table2-breaking-commits.md`, `table1-survival-by-event.md`.
- Table 3: Tables 1 and 2 of `2026-11-week1-regen-policies/regen-policies-report.md` and of `2026-11-week2-java-util-all-classes/java-util-all-classes-report.md` in the report repository; the json-io row from Table 1 of this report.
- Reruns of the lost methods (observation 2, first limitation): `experiments/commit-survival/results/recheck/` (`dev118-recheck.csv`, `dev118-window.csv`, `dead-recheck.csv`; the method is in its `README.md`).
- Classes given to the generators: `experiments/commit-survival/population/scope.md`. Regeneration commits: `experiments/regen-policies/results/policy/plan.md`. Methods left out at *t*: `experiments/commit-survival/results/control-excluded.csv`. Commits that do not build: `experiments/commit-survival/results/consistency.md`.
