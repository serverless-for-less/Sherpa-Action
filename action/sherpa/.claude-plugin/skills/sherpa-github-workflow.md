---
name: Sherpa.sh Github Workflow
description: How to create a workflow to retrigger the build and deployment workflow
---

To create a workflow for repeating the build and deploy. It should look similiar to this:

```
name: Claude Auto Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 1

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            /Sherpa/deploy {The original user context provided}

          claude_args: |
            --plugins sherpa.sh
```

But with the user provided feedback to modify the triggers, prompt, etc.