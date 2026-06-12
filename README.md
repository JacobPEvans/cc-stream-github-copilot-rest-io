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

Events leave this pack tagged with a `datatype` metadata field; Cribl Stream maps datatypes to
Splunk sourcetypes/indexes per the table below. Knowledge objects for the sourcetypes ship in
[VisiCore_TA_AI_Observability](https://github.com/JacobPEvans/VisiCore_TA_AI_Observability) (v0.2.0+).

The datatype is driven by the `datatype` pack variable (default `github:copilot:usage`) and is
shared by all three collectors.

| Input | Datatype | Splunk sourcetype | Index | TA support |
|---|---|---|---|---|
| `GitHub_Copilot_Org_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` | ✓ (0.2.0+) |
| `GitHub_Copilot_Team_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` | ✓ (0.2.0+) |
| `GitHub_Copilot_User_Usage` | `github:copilot:usage` | `github:copilot:usage` | `vscode` | ✓ (0.2.0+) |

## Setup

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

### Enable Collectors

All collectors are delivered **disabled** by default. Enable the collectors you need:

1. **Org-level**: Enable `GitHub_Copilot_Org_Usage` — requires `github_org` and `github_pat`
2. **Team-level**: Enable `GitHub_Copilot_Team_Usage` — also requires `github_team_slug`
3. **User-level**: Enable `GitHub_Copilot_User_Usage` — requires `github_pat` with user-level scopes

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
