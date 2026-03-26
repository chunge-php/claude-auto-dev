# Auto-Dev v12

$ARGUMENTS

<!-- 铁律: 先读后写 | 先查后建 | 不破坏现有 | 跟随项目 | 不问不启 | 只推origin -->
<!-- 循环: 读→查→写→检→格→存 | commit说WHY | 单commit<300行 -->
<!-- 快速: 假设→验证→修→验证→推 | 标准: 规划→校验→理解→开发→验证→审查→推→CI | 大型: +分支+PR+切片 -->

## 铁律

1. **先读后写** — 改文件前必须 Read
2. **先查后建** — 新代码前 Grep 已有实现
3. **不破坏现有** — 改导出/接口/schema/类型前 Grep 所有消费者
4. **跟随项目** — CLAUDE.md > 项目约定 > 通用实践
5. **不问不启** — 不确认、不启动服务、不生成管理文件
6. **只推 origin** — commit: `type(scope): 做了什么 — 因为什么`

## 决策框架

| 场景 | 决策 |
|------|------|
| 模糊需求 | 选最常见理解，commit 记录假设 |
| 多种方案 | 选最简 |
| 不确定安全性 | 不改，跳过汇报 |
| 范围膨胀 | 只做明确要求的 |
| 约定 vs 最佳实践 | 跟随约定 |
| 同类错误 ≥3 | 停，分析根因 |
| context 紧张 | 保存进度到 Task 描述 → 降级快速模式完成剩余 |
| CI 失败 | 最多修2轮 |
| 需求过大且模糊 | 先拆解范围 |
| 发现不可行 | 中止，汇报原因+替代方案 |
| **Monorepo 范围** | Grep 需求关键词定位包；模糊时选离需求最近的包；跨包改动明确标注 |

---

## Phase 0: 启动

### 输入

| 输入 | 处理 |
|------|------|
| 文本 | 直接用 |
| 文件路径 | Read |
| Issue URL | `gh issue view` |
| `+`分隔 | 逐个 |
| 空 | 审计：健康(tsc/lint/audit) → 完整性(TODO/路由匹配) → 质量(重复/大文件) |

### 意图

| 意图 | 判断 | 差异 |
|------|------|------|
| **fix** | 修复/bug/报错 | 最小改动 + 回归测试 |
| **feat** | 添加/新增/实现 | 完整实现 + 测试 + 文档 + UI完整性 |
| **refactor** | 重构/整理 | 锁行为再改 + 清死代码 |
| **perf** | 性能/慢/卡 | 定位瓶颈 |
| **chore** | 升级/配置/迁移 | 最小风险 |

### 环境感知

**快速**（<3秒）：CLAUDE.md + 技术栈 + 包管理器，其余按需
**标准/大型**：全面检测（技术栈、包管理器、项目类型、Monorepo、代码风格、数据库、测试框架、i18n、Git Hooks、状态管理、认证方案、API响应格式）+ 学习约定（git log + 核心文件 → 命名/分层/文件放置/认证/状态管理）
**大型 + Monorepo**：额外检测分支命名约定（`git branch -a` 观察 feat/feature/dev 前缀模式）

### 幂等 → Grep 关键词，已实现则终止

### 模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **脚手架** | 新项目 | 初始化(+远程仓库) → 开发 → 验证 → 推送 |
| **快速** | <5文件/fix/chore | 假设→验证→开发→验证→推送 |
| **标准** | 5-20文件/feat/refactor | 规划→校验→理解→开发→验证→审查→推送→CI |
| **大型** | >20文件/跨模块 | 同标准 + feature分支(跟随命名约定) + tag备份 + 切片交付 + draft PR + 进度存档 |

### 安全网

起始hash记录。大型加 backup tag + feature分支。未提交→stash。未完成Task→续传。大型进度存 `.auto-dev-progress.json`。

### 降级与中止

5次失败→降快速 | 任务>15→只做前5 | >20错→只修error | context紧张→保存进度到Task后降级
不可行/需用户信息/代码异常 → 中止

<!-- ═══════════ 快速模式到此为止，以下仅标准/大型需要 ═══════════ -->

---

## 脚手架（仅新项目）

脚手架初始化 → 依赖 + CLAUDE.md + .env.example → `init: 项目初始化` → `$GH_TOKEN` 创建远程仓库(SSH) → push

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect")，提示词模板：

```
分析以下项目并为需求设计实现方案。

项目信息:
- 技术栈: {技术栈}
- 目录结构: {ls输出}
- 约束: {CLAUDE.md内容}
- 认证方案: {auth方式}
- 状态管理: {store方式}
- API格式: {响应格式}

需求: {用户需求}

请输出:
1. 需要创建的文件列表（含目录）
2. 需要修改的文件列表（含修改意图）
3. 按依赖顺序排列（底层先于上层）
4. 数据模型变更方案（如需）
5. Monorepo时标注每个文件属于哪个包
```

**规划校验**：Glob 确认要修改的文件存在、要创建的文件不存在；不存在→调整。
**全栈分层**：DB → Model/Service → API(+认证) → 状态层 → UI → 页面(+守卫)。每层单独 commit。
**大型切片**：按功能切片，每片含完整前后端，完成即 push。

TaskCreate。不commit。

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer")，提示词模板：

```
深度分析以下文件的代码架构。

文件列表: {规划阶段确定的已有文件}

请分析:
1. 每个文件的核心职责和执行路径
2. 文件间的 import 依赖关系
3. 认证/权限检查的实现方式
4. 被修改的 type/interface 的完整引用链（谁在用它）
5. 项目中的设计模式和抽象层
```

## 阶段3: 开发

### 快速模式"先想后做"

1. 形成假设（问题在哪/怎么改）→ 2. Read+Grep 验证 → 3. 假设错则调整。脑中完成，不输出。

### 会话内学习：记住本次错误/模式，后续避免。同类≥3→根因分析。

### 开发循环：读→查→写→检→格→存

**读**: Read 目标文件
**查**: Grep 可复用代码
**写**: 核心约束：
- **跟随项目一切已有模式**——检测到什么就跟随什么，没有就不强加
- DB迁移只加法 | 环境变量→默认值+.env.example | i18n→locale文件
- 新路由/API 默认加认证 | 新依赖先检查替代
- 性能：N+1/索引/分页/虚拟滚动
- feat+新页面：三态+响应式+a11y
- 遵循语言惯用写法
- **Monorepo**：确认改动在正确的包内，跨包引用用正确的包名

**检**: 导出→消费者 | schema→查询 | type→依赖图 | refactor→删死代码
**格**: git hooks→跳过 | 否则项目工具 | 无→跳过
**存**: `type(scope): WHY` | 单commit<300行 | 不混合无关改动

### 文档（feat）：README/API docs + CHANGELOG → `docs: 描述`
### 测试（写不跑）：fix→回归 | feat→单测 | refactor→锁行为 | perf/chore→不写
### 完成标准：**必检**（完整/无密钥/同步消费者/文件位置/惯用写法）+ **条件检**（仅相关时）
### 一致性：独立→保留，不独立→revert
### 并行（大型）：并行编码，串行commit，无共同文件
### 错误：语法→修 | 依赖→装 | 环境变量→默认值 | 循环依赖→提取 | 类型→沿图更新 | 网络→跳 | 未知→换1次

## 阶段4: 验证（所有模式）

秒级静态检查（tsc/eslint/py_compile/cargo check/go vet），超时10s跳过。有错→修。>20错→只修error。

## 阶段5: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer")，提示词模板：

```
审查以下代码变更。

Diff: {git diff 起始hash..HEAD}
项目约定: {CLAUDE.md}

审查清单（按优先级）:
1. 安全漏洞（注入/XSS/硬编码密钥/未授权访问）
2. 逻辑错误和边界情况
3. 是否违反铁律（未读就写/未查就建/破坏导出）
4. CLAUDE.md 违规
5. 性能问题（N+1/缺索引/不必要渲染）
6. 类型安全问题
7. 缺失认证/权限检查

只报告 HIGH confidence 的问题。
```

有问题→修→`fix: 描述`

## 阶段6: 推送

`git diff --stat` 确认范围 → seed(DB项目) → `git push`（失败→rebase→二次报错）→ 大型存记忆+progress

## 阶段7: 收尾

CI检查（通过/失败修2轮/pending注明）→ PR(大型draft) → stash提醒

### 汇报

**快速**: `Auto-Dev | 项目名 | fix ⏐ 改动: file(+N-N) | 验证: ✅ ⏐ 成果: 一句话 ✅`

**标准/大型**:
```
══════════════════════════════
Auto-Dev v12 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件（校验通过）
✅ 理解 — N模块
✅ 开发 — N功能 N commits (+X/-Y)
✅ 验证 — tsc ✅ eslint ✅
✅ 审查 — N问题已修
✅ 推送 — branch → origin
✅ CI — passed
✅ PR — #123 (draft)
──────────────────────────────
成果: A / B / C
回滚: X(原因)
遗留: Y(原因)
建议: Z
══════════════════════════════
```

---

## 示例

### 快速：bug 修复（含错误恢复）

> `/auto-dev 修复文章详情页点赞数不更新`

```
Phase 0: fix | 快速 | Next.js+pnpm
假设: 点赞API返回后前端状态未更新
Read useLike.ts → mutate后未revalidate ✓
修复 → commit: fix(article): 点赞后刷新计数 — mutate未触发revalidate
验证: tsc → error: revalidate not exported → 修复用法 → tsc ✅
推送 ✅
```

### 标准：新功能（Next.js 全栈）

> `/auto-dev 添加文章收藏功能`

```
Phase 0: feat | 标准 | Next.js+Prisma+Zustand+next-auth+Tailwind(dark)
规划: architect → 6任务 | 校验: Glob确认文件 ✅
理解: explorer → Article模型 + auth中间件
开发: schema→API(auth+格式)→store(zustand)→组件(dark+a11y)→页面(三态+响应式)→test+docs
diff --stat: 8files +320/-12 | 验证: tsc✅ eslint✅ | 审查: 0问题
推送 → CI passed ✅
```

### 标准：后端功能（Python FastAPI）

> `/auto-dev 添加用户注册接口，支持邮箱+密码`

```
Phase 0: feat | 标准 | FastAPI+SQLAlchemy+Alembic+pytest
规划: architect → 4任务 | 校验 ✅
理解: explorer → User模型 + auth依赖 + 现有API模式(BaseResponse)
开发:
  1. feat(db): User表+alembic迁移 — 支持邮箱注册
  2. feat(api): POST /auth/register — 密码bcrypt, 遵循BaseResponse
  3. feat(service): 注册逻辑 — 邮箱唯一校验+密码强度
  4. test(auth): 注册成功/重复邮箱/弱密码
验证: py_compile ✅ | 审查: 0问题
推送 → CI passed ✅
```

---

## 上下文预算

按需Read | 架构+审查用独立Agent | 任务>10→分批5个 | 只输出状态行 | 大型按切片 | 紧张→保存进度后降级
