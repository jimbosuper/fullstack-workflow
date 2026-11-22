# fullstack-workflow

**一套「AI + 人」的端到端交付规范。** 它把产品、设计、架构、开发、测试、审查、运维七个角色写成可执行的岗位规则，再用八道门把人和 AI 的分工钉死。

```text
10 个 skill  +  7 个角色  +  1 份全局约束  +  2 份模板
```

仓库地址：`git@github.com:jimbosuper/fullstack-workflow.git`

---

## 零、它解决什么问题

只靠一句「帮我做个功能」，模型会一路往下猜，最后交付的东西没人敢上线。这个仓库把四个失效点各自堵住：

| 失效点 | 堵法 |
|---|---|
| 需求没问清就动手 | 产品角色必须先问，**零问询的 PRD 一律视为草稿**，不许进下游 |
| AI 自己把活干完还自己签字 | **审查必须独立上下文**，审查不给自己盖章 |
| 写完就算完成 | **审查通过才生效**。写完不算，测试通过也不算 |
| 无人值守跑一夜，早上起来全是返工 | **门必须人过**，不设「预授权批量过门」 |

三条铁律贯穿全仓：

```text
1. docs/ 是唯一状态源 —— 飞书、看板只做镜像，不得反向写状态
2. 合并与上线权在人 —— test / prod 只有人能合，AI 最高推到 dev
3. 审查是生效判定 —— 写完不算，测试通过也不算，审查通过才算
```

---

## 一、仓库结构与装什么

| 类型 | 数量 | 内容 | 装到哪 |
|---|---|---|---|
| skill | 10 | `skills/` | `~/.agents/skills/`（`npx skills add` 自动放） |
| 角色（agent） | 7 | `agents/` | `~/.config/opencode/agents/`（手拷） |
| 全局常驻规则 | 1 | `templates/AGENTS.md` | `~/.config/opencode/AGENTS.md` |
| 项目级模板 | 1 | `templates/project-AGENTS.md` | 每个项目复制成 `AGENTS.md` |

```text
skills/
├─ init-environment/SKILL.md       ← 装/重装全局环境
├─ init-project/SKILL.md           ← 新仓库建骨架
├─ full-workflow/                  ← 编排：分级 / 门 / 派谁
│  ├─ SKILL.md
│  └─ references/
│     ├─ doc-templates/            ← 8 份文档模板，一个域一份
│     │  ├─ 00-readme.md           ← docs/README.md：在途需求表 + 状态定义
│     │  ├─ 01-product.md           BRD / MRD / PRD / 术语表
│     │  ├─ 02-design.md            设计系统 / 原型图 / 页面规格
│     │  ├─ 03-architecture.md      系统架构 / 图示 / 状态机 / 环境 / DB / ER / ADR
│     │  ├─ 04-dev.md               代码规范 / 技术方案 / 实现记录
│     │  ├─ 05-test.md              测试策略 / 测试方案 / 测试报告
│     │  ├─ 06-ops.md               服务详情 / 部署拓扑 / 运维手册 / 发布记录
│     │  └─ 07-review.md            审查清单 / 审查报告
│     └─ project-layout.md         ← 项目目录树 + 版本规则 + 分支策略
├─ product-workflow/SKILL.md       ← 产品（门 1）
├─ design-workflow/SKILL.md        ← 设计（门 2）
├─ architect-workflow/SKILL.md     ← 架构（门 3）
├─ develop-workflow/SKILL.md       ← 全栈开发（门 4）
├─ test-workflow/SKILL.md          ← 测试（门 5）
├─ review-workflow/SKILL.md        ← 审查（不设门）
└─ ops-workflow/SKILL.md           ← 运维（门 6）

agents/                            ← 7 个 OpenCode 专用角色，与 skill 同名同内容
├─ product-workflow.md
├─ design-workflow.md
├─ architect-workflow.md
├─ develop-workflow.md
├─ test-workflow.md
├─ review-workflow.md
└─ ops-workflow.md

templates/
├─ AGENTS.md                       ← 全局常驻（装到 ~/.config/opencode/AGENTS.md）
└─ project-AGENTS.md               ← 项目级（init-project 用）

backup-docs/                       ← v1.0.0 原始素材，只读，不装
```

**`agents/` 和 `skills/*-workflow/` 是同一份内容的两种形态。** skill 版多一段「你在八道门里的位置」；agent 版是 skill 内容 + `mode: subagent` + `model`。改规则要**两边一起改**，只改一边会漂。

**`backup-docs/` 是拆分成 skill 之前的 7 份源文件**（5643 行），保留只为溯源，不参与安装。

---

## 二、安装

### 2.1 装之前确认四件事

| 项 | 怎么确认 | 没有会怎样 |
|---|---|---|
| OpenCode / OpenChamber 已装 | `opencode --version` | 后面全做不了 |
| 至少一个 provider 可用 | 会话里 `/models` 能看到模型 | 角色装上了但跑不起来 |
| Git 配了真人身份 | `git config --global user.name` 有值 | 提交记录追不到人 |
| 目标项目已 `git init` | 目录里有 `.git/` | 分支、提交、审查全失效 |

缺任何一项就停下补齐，不要「装完再说」。

### 2.2 装 10 个 skill

```bash
npx skills add git@github.com:jimbosuper/fullstack-workflow.git -a opencode
```

跨 agent（OpenCode + Codex + ZCode）：

```bash
npx skills add git@github.com:jimbosuper/fullstack-workflow.git \
  -a opencode -a codex -a zcode
```

**skill 跨 agent 通用；`agents/` 那 7 个是 OpenCode 专用。** 其它 agent 用 skill 版就够了。

### 2.3 装 7 个角色（OpenCode 专用）

```bash
cp agents/*.md ~/.config/opencode/agents/
```

**装之前必须改两处，否则装上也跑不起来：**

**① `model` 一行是占位，必须换成你本机真有的模型。**

```yaml
model: <provider>/<model>#high     # ← 改这里
```

`#` 后面是推理档位（reasoning effort），可选值取决于你的模型定义。仓库里给的默认档位：

| 角色 | 默认档位 | 理由 |
|---|---|---|
| 产品 | `#low` | 主要是提问与整理，不需要深推 |
| 设计 | `#high` | 要读懂 PRD 并翻译成 brief |
| 架构 | `#high` | 结构与数据模型错了下游全废 |
| 全栈开发 | `#high` | 实现 + 自测 |
| 测试 | `#low` | 按 TP 逐条跑 |
| 审查 | `#max` | 唯一的把关者，最该花算力 |
| 运维 | `#low` | 写文档、查状态 |

**② 文件名和机器上已有的旧名对不上时，先删旧文件再装。**

| 本仓库文件名 | 旧版（备份素材）用的名字 |
|---|---|
| `product-workflow.md` | `product-manager.md` |
| `design-workflow.md` | `designer.md` |
| `architect-workflow.md` | `architect.md` |
| `develop-workflow.md` | `fullstack-dev.md` |
| `test-workflow.md` | `test-runner.md` |
| `review-workflow.md` | `code-reviewer.md` |
| `ops-workflow.md` | `ops-engineer.md` |

两套并存会有 14 个角色，模型派错人。**确认旧版没用之后再删。** `mode: subagent` 那一行不能删——不写会被当成主 agent，反而抢走本该派给子代理的活。

### 2.4 装全局 AGENTS.md

```bash
cp templates/AGENTS.md ~/.config/opencode/AGENTS.md
```

**已有 `AGENTS.md` 的先看内容再决定覆盖还是合并**，直接覆盖会冲掉你自己写的规则。

**⚠️ 目前仓库里的 `templates/AGENTS.md` 只有 9 节，比机器上在用的 12 节少三节**（任务分级、八道门、返工与并行）。**这一份不要直接覆盖**，见第七节缺口 1。合并时以机器上那份的这三节为准。

### 2.5 初始化项目（每个新仓库一次）

```bash
# 在新项目里，让 AI 走 init-project skill，或手工：
mkdir -p docs/99-模板文档
cp -r <本仓库>/* docs/99-模板文档/
cp docs/99-模板文档/templates/project-AGENTS.md AGENTS.md
```

然后把 `AGENTS.md` 里的「待补充」换成实际值：技术栈（带版本）、真实跑得通的命令、受保护分支名。**保持 100 行以内**——规则堆多了遵守率反而下降。

模块级规矩写在那个目录自己的 `AGENTS.md` 里，不要堆到根目录。`AGENTS.md` 是分层加载的：全局 → 项目根 → 子目录，全部合并。

### 2.6 装完怎么验

前四步机器能验，第五步只能人验。

```bash
# 1. 角色：应为 7（或多出旧版，需清理）
ls ~/.config/opencode/agents/ | wc -l

# 2. skill：应为 8（7 个岗位 + full-workflow），另加 init-* 两个
ls ~/.agents/skills/ | grep -c workflow

# 3. 全局规则在。注意：仓库里的 templates/AGENTS.md 目前只有 9 节，
#    缺「任务分级 / 八道门 / 返工并行」，见第七节已知缺口 1
grep -c "^## " ~/.config/opencode/AGENTS.md

# 4. MCP 连上了
opencode mcp list | grep connected
```

**5. 最后一步只能人验**：新开会话，说「用产品角色写个 PRD」。**能派出去才算装好。**

### 2.7 重装与升级

| 情况 | 怎么做 |
|---|---|
| 完全没装 | 按上面全装 |
| 已有旧版（旧角色名） | 确认后删旧文件，再装新的 |
| 只想补缺 | 只拷缺的文件，**不覆盖已有的** |

不许盲目 `cp -r` 覆盖：手改过的配置会被冲掉。

---

## 三、核心逻辑

这是本仓库最重要的部分。**装好只是把手装上，这一节决定它怎么动。**

### 3.1 第一步永远是定级

不是所有任务都值得走完整流程。让框架修一个拼写错误却产出四个用户故事、十六条验收标准，是这类框架被弃用的第一原因。

| 级别 | 判定（满足任一即升级） | 谁来做 | 过几道门 |
|---|---|---|---|
| **小** | 单文件、不改数据结构、可一次回滚、不影响他人 | 编排者自己做，不派 | **0 道**，自查 diff |
| **中** | 多文件、跨模块；或新增功能 | 产品 → 全栈开发 → 测试 → 审查 | **3 道**：门 1 / 4 / 7 |
| **大** | 改数据结构、需迁移、上生产、影响外部系统 | 七个角色，架构按需介入 | **全部 8 道** |

判定表：

| 你在做什么 | 级别 |
|---|---|
| 改文案、注释、文档；修单文件 bug | 小 |
| 新增一个页面或一个接口；跨前后端联调 | 中 |
| 加第三方依赖 | 中（涉许可或安全则为大） |
| 新增数据表、改字段类型、改接口出入参 | 大 |
| 改 CI / 部署配置、上生产 | 大 |

两条升级规则：**拿不准就升一级**（升错只多花时间，降错会出事）；**执行中发现范围变大，立刻停下重新定级**，门数跟着变。

### 3.2 大任务走完整条链

```text
产品   → PRD + BRD/MRD/术语表              →【CR1】→【门 1】人放行
设计   → 原型 + 页面规格 + 设计系统        →        →【门 2】人放行
架构   → 架构 + 图示 + 状态机 + DB + ER + ADR →【CR2】→【门 3】人放行
全栈   → 技术方案（TD）                     → 同上随 CR2
       → 代码 + 实现记录                                  →【门 4】人放行
测试   → 测试方案（TP）                     →        →【门 5】人放行
       → 测试报告（TR，通过 / 有条件通过才继续）
审查   → 代码审查（CR3，通过才生效）        ← 不设门
运维   → 服务详情 + 部署拓扑 + 运维手册      →        →【门 6】人放行
                                                 →【门 7】人验收
                                                 →【门 8】人放行上线
人     → 推代码，流水线部署
编排者 → 发布快照：本版产物搬进 docs/99-归档/发布/vX.Y.Z/
```

中任务走精简版：

```text
产品 → PRD ────────────────────→【门 1】人放行（不审）
全栈开发 → TD →【CR2】→【门 4】人放行 → 代码 → 实现记录
测试 → TR（通过才继续）
审查 → 代码审查（CR3）→【门 7】人验收
```

### 3.3 八道门

**门后必须是机器无法代判的内容。能机器判的不设门。**

| # | 门 | 时机 | 先审吗 | 人只回答这一个问题 |
|---|---|---|---|---|
| **1** | 产品定稿 | PRD + BRD/MRD/术语表写完 | 大任务审（CR1） | **哪些不是我们的真实业务？** |
| **2** | 设计定稿 | 原型 + 页面规格 + 设计系统完 | 不审 | 这个界面**像不像**实际要用的？ |
| **3** | 架构定稿 | 架构 + 图示 + 状态机 + DB + ER + ADR 完 | 审（CR2） | 表、状态、边界**有没有漏掉真实存在的？有多出来的吗**？ |
| **4** | 技术方案定稿 | TD 六节写完，代码未写 | 审（随 CR2） | 实施顺序和已知的坑，**有没有你不知道的约束**？ |
| **5** | 测试方案定稿 | TP 七节写完，测试未跑 | 不审 | **你最怕出问题的地方**，测到了吗？ |
| **6** | 运维定稿 | 服务详情 + 拓扑 + 手册完 | 不审 | 机器、端口、边界，**和真实环境一致吗**？ |
| **7** | 验收 | 审查结论为通过 / 有条件通过 | CR3 已在之前完成 | 功能符合预期吗？可以发吗？ |
| **8** | 上线 | 发布前检查完成 | 不审 | 现在就发吗？ |

**这八个问题的共同点：都不是「这东西好不好」，而是「这东西是不是真的」。** 好不好交给审查，真不真只有人说得出。

三条硬规矩：

```text
1. 人没说「过」，状态就是「待确认」，不进下游。没有「编排者代决」这条路径。
   推断、沉默、「你觉得呢」都不算。
2. 只有人能说「过」。文首的「放行」行由编排者代填，依据是人说过的那句话。
3. demo 的失败模式是「门由编排者代决」——门永远不能代决；
   非门事项（技术选型、命名、目录结构）可以代决，要留痕。
```

**门的顺序是「审查 → 人 → 通过」，不是「人看完直接过」。** 审查先把技术问题挑完，人看到的是已经审干净的版本——否则人会花时间在错别字和漏字段上，那是审查该拦的。

审查裁决不是「通过 / 有条件通过」时，**人这一关根本不启动**，产物直接回原作者。

### 3.4 审查：不设门，但审三次

审查回答「对不对」，不回答「是不是」。所以它没有自己的门，只当门 7 的人的证据与抽查对象。

**大任务审三次，是三个不同对象，不是同一份审三遍：**

| 审查 | 对象 | 为什么 |
|---|---|---|
| CR1 | 大任务的 PRD | PRD 的缺陷会被设计、架构、TD 三层放大，越早拦越省 |
| CR2 | 架构 + 技术方案，一起审 | 结构错了代码全废；两份高度相关，分开审看不出对不上 |
| CR3 | 代码 | 上线前最后一道 |

**每个对象最多 2 轮**（首审 + 1 次复审），所以最多 6 次审查。第 2 轮仍不通过就**不开第 3 轮**，直接上报人，判「规格有问题 / 能力不够 / 任务太大」三者之一。

审查的四条自我约束：

| 约束 | 值 | 意义 |
|---|---|---|
| 分诊 → 详写两遍制 | 第一遍只出四列清单，第二遍只详写 P0/P1 | 审查慢的主因是给每条 P2 都写五段 |
| P0/P1 上限 | ≤ 15 条，超了判「退回重写」 | 15 条以上不是「有小问题」，是「还没想清楚」 |
| 代码 AC 抽样 | 30% 且 ≥ 10 条，缺陷率 > 20% 停下退回 | 抽样代替穷举，命中率就是信号 |
| 自检可发现占比 | > 50% 判「自检不足」退回 | 审查的价值是「找出作者找不到的」 |

**每条 P0/P1 必须附一个可重跑的复现脚本或命令**，路径写进报告。门 7 上的人有权说「把 P0-1 的探针跑给我看」——跑不出来，那条发现作废。这是唯一能约束「审查自证」的机制。

### 3.5 产物：两条版本线

| 类 | 文档 | 文件名带版本 |
|---|---|---|
| **链上** | PRD、页面规格、系统架构、架构图示、状态机、数据库设计、ER 图、ADR、TD、实现记录、TP、TR、CR | ✅ 与整条链同版本 |
| **项目级** | BRD、MRD、术语表、设计系统、环境规范、代码规范、测试策略、服务详情、部署拓扑、运维手册、发布记录、审查清单 | ❌ 不带版本，文首写「最后更新」 |

**同一条链上的版本必须全部相同。** 有一格不一样，说明有一份没跟着升。登记处只有一个：`docs/README.md` 的在途需求表。

升版本三步：

```text
1. 链上全部产物一起拷到新版本号（一份都不能漏）
2. 内容真改的改内容；没改的只改文首版本行
3. 旧的一整套立刻搬进 docs/99-归档/版本/<需求>/<旧版本>/
```

看起来「没改内容也拷一份」是浪费，但**文件名是唯一能让人一眼看出「这些是一套」的地方**。不拷就会出现「PRD v1.3、架构 v1.0、TD v1.0」——内容没错，但没人看得出配对。

### 3.6 返工：谁写的谁改

| 情况 | 回给谁 | 上限 |
|---|---|---|
| 测试裁决「不通过」 | 写代码的人 | 同一问题 2 轮 |
| 审查裁决「退回修改 / 退回重写」 | **写这份产物的人**：代码回开发、PRD 回产品、架构回架构、TD 回开发 | 同一份产物 2 轮 |
| 超过 2 轮 | 上报人 | — |

**「谁写的谁改」意味着审查可以退回任何角色。** 退回 PRD 时，产品改完要**重新过门 1**；退回架构时，架构改完要**重新过门 3**。产物改过之后，人没看过的新版本不能进下游。

并行规则一句话：**写单线程，只读可并行。**

### 3.7 子代理的报告只是声明

子代理回「已产出 N 个文件」，编排者必须在派发它的会话里当场验：

```bash
for f in <它声明的每个路径>; do
  test -s "$f" || echo "MISSING: $f"
done
```

任一文件不存在或为空 → 当作没产出，任务拆成「只写第一个文件」再派一次。**一次派发让子代理产出的文件数 ≤ 2。**

**为什么写成命令**：把「验证」当口头要求就会漏；写成 `test -s` 才不会。

### 3.8 门的代价

**这套门一旦落地，就不再支持「睡一觉让它跑完」。** 一个角色交活 → 等人放行 → 下一个角色才能开始，两次回话之间最多推进一个角色。

代价是真实的：一次 12 小时无人值守换来的是「产品零提问、设计跑在门 1 之前、TD 写成事后归纳、ER 只画了 9/22 表」，全部要花一整个会话考古。**所以不设「预授权批量过门」的例外。**

---

## 四、日常怎么用

### 4.1 一次交付的典型走法

```text
1. 在项目里开一个会话，说「按 PRD1 做」或「做个 X」
2. 编排者先定级，告诉你走哪几道门
3. 每交完一份产物就停下等人放行
4. 你说过「可以」，它才继续
```

**人只回答一个问题**：这是不是真的。质量归审查，不归人。

### 4.2 说什么话触发什么 skill

| 你说 | 触发 |
|---|---|
| 「这个改动走什么流程」「下一步是什么」「该派谁」 | `full-workflow` |
| 「写个 PRD」「需求不清楚」「要做个 X」 | `product-workflow` |
| 「出原型」「改界面」 | `design-workflow` |
| 「加张表」「要不要拆模块」「画架构图 / ER 图」 | `architect-workflow` |
| 「写技术方案」「开始实现」 | `develop-workflow` |
| 「写测试」「跑一下测试」「为什么挂了」 | `test-workflow` |
| 「审一下这个改动」 | `review-workflow` |
| 「部署」「写运维手册」「上线检查」 | `ops-workflow` |
| 「装工作流」「角色不见了」 | `init-environment` |
| 「新建项目」「建 docs 结构」 | `init-project` |

**团队分工时，每个人只装自己那个 `*-workflow`，但都要装 `full-workflow`**——否则不知道自己现在站在门几。

### 4.3 产物落在哪

```text
docs/01-产品/   PRD（链上带版本）· BRD / MRD / 术语表（项目级）
docs/02-设计/   页面规格（链上）· 设计系统 / 原型图（项目级 + OpenDesign 产物）
docs/03-架构/   系统架构 / 图示 / 状态机 / DB 设计 / ER / ADR（链上）
                项目环境规范（项目级）
docs/04-开发/   技术方案 / 实现记录（链上）· 代码规范（项目级）
docs/05-测试/   测试方案 / 测试报告（链上）· 测试策略（项目级）
docs/06-运维/   服务详情 / 部署拓扑 / 运维手册 / 发布记录（全项目级）
docs/07-审查/   审查报告（链上）· 审查清单（项目级）
docs/99-tmp/    技能原料，任务结束清空，不入库
docs/99-归档/   版本 / 废弃 / 发布快照，只读，只在 dev 分支
```

完整目录树、扩展名白名单、多图归目录规则见 `skills/full-workflow/references/project-layout.md`。

### 4.4 分支策略

| 分支 | 推什么 | 为什么 |
|---|---|---|
| `dev` | 全量，含 `99-归档/` | 开发本地仓库，信息必须对等 |
| `test` | 只推当前版本 + 当次需要的产物，不含归档 | 测试环境只跑当前这套，冗余拖慢流水线 |
| `prod` | 只推运行需要的内容 + 当次发布快照 | 生产最小化，减少误操作面 |

**`test` / `prod` 只有人能合，AI 最高推到 `dev`。**

### 4.5 测试分两段

| 阶段 | 谁跑 | 跑什么 | 在哪 |
|---|---|---|---|
| 开发自测 | 全栈开发 | 单元 + 组件 + 类型 + lint | 本地 `dev` |
| 独立验证 | 测试角色 | 全量 + e2e | 本地先跑通，合进 `test` 后 CI 再跑，**报告以 CI 为准** |

两段都要过。本地过了不代表 CI 能过（环境差异），CI 过了也不代表测对了（用例可能没覆盖）。

---

## 五、外部工具依赖

下面这些**不在本仓库**，要单独装。不装相关角色也能跑，只是少一个主力工具。

| 工具 | 形态 | 谁用 | 装法 |
|---|---|---|---|
| `grill-me` / `grilling` | skill | 产品 | `npx skills add mattpocock/skills --skill=grill-me --skill=grilling -a opencode` |
| `open-design` | MCP + CLI | 设计 | 见下 |
| `archify` | skill + CLI | 架构 / 运维 | `npx skills add https://archify.si --skill archify --agent opencode --global --copy --yes` |
| 飞书项目 / GitLab / Context7 / Postgres / Sentry / 阿里云 | MCP | 各角色 | `opencode mcp add <name> --global --url <url>` |

**`grill-me` 有一个已知限制**：它的 frontmatter 带 `disable-model-invocation: true`，只能由人在会话里手动 `/grill-me` 触发，子代理自己调不动。所以 `product-workflow` 的 §4.3 把 grilling 的协议（设计树、frontier、每问带推荐答案、frontier 空才收工）**内联进了文档**——skill 触发不了也能按文档做。

**不装 `grill-with-docs`。** 它往仓库根写 `GLOSSARY.md`、往 `docs/adr/` 写 ADR，和本套的 `docs/01-产品/04-产品术语表.md` / `docs/03-架构/07-ADR重要决策/` 是两套。两套术语表同时存在比没有更糟。

### 5.1 OpenDesign（设计角色出原型）

官方 `od mcp install opencode` 生成的配置**不能直接用**，有两个坑：`od` 在 macOS 上被 `/usr/bin/od`（八进制 dump）占住；daemon 端口随机，不一定是 7456。

```bash
# 1. 打开桌面版，daemon 必须开着
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

**三个必须**：`node` 用绝对路径、`daemon-cli.mjs` 用绝对路径、`--daemon-url` 写真实端口。少一样就连不上。**端口每次重开桌面版都可能变**，连不上先重查端口。

### 5.2 Archify（架构与运维出图）

skill + Node CLI，不是 MCP。`npx skills add` 那一步**源必须写 `https://archify.si`**（带 https，走 well-known 端点只拉 skill 本体）；写成 `archify.si` 会被当 git 仓库去克隆，直接报 `repository does not exist`。也不要写 `tt-a1i/archify`——仓库 141 MB，大概率超时。

```bash
node ~/.agents/skills/archify/bin/archify.mjs doctor   # 20 多项全 [ok] 才算装好

node ~/.agents/skills/archify/bin/archify.mjs finalize architecture \
  candidate.json 02-架构图示.html --quality showcase --json \
  --out-dir docs/99-tmp/架构-<任务>/receipts/
```

四条硬约束：

| 约束 | 值 |
|---|---|
| 节点数 | **8–12 个**，超过就拆图 |
| 表数（ER） | 一张 **8–12 张表**，超过就拆图 |
| 画布宽 | ≤ 1240px（宽高比 ≥ 1.55 时可到 1790px） |
| 收据位置 | `--out-dir` 指到 `99-tmp`，**不许落在业务目录** |

**一次 `finalize` 只能出一种图。** 架构、状态机、ER 是三个独立文件——`meta.views` 只是同一张图的聚焦切换，不是把两种图拼一起。**流程时序图仍然用 Mermaid 写在 `01-系统架构.md` 里**：它数量多、改得勤、要对着读，做成 HTML 不划算。

**校验过了不等于图好看。** 过了还要看返回里的 `visualReviewRecommendation`，有交叉连线就按 hints 调坐标重跑，最多两轮。

---

## 六、常见问题

| 现象 | 原因 | 怎么修 |
|---|---|---|
| 角色装上了但调不出来 | 缺 `mode: subagent`，或 `model` 还是占位 | 补 frontmatter，重启 |
| 机器上有 14 个角色，派错人 | 新旧两套名字并存 | 确认后删旧名文件 |
| 主 agent 自己把活干完了 | `description` 太笼统 | 写清「什么时候用」 |
| 技能不触发 | `description` 与任务不匹配 | 改 description，写清触发场景 |
| 该用的技能不在表里 | 表只覆盖常用 | 让它用 `find-skills` 现找。**注意 `code-review`、`find-skills` 不一定装了**，没装就走内置工具 |
| MCP 显示 needs authentication | 远程 server 要 OAuth | 会话里跑 `/mcps` 登录 |
| 子代理读不到文件 | 路径写成了相对当前目录 | 一律用项目根开始的相对路径 |
| 一次派了多个角色 | 没读到「一次只派一个环节」 | 确认全局 `AGENTS.md` 装的是本仓库 `templates/AGENTS.md` |
| `AGENTS.md` 装完少了分级和门的规则 | 仓库那份只有 9 节，缺最核心的三节 | 别直接覆盖；合并到现有的 12 节版本，见第七节缺口 1 |
| 睡一觉让它跑完，早上全是返工 | 门必须人过 | 不设预授权批量过门，见 3.8 |
| OpenDesign 连不上 | daemon 端口变了，或没开桌面版 | 重查端口改 `--daemon-url` |
| Archify 报 clone timeout | 没走代理，或源写成 `archify.si` | 导出 `https_proxy`；源写 `https://archify.si` |
| Archify 图不好看但校验全过 | 有交叉连线没修 | 看 `visualReviewRecommendation` 的 hints，调 `pos` 重跑 |
| 部署拓扑报 `deployment-private-state` | 数据库没放进 security-group | 每个环境建一个私网安全组，包住应用 + 数据库 |
| 部署拓扑报 `deployment-crossing-mechanism` | 连线 label 太笼统 | 写成「协议 + 端口 + 路径」，如 `HTTP 8080 轮询` |
| ER 图报 `references unknown entity` | `references` 写成了表名 | 用 entity 的 `id`：`"user.id"` 而不是 `"users.id"` |

---

## 七、已知缺口

诚实列出来，不假装完整：

| # | 位置 | 说明 |
|---|---|---|
| 1 | `templates/AGENTS.md`（254 行） | **比 `~/.config/opencode/AGENTS.md`（400 行）少了三节：任务分级、八道门、返工与并行。** 照 2.4 直接 `cp` 装，会把最核心的三节丢掉。第 21 行还写着「分级规则见第九节」，而第九节现在是「上下文」；末尾「## 九、上下文」下挂「### 12.1 上下文压缩」，节号断裂。**修好之前，2.4 那一步应改为合并，不要覆盖** |
| 2 | `agents/*.md` | `model` 全是 `<provider>/<model>#…` 占位。`cp` 装过去之前必须改，见 2.3 |
| 3 | `agents/*.md` vs `skills/*-workflow/SKILL.md` | 同一份内容双写，除头部外完全重复。改规则要改两处，容易漂 |
| 4 | `agents/*.md` | frontmatter 只有 `description / mode / model`，**没有 `permissions` 段**。需要路径级硬约束时自己补 |
| 5 | 各角色「可用工具」表 | 写了 `superpowers:*`、`mattpocock:grill-me`、`code-review`、`find-skills`，但命名与本机 `~/.agents/skills/` 里的扁平名（`brainstorming`、`executing-plans`…）不完全一致，`code-review` 与 `find-skills` 未随本仓库提供 |
| 6 | `skills/init-environment/SKILL.md` | archify 的装法写成 `npx skills add archify.si`，缺 `https://`——照它跑会报 `repository does not exist`。第 59 行还写「见 README 的 OpenDesign 一节」，而本 README 已改版，指代可能失配 |
| 7 | `doc-templates/00-readme.md` | 「状态定义八种固定」与完成检查「九种内」不一致，实际表里 17 种 |
| 8 | `doc-templates/05-test.md` | 「这份不设门——同上」，拆成独立文件后没有「上文」，指代失效 |
| 9 | 本仓库无 LICENSE、无 `.gitignore`、无 CI | 对外分发前需定；目前靠人工 `grep` 查双写一致性与占位残留 |

---

## 八、版本

**v1.0.0** — 从 `backup-docs/` 的 7 份源文件拆成 10 个 skill + 7 个角色 + 2 份模板；八道门、三次审查、两轮上限、链版本与项目级版本两条线定型。

原始素材在 `backup-docs/`，只读，用于溯源。**活版本就是本仓库根目录下的 `skills/`、`agents/`、`templates/`。**
