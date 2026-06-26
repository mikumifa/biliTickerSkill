# 抢票执行

这个 reference 只在已经确认活动、日期、票档、购票人、联系人、联系电话、地址，并且准备真正开始抢票时再读取。

如果还停留在搜票、登录、确认账号、选择票档这些阶段，不要提前进入这里。

## 进入条件

只有同时满足下面条件，才进入抢票执行阶段：

1. 已经确认使用哪个登录账号。
2. 已经拿到完整的 `purchase_context`。
3. 用户已经确认日期和票档。
4. 用户已经确认购票人、联系人、联系电话、收货地址。
5. 已经准备好最终配置，且用户明确表示要开始抢票。

## 执行顺序

1. 调用 `btb.build_ticket_config_from_selection(...)` 生成最终配置。
2. 调用 `btb.validate_config(...)` 做校验。
3. 如果校验失败，直接把错误告诉用户，不要启动任务。
4. 如果校验通过，再根据任务形态选择执行接口。
5. 短时单次执行优先 `btb.run_buy_sync(...)`。
6. 长时间等待、跨轮查询、多开并行时优先 `btb.start_managed_buy(...)`。
7. 只有在同一个 Python 进程里立即轮询的短任务，才用 `btb.start_buy(...)` + `btb.task_status(...)`。
8. 一旦拿到支付链接，立刻通过结构化结果返回给用户，不要只留在日志里。
9. 当前接口返回里要区分三种支付字段：
   `order_id` 是订单号，
   `order_detail_url` 是订单详情页，
   `payment_code_url` 才是 `getPayParam` 拿到的真实二维码链接；
   `payment_qr_url` 仅保留为兼容旧调用方的别名。

## 接口建议

推荐优先按下面顺序使用：

```python
import interface as btb

config = btb.build_ticket_config_from_selection(purchase_context, selection)
validation = btb.validate_config(config)
```

如果 `validation.ok` 为 `True`，再继续：

```python
task = btb.start_managed_buy(
    config,
    runtime_options={
        "time_start": btb.normalize_time_start("2026-04-12T00:36"),
        "interval": btb.normalize_interval("500ms"),
    },
)
```

后续查询状态：

```python
status = btb.managed_task_status(task["run"]["run_id"])
```

如果用户明确要求停止已经创建的持久化任务，使用：

```python
cancel_result = btb.cancel_managed_buy(task["run"]["run_id"])
```

如果用户明确要求删除任务记录，或需要先清理旧任务再重建，使用：

```python
delete_result = btb.delete_managed_buy(task["run"]["run_id"])
```

如果任务仍在运行，且调用方明确接受“先停再删”，才允许：

```python
delete_result = btb.delete_managed_buy(task["run"]["run_id"], force=True)
```

如果你不希望处理跨轮询状态，也可以直接：

```python
result = btb.run_buy_sync(
    config,
    runtime_options={
        "show_qrcode": False,
    },
)
```

如果你确实要走原生命令行而不是 import 层，参数名要和当前 CLI 保持一致，例如：

```bash
python main.py buy ./ticket.json --time-start 2026-04-12T00:36 --interval 500 --no-show-qrcode
```

## 时间与轮询格式

- `time_start`
  推荐使用 `YYYY-MM-DDTHH:MM[:SS]`
  也允许 `HH:MM[:SS]`，但执行前必须标准化成完整时间
- `interval`
  最终统一成毫秒整数
  可接受 `500`、`500ms`、`0.5s`、`0.36m`

建议在真正开跑前先调用：

```python
normalized_time = btb.normalize_time_start("0:36")
normalized_interval = btb.normalize_interval("0.36m")

task = btb.start_managed_buy(
    config,
    runtime_options={
        "time_start": normalized_time,
        "interval": normalized_interval,
    },
)
```

## 持久化运行输出

如果使用 `start_managed_buy(...)`，每次运行都应该在 `btb_runs/<run_id>/` 里至少写出：

- `status.json`
- `result.json`
- `events.log`

其中：

- `status.json` 用于实时读状态
- `result.json` 用于读取最终支付链接和最终结论
- `events.log` 只是调试补充，不应成为支付结果的唯一出口

## 持久化任务生命周期

对于 `start_managed_buy(...)` 创建的任务，推荐按下面的生命周期处理：

1. `start_managed_buy(...)`
   创建任务并返回 `run_id`
2. `managed_task_status(run_id)`
   查询任务当前状态、支付链接和结果
3. `cancel_managed_buy(run_id)`
   当用户明确要求“停止”“取消”“先别抢了”时，结束任务并把状态写成 `cancelled`
4. `delete_managed_buy(run_id)`
   当用户明确要求“删除任务”“清理旧任务”时，删除对应 `btb_runs/<run_id>/` 目录

建议把“取消”和“删除”视为两个不同动作：

- `cancel_managed_buy(...)` 负责停止任务，并保留状态与结果记录
- `delete_managed_buy(...)` 负责删除持久化记录
- 如果任务还在运行，优先先 `cancel`，再 `delete`
- 只有在调用方明确接受强制清理时，才使用 `delete_managed_buy(..., force=True)`

## 规则

1. 不要在用户还没确认选择前就提前启动抢票。
2. 不要把“生成配置”和“真正开抢”混成一步。
3. `task_status(...)` 是进程内存态；如果用了 `start_buy(...)`，轮询必须留在同一个 Python 进程里。
4. 长任务、多开任务、跨回合任务不要只靠 `start_buy(...)`。
5. 如果只是需要一次执行完并拿结果，优先考虑 `run_buy_sync(...)`。
6. 支付链接不要只靠日志传递；应优先通过 `result.json` 或 `managed_task_status(...)` 返回。
7. 多开时每单都要使用独立 `run_id` 和独立运行目录。
8. 对于持久化任务，应该明确区分“开始”“查询”“取消”“删除”四种动作，不要把它们混在同一个接口语义里。
9. 删除任务前应先判断任务是否仍在运行；如果仍在运行，默认先取消，除非用户明确要求强制删除。
