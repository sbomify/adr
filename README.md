# sbomify Architecture Decision Records

This is the **organization-wide** ADR repository for [sbomify](https://github.com/sbomify). It holds architectural decisions that span multiple repositories or affect the organization as a whole.

## What are ADRs?

Architecture Decision Records (ADRs) are short documents that capture important architectural decisions along with their context and consequences. The concept was introduced by Michael Nygard in his blog post [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).

Each ADR follows the Nygard template:

- **Title** — A short noun phrase describing the decision (e.g., "Use PostgreSQL for persistence")
- **Status** — Proposed, Accepted, Deprecated, or Superseded
- **Context** — The forces at play, including technical, political, social, and project-specific constraints
- **Decision** — The change that is being proposed or agreed upon
- **Consequences** — What becomes easier or harder as a result of this decision

## Repo-specific ADRs

Decisions scoped to a single repository live in that repository:

- [sbomify/sbomify → docs/ADR/](https://github.com/sbomify/sbomify/tree/master/docs/ADR)
- [sbomify/sbomify-action → docs/adr/](https://github.com/sbomify/sbomify-action/tree/master/docs/adr)

## Managing ADRs

We recommend [adr-tools](https://github.com/npryce/adr-tools/tree/master) for creating and managing ADRs.

```bash
# Install (macOS)
brew install adr-tools

# Create a new ADR
adr new "Use event-driven architecture for inter-service communication"

# List existing ADRs
adr list

# Supersede a previous ADR
adr new -s 2 "Use message queues instead of direct HTTP calls"
```
