---
name: bili-ticker-buy
description: Use this skill when the user wants to prepare or run Bilibili ticket-buying workflows with a local biliTickerBuy project, including locating the project, validating prerequisites, shaping a ticket config, and generating or running the correct buy command in Codex or Claude Code.
version: "0.1.0"
user-invocable: true
---

# bili-ticker-buy

This skill orchestrates a local `biliTickerBuy` checkout. It does not reimplement the ticketing logic.

The `biliTickerBuy` dependency is tracked as a git submodule in `./biliTickerBuy`. If it is missing, initialize submodules before using the skill.

Read these first when the skill triggers:

- `references/workflow.md`
- `references/integration.md`
- `references/claude-code.md`
- `assets/examples/tickets.template.json`

Prefer Python imports over shell commands:

```python
import sys
from pathlib import Path

sys.path.insert(0, str(Path.cwd() / "biliTickerBuy"))

from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

Default API surface:

```python
generate_ticket_config(parameters: dict) -> dict
validate_config(config_or_path) -> dict-like ValidationResult
start_buy(config_or_path, runtime_options: dict | None = None) -> dict
task_status(task_id: str) -> dict
```

## Operating rules

1. Treat `biliTickerBuy` as the source of truth for runtime behavior and config fields.
2. Prefer helping the user generate config, validate it, and call the library API directly.
3. Do not invent unsupported flags, API routes, or config keys.
4. If direct import usage is enough, do not route back through `btb`.
5. If the user wants multi-agent orchestration, remote control, status polling, or external scheduling, then consult `references/integration.md` and propose a minimal interface to expose from `biliTickerBuy`.
6. Keep secrets out of committed files. Never persist real cookies, tokens, phone numbers, or buyer identity data in this skill repo.
7. When the local repo has not been installed as a package, add `./biliTickerBuy` to `sys.path` before importing `bilitickerbuy`.
8. If `./biliTickerBuy` is absent in a fresh clone, run `git submodule update --init --recursive`.

## Default workflow

1. Import `bilitickerbuy`.
2. Build or review a ticket config based on the upstream project schema.
3. Run `validate_config` first.
4. Use `start_buy` to create a background task.
5. Poll with `task_status`.

## When API exposure is justified

Only recommend new `biliTickerBuy` interfaces if one of these is true:

- the skill must control long-running tasks from another process
- the skill must fetch task status without owning the terminal session
- the skill must prepare config in one tool and execute in another
- the user explicitly wants a stable machine-facing contract

If so, keep the first interface small: health check, config validation, run task, task status.
