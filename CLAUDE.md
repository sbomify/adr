# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is the **organization-wide** ADR (Architecture Decision Record) repository for [sbomify](https://github.com/sbomify). It holds cross-cutting architectural decisions that span multiple repositories or affect the organization as a whole. Repo-specific ADRs live in their respective repos.

## ADR Format

All ADRs follow the Nygard template with sections: Title, Status (Proposed/Accepted/Deprecated/Superseded), Context, Decision, Consequences. See `0001-record-architecture-decisions.md` for the canonical example.

## Managing ADRs

ADRs are managed with [adr-tools](https://github.com/npryce/adr-tools/tree/master):

```bash
adr new "Title of the decision"       # Create a new ADR
adr list                               # List existing ADRs
adr new -s <number> "New title"        # Supersede a previous ADR
```

The `.adr-dir` file is set to `.` — ADR files live in the repository root (not a subdirectory).

## Naming Convention

ADR files are numbered sequentially: `NNNN-slug.md` (e.g., `0001-record-architecture-decisions.md`). The `adr-tools` CLI handles numbering automatically.
