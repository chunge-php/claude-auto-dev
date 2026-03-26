# Auto-Dev v8

$ARGUMENTS

## 铁律（违反任何一条都是失败）

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
| 多种方案 | 选最简（少文件、少依赖） |
| 不确定安全性 | 不改，跳过汇报 |
| 范围膨胀 | 只做明确要求的 |
| 约定 vs 最佳实践 | 跟随约定 |
| 同类错误 ≥3 | 停，分析根因 |
| context 紧张 | 降级到快速模式，砍掉可选步骤 |
| CI 失败 | 最多修2轮，仍失败汇报 |

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
| **feat** | 添加/新增/实现 | 完整实现 + 测试 + 文档 + 三态UI |
| **refactor** | 重构/整理 | 锁行为再改 + 清死代码 |
| **perf** | 性能/慢/卡 | 定位瓶颈，针对性优化 |
| **chore** | 升级/配置/迁移 | 最小风险 |

### 环境感知（并行 Glob/Read）

一次性检测所有：
```
技术栈 | 包管理器 | 项目类型 | Monorepo | 代码风格 | 数据库
测试框架 | i18n | Git Hooks | 状态管理 | 认证方案 | API响应格式
```

**API 响应格式检测**（新增）：
- 读一个已有的 API handler/controller，提取返回格式（如 `{code, data, message}` 或 `{success, data, error}`）
- 后续新 API 严格遵循此格式

### 学习项目约定

CLAUDE.md + 项目记忆 + 快速扫描：
- `git log --oneline -10` → commit 风格
- 2-3核心文件 → 命名/分层/模式
- 目录结构 → 文件放置规则
- auth middleware/guard → 认证模式
- store 文件 → 状态管理模式
- 无 src/ 且 git log 空 → 脚手架模式

### 幂等检查

Grep 需求关键词：完全实现→终止，部分→只做剩余

### 选择模式

| 模式 | 条件 | 流程 |
|------|------|------|
| **脚手架** | 新项目 | 初始化 → 开发 → 验证 → 推送 |
| **快速** | <5文件/fix/chore | 开发 → 验证 → 推送 |
| **标准** | 5-20文件/feat/refactor | 规划 → 理解 → 开发 → 验证 → 审查 → 推送 → CI |
| **大型** | >20文件/跨模块 | 同标准 + feature分支 + 备份tag + 切片交付 + 进度存档 |

### 安全网
- 标准/大型：记录起始 commit hash
- 大型：`git tag auto-dev-backup-$(date +%s)` + feature 分支
- 未提交改动 → stash
- 有未完成 auto-dev Task → 断点续传
- **大型模式进度存档**：每完成一个切片，将进度写入 `.auto-dev-progress.json`（gitignore），conversation 中断后可恢复

### 降级机制（新增）

当出现以下信号时，自动从当前模式降级：
- 连续5个工具调用失败 → 降级到快速模式
- 感知到 context 紧张（任务列表 >15 个未完成）→ 剩余任务分批，当前批只做最高优先级5个
- 验证阶段发现 >20 个错误 → 只修 severity:error，忽略 warning

---

## 脚手架（仅新项目）

1. 技术栈选择（优先用户指定）→ 官方脚手架初始化
2. 基础依赖 + CLAUDE.md + .env.example
3. commit: `init: 项目初始化` → 进入开发

---

## 阶段1: 规划（标准/大型）

Agent(subagent_type="feature-dev:code-architect")：
- 输入：技术栈 + 需求 + CLAUDE.md + 目录结构 + 认证 + 状态管理 + API格式
- 输出：文件清单 + 依赖顺序 + 数据模型

### 全栈分层

```
DB Schema → Model/Service → API(+认证) → 状态层(store/hooks) → UI组件 → 页面(+路由守卫)
```
每层单独 commit。

### 大型：切片交付

按功能切片，每片含完整前后端，完成即 push。

TaskCreate 建任务。**不commit。**

## 阶段2: 理解（标准/大型，已有项目）

Agent(subagent_type="feature-dev:code-explorer")：
- 执行路径、数据流、import 关系
- 认证链路 + 类型依赖图
- 结果喂给开发。不commit。

## 阶段3: 开发

### 会话内学习

记住本次遇到的错误/模式/问题类型，后续自动避免。同类错误≥3次→停下分析根因。

### 开发循环

```
读 → 查 → 写 → 检 → 格 → 存
```

**读**: Read 目标文件（大文件 Grep 定位后读局部）
**查**: Grep 可复用代码
**写**: 实现功能，遵循以下约束：

- 文件放置跟随项目结构
- DB迁移只加法（不drop/rename）
- 环境变量 → 默认值 + .env.example
- i18n 项目 → locale 文件，不硬编码
- 新路由/API 默认加认证（公开接口除外）
- 新状态跟随项目状态管理方案
- 新 API 严格遵循检测到的响应格式
- 新依赖前检查已有替代，优先标准库
- **性能**: 避免 N+1；新表加索引；列表必分页；大列表虚拟滚动
- **前端三态**（feat 意图新增页面时必须）：
  - **Loading 态**: 数据加载中显示 skeleton/spinner
  - **Error 态**: 请求失败显示错误提示 + 重试按钮
  - **Empty 态**: 数据为空显示引导信息
  - 跟随项目已有的三态实现方式（如有）
- **前端可访问性**（a11y 基础）：
  - 可交互元素有 aria-label 或可见文本
  - 图片有 alt
  - 表单有 label
  - 颜色对比度足够（不用纯灰色文字）
- 遵循语言惯用写法（Claude 已知，不列明细）

**检**:
- 改导出 → Grep 消费者兼容性
- 改 schema → 检查查询代码
- 改 type/interface → 沿依赖图同步更新
- refactor → 删除死代码

**格**:
- 有 husky/lint-staged → 跳过
- 否则 prettier/biome/rustfmt/ruff
- 无配置 → 跳过

**存**: `type(scope): 描述`

### 文档同步（feat 意图）

改了用户可见行为 → 更新 README/API docs → `docs: 描述`

### 测试（写但不跑）

| 意图 | 要求 |
|------|------|
| fix | 回归测试 |
| feat | 核心逻辑单测 |
| refactor | 先锁行为再改 |
| perf/chore | 不写 |

无测试框架→不写。有→跟随已有模式。

### 完成标准

- [ ] 功能完整
- [ ] 无硬编码密钥
- [ ] 导出/类型变更已同步
- [ ] 新文件在正确目录
- [ ] DB迁移只加法
- [ ] 新路由有认证
- [ ] 新状态跟随方案
- [ ] 新API遵循响应格式
- [ ] i18n 无硬编码
- [ ] 前端有三态（feat+新页面）
- [ ] 前端基础 a11y
- [ ] 测试已写
- [ ] 文档已同步（feat）
- [ ] 无死代码（refactor）

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

| 栈 | 命令 |
|----|------|
| TS | `npx tsc --noEmit 2>&1 \| tail -30` |
| ESLint | `npx eslint {changed_files} --quiet 2>&1 \| tail -20` |
| Python | `python -m py_compile {file}` |
| Rust | `cargo check 2>&1 \| tail -30` |
| Go | `go vet ./... 2>&1 \| tail -20` |

有错→修→`fix: 静态检查修复`。>20个错误→只修 error 级别。

## 阶段5: 审查（标准/大型）

Agent(subagent_type="feature-dev:code-reviewer")：
- `git diff {起始hash}..HEAD`
- 安全 + 铁律 + CLAUDE.md + 逻辑 + 性能 + 认证 + 类型 + a11y
- 有问题→修→`fix: 描述`

## 阶段6: 推送

1. DB项目 → seed 覆盖新功能 → `chore(data): 描述`
2. `git push origin {branch}`（失败→rebase→二次失败报错）
3. 大型模式 → 技术决策存项目记忆 + 更新 .auto-dev-progress.json
4. 有 stash → 提醒 pop

## 阶段7: CI 反馈（标准/大型，新增）

push 后检查 CI 状态：
```bash
sleep 5 && gh run list --limit 1 --json status,conclusion 2>/dev/null
```

| CI 状态 | 处理 |
|---------|------|
| 通过 / 无 CI | 完成 |
| 失败 | `gh run view --log-failed` 查看原因 → 修复 → push → 再检查（最多2轮） |
| pending | 汇报中注明"CI运行中" |

## 汇报

**快速**:
```
Auto-Dev | 项目名 | fix
改动: file1, file2 | 验证: tsc ✅ eslint ✅
成果: 一句话 ✅
```

**标准/大型**:
```
══════════════════════════════
Auto-Dev v8 | 项目名 | feat/标准
──────────────────────────────
✅ 规划 — N任务 N文件
✅ 理解 — N模块
✅ 开发 — N功能 N commits
✅ 验证 — tsc ✅ eslint ✅
✅ 审查 — N问题已修 / 无问题
✅ 推送 — branch → origin
✅ CI — passed / pending / fixed(N轮)
──────────────────────────────
成果: A / B / C
回滚: X(原因)
遗留: Y(原因)
建议: Z
══════════════════════════════
```

---

## 上下文预算

- 按需 Read，不预读
- 架构 + 审查用独立 Agent
- 任务>10 → 分批5个
- 阶段间只输出状态行
- 大型按切片交付
- 紧张时自动降级
