# Do last week's java-util results hold when tests are generated for every class instead of four scope classes?

## Question

Last week's java-util run generated tests for four scope classes only. Does the picture change when the generators are given every class of the project, and how do the two runs compare on the same lines?

## Setup

- **Same as last week:** java-util from *t* = 2.18.0 (`a85c604`), 300 first-parent commits to 2025-06-15; survival and changed-line coverage measured at every commit; the same regeneration policies. The release policy regenerates at #125 (3.0.2) and #213 (3.3.2). The two 50% policies did not trigger in either week.
- **Changed:** tests are generated for all 102 classes of the project that have a method to test (1,118 methods; deprecated classes excluded) instead of `CompactMap`, `DeepEquals`, `ClassUtilities` and `ReflectionUtils` (58 methods). Dev is now all developer tests of the project, not only those that execute the scope classes. The run used the lab server.
- **Groups** (test methods that passed one run at *t*): Dev 1,185, KTester 2,688, Ouédraogo 3,403 (last week 346, 171, 375).

## Results

**Table 1. Tests alive at the last commit** (alive / size of the last segment)

| Policy | Group | Week 1 (four scope classes) | Week 2 (all classes) |
|---|---|---:|---:|
| No regeneration | Dev | 181/346 (52.3%) | 598/1,185 (50.5%) |
| | KTester | 135/171 (78.9%) | 1,860/2,688 (69.2%) |
| | Ouédraogo | 241/375 (64.3%) | 2,113/3,403 (62.1%) |
| Regenerate at each release | Dev | 516/554 (93.1%) | 1,616/1,678 (96.3%) |
| | KTester | 256/265 (96.6%) | 2,426/2,495 (97.2%) |
| | Ouédraogo | 319/322 (99.1%) | 2,729/3,007 (90.8%) |

**Table 2. Changed lines executed, by policy** (the two weeks have different totals, so the columns are not on the same lines)

| Policy | Group | Week 1 (four scope classes, 6,565 lines) | Week 2 (all classes, 11,572 lines) |
|---|---|---:|---:|
| No regeneration | Dev | 3,824 (58.2%) | 6,715 (58.0%) |
| | KTester | 2,503 (38.1%) | 3,993 (34.5%) |
| | Ouédraogo | 1,910 (29.1%) | 4,785 (41.3%) |
| Regenerate at each release | Dev | 3,906 (59.5%) | 7,020 (60.7%) |
| | KTester | 2,555 (38.9%) | 4,117 (35.6%) |
| | Ouédraogo | 1,850 (28.2%) | 4,757 (41.1%) |

**Table 3. Changed lines executed, no regeneration, split by class set** (this week's run restricted to last week's four classes, so both runs are read on the same 6,565 lines)

| Group | Week 1, four classes (6,565) | Week 2, same four classes (6,565) | Week 2, other classes (5,007) | Week 2, all classes (11,572) |
|---|---:|---:|---:|---:|
| Dev | 3,824 (58.2%) | 3,824 (58.2%) | 2,891 (57.7%) | 6,715 (58.0%) |
| KTester | 2,503 (38.1%) | 1,331 (20.3%) | 2,662 (53.2%) | 3,993 (34.5%) |
| Ouédraogo | 1,910 (29.1%) | 2,033 (31.0%) | 2,752 (55.0%) | 4,785 (41.3%) |

## Observations

1. **Survival looks the same at eight times the size, except for KTester, which loses a fifth of its set at one commit.** The groups grow from 892 to 7,276 methods in total, and without regeneration the share alive at the last commit stays close to last week's for Dev (52.3% → 50.5%) and Ouédraogo (64.3% → 62.1%) and falls for KTester (78.9% → 69.2%). The project deletes five classes during the 300 commits (`SealableList`, `SealableMap`, `SealableNavigableMap`, `SealableNavigableSet`, `SealableSet`), and at #123 every test of them stops compiling: 536 of the 828 KTester methods lost over the 300 commits (Ouédraogo 398 of 1,290, Dev 78 of 587). On the four scope classes KTester loses 29 of 130 methods, against 36 of 171 last week. Sets regenerated at each release end above 90% in both weeks, and in both weeks regeneration adds few changed lines (Table 2: at most 305 for Dev, 124 for KTester, and slightly fewer for Ouédraogo).
2. **The four scope classes of last week are where the LLM tests are weakest.** On the other classes the three groups execute similar shares of the changed lines (Dev 57.7%, KTester 53.2%, Ouédraogo 55.0%); on the four scope classes Dev executes 58.2% against 20.3% and 31.0%. The rise of Ouédraogo from 29.1% to 41.3% overall therefore comes from the added classes: on the same four classes it moves from 1,910 to 2,033 lines.
3. **KTester falls on the four classes (2,503 → 1,331 lines) because none of its `DeepEquals` tests compiled this time.** KTester writes one test file per method of the class, six for `DeepEquals`. Last week's six files passed at *t*; the six generated anew this week (120 methods) all fail to compile at *t*, and `DeepEquals` has the most changed lines of the four (2,574, of which KTester executes 178). The 95 Ouédraogo methods for `DeepEquals` pass at *t* and all break, 92 of them at #63, the same commit as last week.

## Limitations

- Each generation is a single run. The difference between the two weeks on the same four classes includes run-to-run variation of the generators, which was not measured. Ouédraogo was also generated in one pass this week and in two passes last week.
- 30 methods that fail at *t* in the experiment's shared test JVM but pass in the project's own build are left out of the groups (Dev 17, KTester 11, Ouédraogo 2). 12 of the 17 Dev methods are tests of `CompactMap`.
- The KTester compile errors on `DeepEquals` are a helper class or method defined twice (4 files), `@ExtendWith(MockitoExtension.class)` without its imports (3 files) and an import of a package that does not exist (1 file). KTester is run as published, with its own repair loop (three attempts); the import repair we add after generation is applied to the Ouédraogo output only. Both settings are the same as last week.
- Six commits that do not build carry the previous result and have no changed-line coverage (#29, #225 to #227, #256, #257), as last week.

## Repository

Report: https://github.com/Hamjoon/test-survival-reports, `2026-11-week2-java-util-all-classes/java-util-all-classes-report.md` (this file). Last week's report: `2026-11-week1-regen-policies/regen-policies-report.md`. Experiment branch: https://github.com/Hamjoon/java-util/tree/experiment/2026-11-week2-java-util-all-classes, with the one-time-generation run in `experiments/commit-survival/` and the policy run in `experiments/regen-policies/`.

- Tables 1 and 2: `experiments/regen-policies/results/policy/summary.csv`; `experiments/commit-survival/results/final-summary.json`; week 1 columns from Tables 1 and 2 of last week's report.
- Losses per commit (observation 1): `experiments/commit-survival/results/first-failures.csv`.
- Table 3 and observations 2 and 3: `experiments/commit-survival/results/scope4-comparison.md`.
- Classes given to the generators: `experiments/commit-survival/population/scope.md`.
- Regeneration commits: `experiments/regen-policies/results/policy/plan.md`. Methods left out at *t*: `experiments/commit-survival/results/control-excluded.csv`.
