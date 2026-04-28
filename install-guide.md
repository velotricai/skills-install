# Velotric Skills 安装指南（v0.8.7）

以下步骤面向 AI Agent。macOS 是主支持路径；Windows 10/11 现在有实验性 PowerShell 路径，需要真机确认 Codex / Claude Code 的 skill 加载目录。

> ⚠️ AI 注意：每一步开始前**用一句话告诉用户在做什么、为什么、还有多久**，不要闷头跑命令。这是建立信任的关键。

> ⚠️ 更重要：不要把这份指南整段丢给用户读。用户只需要完成当前一个动作；AI 必须自己读指南、自己执行命令、自己判断下一步。

---

## 第 -2 步：AI 安装向导协议

AI 必须按"一次一个动作"推进，不要让用户自己理解全流程。

每个需要用户配合的步骤都按这个格式说：

```text
我现在需要你做一件事：<动作>。
为什么：<一句话原因>。
完成标志：<你看到什么就算完成>。
做完告诉我"好了"，我会继续检查。
```

执行规则：

1. **不要让用户读步骤**：AI 自己读本文档，用户只看当前动作。
2. **不要连续抛多个要求**：注册、登录、OAuth、收邮件、重启 AI 要拆开。
3. **每步都要验证**：用户说"好了"后，用命令确认，再继续下一步。
4. **不要要求用户解释技术错误**：失败时跑检查命令，给用户一个下一动作。
5. **不要索要 token**：GitHub token / PAT 不得进入 AI 聊天、飞书、文档或截图。
6. **用户卡住时降级为最小动作**：例如"只点 Authorize"，不要解释 OAuth 原理。

用户必须亲手完成的只有这些系统级动作：

| 动作 | AI 应该怎么说 |
|---|---|
| GitHub OAuth | "浏览器会打开 GitHub，请点 Authorize。不要把 token 发给我。" |
| GitHub 组织邀请 | "打开邮箱，点 GitHub 邀请里的 Accept invitation。" |
| macOS 密码 / sudo | "输入你的 Mac 登录密码，屏幕不显示字符是正常的。" |
| Windows UAC / winget | "Windows 会弹确认框，请点允许或安装。" |
| 重启 AI | "关闭当前 Codex / Claude Code 窗口后重新打开，让它加载 skill。" |

如果用户说"我不知道做完没有"，AI 直接重新运行当前步骤的验证命令，不要让用户猜。

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

先尝试自己检查，不要先让用户读说明。跟用户说：

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

**情况 B：USERNAME 拿到但不是成员 → 引导用户提交申请**

直接给用户一个**预填好的 issue URL**：

> "我现在需要你做一件事：打开这个加入申请表单，直接点底部 **Submit Issue**。
>
> 为什么：私有 skill 仓库只对 velotricai 组织成员开放。
>
> 完成标志：页面上出现一个新的 issue，或看到机器人评论。
>
> https://github.com/velotricai/skills-install/issues/new?template=join-org.yml&title=%5B%E5%8A%A0%E5%85%A5%E7%94%B3%E8%AF%B7%5D
>
> 做完告诉我"好了"，我会继续检查。"

用户说"好了"后，AI 等 30 秒，再检查：

```bash
gh api /orgs/velotricai/members/$USERNAME --silent
```

如果还不是成员，继续给用户一个动作：

> "我现在需要你做一件事：打开 GitHub 注册邮箱，找到 velotricai 的邀请邮件，点 **Accept invitation**。
>
> 为什么：提交申请只是发出邀请，必须接受后才有私有仓库权限。
>
> 完成标志：GitHub 页面显示你已加入组织。做完告诉我"好了"。"

用户说"好了"后再检查组织成员资格。2 分钟内仍失败，才让用户把 GitHub username 和申请 issue 链接发给平台维护者。

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

如果 `winget install --id GitHub.cli -e` 因为 `msstore` 源证书、网络或商店源问题失败，不要让用户自己排查，直接改用 winget 官方源：

```powershell
winget install --id GitHub.cli -e --source winget
```

如果安装完成后当前 PowerShell 仍提示 `gh` 找不到，通常只是 PATH 没刷新。AI 先自动查找常见路径，不要要求用户重启电脑：

```powershell
$GhPath = (Get-Command gh.exe -ErrorAction SilentlyContinue).Source
if (-not $GhPath) {
  $GhPath = Get-ChildItem 'C:\Program Files','C:\Users\*\AppData\Local\Programs' -Recurse -Filter gh.exe -ErrorAction SilentlyContinue |
    Select-Object -First 1 -ExpandProperty FullName
}
if (-not $GhPath) { throw "GitHub CLI 已安装但当前会话找不到 gh.exe，请打开新 PowerShell 后重试。" }
& $GhPath --version
```

后续本页里的 `gh ...` 命令，在 Windows 上如果当前会话找不到 `gh`，就用 `& $GhPath ...` 代替。

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

跟用户说：

> "我现在需要你做一件事：浏览器会打开 GitHub，请点 **Authorize**。
>
> 为什么：GitHub CLI 需要得到你本人授权，才能 clone 公司私有仓库。
>
> 完成标志：终端显示 Logged in 或我这边能读到你的 GitHub 用户名。
>
> 不要把 GitHub token 发给我。做完告诉我'好了'。"

⚠️ 安全红线：不要把 GitHub token / PAT 粘贴到 AI 聊天、飞书、文档或截图里。`gh auth login` 的授权应该在浏览器或终端里的 GitHub CLI 提示中完成。如果网络或浏览器回传超时，先重试 `gh auth login --git-protocol https --web`，并按命令输出的 device code 到 GitHub 页面完成授权；不要把 token 发给 AI。

如果用户已经把 token 发进聊天或文档，立刻停止继续使用该 token，并让用户去 GitHub 撤销后重新授权。

用户说"好了"后，AI 必须验证：

```bash
gh auth status
gh api /user -q .login
```

如果浏览器授权没有回传，不要让用户生成 token。只给一个动作：

> "这次授权没有回传。我会重新发起登录；如果终端显示 8 位 device code，你只需要复制这个 code 到 GitHub 页面，不要生成 token。"

如果第二次仍然因为 `github.com/login/device/code` 网络超时，不要切到 PAT。按这个顺序处理：

1. 让用户换网络、开手机热点或公司可访问 GitHub 的网络后重试。
2. 仍失败时，让用户把 GitHub username 发给平台维护者，由维护者协助完成登录或现场处理。
3. 普通用户安装流程禁止引导创建 classic PAT；PAT 只允许平台维护者在受控环境里使用，且不得进入聊天记录。

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

如果用户是 Adam / CEO / 高层 owner，首装后的第一句不要让他看仓库，也不要让他先学 GitHub。让他直接验证两件事：公司 skill 目录能不能出现，以及他自己的 skill 如何进入公司机制。

跟他说：

> "如果你是 Adam，重开 AI 后直接说这句：
>
> \`\`\`
> 公司有什么 skill？我是 Adam，下一步怎么把我的 business thinking skill 第一版发到公司仓库。
> \`\`\`
>
> 预期结果：AI 应该先说明现在默认只装了 `velotric-skill-meta`，然后告诉你可以把本地文件夹、截图、会议纪要或一段说明交给它；AI 会帮你整理成 `SKILL.md`、补版本和 owner、跑审计并开 PR。你不需要先懂 Git、YAML 或 PR。"

如果用户是招聘 / HR 场景，让他试这个：
> "用 ops-hiring-workflow 帮我初始化一个岗位：[岗位名 + 简单背景]"

---

## 故障排查

| 现象 | 怎么办 |
|---|---|
| 2 分钟内没收到 GitHub 邀请 | 飞书 @ perry，提供你的 GitHub username 和申请 issue 链接 |
| `gh: command not found` | macOS 检查 Homebrew；Windows 用 `winget install --id GitHub.cli -e` |
| Windows 安装 gh 时 msstore 源报错 | 改用 `winget install --id GitHub.cli -e --source winget` |
| Windows 安装 gh 后当前窗口仍找不到 | 自动查找 `C:\Program Files\GitHub CLI\gh.exe`，或打开新 PowerShell |
| `gh auth login` 浏览器没弹 | 复制命令输出的 8 位 device code，手动粘到弹出的 URL |
| `github.com/login/device/code` 网络超时 | 换网络/热点后重试；仍失败时找平台维护者，不走普通用户 PAT 路径 |
| 用户把 GitHub token 发进聊天 | 立即去 GitHub 撤销该 token；重新用 `gh auth login --git-protocol https --web` 登录，不要在聊天里传 token |
| Codex/Claude 收不到 skill | 关掉 AI 窗口（Cmd+Q）重开，让它重新加载 skill |
| Windows 上 `vsk-list` 不存在 | 开新 PowerShell，或运行 `. $PROFILE` |
| Windows 上链接失败 | `install.ps1` 会自动从 Junction fallback 到复制；仍失败时检查用户目录权限 |
| 装到一半报错 | 在 Terminal 跑 `vsk-status` 一行命令，把输出贴给 perry |

完整诊断：跟你的 AI 说"刚才报错了"，velotric-skill-meta 的 J 分支会引导。

---

**版本**：v0.8.7
**更新**：2026-04-28
**仓库**：https://github.com/velotricai/velotric-skills（私有，需 org 成员资格）
**维护**：@PerryChen / @velotricai/platform-maintainers
