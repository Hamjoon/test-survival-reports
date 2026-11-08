# Does regenerating tests help, and when? Three regeneration policies on five Java projects at commit granularity, with the developer tests regenerated too

## Key findings

1. **A reset restores the set only until the next large change to the scope classes, so the set at the last commit depends on whether such a change comes after the last reset.** On ojAlgo the commit #299 ("Additional file formats") comes after the last release reset (#204) and drops the regenerated Dev(scope) set from 462 to 153 of 494; the no-regeneration set had fallen to 8 of 353 at #144. On smallrye-config the refactoring of `ConfigMappings` at #478 comes after the last reset (#381) and leaves KTester at 15 of 55 and Ouédraogo at 0 of 49 under the release policy, while the Dev(scope) set re-established at #504 under the 50% policy ends at 242 of 246. On java-util the last reset (#213) comes after the large changes (#18, #134, #193) and every regenerated set ends above 93%; on jinjava the changes after the last reset (#64) cost the regenerated Dev(scope) set 139 of 1220 methods and the two LLM sets none.
2. **The developer tests execute more of the changed scope lines than either LLM group on all five projects, and regeneration does not close the gap.** Without regeneration Dev(scope) covers 50.7% of the changed lines on ojAlgo against 23.3% for KTester, 74.9% against 1.8% on smallrye-config, 96.4% against 0.0% on jinjava, 58.2% against 38.1% on java-util and 72.4% against 24.1% on logback (Table 2). The release resets add lines on both sides (KTester +21, +31, +7, +52 and +3; Dev(scope) +46, +34, +2, +82 and +1), so the gap in percentage points moves little: it widens on ojAlgo (27.4 → 34.0), narrows on logback (48.3 → 41.4) and moves by less than 4 points on the other three.
3. **Survival and changed-line coverage have to be read separately.** On jinjava the KTester set of *t* passes at every commit (21/21) but executes 0 of the 165 changed scope lines, and so does the Ouédraogo set (0 of 165); Dev(scope) executes 159. Both generators produced passing tests only for `Functions` (KTester, all 21) or for `Functions`, `DeferredValueUtils`, `EagerBlockSetTagStrategy` and `EagerImportingStrategyFactory` (Ouédraogo, 89), while the changes in the range fall on the macro classes (#8, #28, #38), for which neither generator produced a passing test at *t*. After the two resets the KTester sets cover 7 and the Ouédraogo sets 1 of the 165 lines.

## Question

A test set generated at a starting commit *t* breaks as the code under test moves on. If the generated tests are regenerated along the way, when should that happen and how much is gained? Three regeneration policies are compared against no regeneration on five Java projects, with two measurements taken at every commit after *t*: how many of the tests still pass (survival), and how many of the lines changed at that commit the surviving tests execute (changed-line coverage). New this week, the developer reference set is re-established at the reset commits as well, so that the LLM sets are compared with the developer tests of the same code.

## Setup

**Projects.** Five Java projects, each from a starting commit *t* along the first-parent line of the main branch. Commit #*n* is the *n*-th first-parent commit after *t* (*t* = #0). Commits that change no production code are skipped; a commit that does not build carries the previous result.

| Project | *t* | Range | Scope classes (methods) |
|---|---|---|---|
| ojAlgo | v53.1.1 (`edde296`, 2023-10-16) | 300 commits to 2026-07-22, 240 evaluated | `ExpressionsBasedModel` (104) |
| smallrye-config (`implementation` module) | 3.6.0 (`838b091`, 2024-02-28) | 586 commits to 2026-11-06, 439 evaluated | `ConfigMappingInterface`, `ConfigMappingLoader`, `ConfigMappings`, `ConfigMappingContext` (41) |
| jinjava | the `master` merge of release 2.7.2 (`446cff7`, 2024-02-15) | 116 commits to 2026-11-06, 104 evaluated | `EagerMacroFunction`, `Functions`, `MacroFunction`, `EagerBlockSetTagStrategy`, `EagerSetTagStrategy`, `EagerImportingStrategyFactory`, `DeferredValueUtils` (46) |
| java-util | 2.18.0 (`a85c604`, 2024-10-17) | 300 commits to 2025-06-15, 236 evaluated | `CompactMap`, `DeepEquals`, `ClassUtilities`, `ReflectionUtils` (58) |
| logback (`logback-core` module) | v_1.4.12 (`88abf59`, 2023-11-27) | 100 commits to 2024-09-06, 73 evaluated | `JoranConfiguratorBase`, `PropertySetter`, `ModelInterpretationContext` (52) |

Scope classes are public classes that change repeatedly inside the range and whose developer tests number at most a few hundred.

**Groups.** Every count is the number of test methods that passed one run at *t*.

| Project | Dev(scope) | KTester | Ouédraogo |
|---|---:|---:|---:|
| ojAlgo | 353 | 197 | 15 |
| smallrye-config | 168 | 58 | 87 |
| jinjava | 590 | 21 | 89 |
| java-util | 346 | 171 | 375 |
| logback | 45 | 159 | 169 |

- **Dev(scope)**: the methods among the project's own tests that the build runs and that execute a line of a scope class. Under the two reset policies this set is re-established at the reset commit from the tests of that commit; under the fill policy and under no regeneration it is the set of *t*.
- **KTester**: Li et al., ICSE 2026, the released tool with its default of three repair rounds.
- **Ouédraogo**: Ouédraogo et al., EMSE 2026, the five prompt templates with three adjustments: the signatures of the project types the class refers to are added to the prompt, the model is told to use only the API shown and no mocks, and missing imports are inserted after generation. Two generation rounds at every generation point.

**Regeneration policies.** Each group is run under four policies; a regeneration generates tests for the scope classes at that commit with the same generator settings as at *t*.

| Policy | When | What |
|---|---|---|
| No regeneration | never | the set generated at *t* |
| Reset at each release | two releases per project: ojAlgo v55.1.0 (#104) and v56.1.0 (#204); smallrye-config 3.11.2 (#194) and 3.16.0 (#381); jinjava 2.7.3 (#36) and 2.8.2 (#64); java-util 3.0.2 (#125) and 3.3.2 (#213); logback 1.5.1 (#32) and 1.5.6 (#65) | the group is discarded and generated again; Dev(scope) is re-established |
| Reset when 50% broken | the first commit where at least half of the group has broken, read from the no-regeneration curve, once per group: ojAlgo #34 (Dev(scope), Ouédraogo); smallrye-config #504 (Dev(scope)), #130 (KTester), #30 (Ouédraogo); logback #28 (Dev(scope)), #21 (Ouédraogo); never for KTester on ojAlgo and logback and for every group on jinjava and java-util | the same |
| Fill when 50% broken | the same commit | the surviving tests are kept; new tests are added for the focal methods (Ouédraogo: the classes) of the broken ones, at most as many as broke per focal method; Dev(scope) stays the set of *t* |

**Measurement.**

- A method survives at commit *c* if it passed at every evaluated commit up to *c*; a method that fails once is not run again.
- A regeneration starts a new segment: the methods that pass one run at the regeneration commit are the segment denominator, and survival inside a segment only decreases.
- Changed-line coverage is, for each commit that changes executable lines of the scope classes, the share of those lines executed by the group's surviving methods at that commit.

## Results

Table 1 gives survival at the last commit, Table 2 the changed-line coverage over the range and Table 3 how the sets at *t* were selected. The per-commit series (survival and changed-line coverage under each policy, one sheet per project) are in the shared spreadsheet: https://docs.google.com/spreadsheets/d/1MSN_emRi9AvQZrceqIOdhmyIRAZVpMfxNiYZWANVhZk/edit. There, a survival row is shown when a group lost three or more methods, and a coverage row when the commit changed ten or more scope lines or a group lost methods; the remaining commits are aggregated.

**Table 1. Surviving methods at the last commit of the range** (alive / denominator of the last segment; number of regenerations in brackets). Under the two reset policies Dev(scope) is the developer tests of the reset commit, including tests added since *t* and tests that started to execute a scope class after *t* (segment denominators 353 → 368 → 494 on ojAlgo, 168 → 193 → 209 on smallrye-config, 590 → 1169 → 1220 on jinjava, 346 → 538 → 554 on java-util, 45 → 39 → 39 on logback), so its percentages there are not comparable with the no-regeneration column; under the fill policy it is the set of *t*. Where the 50% trigger did not fire (jinjava and java-util; KTester on ojAlgo and logback), the two 50% columns repeat the no-regeneration column.

| Project | Group | No regeneration | Reset at each release | Reset when 50% broken | Fill when 50% broken |
|---|---|---|---|---|---|
| ojAlgo | Dev(scope) | 3/353 (0.8%) | 153/494 (31.0%) [2 regen] | 34/351 (9.7%) [1 regen] | 3/353 (0.8%) |
|  | KTester | 181/197 (91.9%) | 143/149 (96.0%) [2 regen] | 181/197 (91.9%) | 181/197 (91.9%) |
|  | Ouédraogo | 0/15 (0.0%) | 0/12 (0.0%) [2 regen] | 0/6 (0.0%) [1 regen] | 0/6 (0.0%) [1 regen] |
| smallrye-config | Dev(scope) | 33/168 (19.6%) | 174/209 (83.3%) [2 regen] | 242/246 (98.4%) [1 regen] | 33/168 (19.6%) |
|  | KTester | 6/58 (10.3%) | 15/55 (27.3%) [2 regen] | 6/75 (8.0%) [1 regen] | 6/30 (20.0%) [1 regen] |
|  | Ouédraogo | 0/87 (0.0%) | 0/49 (0.0%) [2 regen] | 0/22 (0.0%) [1 regen] | 0/51 (0.0%) [1 regen] |
| jinjava | Dev(scope) | 503/590 (85.3%) | 1081/1220 (88.6%) [2 regen] | 503/590 (85.3%) | 503/590 (85.3%) |
|  | KTester | 21/21 (100.0%) | 86/86 (100.0%) [2 regen] | 21/21 (100.0%) | 21/21 (100.0%) |
|  | Ouédraogo | 55/89 (61.8%) | 125/125 (100.0%) [2 regen] | 55/89 (61.8%) | 55/89 (61.8%) |
| java-util | Dev(scope) | 181/346 (52.3%) | 516/554 (93.1%) [2 regen] | 181/346 (52.3%) | 181/346 (52.3%) |
|  | KTester | 135/171 (78.9%) | 256/265 (96.6%) [2 regen] | 135/171 (78.9%) | 135/171 (78.9%) |
|  | Ouédraogo | 241/375 (64.3%) | 319/322 (99.1%) [2 regen] | 241/375 (64.3%) | 241/375 (64.3%) |
| logback | Dev(scope) | 10/45 (22.2%) | 33/39 (84.6%) [2 regen] | 33/39 (84.6%) [1 regen] | 10/45 (22.2%) |
|  | KTester | 145/159 (91.2%) | 167/171 (97.7%) [2 regen] | 145/159 (91.2%) | 145/159 (91.2%) |
|  | Ouédraogo | 47/169 (27.8%) | 34/34 (100.0%) [2 regen] | 35/66 (53.0%) [1 regen] | 77/108 (71.3%) [1 regen] |

**Table 2. Changed scope lines executed by the surviving tests** (lines executed, share of the executable scope lines changed in the range; the number of changed lines is given with the project). A line counts as covered when a method alive at that commit executes it, so a set that breaks early covers only the lines changed before it broke; read with Table 1. "=" means the same as the no-regeneration column because the trigger did not fire for that group. A line changed at several commits is counted at each of them.

| Project | Group | No regeneration | Reset at each release | Reset when 50% broken | Fill when 50% broken |
|---|---|---|---|---|---|
| ojAlgo (377 lines) | Dev(scope) | 191 (50.7%) | 237 (62.9%) | 215 (57.0%) | = |
|  | KTester | 88 (23.3%) | 109 (28.9%) | = | = |
|  | Ouédraogo | 0 (0.0%) | 1 (0.3%) | 11 (2.9%) | 11 (2.9%) |
| smallrye-config (1,632 lines) | Dev(scope) | 1,223 (74.9%) | 1,257 (77.0%) | 1,249 (76.5%) | = |
|  | KTester | 30 (1.8%) | 61 (3.7%) | 106 (6.5%) | 30 (1.8%) |
|  | Ouédraogo | 23 (1.4%) | 95 (5.8%) | 24 (1.5%) | 30 (1.8%) |
| jinjava (165 lines) | Dev(scope) | 159 (96.4%) | 161 (97.6%) | = | = |
|  | KTester | 0 (0.0%) | 7 (4.2%) | = | = |
|  | Ouédraogo | 0 (0.0%) | 1 (0.6%) | = | = |
| java-util (6,565 lines) | Dev(scope) | 3,824 (58.2%) | 3,906 (59.5%) | = | = |
|  | KTester | 2,503 (38.1%) | 2,555 (38.9%) | = | = |
|  | Ouédraogo | 1,910 (29.1%) | 1,850 (28.2%) | = | = |
| logback (29 lines) | Dev(scope) | 21 (72.4%) | 22 (75.9%) | 22 (75.9%) | = |
|  | KTester | 7 (24.1%) | 10 (34.5%) | = | = |
|  | Ouédraogo | 10 (34.5%) | 10 (34.5%) | 14 (48.3%) | 13 (44.8%) |

**Table 3. Selection of the LLM sets at *t*** (generated methods → methods passing one run → after removing duplicates; the last column is the no-regeneration denominator). Ouédraogo rows sum its two generation rounds.

| Project | Group | Generation runs | Generated methods | Passed at *t* | Duplicates removed | Set at *t* |
|---|---|---:|---:|---:|---:|---:|
| ojAlgo | KTester | 1 | 1184 | 198 | 1 | 197 |
| ojAlgo | Ouédraogo | 2 | 209 | 15 | 0 | 15 |
| smallrye-config | KTester | 1 | 471 | 58 | 0 | 58 |
| smallrye-config | Ouédraogo | 2 | 574 | 87 | 0 | 87 |
| jinjava | KTester | 1 | 336 | 28 | 7 | 21 |
| jinjava | Ouédraogo | 2 | 908 | 94 | 5 | 89 |
| java-util | KTester | 1 | 813 | 171 | 0 | 171 |
| java-util | Ouédraogo | 2 | 964 | 426 | 51 | 375 |
| logback | KTester | 1 | 466 | 159 | 0 | 159 |
| logback | Ouédraogo | 2 | 614 | 182 | 13 | 169 |

## Observations

1. **The developer tests of the scope classes and the LLM tests break at different commits.** Dev(scope) loses most at commits that change other classes the tests also run through: ojAlgo #34 (168 of 330 lost; KTester 8, Ouédraogo 15), ojAlgo #144 (83 of 91; KTester 3), java-util #18 (116 of 345; KTester and Ouédraogo 0), smallrye-config #504 (93 of 126; KTester 0). The LLM groups lose most at changes to the scope classes' own API: smallrye-config #478 (KTester 51 → 15, Ouédraogo 35 → 0 in the release-reset sets; Dev(scope) 185 → 175), java-util #63 (Ouédraogo 110; Dev(scope) 0), logback #21 (Ouédraogo 122 of 169; Dev(scope) 17 of 44).
2. **The number of passing methods a regeneration produces varies between commits of the same project, and on ojAlgo the Ouédraogo generation almost never compiles.** KTester: 327 and 149 at ojAlgo #104 and #204; 99 and 86 at jinjava #36 and #64 (21 at *t*); 157 and 265 at java-util #125 and #213. Ouédraogo: 6, 0 and 12 at ojAlgo #34, #104 and #204 (1, 0 and 1 of 10 files compiled; 1 of 10 at *t*); 66, 61 and 34 at logback #21, #32 and #65.
3. **At the 50% trigger a fill keeps what a reset discards, and on logback that is the better half.** The 47 Ouédraogo methods carried at logback #21 all survive to the end of the range, while of the 61 methods added there 30 survive (77/108); the reset set of 66 generated at the same commit keeps 35. On smallrye-config the fill for KTester at #130 adds 2 methods to the 28 carried, although the generation at #130 produced 75 passing methods (the reset set), and the refactoring at #478 leaves 6 methods under all three variants. On ojAlgo every Ouédraogo method breaks at #34, so fill and reset are the same 6 methods, lost at #113.

## Limitations

- Commits that do not build carry the previous result: ojAlgo 4 (#145 compile error in the project; #274, #275, #300 an unresolved test dependency), smallrye-config 9 (#486 to #494, compile error in the project), java-util 6 (#29, #225 to #227, #256, #257), logback 4 (#11, #68, #70, #79). No changed-line coverage is recorded at these commits.
- Developer tests that pass in the project's own build but not in the experiment's shared test JVM are left out of Dev(scope), and some of them exercise the scope classes: on logback 28 methods fail at *t* in the experiment (15 of `AsyncAppenderBaseTest` do not compile on the JDK 25 runner, 11 of `IncludeActionTest`, 2 others), and the 11 of `IncludeActionTest` belong to a class whose passing method executes 26 scope lines; on java-util 18 methods fail in the experiment against 1 in the project's build, 15 of them tests of `CompactMap` and `ReflectionUtils` (`results/dev-method-coverage-t.csv`).
- On ojAlgo 1 of the 10 Ouédraogo files compiled at *t* (15 methods) and 1, 0 and 1 of 10 at the three regenerations, so the Ouédraogo rows for ojAlgo carry little information. The scope of ojAlgo was also changed once: the four solver classes chosen first are package-private or abstract and the LLM tests for them almost never compiled (KTester 19 of 73 files, Ouédraogo 1 of 40), so the public `ExpressionsBasedModel` replaced them.

## Repository

Report: https://github.com/Hamjoon/test-survival-reports, `2026-11-week1-regen-policies/regen-policies-report.md` (this file). Experiment branches `experiment/2026-11-week1-<project>-regen-policies` in `Hamjoon/ojAlgo`, `Hamjoon/smallrye-config`, `Hamjoon/jinjava`, `Hamjoon/java-util` and `Hamjoon/logback`; the no-regeneration run is `experiments/commit-survival/` and the policy run `experiments/regen-policies/` in each. Per-commit tables: https://docs.google.com/spreadsheets/d/1MSN_emRi9AvQZrceqIOdhmyIRAZVpMfxNiYZWANVhZk/edit.

- Settings: `experiment.yaml`; scope: `population/scope.md`; Dev(scope) of *t*: `dev/dev-scope-denominator.csv`; generation at *t*: `generated/<group>/round*/funnel.md` and `sources/`.
- No regeneration: `results/survival.csv`, `final-summary.json`, `coverage-commits.csv`, `first-failures.csv`, `skipped-commits.csv`.
- Policies: `results/policy/plan.md`, `summary.csv`, `consistency.md`; per policy `results/policy/<policy>/survival.csv`, `coverage-commits.csv`, `first-failures.csv`; regeneration pools `results/policy/pools/<group>/<commit>/` (funnel and sources).
