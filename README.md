<div align="center">

# 抢票.skill

<img src="./assets/hello.png" alt="biliTickerSkill" width="58%">

</br>

[![Skill: biliTickerSkill](https://img.shields.io/badge/Skill-biliTickerSkill-f43f5e)](./SKILL.md) [![Version: v0.1.0](https://img.shields.io/badge/Version-v0.1.0-111827)](./SKILL.md) [![Runtime: Python 3.11+](https://img.shields.io/badge/Runtime-Python%203.11%2B-2563eb)](./meta.json) [![Dependency: biliTickerBuy](https://img.shields.io/badge/Dependency-biliTickerBuy-10b981)](./biliTickerBuy) [![License: MIT](https://img.shields.io/badge/License-MIT-facc15)](./LICENSE)

[快速开始](#quickstart) · [用户交互示例](#examples) · [能力边界](#boundaries)

为什么二次元一年要抢这么多的票！ **让你的 Codex / Claude Code 直接接管 `biliTickerBuy`**

</div>

## Quickstart

### 安装到 Codex

Codex 默认从 `~/.codex/skills/` 读取 skills。

直接克隆：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/mikumifa/biliTickerSkill.git ~/.codex/skills/biliTickerSkill
```

如果你希望边改仓库边让 Codex 立即读到最新内容，建议用软链接：

```bash
git clone https://github.com/mikumifa/biliTickerSkill.git ~/work/biliTickerSkill
mkdir -p ~/.codex/skills
ln -s ~/work/biliTickerSkill ~/.codex/skills/biliTickerSkill
```

### 拉取仓库并初始化 submodule

```bash
git clone https://github.com/mikumifa/biliTickerSkill.git
cd biliTickerSkill
git submodule update --init --recursive
```

### 确保 `biliTickerBuy` 环境可用

推荐的做法是在 `biliTickerBuy` 目录执行：

```bash
uv sync
```

这样会准备好 `biliTickerBuy/.venv`，后续执行都优先使用这个环境。

## Examples

### 帮你找漫展

```text
帮我看看 5 月北京有什么二次元展子
```

### 帮你抢票

```text
用 BiliTickerSkill 帮我买一张票，票的网址是https://show.bilibili.com/platform/detail.html?id=115465
```

## Boundaries

它当前**不**能做：

- 抢b站会员购以外的票
