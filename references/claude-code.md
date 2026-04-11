# Claude Code 用法

## 目标

通过 `biliTickerBuy/.venv` 中的 Python 环境直接 `import biliTickerBuy`，不要默认使用系统 Python，也不要优先走 `btb` 命令。

## 前置步骤

先定位 skill 根目录和 `biliTickerBuy` 目录。

如果 `biliTickerBuy/.venv` 不存在，先进入 `biliTickerBuy` 目录执行：

```bash
uv sync
```

如果 `.venv` 已经存在且可用，直接复用。

## 推荐执行方式

推荐始终使用 `biliTickerBuy/.venv` 里的解释器执行脚本。

```python
import subprocess
from pathlib import Path

def find_skill_root() -> Path:
    start = Path(__file__).resolve().parent if "__file__" in globals() else Path.cwd()
    for candidate in [start, *start.parents]:
        if (candidate / "SKILL.md").exists() and (candidate / "biliTickerBuy").exists():
            return candidate
    raise RuntimeError("找不到 BiliTickerSkill 根目录")

SKILL_ROOT = find_skill_root()
VENV_PYTHON = SKILL_ROOT / "biliTickerBuy" / ".venv" / "Scripts" / "python.exe"

if not VENV_PYTHON.exists():
    raise RuntimeError("请先在 biliTickerBuy 目录执行 uv sync，生成 .venv")
```

## 推荐 import 方式

如果已经进入 `biliTickerBuy/.venv` 对应的解释器环境，可以直接：

```python
import biliTickerBuy
```

如果只是从 skill 仓库旁路调用，也要基于 skill 根目录补 `sys.path`：

```python
import sys
from pathlib import Path

def find_skill_root() -> Path:
    start = Path(__file__).resolve().parent if "__file__" in globals() else Path.cwd()
    for candidate in [start, *start.parents]:
        if (candidate / "SKILL.md").exists() and (candidate / "biliTickerBuy").exists():
            return candidate
    raise RuntimeError("找不到 BiliTickerSkill 根目录")

SKILL_ROOT = find_skill_root()
sys.path.insert(0, str(SKILL_ROOT.parent))

import biliTickerBuy
```

## 预期流程

1. 先确认 `biliTickerBuy/.venv` 可用；不可用时先执行 `uv sync`。
2. 先调用 `biliTickerBuy.get_login_state`。
3. 如果还没登录，就要求用户先扫码登录，不要继续。
4. 再调用 `biliTickerBuy.fetch_purchase_context` 获取活动上下文。
5. 让用户依次选择日期、票档、实名人、地址，并补全联系人姓名和电话。
6. 如果用户没选日期或票档，不要猜，直接展示选项并等待。
7. 用 `biliTickerBuy.build_ticket_config_from_selection` 生成最终配置。
8. 用 `biliTickerBuy.validate_config` 校验。
9. 用 `biliTickerBuy.start_buy` 启动任务。
10. 用 `biliTickerBuy.task_status` 轮询到结束。

## 示例

```python
import sys
import time
from pathlib import Path

def find_skill_root() -> Path:
    start = Path(__file__).resolve().parent if "__file__" in globals() else Path.cwd()
    for candidate in [start, *start.parents]:
        if (candidate / "SKILL.md").exists() and (candidate / "biliTickerBuy").exists():
            return candidate
    raise RuntimeError("找不到 BiliTickerSkill 根目录")

SKILL_ROOT = find_skill_root()
sys.path.insert(0, str(SKILL_ROOT.parent))

import biliTickerBuy

login_state = biliTickerBuy.get_login_state(
    cookies_path=str(SKILL_ROOT / "biliTickerBuy" / "cookies.json"),
)
if not login_state["logged_in"]:
    raise RuntimeError("请先让用户扫码登录，再继续后续步骤")

context = biliTickerBuy.fetch_purchase_context(
    "https://show.bilibili.com/platform/detail.html?id=123",
    cookies_path=str(SKILL_ROOT / "biliTickerBuy" / "cookies.json"),
)

config = biliTickerBuy.build_ticket_config_from_selection(
    context,
    {
        "ticket_index": 0,
        "buyer_indices": [0],
        "address_index": 0,
        "buyer": "张三",
        "tel": "13800000000",
    },
)

started = biliTickerBuy.start_buy(config, runtime_options={"interval": 800})
task_id = started["task"]["task_id"]

while True:
    status = biliTickerBuy.task_status(task_id)
    task = status["task"]
    if task["status"] in {"succeeded", "completed", "failed"}:
        break
    time.sleep(1)
```
