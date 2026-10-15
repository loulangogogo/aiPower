# aiPower 仓库作业约定

本文件写的是**改动这个仓库时必须遵守的约定**。项目背景、设计思路、安装方式见 `README.md`，此处不重复。

一句话概括本仓库：一套 Claude Code 配置，**每个目录都有中英两份**，两个包可以整包互换。

---

## 一、中英文联动（最重要）

### 1.1 配对关系

| 中文包 | 英文包 |
| --- | --- |
| `agents-zh/` | `agents/` |
| `commands-zh/` | `commands/` |
| `rules-zh/` | `rules/` |
| `skills-zh/` | `skills/` |

四个目录各自成对，**每对内部一一对应，没有多余也没有缺失**。

### 1.2 必须完全相同的项

同一份文档在两包中，以下内容**逐字节相同**：

- **文件名**：`code-java.md` ↔ `code-java.md`（不是 `code-java-zh.md`）
- **目录内相对路径**：`skills-zh/git-commit/SKILL.md` ↔ `skills/git-commit/SKILL.md`
- **frontmatter 的 `name`**：两边都是 `code-java`，**永不翻译、永不加后缀**
- **frontmatter 的 `tools`、`model`、`color`、`disable-model-invocation`** 等所有非语言字段
- **章节结构**：标题层级、章节顺序、数量
- **表格列结构与行数**
- **代码块内容**（命令、路径、变量名）
- **emoji 标记**（🔴🟠🟡🔵 这类分级符号）
- **交叉引用**：中文版说「委派 `code-review`」，英文版必须说 "dispatch `code-review`"，指向同一个名字

### 1.3 唯一允许不同的项

**只有正文的自然语言**。中英两份的差异应当且仅当是语言本身。

### 1.4 硬规则：禁止单向改动

**改动任一侧后，必须同步改动另一侧。** 这是本仓库最强的约束，没有例外。

- 改中文 → 同步改英文
- 改英文 → 同步改中文
- 只改一侧就收工，视为未完成任务

同步时保持**结构对齐**：中文版删了一个小节，英文版对应位置也要删；中文版表格加了一行，英文版同位置也要加一行。

### 1.5 三类「故意不同」的内容

以下是**设计如此**，不是漏翻，不要「修好」它们：

| 类别 | 中文包 | 英文包 |
| --- | --- | --- |
| 输出语言条款 | 「全程使用中文」「所有对话…必须使用中文」 | "Write entirely in English" |
| 提交信息 subject | 「`subject` 用中文」 | "`subject` is written in English" |
| 报告语言 | 「输出中文报告」 | "print a report" |

改动这些条款前，先确认是有意调整语言策略，而不是顺手统一。

---

## 二、改完必做的校验

任何涉及两包的改动，收工前跑一遍：

```bash
python3 - <<'EOF'
import os, re
# 跳过 .DS_Store 等隐藏文件，否则 macOS 会产生假告警
keep = lambda fs: [f for f in fs if not f.startswith(".")]
for zh, en in [("agents-zh","agents"),("commands-zh","commands"),
               ("rules-zh","rules"),("skills-zh","skills")]:
    rel = lambda root: {os.path.relpath(os.path.join(d,f), root)
                        for d,_,fs in os.walk(root) for f in keep(fs)}
    a, b = rel(zh), rel(en)
    flag = "一致" if a == b else f"不一致 {a ^ b}"
    print(f"{zh} <-> {en}: 文件名{flag}")
    for r in sorted(a & b):
        tz = open(os.path.join(zh,r), encoding="utf-8").read()
        te = open(os.path.join(en,r), encoding="utf-8").read()
        nz = re.search(r"^name:\s*(\S+)", tz, re.M)
        ne = re.search(r"^name:\s*(\S+)", te, re.M)
        if nz and ne and nz.group(1) != ne.group(1):
            print(f"  !! {r} name 不一致: {nz.group(1)} vs {ne.group(1)}")
        cjk = len(re.findall(r'[一-鿿]', te))
        print(f"  {r:28} 中文{len(tz.splitlines()):>4}行  英文{len(te.splitlines()):>4}行"
              + (f"  英文包残留汉字 {cjk}" if cjk else ""))
EOF
```

**三项通过标准：**

1. 文件名集合两侧一致
2. 每对的 `name` 字段相同
3. **英文包汉字残留为 0**（这是漏翻的硬信号）

行数不必强求相同，但**差异超过 5 行就要能说清原因**（通常是某侧多了一段说明）。

---

## 三、新增一个语言智能体

1. `agents/` 与 `agents-zh/` 各建一个文件，**文件名相同**，例如 `code-go.md`；`name: code-go` 两侧一致。
2. frontmatter 直接照抄现有语言版，只改 `name`、`description`、`color`：

   ```yaml
   ---
   name: code-go
   description: <角色 + 覆盖范围 + 使用场景 + 委派说明 + 权限声明>
   tools: Read, Grep, Glob, Bash, Edit, Write, Agent
   model: inherit
   color: <未占用的颜色>
   ---
   ```

3. 正文沿用**九个章节**的骨架：

   ```
   角色定位 → 工作范围 → 与 code-review 子智能体的协作 → 工作流
   → <语言>技术栈关注点 → 代码质量底线 → 验证要求 → 输出格式 → 纪律
   ```

4. **只重写第 5 节**。前四节、后四节在两包之间逐字复用，只换语言。
5. 跑第二节的校验脚本。

已占用的颜色：`cyan`(review) `orange`(java) `green`(c) `purple`(python) `blue`(ts) `yellow`(js) `pink`(front)。

---

## 四、仓库约束

- **`name` 必须全树唯一。** 子智能体身份只由 frontmatter 的 `name` 决定，与文件名、子目录无关。同名时项目级压过用户级，低优先级的被**静默丢弃**。
- **`tools` 里要写 `Agent`。** 省略 `tools` 字段才默认继承全部工具；一旦显式列出清单，不写 `Agent` 就完全无法派发子智能体。`code-review` 系列**故意不含** `Agent`，以保持只读隔离。
- **两个语言包不能同时部署**到同一个 `~/.claude/`。同名定义会互相覆盖。
- **本仓库不放任何凭证。** 只放规则、智能体、命令定义。`settings.json`、`*.local.json`、`.env` 之类永远不提交。

---

## 五、禁止行为

- 只改一侧、不更另一侧（本仓库头号禁令）
- 给 `name` 加 `-zh` 后缀或翻译 `name`
- 把「故意不同」的三类条款当成漏翻改掉
- 把两个包合并到同一个部署目录
- 重命名顶层目录（`agents-zh/` 这种命名是刻意设计，不是笔误）
- 在仓库内写入 API key、token 或任何凭证

---

## 六、文档分工

| 文件 | 读者 | 内容 |
| --- | --- | --- |
| `README.md` | 人 | 项目简介、设计思路、安装方式、已知约束 |
| `CLAUDE.md`（本文件） | Claude | 改动约定、联动规则、校验方法 |
| 各 agent / command / skill | Claude | 各自的职责定义 |

改动**仓库结构或安装方式**时更新 `README.md`；改动**作业约定**时更新本文件。两份文档的中英版本——本仓库暂无 `CLAUDE.md` 的英文版，`README.md` 也无——如需补充按第一节的联动规则处理。
