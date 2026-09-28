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
| **Explorer** | `tw-explorer` | Research over the repo/landscape/domain, including web sources. Writes exactly one artifact — its own digest in `findings/` — and never touches the project tree, so it is safe to fan out in parallel. |
| **Worker** | general-purpose subagent | Implements **one deliverable file** with file+terminal tools, restricted to its owned files. Writes the deliverable's skeleton early and extends it, so a stop mid-flight still leaves content on disk. Runs its verification commands before reporting. |
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
├── project-plan.md        # milestones, dependencies, parallel tracks, file-ownership map,
│                          #   machine-readable milestone status table, dated revision sections
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
1. Orchestrator → `project-plan.md` (milestones + ownership map + status table).
2. **Harness before content.** Build and self-test the verification gates first,
   serialized, before the parallel content fan-out. Gates authored in the same
   wave as the content they judge cannot check that content, and every Worker
   that wants to self-verify ends up running without one.
3. Per stage, parallel: Explorers → `findings/` digests.
4. Parallel Workers per independent milestone (sequential if files overlap),
   one deliverable file each, written early and extended.
5. Per milestone: Critic review → fixes → Challenger suite → fixes (loop until
   Challenger passes; cap loops at 5, then escalate to user).
   For a **non-code deliverable** (spec set, report, roadmap, proof write-up)
   there is no executable behaviour to mutate — substitute executable probes
   with: a structural gate over the corpus (schema/ID/citation resolution run
   against a clean tree), an independent judge against a fixed rubric whose
   output format is machine-parseable, and citation/reference existence checked
   mechanically. Adversarial effort goes on the *claims*, not the runtime.
6. Fresh-stage Orchestrator until plan complete.
7. Auditor (integrity + criteria) → Success Auditor → deliver or loop back with gaps.

**When the loop cap binds, stop dispatching and name the shape of the blockage.**
Repeated loops against the same red criterion usually mean one of three things,
only the first of which another Worker can fix: (a) a real defect in the work;
(b) a criterion that cannot be verified objectively as written — a universal
claim over a corpus policed by a sampling spot-check is the classic shape, since
it is guaranteed to fail eventually and can loop forever; (c) an exemption or
allow-list entry that only the user can ratify. For (b) and (c) the correct move
is to **return to the user with the quoted criterion and a proposed narrower
wording**, not to spend the remaining loops. Record which of the three it is,
with the measurement that says so.

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

## Runtime budget and durability (measured on our own runs, not inferred)

These are hard operating limits of the runtime, and the dominant cause of failed
milestones. Treat them as design inputs when decomposing, not as accidents to
recover from.

**A subagent invocation is capped at a finite turn budget (observed: 150 turns).**
When it is exhausted the agent stops mid-sentence, often *after* doing real work
and often *before* writing anything. The failure is not evenly distributed: a
milestone phrased as "survey many files, then author one large artifact" spends
the whole budget reading and lands nothing. Measured in one run: 4 of 7 Stage-0
dispatches returned zero or partial deliverables for exactly this reason.

- Decompose to **one deliverable file per Worker invocation**, and say so in the
  brief. Multiple artifacts in one brief is the single most reliable way to
  produce an empty directory.
- Put **`WRITE EARLY: create the file with its headings and first content before
  any long read, then extend with Edit`** in every Worker and Explorer brief.
- Bound the reading: tell the agent which files matter, from the plan's measured
  inventory, rather than letting it rediscover the corpus.
- A brief may carry numbers, but label them **"a claim to re-check, not a value to
  copy"** — and require the agent to report the contradiction if its own
  measurement differs. Numbers transcribed from briefs have been published as
  measurements and were wrong.
- If a dispatch returns a turn-limit message, treat the milestone as incomplete:
  measure what landed, then re-dispatch the **remainder only**, narrowed further,
  with the already-landed files named so it does not redo them.

**Verify disk after every dispatch — never a report.** Agent-reported success is
wrong often enough to be treated as unpromising, and it is wrong in both
directions: agents have reported work complete that never reached disk, reported
files at line counts disk contradicted, and reported their own work as absent
while it was landing. The Sentinel must re-measure (`wc`, `grep`, `stat`,
`git status`) before recording any milestone state.

Corollary, and it burned a session: **do not record a Worker integrity event from
a single measurement taken while other writers are still running.** One Sentinel
grepped minutes after a dispatch, found nothing, published the report as
fabricated, and was wrong — the edits landed later. Re-run after quiescence, twice,
before accusing.

**Deliverables must survive a destructive agent.** An adversarial probe once
deleted a session's three main deliverables and replaced them with empty files,
and the integrity gate stayed green because an empty sanctioned directory is
invisible to it. Recovery depended on the runtime's private transcript store —
not a control the session owned. Defenses, in order of strength:

1. Ask about versioning during Phase 1 (see `crafting-workflow.md` Step 8), not
   after the loss. If the user opts in, the Sentinel commits at every stage
   boundary with a one-line message naming the stage.
2. If the workspace must stay unversioned, the Sentinel writes a
   `progress.md` **snapshot table** at each stage boundary — every deliverable
   path with its byte size and line count — so silent truncation is detectable
   at the next boundary rather than at the final gate.
3. Every gate must fail **loudly and non-zero** when its declared input is
   missing or empty. A check that exits 0 over an empty directory is worse than
   no check: it manufactures evidence.

**Evidence has a budget.** Gate transcripts and scorecards accumulate fast (one
spec session produced ~940 files under `audits/`). Write them under a dated
`audits/gates/` tree, keep the *latest* full run plus anything a report cites,
and prune superseded duplicates — record in `audits/verify/README.md` what the
retention rule is so an auditor knows what absence means.

**Attribute every producer, or the gate manufactures forgery.** If evidence
files must be named after the agent that wrote them, the naming scheme has to
admit every role that produces evidence — auditor, critic, challenger, and the
coordinator — not just milestone ids. A terminating auditor that cannot legally
sign its own transcript is pushed into borrowing a milestone id and declaring the
deviation, which corrupts the ownership map the rule exists to protect.

**Resuming.** `progress.md` prose is not a resume mechanism. Read the
machine-readable milestone status table in `project-plan.md`
(`id | owner-agent | deliverable files | verification command | state`), verify
each `landed-verified` row against disk, and continue from the first row that is
not. On re-invocation, if `prompt_draft.md` + `project-plan.md` exist in the
working dir, offer "resume" instead of re-crafting.

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
- **Re-measure disk after every dispatch**, before recording any milestone as
  done, and before deciding a Worker needs re-dispatching. The Agent tool's own
  success signal is not evidence of content on disk.
- **Brief every dispatch with**: its one deliverable file, the files it owns,
  `WRITE EARLY`, the acceptance criteria it must satisfy verbatim, the integrity
  mode, and any counts you are handing over labelled as claims to re-check rather
  than values to copy.
- At each stage boundary, update the milestone status table in `project-plan.md`
  and take the **durability snapshot** (see "Runtime budget and durability"): a
  version commit, or a recorded path/size/line table if the workspace is
  unversioned.
- Never widen a gate, an allow-list, or an exemption to reach green. A check that
  cannot be satisfied is escalated to the user as a criterion question; a check
  the team quietly edited until it passed is an integrity failure regardless of
  what it then prints.

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
exist: spawn `general-purpose` for all roles — `Explore` only for read-only
lenses, since it cannot write its digest or reach the web — and prefix each task
message with the corresponding role contract from this repo's
`agents/tw-<role>.md` body as the role's identity
block. Expect weaker discipline (tool limits are advisory only) and re-state
the file-ownership rule verbatim in every Worker task.

**Check the gates before trusting a role.** Mode B detection is not enough: the
installed `tw-*.md` may be an older copy than this repo's, and a stale tool list
silently re-creates a limitation the fix removed (`diff` the installed files
against `agents/` at the start of Phase 2 and say which you are running on).

## Delegation Protocol (Phase 1 → 2 handoff)

When the user approves ("go", "looks good", "launch", "run it", or similar):

1. Extract the complete prompt **text** from prompt_draft.md (copy it — the artifact
   may change after launch).
2. Create the working directory and write `request.md` from that text.
3. Set artifact status to: Launched.
4. Route per the table above and run the pipeline as Sentinel.
