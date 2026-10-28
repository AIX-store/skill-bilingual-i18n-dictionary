# bilingual-i18n-dictionary

An AI skill for running one English/Arabic translation dictionary in an app. It sets the key
convention, writes ICU messages with all six Arabic plural categories and gender selects,
keeps interpolation bidi-safe, audits missing, orphaned and stale keys, and defines a
review workflow your CI can enforce.

## Install

Claude Code:

    git clone https://github.com/AIX-store/skill-bilingual-i18n-dictionary ~/.claude/skills/bilingual-i18n-dictionary

Other agents (Codex, Cursor, Claude Desktop, claude.ai): copy `SKILL.md` into the agent's
instructions or project knowledge.

## Example prompts

- "Set up an EN/AR dictionary structure for our React app and convert the checkout page."
- "Audit locales/en.json against locales/ar.json and list missing and stale keys."
- "Write the Arabic plural forms for 'N files uploaded'."
- "Find hard-coded English strings in src/ that bypass the translate function."

## Licence

MIT, © 2026 CodeMeld LLC.

Listed on AiX: https://aixstore.ai/store
