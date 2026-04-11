# 工作流

## 目标

这个 skill 是本地 `biliTickerBuy` 项目的轻量编排层。

仓库默认把依赖放在 `./biliTickerBuy` 这个 git submodule 下。如果是新仓库，先初始化：

```bash
git submodule update --init --recursive
```

初始化后，优先在 `biliTickerBuy` 目录执行：

```bash
uv sync
```

目标是确保 `biliTickerBuy/.venv` 存在。后续执行默认都走这个虚拟环境；如果 `.venv` 已经存在且可用，直接复用即可。

## 优先使用 `.venv`

推荐优先使用 `biliTickerBuy/.venv` 中的 Python 解释器。

例如在 Windows PowerShell 下可以直接执行：

```powershell
.\biliTickerBuy\.venv\Scripts\python.exe -c "import bilitickerbuy; print('ok')"
```

只有在你已经确认当前解释器里能直接导入 `bilitickerbuy` 时，才可以跳过 `.venv`。

## 优先使用 import

推荐直接 import：

```python
import biliTickerBuy
```

如果项目只是本地目录，没有安装成包：

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

这里查找的是 skill 仓库根目录，不是当前 shell 的工作目录。

## 当前暴露的库接口

上游现在提供这些函数：

- `fetch_project_detail`
- `fetch_ticket_options`
- `fetch_buyers`
- `fetch_addresses`
- `fetch_purchase_context`
- `build_ticket_config_from_selection`
- `load_ticket_config`
- `save_ticket_config`
- `generate_ticket_config`
- `get_login_state`
- `build_runtime_options`
- `validate_config`
- `run_buy_sync`
- `start_buy`
- `task_status`

CLI 当前支持这些参数：

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

## 配置结构

抢票配置 JSON 的核心字段：

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

上游里还能看到这些可选字段：

- `phone`
- `is_hot_project`
- `link_id`
- `order_type`

可参考 `assets/examples/tickets.template.json` 作为脱敏模板，不要提交真实数据。

## 引导式交互流程

这类自然语言请求适用，例如“帮我买这张票”：

1. 先调 `biliTickerBuy.get_login_state(...)`。
2. 如果 `logged_in` 是 `false`，流程必须停下，要求用户先扫码登录，不能继续取活动信息。
3. 登录完成后，调 `biliTickerBuy.fetch_purchase_context(url_or_project_id, cookies=..., selected_date=None)`。
4. 如果 `sales_dates` 非空且用户还没选日期，先把日期列出来让用户选。
5. 把 `ticket_options` 列出来，让用户明确选中具体票档。
6. 把登录账号下的 `buyers` 和 `addresses` 列出来，让用户明确选择。
7. 额外收集 `buyer` 和 `tel` 这两个联系人字段。
8. 用 `biliTickerBuy.build_ticket_config_from_selection` 生成最终配置。
9. 然后再做校验和启动。

有两个必须遵守的关卡：

1. 登录关卡：未登录就不能继续。
2. 选择关卡：用户没选日期或票档时，不能猜，必须停下来问。

## 安全执行模式

1. 先确认 `biliTickerBuy/.venv` 可用；不可用时先在 `biliTickerBuy` 目录运行 `uv sync`。
2. 根据用户参数或交互式选择构造配置，或者读取现成 JSON。
3. 先调用 `biliTickerBuy.validate_config`，有错误就停。
4. 再调用 `biliTickerBuy.start_buy` 并传入运行参数。
5. 持续轮询 `biliTickerBuy.task_status`，直到任务结束。
6. 运行时产物不要写回 git 跟踪的示例文件。
