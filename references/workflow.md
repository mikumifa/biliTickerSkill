# 工作流

## 目标

这个 skill 是本地 `biliTickerBuy` 项目的轻量编排层。

当前能力范围包含两类入口：

1. 用户已经给出活动链接或项目 ID，直接继续登录确认、读详情、选票和抢票。
2. 用户还没给链接，只是说想找某个漫展、某个 IP 的活动、某个城市最近有什么展，这时先帮用户搜索候选活动，再继续后面的流程。

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
.\biliTickerBuy\.venv\Scripts\python.exe -c "import interface as btb; print('ok')"
```

## 优先使用 import

推荐直接 import：

```python
import interface as btb
```

## 当前暴露的库接口

上游现在提供这些函数：

- `start_qr_login`
- `poll_qr_login`
- `login_with_cookies`
- `search_tickets`
- `format_ticket_search_results_text`
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

1. 先调 `btb.get_login_state(...)`。
2. 如果已经登录，把当前登录用户名显示给用户，并让用户确认是否使用这个账号继续。
3. 如果 `logged_in` 是 `false`，先调 `btb.start_qr_login(...)` 获取登录信息。
4. 把 `login_url` 放进返回给用户的结果里，让用户在手机上打开。
5. 在同一个结果里明确让用户选择“已在手机打开并登录，继续检查”或“还没登录，稍后再试”。
6. 只有当用户确认已登录时，再调 `btb.poll_qr_login(...)` 等待登录完成。
7. 登录完成后，再把当前登录用户名显示给用户确认。
8. 然后调 `btb.fetch_purchase_context(url_or_project_id, cookies=..., selected_date=None)`。
9. 第一次进入票种选择阶段时，要一次性把完整关键信息发给用户，不要拆成多轮零散补充。
10. 这一步优先使用表格，把活动、日期、票档、购票人、地址都排清楚。
11. 如果 `sales_dates` 非空且用户还没选日期，先把日期列出来让用户选。
12. 把 `ticket_options` 列出来，让用户明确选中具体票档。
13. 把登录账号下的 `buyers` 和 `addresses` 列出来，让用户明确选择。
14. 额外收集 `buyer` 和 `tel` 这两个联系人字段。
15. 用 `btb.build_ticket_config_from_selection` 生成最终配置。
16. 然后再做校验和启动。

真正开始执行抢票前，再参考 `references/buy.md`。如果还在登录、搜索、确认用户选择阶段，不要提前进入抢票执行 reference。

## 关键词找票

如果用户不是直接给链接，而是只给活动名字、关键词、IP 名称，或者直接说想找漫展，先参考 `references/search.md`。

原则是：

1. 先搜，再让用户确认目标活动。
2. 不要先要求用户自己去复制活动链接。
3. 搜索结果优先整理成文字格式发给用户。
4. `btb.search_tickets(...)` 如果发现当前未登录，会直接返回需要登录的结果；这时就停止搜索并进入登录流程，不允许改走公开网页。
5. “帮我找漫展”是当前 skill 明确支持的用法，不要把它误判成能力外需求。

## 首次展示格式

当第一次让用户选择票种时，应尽量一次性展示这些信息：

- 活动名称
- 日期信息
- 场地信息
- 票档列表
- 购票人列表
- 地址列表

优先用 Markdown 表格，而不是普通散文段落。

关于登录复用：

1. 登录成功后，优先复用当次拿到的 cookies。
2. 如果没有显式传 cookies，也应默认走持久化 cookie 存储。
3. 只有当 cookie 失效、用户明确要换号、或者当前登录账号不是用户想要的账号时，才重新登录。

额外规则：

1. 不允许打开原版 `biliTickerBuy` Gradio 页面。
2. 所有步骤都必须通过对话和 import 接口完成。
3. 默认优先返回登录链接，不要求必须展示二维码。
4. 对用户的不完整自然语言要允许做合理推断，再让用户确认，不要死板要求固定模板。

## 对话风格

和用户对话时，优先用自然、口语化的问法，不要像在让用户填表。

好的问法应该更像：

- `你想买普通票还是 VIP？`
- `联系电话我先默认用这个地址里的 1234567890，可以的话我就继续。`

不好的问法是：

- `请按以下格式回复：票档: 2`
- `请填写 联系人: xxx`
- `请填写 联系电话: xxx`

原则是：

1. 能推断就先推断。
2. 能确认就先给默认理解让用户确认。
3. 只有真的有歧义，才把候选方案抛给用户选。

## 票种推测

当用户没有严格按固定格式回答时，不要立刻打回重填。

优先按下面策略理解：

1. 票档：优先按票档名称匹配，例如“普通车票”直接匹配对应票档。
2. 购票人：优先按姓名匹配，如果只有一个明显候选，直接采用。
3. 联系人：如果用户没写，优先沿用购票人姓名。
4. 联系电话：如果用户没写，优先取最终选中地址里的电话。
5. 地址：如果用户说“随便填一个”，优先选与购票人同名、或与联系人最匹配的地址。

如果有多个同样合理的解释，不要强行选死，而是返回 1 到 3 个候选方案给用户选。

例如：

```text
用户说：普通车票 xxx 收货地址随便填一个
```

可以返回：

```text
我理解你可能是下面几种意思，请选一个：

1. 票类型x / 购票人 xxx / 联系人 xxx / 联系电话 12345678900 / 地址 3
2. 票类型x / 购票人 xxx / 联系人 xxx / 联系电话 12345678900 / 地址 4
3. 票类型x / 购票人 xxx / 联系人 xxx / 联系电话 12345678900 / 地址 1
```

如果有一个最明显的默认解释，也可以直接先给默认值，再让用户确认是否需要修改。

推荐优先这样说：

```text
我先按这个理解继续：
普通车票，购票人是xxx，联系人也先用xxx，电话先用地址里的号码。
你看可以的话我就继续；如果你想换地址或者电话，直接跟我说。
```

## 安全执行模式

1. 先确认 `biliTickerBuy/.venv` 可用；不可用时先在 `biliTickerBuy` 目录运行 `uv sync`。
2. 根据用户参数或交互式选择构造配置，或者读取现成 JSON。
3. 如果用户只是确认信息，还不要开抢；真正开始执行时再去读 `references/buy.md`。
4. 运行时产物不要写回 git 跟踪的示例文件。
