# Hook 机械拦截指导

> AGENTS.md / Rules / Skills 都是"软约束"——靠 AI 自觉遵守,概率性的(模型可能无视)。
> Hook 是"硬约束"——**确定性的脚本,事件触发就执行,不依赖模型听话**。本文档教你什么时候该用 Hook、怎么配置、和软约束怎么分工。

---

## 0. 一句话定位

> **软约束管"应该怎么做"(概率遵守),硬约束管"绝对不能怎么做"(100% 拦截)。**
> 同一个关注点,先写进 AGENTS.md(软),真正不可妥协的再落地成 Hook(硬)。

---

## 1. Hook 是什么 / 和 Rules、Skills 的区别

| | Rules / Skills / AGENTS.md | **Hook** |
|---|---|---|
| 本质 | 给 AI 的提示词(AI 解析后决定做不做) | **系统级脚本**(事件触发就跑,确定性) |
| 遵守概率 | 概率性(模型可能无视、谎报) | **100%**(不通过就阻断) |
| 能不能拦截工具调用 | 不能,只能"建议" | **能**(exit 2 阻断) |
| 适合管什么 | 流程、方法论、编码风格 | 安全红线、质量机械检查、防误操作 |
| 代价 | 几乎无(只是文本) | 要写脚本、有维护成本 |

**关键洞察(来自一个真实团队的实践)**:
> "它把'AI 守规矩'的概率从 60% 提到 90%,但那 10% 依然存在。真正的强约束应该落到 Hook 这种确定性环节。"

**所以**:AGENTS.md 把概率提到 90%,剩下那 10% 不可妥协的,用 Hook 兜底到 100%。

---

## 2. Qoder Hook 能拦什么(五种事件)

依据 Qoder 官方文档(docs.qoder.com/extensions/hooks)。Hook 在 Agent 执行的五个生命周点触发:

| 事件 | 触发时机 | 能阻断? | 典型用途 |
|---|---|---|---|
| **UserPromptSubmit** | 用户提交 prompt 后、Agent 处理前 | ✅ 能 | 检查 prompt 质量、注入提示、拦截危险指令 |
| **PreToolUse** | 工具即将执行前 | ✅ 能 | **拦截危险操作**(svn commit、删文件、写敏感路径) |
| **PostToolUse** | 工具执行成功后 | ❌ 不能 | 自动 lint、质量检查、自动补全 |
| **PostToolUseFailure** | 工具执行失败后 | ❌ 不能 | 失败日志、告警 |
| **Stop** | Agent 准备结束响应时 | ✅ 能(不让它停) | 强制再跑一轮验证、触发回顾 |

> **只有三种事件能阻断**:UserPromptSubmit、PreToolUse、Stop。要用 Hook 拦截操作,必须挂在这三种上(最常用 PreToolUse)。

---

## 3. 怎么配置(JSON,三处合并)

Hook 用 **JSON** 配置,优先级从低到高合并三处:

```
1. ~/.qoder/settings.json          (用户级,所有项目)
2. .qoder/settings.json            (项目级,随 git 共享给团队)
3. .qoder/settings.local.json      (项目级本地,gitignore)
```

### 配置格式

```json
{
  "hooks": {
    "事件名": [
      {
        "matcher": "匹配工具名(可省略=全匹配)",
        "hooks": [
          { "type": "command", "command": "脚本路径" }
        ]
      }
    ]
  }
}
```

### matcher 怎么写

- 省略 → 匹配所有工具
- `"Bash"` → 精确匹配
- `"Write | Edit"` → 多个(管道分隔)
- `"mcp__.*"` → 正则(拦截所有 MCP 工具)

> 工具名支持原生(`run_in_terminal`)和 Claude 兼容(`Bash`)两种写法。

---

## 4. 脚本怎么写(exit code 是关键)

脚本是普通 shell 脚本。IDE 通过 **stdin 传 JSON 事件上下文**(通常用 `jq` 解析)。

**exit code 决定动作**:

| exit code | 含义 |
|---|---|
| `0` | 放行,继续执行 |
| `2` | **阻断**(仅可阻断事件),stderr 内容会被注入对话告诉 AI 为啥被拦 |
| 其他 | 非阻断错误(记录但不拦) |

### 最小脚本骨架

```bash
#!/usr/bin/env bash
# 读 stdin 的 JSON 事件上下文
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path // empty')

# 你的检查逻辑
if [[ "$file_path" == *.prod.* ]]; then
  echo "禁止修改生产配置文件: $file_path" >&2
  exit 2   # 阻断,stderr 注入对话
fi

exit 0     # 放行
```

---

## 5. 完整配置示例(Qoder 官方结构)

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          { "type": "command", "command": "~/.qoder/hooks/check-prompt.sh" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.qoder/hooks/block-dangerous.sh" }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": ".qoder/hooks/validate-file-path.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": ".qoder/hooks/auto-lint.sh" }
        ]
      }
    ],
    "PostToolUseFailure": [
      {
        "hooks": [
          { "type": "command", "command": "~/.qoder/hooks/log-failure.sh" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "~/.qoder/hooks/notify-done.sh" }
        ]
      }
    ]
  }
}
```

---

## 6. 推荐落地的 Hook(结合我们体系和理赔实践)

这些是**真正值得做成 Hook** 的——AGENTS.md 管不住或管不好的硬红线:

### 🔴 必做(安全红线,PreToolUse 阻断)

| Hook | 拦什么 | 为什么必须 Hook |
|---|---|---|
| **禁 svn 写操作** | `svn commit/ci/rm/delete` | AGENTS.md 说"禁自动提交"是软的,模型可能偷偷提交;Hook 100% 拦 |
| **禁删 java 源文件** | 删 `*.java` | 防误删生产代码,不可逆 |
| **禁改生产配置** | 写 `*.prod.*` / `application-prod.yml` | 生产配置人工把关 |

### 🟡 推荐(质量机械检查,PostToolUse)

| Hook | 检查什么 | 触发 |
|---|---|---|
| **禁 System.out.println** | Java 文件写入后检测 | 强制用统一日志框架(团队规范) |
| **禁 debugger 语句** | JS 文件写入后检测 | 前端规范 |
| **spec 质量检查** | spec.md 写入后检测行号引用/H1重复/表格空列 | 对应理赔 25 项清单的 A/B/E 类 |

### 🟢 可选(增强)

| Hook | 做什么 |
|---|---|
| **超 3 文件先列清单** | Write/Edit 命中第 4 个文件时,要求先输出变更清单(理赔实践) |
| **完成时强制验证** | Stop 事件,检查是否跑过测试,没跑就不让停 |

### 示例:禁 svn commit 的 Hook

`.qoder/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".qoder/hooks/block-svn-write.sh" }
        ]
      }
    ]
  }
}
```

`.qoder/hooks/block-svn-write.sh`:
```bash
#!/usr/bin/env bash
input=$(cat)
cmd=$(echo "$input" | jq -r '.tool_input.command // empty')

# 拦截 svn 写操作
if echo "$cmd" | grep -qE 'svn\s+(commit|ci|rm|delete|del)'; then
  echo "❌ 禁止 svn 写操作(commit/rm/delete)。SVN 提交必须人工执行。" >&2
  exit 2
fi
exit 0
```

---

## 7. 软约束 vs 硬约束:怎么分工(决策表)

| 关注点 | 软(AGENTS.md/Rule) | 硬(Hook) | 选哪个 |
|---|---|---|---|
| 编码风格、命名 | ✅ 写 Rule | ❌ 太细,Hook 写不过来 | **软** |
| 方法论流程(七阶段) | ✅ AGENTS.md | ❌ 流程不能机械拦 | **软** |
| 禁 svn commit | 🟡 AGENTS.md 说一句 | ✅ Hook 100% 拦 | **硬**(软的兜不住) |
| 禁 System.out | 🟡 规范提一句 | ✅ Hook 写入就查 | **硬**(机械可查) |
| spec 行号引用 | 🟡 规范说"禁" | ✅ Hook 写入就查 | **硬** |
| "先澄清再动手" | ✅ AGENTS.md | ❌ 没法机械判断 | **软** |

**决策原则**:
- **能机械判断的(正则/语法/文件路径)→ Hook**(100% 可靠)
- **需要语义理解的(流程/设计/好坏)→ 软约束**(AI 判断)
- **不可妥协的安全红线 → 两个都写**(软提示 + 硬兜底)

---

## 8. 反模式(Never)

- **Never** 用 Hook 管流程——它判断不了"该不该走某阶段",只会机械拦,用错地方会卡死正常工作
- **Never** Hook 脚本写复杂业务逻辑——脚本要薄(查一下、exit 0/2),复杂逻辑交给 AI/skill
- **Never** 只靠 Hook 不写软约束——Hook 只能拦已知模式,新情况还得 AGENTS.md 引导
- **Never** Hook 脚本不加 `#!/usr/bin/env bash` 和执行权限——会静默不执行,你以为拦住了其实没拦
- **Never** 把项目级 Hook 写进 `~/.qoder/settings.json`(用户级)——会污染其他项目;项目级放 `.qoder/settings.json`

---

## 9. 和我们体系的对应

```
┌─────────────────────────────────────┐
│  AGENTS.md(软·编排)                 │  ← 七阶段流程、subagent 纪律
├─────────────────────────────────────┤
│  Rules(软·约束) + Skills(软·技能)  │  ← 编码规范、方法论
├─────────────────────────────────────┤
│  Hook(硬·机械拦截)                  │  ← 本文档:安全红线、质量检查
├─────────────────────────────────────┤
│  MCP(工具)                          │  ← nacos2-mcp 等
└─────────────────────────────────────┘
```

> Hook 是这套体系里**唯一 100% 确定性的层**。其他层都靠 AI 概率遵守,Hook 不靠。

---

## 参考文档

- [Qoder Hooks 官方文档](https://docs.qoder.com/extensions/hooks.md)
- 真实团队实践:6 条 Hook(svn 拦截/日志强制/spec 质量检查等)
- 我们体系:[AGENTS.md](../AGENTS.md)(软编排)、[Qoder 使用指南](./qoder-使用指南.md)(配置落点)
