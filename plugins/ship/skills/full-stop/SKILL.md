---
name: full-stop
description: Use when the user invokes "Full Stop", "$full-stop" (Codex) or "/full-stop" (Claude Code), or explicitly asks to get a change ready to merge.
---

# Full Stop

Take the work from wherever it is now to a reviewed, DEV-validated PR that is ready to merge, then stop and hand the merge to the user. Once the user merges, do whatever post-merge work applies: watch the production deploy, validate in PROD, update the ticket. Assume the user can't be reached: don't ask questions. When something is ambiguous, pick the smallest correct interpretation and note the decision for the report.

Model routing, delegation and git safety follow the global `~/.codex/AGENTS.md`. In T3 Code, link every PR you create or work on. Delegated children return results and never watch PRs.

## Repo setup

Each repo's `AGENTS.md` defines:
- **DEV:** a local app (against cloud dev services or local mocks) or a deployed dev environment, and how to validate there.
- **Production deploy:** what the merge to `main` deploys automatically (production, staging or nothing) and where (Vercel, Convex, Cloud Run), and which rollouts stay manual.
- **PROD test target:** the test project or account used to validate after the merge.
- **Tickets:** whether and how this workflow tracks the change in a ticket or issue tracker.

If it's silent on deploys, assume the default stack when the repo shows it: Vercel and Convex deploys triggered by the merge to `main`. For anything else it doesn't cover, use what the repo shows (README, CI, scripts, plan docs) and note the assumption.

**Skip, don't fail.** At intake, work out which steps apply to this PR. When one doesn't (no ticket tracking, no deploy for this PR), skip it and give the reason in the report.

**Pending human requirements.** A missing credential or access that blocks only part of validation isn't a stop. Validate everything else (for example, against a local mock), list the gap in the PR, the readiness snapshot and the report, and carry on. If the gap leaves the change's own behavior unvalidated, stop and report instead.

## Never merge

Never merge, enable auto-merge, add the PR to a merge queue, ask a bot to merge, or authorize a delegate to do any of these. State this boundary in every delegated task and in the `babysit-pr` handoff. When reusing an existing PR, disable auto-merge and remove any active merge-queue entry before pushing. If that fails, stop and report.

## Workflow

### Until ready to merge

1. **Intake.** Work out the starting point: idea, plan, branch or PR. Create or reuse a feature branch. If the repo tracks tickets, find or create this change's ticket as its `AGENTS.md` describes, and don't adopt one from a fuzzy match. Work out what applies after the merge: a deploy, PROD validation or a tracked ticket.
2. **Plan.** If a plan exists, follow it and don't expand the scope. If not, write one, have Opus 5.5 and GPT-6.2-Sol each review it independently, and incorporate the valid points.
3. **Implement** with Opus 5.5 at high reasoning. For large plans, use subagents with separate file ownership. Commit.
4. **Independent review.**
    - **Before the first review**, list what the change touches and cover it with tests: every writer of each changed field or table (search the inserts, updates and deletes), every state of in-flight work (queued, running, settled, cancelled, unmounted mid-save), each changed limit at limit − 1, limit and limit + 1, and results that complete out of order or concurrently. Put the list in the review brief.
    - **Review:** send the same final diff to independent, read-only Opus 5.5 and GPT-6.2-Sol reviewers. Fix the valid findings, each with its whole class as `AGENTS.md` says, and rerun the affected checks. If the fixes materially change behavior, run another review round.
    - **Cap:** at most two re-check rounds after the first review, any new dual review included. Then file what is left as follow-up issues (or tickets, as the repo describes), list them in the PR and move on. Only a finding that loses data or breaks the change's main path keeps the loop going.
5. **Validate in DEV**, as the repo defines it. Validate programmatically wherever possible: tests, type-check, lint, API calls. Where it's warranted, add an end-to-end check through the UI, delegated to GPT-6.2-Sol. Save the proof to `~/.full-stop/<repo>-<branch>/dev/`: command output, screenshots, recordings.
6. **Open the PR.** Push and open a ready PR to `main` with a summary, the review outcome, the DEV proof in text form, pending human requirements and the ticket link if there is one. Link the PR to the ticket as the repo describes. Then run the `babysit-pr` skill until the review is clean.
7. **Cover the final head.** If babysitting changed code, make sure the DEV proof and reviews cover the final pushed head. Rerun the affected checks and UI checks. If behavior changed materially, run another dual-model review.
8. **Readiness snapshot.** At the final head, record the head SHA, required checks, draft state, mergeability and conflicts, unresolved review threads, the latest AI review of that head and pending human requirements. Save it to `~/.full-stop/<repo>-<branch>/readiness.md`. Call the PR ready to merge only when required checks are green, it isn't a draft, it merges without conflicts, no threads are unresolved and the latest AI review of that head is positive. If any of these is unknown or blocked, say so instead: post the step 9 report without calling the PR ready, skip step 10 and stop (during step 10, update the readiness comment and keep watching instead). Report pending human approvals, credentials and other requirements as they stand. Never bypass them.

### Hand off and wait for the merge

9. **Hand off.** Post a readiness report as a PR comment: PR link and final SHA, decisions made, review findings and pushbacks, how DEV was tested, the proof folder, the readiness snapshot, pending human requirements, the ticket state, and what happens after the merge (or that nothing does, and why). State plainly that the PR is unmerged and waiting for the user. If anything applies after the merge, do step 10, then end your turn with the same report. Otherwise say that nothing watches the PR, clean up as in Stop and report, end your turn with the report and stop.
10. **Watch the PR.** Cancel any `babysit-pr` fallback timer, stop any background command or Monitor, then call `watch_pull_request` with `until: "merged"`. The watch wakes this thread on failed or passed checks, new comments or reviews, conflicts, and once when the PR merges or closes. It is the only wait: don't create a `schedule_task`, timer or poll for the merge. A watched thread sits in the user's Working list, so call `t3_thread_organize` with `action: "pin"` (no `threadId`, this thread) to keep it at the top of the sidebar while it waits; the pin keeps the watch running. On each wake:
    - **New comments or reviews:** handle them as `babysit-pr` would, redo steps 7–8, update the readiness comment and keep watching.
    - **Failed checks or conflicts:** fix them if the fix is small and obvious, then redo steps 7–8. Otherwise post the blocker on the PR and keep watching.
    - **Still open, nothing to do** (for example passed checks): end the turn with a one-line status.
    - **Whenever you keep watching:** end the turn with a short status of what changed. Don't call `watch_pull_request` or `pin` again; both stay on.
    - **Closed without merging:** call `unwatch_pull_request` and `unpin`, report and stop. Leave any ticket alone.
    - **Merged:** continue with step 11. Stay pinned until the final report.

### After the user merges

11. **Watch the deploy** the merge triggers, as the repo defines it, until it is live or has failed. Record the deployment ID or URL and its status.
12. **Validate in PROD** against the merge commit, in the repo's PROD test target (or staging, reported as staging, if the merge only deploys there), delegating browser work to GPT-6.2-Sol. Save the proof to `~/.full-stop/<repo>-<branch>/prod/`. If a deploy happened but no test target is defined, run a read-only smoke check and say so.
13. **Close out the ticket** as the repo describes, or skip if it doesn't track tickets. Record a `Production validation` section: date, PR links, merge SHA, deployment, each check and its result, and the proof folder path. Invoking this skill counts as the request to close it, but only once PROD validation passes. If PROD was skipped, follow the repo's rule for that case, or leave the ticket open for the user. Verify every write. If a write fails, read the ticket back before retrying and report what's done.
14. **Final report:** what shipped, the merge SHA, the deployment, how PROD was tested, skipped steps and why, pending human requirements and manual rollouts, the proof folder and the ticket with its state. Post the same summary as a PR comment, clean up as in Stop and report, and call `unpin`.

## Stop and report

Stop instead of guessing when:
- a required service or integration is logged out or unavailable, including a ticket tracker the repo requires (a missing credential that blocks only part of validation is a pending human requirement instead);
- `babysit-pr` stops;
- DEV validation fails and the cause isn't an obvious, small fix;
- the plan turns out to be wrong or unsafe.

Write the report with the current state and the exact blocker. Keep the PR unmerged, call `unwatch_pull_request` and `unpin`, and cancel any `babysit-pr` fallback timer.

If the production deploy or PROD validation fails, don't close the ticket. Record the failure and its proof as the repo describes and leave the ticket open. If the cause is an obvious, small fix, take a follow-up PR through steps 2–10 on a new branch, link it to the ticket and hand it to the user to merge. Anything bigger: report it and stop.

## Boundaries

This skill authorizes pushing its own feature branches, opening ready PRs and commenting on them, validating in the repo's PROD test target, and the ticket updates the repo's `AGENTS.md` defines for this workflow. No merges, no manual production deploys, no data changes outside the test environments and projects, and no external messages or uploads beyond PR comments and the ticket.
