# 项目目录模板

新建项目时按本文建目录。**`docs/` 照抄，另外三个目录按项目裁剪。**

---

## 一、根目录

```text
<项目根>/
├─ README.md       # 项目是什么、怎么跑；新人第一个打开的文件
│
├─ AGENTS.md       # 给 AI 的项目说明；命令、技术栈、已知陷阱
│
├─ .gitignore      # 五项：deploy/env/*（放行 .env.example）、docs/99-tmp/、.file-versions/、*.artifact.json、.openchamber/
│
├─ .gitlab-ci.yml  # GitLab 只认根目录，放别处不会执行
│
├─ src/            # 源码；分层确定，结构按技术栈
│
├─ test/           # 按测试层分层；用不到的层不建
│
├─ deploy/         # 按平台分层；用不到的层不建
│
└─ docs/           # 全部项目文档；结构固定
```

**根目录不放其他东西。** 临时文件进 `docs/99-tmp/`。

---

## 二、src/ —— 分层固定，结构按技术栈

**这份模板描述的是「一个部署单元」。** 一个仓库 = 一个项目，多端是多仓库。如果一个产品有多个端，**每端一个仓库、各用本模板各建一次**，不在一个仓库里塞多端。

```text
某个全栈项目的GitLab仓库
├─ repo-backend/        ← 后端仓库（本模板完整一份：src/ test/ deploy/ docs/）
├─ repo-app/            ← 移动端仓库（本模板完整一份，src/ 只放 app/）
└─ repo-wx/             ← 微信小程序仓库（本模板完整一份，src/ 只放 wx/）
```

**单仓库内的分层，固定的三条**：按功能模块分；前端与后端模块同名对齐；共享代码只放一处。

```text
src/
├─ frontend/
│  ├─ app/            # 页面与路由；框架有强制路径就按框架来
│  ├─ components/     # 通用组件；跨模块复用才放这里
│  ├─ modules/        # 功能模块，一个功能一个目录
│  │  └─ notes/       # 模块名与 backend/modules 下同名
│  │     ├─ index.ts  # 模块入口
│  │     ├─ api.ts    # 调后端接口
│  │     └─ types.ts  # 仅前端用的类型
│  └─ common/         # 前端共用：类型、常量、工具函数
│
├─ backend/
│  ├─ api/            # 路由与控制器；只做参数校验与转发
│  ├─ modules/        # 与 frontend/modules 同名对齐
│  ├─ db/             # schema 与迁移；迁移必须可回滚
│  └─ common/         # 后端共用：中间件、工具
│
└─ common/            # 前后端共用类型；不写业务逻辑
```

**上面这套只适用于前后端分离的 Web 应用。** 纯 CLI、纯库、移动端不能照抄。

**只留用得上的层。** 纯后端删 `frontend/`，纯前端删 `backend/`，单端删顶层 `common/`。模块内部怎么分写进 `docs/04-开发/01-代码规范/01-代码规范.md`。

---

## 三、test/ —— 按测试层分

**分层依据是「跑起来需要什么」，不是「测什么功能」。** 要浏览器的那层靠后，不要的靠前。

```text
test/
├─ unit/                            # 不启服务、不连库、不起浏览器
│  ├─ pure/                         # 纯函数、hook、route handler
│  │  ├─ amount_test.ts             # 金额公式逐位相等（62.668800）
│  │  ├─ mask_test.go               # 脱敏单一入口
│  │  └─ state_machine_test.go      # 非法迁移一律 409
│  └─ components/                   # jsdom 下的渲染与交互（多于 20 个组件才分这层）
│     ├─ ProviderRow.test.tsx       # 五行数据渲染 + 筛选 + 排序
│     └─ InvoiceTimeLine.test.tsx   # 7 态时间线，逐态断言
│
├─ integration/                     # 起真实依赖：数据库、HTTP 层、队列
│  ├─ api/                          # 接口契约、鉴权、错误码
│  │  ├─ auth_register_test.go      # 注册 → 验证 → 建 Key 全链
│  │  ├─ key_create_test.go         # 绑在架分组；下架分组 409
│  │  ├─ error_codes_test.go        # 33 码各一条，两两不重码
│  │  └─ isolation_test.go          # 跨用户隔离：A 的 7 个数字不被 B 改
│  └─ db/                           # 迁移、约束、不变量（真库）
│     ├─ migrate_up_down_test.go    # 0001–0009 各跑一遍 up/down
│     ├─ ledger_append_only_test.go # UPDATE / DELETE 被拒率 100%
│     ├─ invariant_test.go          # 17 条不变量逐条打红
│     └─ seed_test.go               # 首播 11 模型 / 24 分组行数对得上
│
├─ e2e/                             # 真实浏览器
│  ├─ flows/                        # 完整用户路径
│  │  ├─ register-flow.spec.ts      # 注册 → 充值 → 建 Key → 调用 → 开票
│  │  ├─ providers-hall.spec.ts     # 24 分组筛选、排序、下架
│  │  └─ refund-flow.spec.ts        # 申请 → 审批 → 退款 → 冲销
│  ├─ visual/                       # 截图比对
│  │  ├─ home.spec.ts               # 首页开屏地球落位后截图
│  │  └─ console-overview.spec.ts   # 三卡 + 7 日柱图 + 模型占比
│  └─ a11y/                         # 可访问性扫描
│     ├─ console.spec.ts            # 侧栏 7 项键盘可达 + 焦点可见
│     └─ contrast.spec.ts           # 四色状态徽标对比度 ≥ 4.5
│
├─ perf/                            # 压力测试；要压测才建
│  └─ scenarios/
│     ├─ chat-1000-mixed.js         # 1000 次混跑：成功/上游失败/取消/未知
│     └─ providers-list-500vu.js    # 大厅列表 500 并发，首屏 ≤ 1.5s
│
├─ mobile/                          # 移动端流程；有 App 才建
│  └─ flows/
│     ├─ login.yaml                 # 登录 → 控制台
│     └─ topup.yaml                 # 兑换码充值
│
├─ fixtures/                        # 各层共用的数据与账号
│  ├─ accounts.json                 # user_a / user_b / admin，各带初始额度
│  ├─ models.json                   # 11 个启用模型 + 2 个下线，四价齐全
│  ├─ groups.json                   # 24 分组：倍率、可用模型、三项指标
│  └─ ledger_cases.json             # 四类入账各 3 条，供复算断言
│
└─ helpers/                         # API 客户端、等待、清理
   ├─ api_client.ts                 # 带鉴权的请求封装，自动带 trace id
   ├─ db_reset.ts                   # 每例前清库 + 重跑种子
   ├─ wait.ts                       # 状态轮询（退款审批、对账落库）
   └─ fixtures.ts                   # 按 fixtures/ 造实体，返回 id
```

**示例里的文件名取自真实项目的形状，不是通用的 `foo_test.ts`。** 换成自己的只要保持三个特征：**名字说清测什么**（`ledger_append_only_test.go` 而不是 `db_test.go`）、**一条断言对应一条规则或 AC**、**失败时看文件名就知道该翻哪节文档**。

**开发自写的单元测试跟源码放 `src/`；`test/` 归测试角色做独立验证。** 开发交付前只跑 `test:unit`。

---

## 四、deploy/ —— 四层固定，层内按需

**分层依据是「跑在什么平台上」，不是文件类型。** 一个平台一个目录，装齐这个平台要的一切。

```text
deploy/
├─ docker/
│  ├─ Dockerfile                 # 统一构建入口：多阶段，--target frontend|backend
│  ├─ .dockerignore              # 没有它，node_modules 与 .git 会进镜像
│  ├─ docker-compose.dev.yml     # 开发覆盖：源码挂载、暴露 db 端口
│  ├─ docker-compose.test.yml    # 测试覆盖
│  └─ docker-compose.prod.yml    # 生产覆盖：重启策略、资源限制、对外端口
│
├─ k8s/                          # 真上集群才建
│  ├─ config/                    # 先建；configmap 不到位，workload 起不来
│  │  ├─ namespace.yaml
│  │  ├─ configmap.yaml
│  │  └─ secret.yaml.example     # base64 不是加密，只放示例
│  ├─ workload/
│  │  ├─ deployment-frontend.yaml
│  │  ├─ deployment-backend.yaml
│  │  └─ statefulset-db.yaml     # 用云数据库就不建
│  ├─ network/
│  │  ├─ service-frontend.yaml
│  │  ├─ service-backend.yaml
│  │  └─ ingress.yaml
│  └─ storage/
│     └─ pvc.yaml
│
├─ scripts/
│  ├─ start.sh                    # 拉起已构建的服务
│  ├─ stop.sh                     # 优雅停止
│  ├─ deploy.sh                   # 构建 → 迁移 → 启动 → 冒烟
│  ├─ rollback.sh                 # 应用与数据库回滚
│  └─ migrate.sh                  # 数据库迁移，可单独执行
│
└─ env/
   ├─ .env.example                # 变量清单与默认值，**唯一入库的一份**
   ├─ .env.dev                    # 真实值，整目录不入库
   ├─ .env.test
   └─ .env.prod
```

`nginx/` 这类配置归到使用它的平台目录下（如 `docker/nginx/`），不单列。

**环境变量不在项目根，全部在 `deploy/env/`**，应用与脚本都从那里读。**真实值整目录不入库，只放行 `.env.example`：**

```gitignore
# 真实环境变量整目录不入库，只放行示例文件
deploy/env/*
!deploy/env/.env.example

# 临时区与工具产物
docs/99-tmp/
docs/01-设计/03-原型图/.file-versions/
*.artifact.json
.openchamber/
```

**`.env.example` 是变量清单的唯一出处**：写清每个变量的名字、默认值、必填与否，值留空或写占位。每个环境要哪几个变量、放在哪台服务器上，写在 `docs/06-运维/01-服务详细情况/01-服务详细情况.md`，两处不重复同一份清单。

---

## 五、docs/ —— 照抄，不改

```text
docs/
├─ README.md                            # 在途需求表 + 待补清单
│
├─ 01-设计/
│  ├─ 01-设计系统/
│  │  └─ 01-设计系统.md                 # 项目级 · 不带版本
│  ├─ 02-组件规范/
│  │  └─ 01-组件规范.md                 # 项目级 · 不带版本
│  ├─ 03-原型图/
│  │  ├─ index.html                     # 单文件 + hash 路由；单端时只有一个 html
│  │  ├─ assets/
│  │  └─ .file-versions/                # 工具内部版本库，不入库
│  └─ 04-页面规格/
│     └─ 01-<短名>_页面规格_v<版本>_<日期>.md      # 链上
│
├─ 02-产品/
│  ├─ 01-需求池/
│  │  └─ 01-需求池清单.md               # 一张表：需求全在这一份里
│  ├─ 02-BRD商业需求/
│  │  └─ 01-BRD商业需求.md              # 项目级
│  ├─ 03-MRD市场需求/
│  │  └─ 01-MRD市场需求.md              # 项目级
│  ├─ 04-PRD产品需求/
│  │  └─ <需求编号>/                    # 一个需求一个目录
│  │     └─ 01-<短名>_PRD_v<版本>_<日期>.md       # 链上 · 一册不拆
│  └─ 05-产品术语表/
│     └─ 01-产品术语表.md               # 项目级
│
├─ 03-架构/
│  ├─ 01-系统架构/
│  │  └─ 01-<短名>_系统架构_v<版本>_<日期>.md     # 链上
│  ├─ 02-架构图示/
│  │  ├─ 01-<短名>_总览_v<版本>.html              # ≤12 节点时单张平铺
│  │  └─ 02-<短名>_模块与依赖_v<版本>.html
│  ├─ 03-状态机/
│  │  ├─ 01-<实体>_状态机_v<版本>.html
│  │  └─ 02-<实体>_状态机_v<版本>.html
│  ├─ 04-项目环境规范/
│  │  └─ 01-项目环境规范.md             # 项目级
│  ├─ 05-数据库设计/
│  │  └─ 01-<短名>_数据库设计_v<版本>_<日期>.md   # 链上
│  ├─ 06-ER图/
│  │  ├─ 01-<域>_ER图_v<版本>.html
│  │  └─ 02-<域>_ER图_v<版本>.html
│  └─ 07-ADR重要决策/
│     └─ <ADR编号>/
│        └─ 01-<短名>_ADR_v<版本>_<日期>.md       # 链上
│
├─ 04-开发/
│  ├─ 01-代码规范/
│  │  └─ 01-代码规范.md                 # 项目级
│  ├─ 02-技术方案/
│  │  └─ <需求编号>/
│  │     └─ 01-<短名>_技术方案_v<版本>_<日期>.md  # 链上 · 先落盘，再写代码
│  └─ 03-实现记录/
│     └─ <需求编号>/
│        └─ 01-<短名>_实现记录_v<版本>_<日期>.md  # 链上 · 沿用需求编号
│
├─ 05-测试/
│  ├─ 01-测试策略/
│  │  └─ 01-测试策略.md                 # 项目级
│  ├─ 02-测试方案/
│  │  └─ <需求编号>/
│  │     └─ 01-<短名>_测试方案_v<版本>_<日期>.md  # 链上
│  └─ 03-测试报告/
│     └─ <需求编号>/
│        └─ 01-<短名>_测试报告_v<版本>_<日期>.md  # 链上 · 报告以 CI 为准
│
├─ 06-运维/
│  ├─ 01-服务详细情况/
│  │  └─ 01-服务详细情况.md             # 项目级
│  ├─ 02-部署拓扑/
│  │  └─ 01-<环境>_部署拓扑.html        # 一张图解所有环境；超 14 节点按环境拆成 02/03…
│  ├─ 03-运维手册/
│  │  └─ 01-运维手册.md                 # 项目级
│  └─ 04-发布记录/
│     └─ 01-发布记录.md                 # 项目级 · 单份追加，不按版本拆
│
├─ 07-审查/                            # 编号排最后，但 CR1 在阶段 1 后就落盘
│  ├─ 01-审查清单/
│  │  └─ 01-审查清单.md                 # 项目级 · init-project 生成
│  └─ 02-审查报告/
│     └─ <需求编号>/
│        ├─ 01-<短名>_CR1_设计与产品_v<版本>_<日期>.md
│        ├─ 02-<短名>_CR2_架构与实现_v<版本>_<日期>.md
│        └─ 03-<短名>_CR3_验证与发布_v<版本>_<日期>.md
│
├─ 99-tmp/                             # 临时区，任务结束清空；不入库
│  └─ <角色>-<任务>/                     # 一个角色一次任务一个目录
│     ├─ 01-任务单.md
│     ├─ 02-原料/                       # 问答稿、备选方案、草稿、调研
│     ├─ 03-receipts/                   # Archify 交付收据 json
│     └─ 04-截图/
│
├─ 99-外部文档/                        # 对外联调：接口说明、对接方案、对方资料
│  └─ <来源>/
│     ├─ 01-<系统>_接口说明_v<对方版本>_<日期>.pdf
│     └─ 02-<系统>_字段对照表_<日期>.xlsx
│
├─ 99-模板文档/                        # 工作流副本，随项目走；init-project 复制
│  ├─ project-layout.md                # 本文件
│  ├─ questioning.md                   # 提问协议
│  ├─ doc-templates/                   # 产物模板
│  └─ roles/                           # 岗位规则
│
└─ 99-归档/                            # 只读
   ├─ 版本/<需求编号>/v<旧版本>/        # 被新版替代的整条链
   ├─ 废弃/<需求编号>/v<版本>/          # 整条需求被取消
   └─ 发布/v<上线版本>/                 # 一次上线的整批快照
```

**一句话记住三件事**：**一个产物一个目录**（给增补留位置）、**项目级不带版本**（改就替换）、**链上带版本和日期**（同版返修也能区分）。

**判断文档有效还是废弃，看位置不看状态字段。** 在工作目录里就是有效，在 `99-归档/废弃/` 里就是废弃。状态字段会忘记改，位置不会说谎。

**归档只进不出。** 从 `99-归档/` 往外搬东西说明判断错了，要先说清为什么。要恢复旧版，正确做法是从归档里 `cp` 出来当新版本升号，不是把它搬回去顶替当前版。
