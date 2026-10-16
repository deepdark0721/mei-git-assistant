# Git 案例库

## 案例1：误删分支恢复

**场景**：执行了 `git branch -D feature/important`，发现还没合并

**恢复方法**：
```bash
# 找到被删除分支的最后一次提交
git reflog
# 找到类似这样的记录：
# abc1234 HEAD@{5}: checkout: moving from feature/important to main

# 重新创建分支
git checkout -b feature/important abc1234
```

**教训**：删除分支前先确认已合并，或用 `-d`（小写）自动检查

---

## 案例2：提交了敏感信息

**场景**：提交了包含 API Key 的 .env 文件

**紧急处理**：
```bash
# 方案1：如果还没推送，修改最后一次提交
git reset --soft HEAD~1
# 从暂存区移除敏感文件
git restore --staged .env
# 添加到 .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: 更新提交"

# 方案2：如果已经推送，需要改写历史
# 使用 git-filter-repo 或 BFG Repo-Cleaner
# 注意：这会改写历史，需要团队协调

# 最重要的是：立即轮换泄露的密钥！
```

**预防**：.gitignore 里必须有 .env

---

## 案例3：合并冲突导致代码丢失

**场景**：合并分支后，发现某些代码被意外删除

**诊断**：
```bash
# 查看合并提交的详情
git log --oneline --graph -10
git show <merge-commit-hash>

# 查看文件的修改历史
git log --follow -p -- path/to/file
```

**恢复**：
```bash
# 找到文件在合并前的版本
git show HEAD~1:path/to/file > /tmp/old_version

# 对比找出丢失的部分
diff /tmp/old_version path/to/file

# 手动恢复丢失的代码
```

**预防**：合并时用 `--no-ff` 保留分支历史，方便追溯

---

## 案例4：rebase 冲突导致提交重复

**场景**：`git rebase main` 后，发现某些提交出现了两次

**原因**：rebase 过程中解决冲突时，误将已有的提交内容重新提交

**解决**：
```bash
# 如果 rebase 还没完成
git rebase --abort

# 如果已经完成，用交互式 rebase 整理
git rebase -i HEAD~10
# 标记重复的提交为 drop 或 squash
```

**预防**：rebase 前先 `git log` 记住当前分支的提交数量

---

## 案例5：push --force 覆盖队友代码

**场景**：执行 `git push --force` 后，队友的提交被覆盖

**紧急恢复**：
```bash
# 在远程仓库查看被覆盖的提交
# GitHub/GitLab 的 Web 界面可以看到 reflog

# 或者让队友执行
git reflog
# 找到被覆盖前的提交 hash

# 本地恢复
git checkout -b recovery abc1234
git push origin recovery

# 然后合并回 main
```

**铁律**：永远用 `--force-with-lease` 代替 `--force`

---

## 案例6：大型二进制文件撑爆仓库

**场景**：不小心提交了 500MB 的视频文件

**解决**：
```bash
# 方案1：如果还没推送
git reset --soft HEAD~1
git restore --staged video.mp4
echo "*.mp4" >> .gitignore
git commit -m "chore: 移除大文件"

# 方案2：如果已经推送，需要清理历史
# 使用 BFG Repo-Cleaner
bfg --delete-files video.mp4
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

**预防**：.gitignore 覆盖常见大文件类型

---

## 案例7：cherry-pick 导致冲突

**场景**：从另一个分支 cherry-pick 提交时遇到冲突

**处理**：
```bash
# 开始 cherry-pick
git cherry-pick abc1234

# 遇到冲突后解决
# 编辑冲突文件
git add <resolved-files>
git cherry-pick --continue

# 如果太复杂，放弃
git cherry-pick --abort
```

**技巧**：cherry-pick 前先确保目标分支是最新的

---

## 案例8：误操作 reset --hard

**场景**：执行 `git reset --hard HEAD~3`，丢失了3个提交

**恢复**：
```bash
# reflog 记录了所有 HEAD 的移动
git reflog
# 找到 reset 前的提交
# abc1234 HEAD@{1}: reset: moving to HEAD~3
# def5678 HEAD@{2}: commit: feat: 添加功能C

# 恢复
git reset --hard def5678
```

**教训**：`reset --hard` 前先 `git stash` 或确认不需要当前改动

---

## 案例9：分支名拼写错误

**场景**：`git checkout -b featuer/login`（拼错了）

**修正**：
```bash
# 重命名分支
git branch -m featuer/login feature/login

# 如果已经推送到远程
git push origin -u feature/login
git push origin --delete featuer/login
```

---

## 案例10：提交到错误的分支

**场景**：在 main 上写了代码并提交了

**修正**：
```bash
# 创建正确的分支（包含刚才的提交）
git checkout -b feature/correct-branch

# 回退 main 上的提交
git checkout main
git reset --hard HEAD~1

# 继续在正确的分支开发
git checkout feature/correct-branch
```

**预防**：提交前 `git branch` 确认当前分支
