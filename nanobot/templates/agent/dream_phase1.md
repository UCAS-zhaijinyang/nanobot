You have TWO equally important tasks:
1. Extract new facts from conversation history
2. Deduplicate **memory/MEMORY.md** — find and flag redundant, overlapping, or stale content even if NOT mentioned in history

Use headings already present in MEMORY.md (e.g. User Information, Preferences, Project Context, Important Notes, bot tone / communication style) so personality, user facts, and project facts stay organized.

Output one line per finding:
[MEMORY] atomic fact (not already captured appropriately)
[MEMORY-REMOVE] reason for removal
[SKILL] kebab-case-name: one-line description of the reusable pattern

Rules:
- Atomic facts: "has a cat named Luna" not "discussed pet care"
- Corrections: [MEMORY] location is Tokyo, not Osaka — merge under the right heading
- Capture confirmed approaches the user validated

Deduplication — scan **memory/MEMORY.md** for these redundancy patterns:
- Same fact stated in multiple sections or bullets
- Overlapping sections covering the same topic
- Verbose entries that can be condensed without losing information
For each duplicate found, output [MEMORY-REMOVE] for the less authoritative copy

Staleness — lines may show a ``← Nd`` suffix (days since last line edit):
- Age only indicates when content was last touched in git, not whether it must be removed
- Use content judgment: user habits/preferences/personality traits may remain indefinitely
- Only prune content that is objectively outdated: passed events, resolved tracking, superseded approaches
- Lines with ``← Nd`` (N>{{ stale_threshold_days }}) deserve closer review but are NOT automatically removable
- When removing: prefer deleting individual items over entire sections

Skill discovery — flag [SKILL] when ALL of these are true:
- A specific, repeatable workflow appeared 2+ times in the conversation history
- It involves clear steps (not vague preferences like "likes concise answers")
- It is substantial enough to warrant its own instruction set (not trivial like "read a file")
- Do not worry about duplicates — the next phase will check against existing skills

Do not add: current weather, transient status, temporary errors, conversational filler.

[SKIP] if nothing needs updating.
