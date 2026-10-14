# Live GitLab validation, second pass: firstmate PR #7226 at ca86a7ab (test phase, run 01M4W96VYH6N7W5D43QCJNMY5G)

Date: 2026-10-14. This is an independent re-drive of the test phase at the same target commit. The files
one directory up are from the earlier attempt of this phase, which used MR !3; this pass used its own MR:
https://gitlab.com/gh0stwin/firstmate-gitlab-port-fixture/-/merge_requests/4 (created, drafted, readied,
advanced by a fix-round push, and squash-merged in this pass). The fixture is the public throwaway
project https://gitlab.com/gh0stwin/firstmate-gitlab-port-fixture, approved by the captain.

Every firstmate command ran from the gate worktree at ca86a7ab against a disposable lab home made with
`bin/fm-lab-home.sh create` (`FM_HOME=<lab>`, every `FM_*_OVERRIDE` unset). Each lifecycle call printed
`fm-gate-refuse: gate agent lifecycle permitted only against lab home <lab>`. Real tools: `glab` 1.122.0,
`glab-axi`, `git` 2.39.5, `jq` 1.6, `tasks-axi`, and `tmux` 3.7c on the lab's private socket
(`fm-lab-home.sh tmux-dir`), with a real worker pane per task. `treehouse` v3.1.1 was also real, using a
pool inside the lab (`TREEHOUSE_ROOT=<lab>/pool`) with leased slots that teardown returned. The one guard was
`no-mistakes`: read-only `axi status` passed through to the real binary, and pipeline-controlling verbs were
refused. None were attempted. Pushes used `-o ci.skip`, so the source-branch pipelines show `skipped`. The
lab was removed when this pass finished.

| # | Scenario | Result | Log |
|---|----------|--------|-----|
| 1 | Arming the real draft MR !4 refuses (exit 1). The task record stays byte-identical, and no poll is armed | pass | 01-arm-refuses-draft-mr.log |
| 2 | With jq off PATH (the ruled jq-soft arm), the forge head is not read. With no remote ref, the unchanged ship-done gate refuses (02a). With a remote-tracking ref, the draft MR arms with no draft check and no `pr_head=` (02b) | pass | 02a-…, 02b-… |
| 3 | Marked ready, the same MR arms with jq. `pr_head=` equals the forge head, and that head alone satisfies the gate, because the copy has no remote ref | pass | 03-arm-ready-mr-records-head.log |
| 4 | After a fix-round push from another clone, the review diff shows a freshly fetched `refs/merge-requests/4/head` (cd73c80), not the stale `pr_head=` or the local branch (58d01a1) | pass | 04-review-diff-fetches-fresh-mr-head.log |
| 5 | Forge unreachable: the unchanged base-branch fetch exits 128 before the `pr_head=` fallback applies (same on GitHub; outside this change) | observed | 05-review-diff-forge-unreachable-aborts-at-base-fetch.log |
| 6 | Teardown while the MR is open refuses, even though the MR head ref contains HEAD. The worktree, record, backlog row, and worker pane all survive | pass | 06-teardown-refuses-open-mr.log |
| 7 | Real squash merge of MR !4. The source branch is deleted, and `refs/merge-requests/4/head` survives | setup | 07-merge-throwaway-mr.log |
| 8 | The MR is merged, but a later local commit is not in its head: teardown refuses and keeps everything | pass | 08-teardown-refuses-later-local-commit.log |
| 9 | The merged state plus the MR head-ref containment prove the work landed (the content fallback has an add/add conflict, so only the MR proof can accept). Teardown closes the pane, returns the slot, syncs the clone, and closes the backlog row with `GitLab merge request <url>` | pass | 09-teardown-accepts-merged-mr-landed-work.log |
| 10 | Intake proposes `forge=gitlab` for gitlab.com https, ssh, and scp origins, and `none` for lookalike, self-hosted, or non-22-port hosts. Registry: gitlab keeps yolo, and local-only or a typo is refused | pass | 10-forge-intake-and-registry.log |
| 11 | `--forge gitlab` briefs: glab-axi tool rule, MR publish with `--target-branch`, and glab-axi draft readback in both modes. Refused: local-only, a gerrit base branch, an unknown forge, `--shape` | pass | 11-ship-brief-gitlab.log |
| 12 | The brief's `glab-axi mr view <iid> --full` prints `draft: no` on the real MR, and `mr update --ready` exists | pass | 12-brief-readback-commands-on-real-mr.log |
| 13 | Spawn refuses a brief whose forge disagrees with the registry, in both directions, and lets an agreeing gitlab brief through | pass | 13-spawn-forge-agreement.log |
| 14 | A captain-held GitLab task lands through the MR proof. Teardown keeps the call open with the MR as its deliverable, and only the answer closes it | pass | 14-captain-held-gitlab-retain.log |

`suite-*.log` are the targeted unit suites for the touched scripts, run at the same commit.

`suite-fm-teardown.log` has 117 passes, including all four GitLab cases. It then stops at the inherited `leaked-process-reap` case, because this host has no `lsof`. That is the same failure as on the base commit, already recorded as declined; CI owns the rest of that file.
