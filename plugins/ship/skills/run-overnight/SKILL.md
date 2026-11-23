---
name: run-overnight
description: Use when the user invokes "Run overnight", "$run-overnight" (Codex) or "/run-overnight" (Claude Code), or says they're leaving for the night and want work or experiments to continue without them until morning.
---

# Run Overnight

The user is leaving and can't be reached until 06:00 the next morning, unless another deadline is given. Use the whole night to run experiments toward the given goal, with full autonomy. Aim to finish close to the deadline with results ready for review, not hours early and not late.

Model routing, delegation and git safety follow the global `~/.codex/AGENTS.md`. Repo `AGENTS.md` files and other skills (for example a training or evaluation skill) define how to run things in their domain.

## 1. Check-in window: the first 5 minutes

The user may still be around for about 5 minutes after invoking the skill. Use that window only for things that need the user, and do them before anything else:

- Run `date`, record the start time and the deadline.
- Check access to every service the night needs: compute (GPU, cloud instances), git push and `gh`, API keys, logged-in integrations, free disk space. A missing login found now costs a minute; found at 02:00 it costs the night.
- Keep the machine awake for the run, for example with `systemd-inhibit --what=sleep:idle` around the long-running work.
- Send one message with your understanding of the goal, the planned experiments and any questions. Give the default you'll use for each question.

Don't block waiting for answers. Continue with your defaults right away and incorporate any reply that arrives. If the harness requires ending the turn to receive a reply, first arm a background timer of about 5 minutes so the run resumes on its own.

## 2. Work autonomously

After the window, the user is gone. Don't ask questions and don't wait for approval.

- When something is ambiguous, choose the option that best serves the goal and log the decision.
- When something fails, debug it, retry or find a workaround. If it stays blocked, log it and move on to another part of the plan. One blocker should not end the night.
- Never end your turn before the deadline: an ended turn means the night's work stops. Wait for long jobs with background processes, monitors or scheduled wake-ups.

## 3. Plan and run

- Order the experiments by expected value and budget their time, so the planned work finishes by about 05:30.
- Keep a journal at `~/.overnight/<YYYY-MM-DD>-<slug>/journal.md` with the plan, timestamps, commands, results and decisions. Store outputs next to it. Update it as you go so the run survives context compaction.
- Create branches and open ready PRs as useful, following the review rules in `AGENTS.md`. Never merge.

## 4. Use the remaining time

If the plan finishes early and there is reasonable time left, keep going on work aligned with the goal: improve the experiments, run follow-ups the results suggest, check surprising results, or fill knowledge gaps. Only start a job that can finish or checkpoint before the deadline. When the time left is too short for anything useful, stop and write the report.

## 5. Morning report, ready by the deadline

Write `report.md` in the journal folder and give the same content as the final message:

- the goal and what ran;
- results, with the key numbers and where the outputs are;
- decisions made and the defaults used for unanswered questions;
- failures and open blockers;
- branches and PRs created;
- recommended next steps.

## Boundaries

Autonomy doesn't widen authorization. No merges, no production deploys, no data changes outside experiment or test resources, and no external messages beyond PR comments.
