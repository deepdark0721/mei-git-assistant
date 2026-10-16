# 分支策略

## Git Flow（复杂项目）

适合有明确发布周期的项目。

### 五种分支类型

| 分支类型 | 命名 | 生命周期 | 用途 |
|---------|------|---------|------|
| **main** | main/master | 永久 | 生产环境代码 |
| **develop** | develop | 永久 | 开发主线 |
| **feature** | feature/xxx | 临时 | 新功能开发 |
| **release** | release/x.x.x | 临时 | 版本发布准备 |
| **hotfix** | hotfix/xxx | 临时 | 紧急修复 |

### 工作流程

```
main ─────────●─────────────────●─────────────────
              │                 ↑
              │                 │ merge + tag
              │                 │
release ──────┼─────────●───────●
              │         ↑       │
              │         │ merge │
              │         │       │
develop ──────●────●────●───────●────●─────────────
                   ↑    ↑            ↑
                   │    │            │
feature/A ─────────●────┘            │
                                     │
feature/B ───────────────────────────●
```

### 命令示例

```bash
# 开始新功能
git checkout develop
git checkout -b feature/user-auth

# 完成开发
git checkout develop
git merge --no-ff feature/user-auth
git branch -d feature/user-auth

# 准备发布
git checkout develop
git checkout -b release/1.0.0
# 修复发布分支上的bug...

# 正式发布
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# 紧急修复
git checkout main
git checkout -b hotfix/fix-login
# 修复...
git checkout main
git merge --no-ff hotfix/fix-login
git tag -a v1.0.1 -m "Hotfix: fix login"
git checkout develop
git merge --no-ff hotfix/fix-login
git branch -d hotfix/fix-login
```

---

## GitHub Flow（简单项目）

适合持续部署的项目。

### 核心原则

1. main 分支永远可部署
2. 所有开发在 feature 分支
3. 通过 Pull Request 合并
4. 合并后立即部署

### 工作流程

```
main ─────────●─────────────●─────────────●────────
              │             ↑             ↑
              │             │             │
              │         PR merge      PR merge
              │             │             │
feature/A ────●────●────────●             │
                   ↑                      │
                   │ commit               │
                   │                      │
feature/B ───────────────────●────●───────●
                              ↑    ↑
                              │    │
                           commit commit
```

### 命令示例

```bash
# 从 main 创建 feature
git checkout main
git pull
git checkout -b feature/add-search

# 开发并提交
git add .
git commit -m "feat: add search functionality"
git push origin feature/add-search

# 在 GitHub 上创建 Pull Request
# Code Review 通过后合并

# 合并后清理
git checkout main
git pull
git branch -d feature/add-search
```

---

## Trunk-Based Development（极速发布）

适合高频发布的项目。

### 核心原则

1. 所有人直接向 main 提交（或用极短命的分支）
2. 分支寿命 < 1 天
3. 依赖 Feature Flag 控制功能发布
4. 持续集成，频繁合并

### 适用条件

- 完善的自动化测试
- Feature Flag 基础设施
- 高度协作的团队

---

## 选择建议

| 项目类型 | 推荐策略 | 理由 |
|---------|---------|------|
| 个人项目 | GitHub Flow | 简单直接 |
| 团队项目（有发布周期） | Git Flow | 结构清晰 |
| 团队项目（持续部署） | GitHub Flow | 灵活高效 |
| 大型团队（高频发布） | Trunk-Based | 减少分支冲突 |
| 学生作业/课程项目 | GitHub Flow | 够用就好 |
