# Claude Code Usage

## Goal

Use `biliTickerBuy` through Python imports, not through the `btb` command.

## Preferred import pattern

If `biliTickerBuy` has been installed into the environment:

```python
from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

If the repository is only available as a local folder next to this skill repo:

```python
import sys
from pathlib import Path

sys.path.insert(0, str(Path.cwd() / "biliTickerBuy"))

from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

## Expected flow

1. Build a config dict from user parameters with `generate_ticket_config`.
2. Validate it with `validate_config`.
3. Start the background task with `start_buy`.
4. Poll with `task_status` until the task reaches `succeeded`, `completed`, or `failed`.

## Example

```python
import sys
import time
from pathlib import Path

sys.path.insert(0, str(Path.cwd() / "biliTickerBuy"))

from bilitickerbuy import generate_ticket_config, start_buy, task_status

config = generate_ticket_config(
    {
        "username": "demo-user",
        "project_id": 123,
        "screen_id": 456,
        "sku_id": 789,
        "count": 1,
        "unit_price": 6800,
        "buyer_info": [{"name": "张三", "personal_id": "ID-CARD"}],
        "buyer": "张三",
        "tel": "13800000000",
        "deliver_info": {
            "name": "张三",
            "tel": "13800000000",
            "addr_id": 1,
            "addr": "测试地址",
        },
        "cookies_path": str(Path.cwd() / "biliTickerBuy" / "cookies.json"),
    }
)

started = start_buy(config, runtime_options={"interval": 800})
task_id = started["task"]["task_id"]

while True:
    status = task_status(task_id)
    task = status["task"]
    if task["status"] in {"succeeded", "completed", "failed"}:
        break
    time.sleep(1)
```
