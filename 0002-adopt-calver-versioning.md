# 2. Adopt CalVer (YY.MM.MICRO) versioning format

Date: 2026-07-06

## Status

Proposed

## Context

We currently use [Semantic Versioning](https://semver.org/) (SemVer, MAJOR.MINOR.PATCH) across all sbomify projects. While SemVer has served us well, we want to revisit whether it remains the best fit as the organization and its release practices evolve.

Below we evaluate the major versioning schemes, presenting the strongest case for each before identifying trade-offs.

### SemVer (MAJOR.MINOR.PATCH)

**Best case:** SemVer is the most widely adopted versioning standard in open source. Its explicit compatibility contract — increment MAJOR for breaking changes, MINOR for features, PATCH for fixes — gives consumers a machine-readable signal about upgrade safety. Ecosystem tooling (npm, Cargo, pip version specifiers, Dependabot) is built around SemVer semantics, making automated dependency management straightforward.

**Challenges:** In practice, the MAJOR version bump is often political rather than technical — teams delay it for years or inflate it unnecessarily, leading to what Mahmoud Hashemi calls the ["fatuous 2.0 problem"](https://sedimental.org/designing_a_version.html). Version numbers carry no temporal information: `3.12.1` reveals nothing about when the software was released or how current it is. For supply chain security tooling, where freshness signals matter, this is a meaningful gap.

### CalVer — YY.MINOR.PATCH

**Best case:** Combines a temporal prefix with the familiar three-segment shape of SemVer. The two-digit year signals the calendar year of release; sequential MINOR and PATCH segments retain intuitive incremental semantics. Accommodates any release cadence equally well — two releases per year or twelve. Migration from SemVer is operationally smooth because the version shape is structurally identical, so existing CI/CD pipelines and most dependency tooling that expect three numeric segments continue to parse these versions correctly. [pip](https://pip.pypa.io/) adopted a year-based scheme (`YY.MINOR`, e.g. `18.0`, `23.1`, `24.0`) with [pip 18.0 in 2018](https://pip.pypa.io/en/stable/news/#v18-0), demonstrating a year-prefixed scheme at scale, and it is PEP 440 compliant.

**Challenges:** Only the **year** segment carries calendar meaning — MINOR is a sequential counter with no temporal content, so the scheme communicates *what year* a release shipped but nothing finer. Two releases eleven months apart (`26.1.0` in January, `26.2.0` in December) look adjacent, which understates how stale the earlier one is by the time the later ships. For a tool whose freshness is itself a security signal, month-level resolution is worth more than the incremental-counter semantics MINOR provides — especially since CalVer already discards the SemVer compatibility contract that made a dedicated MINOR segment meaningful in the first place. Consumers may also need to revisit version constraints: npm/Cargo/Helm-style `^26.1.0` ranges typically cap at `<27.0.0`, which in a year-prefixed scheme can prevent upgrades in the next calendar year, and `^` is not a valid operator under [PEP 440](https://peps.python.org/pep-0440/).

### CalVer — YY.MM.MICRO

**Best case:** Encodes both the **year and month** of release at a glance — `26.7.0` is unmistakably July 2026. Maintenance status is immediately legible: a user or auditor can tell not just the year but the month a release shipped, which is the highest-resolution freshness signal that still fits the three-segment `X.Y.Z` shape our tooling already expects. Proven at scale by [Black](https://black.readthedocs.io/) (the Python formatter), [Twisted](https://twisted.org/), and in a related zero-padded form by [Ubuntu](https://ubuntu.com/) (YY.0M). When months are skipped the sequence simply has gaps (`26.5.0` → `26.7.0`) — and that gap is *informative*: it tells the reader no release shipped in June. Structurally identical to SemVer, so CI/CD pipelines, Docker tags, and changelog tooling that only assume a numeric `X.Y.Z` pattern require no changes.

**Challenges:** The middle segment no longer distinguishes features from fixes — multiple releases within one month increment MICRO regardless of content, so consumers cannot infer change type from the version alone. This is an acceptable loss: CalVer already abandons the SemVer compatibility contract, so release notes (not the version string) are the source of truth for what changed. Zero-padding must be avoided: `26.07.0` is **invalid SemVer** (npm rejects leading zeros) and PEP 440 normalizes `26.07` → `26.7`, so the month is written **unpadded** (`26.7.0`, not `26.07.0`) to stay valid across our npm and Python packaging.

### CalVer — YYYY.MINOR.PATCH

**Best case:** The four-digit year makes it unambiguous that this is calendar versioning — no one mistakes `2026` for a SemVer major version. Recognized in enterprise contexts through [JetBrains IDEs](https://www.jetbrains.com/) (e.g., PyCharm 2025.3.2).

**Challenges:** Versions are visually longer, and a four-digit major version can trigger warnings or look odd in tools that expect small integers. Docker tag sorting, Helm chart version constraints, and some CI tools handle four-digit major versions less gracefully than two-digit ones.

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
- [Black — The Black Code Style](https://black.readthedocs.io/) — precedent for YY.MM.MICRO at scale
- [pip CalVer adoption](https://pip.pypa.io/en/stable/news/#v18-0) — year-prefixed CalVer since pip 18.0 (2018)
- [PEP 440](https://peps.python.org/pep-0440/) — Python version identification and dependency specification

## Decision

We will adopt **CalVer with the `YY.MM.MICRO` format** across all sbomify projects.

The convention is:

- **YY** — two-digit calendar year (26, 27, 28…)
- **MM** — month of release, `1`–`12`, **not zero-padded** (`26.7.0`, not `26.07.0`). Unpadded months keep every version a valid SemVer identifier (npm rejects leading zeros) and a stable PEP 440 identifier (which would otherwise normalize `26.07` → `26.7`).
- **MICRO** — release counter *within that month*, starting at `0`. The first release in a month is `.0`; subsequent releases the same month (features or fixes alike) increment it: `26.7.0`, `26.7.1`, `26.7.2`.
- **Pre-releases** — ecosystem-specific suffixes, e.g. SemVer-style `26.7.0-rc.1`, `26.7.0-alpha.1` and PEP 440–style `26.7.0rc1`, `26.7.0a1` for Python packages.

This format best fits sbomify because:

1. **Freshness signal matters in supply chain security, at the finest useful resolution.** Version numbers appear in SBOMs, VEX documents, compliance reports, and procurement records. A `YY.MM` prefix communicates not just the year but the *month* a release shipped — running outdated supply chain tooling is itself a security risk, and month-level currency is materially more useful than year-level when assessing how current a deployment is.
2. **Gaps are informative, not a defect.** Our release cadence is irregular — driven by specification updates (SPDX 3.x, CycloneDX), vulnerability disclosures, and customer needs rather than a fixed calendar. Under `YY.MM.MICRO`, a skipped month simply produces a gap in the sequence (`26.5.0` → `26.7.0`), which correctly reflects that no release shipped that month rather than hiding it behind an opaque counter.
3. **Seamless migration for internal tooling.** The three-segment structure is identical to SemVer, so existing CI/CD pipelines, Docker tags, and changelog tooling that only assume a numeric `X.Y.Z` pattern require no syntax changes.
4. **Proven at scale.** Black and Twisted use `YY.MM`-based CalVer across millions of installations.
5. **MICRO segment for same-month hotfixes.** We retain the ability to ship targeted security patches within a month without waiting for the next month's release.

### Transition from the earlier sequential scheme

The first three 2026 releases (`26.1.0`, `26.2.0`, `26.3.0`) were cut under an earlier interpretation in which the middle segment was a **sequential feature counter**, not the calendar month. Those tags are immutable and will **not** be re-versioned. From this ADR onward, the middle segment is the **month of release**: the next release, cut in July 2026, is `26.7.0`. The one-time discontinuity between the March-era `26.3.0` and the July `26.7.0` is expected and documented here.

## Consequences

- Version numbers across all sbomify projects will communicate the release **year and month**, letting users and auditors assess software currency to the month rather than only the year.
- The "major version anxiety" problem disappears — there is no agonizing over when to bump from 2.x to 3.x.
- Users accustomed to SemVer compatibility semantics (where MAJOR changes signal breaking changes) will need clear documentation that our versioning does not carry that contract. Release notes must explicitly flag breaking changes.
- The middle segment no longer separates features from fixes; the changelog and release notes become the authoritative record of change type. This is consistent with CalVer's premise that the version encodes *when*, not *what*.
- The month segment must always be written unpadded. Tooling or scripts that construct version strings must not zero-pad the month, or they will emit invalid SemVer / non-normalized PEP 440 identifiers.
- Downstream consumers who parse versions for compatibility signals will need to adjust expectations; the versioning scheme is documented in each repository's README and in our public documentation.
- Existing version references in package managers, Docker registries, and CI configurations continue to work — the structural change is semantic, not syntactic.
