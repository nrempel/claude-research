# claude-research

Deep research to eliminate all assumptions before implementation.

## What It Does

Normal Claude: "I'll use library X for this..." (assumption)

With `/research`: Verifies every assumption through web search and codebase exploration before proceeding.

## How It Works

1. **Draft approach** - Outline initial plan
2. **Extract assumptions** - Flag everything unverified
3. **Research each** - WebSearch, codebase exploration, documentation
4. **Log findings** - Document evidence in state file
5. **Update approach** - Adapt plan based on findings
6. **Re-scan** - Look for new assumptions, repeat until none remain

## Installation

### Via Marketplace

```
/plugin marketplace add nrempel/claude-plugins
/plugin install research@nrempel-plugins
```

### Local Development

```bash
claude --plugin-dir ~/Projects/claude-research
```

## Usage

```
/research Add authentication to the API
```

### Commands

| Command | Description |
|---------|-------------|
| `/research <task>` | Start new research session |
| `/research` | Continue existing session, or prompt for task |
| `/research status` | Show assumptions and progress |
| `/research cancel` | Stop and clean up |

**To stop mid-research:** Say "stop", "done", or "that's enough".

## Example

```
> /research Add Stripe subscription billing

[Drafts initial approach]
- Use Stripe Billing API
- Webhooks for events
- Store status in DB

[Extracts assumptions]
- [ ] Stripe supports usage-based billing
- [ ] Project has webhook infrastructure
- [ ] DB can store subscription metadata

[Researches each]
✅ Stripe supports metered billing (verified in docs)
❌ No webhook infrastructure exists (searched codebase)
✅ Postgres with Drizzle - can add columns

[Updates approach based on findings]
Need to add webhook endpoint - wasn't in original plan

[Re-scans for new assumptions]
- [ ] Next.js API routes can handle webhooks
→ Researches, verifies

## Research Complete

Verified approach with 0 assumptions:
1. Use Stripe metered billing API
2. Create /api/webhooks/stripe endpoint (NEW)
3. Add subscription_id column to users table
...
```

## Compaction-Safe

This plugin survives context compaction:

**State file:** Creates `.claude/research/<branch>-<timestamp>.md` containing:
- Original goal
- Current approach
- Assumptions (verified/unverified/refuted)
- Research log with findings

**Resume:** Run `/research` again after compaction. It will:
1. Find the state file
2. Read the goal and progress
3. Continue from unverified assumptions

**Cleanup:** State file is deleted when research completes.

### Gitignore

Add to your `.gitignore`:
```
.claude/research/
```

## What Gets Verified

| Type | How |
|------|-----|
| External libraries | WebSearch for docs, npm/pypi |
| API behavior | Official documentation |
| Codebase patterns | Grep, Glob, Read tools |
| Version compatibility | Changelogs, compatibility docs |
| Service limits | Official docs, pricing pages |

## Plan Mode Integration

This plugin integrates with Claude Code's plan mode:

**1. Hook on plan mode entry:**
When you enter plan mode, the plugin displays a reminder:
```
💡 Tip: Run /research to eliminate assumptions before finalizing your plan.
```

**2. Writes to plan file on completion:**
When research completes, verified findings are written to the active plan file at `~/.claude/plans/`. This ensures your plan is backed by verified research.

## vs /iterate

| /iterate | /research |
|----------|-----------|
| Execution | Planning |
| Does the work | Verifies the approach |
| Tests/builds verify | Docs/code verify |
| After planning | Before execution |

**Recommended workflow:**
1. Enter plan mode
2. Run `/research` to verify approach
3. Finalize plan with verified facts
4. Exit plan mode
5. Run `/iterate` to execute

## License

MIT
