# Junie ACP GA Release Channel

This repository has a single purpose: it is the **release channel** that the public
[Agent Client Protocol (ACP) registry](https://agentclientprotocol.com/) watches to discover new
**GA (General Availability)** versions of the [Junie CLI](https://github.com/JetBrains/junie).

It does not contain any source code, and it never hosts any binaries.

## Why this repository exists

ACP's registry keeps itself up to date via an **hourly cron job** that polls the repositories
referenced by each agent's manifest. For a `binary`-distributed agent like Junie, that cron only
looks at the **tags/releases** of the referenced repository — it never inspects file contents
there. It considers the highest non-draft, non-prerelease tag it finds, converts it to the ACP
semver form, and rewrites the download URLs for the six platform archives accordingly.

Junie CLI itself releases much more frequently (and with far more artifact types — nightly,
EAP, staging, etc.) than we want the public ACP registry to ever see. Pointing ACP directly at
`JetBrains/junie` would mean losing all control over *which* version becomes publicly visible,
and *when*.

This repository closes that gap: it is a small, dedicated, JetBrains-owned "signal" repo whose
only job is to be observed by ACP. A human QA engineer promotes a specific, already-tested GA
version by creating a release here — and nothing happens automatically before that.

## How a release gets here

1. A QA engineer manually starts a dedicated TeamCity job ("Promote Staging to ACP GA Release"),
   naming the exact Junie CLI technical version to promote (e.g. `2144.9`). There is no automatic
   trigger tied to the CLI release pipeline, and "latest" is never used.
2. The job verifies all six platform archives (macOS, Linux, and Windows, each for x64/arm64)
   against Junie's production `update-info.jsonl`, without downloading them.
3. If verification succeeds, the job creates a **non-draft, non-prerelease** GitHub Release here,
   with the CLI's technical version as the tag (e.g. `2144.9`).
4. From that point on, no further manual action is needed: ACP's own hourly cron will eventually
   notice the new release and propagate the version into the public ACP registry.

Re-running the promotion for a version that's already been released here is a no-op: the
existing release is detected and reported, never duplicated.

## What you will NOT find here

- **No binaries.** The actual Junie CLI archives always stay hosted on
  `JetBrains/junie` releases; this repository never re-hosts them.
- **No manifest files.** No `agent.json` or similar file is ever committed here — the release
  (tag + non-draft/non-prerelease flag) *is* the entire signal ACP needs.
- **No branches or Pull Requests.** Promotion happens as a direct release creation, not a
  file commit, branch, or PR.

## Scope

This repository is dedicated **solely to the GA channel**. Staging and EAP builds continue to
use existing, separate processes and are entirely out of scope here.

## Manual releases

Releases in this repository should only be created through the promotion job described above.
Please avoid creating or deleting releases/tags manually, since ACP will treat whatever is the
highest non-draft, non-prerelease tag here as the next version to publish.
