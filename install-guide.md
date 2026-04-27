# Velotric Skills 安装指南（v0.6）

以下步骤面向 AI Agent，部分步骤需要用户在浏览器或 Terminal 中配合完成。

## 前提

- macOS（Linux 实验性支持，Windows 暂不支持）
- 用户已装 Codex CLI 或 Claude Code（这些是公司主流 AI 工具，与本指南无关）

> ⚠️ AI 注意：每一步开始前**用一句中文告诉用户在做什么、为什么要做、大概多久**。不要直接闷头跑命令——这是 v0.6 的核心改进，建立用户信任。

## 第 0 步：GitHub 账号 + 组织成员资格检查

```bash
# 检查 git config 推断用户身份
git config user.email 2>/dev/null || echo "(尚未配置)"
git config user.name 2>/dev/null || echo "(尚未配置)"
```

**情况 A：用户已有 GitHub 账号 + 已加入 velotricai 组织**
跑下面验证：
```bash
gh api /orgs/velotricai/members/$(gh api /user -q .login) 2>&1
```
状态 204 = 是成员，跳到第 1 步。

**情况 B：用户已有 GitHub 账号但不是 velotricai 成员**
告诉用户类似下面的话（用他的真实 GitHub 用户名替换）：

> "你还不是 velotricai 组织的成员，需要管理员邀请。我帮你起草请求消息——请把下面这段复制发到团队 Slack 找 perry，或者用你的飞书发给 perry：
>
> ```
> Hi perry，请把我加进 velotricai 组织
> 我的 GitHub 用户名：<github-username>
> 我的工作邮箱：<email>
> 加入后请告诉我，我接到邀请就继续装。
> ```
>
> ⚠️ 注意：GitHub username 跟你的工作邮箱地址**不是同一个东西**——它是你在 GitHub 注册时起的用户名。
>
> perry 收到后跑一行命令就能邀请你。"

**情况 C：用户没有 GitHub 账号**
告诉用户：
> "你还没有 GitHub 账号——这是公司技术工作的标配，我帮你引导：
> 1. 浏览器打开 https://github.com/signup
> 2. 用任何你方便的邮箱注册（个人邮箱也可以，GitHub username 跟邮箱独立）
> 3. 注册完，验证邮箱
> 4. 告诉我注册的 GitHub 用户名，我接着帮你发入组请求"

⚠️ 用户配合点：
- 给 perry 发消息（情况 B / C 都需要）
- perry 接受邀请后，Adam 在邮箱里点 Accept
- 全程异步，可能要等几分钟到几小时

## 第 1 步：环境探测（AI 自己做，缺什么补什么）

在跑下面命令前，**先告诉用户**：
> "我接下来会检查你电脑上有没有 git、Homebrew、GitHub CLI。如果没有，我会装上。其中装 Homebrew 时**会要你输入 macOS 登录密码**——这是 Homebrew 自己的安全要求，不是我要你的密码。装完它就再不要了。"

```bash
# 1. git（macOS 自带或 Xcode CLT 提供）
which git || xcode-select --install   # 弹系统对话框让用户点同意

# 2. Homebrew（包管理器，装其他工具用）
which brew || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 3. GitHub CLI（认证用）
which gh || brew install gh
```

⚠️ 用户配合点：
- **Xcode CLT 系统弹窗**（如果出现）：点"安装"，等 5-10 分钟
- **Homebrew sudo 密码**：输入 macOS 登录密码（不是 GitHub 密码！）

## 第 2 步：配置 GitHub 认证（HTTPS 优先）

**v0.6 改进**：默认走 HTTPS 而非 SSH——SSH 22 端口在很多中国网络被屏蔽，HTTPS 端口 443 都通；gh CLI 帮你 cache token，免去 SSH key 配置。

```bash
# 优先：gh CLI 走 HTTPS 一键登录（token 自动 cache）
gh auth login --git-protocol https --web
gh auth setup-git   # 让 git 用 gh 的 token 做 push/pull
```

⚠️ 用户配合点：浏览器打开 GitHub 授权页 → 用户点 "Authorize"。

**SSH fallback**（如果用户坚持用 SSH）：

```bash
# 测试 SSH 22
if timeout 5 ssh -T -o StrictHostKeyChecking=accept-new git@github.com 2>&1 | grep -q "successfully authenticated"; then
    echo "✓ SSH 22 端口通"
else
    echo "⚠ SSH 22 端口不通，配 ssh.github.com:443 fallback"
    grep -q "Host github.com" ~/.ssh/config 2>/dev/null || cat >> ~/.ssh/config <<'EOF'

Host github.com
    Hostname ssh.github.com
    Port 443
    User git
EOF
    chmod 600 ~/.ssh/config
    ssh-keyscan -p 443 -H ssh.github.com 2>/dev/null >> ~/.ssh/known_hosts
fi
```

## 第 3 步：Clone 仓库 + 安装

**先问用户安装位置**：
> "我准备把公司 skill 仓库 clone 到 `~/velotric-skills`。你想换个位置吗（比如 `~/AI x Velotric/velotric-skills`）？默认就好的话直接说'默认'。"

```bash
DEFAULT_HOME="${VELOTRIC_SKILLS_HOME:-$HOME/velotric-skills}"
# v0.6: HTTPS 优先（避免 SSH 端口屏蔽问题）
git clone https://github.com/velotricai/velotric-skills.git "$DEFAULT_HOME"
bash "$DEFAULT_HOME/velotric-skill-meta/scripts/install.sh"
```

`install.sh` 默认只装 `velotric-skill-meta`（元 skill），其他 skill 装完后 AI 会引导用户**按需选装**。

## 第 4 步：验证

```bash
ls -la "$DEFAULT_HOME/.git" >/dev/null && echo "✓ 仓库已 clone"
ls "${HOME}/.agents/skills/velotric-skill-meta" >/dev/null && echo "✓ skill 已链接"
cat ~/.velotric-skills/installed.txt
```

## 第 5 步：开始用（"hello world"时刻）

跟用户说类似这样的话（用他真实信息）：

> "🎉 装好了，<Adam 的名字>。现在请你**关掉当前 Codex 窗口（Cmd+Q），重新打开**——这是为了让它加载新装的 skill。
>
> 重开后跟它说一句：
> ```
> 我刚装好 velotric skills，给我推荐 3 个跟我工作相关的
> ```
>
> 它会按你的角色推荐合适的 skill，并帮你装上。"

## 故障排查

| 现象 | 怎么办 |
|---|---|
| `Permission denied (publickey)` | SSH key 没配好——v0.6 推荐用 HTTPS 而非 SSH |
| `gh: command not found` | 第 1 步 brew install gh 失败，看 brew 是否装了 |
| `git clone` 提示 `repository not found` | 没加入 velotricai 组织——回第 0 步联系 perry |
| `Connection closed by ... port 22` | SSH 22 端口被屏蔽——切 HTTPS clone |
| Codex/Claude 收不到 skill | **重启 AI 会话**（Cmd+Q 后重开） |

完整问题：跟 AI 说"刚才报错了"，velotric-skill-meta 的 J 分支会引导。

---

**版本**：v0.6
**更新**：2026-04-27
**仓库**：https://github.com/velotricai/velotric-skills（私有，需 org 成员资格）
**维护**：@perry-czp / @velotricai/platform-maintainers
