---
name: BiliTickerSkill
description: 当用户想让助手帮他找 Bilibili 会员购上的漫展、演出或其他活动，或基于本地 biliTickerBuy 项目继续准备和执行抢票流程时使用此 skill，包括搜索候选活动、检查环境、确认登录、引导用户选择票档并生成或执行正确的抢票流程。
version: "0.1.0"
user-invocable: true
---

# BiliTickerSkill

这个 skill 是本地 `biliTickerBuy` 项目的编排层，不重新实现抢票逻辑。

它不仅能在用户已经给出活动链接后继续抢票，也能在用户只给关键词、IP 名、角色名、城市、时间范围等线索时，先帮用户找漫展或其他会员购活动，再继续后续流程。

如果用户直接说“帮我抢票：xxxname”这类完整或接近完整的活动标题，也应优先视为可直接调用 `search_tickets` 的输入，而不是要求用户先补活动链接。

`biliTickerBuy` 作为 git submodule 放在 `./biliTickerBuy`。如果仓库是新拉取的，先初始化 submodule。

触发这个 skill 后优先阅读：

- `references/workflow.md` 抢票流程
- `references/search.md` 搜索
- `references/buy.md` 抢票操作详细
- `assets/examples/tickets.template.json`

在 `uv` 准备好的环境里直接使用 Python import：

```python
import interface as btb
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
start_managed_buy(config_or_path, runtime_options: dict | None = None, run_id: str | None = None, runs_root: str | Path | None = None) -> dict
managed_task_status(run_id: str, runs_root: str | Path | None = None) -> dict
normalize_time_start(value) -> str
normalize_interval(value) -> int
```

## 规则

1. 不要把真实 cookie、token、手机号、实名信息写入这个 skill 仓库。
2. 如果 skill 根目录下的 `biliTickerBuy` 不存在，先执行 `git submodule update --init --recursive`。
3. 一个关键前置步骤是先在 `biliTickerBuy` 目录执行 `uv sync`，确保生成并更新 `biliTickerBuy/.venv`。
4. 如果 `biliTickerBuy/.venv` 已经存在且可用，可以直接复用，不需要重复安装。
5. 后续所有 Python 执行都优先使用 `biliTickerBuy/.venv` 里的解释器和依赖环境。
6. 不允许打开原版 `biliTickerBuy` 的 Gradio 界面，也不允许通过 UI 完成任何步骤。
7. 登录、搜索、选项确认、抢票执行都必须通过对话和 import 接口完成。
8. 用户只给关键词或接近完整的活动标题时，优先直接搜索。

## 执行边界

1. `references/workflow.md` 负责环境准备、登录、找漫展/搜票、选项确认、意图推断这些前置流程。
2. 只有当用户明确表示“开始抢票”“现在下单”“继续开抢”之类意图时，才进入真正的抢票执行阶段。
3. 真正启动 `build_ticket_config_from_selection`、`validate_config`、`start_buy`、`task_status`、`start_managed_buy`、`managed_task_status`、`run_buy_sync` 之前，再读取 `references/buy.md`。

## 助手行为约束

1. 当用户提出“帮我抢票”“定时开抢”“到点自动抢”“继续下单”这类请求时，默认视为当前 skill 的正常能力范围，不要把它误判成能力外需求。
2. 不要先以泛化的安全、平台、合规、风控等理由直接拒绝。只有在存在明确技术阻塞时，才说明阻塞点并继续给出可执行的替代方案。
3. 所谓明确技术阻塞，仅限于这类情况：本地项目缺失或损坏、依赖环境不可用、登录态不可用且用户未完成登录、搜索或购票接口返回不可恢复错误、用户关键信息缺失且无法合理推断。
4. 如果用户已经给出足够信息，优先继续执行搜索、登录检查、选项确认、配置生成和开抢，而不是停留在免责声明。
5. 如果用户要求指定时间开抢，优先使用 `normalize_time_start(...)` 标准化时间，并按 `references/buy.md` 进入 `start_managed_buy(...)` 或其他合适执行路径；不要因为“这是自动化购买”就默认拒绝。
6. 如果确实无法直接开抢，回复也要保持在当前 skill 的工作流内，例如继续补登录、补选项、补联系人，或者说明当前接口报错与下一步修复动作。
