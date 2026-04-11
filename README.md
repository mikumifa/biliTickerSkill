# bili-ticker-buy

Skill scaffold for using `biliTickerBuy` from Codex or Claude Code.

Current layout:

- `SKILL.md`: skill entry and operating rules
- `agents/openai.yaml`: Codex UI metadata
- `references/`: workflow and future integration contract
- `assets/examples/`: sanitized config template
- `biliTickerBuy/`: upstream dependency tracked as a git submodule

Clone after setup:

```bash
git submodule update --init --recursive
```

This repository is intentionally structured so the skill can call `biliTickerBuy` as a Python library first, then optionally expose richer interfaces later if needed.
