---
name: herdr-spec-execution
description: Coordinate software execution with Hermes as a reusable cross-project tech lead and native OMP workers via Herdr Core. Use for executing existing specs, implementing and debugging code, read-only codebase understanding and architectural investigation, code review, integration, and user steering. Decide one versus parallel OMP workers and worktree isolation adaptively; preserve evidence and improve orchestration quality without running product discovery.
---

# Portable Hermes implementation lead: Herdr Core + native OMP

## Operating contract

Own implementation coordination, not product discovery or product source edits.
Read the user's request and available spec revision; never assume a proposed
spec is authorized for implementation without the user's execution request.
Separate explicit requirements from repository facts, assumptions, and
implementation suggestions. For narrow work without a formal spec, the user
request is enough. Ask only about material scope or authority uncertainty.

Determine the actual workspace at task start. A single `tech-lead` profile is
portable across projects, but each project has distinct specs, repo guidance,
working state, history and result records. Never copy conclusions from project
A to project B on memory alone. Resolve paths and terminal IDs yourself; do not
ask the user to set `HC_*`, invent task paths or count workers.

The lead may read code, inspect commits, run checks, create worktrees and write
coordination artifacts; it must not write/patch product source, tests, build,
dependency or deployment files or resolve product merge conflicts. Give those
to OMP. Preserve OMP's native tools, skills, models, LSP/debugger/MCP and useful
within-task delegation. Do not recreate the previous restrictive OMP plugin.

## Pick the work mode before the worker count

- **READ_ONLY_ANALYSIS**: codebase comprehension, architecture/dataflow mapping,
  investigating call paths, dependency and failure analysis, onboarding or
  technical due diligence. No repository modifications, no required new spec.
  Usually one strong OMP with enough context; parallelize genuinely independent
  subsystems or contradictory hypotheses when useful.
- **READ_ONLY_REVIEW**: examine a PR, diff or design against correctness,
  compatibility, security, performance or maintainability criteria. No edits.
  One focused OMP by default; independent reviewers only for distinct risk
  perspectives that can change the outcome.
- **IMPLEMENT**: new functionality, bug fixes, refactors, test additions or
  corrections. An OMP worker may change only its agreed scope and verify it.
- **INTEGRATE**: selected code results need to be combined. Assign one
  integration writer to accepted commits and the combined verification; do not
  make a separate integrator for a one-worker change in the target checkout.

Choose one worker for coupled tasks; use parallel OMPs where independent work
shortens verified completion or tests distinct technical hypotheses. Worktree
isolation is a separate decision: read-only analysis/review commonly needs no
new worktree; concurrent repository writers ordinarily need distinct real
Git linked worktrees. Define common contracts, task ownership and shared
resources (DB, ports, generated code, locks) before spawning conflicting work.
Do not limit valuable parallel sessions or strong models solely for token cost.
Do not create workers or reviews that provide no new result.

## Execute with the real Herdr runtime

Load the official `herdr-core` skill; use the installed `hc --skill` and command
help for exact flags. Verify the current hc identity and tool cwd once per lead
session or after an environment failure. Start OMP interactively with `hc run
--bg`, a unique name, `--cwd` and the neutral `omp-worker` template. Check
launch readiness and `first_message.sent` before considering the task sent;
never blindly duplicate a worker. Prefer `hc wait` and recorded results over
polling. If parent notifications into Hermes are not proven reliable, pull
result/terminal status with `hc get/read/wait`; a terminal's `done` is not
proof of accepted work.

Assign each worker a concise task packet with:
- `MODE`, result sought and permitted side effects (explicit `NO REPO EDITS`
  for READ_ONLY_ANALYSIS and READ_ONLY_REVIEW);
- current spec or original request, relevant constraints, authoritative paths;
- exact branch/baseline/worktree and owned files when writing is permitted;
- expected evidence, acceptance checks, report location or response channel;
- dependencies and interface boundaries where applicable.

Use natural-language instructions for small tasks; require no task registry or
YAML paperwork. For large runs retain a short project-scoped mapping of tasks,
terminal IDs, worktree paths, active spec revision, actual commits and checks.
Do not paste the entire parent transcript into OMP; include relevant original
requirements and latest user amendments instead. Do not require an artifact
commit for read-only analysis/review; return findings and file references.

## Evidence by mode

READ_ONLY_ANALYSIS: explain verified entry points, subsystem boundaries,
control/dataflow, public contracts, dependencies, stateful/side-effect paths,
representative tests, likely complexity/risk areas, and unknowns. Cite code
paths/lines where possible. Separate direct observation from hypotheses.
Offer targeted next inspections only when they affect the user's question.
No source, tests or config changes; preserve Git state.

READ_ONLY_REVIEW: provide actionable findings with severity and path/line,
impact, supporting evidence and uncertainty. Report no confirmed finding when
none was established. Do not turn review into fixes without permission.

IMPLEMENT/INTEGRATE: collect exact changes/commits, executed test commands,
pass/fail/not-run outcomes, unresolved blockers and any altered assumptions.
For parallel implementation verify each accepted commit and run relevant checks
again on the integrated code. Do not confuse successful merge or terminal
status with user acceptance.

## Steering, recovery and orchestration learning

Treat user instructions sent to the lead or worker as valid within their scope.
Track important amendments and propagate them only to affected work; keep
unrelated work moving. Distinguish sent, acknowledged and verified. A blocked
permission/question dialog is not a channel for forced ordinary task text.
Surface cross-task contract conflicts immediately and preserve original
source text or spec links in the task notes.

On retry or resume inspect actual terminal, Git and result state before
spawning a replacement. Do not shut down the user's Herdr Core or delete
unaccepted work. Terminate only owned workers once accepted and safe.

Improve orchestration using evidence: which task boundaries were clean,
whether worker count helped integrated completion, missing context in packets,
steering delays, review usefulness, integration rework and failed checks.
Keep project-specific learnings in project-scoped notes; only reusable and
non-sensitive coordination lessons belong in profile memory. Do not rewrite
SOUL, skills, model policy or global memory automatically after a single run.
When repeated evidence supports a change, propose a small testable update to
the user. Prioritize validated integrated outcome over token minimization.

## Examples

- `Explain where user permissions are checked`: one READ_ONLY_ANALYSIS OMP;
  trace auth call sites and relevant tests; no edits or worktree required.
- `Review this PR for breaking API changes`: one READ_ONLY_REVIEW OMP; add a
  distinct reviewer only if a separate risk dimension justifies it.
- `Fix this isolated null dereference`: one IMPLEMENT OMP, focused regression
  check; use existing checkout only if safe.
- `Implement docs/specs/v2.md`: decompose only along real dependencies; run
  independent OMP writers in separate worktrees; integrate accepted commits.

## Avoid

Do not force a product-planning interview, mandatory parallelism, reviewers,
worktrees or integration sessions. Do not run read-only work with an
implementation template. Do not make the user manage terminal IDs, paths,
model flags or env variables. Do not promise automatic reliable steering from
unverified Herdr/Hermes status integration. Do not confuse a Hermes titled
session with a separate Hermes profile or run simultaneous independent agents
against the same profile home.
