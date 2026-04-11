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
4. 如果校验通过，再调用 `btb.start_buy(...)` 或 `btb.run_buy_sync(...)`。
5. 如果走异步模式，再用 `btb.task_status(...)` 在同一个 Python 进程里轮询。
6. 一旦拿到支付链接或支付二维码，立刻返回给用户。

## 接口建议

推荐优先按下面顺序使用：

```python
import interface as btb

config = btb.build_ticket_config_from_selection(purchase_context, selection)
validation = btb.validate_config(config)
```

如果 `validation.ok` 为 `True`，再继续：

```python
task = btb.start_buy(config)
```

如果你不希望处理跨轮询状态，也可以直接：

```python
result = btb.run_buy_sync(config)
```

## 规则

1. 不要在用户还没确认选择前就提前启动抢票。
2. 不要把“生成配置”和“真正开抢”混成一步。
3. `task_status(...)` 是进程内存态；如果用了 `start_buy(...)`，轮询必须留在同一个 Python 进程里。
4. 如果只是需要一次执行完并拿结果，优先考虑 `run_buy_sync(...)`。
5. 抢票成功不等于结束；支付链接或支付二维码要尽快返回给用户。
