---
name: council
description: Only when the user explicitly asks for the council ("ask the council", "use the council", "council round", "$council" in Codex, "/council" in Claude Code). Runs the request past four independent models (the newest Claude Fable, Claude Opus, GPT Sol and GPT Astra) and consolidates their opinions, issue lists or votes. Not for ordinary second opinions, routine code reviews or PR review loops; incidental mentions such as file names or organisations don't invoke it.
---

# Council

Run the user's request past four independent council members and consolidate their answers. The user supplies the question; this skill defines only how the council runs.

Before any tool call, check both:

- **The user explicitly asked for the council.** A request for a second opinion, a review or feedback without naming the council is not one. Answer it normally.
- **You are not a council member yourself.** If your task is a council brief (it says you are a council member), answer the brief directly. Don't run this skill and don't start subagents.

The council is for focused questions the user wants several independent views on. It does not replace the independent reviews in `AGENTS.md`, and it never fixes, commits or merges anything on its own.

## Roles

- **Members** each answer the brief on their own.
- **The main thread only orchestrates.** It resolves the members, writes the brief, starts the members, waits, and consolidates. It doesn't answer the request itself, doesn't add its own findings, options or votes, doesn't break ties with its own opinion, and doesn't answer members' questions. It reports what the members said.

## 1. Resolve the members

The council has four seats. Fill each one with the newest model of its family:

| Seat | Provider | Model family |
| --- | --- | --- |
| Fable | Claude | Claude Fable |
| Opus | Claude | Claude Opus |
| Sol | Codex | GPT Sol |
| Astra | Codex | GPT Astra |

Call `orchestrator_capabilities` (use `search` with the family name if a family isn't in the short list) and pick the highest version of each family. Run every member at high reasoning effort (`effort` for Claude, `reasoningEffort` for Codex) unless the user names another level.

If the user names models:

- "X instead of Y" swaps that one seat;
- "without Y" drops that seat;
- a plain list ("council with Opus and Sol") replaces the whole roster.

If a seat can't be filled, don't substitute another model. Run the council with the remaining members and say which seat is missing and why. A council needs at least two members: if fewer than two can run, stop before starting any and say why.

Start every seat with T3 Code's `delegate_task`, including a seat that matches the main thread's own provider, model and settings, so all members run and report the same way. Without `delegate_task`, report that the council can't run rather than falling back to other subagents.

## 2. Work out the mode

Read the mode from the user's request. The user's instructions on how to consolidate always win over the defaults below.

| Mode | Typical request | Each member returns |
| --- | --- | --- |
| **Opinions** | How should we build X? What do you think of this plan? | A recommendation, its reasoning, the main risks, and the alternatives considered with why they lost |
| **Issues** | Find problems in X. What could go wrong? | Findings, most severe first, each with severity (high, medium, low), location, a concrete failure scenario and confidence |
| **Vote** | Which option is best? Pick the next experiments. | A ranking of every listed option, best first, each exactly once, with one line of reasoning per option. Members may propose a missing option separately, unranked. |

If the mode is unclear, use Opinions.

A vote needs one fixed list of options that every member ranks. When the options don't exist yet ("find the issues and vote on which to fix first", "collect ideas, then vote"), run it as two rounds (step 6): first collect, then vote on the consolidated list.

## 3. Brief the members

Write one brief per round and send it, unchanged, to every member. It contains:

- **The question.** Restate the user's request in a clearly marked block, keeping the user's wording, but without the council invocation (`$council`, `/council`, "ask the council") and without instructions meant for the main thread (how to consolidate, later rounds). In a multi-round run, state only the current round's task and deliverable.
- **Pinned context.** Working directory, branch and the exact HEAD SHA. For uncommitted work, save `git diff` (and untracked files that matter) to a file under `$TMPDIR` and point to it. Add files, plans, links, data, and for later rounds the previous round's consolidated output.
- **The mode and its return format** from step 2.
- **The rules:**
  - You are one of several independent council members whose answers are compared. Answer directly. Don't run the council skill, delegate or start subagents.
  - Don't ask questions. State your assumptions and continue.
  - The pinned SHA plus the saved diff are the code under review, not whatever the working tree holds when you read it. Read committed files with `git show <sha>:<path>`, apply the diff in your own worktree if you need the full tree, and flag it if the working tree no longer matches.
  - Read-only in the shared checkout: no edits, commits, pushes, PR comments or external messages. Never change HEAD, the index or the stash, and don't run builds or tests that write into the tree. To inspect another ref or run something, use `git worktree add` under `$TMPDIR` and remove it afterwards.
  - Scratch files and small repros go under `$TMPDIR`.
  - Say what you couldn't check instead of guessing.

Keep credentials and secrets (tokens, passwords, keys, signed URLs) out of every part of the brief and any file it points to; write `<redacted>` instead.

The brief must not hint at an expected answer, the main thread's view, or another member's answer.

## 4. Run

Start all members in parallel with `delegate_task` (async, default shared isolation), one per seat. Keep every task ID. Follow each delegate's `next` instructions: don't poll, and end your turn while members run. Each completion wakes the thread.

- After starting the members, tell the user who is running and end the turn.
- A member **failed** when its delegate reports an error or ends without an answer. Note it and carry on.
- When some members are done and others still run, post a short status (who answered, who is still running) and end the turn. Consolidate when the last one completes, or earlier if the user says so; then `task_cancel` the members still running and report them as missing.
- If fewer than two members give a usable answer, don't consolidate. Report what you have and stop.

## 5. Consolidate

Name the model behind every point. Then consolidate per mode, unless the user said how:

- **Opinions:** where members agree; where they disagree, with each side's strongest argument (keep dissent, don't average it away); then the recommendation the majority supports, or no recommendation if there is no majority.
- **Issues:** one deduplicated list. Merge findings that describe the same problem, sort by severity, then by how many members raised it. Mark each item with the members who found it. Keep single-member findings.
- **Vote:**
  - A table of every member's ranking, then a Borda count: with n options, first place earns n points, last earns 1.
  - Options a member left unranked earn 0 from that ballot, and tied options share their points. Flag every incomplete ballot, and say so if leaving the incomplete ballots out would change the winner.
  - Report the winner and margin, or the top k when the user asked for several picks. Report ties as ties.
  - List options members proposed themselves, with the proposer, unscored, unless a later round votes on them.

## 6. Multiple rounds

Run one round unless the user asks for more. When the user describes a sequence, such as "collect ideas, consolidate them, then vote and keep the majority picks", run each round as described:

- Start every round with a fresh `delegate_task` per seat, with the same models and settings. Don't continue earlier member threads.
- The next round's brief carries the previous round's consolidated output verbatim, so every member works from the same list.
- Retry a seat that failed in the previous round once.

## 7. Report

Reply with:

1. the consolidated result;
2. one short section per member: model, effort and their answer in brief;
3. notes: missing or failed seats, incomplete ballots, and anything a member couldn't verify.

Then stop. Acting on the result is the user's call.
