# Troubleshooting

## Terminal shows "(END)" and looks stuck
This is a pager (usually `less`).
- Press `q` to quit.
- To disable paging for GitHub CLI output:
  - `gh config set pager cat`
  - or for one command: `GH_PAGER=cat gh <command>`

## GitHub API returns "204 No Content"
204 means the request succeeded but there is no response body.
Example: starring a repo returns 204 on success.

## “It won’t run” / missing API key
The action requires `anthropic_api_key` (Anthropic/Claude API key) to execute.
If you don’t have a key, you can still contribute via docs, tests, and UX improvements.

## GitHub Actions permissions
If you expect the action to commit `.sherpa.sh` memories, ensure your workflow has:
- `permissions: contents: write`

## Debug tips
- Check workflow logs in GitHub Actions UI.
- Or via CLI:
  - `gh run list`
  - `gh run view --log <run-id>`
