# Obsidian Second Brain Skill

A reusable Hermes skill for maintaining an Obsidian-compatible Markdown knowledge base across desktop and headless hosts.

## What it does

- reads project context before asking repeated setup questions;
- routes captures, references, project records, and session handoffs consistently;
- keeps project entries connected through Obsidian wikilinks;
- conditionally commits and pushes verified notes without hardcoding a repository;
- treats Obsidian as an optional UI over portable Markdown files.

## Install

```sh
git clone https://github.com/YanisYe/obsidian-second-brain-skill.git \
  ~/.hermes/skills/note-taking/obsidian-second-brain
```

Start a new Hermes session after installation so the skill is discovered.

## Optional Git publication

On each host that is allowed to publish, add the local vault path to an existing shell startup file:

```sh
export OBSIDIAN_KNOWLEDGE_REPO="/absolute/path/to/vault"
```

The configured directory must be a Git repository with working authentication. An existing upstream is preferred. If it is missing, provide the local vault path and repository URL together in the current task; the skill may bind that exact remote only after confirming the histories are compatible. It never overwrites a mismatched remote or force-pushes divergent history.

Hosts without the variable stay local-only unless the current task explicitly supplies both the vault path and repository URL.

## Profile-wide use and on-demand refresh

Install into the **active Hermes profile's** skill directory, not a project's worktree. The command above targets the default profile; for another active profile, resolve its actual home and use its `skills/note-taking/obsidian-second-brain/`. This makes the skill discoverable across sessions, worktrees and projects within that profile, not across every profile implicitly.

A compact pointer in the profile's supported persistent memory/instruction layer should identify this skill and the configured vault. Installation alone does not guarantee that every task loads the skill. Verify discovery in a fresh session; keep the public skill free of personal paths.

Synchronization is **on demand**, as part of actual knowledge work:

```text
Need project context or need to save knowledge
  → resolve vault / branch / upstream
  → check clean worktree; stop if dirty or conflicted
  → pull --ff-only
  → read latest notes
  → edit and verify task-related notes
  → commit with user's identity
  → pull --rebase to reconcile concurrent remote updates
  → revalidate affected notes, push, verify remote revision
```

Read-only tasks stop after reading. Local-only vaults skip Git and report that distinction. A pre-edit pull prevents writing against stale knowledge; the final pull handles updates that arrive while the task is running. Never auto-stash unrelated edits or force-push to resolve conflicts.

There is no periodic synchronization, cron, heartbeat or background agent loop. Refresh once per coherent knowledge task, not once per turn or file. This avoids model polling when no work needs doing. A plain Git timer need not consume model tokens, but it still introduces concurrent-worktree and conflict handling concerns; it is not installed by this skill.

## Repository contents

- `SKILL.md` — the reusable workflow.
- `references/paper-to-project-mapping.md` — a generic paper-to-project research pattern.

The public version intentionally excludes personal vault contents, machine-specific paths, credentials, private project references, and local cron identifiers.

## License

MIT
