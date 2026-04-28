# AGENTS.md

## Scope

These instructions apply to the entire `velotricai/skills-install` repository.

## Repository Intent

This is the public one-line entrypoint for Velotric Skills. It must remain safe to show to any employee or external reviewer. The actual skill content lives in the private `velotricai/velotric-skills` repository and requires organization membership.

## Review Guidelines

When reviewing pull requests, prioritize issues that can break first install, leak private information, or increase perry's manual support load.

Flag as high priority:

- The one-line install prompt becomes harder for a non-technical employee to copy, understand, or paste into an AI agent.
- Public docs expose secrets, PAT values, webhook URLs, private production configs, `.env` content, or internal data beyond the expected private repo name.
- The guide stops matching the real installer path in `velotricai/velotric-skills`.
- macOS and Windows install instructions drift apart in behavior without a clear reason.
- A workflow uses a secret but echoes it, writes it to logs, or assumes it exists without a clear failure message.
- The default install path changes from "install meta first, add business skills by role" to "install everything".
- Auto-invite workflow changes weaken auditability: issue comments, labels, invite target team, or Feishu notification behavior.
- The public repo starts duplicating private skill content instead of linking to the private monorepo.

Check install docs for:

- Clear separation between public bootstrap repo and private skill monorepo.
- GitHub org membership flow remains understandable.
- Failure recovery tells the user what to do next without requiring git knowledge.
- Windows instructions mention PowerShell, GitHub CLI, profile reload, and possible Junction/copy behavior.

Check workflows for:

- `install-cross-platform.yml` validates the real private installers on `macos-latest` and `windows-latest`.
- `VELOTRIC_SKILLS_PAT`, `ORG_INVITE_TOKEN`, and `FEISHU_NOTIFY_WEBHOOK` are referenced only as secrets.
- Scheduled or push-triggered workflows do not require personal machine state.

Do not require Codex review to be the final merge authority. Treat Codex review as an additional reviewer; deterministic CI remains the hard gate, and the owner/maintainer makes the final call.
