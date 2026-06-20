# cc-stream-github-copilot-rest-io

## Overview

Collects GitHub Copilot usage metrics from the GitHub REST API at the org, team, and user level.
Polls the Copilot usage metrics endpoints on a configurable schedule and breaks JSON array
responses into individual events for downstream analysis.

This is a **Cribl Stream** pack (scheduled REST collectors), not a Cribl Edge pack.

## Pack Components

| Component | Type | Schedule | Enabled | Description |
|---|---|---|---|---|
| `GitHub_Copilot_Org_Usage` | REST Collector | Every 6 hours | No | Organization-level Copilot usage metrics |
| `GitHub_Copilot_Team_Usage` | REST Collector | Every 6 hours | No | Team-level Copilot usage metrics |
| `GitHub_Copilot_User_Usage` | REST Collector | Every 6 hours | No | User-level Copilot usage metrics |
| `GitHub Copilot Usage Ruleset` | Event Breaker | — | — | Breaks JSON array response into individual daily metrics |

## Data Sources

- **GitHub REST API — Copilot usage metrics** at the org, team, and user level, polled every
  6 hours by three scheduled REST collectors (all delivered disabled by default)

## Data Contract

Events leave this pack tagged with a `datatype` metadata field. The datatype is the routing
contract: a downstream Cribl Stream worker maps each datatype to a Splunk sourcetype and index,
and the consuming Splunk app supplies the matching field extractions and knowledge objects. As
long as a consumer honors the datatype below, this pack can change collectors without breaking it.

The datatype is driven by the `datatype` pack variable (default `github:copilot:usage`) and is
shared by all three collectors.

| Input | Datatype | Splunk sourcetype | Index |
|---|---|---|---|
| `GitHub_Copilot_Org_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` |
| `GitHub_Copilot_Team_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` |
| `GitHub_Copilot_User_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` |

## Installation

Build the pack archive and import it into Cribl Stream
(**Processing > Packs > Add Pack > Import from File**):

```bash
# from the repo root, produce cc-stream-github-copilot-rest-io.crbl
tar -czf cc-stream-github-copilot-rest-io.crbl data default package.json README.md
```

Each published GitHub release also ships a prebuilt `.crbl` you can import directly
instead of building locally.

### Prerequisites

- Cribl Stream 4.14.0 or later
- GitHub Personal Access Token with:
  - `manage_billing:copilot` scope (for org/team metrics)
  - `read:org` scope (for org/team metrics)
  - `copilot` scope (for user metrics)

### Configuration Variables

Configure these in **Knowledge > Variables**:

| Variable | Type | Default | Description |
|---|---|---|---|
| `datatype` | string | `github:copilot:usage` | Datatype for downstream routing |
| `github_org` | string | `your-org-name` | GitHub organization name |
| `github_pat` | encrypted | `changeme` | GitHub PAT |
| `github_team_slug` | string | *(empty)* | Team slug for team-level metrics |
| `api_base_url` | string | `https://api.github.com` | API base URL (supports GHES) |

## Usage

All collectors are delivered **disabled** by default. Enable the collectors you need:

1. **Org-level**: Enable `GitHub_Copilot_Org_Usage` — requires `github_org` and `github_pat`
2. **Team-level**: Enable `GitHub_Copilot_Team_Usage` — also requires `github_team_slug`
3. **User-level**: Enable `GitHub_Copilot_User_Usage` — requires `github_pat` with user-level scopes

Once enabled, each collector polls its endpoint every 6 hours, breaks the JSON array response
into individual daily-metric events via the `GitHub Copilot Usage Ruleset`, and tags every event
with the configured `datatype` for downstream routing (see [Data Contract](#data-contract)).

## Troubleshooting

### Rate Limiting

All collectors include automatic retry with exponential backoff (1s base, 3 retries, 2x multiplier) for HTTP 429 and 503 responses.

### Event Breaker

The `jsonArrayField` is left empty for top-level JSON arrays. If the API response structure
changes, adjust the `jsonArrayField` in the `GitHub Copilot Usage Ruleset`.

## Release Notes

### v1.0.1

- Docs: add Overview, Data Sources, and Data Contract sections; clarify this is a Cribl Stream pack
- Chore: normalize `package.json` metadata (trim author whitespace, move `vct` first in tags, pretty-print)

### v1.0.0

- Initial release
- 3 REST collectors (org, team, user level)
- JSON array event breaker for Copilot usage metrics
- Configurable variables for org, PAT, team, and API base URL

---

> Part of a [larger ecosystem of ~40 repos](https://docs.jacobpevans.com) — see how it all fits together.
