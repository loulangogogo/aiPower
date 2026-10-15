# aiPower

Claude Code 配置仓库：一套**子智能体 + 命令/技能 + 规则**，中英双语，可整包替换。

## 这个仓库解决什么问题

Claude Code 的默认行为是「一个通用助手对什么语言都懂一点」。这个仓库把它变成**一支分工明确的小队**：

- 写 Java 时，调的是懂 Spring 事务失效、`ConcurrentHashMap` 先查后放、虚拟线程钉住问题的 Java 工程师；
- 写 C 时，调的是对缓冲区边界、`realloc` 失败语义、未定义行为敏感的系统工程师；
- 需要全项目体检时，有一个**只读的专职审查者**，不碰代码、只出分级报告。

同时把日常重复动作（提交、推送、编码协作规范）固化成命令与规则，避免每次重新解释。

---

## 目录结构

```
aiPower/
├── agents/          英文包子智能体（7 个）
├── agents-zh/       中文包子智能体（7 个）      ← 与 agents/ 一一对应
├── commands/        英文包斜杠命令（2 个）
├── commands-zh/     中文包斜杠命令（2 个）
├── rules/           英文包全局规则（3 个）
├── rules-zh/        中文包全局规则（3 个）
├── skills/          英文包技能（2 个）
└── skills-zh/       中文包技能（2 个）
```

### 为什么每个目录都有两份

四个目录各自成对，**每对内部的文件名和标识完全相同，只有正文语言不同**：

| 中文包 | 英文包 | 文件名 | `name` 字段 |
| --- | --- | --- | --- |
| `agents-zh/code-java.md` | `agents/code-java.md` | 相同 | 相同（`code-java`） |
| `commands-zh/git-commit.md` | `commands/git-commit.md` | 相同 | — |
| `rules-zh/Code.md` | `rules/Code.md` | 相同 | — |
| `skills-zh/git-commit/SKILL.md` | `skills/git-commit/SKILL.md` | 相同 | 相同（`git-commit`） |

这样设计是为了让两个包成为**可直接互换的整体**：装了中文包，agent 叫 `code-java`；换成英文包，agent 还是叫 `code-java`。区别只在它用中文还是英文跟你说话。

> ⚠️ **两个包不能同时装。** 同名定义在同一棵树里会互相覆盖，且是**静默丢弃**——不报错、不提示，只是有一个不生效。见「已知约束」。

---

## 安装

### 部署路径对照

Claude Code 只从固定位置读取配置，**不认识 `-zh` 后缀的目录名**。所以同步时要做名字映射：

| 本仓库目录 | 部署目标（用户级） | 部署目标（项目级） |
| --- | --- | --- |
| `agents/` 或 `agents-zh/` | `~/.claude/agents/` | `<项目>/.claude/agents/` |
| `commands/` 或 `commands-zh/` | `~/.claude/commands/` | `<项目>/.claude/commands/` |
| `rules/` 或 `rules-zh/` | `~/.claude/rules/` | `<项目>/.claude/rules/` |
| `skills/` 或 `skills-zh/` | `~/.claude/skills/` | `<项目>/.claude/skills/` |

**一次只同步一个包。** 想切换语言，先清空目标目录再同步另一个包。

### 方式一：复制（最稳妥）

```bash
# 安装中文包（把 zh 后缀的目录内容复制到标准位置）
cp -R rules-zh/.      ~/.claude/rules/
cp -R commands-zh/.   ~/.claude/commands/
cp -R skills-zh/.     ~/.claude/skills/
cp -R agents-zh/.     ~/.claude/agents/
```

### 方式二：符号链接

各目录的官方支持程度不同：

| 目录 | 符号链接支持 |
| --- | --- |
| `rules/` | ✅ 官方明确支持，文档最完整 |
| `skills/` | ✅ 支持（`<skill-name>` 这一层条目可为 symlink） |
| `agents/` | ⚠️ 文档未提及，文件系统层面可用但无官方背书 |
| `commands/` | ⚠️ 文档未提及 |

两个必须知道的坑：

1. **指向工作目录之外的 symlink 会被当作「外部导入」，默认不加载**，需要逐项目在会话开始时批准一次。唯一的例外是放在 `~/.claude/rules/` 里的规则——官方原话是放这里可以免批准、且作用于所有项目。
2. **agents 目录的热加载只覆盖会话启动时已存在的目录。** 所以必须先建好 symlink、再启动会话；会话启动后才创建的 `agents/` 目录需要重启才能被识别（新建 scope 的第一个 agent 文件同理）。

---

## 子智能体（agents）

7 个智能体，每个包一份，职责互不重叠。

| 名称 | 角色 | 工具权限 | 颜色 |
| --- | --- | --- | --- |
| `code-review` | 多语言代码审查专家。**只读**，输出按严重程度排序的结构化报告 | `Read, Grep, Glob, Bash` | cyan |
| `code-java` | Java 高级开发工程师 | `Read, Grep, Glob, Bash, Edit, Write, Agent` | orange |
| `code-c` | C 高级开发工程师 | 同上 | green |
| `code-python` | Python 高级开发工程师 | 同上 | purple |
| `code-ts` | TypeScript 高级开发工程师 | 同上 | blue |
| `code-js` | JavaScript 高级开发工程师 | 同上 | yellow |
| `code-front` | 前端高级开发工程师（Vue / React / HTML / CSS） | 同上 | pink |

### 设计要点

**分工按「故障源」而不是「语法」划分。** 每个语言版都有一节专项关注点，写的是那个语言真正会出事的地方：

- `code-java`：`@Transactional` 自调用失效、`ConcurrentHashMap` 先查后放、虚拟线程被 `synchronized` 钉住、JPA 的 N+1
- `code-c`：`realloc` 失败时原指针仍有效（`p = realloc(p, n)` 直接泄漏）、`strncpy` **不保证** NUL 终止、未定义行为会被优化器利用
- `code-python`：`async def` 里调同步阻塞函数会卡死整个事件循环、可变默认参数跨调用共享、`open()` 不指定 `encoding` 在 Windows 上是 GBK
- `code-ts`：类型只存在于编译期、对接口响应做 `as SomeType` 等于没校验、`??` 与 `||` 的差别
- `code-js`：没有类型系统兜底、微任务先于宏任务、同步阻塞事件循环、依赖供应链风险
- `code-front`：Vue 2 与 Vue 3 的响应式机制不同、React Hooks 依赖数组、`z-index` 失效的层叠上下文真因、`v-html` / `dangerouslySetInnerHTML` 的 XSS

**工作流里都有一道「先摸清项目环境」的前置。** 因为版本差异会让同一段代码完全失效——C 的语言标准、Python 的 `requires-python`、Vue 2/3、React 17/18/19、`tsconfig` 的 `strict` 开关。读错版本写出的代码在项目里根本不成立。

**质量底线是共通的**：最小改动、不擅自升级依赖或改公共接口、不引入新框架、改完必须验证且**区分「已编译通过 / 已运行测试通过 / 未验证」**，绝不声称没跑过的验证。

### 协作链路

```
主会话
  ├── code-java / code-c / code-python / code-ts / code-js / code-front   （读写，可派发）
  │        └── code-review                                                （只读，不可再派发）
  └── code-review                                                          （也可直接调用）
```

**为什么语言版要委派给 code-review：** 局部精读、功能实现、缺陷修复是开发者的活；**全库扫描、跨模块影响面分析、正式分级报告、安全审计**是审查者的活。语言版正文里明确划了这条界线。

**机制上的两个事实**（都影响你怎么配置）：

1. `code-review` 的 `tools` 里**不含 `Agent`**，所以它天然无法再派发下级——这是官方的只读隔离写法。
2. 语言版的 `tools` 里**必须显式写上 `Agent`**。省略 `tools` 字段才会默认继承全部工具；一旦显式列出工具清单，不写 `Agent` 就完全无法派发子智能体。这是最常见的坑。
3. 「委派给 code-review」是靠**正文的自然语言指令**引导的，不是硬约束。frontmatter 里的 `Agent(agent_type)` 指名白名单**只在 `claude --agent` 主线程模式下生效**，写在子智能体定义里括号内容会被静默忽略。

嵌套层级为 2 层，在默认上限（3 层）之内。

---

## 命令与技能

| 名称 | 作用 |
| --- | --- |
| `/git-commit` | 同步远端 → 检查中间状态 → 列出变更供确认 → 生成 Conventional Commits 规范提交信息 → 提交 |
| `/git-push` | 列出变更供勾选 → 提交 → rebase 同步远程 → 安全推送；冲突则回滚并输出报告 |

两者都贯彻同一条安全原则：**提交范围由用户确认，绝不自行拍板。** 明令禁止 `git add .`、`--force`、`--no-verify`、rebase 冲突时自行解冲突等操作。

### ⚠️ 这里有个需要你决定的问题：命令与技能同名

官方机制确认：**skill 与 `.claude/commands/` 下的同名文件不会产生两个命令，skill 胜出，命令文件被静默遮蔽。**

本仓库里 `commands/git-commit.md` 与 `skills/git-commit/SKILL.md` 同名，`git-push` 同理。但两个版本内容**机制不同**：

| | 命令版（`commands/`） | 技能版（`skills/`） |
| --- | --- | --- |
| 确认控件 | `AskUserQuestion`，二选一「全部提交 / 取消」 | `ask_user_question`，多选逐文件勾选 |
| 选项数限制 | 每次 2–4 个 | 每文件一个选项 |
| 子代理场景 | 未提及 | 提到 `DELEGATED_CALLER` 回退 |

**所以实际生效的只有技能版。** 如果你本来期望的是命令版的「全部提交 / 取消」交互，那它不会被执行。三选一：

- 接受现状，删掉 `commands/` 下的两个文件；
- 保留两者但改名（例如命令版叫 `/git-commit-all`），让它们共存；
- 确认命令版是给别的宿主用的，那就维持现状并在本 README 注明。

---

## 规则（rules）

| 文件 | 内容 |
| --- | --- |
| `All.md` | 总要求：先调用相关 skill；不修改当前目录之外的文件 |
| `Code.md` | 编码协作规范：编码前思考、简洁优先、精准修改、目标驱动执行、交流规范、注释规范、可读性规范、禁止行为 |
| `Python.md` | 环境要求：优先 `python3`、有限使用 `pip3`、禁止默认 `python` / `pip` |

`~/.claude/rules/` 是**官方支持的用户级规则目录**，不是自定义配置。几个值得知道的机制：

- 无 `paths` 字段的规则**启动即加载**，优先级与 `.claude/CLAUDE.md` 相同；
- 有 `paths` 字段的规则**只在 Claude 读写命中 glob 的文件时才加载**（`cat`、`head` 这类算读取的 Bash 命令也算）；
- `paths` 是规则唯一会被读取的 frontmatter 字段，**其他字段静默忽略**；
- 用户级规则先于项目级加载，两者**不互相覆盖**——冲突时 Claude 可能遵循任意一个，所以要保持一致。

---

## 命名约定与扩展

### 约定

| 项目 | 规则 |
| --- | --- |
| 目录 | 无后缀 = 英文包；`-zh` 后缀 = 中文包 |
| 文件名 | 两个包**完全一致**（`code-java.md`，不是 `code-java-zh.md`） |
| `name` 字段 | 两个包**完全一致**（`code-java`），且必须全树唯一 |
| 语言版 `color` | 每种语言一个颜色，两包一致 |

### 新增一个语言智能体

1. 在 `agents/` 与 `agents-zh/` 各建一个文件，**文件名与 `name` 相同**，例如 `code-go.md` / `name: code-go`。
2. frontmatter 直接复用现有语言版：

   ```yaml
   ---
   name: code-go
   description: <角色 + 覆盖范围 + 使用场景 + 委派说明 + 权限声明>
   tools: Read, Grep, Glob, Bash, Edit, Write, Agent
   model: inherit
   color: <未占用的颜色>
   ---
   ```

3. 正文沿用九个章节的骨架，**只重写「技术栈关注点」一节**——那是每个语言真正不同的部分：
   `角色定位` → `工作范围` → `与 code-review 子智能体的协作` → `工作流` → **`<语言>技术栈关注点`** → `代码质量底线` → `验证要求` → `输出格式` → `纪律`

4. 除「技术栈关注点」外，其余八节在中英两版之间逐字复用，只换语言。

---

## 已知约束

**1. `name` 必须全树唯一。** 子智能体的身份**只由 frontmatter 的 `name` 决定**，与文件名、子目录无关。同名时高优先级位置的定义胜出（项目级 `.claude/agents/` > 用户级 `~/.claude/agents/`），低优先级的被**静默丢弃**。

**2. 两个语言包不能同时部署。** 见上。切换语言要清空目标目录再同步。

**3. 本仓库不含任何凭证。** 只放规则、智能体、命令定义。`settings.json`、`*.local.json`、`.env` 之类的文件**永远不要提交**——配置文件里的 API key 泄漏是不可逆的。`.gitignore` 已忽略 `.mcp.json`、`CLAUDE.md`、`/CLAUDE.md` 等，建议按需补充。

**4. `rules/` 与 `CLAUDE.md` 是「上下文」而非强制配置。** 官方原话：Claude 把它们当作 context，不是 enforced configuration。需要确定性行为应该用 hooks 或 permissions。

**5. 子智能体目录支持递归扫描，但子目录不影响身份。** 可以整理成 `agents/review/foo.md`，但 `name` 仍须全树唯一。

---

## 参考

- 子智能体：<https://code.claude.com/docs/en/sub-agents>
- 技能：<https://code.claude.com/docs/en/skills>
- 记忆与规则：<https://code.claude.com/docs/en/memory>
- `.claude/` 目录总览：<https://code.claude.com/docs/en/claude-directory>
