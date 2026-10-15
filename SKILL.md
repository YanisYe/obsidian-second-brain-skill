---
name: obsidian-second-brain
description: Organize, connect, and safely sync an Obsidian vault.
version: 1.0.1
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

## Profile-Level Availability

Install this skill in the **active Hermes profile's** `skills/note-taking/obsidian-second-brain/`, not inside an individual project or worktree. Resolve the active profile home using the host's supported configuration; the default profile commonly uses `~/.hermes`, while named profiles have their own home. Do not install into or modify other profiles implicitly.

Profile-level installation makes the skill discoverable across that profile's sessions, worktrees and projects. It does not automatically load the full skill or synchronize the vault on every turn. When authorized, store a compact discovery pointer in the profile's supported persistent memory/instruction mechanism: use this skill for project recall and knowledge capture, and resolve the configured vault before work. Keep project-specific rules in their own project notes; do not carry one project's decisions into another.

After installation, verify discovery in a fresh session. A project-local AGENTS.md is not a substitute for profile-wide discovery. Do not hardcode a personal vault path into the distributed skill.

## On-Demand Refresh Before Reading or Writing

Use task-triggered synchronization, not periodic polling. At the start of a task that needs existing vault context or will update notes, resolve the vault and refresh it **before reading notes used for decisions or drafting edits**.

For an explicitly configured Git vault:
1. Inspect `git status --porcelain`, the current branch and its configured upstream. Require an attached branch and known upstream; do not choose a remote or branch implicitly.
2. Inspect and preserve uncommitted changes and any merge/rebase in progress. Use **Conflict Recovery** below to reconcile task-related changes or isolate this task from unrelated edits; a dirty worktree is not a reason to abandon knowledge capture. Do not blindly pull, auto-stash, discard, or commit unrelated edits.
3. With a clean worktree, run `git pull --ff-only` against the configured upstream. On divergence or conflict, use **Conflict Recovery**. On network/authentication failure, preserve pending content and report that the local context could be stale; do not claim a successful refresh.
4. Read the project entry and relevant notes after refresh/reconciliation. Record the base revision and whether remote freshness was verified.
5. Refresh once per coherent knowledge task, not per tool call or note. If returning after a long-running task, refresh/re-read before making edits, provided the worktree is still clean. If local edits already exist, reconcile deliberately rather than blindly pulling.

A local-only vault without an authorized Git configuration still supports reading and writing; report refresh/publication as skipped. Missing credentials do not justify reading secret stores or inventing authentication. If several local agents share one vault worktree, serialize the pull/read/edit/commit/push transaction or use isolated worktrees and explicit review; a status check alone is not a lock.

No cron job, timer, heartbeat, permanent background loop or automatic-sync service is part of this skill. Invoking Git during an actual knowledge task avoids idle model polling and preserves an explicit unit of work.

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
2. Require every note being published to live inside that configured worktree or a recovery worktree of the same repository. A recovery worktree inherits only the verified original upstream destination; it does not authorize another repository or branch.
3. Discover the current branch and destination from the worktree's configured upstream. Never create a remote or choose a repository implicitly.
4. Verify authentication without printing credentials.
5. Account for every worktree change. Preserve unrelated or unexplained changes and use an isolated worktree as described in **Conflict Recovery** when necessary; do not include them in this task's commit.
6. Stage only files written or updated by the current task.
7. Run `git diff --cached --check` and review the staged scope, then commit only the task's notes using the user's configured Git identity. Pull/rebase onto the configured upstream to incorporate changes published since the initial refresh. Re-read and validate affected notes/links after a successful rebase, then push that upstream. This final reconciliation complements, never replaces, the pre-edit pull.
8. On conflict or rejected push, use **Conflict Recovery**, validate the reconciled notes, then retry the normal push. Authentication failure requires durable pending content and a concrete recovery handoff; it does not justify losing the latest decisions. Never auto-stash, discard, force-push, or include unrelated files.
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

## Conflict Recovery

Resolve ordinary synchronization conflicts within the current task. A pause protects an in-progress merge from another overwrite; it must not become a blanket ban on new knowledge capture. Keep the existing publication authorization requirements.

1. **Preserve and inspect.** Identify the active writer, branch/upstream, unmerged files, and merge/rebase state. Before reconciliation, save recoverable copies of affected local files, both conflict versions and pending notes outside the synced vault, including intended paths, source/session and base revision. Never reset hard, select ours/theirs for the entire vault, or delete conflict copies to make status clean.
2. **Choose a safe workspace.** Serialize work with other writers. Finish an existing task-owned merge/rebase before starting another Git operation. If edits or an operation belong to another task, leave them intact and use an isolated worktree based on the verified configured upstream for this task's notes. Keep the original worktree and its local changes intact; report its state separately. Read relevant local edits as context, but do not publish them without authorization. If ownership cannot be established, preserve pending content rather than interfering with an active writer.
3. **Merge content deliberately.** Keep independent additions from both sides; deduplicate identical records and links; preserve valid frontmatter, full vault-relative links and dated history. Use explicit decisions and source evidence to identify supersession, not file modification time alone. When two claims cannot be reconciled, retain both with dates/sources and a marked unresolved question linked from the project entry. Ask only for the missing semantic choice while saving the unambiguous decisions. A clean textual merge still requires reviewing meaning.
4. **Finish and retry.** Validate the resolved files before marking them resolved and continuing the task-owned Git operation. Re-read targets before writing to detect concurrent edits. On a rejected push, fetch the configured upstream, reconcile, revalidate and retry a normal push. After two failed recovery cycles, inspect the concrete cause; persistent concurrent writes require coordination, not endless retries. Authentication, access and network failures must be reported precisely without changing credentials or publication scope implicitly.
5. **Capture even when blocked.** If reconciliation/publication cannot finish, save complete pending Markdown in a durable staging directory outside the conflicted vault, with a manifest of intended vault-relative paths, source/session, base revision, blocker and next retry action. Record that location in the session handoff or final report. Save unaffected notes in the vault when safe. Do not report staging as synchronization or end with only "writes paused".
6. **Drain and verify.** At the next sync, recover previously recorded staging locations and replay pending notes after searching for duplicates. Include recoverable missed decisions from prior sessions; never invent missing history. Update each affected project entry's links and dated progress and create the session handoff. Read back notes, check links/frontmatter and conflict markers, ensure no unmerged paths remain in the publication worktree, and verify the destination revision after pushing. Retain backups until recovery is verified. Report local capture, local reconciliation and remote publication separately; list any remaining ambiguity, staged content or untouched original-worktree conflict.

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
