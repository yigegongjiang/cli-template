---
description: "Create or update workflow.md for AI-only (Vibe Coding) project — debug and release lifecycle. Adapts to the project's actual toolchain. Use when initializing, resetting, or modifying project workflow."
allowed-tools: Bash Read Write Edit
---

# 生成 / 更新 workflow.md

读取目标工程上下文，生成或更新适配其工具链的 workflow.md。

## 前置门禁（MUST）

验证 AGENTS.md 已作为 system prompt 在当前会话生效。

判定方式：检查你的 project instructions（会话启动时从 CLAUDE.md 自动加载）中是否包含 AGENTS.md 的核心约束——特征关键词：「当前 project 是 AI Only 工程」+「文档编写规范」+「工作模式」。

- **已生效**（project instructions 中有上述内容）→ 继续执行
- **未生效** → **停止执行**，输出：「AGENTS.md 未作为 system prompt 生效。请先执行 `/vibe-setup:agents`，然后**重启 Claude Code 会话**后再运行本 skill。」

> 磁盘上存在 AGENTS.md ≠ 已加载。CLAUDE.md 仅在会话启动时加载；本会话内创建/修改的 AGENTS.md 必须重启后才生效。

## 用户上下文

> $ARGUMENTS

用户调用时附带的补充描述（如「无调试」「无预部署」「CLI 项目」等）。探测阶段与工程文件并用，冲突时用户描述优先。无内容则忽略本段。

## 执行步骤

1. **探测工程上下文** — 读取包管理文件、CI 配置（`.github/workflows/`）、部署配置（wrangler / vercel / Dockerfile / ...），识别：
   - 类型检查命令（tsc / mypy / cargo check / ...）
   - 构建命令
   - 测试命令
   - 本机交付命令（install script / `cargo install --path .` / `wrangler deploy` / `npm publish` / ...）= 预部署依据
   - 版本号存储位置（package.json#version / Cargo.toml / pyproject.toml / ...）
   - 已登录的外部服务（`wrangler whoami` / `gh auth status` / `npm whoami` / ...）
   - 读取已有 workflow.md（若存在）
   - 检测独立部署文档（`deploy.md` / `DEPLOY.md` / `deployment.md` / `release.md` / `RELEASE.md`）
2. **确定章节** — 根据工程探测 + 用户参数决定各段落保留或删除；发布段必定保留（步骤可精简）；保留的段落填充具体命令
3. **生成或更新 workflow.md** — 不存在 → 按骨架新建，用探测事实替换命令占位；已存在但不符合骨架（缺 When Editing 块 / 结构不匹配 / 含违规内容如账号细节）→ 按骨架重新生成，用探测事实填充；已存在且符合 → 按用户参数施加变更，无参数则跳过

## workflow.md 骨架

````markdown
```When Editing
本文档作用: 工程工作流程 (可用工具 / 调试 / 发布); MUST NOT 写工程说明 (→ README.md) / LLM 约束 (→ AGENTS.md)
遵循 AGENTS.md 文档编写规范
- 所有段落均为条件段, 根据工程实际决定保留或删除; 存在即为明确流程, MUST NOT 附加强度标记
- 发布内按顺序编号步骤; 顶部 TL;DR ≤ 5 行; 删除子段后重编号保持连续
- 风险点 / 不可逆操作用 `>` 引用块; 高危操作 MUST 标禁用条件
```

# 可用工具

_仅列出已登录的 CLI 工具，MUST NOT 写账号细节._

# 调试

_启动命令 + 验证方式._

example: `bun run dev`，验证 → `curl` 或 Chrome DevTools MCP 检查页面(以实际 dev url 为准，如 `http://localhost:5173`)

# 发布

代码变更完成后立即执行（= 需求交付的最后环节）。交付 = 预部署 + push。

## TL;DR

依序执行(example)：
_≤ 5 行, 按顺序写完整个发布概览:_

1. 验证：`bun run typecheck ...`
2. 写版本：`package.json` + `CHANGELOG.md` + `CHANGELOG.dev.md` 同步编辑 (与 tag 一致)
3. 预部署：`bun run deploy ...`
4. 发布：commit + annotated tag (`-a -m`) + push branch + tag

## 1. 验证

`_验证命令_`

example: `bun run typecheck ...`

## 2. 写版本

- 版本号: 默认递增 PATCH (第三位); 超大功能更新/调整 → MINOR; 禁止 → MAJOR（除非人类主动要求）.
- `_version-files(所有强关联版本号的 files)_`同步编辑 (与 tag 一致)

## 3. 预部署

本机完成实际交付 (install / deploy / publish)。

`_预部署命令_`

example: `bun run deploy ...`

## 4. 发布

_发布命令序列：git add / commit / tag / push._

_如不允许 push，也一定说明不允许 `git push`._

```bash example
git commit -m "release: vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin branch
git push origin vX.Y.Z
```
````

## 适配规则

- 可用工具段：探测已登录的外部服务（`wrangler whoami` / `gh auth status` / `npm whoami` / ...）→ 仅列出已登录的工具名 + "已登录"；未登录的工具不列出；无任何已登录服务 → 删除整段；MUST NOT 写入账号名 / 邮箱 / org / plan 等任何账号细节
- 调试段：有 dev server / 可运行入口 → 保留；纯 library 且无可调试内容 → 删除整段；用户明确不要 → 删除整段
  - 调试内容 = 启动命令 + 验证方式
  - 验证方式按项目类型：Web 服务 → `curl http://localhost:<port>` / Chrome DevTools MCP 检查页面；API → `curl` 端点示例；CLI → 运行示例命令 + test 命令
- 发布段：必定保留；根据工程实际裁剪子步骤（验证 / 写版本 / 预部署 / 发布），至少保留「写版本」+「发布」步骤
- 验证命令 → 从 scripts / Makefile / CI 中提取实际 typecheck + build + test 命令
- 预部署 → 有本机交付命令（CLI install / `wrangler deploy` / `npm publish` / docker / ...）→ MUST 保留；产物仅 CI 能产出（多平台二进制 / release page）→ 删除整段 + TL;DR 对应行
- 写版本 MUST 先于预部署（交付产物需携带新版本号）
- CI(GHA) → MUST NOT 写入等待 / 轮询 / 验证 CI 结果的步骤（`gh run watch` / `gh run list` / ...）；本条不影响「验证」段的本机 typecheck / build / test
- 删除段落后重编号保持连续
- 版本文件 → 识别所有含版本号的文件（package.json / CHANGELOG.md / CHANGELOG.dev.md / Cargo.toml / ...）
- 发布方式 → commit + annotated tag + push（branch + tag）；本机 `cargo publish` / `npm publish` 归入预部署段
- git 命令 → 使用当前 branch 名，MUST NOT 硬编码 `main`
- 用户参数含明确修改指令 → 按指令更新已有内容，不跳过
- 工程存在独立部署文档（`deploy.md` / `DEPLOY.md` / `deployment.md` / `release.md` / `RELEASE.md`）→ 合并其内容入 workflow.md 发布段，删除原文件；workflow.md 是唯一部署流程文档。**即使主文件跳过生成，本条仍 MUST 执行**
- 工程存在双语结构（`workflow.zh.md` 与 `workflow.md` 并存）→ 合并 `.zh.md` 独有事实入 workflow.md，删除 `workflow.zh.md`。**即使主文件跳过生成，本条仍 MUST 执行**
