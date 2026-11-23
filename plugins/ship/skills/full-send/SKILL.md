---
name: full-send
description: Use when the user invokes "Full Send", "$full-send" (Codex) or "/full-send" (Claude Code), or asks to implement or ship a plan end to end, take something to production, or finish and merge a change without them.
---

# Full Send

Take the work from wherever it is now through merge and, where the repo has a production deploy, a verified production release. Assume the user can't be reached: don't ask questions. When something is ambiguous, pick the smallest correct interpretation and note the decision for the report. Invoking this skill counts as explicit permission to merge this PR into `main`.

Model routing, delegation and git safety follow the global `~/.codex/AGENTS.md`.

## Repo setup

Each repo's `AGENTS.md` defines:
- **DEV:** a local app (against cloud dev services or local mocks) or a deployed dev environment, and how to validate there.
- **Production deploy:** how and where it happens (Vercel, Convex, Cloud Run, manual, or none).
- **PROD test target:** the test project or account used to validate in PROD.
- **Tickets:** whether and how this workflow tracks the change in a ticket or issue tracker.

If it's silent on deploys, assume the default stack: Vercel and Convex production deploys triggered by the merge to `main`. For anything else it doesn't cover, use what the repo shows (README, CI, scripts) and note the assumption.

**Skip, don't fail.** When a step doesn't apply (no ticket tracking, no production deploy, no PROD test target), skip it and give the reason in the report.

**Pending human requirements.** A missing credential or access that blocks only part of validation isn't a stop. Validate everything else (for example, against a local mock), list the gap in the PR and the report as a pending human requirement, and carry on. If the gap leaves the change's own behavior unvalidated, stop and report instead.

## Workflow

1. **Intake.** Work out the starting point: idea, plan, ticket, branch or PR. Create or reuse a feature branch. Read the repo setup above. If the repo tracks tickets, find this change's ticket as its `AGENTS.md` describes, and don't adopt one from a fuzzy match. Otherwise skip tickets and say so in the report.
2. **Plan.** If a plan exists, follow it and don't expand the scope. If not, write one, have Opus 5.5 and GPT-6.2-Sol each review it independently, and incorporate the valid points.
3. **Implement** with Opus 5.5 at high reasoning. For large plans, use subagents with separate file ownership. Commit.
4. **Adversarial review.** Send the same final diff to independent, read-only Opus 5.5 and GPT-6.2-Sol reviewers. Fix the valid findings and rerun the affected checks. If the fixes materially change behavior, run another review round.
5. **Validate in DEV** as the repo defines it. Validate programmatically wherever possible: tests, type-check, lint, API calls. Where it's warranted, add an end-to-end check through the UI, delegated to GPT-6.2-Sol. Save the proof to `~/.full-send/<repo>-<branch>/dev/`: command output, screenshots, recordings.
6. **Open the PR.** Push and open a ready PR to `main` with a summary, the review outcome, the DEV proof in text form, pending human requirements and the ticket link if there is one. Link the PR to the ticket as the repo describes. Then run the `babysit-pr` skill until the review is clean.
7. **Merge** with the repo's default method. Check `gh repo view --json mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed`. If more than one is allowed, use whichever recent merged PRs used.
8. **Watch the production deploy** the merge triggers until it is live or has failed, and record the deployment ID or URL. If it fails with an obvious error caused by this change, fix it in a small follow-up PR, link it to the ticket, run `babysit-pr`, merge, and watch again. Anything bigger: stop and report. If the repo has no production deploy, or it only deploys manually, skip steps 8–9 and say so.
9. **Validate in PROD** against the merge commit, in the repo's PROD test target, delegating browser work to GPT-6.2-Sol. Save the proof to `~/.full-send/<repo>-<branch>/prod/`. If no test target is defined, run a read-only smoke check and say so.
10. **Close out the ticket** as the repo describes, or skip if it doesn't track tickets. Record a `Production validation` section: date, PR links, merge SHA, deployment, each check and its result, and the proof folder path. Invoking this skill counts as the request to close it, but only once PROD validation passes. If PROD was skipped, follow the repo's rule for that case, or leave the ticket open for the user. Verify every write. If a write fails, read the ticket back before retrying and report what's done.
11. **Report:** a short summary covering what shipped, decisions made, review findings and pushbacks, how DEV and PROD were tested, skipped steps and why, pending human requirements, the PR link, the proof folder and the ticket with its state. Post the same summary as a PR comment.

## Stop and report

Stop instead of guessing when:
- a required service or integration is logged out or unavailable, including a ticket tracker the repo requires (unless it's a pending human requirement as defined above);
- `babysit-pr` stops;
- DEV or PROD validation fails and the cause isn't an obvious, small fix;
- the plan turns out to be wrong or unsafe.

Write the report with the current state and the exact blocker. Leave anything unmerged unmerged.

If the production deploy or PROD validation fails, don't close the ticket. Record the failure and its proof as the repo describes and leave the ticket open.

## Boundaries

This skill only authorizes merging its own PRs, the production deploys the merge triggers automatically, and the ticket updates the repo's `AGENTS.md` defines for this workflow. No manual production deploys, no data changes outside the test projects, and no external messages beyond PR comments and the ticket.
