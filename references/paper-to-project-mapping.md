# Paper-to-Project Mapping

Use this workflow when the user wants a paper or literature survey to inform an ongoing project.

## 1. Read the project first

- Load the vault workflow.
- Locate `1_Projects/<ProjectName>.md`.
- Read the newest relevant record under `z_Archive/Project Notes/<ProjectName>/`.
- Build a concise model of the goal, current stage, constraints, and prior decisions.

Do not ask the user to repeat project context already present in the vault.

## 2. Survey the source

Read the full paper rather than only its abstract. Capture:

- problem setting and assumptions;
- method and task decomposition;
- datasets, baselines, metrics, and ablations;
- important negative results and stated limitations;
- reliable follow-up work when the user asks for current impact.

## 3. Build the structural map

Lead with a correspondence table:

| Paper concept | Project concept | Transferable element | Required adaptation |
|---|---|---|---|
| Entity or actor | Project entity | Representation or relation | Domain mismatch |
| Task | Project task | Baseline or evaluation | Missing constraints |
| Evidence | Project evidence | Dataset or metric | Coverage limits |

The mapping is more valuable than a section-by-section retelling of the paper.

## 4. Produce actionable recommendations

For each recommendation include:

- the project component it changes;
- the specific pattern borrowed from the source;
- evidence supporting the recommendation;
- the adaptation required;
- a priority such as P0, P1, or P2.

Do not claim that an observed association, pseudo-label, or benchmark result proves a causal intervention.

## 5. State the gaps

Explicitly identify:

- tasks the source does not solve;
- assumptions that fail in the project setting;
- missing data or evaluation coverage;
- ideas that require validation rather than direct adoption.

## 6. Save and connect the result

- Save the mapping to `z_Archive/Project Notes/<ProjectName>/YYYY-MM-DD <Title>.md`.
- Link it from `1_Projects/<ProjectName>.md` through `related:`.
- Add a dated progress entry to the project page.
- Write a session handoff when the work changes project direction or creates unresolved actions.

A pure paper summary with no project connection belongs in `3_Resources/` instead.

## Anti-patterns

- Restating the paper without a project correspondence map.
- Giving generic recommendations without citations or measured evidence.
- Hiding the main mapping below a long literature review.
- Saving a project deliverable as an orphaned general note.
- Forgetting to update the project entry and session handoff.
