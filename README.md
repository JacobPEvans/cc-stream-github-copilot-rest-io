# cc-stream-github-copilot-rest-io

Collects GitHub Copilot usage metrics from the GitHub REST API at the org, team, and user level. Polls the Copilot usage metrics endpoints on a configurable schedule and breaks JSON array responses into individual events for downstream analysis.

## Pack Components

| Component | Type | Schedule | Enabled | Description |
|---|---|---|---|---|
| `GitHub_Copilot_Org_Usage` | REST Collector | Every 6 hours | No | Organization-level Copilot usage metrics |
| `GitHub_Copilot_Team_Usage` | REST Collector | Every 6 hours | No | Team-level Copilot usage metrics |
| `GitHub_Copilot_User_Usage` | REST Collector | Every 6 hours | No | User-level Copilot usage metrics |
| `GitHub Copilot Usage Ruleset` | Event Breaker | — | — | Breaks JSON array response into individual daily metrics |

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

The `jsonArrayField` is left empty for top-level JSON arrays. If the API response structure changes, adjust the `jsonArrayField` in the `GitHub Copilot Usage Ruleset`.

## Release Notes

### v1.0.0

- Initial release
- 3 REST collectors (org, team, user level)
- JSON array event breaker for Copilot usage metrics
- Configurable variables for org, PAT, team, and API base URL
