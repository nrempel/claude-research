---
name: research
description: Deep research to eliminate all assumptions before implementation. Use when user wants thorough, verified planning.
user-invocable: true
---

# Research

You are entering **deep research mode**. Your job is to eliminate ALL assumptions through systematic research before any implementation begins. Don't guess - verify.

## First: Check for Existing Session

Before doing anything else, check for an existing research session:

```bash
# Get current branch (sanitized)
git branch --show-current 2>/dev/null || echo "no-git"

# Check for existing state files
ls .claude/research/<branch>-*.md 2>/dev/null
```

**If state file exists AND there are unverified assumptions:**
- You are **resuming** after context compaction
- Read the state file to recall the goal and progress
- Say: "Resuming research session. Goal: [goal]. X assumptions verified, Y remaining."
- Continue from the next unverified assumption

**If no state file:**
- This is a **new session**
- Proceed to "Initialize Session" below

## Invocation Modes

- `/research <task>` → Start new research session
- `/research` (no args) → Continue existing OR prompt for task
- `/research status` → Show goal, assumptions, and progress
- `/research cancel` → Delete state file, clear todos, stop

## Initialize Session

When starting a new research session:

### 1. Create State File

```bash
mkdir -p .claude/research
```

Filename: `.claude/research/<branch>-<timestamp>.md`
- Branch: sanitize by replacing non-alphanumeric chars with `-`, truncate to 50 chars
- Timestamp: ISO format like `20260113-213000`
- Example: `.claude/research/main-20260113-213000.md`

**For detached HEAD:** Use `detached-<short-hash>`
**For no git repo:** Use `no-git`

### 2. Draft Initial Approach

Before extracting assumptions, draft a rough approach:

```markdown
---
started: 2026-01-13T21:30:00Z
branch: main
---

# Goal
[The user's task in their words]

## Current Approach
[Your initial plan - technologies, patterns, architecture]

## Assumptions

### Verified ✅
(none yet)

### Unverified ❓
(to be extracted)

### Refuted ❌
(none yet)

## Research Log
(findings will be logged here)
```

### 3. Extract ALL Assumptions

Scan your approach ruthlessly. Flag EVERYTHING that isn't verified fact:

**Technology assumptions:**
- "We'll use X library" → Does it exist? Is it maintained? Does it do what we need?
- "X supports Y feature" → Verify in docs
- "Version X is compatible" → Check compatibility

**Codebase assumptions:**
- "The project uses pattern X" → Search and verify
- "There's an existing Y we can use" → Find it
- "This file handles Z" → Read it

**Behavior assumptions:**
- "The API returns X" → Check docs or test it
- "This will be fast enough" → Research performance characteristics
- "Users will want X" → Is this stated anywhere?

**External service assumptions:**
- "Service X allows Y" → Check their docs
- "Rate limits are Z" → Verify
- "Authentication works via W" → Research

For each assumption, create a todo:
```
TodoWrite:
- [ ] Verify: React 18 is used in this project
- [ ] Verify: Postgres supports JSONB columns
- [ ] Verify: Stripe API allows partial refunds
```

## Research Loop

For each unverified assumption:

### a) Mark in progress
```
TodoWrite: mark current assumption as in_progress
```

### b) Research it

**For external tools/services/libraries:**
- Use WebSearch to find official documentation
- Search for version compatibility, limitations, gotchas
- Look for recent changes or deprecations

**For codebase questions:**
- Use Grep to search for patterns
- Use Glob to find relevant files
- Use Read to examine actual code
- Check package.json, config files, etc.

**For behavior questions:**
- Search for documentation
- Look for examples or tests
- Check issue trackers for edge cases

### c) Log the finding

Update the state file with your finding:

```markdown
## Research Log
### 2026-01-13 21:35
**Assumption:** Postgres supports JSONB columns
**Research:** WebSearch "postgres jsonb support version"
**Finding:** JSONB supported since Postgres 9.4. Project uses Postgres 14 (verified in docker-compose.yml)
**Result:** ✅ Verified
```

### d) Update assumption status

Move the assumption to the appropriate section:
- **Verified ✅** - Evidence confirms it's true
- **Refuted ❌** - Evidence shows it's false (update approach!)

### e) If refuted, update approach

When an assumption is refuted:
1. Update the "Current Approach" section
2. Extract any NEW assumptions from the updated approach
3. Add new assumptions to the unverified list

### f) Move to next assumption immediately

Don't stop. Don't ask. Pick the next unverified assumption and research it.

## Re-scan for New Assumptions

After processing all assumptions:

1. Re-read the "Current Approach" section
2. Look for any NEW implicit assumptions
3. If found, add them to unverified list and continue
4. If none found, research is complete

## Completion

Research is done when:
- All assumptions are verified or refuted
- The approach has been updated based on findings
- No new assumptions remain

**When complete:**

1. Present the assumption-free plan:
```
## Research Complete

### Goal
[Original goal]

### Verified Approach
[Final approach with all assumptions verified]

### Key Findings
- [Important discovery 1]
- [Important discovery 2]

### Verified Facts
- ✅ [Fact 1 - how verified]
- ✅ [Fact 2 - how verified]

### Refuted Assumptions (approach adjusted)
- ❌ [Original assumption] → [What's actually true]
```

2. **Write to Claude Code plan file** (if one exists):
```bash
# Check for existing plan file
ls ~/.claude/plans/*.md 2>/dev/null | head -1
```

If a plan file exists, update it with the verified plan:
- Read the existing plan file
- Replace or append a "## Verified Research" section with:
  - Goal
  - Verified approach
  - Key findings
  - All verified facts with sources

This integrates your research directly into plan mode's plan file.

3. Delete the research state file:
```bash
rm .claude/research/<branch>-<timestamp>.md
```

4. Clear todos

## What Counts as "Verified"

| Type | How to Verify |
|------|---------------|
| Library exists | Find it on npm/pypi/crates.io |
| Library does X | Find in official docs |
| API behavior | Official API documentation |
| Codebase pattern | Find actual code using Grep/Read |
| Version compatibility | Check docs, changelogs, compatibility matrices |
| Service limits | Official documentation or pricing page |
| Performance | Benchmarks, documentation, or reasoning from first principles |

**NOT verification:**
- "I think it works this way" ❌
- "It probably supports that" ❌
- "Most libraries do this" ❌
- Stack Overflow answers (use as lead, verify in official docs) ⚠️

## Anti-Patterns (Don't Do These)

- ❌ "I assume X..." → Research it
- ❌ Skipping WebSearch for external tools → Always check docs
- ❌ "This probably works..." → Verify or mark as assumption
- ❌ Marking verified without evidence → Cite your source
- ❌ Stopping with unverified assumptions → Keep going
- ❌ Not updating approach when refuted → Adapt the plan
- ❌ Forgetting to re-scan for new assumptions → Always re-scan
- ❌ Not creating state file → Always create it first

## Token Efficiency

- Log findings concisely (not full docs)
- Don't re-research verified assumptions
- Update state file incrementally
- Brief status updates between assumptions

## Example Flow

```
User: /research Add Stripe subscription billing to the app

[Check for existing session]
→ No state file found, starting new session

[Initialize]
→ Create .claude/research/main-20260113-213000.md
→ Goal: Add Stripe subscription billing
→ Draft approach: Use Stripe Billing API, webhooks for events, store subscription status in DB

[Extract assumptions]
TodoWrite:
- [ ] Verify: Stripe Billing API supports usage-based billing
- [ ] Verify: Webhooks can be configured per-event type
- [ ] Verify: Project has webhook endpoint infrastructure
- [ ] Verify: DB schema can store subscription metadata

[Research loop]
→ Mark "Stripe Billing API supports usage-based billing" in_progress
→ WebSearch "stripe billing api usage based"
→ Finding: Yes, via "metered billing" - requires usage records API
→ Log finding, mark ✅ verified

→ Mark "Webhooks can be configured per-event type" in_progress
→ WebSearch "stripe webhook event types filter"
→ Finding: Yes, can select specific events in dashboard or API
→ Log finding, mark ✅ verified

→ Mark "Project has webhook endpoint infrastructure" in_progress
→ Grep for "webhook" in codebase
→ Finding: No webhook handling found
→ Log finding, mark ❌ refuted
→ Update approach: Need to add webhook endpoint

[Re-scan]
→ New assumption from updated approach: "Next.js API routes can handle webhooks"
→ Add to unverified list, continue researching

[...continues until all verified...]

[Complete]
→ Present verified plan with all findings
→ Delete state file

## Research Complete

### Goal
Add Stripe subscription billing to the app

### Verified Approach
1. Use Stripe Billing API with metered billing
2. Create new webhook endpoint at /api/webhooks/stripe
3. Handle subscription.created, invoice.paid, subscription.deleted events
4. Store subscription_id and status in users table (add migration)

### Key Findings
- Metered billing requires usage records API calls
- No existing webhook infrastructure - needs to be built
- Stripe webhook signature verification required

### Verified Facts
- ✅ Stripe supports metered billing (docs)
- ✅ Can filter webhook events (docs)
- ✅ Project uses Next.js 14 (package.json)
- ✅ DB uses Postgres with Drizzle ORM (schema.ts)

### Refuted Assumptions
- ❌ "Project has webhook handling" → No, needs to be added
```

## Resuming After Compaction

If context compacted mid-research:

```
User: /research

[Check for existing session]
→ Found .claude/research/main-20260113-213000.md
→ Read state file: Goal is "Add Stripe subscription billing"
→ 3 assumptions verified, 2 remaining

Resuming research session.
Goal: Add Stripe subscription billing
Progress: 3 of 5 assumptions verified

[Continue from next unverified assumption]
→ Mark "DB schema can store subscription metadata" in_progress
→ ...
```

## Remember

You are not making educated guesses. You are a researcher establishing facts. Every claim in your final plan should trace back to verified evidence. Keep researching until zero assumptions remain.

**The state file is your memory.** Create it at the start, read it on resume, delete it when done.
