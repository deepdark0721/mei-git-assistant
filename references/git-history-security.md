# Git 历史安全审计

> 来源：Anthropic 2026-09 威胁情报报告 GTG-50020 + 公开仓库历史泄露事件 + 实战经验
> 核心观点：**Git 历史不是"已删除"，而是"仍在但不再显示"。**

---

## 一、为什么历史比文件更危险

当前文件的泄露容易被发现（`.env` 在目录里一目了然），但 Git 历史里的泄露隐蔽得多：

| 场景 | 当前文件 | Git 历史 |
|------|---------|---------|
| `.env` 已被删除 | 看不见 | `git log -p` 里完整保留 |
| API Key 被替换为占位符 | 安全 | 旧版 Key 仍在历史中 |
| `.gitignore` 已配置 | 新文件不再追踪 | **已在历史中的文件不受影响** |
| 仓库设为私有 | 外部不可见 | 合作者/fork/API/平台漏洞仍可访问 |

**GTG-50020 案例**：攻击者从公开 GitHub 仓库的历史提交中提取被删除的 API Key，用于攻击云基础设施。

---

## 二、`.gitignore` 的五个认知陷阱

| 陷阱 | 真相 |
|------|------|
| 加了 `.env` 到 `.gitignore` 就安全了 | 只管**未来**的新文件，已在历史里的不管 |
| `git rm --cached .env` 删了追踪 | 文件从当前版本消失，但历史里还有完整内容 |
| 仓库转私有了 | 历史对合作者/fork/平台仍可见 |
| 删了仓库重建了 | 如果有人 fork 过，历史在 fork 里 |
| BFG 清理过了 | GitHub 缓存可能仍有残留（需联系 GitHub 支持清除） |

---

## 三、推送前历史扫描

**什么时候做**：首次推送到远程、仓库从私有转公开、接手他人仓库。

### 扫描方法一：grep 全量 diff（最可靠）

```bash
# 扫描所有历史提交中的敏感模式
git log -p --all | grep -E "(sk-[a-zA-Z0-9]{20,}|AKIA[A-Z0-9]{16}|ghp_[a-zA-Z0-9]{36}|xoxb-|-----BEGIN)" | head -50
```

常用模式：
- `sk-` / `sk-proj-`：OpenAI API Key
- `AKIA`：AWS Access Key
- `ghp_` / `gho_` / `ghu_` / `ghs_`：GitHub Token
- `xoxb-` / `xoxp-`：Slack Token
- `-----BEGIN`：私钥/证书
- `password` / `passwd` / `secret`：密码字段

### 扫描方法二：git-secrets（AWS 出品，推荐）

```bash
# 安装
brew install git-secrets  # macOS
# 或从源码安装

# 配置扫描模式（一次性）
git secrets --install
git secrets --register-aws  # AWS 密钥模式

# 扫描当前仓库
git secrets --scan

# 扫描全量历史
git secrets --scan-history
```

### 扫描方法三：trufflehog（社区工具，覆盖更广）

```bash
# 安装
pip install trufflehog

# 扫描本地仓库全量历史
trufflehog git file://. --only-verified

# 扫描远程仓库（无需 clone）
trufflehog git https://github.com/user/repo
```

### 扫描方法四：GitHub Secret Scanning（远程仓库）

- GitHub 仓库 Settings → Security → Secret scanning
- 公开仓库自动扫描
- 私有仓库需 GitHub Advanced Security（付费）

---

## 四、已有泄露的检测流程

### Step 1：确认泄露范围

```bash
# 找出所有包含敏感字符串的提交
git log --all --oneline | while read hash msg; do
  if git show "$hash" | grep -q "sk-proj-xxx"; then
    echo "$hash $msg"
  fi
done
```

### Step 2：判断是否已在远程

```bash
# 检查泄露提交是否已推送到远程
git branch -r --contains <commit-hash>
```

- 未推送 → 只需本地清理（`git rebase -i` 或 `git filter-repo`）
- 已推送 → 本地清理 + 强制推送 + **立即轮换泄露的凭据**

### Step 3：轮换凭据（最高优先级）

**历史清理是第二步，轮换凭据是第一步。**

| 泄露类型 | 动作 |
|---------|------|
| API Key | 立即在服务商控制台撤销并重新生成 |
| 密码 | 立即修改密码 |
| 私钥 | 重新生成密钥对，废弃旧私钥 |
| Token | 撤销 Token，重新授权 |

---

## 五、历史清理工具对比

| 工具 | 推荐度 | 适用场景 | 副作用 |
|------|--------|---------|--------|
| **git filter-repo** | ⭐⭐⭐ 首选 | 通用历史重写 | 改写所有受影响的 commit hash |
| BFG Repo-Cleaner | ⭐⭐ | 简单的文件/字符串删除 | 同上 |
| `git filter-branch` | ⛔ 已废弃 | 无 | 极慢、易出错、官方不推荐 |

### git filter-repo 操作步骤

```bash
# 安装
pip install git-filter-repo

# 删除特定文件的所有历史记录
git filter-repo --path .env --invert-paths

# 替换所有历史中的敏感字符串
git filter-repo --replace-text <(echo "sk-proj-xxx==>***REMOVED***")

# 强制推送到远程（需确认）
git push --force --all
git push --force --tags
```

**铁律**：历史重写会改写所有受影响的 commit hash。协作项目中，所有合作者必须重新 clone，不能在旧仓库上 pull。

---

## 六、私有仓库的风险清单

"设为私有就安全了"是最危险的假设：

| 风险来源 | 说明 |
|---------|------|
| 合作者权限 | 所有合作者都能看到完整历史（含已删除的敏感文件） |
| Fork | 如果仓库曾被 fork，fork 里保留了当时的完整历史 |
| GitHub API | 有 Token 的第三方应用可以读取仓库全部内容 |
| 平台漏洞 | GitHub 自身的安全事件可能暴露私有仓库 |
| 转公开 | 仓库从私有转公开时，历史中的所有内容瞬间可见 |
| GitHub Secret Scanning 限制 | 私有仓库需付费功能；免费版不扫描私有仓库 |

---

## 七、日常防护检查清单

推送前过一遍（每次 `git push` 前 30 秒）：

- [ ] `git diff --cached` 里有没有 API Key / 密码 / Token？
- [ ] `.gitignore` 是否覆盖了 `.env`、`*.key`、`*.pem`？
- [ ] 如果是**首次推送**到远程，是否做过历史扫描？
- [ ] 如果是从**私有转公开**，是否扫描了全部历史？
- [ ] 如果接手他人仓库，是否做过 `git secrets --scan-history`？
