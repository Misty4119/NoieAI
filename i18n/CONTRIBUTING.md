# Translation Guidelines

Thank you for your interest in translating NoieAGENTS documentation!

## Translation Principles

1. **Faithfulness** — Translate the meaning accurately, not word-for-word
2. **Consistency** — Use consistent terminology throughout
3. **Cultural Adaptation** — Adapt examples if needed for cultural context

## Language Code Standards

| Region | Code | Example |
|--------|------|---------|
| 繁體中文 (台灣) | `zh_tw` | README.zh_tw.md |
| English (US) | `en_us` | README.en_us.md |
| 日本語 (日本) | `ja_jp` | README.ja_jp.md |

## File Naming Convention

```
README.<code>.md          # For README translations
<original_name>.<code>.md  # For document translations
```

Example:
- `AGENTS.md` → `AGENTS.en_us.md`
- `SOUL.md` → `SOUL.ja_jp.md`

## Translation Header Template

Each translated file should include:

```markdown
---
translation:
  version: <original version>
  language: <language code>
  last_updated: <YYYY-MM-DD>
  translator: <your name/alias>
---
```

## Questions?

If you have questions about translation guidelines, please open an issue for discussion.
