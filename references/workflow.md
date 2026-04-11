# Workflow

## Purpose

This skill is a thin orchestration layer around the local `biliTickerBuy` project.

The dependency is expected at `./biliTickerBuy` as a git submodule. In a fresh clone, initialize it first with:

```bash
git submodule update --init --recursive
```

## Import-first integration

Preferred import:

```python
from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

If the project is only present as a local folder, add it to `sys.path`:

```python
import sys
from pathlib import Path

sys.path.insert(0, str(Path.cwd() / "biliTickerBuy"))
from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

## Exposed library surface

The upstream package now exposes:

- `generate_ticket_config`
- `validate_config`
- `start_buy`
- `task_status`

The upstream CLI currently supports:

- `buy <tickets_info>`
- `--interval`
- `--time_start`
- `--https_proxys`
- `--audio_path`
- `--pushplusToken`
- `--serverchanKey`
- `--serverchan3ApiUrl`
- `--barkToken`
- `--ntfy_url`
- `--ntfy_username`
- `--ntfy_password`
- `--web`
- `--hide_random_message`

## Config shape

The buy task expects JSON with these core fields:

- `detail`
- `count`
- `screen_id`
- `project_id`
- `sku_id`
- `pay_money`
- `buyer_info`
- `buyer`
- `tel`
- `deliver_info`
- `cookies`

Optional fields seen upstream:

- `phone`
- `is_hot_project`
- `link_id`
- `order_type`

Use `assets/examples/tickets.template.json` as a sanitized template. Do not commit real values.

## Safe operating pattern

1. Build a config dict from user parameters or load a JSON file.
2. Call `validate_config` and stop on errors.
3. Call `start_buy` with runtime options.
4. Poll `task_status` until the task is no longer running.
5. Keep runtime artifacts outside git-tracked example files.
