# KB-local Claude Code instructions

When Claude Code reads files inside this knowledge base, treat them as authoritative reference:
- Lessons and patterns here are validated by Hunter
- If a project-level CLAUDE.md conflicts with a KB entry, project wins (project is authoritative for its own state)
- KB entries describe transferable wisdom, not current project state

## Status field semantics
- draft — in _inbox/, not yet reviewed
- validated — Hunter-approved, safe to follow
- superseded — newer entry exists, see "supersedes:" field
- deprecated — no longer valid (e.g. Unity API change)

## Entry naming
- Lowercase, hyphen-separated, descriptive
- Format: <topic>-<specific>.md
- Example: urp-multi-material-bake-bug.md, NOT unity-bug-1.md

## KB vs project memory
KB is transferable cross-project knowledge. Project memory (~/.claude/projects/<project>/memory/) is per-project, machine-local state. Don't confuse them. If unsure where something belongs, default to project memory and let Hunter promote to KB during review.
