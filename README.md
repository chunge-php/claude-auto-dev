# Auto-Dev v7

Claude Code 全自动开发流水线技能 -- 输入需求，自动完成开发并推送到 GitHub。

## 这是什么

Auto-Dev 是一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 自定义 Slash Command（技能），它将 Claude 变成一个完整的 AI 开发团队：

- **自动分析项目** -- 识别技术栈、包管理器、代码风格、数据库、测试框架、i18n、认证方案、状态管理
- **自动规划架构** -- 调用专业 Agent 做架构设计和代码理解
- **自动开发** -- 按全栈分层顺序实现，带语言特定的质量检查
- **自动验证** -- 秒级静态分析（tsc/eslint/cargo check），不启动服务
- **自动审查** -- 安全 + 性能 + 认证 + 类型安全全面检查
- **自动推送** -- commit + push 到 GitHub

## 核心特性

| 特性 | 说明 |
|------|------|
| 自适应复杂度 | 脚手架/快速/标准/大型四档，新项目自动初始化 |
| 意图分类 | fix/feat/refactor/perf/chore 五种意图，各有策略 |
| 决策框架 | 模糊需求、多方案、范围膨胀等场景有明确思考规则 |
| 审计模式 | 空输入三层审计：健康检查 → 功能完整性 → 代码质量 |
| 全栈分层 | Schema → Model → API → State → UI → 页面 |
| 渐进交付 | 大型模式按功能切片 push，用户实时看进展 |
| 轻量验证 | tsc --noEmit / eslint / cargo check / go vet，秒级完成 |
| 会话内学习 | 同类错误 ≥3次自动停止分析根因，不逐个修补 |
| 认证感知 | 新路由/页面自动添加 auth middleware/guard |
| 状态管理感知 | 检测 zustand/redux/pinia，新状态跟随已有模式 |
| 语言特定检查 | TS无any / Python有type hints / Go错误必检查 / Rust不滥用unwrap |
| 跨文件类型传播 | 改了 interface/type 自动找到所有引用并同步更新 |
| 依赖卫生 | 新增包前检查替代、优先标准库 |
| DB迁移安全 | 只做加法，不 drop/rename |
| 死代码清理 | refactor 后自动检测并删除无人引用的导出 |
| 文档同步 | feat 改变行为时自动更新 README/API docs |
| i18n 感知 | 检测到国际化配置后文案写入 locale 文件 |
| Git Hooks 感知 | 有 husky/lint-staged 跳过手动格式化 |
| 一致性保障 | 关联任务部分失败时自动回滚不完整的部分 |
| 幂等安全 | 开始前检测需求是否已实现 |
| 上下文管理 | 按需读取 + Agent 隔离 + 分批执行 |

## 安装

### 一键安装（全局，推荐）

```bash
mkdir -p ~/.claude/commands && curl -o ~/.claude/commands/auto-dev.md \
  https://raw.githubusercontent.com/chunge-php/claude-auto-dev/main/auto-dev.md
```

### 仅当前项目

```bash
mkdir -p .claude/commands && curl -o .claude/commands/auto-dev.md \
  https://raw.githubusercontent.com/chunge-php/claude-auto-dev/main/auto-dev.md
```

## 使用

```bash
# 文本需求
/auto-dev 添加用户登录功能，支持手机号+验证码登录

# 修复 bug
/auto-dev 修复首页加载慢的问题

# 多个需求
/auto-dev 添加搜索功能 + 优化列表性能

# 从文件读取
/auto-dev ./requirements.md

# 从 GitHub Issue
/auto-dev https://github.com/user/repo/issues/42

# 审计模式 -- 三层健康检查
/auto-dev
```

### 工作模式

| 模式 | 自动触发条件 | 流程 |
|------|-------------|------|
| 脚手架 | 新项目 | 初始化 -> 开发 -> 验证 -> 推送 |
| 快速 | < 5 文件 / bug 修复 | 开发 -> 验证 -> 推送 |
| 标准 | 新功能 / 重构 | 规划 -> 理解 -> 开发 -> 验证 -> 审查 -> 推送 |
| 大型 | 跨模块 / 大型重构 | 同标准 + feature 分支 + 渐进交付 |

### 意图自动识别

| 你说 | 识别为 | 行为 |
|------|--------|------|
| "修复XXX报错" | fix | 最小改动 + 回归测试 |
| "添加XXX功能" | feat | 完整实现 + 单测 + 文档同步 |
| "重构XXX模块" | refactor | 先锁行为再改 + 清理死代码 |
| "XXX太慢了" | perf | 定位瓶颈，针对性优化 |
| "升级XXX版本" | chore | 最小风险，逐步执行 |

### 审计模式

空输入触发三层审计：

| 层级 | 检查内容 |
|------|----------|
| 健康检查 | TypeScript 错误、ESLint 错误、安全漏洞、过期依赖 |
| 功能完整性 | TODO/FIXME、路由-页面匹配、API-前端匹配 |
| 代码质量 | 重复代码、超大文件、无类型导出 |

## 支持的技术栈

- **前端**: React / Vue / Svelte / Next.js / Nuxt
- **后端**: Node.js / Java / Python / Go / Rust
- **数据库**: Prisma / Supabase / Drizzle / SQL
- **包管理器**: pnpm / yarn / npm / bun
- **代码风格**: ESLint / Prettier / Biome / Rustfmt / Ruff
- **测试框架**: Vitest / Jest / Pytest / Go test
- **国际化**: next-intl / i18next / vue-i18n
- **状态管理**: Zustand / Redux / Pinia / Jotai / MobX
- **认证**: NextAuth / Lucia / Supabase Auth / 自定义 middleware
- **Git Hooks**: Husky / lint-staged
- **项目结构**: 单体 / Monorepo

## 自定义

### 配合 CLAUDE.md

在项目根目录创建 `CLAUDE.md`，Auto-Dev 会**严格遵守**：

```markdown
# 项目约定
- 使用 pnpm
- 组件使用 shadcn/ui
- 不要用 any 类型
- API 返回格式: { code, data, message }
- 数据库查询用 Drizzle ORM
- 路由认证用 middleware
```

### 铁律（内置，不可关闭）

1. 先读后写 -- 改文件前必须先读它
2. 先查后建 -- 建新代码前先搜索已有实现
3. 不破坏现有 -- 改接口前检查所有消费者
4. 跟随项目 -- CLAUDE.md 约定最高优先
5. 不问不启 -- 全程自动，不启动服务
6. 只推 origin -- 语义化 commit

## 版本演进

| 版本 | 核心变化 |
|------|----------|
| v1 | 7部门串行流水线 |
| v2 | 技术栈检测 + 断点续传 |
| v3 | 自适应模式 + CLAUDE.md + 格式化 |
| v4 | 铁律体系 + Agent调度 + 影响面分析 |
| v5 | 意图分类 + 测试策略 + 幂等检查 + 性能意识 |
| v6 | 决策框架 + 全栈分层 + 脚手架 + 依赖卫生 + i18n/hooks感知 + 一致性保障 |
| v7 | 轻量验证 + 审计模式 + 渐进交付 + 会话内学习 + 认证/状态感知 + 语言特定检查 + 类型传播 |

## License

MIT
