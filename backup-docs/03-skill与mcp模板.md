# skill 与 mcp 模板

> **这是备份，与 `skills/init-environment/SKILL.md` 第四节的工具部分同源。**
> **活版本是那个 skill；这份只是备份，不参与安装。**

装在哪、怎么装见 `01-全局AGENTS模板.md` 与仓库 README。**本文只回答一件事：每个角色用哪个 skill、连哪个 MCP。**

## 一、先装通用工具

**先装通用的（skills 市场 + 常用 MCP），再单独接两个工具：OpenDesign 与 Archify。**

```bash
# 技能来源
/plugin marketplace add anthropics/skills
/plugin marketplace add obra/superpowers

# 产品角色的提问技能（只装这两个，不装 grill-with-docs）
npx skills@latest add mattpocock/skills --skill=grill-me --skill=grilling --agent opencode --global

# MCP 按需添加
opencode mcp add context7 --global --url https://mcp.context7.com/mcp
opencode mcp add gitlab --global --url https://gitlab.example.com/api/v4/mcp
```

验证：`opencode mcp list` 里已配的 server 显示 `connected`；`ls ~/.agents/skills/` 能看到 `grill-me` 与 `grilling`。

**`grill-with-docs` 不装。** 它会往仓库根写 `GLOSSARY.md`、往它自己的 `docs/adr/` 写 ADR，和我们的 `docs/02-产品/05-产品术语表/01-产品术语表.md` / `docs/03-架构/07-ADR重要决策/` 是两套。两套术语表同时存在比没有更糟。

**`grill-me` 的已知限制**：它的 frontmatter 带 `disable-model-invocation: true`，只能由人在会话里手动 `/grill-me` 触发，**子代理自己调不动**。所以 `product-workflow` 的 §4.3 把 grilling 的协议（设计树、frontier、每问带推荐答案、frontier 空才收工）内联进了文档——skill 触发不了也能按文档做。

### 1.1 每个角色用哪个 skill、连哪个 MCP

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

**Archify 的 `finalize` 必带 `--out-dir docs/99-tmp/<角色>-<任务>/03-receipts/`**，否则 4 个 json 收据会落在业务目录里。

### 1.2 OpenDesign（设计角色出原型）

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

### 1.3 Archify（架构角色与运维角色出图）

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
  candidate.json 03-架构/02-架构图示/01-<短名>_总览_v<版本>.html --quality showcase --json
```

**四个校验全过才算交付。** 过了还要看返回里的 `visualReviewRecommendation`——有交叉连线就按它给的 hints 调坐标重跑，最多两轮。**校验过了不等于图好看。**

| 项 | 说明 |
|---|---|
| 产物 | **单个自包含 HTML**（约 850 KB），零依赖，`file://` 直接打开 |
| 附带 | `.finalize.json` + `.browser-check.json` 交付凭据 |
| 导出 | PNG / JPEG / WebP / SVG（双主题）/ WebM（带运动） |
| 运行依赖 | Node.js ≥ 18，无其他依赖 |
| 输出位置 | 架构角色 → `docs/03-架构/02-架构图示/`、`03-架构/03-状态机/`、`03-架构/06-ER图/`（三者恒为目录，文件名带序号，不建 `index.html`）；运维角色 → `docs/06-运维/02-部署拓扑/` |

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
| 架构图示（逻辑架构 + 系统上下文） | `architecture` | `03-架构/02-架构图示/01-<短名>_总览_v<版本>.html` | 架构角色 |
| 状态机 | `lifecycle` | `03-架构/03-状态机/01-<实体>_状态机_v<版本>.html`（每个实体一张，恒为目录） | 架构角色 |
| ER 图（全字段表结构） | `erd` | `03-架构/06-ER图/01-<域>_ER图_v<版本>.html`（一张 8–12 表） | 架构角色 |
| 部署拓扑（测试 + 生产，各一个区域 + 私网） | `architecture` + `engineering_profile: deployment-ownership` | `06-运维/02-部署拓扑/01-<环境>_部署拓扑.html` | 运维角色 |

**一次 `finalize` 只能出一种图。** Archify 的 `meta.views` 是同一张图的聚焦切换（比如只高亮某几个节点），不是把架构图和状态机拼在一起——两者 schema 完全不同。所以架构、状态机、ER 是三个独立文件。

**流程时序图仍然用 Mermaid 写在 `03-架构/01-系统架构/01-*.md` 里。** 它数量多、改得勤、要对着读，做成 HTML 不划算。
