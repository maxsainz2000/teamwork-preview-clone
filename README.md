# teamwork-preview — an open clone of Google Antigravity's multi-agent teamwork command

A behavioral clone of the `/teamwork-preview` slash command from Google
Antigravity (IDE + CLI): a two-phase multi-agent workflow where a structured
9-step scoping interview crafts an approved task prompt, which is then
delegated to an autonomous role-based agent team with adversarial verification.

Built for [Qoder CLI](https://docs.qoder.com/cli/subagent) (skills + custom
subagents); the workflow itself is runtime-agnostic.

## Contents

```
REPORT.md                         deep-research report: full specification, timeline,
                                  community evidence, clone blueprint (38 sources)
skill/teamwork-preview/           the clone skill
  SKILL.md                        Sentinel orchestrator (Phase 1 → gate → Phase 2)
  references/crafting-workflow.md Phase 1: 9-step interview spec, integrity modes, routing
  references/team-architecture.md Phase 2: role contracts, team shapes, artifacts, gates
agents/tw-*.md                    seven role subagents with tool gates
```

## Install

1. **Skill** — copy `skill/teamwork-preview/` to `~/.qoder/skills/` (user
   scope) or `.qoder/skills/` (project scope).
2. **Role agents (recommended)** — copy the seven `agents/tw-*.md` files to
   `~/.qoder/agents/`, then run `/agents reload` in Qoder CLI. Without them
   the skill auto-falls back to built-in Explore/general-purpose agents with
   the role contracts injected per call.
3. **Use** — restart or reload skills, then:
   ```
   /teamwork-preview Design and implement <your project goal>
   ```

## How it works

- **Phase 1** — the agent maintains a live `prompt_draft.md` artifact and
  walks nine steps: idea, ambiguity (incl. two opt-in team-size choices),
  integrity mode (development / demo / benchmark), 2-5 "what not how"
  requirements, objective verification design, acceptance criteria calibrated
  to purpose, infrastructure constraints, working directory
  (`~/teamwork_projects/{name}`), assembly + validation. Nothing launches
  without explicit user approval.
- **Phase 2** — the approved prompt routes to one of five team shapes
  (document review / proof pipeline / very-large proof team / small focused
  team / full team). A Sentinel coordinates fresh role agents — Orchestrator,
  Explorers, Workers, Critic, Challenger (falsifier + mutation tester),
  Auditor, Success Auditor — that communicate only through file-based
  artifacts, with exclusive file ownership per parallel track. The run ends
  when the Success Auditor passes every criterion from zero inherited trust.

Validated end-to-end: a full-team run built a tested single-file browser
Snake game (19/19 tests, 9 adversarial probes, clean success audit).

## Fidelity & legal notes

- This is an **independent reimplementation**, not affiliated with or endorsed
  by Google. Phase 1/2 follow Google's **public documentation** ([Teamwork
  docs](https://antigravity.google/docs/teamwork/), [launch
  blog](https://antigravity.google/blog/teamwork-when-ai-becomes-a-research-partner),
  [CLI reference](https://antigravity.google/docs/cli/reference/)).
- Proprietary material extracted during research (the shipped runtime prompt
  text, bulk CLI string dumps) is **not included in this repository**.
- Original content here is MIT-licensed (see `LICENSE`).

## Sources

See `REPORT.md` bibliography (38 sources).
