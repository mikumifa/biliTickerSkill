---
name: BiliTickerSkill
description: 当用户想让助手帮他找 Bilibili 会员购上的漫展、演出或其他活动，或基于本地 biliTickerBuy 项目继续准备和执行抢票流程时使用此 skill，包括搜索候选活动、检查环境、确认登录、引导用户选择票档并生成或执行正确的抢票流程。
version: "0.1.0"
user-invocable: true
---

# BiliTickerSkill

这个 skill 是本地 `biliTickerBuy` 项目的编排层，不重新实现抢票逻辑。

它不仅能在用户已经给出活动链接后继续抢票，也能在用户只给关键词、IP 名、角色名、城市、时间范围等线索时，先帮用户找漫展或其他会员购活动，再继续后续流程。

`biliTickerBuy` 作为 git submodule 放在 `./biliTickerBuy`。如果仓库是新拉取的，先初始化 submodule。

触发这个 skill 后优先阅读：

- `references/workflow.md`
- `references/search.md`
- `references/buy.md` 只在真正准备开始抢票时再读取
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
```

## 规则

1. 不要把真实 cookie、token、手机号、实名信息写入这个 skill 仓库。
2. 如果 skill 根目录下的 `biliTickerBuy` 不存在，先执行 `git submodule update --init --recursive`。
3. 一个关键前置步骤是先在 `biliTickerBuy` 目录执行 `uv sync`，确保生成并更新 `biliTickerBuy/.venv`。
4. 如果 `biliTickerBuy/.venv` 已经存在且可用，可以直接复用，不需要重复安装。
5. 后续所有 Python 执行都优先使用 `biliTickerBuy/.venv` 里的解释器和依赖环境。
6. 默认假设后续就在 `biliTickerBuy/.venv` 里直接 `import interface as btb`，不需要再手动拼 `sys.path`。
7. 不允许打开原版 `biliTickerBuy` 的 Gradio 界面，也不允许通过 UI 完成任何步骤。
8. 登录、日期选择、购票人选择、地址选择都必须通过对话和 import 接口完成。
9. 登录默认给用户返回登录链接，让用户在手机上打开并完成登录，不需要依赖二维码。
10. 如果已经登录，必须把当前登录用户名显示给用户，并让用户确认这是不是他想使用的账号。
11. 对用户的自然语言回复要允许做有限推断，不要机械要求每一项都必须逐字填写。
12. 当用户表达不完整但意图明显时，可以先整理出 1 到 3 个最可能的理解，再让用户确认，不要直接报缺字段。
13. 具体的登录流程、自然语言补全和交互细节都以 `references/workflow.md` 为准。
14. 与用户对话时要尽量口语化、人性化，不要频繁要求用户按固定格式回复。
15. 登录成功后的 cookies 默认应持久化复用；新开一个 skill 时，应先检查现有登录态，而不是默认重新登录。
16. 当进入票种选择阶段时，第一次必须把活动关键信息一次性完整展示给用户，并优先用表格呈现。
17. 如果用户说的是“帮我找漫展”“看看最近有什么展”“上海下个月有什么原神展”这类需求，应视为当前 skill 的正常能力范围，先进入搜索和筛选流程，而不是说做不到。

## 执行边界

1. `references/workflow.md` 负责环境准备、登录、找漫展/搜票、选项确认、意图推断这些前置流程。
2. 只有当用户明确表示“开始抢票”“现在下单”“继续开抢”之类意图时，才进入真正的抢票执行阶段。
3. 真正启动 `build_ticket_config_from_selection`、`validate_config`、`start_buy`、`task_status`、`run_buy_sync` 之前，再读取 `references/buy.md`。
