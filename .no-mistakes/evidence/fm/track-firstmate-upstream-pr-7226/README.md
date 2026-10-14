# Live GitLab validation: firstmate PR #7226 at ca86a7ab (test phase, run 01M4W96VYH6N7W5D43QCJNMY5G)

Date: 2026-10-14. Forge: gitlab.com, the captain-approved public throwaway fixture
https://gitlab.com/gh0stwin/firstmate-gitlab-port-fixture. Merge request created and merged in this run:
https://gitlab.com/gh0stwin/firstmate-gitlab-port-fixture/-/merge_requests/3

Every firstmate command ran from the gate worktree at the target commit against a disposable lab
home created with `bin/fm-lab-home.sh create`, with no `FM_*_OVERRIDE` set. That home had a private
tmux socket and used the real `glab` 1.122.0, `glab-axi`, `git` 2.39.5, `jq` 1.6, `tmux` 3.7c,
and `tasks-axi` 0.2.6. Two shared services were replaced with logging stubs because they are
shared fleet infrastructure: the treehouse pool and the no-mistakes daemon
(`13-stubbed-infrastructure-calls.log`). Pushes used `-o ci.skip` and merge commits carried
`[skip ci]`, because the fixture has no runner; every pipeline GitLab created shows `skipped`. The lab
and its tmux server were removed when the run finished. `$LAB` in the logs is that removed lab home.

| # | Scenario | Result | Log |
|---|----------|--------|-----|
| 1 | Intake proposes `forge=gitlab` from a real gitlab.com clone; look-alike or self-hosted hosts give `none`; the registry binding resolves; a typo is refused | pass | 01-forge-intake.log |
| 2 | A `--forge gitlab` ship brief uses glab-axi for the tool rule, the publish step, and the draft readback, and `--target-branch` for the base branch; local-only, a gerrit base branch, and an unknown forge are refused | pass | 02-brief-gitlab.log |
| 3 | Arming a real draft MR refuses (exit 1); the meta stays byte-identical and no poll is armed | pass | 03-draft-mr-arm-refusal.log |
| 4 | With no jq on PATH, the arm skips the draft check and records no `pr_head=` (ruling: arming must not depend on jq) | pass | 04-jqless-arm.log |
| 5 | The brief's own `glab-axi mr update --ready` and `mr view --full` commands work on the real MR; a re-arm records `pr_head=` equal to the forge head | pass | 05-ready-arm-records-head.log |
| 6 | After a fix-round push from another clone, the review diff reads a fresh `refs/merge-requests/3/head`, not the stale record or the local branch | pass | 06-review-fresh-mr-head.log |
| 7 | Teardown refuses while the MR is open, even though the MR head ref contains HEAD (the glab recorder shows `state: opened`) | pass | 07-teardown-open-mr-refuses.log |
| 8 | Real squash merge with source-branch deletion; main then moves on, so a content-only check is inconclusive | setup | 08-merge-and-main-moves-on.log |
| 9 | MR merged, but a later local commit is not in its head: teardown refuses and keeps everything | pass | 09-teardown-merged-later-commit-refuses.log |
| 10 | Control: with no glab, the same teardown refuses. With real glab, the merged-state read plus the `refs/merge-requests/3/head` fetch prove the work landed; teardown completes and the backlog row closes with a `GitLab merge request <url>` note | pass | 10-teardown-merged-mr-landed-proof.log |
| 11 | A captain-held GitLab task: teardown keeps the call open with `Deliverable of the finished work: GitLab merge request <url>`, and the answer closes it | pass | 11-captain-held-gitlab-retain.log |
