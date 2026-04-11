<div align="center">

# 抢票.skill

<img src="./assets/hello.png" alt="biliTickerSkill" width="58%">

</br>
</br>

[![Skill: bili-ticker-buy](https://img.shields.io/badge/Skill-bili--ticker--buy-f43f5e)](./SKILL.md) [![Version: v0.1.0](https://img.shields.io/badge/Version-v0.1.0-111827)](./SKILL.md) [![Runtime: Python 3.11+](https://img.shields.io/badge/Runtime-Python%203.11%2B-2563eb)](./meta.json) [![Dependency: biliTickerBuy](https://img.shields.io/badge/Dependency-biliTickerBuy-10b981)](./biliTickerBuy) [![Integration: import bilitickerbuy](https://img.shields.io/badge/Integration-import%20bilitickerbuy-f59e0b)](./references/workflow.md) [![License: MIT](https://img.shields.io/badge/License-MIT-facc15)](./LICENSE)

[快速开始](#quickstart) · [用户交互示例](#examples) · [能力边界](#boundaries)

为什么二次元一年要抢这么多的票！ **让你的 Codex / Claude Code 直接接管 `biliTickerBuy`**

</div>

## Quickstart

### 1. 拉取仓库并初始化 submodule

```bash
git clone https://github.com/mikumifa/biliTickerSkill.git
cd biliTickerSkill
git submodule update --init --recursive
```

### 2. 确保 `biliTickerBuy` 依赖可用

推荐直接使用 `biliTickerBuy` 自己的 Python 环境，或在同一个 Python 3.11+ 环境中安装它的依赖。

### 3. 在 Codex / Claude Code 中通过 import 调用

```python
import sys
from pathlib import Path

sys.path.insert(0, str(Path.cwd() / "biliTickerBuy"))

from bilitickerbuy import generate_ticket_config, validate_config, start_buy, task_status
```

## Examples

### 让 skill 生成配置

```text
用 $bili-ticker-buy 帮我生成一个 tickets.json。
项目 id 是 123，场次 id 是 456，票档 sku_id 是 789，买 1 张，单价 6800。
购票人叫张三，身份证号是 XXXXXXXXXXXXXX，联系人手机号是 13800000000。
收货地址 addr_id 是 1，地址写成“上海市浦东新区测试地址”。
cookies 从 ./biliTickerBuy/cookies.json 读取，先不要启动抢票，先帮我校验配置。
```

### 让 skill 校验现有配置

```text
用 $bili-ticker-buy 检查一下 ./workspace/tickets.json 是否能直接给 biliTickerBuy 使用。
不要启动任务，只告诉我缺了哪些字段，并给我一份修正后的标准结构。
```

### 让 skill 启动任务并持续汇报状态

```text
用 $bili-ticker-buy 读取 ./workspace/tickets.json 并启动抢票。
通过 import bilitickerbuy 调用，不要走 btb。
每隔 1 秒查询一次任务状态，把最近日志持续告诉我；
如果拿到支付二维码链接，也直接输出给我。
```

### 让 skill 处理完整流程

```text
用 $bili-ticker-buy 帮我完整处理一次抢票准备：
1. 从我给的参数生成 tickets.json
2. 校验配置
3. 用 biliTickerBuy 的 Python API 启动任务
4. 轮询任务状态
5. 如果失败，整理失败原因和最后几条日志给我
```

## Boundaries

这个 skill 现在优先做三件事：

1. 从参数生成标准 `tickets.json` 结构
2. 在真正启动前做结构化校验
3. 通过 `start_buy / task_status` 管理后台任务

它当前**不**追求：

- 重新实现 `biliTickerBuy` 的核心购票逻辑
- 强依赖 `btb` 命令行入口
- 先做一层很重的 HTTP 服务包装

如果后面确实需要跨进程调度、远程控制或统一任务池，再在 `biliTickerBuy` 的 Python API 之上包一层很薄的服务即可。

## Structure

```text
biliTickerSkill/
├── SKILL.md
├── meta.json
├── agents/
│   └── openai.yaml
├── assets/
│   └── examples/
│       └── tickets.template.json
├── references/
│   ├── workflow.md
│   ├── integration.md
│   └── claude-code.md
└── biliTickerBuy/      # git submodule
    └── bilitickerbuy/  # Python package
```

## 相关文件

- [SKILL.md](./SKILL.md)：skill 触发说明与运行规则
- [references/workflow.md](./references/workflow.md)：import-first 集成方式
- [references/claude-code.md](./references/claude-code.md)：Claude Code 调用约定
- [assets/examples/tickets.template.json](./assets/examples/tickets.template.json)：脱敏配置模板

## 许可证

Skill 部分按当前仓库约定维护；`biliTickerBuy` 上游代码遵循其仓库内的 [MIT License](./biliTickerBuy/LICENSE)。
