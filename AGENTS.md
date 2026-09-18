# AGENTS.md — vibe-coding 全流程编排

> 把 AI 编码从"随手干"变成**按流程编排**。这份 rules 是项目根级始终生效的指令(放进项目根目录,或 `CLAUDE.md` / `AGENTS.md` / `.cursorrules` / Qoder rule 任一位置),AI 全程遵守。
>
> **它不是新方法论**,是把 [vibe-coding](https://gitee.com/IvenKooLab/vibe-coding)(方法论 skill 合集)+ `skill` 库(内部)(具体技能)两仓库的包,**串成一个完整的开发流程**。理念同源于 superspecflow,但跨 IDE 通用、subagent 可控、轻量任务可跳过。

---

## 0. 三条铁律(始终生效)

1. **先澄清,再动手**——除非用户明确说"直接改",否则任何创造性任务都先走 §1 澄清。**违反 = 返工**。
2. **subagent 不是默认动作**——只有满足 §4 的派发判据(重活 + 独立可验证 + 占大量 context)才派。**轻任务单 agent 直接干,别为拆而拆**。
3. **每个阶段有"出口"**——澄清要产出决策记录,实现要产出 diff,验证要产出证据。**没产出不算完成,不进下一阶段**。

---

## 1. 七阶段全流程

```
用户输入需求
    ↓
【① 澄清】 grill-me 把决策问透 → 产出:决策记录
    ↓
【② 选包】 /vibe 或按任务特征,定本任务走哪条路
    ↓
【③ 契约】 复杂任务 → openspec 写 spec(唯一可信基线)
          轻任务   → 跳过,决策记录即契约
    ↓
【④ 拆派】 任务拆成可分发单元 → 按判据决定哪些派 subagent
    ↓
【⑤ 实现】 主 agent + subagent 协同完成 → 产出:diff
    ↓
【⑥ 审查】 code-review(找缺陷)+ ponytail-review(找冗余)
    ↓
【⑦ 验证+归档】 跑测试给证据 → 决策/diff 回写 spec → 交付
```

---

## 2. 各阶段详解

### ① 澄清(Clarify) — grill-me

**做什么**:对需求做"无情盘问",一次一个问题,带推荐答案,把隐藏决策逼出来。
**产出**:决策记录(写进对话 / commit notes / openspec spec,看任务复杂度)。
**出口判据**:用户对"做什么、不做什么、成功标准"达成共识,无悬而未决的歧义。
**跳过条件**:用户明确"直接改"/ 任务是改错别字这种单一明确动作。

> 配套 skill:`grill-me`(vibe-coding)
> **subagent**:本阶段**不派**——澄清是和用户的对话,子 agent 没资格替用户拍板。

### ② 选包(Route) — /vibe

**做什么**:根据澄清后的任务,确定走哪条路。可用 `/vibe` 弹菜单,或直接按特征判断:
- 新功能/重构 → `superpowers`(brainstorming→writing-plans→TDD→debug→verify)
- 需求要跨会话/团队共识 → `openspec`
- 嫌啰嗦/过度设计 → 叠加 `ponytail`
- 跨会话记忆/多 agent → 叠加 `Trellis`

**出口判据**:明确"本任务启用哪几个包、以谁为主"。

> 配套 skill:`vibe`(vibe-coding)做总入口
> **subagent**:本阶段**不派**。

### ③ 契约(Contract) — openspec(可选)

**做什么**:复杂任务把需求/设计沉淀成仓库 spec(proposal/specs/design/tasks 四件套),作为唯一可信基线。
**判据**(满足任一即走 openspec):
- 跨会话要延续 / 多人要共识 / 合规要留痕
- 改动涉及契约(接口、消息体、数据模型)
- 任务复杂到需要 design.md

**轻任务跳过**:改个文案、调一行配置、修个明确 bug——决策记录即契约,不引入 openspec。

> 配套 skill:`openspec`(vibe-coding)
> **红线**:本阶段定义需求基线;后续阶段只读 spec,不改基线。
> **subagent**:可派一个子 agent 做"代码现状探查"(只读),主 agent 基于探查结果写 spec。

### ④ 拆派(Decompose & Dispatch)

**做什么**:把任务拆成可分发单元,并决定**每个单元是主 agent 干还是派 subagent**。

**subagent 派发判据**(三个**全部**满足才派):
1. **重活**——预计要读大量文件 / 跑大量探查 / 大段实现
2. **独立可验证**——子任务有明确产出,能在隔离 context 里独立完成
3. **占大量 context**——若主 agent 干会显著挤占窗口

**派发时必须给子 agent**:
- 任务边界(只干什么,不干什么)
- 上下文(需要的 spec/文件/约束,**不要**把主对话历史全带过去)
- 验收标准(产出什么算完成)

**不满足判据的,主 agent 直接干**——别为拆而拆,拆协调成本 > 收益。

> 配套 skill:`context-engineering`(skill 仓库)技能③「子 agent 分流」
> **反模式**:无脑拆成 N 份并行(= 专家团费 token 的根因)。判据是"该派才派"。

### ⑤ 实现(Implement) — superpowers

**做什么**:主 agent + subagent 协同完成编码。按选定的包执行:
- `superpowers` 5 纪律:brainstorming→writing-plans→TDD→systematic-debugging→verification
- 叠加 `ponytail`:实现时精简优先(stdlib/原生/复用)
- 涉及内部框架 → `openspec` Part B 路由到对应框架 skill

**主 agent 职责**:整体编排、跨子任务协调、关键决策。
**subagent 职责**:干分配的独立单元,**只回结论 + file:line 锚点**,不回完整探查过程。

**产出**:代码 diff。
**出口判据**:所有子任务完成,diff 可 review。

> 配套 skill:`superpowers`(vibe-coding)+ `ponytail`(vibe-coding)+ `context-engineering`(skill)
> **subagent 协调**:主 agent 收子 agent 结论后,主动压缩(技能②),不让子 agent 产出堆满主 context。

### ⑥ 审查(Review) — code-review + ponytail-review

**做什么**:对 diff 做双视角审查。
- `code-review`:7 维度(Security/Performance/Correctness/Maintainability/Testing/Accessibility/Docs)找**缺陷**
- `ponytail-review`:找**冗余**(过度设计/能砍的代码)

**两者正交,都跑**。code-review 按严重度(CRITICAL/MAJOR/MINOR/NIT)分级;ponytail-review 给"能砍多少行"。

**出口判据**:无 CRITICAL / MAJOR 阻塞项;或已修完。

> 配套 skill:`code-review`(skill 仓库)+ `ponytail-review`(vibe-coding/ponytail 子包)
> **subagent**:可派一个子 agent 跑 code-review(独立可验证、占 context),主 agent 收审查报告。轻 diff 直接主 agent 审。

### ⑦ 验证 + 归档(Verify & Archive)

**做什么**:
1. **验证**:实跑构建/测试/lint,给可核对证据。未验证项明确标"unverified + 谁来覆盖"。
2. **归档**:
   - 走了 openspec 的 → 决策和 diff 回写进 `openspec/specs/`(跨会话可查)
   - 跨会话项目 → Trellis 记 memory,下个会话不"失忆"
   - 决策有价值的 → 沉淀进 CLAUDE.md / 本 AGENTS.md(让下个任务更聪明)

**出口判据**:每个验收项有实跑证据;归档完成。
**交付**:代码 + 审查报告 + 验证证据 + 归档记录。

> 配套 skill:`superpowers` §verification-before-completion + `openspec` archive + `Trellis`

---

## 3. 全流程速查(贴墙上)

| 阶段 | 做什么 | 主 skill | 派 subagent? | 出口 |
|---|---|---|---|---|
| ① 澄清 | 盘问需求 | grill-me | 否 | 决策记录 |
| ② 选包 | 定路线 | vibe | 否 | 启用包清单 |
| ③ 契约 | 写 spec(复杂才走) | openspec | 可派探查 | 四件套/跳过 |
| ④ 拆派 | 拆单元 + 定派发 | context-engineering | —(本阶段决定派不派) | 任务+派发单 |
| ⑤ 实现 | 编码 | superpowers+ponytail | 按判据派 | diff |
| ⑥ 审查 | 双视角审 | code-review+ponytail-review | 可派 review | 审查报告 |
| ⑦ 验证归档 | 跑测试+回写 | verification+openspec+Trellis | 否 | 证据+归档 |

---

## 4. subagent 调度纪律(重点,防 token 黑洞)

> 专家团/多 agent 之所以费 token,根因是**无脑拆**。本节是把"派不派"变成可控判据。

### 派发判据(三个全满足才派)

```
重活(读大量文件/大段实现)
  AND 独立可验证(有明确产出)
  AND 占大量 context(主 agent 干会挤爆)
→ 派 subagent,隔离 context 干,只回结论
```

### 派发时给子 agent 的"上下文包"

- ✅ 任务边界(只干什么)
- ✅ 必要 spec / 相关文件路径
- ✅ 约束和验收标准
- ❌ **不要**主对话完整历史(只给相关片段)
- ❌ **不要**让子 agent "自由探索"(边界要明确)

### 收结论时

主 agent 收子 agent 产出后,**立即压缩**(context-engineering 技能②):保留结论 + file:line,丢掉探查过程。不让子 agent 产出在主 context 越积越多。

### 典型场景

| 场景 | 派不派 | 理由 |
|---|---|---|
| 全仓找某接口所有调用点 | ✅ 派 | 重活+独立+占 context,只回调用清单 |
| 改一个 bug | ❌ 不派 | 单 agent 直接干更快 |
| 大重构(动 20+ 文件) | ✅ 按模块拆派 | 每模块独立,主 agent 协调 |
| 写一个 CRUD 接口 | ❌ 不派 | 不重,直接干 |
| 跑全量测试套件 | ✅ 派 | 独立+占 context,只回 pass/fail |
| 澄清需求/和用户对话 | ❌ 不派 | 子 agent 没资格替用户拍板 |

---

## 5. 轻量通道(不是所有任务都走全流程)

**小任务快速通道**(满足任一):
- 改错别字 / 调一行配置 / 改文案
- 单文件小改,边改边验证
- 用户明确"直接改别问"

→ 跳过 ①②③,直接 ⑤(用 ponytail 精简)+ ⑦(验证)。**别用全流程杀鸡**。

**判定**:任务能在 5 分钟内独立完成且单一明确 = 走快速通道。

---

## 6. 内部框架编码规范(跨项目硬约束·模板)

> 本节放**你的框架级底线**——所有项目都要遵守、不属于任何单一中间件的通用硬约束。公开版只给模板示意;把你框架的真实规范填进来,违反即 CRITICAL。示例骨架:

1. **父 POM 与版本管理**:所有项目必须继承框架父 POM;starter 依赖**禁止显式指定 `<version>`**(由父 POM BOM 统一管理)。
2. **包名规范**:遵循你组织的包名约定(如 `com.yourorg.{业务线}.{服务名}`)。
3. **配置命名空间**:框架配置统一在你的框架命名空间下(如 `your-framework:`)。
4. **配置文件位置**:业务配置放 `config/` 子目录,禁止放 classpath 根(会遮蔽框架内置配置);环境相关配置走 `application-{env}.yml`。
5. **生产配置来源**:中间件地址/密码等**必须由配置中心或平台提供,禁止硬编码**。
6. **事务规范**:写操作 Service 方法必须 `@Transactional(rollbackFor = Exception.class)`。
7. **扫描路径**:集成框架模块时启动类必须配对应 `@ComponentScan`(按你的框架约定)。
8. **版本下限**:为各框架组件定最低版本,项目统一锁定到满足所有下限的版本。
9. **生产日志/端点安全**:生产禁止全量 SQL 日志;Actuator 端点禁止 `"*"` 暴露,必须精确控制。
10. **代码生成器产物不进生产**。

> 任何一条违反,§⑥ code-review 阶段必须标 `[CRITICAL]` 或 `[MAJOR]`。

---

## 7. 中间件规范索引(按需查阅,不抄全文·模板)

> 每个中间件的专属硬约束放在**对应的框架 skill 里**,本节只做索引——AI 用到某中间件时,**先读对应 skill 再动手**。公开版为占位示意:

| 涉及场景 | 必查 skill | 索引写什么(示意) |
|---|---|---|
| 缓存/分布式锁 | `your-framework-redis` | 禁用命令清单、连接池上限、Bean 命名约定 |
| 数据库访问 | `your-framework-orm` | 多参数注解、分页姿势、命名空间约定 |
| 消息队列 | `your-framework-mq` | 前缀规范、保留名、幂等要求 |
| 单点登录 | `your-framework-sso` | session 共享、双 URL 配置、前端适配头 |

**怎么用**:在 §⑤ 实现阶段,识别任务涉及的中间件 → 读对应 skill 全文 → 按其规范写码。**禁止凭记忆写中间件代码**。

> **为什么不抄进 AGENTS.md**:① 全文塞进来会爆 context 且每个项目只用其中几个;② 框架 skill 是单一数据源,抄进来双份维护、易脱节;③ 符合 context-engineering「按需读取」——用到才读,不预载。

---

## 8. 与三个仓库的对应

| 阶段 | 调用什么 | 仓库 |
|---|---|---| 
| ①②③④⑤⑥⑦ | 方法论 skill | vibe-coding |
| ④⑤⑥ | 具体技能(code-review / context-engineering) | skill |
| ⑤(读配置/读笔记) | MCP 工具 | mcp |
| **⑤(写框架相关代码)** | **框架规范(§6 硬约束 + §7 索引)** | **框架 skill(内部 skill 库)** |

> 这份 AGENTS.md 是**编排层**,本身不含方法论——方法论在 vibe-coding/skill 仓库,工具在 mcp 仓库,框架规范在框架 skill。它只管"怎么把它们串起来"。

---

## 9. 怎么用

放项目根命名 `AGENTS.md`(Qoder 放 `.qoder/rules/` 配 `trigger: always_on`),装好 §13 清单里的 skill,对 AI 说"按 AGENTS.md 流程处理 [需求]"。纯文本约定,Claude Code / Qoder / Trae 通用,subagent 靠 AI 内置 tool 能力,不依赖专家团功能。

---

## 10. 关注点的垂直贯穿(借鉴理赔设计)

> 前面那份团队实践总结里有个精妙设计:**同一关注点在三阶段各出现一次,抽象层次递进不重复**。本流程吸收这个思路——同一个东西,在 ① 澄清 / ③ 契约 / ⑤ 实现 三个阶段,各问不同层次的问题,逐步从"要不要"逼到"怎么做"。

| 关注点 | ① 澄清(要不要) | ③ 契约(怎么设计) | ⑤ 实现(代码怎么写) |
|---|---|---|---|
| **并发** | 这功能需要锁吗? | 锁策略与粒度(RedisLock?DB锁?) | RedisLock 代码(查 §7 your-framework-*) |
| **数据** | 有 DDL/刷数吗? | OB 兼容 + DDL 方案 | PO + Mapper(查 §7 your-framework-*) |
| **接口** | 新增还是改?出入参? | 接口规格 + 兼容 + 缓存 + 加密 | DTO + Serializable |
| **消息** | 要异步吗? | Topic/Tag 设计 + 幂等方案 | Producer/Consumer(查 §7 your-framework-*/rocketmq) |
| **缓存** | 要缓存吗?读多写少? | TTL + Key 设计 + 一致性策略 | RedisService 调用(查 §7 your-framework-*) |

> 别在一个阶段问完所有层次的细节。**澄清只问"要不要",契约只定"怎么设计",实现才写"代码"**。每个阶段只做自己这一层,下一阶段接着做更深一层。

---

## 11. 反模式(Never)

- **Never** 跳过澄清直接动手(除非用户说"直接改")
- **Never** 无脑拆 subagent(= 专家团费 token 根因)——满足 §4 三判据才派
- **Never** 让子 agent 带主对话完整历史——只给相关片段
- **Never** 收子 agent 结论不压缩——产出会挤爆主 context
- **Never** 用全流程杀鸡(改错别字走快速通道)
- **Never** 实现完不审查/不验证就声称完成
- **Never** 复杂任务不写 spec 就开工(契约缺失=后期返工)
- **Never** 凭记忆写框架/中间件代码——涉及 Redis/MQ/SSO 等必须先读对应 §7 skill(context-engineering 技能⑤)
- **Never** 违反 §6 的 10 条框架硬约束(如 starter 显式指定 version、配置放 classpath 根、生产硬编码中间件地址)——code-review 标 CRITICAL
- **Never** 在一个阶段问完所有层次——澄清只问"要不要",契约定"怎么设计",实现写"代码"(§11 垂直贯穿)

---

## 12. 启动自检(会话开始时跑一次)

> 本流程依赖一组外部 skill / 工具。**会话开始时(AI 第一次读 AGENTS.md 后),先跑一遍自检**,确认引用的能力是否就位。缺了就告诉用户缺什么、影响哪个阶段,不要等跑到一半才断链。

### 怎么自检

1. **查 skill 是否可发现**:对下面"必装清单"每个 skill,判断当前环境能不能加载(看 skills 目录 / 问用户)。
2. **查 mcp 是否就位**:nacos2-mcp 等工具,问用户是否已配置。
3. **查引导层**:确认本 AGENTS.md 是否被工具真正加载(Claude Code 放根目录自动读;Qoder 需在 `.qoder/rules/` 配 `trigger: always_on`;Trae 在 `.trae/rules/`)。
4. **输出自检报告**:

```
## AGENTS.md 启动自检

引导层: ✅ 已加载 / ⚠️ 未加载(说明怎么配)
skill 就位: N/9
  ✅ grill-me / ponytail / openspec / superpowers / vibe(vibe-coding 仓库)
  ✅ code-review / context-engineering(skill 仓库)
  ✅ your-framework-* / your-framework-* / ...(内部 skill 库,按项目用到的报)
工具就位:
  ✅ / ⚠️ nacos2-mcp
缺失影响:
  [缺什么] → [影响哪个阶段] → [怎么补]
```

### 必装清单(按阶段)

| 阶段 | skill / 工具 | 来源 | 缺了的影响 |
|---|---|---|---|
| ① 澄清 | `grill-me` | vibe-coding | 无法走需求盘问,① 降级为普通对话 |
| ② 选包 | `vibe` | vibe-coding | 无 /vibe 菜单,② 靠 AI 按特征判断 |
| ③ 契约 | `openspec` | vibe-coding | 无法写 spec,复杂任务契约缺失 |
| ④ 拆派 | `context-engineering` | skill 仓库 | subagent 调度无纪律,易退化成无脑拆 |
| ⑤ 实现 | `superpowers`, `ponytail` | vibe-coding | 无方法论纪律,实现质量靠运气 |
| ⑤ 写码 | 框架 skill(按项目用到的) | 内部 skill 库 | 违反框架硬约束(§6),code-review 标 CRITICAL |
| ⑥ 审查 | `code-review`, `ponytail-review` | skill / vibe-coding | 审查降级,缺陷可能漏过 |
| ⑤ 读配置 | `nacos2-mcp` | mcp 仓库 | AI 读不到 nacos,逻辑靠猜(可选,看项目) |

### 自检规则

- **缺 skill 不阻塞启动**——AI 仍可工作,但要把"缺失 + 影响 + 怎么补"明确报给用户,让用户决定补不补。
- **缺关键 skill 时降级运行**:比如缺 grill-me,① 澄清降级为"AI 直接问几个关键问题",不强制走无情盘问。
- **只报不修**——自检不负责装 skill(那是用户的事),只负责让用户知道当前能力边界。
- **后续会话不重复自检**——同一项目同一环境,自检跑一次即可(除非用户换了工具/重装了 skill)。

> 自检的目的不是"卡住流程",是**让用户对当前能力边界心里有数**。缺什么先说清楚,比跑到 ⑥ 审查才发现没装 code-review 强得多。

---

## 致谢

- 理念同源于 superspecflow(规格驱动 + 方法论纪律)
- 编排对象:[vibe-coding](https://gitee.com/IvenKooLab/vibe-coding) + `skill` 库(内部) + [mcp](https://gitee.com/IvenKooLab/mcp) 三仓库
- subagent 调度纪律参考 [context-engineering](你的内部 git 仓库) skill 技能③
