# Qoder 里用 vibe-coding 这套(使用指南)

> 怎么在 Qoder(阿里 AI IDE)里把我们做的 AGENTS.md + vibe-coding skill + skill 仓库 + mcp 跑起来。
>
> 配置细节依据 Qoder 官方文档(docs.qoder.com),不凭记忆。

---

## 0. 先搞清楚 Qoder 的四样东西

| 概念 | 是什么 | 放哪 | 加载方式 |
|---|---|---|---|
| **AGENTS.md** | 项目级总指令(我们那套全流程编排) | 项目根目录 | **自动识别**,AI 启动就读 |
| **Rules(规则)** | 项目级约束(始终/按需生效) | `.qoder/rules/*.md` | 看规则类型(四种,见 §2) |
| **Skills(技能)** | 可复用工作流(vibe-coding 的包) | `~/.qoder/skills/` 或 `<项目>/.qoder/skills/` | **渐进披露**:先加载 metadata,触发才读全文 |
| **MCP** | 工具扩展(nacos2-mcp 等) | 个人设置 → MCP | AI 按需调用 |

**关键区别**:
- **Rules 永久占 context**(Always Apply 类型);**Skills 渐进披露**(不触发不占 context)。所以——
- 重型内容(方法论、技能)做 **Skill**(vibe-coding 那套)
- 轻型硬约束(编码规范、命名)做 **Rule** 或进 AGENTS.md

> 我们的 AGENTS.md 本身就是"项目级总指令",Qoder 直接认——不用转成 Rule。

---

## 1. 装我们的东西:三步

### 第 1 步:放 AGENTS.md(项目级编排)

把 [rules 仓库](你的内部 git 仓库) 里 `sub-agent/AGENTS.md` **复制到你的项目根目录**,保持文件名 `AGENTS.md`。

Qoder 会**自动识别**项目根的 AGENTS.md,AI 启动就读。不需要额外配置。

> ⚠️ 如果 AGENTS.md 和 `.qoder/rules/` 里的规则冲突,**rules 内容优先**。所以别在两边写矛盾的东西。

### 第 2 步:装 Skill(方法论 + 技能)

从三个仓库 clone,把 skill 目录复制到 Qoder 的 skills 目录:

```
# 全局(所有项目可用)
~/.qoder/skills/

# 或项目级(只当前项目)
<你的项目>/.qoder/skills/
```

**目录结构必须是 `<skill-name>/SKILL.md`**(kebab-case 目录名,SKILL.md 大小写固定):

```
~/.qoder/skills/
├── vibe/                    ← 来自 vibe-coding 仓库
│   └── SKILL.md
├── superpowers/             ← 来自 vibe-coding 仓库
│   └── SKILL.md
├── grill-me/                ← 来自 vibe-coding 仓库
│   └── SKILL.md
├── ponytail/                ← 来自 vibe-coding 仓库(6 个子 skill 全拷)
│   └── ...
├── openspec/                ← 来自 vibe-coding 仓库
│   └── SKILL.md
├── trellis/                 ← 来自 vibe-coding 仓库
│   └── ...
├── code-review/             ← 来自 skill 仓库
│   └── SKILL.md
└── context-engineering/     ← 来自 skill 仓库
    └── SKILL.md
```

**怎么拷**(从 vibe-coding 仓库):
```bash
# 假设你 clone 了 vibe-coding 到 ~/vibe-coding
cp -r ~/vibe-coding/vibe/skills/* ~/.qoder/skills/          # vibe 入口(注意是 vibe/skills/ 下的子目录)
cp -r ~/vibe-coding/grill-me ~/.qoder/skills/               # 单文件包直接拷目录
cp -r ~/vibe-coding/openspec ~/.qoder/skills/
cp -r ~/vibe-coding/superpowers/skills/* ~/.qoder/skills/
cp -r ~/vibe-coding/ponytail/skills/* ~/.qoder/skills/
cp -r ~/vibe-coding/Trellis/skills/* ~/.qoder/skills/
```

> 拷完后,SKILL.md 的 frontmatter(`name` + `description`)会被 Qoder 加载成 metadata。AI 根据 description 判断该不该触发——**所以 description 写的是"何时用",不是"怎么做"**(我们 vibe-coding 的 skill 都遵循这个规范)。

### 第 3 步:装 MCP(可选,看项目需要)

如果项目要用 nacos2-mcp(让 AI 读 nacos 配置):

在 Qoder 的**个人设置 → MCP** 里配置 nacos2-mcp 服务(具体配置见 [mcp 仓库](你的内部 git 仓库) 的 README)。

> MCP 是工具扩展,AI 按需调用,不需要像 skill 那样"触发"。

---

## 2. Rules(规则)——可选,看你要不要加项目级硬约束

AGENTS.md 已经是项目级总指令,**大多数情况不需要再单独写 Rules**。但如果你想要一些"始终生效"的轻约束(且不想塞进 AGENTS.md 让它变厚),可以加 Rule。

### Rules 的四种类型(Qoder 官方)

| 类型 | 怎么生效 | 适合放什么 | 占 context? |
|---|---|---|---|
| **Always Apply** | 所有 Chat/Inline Chat 请求都带 | 项目级硬标准(编码规范、命名约定) | ✅ 始终占 |
| **Model Decision** | AI 读 description 自己决定何时触发 | 场景化任务(如"涉及金融计算时走特殊流程") | 触发才占 |
| **Apply Manually** | 用户输入 `@rule名称` 触发 | 按需工作流 | 用到才占 |
| **Specific Files** | 匹配文件路径通配符(`*.java`/`src/*`)自动触发 | 目录专属规则 | 命中文件才占 |

### 怎么加 Rule

1. 在项目下创建 `.qoder/rules/` 目录
2. 写一个 `.md` 文件(自然语言,支持 markdown 格式)
3. 在 Qoder 设置里把它配成对应类型(Always Apply / Model Decision 等)

> **Rule 文件是自然语言**,不像 Skill 有 frontmatter。就是一段段约束说明。

### Rule vs Skill vs AGENTS.md 怎么选

| 你要的东西 | 放哪 | 理由 |
|---|---|---|
| 全流程编排(7 阶段) | **AGENTS.md** | 项目级总指令,Qoder 自动读 |
| 方法论/技能(grill-me/ponytail/code-review) | **Skill** | 重型内容,渐进披露,触发才占 context |
| 项目硬约束(必须用 Druid、禁 sysout) | **Rule(Always Apply)** | 轻型,始终生效 |
| 场景化流程(涉及金融走特殊) | **Rule(Model Decision)** | AI 按场景触发 |

> **原则**:能用 AGENTS.md 一句带过的别开 Rule;能做 Skill 的别做成 Always Apply Rule(省 context)。

---

## 3. SKILL.md 格式要求(Qoder 兼容)

我们 vibe-coding/skill 仓库的 SKILL.md **已经符合 Qoder 要求**,但你如果自己写新 skill,要注意:

### 必填 frontmatter

```yaml
---
name: your-skill-name          # 必填,kebab-case,小写+短横线,= 目录名
description: |                 # 必填,≤1024 字符,是 AI 判断"是否调用"的唯一依据
  [做什么] + [何时用,含触发词]
  例如:当用户需要 X,或说到"关键词1""关键词2"时,使用本 skill。
---
```

### 可选 frontmatter

```yaml
license: MIT
metadata:
  author: 你的名字
  version: 1.0.0
  category: development
  tags: [api, testing]
```

### 触发方式

1. **AI 自动**:读 description,任务匹配就调用
2. **手动**:`/skill-name`(skill 名当斜杠命令)

> **Slash Command 能做的,Skill 都能做**。所以我们没单独做 slash command 文件——`/grill-me`、`/code-review` 这些靠 skill 本身的 name 触发即可。

---

## 4. 跑起来:典型工作流

装好上面三步后,典型用法:

```
你: 按项目根 AGENTS.md 的流程,帮我加一个导出功能

AI: (启动自检 §12,报告 skill/MCP 就位情况)
    (走 §1 七阶段流程)
    ① 澄清:先问你几个关键问题(grill-me)
    ② 选包:这是新功能,建议走 superpowers
    ...
```

或者直接用某个 skill:
```
你: /grill-me 拷问我这个方案
你: /code-review 审一下这段代码
你: /vibe   (弹菜单选包)
```

---

## 5. 常见坑

1. **SKILL.md 大小写错** → 必须正好是 `SKILL.md`(全大写),写成 `skill.md` / `Skill.md` 都不行
2. **目录名和 name 不一致** → 目录名必须 = frontmatter 的 `name` 字段(kebab-case)
3. **description 只写"做什么"没写"何时用"** → AI 不知道何时触发,skill 等于白装
4. **Always Apply Rule 塞太多** → 每次对话都占 context,工具变慢变贵;重型内容改做 Skill
5. **AGENTS.md 和 Rule 写矛盾** → Rule 优先,AGENTS.md 被覆盖,排查困难

---

## 6. 一句话总结

**放 AGENTS.md 到项目根 + 拷 skill 到 `~/.qoder/skills/` + (可选)配 MCP** —— 三步,Qoder 里就能用上 vibe-coding 全流程编排。Rule 是可选的补充,大多数情况 AGENTS.md 够了。

---

## 参考文档

- [Qoder Rules 官方文档](https://docs.qoder.com/user-guide/rules.md)
- [Qoder Skills 官方文档](https://docs.qoder.com/extensions/skills.md)
- [Qoder MCP 配置(阿里云)](https://help.aliyun.com/zh/lingma/qoder-cn/user-guide/guide-for-using-mcp)
- [Qoder 设置](https://docs.qoder.com/zh/plugins/settings)
- 我们的三仓库:[vibe-coding](https://gitee.com/IvenKooLab/vibe-coding) / `skill` 库(内部) / [mcp](https://gitee.com/IvenKooLab/mcp) / [rules](你的内部 git 仓库)
