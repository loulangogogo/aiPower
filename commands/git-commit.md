---
name: git-commit
description: 智能生成符合规范的 Git 提交信息并提交代码变更
---

# Git Commit 命令

## 功能描述

此命令用于智能分析当前工作区的代码变更，自动生成符合 Conventional Commits 规范的提交信息，并完成 `git add`、`git commit` 操作。

## 工作流程

1. **检查 Git 状态**
   - 执行 `git status` 查看当前变更文件列表
   - 确认是否有未暂存的更改

2. **分析变更内容**
   - 读取已暂存（staged）和未暂存（unstaged）的文件差异
   - 识别变更类型（feat、fix、docs、style、refactor、perf、test、chore、build、ci、revert 等）

3. **生成提交信息**
   - 按照以下格式生成提交信息：
     ```
     <type>(<scope>): <subject>
     
     <body>
     
     <footer>
     
     ```
   - 示例：
     ```
     feat(auth): 添加用户登录验证功能
     
     实现了基于 JWT 的用户认证机制，包括：
     - 登录接口开发
     - Token 生成与验证
     - 过期时间管理
     
     Closes #123
     
     ```

4. **确认并提交**
   - 向用户展示生成的提交信息
   - 无需用户确认或修改
   - 执行 `git add .` 和 `git commit -m "<message>"`

## 注意事项

- 确保当前目录是一个有效的 Git 仓库
- 如果存在未跟踪的文件，建议先通过 `.gitignore` 排除
- 提交信息必须符合 [Conventional Commits](https://www.conventionalcommits.org/) 规范
- 支持中文提交信息

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
