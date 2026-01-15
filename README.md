# Sherpa.sh Deploy Action

**AI that ships your code.** A GitHub Action that transforms any cloud provider into a deployment platform. Just describe what you want in plain English.

```yaml
prompt: "Deploy my Nextjs app on Cloudflare"
```

Sherpa's AI automatically creates and configures your infrastructure: servers, DNS, SSL certificates, CDN, databases, backups, load balancing, and more.

## Why Sherpa?

- **Any Cloud** - AWS, Google Cloud, Digital Ocean, Hetzner, Linode, Vultr, Akamai, Cloudflare. Change anytime.
- **Any Framework** - Next.js, React, SvelteKit, Nuxt.js, Remix, Astro, Django, Laravel, Docker, and more.
- **Plain English** - No YAML configs, no Terraform, no DevOps expertise required
- **Open Source** - Community-driven and transparent

**Our Vision**
To make infrastructure invisible!  To create a world where developers just describe what they want and the AI handles everything infrastructure related.

## Usage

### Frontend / Edge (Cloudflare)

For static sites, SPAs, or edge-rendered apps, deploy to Cloudflare:

```yaml
name: Deploy to Cloudflare
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
          prompt: "Deploy my Next.js app to Cloudflare"
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### Backend / Server (AWS EC2)

For full-stack apps with server-side rendering, APIs, or databases, deploy to a VM:

```yaml
name: Deploy to AWS
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
          prompt: "Deploy my Next.js app to AWS EC2"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: us-east-1
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_PUBLIC_KEY: ${{ secrets.SSH_PUBLIC_KEY }}
```

#### SSH Key Setup

Backend deployments require SSH keys for VM access. Generate them once:

```bash
# Generate an ed25519 keypair (recommended)
ssh-keygen -t ed25519 -f sherpa-deploy -N "" -C "sherpa-deploy"

# Add to GitHub secrets:
# SSH_PRIVATE_KEY = contents of sherpa-deploy
# SSH_PUBLIC_KEY  = contents of sherpa-deploy.pub
```

Then add these as repository secrets in GitHub (Settings → Secrets → Actions).

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
    # AWS credentials
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_REGION: us-east-1

    # SSH keys for VM access (backend deployments)
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
    SSH_PUBLIC_KEY: ${{ secrets.SSH_PUBLIC_KEY }}

    # Cloudflare credentials
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

Any environment variables you pass will be available to Sherpa.sh during execution.

### Using a Secret File

For projects with many environment variables, you can store an entire `.env` file as a single GitHub secret:

1. Create a secret named `ENV_FILE` containing your environment variables:
   ```
   DATABASE_URL=postgres://...
   REDIS_URL=redis://...
   API_KEY=sk-...
   ```

2. Pass it to the action:
   ```yaml
   - uses: sherpa-sh/sherpa-action@v1
     with:
       anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
       prompt: "Deploy to AWS"
     env:
       ENV_FILE: ${{ secrets.ENV_FILE }}
   ```

### Environment Files (.env)

If your project uses `.env` files for different environments (`.env.production`, `.env.staging`, etc.), just tell Sherpa which one to use in plain English:

```yaml
prompt: "Deploy to AWS using .env.production"
```

```yaml
prompt: "Deploy with NODE_ENV=production"
```

```yaml
prompt: "Deploy to staging environment"
```

Sherpa will automatically configure the correct environment variables and `NODE_ENV` based on your prompt.

## Provider Credentials Setup

### Cloudflare

Create an API token at [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens) with the appropriate permissions for your deployment (e.g., Workers, Pages, DNS). Add it as `CLOUDFLARE_API_TOKEN` in your repository secrets.

### AWS

Create an IAM user with programmatic access and the necessary permissions for your deployment. Add these secrets to your repository:

```
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

## Features

Describe your infrastructure needs in plain English. Here are some examples:

### Specify Resources

```yaml
prompt: "Deploy my app to AWS Lambda"
```

```yaml
prompt: "Deploy to an EC2 t3.medium instance"
```

```yaml
prompt: "Deploy to EKS with 2 replicas"
```

### Custom Domains

```yaml
prompt: "Deploy to Cloudflare and use my domain myapp.com"
```

```yaml
prompt: "Deploy to AWS and configure api.mycompany.com"
```

### Load Balancers

```yaml
prompt: "Deploy with a load balancer, route /api/* to the backend service and /* to the frontend"
```

```yaml
prompt: "Set up an ALB with health checks on /health"
```

### CDN Configuration

```yaml
prompt: "Deploy with CloudFront CDN, cache static assets for 1 year"
```

```yaml
prompt: "Enable Cloudflare CDN with aggressive caching for images"
```

### Full Example

Combine multiple features in a single prompt:

```yaml
name: Deploy Full Stack
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

      - uses: sherpa-sh/sherpa-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Deploy my Next.js app with:
            - Backend on a Hetzner CX22 server
            - Cloudflare CDN in front with aggressive caching for static assets
            - Custom domain app.mycompany.com with SSL
            - Load balancer routing /api/* to the backend
            - Use .env.production for environment variables
            - NODE_ENV=production
        env:
          HETZNER_API_TOKEN: ${{ secrets.HETZNER_API_TOKEN }}
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_PUBLIC_KEY: ${{ secrets.SSH_PUBLIC_KEY }}
          ENV_FILE: ${{ secrets.ENV_FILE }}
```

## How It Works

1. Runs Claude Code with the Sherpa plugin
2. Executes the `/Sherpa/deploy` command with your prompt (See command descriptions below)
3. Saves memories into your repo in `.sherpa.sh`.  (The github action commits these into the repo)

### Memories

Memories are files that represent Sherpa's understanding of your project. They are saved into `.sherpa.sh/` and committed to your repo. Sherpa uses these to maintain context between deployments.

| File | Purpose |
|------|---------|
| `executionlog.md` | Records successful build/deploy commands. Makes subsequent deploys much faster by skipping discovery. |
| `infrastructure.md` | Tracks provisioned resources (servers, DNS, etc.) so Sherpa knows what exists. This is just a starting point and not a "source of truth". The Sherpa AI will look at existing infra to see if its changed ferom the markdown file and make the correct provisioning decisions. |
| `build-info.md` | Stores detected framework, build commands, and output directories. |

**When to modify memories:**

- **Delete `executionlog.md`** - To force Sherpa to re-discover the optimal deployment steps
- **Delete `build-info.md`** - If you changed frameworks or significantly altered your build process
- **Delete `infrastructure.md`** - If you manually deleted cloud resources and want Sherpa to replan and provision

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
| AWS | Supported |
| Google Cloud | Planned |
| Digital Ocean | Planned |
| Hetzner | Planned |
| BunnyCDN | Planned |
| Linode | Planned |
| Vultr | Planned |
| Akamai | Planned |

### Infrastructure Features

| Feature | Status |
|---------|--------|
| Static site hosting | Supported |
| Serverless functions | Supported |
| VM provisioning (EC2) | Supported |
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
