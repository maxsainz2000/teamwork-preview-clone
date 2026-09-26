# Phase 1 — Prompt Crafting Workflow (Steps 1-9)

Operational specification for the scoping interview, reconstructed from the
official Antigravity Teamwork documentation (public docs page + launch blog)
and validated behaviorally. This is a paraphrase, not a reproduction of any
proprietary prompt text. Two short canonical trigger phrases are preserved
verbatim in quotes because Phase 2 routing depends on them being present in
the final prompt exactly as written.

Two phases, both required: **(1)** craft a well-structured task prompt with
the user through Steps 1-9; **(2)** delegate it to the agent team. Crafting
without delegation is incomplete.

## Working artifact

Create `prompt_draft.md` in the project working directory immediately, before
substantive questioning. It is the user's live view of the draft and the
agent's step tracker. Sections:

- Status header: current step, goal line (craft → approve → delegate), and a
  "Requested team" line (normally: none — routing follows the description).
- 1-2 sentence project description.
- Working directory (TBD until Step 8).
- Requirements (R1, R2, ... placeholders).
- Acceptance criteria (checkboxes).
- Footer noting the delegation step follows approval.

Update the artifact after every step; never lose the draft if the user
iterates after Step 8 — revise and re-present.

## Four principles

1. **What, not how** — specify requirements and criteria; avoid prescribing
   implementation (files, architecture, algorithms, libraries) unless the user
   explicitly asks for the constraint.
2. **Objective verification** — every requirement needs a check that is
   independent of the implementing agent's own judgment. Prefer programmatic
   checks; an independent judge agent with an explicit rubric is acceptable.
3. **Acceptance criteria as guardrails** — calibrate the bar to the user's
   real purpose to prevent the team certifying weak work as done. If a run
   falls short, tighten criteria and re-run rather than adding hints.
4. **Minimal requirements** — state only what the user cares about; surplus
   requirements squeeze the team's solution space.

## Interview conduct

- Present choices with the structured question tool, not open prose.
- If the user hands over a pre-written prompt, scan it against Steps 1-9 and
  work only the gaps (missing verification and criteria are the common ones).
- If the user wants to skip straight to delegation, push back once (brief:
  underspecified prompts are the main cause of weak results), then respect
  their choice while warning that more iteration may be needed.

### Step 1 — Elicit the idea
Ask what to build, its purpose (demo / production / eval / exploration), and
audience. Summarize in 1-2 sentences; that becomes the prompt's opening.

### Step 2 — Resolve ambiguity
List points with multiple reasonable readings and offer concrete options for
each. Probe: scope, hard technology constraints, infrastructure needs, quality
bar, integrity strictness, existing verification resources. Ask only about
decisions affecting scope or verification.

Two **opt-in scale choices** exist — neither may be inferred, both must be
asked if plausible, and the answer must be recorded in the prompt opening:
- **Small, focused team** (one implementer followed by repeated adversarial
  review; cannot decompose multi-part work): only offered for a single
  self-contained fix/feature/refactor. If chosen, the prompt opens with
  "This is a single self-contained fix; keep it small and focused."
- **Very large proof team** (massive parallel search, math/proof tasks only):
  if chosen, the prompt opens with "Use a very large team of agents." —
  routing looks for this explicit phrasing and defaults to the standard proof
  pipeline without it.

Do not route multi-part work to the small team just because it looks small;
keep the R1/R2 decomposition in that case.

### Step 3 — Integrity mode
Do not ask the user to pick a mode by name. Ask one multi-select behavioral
question about which shortcuts the team may take, offering: copying core
logic from existing open-source projects; using libraries/frameworks for core
functionality; running external scripts or delegating execution; reading test
source before implementing; or no restrictions.

Mapping: "no restrictions" (or nothing selected) → `development` (default).
Some restrictions declined → `demo`. All four declined → `benchmark`
(strictest). For non-build work, ask the equivalent shortcut question (for a
proof: whether existing results may be cited instead of proved) and map
identically. Suggest `demo` when the project is clearly a capability
showcase.

### Step 4 — Requirements
Write 2-5 blocks, each 1-3 sentences on what is needed, no implementation
hints. Litmus test: would a skilled engineer feel over-constrained? If yes,
cut. Never add a requirement the user did not state.

### Step 5 — Verification design
Verification exists to force a real build-test-debug loop; it is a means, not
a mirror of the user's ideal goal. Choose something easy to run and hard to
fake. Programmatic preferred (test suites with known I/O, benchmark or metric
scripts, harnesses); independent judge with a concrete rubric otherwise;
non-build work needs its equivalent forcing function (e.g., a point-by-point
review rubric). Ask whether the user has existing tests, scripts, guidelines,
or a reference implementation — include any as a Verification Resources
section; partial resources still help auditors.

Reject: self-assessment by the implementer, subjective or unfalsifiable
criteria, absent criteria, impossible thresholds.

### Step 6 — Acceptance criteria
Convert each verification mechanism into concrete checkable conditions,
calibrated to purpose: demo = impressive but achievable in budget;
production = target quality standards; eval = precise and reproducible over
polish; exploration = prove feasibility only. Adjust on user feedback
(tighten / relax / unbundle).

### Step 7 — Infrastructure constraints
If the work needs network, remote storage, or job launching, add a requirement
that routes those operations through a controlled interface ("use the provided
API for X; the team writes logic, the environment stays managed"), stating
what is controlled and why. Skip entirely when no infrastructure is involved.

### Step 8 — Working directory
Ask where files should live; default `~/teamwork_projects/{PROJECT_NAME}` with
a short lowercase underscore name. Record it as a top-level directive in the
prompt.

### Step 9 — Assemble and validate
Final prompt structure: 1-2 sentence description; working directory line;
integrity mode line; optional reference material; Requirements (R1..Rn);
Acceptance Criteria (checkboxes by category).

Pre-flight checks before presenting: no unplanned implementation hints; every
criterion objectively checkable; scope set by user needs; infrastructure
constraints state what/why; an engineer would not feel over-constrained; a
team could not trivially self-certify; Step 2 opt-in choices (if any) appear
in the opening in the canonical phrasing; any user-requested team appears in
their own words.

Then tell the user, in one line, which team shape you expect this to route to
(describe the outcome, not internals), and ask for approval.

## Delegation gate

On explicit approval only ("go", "looks good", "launch" and equivalents): copy
the full prompt text out of the draft (never pass the file path — the draft
may change after launch), set the artifact status to Launched, and begin
Phase 2 per SKILL.md. Invoking the team before explicit approval, or skipping
the artifact, are prohibited.

## Routing summary (for Step 9 expectations)

The team reads the prompt and selects one of five execution paths; the prompt
text is the only input: document review (a supplied document to review);
proof pipeline (math/proofs/verification); proof with very large team (opt-in);
small focused team (opt-in); full team (everything else — builds, research,
ops). A user-requested team is a strong signal, not a switch: if it
contradicts the described work, the work wins — say so rather than promising.
Never volunteer a non-default team the user did not ask for.

## Iterate after the first run

If results fall short: tighten acceptance criteria or strengthen verification,
then re-run. Adding implementation hints is the dispreferred last resort.
