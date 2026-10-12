# sub-agent — AI Coding Orchestration Discipline

**English** · [简体中文](#简体中文--ai-编码全流程编排)

> Skill packs (grill-me, ponytail, openspec, trellis, superpowers…) are the parts.
> This repo is the **assembly manual** — one `AGENTS.md` that chains them into a
> controlled 7-phase flow, dispatching subagents only when it actually pays off.

## The problem

"Expert team / multi-agent" AI coding modes burn tokens for one reason: **blind
dispatching**. Every step spawns a subagent, the main agent degrades into a pure
scheduler, and you pay coordination overhead for work it could have done itself.

## The fix: three dispatch criteria

A subagent is dispatched only when **all three** hold:

1. **Heavy** — reads many files / a large implementation
2. **Independently verifiable** — has a concrete, checkable deliverable
3. **Context-heavy** — doing it in the main agent would crowd out its context

Otherwise the main agent does the work itself. Not every task deserves a
committee — fixing a typo through a full pipeline is killing a chicken with an
orchestra.

## The 7-phase flow

| Phase | What | Lead skill | Subagent? |
|---|---|---|---|
| ① Clarify | Interrogate the requirement | grill-me | No |
| ② Route | Pick the path / packs | vibe | No |
| ③ Contract | Spec for complex work (skip for light) | openspec | Optional scout |
| ④ Decompose | Split into units + dispatch plan | context-engineering | — |
| ⑤ Implement | Code | superpowers + ponytail | By criteria |
| ⑥ Review | Two-lens review | code-review + ponytail-review | Optional |
| ⑦ Verify & archive | Tests + write-back | verification + openspec | No |

## Quick start

1. Copy [`AGENTS.md`](AGENTS.md) into your project root (or append it to
   `CLAUDE.md` / your Qoder rule / `.cursorrules`)
2. Make sure the vibe-coding skill packs are installed
3. Tell your agent: "Follow AGENTS.md for this requirement: [requirement]"

## Ecosystem

| Repo | Role | Content |
|---|---|---|
| [vibe-coding-skills](https://github.com/IvenKooLab/vibe-coding-skills) | Methodology skills | grill-me / ponytail / openspec / trellis / superpowers + vibe entry |
| [nacos2-mcp](https://github.com/IvenKooLab/nacos2-mcp) | Tooling | Nacos 2.x read-only MCP server |
| Framework skills (BYO) | Framework knowledge | Your middleware specs (see AGENTS.md §7 template) |
| **This repo** | **Orchestration** | **AGENTS.md: full flow + dispatch discipline + framework constraint templates** |

## License

MIT

---

# 简体中文 · sub-agent · AI 编码全流程编排

> vibe-coding / nacos2-mcp 等 skill 包是"零件"，本仓库是**装配图**——一份 `AGENTS.md` 把它们串成可控的 7 阶段开发流程，该派 subagent 时才派。

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 解决什么问题

AI 编码的"专家团/多 agent"模式，费 token 的根因是**无脑拆**——每步都派 subagent，主 agent 变成纯调度器。

本仓库用 **subagent 派发三判据** 把"派不派"变成可控决策：

> 重活（读大量文件/大段实现）**AND** 独立可验证（有明确产出）**AND** 占大量 context（主 agent 干会挤爆）
> → 满足全部三条才派，隔离 context 干，只回结论

## 7 阶段全流程

| 阶段 | 做什么 | 主 skill | 派 subagent? |
|---|---|---|---|
| ① 澄清 | 盘问需求 | grill-me | 否 |
| ② 选包 | 定路线 | vibe | 否 |
| ③ 契约 | 写 spec（复杂才走） | openspec | 可派探查 |
| ④ 拆派 | 拆单元 + 定派发 | context-engineering | — |
| ⑤ 实现 | 编码 | superpowers+ponytail | 按判据派 |
| ⑥ 审查 | 双视角审 | code-review+ponytail-review | 可派 review |
| ⑦ 验证归档 | 跑测试+回写 | verification+openspec | 否 |

> **不是所有任务都走全流程**——改错别字走轻量通道（直接实现+验证）。别用全流程杀鸡。

## 怎么用

1. 复制 [`AGENTS.md`](AGENTS.md) 到你的业务项目根目录（或追加进 `CLAUDE.md` / Qoder rule / `.cursorrules`）
2. 确保项目装了 vibe-coding 相关包
3. 对 AI 说："按 AGENTS.md 流程，处理这个需求:[需求]"

## 仓库分工

| 仓库 | 性质 | 内容 |
|---|---|---|
| [vibe-coding-skills](https://github.com/IvenKooLab/vibe-coding-skills) | 方法论级 skill | grill-me / ponytail / openspec / trellis / superpowers + vibe 入口 |
| [nacos2-mcp](https://github.com/IvenKooLab/nacos2-mcp) | 工具型 | Nacos 2.x 只读 MCP server |
| 框架 skill（自备） | 框架知识级 | 各中间件规范（示例见 AGENTS.md §7 模板） |
| **本仓库** | **编排层** | **AGENTS.md：全流程 + 调度纪律 + 框架硬约束模板** |

## License

MIT
