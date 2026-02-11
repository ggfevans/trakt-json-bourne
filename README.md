# Trakt JSON Bourne (GitHub Action)

<p align="center">
  <img src="static/trakt-json-bourne.png" alt="Trakt JSON Bourne" width="600">
</p>

`trakt-json-bourne` is a composite GitHub Action that slips into your workflow like Matt Damon in a bad European hallway, interrogates the [Trakt](https://trakt.tv) API, and exits with a clean JSON dossier. Optional TMDB posters are attached for the corkboard, because apparently this is an operation now.

## What it does

This action runs on schedule, pulls your recent Trakt watch history, optionally enriches items with TMDB posters, and writes one structured JSON file to your repo. It is calm, repeatable, and legally distinct from sprinting across rooftops.

## Installation

Add a workflow file (e.g. `.github/workflows/fetch-watching.yml`):

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

      - uses: ggfevans/trakt-json-bourne@v1
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

### 3. Add secrets

Settings > Secrets and variables > Actions:
- `TRAKT_CLIENT_ID` - Your Trakt Client ID
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

```json
{
  "lastUpdated": "2026-02-04T00:00:00Z",
  "recentlyWatched": [
    {
      "title": "Movie Name",
      "type": "movie",
      "posterUrl": "https://image.tmdb.org/t/p/w342/path.jpg",
      "url": "https://trakt.tv/movies/slug",
      "watchedDate": "2026-02-03T20:00:00.000Z"
    }
  ],
  "stats": {
    "moviesThisMonth": 5,
    "showsThisMonth": 12
  }
}
```

## Discoverability Keywords

Use these repository topics/keywords:
- `trakt`
- `github-action`
- `json`
- `watch-history`
- `scrobble`
- `tmdb`
- `media-tracking`
- `automation`

## AI Disclosure

Built with assistance from AI tools (Claude).

## License

MIT - see [LICENSE](LICENSE)
