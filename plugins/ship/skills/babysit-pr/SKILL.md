---
name: babysit-pr
description: Watch a GitHub PR and close its review feedback until the AI review comes back clean. Use when the user invokes "Babysit PR", "$babysit-pr" (Codex) or "/babysit-pr" (Claude Code), asks to watch, monitor or babysit a PR, or when another skill (such as full-send) hands over a PR to get through review.
---

# Babysit PR

Get an open PR through AI review with the smallest correct changes. Never merge. Merging is the caller's decision.

## Loop

1. **Setup.** Identify the PR. In T3 Code, call `link_pull_request`. If no review is running yet, comment `@codex review`.
2. **Wait.** In T3 Code, call `watch_pull_request` and end the turn. Elsewhere, poll with `gh` about every 5 minutes. Also arm a 15-minute fallback timer, for example a background `sleep 900`.
3. **On each wake-up**, fetch unresolved review threads (GraphQL), the latest review and failing checks.
   - **Review comments:** triage each thread with the `fix-pr-issues` rules. Check it against the current code. Fix real issues with the narrowest change. Push back with a concrete reason on anything out of scope, speculative or not worth the churn. No scope creep. Reply, then resolve the thread.
   - **Failing CI:** fix it the same way.
   - Batch the fixes into one commit per round, rerun the affected checks, push, then comment `@codex review`.
4. **Dead reviewer.** If 15 minutes pass after a push or review request with no review activity, comment `@codex review` again.
5. **Done** when the latest review is positive, no threads are unresolved and the checks are green. Call `unwatch_pull_request` in T3 Code.

## Stop and report

Stop after about 5 review rounds, when the same issue comes back after a fix or pushback, or when the reviewer stays dead after 3 re-triggers. The report covers: rounds, fixes made, each pushback with its reason, and the final state.
