---
name: git-commit
description:
  智能提交：有暂存文件则只提交暂存内容，否则提交所有已追踪的改动。提交信息用 Conventional Commits（默认英文），类型/scope/粒度与仓库习惯保持一致，只提交、不推送。
argument-hint: "[可选的补充说明，会作为提交信息的参考]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git log:*)
---

# Git 提交

按下面的规则提交当前仓库的改动。**只提交，不要 push、不要建/切分支、不要改 git config。**

## 当前 Git 状态

- 分支与状态：!`git status --short --branch`
- 已暂存文件：!`git diff --cached --name-only`
- 已暂存改动概览：!`git diff --cached --stat`
- 工作区未暂存改动概览：!`git diff --stat`
- 最近 10 条提交（判断仓库的类型、scope 与语言习惯）：!`git log --oneline -10`

## 1. 提交范围

1. **已有暂存文件**：用户已经选好范围，只提交这些文件，不要再 `git add` 任何东西。
2. **没有暂存文件**：执行 `git add -u`，暂存所有**已追踪**文件的修改和删除；新增文件用 `git add <path>` 显式加入。不要因为"看起来无关"就把改动排除在外，也不要检查 `.gitignore`——被忽略的文件本来就不会进索引。
3. **提交前复核**：`git diff --cached` 过一遍，确认
   - 没有密钥、凭据、`.env`、私钥；
   - 没有 sub-agent 留下的分析报告、临时数据（有就先删掉再提交）；
   - 变更都在预期内，提交信息能覆盖全部改动。
4. **暂存与提交连着做**：用 `git add ... && git commit ...` 串成一条命令，中间不要插入提问或其他操作，避免期间的新改动漏提交。

## 2. 提交信息

格式遵循 [Conventional Commits](https://www.conventionalcommits.org/)：

```text
<type>(<scope>): <summary>

<body>

<footer>
```

- **语言**：默认英文；只有仓库历史明显全中文时才跟随中文。技能正文用中文不影响提交信息用英文。
- **summary**：命令式（add / fix / remove，不用 added / fixes）、≤ 72 字符、结尾不加句号。
- **scope**：可省略；优先用仓库里已经出现过的 scope（例如 `fix(hermes)`、`feat(niri)`），不要临时发明新词。
- **body**：只写重点，**不超过 10 个非空行**，能一句话说清就不写（见下方 body 规则）。
- **footer**：`Closes #123`、`Refs: #456` 这类 `Token: value` 形式。
- **破坏性变更**：`feat(api)!: ...` 加 `BREAKING CHANGE:` footer。

### body 规则

- **只写重点**：为什么改、关键取舍、影响面（哪些行为或模块受影响）。不复述 summary，不逐文件罗列"改了哪个文件"。
- **不写验证过程**：不写"测试通过""已验证""跑了某命令"之类；验证结果放在最后的汇报里，不写进提交信息。
- **简洁**：短句；需要列举时用 `-` 列表；不写空话、"本 PR 将……"这类铺垫。
- **长度**：正文不超过 **10 个非空行**（段间空行不计）。写超了说明 summary 没提炼够，先压缩；只有确实压不下去时才例外，例如：
  - 破坏性变更的迁移步骤；
  - 删除就会被误解的关键设计取舍；
  - 必须完整的复现或回滚步骤。
- **可省略**：一句话能说清的改动不写 body。

类型：

| type | 用途 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 修 bug |
| `docs` | 只改文档 |
| `style` | 只改格式，不动逻辑 |
| `refactor` | 既不是新功能也不是修复的代码改动 |
| `perf` | 性能优化 |
| `test` | 新增或修改测试 |
| `build` | 构建系统或依赖 |
| `ci` | CI 配置 |
| `chore` | 其他杂务（发布、清理、配置微调） |
| `revert` | 回滚之前的提交 |

**仓库约定优先**：仓库已有 commitlint/`CONTRIBUTING.md`/最近的提交风格与本节冲突时，按仓库来。但 summary 不要加 `[verified]`、`[wip]` 这类状态前缀——验证或进度状态不属于提交信息。

反例（不要写）：

- `chore : rm 3 skills not used` —— 冒号前多了空格；
- `update stuff` —— 没有类型、没有信息量；
- `fix: 修复了 xxx。` —— 中文 + 句号结尾，和仓库历史不一致；
- `[verified] fix(php): ...` —— 标题带状态前缀，验证状态不进提交信息；
- 三十行的 body 逐文件列改动、附带测试命令和输出 —— 这些属于汇报内容，不属于提交信息。

## 3. 提交粒度

| 场景 | 做法 |
| --- | --- |
| 互相独立的修复 / 功能 / 文档 | 各自单独 commit |
| 同一问题的代码 + 指令 + 文档（含 skill） | 合并为一个 commit |
| 删除文件与其关联的代码改动 | 合并为一个 commit |
| skill/md 文档与触发它的代码变更 | 合并为一个 commit |
| 实现过程中发现规范要改（如本 skill） | 与触发这次更新的代码变更合并 |
| 对同一处的反复修补（未推送） | `git commit --amend` |
| 合并多个连续 commit（未推送） | `git reset --soft HEAD~n` 后重新 commit |
| 合并非连续 commit（未推送） | 交互式 `git rebase -i`，需要用户明确同意 |

动历史（amend / reset --soft / rebase -i）只在**未推送**且用户同意时使用；已经推送的提交一律不做历史改写。

## 4. 提交时机

- **用户消息就是提交指令**（"提交"、"commit 一下"、"先提交"）：直接提交一次，不要再问。
- **完成一个任务**：直接提交（本仓库约定：改完只提交，不推送），不需要额外确认。
- **需要动历史、或提交范围有歧义**（例如工作区里明显混着两项独立工作）：先问用户，再执行。
- **没有可提交的内容**：直接说明"没有可提交的改动"并停止，不要造空提交。
- **合并冲突 / rebase / merge 进行中**：停止并提示用户先处理。
- **不要凭记忆写 message**：以 `git diff --cached` 为准。

## 5. 安全边界

- 不 push、不建/切分支、不改 `git config`、不使用 `--no-verify`。
- hook 失败时：修好问题、重新 `git add`、重跑同一条 `git commit`；不要用 `--amend` 绕过 hook。
- 只有用户明确要求时才用 `git reset --hard`；执行前先 `git status` 确认，有未提交改动就先 `git stash`，事后 `git stash pop`；工作区干净时不要 stash。
- 不要清理或排除用户/外部工具产生的改动，除非用户明确要求。

## 6. 完成后汇报

提交成功后运行 `git log --oneline -1` 和 `git status --short`，向用户简要汇报：提交哈希、提交信息首行、本次提交了哪些文件。

## 用户补充说明

$ARGUMENTS
