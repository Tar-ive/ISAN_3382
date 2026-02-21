# MEMORY.md - Long-Term Memory

Curated, long-lived notes. Daily raw logs live in `memory/YYYY-MM-DD.md`.

## Rewind / reminders direction (2026-02-21)
- Reminders product direction: implement reminders **Rewind-native**, borrowing patterns from ZeroClaw iMessage/scheduling where useful, and bake **STS/MTS/LTS** into reminder policy.
- Current status as of today: STS/MTS confirmed implemented; LTS scaffolding exists locally (was noted as uncommitted at the time of writing).
- Reference doc pushed: `docs/rewind-native-imessage-reminders-plan.md` (repo: `rewind/rust-native`, commit `04df1dc`).

## Ops / reliability practices (2026-02-21)
- Cron reliability: failing isolated cron jobs were patched to use model `openai-codex/gpt-5.3-codex`; disk maintenance jobs normalized.
- Model availability note: `zai/glm-4.7` unusable (subscription over). `nvidia/moonshotai/kimi-k2.5` also not working in practice. Prefer Anthropic Sonnet + Google Gemini as fallbacks.
- Disk maintenance: added safe size-cap rotation for `/data/build/rewind-target` (default 6GB) to prevent disk pressure recurrence; post-cleanup snapshot: `/` ~86–88%, `/data` ~64%.
