---
title: "How I set up Cursor rules and skills to stop vibe coding"
date: 2026-11-08
status: published
tags: [ai-coding, cursor, prompt-engineering, agent-reliability]
---

# How I set up Cursor rules and skills to stop vibe coding

**TL;DR:** AI makes writing code cheap, so the hard parts become knowing what to build and proving it works. I put my coding agent in a loop of plan, small change, and proof, enforced by 13 lines of always-on User Rules plus a handful of skills that only load when they're relevant. The rules also make the agent say how it knows things: verified, claimed, or inferred.

## The problem

"Vibe coding" is prompting an agent, skimming the result, and accepting it if it looks about right. It works for throwaway scripts. In a real codebase, the agent fills every gap in your prompt with a guess, and you inherit the guesses.

The data backs up the feeling. Google's 2025 DORA report found that AI adoption now improves software delivery throughput, but is still associated with more delivery instability. The teams that benefit have fast feedback loops, strong automated tests, and work in small batches.

Testing has its own trap: one 2026 case study found that up to 40% of AI-generated tests were "weak": they raised coverage but still passed when the code was broken.

## The approach

I wanted the same habits in every project, so I built two layers:

- **Personal defaults** that apply everywhere: Cursor User Rules plus personal skills in `~/.cursor/skills/`.
- **Project rules** that live in each repo: build and test commands, architecture decisions, team conventions. These take precedence.

### Always-on rules: short, one concern per line

Rules are loaded into every conversation, so every line costs context. I kept mine to 13 lines:

```text
# Engineering workflow (personal defaults)
- Project rules, AGENTS.md, and existing code patterns take precedence over these defaults.
- Before non-trivial changes: read the relevant code, then propose a short plan (files to change, approach, edge cases, tests) and wait for approval. Skip the plan for one-line or obvious fixes.
- Separate ambiguities into blocking (stop and ask, with options) and non-blocking (state the default you'll use and proceed).
- Stay in scope. Don't refactor, rename, reformat, or add dependencies outside the task; list them as follow-ups instead.
- Prefer the smallest change that meets the goal. Keep diffs reviewable and split large work into steps.
- Match the surrounding code's style, error handling, and test layout; find a similar existing feature and copy its pattern.
- Ground claims about the codebase in files you've read (cite path:line). Label key claims as verified (ran/read this session), claimed (a tool, doc, or subagent said so), or inferred; "should" and "probably" are not evidence.
- Say "I don't know" or "not verified" rather than guess; retract any claim you can't support.
- Before declaring done, run the project's typecheck, lint, and relevant tests, and report the commands and results. Never claim a check passed that you didn't run.
- Never weaken, skip, or delete tests to make them pass, and never swallow errors to hide failures.
- If a task goes sideways, stop and re-plan instead of patching on top.
- Use the matching skill: fix-bugs-root-cause for bugs, write-meaningful-tests for tests, review-code-independently for reviews, map-codebase for unfamiliar repos, record-architecture-decisions for architecture choices.
- At the end of a meaningful task (a bug fixed, a feature built, a concept learned), offer once to write a dev-journal entry. Don't write it without my OK.
```

Two lines do most of the work against wrong assumptions:

- **Blocking vs. non-blocking questions.** "Always ask when unsure" makes the agent stall on trivia; "never ask" makes it guess on expensive decisions. Splitting the two means I only get interrupted when a wrong guess would cause rework.
- **Labeling claims.** A subagent reporting "tests pass" is a claim until the agent re-runs them. Anthropic's own guidance also notes that explicitly allowing the model to say "I don't know" reduces made-up answers.

### Skills: long procedures that load only when needed

Multi-step workflows don't belong in always-on rules. Each of these is a short `SKILL.md` that the agent picks up when the task matches:

| Skill | What it enforces |
|---|---|
| `fix-bugs-root-cause` | Reproduce with a failing test first, trace the root cause with file:line evidence, make the smallest fix, no weakened tests |
| `write-meaningful-tests` | Approve a behavior list before writing tests, mock only external dependencies, break the code on purpose to prove each test can fail |
| `review-code-independently` | Fixed review passes (correctness, security, performance, design, tests); every finding has a location, severity, and fix; one verdict |
| `map-codebase` | One-page, read-only map of an unfamiliar repo, with a read path and a write path traced end to end |
| `record-architecture-decisions` | Draft decision records the agent can implement from, and check changes against accepted ones |
| `dev-journal` | Record what I learned after meaningful work, quiz me on it, and turn it into posts like this one |

## What went wrong, and what I'd do differently

- **My first research pass was too long to use.** Asking for bullets of at most three sentences each made it practical. Next time I'd ask for the short version first and expand only where needed.
- **New rule lines contradicted old ones.** I added "split blocking from non-blocking questions" while an older line still said "always ask when ambiguous," and another line duplicated an existing one. I caught it by reviewing the full rule set together. Rules need to be reviewed as a whole, not line by line.

## Gotchas

- **A skill can't trigger itself.** If you want the agent to do something at the end of every task (like offering a journal entry), that instruction has to live in always-on rules.
- **Skills are picked up when a chat starts.** A chat that was already open won't see a new skill on its own; point it to the `SKILL.md` file directly or start a new chat.
- **Personal rules aren't shared with your team.** Anything the team must follow belongs in the repo, and your personal rules should say project rules win.

## Takeaways

1. Make the agent plan before it codes, and approve the plan.
2. Keep always-on rules short; move long procedures into skills.
3. Ask only blocking questions; state defaults for the rest.
4. Require proof: fresh test output, failing-then-passing tests, file:line references.
5. Make the agent say how it knows: verified, claimed, or inferred.

## Further reading

- [Announcing the 2025 DORA Report](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report) (Google Cloud)
- [Working in small batches](https://dora.dev/capabilities/working-in-small-batches/) (DORA)
- [Best practices for coding with agents](https://cursor.com/blog/agent-best-practices) (Cursor)
- [Claude Code best practices](https://www.anthropic.com/research/claude-code-best-practices) (Anthropic)
- [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations) (Anthropic)
- [AI-Assisted Unit Test Writing and Test-Driven Code Refactoring: A Case Study](https://www.alphaxiv.org/abs/2604.03135)
