# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.0.x   | Yes       |
| < 1.0   | No        |

## Reporting a Vulnerability

Do not open public issues. Email the maintainer or use GitHub's private vulnerability reporting.

## Security Considerations

- Trakt Client ID is safe for logs (public API access)
- TMDB API key should be kept secret
- Output paths are validated to stay within workspace
- No external dependencies downloaded at runtime
