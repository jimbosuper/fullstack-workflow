---
name: 初始化项目
description: 在一个仓库里初始化项目规范（项目 AGENTS.md、docs 目录树、审查清单、gitignore、CI 配置）。
  用户说「新建项目」「初始化仓库」「建 docs 结构」时使用。
---

# 初始化项目

**这个 skill 每个新仓库用一次，由 `fullstack-workflow` 主 agent 自己在阶段 1 的第 0 步调用**（见主 agent 正文 §0.1）。它建骨架、生成审查清单、把模板落到项目里，不写业务内容。

**不要在 `build` 里手工跑它，也不要派某个角色跑它。** 骨架是编排者的活：建完之后产物才有地方落、作者才有清单可自检。

**它必须产出两样容易漏的东西：`docs/07-审查/01-审查清单/01-审查清单.md` 和 `docs/99-模板文档/`。** 前者是「作者自检」的前提，后者是项目自包含的前提——旧做法两个都没做，结果是审查清单等到第一次审查才建、`99-模板文档/` 一直是空的却被人当出处引用。

## 一、动手前先看现状

**不许盲目建目录。** 先跑：

```bash
ls -a
git status --short | head
```

| 情况 | 怎么做 |
|---|---|
| 空仓库 | 全建 |
| 已有部分目录 | **只补缺的**，不覆盖 |
| 已有 `docs/` 结构 | 逐个目录比对，缺什么补什么 |
| 已有 `AGENTS.md` | 看内容再决定合并还是保留 |

**这跟常驻规则「动手前必查四个问题」是同一条。**

## 二、根目录要建什么

```text
<项目根>/
├─ README.md       # 项目是什么、怎么跑
├─ AGENTS.md       # 给 AI 的项目说明
├─ .gitignore      # 见第三节
├─ .gitlab-ci.yml  # GitLab 只认根目录
├─ src/
├─ test/           # unit / integration / e2e / perf / mobile / fixtures / helpers
├─ deploy/
└─ docs/
```

**`test/` 的层目录按实际建，用不上的整层不建。** 最小集合是 `unit/`、`integration/`、`fixtures/`、`helpers/`；`e2e/` 在有浏览器场景时建。

## 三、.gitignore 必须补上的五条

```gitignore
deploy/env/*
!deploy/env/.env.example
docs/99-tmp/
docs/01-设计/03-原型图/.file-versions/
docs/01-设计/03-原型图/*.artifact.json
.openchamber/
node_modules/
```

| 行 | 防什么 |
|---|---|
| `deploy/env/*` + `!.env.example` | 环境变量的真实值入库；**`.env.example` 要入库**，它是变量清单的唯一出处 |
| `docs/99-tmp/` | 临时区入库（踩过 3715 个文件的坑） |
| `.file-versions/` | **OpenDesign 的内部版本库**，不是我们的产物；入库会让 git 历史被工具的中间态塞满 |
| `*.artifact.json` | OpenDesign 元数据，同上 |
| `.openchamber/` | 截图与本地状态 |

**`99-tmp/` 与 `.file-versions/` 的区别**：前者是我们的临时原料，后者是工具的版本库。**两者都不入库，理由不同**——一个是「还没定稿」，一个是「不归我们管」。

**建完验证：**

```bash
for p in deploy/env/pg.env docs/99-tmp docs/01-设计/03-原型图/.file-versions "docs/01-设计/03-原型图/x.artifact.json" .openchamber; do
  git check-ignore -q "$p" && echo "OK  $p" || echo "MISS $p"
done
# 示例文件必须反过来：不是 ignored 才算对
git check-ignore -q deploy/env/.env.example && echo "FAIL  示例文件被误忽略" || echo "OK  .env.example 可入库"
```

## 四、docs/ 目录树

**完整结构见 `fullstack-workflow` skill 的 `references/project-layout.md`。**

这里只给必须建的空目录清单：

```bash
mkdir -p docs/01-设计/{01-设计系统,02-组件规范,03-原型图,04-页面规格} \
  docs/02-产品/{01-需求池,02-BRD商业需求,03-MRD市场需求,04-PRD产品需求,05-产品术语表} \
  docs/03-架构/{01-系统架构,02-架构图示,03-状态机,04-项目环境规范,05-数据库设计,06-ER图,07-ADR重要决策} \
  docs/04-开发/{01-代码规范,02-技术方案,03-实现记录} \
  docs/05-测试/{01-测试策略,02-测试方案,03-测试报告} \
  docs/06-运维/{01-服务详细情况,02-部署拓扑,03-运维手册,04-发布记录} \
  docs/07-审查/{01-审查清单,02-审查报告} \
  docs/99-tmp/{01-任务单,02-原料,03-receipts,04-截图} \
  docs/99-外部文档 docs/99-模板文档/{doc-templates,roles} \
  docs/99-归档/{版本,废弃,发布}
```

**建完立刻验证：**

```bash
find docs -type d | sort
```

**目录序号就是产生顺序**：`01-设计` 最先（原型），`02-产品` 在原型稳定后。**只有 `07-审查` 的编号和产生顺序不一致**——它按产物类型排最后，但 CR1 在阶段 1 后就落盘。这一点写进 `docs/README.md`，否则新人会以为审查是最后才做的。

## 五、把模板落进项目（`docs/99-模板文档/`）

**项目要自包含。** 全局 skill 会被升级、会被换掉；项目里的规则必须能独立读到。

从 skill 的安装位置复制（全局装时是 `~/.config/opencode/skills/`，本仓库开发时是 `<repo>/skills/`）：

```bash
SKILLS=<skills 目录>

cp "$SKILLS/fullstack-workflow/references/project-layout.md" docs/99-模板文档/
cp "$SKILLS/fullstack-workflow/references/questioning.md" docs/99-模板文档/
cp -r "$SKILLS/fullstack-workflow/references/doc-templates/." docs/99-模板文档/doc-templates/
cp "$SKILLS"/*-workflow/SKILL.md docs/99-模板文档/roles/ 2>/dev/null || true
```

**复制后验证：**

```bash
ls docs/99-模板文档/project-layout.md docs/99-模板文档/questioning.md
ls docs/99-模板文档/doc-templates/ | wc -l     # 期望 8
ls docs/99-模板文档/roles/ | wc -l             # 期望 8（7 个角色 + fullstack-workflow）
```

**项目里的引用一律指向 `docs/99-模板文档/`，不指向全局路径。** 曾经 `docs/README.md` 写「目录规则见 `docs/99-模板文档/skills/fullstack-workflow/references/project-layout.md`」，而 `99-模板文档/` 是空的——**这是一个永远打不开的链接，被当成目录规则的出处引用了整个项目周期。**

## 六、生成 `docs/07-审查/01-审查清单/01-审查清单.md`

**这份清单必须在第一个角色交付前就存在。** 没有它，「照清单自检」客观不可执行；审查角色的第一道工序（验自检表）也无从开始。

**生成方式：把 `doc-templates/07-review.md` 第一个产物块（8.1 审查清单）整段搬过来，只换掉「待补充」和示例编号，不加节、不删节。**

```text
docs/07-审查/01-审查清单/01-审查清单.md
├─ 一、本文结论            ← 从 07-review.md 复制
├─ 二、审什么              ← 同上（2.1 审代码 / 2.2 审产物）
├─ 三、按什么审            ← 同上
├─ 四、分阶段必检项        ← 同上（4.1 阶段 1 / 4.2 阶段 2 / 4.3 阶段 3）
├─ 五、必检项（按对象类型）← 同上（5.1 审代码七条 / 5.2 审产物四条）
├─ 六、分级                ← 同上
├─ 七、结论四档            ← 同上
├─ 八、不审什么            ← 同上
└─ 九、拿不准时            ← 同上
```

**没有「各角色完成检查」这一节。** 角色交活前的自检就是照上表**第四节**里属于自己阶段那一小节（4.1 / 4.2 / 4.3）逐条过；审查角色另外过**第五节**「必检项（按对象类型）」。**不要另建第九节按角色分组**——那会让同一批必检项存在两处，改一处漏一处。

**判据来源必须写本项目的具体路径与编号，不许照抄「TD」「PRD」这类泛指。** 判据：清单里引用本项目编号（R / AC / 表名 / 文件路径）的行 ≥ 50%。

**清单必须在第一个角色交付前就在。** 清单缺席时，作者的「自检」没有依据，审查变成替粗心买单。

## 七、项目 AGENTS.md 要写什么

| 节 | 内容 |
|---|---|
| 一、技术栈 | **带版本，不写「最新」** |
| 二、项目目录 | 指向 `docs/99-模板文档/project-layout.md` |
| 三、编码规范 | 只说本项目特有的，通用规范见 `04-开发/01-代码规范/01-代码规范.md` |
| 四、基本命令 | **只写真实跑得通的**，跑不通的删掉 |
| 五、验证清单 | 什么算通过 |
| 六、禁止内容 | 本项目不能碰的 |

**模板在 `templates/project-AGENTS.md`。**

**「只写真实跑得通的」这条最要紧**——留占位等于教模型猜。

## 八、装完怎么验

```bash
# 1. 目录齐了
find docs -type d | wc -l

# 2. gitignore 生效（五个要 OK，示例文件要 NOT ignored）
for p in deploy/env/pg.env docs/99-tmp docs/01-设计/03-原型图/.file-versions "docs/01-设计/03-原型图/x.artifact.json" .openchamber; do
  git check-ignore -q "$p" && echo "OK  $p" || echo "MISS $p"
done
git check-ignore -q deploy/env/.env.example && echo "FAIL  示例文件被误忽略" || echo "OK  deploy/env/.env.example 可入库"

# 3. 审查清单已生成，九节齐全（应为 9），三个阶段的必检项都在
test -s docs/07-审查/01-审查清单/01-审查清单.md \
  && test "$(grep -c '^## [一二三四五六七八九]、' docs/07-审查/01-审查清单/01-审查清单.md)" = 9 \
  && grep -c '^### 4\.' docs/07-审查/01-审查清单/01-审查清单.md

# 4. 模板已落地
test -s docs/99-模板文档/project-layout.md && ls docs/99-模板文档/doc-templates | wc -l

# 5. AGENTS.md 有内容
head -5 AGENTS.md
```

**五条全过才算初始化完成。**

## 九、卡住时

| 现象 | 原因 |
|---|---|
| `git check-ignore` 没输出 | 该目录还没创建，或 .gitignore 写法不对 |
| 目录建了但 Git 看不到 | 目录是空的，Git 不跟踪空目录；放一个 `.gitkeep` |
| `cp` 找不到 skill 目录 | 先 `ls ~/.config/opencode/skills` 确认安装位置；本仓库开发时用 `skills/` |
| 审查清单里全是模板原文 | 第九节没按本项目编号改写；判据来源要写具体路径 |
| 已有项目结构不同 | **不要强行统一**，先问人要不要迁移 |
