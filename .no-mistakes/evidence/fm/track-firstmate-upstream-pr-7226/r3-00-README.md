# Round 3 re-validation - fm/track-firstmate-upstream-pr-7226 @ b36607e0 (target commit)

Re-drove the full live GitLab surface at the committed head b36607e0, against real
gitlab.com through the machine's authenticated glab (gh0stwin), from the gate worktree,
through a fresh disposable marked lab home (/tmp/fm-lab.NElw3W), on fresh throwaway
public fixtures A=gh0stwin/firstmate-t3-a-232901 (MR !1 merge-commit merged, MR !2 squash-merged
with source branch deleted) and B=gh0stwin/firstmate-t3-b-232901 (MR !1 draft -> ready -> head
advanced). Only non-forge surroundings ran as stubs; glab, git, jq, tasks-axi real.

## Live scenarios (all pass)
- r3-01-forge-detect-gitlab.log - forge intake on a real gitlab.com clone -> forge=gitlab
- r3-02-arm-draft-refusal.log - arming the real draft B!1 refused; meta byte-identical; nothing armed
- r3-03-arm-ready.log - after --ready, arm records pr_head equal to the live forge sha; poll armed
- r3-04-review-fresh-mr-head.log - B!1 head advanced on the forge; review fetches fresh refs/merge-requests/1/head (f8276c84) over the stale recorded b30b8183 (2 insertions vs 1)
- r3-05-review-cross-project-fallback.log - adversarial: recorded URL names B while origin is A; no MR ref fetched from A, recorded pr_head wins over the moved local branch, stderr clean (the b36607e0 set-u fix path)
- r3-05b-review-cross-project-prefix.log - identical input against parent 487f4f0d: 'resolved: unbound variable', review degraded to the moved local branch - the regression the fix removes
- r3-06-teardown-merged-mr-allow.log - task head on no remote-tracking branch, landed only through A!2 (merged, branch deleted): teardown completes, backlog row done, real glab read + real refs/merge-requests/2/head fetch
- r3-07-teardown-cross-project-refuse.log - adversarial: recorded URL names merged A!1 while origin is B whose MR !1 head contains the worktree HEAD: REFUSED, meta retained, worktree kept, backlog still in flight
- r3-08-brief-forge-gitlab.log - --forge gitlab ship brief scaffolds: glab-axi Rule 3, forge=gitlab delivery contract, glab-axi draft readback

## Unit suites at b36607e0
- fm-review-diff: 22 ok (includes the cross-project twin advancing the local branch past the recorded pr_head)
- fm-pr-check-security: 50 ok; fm-brief: all ok; fm-dod-lib: all ok; fm-forge-detect: all ok; fm-project-mode: all ok; fm-task-delivery: all ok
- fm-teardown: 144 ok incl. every GitLab case (merged-lands under https+ssh origins, open refuses, stale-head refuses, cross-project refuses, MR-note close on its own fixture); then stops at the known host-local lsof case 'leaked-process-reap' (inherited: lsof not installed; recorded earlier)
- fm-captain-hold-lifecycle: full suite hangs at test_interrupted_origin_move_does_not_inherit_the_previous_answer on this host; the IDENTICAL hang reproduces at base 41a47c09 (r3-10b-captain-hold-base-hang.log), so it is inherited, not from this change; the change's two new GitLab cases pass when driven directly (r3-10-captain-hold-gitlab-cases.log)

Versions: glab 1.123.0 (5686d7484), jq-1.6, git 2.39.5, GNU bash 5.2.15(1)-release.
Fixtures and the lab home were deleted in the same evidence turn.
