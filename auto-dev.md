# Auto-Dev v9

$ARGUMENTS

<!-- 快速参考：整个流程的压缩版，Claude 每个阶段可回看 -->
<!-- 铁律: 先读后写 | 先查后建 | 不破坏现有 | 跟随项目 | 不问不启 | 只推origin -->
<!-- 循环: 读→查→写→检→格→存 -->
<!-- 模式: 快速(直接干) | 标准(规划→理解→开发→验证→审查→推送→CI) | 大型(+分支+PR+切片) -->

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
| context 紧张 | 降级到快速模式 |
| CI 失败 | 最多修2轮 |
| **需求过大**(>10文件且模糊) | 先拆解输出范围清单，标注本次做哪些，其余记入建议 |
| **发现不可行** | 立即中止，汇报原因 + 已完成部分 + 建议替代方案 |

---

## Phase 0: 启动

### 输入解析

| 输入 | 识别 | 处理 |
|------|------|------|
| 文本 | 默认 | 直接用 |
| 文件路径 | `/` `~` `./` 开头 | Read |
| Issue URL | github.com/issues/ | `gh issue view` |
| 多需求 | `+` 分隔 | 逐个 |
| 空 | 无参数 | 审计模式 |

### 审计模式（空输入）

三层审计，输出列表让用户选择：
1. **健康**: tsc错误 / lint错误 / npm audit / 过期依赖
2. **完整性**: TODO/FIXME / 路由-页面匹配 / API-前端匹配
3. **质量**: 重复代码 / 超大文件(>500行) / 无类型导出

### 意图分类

| 意图 | 判断 | 差异 |
|------|------|------|
| **fix** | 修复/bug/报错 | 最小改动 + 回归测试 |
| **feat** | 添加/新增/实现 | 完整实现 + 测试 + 文档 + 三态 + 响应式 |
| **refactor** | 重构/整理 | 锁行为再改 + 清死代码 |
| **perf** | 性能/慢/卡 | 定位瓶颈，针对性优化 |
| **chore** | 升级/配置/迁移 | 最小风险 |

### 环境感知

**快速模式精简版**（<3秒）：
- 只检测：技术栈 + 包管理器 + CLAUDE.md
- 其他在需要时按需检测

**标准/大型完整版**：
```
技术栈 | 包管理器 | 项目类型 | Monorepo | 代码风格 | 数据库
测试框架 | i18n | Git Hooks | 状态管理 | 认证方案 | API响应格式
响应式方案(tailwind/media queries) | 暗色模式(dark:/prefers-color-scheme)
```

API 响应格式：读一个已有 handler 提取格式，新 API 严格遵循。

### 学习项目约定（标准/大型）

CLAUDE.md + 项目记忆 + 快速扫描：
- `git log --oneline -10` → commit 风格
- 2-3核心文件 → 命名/分层/模式
- 目录结构 → 文件放置规则
- auth/store → 认证和状态管理模式
- 无 src/ 且 git log 空 → 脚手架

### 幂等检查

Grep 需求关键词：完全实现→终止，部分→只做剩余

### 选择模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **脚手架** | 新项目 | 初始化 → 开发 → 验证 → 推送 |
| **快速** | <5文件/fix/chore | **直接开发** → 验证 → 推送 |
| **标准** | 5-20文件/feat/refactor | 规划 → 理解 → 开发 → 验证 → 审查 → 推送 → CI |
| **大型** | >20文件/跨模块 | 同标准 + feature分支 + 备份tag + 切片交付 + PR + 进度存档 |

### 安全网
- 标准/大型：记录起始 commit hash
- 大型：`git tag auto-dev-backup-$(date +%s)` + feature 分支
- 未提交改动 → stash
- 有未完成 auto-dev Task → 断点续传
- 大型进度存档：`.auto-dev-progress.json`（gitignore）

### 降级与中止

**降级**：连续5次工具失败→快速模式；任务>15→只做前5个；验证>20错误→只修error
**中止**：技术上不可行 / 需要用户提供信息(API key等) / 代码库状态异常 → 停止，汇报原因+建议

---

## 脚手架（仅新项目）

官方脚手架初始化 → 基础依赖 → CLAUDE.md + .env.example → `init: 项目初始化`

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect")：
- 输入：技术栈 + 需求 + CLAUDE.md + 目录结构 + 认证 + 状态管理 + API格式
- 输出：文件清单 + 依赖顺序 + 数据模型

**全栈分层**：
```
DB Schema → Model/Service → API(+认证) → 状态层 → UI组件 → 页面(+守卫)
```
每层单独 commit。

**大型切片交付**：按功能切片，每片含完整前后端，完成即 push。

TaskCreate。**不commit。**

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer")：执行路径 + 数据流 + 认证链路 + 类型依赖图。结果喂给开发。

## 阶段3: 开发

### 会话内学习

记住本次错误/模式/问题，后续避免。同类≥3→根因分析。

### 开发循环：读→查→写→检→格→存

**读**: Read 目标文件
**查**: Grep 可复用代码
**写**: 实现功能

约束清单：
- 文件放置跟随项目结构
- DB迁移只加法
- 环境变量 → 默认值 + .env.example
- i18n → locale 文件
- 新路由/API 默认加认证
- 新状态跟随项目方案
- 新 API 遵循响应格式
- 新依赖前检查替代
- 性能: N+1/索引/分页/虚拟滚动
- **前端三态**（feat+新页面）：Loading(skeleton) / Error(重试) / Empty(引导)
- **响应式**（feat+前端）：
  - 项目用 Tailwind → 用 responsive 前缀 (sm:/md:/lg:)
  - 项目用 CSS → 用 media queries
  - 移动端优先，逐步增强
- **暗色模式**（如果项目已有 dark mode 支持）：
  - Tailwind → 用 dark: 前缀
  - CSS → 用 prefers-color-scheme
  - 无暗色模式的项目不强加
- **a11y 基础**：aria-label / alt / label / 对比度
- 遵循语言惯用写法

**检**: 改导出→消费者 | 改schema→查询代码 | 改type→依赖图更新 | refactor→删死代码
**格**: husky→跳过 | 否则 prettier/biome/rustfmt/ruff | 无配置→跳过
**存**: `type(scope): 描述`

### 文档同步（feat）

改了用户可见行为 → 更新 README/API docs → `docs: 描述`
项目有 CHANGELOG.md → 追加本次变更 → 同一 commit

### 测试（写但不跑）

fix→回归 | feat→单测 | refactor→先锁行为 | perf/chore→不写
无框架→不写 | 有→跟随模式

### 完成标准

**必检**（每个功能都检）：
- [ ] 功能完整
- [ ] 无硬编码密钥
- [ ] 导出/类型变更已同步消费者
- [ ] 新文件在正确目录
- [ ] 语言惯用写法

**条件检**（仅相关时检）：
- [ ] DB迁移只加法 ← 有schema变更时
- [ ] 新路由有认证 ← 有新路由时
- [ ] 新API遵循响应格式 ← 有新API时
- [ ] i18n 无硬编码 ← 项目有i18n时
- [ ] 前端三态 ← feat+新页面时
- [ ] 响应式适配 ← feat+前端时
- [ ] 暗色模式 ← 项目有dark mode时
- [ ] a11y基础 ← 有新UI时
- [ ] 测试 ← 按意图策略
- [ ] 文档/CHANGELOG ← feat时
- [ ] 无死代码 ← refactor时

### 一致性保障

部分失败：能独立工作→保留，不能→revert。

### 并行（仅大型）

只并行编码，串行 commit。Agent 间无共同文件。

### 错误处理

| 类型 | 策略 |
|------|------|
| 语法/类型错误 | 直接修复 |
| 依赖缺失 | 安装继续 |
| 环境变量缺失 | .env.example + 默认值 |
| 循环依赖 | 提取独立模块 |
| 导出/类型不兼容 | 沿依赖图更新 |
| 权限/网络 | 记录跳过 |
| 同类≥3 | 根因分析 |
| 未知 | 换实现1次，失败跳过 |

## 阶段4: 验证（所有模式）

秒级静态检查（检测到才跑，超时10s跳过）：
TS→`tsc --noEmit` | ESLint→`eslint {files} --quiet` | Python→`py_compile` | Rust→`cargo check` | Go→`go vet`
有错→修→`fix: 静态检查修复`。>20错→只修error。

## 阶段5: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer")：
`git diff {起始hash}..HEAD` → 安全+铁律+CLAUDE.md+逻辑+性能+认证+类型+a11y
有问题→修→`fix: 描述`

## 阶段6: 推送

1. DB项目 → seed → `chore(data): 描述`
2. `git push origin {branch}`（失败→rebase→二次报错）
3. 大型 → 存记忆 + 更新 progress.json

## 阶段7: 收尾

### CI 反馈（标准/大型）
```bash
sleep 5 && gh run list --limit 1 --json status,conclusion 2>/dev/null
```
通过/无CI→完成 | 失败→查日志→修→push（最多2轮）| pending→汇报注明

### PR 创建（大型模式，feature分支时）
```bash
gh pr create --title "feat: 需求概要" --body "$(cat <<'EOF'
## 变更内容
- 功能1
- 功能2

## Auto-Dev v9 自动生成
EOF
)" --draft
```
> 创建 draft PR，用户 review 后自行合并

### 有 stash → 提醒 pop

### 汇报

**快速**:
```
Auto-Dev | 项目名 | fix
改动: file1, file2 | 验证: tsc ✅ eslint ✅
成果: 一句话 ✅
```

**标准/大型**:
```
══════════════════════════════
Auto-Dev v9 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件
✅ 理解 — N模块
✅ 开发 — N功能 N commits
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

## 示例：一次标准模式 feat 的完整流程

> 输入: `/auto-dev 添加文章收藏功能`

```
Phase 0: 检测到 Next.js + Prisma + Zustand + next-auth + Tailwind(dark mode)
         意图=feat | 模式=标准 | 预计8文件
         幂等检查: Grep "favorite/collect/收藏" → 未实现

阶段1-规划: code-architect → 6个任务
  1. prisma/schema: 新增 Favorite 表
  2. src/server/api: 收藏/取消收藏 API
  3. src/stores: 收藏状态 store
  4. src/components: FavoriteButton 组件
  5. src/app/favorites: 我的收藏页
  6. 测试 + 文档

阶段2-理解: code-explorer → 分析 Article 模型、现有 API 模式、auth 中间件

阶段3-开发:
  任务1: schema添加Favorite表 → commit: feat(db): 添加收藏表
  任务2: API(带auth middleware) → 遵循 {code,data,message} 格式
         commit: feat(api): 收藏/取消收藏接口
  任务3: zustand store → 用 create() 跟随项目模式
         commit: feat(store): 收藏状态管理
  任务4: FavoriteButton → 带 Loading态 + dark: 前缀 + aria-label
         commit: feat(ui): 收藏按钮组件
  任务5: 收藏页 → 三态(Loading/Error/Empty) + 响应式 + 分页
         commit: feat(page): 我的收藏页面
  任务6: 测试 + README更新 + CHANGELOG
         commit: test(favorite): 收藏逻辑测试
         commit: docs: 更新文档

阶段4-验证: tsc --noEmit ✅ | eslint ✅
阶段5-审查: code-reviewer → 0问题
阶段6-推送: git push origin main
阶段7-CI: gh run → passed ✅
```

---

## 上下文预算

- 按需 Read，不预读
- 架构 + 审查用独立 Agent
- 任务>10 → 分批5个
- 阶段间只输出状态行
- 大型按切片交付
- 紧张时自动降级
