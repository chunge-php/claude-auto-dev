# Auto-Dev v6

Claude Code 全自动开发流水线技能 -- 输入需求，自动完成开发并推送到 GitHub。

## 这是什么

Auto-Dev 是一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 自定义 Slash Command（技能），它将 Claude 变成一个完整的 AI 开发团队：

- **自动分析项目** -- 识别技术栈、包管理器、代码风格、数据库、测试框架、i18n、git hooks
- **自动规划架构** -- 调用专业 Agent 做架构设计和代码理解
- **自动开发** -- 按全栈分层顺序逐个实现（DB → 后端 → 前端），遵循项目约定
- **自动审查** -- 安全 + 性能 + 死代码 + i18n + CLAUDE.md 合规检查
- **自动推送** -- commit + push 到 GitHub

## 核心特性

| 特性 | 说明 |
|------|------|
| 自适应复杂度 | 脚手架/快速/标准/大型四档模式，新项目自动初始化，小修3步搞定 |
| 意图分类 | 自动识别 fix/feat/refactor/perf/chore，不同意图不同开发策略 |
| 决策框架 | 面对模糊需求、多方案选择、范围膨胀时有明确的思考规则 |
| 先读后写 | 修改前必须理解上下文，搜索可复用代码，检查影响面 |
| 全栈分层 | Schema → Model → API → Hooks → UI → 页面，严格按依赖顺序 |
| 幂等安全 | 开始前检测需求是否已实现，避免重复劳动 |
| 智能测试 | fix 写回归测试，feat 写单测，refactor 先锁行为 -- 跟随项目已有风格 |
| 依赖卫生 | 新增包前检查是否有替代、是否已有类似依赖、优先用标准库 |
| DB 迁移安全 | 只做加法（add column/table），不 drop/rename |
| 死代码清理 | refactor 后自动检测并删除无人引用的导出 |
| 文档同步 | feat 改变用户可见行为时自动更新 README/API docs |
| i18n 感知 | 检测到国际化配置后，文案写入 locale 文件，不硬编码 |
| Git Hooks 感知 | 检测到 husky/lint-staged 后跳过手动格式化，避免重复 |
| 一致性保障 | 关联任务部分失败时，不能独立工作的部分自动回滚 |
| 多输入格式 | 纯文本 / 文件路径 / GitHub Issue URL / 空参数（自动审计） |
| 性能意识 | 避免 N+1、加索引、分页、虚拟滚动、lazy load |
| 专业 Agent 调度 | code-architect 规划 + code-explorer 理解 + code-reviewer 审查 |
| 上下文管理 | 按需读取 + Agent 隔离 + 分批执行，大项目不怕撑爆 |

## 安装

### 方法一：一键安装（全局，推荐）

```bash
mkdir -p ~/.claude/commands && curl -o ~/.claude/commands/auto-dev.md \
  https://raw.githubusercontent.com/chunge-php/claude-auto-dev/main/auto-dev.md
```

### 方法二：仅当前项目

```bash
mkdir -p .claude/commands
curl -o .claude/commands/auto-dev.md \
  https://raw.githubusercontent.com/chunge-php/claude-auto-dev/main/auto-dev.md
```

### 方法三：手动

下载 `auto-dev.md` 放入 `~/.claude/commands/`（全局）或 `.claude/commands/`（项目）。

## 使用

在 Claude Code 中输入 `/auto-dev` 加上你的需求：

```
/auto-dev 添加用户登录功能，支持手机号+验证码登录
```

### 输入格式

```bash
# 文本需求
/auto-dev 修复首页加载慢的问题

# 多个需求（用 + 分隔）
/auto-dev 添加搜索功能 + 优化列表性能

# 从文件读取需求
/auto-dev ./requirements.md

# 从 GitHub Issue 读取
/auto-dev https://github.com/user/repo/issues/42

# 空参数 -- 自动审计项目，列出待完善项
/auto-dev
```

### 工作模式

| 模式 | 自动触发条件 | 流程 |
|------|-------------|------|
| 脚手架 | 新项目（无代码） | 初始化 -> 开发 -> 推送 |
| 快速 | 改动 < 5 文件 / bug 修复 | 直接开发 -> 推送 |
| 标准 | 新功能 / 模块重构 | 规划 -> 理解 -> 开发 -> 审查 -> 推送 |
| 大型 | 跨模块 / 大型重构 | 同标准 + feature 分支 + 备份 tag + 分批 |

### 意图自动识别

你不需要指定意图，Auto-Dev 会根据描述自动判断：

| 你说 | 识别为 | 行为 |
|------|--------|------|
| "修复XXX报错" | fix | 最小改动 + 回归测试 |
| "添加XXX功能" | feat | 完整实现 + 单测 + 文档同步 |
| "重构XXX模块" | refactor | 先锁行为再改 + 清理死代码 |
| "XXX太慢了" | perf | 定位瓶颈，针对性优化 |
| "升级XXX版本" | chore | 最小风险，逐步执行 |

### 决策框架

面对选择时，Auto-Dev 的决策逻辑：

| 场景 | 决策 |
|------|------|
| 需求模糊 | 选最常见理解，commit 中记录假设 |
| 多种方案 | 选最简单的（少文件、少依赖、少抽象） |
| 不确定是否安全 | 不改，跳过并汇报 |
| 功能范围膨胀 | 只做明确要求的，延伸记入"后续建议" |
| 项目约定 vs 最佳实践 | 跟随项目约定 |

## 支持的技术栈

Auto-Dev 会自动检测并适配：

- **前端**: React / Vue / Svelte / Next.js / Nuxt 等
- **后端**: Node.js / Java / Python / Go / Rust
- **数据库**: Prisma / Supabase / Drizzle / 通用 SQL
- **包管理器**: pnpm / yarn / npm / bun
- **代码风格**: ESLint / Prettier / Biome / Rustfmt / Ruff
- **测试框架**: Vitest / Jest / Pytest / Go test
- **国际化**: next-intl / i18next / vue-i18n
- **Git Hooks**: Husky / lint-staged
- **项目结构**: 单体 / Monorepo

## 自定义

### 铁律（内置，不可关闭）

1. 先读后写 -- 改文件前必须先读它
2. 先查后建 -- 建新代码前先搜索已有实现
3. 不破坏现有 -- 改接口前检查所有消费者
4. 跟随项目 -- CLAUDE.md 约定优先级最高
5. 不问不启 -- 全程自动，不确认不启动服务
6. 只推 origin -- 语义化 commit 信息

### 配合 CLAUDE.md 使用

在项目根目录创建 `CLAUDE.md` 写入项目约定，Auto-Dev 会**严格遵守**：

```markdown
# 项目约定
- 使用 pnpm，不要用 npm
- 组件使用 shadcn/ui
- 不要用 any 类型
- API 返回格式统一用 { code, data, message }
- 数据库查询用 Drizzle ORM
```

## 版本演进

| 版本 | 核心变化 |
|------|----------|
| v1 | 7部门串行流水线 |
| v2 | 技术栈检测 + 断点续传 + 统一push |
| v3 | 3档自适应模式 + CLAUDE.md集成 + 依赖安装 + 格式化 |
| v4 | 铁律体系 + 专业Agent调度 + 影响面分析 + 上下文管理 |
| v5 | 意图分类 + 测试策略 + 幂等检查 + 性能意识 + 完成标准 |
| v6 | 决策框架 + 全栈分层 + 脚手架 + 依赖卫生 + DB迁移安全 + 死代码清理 + 文档同步 + i18n/hooks感知 + 一致性保障 |

## License

MIT
