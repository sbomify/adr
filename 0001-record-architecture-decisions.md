# 1. Record architecture decisions

Date: 2026-03-20

## Status

Accepted

## Context

We need to record architectural decisions that affect the sbomify organization as a whole or span multiple repositories.

Individual repositories already maintain their own ADRs for repo-scoped decisions (e.g., [sbomify/sbomify](https://github.com/sbomify/sbomify/tree/master/docs/ADR) and [sbomify/sbomify-action](https://github.com/sbomify/sbomify-action/tree/master/docs/adr)), but cross-cutting decisions have no central home.

## Decision

We will use Architecture Decision Records, as [described by Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions), to document organization-wide architectural decisions in this dedicated repository.

ADRs will be managed with [adr-tools](https://github.com/npryce/adr-tools/tree/master).

## Consequences

- There is a central, version-controlled place for organization-wide architectural decisions.
- The process is lightweight — one short Markdown file per decision.
- Developers can understand past decisions and the context behind them.
- Repo-specific decisions continue to live in their respective repositories; only cross-cutting decisions go here.
