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

The configured directory must already be a Git repository with an upstream and working GitHub authentication. The variable contains no token and does not itself schedule synchronization.

Hosts without the variable can still use the skill for local notes but must not commit or push automatically.

## Profile-wide use and on-demand refresh

Install into the **active Hermes profile's** skill directory, not a project's worktree. The command above targets the default profile; for another active profile, resolve its actual home and use its `skills/note-taking/obsidian-second-brain/`. This makes the skill discoverable across sessions, worktrees and projects within that profile, not across every profile implicitly.

A compact pointer in the profile's supported persistent memory/instruction layer should identify this skill and the configured vault. Installation alone does not guarantee that every task loads the skill. Verify discovery in a fresh session; keep the public skill free of personal paths.

Synchronization is **on demand**, as part of actual knowledge work:

```text
Need project context or need to save knowledge
  → resolve vault / branch / upstream
  → inspect and preserve local changes / in-progress operations
  → reconcile task-owned conflicts or isolate from unrelated edits
  → refresh from the configured upstream
  → read latest notes
  → edit and verify task-related notes
  → commit with user's identity
  → pull --rebase to reconcile concurrent remote updates
  → revalidate affected notes, push, verify remote revision
```

Read-only tasks stop after reading. Local-only vaults skip Git and report that distinction. Refresh before editing and reconcile updates that arrive during the task. Dirty worktrees and rejected pushes trigger the `SKILL.md` Conflict Recovery workflow, not abandonment of new decisions: preserve both sides, merge notes and project links, validate, and retry. Keep unrelated edits intact, using an isolated worktree when needed; never auto-stash or force-push.

If network, authentication or an active writer still prevents completion, preserve complete pending notes with their destinations and recovery instructions in durable staging. Replay them at the next sync before claiming completion. Report local capture, reconciliation and verified publication separately; unresolved semantic claims retain their sources instead of silently choosing a winner.

There is no periodic synchronization, cron, heartbeat or background agent loop. Refresh once per coherent knowledge task, not once per turn or file. This avoids model polling when no work needs doing. A plain Git timer need not consume model tokens, but it still introduces concurrent-worktree and conflict handling concerns; it is not installed by this skill.

## Repository contents

- `SKILL.md` — the reusable workflow.
- `references/paper-to-project-mapping.md` — a generic paper-to-project research pattern.

The public version intentionally excludes personal vault contents, machine-specific paths, credentials, private project references, and local cron identifiers.

## License

MIT
