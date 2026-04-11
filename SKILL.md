---
name: BiliTickerSkill
description: 当用户想基于本地 biliTickerBuy 项目准备或执行 Bilibili 会员购抢票流程时使用此 skill，包括检查环境、确认登录、引导用户选择票档并生成或执行正确的抢票流程。
version: "0.1.0"
user-invocable: true
---

# BiliTickerSkill

这个 skill 是本地 `biliTickerBuy` 项目的编排层，不重新实现抢票逻辑。

`biliTickerBuy` 作为 git submodule 放在 `./biliTickerBuy`。如果仓库是新拉取的，先初始化 submodule。

触发这个 skill 后优先阅读：

- `references/workflow.md`
- `assets/examples/tickets.template.json`

优先使用 Python import，不要优先走 shell：

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

默认 API：

```python
fetch_purchase_context(project_input, *, cookies=None, cookies_path=None, selected_date=None, phone="") -> dict
build_ticket_config_from_selection(purchase_context: dict, selection: dict) -> dict
generate_ticket_config(parameters: dict) -> dict
get_login_state(*, cookies=None, cookies_path=None) -> dict
validate_config(config_or_path) -> dict-like ValidationResult
start_buy(config_or_path, runtime_options: dict | None = None) -> dict
task_status(task_id: str) -> dict
```

## 规则

1. 不要把真实 cookie、token、手机号、实名信息写入这个 skill 仓库。
2. 如果 skill 根目录下的 `biliTickerBuy` 不存在，先执行 `git submodule update --init --recursive`。
3. 一个关键前置步骤是先在 `biliTickerBuy` 目录执行 `uv sync`，确保生成并更新 `biliTickerBuy/.venv`。
4. 如果 `biliTickerBuy/.venv` 已经存在且可用，可以直接复用，不需要重复安装。
5. 后续所有 Python 执行都优先使用 `biliTickerBuy/.venv` 里的解释器和依赖环境。
6. 只有在明确已经把 `biliTickerBuy` 安装进当前解释器时，才可以直接 import；否则要显式使用 `biliTickerBuy/.venv`。

## 抢票流程

1. 先确认 `biliTickerBuy/.venv` 是否存在；如果不存在，进入 `biliTickerBuy` 目录执行 `uv sync`。
2. 之后所有 Python 操作都使用 `biliTickerBuy/.venv`。
3. 通过 `import biliTickerBuy` 导入统一入口。
4. 先调用 `biliTickerBuy.get_login_state`。
5. 如果未登录，流程必须停下，先要求用户扫码登录；登录完成前不能继续取票信息。
6. 登录成功后，再调用 `biliTickerBuy.fetch_purchase_context`，传入活动 URL 或 `project_id`。
7. 把拿到的可选项展示给用户，而不是自己猜：
   - 如果活动有多个日期，先展示 `sales_dates`
   - 展示当前日期下的 `ticket_options`
   - 展示当前账号下的 `buyers`
   - 展示当前账号下的 `addresses`
8. 如果日期或票档不唯一，必须让用户明确选择，不能默认取第一项。
9. 让用户明确确认这些信息：票档、实名人、联系人、联系电话、收货地址。
10. 调用 `biliTickerBuy.build_ticket_config_from_selection` 生成最终配置。
11. 先运行 `biliTickerBuy.validate_config`。
12. 再用 `biliTickerBuy.start_buy` 启动后台任务。
13. 用 `biliTickerBuy.task_status` 轮询状态。

当用户说“帮我买这张票”时，除非用户已经明确提供以下全部信息，否则不要直接进入 `start_buy`：

- 已完成登录
- 具体日期（如果存在多个日期）
- 具体票档
- 购票人选择
- 联系人姓名
- 联系人电话
- 收货地址选择

如果用户还没登录，下一步就是提示用户扫码登录并等待。如果用户不知道买哪一天或哪一档，下一步就是把选项列出来并让用户选。
