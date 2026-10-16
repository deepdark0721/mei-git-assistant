# Git 命令速查表

## 基础操作

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git init` | 初始化仓库 | 新项目开始 |
| `git clone <url>` | 克隆远程仓库 | 获取项目代码 |
| `git status` | 查看状态 | 每次操作前必看 |
| `git add <file>` | 暂存文件 | 准备提交 |
| `git add .` | 暂存所有变更 | 快速提交（慎用） |
| `git commit -m "msg"` | 提交 | 保存变更 |
| `git push` | 推送到远程 | 分享代码 |
| `git pull` | 拉取远程更新 | 同步代码 |
| `git pull --rebase` | 变基拉取 | 保持历史干净 |

## 分支操作

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git branch` | 查看本地分支 | 确认当前位置 |
| `git branch -a` | 查看所有分支 | 包含远程分支 |
| `git branch <name>` | 创建分支 | 新功能开发 |
| `git checkout <name>` | 切换分支 | 切换工作区 |
| `git checkout -b <name>` | 创建并切换 | 常用快捷方式 |
| `git switch <name>` | 切换分支（新版） | 推荐用法 |
| `git switch -c <name>` | 创建并切换（新版） | 推荐用法 |
| `git branch -d <name>` | 删除分支（已合并） | 清理完成的分支 |
| `git branch -D <name>` | 强制删除分支 | 慎用！ |

## 合并与变基

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git merge <branch>` | 合并分支 | 集成代码 |
| `git merge --no-ff <branch>` | 禁用快进合并 | 保留分支历史 |
| `git rebase <branch>` | 变基 | 线性历史 |
| `git rebase -i HEAD~3` | 交互式变基 | 整理提交历史 |
| `git cherry-pick <commit>` | 摘取提交 | 选择性合并 |

## 撤销与回退

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git checkout -- <file>` | 撤销工作区修改 | 丢弃未暂存的改动 |
| `git restore <file>` | 撤销工作区修改（新版） | 推荐用法 |
| `git restore --staged <file>` | 取消暂存 | 从暂存区移除 |
| `git reset HEAD <file>` | 取消暂存（旧版） | 等同上面 |
| `git reset --soft HEAD~1` | 回退提交，保留暂存 | 修改提交信息 |
| `git reset --mixed HEAD~1` | 回退提交和暂存 | 重新组织提交 |
| `git reset --hard HEAD~1` | 完全回退 | 危险！丢失改动 |
| `git revert <commit>` | 创建反向提交 | 安全撤销已推送的提交 |

## 查看历史

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git log` | 查看提交历史 | 基础历史查看 |
| `git log --oneline` | 简洁历史 | 快速浏览 |
| `git log --graph` | 图形化历史 | 看分支合并情况 |
| `git log -p` | 查看差异 | 看每次提交改了什么 |
| `git diff` | 查看未暂存差异 | 工作区 vs 暂存区 |
| `git diff --staged` | 查看已暂存差异 | 暂存区 vs 上次提交 |
| `git show <commit>` | 查看某次提交 | 分析特定改动 |

## 远程操作

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git remote -v` | 查看远程仓库 | 确认远程地址 |
| `git remote add <name> <url>` | 添加远程仓库 | 关联新远程 |
| `git fetch` | 获取远程更新（不合并） | 先查看再决定 |
| `git push -u origin <branch>` | 推送并关联分支 | 首次推送新分支 |
| `git push --force` | 强制推送 | 危险！覆盖远程 |
| `git push --force-with-lease` | 安全强制推送 | 比 --force 安全 |

## 暂存工作

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git stash` | 暂存当前工作 | 临时切换分支 |
| `git stash list` | 查看暂存列表 | 管理多次暂存 |
| `git stash pop` | 恢复并删除暂存 | 恢复工作 |
| `git stash apply` | 恢复暂存（保留） | 多次应用同一暂存 |
| `git stash drop` | 删除暂存 | 清理 |

## 标签

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git tag <name>` | 创建轻量标签 | 标记版本 |
| `git tag -a <name> -m "msg"` | 创建附注标签 | 带说明的版本标记 |
| `git push origin <tag>` | 推送标签 | 分享标签 |
| `git push origin --tags` | 推送所有标签 | 批量分享 |

## 撤销场景选择（按状态定命令）

| 状态 | 命令 | 说明 |
|------|------|------|
| 未提交的改动 | `git restore <file>` | 丢弃工作区修改 |
| 已提交但未推送 | `git reset --soft HEAD~1` | 撤销提交保留改动 |
| 已推送 | `git revert <commit>` | 不改写历史，新增反向提交 |

> 层级逻辑详见 `concept-semantics.md`——撤销前先判断改动处于哪个阶段。

## 补丁

| 命令 | 作用 | 常用场景 |
|------|------|---------|
| `git diff > change.patch` | 从未提交修改生成补丁 | 分享局部改动 |
| `git apply change.patch` | 应用补丁 | 应用他人补丁 |
