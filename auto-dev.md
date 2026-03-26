# Auto-Dev v5

$ARGUMENTS

## 铁律

1. **先读后写** — 改文件前必须 Read 它
2. **先查后建** — 建新代码前 Grep 是否已有类似实现
3. **不破坏现有** — 改导出/接口/schema 前 Grep 所有消费者确认兼容
4. **跟随项目** — CLAUDE.md > 项目隐性约定 > 通用实践
5. **不问不启** — 不确认、不启动服务、不跑 build/test、不生成管理文件
6. **只推 origin** — commit: `type(scope): 中文描述`

---

## Phase 0: 启动

### 输入解析

| 输入 | 识别 | 处理 |
|------|------|------|
| 文本 | 默认 | 直接用 |
| 文件路径 | `/` `~` `./` 开头 | Read 内容 |
| Issue URL | 含 github.com/issues/ | `gh issue view` |
| 多需求 | `+` 分隔 | 逐个执行 |
| 空 | 无参数 | 审计项目，列出待完善项让用户选择（不自动全做） |

### 意图分类（决定后续行为差异）

| 意图 | 判断依据 | 行为差异 |
|------|----------|----------|
| **fix** | "修复/bug/报错/崩溃" | 最小改动原则，不重构周边代码，写回归测试 |
| **feat** | "添加/新增/实现/支持" | 完整实现，含测试+文档更新 |
| **refactor** | "重构/优化结构/整理" | 行为不变，先写测试锁定行为再改 |
| **perf** | "性能/慢/优化/卡" | 定位瓶颈，针对性优化，避免过度 |
| **chore** | "升级/配置/迁移/清理" | 最小风险，逐步执行 |

### 环境感知（并行 Glob/Read）

```
技术栈:   package.json | pom.xml | go.mod | Cargo.toml | pyproject.toml
包管理器: pnpm-lock→pnpm | yarn.lock→yarn | package-lock→npm | bun.lockb→bun
项目类型: 前端 | 后端 | 全栈 | 库 | CLI
Monorepo: apps/ | packages/ | workspaces
代码风格: .eslintrc* | .prettierrc* | biome.json | .editorconfig
数据库:   prisma/ | supabase/ | drizzle/ | migrations/
测试框架: vitest | jest | pytest | go test | 无测试（记录但不强建）
```

### 学习项目约定

读取 CLAUDE.md + 项目记忆。然后快速扫描：
- `git log --oneline -10` → commit 风格
- 2-3个核心文件 → 命名/分层/模式/注释风格
- **新文件放置规则**：扫描目录结构，记录各类文件的存放位置（组件→components/，API→routes/，工具→utils/，类型→types/），后续创建文件严格遵循

### 幂等检查

开始前扫描项目是否**已经实现了需求的部分或全部**：
- Grep 需求中的关键词（函数名、路由、组件名）
- 如果已完全实现 → 告知用户 "已存在，无需重复"，终止
- 如果部分实现 → 标记已完成部分，只做剩余

### 选择模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **快速** | < 5文件 / fix / chore | 开发 → 推送 |
| **标准** | 5-20文件 / feat / refactor | 规划 → 理解 → 开发 → 审查 → 推送 |
| **大型** | > 20文件 / 跨模块 | 同标准 + feature分支 + 备份tag + 串行分批 |

### 安全网
- 标准/大型：记录起始 commit hash
- 大型：`git tag auto-dev-backup-$(date +%s)` + feature分支
- 未提交改动 → stash（结束时提醒）
- 有未完成 auto-dev Task → 从断点继续

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect")：
- 输入：技术栈 + 需求 + CLAUDE.md 约束 + 目录结构
- 输出：文件清单（创建/修改）+ 依赖顺序 + 数据模型方案

转化为 TaskCreate 任务。**不commit。**

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer") 分析规划涉及的已有文件：
- 执行路径、数据流、import 关系
- 结果传递给开发阶段，**不输出**。不commit。

## 阶段3: 开发

### 每个任务的开发循环

```
读 → 查 → 写 → 检 → 格 → 存
```

**读**: Read 目标文件（>500行时 Grep 定位后读局部）
**查**: Grep 可复用代码。有→复用，无→新建
**写**: 实现功能
  - 新文件按 Phase 0 学到的放置规则创建
  - 新依赖立即 `{pm} add {pkg}`
  - DB变更同步迁移文件
  - 环境变量 → 默认值 + .env.example
  - **性能注意**：
    - DB查询避免 N+1（用 include/join/eager load）
    - 前端避免不必要 re-render（memo/useMemo/useCallback 按需用，不滥用）
    - 列表渲染加 key，大列表考虑虚拟滚动
    - API 响应只返回必要字段
**检**: 改了导出→Grep消费者确认兼容；改了schema→检查查询代码
**格**: prettier/biome/rustfmt/ruff（有配置才跑）
**存**: `type(scope): 描述`

### 测试策略（写但不跑）

| 意图 | 测试要求 |
|------|----------|
| fix | 必须写回归测试（覆盖修复的 case） |
| feat | 写核心逻辑的单元测试（跟随项目已有测试风格） |
| refactor | 先写/补测试锁定现有行为，再重构 |
| perf | 不写测试 |
| chore | 不写测试 |

规则：
- 项目无测试框架 → 不写测试（不强建测试体系）
- 项目有测试 → 跟随已有模式（文件位置、命名、断言风格）
- 测试文件跟随项目约定放置（`__tests__/` 或 `*.test.*` 或 `*_test.*`）
- commit: `test(scope): 描述`

### 完成标准（每个功能模块必须满足）

- [ ] 功能按需求实现完整
- [ ] 无硬编码密钥/凭证
- [ ] 导出变更已同步消费者
- [ ] 新文件在正确目录
- [ ] DB迁移文件同步（如有schema变更）
- [ ] 测试已写（按意图策略）
- [ ] 格式化通过（如有配置）

### 并行开发（仅大型模式）

> v4 的 bug 修复：同分支两个 Agent 同时 git add/commit 会冲突

策略：**不并行 commit，只并行编码**
1. 多个 Agent 同时编写不同文件集的代码（不 commit）
2. Agent 完成后，主流程**串行** commit 每个 Agent 的产出
3. 这样 staging area 不冲突

### 错误处理

| 类型 | 策略 |
|------|------|
| 语法/类型错误 | 直接修复 |
| 依赖缺失 | 安装后继续 |
| 环境变量缺失 | .env.example + 代码默认值 |
| 循环依赖 | 提取独立模块 |
| 导出不兼容 | 同步更新消费者 |
| 权限/网络 | 记录跳过 |
| 未知 | 换实现1次，仍失败跳过 |

## 阶段4: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer")：
- 输入：`git diff {起始hash}..HEAD`
- 重点：安全漏洞、铁律违反、CLAUDE.md违规、逻辑错误、性能问题（N+1等）
- 有问题→修复→`fix: 描述`

## 阶段5: 推送

1. DB项目 → seed 数据覆盖新功能 → `chore(data): 描述`
2. `git push origin {branch}`（失败→rebase重试→二次失败报错不强推）
3. 大型模式 → 非显然的技术决策存项目记忆
4. 有 stash → 提醒 pop
5. 汇报

---

## 上下文预算

- 按需 Read，不预读
- 架构感知 + 代码审查用独立 Agent
- 任务 > 10 个时分批（每批 5 个）
- 阶段间只输出状态行

---

## 汇报

**快速**:
```
Auto-Dev | 项目名 | fix
改动: file1, file2 | 测试: test1
成果: 一句话 ✅
```

**标准/大型**:
```
══════════════════════════════
Auto-Dev v5 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件
✅ 理解 — 分析了N个模块
✅ 开发 — N功能 N次commit
✅ 测试 — 写了N个测试
✅ 审查 — N问题已修 / 无问题
✅ 推送 — branch → origin
──────────────────────────────
成果: A / B / C
遗留: X(原因)
══════════════════════════════
```
