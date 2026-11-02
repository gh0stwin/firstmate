# Live GitLab validation - fm/track-firstmate-upstream-pr-7226 @ 487f4f0d (+ review-diff set-u fix)

Driven against real gitlab.com with the authenticated `glab` on PATH, from the
gate worktree, through a disposable marked lab home (`/tmp/fm-lab.9LaQrH`,
created with `bin/fm-lab-home.sh create`, removed after the run). Throwaway
public fixture projects (both deleted after the run):

- A = gitlab.com/gh0stwin/firstmate-p1-a-11012302 — MR !1 merged (merge
  commit), MR !2 squash-merged; source branches auto-deleted, so only the
  `refs/merge-requests/<iid>/head` refs hold the task commits.
- B = gitlab.com/gh0stwin/firstmate-p1-b-11012302 — MR !1 open, head advanced
  past the task commit.

Only the non-forge surroundings ran as stubs (treehouse return, tmux window
close, gh lookups, the no-mistakes run check) - the same hermetic split the
unit suites and the recorded 2026-10-13 live pass use. glab, git, jq, and
tasks-axi were real.

## Scenarios and evidence files

1. `01-arm-draft-refusal.log` - arming a watch on a real draft merge request
   (B!1 marked `--draft` live) is refused with the draft message; meta stays
   byte-identical; no poll armed. `02-arm-ready.log` - after `--ready`, the
   same arm succeeds and records the live head `pr_head=292cc11f`.
2. `03-teardown-merged-mr-allow.log` - teardown of a no-mistakes ship task
   whose only proof is A!2 (squash-merged; the task commit lives solely in
   `refs/merge-requests/2/head`) completes: exit 0, backlog row closed. The
   gate lib permitted the lifecycle only against the marked lab home.
3. `04-teardown-cross-project-refuse.log` - adversarial: recorded URL names
   A!1 (merged, read live), worktree origin is B whose same-numbered MR !1
   head contains the worktree HEAD. Fixed code REFUSES (exit 1, REFUSED
   line, meta and backlog row retained).
   `05-teardown-cross-project-prefix-hazard.log` - the identical input run
   against the parent commit 9ea8325e (pre-fix): teardown COMPLETES, exit 0,
   backlog closed - the hazard actually fired before the fix.
4. `06-review-fresh-mr-head.log` - review of a B task whose recorded pr_head
   is stale: the fresh `refs/merge-requests/1/head` (292cc11f, "pr fixed on b
   live") is fetched from the URL's own project and wins over the stale
   record; no "PR head unavailable" warning.
5. `07-review-cross-project-fallback.log` - adversarial: recorded URL names
   B!1 while origin is A: no merge-request ref is fetched from A, the
   recorded pr_head is reviewed instead ("+local task work"), no unbound-
   variable noise. `08-review-cross-project-prefix-hazard.log` - pre-fix
   (9ea8325e) on the identical input fetches A's MR !1 head and reports "no
   changes vs origin/main", hiding the task's real work.
6. `09-brief-forge-gitlab.log` / `09-brief-forge-gitlab.md` - a
   `--mode no-mistakes --forge gitlab` ship brief scaffolds (the 9ea8325e fix:
   the heredoc no longer dies on an unbound RULE3) and renders the glab-axi
   Rule 3 line, the `forge=gitlab` delivery contract, and the glab-axi draft
   readback.
7. `10-unit-teardown-gitlab-cases.log` - the GitLab cases inside the
   fm-teardown unit suite, all ok (merged-lands under https and ssh origin
   spellings, open refuses, stale-head refuses, cross-project refuses, MR-URL
   note close on its own fresh fixture - no Git setup errors, the greptile P2
   rename). The suite then stops at the known host-local lsof case
   ('leaked-process-reap'), recorded inherited: lsof is not installed here.
8. `11-unit-review-diff-suite.log` - full fm-review-diff suite, 22 ok, after
   the set-u fix and its extended adversarial twin.

## Defect found and fixed during this round

bin/fm-review-diff.sh resolve_pr_head declared `resolved` without assigning
it; under `set -u` the new GitLab origin-mismatch arm (commit 487f4f0d)
skips the assignment, so `[ -n "$resolved" ]` crashed the function before the
recorded-pr_head fallback could run - stderr printed
"line 185: resolved: unbound variable" and the review degraded to the local
branch instead of the recorded head. Fixed by initializing `resolved=`; the
unit twin now advances the local branch past the recorded head and asserts
the recorded head wins with clean stderr. The twin fails on the pre-fix
script and passes after the fix.
