---
name: council
description: Ask a council of four independent models (the newest Claude Fable, Claude Opus, GPT Sol and GPT Astra) for opinions on how to build something, to find issues, or to vote on options, then consolidate their answers. Use only when the user explicitly asks for the council by name, for example "council", "ask the council", "$council" (Codex) or "/council" (Claude Code). Never use it on your own initiative, for routine adversarial reviews or for PR review loops.
---

# Council

Run the user's request past four independent council members and consolidate their answers. The user supplies the question; this skill defines only how the council runs.

The council is for focused questions the user wants several independent views on. It does not replace the adversarial reviews in `AGENTS.md`, and it never fixes, commits or merges anything on its own.

## Roles

- **Members** each answer the request on their own.
- **The main thread only orchestrates.** It resolves the members, writes the brief, starts the members, waits, and consolidates. It doesn't answer the request itself, doesn't add its own findings, options or votes, and doesn't break ties with its own opinion. It reports what the members said.

## 1. Resolve the members

The council has four seats. Fill each one with the newest model of its family:

| Seat | Provider | Model family |
| --- | --- | --- |
| Fable | Claude | Claude Fable |
| Opus | Claude | Claude Opus |
| Sol | Codex | GPT Sol |
| Astra | Codex | GPT Astra |

Call `orchestrator_capabilities` (use `search` with the family name if a family isn't in the short list) and pick the highest version of each family. Run every member at high reasoning effort (`effort` for Claude, `reasoningEffort` for Codex) unless the user names another level. If the user names members or models, use those instead.

If a seat can't be filled, don't substitute another model. Run the council with the remaining members and say which seat is missing and why. If fewer than two members can run, stop and report.

This skill needs T3 Code's `delegate_task` to reach both providers. Without it, report that the council can't run rather than falling back to single-provider subagents.

## 2. Work out the mode

Read the mode from the user's request. A request can combine modes or define its own consolidation; the user's instructions always win over the defaults below.

| Mode | Typical request | Each member returns |
| --- | --- | --- |
| **Opinions** | How should we build X? What do you think of this plan? | A recommendation, its reasoning, the main risks, and the alternatives considered with why they lost |
| **Issues** | Find problems in X. What could go wrong? | Findings, most severe first, each with severity (high, medium, low), location, a concrete failure scenario and confidence |
| **Vote** | Which option is best? Pick the next experiments. | A full ranking of every option, best first, with one line of reasoning per option. Members may propose a missing option, but rank it separately. |

If the mode is unclear, use Opinions.

## 3. Brief the members

Write one brief and send it, unchanged, to every member. It contains:

- the user's request, verbatim;
- the context needed to answer it: working directory, branch, diff range, files, plans, links, data, and any earlier council round's output the request builds on;
- the mode and its return format from step 2;
- the rules: answer independently, as one of several members whose answers are compared; stay read-only in the shared checkout (no edits, commits, pushes, PR comments or external messages); scratch files and small repros may go under `$TMPDIR`; state what you couldn't check instead of guessing.

The brief must not hint at an expected answer, the main thread's view, or another member's answer.

## 4. Run

Start all members in parallel with `delegate_task`, one per seat, in the current checkout. Follow each delegate's `next` instructions and wait for every member. If one fails or stalls, cancel it, report it as missing, and consolidate the rest.

## 5. Consolidate

Name the model behind every point. Then consolidate per mode, unless the user said how:

- **Opinions:** where members agree; where they disagree, with each side's strongest argument (keep dissent, don't average it away); then the recommendation the majority supports, or no recommendation if there is no majority.
- **Issues:** one deduplicated list. Merge findings that describe the same problem, sort by severity, then by how many members raised it. Mark each item with the members who found it. Keep single-member findings.
- **Vote:** a table of every member's ranking, then a Borda count (with n options, first place earns n points, last earns 1), the winner, and the margin. Report ties as ties. Rank options members proposed themselves separately, with who proposed them.

## 6. Multiple rounds

Run one round unless the user asks for more. When the user describes a sequence, such as "collect ideas, consolidate them, then vote and keep the majority picks", run each round as described: the consolidated output of one round goes into the next round's brief verbatim, and the same members are reused.

## 7. Report

Reply with:

1. the consolidated result;
2. one short section per member: model, effort and their answer in brief;
3. notes: missing or failed seats and any member that couldn't verify part of its answer.

Then stop. Acting on the result is the user's call.
