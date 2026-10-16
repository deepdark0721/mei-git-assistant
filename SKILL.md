---
name: mei-git-assistant
description: "Git操作助手——核心概念、协作流程、分支策略、冲突解决、AI结合Git操作、安全红线。触发词：git/GitHub/版本控制/分支/合并/冲突解决/PR/pull request/commit/push/pull/clone/fork/rebase/merge/冲突/提交代码/推送代码"
triggers:
  - git
  - Git
  - GitHub
  - 版本控制
  - 分支
  - 合并
  - 冲突解决
  - PR
  - pull request
  - commit
  - push
  - pull
  - clone
  - fork
  - rebase
  - merge
  - 冲突
  - 提交代码
  - 推送代码
  - git操作
version: "2.2.0"
complexity: 中
estimated_time: 5-15分钟
---

# mei-git-assistant

Git操作助手——核心概念、协作流程、分支策略、冲突解决、AI结合Git操作、安全红线。

---

## 设计哲学

### 原则1：先看后动

任何 Git 操作前，先 `git status` 确认当前状态。
不知道当前状态就执行操作 = 盲人过马路。

### 原则2：小步快跑

一次提交只做一件事，小步提交比大步合并安全100倍。
提交历史 = 项目的可回溯记忆，每次提交都是一个检查点。

### 原则3：分支隔离

永远不要在 main/master 上直接开发。
功能 → feature，修复 → fix，实验 → experiment。

### 原则4：安全第一

破坏性操作（reset --hard、force push、branch -D）必须确认。
Git 有 reflog 可以恢复，但预防比恢复成本低100倍。

### 原则5：AI不背锅

AI 执行 Git 操作出问题 = 用户没确认。
AI 的上下文记忆不可靠，任何破坏性操作必须用户确认后才执行。

---

## Phase 0：状态检查（每次操作前必做）

### 触发条件
- 用户说"提交"/"推送"/"合并"等操作指令
- AI 准备执行任何 git 命令

### 检查清单

```bash
# 1. 确认当前分支
git branch

# 2. 确认工作区状态
git status

# 3. 确认是否有未提交的改动
git diff --stat

# 4. 确认是否有未暂存的改动
git diff
```

### 输出格式

```
当前分支：feature/login
工作区状态：干净 / 有 3 个文件修改
未暂存改动：login.py (+10, -5)
建议操作：可以安全提交 / 需要先处理改动
```

---

## Phase 1：操作执行

### 1.1 提交流程
`git diff` → `git add <file>`（逐个添加，不要 `git add .`）→ `git diff --staged` → `git commit -m "feat: xxx"`

### 1.2 推送流程
`git pull --rebase` → 解决冲突（如有）→ `git push`（首次：`git push -u origin <branch>`）

### 1.3 合并流程
`git checkout main` → `git pull` → `git merge --no-ff <branch>` → `git push` → `git branch -d <branch>`

### 1.4 变基流程
`git checkout <branch>` → `git rebase main` → 解决冲突 → `git add` → `git rebase --continue` → `git push --force-with-lease`

---

## Phase 2：验证（操作后必做）

### 检查清单

```bash
# 1. 确认提交成功
git log --oneline -3

# 2. 确认分支状态
git status

# 3. 确认远程同步
git log --oneline origin/main..main
```

### 验证标准

| 检查项 | 通过条件 |
|--------|---------|
| 提交存在 | git log 能看到新提交 |
| 工作区干净 | git status 显示 nothing to commit |
| 远程同步 | 本地领先远程 0 或 1 个提交 |

---

## Phase 3：冲突解决

### 3.1 识别冲突

```bash
# 合并时遇到冲突
git status
# 显示：both modified: login.py

# 查看冲突内容
git diff
```

### 3.2 解决流程

```bash
# 1. 打开冲突文件，找到 <<<<<<< 标记
# 2. 理解两边改动意图
# 3. 保留正确内容，删除冲突标记
# 4. 标记为已解决
git add login.py

# 5. 继续合并/变基
git merge --continue
# 或
git rebase --continue
```

### 3.3 放弃合并

```bash
# 如果冲突太复杂
git merge --abort
# 或
git rebase --abort
```

**详细冲突解决指南**：参见 `references/conflict-resolution.md`

---

## Phase 4：历史安全审计（推送前 / 转公开前）

> 来源：Anthropic 2026-09 威胁情报报告 GTG-50020——攻击者从公开仓库 Git 历史中提取被删除的 API Key。
> 核心观点：**Git 历史不是"已删除"，而是"仍在但不再显示"。**
> 详细方法论：`references/git-history-security.md`

### 触发条件

| 信号 | 动作 |
|------|------|
| 首次推送本地仓库到远程 | 必须做全量历史扫描 |
| 仓库从私有转公开 | 必须做全量历史扫描 |
| 接手他人仓库（clone 后） | 必须做全量历史扫描 |
| 删除了包含敏感信息的文件后 | 必须做历史清理 + 凭据轮换 |
| 定期（每月一次，持续项目） | 推荐做历史扫描 |

### Step 1：推送前快速扫描（30 秒）

每次 `git push` 前，先看 diff 里有没有敏感内容：

```bash
git diff --cached | grep -E "(sk-[a-zA-Z0-9]{20,}|AKIA[A-Z0-9]{16}|ghp_[a-zA-Z0-9]{36}|xoxb-|-----BEGIN|password\s*=)"
```

- 有命中 → 立即停止推送，检查是否为真实泄露
- 无命中 → 安全，继续推送

### Step 2：全量历史扫描（首次推送到远程 / 转公开前）

```bash
# 方法一：grep 全量 diff（零依赖，最可靠）
git log -p --all | grep -E "(sk-[a-zA-Z0-9]{20,}|AKIA[A-Z0-9]{16}|ghp_[a-zA-Z0-9]{36}|xoxb-|-----BEGIN)" | head -50

# 方法二：git-secrets（AWS 出品，推荐长期使用）
git secrets --scan-history

# 方法三：trufflehog（社区工具，覆盖最广）
trufflehog git file://. --only-verified
```

### Step 3：发现泄露后的处理

**铁律：先轮换凭据，再清理历史。** 顺序不能反。

```
发现历史中有泄露的 Key
  → 立即去服务商控制台撤销该 Key 并重新生成（第一步）
  → git filter-repo 清理历史中的泄露内容
  → git push --force --all（协作项目中通知所有合作者重新 clone）
```

### Step 4：历史清理工具选择

| 工具 | 推荐度 | 适用场景 |
|------|--------|---------|
| **git filter-repo** | 首选 | 通用历史重写（删除文件/替换字符串） |
| BFG Repo-Cleaner | 次选 | 简单的文件删除或字符串替换 |
| `git filter-branch` | 禁用 | 已废弃，极慢且易出错 |

```bash
# 删除历史中的特定文件
git filter-repo --path .env --invert-paths

# 替换历史中的敏感字符串
git filter-repo --replace-text <(echo "sk-proj-xxx==>***REMOVED***")
```

**铁律**：历史重写会改写所有受影响的 commit hash。协作项目中，所有合作者必须重新 clone。

### 私有仓库不等于安全

推送到私有仓库前也要做历史扫描。风险来源：合作者权限、fork 历史快照、GitHub API 访问、平台漏洞、仓库转公开时历史瞬间暴露。

---

## AI 结合 Git 操作

### 规则1：AI 执行 Git 必须确认

AI 执行以下操作前必须让用户确认：
- `git reset --hard`
- `git push --force`
- `git branch -D`
- `git rebase`（已推送的分支）
- `git cherry-pick`（可能冲突）

### 规则2：AI 不猜测分支意图

用户说"提交代码"时，AI 必须先问：
1. 提交到哪个分支？
2. 提交信息是什么？
3. 哪些文件需要提交？

### 规则3：AI 操作后必须验证

AI 执行任何 Git 操作后，必须执行 Phase 2 验证。

### 规则4：AI 不替代用户解决冲突

AI 可以展示冲突内容和两边的改动，但**不能自动选择保留哪个**。
冲突解决必须用户决定。

### 规则5：AI 提交信息必须规范

AI 生成的提交信息必须遵循 Conventional Commits：
- `feat:` 新功能
- `fix:` 修复 bug
- `docs:` 文档更新
- `refactor:` 重构
- `test:` 测试
- `chore:` 杂项

---

## 命令速查

**完整命令速查**：参见 `references/git-commands.md`

**历史安全命令速查**：
```bash
# 扫描全量历史中的敏感模式
git log -p --all | grep -E "(sk-[a-zA-Z0-9]{20,}|AKIA[A-Z0-9]{16}|ghp_[a-zA-Z0-9]{36})" | head -50

# git-secrets 扫描历史（需安装）
git secrets --scan-history

# trufflehog 扫描（需安装）
trufflehog git file://. --only-verified

# 清理历史中的特定文件
git filter-repo --path .env --invert-paths

# 替换历史中的敏感字符串
git filter-repo --replace-text <(echo "PATTERN==>REPLACEMENT")
```

---

## 分支管理策略

### Git Flow（复杂项目）

| 分支 | 用途 | 生命周期 |
|------|------|---------|
| main | 生产环境代码 | 永久 |
| develop | 开发主分支 | 永久 |
| feature/* | 功能开发 | 临时 |
| release/* | 发布准备 | 临时 |
| hotfix/* | 紧急修复 | 临时 |

### GitHub Flow（推荐简单项目）

1. 从 main 创建 feature 分支
2. 在 feature 分支上开发
3. 提交 Pull Request
4. 代码审查
5. 合并到 main

**详细分支策略**：参见 `references/branching-strategies.md`

---

## 协作规范

### Commit Message 规范

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type 类型**：
- feat: 新功能
- fix: 修复 bug
- docs: 文档更新
- style: 代码格式（不影响功能）
- refactor: 重构
- test: 测试
- chore: 构建/工具变更

### Pull Request 规范

1. 标题简洁明了
2. 描述包含：做了什么、为什么做、怎么测试
3. 关联相关 Issue
4. 请求适当的审查者

---

## 安全红线

### 禁止操作1：不确认就 force push

**错误**：
```bash
git push --force  # 危险！可能覆盖队友代码
```

**正确**：
```bash
git push --force-with-lease
```

### 禁止操作2：在 main 上直接 reset

**错误**：
```bash
git checkout main
git reset --hard HEAD~3  # 丢失3个提交！
```

**正确**：
```bash
git revert <commit-hash>
```

### 禁止操作3：不看 status 就操作

**错误**：
```bash
git add .  # 可能添加不该提交的文件
git commit -m "update"
```

**正确**：
```bash
git status
git diff
git add login.py  # 逐个添加
git commit -m "feat: 实现登录功能"
```

### 禁止操作4：提交敏感信息

**错误**：
```bash
git add .env  # 包含 API Key
```

**正确**：
```bash
# .gitignore 必须包含
.env
*.key
*.pem
node_modules/
```

### 禁止操作5：把 `.gitignore` 当历史清除

**错误认知**：加了 `.env` 到 `.gitignore` → "历史里的 `.env` 也安全了"

**真相**：`.gitignore` 只管**未来的新文件**。已在 Git 历史中的内容不受影响。`git rm --cached` 只从当前版本移除追踪，历史里仍有完整内容。

**正确做法**：已在历史中的敏感文件 → `git filter-repo` 清理全量历史 → 立即轮换泄露的凭据。详见 Phase 4。

### 禁止操作6：假设私有仓库就安全

**错误认知**：仓库设为 Private → "历史里的密钥没关系"

**真相**：合作者能看完整历史；fork 保留了历史快照；GitHub API 可读取；仓库转公开时历史瞬间暴露。私有只防外部陌生人，不防内部和平台。

**正确做法**：推送到任何远程之前，先做 Phase 4 历史扫描。私有仓库也需要。

---

## 常见场景

### 场景1：提交代码
```
用户：提交代码
AI → Phase 0: git status + git branch
AI → Phase 1: git add <files> + git commit -m "xxx"
AI → Phase 2: git log --oneline -3
```

### 场景2：推送代码
```
用户：推送代码
AI → Phase 0: git status + git log origin/main..main
AI → Phase 1: git pull --rebase + git push
AI → Phase 2: git status
```

### 场景3：创建分支
```
用户：创建新分支
AI → 问：分支名？从哪个分支创建？
AI → Phase 1: git checkout -b <branch> <base>
AI → Phase 2: git branch
```

### 场景4：合并分支
```
用户：合并 feature 到 main
AI → Phase 0: git status + 确认 feature 分支已测试
AI → Phase 1: git checkout main + git merge --no-ff feature
AI → Phase 2: git log --oneline --graph
AI → 问：是否删除已合并的 feature 分支？
```

### 场景5：解决冲突
```
用户：有冲突
AI → Phase 3: git status + git diff
AI → 展示冲突内容 + 两边改动
AI → 问：保留哪个？
AI → 用户决定后 git add + git merge --continue
```

---

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| 误提交了敏感信息 | `git filter-repo`（首选）或 BFG Repo-Cleaner。**先轮换泄露的凭据，再清理历史** |
| 想撤销最后一次提交 | `git reset --soft HEAD~1`（保留变更） |
| 想撤销已推送的提交 | `git revert <commit-hash>` |
| 分支太多想清理 | `git branch -d <branch>` / `git push origin --delete <branch>` |
| 想查看某个文件的历史 | `git log --follow -p -- <file>` |
| 已删除的 `.env` 还在历史里 | `git filter-repo --path .env --invert-paths` + 立即轮换 Key |
| `.gitignore` 加了敏感文件但历史还有 | `.gitignore` 不管历史。必须 `git filter-repo` 清理 + 凭据轮换 |
| 仓库要从私有转公开 | 先做 Phase 4 全量历史扫描，确认无泄露后再转 |
| 接手他人仓库不确定历史是否干净 | `git secrets --scan-history` 或 `trufflehog git file://.` |

**更多案例**：参见 `references/case-studies.md`

---

## 铁律

1. **先 status 后操作**：任何 Git 操作前必须 `git status`
2. **小步提交**：一次提交只做一件事
3. **分支隔离**：永远不在 main 上直接开发
4. **提交信息规范**：遵循 Conventional Commits
5. **破坏性操作确认**：reset/revert/force push 必须确认
6. **先拉后推**：推送前先 `git pull --rebase`
7. **冲突不自动解决**：AI 展示冲突，用户决定保留哪个
8. **操作后验证**：提交/推送/合并后必须验证结果

---

## 联动技能

| 技能 | 联动方式 |
|------|---------|
| **li-workflow** | 调用 mei-git-assistant 执行 Git 操作 |
| **li-script** | 脚本版本控制 |
| **li-debug** | 调试时查看 git 历史 |

---

## 参考技能

- **li-workflow**：工作流编排，Git Hooks、CI/CD 配置
- **li-script**：脚本库管家，先搜库再动手
- **li-debug**：调试循环，反馈循环优先

---

## 禁止模式

- ❌ 不确认就执行 reset --hard
- ❌ 不确认就执行 force push
- ❌ 不看 status 就 git add .
- ❌ 提交信息写 "fix"、"update"、"change"
- ❌ 在 main 上直接开发
- ❌ 提交 .env、密钥等敏感文件
- ❌ 自动解决冲突（必须用户决定）
- ❌ 认为加了 `.gitignore` 就清除了历史中的敏感文件
- ❌ 首次推送到远程前不做历史扫描
- ❌ 用已废弃的 `git filter-branch` 做历史清理

---

## 参考文件

| 文件 | 内容 |
|------|------|
| `references/git-commands.md` | 命令速查表 |
| `references/branching-strategies.md` | 分支策略（Git Flow/GitHub Flow/Trunk-Based） |
| `references/conflict-resolution.md` | 冲突解决指南 |
| `references/anti-patterns.md` | 反模式（10种常见错误） |
| `references/case-studies.md` | 案例库（10个真实场景） |
| `references/clion-ide-mapping.md` | CLion 界面操作 ↔ CLI 命令映射（JetBrains 系 IDE） |
| `references/concept-semantics.md` | Git 概念与语义辨析（HEAD/三层存储/detached HEAD） |
| `references/git-history-security.md` | Git 历史安全审计（推送前扫描/泄露检测/历史清理/私有仓库风险） |
