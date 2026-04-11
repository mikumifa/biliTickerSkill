<div align="center">

# 抢票.skill

<img src="./assets/hello.png" alt="biliTickerSkill" width="58%">

</br>

[![Skill: biliTickerSkill](https://img.shields.io/badge/Skill-biliTickerSkill-f43f5e)](./SKILL.md) [![Version: v0.1.0](https://img.shields.io/badge/Version-v0.1.0-111827)](./SKILL.md) [![Runtime: Python 3.11+](https://img.shields.io/badge/Runtime-Python%203.11%2B-2563eb)](./meta.json) [![Dependency: biliTickerBuy](https://img.shields.io/badge/Dependency-biliTickerBuy-10b981)](./biliTickerBuy) [![Integration: import biliTickerBuy.interface](https://img.shields.io/badge/Integration-import%20biliTickerBuy.interface-f59e0b)](./references/workflow.md) [![License: MIT](https://img.shields.io/badge/License-MIT-facc15)](./LICENSE)

[快速开始](#quickstart) · [用户交互示例](#examples) · [能力边界](#boundaries)

为什么二次元一年要抢这么多的票！ **让你的 Codex / Claude Code 直接接管 `biliTickerBuy`，先帮你找漫展，再继续查票或抢票**

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

更推荐的做法是在 `biliTickerBuy` 目录执行：

```bash
uv sync
```

这样会准备好 `biliTickerBuy/.venv`，后续执行都优先使用这个环境。

## Examples

### 帮你找漫展

```text
用 BiliTickerSkill 帮我找一下上海最近的原神漫展
```

```text
帮我看看 5 月北京有什么二次元展子
```

这类请求也属于当前 skill 的能力范围。它不应该先让用户自己去翻网页找链接，而应该先基于关键词、城市、时间等线索搜索会员购活动，整理候选结果，再让用户确认目标活动。

### 帮你抢票

```text
用 BiliTickerSkill 帮我买一张票，票的网址是https://show.bilibili.com/platform/detail.html?id=115465
```

这个 skill 现在应该按阶段工作，而不是直接假设用户已经给全参数：

1. 先检查是否已登录，未登录时必须让用户扫码
2. 登录成功后再读取活动页面并列出可选日期、场次、票档
3. 读取当前账号下的购票人和地址
4. 如果用户不知道买哪一天或哪一档，就展示选项并让用户选择
5. 最后才生成配置并校验后启动抢票

它也应该能理解不那么规范的自然语言回复。例如用户说“普通车票 谢其卓 收货地址随便填一个”，skill 不应该机械地要求重填，而应该基于已有列表先推断联系人、电话和候选地址，再让用户确认。

## Boundaries

它当前**不**能做：

- 抢b站会员购以外的票

它当前**能**做：

- 帮用户按关键词、IP、城市、时间范围找 Bilibili 会员购上的漫展和其他活动
- 在用户确认目标活动后继续读取活动详情、整理票档并进入抢票流程

## 相关文件

- [SKILL.md](./SKILL.md)：skill 触发说明与运行规则
- [references/workflow.md](./references/workflow.md)：主要工作流说明

## 许可证

Skill 部分按当前仓库约定维护；`biliTickerBuy` 上游代码遵循其仓库内的 [MIT License](./biliTickerBuy/LICENSE)。
