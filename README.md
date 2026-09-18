# rules · vibe-coding 全流程编排

> vibe-coding / skill / mcp 三个仓库是"零件",本仓库是**装配图**——把那些 skill 包串成一个完整的 AI 编码开发流程,可控地调度 subagent,而不是依赖某个 IDE 自带的"专家团"。

---

## 这是什么

一份 `AGENTS.md`,放进项目根目录即始终生效。它定义了 **7 阶段全流程**(澄清→选包→契约→拆派→实现→审查→验证归档),把三个仓库的能力编排起来:

```
vibe-coding(方法论 skill)┐
skill(具体技能)          ├──→ AGENTS.md 编排 ──→ 完整开发流程
mcp(工具)                ┘
```

**核心理念**:不是新方法论,是**编排层**——方法论在 vibe-coding/skill,工具在 mcp,本仓库只管"怎么把它们串成一个可控的流程"。

---

## 解决什么问题

- **专家团/多 agent 费 token**——根因是"无脑拆"。本 rules 用 **subagent 派发三判据**(重活 + 独立可验证 + 占大量 context)把"派不派"变成可控决策,该派才派。
- **跨 IDE 不通用**——各 IDE 的"专家团"功能各自为政。本 rules 是纯文本约定,Claude Code / Qoder / Trae 都读,subagent 靠 AI 内置 tool 能力派发,不依赖任何 IDE 专属功能。
- **vibe-coding 缺总编排**——vibe-coding 有 6 个包 + /vibe 入口,但没有"怎么串成完整流程"。本 rules 补这层。

---

## 7 阶段速查

| 阶段 | 做什么 | 主 skill | 派 subagent? |
|---|---|---|---|
| ① 澄清 | 盘问需求 | grill-me | 否 |
| ② 选包 | 定路线 | vibe | 否 |
| ③ 契约 | 写 spec(复杂才走) | openspec | 可派探查 |
| ④ 拆派 | 拆单元 + 定派发 | context-engineering | — |
| ⑤ 实现 | 编码 | superpowers+ponytail | 按判据派 |
| ⑥ 审查 | 双视角审 | code-review+ponytail-review | 可派 review |
| ⑦ 验证归档 | 跑测试+回写 | verification+openspec+Trellis | 否 |

> **不是所有任务都走全流程**——改错别字这种走 §5 轻量通道(直接实现+验证)。别用全流程杀鸡。

---

## 怎么用

1. 复制本目录(`sub-agent/`)下的 `AGENTS.md` 到你的业务项目根目录(或追加进 `CLAUDE.md` / Qoder rule / `.cursorrules`)
2. 确保项目装了 vibe-coding 相关包 + skill 仓库的 code-review/context-engineering
3. 对 AI 说:"按 AGENTS.md 流程,处理这个需求:[需求]"
4. AI 按 7 阶段走,该派 subagent 时派,该走快速通道时走

---

## subagent 调度纪律(重点)

专家团费 token 的根因是**无脑拆**。本 rules 把"派不派"变成三判据:

```
重活(读大量文件/大段实现)
  AND 独立可验证(有明确产出)
  AND 占大量 context(主 agent 干会挤爆)
→ 派 subagent,隔离 context 干,只回结论
```

不满足判据的,主 agent 直接干。典型场景:

| 场景 | 派不派 | 理由 |
|---|---|---|
| 全仓找某接口所有调用点 | ✅ 派 | 重活+独立+占 context |
| 改一个 bug | ❌ 不派 | 单 agent 更快 |
| 大重构(动 20+ 文件) | ✅ 按模块拆派 | 每模块独立 |
| 写一个 CRUD 接口 | ❌ 不派 | 不重,直接干 |
| 跑全量测试套件 | ✅ 派 | 独立+占 context |

详见 `AGENTS.md` §4。

---

## 仓库分工

| 仓库 | 性质 | 内容 |
|---|---|---|
| [vibe-coding](https://gitee.com/IvenKooLab/vibe-coding) | 方法论级 skill | grill-me / ponytail / openspec / trellis / superpowers + vibe 入口 |
| `skill` 库(内部) | 具体技能级 skill | code-review / context-engineering 等 |
| [mcp](https://gitee.com/IvenKooLab/mcp) | 工具型 | nacos2-mcp |
| 框架 skill(内部 skill 库) | 框架知识级 | 各中间件规范(redis/mybatis/mq/sso 等) |
| **rules**(本仓库) | **编排层** | **`sub-agent/`(AGENTS.md 把上面四者串成全流程 + 框架硬约束模板 §6 + 中间件索引模板 §7)** |

> AGENTS.md 现在含四块:① 7 阶段全流程编排 ② subagent 调度纪律 ③ **框架编码规范模板(10 条跨通用硬约束)** ④ **中间件规范索引模板(指向你的框架 skill,不抄全文,按需查阅)**。框架规范不照搬框架 skill 全文(避免双份维护),只放跨通用红线 + 索引。

---

## 与 superspecflow 的关系

同源于"规格驱动 + 方法论纪律"路线。差异:
- superspecflow 用三命令强绑流程,**重型**
- 本 rules 用 7 阶段 + 轻量通道 + subagent 判据,**重任务走完整流程,轻任务可跳**
- 底料同源(OpenSpec + Superpowers),可叠加

---

## 致谢

- 理念同源于 superspecflow(规格驱动 + 方法论纪律)
- 编排对象:vibe-coding + skill + mcp 三仓库
- subagent 调度纪律参考 context-engineering skill 技能③
