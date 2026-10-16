# mei-git-assistant

> Git 操作助手 Skill——为 AI 编程助手（Claude Code / Codex 等）设计的 Git 工作流技能：状态检查、规范化提交、分支策略、冲突解决、**推送前历史安全审计**，以及一套“AI 执行 Git 操作”的安全红线。

当前版本：**v2.2.0** · 约定：Conventional Commits · 适用：Claude Code 及兼容 Skill 格式的 Agent

---

## 这个 Skill 解决什么问题

AI 助手执行 Git 操作时最常见的四类事故：

1. **盲操作**——不看 `git status` 就 `git add .`，把不该提交的文件推上去
2. **破坏性操作不确认**——直接 `reset --hard` / `force push`，覆盖他人代码
3. **敏感信息入库**——API Key、`.env` 随手提交，且误以为 `.gitignore` 能救历史
4. **首次推送/转公开不扫历史**——以为"删掉的文件"不在历史里（它在）

本 Skill 把这四类事故逐一固化为强制流程。

---

## 核心内容

### 五条设计哲学

| 原则 | 含义 |
|---|---|
| 先看后动 | 任何 Git 操作前必先 `git status`——不知道状态就操作 = 盲人过马路 |
| 小步快跑 | 一次提交只做一件事，提交历史 = 可回溯的检查点 |
| 分支隔离 | 永不在 main 上直接开发；feature / fix / experiment 各归各 |
| 安全第一 | 破坏性操作必须确认——预防比 reflog 恢复成本低 100 倍 |
| AI 不背锅 | AI 执行 Git 出问题 = 用户没确认；破坏性操作一律先问 |

### 五阶段执行流程

- **Phase 0 · 状态检查**（每次操作前必做）：分支 / 工作区 / 未暂存改动四件套，结构化输出当前状态与建议操作
- **Phase 1 · 操作执行**：提交（逐文件 `git add`，禁 `git add .`）、推送（先 `pull --rebase`）、合并（`--no-ff`）、变基（`--force-with-lease`）四套标准流程
- **Phase 2 · 操作后验证**：提交存在 / 工作区干净 / 远程同步三条件，缺一不算完成
- **Phase 3 · 冲突解决**：识别 → 理解两边意图 → 用户决定保留哪边（**AI 不自动解决冲突**）
- **Phase 4 · 历史安全审计**（本 Skill 的特色章节）：
  - 触发条件：首次推送 / 私有转公开 / 接手他人仓库 / 删除敏感文件后
  - 三步扫描：push 前 30 秒 diff 快扫 → 全量历史扫描（grep / git-secrets / trufflehog）
  - 铁律：**先轮换凭据，再清理历史**（顺序不能反）；历史清理首选 `git filter-repo`，禁用已废弃的 `filter-branch`
  - 澄清两个致命误区：`.gitignore` 不管历史；私有仓库也不等于安全

### AI 结合 Git 的五条规则

1. 破坏性操作（reset --hard / force push / branch -D / 已推送分支的 rebase）必须用户确认
2. 不猜测分支意图——提交到哪个分支、信息写什么、包含哪些文件，逐项问清
3. 操作后必须跑 Phase 2 验证
4. 不替代用户解决冲突——展示两边改动，用户拍板
5. 提交信息必须遵循 Conventional Commits（feat / fix / docs / refactor / test / chore）

### 六条安全红线（禁止模式）

不确认就 force push · 在 main 上直接 reset · 不看 status 就 `git add .` · 提交敏感信息 · 以为 `.gitignore` 清了历史 · 以为私有仓库就安全——每条都附错误示例与正确做法。

---

## 目录结构

```
mei-git-assistant/
├── SKILL.md                      # 主文件：五原则 / 五阶段 / 红线 / 铁律 8 条
├── _meta.json                    # 元数据与完整性签名
├── eval.json / golden_rules.md   # 质量门禁与黄金规则
├── lifecycle-log.md              # 生命周期记录
└── references/
    ├── git-commands.md           # 命令速查表
    ├── branching-strategies.md   # 分支策略（Git Flow / GitHub Flow / Trunk-Based）
    ├── conflict-resolution.md    # 冲突解决指南
    ├── anti-patterns.md          # 10 种常见反模式
    ├── case-studies.md           # 10 个真实场景案例
    ├── concept-semantics.md      # 概念语义辨析（HEAD / 三层存储 / detached HEAD）
    ├── clion-ide-mapping.md      # CLion 界面操作与 CLI 命令映射
    └── git-history-security.md   # 历史安全审计详解（Key 泄露检测与清理）
```

---

## 安装

Claude Code / 兼容 Agent 的 skill 目录：

```bash
# 用户级安装
git clone https://github.com/deepdark0721/mei-git-assistant.git
cp -r mei-git-assistant ~/.claude/skills/mei-git-assistant
```

安装后对 AI 说“提交代码”“推送代码”“解决冲突”等触发词即可自动进入对应 Phase。

---

## License

MIT
