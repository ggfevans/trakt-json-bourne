# Trakt JSON Bourne (GitHub Action)

<p align="center">
  <img src="static/trakt-json-bourne.png" alt="Trakt JSON Bourne" width="600">
</p>

`trakt-json-bourne` is a composite GitHub Action that slips into your workflow like Matt Damon in a bad European hallway, interrogates the [Trakt](https://trakt.tv) API, and exits with a clean JSON dossier. Optional TMDB posters are attached for the corkboard, because apparently this is an operation now.

## What it does

This action is built for scheduled automation via GitHub Actions cron triggers (`on.schedule`) and manual runs (`workflow_dispatch`). It validates inputs, runs shell scripts to fetch Trakt history, optionally enriches poster metadata from TMDB, and writes normalized JSON to your configured output path. The implementation is dependency-light and shell-first (bash + curl + jq on GitHub-hosted runners), with no npm/pip install step.

## Installation

Copy [`example.yml`](example.yml) to `.github/workflows/fetch-watching.yml`, replace `your-trakt-username` with your Trakt username, and store API values as repository secrets (`TRAKT_CLIENT_ID`, optional `TMDB_API_KEY`).

GitHub docs:
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [Creating secrets for a repository](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)

Or use this minimal snippet:

```yaml
name: Fetch Watching Data

on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

jobs:
  fetch:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

      - uses: ggfevans/trakt-json-bourne@499e6aabaab35c43ccccb852a0498324dca080f2 # v2.0.1
        with:
          trakt_client_id: ${{ secrets.TRAKT_CLIENT_ID }}
          trakt_username: your-trakt-username
          tmdb_api_key: ${{ secrets.TMDB_API_KEY }}

      - name: Commit and push
        uses: stefanzweifel/git-auto-commit-action@b863ae1933cb653a53c021fe36dbb774e1fb9403 # v5
        with:
          commit_message: 'chore: update watching data'
          file_pattern: src/data/watching.json
```

## Setup

### 1. Create a Trakt API application

1. Go to [Trakt API Applications](https://trakt.tv/oauth/applications)
2. Create a new application
3. Copy the **Client ID**

### 2. (Optional) Get a TMDB API key

1. Create an account at [TMDB](https://www.themoviedb.org/)
2. Go to Settings > API > Create > Developer
3. Copy your API key

### 3. Add repository secrets

Create repository secrets in **Settings > Secrets and variables > Actions**.

GitHub docs:
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [Creating secrets for a repository](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)

Required/optional secrets:
- `TRAKT_CLIENT_ID` - Your Trakt Client ID (required)
- `TMDB_API_KEY` - Your TMDB API key (optional)

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `trakt_client_id` | Yes | -- | Trakt API client ID |
| `trakt_username` | Yes | -- | Trakt username |
| `tmdb_api_key` | No | -- | TMDB API key for posters |
| `output_path` | No | `src/data/watching.json` | Output file path |
| `history_limit` | No | `30` | Number of items to fetch |

## Output JSON

The action writes a single JSON file with these fields:

- `lastUpdated` -- ISO 8601 timestamp of when the data was generated
- `recentlyWatched` -- array of most-recent watch entries with `title`, `type`, `posterUrl`, `url`, `watchedDate`
- `stats` -- object with monthly totals: `moviesThisMonth`, `showsThisMonth`

`recentlyWatched` items have these behaviors:

- `type` is either `"movie"` or `"show"`
- show titles are normalized as `Show Title — S01E02`
- `posterUrl` is `null` when TMDB enrichment is disabled or no poster is available

Poster enrichment status is exposed via the action output `posters_status` with one of:

- `"ok"` -- poster lookup completed successfully
- `"skipped"` -- TMDB key was not provided
- `"partial"` -- some poster lookups failed
- `"error"` -- poster enrichment failed completely

The action fails the workflow only if Trakt history cannot be fetched or validated. TMDB poster failures are non-fatal.

<details>
<summary>Example output (5 completely normal Matt Damon watches)</summary>

```json
{
  "lastUpdated": "2026-02-11T00:00:00Z",
  "recentlyWatched": [
    {
      "title": "The Bourne Ultimatum",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/3L6N9Uj7Q9mNQ2p0hP7QWQ6G5V4.jpg",
      "url": "https://trakt.tv/movies/the-bourne-ultimatum-2007",
      "watchedDate": "2026-02-10T23:58:00.000Z"
    },
    {
      "title": "The Bourne Supremacy",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/kqjL17yufvn9OVLyXYpvtyrFfak.jpg",
      "url": "https://trakt.tv/movies/the-bourne-supremacy-2004",
      "watchedDate": "2026-02-10T22:04:00.000Z"
    },
    {
      "title": "The Bourne Identity",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/aP8swke3gmowbkfZ6lmNidu0Frh.jpg",
      "url": "https://trakt.tv/movies/the-bourne-identity-2002",
      "watchedDate": "2026-02-10T20:11:00.000Z"
    },
    {
      "title": "The Martian",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/5aGhaIHYuQbqlHWvWYqMCnj40y2.jpg",
      "url": "https://trakt.tv/movies/the-martian-2015",
      "watchedDate": "2026-02-10T17:41:00.000Z"
    },
    {
      "title": "Good Will Hunting",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/z2FnLKpFi1HPO7BEJxdkv6hpJSU.jpg",
      "url": "https://trakt.tv/movies/good-will-hunting-1997",
      "watchedDate": "2026-02-10T15:03:00.000Z"
    }
  ],
  "stats": {
    "moviesThisMonth": 874,
    "showsThisMonth": 0
  }
}
```

</details>

## AI Disclosure

This project was built with the assistance of AI tools (Claude). The design, specification, and implementation were developed collaboratively with AI-generated code. All code has been reviewed and tested, but use at your own discretion.

## License

MIT - see [LICENSE](LICENSE)
