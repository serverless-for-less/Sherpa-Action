# Sherpa.sh Deploy Action

**AI that ships your code.** A GitHub Action that transforms any cloud provider into a deployment platform. Just describe what you want in plain English.

```yaml
prompt: "Deploy my Nextjs app on Cloudflare"
```

Sherpa's AI automatically creates and configures your infrastructure: servers, DNS, SSL certificates, CDN, databases, backups, load balancing, and more.

## Why Sherpa?

- **Any Cloud** - AWS, Google Cloud, Digital Ocean, Hetzner, Linode, Vultr, Akamai, Cloudflare
- **Any Framework** - Next.js, React, SvelteKit, Nuxt.js, Remix, Astro, Django, Laravel, Docker
- **Plain English** - No YAML configs, no Terraform, no DevOps expertise required
- **Open Source** - Community-driven and transparent

**Our Vision**
A world where developers just write code and describe what they want. The AI handles everything else, deploying to any cloud on earth without them ever thinking about infrastructure.

Make infrastructure invisible!

Read more about our super secret plan here: https://www.sherpa.sh/blog/vision

## Usage

### In a GitHub Action (Recommended)

Add this to your workflow file (e.g., `.github/workflows/deploy.yml`):

```yaml
name: Push to Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 1

      - uses: sherpa-sh/sherpa-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_api_key: ${{ secrets.}}

          prompt: "Deploy my nextjs app to cloudflare"
        env:
          CLOUDFLARE_API_KEY: ${{ secrets.CLOUDFLARE_API_KEY }}
```

### Local Development

To use a local copy of the action (for development or customization):

1. Clone the action repository into your project:
   ```bash
   git clone https://github.com/sherpa-sh/sherpa-action.git .github/actions/sherpa
   ```

2. Reference it locally in your workflow:
   ```yaml
   - uses: ./.github/actions/sherpa
     with:
       anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
   ```

### Using with Claude Code CLI (Local Testing)

To test the Sherpa plugin locally with Claude Code CLI:

```bash
# Clone the action repository
git clone https://github.com/sherpa-sh/sherpa-action.git

# Point to the sherpa plugin directory inside the action
claude --plugin-dir ./sherpa-action/sherpa
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `anthropic_api_key` | Anthropic API key for Claude | Yes | - |
| `prompt` | Context/instructions for the deployment | No | `''` |
| `github_token` | GitHub token for repo access | No | `${{ github.token }}` |
| `allowed_tools` | Comma-separated list of allowed Claude Code tools | No | `''` |
| `disallowed_tools` | Comma-separated list of disallowed Claude Code tools | No | `''` |
| `max_turns` | Maximum number of agentic turns | No | `''` |
| `timeout_minutes` | Timeout for Claude Code execution | No | `30` |

## Outputs

| Output | Description |
|--------|-------------|
| `result` | The result of the Sherpa deployment |
| `sherpa_changes` | Whether `.sherpa.sh` memories were committed to the repo (`true`/`false`) |

## Passing Secrets

To give Sherpa access to your cloud provider credentials and other secrets, you need to:

1. **Add secrets to your repository** - See [GitHub's documentation on using secrets](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets) for how to create and manage repository secrets.

2. **Pass secrets as environment variables** to the action:

```yaml
- uses: sherpa-sh/sherpa-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Deploy to AWS"
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

Any environment variables you pass will be available to Sherpa.sh during execution.

## How It Works

1. Runs Claude Code with the Sherpa plugin
2. Executes the `/Sherpa/deploy` command with your prompt (See command descriptions below)
3. Saves memories into your repo in `.sherpa.sh`.  (The github action commits these into the repo)

### Memories
Memories are files that represent Sherpa's understanding of the code intent, the build and deploy commands, infrastrucutre changes, and settings. They are saved into `.sherpa.sh/`.  Sherpa uses memories to maintain context between deployments.

## Required Github Action Permissions

Your workflow needs these permissions:

```yaml
permissions:
  contents: write      # To push .sherpa.sh memories
  pull-requests: write # To comment on PRs (optional)
  id-token: write      # For OIDC auth with cloud providers (optional)
```

## Roadmap

### Frameworks

| Framework | Status |
|-----------|--------|
| Next.js | Supported |
| React (static) | Planned |
| SvelteKit | Planned |
| Nuxt.js | Planned |
| Remix | Planned |
| Astro | Planned |
| Django | Planned |
| Laravel | Planned |
| Rails | Planned |
| Express / Node.js | Planned |
| Go | Planned |
| Docker (custom) | Planned |

### Cloud Providers

| Provider | Status |
|----------|--------|
| Cloudflare | Supported |
| AWS | Planned |
| Google Cloud | Planned |
| Digital Ocean | Planned |
| Hetzner | Planned |
| Linode | Planned |
| Vultr | Planned |
| Akamai | Planned |

### Infrastructure Features

| Feature | Status |
|---------|--------|
| Static site hosting | Supported |
| Serverless functions | Supported |
| SSL certificates | Supported |
| Custom domains | Planned |
| CDN configuration | Partial |
| Database provisioning | Planned |
| Automated backups | Planned |
| Load balancing | Planned |
| Kubernetes orchestration | Planned |
| Environment management | Planned |

Want to contribute or request a feature? [Open an issue](https://github.com/sherpa-sh/sherpa-action/issues) or [join the community](https://discord.com/invite/Pn7N2Wwbjy).

## Links

- [Sherpa Website](https://sherpa.sh)
- [Docs](https://github.com/sherpa-sh/sherpa-action)
- [GitHub](https://github.com/sherpa-sh/sherpa-action)
- [Discord](https://discord.com/invite/Pn7N2Wwbjy)
- [Youtube](https://www.youtube.com/@sherpa-sh)
