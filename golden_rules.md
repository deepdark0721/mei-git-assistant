# Golden Rules - mei-git-assistant

## 1. 先看再动

任何 Git 操作前，先 `git status` 确认当前状态。
不知道当前状态就执行操作 = 盲人过马路。

## 2. 小步提交

一次提交只做一件事。
`git add .` 是懒人的做法，也是出问题时无法回滚的原因。

## 3. 分支隔离

永远不要在 main/master 上直接开发。
功能开发 → feature 分支，修复 bug → fix 分支，实验性工作 → experiment 分支。

## 4. 冲突是正常的

遇到 merge conflict 不要慌，也不要盲目接受某一方。
先理解两边的改动意图，再决定保留什么。

## 5. 提交信息有意义

`fix bug` 不是有意义的提交信息。
说清楚修了什么 bug、为什么修、怎么修的。

## 6. 不碰历史（除非你知道自己在什么）

`git reset --hard`、`git rebase`、`git force push` 会改写历史。
协作项目中改写历史 = 给队友制造噩梦。

## 7. 先拉后推

推送前先拉取最新代码，解决冲突后再推。
`git pull --rebase` 比 `git pull` 保持更干净的历史。

## 8. AI 操作 Git 必须确认

AI 执行任何破坏性 Git 操作（reset、revert、force push、branch -D）前必须确认。
AI 的上下文记忆不可靠，不要假设它知道你想要什么。

## 9. 历史比文件更危险

当前文件里的敏感信息容易发现，Git 历史里的难。`.gitignore` 只管未来，不管历史。`git rm --cached` 只移除当前追踪，历史里还有。
推送到远程之前，先扫历史。仓库从私有转公开之前，先扫历史。
发现泄露的正确顺序：**先轮换凭据，再清理历史**——顺序不能反。
