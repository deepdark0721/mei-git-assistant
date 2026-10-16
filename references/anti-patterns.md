# Git 反模式

## 1. `git add .` 综合症

**现象**：每次都用 `git add .` 暂存所有文件

**问题**：
- 可能提交不该提交的文件（临时文件、密钥、调试代码）
- 无法做到"一次提交只做一件事"
- 出问题时无法精确回滚

**正确做法**：
```bash
# 逐个添加
git add src/login.py
git add tests/test_login.py

# 或使用交互模式
git add -p  # 逐块选择
```

---

## 2. 在 main 上直接开发

**现象**：不创建分支，直接在 main/master 上写代码

**问题**：
- 半成品代码可能被部署
- 无法并行开发多个功能
- 合并冲突风险高

**正确做法**：
```bash
# 永远从 main 创建 feature 分支
git checkout main
git checkout -b feature/xxx
```

---

## 3. 提交信息写 "fix"、"update"、"change"

**现象**：提交信息模糊不清

**问题**：
- 无法通过 `git log` 理解项目历史
- 出问题时无法定位是哪次提交引入的
- Code Review 时无法理解改动目的

**正确做法**：
```
# 不好
fix bug
update code
change something

# 好
fix: 修复用户登录时密码验证失败的问题
feat: 添加用户头像上传功能
docs: 更新 README 安装说明
refactor: 重构登录模块，提取验证逻辑
```

---

## 4. 长期不提交

**现象**：写了一天代码，最后一次性提交

**问题**：
- 提交历史无法反映开发过程
- 出问题时无法精确回滚到某个状态
- 合并冲突可能很大

**正确做法**：
```bash
# 每完成一个逻辑单元就提交
git add login.py
git commit -m "feat: 实现登录表单验证"

git add auth.py
git commit -m "feat: 添加 JWT token 生成"

git add test_auth.py
git commit -m "test: 添加认证模块单元测试"
```

---

## 5. `git push --force` 不加 `--with-lease`

**现象**：直接 `git push --force`

**问题**：
- 可能覆盖队友的提交
- 无法恢复被覆盖的代码
- 协作噩梦

**正确做法**：
```bash
# 如果必须强制推送，用 --force-with-lease
git push --force-with-lease

# 这会检查远程分支是否有你没拉取的更新
# 如果有，会拒绝推送
```

---

## 6. 不写 .gitignore

**现象**：提交了 node_modules/、.env、IDE 配置等

**问题**：
- 仓库体积膨胀
- 敏感信息泄露（API Key、密码）
- 依赖冲突

**正确做法**：
```bash
# 在项目根目录创建 .gitignore
echo "node_modules/" >> .gitignore
echo ".env" >> .gitignore
echo "*.pyc" >> .gitignore
echo ".idea/" >> .gitignore
```

---

## 7. 直接改写已推送的历史

**现象**：`git rebase` 或 `git reset` 已经推送到远程的提交

**问题**：
- 队友基于旧历史开发的代码会出问题
- 合并时会出现重复提交
- 信任崩塌

**正确做法**：
```bash
# 已经推送的提交，用 revert 而不是 reset
git revert <commit-hash>

# 这会创建一个新的反向提交，安全撤销
```

---

## 8. 不拉取就推送

**现象**：直接 `git push`，发现被拒绝才拉取

**问题**：
- 频繁的非快进推送
- 可能需要处理意外的冲突

**正确做法**：
```bash
# 推送前先拉取
git pull --rebase
git push

# 或配置自动 rebase
git config --global pull.rebase true
```

---

## 9. 提交大文件到仓库

**现象**：把视频、大型数据集、编译产物提交到 Git

**问题**：
- 仓库体积爆炸
- clone 超慢
- 历史记录永久膨胀

**正确做法**：
```bash
# 使用 Git LFS 管理大文件
git lfs install
git lfs track "*.mp4"
git lfs track "*.zip"
git add .gitattributes

# 或把大文件放在外部存储，只提交链接
```

---

## 10. 不看 `git status` 就操作

**现象**：不确认当前状态就执行 git 命令

**问题**：
- 可能在错误的分支上操作
- 可能丢失未提交的改动
- 可能提交不该提交的文件

**正确做法**：
```bash
# 任何操作前先看状态
git status

# 确认当前分支
git branch

# 确认要提交的内容
git diff --staged
```
