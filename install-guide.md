# Velotric Skills 安装指南（v0.8.4）

以下步骤面向 AI Agent。macOS 是主支持路径；Windows 10/11 现在有实验性 PowerShell 路径，需要真机确认 Codex / Claude Code 的 skill 加载目录。

> ⚠️ AI 注意：每一步开始前**用一句话告诉用户在做什么、为什么、还有多久**，不要闷头跑命令。这是建立信任的关键。

---

## 第 -1 步：先识别操作系统

AI 先判断用户机器是 macOS / Windows / Linux，再走对应分支。不要在 Windows 上跑 Homebrew / xcode-select / `.zshrc` 命令。

```bash
uname -s 2>/dev/null || ver
```

- `Darwin` → 走 macOS 分支
- `MINGW*` / `MSYS*` / `CYGWIN*` / `Windows` → 走 Windows PowerShell 分支
- `Linux` → 可试 macOS/Linux bash 分支，但先提醒“实验性”

---

## 第 0 步：确认入组（最快路径）

跟用户说：

> "你需要是 velotricai GitHub 组织的成员才能装公司 skill。我先帮你检查并申请——只要 30 秒。"

```bash
# 拿当前 gh 登录用户的 username
USERNAME=$(gh api /user -q .login 2>/dev/null || true)
```

**情况 A：USERNAME 拿到了 + 已是成员**
```bash
gh api /orgs/velotricai/members/$USERNAME --silent
```
返回 204 → 跳到第 1 步。

**情况 B：USERNAME 拿到但不是成员 → 自动申请入组**

直接给用户一个**预填好的 issue URL**：

> "我给你打开一个加入申请表单——你不用填任何东西，**直接点 Submit Issue**：
>
> https://github.com/velotricai/skills-install/issues/new?template=join-org.yml&title=%5B%E5%8A%A0%E5%85%A5%E7%94%B3%E8%AF%B7%5D
>
> 提交后等 30 秒，机器人自动给你发邀请邮件。
>
> 如果 2 分钟内没收到 issue 评论或邀请邮件，让用户把 GitHub username 发给 perry。"

**情况 C：用户还没登录 GitHub / 没安装 gh**
告诉用户：
> "你的 GitHub 账号还没准备好。我先帮你装 GitHub CLI 并登录（约 5 分钟）。"

跳到第 1 步，做完再回到 0 步检查 membership。

---

## 第 1 步：环境准备

### macOS 分支

跟用户说：

> "我接下来会装 GitHub CLI（gh）。装的时候**会要你输一次 macOS 登录密码**——这是 macOS 自己的安全机制，不是我要你的密码。一次性事件，再也不会要了。"

```bash
which git || xcode-select --install   # macOS 自带或弹窗装 Xcode CLT
which brew || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
which gh   || brew install gh
```

⚠️ 用户配合点：
- Xcode CLT 系统弹窗（如果出现）：点"安装"，等 5-10 分钟
- Homebrew sudo 密码：输 macOS 登录密码（**不是 GitHub 密码**）

### Windows PowerShell 分支

跟用户说：

> "我接下来会检查 Windows 上有没有 Git 和 GitHub CLI。如果没有，我会用 winget 安装。Windows 可能会弹一次安装确认或 UAC 授权。"

在 PowerShell 里执行：

```powershell
git --version
gh --version
```

如果缺失：

```powershell
winget install --id Git.Git -e
winget install --id GitHub.cli -e
```

⚠️ 用户配合点：
- Windows 可能弹出安装确认 / UAC 授权
- 如果没有 winget，让用户先从官网安装 Git for Windows 和 GitHub CLI，再继续

---

## 第 2 步：GitHub 认证（HTTPS 默认）

### macOS / Windows 通用

```bash
# v0.8 改进：默认走 HTTPS（避免 SSH 22 端口屏蔽问题，国内网络更稳）
gh auth login --git-protocol https --web
gh auth setup-git    # 让 git 用 gh 的 token 做 push/pull
```

⚠️ 用户配合点：浏览器打开 GitHub 授权页 → **点 Authorize**。

---

## 第 3 步：Clone 仓库 + 安装

### macOS 分支

```bash
# 默认装到 ~/velotric-skills（不问用户，避免增加心智成本）
if [ -d ~/velotric-skills/.git ]; then
  git -C ~/velotric-skills pull --ff-only origin main
else
  if [ -e ~/velotric-skills ]; then
    mv ~/velotric-skills ~/velotric-skills.velotric-bak.$(date +%Y%m%d-%H%M%S)
  fi
  gh repo clone velotricai/velotric-skills ~/velotric-skills
fi
bash ~/velotric-skills/velotric-skill-meta/scripts/install.sh
```

`install.sh` 默认只装 **velotric-skill-meta**。业务 skill 按需安装，避免新同事一上来装一堆暂时用不到的东西。

如果用户当前就是招聘 / HR 场景，继续执行：

```bash
bash ~/velotric-skills/velotric-skill-meta/scripts/install.sh --add ops-hiring-workflow
```

### Windows PowerShell 分支

```powershell
$DefaultHome = if ($env:VELOTRIC_SKILLS_HOME) { $env:VELOTRIC_SKILLS_HOME } else { Join-Path $env:USERPROFILE "velotric-skills" }
if (Test-Path (Join-Path $DefaultHome ".git")) {
  git -C $DefaultHome pull --ff-only origin main
} else {
  if (Test-Path $DefaultHome) {
    Move-Item -Path $DefaultHome -Destination "$DefaultHome.velotric-bak.$(Get-Date -Format 'yyyyMMdd-HHmmss')" -Force
  }
  gh repo clone velotricai/velotric-skills $DefaultHome
}
powershell -ExecutionPolicy Bypass -File "$DefaultHome\velotric-skill-meta\scripts\install.ps1"
```

`install.ps1` 默认只装 **velotric-skill-meta**，并注册 PowerShell profile 函数：`vsk-update` / `vsk-list` / `vsk-add` / `vsk-remove`。

如果用户当前就是招聘 / HR 场景，继续执行：

```powershell
powershell -ExecutionPolicy Bypass -File "$DefaultHome\velotric-skill-meta\scripts\install.ps1" -Add ops-hiring-workflow
```

> ⚠️ **Junction 失败回退提示**：如果 install 输出 `junction failed, falling back to copy`（多发生在无管理员权限或跨卷场景），后续 `vsk-update` 拉到的更新**不会**自动反映到 skill 加载目录——必须重新跑 `install.ps1` 才能生效。建议解决方法：以管理员身份运行 PowerShell 重装一次，让 Junction 创建成功，之后 `vsk-update` 就是即时生效。

---

## 第 4 步：验证

### macOS 分支

```bash
ls -la ~/velotric-skills/.git >/dev/null && echo "✓ 仓库已就位"
ls ~/.agents/skills/velotric-skill-meta >/dev/null && echo "✓ velotric-skill-meta 已链接"
test -e ~/.agents/skills/ops-hiring-workflow && echo "✓ ops-hiring-workflow 已链接（如果你选择安装招聘 skill）" || true
```

### Windows PowerShell 分支

```powershell
Test-Path "$DefaultHome\.git"
Test-Path "$env:USERPROFILE\.agents\skills\velotric-skill-meta\SKILL.md"
if (Test-Path "$env:USERPROFILE\.agents\skills\ops-hiring-workflow\SKILL.md") { "✓ ops-hiring-workflow 已链接（如果你选择安装招聘 skill）" }
Get-Content "$env:USERPROFILE\.velotric-skills\installed.txt"
```

---

## 第 5 步：让用户立即体验（aha moment）

⚠️ 重要：**重启 Codex / Claude Code 会话**让它加载新装的 skill。macOS 可以 Cmd+Q；Windows 直接关闭窗口后重新打开。

跟用户说：

> "🎉 装好了，<用户名字>！
>
> 你现在已经装好 Velotric 的 skill 管理入口。接下来先让 AI 列出公司可用 skill，再按你的角色安装具体 skill。
>
> **请你做两件事**：
> 1. 关掉当前 Codex 窗口（Cmd+Q），重开
> 2. 跟它说一句话试试看：
>
>    \`\`\`
>    公司有什么 skill？我是做 [你的岗位/场景] 的，推荐我装哪个。
>    \`\`\`
>
> 这是你今天接下来 30 秒能验证装上是否成功的最快方式。"

如果用户是招聘 / HR 场景，让他试这个：
> "用 ops-hiring-workflow 帮我初始化一个岗位：[岗位名 + 简单背景]"

---

## 故障排查

| 现象 | 怎么办 |
|---|---|
| 2 分钟内没收到 GitHub 邀请 | 飞书 @ perry，提供你的 GitHub username 和申请 issue 链接 |
| `gh: command not found` | macOS 检查 Homebrew；Windows 用 `winget install --id GitHub.cli -e` |
| `gh auth login` 浏览器没弹 | 复制命令输出的 8 位 device code，手动粘到弹出的 URL |
| Codex/Claude 收不到 skill | 关掉 AI 窗口（Cmd+Q）重开，让它重新加载 skill |
| Windows 上 `vsk-list` 不存在 | 开新 PowerShell，或运行 `. $PROFILE` |
| Windows 上链接失败 | `install.ps1` 会自动从 Junction fallback 到复制；仍失败时检查用户目录权限 |
| 装到一半报错 | 在 Terminal 跑 `vsk-status` 一行命令，把输出贴给 perry |

完整诊断：跟你的 AI 说"刚才报错了"，velotric-skill-meta 的 J 分支会引导。

---

**版本**：v0.8.4
**更新**：2026-04-28
**仓库**：https://github.com/velotricai/velotric-skills（私有，需 org 成员资格）
**维护**：@PerryChen / @velotricai/platform-maintainers
