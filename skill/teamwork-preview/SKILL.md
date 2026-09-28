---
name: teamwork-preview
description: Clone of Google Antigravity's /teamwork-preview slash command. Runs a two-phase multi-agent workflow — Phase 1, a 9-step scoping interview that crafts an approved task prompt via a prompt_draft.md artifact; Phase 2, autonomous execution by a role-based agent team (Orchestrator, Explorers, Workers, Critic, Challenger, Auditors) with file-based artifacts and integrity modes. Use when the user invokes /teamwork-preview or /teamwork, or asks to "run a teamwork session", build something "with a team of agents", or launch multi-agent autonomous project execution.
argument-hint: <project goal description>
---

# Teamwork Preview

## Overview

Reproduces Antigravity's `/teamwork-preview`: craft a well-structured task prompt
with the user (Phase 1), then delegate it to an autonomous multi-agent team
(Phase 2). **Both phases are required — crafting without delegation is
incomplete.**

Fidelity: Phase 1 reproduces the workflow as documented in Google's public
Antigravity Teamwork docs and launch blog (see REPORT.md §1); Phase 2 is
spec-equivalent — Google's internal launched-team prompts are closed-source
and undocumented; the role pipeline here follows the official documentation.
State this honestly if the user asks about fidelity.

## Trigger

The goal follows the command: `/teamwork-preview Design and implement a
cycle-accurate memory controller with lockstep simulation tests.` No flags
exist — team sizing is tuned only by natural-language signals captured in
Phase 1 ("keep it small and focused" / "Use a very large team of agents").

## Phase 1 — Prompt Crafting

1. **Read `references/crafting-workflow.md` now and follow it exactly** —
   Steps 1-9, the artifact structure, integrity-mode mapping, validation
   checks, and routing rules.
2. Create `prompt_draft.md` **immediately** with the artifact structure before
   asking anything substantive; update it after every step.
3. Use `AskUserQuestion` for all choice points.
4. End only on **explicit user approval** of the final prompt text. Never
   delegate on implied consent.

If re-invoked and a `prompt_draft.md` with status "Ready for launch" exists in
the target working directory, offer to resume at delegation instead of
re-crafting.

## Phase 2 — Autonomous Team Execution

1. **Read `references/team-architecture.md`** and act as the **Sentinel** (the
   coordinator role): record `request.md`, route the prompt to its team shape,
   and drive the pipeline.
2. Delegate every team role to the `Agent` tool — one fresh subagent
   invocation per agent role instance. **Check for named role agents first**
   (`tw-orchestrator, tw-explorer, tw-worker, tw-critic, tw-challenger,
   tw-auditor, tw-success-auditor` — Mode B, payload-only task messages);
   otherwise fall back to built-ins with the role prompt prepended (Mode A).
   See "Mode A vs Mode B" in `references/team-architecture.md`. Run
   independent agents in parallel in a single message.
3. The Sentinel coordinates only — it must NOT write implementation, test,
   review, or audit files itself (Workers do). Update `progress.md` at every
   stage transition, not retroactively.
4. Agents coordinate **only through artifacts** in the working directory
   (request / project-plan / progress / findings / reviews / challenges /
   audits) — never by relaying one agent's self-assessment as another's
   premise. Enforce the file-ownership map so parallel Workers never edit the
   same files.
5. Honor the **integrity mode** (development / demo / benchmark) in all
   Auditor prompts.
6. Post one-line status updates per stage. Before launching a Full-team or
   very-large run, warn the user it is long-running and token-heavy and get
   confirmation.
7. Respect the runtime's per-agent turn budget: **one deliverable file per
   Worker, `WRITE EARLY`, and re-measure disk after every dispatch before
   recording a milestone as done.** See "Runtime budget and durability" in
   `references/team-architecture.md` — exceeding budget is the dominant way a
   team silently produces nothing.
8. Never widen a gate, allow-list, or exemption to reach green; escalate that to
   the user as a criterion question instead.
9. The session ends when the **Success Auditor** passes all acceptance
   criteria; on failure, feed named gaps back to the Orchestrator loop (cap
   5 loops per milestone, then escalate to the user). When the cap binds, name
   which kind of blockage it is — a real defect, a criterion unverifiable as
   written, or a ruling only the user can make — and escalate the latter two
   rather than spending the remaining loops.

## After the Run

Deliver: what was built, where it lives, the audit verdicts, and any criteria
relaxed or left unchecked. Remind the user that the documented iteration loop
applies — if results fell short, tighten acceptance criteria and re-run rather
than adding implementation hints.

## Resources

- `references/crafting-workflow.md` — Phase 1 Steps 1-9, reconstructed from public documentation (the operative spec).
- `references/team-architecture.md` — Phase 2 roles, routing table, team-shape pipelines, artifact layout, integrity modes, Mode A/B detection.
- Role agents (Mode B): the seven `tw-*.md` files in this repo's `agents/` directory. Install by copying them to `~/.qoder/agents/` (user scope) or `.qoder/agents/` (project scope), then `/agents reload`. Without them, Mode A (built-in fallback) auto-applies.
