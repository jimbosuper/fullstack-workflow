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

**装之前必须改两处：**

**① `model` 一行是占位，要换成你本机真有的模型。**

```yaml
model: <provider>/<model>#high     # ← 改这里
```

`#` 后是推理档位，可选值取决于你的模型定义。源文件给的默认档位：产品 `#low`、设计 `#high`、架构 `#high`、全栈开发 `#high`、测试 `#low`、审查 `#max`、运维 `#low`。

**② 机器上已有旧角色名时，先删旧的再装。** 新旧并存会让模型派错：

| 本仓库文件名 | 源文件（backup-docs 02）用的名字 |
|---|---|
| `product-workflow.md` | `product-manager.md` |
| `design-workflow.md` | `designer.md` |
| `architect-workflow.md` | `architect.md` |
| `develop-workflow.md` | `fullstack-dev.md` |
| `test-workflow.md` | `test-runner.md` |
| `review-workflow.md` | `code-reviewer.md` |
| `ops-workflow.md` | `ops-engineer.md` |

**`mode: subagent` 不能删**——不写会被当成主 agent，还抢走本该派给子代理的活。

### 3.3 装全局 AGENTS.md

```bash
cp templates/AGENTS.md ~/.config/opencode/AGENTS.md
```

**如果已有 AGENTS.md，先看当前内容再决定覆盖还是合并**——直接覆盖会冲掉用户自己写的规则。

**但旧版必须换掉。** 旧版写的是「四道门」、门 3 叫验收、用「已批准」；本仓库 7 个 skill 讲的是「八道门」、门 3 = 架构、门 7 = 验收、统一叫「放行」。**两套门并存，agent 会按旧的那套做事。** 认旧版看两个特征：搜得到「四道门」、搜不到「八道门」。

## 四、MCP 与 CLI 工具

**先装通用的（skills 市场 + 常用 MCP），再单独接两个工具：OpenDesign 与 Archify。**

```bash
# 技能来源
/plugin marketplace add anthropics/skills
/plugin marketplace add obra/superpowers

# 产品角色的提问技能（只装这两个，不装 grill-with-docs）
npx skills@latest add mattpocock/skills --skill=grill-me --skill=grilling -a opencode

# MCP 按需添加
opencode mcp add context7 --global --url https://mcp.context7.com/mcp
opencode mcp add gitlab --global --url https://gitlab.example.com/api/v4/mcp
```

验证：`opencode mcp list` 里已配的 server 显示 `connected`；`ls ~/.agents/skills/` 能看到 `grill-me` 与 `grilling`。

**`grill-with-docs` 不装。** 它会往仓库根写 `GLOSSARY.md`、往 `docs/adr/` 写 ADR，和我们的 `docs/01-产品/04-产品术语表.md` / `docs/03-架构/07-ADR重要决策/` 是两套。两套术语表同时存在比没有更糟。

**`grill-me` 的已知限制**：它的 frontmatter 带 `disable-model-invocation: true`，只能由人在会话里手动 `/grill-me` 触发，**子代理自己调不动**。所以 `product-workflow` 的 §4.3 把 grilling 的协议（设计树、frontier、每问带推荐答案、frontier 空才收工）内联进了文档——skill 触发不了也能按文档做。

### 4.1 每个角色用哪个 skill、连哪个 MCP

**表里写死的是常用技能。要做的事跟表里对不上时，用 `find-skills` 现找，不硬扛。**

| 角色 | skill | MCP |
|---|---|---|
| **产品** | `mattpocock:grill-me`（提问主力，见上面的限制）、`superpowers:brainstorming`（起草候选方案）、`superpowers:writing-plans`（落成带验收标准的文档） | 飞书项目、Context7 |
| **设计** | `find-skills` | **`open-design`（出原型主力）**、`od` CLI（兜底）、Playwright |
| **架构** | `superpowers:writing-plans`（写 ADR）、**`archify`（出图，skill + CLI）** | GitLab、Context7、Postgres |
| **全栈开发** | `superpowers:test-driven-development`、`superpowers:systematic-debugging`、`superpowers:executing-plans` | GitLab、Context7、Postgres、Chrome DevTools |
| **测试** | `anthropics:webapp-testing`（浏览器验证）、`superpowers:verification-before-completion` | GitLab、Postgres |
| **审查** | `code-review`（内置，只用于代码）、`superpowers:verification-before-completion` | GitLab、Serena、Semgrep |
| **运维** | `anthropics:webapp-testing`（冒烟）、`superpowers:verification-before-completion`、**`archify`（部署拓扑）** | GitLab、Sentry、阿里云 |

**后端 MCP（Postgres / Sentry / 阿里云）按项目实际技术栈装，用不到就不装。**

**Archify 的 `finalize` 必带 `--out-dir docs/99-tmp/<角色>-<任务>/receipts/`**，否则 4 个 json 收据会落在业务目录里。

### 4.2 OpenDesign（设计角色出原型）

**下面两个工具不走上面那套**：OpenDesign 是 MCP + CLI，Archify 是 skill + CLI，各有各的装法。

官方 `od mcp install opencode` 生成的配置**不能直接用**，两个坑：`od` 在 macOS 上被 `/usr/bin/od`（八进制 dump）占住；daemon 端口随机，不一定是 7456。

```bash
# 1. 打开桌面版（daemon 必须开着）
open -a "Open Design"

# 2. 查实际端口
lsof -nP -iTCP -sTCP:LISTEN | grep -i design
```

```jsonc
// 3. 写进 ~/.config/opencode/opencode.json 的 mcp 段
"open-design": {
  "type": "local",
  "command": [
    "/Users/<你>/.local/bin/node",
    "/Applications/Open Design.app/Contents/Resources/app/prebundled/daemon/daemon-cli.mjs",
    "mcp",
    "--daemon-url", "http://127.0.0.1:<上一步查到的端口>"
  ],
  "enabled": true
}
```

**三个必须**：`node` 用绝对路径；`daemon-cli.mjs` 用绝对路径；`--daemon-url` 写真实端口。少一样就连不上。

**CLI 兜底**（MCP 不通时等价可用）：

```bash
export OD_DAEMON_URL=http://127.0.0.1:<端口>
node "/Applications/Open Design.app/Contents/Resources/app/prebundled/daemon/daemon-cli.mjs" project list --json
```

验证：重启 OpenChamber，让它调 `list_projects`，能返回项目列表（空列表也算通）。

**端口会变。** 每次重开桌面版端口都可能不同，连不上时先重新查端口改配置。

### 4.3 Archify（架构角色与运维角色出图）

**它是 skill + Node CLI，不是 MCP。** skill 让 agent 知道怎么画，CLI 负责校验和渲染。

**走代理，否则克隆会 300 秒超时。** 下面按 Clash 默认端口 7897 写，你自己的端口用 `nc -z 127.0.0.1 7897` 之类确认：

```bash
export https_proxy=http://127.0.0.1:7897
export http_proxy=http://127.0.0.1:7897

# 全局装（推荐）
npx -y skills@latest add https://archify.si --skill archify --agent opencode --global --copy --yes
```

**源必须是 `https://archify.si`，不是 `archify.si`。** 前者走 well-known 端点只拉 skill 本体（203 个文件），后者会被当成 git 仓库去克隆，直接报 `repository 'archify.si' does not exist`。也不要写 `tt-a1i/archify`——仓库 141 MB，大概率超时。

装到 `~/.agents/skills/archify`。

**验证：**

```bash
node ~/.agents/skills/archify/bin/archify.mjs doctor
```

20 多项全 `[ok]` 才算装好（Node 版本、11 个 renderer、schema、viewer 运行时）。

**怎么用：** 填一份 JSON 描述结构，跑 `finalize`，它做四个校验（schema / 交付 / 产物 / 真实浏览器渲染）。

```bash
node ~/.agents/skills/archify/bin/archify.mjs finalize architecture \
  candidate.json 02-架构图示.html --quality showcase --json
```

**四个校验全过才算交付。** 过了还要看返回里的 `visualReviewRecommendation`——有交叉连线就按它给的 hints 调坐标重跑，最多两轮。**校验过了不等于图好看。**

| 项 | 说明 |
|---|---|
| 产物 | **单个自包含 HTML**（约 850 KB），零依赖，`file://` 直接打开 |
| 附带 | `.finalize.json` + `.browser-check.json` 交付凭据 |
| 导出 | PNG / JPEG / WebP / SVG（双主题）/ WebM（带运动） |
| 运行依赖 | Node.js ≥ 18，无其他依赖 |
| 输出位置 | 架构角色 → `docs/03-架构/02-架构图示.html`（或 `02-架构图示/`）、`03-状态机/`、`06-ER图/`（后两个恒为目录，文件名带序号，不建 `index.html`）；运维角色 → `docs/06-运维/02-部署拓扑.html`（或 `02-部署拓扑/`） |

**10 种图型**：架构、工作流、时序、数据流、生命周期、ERD、类图、树、时间线、瀑布。

**两条硬约束：**

| 约束 | 值 |
|---|---|
| 节点数 | **8–12 个**，超过就拆图 |
| 画布宽 | ≤ 1240px（宽高比 ≥ 1.55 时可到 1790px）；再宽文字会小到读不了 |

**它不是绘图编辑器，也不是 Mermaid 换皮。** 官方定位是「把技术意图变成可沟通的产物」——不做拖拽精修，不做托管分享，不是无限画布。复杂到 40 个节点以上，该拆成多张图。

**我们只用它画三种东西：**

| 图 | Archify 类型 | 产物 | 谁产出 |
|---|---|---|---|
| 架构图示（逻辑架构 + 系统上下文） | `architecture` | `02-架构图示.html` | 架构角色 |
| 状态机 | `lifecycle` | `03-状态机/1-<实体>.html`（每个实体一张，恒为目录） | 架构角色 |
| ER 图（全字段表结构） | `erd` | `06-ER图/<N>-<域>.html`（一张 8–12 表） | 架构角色 |
| 部署拓扑（测试 + 生产，各一个区域 + 私网） | `architecture` + `engineering_profile: deployment-ownership` | `02-部署拓扑.html` | 运维角色 |

**一次 `finalize` 只能出一种图。** Archify 的 `meta.views` 是同一张图的聚焦切换（比如只高亮某几个节点），不是把架构图和状态机拼在一起——两者 schema 完全不同。所以架构、状态机、ER 是三个独立文件。

**流程时序图仍然用 Mermaid 写在 `01-系统架构.md` 里。** 它数量多、改得勤、要对着读，做成 HTML 不划算。

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

**3. 全局 AGENTS.md 在，且是完整版**

```bash
grep -c "^## " ~/.config/opencode/AGENTS.md   # 应为 12
grep -c "八道门" ~/.config/opencode/AGENTS.md # 应 ≥ 1；只有「四道门」的是旧版
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
