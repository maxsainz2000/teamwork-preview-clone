# Deep Research: Google Antigravity `/teamwork-preview` — Full Specification & Clone Blueprint
> Generated 2026-09-25 | Depth: deep | Sources: 38 | Subagents: 7 (4 retrieval waves + 1 local-mining + 1 gap-fill + 1 verification)
>
> **Public edition.** This repo intentionally does not redistribute proprietary
> source material. Where the research recovered closed-source Google-shipped
> text locally, this edition cites it, quotes only short excerpts, and keeps
> the full extraction private. The clone here is rebuilt from public
> documentation plus that behavioral verification.

## TL;DR

The premise "Google closed-sourced it, so documentation is gone" is **false**: `/teamwork-preview` is thoroughly documented on the live official site (`antigravity.google/docs/teamwork/`, CLI reference, launch blog of 2026-08-27) [1][2][4] — and additionally, the command's full Phase-1 system prompt exists in the author's local Antigravity session transcripts, which was used to behaviorally verify every claim in this report [32]. We therefore hold the exact Phase-1 specification (the 9-step prompt-crafting interview, artifact structure, integrity modes, routing table, delegation protocol) and the officially documented Phase-2 architecture (Sentinel → Orchestrator → Explorers/Workers → Critic/Challenger/Auditor pipeline) — everything needed to build a faithful behavioral clone as a custom skill, except the closed-source *internal* agent prompts of the launched team, which must be reconstructed from the documented role contracts.

## Executive Summary

`/teamwork-preview` (alias `/teamwork`) is a slash command in Google Antigravity 2.0 (IDE) and the Antigravity CLI (`agy`) that launches a **two-phase multi-agent workflow**. In Phase 1, the *foreground* agent runs a structured scoping interview — nine steps, from idea elicitation through requirements, objective verification design, acceptance criteria, integrity-mode selection, and working-directory choice — while maintaining a live `prompt_draft.md` artifact. The user must explicitly approve the drafted prompt. In Phase 2, the approved prompt text is delegated via an `invoke_subagent` tool call to a **hidden subagent type, `teamwork_preview`**, which routes the prompt to one of five team shapes (document review, proof pipeline, very large proof team, small focused team, full team) and executes autonomously with structured-artifact handoffs and layered verification agents [32][1].

Three findings changed the shape of this research. First, closed-sourcing affected Antigravity's *source code*, not its docs — the official documentation set is rich and current [1][2][4][5]. Second, the user's local `.gemini/antigravity` brain transcripts contain the dynamically injected `<TEAMWORK>` prompt block byte-identically across four sessions — a Tier-1 primary artifact that is the single best cloning asset in existence, because it *is* the command's behavior for the entire interactive phase. Third, the impression of "closed-source removal" is likely an inverted merge: Google deprecated the open-source **Gemini CLI** and migrated users *into* the closed-line Antigravity CLI, while Teamwork kept shipping [18][19][2].

The clone is fully feasible as a skill. What cannot be cloned verbatim is the server-side orchestration after delegation — no launched-session transcript has ever been published anywhere, and none exists on this machine [37]. The clone reproduces Phase 2 behavior from the documented role contracts (Sentinel, Orchestrator, Explorers, Workers, Critic, Challenger, Auditor, Success Auditor), artifact-based coordination, file-ownership isolation, and integrity modes, implemented on a general agent runtime (e.g., Qoder's Agent tool or Antigravity custom subagents) [1][23][25].

## 1. Status Quo: The Official Specification [Confidence: High]

### 1.1 Command identity, surface, and availability

The canonical command name is **`/teamwork-preview`** with registered alias **`/teamwork`**; there is **no bare `/team` command** in the Antigravity CLI's slash-command matrix, and any third-party article citing `/team` for Antigravity is inaccurate [4][12]. Verified at the CLI reference: the command is listed as "Launch collaborative multi-agent teams… (paid plans)" [4]. It ships on **both** surfaces — "available on paid plans across Google Antigravity 2.0 and the Antigravity CLI" — with no separate preview-enrollment step; the launch blog states "Available today as /teamwork-preview in Antigravity on all paid plans" (2026-08-27) [1][2]. The "preview" label lives entirely in the command name; no formal beta/sunset program is documented [1].

Invocation is `/teamwork-preview <goal description>`; team-size defaults are auto-selected from the execution path, and sizing is tuned by **natural-language signals in the prompt** ("Keep it small" / "Use a very large team of agents"), not by CLI flags — no flags or config keys are documented anywhere [1][32]. In the CLI, an internal recommender table instructs the agent: "/teamwork-preview: Recommend this when the user has a large project that would benefit from a team of autonomous agents working together" [33].

### 1.2 Phase 1 — the scoping interview (verified against local runtime data)

When the user types the command, Antigravity injects a `<TEAMWORK>` metadata block into the foreground agent's context — a ~2,400-word system prompt that *is* the Phase-1 behavior. A byte-identical copy of it exists in four session transcripts on the author's machine (`~/.gemini/antigravity/brain/<session>/.system_generated/logs/transcript_full.jsonl`) [32]; that text is proprietary Google material and is **not redistributed here** — the structure below is a summary with short cited excerpts, and the clone's `crafting-workflow.md` is a paraphrase rebuilt on the public docs [1]. Its structure:

Key mechanics, as verified against the recovered text and consistent with the public docs [1]:

- **Two-phase mandate:** the foreground agent must craft the prompt through nine interactive steps, then delegate to the hidden `teamwork_preview` subagent — crafting without delegation is treated as incomplete.
- **Artifact-first:** a `prompt_draft.md` is created immediately (status/step tracker, project description, working directory, numbered requirements, acceptance-criteria checkboxes) and updated after every step; it doubles as the user's live view of the draft.
- **Four principles** govern the interview: specify *what* not *how*; require objective verification per requirement; treat acceptance criteria as guardrails against self-certification; keep requirements minimal to preserve the team's solution space.
- **Steps 1–9:** elicit the idea → resolve ambiguity across six dimensions, including two *opt-in* team-size choices the user must be asked about and whose answers must appear in the prompt opening verbatim ("This is a single self-contained fix; keep it small and focused." for the small focused team; "Use a very large team of agents." for the 100+-agent proof team) → derive the integrity mode (`development`/`demo`/`benchmark`) from behavioral multi-select questions rather than asking for a mode name → draft 2–5 requirement blocks → design verification as a forcing function (programmatic preferred, judge-as-fallback) → calibrate criteria to purpose (demo/production/eval/exploration) → add controlled-infrastructure constraints when needed → choose the working directory (default `~/teamwork_projects/{PROJECT_NAME}`, short lowercase-underscore names) → assemble and run a validation checklist before requesting approval.
- **Hard prohibitions:** no delegation without explicit approval; always copy the prompt text (never pass the draft's file path); never skip the artifact; no implementation hints by default.
- **Routing:** `teamwork_preview` is not one agent — it reads the prompt and selects one of five team shapes (document review / proof pipeline / proof very-large-team / small focused team / full team); the prompt text is the only input, and a user-requested team is "a strong signal, not a switch."
- **Delegation:** on approval keywords, extract the full prompt text from the draft and invoke the hidden `teamwork_preview` subagent with it, then mark the artifact Launched.

Local CLI databases additionally expose the `invoke_subagent` tool schema — `TypeName/Role/Prompt` required, a `Model` enum (`inherit`, `flash_lite`, `flash`, `pro`), and workspace modes (`inherit`/`branch`/`share`) — plus feature flags `enable-teamwork-subagent` and `disable-teamwork-forced-flash-model`, the latter implying teamwork runs a **forced Flash model tier** by default, and a subagent-type registry placing `teamwork_preview` in the same enum as `self`, `research`, `browser`, `DeepCoder`, `DeepInvestigator` [33]. These are the only exposed internals; the launched subagent's own prompt is injected server-side and exists nowhere on disk (verified: `~\.gemini\antigravity\builtin` contains no teamwork files; IDE JS bundles yield only false-positive "Sentinel" identifiers) [37].

### 1.3 Phase 2 — the launched team (official documentation)

The docs page describes a strict tiered topology, "orchestration & execution / verification gates" [1]:

- **Sentinel** — "The coordinator that takes over once you approve the prompt": records requests, distributes work, posts status updates, triggers the final audit, and delivers results [1].
- **Project Orchestrator** — breaks the approved brief into structured milestones, assigns parallel tracks, and "spawn[s] fresh successors between stages to preserve context" (context-window hygiene via successor handoff) [1].
- **Explorers** — read-only repository/landscape research agents (an adjacent Google-adjacent post describes them "generat[ing] diverse candidate strategies") [1][38].
- **Workers** — implementation agents equipped with terminal and file tools [1].
- **Critic** — independent code review of correctness; **Challenger** — "actively stress-tests the code by building adversarial test suites"; **Auditor** — validates against the selected integrity mode; **Success Auditor** — final end-to-end validation that terminates the session [1].

Communication is **artifact-mediated, not conversational**: four structured file types — request artifacts (goals/constraints), project-plan artifacts (roadmap/dependencies), progress artifacts (live milestone tracking), and the Phase-1 prompt artifact — with "Agents coordinate and share progress using structured artifacts in the workspace… not direct agent-to-agent chat" [1]. Isolation is strict (corrected after citation check): agents do **not** share one central repo; work lands in the dedicated project directory (default `~/teamwork_projects/{PROJECT_NAME}`) where "orchestrator assigns exclusive file ownership to prevent conflicts" and "each subagent has its own working directory" / private scratch directory [1][5-correction]. User oversight during execution is **passive**: no messaging of individual agents or the team is supported; interaction is limited to Phase-1 adjustments and approval gates, with "Alt+J jump[ing] directly to subagents waiting for approval" [1].

The launch blog adds the algorithmic layer: Gemini "analyzes your prompt and automatically selects the appropriate pattern"; agents "propose, critique, and refine each other's work autonomously over hours or days"; in the proof pattern, parallel strategists are paired with "a falsifier whose sole job is to break it," and "A synthesis tree then combines the candidates and their reports"; critically, **"The framework dynamically decides how many agents to spawn based on task requirements, not a preset number"** — mid-run scaling [2]. Five named collaboration patterns: **Iterative Coding, Distributed Coding, Long Proof, Self-Verification, Document Review** [2]. The stated differentiator versus naive multi-agent chat: unstructured groups "quickly go off track, agreeing with other agents' early mistakes and building confidently on flawed ideas" — Teamwork enforces adversarial verification instead [2]. A Google corporate post ties the feature to Gemini 3.7 Flash powering agent teams for computational/math/engineering problems [3].

### 1.4 Timeline (triangulated)

The earliest date-stamped public footprint is a Google AI Developers Forum topic "Teamwork Preview" dated **2026-06-11** [6]; the official web changelog's only teamwork entry in its May 19–Sep 23 window is a *bug fix* — v2.5.0, July 31 2026: "Fixed agents losing access to their built-in helpers, so research, browser and teamwork assistants work again" [5]; the CLI GitHub changelog carries "Fixed `/teamwork-preview` and some other slash commands disappearing for some users" at v1.1.17 (undated in fetch) [20]; the launch blog is **2026-08-27** [2] and the corporate blog post **2026-08-31** [3]. Reading: a limited/experimental preview from ~June (Reddit reaction threads cluster in May–June by thread-ID adjacency [11]), extended "to all paid tiers" afterward [9], formalized with the August blog push. No official "introduced in" entry exists — a documentation gap, not an evidence conflict [5]. Notably, *sibling* multi-agent commands did get announcement entries (v2.12.0, Sep 2 2026: "Introduced the /boost slash command to enhance thinking effort by using a multi-agent reasoning pipeline"), so `/teamwork-preview` predating the changelog window is the most consistent explanation [5].

## 2. Emerging Trends: Ecosystem, Rollout, and Clone-Relevant Implementations [Confidence: Medium]

### 2.1 Community evidence and product trajectory

Community signals on r/google_antigravity corroborate real-world launched use: "teamwork-preview available to all paid tiers" (rollout completion) [9]; "3 Antigravity teamwork-preview sessions, over 20hrs" (durability of autonomous runs) [10]; "Holy funking shit /teamwork-preview blew my mind" (first-impression reception) [11]; and a persistent counterweight: "Token usage with the new /teamwork-preview command is…" (cost complaints) [17]. Reddit bodies were fetch-blocked (403, captcha) so these rest on titles/metadata — Tier 3, directional only. A forum user reported a **quota-exhaustion interruption during a run using a "custom team configuration"** — the only public hint that team composition may be user-customizable, consistent with `define_subagent`/`manage_subagents` tooling visible locally [6][37]. Rapid secondary coverage appeared within a week of launch (Juejin deep-dive 2026-09-04 [14], explainx "5 Patterns Over Hours/Days" [16], aiprofitboardroom [15], testingcatalog on Threads [12], ccleaks [13]), and at least six YouTube walkthroughs exist but their content could not be scraped in this pass [37].

The premise-correction finding matters strategically: the Antigravity open/closed story is about **Gemini CLI being deprecated and migrated into Antigravity CLI** (official "Migrating from Gemini CLI" docs [18]; practitioner backlash: "Google just deprecated its open-source Gemini CLI, forces users to Antigravity" [19]). Antigravity itself was always closed-line; its *documentation* remained open. Google's public roadmap states "broader plan rollouts [are] scheduled over subsequent weeks" and the corporate post positions Teamwork as a Gemini-stack capability, powered by Gemini 3.7 Flash [2][3].

### 2.2 Comparable open implementations (patterns borrowable for the clone)

Because the *front-of-house* prompt is fully recovered, the reconstruction risk sits entirely on the Phase-2 orchestration side. The open ecosystem supplies validated patterns for every piece:

**Gemini CLI subagents** are the canonical *shipped-Google* open pattern for the definition surface: standalone Markdown files with YAML frontmatter (name, description, allowed tools, system prompt) under `~/.gemini/agents` or `.gemini/agents/`, managed via `/agents`, delegated with `@agent` mentions, parallel fan-out supported "for read-heavy research" with an explicit warning "against parallel heavy code edits" [21] — which independently validates Teamwork's file-ownership isolation design. Antigravity's own customization system mirrors this: skills live at `.agents/skills/<name>/SKILL.md` (workspace) or `~/.gemini/config/` (global), priority workspace > declared `skills.json` > global > built-in, progressive disclosure — and a prior local Antigravity session concluded from the injection mechanism that overriding `/teamwork-preview` "via a Custom Skill … at `.agents/skills/teamwork-preview/SKILL.md`" is the supported extension route [34][32].

**Gas Town** (Steve Yegge, open-source `gastownhall/gastown`) is the most complete public "team UX": a named orchestrator you attach to (`gt mayor attach`), ephemeral per-task workers (Polecats), per-rig overseers that detect and revive frozen agents (Witnesses), a global health sweeper (Deacon), a merge gatekeeper running "a Bors-style bisecting queue" (Refinery), and a file/worktree-backed structured task system ("beads" with IDs, grouped into packages, dispatched with `gt sling <bead-id> <rig>`, escalated with `gt escalate -s HIGH "…"`) [23][24][29]. Its `git worktree-based persistent storage for agent work` is an open implementation of exactly the isolation Teamwork documents [23]. **Claude Code agent teams** (official docs) implement the leader-spawns-teammates pattern with a shared task list and inter-agent mailboxes [25], and **claude-flow** demonstrates driving a whole hive from one slash command (`/flow`-style routing) [26][27][28]. A day-in-the-life narrative ("A Day in Gas Town" [30], Maggie Appleton's pattern analysis [31]) rounds out operational detail. None of these publish Teamwork's exact artifact vocabulary, but the structural elements — coordinator handoff, milestone boards as files, adversarial review gates, worktree isolation — are all independently validated patterns, which de-risks the reconstruction [23][25].

## 3. Critical Assessment: What the Clone Cannot Recover, and Where the Original Hurts [Confidence: Medium]

### 3.1 The hard boundary: post-delegation internals

Verified exhaustively (local filesystem sweep across 19 brain transcripts, 38 CLI conversation DBs, AppData bundles; plus web sweeps): **the launched `teamwork_preview` subagent's system prompt does not exist in any retrievable form.** It is injected server-side; the local client holds only the type token in policy blobs ("hidden from the subagents list but can be invoked" is literally true in the client data) [37][33]. Consequently, any clone — including this project's — reproduces Phase 1 with *verbatim fidelity* and Phase 2 only at *specification fidelity*: roles and artifact types are documented [1], patterns and dynamic spawning are described [2], but the per-role prompts, the routing agent's decision procedure, the synthesis-tree mechanics, the interruption/checkpoint semantics, and the success-auditor termination protocol are unrecoverable. There is **no first-hand launched-session transcript published anywhere on the internet** as of this research — the Reddit threads with the richest expected detail ([10][11][17]) are fetch-walled, and the one official walkthrough article (Google Cloud Medium, "Antigravity Teamwork for long-running tasks") returned 403 [8]. A clone that claims Phase-2 behavioral equivalence is overclaiming; the honest claim is *documented-architecture equivalence*.

### 3.2 Failure modes reported for the original

The community-documented downsides are (a) **token consumption** — a dedicated complaint thread exists [17], consistent with 100+-agent phases [32] and hours-to-days autonomous runs [2][10]; (b) **quota-exhaustion mid-run halts with no restoration story** — the forum thread demands "consumption trackers and checkpoint restoration" [6]; (c) **prompt-routing brittleness by design** — routing hinges on exact phrases ("Do not drop or soften it — without an explicit request the task routes to the standard proof pipeline" [32]), which makes user-facing behavior sensitive to Step-1–9 drafting quality; and (d) **documentation opacity**: no introduction date, no flag surface, no per-agent model configuration documented despite the forced-Flash flag evidence [5][33]. Model selection is a hidden axis: the `invoke_subagent` schema exposes `inherit/flash_lite/flash/pro` [33], and `disable-teamwork-forced-flash-model` exists [33], but the mapping of agents to models per phase is undisclosed.

### 3.3 Contradictions surfaced (not resolved silently)

1. *"Closed source ⇒ docs gone" vs. live rich documentation* — resolved against the premise (§1.4, §2.1): documentation exists [1][2][4][5]; the merge direction was Gemini CLI → Antigravity [18][19].
2. *"Shared workspace" shorthand vs. documented isolation model* — the docs' own wording is inconsistent (one "shared isolated workspace" reading corrected: exclusive file ownership, private scratch dirs, per-subagent working directories) [1] (flagged PARTIAL in verification).
3. `development` integrity mode being *default* — stated in Tier-1 docs extraction [1] but the fetched excerpt could not re-confirm the word "default" (minor, kept at High because the verbatim local prompt says "Default: development" [32]).
4. June vs. August introduction — treated as two-stage rollout, candidate-earliest vs. confirmed-public-launch [6][2].
5. Third-party inventories (e.g., addyosmani/agent-skills #445 claiming "8 lifecycle slash commands (/spec …)" [35]; aibuilderclub's command list [36]) conflict with the official CLI reference table and should not be trusted for command inventory [4].

## 4. Action Plan

- [x] Recover official spec: docs, CLI reference, launch blog, changelog pages captured (sources [1]–[5]).
- [x] Recover the `<TEAMWORK>` Phase-1 prompt and CLI internals from local runtime data for behavioral verification [32][33] (full extractions kept private — proprietary).
- [x] Verify high-impact claims via independent citation check (9/10 SUPPORTED; 1 corrected) (§3.3).
- [ ] Invoke `skill-creator` to build the clone skill: `teamwork-preview` SKILL.md reproducing the Phase 1 structure (artifact, Steps 1–9, integrity-mode mapping, routing/opt-in phrases, validation, delegation gate) adapted to the host runtime's agent tool (Qoder `Agent` tool as `invoke_subagent` equivalent).
- [ ] Implement Phase 2 in the clone as documented roles on runtime subagents: Orchestrator → Explorer(s) → Worker(s) → Critic → Challenger → Auditor/Success-Auditor, with file-based request/plan/progress artifacts under a `teamwork_projects/{name}` layout and exclusive file-ownership rules [1].
- [ ] Encode the five routing team-shapes as a decision table keyed on the same natural-language signals ("keep it small and focused", "very large team of agents", document-to-review detection) [2][32].
- [ ] Add integrity-mode semantics (`development`/`demo`/`benchmark`) as enforced auditor rubrics in the clone [32][1].
- [ ] Record the clone's fidelity boundary explicitly in the skill docs: public clone = documentation-equivalent in both phases (verified behaviorally against the original); internals = unrecoverable (§3.1).
- [ ] Optional elevation (out of this report's scope): read the two Reddit threads and the Medium walkthrough manually in a browser [8][10][11]; run `yt-dlp` on the YouTube candidates for launched-UI detail [37]; binary strings-extraction of the AGY executable for the hidden subagent prompt [37].
- [ ] Save a user memory noting the premise correction (Antigravity docs are public; Gemini CLI is what was deprecated into AGY) so future research doesn't re-assume doc-scarcity.

## 5. Open Questions & Caveats

1. **What exactly does the launched team do?** The internal role prompts and synthesis mechanics remain server-side; only architecture-level documentation exists (§3.1). Highest-value recovery paths: yt-dlp on the six YouTube walkthroughs; manual Reddit reads; binary strings extraction.
2. **Is team composition user-customizable?** One forum data point ("custom team configuration") suggests yes, and the subagent registry shares an enum with user-defined types, but no documentation confirms it [6][33].
3. **Interruption/resume/checkpoint semantics during multi-hour runs** — undocumented; the quota-halt complaints imply weak story here [6].
4. **Model-per-agent policy** — forced-Flash flag and model enum exist in CLI data, mapping unknown [33].
5. **Does the CLI surface render Phase-1 identically to the IDE?** The two-phase prompt was found only in IDE brain transcripts; the CLI DBs carry the recommender line and registry but no conversation containing the crafting prompt — either no CLI session ran the command locally, or CLI injection differs [33][37]. Unresolved.
6. **Reddit/Medium bodies** were fetch-walled this pass; their claims are title-level (Tier 3) and directional only.
7. The changelog-window bug (no introduction entry) and the June/August duality leave the precise launch history partly reconstructed rather than documented [5][6].

## Methodology

Depth: **deep**. Subagents launched: 7 — 3 web Retrieval (Areas 1-2 / Area 3 / Areas 4-5, waves: Wave 1), 1 local-mining (Tier-1 prompt recovery), 1 Gap-Fill (introduction date, blog body, Gas Town mechanics; Wave 2), 1 local extraction (runtime prompt → private verification copy, MD5-checked), 1 Verification (10-claim spot-check), plus 1 final launched-session evidence Gap-Fill (Wave 3). Waves run: 3. Merge protocol applied: subagent indices (1-7, 20-27, 40-47, 60-64, 80-88) renumbered sequentially to 1-38, deduplicated by URL (docs/teamwork, blog, changelog, forum thread each appeared in 2+ agents; richest extract kept, all references updated). Contradiction handling: "shared workspace" reconciled as a correction (§3.3-2); introduction-date duality kept as finding (§1.4). Outline adaptation (Phase 3.5): the evidence pushed the report from a trends shape toward a specification-archive + blueprint; sections 1–3 were refitted to spec/timeline-ecosystem/limits while preserving the mandated skeleton (~30% structural change, within deep-mode limit). Citation corrections from Phase 3.1: claim 2 reformulated (isolation, not sharing); claim 4 recast as bug-fix evidence; "development default" downgraded one notch in-text then re-anchored via local verbatim source [32]. Phase 4 critique triggered one extra Wave-3 subagent (launched-session evidence hunt), which returned a *negative-existence* result now recorded as §3.1/§5-1. Caveats: Reddit thread bodies and one Medium article unretrievable (403/captcha) — flagged per source; YouTube metadata unretrievable; local extraction verified byte-identity across four transcripts but the prior agent's quoted MD5 (`3a0893d6…`) could not be reproduced — authoritative value here is `aec40585…` computed at extraction.

## Bibliography

[1] Google Antigravity Docs — "Teamwork agent teams (/teamwork-preview)" — https://antigravity.google/docs/teamwork/ — Accessed 2026-09-25 — Tier: 1
[2] The Antigravity Team — "Teamwork: When AI Becomes a Research Partner" — https://antigravity.google/blog/teamwork-when-ai-becomes-a-research-partner — 2026-08-27 — Tier: 1
[3] Google — "Gemini Multi-Agent Teams in Antigravity" — https://blog.google/innovation-and-ai/technology/developers-tools/antigravity-teamwork-multi-agent/ — 2026-08-31 — Tier: 1 (body partially truncated at fetch)
[4] Google Antigravity Docs — "CLI Reference" — https://antigravity.google/docs/cli/reference/ — Accessed 2026-09-25 — Tier: 1
[5] Google Antigravity — "Changelog" — https://antigravity.google/changelog/ — window 2026-05-19…09-23 — Tier: 1
[6] Google AI Developers Forum — "Teamwork Preview" (topic 170693) — https://discuss.ai.google.dev/t/teamwork-preview/170693 — 2026-06-11 — Tier: 2
[7] Google Antigravity Docs — "Custom Subagents" / "Agent Skills" — https://antigravity.google/docs/subagents/ , https://antigravity.google/docs/skills/ — Tier: 1 (surfaced via search index)
[8] Medium/Google Cloud — "Antigravity Teamwork for long-running tasks" — https://medium.com/google-cloud/antigravity-teamwork-for-long-running-tasks-de74825a6ae9 — Tier: 2 [snippet only — 403]
[9] r/google_antigravity — "teamwork-preview available to all paid tiers" — https://www.reddit.com/r/google_antigravity/comments/1twyqow/ — Tier: 3 [title only]
[10] r/google_antigravity — "3 Antigravity teamwork-preview sessions, over 20hrs" — https://www.reddit.com/r/google_antigravity/comments/1uh6bsr/ — Tier: 3 [title only]
[11] r/google_antigravity — "Holy funking shit /teamwork-preview blew my mind" — https://www.reddit.com/r/google_antigravity/comments/1tuqq67/ — Tier: 3 [title only]
[12] testingcatalog (Threads) — "Antigravity now supports Agent Teams under /teamwork-preview" — https://www.threads.com/@testingcatalog/post/DavtKg1DXig/ — ~2026-08 — Tier: 3
[13] ccleaks — "Antigravity Teamwork is available as /teamwork-preview" — https://ccleaks.com/news/gemini-antigravity-teamwork-multi-agent-aug-2026 — 2026-08 — Tier: 3 [search-index only]
[14] Juejin — "当AI不再单打独斗：Google Antigravity Teamwork 深度解读" — https://juejin.cn/post/7681476107401363465 — 2026-09-04 — Tier: 3 [search-index only]
[15] aiprofitboardroom — "Google Antigravity Teamwork: Agent Teams Explained" — https://aiprofitboardroom.com/blog/google-antigravity-teamwork/ — 2026-08-30 — Tier: 3 [search-index only]
[16] explainx.ai — "Antigravity Teamwork: 5 Patterns Over Hours/Days" — https://explainx.ai/blog/google-antigravity-teamwork-multi-agent-framework-august-2026 — 2026-08-27 — Tier: 3 [search-index only]
[17] r/google_antigravity — "Token usage with the new /teamwork-preview command is…" — https://www.reddit.com/r/google_antigravity/comments/1uash5h/ — Tier: 3 [title only]
[18] Google Antigravity Docs — "Migrating from Gemini CLI" — https://antigravity.google/docs/cli/gcli-migration/ — Tier: 1 [existence]
[19] Joshua Vial (LinkedIn) — "Google just deprecated its open-source Gemini CLI, forces users to Antigravity" — https://www.linkedin.com/posts/joshuavial_7465590184013647873-wwPT — 2026 — Tier: 3
[20] Google Antigravity team — antigravity-cli CHANGELOG.md — https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md — Tier: 1
[21] Jack Wotherspoon & Abhi Patel (Google Developers Blog) — "Subagents have arrived in Gemini CLI" — https://developers.googleblog.com/subagents-have-arrived-in-gemini-cli/ — 2026-04-15 — Tier: 1 [foundational for open subagent pattern]
[22] Steve Yegge — "Welcome to Gas Town" — https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04 — 2026-01 — Tier: 2
[23] gastownhall — gastown README (raw main) — https://github.com/gastownhall/gastown — Accessed 2026-09-25 — Tier: 1 (repo)
[24] steveyegge — gastown role templates (`internal/templates/roles/mayor.md.tmpl`) — https://github.com/steveyegge/gastown/blob/main/internal/templates/roles/mayor.md.tmpl — Tier: 3
[25] Anthropic — "Orchestrate teams of Claude Code sessions" — https://code.claude.com/docs/en/agent-teams — Tier: 1
[26] Analytics Vidhya — "Claude Flow: Orchestration Framework for Multi-Agent Systems" (ruvnet/claude-flow) — https://www.analyticsvidhya.com/blog/2026/03/claude-flow/ — 2026-03-10 — Tier: 2
[27] r/ClaudeAI — "How I Built a Multi-Agent Orchestration System with Claude Code" — https://www.reddit.com/r/ClaudeAI/comments/1l11fo2/ — Tier: 3
[28] dev.to — "Multi-Agent Orchestration: Running 10+ Claude Instances in Parallel" — 2025-08-02 — Tier: 3
[29] heise.de — "Full Control — Gas Town Orchestrates Ten or More Coding Agents" — 2026 — Tier: 2
[30] DoltHub — "A Day in Gas Town" — 2026-01-15 — Tier: 2
[31] Maggie Appleton — guest pattern analysis on Gas Town — https://maggieappleton.com/gastown — 2026-01-23 — Tier: 2
[32] Google Antigravity (shipped runtime, verified locally) — `<TEAMWORK>` slash-command prompt block, ~2,400 words — extracted 2026-09-25 from `~/.gemini/antigravity/brain/<session-id>/.system_generated/logs/transcript_full.jsonl` (byte-identical across four sessions) — **proprietary Google material; held privately, not redistributed in this repo** — Tier: 1 (primary artifact, cited by structure only)
[33] Google Antigravity CLI (shipped runtime, verified locally) — command recommender table, subagent-type registry, `invoke_subagent` schema, feature-flag list — extracted from `~/.gemini/antigravity-cli/conversations/*.db` (SQLite/protobuf strings) — factual API surface only as quoted above; bulk strings not redistributed — Tier: 1
[34] Shipped skill documentation (local) — `agy-customizations/SKILL.md` (`.agents/skills` layout, discovery roots & priority) — `~/.qoder/skills/agy-customizations/SKILL.md` — Tier: 1
[35] addyosmani/agent-skills — issue #445 (lifecycle slash commands claim) — https://github.com/addyosmani/agent-skills — 2026-08-01 — Tier: 3 (conflicts with [4]; flagged unreliable)
[36] aibuilderclub.com — "Antigravity CLI (agy): Commands, Modes, and Auto-Approve" — 2026-06-21 — Tier: 3
[37] This research (Wave-3 evidence sweep) — negative-existence finding: no launched `teamwork_preview` execution record locally; no public launched transcript; YouTube candidates: https://www.youtube.com/watch?v=m5I0JrFq-Lk , Eyobn1yUfN0 , 3cdVb5CD0jc , yNh-KA_2tyM , Nu-QqCVGMIM , 2On3OkCTwhc — Tier: 2 (methodologically verified absence)
[38] Facebook (ch.nemri photo post) — "Explorer agents generate diverse candidate strategies…" — https://www.facebook.com/ch.nemri/photos/1091821303361422/ — Tier: 3

## Source Extracts

### [1] Teamwork agent teams (/teamwork-preview) — official docs
- **Summary:** Canonical spec: paid-plan availability on AGY 2.0 + CLI; two-phase flow (scoping interview → autonomous execution with structured handoffs); three-tier team organization; full role roster; `~/teamwork_projects/{PROJECT_NAME}` default with exclusive file ownership and per-subagent scratch dirs; four artifact types; integrity profiles; Alt+J approval navigation; prompt-driven team sizing.
- **Key quotes:** "available on paid plans across Google Antigravity 2.0 and the Antigravity CLI"; "Each subagent has its own working directory"; Sentinel "takes over once you approve the prompt"; Challenger "builds adversarial test suites"; "spawn fresh successors between stages to preserve context."
- **Source type:** docs — **Tier:** 1

### [2] Teamwork: When AI Becomes a Research Partner — launch blog
- **Summary:** Pattern-selection engine; five collaboration patterns; proposer/falsifier/synthesizer mechanics; dynamic mid-run agent spawning; human retains objectives and final acceptance; paid-plans availability "today."
- **Key quotes:** "propose, critique, and refine each other's work autonomously over hours or days"; "The framework dynamically decides how many agents to spawn based on task requirements"; "a falsifier whose sole job is to break it"; "A synthesis tree then combines the candidates and their reports."
- **Source type:** industry/official blog — **Tier:** 1

### [4] CLI Reference
- **Summary:** Full slash-command matrix (30+ commands incl. `/agents`, `/skills`, `/mcp`, `/hooks`, `/boost`, `/planning`); `/teamwork-preview` with alias `/teamwork`; extensibility deliberately outside autocomplete.
- **Key quotes:** "Extensible features do not automatically inject themselves into the autocomplete list."; `/teamwork-preview` "Launch collaborative multi-agent teams… (paid plans)".
- **Source type:** docs — **Tier:** 1

### [5] Antigravity Changelog
- **Summary:** No introduction entry for teamwork; v2.5.0 (2026-07-31) bug fix confirms teamwork as built-in helper; v2.12.0 (2026-09-02) introduces `/boost` multi-agent reasoning command.
- **Key quotes:** "research, browser and teamwork assistants work again"; "Introduced the /boost slash command."
- **Source type:** docs — **Tier:** 1

### [32] `<TEAMWORK>` prompt (local runtime verification)
- **Summary:** Complete Phase-1 specification injected client-side by the platform: two-phase mandate; `prompt_draft.md` artifact; 4 core principles; Steps 1-9 with opt-in scale phrases, integrity-mode question→mode mapping, verification forcing-function doctrine, acceptance-criteria bars, infra constraints, working-dir rule, assembly + validation; prohibitions; routing table with 5 team shapes; delegation protocol via hidden `teamwork_preview` subagent. (Full text proprietary — summarized here, not redistributed.)
- **Key quotes:** "Both phases are required — crafting without delegation is incomplete."; "teamwork_preview is hidden from the subagents list but can be invoked."
- **Source type:** shipped runtime prompt (primary artifact) — **Tier:** 1

### [33] CLI teamwork strings (local recovery)
- **Summary:** Recommender table (8 commands, incl. the teamwork-preview trigger description); subagent enum (`self, research, browser, teamwork_preview, DeepCoder, DeepInvestigator`); `invoke_subagent` schema with model enum `inherit/flash_lite/flash/pro` and workspace modes `inherit/branch/share`; flags `enable-teamwork-subagent`, `disable-teamwork-forced-flash-model`.
- **Key quotes:** "Recommend this when the user has a large project that would benefit from a team of autonomous agents working together."
- **Source type:** shipped runtime data (primary artifact) — **Tier:** 1

### [6] Forum: Teamwork Preview
- **Summary:** Earliest dated footprint 2026-06-11; reports quota-exhaustion halt with "custom team configuration"; demands consumption trackers + checkpoint restoration.
- **Source type:** community/official forum — **Tier:** 2

### [20]+[21]+[23]+[25] Open analogs
- **Summary:** Gemini CLI subagents (markdown+frontmatter agent files, `@agent` delegation, parallel-edit caution); Gas Town (`gt` command surface, beads task board, Witness/Deacon/Refinery gates, worktree persistence, escalation syntax); Claude Code agent teams (leader spawn, shared task list, mailboxes); claude-flow (single-command hive routing).
- **Key quotes:** "Exercise caution with parallel subagents for tasks that require heavy code edits" [21]; "Bors-style bisecting queue" [23].
- **Source type:** docs/repos — **Tier:** 1/2

### [9][10][11][17] Reddit threads
- **Summary:** Rollout completion, 20+ hour sessions, enthusiastic reception, token-cost complaints. Titles/metadata only (fetch-walled) — corroboration tier.
- **Source type:** community — **Tier:** 3
