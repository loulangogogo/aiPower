---
description: 分析当前工作区变更，生成符合 Conventional Commits 规范的中文提交信息并完成提交
---

# /git-commit 命令

本命令仅在用户显式执行 `/git-commit` 时运行，不被模型主动调用。收到该命令后按下述步骤依次执行，不要跳步，也不要凭印象直接提交。

## 执行流程

### 1. 同步远端更新

先确认在 Git 仓库内，再拉取当前分支的上游更新。**远端不存在当前分支时跳过拉取**，直接进入下一步。

```bash
git rev-parse --is-inside-work-tree    # 失败 → 报告「当前目录不是 Git 仓库」并终止
git fetch --prune                      # 失败（无远程 / 网络不可达）→ 立即中止

if upstream=$(git rev-parse --abbrev-ref --symbolic-full-name '@{u}' 2>/dev/null); then
  git pull                             # 使用默认 merge 策略
else
  echo "远端不存在当前分支，跳过拉取"
fi
```

- 判定「远端不存在当前分支」的两种情况：未关联上游分支；上游分支已在远端被删除（`git branch -vv` 显示 `gone`）。必须先 `git fetch --prune` 再判定，否则本地残留的过期远程引用会让判断失真。
- `git pull` 采用默认 merge 策略，因此本地落后于远端时会引入一个合并提交，这是预期行为。
- 拉取失败（网络不可达、本地改动会被覆盖、合并冲突）→ **立即中止整个提交流程**，把 Git 的原始输出转述给用户，不执行任何自动恢复。若失败后处于合并中间状态，只提示用户可自行执行 `git merge --abort` 回到拉取前的状态，不要代劳。

### 2. 检查变更与中间状态

```bash
git status --porcelain
git diff --check
gitdir=$(git rev-parse --git-dir); ls "$gitdir"/MERGE_HEAD "$gitdir"/CHERRY_PICK_HEAD "$gitdir"/REVERT_HEAD "$gitdir"/rebase-merge "$gitdir"/rebase-apply 2>/dev/null
```

- `git status --porcelain` 无输出 → 报告「当前无代码变更，无需提交。」并终止。
- `git diff --check` 报出冲突标记 → 列出相关文件并终止。
- 最后一条命令有输出 → 正处于 merge / rebase / cherry-pick / revert 中间状态，立即终止并提示先完成或中止该操作。中途提交会把冲突解决过程伪装成一次普通提交。

### 3. 阅读差异并确定提交范围

```bash
git diff           # 未暂存改动
git diff --cached  # 已暂存改动
```

逐个文件判断：哪些属于本次要提交的逻辑变更，哪些是调试残留、临时文件或与本次任务无关的改动。

**提交范围由用户决定，不要自行拍板。** 先把候选清单完整写在回复正文里，再用 `AskUserQuestion` 做一次确认。

清单逐行给出 `<状态> <文件路径>`，状态翻译为：新增 / 修改 / 删除 / 未跟踪（将被新增）/ 重命名。

- **未跟踪文件必须单独标注**：临时脚本、密钥、构建产物最容易在这一步被误提交。
- 疑似敏感文件（`.env`、`*.pem`、`id_rsa`、`credentials*` 等）照常列出，但标注「⚠️ 疑似密钥」并排在最后。
- 清单末尾附一行统计：`共 N 个文件，+X / −Y 行`（行数由 `git diff --stat HEAD` 汇总，不含未跟踪文件）。
- 不主动粘贴完整 diff；用户当场要求时再展示。

`AskUserQuestion` 每次只接受 **2–4 个选项**，所以**不要为每个文件生成一个选项**。用二选一发起确认：

- 问题文本：「以上 N 个文件是本次要提交的内容吗？」
- 选项：`[全部提交]` / `[取消]`

用户需要只提交其中一部分时，会通过该工具自带的 Other 输入路径。以用户给出的路径为准；若 Other 里提到清单之外的文件，先核实它确实有变更再纳入，否则忽略并说明。

分支处理：

- **全部提交** → 进入第 4 步。
- **取消**，或用户指出清单有误 → **整个命令中止**，不执行任何 `git add` / `git commit`，输出「已取消，未提交任何文件。」并把控制权交回用户。
- **无人应答或交互不可用** → 同样中止（fail closed），**绝不默认继续**。

拿到确认的文件后逐路径暂存（用 `--` 分隔并加引号，兼容空格、中文与特殊字符；重命名条目按新路径传入）：

```bash
git add -- "<file1>" "<file2>"
```

不要用 `git add .`、`git add -A` 或 `git commit -a`：它们会**直接绕过刚刚的用户确认**，把无关改动、临时文件和密钥一并写进历史。

若 `AskUserQuestion` 工具不可用，或调用失败 → **不要退化成「自己决定并直接提交」**：改为在正文中完整列出候选文件及其状态，明确请求用户确认，等用户回复后再继续。

### 4. 确认暂存区非空

```bash
git diff --cached --stat
```

无输出说明第 3 步没有暂存任何内容，终止，不要创建空提交。

### 5. 生成提交信息

按「提交信息格式」一节生成，并在提交前把最终提交信息展示给用户，作为本次变更的记录（要提交的文件已在第 3 步确认）。

### 6. 校验提交者身份并提交

身份缺失时 Git 会拒绝提交，或写入错误的作者信息，因此提交前先确认：

```bash
git config user.name
git config user.email
```

任一为空 → 报告需要的 `git config user.name "..."` 与 `git config user.email "..."` 命令并终止。

```bash
# 仅标题
git commit -m "<type>(<scope>): <subject>"

# 正文为单段时，每个 -m 生成一个独立段落
git commit -m "<type>(<scope>): <subject>" -m "<body>" -m "<footer>"

# 正文包含多行列表时用 heredoc，避免 -m 中的换行转义出错
git commit -F - <<'EOF'
<type>(<scope>): <subject>

<body>

<footer>
EOF
```

hook 失败或提交报错时：**保留暂存区原样**，原样转述 Git 的输出，不要用 `--no-verify` 绕过，也不要擅自 `git reset` 撤销用户的暂存内容。

### 7. 报告结果

提交成功后输出：

```text
已提交 <短哈希>：<type>(<scope>): <subject>
变更：新增 N 个 / 修改 M 个 / 删除 K 个
```

## 提交信息格式

```text
<type>(<scope>): <subject>

<body>

<footer>
```

- `scope` **可选**，表示影响范围（如 `auth`、`api`、`ui`）；没有明确范围时省略括号，写成 `feat: 添加用户登录模块`。
- `subject` 用中文，一句话说明做了什么，不超过 50 字符，结尾不加句号。
- `body` 说明「为什么改」与「改了什么」，可分行列举；无正文时整段省略。
- `footer` 用于 `Closes #123`、`Refs #45` 等关联信息；无脚注时整段省略。
- 破坏性变更：在 type 或 scope 后加 `!`（如 `feat(api)!: 移除 v1 接口`），并在 footer 写明 `BREAKING CHANGE: <影响说明>`。
- 回滚提交：标题写 `revert: feat(auth): 添加用户登录验证功能`，正文写 `This reverts commit <被回滚的哈希>`。

示例：

```text
feat(auth): 添加用户登录验证功能

实现基于 JWT 的用户认证机制：
- 新增登录接口
- Token 生成与校验
- 过期时间管理

Closes #123
```

## 支持的 Type 类型

| Type       | 说明                                   |
| ---------- | -------------------------------------- |
| `feat`     | 新功能                                 |
| `fix`      | 修复 bug                               |
| `docs`     | 文档变更                               |
| `style`    | 代码格式（不影响代码运行）             |
| `refactor` | 重构（既不是新增功能，也不是修复 bug） |
| `perf`     | 性能优化                               |
| `test`     | 测试相关变更                           |
| `chore`    | 构建过程或辅助工具变动                 |
| `build`    | 构建系统或外部依赖变更                 |
| `ci`       | CI 配置变更                            |
| `revert`   | 回滚提交                               |

[Conventional Commits](https://www.conventionalcommits.org/) 规范本身只规定 `feat`（新功能）与 `fix`（修复）的语义，其余类型沿用 Angular 约定，可按项目需要扩展。

## 禁止事项

- 不用 `git add .`、`git add -A`、`git commit -a` 全量暂存
- 不用 `--no-verify` 绕过 hook，不用 `--amend` 改写已有提交
- 不在 merge / rebase / cherry-pick 中间状态提交
- 不提交未经确认的敏感文件与无关改动
- 提交失败后不擅自 `git reset` 撤销用户的暂存内容
- 拉取失败后不执行任何自动恢复（`git merge --abort`、`git reset`、`git stash` 等），只报告并给出建议命令
