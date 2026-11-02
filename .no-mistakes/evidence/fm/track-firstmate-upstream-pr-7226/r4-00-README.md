# Round 4 re-validation - fm/track-firstmate-upstream-pr-7226 @ e92d1b35 (target commit)

New since round 3's validated head (b36607e0 = today's 076c27a7 content): the rebase onto
newer main b7d2533f (delta content-identical, 25 files + the two below = 27), docs-only
707e6a62, and e92d1b35 "fix(pr-merge): refuse a draft GitLab merge request at merge time"
(bin/fm-pr-merge.sh + tests/fm-pr-merge.test.sh).

Re-drove the live GitLab surface at head e92d1b35 against real gitlab.com through the
machine's authenticated glab (gh0stwin), from the gate worktree, through a fresh disposable
marked lab home (/tmp/fm-lab.m9GwRz), on fresh throwaway public fixtures
A=gh0stwin/firstmate-t4-a-641207 (MR !1 merge-commit merged; MR !2 squash-merged with source
branch deleted, worker sha 1e12a244) and B=gh0stwin/firstmate-t4-b-641207 (MR !1
draft -> ready -> head advanced 8ebc1b1c -> c26dbdb -> draft again -> ready -> merged).
Only non-forge surroundings ran as stubs; glab, git, jq, tasks-axi real. Both repos and the
lab home were deleted in this same evidence turn.

## Live scenarios (all pass)
- r4-01-forge-detect-gitlab.log - forge intake on a real gitlab.com clone -> forge=gitlab
- r4-02-arm-draft-refusal.log - arming the real draft B!1 refused; meta byte-identical; nothing armed
- r4-03-arm-ready.log - after --ready, arm records pr_head 8ebc1b1c equal to the live forge sha; poll armed
- r4-04-review-fresh-mr-head.log - B!1 head advanced on the forge to c26dbdb; review fetches fresh refs/merge-requests/1/head (2 insertions vs 1 at the stale recorded 8ebc1b1c); private ref equals live sha
- r4-05-teardown-cross-project-refuse.log - adversarial: recorded URL names merged A!1 while origin is B whose MR !1 head contains the worktree HEAD: REFUSED, meta retained, worktree kept, backlog in_flight
- r4-06-merge-draft-refusal.log - NEW (e92d1b35): MR armed while ready then marked draft; fm-pr-merge --allow-no-pipeline REFUSES naming "the merge request is a draft" (plus draft_status), MR still open on the forge, main untouched
- r4-07-merge-ready-success.log - same MR ready again: verified at live head c26dbdb under the attended waiver and really merged; the armed poll answers merged
- r4-08-teardown-merged-mr-allow.log - task head 1e12a24 on NO remote-tracking branch, landed only through A!2 (merged, branch deleted): teardown completes, backlog row done, real refs/merge-requests/2/head fetch pinned 1e12a244
- r4-15-brief-forge-gitlab.log - ship brief for forge=gitlab: glab-axi Rule 3, delivery contract, glab-axi draft readback

## Regression proof for the new commit
- r4-14-merge-draft-regression-before-after.log - same fixture, same fake glab, MR JSON with work_in_progress=true and everything else green: parent 707e6a62 verified "open and mergeable" and CALLED glab mr merge; target e92d1b35 refuses with "the merge request is a draft" and no merge call.

## Unit suites at e92d1b35
- fm-pr-merge: 125 ok, 0 failures (includes the new draft-condition, every-failing-condition, and unreadable-draft twins) - r4-13
- fm-review-diff: 22 ok; fm-pr-check-security: 51 ok; fm-brief: 43 ok; fm-dod-lib: 27 ok; fm-forge-detect: 7 ok; fm-project-mode: 13 ok; fm-task-delivery: 35 ok - r4-10..r4-12
- fm-teardown: 144 ok incl. every GitLab case (merged-lands under https+ssh origins, open refuses, stale-head refuses, cross-project refuses, both MR-note closes); then stops at the known host-local lsof case 'leaked-process-reap' (inherited, lsof not installed; identical to the r3 stop and the recorded earlier ruling) - r4-16
- fm-captain-hold-lifecycle: full suite still hangs on this host at test_interrupted_origin_move (inherited, base-reproduced in r3 evidence r3-10b); the change's two GitLab cases pass when driven directly - r4-09

Versions: glab 1.123.0 (5686d7484), jq-1.6, git 2.39.5, GNU bash 5.2.15(1)-release.
