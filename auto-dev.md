# Auto-Dev v11

$ARGUMENTS

<!-- 铁律: 先读后写 | 先查后建 | 不破坏现有 | 跟随项目 | 不问不启 | 只推origin -->
<!-- 循环: 读→查→写→检→格→存 | commit说WHY不说WHAT | 单commit<300行 -->
<!-- 模式: 快速(假设→验证→修→推) | 标准(规划→校验→理解→开发→验证→审查→推→CI) | 大型(+分支+PR+切片) -->

## 铁律

1. **先读后写** — 改文件前必须 Read
2. **先查后建** — 新代码前 Grep 已有实现
3. **不破坏现有** — 改导出/接口/schema/类型前 Grep 所有消费者
4. **跟随项目** — CLAUDE.md > 项目约定 > 通用实践
5. **不问不启** — 不确认、不启动服务、不生成管理文件
6. **只推 origin** — commit: `type(scope): 中文描述`

## 决策框架

| 场景 | 决策 |
|------|------|
| 模糊需求 | 选最常见理解，commit 记录假设 |
| 多种方案 | 选最简 |
| 不确定安全性 | 不改，跳过汇报 |
| 范围膨胀 | 只做明确要求的 |
| 约定 vs 最佳实践 | 跟随约定 |
| 同类错误 ≥3 | 停，分析根因 |
| context 紧张 | 降级快速模式 |
| CI 失败 | 最多修2轮 |
| 需求过大且模糊 | 先拆解范围，标注本次做哪些 |
| 发现不可行 | 立即中止，汇报原因+替代方案 |

---

## Phase 0: 启动

### 输入

| 输入 | 处理 |
|------|------|
| 文本 | 直接用 |
| 文件路径(`/` `~` `./`) | Read |
| Issue URL | `gh issue view` |
| `+`分隔 | 逐个执行 |
| 空 | 审计模式：健康(tsc/lint/audit) → 完整性(TODO/路由匹配) → 质量(重复/大文件) |

### 意图

| 意图 | 判断 | 差异 |
|------|------|------|
| **fix** | 修复/bug/报错 | 最小改动 + 回归测试 |
| **feat** | 添加/新增/实现 | 完整实现 + 测试 + 文档 + UI完整性 |
| **refactor** | 重构/整理 | 锁行为再改 + 清死代码 |
| **perf** | 性能/慢/卡 | 定位瓶颈 |
| **chore** | 升级/配置/迁移 | 最小风险 |

### 环境感知

**快速模式**（<3秒）：只读 CLAUDE.md + 检测技术栈/包管理器，其余按需

**标准/大型**：全面检测技术栈、包管理器、项目类型、Monorepo、代码风格、数据库、测试框架、i18n、Git Hooks、状态管理、认证方案、API响应格式。读已有 handler 提取 API 返回格式。

**学习约定**：CLAUDE.md + 项目记忆 + git log + 核心文件扫描 → 命名/分层/文件放置/认证/状态管理模式

### 幂等 → Grep 关键词，已实现则终止

### 模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **脚手架** | 新项目 | 初始化(+创建远程仓库) → 开发 → 验证 → 推送 |
| **快速** | <5文件/fix/chore | **假设→验证→**开发 → 验证 → 推送 |
| **标准** | 5-20文件/feat/refactor | 规划 → **校验** → 理解 → 开发 → 验证 → 审查 → 推送 → CI |
| **大型** | >20文件/跨模块 | 同标准 + feature分支 + tag备份 + 切片交付 + draft PR + 进度存档 |

### 安全网

标准/大型记录起始hash。大型加 backup tag + feature分支。未提交→stash。有未完成Task→续传。大型进度存 `.auto-dev-progress.json`。

### 降级与中止

5次工具失败→降快速 | 任务>15→只做前5 | 验证>20错→只修error
不可行/需用户信息/代码异常 → 中止，汇报原因+建议

---

## 脚手架（仅新项目）

1. 技术栈选择（优先用户指定）→ 官方脚手架初始化
2. 基础依赖 + CLAUDE.md + .env.example + .gitignore
3. `git init` + commit: `init: 项目初始化`
4. **创建远程仓库**：`$GH_TOKEN` + curl GitHub API 创建 repo → `git remote add origin` (SSH) → push
5. 进入开发阶段

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect") → 文件清单 + 依赖顺序 + 数据模型

**全栈分层**：DB → Model/Service → API(+认证) → 状态层 → UI → 页面(+守卫)。每层单独 commit。
**大型切片**：按功能切片（每片含完整前后端），完成即 push。

### 规划校验（新增）

architect 返回的文件清单必须校验：
- 要修改的文件 → Glob 确认存在，不存在则从清单移除或改为创建
- 要创建的文件 → Glob 确认不存在，已存在则改为修改
- 依赖顺序是否合理（下游不能先于上游）

校验失败 → 调整规划，不盲目执行。

TaskCreate。不commit。

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer") → 执行路径 + 数据流 + 认证链路 + 类型依赖图

## 阶段3: 开发

### 快速模式的"先想后做"

即使是 fix/chore，开发前也要：
1. **形成假设**：基于需求描述，猜测问题在哪 / 该怎么改
2. **验证假设**：Read + Grep 确认假设是否正确
3. **假设错误**：调整方向，不硬改

> 这3步在脑中完成，不输出，但必须经过

### 会话内学习：记住本次错误/模式，后续避免。同类≥3→根因分析。

### 开发循环：读→查→写→检→格→存

**读**: Read 目标文件
**查**: Grep 可复用代码

**写**: 实现功能，核心约束：
- **跟随项目一切已有模式**：文件放置、命名、分层、认证、状态管理、API格式、响应式、暗色模式、组件风格——检测到什么就跟随什么，没有就不强加
- DB迁移只加法（不drop/rename）
- 环境变量 → 默认值 + .env.example
- i18n项目 → locale 文件，不硬编码文案
- 新路由/API 默认加认证（公开接口除外）
- 新依赖前检查已有替代，优先标准库
- 性能：避免 N+1、新表加索引、列表分页、大列表虚拟滚动
- **feat + 新页面时**：Loading/Error/Empty 三态 + 响应式 + 基础a11y
- 遵循语言惯用写法

**检**: 改导出→消费者 | 改schema→查询代码 | 改type→依赖图更新 | refactor→删死代码
**格**: 有git hooks→跳过 | 否则用项目配置的工具 | 无配置→跳过

**存**: commit 纪律
- 格式：`type(scope): 做了什么 — 因为什么`（WHY 比 WHAT 重要）
- 单个 commit 改动不超过 **300行**，超过则拆分为多个功能粒度的 commit
- 不混合不相关的改动

### 文档（feat）

改了用户可见行为 → 更新 README/API docs。有 CHANGELOG → 追加。commit: `docs: 描述`

### 测试（写但不跑）

fix→回归 | feat→单测 | refactor→先锁行为 | perf/chore→不写。无框架→不写。

### 完成标准

**必检**：功能完整 | 无硬编码密钥 | 导出/类型变更已同步 | 新文件位置正确 | 语言惯用写法
**条件检**：DB迁移安全 | 认证 | API格式 | i18n | 三态/响应式/a11y | 测试 | 文档 | 死代码清理 — 仅在相关时检查

### 一致性：部分失败能独立→保留，不能→revert
### 并行（仅大型）：只并行编码，串行commit，Agent间无共同文件
### 错误：语法→修 | 依赖缺→装 | 环境变量缺→默认值 | 循环依赖→提取 | 类型不兼容→沿图更新 | 权限/网络→跳过 | 未知→换实现1次

## 阶段4: 验证（所有模式）

按技术栈跑秒级静态检查（tsc/eslint/py_compile/cargo check/go vet），超时10s跳过。
有错→修→`fix: 静态检查修复`。>20错→只修error。

## 阶段5: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer") → `git diff {起始hash}..HEAD`
审查：安全 + 铁律 + CLAUDE.md + 逻辑 + 性能 + 认证 + 类型 + a11y。有问题→修。

## 阶段6: 推送

**推送前检查**：`git diff --stat {起始hash}..HEAD` 确认改动范围符合预期，异常大则暂停审视。

DB项目→seed→`chore(data): 描述`。`git push origin {branch}`（失败→rebase→二次报错）。
大型→存记忆+更新progress。

## 阶段7: 收尾

**CI**（标准/大型）：`gh run list` 检查。通过→完成 | 失败→查日志修复push（最多2轮）| pending→汇报注明
**PR**（大型+feature分支）：`gh pr create --draft` 含变更摘要
**stash** → 提醒 pop

### 汇报

**快速**:
```
Auto-Dev | 项目名 | fix
改动: file1(+5-3) | 验证: ✅
成果: 一句话 ✅
```

**标准/大型**:
```
══════════════════════════════
Auto-Dev v11 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件（校验通过）
✅ 理解 — N模块
✅ 开发 — N功能 N commits (+X/-Y lines)
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

### 快速模式：修复 bug（含错误恢复）

> `/auto-dev 修复文章详情页点赞数不更新的问题`

```
Phase 0: 意图=fix | 模式=快速 | 精简检测(Next.js+pnpm)

阶段3-开发:
  假设: 点赞 API 返回后前端状态未更新
  Read src/app/article/[id]/page.tsx → 点赞调用 useLike hook
  Read src/hooks/useLike.ts → mutate 后未 revalidate ← 假设正确

  修复: mutate 成功后调 revalidate
  commit: fix(article): 点赞后刷新计数 — mutate未触发revalidate

阶段4-验证: tsc → 1 error: 'revalidate' is not exported from 'swr'
  → Read swr 文档用法 → 改用 mutate() 的 revalidate 选项
  → commit: fix(article): 使用正确的 SWR revalidate API

阶段4-验证(重跑): tsc ✅
阶段6-推送: origin/main ✅
```

### 标准模式：新功能

> `/auto-dev 添加文章收藏功能`

```
Phase 0: 意图=feat | 模式=标准 | 检测到 Next.js+Prisma+Zustand+next-auth+Tailwind(dark)
         幂等: Grep "favorite/collect/收藏" → 未实现

阶段1-规划: architect → 6任务
  校验: Glob确认 prisma/schema.prisma ✅ src/server/api/ ✅ src/stores/ ✅
阶段2-理解: explorer → Article模型 + API模式 + auth中间件

阶段3-开发:
  1. feat(db): 添加收藏表 — 支持用户收藏文章的多对多关系
  2. feat(api): 收藏/取消收藏接口 — 带auth, {code,data,message}格式
  3. feat(store): 收藏状态 — zustand create() 跟随项目模式
  4. feat(ui): FavoriteButton — loading态 + dark: + aria-label
  5. feat(page): 我的收藏 — 三态 + 响应式 + 分页
  6. test(favorite) + docs + CHANGELOG

推送前: git diff --stat → 8 files, +320/-12 ✓
阶段4-验证: tsc ✅ eslint ✅
阶段5-审查: reviewer → 0问题
阶段6-推送: origin/main
阶段7-CI: passed ✅
```

---

## 上下文预算

按需Read | 架构+审查用独立Agent | 任务>10→分批5个 | 只输出状态行 | 大型按切片 | 紧张→降级
