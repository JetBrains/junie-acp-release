# Junie ACP Release

Public release host for **[Junie](https://github.com/JetBrains/junie)** when it is distributed through the
[Agent Client Protocol (ACP)](https://agentclientprotocol.com/).

This repository is **not** the Junie source tree. It holds:

1. **GitHub Releases** with platform binaries (zip archives)
2. **Registry pointer files** on `main` used by JetBrains-internal and pre-release channels

The product source and the general CLI distribution live elsewhere
([`JetBrains/junie`](https://github.com/JetBrains/junie) and related JetBrains channels).

---

## Who consumes what

| Consumer | What it looks at | What it ignores |
|---|---|---|
| **Public ACP registry** (hourly cron for `binary` agents) | Highest **non-draft, non-prerelease** release whose **tag** is a plain version `x.y` or `x.y.z` | Prerelease releases; tags that are not plain numbers (e.g. `2698.4-EAP`, `2411.2-staging`); files on `main` |
| **JetBrains IDE / internal tooling** (external registry URLs) | JSON files on `main` (see below) and the release assets those files point at | — |
| **Humans** | Release list + this README | — |

So: **only a plain GA tag** (for example `2698.3`) with `prerelease=false` is what the public ACP cron is designed to pick up.  
EAP and staging tags are published here for controlled testing and **must not** be treated as public GA.

---

## Repository layout

### `main` branch (pointer files)

Machine-updated JSON registries (ACP registry document shape). Typical files:

| File | Role |
|---|---|
| `registry-eap.json` | Current **EAP** pointer (pre-release IDE / internal use) |
| `registry-ga-staging.json` | Current **GA staging** smoke pointer (pre-promote candidate) |
| `registry-hotfix.json` | **Interrupt / hotfix** smoke pointer (does not replace the scheduled GA staging pointer) |

These files are **not** what the public ACP hourly cron uses to discover GA versions.  
They exist so clients can be pointed at a stable raw URL while binaries live on GitHub Releases.

Raw URL shape:

```text
https://raw.githubusercontent.com/JetBrains/junie-acp-release/main/<registry-file>.json
```

### Releases (tags + assets)

Each relevant channel publishes a GitHub Release with downloadable archives for the supported platforms
(macOS / Linux / Windows × amd64 / aarch64), plus helper scripts when applicable (e.g. `junie.sh`).

| Channel | Tag example | Prerelease flag | Typical zip prefix | Visible to public ACP cron? |
|---|---|---|---|---|
| **EAP** | `2698.4-EAP` | `true` | `junie-eap-<ver>-…` | **No** |
| **GA Staging / Hotfix candidate** | `2411.2-staging` | `true` | `junie-release-<ver>-…` | **No** |
| **GA (public)** | `2411.2` | `false` | `junie-release-<ver>-…` | **Yes** |

Download URL shape:

```text
https://github.com/JetBrains/junie-acp-release/releases/download/<tag>/<archive-name>.zip
```

Examples:

```text
…/releases/download/2698.4-EAP/junie-eap-2698.4-macos-aarch64.zip
…/releases/download/2411.2-staging/junie-release-2411.2-linux-amd64.zip
…/releases/download/2411.2/junie-release-2411.2-windows-amd64.zip
```

---

## How public GA discovery works

For a `binary` agent, the public ACP registry automation:

1. Reads the agent’s declared **repository** (this repo for Junie ACP)
2. Lists **GitHub Releases / tags**
3. Skips drafts and **prereleases**
4. Keeps only tags that look like plain numeric versions (`x.y` / `x.y.z`)
5. Selects the highest matching version and rewrites platform archive URLs accordingly

Implications:

- Publishing an EAP tag such as `2698.4-EAP` does **not** move the public GA channel.
- Marking a release as prerelease is an extra safety net; non-plain tags are already out of scope for that cron.
- Creating or deleting GA tags manually can change what the public registry eventually publishes — treat GA releases as intentional.

The public registry entry for Junie lives in the
[ACP registry](https://github.com/agentclientprotocol/registry) (see the Junie agent manifest there).
That manifest’s `repository` field and archive URLs should refer to this host for ACP distribution.

---

## Channels (product view)

```text
EAP
  → prerelease tag <ver>-EAP + assets
  → registry-eap.json on main

GA Staging (scheduled pre-release smoke)
  → prerelease tag <ver>-staging + assets
  → registry-ga-staging.json on main

Hotfix / interrupt smoke (optional parallel pointer)
  → prerelease tag <ver>-staging + assets
  → registry-hotfix.json on main
  → does not overwrite the scheduled GA staging pointer

GA (public)
  → plain tag <ver>, prerelease=false + assets
  → discovered by the public ACP cron
```

GA is intentionally gated: a staging or hotfix candidate is smoked first; only then is a **plain** GA release published.

---

## What this repository is not

- Not the Junie **source code** repository
- Not the general-purpose CLI download site for every JetBrains channel (Homebrew, npm, IDE-bundled flows, etc. may use other hosts)
- Not a place for ad-hoc experimental tags that look like public GA (`1.2.3` without prerelease) unless that version is meant for the public ACP channel
- Not documentation for JetBrains internal CI wiring — see internal docs (below)

---

## Integrity and support

- Prefer downloading **release assets** from the official Releases page of this repository.
- File names and version strings in registry JSON must match the assets attached to the corresponding tag.
- For product issues with Junie itself, use normal JetBrains / Junie support and issue channels.
- For ACP registry packaging questions, see [agentclientprotocol/registry](https://github.com/agentclientprotocol/registry).

---

## Releasing (maintainers)

Releases and registry pointer updates are produced by **JetBrains internal automation**.  
Operational runbooks, CI job names, credentials, and promote procedures are **not** documented in this public README.

Internal documentation (JetBrains only), including the independent ACP release workflow, lives in Junie-DEV internal docs
Junie Dev Team is responsible for this repo, the internal automation and release process for ACP

---

## License

Junie binaries distributed from this repository remain under JetBrains’ product licensing terms.
This repository’s metadata and pointer files do not re-license the product.
