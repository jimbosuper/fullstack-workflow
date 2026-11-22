---
name: 初始化全局环境
description: 安装或重新配置全局 Agent 环境（全局 AGENTS.md、7 个角色、必需的 skill 与 MCP）。
  用户说「装工作流」「配置 OpenChamber」「角色不见了」「怎么装」时使用。
---

# 初始化全局环境

**这个 skill 是安装手册。** 它不描述工作流怎么走，只回答「装什么、装到哪、怎么验」。

## 一、装之前确认四件事

| 项 | 怎么确认 | 没有会怎样 |
|---|---|---|
| OpenChamber 已安装 | `opencode --version` | 后面全部做不了 |
| 至少配好一个 provider | 会话里输入 `/models`，能看到模型 | 角色装上了但跑不起来 |
| Git 配了真人身份 | `git config --global user.name` 有值 | 提交记录追不到人 |
| 目标项目已 `git init` | 项目目录里有 `.git/` | 分支、提交、审查全失效 |

**缺任何一项就停下，先补齐再装。** 不要「装完再说」。

## 二、装什么

| # | 装什么 | 装到哪 | 来源 |
|---|---|---|---|
| 1 | 全局 AGENTS.md | `~/.config/opencode/AGENTS.md` | `templates/AGENTS.md` |
| 2 | 7 个角色 | `~/.config/opencode/agents/` | `agents/*.md` |
| 3 | 10 个 skill | `~/.agents/skills/` 或 `~/.config/opencode/skills/` | `skills/` |
| 4 | 必需的 MCP | 见第四节 | — |

## 三、装的三步

### 3.1 装 skill

```bash
npx skills add <仓库> -a opencode
```

装到哪取决于 `-a` 参数。要跨 agent 用就加多个：`-a opencode -a codex -a zcode`。

### 3.2 装角色（OpenCode 专用）

```bash
cp agents/*.md ~/.config/opencode/agents/
```

### 3.3 装全局 AGENTS.md

```bash
cp templates/AGENTS.md ~/.config/opencode/AGENTS.md
```

**如果已有 AGENTS.md，先看当前内容再决定覆盖还是合并**——直接覆盖会冲掉用户自己写的规则。

## 四、MCP 与 CLI 工具

| 工具 | 类型 | 装法 | 谁用 |
|---|---|---|---|
| `open-design` | MCP + CLI | 见 README 的 OpenDesign 一节 | 设计 |
| `archify` | skill + CLI | `npx skills add archify.si` | 架构 / 运维 |
| `grill-me` | skill | `npx skills add mattpocock/skills --skill=grill-me --skill=grilling -a opencode` | 产品 |
| 飞书项目 / GitLab / Context7 / Postgres / Sentry / 阿里云 | MCP | `opencode mcp add <name> --global --url <url>` | 各角色 |

**后端 MCP（Postgres / Sentry / 阿里云）按项目实际技术栈装，用不到就不装。**

## 五、装完怎么验

按顺序做，**每一步都要真实输出**。前四步机器能验，第五步只能人验。

**1. 角色装上了**

```bash
ls ~/.config/opencode/agents/ | wc -l    # 应为 7
```

**2. skill 装上了**

```bash
ls ~/.agents/skills/ | grep -c workflow   # 应为 8（7 角色 + full-workflow）
```

**3. 全局 AGENTS.md 在**

```bash
head -3 ~/.config/opencode/AGENTS.md
```

**4. MCP 连上了**

```bash
opencode mcp list | grep connected
```

**5. 人能派出去**（这一步机器验不了）

新开一个会话，说「用产品角色写个 PRD」。**能派出去才算装好。**

## 六、重装与升级

**先看现状，再动手。**

```bash
ls ~/.config/opencode/agents/
```

| 情况 | 怎么做 |
|---|---|
| 完全没装 | 按第三节全装 |
| 已有旧版（旧角色名） | **先删旧文件**，再装新的。新旧并存会让模型派错 |
| 只想补缺 | 只拷缺的文件，不覆盖已有的 |

**不许盲目 `cp -r` 覆盖。** 用户手改过的配置会被冲掉。

## 七、卡住时

| 现象 | 原因 | 怎么修 |
|---|---|---|
| 角色装上了但调不出来 | 缺 `mode: subagent` | 检查 frontmatter |
| 主 agent 自己把活干完了 | `description` 太笼统 | 写清「什么时候用」 |
| 技能不触发 | `description` 与实际任务不匹配 | 改 description，写清触发场景 |
| MCP 显示 needs authentication | 远程 server 要 OAuth | 会话里跑 `/mcps` 登录 |
| 子代理读不到文件 | 路径写成了相对当前目录 | 一律用项目根开始的相对路径 |
| OpenDesign 连不上 | daemon 端口变了 | 重查端口，桌面版必须开着 |
| Archify 图不好看但门都过了 | 有交叉连线没修 | 看 `visualReviewRecommendation` 的 hints |
