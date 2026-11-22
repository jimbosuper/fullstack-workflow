---
name: 初始化项目
description: 在一个仓库里初始化项目规范（项目 AGENTS.md、docs 目录树、gitignore、CI 配置）。
  用户说「新建项目」「初始化仓库」「建 docs 结构」时使用。
---

# 初始化项目

**这个 skill 每个新仓库用一次。** 它建骨架，不写内容。

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
├─ .gitignore      # 必须含 deploy/env/ 与 docs/99-tmp/
├─ .gitlab-ci.yml  # GitLab 只认根目录
├─ src/
├─ test/
├─ deploy/
└─ docs/
```

## 三、.gitignore 必须有的三行

```gitignore
deploy/env/
docs/99-tmp/
node_modules/
```

**第一条防环境变量入库，第二条防临时区入库（这次踩过 3715 个文件的坑）。**

## 四、docs/ 目录树

**完整结构见 `full-workflow` skill 的 `references/project-layout.md`。**

这里只给必须建的空目录清单：

```bash
mkdir -p docs/{01-产品/03-PRD产品需求,02-设计/02-原型图,03-架构/{03-状态机,06-ER图,07-ADR重要决策},04-开发/{02-技术方案,03-实现记录},05-测试/{02-测试方案,03-测试报告},06-运维,07-审查/02-审查报告,99-tmp,99-外部文档,99-模板文档,99-归档/{版本,废弃,发布}}
```

**建完立刻验证：**

```bash
find docs -type d | sort
```

## 五、项目 AGENTS.md 要写什么

| 节 | 内容 |
|---|---|
| 一、技术栈 | **带版本，不写「最新」** |
| 二、项目目录 | 指向 `docs/99-模板文档/skills/full-workflow/references/project-layout.md` |
| 三、编码规范 | 只说本项目特有的，通用规范见 `04-开发/01-代码规范.md` |
| 四、基本命令 | **只写真实跑得通的**，跑不通的删掉 |
| 五、验证清单 | 什么算通过 |
| 六、禁止内容 | 本项目不能碰的 |

**模板在 `templates/project-AGENTS.md`。**

**「只写真实跑得通的」这条最要紧**——留占位等于教模型猜。

## 六、装完怎么验

```bash
# 1. 目录齐了
find docs -type d | wc -l

# 2. gitignore 生效
git check-ignore docs/99-tmp && echo "临时区已忽略"
git check-ignore deploy/env && echo "环境变量已忽略"

# 3. AGENTS.md 有内容
head -5 AGENTS.md
```

**三条全过才算初始化完成。**

## 七、卡住时

| 现象 | 原因 |
|---|---|
| `git check-ignore` 没输出 | 该目录还没创建，或 .gitignore 写法不对 |
| 目录建了但 Git 看不到 | 目录是空的，Git 不跟踪空目录；放一个 `.gitkeep` |
| 已有项目结构不同 | **不要强行统一**，先问人要不要迁移 |
