# Contributing

Thanks for helping improve Sherpa-Action!

## Quick ways to help (no API key required)
The GitHub Action requires an Anthropic API key (`anthropic_api_key` / `ANTHROPIC_API_KEY`) to actually run.
If you don't have one, you can still contribute by:

- Improving documentation and examples
- Reporting bugs (please redact secrets/tokens)
- Adding tests / CI checks / linting
- UX improvements: clearer errors, better logs, safer defaults
- Proposing a “dry-run / plan-only” mode so newcomers can validate setup without paid keys

## Reporting issues
- Include your OS + GitHub Actions runner (if applicable)
- Paste logs after removing secrets
- Explain expected vs actual behavior

## Pull requests
- Keep PRs small and focused (one topic per PR)
- Prefer docs + tests for behavior changes
- Explain the “why” in the PR description

## Security
Never commit secrets. Use GitHub repository secrets for credentials.
