# Claude Code + Codex Agent Configuration System

[![OKF v0.1 compliant](https://img.shields.io/badge/OKF-v0.1%20compliant-4285F4)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)

A practical configuration kit for Claude Code, Codex, and other coding agents. It contains architectural principles, enforcement hooks, skills, drop-in rules, starter templates, and dynamic-workflow commands. Drop the relevant parts into a project so the agent starts from verified working patterns instead of rediscovering them every session.

This is not a collection of tips. It is a **system** that teaches your agent *how to work* - when to use one agent vs many, how to verify its own output, how to manage context across long sessions, how to not get poisoned by malicious packages.

## Notes

Write-ups of the incidents that produced a rule or a hook here. Each states what was measured,
what is inference, and what the fix does not cover.

| | |
|---|---|
| **[Starting from what you remember](docs/starting-from-what-you-remember.md)** | Why a brand-new project arrives years out of date, why an invented package name is now a security problem rather than a 404, and where manifest and install-time checks have to sit to catch either. Ships as [`dependency-currency-guard.py`](hooks/dependency-currency-guard.py) + [`dependency-provenance-guard.py`](hooks/dependency-provenance-guard.py). |
| **[Why an agent circles instead of acting](docs/why-agents-circle-instead-of-acting.md)** | Two agents, identical rules, different gate shapes. Describing a fix instead of applying it turned out to be the only move with no gate on it. |
| **[Gates that cannot bootstrap themselves](principles/30-gates-that-cannot-bootstrap.md)** | A check that only arms once the thing it checks for already exists will never arm. The failure looks exactly like compliance. |
| **[Nine skills, one skeleton, and nobody reaching for them](docs/skills-organised-by-author.md)** | Nine architecture skills existed and one was reachable — the one arguing for less code. How a one-sided advisory becomes a ratchet toward monoliths, and why filing knowledge by source book makes it unreachable. |
| **[The form was available, so it was taken for the content](docs/form-mistaken-for-content.md)** | One failure shape in six materials — including three times inside the tool built to catch it. What formal verification, pytest, ESLint and mutation testing each already answer, and why "empty" has to be its own outcome. |
| **[A launch is a promise to look at it](docs/a-launch-is-a-promise.md)** | A job that died in its first second looks exactly like one running quietly, and "is it running" is three independent questions of which liveness is only the first. 2,958 launches measured, 42 never checked at all. Ships as [`hooks/launch-watch-guard.py`](hooks/launch-watch-guard.py). |
| **[The deferral moves to whichever form is not guarded](docs/finishing-what-you-started.md)** | Who has already built "keep going until it is done", what it cost when it looped, and why 27 of 51 open tickets here carry the same one of five legitimate reasons. Ships as the today's-tickets gate in [`hooks/handoff-closure-audit-guard.py`](hooks/handoff-closure-audit-guard.py). |
| **[A passing test is not a release](docs/a-passing-test-is-not-a-release.md)** | An unavailable signer, VM, or account should block only its own stage, not erase proof of an unchanged parent. Introduces `VERIFIED / SEALED / BLOCKED / SUPERSEDED`, a minimal stage ledger, and the boundary where the ledger would just become bureaucracy. Ships as [`proof-verify`](skills/development/proof-verify/SKILL.md) + [`hooks/plan-gate.py`](hooks/plan-gate.py). |
| **[A save log is not a retention guarantee](docs/a-save-log-is-not-a-retention-guarantee.md)** | Why a handler return or message journal cannot prove photo/document preservation; defines the live read-after-write, cleanup, and restart receipt that must exist before destructive cleanup. |
| **[Runtime wiring and hook lifecycle](docs/runtime-wiring.md)** | Where request intake, safety guards, verification, compaction, and close-out belong; how continuity stays durable without putting archive/index work on every prompt. |

---

## Installation

Three paths depending on what you need:

### Option 1: Claude Code plugin (fastest)

```bash
claude plugin install https://github.com/AnastasiyaW/claude-code-config
```

Then in your Claude Code chat:

```
Read AGENTS.md and pick the principles, hooks, and skills that match my project.
```

### Option 2: Global install (hooks + skills available in every project)

```bash
git clone https://github.com/AnastasiyaW/claude-code-config ~/claude-code-config

# Copy the always-on safety hooks to your global config
python ~/claude-code-config/scripts/install_hooks.py --global

# Claude Code: copy a selected skill directory, not its parent category
mkdir -p ~/.claude/skills
cp -r ~/claude-code-config/skills/ai-ml/ml-research-lab ~/.claude/skills/

# Codex desktop, its shared ~/.agents root, and Claude Code: sync public skills
# with independent backups for changed local copies. Target-only skills remain intact.
python ~/claude-code-config/scripts/sync_skills_to_codex.py --apply --also-claude --also-agents
```

The canonical tracked checkout's `hooks/` directory stores the global hook scripts;
`~/.claude/settings.json` and `~/.codex/hooks.json` register that exact source.
The installer must not create a second active `~/.claude/hooks/` tree; it merges
safe defaults into the existing manifests and keeps backups before changes.

### Option 3: Project-local (hooks/skills only in this project)

```bash
cd /your/project
git clone https://github.com/AnastasiyaW/claude-code-config .claude-config
python .claude-config/scripts/install_hooks.py --local
cp -r .claude-config/skills .claude/skills
```

This keeps everything under `.claude/` in your repo, nothing global.

### Choosing what to install

| Project type | Minimum viable set |
|---|---|
| **Any project** | 5 safety hooks (destructive-command, secret-leak, git-destructive, git-auto-backup, session-drift-validator) + Principles 09 (Supply Chain), 10 (Agent Security), 11 (Documentation Integrity) |
| **Web app** | above + `frontend-design` skill + Principles 04 (Deterministic Orchestration), 05 (Structured Reasoning) |
| **ML / data pipeline** | above + `flux2-*`, `diffusion-engineering`, `vlm-segmentation` skills + Principles 03 (Autoresearch), 12 (Low-Signal Training) |
| **Multi-agent / parallel sessions** | above + [mclaude](https://github.com/AnastasiyaW/mclaude) + Principles 01 (Harness), 06 (Multi-Agent), 18 (Multi-Session Coordination), 19 (Inter-Agent Communication) |
| **Library / package** | above + Principles 08 (Skills Best Practices), 17 (DBS Skill Creation) |
| **More than one CLI agent (Claude + Gemini / Codex)** | above + [rules/cross-harness-agents-md.md](rules/cross-harness-agents-md.md) (one `AGENTS.md` per project, no symlinks) + `gemini-delegate` skill |

See [AGENTS.md](AGENTS.md) for the procedure an agent follows after install,
[HOW-IT-WORKS.md](HOW-IT-WORKS.md) for the mechanics of each layer, and
[docs/runtime-wiring.md](docs/runtime-wiring.md) for the live verification contract.

Moved the config to a new machine or account and most skills stopped being
offered? Nothing raises an error when that happens — see
[docs/skill-tree-recovery.md](docs/skill-tree-recovery.md) and run
`python scripts/recover_skill_trees.py --report`.

---

## What This Gives You

**Architectural Principles** - each one prevents a specific failure mode observed in real agent workflows:

- **Self-evaluation bias?** Separate Generator and Evaluator agents ([Harness Design](principles/01-harness-design.md))
- **Agent claims "done" but it's broken?** Require durable proof artifacts ([Proof Loop](principles/02-proof-loop.md))
- **Tests feel repetitive or a specialized gate blocks smoke?** Use the universal candidate-state sequence: focused slice, risk-based review, one full matrix, then only the relevant immutable-candidate compatibility proof ([testing strategy](docs/research/2026-08-testing-and-agent-evals.md))
- **Need to improve a prompt/skill/config?** Automated Read-Change-Test loop ([Autoresearch](principles/03-autoresearch.md))
- **LLM skips steps in complex workflows?** Shell scripts for mechanical tasks, one step at a time ([Deterministic Orchestration](principles/04-deterministic-orchestration.md))
- **Wrong debugging conclusions?** Structured Premises-Trace-Conclusions format ([Structured Reasoning](principles/05-structured-reasoning.md))
- **Task too big for one agent?** Coordinator + specialized sub-agents ([Multi-Agent Decomposition](principles/06-multi-agent-decomposition.md))
- **Context degrades in long sessions?** Treat CLAUDE.md as runtime config, not docs ([Codified Context](principles/07-codified-context.md))
- **Supply chain attack?** Two config lines block packages younger than 7 days ([Supply Chain Defense](principles/09-supply-chain-defense.md))
- **Prompt injection via repo/MCP/web?** Six-layer defense with real CVEs ([Agent Security](principles/10-agent-security.md))
- **Docs reference files that no longer exist?** SessionStart hook validates every reference ([Documentation Integrity](principles/11-documentation-integrity.md)) - ships with a working validator script
- **Multi-agent infrastructure overhead?** Separate brain from hands with lazy provisioning ([Managed Agents](principles/14-managed-agents.md))
- **Agent cuts corners on critical rules?** Absolute prohibitions with incident history ([Red Lines](principles/15-red-lines.md))
- **Long-running project lost its history?** Condensed timeline per project, alongside handoffs ([Project Chronicles](principles/16-project-chronicles.md))
- **Skill is a monolithic wall of text?** Split into Direction, Blueprints, Solutions ([DBS Framework](principles/17-dbs-skill-creation.md))
- **Parallel chats fight over GPUs or overwrite each other's state?** Append-only handoffs + lock-file coordination ([Multi-Session Coordination](principles/18-multi-session-coordination.md))
- **One chat needs to send a specific request to another?** File-based mailbox with email-style threading and delivery receipts ([Inter-Agent Communication](principles/19-inter-agent-communication.md))
- **AI-assisted code review findings get rediscovered next PR?** Review finding → regression test → invariant → cross-reference ([Knowledge Base Enforcement](principles/21-knowledge-base-enforcement.md))
- **Zero-day vulnerabilities buried in source tree?** LLM + rules + SAST pipeline ([Vulnerability Detection Pipeline](principles/20-vulnerability-detection-pipeline.md))
- **User needs to choose between visual options (UI, design, diagrams)?** HTML fragment server + file-based event queue ([Visual Context Pattern](principles/22-visual-context-pattern.md))
- **Output keeps reverting to generic defaults (Inter font, SELECT *, etc.)?** Anti-attractor procedure + three-layer enforcement ([Anti-pattern as Config](principles/23-anti-pattern-as-config.md))
- **Merge conflict resolved "by logic" and lost half the work?** Two-agent isolated reconciliation + verified-data priority ([Merge Conflict Resolution](principles/24-merge-conflict-resolution.md))
- **Built a coordination primitive from scratch?** Map it to the classical analog first (Chubby lease, WAL, SMTP) and inherit 30 years of failure-mode literature ([Coordination Primitives Mapping](principles/25-coordination-primitives-mapping.md))
- **Bug fix detoured into "this was already broken before me"?** Five valid deferral reasons + mandatory durable proof artifacts ([No-Pre-Existing Evasion](principles/26-no-pre-existing-evasion.md))
- **Long-run project's scope and progress scattered across 30+ handoffs?** Three-artifact harness (PROBLEMS.md + feature_list.json + init.sh) with WIP=1 invariant and L1/L2/L3 evidence requirements ([Feature Tracking](principles/27-feature-tracking.md))
- **Feature rationale evaporates into git log after 6 weeks?** Three-tier KB (Global -> Layer -> Feature narrative) with ULTRAPACK-style task.md, auto-allocated F-NNN ID, hyperlinked invariants ([Feature-Layer Architecture](principles/28-feature-layer-architecture.md))
- **Model collapses to "predict zero" on residual/delta tasks?** Traps and fixes for low-signal training (overlay maps, denoise deltas, color-correction residuals), from 4 rounds of real failure ([Low-Signal Residual Training](principles/12-low-signal-residual-training.md))
- **Deep research results evaporate with the conversation?** Save structured findings to an incoming folder -> review -> knowledge base pipeline ([Research Pipeline](principles/13-research-pipeline.md))
- **Need a human-browsable memory view without a second source of truth?** Use an optional Obsidian-compatible Markdown hub over the private archive ([Obsidian Mind adoption note](docs/research/obsidian-mind-adoption-2026-07-28.md))
- **Need a repeatable claim check or local UI/CLI harness?** Use the selectively adopted [Cursor Team Kit patterns](docs/research/2026-08-cursor-team-kit-adoption.md): `verify-this`, `control-cli`, `control-ui`, `deslop`, and opt-in strict quality review.
- **Building a brand-new agent and not sure what to decide first?** 15-section MVP blueprint: autonomy level -> tool risk classes -> permission matrix -> budgets -> evals -> release checklist ([MVP Agent Blueprint](principles/29-mvp-agent-blueprint.md))

**Need smaller diagnostic command output?** The optional RTK integration is
pinned, checksum-verified, fail-open, and tested separately from safety hooks.
See [docs/rtk-integration.md](docs/rtk-integration.md) and
`scripts/rtk_integration.py`; it is never a substitute for raw evidence.

**Ready-to-use hooks** that enforce rules mechanically, not probabilistically (install via [scripts/install_hooks.py](scripts/install_hooks.py); full map with bypass keys in [rules/safety-hooks.md](rules/safety-hooks.md)):

| Hook | Event | What It Does |
|---|---|---|
| [session-drift-validator](hooks/session-drift-validator.py) | `SessionStart` | Validates file references in CLAUDE.md at session start |
| [destructive-command-guard](hooks/destructive-command-guard.py) | `PreToolUse` | Blocks `rm -rf`, `git push --force`, `DROP TABLE` |
| [secret-leak-guard](hooks/secret-leak-guard.py) | `PreToolUse` | Prevents committing API keys, tokens, passwords |
| [session-handoff-reminder](hooks/session-handoff-reminder.py) | `Stop` | Reminds to write handoff before closing long sessions |
| [session-handoff-check](hooks/session-handoff-check.py) | `SessionStart` | Shows recent handoffs from previous sessions (latest per project) |
| [handoff-closure-audit-guard](hooks/handoff-closure-audit-guard.py) | `PreToolUse` | Blocks handoff writes that lack a closure audit for the primary task and related/scope-adjacent tasks |
| [stop-phrase-guard](hooks/stop-phrase-guard.py) | `Stop` | Detects behavioral-regression phrases (ownership dodging, permission-seeking, premature stopping, deferral-via-"what next?") |
| [keyword-skill-router](hooks/keyword-skill-router.py) | `UserPromptSubmit` | Detects natural-language keywords and suggests matching skills (bilingual RU/EN) |
| [api-key-leak-detector](hooks/api-key-leak-detector.py) | `PostToolUse` | Scans tool output for exposed API keys, tokens, secrets |
| [command-injection-guard](hooks/command-injection-guard.py) | `PreToolUse` | Blocks shell substitution with non-trivial commands |
| [git-destructive-guard](hooks/git-destructive-guard.py) | `PreToolUse` | Blocks `git reset --hard`, `push --force`, force branch deletion (`-D`, `-fD`, `-Df`, long flags); allows merged-only `branch -d` |
| [git-auto-backup](hooks/git-auto-backup.py) | `PreToolUse` | Creates backup branch before destructive git operations |
| [self-harm-guard](hooks/self-harm-guard.py) | `PreToolUse` | Prevents agent from killing its own process, locking SSH, bare reboot |
| [test-muting-guard](hooks/test-muting-guard.py) | `PreToolUse` | Blocks adding `@skip`, `.only()`, `@Ignore` to existing tests |
| [backup-retention-cleanup](hooks/backup-retention-cleanup.py) | `Stop` | Cleans up old backup branches (14-day retention) |
| [file-cohesion-guard](hooks/file-cohesion-guard.py) | `PreToolUse` | Advisory: warns when a durable file is written to a scratch location (home root, Desktop, Downloads, /tmp) instead of the project structure |
| [human-confirmation-guard](hooks/human-confirmation-guard.py) | `PreToolUse` | Blocks destructive actions until a host-verifiable approval record can be checked |
| [ask-question-guard](hooks/ask-question-guard.py) | `PreToolUse` | Blocks deferral/menu `AskUserQuestion` ("what next?", "which of these?") on reversible work — decide and proceed instead |
| [over-engineering-advisor](hooks/over-engineering-advisor.py) | `PostToolUse` | Advisory nudge when an edit adds a large code block or a new dependency — "is this the minimal solution?" (never blocks) |
| [module-shape-advisor](hooks/module-shape-advisor.py) | `PostToolUse` | The mirror of the row above: advisory nudge when the whole FILE has outgrown its shape — "where is the seam?" Fires on cumulative size, not on your edit, because that is how a file gets there (never blocks) |
| [dependency-currency-guard](hooks/dependency-currency-guard.py) | `PreToolUse` | Blocks a manifest edit that names a package which does not exist, is too new or too little used to be a real recall (the slopsquat profile), or pins a fast-moving library far behind current |
| [dependency-provenance-guard](hooks/dependency-provenance-guard.py) | `PreToolUse` | Blocks direct wheels/archives/Git sources and extra indexes; requires lock/hash-aware installs, fails closed on registry outages, and checks exact registry versions plus artifact digests |
| [dependency-alternatives](scripts/dependency-alternatives.py) | On demand | Searches official PyPI/npm metadata and returns only stable, age- and digest-verified candidate packages; never edits or installs |
| [pre-push-public-repo-scan](hooks/pre-push-public-repo-scan.py) | git `pre-push` | Two independent scans — regex and semantic — of a push to a PUBLIC repo; either one alarming blocks it. Private repos skip. Host and script names load from a local list, never from this file |
| [shape_common](hooks/shape_common.py) | *(library)* | Not a hook: the one definition of "what shape is this file in", shared by `module-shape-advisor` and `scripts/architecture_audit.py` so the two cannot answer differently |
| [harness-load-advisor](hooks/harness-load-advisor.py) | `Stop` | Notices when a closing message reports a high-cost or specialized gate (signing, VM/GPU/OS/browser/performance) blocking lower-risk work, and says so. A feedback guard, not a bypass — it never lifts the gate |
| [outward-claim-evidence-guard](hooks/outward-claim-evidence-guard.py) | `Stop` | Blocks a narrow set of externally measurable claims (hash, filename-derived hash, size, version, deploy) when the final report lacks a probe/result line. It enforces reporting discipline, not truth by itself. |
| [repeated-attempt-guard](hooks/repeated-attempt-guard.py) | `PreToolUse` + `PostToolUse` | Stops the guess-and-retry loop: advisory on the third failed attempt at the same target, blocking on the fourth, unless something has been read since the last failure. One `Read` clears it — the block is lifted by the action that would have solved it three attempts earlier. Needs **both** events: `PostToolUse` records outcomes, `PreToolUse` decides |
| [launch-watch-guard](hooks/launch-watch-guard.py) | `PostToolUse` + `Stop` | Starting a job is a promise to look at it. Records every launch (`nohup`, detached `docker run`, `sbatch`, `schtasks`, `run_in_background`) and refuses to end the session while one has never been probed — a job that died in its first second looks exactly like one running quietly. One `nvidia-smi`, `docker ps` or `tail` of its 