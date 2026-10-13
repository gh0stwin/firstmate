# Live validation of the GitLab forge port at `da90713`

Driven on 2026-10-13 against the public throwaway fixture
<https://gitlab.com/gh0stwin/firstmate-gitlab-port-fixture> with the real `glab` 1.122.0, `glab-axi`, `git`, `jq`,
`tmux` 3.7c, `treehouse` 3.1.1 and `tasks-axi` on PATH.

## How it was run

- A marked disposable lab home (`bin/fm-lab-home.sh create`) with a private tmux socket (`tmux -L fm-lab`) and a
  treehouse pool rooted inside the lab. Every firstmate script ran from this worktree's `bin/`, typed into a shell
  pane on that lab server with `FM_HOME` set to the lab.
- The pane ran a plain shell, not a harness CLI: the scenarios are the scripts firstmate runs, so no model was in the
  loop. Ship tasks were assembled the way a spawn leaves them (task window, `treehouse get` slot, task record) without
  launching a worker agent; the worker's steps (commit, push, `glab-axi mr create`) were run by hand from its brief.
- The only stand-in was `no-mistakes`, shadowed by a no-op so lab teardown never queried the real gate daemon.
  One run of scenario 11 also injected a failing `treehouse` to interrupt a cleanup.
- Merge request !1 (merged on 2026-10-13 by the earlier evidence pass) served the landed-work proof. Merge request
  !2 was opened for this run, driven through draft, ready and a head advance, then closed unmerged and its branch
  deleted (`13-fixture-cleanup.log`). Nothing was merged. The lab was removed afterwards.

## Transcripts

| File | What it shows |
| --- | --- |
| `01-intake.log` | `fm-forge-detect.sh` proposes `forge=gitlab` only for a gitlab.com origin; `fm-project-mode.sh` reads the binding back, keeps `+yolo`, refuses a misspelling and `local-only` |
| `02-brief.log`, `02-brief-*.md` | `fm-brief.sh --forge gitlab` output for direct-PR, no-mistakes and a base branch; forgejo and gerrit still refuse `--base-branch` |
| `03-worker.log` | the worker pushes `fm/gl-arm` and opens !2 with `glab-axi`, as its brief says |
| `04-arm.log` | `fm-pr-check.sh` refuses the real draft and leaves the record byte-identical; once ready it records `pr=` and `pr_head=` equal to the forge's head ref |
| `05-review.log` | after the head advances elsewhere, `fm-review-diff.sh` diffs the freshly fetched `refs/merge-requests/2/head`, not the stale recorded head |
| `06-review-pull-segment.log` | a merge request URL with a `pull` namespace segment resolves the fresh head; the same case through commit `e4b2e73` reproduces the stale read |
| `07-arm-nojq.log` | with no `jq` on PATH the watch arms with no draft check and no `pr_head=`; with `jq` back the same draft refuses |
| `08-github-regression.log` | a real GitHub draft still refuses and a ready pull request still arms with `gh`'s head |
| `09-teardown-setup.log`, `10-teardown.log` | teardown refuses an open merge request and a post-merge local commit, and completes for the merged one; each run's decision path is shown |
| `11-captain-hold.log` | a captain-held landed GitLab task keeps its row open with the merge request as deliverable; the answer closes it, also after an interrupted cleanup |
| `14-spawn-binding.log` | `fm-spawn.sh` refuses a brief whose forge disagrees with the registry, in both directions |
| `12-lsof-less-replay-hazard.log` | not part of this change: on this host (no `lsof`) a replayed teardown signalled another pane's process group |
