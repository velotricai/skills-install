# Velotric Skills 安装指南

以下步骤面向 AI Agent，部分步骤需要用户在浏览器或 Terminal 中配合完成。

## 前提

- macOS（Linux 实验性支持，Windows 暂不支持）
- 用户已有 GitHub 账号且**已被邀请加入 velotricai 组织**（如未加入，告诉用户先去 https://github.com/velotricai/people 看自己有没有 invitation 邮件，或联系 perry）
- 用户已装 Codex CLI 或 Claude Code（这些是公司主流 AI 工具，与本指南无关）

## 第 1 步：环境探测（AI 自己做，缺什么补什么）

```bash
# 检查并安装基础工具
which git || xcode-select --install   # macOS Xcode CLT 提供 git，会弹窗让用户同意
which brew || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"   # 装 Homebrew，会要 sudo 密码
which gh || brew install gh           # GitHub CLI
```

⚠️ 用户配合点：
- **Xcode CLT 系统弹窗**：需要用户点"同意"
- **Homebrew sudo 密码**：需要用户输入登录密码
- 这两步任何 AI 都代办不了

## 第 2 步：配置 GitHub 认证

```bash
# 优先方案：用 gh CLI 一键登录（覆盖 SSH + HTTPS）
gh auth login --git-protocol ssh --web
```

⚠️ 用户配合点：浏览器会打开一个 GitHub 授权页，用户需点击 "Authorize"。

如果 `gh auth login` 不可用，回退到手动配 SSH：

```bash
# 生成 SSH key
ssh-keygen -t ed25519 -C "$(git config user.email || echo $USER@velotric.com)" -f ~/.ssh/id_ed25519 -N ""
# 把公钥复制到剪贴板
pbcopy < ~/.ssh/id_ed25519.pub
echo "公钥已复制到剪贴板。请在浏览器打开 https://github.com/settings/ssh/new 粘贴并保存。"
```

⚠️ 用户配合点：用户必须在浏览器粘贴公钥到 GitHub Settings → SSH keys。

## 第 3 步：Clone 仓库 + 安装

```bash
# 默认装到 ~/velotric-skills（用户可改）
DEFAULT_HOME="${VELOTRIC_SKILLS_HOME:-$HOME/velotric-skills}"
git clone git@github.com:velotricai/velotric-skills.git "$DEFAULT_HOME"
bash "$DEFAULT_HOME/velotric-skill-meta/scripts/install.sh"
```

`install.sh` 会自动：
- 把所有 skill 软链接到 `~/.agents/skills/`、`~/.codex/skills/`、`~/.claude/skills/`
- 在 `~/.zshrc` 注册 `VELOTRIC_SKILLS_HOME` env var、自动更新检查、`vsk-update` / `vsk-list` 等 alias
- 备份原 `.zshrc` 到 `.zshrc.velotric-bak.<timestamp>`

## 第 4 步：验证

```bash
# 重启 shell 让 env var 生效
source ~/.zshrc

# 查装了哪些 skill
vsk-list

# 应该看到至少：
# - velotric-skill-meta
```

## 第 5 步：开始用

⚠️ 重要：**重启你的 Codex 或 Claude Code 会话**，让它加载新装的 skill。然后用自然语言对话即可，不需要再敲命令：

```
"公司有什么 skill"            → 浏览公司 skill 库
"帮我做个 skill 看周报"        → 创建新 skill
"用 personal-adam-business-thinking 看下这个增长方案"
                              → 直接调用某个 skill
"velotric 有更新吗"           → 检查/拉取新版本
"卸了 personal-adam-business-thinking-updater"
                              → 移除某个 skill
"刚才报错了"                  → 报错恢复（J 分支）
```

完整的提示词路由表见装好后的 `velotric-skill-meta/references/user-prompts.md`。

## 故障排查

| 现象 | 怎么办 |
|---|---|
| `Permission denied (publickey)` | SSH key 没配好，回到第 2 步 |
| `gh: command not found` | 第 1 步 brew install gh 失败，看 brew 是否装了 |
| `git clone` 提示 repository not found | 用户还没加入 velotricai 组织，找 perry 申请 |
| Codex/Claude 收不到 skill | 重启 AI 会话；检查 `vsk-list` 是否有输出 |
| 跑 install.sh 后 alias 不生效 | 跑 `source ~/.zshrc` 或开新 Terminal |

完整问题：跟你的 AI 说"刚才报错了"，AI 会按 velotric-skill-meta 的 J 分支引导。

## 装完后让 AI 知道你想做什么

一些起手提示词：

```
我是 <部门> 的 <名字>，刚装好 velotric skills。
帮我看看公司有哪些 skill 跟我工作相关。
```

```
我有一个 skill 在 ~/Downloads/<name>，帮我审计 + 发布到团队。
```

---

**版本**：v0.1
**更新**：2026-04-27
**仓库**：https://github.com/velotricai/velotric-skills（私有，需 org 成员资格）
**维护**：@perry-czp / @velotricai/platform-maintainers
