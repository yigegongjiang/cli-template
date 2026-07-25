---
description: "Create or update AGENTS.md + CLAUDE.md symlink for AI-only (Vibe Coding) project. Use when initializing, resetting, or modifying a project's AI autonomy constraints."
allowed-tools: Bash Read Write Edit
---

# 生成 / 更新 AGENTS.md

读取目标工程，生成或更新 AGENTS.md 并确保 CLAUDE.md → AGENTS.md 的 symlink 存在。

## 用户上下文

> $ARGUMENTS

用户调用时附带的补充描述（如「添加路由指向 api.md」「工作模式增加数据库约束」等）。新建时作为补充上下文，更新时作为修改指令。无内容则忽略本段。

## 执行步骤

1. **探测工程上下文** — 读取工程根目录文件列表，确认是否已有 AGENTS.md / CLAUDE.md
2. **生成或更新 AGENTS.md** — 不存在 → 按骨架新建；已存在但不符合骨架 → 对齐骨架；用户参数含修改指令 → 按指令更新；已符合且无指令 → 跳过
3. **创建 symlink** — `ln -sf AGENTS.md CLAUDE.md`（已存在则覆盖）
4. **通知** — AGENTS.md 有任何写入 → MUST 输出：「AGENTS.md 已创建/更新。请**重启 Claude Code / Codex 会话**，使其作为 system prompt 生效后，再执行其他 skill。」；无变更 → 告知无需变更

## AGENTS.md 骨架

```markdown
# AGENTS

> 当前 project 是 AI Only 工程，人类输入的所有需求，都需要 AI(Claude Code、Codex...) 自主完成后续的所有工作。
> 默认 MUST(必须强制遵守); 仅例外用 SHOULD/MAY/MUST NOT/NEVER 标注。

## 指令路由（MUST）

- 【MUST】代码 / 架构 / 命令 / 结构 → [README.md](./README.md)
- 【MUST】调试 / 版本 / changelog / git commit+push / 预部署+发布 → [workflow.md](./workflow.md)

## 工作模式 (MUST、AI-only)

- 【MUST】运行稳定 + 长久可持续 = 第一优先级；MUST NOT 引入随时间劣化的隐患（资源泄漏 / 硬编码时效值 / 未释放连接 / ...）
- 【MUST】代码 / 测试 / 构建 / 部署 / 发布 全部由 AI(Claude Code、Codex、...) 执行，人类输入需求或咨询，AI 自主完成后续工作
- 【MUST】workflow.md 中的所有操作（git push / deploy / publish 等）已预授权，AI 直接执行，MUST NOT 等待人类确认
- 【MUST】决策 (架构 / 选型 / 命名 / 依赖 / ...) 需 AI 自行抉择, MUST NOT 拉人类进设计回路，MUST NOT 反问人类
- 【MUST】决策前通过各种方式（本工程检索、在线检索、其他途径）获取项目/需求事实，根据事实进行最优决策

## 工作流程

1. **分流**：判断人类输入是否涉及行为或交付物变更（feature / fix / config / 行为逻辑变更 = 需求变更）；纯咨询 / 纯文案·注释·md 内容调整（不改变运行行为或交付物）→ 直接响应或编辑，跳过后续步骤
2. AI 抉择并执行后续工作；开发过程中按需走 [workflow.md#调试](./workflow.md) 验证变更
3. 执行 [workflow.md#发布](./workflow.md) 完整流程；未走完 = 需求未完成，MUST NOT 提前停止

## 文档编写规范

- 全部文档只供 AI 查看，MUST 简洁精炼, 零冗余; MUST NOT 废话填充
- 能一行不写两行, 能一个单词不写两个单词, 能列表不写段落; 短句; `->` `/` `+` 替连接词
- 强度词: MUST / MUST NOT / SHOULD / MAY / NEVER
- 单一信源: 跨文档用 link 引用, MUST NOT 复述事实
- AGENTS 只写 LLM 约束, MUST NOT 塞工程说明 / 命令 / 安装
- 本段 = 全局写作标准; 其他 md 的 When Editing 仅补充各自特有约束
```

## 适配规则

- 标准路由（README.md + workflow.md）不可删，MAY 追加项目特有路由——路由是结构声明，目标文件将由对应 skill 创建，无需等待文件先存在
- 骨架内容是固定规范，新建时 MUST NOT 按项目类型裁剪（项目差异由 README.md / workflow.md 承载）
- 用户参数含明确修改指令 → 按指令更新已有内容（可添加路由 / 约束 / 自定义段落）
- AGENTS.md 有任何写入（新建 / 对齐 / 更新）→ MUST 要求重启；CLAUDE.md 仅在会话启动时加载，不重启则变更不生效
- 工程存在双语结构（`AGENTS.zh.md` 与 `AGENTS.md` 并存）→ 合并 `.zh.md` 独有事实入 AGENTS.md，删除 `AGENTS.zh.md`；若存在 `CLAUDE.zh.md` 一并删除。**即使主文件跳过生成，本条仍 MUST 执行**
- 工程存在独立 LLM 文本规范文件（`llm-doc-style.md` / `llm-doc-style.zh.md` 等）→ 删除；AGENTS.md「文档编写规范」段是唯一信源。**即使主文件跳过生成，本条仍 MUST 执行**
