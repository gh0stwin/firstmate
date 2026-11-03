# Live GitLab validation — firstmate GitLab forge port (branch fm/track-firstmate-upstream-pr-7226)

All scenarios below ran against **real gitlab.com** through the machine's existing
glab login (user gh0stwin), in a disposable firstmate lab home created with
`bin/fm-lab-home.sh` (marked lab home — no gate bypass; every fm-teardown run
printed `fm-gate-refuse: gate agent lifecycle permitted only against lab home`).
Only third-party tools outside this change's scope were stubbed on PATH, exactly
like the repo's own test fixtures: `treehouse` (no-op) and `no-mistakes` (no-op).
Everything else — glab, git, tmux (lab socket), tasks-axi — was real.

Evidence kept on gitlab.com (public, for the PR thread):
- Repo: https://gitlab.com/gh0stwin/fm-live-7243-a (kept)
  - MR !1 (merged, the main flow): https://gitlab.com/gh0stwin/fm-live-7243-a/-/merge_requests/1
  - MR !2 (merged, same-numbered twin for the cross-project attack): https://gitlab.com/gh0stwin/fm-live-7243-a/-/merge_requests/2
- Cross-project twin repo fm-live-7243-b (MR !1 open, MR !2 closed-with-head-ref,
  MR !3 squash-merged) was deleted after the run (gitlab renamed it
  `fm-live-7243-b-deletion_scheduled-*`).

## S1 — arm refuses a draft GitLab MR, arms after ready (s1-*)
`bin/fm-pr-check.sh task-x1 <MR !1 url>` on a real draft MR (created with
`glab mr create --draft`):
- s1-arm-draft-refusal.err: refusal "is a draft merge request ... mark it ready
  for review and arm again", rc=1, and the meta recorded nothing (no pr=).
- After `glab mr update 1 --ready`: rc=0, `armed: state/task-x1.check.sh`, meta
  gained `pr=<canonical url>` and `pr_head=44f21f9d…` read from glab's single
  JSON view.

## S2 — review diff uses a freshly fetched refs/merge-requests/1/head (s2-review-diff.*)
A follow-up commit was pushed to the MR branch from a *different* clone, so the
task worktree branch AND the recorded pr_head both lagged the live MR head
(64d742dc). `bin/fm-review-diff.sh task-x1` fetched
`refs/merge-requests/1/head` from gitlab.com and diffed both feature.txt and
followup.txt; `refs/fm-review/mr/1/head` in the worktree resolved to 64d742dc.

## S2b — review diff refuses another project's same-numbered head (s2b-*)
Worktree origin = fm-live-7243-b, recorded pr= names fm-live-7243-a MR !1.
Output: `warning: PR head unavailable; diff may lag the open PR (using local
branch fm/task-x2)` and a diff containing ONLY cb-local.txt — none of
fm-live-7243-b MR !1's head content (b-marker.txt / secret-mr1.txt), and no
`refs/fm-review/mr/1/head` ref was fetched into that worktree.

## S3 — merge-time draft refusal (s3-merge-draft-refusal.err)
After arming, the MR was re-marked draft and
`bin/fm-pr-merge.sh task-x1 <url> --allow-no-pipeline` refused (rc=1) listing
BOTH `detailed_merge_status is "draft_status", not mergeable` AND
`the merge request is a draft` (the new work_in_progress guard). The MR stayed
open/unmerged. Also shows the recorded-vs-live head disagreement notice
(44f21f9d vs 64d742dc) and the FM_PR_CHECK_MERGE=1 re-record path.

## S4 — real merge through fm-pr-merge (s4-merge.*)
`bin/fm-pr-merge.sh task-x1 <url> --allow-no-pipeline` verified live state
("open and mergeable at head 64d742dc…, with no head pipeline, waived by
--allow-no-pipeline") and merged MR !1 on gitlab.com (state=merged,
merge_commit 2b9ac5f3); meta pr_head refreshed to the live head.

## S5 — teardown landed-work proof and its refusals (s5a–s5e)
- s5a stale head: merged MR !1 + a later local-only commit → REFUSED
  ("work not on any remote and not landed"), meta/backlog/worktree retained.
- s5b cross-project (the greptile-P1 attack): recorded URL = fm-live-7243-a
  MR !2 (glab says merged), origin = fm-live-7243-b whose own same-numbered
  refs/merge-requests/2/head == the worktree HEAD → REFUSED, state retained.
- s5c open MR: recorded URL = fm-live-7243-b MR !3 (open), origin matches,
  head ref carries the wt HEAD → REFUSED.
- s5d merged lands (full flow): fm/task-x1 branch deleted + pruned so the
  merged MR is the landing proof → rc=0, meta removed, backlog closed with
  note `GitLab merge request <MR !1 url>` (real tasks-axi).
- s5e merged lands, MR proof uniquely load-bearing: MR !3 was squash-merged
  with --remove-source-branch, so the wt HEAD was unreachable from any
  remote-tracking ref; teardown read merged state through glab, fetched
  refs/merge-requests/3/head, proved containment, killed the real lab tmux
  window (server gone afterwards), synced the project, closed the backlog
  (`GitLab merge request <MR !3 url>`).

## S6 — forge detection on real clones (s6-forge-detect.log)
`bin/fm-forge-detect.sh` on clones with origins:
https://gitlab.com/… → `forge=gitlab evidence=… is on gitlab.com`;
git@gitlab.com:… and ssh://git@gitlab.com/… → forge=gitlab;
https://gitlab.internal.corp/… → forge=none; github.com control → forge=none.

## Targeted regression suites (gate worktree)
fm-forge-detect, fm-brief, fm-dod-lib, fm-project-mode, fm-task-delivery,
fm-pr-check-security, fm-review-diff: full pass. fm-pr-merge: 0 failures
(covered across two runs — the suite outran a single timeout budget).
fm-teardown: 81 oks, 0 failures, all GitLab delta tests green, cut by the
time budget. fm-captain-hold-lifecycle: the two GitLab delta tests pass in
isolation; the suite itself hangs in `test_hold_origins_follow_backend_holds`
— reproduced on the BASE commit (69023626), i.e. upstream main's own breakage.
