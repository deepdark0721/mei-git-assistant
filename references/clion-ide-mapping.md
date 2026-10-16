# CLion 操作与 CLI 命令映射

> 面向 JetBrains 系 IDE（以 CLion 为主，Gitee 托管）的 Git 操作对照。
> 本文件只保留"界面操作 ↔ 命令"的映射与队内红线，界面细节见《CLion操作总结.md》。
> 来源：飞书文档《更适合SRM宝宝的Clion-Git教程》，整理于 2026-08。

---

## 一、核心差异：CLion 提交自动 stage

CLion 的 Git 在提交时**自动把修改加入暂存区并提交**，不需要手动先 stage。

- 与命令行习惯不同：CLI 是 `git add` + `git commit` 两步；CLion 直接勾选文件 + 填备注 + 提交。
- `.gitignore` 的文件显示为黄色，不参与版本管理。
- 想精确控制暂存内容时，仍需回到命令行用 `git add <file>`。

| 界面操作 | CLI 等价 |
|----------|----------|
| 勾选文件 → 填备注 → 提交 | `git add <file>` + `git commit -m "备注"` |
| 新建项目 / VCS 初始化 | `git init` |
| 从 VCS 获取（Get from VCS）克隆 | `git clone <url>` |
| 共享到远端平台（Gitee） | `git remote add origin <repo_url>` |
| 推送 | `git push` / 变基后 `--force-with-lease` |
| 更新（Ctrl+T） | `git pull` / `git pull --rebase` |
| 签出分支 | `git switch <name>` |
| 撤销未提交更改（回滚） | `git restore <file>` |
| 撤销提交（Undo Commit） | `git reset --soft HEAD~1` |
| 恢复已推送提交（Revert） | `git revert <commit>` |
| 新建标记 | `git tag <name>` / `git tag -a <name> -m "msg"` |
| 优选所选更改 | `git cherry-pick <commit>`（单文件版，见下文） |

---

## 二、三个 JetBrains 特有概念

### 1. Change List（更改列表）

- 存在于 IDE 本地工作区，**不依赖 Git 仓库**，是 JetBrains 补充机制。
- 作用：把文件更改按逻辑分组管理，提交时才触发 Git 操作。
- 典型用法：`git reset --soft` 后，被丢弃的更改会落到某个更改列表（可新建），按需再推送。

### 2. Shelve（搁置）

- 临时收起未提交的修改，让工作区变干净再切换。
- 场景：直接签出某个提交（detached HEAD 前），若有已修改而未提交的文件，需先搁置，否则：
  - 冲突的修改：直接切换会报错。
  - 不冲突的修改：能切换成功，但会污染文件（见 `concept-semantics.md` 的"Git 管理的是修改"）。

### 3. Update（Ctrl+T）的两个选项

- **将传入更改合并到当前分支**：与默认 pull 相同，但自动拉与本地分支对应的远端分支（`branch1` 自动拉 `origin/branch1`）。
- **在传入更改上变基当前分支**：本地提交被放到更新后的远端节点之后，重放产生新提交，历史线性。

---

## 三、受保护分支配置

- 受保护分支不允许 force push。
- 配置位置：`设置 | 版本控制 | Git`（快捷键 Ctrl+Alt+S）。
- 若在受保护分支上执行 `git reset --soft` 想撤销已推送提交，会被拦——这是保护机制，不是 bug。

---

## 四、单文件 cherry-pick（优选所选更改）

想把某次提交中的**单个文件**更改应用到当前分支：

1. 签出目标分支。
2. 选中该文件。
3. 右键 → **优选所选更改**。
4. 此时会用到 Change List：把文件更改传输到某个更改列表，再按需推送。

对比：`git cherry-pick <commit>` 取的是整个提交的改动，单文件版只取一个文件。

---

## 五、队内红线（SRM / RoboMaster 场景）

- 克隆仓库需登录 Gitee 账号。
- 建议使用自己的练习仓库练习，**不要直接在 control-2026 上练习破坏性操作**。
- 合并提交不可随意删除。
- rebase 后推送使用 `--force-with-lease`，且确认无人更新远端。
- 受保护分支不允许 force push。

---

## 六、教学模式：解释 Git 概念时的输出框架

回答"解释 Git 概念 / 教新成员"类问题时，按以下结构组织：

1. 先给一句话本质。
2. 用队内实际场景举例（如 control-2026 仓库、RoboMaster 电控代码）。
3. 每个操作同时给出：CLI 命令 + 对应 CLion 界面位置（本节表格）。
4. 强调安全红线：不拿正式仓库练习破坏性操作；merge commit 不可 drop；rebase 后推送用 `--force-with-lease`；revert 与 reset 语义不同。
5. 如果成员的操作会改写历史（rebase / squash / fixup / drop），先提醒确认是否已推送、是否影响他人。
6. 回答保持简洁，给可直接执行的步骤，避免长篇理论。
