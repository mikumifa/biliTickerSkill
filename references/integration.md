# 集成约定

## 为什么有这份文档

当前这个 skill 已经可以通过 Python import 直接驱动 `biliTickerBuy`。

但在真正执行前，应该先确保 `biliTickerBuy/.venv` 存在并可用。也就是说，环境准备本身也是集成约定的一部分。

## 集成前置步骤

1. 确认 `biliTickerBuy` 子模块已经初始化。
2. 进入 `biliTickerBuy` 目录执行 `uv sync`。
3. 确认生成了 `biliTickerBuy/.venv`。
4. 后续所有 Python 侧的调用，都优先使用这个 `.venv`。
