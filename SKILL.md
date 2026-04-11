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
- `references/search.md`
- `assets/examples/tickets.template.json`

在 `uv` 准备好的环境里直接使用 Python import：

```python
import bilitickerbuy
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
6. 默认假设后续就在 `biliTickerBuy/.venv` 里直接 `import bilitickerbuy`，不需要再手动拼 `sys.path`。
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

## 抢票流程

1. 先确认 `biliTickerBuy/.venv` 是否存在；如果不存在，进入 `biliTickerBuy` 目录执行 `uv sync`。
2. 之后所有 Python 操作都使用 `biliTickerBuy/.venv`。
3. 通过 `import bilitickerbuy` 导入统一入口。
4. 先调用 `bilitickerbuy.get_login_state`。
5. 如果已经登录，先把当前登录用户名展示给用户，并询问“是否使用这个账号继续”。
6. 如果未登录，调用 `bilitickerbuy.start_qr_login` 生成登录信息；至少要把 `login_url` 返回给用户。
7. 把登录链接放进返回给用户的结果里，让用户直接在手机上打开完成登录。
8. 在同一个结果里再弹出一个明确选择：“已在手机打开并登录，继续检查登录状态” 或 “还没登录，稍后再试”。
9. 只有当用户明确选择“已在手机打开并登录”后，才调用 `bilitickerbuy.poll_qr_login` 检查登录结果；如果用户说还没登录，就停在这里等待。
10. 整个登录过程不打开任何 UI。
11. 登录成功后，再调用 `bilitickerbuy.get_login_state` 或直接读取登录结果，把当前用户名显示给用户确认。
12. 确认账号无误后，再调用 `bilitickerbuy.fetch_purchase_context`，传入活动 URL 或 `project_id`。
13. 第一次进入票种选择阶段时，把活动关键信息一次性完整展示给用户，并优先用表格呈现，再继续提问。
14. 把拿到的可选项展示给用户，而不是自己猜：

- 如果活动有多个日期，先展示 `sales_dates`
- 展示当前日期下的 `ticket_options`
- 展示当前账号下的 `buyers`
- 展示当前账号下的 `addresses`

15. 如果日期或票档不唯一，必须让用户明确选择，不能默认取第一项。
16. 让用户明确确认这些信息：票档、购票人、联系人、联系电话、收货地址。
17. 调用 `bilitickerbuy.build_ticket_config_from_selection` 生成最终配置。
18. 先运行 `bilitickerbuy.validate_config`。
19. 再用 `bilitickerbuy.start_buy` 启动后台任务。
20. 用 `bilitickerbuy.task_status` 轮询状态。
