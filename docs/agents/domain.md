# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- **`CONTEXT.md`** at the repo root if it exists.
- **`kaup-design/docs/superpowers/adr/`** — architecture decisions live in the design repo.

If any of these files don't exist, **proceed silently**. Don't flag their absence.

## File structure

Single-context repo:

```
/
├── AGENTS.md
├── README.md
├── docs/agents/
└── CONTEXT.md                 (create lazily if needed)
```

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md` or `kaup-design` specs/glossary. Don't drift to synonyms the project explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts `kaup-design/docs/superpowers/adr/XXXX-<slug>.md` — but worth reopening because…_
