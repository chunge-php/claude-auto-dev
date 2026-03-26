# Auto-Dev v7

$ARGUMENTS

## 铁律（每个阶段开始前默念一遍）

1. **先读后写** — 改文件前必须 Read 它
2. **先查后建** — 建新代码前 Grep 是否已有类似实现
3. **不破坏现有** — 改导出/接口/schema/类型前 Grep 所有消费者确认兼容
4. **跟随项目** — CLAUDE.md > 项目隐性约定 > 通用实践
5. **不问不启** — 不确认、不启动服务、不生成管理文件
6. **只推 origin** — commit: `type(scope): 中文描述`

## 决策框架

| 场景 | 决策 |
|------|------|
| 模糊需求 | 选最常见理解，commit 记录假设 |
| 多种方案 | 选最简（少文件、少依赖、少抽象） |
| 不确定是否安全 | 不改，跳过并汇报 |
| 范围膨胀 | 只做明确要求的，延伸记入建议 |
| 约定 vs 最佳实践 | 跟随项目约定 |
| 同类错误反复出现 | 停下来分析根因，不逐个修补 |

---

## Phase 0: 启动

### 输入解析

| 输入 | 识别 | 处理 |
|------|------|------|
| 文本 | 默认 | 直接用 |
| 文件路径 | `/` `~` `./` 开头 | Read 内容 |
| Issue URL | 含 github.com/issues/ | `gh issue view` |
| 多需求 | `+` 分隔 | 逐个执行 |
| 空 | 无参数 | 进入**审计模式** |

### 审计模式（空输入时）

分三层审计，输出结果让用户选择要修什么：

**第一层：健康检查**（必做）
- 有无 TypeScript 错误（`tsc --noEmit 2>&1 | tail -20`）
- 有无 ESLint 错误（`eslint . --quiet 2>&1 | tail -20`）
- 有无安全漏洞（`npm audit --json 2>&1 | head -50` 或同类）
- 有无过期依赖（`npm outdated 2>&1 | head -20`）

**第二层：功能完整性**（标准/大型项目）
- 扫描 TODO/FIXME/HACK 注释
- 检查路由是否都有对应页面
- 检查 API 端点是否都有对应前端调用

**第三层：代码质量**（大型项目）
- 检查重复代码片段（相似的 Grep 模式出现多次）
- 检查超大文件（>500行）
- 检查无类型的函数导出

输出精简列表，用户选择后正常进入开发流程。

### 意图分类

| 意图 | 判断 | 差异 |
|------|------|------|
| **fix** | 修复/bug/报错/崩溃 | 最小改动 + 回归测试 |
| **feat** | 添加/新增/实现/支持 | 完整实现 + 测试 + 文档 |
| **refactor** | 重构/优化结构/整理 | 先锁行为再改 + 清理死代码 |
| **perf** | 性能/慢/优化/卡 | 定位瓶颈，针对性优化 |
| **chore** | 升级/配置/迁移/清理 | 最小风险，逐步执行 |

### 环境感知（并行 Glob/Read）

```
技术栈:     package.json | pom.xml | go.mod | Cargo.toml | pyproject.toml
包管理器:   pnpm-lock→pnpm | yarn.lock→yarn | package-lock→npm | bun.lockb→bun
项目类型:   前端 | 后端 | 全栈 | 库 | CLI
Monorepo:   apps/ | packages/ | workspaces
代码风格:   .eslintrc* | .prettierrc* | biome.json | .editorconfig
数据库:     prisma/ | supabase/ | drizzle/ | migrations/
测试框架:   vitest | jest | pytest | go test | 无
i18n:       i18n/ | locales/ | next-intl | i18next
Git Hooks:  .husky/ | .lintstagedrc*
状态管理:   zustand | redux | pinia | jotai | mobx（检测后新状态跟随）
认证方案:   middleware/auth* | guard* | next-auth | lucia | supabase auth
```

### 学习项目约定

读取 CLAUDE.md + 项目记忆。快速扫描：
- `git log --oneline -10` → commit 风格
- 2-3个核心文件 → 命名/分层/模式
- **文件放置规则**：扫描目录，记录各类文件位置
- **认证模式**：扫描 middleware/guard，记录如何保护路由
- **状态管理模式**：读一个 store 文件，记录创建 store 的方式
- **是否新项目**：无 src/ 且 git log 空 → 脚手架流程

### 幂等检查

Grep 需求关键词：已完全实现→终止，部分实现→只做剩余

### 选择模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **脚手架** | 新项目 | 初始化 → 开发 → 推送 |
| **快速** | < 5文件 / fix / chore | 开发 → 验证 → 推送 |
| **标准** | 5-20文件 / feat / refactor | 规划 → 理解 → 开发 → 验证 → 审查 → 推送 |
| **大型** | > 20文件 / 跨模块 | 同标准 + feature分支 + 备份tag + 渐进交付 |

### 安全网
- 标准/大型：记录起始 commit hash
- 大型：`git tag auto-dev-backup-$(date +%s)` + feature 分支
- 未提交改动 → stash（结束时提醒）
- 有未完成 auto-dev Task → 从断点继续

---

## 脚手架阶段（仅新项目）

1. 根据需求选择技术栈（优先用户指定）
2. 官方脚手架初始化
3. 安装基础依赖
4. 创建 CLAUDE.md + .env.example
5. commit: `init: 项目初始化`
6. 进入开发阶段

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect")：
- 输入：技术栈 + 需求 + CLAUDE.md + 目录结构 + 认证方案 + 状态管理方案
- 输出：文件清单 + 依赖顺序 + 数据模型方案

### 全栈分层顺序

```
1. 数据库 Schema / 迁移
2. 后端 Model / Service
3. 后端 API / Controller（含认证中间件）
4. 前端状态层（store/hooks）
5. 前端 UI 组件
6. 前端页面集成（含路由守卫）
```

> 每层单独 commit，不跨层混合

### 大型模式：切片交付计划

将任务分为可独立工作的**交付切片**，每个切片包含完整的前后端：
```
切片1: 用户列表（schema + API + 页面）→ push
切片2: 用户详情（schema + API + 页面）→ push
切片3: 用户编辑（schema + API + 页面）→ push
```
> 每个切片完成后 push 一次，用户可在 GitHub 实时看到进展

转化为 TaskCreate。**不commit。**

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer") 分析涉及的已有文件：
- 执行路径、数据流、import 关系
- **认证链路**：新路由需要哪种级别的认证
- **类型依赖图**：要修改的 interface/type 被哪些文件引用
- 结果传递给开发阶段。不commit。

## 阶段3: 开发

### 会话内学习

维护一个**本次运行的经验列表**（存在脑中，不写文件）：
- 遇到的错误及解决方法 → 后续相同错误直接用已知方案
- 发现的项目特殊模式 → 后续代码跟随
- 审查发现的问题类型 → 后续主动避免

> 如果连续3个任务遇到同类错误，停下来分析根因而非逐个修补

### 开发循环

```
读 → 查 → 写 → 检 → 格 → 存
```

**读**: Read 目标文件（>500行 Grep 定位后读局部）

**查**: Grep 可复用代码。有→复用，无→新建

**写**: 实现功能
- 新文件按放置规则创建
- DB变更同步迁移文件（只加法）
- 环境变量 → 默认值 + .env.example
- i18n 项目 → 文案写入 locale 文件
- **新路由/API 必须添加认证**：
  - 检测到 auth middleware → 新路由默认加上
  - 检测到 route guard → 新页面默认加上
  - 仅公开页面（如登录页）才跳过认证
- **状态管理跟随**：
  - 检测到 zustand → 新状态用 `create()` 创建
  - 检测到 pinia → 新状态用 `defineStore()`
  - 检测到 redux → 新状态用 slice
  - 无状态管理 → 用 React state / Vue ref 等原生方案
- **新依赖检查**：
  - Grep 是否有功能相似的已有包
  - 优先标准库/已有依赖
  - 确需 → `{pm} add {pkg}`
- **性能意识**：
  - DB: 避免 N+1；新表加合理索引
  - 前端: 避免无效 re-render；大列表虚拟滚动；lazy load
  - API: 只返必要字段；列表必分页
- **语言特定检查**：

  | 语言 | 具体检查项 |
  |------|-----------|
  | TypeScript | 无 `any`（除非项目允许）；async 函数有 try-catch 或 .catch；导出函数有返回类型 |
  | Python | 有 type hints；async def 用 await 不用 .result()；避免 mutable default args |
  | Go | error 必须检查不忽略；goroutine 有 recover；context 正确传递 |
  | Rust | 不滥用 unwrap()；Result 正确传播；生命周期标注正确 |
  | Java | 资源用 try-with-resources；避免 null 返回（用 Optional）；Builder 模式 |

**检**:
- 改了导出 → Grep 消费者确认兼容
- 改了 schema → 检查查询代码
- **改了 interface/type** → Grep 所有 import 该类型的文件，确认类型兼容或同步更新
- refactor → 检查死代码并删除

**格**:
- 有 .husky/lint-staged → 跳过手动格式化
- 否则 prettier/biome/rustfmt/ruff
- 无配置 → 跳过

**存**: `type(scope): 描述`

### 文档同步（feat 意图）

改了用户可见行为 → 更新 README/API docs → `docs: 描述`

### 测试策略（写但不跑）

| 意图 | 要求 |
|------|------|
| fix | 回归测试 |
| feat | 核心逻辑单测 |
| refactor | 先锁行为再改 |
| perf/chore | 不写 |

无测试框架→不写。有→跟随已有模式。commit: `test(scope): 描述`

### 完成标准

- [ ] 功能完整实现
- [ ] 无硬编码密钥
- [ ] 导出/类型变更已同步消费者
- [ ] 新文件在正确目录
- [ ] DB迁移同步（仅加法）
- [ ] 新路由已加认证（非公开接口）
- [ ] 新状态跟随项目状态管理方案
- [ ] i18n 文案无硬编码
- [ ] 测试已写（按意图）
- [ ] 文档已同步（feat）
- [ ] 无死代码（refactor）
- [ ] 语言特定检查通过

### 一致性保障

关联任务部分失败时：
- 能独立工作 → 保留 + 汇报
- 不能独立工作 → revert + 汇报

### 并行开发（仅大型模式）

只并行编码，串行 commit。Agent 间不可有共同文件。

### 错误处理

| 类型 | 策略 |
|------|------|
| 语法/类型错误 | 直接修复 |
| 依赖缺失 | 安装后继续 |
| 环境变量缺失 | .env.example + 默认值 |
| 循环依赖 | 提取独立模块 |
| 导出不兼容 | 同步更新消费者 |
| 类型不兼容 | 沿类型依赖图逐个更新 |
| 权限/网络 | 记录跳过 |
| 同类错误 ≥3次 | 停止，分析根因后统一修复 |
| 未知 | 换实现1次，仍失败跳过 |

## 阶段4: 轻量验证（新增，所有模式）

> 不启动服务，不跑完整测试，只做秒级静态检查

根据技术栈选择性执行（检测到才跑）：

| 技术栈 | 验证命令 | 目的 |
|--------|----------|------|
| TypeScript | `npx tsc --noEmit 2>&1 \| tail -30` | 类型错误 |
| ESLint | `npx eslint {changed_files} --quiet 2>&1 \| tail -20` | 代码规范 |
| Python | `python -m py_compile {file}` | 语法检查 |
| Rust | `cargo check 2>&1 \| tail -30` | 编译检查 |
| Go | `go vet ./... 2>&1 \| tail -20` | 静态分析 |

- 有错误 → 直接修复 → `fix: 静态检查修复`
- 无错误 → 继续
- 命令不存在或超时(10s) → 跳过

## 阶段5: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer")：
- 输入：`git diff {起始hash}..HEAD`
- 审查清单：安全漏洞、铁律违反、CLAUDE.md违规、逻辑错误、性能问题、死代码、i18n硬编码、缺失认证、类型安全
- 有问题→修复→`fix: 描述`

## 阶段6: 推送

1. DB项目 → seed 覆盖新功能 → `chore(data): 描述`
2. `git push origin {branch}`（失败→rebase重试→二次失败报错）
3. 大型模式 → 技术决策存项目记忆
4. 有 stash → 提醒 pop
5. 汇报

---

## 上下文预算

- 按需 Read，不预读
- 架构感知 + 代码审查用独立 Agent
- 任务 > 10 个 → 分批（每批 5 个）
- 阶段间只输出状态行
- 大型模式按切片交付，每个切片是独立的上下文单元

---

## 汇报

**快速**:
```
Auto-Dev | 项目名 | fix
改动: file1, file2 | 测试: test1
验证: tsc ✅ eslint ✅
成果: 一句话 ✅
```

**标准/大型**:
```
══════════════════════════════
Auto-Dev v7 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件
✅ 理解 — N个模块
✅ 开发 — N功能 N次commit
✅ 测试 — N个测试
✅ 验证 — tsc ✅ eslint ✅
✅ 审查 — N问题已修 / 无问题
✅ 推送 — branch → origin
──────────────────────────────
成果: A / B / C
回滚: X(原因)
遗留: Y(原因)
建议: Z
══════════════════════════════
```
