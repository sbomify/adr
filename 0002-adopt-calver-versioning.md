# 2. Adopt CalVer (YY.MINOR.PATCH) versioning format

Date: 2026-03-23

## Status

Proposed

## Context

We currently use [Semantic Versioning](https://semver.org/) (SemVer, MAJOR.MINOR.PATCH) across all sbomify projects. While SemVer has served us well, we want to revisit whether it remains the best fit as the organization and its release practices evolve.

Below we evaluate the major versioning schemes, presenting the strongest case for each before identifying trade-offs.

### SemVer (MAJOR.MINOR.PATCH)

**Best case:** SemVer is the most widely adopted versioning standard in open source. Its explicit compatibility contract — increment MAJOR for breaking changes, MINOR for features, PATCH for fixes — gives consumers a machine-readable signal about upgrade safety. Ecosystem tooling (npm, Cargo, pip version specifiers, Dependabot) is built around SemVer semantics, making automated dependency management straightforward.

**Challenges:** In practice, the MAJOR version bump is often political rather than technical — teams delay it for years or inflate it unnecessarily, leading to what Mahmoud Hashemi calls the ["fatuous 2.0 problem"](https://sedimental.org/designing_a_version.html). Version numbers carry no temporal information: `3.12.1` reveals nothing about when the software was released or how current it is. For supply chain security tooling, where freshness signals matter, this is a meaningful gap.

### CalVer — YY.MM.MICRO

**Best case:** Encodes both year and month of release at a glance. Proven at scale by [Black](https://black.readthedocs.io/) (the Python formatter), [Twisted](https://twisted.org/), and in a related form by [Ubuntu](https://ubuntu.com/) (YY.0M). Ideal for projects with monthly or near-monthly release cadences. Makes maintenance status immediately visible — users can instantly see whether a release is recent.

**Challenges:** When months are skipped, version sequences have gaps (26.1.0, 26.3.0, 26.7.0), which can look inconsistent. Multiple releases within one month are forced into micro-version increments that may obscure whether a release contains features or fixes. The zero-padding question (26.3 vs 26.03) introduces formatting inconsistencies. This scheme works best for projects that release on a predictable monthly cadence, which does not describe sbomify's release pattern.

### CalVer — YYYY.MINOR.PATCH

**Best case:** The four-digit year makes it unambiguous that this is calendar versioning — no one mistakes `2026` for a SemVer major version. Recognized in enterprise contexts through [JetBrains IDEs](https://www.jetbrains.com/) (e.g., PyCharm 2025.3.2).

**Challenges:** Versions are visually longer, and a four-digit major version can trigger warnings or look odd in tools that expect small integers. Docker tag sorting, Helm chart version constraints, and some CI tools handle four-digit major versions less gracefully than two-digit ones.

### CalVer — YY.MINOR.PATCH

**Best case:** Combines the temporal context of CalVer with the familiar three-segment shape of SemVer. The two-digit year signals when the software was released; sequential MINOR and PATCH segments retain intuitive incremental semantics. Accommodates any release cadence equally well — two releases per year or twelve. Migration from SemVer is seamless because the version shape is structurally identical: existing CI/CD pipelines, dependency specifiers (`>=`, `~=`, `^`), and changelog tooling work without modification. This is the scheme adopted by [pip](https://pip.pypa.io/) — the most widely used Python package manager — since 2024, and it is fully [PEP 440](https://peps.python.org/pep-0440/) compliant.

**Challenges:** Without explicit documentation, users might mistake versions for SemVer and expect MAJOR-version compatibility semantics. Requires clear communication in release notes and documentation.

### 0ver (0.x.y)

**Best case:** By never incrementing past 0, projects avoid prematurely signaling stability. This can be appropriate for experimental or rapidly changing software where any release might contain breaking changes.

**Challenges:** Communicates permanent instability, which is unsuitable for enterprise supply chain tooling where customers need confidence in the software's maturity. Offers no temporal or compatibility information.

### Build/hash-based versioning

**Best case:** Fully automated — every build gets a unique identifier derived from the commit hash or build number, with no human judgment required. Eliminates all versioning debates.

**Challenges:** Opaque to users, provides no semantic or temporal signal, and is poorly suited for dependency management. Not viable for published packages or tools consumed by external users.

### References

- [semver.org](https://semver.org/) — Semantic Versioning specification
- [calver.org](https://calver.org/) — Calendar Versioning convention and format notation
- [Mahmoud Hashemi, "Designing a Version"](https://sedimental.org/designing_a_version.html) — argues CalVer provides absolute rather than relative version semantics
- [pip CalVer adoption](https://pip.pypa.io/en/stable/news/) — precedent for YY.MINOR.PATCH at scale (adopted 2024)
- [PEP 440](https://peps.python.org/pep-0440/) — Python version identification and dependency specification

## Decision

We will adopt **CalVer with the `YY.MINOR.PATCH` format** across all sbomify projects.

The convention is:

- **YY** — two-digit calendar year (26, 27, 28…)
- **MINOR** — sequential feature release within the year, starting at 1 (26.1.0, 26.2.0, 26.3.0…)
- **PATCH** — bugfix and security releases (26.1.1, 26.1.2…)
- **Pre-releases** — standard suffixes: `26.1.0rc1`, `26.1.0a1`

This format best fits sbomify because:

1. **Freshness signal matters in supply chain security.** Version numbers appear in SBOMs, VEX documents, compliance reports, and procurement records. A year prefix immediately communicates how current the tooling is — and running outdated supply chain tools is itself a security risk.
2. **Our release cadence is irregular.** Releases are driven by specification updates (SPDX 3.x, CycloneDX), vulnerability disclosures, and customer needs — not a fixed monthly calendar. `YY.MINOR.PATCH` accommodates two releases or twelve releases in a year equally well.
3. **Seamless migration from SemVer.** The three-segment structure is identical to SemVer, so existing CI/CD pipelines, dependency specifiers, Docker tags, and changelog tooling require no changes.
4. **Proven at scale.** pip uses this exact scheme, validating it across millions of installations.
5. **PATCH segment for hotfixes.** We retain the ability to ship targeted security patches without a full minor release.

## Consequences

- Version numbers across all sbomify projects will immediately communicate the release year, making it easy for users and auditors to assess software currency.
- The "major version anxiety" problem disappears — there is no agonizing over when to bump from 2.x to 3.x.
- Users accustomed to SemVer compatibility semantics (where MAJOR changes signal breaking changes) will need clear documentation that our versioning does not carry that contract. Release notes must explicitly flag breaking changes.
- Downstream consumers who parse versions for compatibility signals will need to adjust expectations; we should document the versioning scheme in each repository's README and in our public documentation.
- The first release under this scheme will be versioned `26.1.0` (or the appropriate year at the time of adoption). All subsequent releases follow the convention above.
- Existing version references in package managers, Docker registries, and CI configurations will continue to work — the structural change is semantic, not syntactic.
