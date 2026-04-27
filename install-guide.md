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

**情况 B：用户已有 GitHub 账号但不是 velotricai 成员 → 自动邀请流程**

跟用户说类似下面的话（用他的真实 GitHub 用户名 `<USERNAME>` 替换）：

> "你还不是 velotricai 成员。我帮你打开一个**自动入组申请**——浏览器里会有个表单，你只需要：
>
> 1. 我已经帮你预填好了 GitHub username（`<USERNAME>`），你检查一下
> 2. 选你的部门
> 3. 点 **Submit**
> 4. 等约 30 秒，机器人会自动给你发邀请邮件
> 5. 去你 GitHub 注册邮箱点 **Accept invitation**
>
> 全程**不需要打扰任何人**——是 GitHub Actions 机器人处理的。
>
> 链接（点开就是）：
> https://github.com/velotricai/skills-install/issues/new?template=join-org.yml&title=%5B%E5%8A%A0%E5%85%A5%E7%94%B3%E8%AF%B7%5D"

⚠️ 关键澄清：GitHub username 跟工作邮箱**不是一个东西**——它是 GitHub 注册时起的用户名（如 `AdamWang`、`PerryChen`）。

**情况 C：用户没有 GitHub 账号**
告诉用户：
> "你还没有 GitHub 账号——这是公司技术工作的标配。我引导你：
> 1. 浏览器打开 https://github.com/signup
> 2. 用任何你方便的邮箱注册（个人邮箱也行，GitHub username 跟邮箱完全独立）
> 3. 注册完验证邮箱
> 4. 注册完成后告诉我，我接着走情况 B 的自动入组流程"

⚠️ 用户配合点：
- 提交入组 issue 表单（1 click），或注册 GitHub 账号（5 分钟）
- 检查邮箱点 **Accept invitation**
- **不依赖管理员手动操作**——GitHub Actions 自动处理 1 分钟内完成

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

## 第 2 步：配置 GitHub 认证（含 SSH 22 端口屏蔽 fallback）

**先用 gh CLI 一键登录**（最简单，覆盖 SSH + HTTPS 两种 git 协议）：

```bash
gh auth login --git-protocol ssh --web
```

⚠️ 用户配合点：浏览器打开 GitHub 授权页 → 用户点 "Authorize"。

**测试 SSH 连通性**——很多中国网络 SSH 22 端口被屏蔽：

```bash
# 默认端口测试（5 秒超时）
if timeout 5 ssh -T -o StrictHostKeyChecking=accept-new git@github.com 2>&1 | grep -q "successfully authenticated"; then
    echo "✓ SSH 22 端口通"
else
    echo "⚠ SSH 22 端口不通，自动配 ssh.github.com:443 fallback"
    # 走 GitHub 推荐的 HTTPS 端口 SSH（GFW / 公司防火墙都不屏蔽）
    grep -q "Host github.com" ~/.ssh/config 2>/dev/null || cat >> ~/.ssh/config <<'EOF'

Host github.com
    Hostname ssh.github.com
    Port 443
    User git
EOF
    chmod 600 ~/.ssh/config
    # 把 ssh.github.com 的 host key 加入 known_hosts，避免交互式确认
    ssh-keyscan -p 443 -H ssh.github.com 2>/dev/null >> ~/.ssh/known_hosts
    ssh -T git@github.com
fi
```

⚠️ 这一步如果失败，告诉用户："你的网络可能屏蔽了 GitHub。试试连公司 VPN 或换网络。"

## 第 3 步：Clone 仓库 + 安装

**先问用户安装位置**：

> "我准备把公司 skill 仓库 clone 到 `~/velotric-skills`。你想换个位置吗（比如 `~/AI x Velotric/velotric-skills`）？默认就好的话直接说'默认'。"

```bash
DEFAULT_HOME="${VELOTRIC_SKILLS_HOME:-$HOME/velotric-skills}"
git clone git@github.com:velotricai/velotric-skills.git "$DEFAULT_HOME"
bash "$DEFAULT_HOME/velotric-skill-meta/scripts/install.sh"
```

`install.sh` 默认只装 `velotric-skill-meta`（元 skill），其他 skill 装完后 AI 会引导用户**按需选装**。

## 第 4 步：验证

```bash
# 不依赖 alias —— alias 在新 shell 才生效
ls -la "$DEFAULT_HOME/.git" >/dev/null && echo "✓ 仓库已 clone"
ls "${HOME}/.agents/skills/velotric-skill-meta" >/dev/null && echo "✓ skill 已链接"
cat ~/.velotric-skills/installed.txt
```

## 第 5 步：开始用

⚠️ **重要的"hello world"时刻**：

跟用户说类似这样的话（用他真实信息）：

> "🎉 装好了，<Adam 的名字>。我看了下你的 GitHub profile，你看起来做 <推断出的角色>。
>
> 现在请你**关掉当前 Codex 窗口（Cmd+Q），重新打开**——这是为了让它加载新装的 skill。
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
| `Permission denied (publickey)` | SSH key 没配好 / 没加入 velotricai 组织 |
| `gh: command not found` | 第 1 步 brew install gh 失败，看 brew 是否装了 |
| `git clone` 提示 `repository not found` | 没加入 velotricai 组织——回第 0 步联系 perry |
| `Connection closed by ... port 22` | SSH 22 端口被网络屏蔽——回第 2 步走 ssh.github.com:443 |
| Codex/Claude 收不到 skill | **重启 AI 会话**（Cmd+Q 后重开） |
| 跑 install.sh 后 alias 不生效 | 跑 `source ~/.zshrc` 或开新 Terminal |

完整问题：跟 AI 说"刚才报错了"，velotric-skill-meta 的 J 分支会引导。

---

**版本**：v0.6
**更新**：2026-04-27
**仓库**：https://github.com/velotricai/velotric-skills（私有，需 org 成员资格）
**维护**：@perry-czp / @velotricai/platform-maintainers
