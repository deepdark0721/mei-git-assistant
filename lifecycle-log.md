# Lifecycle Log - mei-git-assistant

## 2026-07-29 - v2.0.0 重建

**操作**：按 li-skillcreate 六层结构完整重建

**变更**：
- 从 v1.0（185行纯SKILL.md）升级为 v2.0.0（完整skill结构）
- 新增 _meta.json、eval.json、golden_rules.md、DEPRECATED.md、lifecycle-log.md
- 新增 references/ 目录（5个参考文件）
- SKILL.md 重构为六层结构（YAML + 设计哲学 + Phase工作流 + 认知框架 + 参考技能 + 禁止模式）
- 触发词从 12 个扩展到 20+ 个
- 新增安全红线章节（4条禁止操作）
- 新增 AI 结合 Git 操作章节

**来源**：
- 原始内容：团队协作知识点/（5个文件）
- 融合内容：niuma-starter-kit-v3 脚本安全检查
- 融合内容：niuma-engine Git归档铁律

**备份**：`workspace/backup/mei-git-assistant-SKILL-backup-20260729-pre-rebuild.md`

## 2026-08-16 - v2.1.0 增量融合

**操作**：从 git_skills(1)/gitAssistant 融合增量（来源《更适合SRM宝宝的Clion-Git教程》）

**变更**：
- 新增 `references/clion-ide-mapping.md`（CLion 界面操作 ↔ CLI 命令映射）
- 新增 `references/concept-semantics.md`（Git 概念与语义辨析）
- `references/git-commands.md` 追加「撤销场景选择」+「补丁」章节
- SKILL.md references 清单更新，version 2.0.0 → 2.1.0

**备份**：`workspace/归档/2026-08-16/pre-gitskills-merge/`

## 2026-09-12 - v2.2.0 历史安全审计能力

**操作**：类型A功能迭代——补充 Git 历史安全审计能力（Phase 4）

**触发**：Anthropic 2026-09 威胁情报报告 GTG-50020 揭示攻击者从公开仓库 Git 历史中提取被删除的 API Key。用户体系中 mei-skills 开源在 GitHub，wechat-cli 读取本地数据，存在同类风险面。

**变更**：
- 新增 `references/git-history-security.md`（历史审计方法论：7节，.gitignore 陷阱/推送前扫描/泄露检测/历史清理工具对比/私有仓库风险/防护检查清单）
- SKILL.md 新增 Phase 4：历史安全审计（推送前快速扫描/全量历史扫描/泄露处理流程/工具选择/私有仓库风险）
- SKILL.md 安全红线新增禁止操作 5（.gitignore ≠ 历史清除）和禁止操作 6（私有仓库 ≠ 安全）
- SKILL.md 常见问题表新增 4 条历史安全相关问题
- SKILL.md 禁止模式新增 3 条历史相关禁止项
- SKILL.md 命令速查新增历史安全命令块
- SKILL.md references 清单新增 git-history-security.md
- golden_rules.md 新增第 9 条：历史比文件更危险
- version 2.1.0 → 2.2.0

**来源**：
- Anthropic 2026-09 威胁情报报告（GTG-50020：公开仓库历史泄露事件）
- trufflehog / git-secrets 工具文档
- 用户分享中转站《Agent安全体系执行建议》§原则5
