# 冲突解决指南

## 冲突的本质

Git 冲突 = 两个分支修改了同一位置，Git 无法自动判断保留哪个。

**不是错误，是正常现象。**

---

## 三种冲突类型

### 1. 内容冲突（最常见）

两个分支修改了同一文件的同一行。

```
<<<<<<< HEAD
这是当前分支的内容
=======
这是要合并的分支的内容
>>>>>>> feature-branch
```

**解决方法**：
1. 理解两边的改动意图
2. 选择保留一边，或手动合并两边的内容
3. 删除冲突标记（`<<<<<<<`、`=======`、`>>>>>>>`）
4. `git add <file>` 标记为已解决

### 2. 删除冲突

一个分支修改了文件，另一个分支删除了文件。

**解决方法**：
- 保留文件：`git add <file>`
- 确认删除：`git rm <file>`

### 3. 文件名冲突（重命名冲突）

两个分支重命名了同一文件，或一个分支重命名、另一个分支修改。

**解决方法**：
- 确定最终文件名
- 手动合并内容
- `git add` 新文件名

---

## 六步解决流程

```bash
# 第一步：冷静，看清楚
git status

# 第二步：查看冲突详情
git diff

# 第三步：逐个解决冲突文件
# 用编辑器打开文件，找到 <<<<<<< 标记
# 理解两边改动，决定保留什么

# 第四步：标记为已解决
git add <resolved-file>

# 第五步：继续合并/变基
git merge --continue
# 或
git rebase --continue

# 第六步：验证结果
git log --oneline -5
```

---

## 冲突标记解读

```bash
<<<<<<< HEAD
你当前分支的代码
=======
要合并进来的代码
>>>>>>> feature-branch
```

- `<<<<<<< HEAD` 到 `=======`：当前分支的内容
- `=======` 到 `>>>>>>> feature-branch`：要合并的内容

---

## 常见场景处理

### 场景1：两个人改了同一函数

```python
<<<<<<< HEAD
def calculate(x, y):
    return x + y  # 当前分支：简单相加
=======
def calculate(x, y):
    return x * y  # feature分支：改为相乘
>>>>>>> feature-branch
```

**解决**：确认需求是加法还是乘法，保留正确的一个。

### 场景2：一个人改了代码，另一个人改了注释

```python
<<<<<<< HEAD
# 这个函数处理用户输入（注释更新）
def process(input):
=======
def process(data):  # 参数名改进
>>>>>>> feature-branch
```

**解决**：两者都保留：
```python
# 这个函数处理用户输入
def process(data):
```

### 场景3：依赖文件冲突（如 package.json）

不要手动编辑，用包管理器解决：
```bash
# 合并后重新安装
npm install
# 或
pip install -r requirements.txt
```

---

## 预防冲突

1. **频繁拉取**：`git pull --rebase` 保持本地最新
2. **小步提交**：减少每次改动的范围
3. **及时合并**：分支不要活太久
4. **沟通协调**：多人改同一文件时提前沟通
5. **模块化**：减少多人同时修改同一文件的概率

---

## 放弃合并

如果冲突太复杂，想回到合并前的状态：

```bash
git merge --abort
```

如果是在 rebase 中：
```bash
git rebase --abort
```

---

## 使用工具

### 命令行工具

```bash
# 使用 vimdiff 查看冲突
git mergetool

# 配置默认合并工具
git config --global merge.tool vimdiff
```

### GUI 工具

- VS Code：内置冲突高亮，点击按钮选择保留哪个
- GitHub Desktop：可视化冲突解决
- Sourcetree：图形化合并工具
- IntelliJ IDEA：强大的合并工具
