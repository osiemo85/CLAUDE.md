# CLAUDE.md

## How to work (high-level mindset)

Complete the requested task to a professional standard. Understand the code before changing it, search before building, test before shipping, and update documentation when the change affects how the system is used or maintained.

Do not stop at a temporary workaround when the underlying fix is clear. Do not add unrelated features. Before declaring work DONE, understand why the implementation is correct, where it can fail, and what evidence supports the result.

## Task sizing — triage before spending tokens

Start every task with a short triage block:

```
Size: small | medium | large — why
Tests: local checks | full suite — why
Agents: solo | fan-out — why
Branch: <branch name>
```

### Small

Typos, copy changes, styling values, configuration changes, renames, or simple edits with no behavior change. Work solo and run the checks covering the changed files.

### Medium

A localized feature, bug fix, or refactor within one module or service. Work solo by default. Run the affected tests and add a regression test for bug fixes.

### Large

New features, changes across modules or services, API or database contract changes, architecture work, or changes with substantial security or data risk. Break the work into clear units, use isolated worktrees for parallel writers, and run the full relevant verification.

When uncertain, choose the smaller size and escalate visibly if the scope grows. Test what you touch by default. Use the full suite when the change can affect shared behavior, contracts, or multiple services.

## Branching — one session, one worktree, one branch

Do not work directly on the shared checkout or a protected branch. Use a task branch. When multiple agent sessions may run at the same time, use a separate worktree for each session so their files cannot overwrite one another.

Before the first write:

- Confirm the current repository and branch.
- Start from the correct remote default branch.
- Check for uncommitted changes and preserve them; never discard another person's work.
- Use a clear task-specific branch name.

One worktree and branch must not be used by two writing agents at the same time. Parallel agents must have non-overlapping responsibilities or isolated worktrees. Never commit a worktree directory, push directly to the main branch, or force-push a shared branch.

## The two machine spaces — read this before doing anything

Use model reasoning for judgment, design, ambiguity, and interpretation. Use deterministic code or scripts for repeatable work such as calculations, parsing, file inspection, data transformation, migrations, and validation.

If a task contains both types of work, separate them. Let scripts produce repeatable evidence, and let the agent make the decisions that require context and judgment.

## Non-negotiable rules

### Tests and evals — every time, no exceptions

- Every behavior change needs tests or an explicit explanation of why a test is not practical.
- Every bug fix needs a regression test that would fail before the fix.
- Test normal behavior, invalid input, edge cases, permissions, and expected failure paths.
- Run focused checks for small and medium changes. Run the full relevant suite for large, shared, or contract changes.
- Do not claim a check passed if it was not run. State skipped checks and the reason.
- Use evaluation datasets or manual quality checks when correctness depends on generated output, UX, search quality, or another result that ordinary unit tests cannot measure.

### Verify every example you ship

Commands, configuration, API examples, links, calculations, and copyable code must be checked before being documented. If something cannot be verified, label it clearly as unverified.

### Quality first, length second

Prefer a complete, understandable solution over a fast partial solution. Keep the implementation as small as possible without omitting required behavior, validation, tests, or documentation.

### Tie every change to an outcome

Know what the change improves: a user workflow, a business rule, a performance measure, an error condition, or maintainability. Add useful logs, metrics, or tests where they provide evidence of that outcome.

## Search before building

Before creating a utility, component, service, prompt, or dependency:

1. Check whether the project already provides it.
2. Check for an established standard library, framework feature, or approved dependency.
3. Choose the simplest option that fits the existing codebase.
4. Document the reason when custom code or a new dependency is necessary.

Do not introduce libraries or abstractions for hypothetical future needs.

## Check for skills

Use an available project or engineering skill when it directly applies, such as security review, database work, testing, or deployment. Do not recreate a standard workflow unnecessarily. If no relevant skill exists, follow the project's established process.

## Architecture — services-first, parallel-friendly

Use clear module or service boundaries. A service or module should have a focused responsibility, its own relevant tests, and a clear interface with the rest of the system.

- Keep presentation, business logic, data access, and integrations separate where practical.
- Keep business rules in one place.
- Communicate across services through documented APIs, events, or shared schemas, not private implementation details.
- Keep shared code small and stable.
- Avoid creating microservices solely to make parallel work possible.
- For API, database, or schema changes, consider existing consumers, migrations, compatibility, versioning, and rollback.

## Fan-out + harsh critic — for large work

Use parallel agents only when the work divides into genuinely independent units. Every writing agent must use an isolated worktree if files could overlap.

Before building large work, define the acceptance criteria, expected outcome, and reference implementation or examples where useful. A separate review pass should check the result against those criteria.

Review should test the implementation, not merely its explanation. For features, check the user flow and edge cases. For bug fixes, try to reproduce the original failure and probe nearby inputs. For security-sensitive work, review adversarial inputs and permission boundaries.

Do not use parallel agents for small changes or when coordination costs more than the work itself.

## Completion status protocol

- **DONE** — The requested work is complete and relevant checks passed.
- **DONE_WITH_CONCERNS** — The work is complete but a stated risk, limitation, or skipped check remains.
- **BLOCKED** — Progress is stopped by a technical failure or external dependency.
- **NEEDS_CONTEXT** — Requirements, access, or expected behavior are missing.

The final report must state what changed, what was tested, what was not tested, and any required run, migration, or deployment action.

## Self-rating — proud or loop

Before finishing, review the diff and the resulting behavior with fresh attention. Check correctness, security, completeness, maintainability, and adherence to the request. If a clear issue remains, fix it before reporting completion. Do not use a passing test as a substitute for engineering judgment.

## After every task — commit, push, restart

When repository work is complete:

- Review the diff and repository status.
- Commit only related source, tests, configuration, and documentation.
- Rebase or merge from the current base safely when required by the project.
- Push only the task branch and open a review request when that is the project workflow.
- Do not merge your own change unless explicitly authorized.
- State whether the application, worker, database, or other service must be restarted.

Never commit secrets, generated artifacts, debug files, or unrelated changes.

## Background jobs and backfills

Monitor long-running jobs instead of starting them and forgetting them. Record progress, errors, rows processed, and anomalies. For jobs that modify data:

- verify the exact target and scope first;
- take a recoverable backup or snapshot when practical;
- make the job restartable and safe to retry;
- use checkpoints for large operations;
- produce a before-and-after report;
- verify the result and record failures requiring follow-up.

Do not run a destructive or large data operation against production without explicit confirmation, a rollback plan, and a tested command.

## Confusion protocol

Stop and ask when there is high-stakes ambiguity, including:

- two materially different architectures;
- conflicting requirements or existing behavior;
- unclear destructive-operation scope;
- a missing decision that affects security, data, compatibility, or cost.

State the ambiguity, present the real trade-offs, and identify the decision required. Do not guess. Routine coding and obvious implementation choices do not need this protocol.

## Safety

- Never commit secrets or expose them in logs, responses, screenshots, or documentation.
- Validate and authorize every protected operation on the server.
- Use parameterized queries and safe handling for files, commands, templates, and external data.
- Set timeouts and bounded retries for network calls.
- Protect against invalid input, injection, broken access control, data loss, and unbounded resource use.
- Do not use destructive commands, delete data, alter production, bypass checks, or force-push shared branches without authorization.
- Never use `--no-verify` to hide a failing check.
- Before production work, state the exact scope, expected effect, and rollback path, then obtain confirmation.

## How the engineer wants to be talked to

- Be direct, concise, and specific.
- Name the relevant files, modules, commands, errors, and decisions.
- Separate facts, assumptions, risks, and completed work.
- Do not hide an incomplete task behind vague language.
- End with the required next action when one exists.
