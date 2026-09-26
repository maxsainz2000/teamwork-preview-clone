# Phase 2 — Team Architecture (Post-Delegation Execution)

Reconstructed from official Antigravity Teamwork docs (antigravity.google/docs/teamwork/,
/blog/teamwork-when-ai-becomes-a-research-partner) — see REPORT.md §1.3.
**Fidelity caveat:** Google's post-delegation internals are closed-source and no
launched-session transcript is public. This file is a spec-equivalent reconstruction, not
a verbatim recovery. See "Mode A vs Mode B" below for spawning roles with
custom agents (preferred) or built-in fallbacks.

## Roles (documented contracts)

| Role | Agent type | Contract |
|------|-----------|----------|
| **Sentinel** | the main agent running this skill | Takes over on approval. Records the request artifact, distributes work, posts status to the user, triggers final audit, delivers results. |
| **Project Orchestrator** | Plan or general-purpose subagent | Breaks the approved brief into structured milestones with dependencies and a **file-ownership map** (exclusive files per track, so parallel Workers never touch the same file). Assigns parallel tracks. Between stages, its successor is a **fresh** orchestrator invocation fed only the artifacts (mirrors "spawn fresh successors between stages to preserve context"). |
| **Explorer** | Explore subagent | Read-only research over the repo/landscape/domain. May run in parallel fan-out (read-only ⇒ safe). Outputs a findings digest artifact. |
| **Worker** | general-purpose subagent | Implements one milestone with file+terminal tools, restricted to its owned files. Runs its verification commands before reporting. |
| **Critic** | general-purpose subagent, read-mostly | Independent code review for correctness. Never the implementing agent. Emits a review artifact with must-fix / nice-to-fix. |
| **Challenger** | general-purpose subagent | Adversarial: builds test suites specifically designed to **break** the implementation (the documented "falsifier whose sole job is to break it"). New failing tests are wins, not failures. |
| **Auditor** | general-purpose subagent | Validates work against the selected **integrity mode** and every acceptance criterion. |
| **Success Auditor** | general-purpose subagent | Final end-to-end validation against the acceptance-criteria checklist. Its pass **terminates the session**; its failure returns named gaps to the Orchestrator loop. |

Communication is **artifact-mediated, never conversational**: subagents never talk to
each other; each receives only the artifacts it needs and writes its own. Never paste a
prior agent's self-assessment as authority — downstream reviewers re-check the files.

## Workspace & artifacts

All output lives under the working directory chosen in Step 8 (default
`~/teamwork_projects/{PROJECT_NAME}`). Create at launch:

```
<workdir>/
├── prompt_draft.md        # copied prompt text (Phase 1 artifact; copy, never reference by path)
├── request.md             # goals, constraints, integrity mode, acceptance criteria (from prompt text)
├── project-plan.md        # milestones, dependencies, parallel tracks, file-ownership map
├── progress.md            # live milestone tracking; Sentinel updates after every stage
├── findings/              # Explorer digests (exNN-<topic>.md)
├── reviews/               # Critic reports (per milestone)
├── challenges/            # Challenger test suites + results
└── audits/                # Auditor + Success Auditor verdicts
```

Sentinel posts a one-line status update to the user at each stage transition
("Milestone 2/5: implementation done, Challenger running"). Oversight during the run is
passive: the user is asked only when an agent genuinely blocks on a decision.

## Routing — pick the team shape from the prompt text

The prompt is the only input. Check in order:

1. Opening contains "keep it small and focused" (or an unambiguous single
   self-contained fix/feature/refactor request) → **Small, focused team**.
2. A paper/document is supplied to be reviewed, with no build → **Document review**.
3. Opening contains "Use a very large team of agents" and task is math/proof →
   **Proof, very large team**.
4. Math problem / formal proof / verification → **Proof pipeline**.
5. Everything else → **Full team**.

A "Requested team" line in the prompt's own words is a **strong signal, not a
switch** — if it contradicts the work itself (review asked for, no document), follow
the work and tell the user.

## Team-shape pipelines

### Full team (default — builds, research, ops)
1. Orchestrator → `project-plan.md` (milestones + ownership map).
2. Per stage, parallel: Explorers (read-only) → findings.
3. Parallel Workers per independent milestone (sequential if files overlap).
4. Per milestone: Critic review → fixes → Challenger suite → fixes (loop until
   Challenger passes; cap loops at 5, then escalate to user).
5. Fresh-stage Orchestrator until plan complete.
6. Auditor (integrity + criteria) → Success Auditor → deliver or loop back with gaps.

### Small, focused team
One Worker implements → then **repeated adversarial review**: fresh Critic and
Challenger subagents attack it; Worker fixes; loop until two consecutive clean
adversarial passes (cap 5). No Orchestrator, no decomposition — the point is one
line of work driven hard. Cheapest shape.

### Document review
Parallel Explorers each read the document from a distinct lens (correctness,
novelty, claims-vs-evidence, clarity). A Critic acts as **falsifier** against the
strongest positive reading. A synthesizer merges reports into one review artifact.
No Workers/Challenger (nothing to build or test).

### Proof pipeline
Explorers propose diverse candidate proof strategies in parallel. For each live
strategy: a prover writes steps, a **falsifier subagent's sole job is to break it**.
Surviving strands are merged in a **synthesis tree**: pair reports, combine,
re-check at each merge. Auditor/Success Auditor verify the final chain step by step.

### Proof, very large team
As Proof pipeline but with wide parallel fan-out: many concurrent strategist/
prover/falsifier invocations per phase (launch in batches of ~6-10 parallel Agent
calls; on the user's hardware this is API-bound, but cap wall-concurrency to keep
coordination sane). Note honestly to the user that concurrency here is bounded by
this runtime, not Google's 100+.

## Integrity modes (Auditor-enforced)

| Mode | Rule for the team |
|------|-------------------|
| development (default) | Lenient — any approach that works; fastest iteration. |
| demo | No copying core logic from existing open-source projects; no pre-built libraries doing the core functionality; no delegating execution to external scripts/tools; no reading test source to shape implementation. Reproducible presentation. |
| benchmark | Strictest — all four restrictions above enforced; fundamentals only (in a build: standard library / from scratch; in a proof: no citing existing results unless allowed). |

Auditor receives the mode plus the restriction list and must check the produced files
for violations (e.g., grep imports for disallowed deps), not trust Worker reports.

## Cost & durability notes (from community evidence about the original)

Long runs consume heavily (users report 20h+ sessions and token-cost complaints).
Before launching a Full team or very-large-proof run, warn the user and confirm.
Keep `progress.md` current so an interrupted run can be resumed from artifacts: on
re-invocation, if `prompt_draft.md` + `project-plan.md` exist in the working dir,
offer "resume" instead of re-crafting.

## Running a stage (Sentinel duties)

- Update `progress.md` **at every stage transition** (before launching the next
  stage, not retroactively at the end) — it is the user's live window into the run.
- The Sentinel coordinates only: it must NOT author implementation, test, review,
  or audit files itself. Workers implement via subagents. Two narrow exceptions:
  (1) the Sentinel may run verification commands and record their output;
  (2) a one-line mechanical fix (typo, path) may be applied directly, but any
  behavioral change goes back through a Worker invocation.
- Before launching Full-team or very-large runs, warn about token/time cost and
  confirm.

## Mode A vs Mode B — how to spawn role agents

**Detection:** check the runtime's available agent/subagent types. If the
following names (or close variants) exist, use **Mode B**; otherwise **Mode A**:
`tw-orchestrator, tw-explorer, tw-worker, tw-critic, tw-challenger, tw-auditor,
tw-success-auditor`.

**Mode B — named custom agents (preferred).** The role contract lives in the
agent's own system prompt; the Sentinel's task message carries ONLY the
payload: milestone text, owned files, criteria excerpt, integrity mode, and
paths to input artifacts. Install: copy the seven `agents/tw-*.md` files from
this repo into `~/.qoder/agents/` (user scope, all projects) or
`.qoder/agents/` (project scope), then run `/agents reload`. Format: markdown
body = system prompt; tools gated per file; `Agent` denied on every role so no
agent delegates further.

**Mode A — built-in fallback (no custom agents loaded).** No `tw-*` agents
exist: spawn `Explore` for Explorers and `general-purpose` for all other roles,
and prefix each task message with the corresponding role contract from this
repo's `agents/tw-<role>.md` body as the role's identity
block. Expect weaker discipline (tool limits are advisory only) and re-state
the file-ownership rule verbatim in every Worker task.

## Delegation Protocol (Phase 1 → 2 handoff)

When the user approves ("go", "looks good", "launch", "run it", or similar):

1. Extract the complete prompt **text** from prompt_draft.md (copy it — the artifact
   may change after launch).
2. Create the working directory and write `request.md` from that text.
3. Set artifact status to: Launched.
4. Route per the table above and run the pipeline as Sentinel.
