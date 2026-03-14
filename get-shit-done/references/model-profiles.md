# Model Profiles

Model profiles control which Claude model each GSD agent uses. This allows balancing quality vs token spend.

## Profile Definitions

| Agent | `deep` | `quality` | `balanced` | `budget` |
|-------|--------|-----------|------------|----------|
| gsd-planner | opus | opus | opus | sonnet |
| gsd-roadmapper | opus | opus | sonnet | sonnet |
| gsd-executor | opus | opus | opus | sonnet |
| gsd-phase-researcher | opus | opus | sonnet | haiku |
| gsd-project-researcher | opus | opus | sonnet | haiku |
| gsd-research-synthesizer | opus | sonnet | sonnet | haiku |
| gsd-debugger | opus | opus | sonnet | sonnet |
| gsd-codebase-mapper | opus | sonnet | haiku | haiku |
| gsd-verifier | opus | sonnet | sonnet | haiku |
| gsd-plan-checker | opus | sonnet | sonnet | haiku |
| gsd-integration-checker | opus | sonnet | sonnet | haiku |
| gsd-nyquist-auditor | opus | sonnet | sonnet | haiku |

## Profile Philosophy

**deep** - Maximum capability with 1M context
- Opus for ALL agents — no model compromise anywhere
- Designed for 1M context windows where depth matters
- Max effort on every agent — comprehensive research, thorough planning, deep execution
- Use when: 1M context available, complex architecture, critical work requiring zero shortcuts
- All agents run with full codebase awareness and comprehensive file reading

**quality** - Maximum reasoning power
- Opus for all decision-making agents
- Sonnet for read-only verification
- Use when: quota available, critical architecture work

**balanced** (default) - Smart allocation
- Opus for planning and execution (1M context makes this affordable)
- Sonnet for research and verification
- Sonnet for execution and research (follows explicit instructions)
- Sonnet for verification (needs reasoning, not just pattern matching)
- Use when: normal development, good balance of quality and cost

**budget** - Minimal Opus usage
- Sonnet for anything that writes code
- Haiku for research and verification
- Use when: conserving quota, high-volume work, less critical phases

## Resolution Logic

Orchestrators resolve model before spawning:

```
1. Read .planning/config.json
2. Check model_overrides for agent-specific override
3. If no override, look up agent in profile table
4. Pass model parameter to Task call
```

## Per-Agent Overrides

Override specific agents without changing the entire profile:

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "opus",
    "gsd-planner": "haiku"
  }
}
```

Overrides take precedence over the profile. Valid values: `opus`, `sonnet`, `haiku`.

## Switching Profiles

Runtime: `/gsd:set-profile <profile>`

Per-project default: Set in `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Design Rationale

**Why Opus for gsd-planner?**
Planning involves architecture decisions, goal decomposition, and task design. This is where model quality has the highest impact.

**Why Sonnet for gsd-executor?**
Executors follow explicit PLAN.md instructions. The plan already contains the reasoning; execution is implementation.

**Why Sonnet (not Haiku) for verifiers in balanced?**
Verification requires goal-backward reasoning - checking if code *delivers* what the phase promised, not just pattern matching. Sonnet handles this well; Haiku may miss subtle gaps.

**Why Haiku for gsd-codebase-mapper?**
Read-only exploration and pattern extraction. No reasoning required, just structured output from file contents.

**Why `inherit` instead of passing `opus` directly?**
Claude Code's `"opus"` alias maps to a specific model version. Organizations may block older opus versions while allowing newer ones. GSD returns `"inherit"` for opus-tier agents, causing them to use whatever opus version the user has configured in their session. This avoids version conflicts and silent fallbacks to Sonnet.

## 1M Context Guidelines

When using `deep` or `quality` profiles with 1M context:
- Agents should read comprehensively before acting (50+ files is normal)
- Analysis paralysis guard is relaxed (15+ reads without action)
- Plans can have 5-8 tasks (up from 2-3)
- Plans can modify 20-35 files (up from 5-8)
- Parallel researchers: 6-8 (up from 4)
- Research depth: comprehensive multi-source verification
- All agents use max effort for reasoning
