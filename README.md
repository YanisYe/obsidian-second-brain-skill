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

## Repository contents

- `SKILL.md` — the reusable workflow.
- `references/paper-to-project-mapping.md` — a generic paper-to-project research pattern.

The public version intentionally excludes personal vault contents, machine-specific paths, credentials, private project references, and local cron identifiers.

## License

MIT
