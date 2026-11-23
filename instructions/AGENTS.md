Answer in concise, structured output! Use bullet points for clarity if it makes sense. Try to keep it to 2-3 paragraphs max, condense the information to what is most important!

## Git safety

- Create or reuse a feature branch; from `origin/main`, use `git switch --no-track -c <branch> origin/main`.
- Never push `main` or let another branch track `origin/main`.
- Push the branch and open a ready PR to `main`; don't merge it unless instructed to do so.

## Model routing

- Default coding planning, implementation and code validation to Claude Opus 5.5 (`claude-opus-5-5`) at high reasoning.
- Default browser control (the user's Chrome) to Claude Opus 5.5 with Claude in Chrome (`mcp__claude-in-chrome__*`). In T3 Code, Codex sessions can't reach Chrome until openai/codex#57641 is resolved.
- Default desktop computer use to GPT-6.2-Sol (`gpt-6.2-sol`) at high reasoning through the Codex provider.
- Route email, Google Drive and other external-service or plugin work through GPT-6.2-Sol and the Codex provider, using the existing OpenAI integrations.
- In T3 Code, use its orchestration tools to delegate across providers. Codex should delegate coding and browser work to Opus; Claude should delegate computer use and external-service work to Codex. For mixed tasks, delegate the relevant subtask with its context and return the result to the coordinating agent.
- Explicit model choices in the user's request override these defaults. If a required provider or integration is unavailable, report the limitation instead of silently substituting another route.
- For larger implementation chunks, use subagents for planning, implementation, testing and validation, with separate file ownership for parallel edits.

## Adversarial review

- After implementation and before creating a ready PR, obtain independent, read-only adversarial reviews from both Opus 5.5 and GPT-6.2-Sol through T3 delegation.
- Have both reviewers examine the same final diff and relevant validation evidence. Assess findings, fix valid issues and repeat affected checks. Request follow-up review when fixes materially change the reviewed behavior.
- One dual-model review covers implementation completion and PR creation when the diff is unchanged. Do not merge unless told to do so.
- Brief reviewers with: the goal, the diff range, what CI already checks (type-check, lint, tests, repo static checks; don't redo them), and decisions already approved (don't re-argue them).
- Reviewers report findings only, most severe first, each with a concrete failure scenario. No "checked and fine" sections, no nits, pre-existing or out-of-scope items unless asked.
- Reviewers look beyond the diff: trace every consumer of changed shared code, try hostile and edge inputs (empty, huge, malformed, other formats), and check state after delete, reload, retry and concurrent edits. Write a small repro instead of reasoning when a claim is testable.
- Re-checks of fixes go to one reviewer (the one that raised the finding) on the fix delta only. Run a new dual review only when behavior changed materially.
- When fixing a finding, fix its whole class: search for the same pattern elsewhere in the change and add a test that would have caught it.

## Temp space

- `/tmp` is a small RAM disk. Agent sessions get `TMPDIR=~/.cache/agent-tmp` (on disk, files untouched for 3 days are deleted); never hardcode `/tmp` for checkouts, builds, venvs, images or other large scratch.
- Review or comparison checkouts: `git worktree add` under `$TMPDIR` (or the repo's worktree convention), never a copied repo; remove the worktree (`git worktree remove`) when the review ends.
- Anything that must survive longer than 3 days goes in a named dir under `~/.cache/<project>-<purpose>`, not in `$TMPDIR`.

## Shared instructions

- This file is the authoritative global guidance for both Codex and Claude. `~/.claude/CLAUDE.md` imports it; keep shared rules here instead of copying them there.
- Keep repository `AGENTS.md` files authoritative for project guidance. Use `CLAUDE.md` only as an import or symlink when needed, without duplicating shared content.
- Preserve existing authorization boundaries when delegating, including for external messages, uploads and deployments.
