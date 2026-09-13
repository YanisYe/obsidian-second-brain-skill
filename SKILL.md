---
name: obsidian-second-brain
description: Organize, connect, and safely sync an Obsidian vault.
version: 1.0.0
author: Yanis Ye (YanisYe), Hermes Agent
license: MIT
platforms: [linux, macos]
metadata:
  hermes:
    tags: [obsidian, knowledge-base, notes, git, multi-agent]
    related_skills: []
---

# Obsidian Second Brain

Treat the vault's Markdown files as the canonical knowledge layer. Obsidian is an optional graphical client; terminal agents can read and write the same files on headless machines.

## When to Use

Use this skill when the user asks to:

- save, organize, search, or connect knowledge in an Obsidian vault;
- inspect prior project context before continuing work;
- maintain project entries, detailed notes, and session handoffs;
- share a Markdown knowledge base across hosts through Git.

Do not use it to upload credentials, silently choose an unknown vault, or publish an entire dirty worktree.

## Prerequisites

- An existing Obsidian vault or Markdown knowledge directory.
- A user-provided vault path, or `OBSIDIAN_KNOWLEDGE_REPO` configured for hosts allowed to publish.
- `git` plus an existing upstream and working authentication when Git publication is desired.

A host can opt in by placing this in an existing shell startup file such as `~/.zshrc`, `~/.zprofile`, `~/.bashrc`, `~/.bash_profile`, or `~/.profile`:

```sh
export OBSIDIAN_KNOWLEDGE_REPO="/absolute/path/to/vault"
```

This variable is permission plus a local path, not a credential. Never store GitHub tokens in shell startup files; use a Git credential helper, `gh`, or SSH.

## Resolve the Vault

1. Prefer an explicit path supplied in the current task.
2. Otherwise use the resolved `OBSIDIAN_KNOWLEDGE_REPO` value.
3. Require the directory to exist before reading or writing.
4. If neither source identifies a vault, ask the user instead of guessing.
5. Never bake a username, home directory, repository owner, remote URL, or branch into this skill.

## Read Before Asking

When the user mentions an ongoing project, inspect the vault before asking for context:

1. Search `1_Projects/` for exact names, aliases, and partial matches.
2. Read the matching project's progress or status section.
3. Read the newest relevant note under `z_Archive/Project Notes/<ProjectName>/`.
4. Ask only for information not recoverable from the vault.

The vault is the source of truth for the user's project state, prior decisions, architecture, and unresolved work.

## Default Routing

| Content | Location |
|---|---|
| Raw capture or temporary input | `0_Inbox/` |
| Active project entry | `1_Projects/<ProjectName>.md` |
| Ongoing responsibility or domain | `2_Areas/` |
| General reference or paper note | `3_Resources/` |
| Person note | `4_People/` |
| Reusable note template | `5_Templates/` |
| Knowledge map or vault configuration | `6_Meta/` |
| Session handoff | `7_Automations/会话总结/` |
| Detailed project record | `z_Archive/Project Notes/<ProjectName>/` |
| Inactive material | `z_Archive/` |

A project has one entry in `1_Projects/`. Put diagnoses, architecture decisions, experiment reports, and other detailed records under that project's archive directory and link them from the entry.

## Write Procedure

1. Determine whether the material is a project record, general reference, or raw capture.
2. Search for an existing note before creating a duplicate.
3. Write Markdown with useful frontmatter such as `title`, `created`, `tags`, `summary`, `source`, and `related`.
4. Add at least one resolvable `[[wikilink]]` when the note belongs to the knowledge graph.
5. For a project record, update both the project entry's `related:` list and its dated progress section.
6. For a significant session, write one compressed handoff note.
7. Verify every written path and every expected entry link before reporting success.

Prefer Chinese filenames for Chinese notes, concise semantic names, and tables for structured comparisons. Do not duplicate the same material across multiple notes; link it instead.

## Session Handoff

A significant session includes decisions, code changes, bugs, architecture work, or multi-agent coordination. Write:

```text
7_Automations/会话总结/YYYY-MM-DD-HHmm 会话总结.md
```

Use this compact structure:

```markdown
---
title: HHMM 会话总结
created: YYYY-MM-DD HH:MM
tags: [会话总结, project-name]
summary: One-line conclusion.
---

## 当前状态
- Conclusion or verified state.

## 待处理
- [ ] Unresolved item.

## 下一步
- Immediate next action.

## 给其他 agent
- Context needed for a safe handoff.
```

Never overwrite an earlier session handoff. When taking over an item, mark it in progress and later completed instead of deleting its history.

## Conditional Git Publication

After writing and verifying notes, publish only when the current host explicitly opts in:

1. Require `OBSIDIAN_KNOWLEDGE_REPO` in an existing shell startup file and resolve it to an existing Git worktree.
2. Require every note being published to live inside that configured worktree.
3. Discover the current branch and destination from the worktree's configured upstream. Never create a remote or choose a repository implicitly.
4. Verify authentication without printing credentials.
5. Account for every worktree change. If unrelated or unexplained changes exist, leave the notes local and report `Git sync skipped`.
6. Stage only files written or updated by the current task.
7. Run `git diff --cached --check`, commit the staged files, rebase onto the configured upstream, and push that upstream.
8. On conflict, rejected push, or authentication failure, stop. Never stash, discard, force-push, or include unrelated files automatically.
9. Compare the local commit hash with the upstream hash before claiming synchronization succeeded.

Use `terminal` for Git checks and commands. A typical verified sequence is:

```text
terminal(command="git -C <vault> status --short --branch")
terminal(command="git -C <vault> add -- <exact-written-files>")
terminal(command="git -C <vault> diff --cached --check")
terminal(command="git -C <vault> commit -m '<specific message>'")
terminal(command="git -C <vault> pull --rebase && git -C <vault> push")
```

If the environment variable, Git worktree, upstream, or authentication is missing, do not commit or push. Local note creation may still succeed and must be reported separately from publication.

## Git Ignore Baseline

For a content-focused vault, start with:

```gitignore
.DS_Store
.trash/
.obsidian/
```

This keeps machine-specific workspace state and plugin credentials out of the repository. Users who intentionally share selected Obsidian settings can narrow the `.obsidian/` rule after auditing those files.

## Paper-to-Project Mapping

When a paper must inform an ongoing project, use `references/paper-to-project-mapping.md`. Read the project first, lead with structural correspondence, ground recommendations in evidence, and save the result as a project record rather than an orphaned general note.

## Pitfalls

- **Obsidian is not the data layer.** Markdown and attachments are; do not require the graphical app on servers.
- **An environment variable is not a scheduler.** It authorizes and locates a repository, but an agent, cron job, systemd timer, or other process must still run Git commands.
- **Shell startup files are host-specific.** A headless scheduler may not source them; automation must read the configured value deliberately.
- **Private and public are different trust boundaries.** Scan all staged content before the first public push.
- **Two sync engines can race.** Do not run cloud-drive sync and Git automation against the same vault without a deliberate conflict strategy.
- **Null is not success.** A local write, local commit, and remote push are three distinct states; report which one was verified.

## Verification

Before finishing, verify:

- every note is in the intended vault-relative location;
- project notes are linked from the single project entry;
- the session handoff exists when required;
- no secret or machine-specific state is staged;
- only task-related files are committed;
- local and upstream hashes match after a claimed push;
- skipped publication is reported with the missing prerequisite.
