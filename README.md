<div align="center">

# 抢票.skill

<img src="./assets/hello.png" alt="biliTickerSkill" width="58%">

</br>
</br>

[![Skill: biliTickerSkill](https://img.shields.io/badge/Skill-biliTickerSkill-f43f5e)](./SKILL.md) [![Version: v0.1.0](https://img.shields.io/badge/Version-v0.1.0-111827)](./SKILL.md) [![Runtime: Python 3.11+](https://img.shields.io/badge/Runtime-Python%203.11%2B-2563eb)](./meta.json) [![Dependency: biliTickerBuy](https://img.shields.io/badge/Dependency-biliTickerBuy-10b981)](./biliTickerBuy) [![Integration: import bilitickerbuy](https://img.shields.io/badge/Integration-import%20bilitickerbuy-f59e0b)](./references/workflow.md) [![License: MIT](https://img.shields.io/badge/License-MIT-facc15)](./LICENSE)

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

## Examples

### 帮你抢票

```text
用 bili-ticker-buy skill帮我买一张票，票的网址是https://show.bilibili.com/platform/detail.html?id=115465
```

## Boundaries

它当前**不**能做：

- 抢b站会员购以外的票

## 相关文件

- [SKILL.md](./SKILL.md)：skill 触发说明与运行规则
- [references/workflow.md](./references/workflow.md)：import-first 集成方式
- [references/claude-code.md](./references/claude-code.md)：Claude Code 调用约定

## 许可证

Skill 部分按当前仓库约定维护；`biliTickerBuy` 上游代码遵循其仓库内的 [MIT License](./biliTickerBuy/LICENSE)。
