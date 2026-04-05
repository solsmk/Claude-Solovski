---
name: self-care
description: Maintenance routines that keep the agent workspace healthy. Runs daily, weekly, monthly, and on error recovery. A neglected agent degrades silently. An uninformed agent falls behind. Staying current is part of the job.
---

# Self-Care Skill

> Maintenance routines that keep the agent workspace healthy.
> A neglected agent degrades silently. This skill prevents that.
> An uninformed agent falls behind. Staying current is part of the job.

## When This Skill Activates

- Morning routine (daily, first thing)
- Weekly maintenance (Sunday evening)
- Monthly deep review (first Monday)
- On crash/error recovery
- When human asks "how are you doing?" or "check yourself"

## Daily Morning Routine

Run these checks before any other work. Takes ~2 minutes.
Only report problems — silence means healthy.

### 1. MEMORY HEALTH
- Did Auto Dream run since last session? Check modification times
- Any memory files over 200 lines? → Flag or trigger /dream
- Memory directory total size — is it growing abnormally?

### 2. DISK & SYSTEM
- Filesystem usage: warn at 80%, emergency clean at 90%
- Docker: dangling images, stopped containers eating space
- Temp files: clean anything older than 7 days from /tmp, test outputs
- Log files: rotate or clean if over 100MB total

### 3. PROCESS HEALTH
- Is Claude Desktop running normally?
- Are scheduled tasks still active? Any that failed overnight?
- Any error logs from overnight runs?

### 4. CREDENTIALS
- Google OAuth token: valid? Expiring within 48h? → Warn human
- GitHub token: valid? → Warn human
- Any MCP server connections failing? → Diagnose and report

### 5. GIT STATE
- Any uncommitted changes in workspace? → Commit or stash
- Any unfinished merges or rebases? → Resolve or report
- Working tree clean? → Good

**Report format** (only if issues found):
```
🔧 Morning self-check:
- [issue 1]
- [issue 2]
All other systems healthy.
```

If everything is fine, say nothing. Start the day's work.

---

## Weekly Maintenance (Sunday Evening)

### Skill Audit
For each skill in `/workspace/project/.claude/skills/`:
1. Read SKILL.md — is it still relevant?
2. Last used? Over 30 days → flag as potentially stale. Over 60 days → candidate for retirement
3. Has the underlying service likely changed? → Flag for review
4. Compile skill health report

### Memory Review
1. List all files in `/workspace/group/memory/` with sizes and last modified dates
2. Any files not modified in 60+ days? → Still relevant?
3. Any contradictions Auto Dream might have missed? Search for the same entity with different facts
4. Client files: is info still current?
5. Research files older than 90 days: mark as `[STALE - verify before using]`

### Workspace Housekeeping
1. Git: delete merged branches, check for orphaned worktrees
2. `proposals/`: any old proposals that were decided? Archive them
3. Total workspace size: trend over past 4 weeks
4. Docker: system prune if reclaimable space > 5GB

### Landscape Watch
1. **CHECK ANTHROPIC UPDATES**
   - Search: "Anthropic Claude update" (past 7 days)
   - Check: blog.anthropic.com
   - Check: Claude Code changelog / releasebot.io
   - Check: github.com/aaddrick/claude-desktop-debian/releases
   - Look for: new features, new capabilities, deprecations, security fixes

2. **CHECK TOOL UPDATES**
   - MCP servers in use: any new versions?
   - Services I interact with: Bugsink, Coolify, MeiliSearch, MedusaJS updates?
   - Any new MCP servers on mcp.so that could replace custom skills?

3. **CHECK ECOSYSTEM**
   - GitHub trending in AI/agents space: anything relevant?
   - New Claude Code plugins in marketplace?
   - OpenClaw community: any skills worth adapting?

4. **EVALUATE FINDINGS**
   For each finding, assess:
   - Does this affect current operation? (security fix, breaking change)
   - Could this improve an existing skill? (better API, new MCP)
   - Could this enable a new capability? (new feature, new connector)
   - Is this just interesting but not actionable right now?

5. **REPORT** (if anything found):
   ```
   Weekly landscape update:
   - [Critical] Description (affects current operation)
   - [Opportunity] Description (could improve us)
   - [FYI] Description (interesting, not urgent)
   Full details saved to memory/research/landscape-watch-YYYY-MM-DD.md
   ```

6. **SAVE** to `memory/research/landscape-watch-YYYY-MM-DD.md`

---

## Monthly Deep Review (First Monday)

### Anthropic Deep Dive
1. Read ALL Claude release notes from past month
2. Specifically check:
   - New agent features (tools, scheduling, permissions)
   - MCP protocol changes
   - New connectors/plugins that replace custom skills
   - Rate limit or billing changes
   - Context window changes
   - Security advisories
   - Computer use improvements
   - Claude Desktop changes

3. Write monthly briefing:
   ```
   What changed in Claude this month:
   - [Must act] Changes that require action from us
   - [Should adopt] Features that would improve our workflow
   - [Watch] Things to monitor for future adoption
   - [No action] Changes that don't affect us
   ```

### Self-Update Check
1. Current claude-desktop-debian version vs latest release on github.com/aaddrick/claude-desktop-debian
2. If update available:
   - Read release notes
   - Assess risk: breaking changes? New dependencies?
   - Propose update to human with summary
   - NEVER auto-update
3. Check for system-level updates (Ubuntu, Node.js, npm)
   - Security updates: flag as urgent
   - Feature updates: propose with reasoning

### Performance Report
Generate and send:
- Tasks completed this month: N
- Skills created: N (list)
- Skills improved: N (list with before/after scores)
- Skills retired: N (list with reasons)
- Average self-score: X/6
- Weakest scoring criterion: [which one]
- Constitution proposals: N made, N accepted, N rejected
- Landscape findings: N total, N adopted
- Biggest win this month: [description]
- Biggest failure this month: [description, what I learned]

---

## Error Recovery

### On Crash / Unexpected Restart
1. **DIAGNOSE**
   - Check logs: what was the last action?
   - Check git status: any interrupted operations?
   - Check scheduled tasks: any that were running?
   - Check disk: was it a space issue?

2. **CLEAN UP**
   - Resolve any interrupted git operations (merge, rebase, cherry-pick)
   - Commit or stash any uncommitted work
   - Clear any lock files (.lock, .pid)
   - Verify workspace integrity

3. **RECOVER**
   - Resume interrupted tasks if safe to do so
   - If task state is unclear, start fresh (don't guess)
   - Report to human: "I restarted. Cause: [X]. State: [clean/recovered]. Resuming: [Y]."

4. **POST-MORTEM**
   - If crash was caused by my action → write to memory, propose skill fix
   - If crash was external (OOM, disk, network) → note in memory, check if preventable
   - If crash is recurring → escalate to human

### Disk Full Emergency
1. `docker system prune -f` (reclaim dangling images/containers)
2. Clear logs older than 3 days (emergency threshold, normally 7)
3. Clear test-cases output artifacts
4. Clear /tmp
5. If still above 90% → report to human with breakdown of disk usage
6. **NEVER delete:** `memory/`, `skills/`, `proposals/`, `.git/`

### Auth Failure
1. Identify which credential failed (Google, GitHub, MCP server)
2. **DO NOT retry in a loop** (wastes rate limits, may trigger lockouts)
3. Notify human immediately:
   ```
   [Service] authentication failed. Error: [message].
   I need you to re-authenticate. Here's how: [steps]
   ```
4. Continue with tasks that don't need the failed credential
5. Queue tasks that need it for retry after re-auth

---

*Self-care is not optional overhead. It's what separates an agent that works for a week from one that works for a year.*
