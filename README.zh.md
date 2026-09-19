<div align="center">

# I am not useless _(i-am-not-useless)_

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Agent-Agnostic](https://img.shields.io/badge/Agent-Agnostic-blueviolet)](https://skills.sh)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)
[![Standard-Readme](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=badge)](https://github.com/RichardLitt/standard-readme)

**把用户当作真实上下文的来源与决策者，而不是被动的命令发出者。**

这是一个可移植的 [Agent Skills](https://agentskills.io/specification) 技能包，用来阻止智能体在「只有用户知道真相」的时候凭空猜测。它定义了：什么时候该问、怎么问、什么算证据、以及什么时候应该停下来把决定交还给用户。

</div>

[English](README.md) | 简体中文

## 快速安装

```
npx skills add NaivG/i-am-not-useless
```

## 目录

- [背景](#背景)
- [安装](#安装)
- [用法](#用法)
- [行为契约](#行为契约)
- [维护者](#维护者)
- [致谢](#致谢)
- [贡献](#贡献)
- [许可证](#许可证)

## 背景

智能体会以两种相反的方式失败：要么甩出一份完整的「可复现环境登记表」来审问用户，要么默默编造一个用户五秒就能回答的事实。两者都把用户当成了不可靠的输入设备。

本技能所编码的前提正相反：**用户并非完全不可靠，他们只是不知道如何专业地表达。** 智能体应当先推断真正需要什么，接受用户真正拿得出来的证据——截图、照片、日志、命令输出、一句话、甚至只是「A 还是 B」——并让认识收敛于现实，而不是收敛于自己此前的叙述。

它也划出了智能体无权替用户决定的地方：成本与性能、临时绕行与彻底修复、兼容与破坏性变更。这些应当被摆成取舍，交还给用户。

同样地，它也划出了智能体无权去问的地方：当前目录、文件编码、一个可逆的格式默认值、一项不影响下一步的细节。每一次提问都在消耗用户的注意力，所以一个智能体本可以自己回答的问题，是缺陷，而不是尽责。

概括地说，它的行为是：每轮只提一个阻塞性问题；不问任何工具能读取、检测或可以放心采用默认值的信息；每个问题都带出口（`idk`／`skip`／`you decide`）；不可逆操作在执行前必须说明对象、范围与回滚方式。

## 安装

技能包本身不包含任何代码，仅提供指令文本。

### 推荐：`skills` CLI

这是开放 Agent Skills 生态的标准安装器，覆盖 75 个以上智能体。使用 skills CLI 可直接安装：

```sh
npm i -g skills@latest
npx skills add NaivG/i-am-not-useless
```

加 `-g` 装到本机所有项目（默认只装当前项目），用 `-a` 指定目标智能体，用 `-y` 跳过交互确认：

```sh
npx skills add NaivG/i-am-not-useless -g -a claude-code -a codex -y
npx skills add NaivG/i-am-not-useless --list   # 只列出技能，不安装
```

### Agent 安装

直接与你现有的 Agent 对话：

```prompt
为当前的框架安装技能 [i-am-not-useless](https://github.com/NaivG/i-am-not-useless)
```

### 手动克隆

目录名必须保持为 `i-am-not-useless`，因为 Agent Skills 规范要求目录名与 `SKILL.md` 中的 `name` 字段一致：

```sh
# 项目级
mkdir -p .claude/skills && cd .claude/skills
git clone https://github.com/NaivG/i-am-not-useless i-am-not-useless

# 用户级
mkdir -p ~/.claude/skills && cd ~/.claude/skills
git clone https://github.com/NaivG/i-am-not-useless i-am-not-useless
```

Codex 同理，克隆到 `${CODEX_HOME:-$HOME/.codex}/skills/` 即可。

### 更新

执行 `npx skills update i-am-not-useless`；手动克隆的目录里执行 `git pull` 即可。本技能不保存任何状态，更新只会替换指令文本。

## 用法

本技能**只在用户显式调用时激活**，智能体不会自行加载。当任务依赖只有用户能观察到或能决定的信息时，按名称调用：

```text
/i-am-not-useless  打印机没有响应，帮我排查。
/i-am-not-useless  这次迁移该选哪条路——停机还是双写？
```

接下来变化的不是语气，而是交互的形状：

- 智能体先自行解决能解决的部分，然后只提最小的那个阻塞问题——绝不问工具能读取、能检测或已有安全默认值的信息。
- 问题可以用大白话回答，并会说明什么样的回答是有用的。
- 多模态证据是一等公民：截图、照片、日志、终端输出、配置文件。
- 索取证据时附带打码提醒，且把范围收窄到容易打码的程度。
- 不可逆操作在执行前确认，并说明对象、范围与回滚方式。
- 用户的纠正被视为证据，而不是需要辩赢的攻击。

它运行的循环：

```text
观察 → 推断 → 行动 → 验证 → 信息够了吗？ → 够：继续
                               不够：找出最小的缺失事实 → 提问
                                    → 收到证据 → 更新模型 → 继续
```

## 行为契约

文件真正约束智能体遵守的规则，以及它们可被观察到的结果：

| 规则 | 可观察的结果 |
| --- | --- |
| 晚问，但问得果断 | 第一个问题出现之前，可逆的工作已经完成。 |
| 能自己解决的不问 | 目录、编码、工具输出和安全默认值都不会去向用户索取。 |
| 每轮只问一个阻塞性问题 | 紧密耦合的子问题算一个；彼此独立的问题列表不算。 |
| 每个问题都带出口 | `idk`、`skip`、`you decide` 都会被妥善处理，绝不重复追问。 |
| 不可逆操作先确认 | 删除、付款、生产变更、硬件改动、隐私数据。 |
| 证据高于假设 | 文档、工具输出与用户反馈在收敛之前同等重要。 |
| 不做「固化假设」的测试 | 只测不变量；不把旧环境的产物升级为新需求。 |
| 不臆造文档 | 拿不到的文档就去要，绝不编造内容。 |
| 专业不等于啰嗦 | 一次简短的请求，说明为什么、要什么、怎么给、之后会怎样。 |

## 维护者

[NaivG](https://github.com/NaivG)。

## 致谢

- Agent Skills 格式: [agentskills.io](https://agentskills.io/specification) 
- Claude Code 的技能加载机制:  [Anthropic](https://code.claude.com/docs/en/skills)
- `skills` CLI 及其发现规则: [Vercel Labs](https://github.com/vercel-labs/skills) / [skills.sh](https://www.skills.sh/docs/cli)
- README 规范: [Standard Readme](https://github.com/RichardLitt/standard-readme)

## 贡献

欢迎在 [GitHub Issues](https://github.com/NaivG/i-am-not-useless/issues) 提问或报告失败案例。接受 Pull Request，改动越小越聚焦，合并越快。

提 PR 之前：保持 `SKILL.md` 在 500 行以内，保持 frontmatter 符合 Agent Skills 规范，并尽可能沿用文件中已有的 Markdown 风格。

## 许可证

MIT © NaivG。详见 [LICENSE](LICENSE)。
