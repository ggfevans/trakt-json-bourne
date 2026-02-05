# Trakt GitHub Action

A composite GitHub Action that fetches watch history from [Trakt](https://trakt.tv) and writes it to a structured JSON file with optional poster enrichment from TMDB.

## What it does

1. Fetches your recent watch history from the Trakt API
2. Optionally enriches items with poster images from TMDB
3. Writes a JSON file with watch data and monthly stats

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

      - uses: ggfevans/trakt-github-action@v1
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

## AI Disclosure

Built with assistance from AI tools (Claude).

## License

MIT - see [LICENSE](LICENSE)
