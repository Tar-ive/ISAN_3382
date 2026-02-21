# MEMORY.md - Long-Term Memory

Curated, long-lived notes. Daily raw logs live in `memory/YYYY-MM-DD.md`.

## Rewind philosophy: goal-driven autonomous tasks (2026-02-21)
Rewind should evolve from reactive to proactive via a heartbeat-style loop: generate small daily tasks that move the user toward explicit goals using implicit signals, while keeping trust and clarity high.

### Core philosophies / constraints
1) **Trust is earned incrementally**
   - Start with low-risk, mundane, reviewable tasks.
   - Default to *drafts for review* (draft email/scripts/notes) rather than taking irreversible external actions (sending/posting).
2) **Transparency beats speed**
   - Users lose trust when agents act invisibly/too quickly.
   - Always track work (kanban/log/checklist) and report what changed and why.
3) **Accessibility matters**
   - Proactive assistance must be reliably reachable and not depend on the user remembering to ask.
4) **Humans resist what helps them**
   - Expect inconsistent behavior (alarms set → snoozed → frustration).
   - System should stay resilient and supportive, prioritizing long-term wellbeing over short-term compliance.

### Product direction
- Implement a “heartbeat” workflow similar in spirit to ZeroClaw: propose 4–5 tasks/day, execute within guardrails, and keep the user in the loop.
