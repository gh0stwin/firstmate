# Live GitLab end-to-end validation — branch fm/track-firstmate-upstream-pr-7226, head 18020c23

Every command below ran in this test round against the real gitlab.com through the
machine's existing `glab` login (gh0stwin), on a fresh throwaway public fixture repo:

- Repo: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2
- MR !1: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1 (draft -> ready -> merge-commit merged)
- MR !2: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/2 (squash-merged, source branch deleted)

The firstmate home was a disposable marked lab home (`bin/fm-lab-home.sh create`),
removed after the run. Only the non-forge surroundings (tmux window close, treehouse
worktree return, the no-mistakes run check) ran as no-op stubs on PATH, exactly the
hermetic split the 2026-10-13 pass in docs/gitlab-merge-watch.md used; every forge
read, fetch, push, and merge used the real `glab`/`git`.

## Versions

```
$ glab --version | head -1
glab 1.123.0 (5686d7484)
$ jq --version
jq-1.6
$ git --version
git version 2.39.5
$ bash --version | head -1
GNU bash, version 5.2.15(1)-release (x86_64-pc-linux-gnu)
```

## 1. Forge intake proposes gitlab from a real gitlab.com clone

```
$ bin/fm-forge-detect.sh <clone of the fixture>          # origin https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2.git
forge=gitlab evidence=origin remote https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2.git is on gitlab.com
$ bin/fm-forge-detect.sh <clone with origin https://gitlab.example.corp/team/proj>
forge=none
$ bin/fm-forge-detect.sh <clone with origin ssh://someone@review.example:29418/group/console>
forge=gerrit evidence=origin remote ssh://someone@review.example:29418/group/console uses SSH port 29418
```

A self-hosted lookalike is deliberately not guessed; the gerrit protocol fact still wins its own case.

## 2. Arming refuses a real draft merge request, then arms the ready one and records its head

The MR was created with `glab mr create --draft`; the forge reported
`{"state":"opened","work_in_progress":true,"sha":"61b15a7338b470a3bdfc6ba4b0589fe50dcaf98d"}`.

```
$ FM_HOME=$LAB bin/fm-pr-check.sh live-arm https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1
error: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1 is a draft merge request; a draft cannot be merged, so merge monitoring would wait for an event that cannot occur - mark it ready for review and arm again, or declare a wait instead of done if the draft is deliberate
$ echo $?
1
```

Task metadata was byte-identical after the refusal and no poll was armed.

Marked ready (`glab mr update 1 --ready`), the same arm records the head the forge reports:

```
$ FM_HOME=$LAB bin/fm-pr-check.sh live-arm https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1
armed: state/live-arm.check.sh
$ grep -E '^(pr|pr_head)=' $LAB/state/live-arm.meta
pr=https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1
pr_head=61b15a7338b470a3bdfc6ba4b0589fe50dcaf98d
```

The recorded pr_head equals the live `sha` field of the merge request read through
`glab mr view 1 -F json` at arm time.

## 3. Review fetch prefers the fresh refs/merge-requests/1/head over the stale record

Baseline at the arm-time head (NOTES.md carries one line):

```
$ FM_HOME=$LAB bin/fm-review-diff.sh live-arm --stat
diff base: origin/main
 NOTES.md | 1 +
 1 file changed, 1 insertion(+)
```

The source branch then advanced on gitlab.com to
453019eac7c44524cd6fc75fd96feb22b4b273a0 (second NOTES.md line) while the recorded
pr_head stayed 61b15a7338b470a3bdfc6ba4b0589fe50dcaf98d:

```
$ FM_HOME=$LAB bin/fm-review-diff.sh live-arm --stat
diff base: origin/main
 NOTES.md | 2 ++
 1 file changed, 2 insertions(+)
$ git -C $LAB/wt rev-parse refs/fm-review/mr/1/head
453019eac7c44524cd6fc75fd96feb22b4b273a0
```

Two insertions prove the freshly fetched refs/merge-requests/1/head (453019ea, equal
to the live forge sha) won; the stale recorded pr_head would have shown one.

## 4. Teardown refuses unlanded work while the merge request is open

With one local-only commit ahead of every remote (`git log HEAD --not --remotes`
showed 15483fc) and MR !1 still open:

```
$ FM_HOME=$LAB PATH=$LAB/stubbin:$PATH bin/fm-teardown.sh live-mr
REFUSED: worktree /tmp/fm-lab.tdSIih/wt has work not on any remote and not landed.
unpushed commits:
15483fc local-only follow-up (unpushed)
$ echo $?
1
```

The task record was retained after the refusal.

(Earlier control, by design: with every commit pushed to a normal remote branch the
same teardown completes, because pushed-to-remote work is landed per the
bin/fm-teardown.sh contract - the landed-work gate exists to protect work a discard
would destroy.)

## 5. Guarded merge through the product, twice, including a squash

No runner exists for the throwaway repo, so the documented attended waiver was used:

```
$ FM_HOME=$LAB PATH=$LAB/stubbin:$PATH bin/fm-pr-merge.sh live-mr https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1 --allow-no-pipeline
armed: state/live-mr.check.sh
notice: recorded head 61b15a7338b470a3bdfc6ba4b0589fe50dcaf98d disagrees with the live head 453019eac7c44524cd6fc75fd96feb22b4b273a0; verifying the live head
verified: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1 is open and mergeable at head 453019eac7c44524cd6fc75fd96feb22b4b273a0, with no head pipeline, waived by --allow-no-pipeline
! No pipeline running on fm/live-arm
✓ Merged!
https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/1

$ FM_HOME=$LAB PATH=$LAB/stubbin:$PATH bin/fm-pr-merge.sh live-mr2 https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/2 --allow-no-pipeline -- --squash
armed: state/live-mr2.check.sh
verified: https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/2 is open and mergeable at head a63031a902da7b2860441329ca2828e2cd04d5ae, with no head pipeline, waived by --allow-no-pipeline
✓ Merged!
https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/2
```

The first merge named the stale recorded head and verified the live head, as designed.

## 6. Teardown landed-work proof through the merged merge request

MR !2 was squash-merged and GitLab deleted its source branch, so the worker's commit
a63031a9 lived nowhere on a remote-tracking branch - only in refs/merge-requests/2/head:

```
$ git -C $LAB/wt log --oneline HEAD --not --remotes
a63031a add NOTES four
$ git -C $LAB/wt merge-base --is-ancestor HEAD origin/main || echo "NOT reachable - the MR ref was the only proof"
NOT reachable - the MR ref was the only proof

$ FM_HOME=$LAB PATH=$LAB/stubbin:$PATH bin/fm-teardown.sh live-mr2
/tmp/fm-lab.tdSIih/project: synced dd2cec7..30ca504
teardown live-mr2 complete (window firstmate:fm-live-mr2, worktree /tmp/fm-lab.tdSIih/wt)
$ echo $?
0
```

The private ref the proof fetched names the exact live merge request head:

```
$ git -C $LAB/wt rev-parse refs/fm-review/mr/2/head
a63031a902da7b2860441329ca2828e2cd04d5ae
```

The merged-state read runs before that fetch, so the ref's existence is also evidence
the real forge answered state=merged for MR !2.

## 7. The armed poll answers merged for both real merge requests

```
$ bin/fm-pr-poll.sh --validated $(tr '\n' ' ' < $LAB/state/live-poll.pr-poll)   # MR !1
merged
$ bin/fm-pr-poll.sh --validated gitlab https://gitlab.com/gh0stwin/firstmate-gitlab-live-r2/-/merge_requests/2 gitlab.com gh0stwin/firstmate-gitlab-live-r2 2
merged
```

## 8. Registry binding and brief rendering for forge=gitlab

```
$ FM_HOME=$LAB bin/fm-project-mode.sh --forge fp        # registry: - fp [no-mistakes forge=gitlab] - fixture
gitlab
$ FM_HOME=$LAB bin/fm-project-mode.sh fp
no-mistakes off
```

Adversarial arms, both refused as designed (exit 3, empty stdout):
`forge=bitbucket` -> "refused: unknown forge ... accepted values are forge=gerrit,
forge=gitlab, forge=forgejo, and forge=external"; `[local-only forge=gitlab]` ->
"local-only publishes nothing, so a forge has no meaning there".

The scaffolded ship briefs carry the GitLab wording:

```
3. Use glab-axi for GitLab operations, gh-axi for GitHub operations, and chrome-devtools-axi for browser operations.
Delivery contract: mode=direct-PR forge=gitlab
When it is implemented and committed, push your branch and open a merge request with `glab-axi` that is ready for review, not a draft.
Before you report done, read the merge request back from the forge and confirm it is not a draft (`glab-axi mr view <iid> --full` must print `draft: no`, ...); if it is a draft, mark it ready with `glab-axi mr update <iid> --ready`.
```

The no-mistakes brief renders the same glab-axi readback after the CI-ready return
point, and `--base-branch main` scaffolds on gitlab while `--forge forgejo
--base-branch main` refuses ("a base branch is not supported on forge=forgejo").
