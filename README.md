# Sherpa.sh - AI that ships your code

Transform any cloud provider into a deployment platform. Just describe what you want in plain English.

```yaml
prompt: "Deploy my Nextjs app on Cloudflare"
```

Sherpa's AI automatically creates and configures your infrastructure: servers, DNS, SSL certificates, CDN, databases, backups, load balancing, and more.

## Why Sherpa?

Infrastructure should be invisible. Developers want to ship products, not configure YAML files. Sherpa is the open-source intelligence layer that makes deployment disappear—in the best possible way.

**Any Cloud** — AWS, Google Cloud, Hetzner, Akamai, Cloudflare, your own servers. Switch anytime. No lock-in, ever.
**Any Framework** — Next.js, SvelteKit, Nuxt, Remix, Astro, Django, Laravel, Docker, and more.
**Plain English** — No Terraform. No DevOps expertise. Describe what you want, Sherpa figures out how.
**Open Source** — Fully transparent, auditable, and community-driven. See exactly what runs in your infrastructure.

We believe the future of deployment is AI that reads your code, understands your intent, and configures optimal infrastructure automatically. No dashboards to learn. No vendor lock-in to escape. Just code that ships.

### Where we are going

We're building toward infrastructure that manages itself entirely — AI that reads your code, picks optimal providers automatically, reroutes traffic when outages hit, and continuously optimizes costs across every cloud.

The end state? You push code and it's live, globally distributed, auto-scaling, always improving. Infrastructure becomes invisible.

And because it's open source, you can see exactly how it works, contribute to the roadmap, and never get locked in to a vendor.

## Usage

### With Github Actions

Run Sherpa on repo pushes, deploys, PRs, and more.  Integrate into your existing Github Workflows.  Get the Vercel/Netlify/Heroku experience on your own terms.

Create a file at `.github/workflows/deploy.yml` in your repository:

```yaml
name: Push to deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write # Required for memories.
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 1

      - uses: sherpa-sh/sherpa-action@v1.0.0-alpha.1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Deploy my nexjs app on AWS lambda and Cloudfront"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: us-east-1
```

### With Claude Code

### Using with Claude Code CLI (Local Testing)

```bash
# Clone the action repository
git clone https://github.com/sherpa-sh/sherpa-action.git

# Point to the sherpa plugin directory inside the action
claude --plugin-dir ./sherpa-action/sherpa
```

Prompt:
```
Deploy my static nextjs app to cloudflare
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

      - uses: sherpa-sh/sherpa-action@v1.0.0-alpha.1
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
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
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

## Settings


### Github Action
#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `anthropic_api_key` | Anthropic API key for Claude | Yes | - |
| `prompt` | Context/instructions for the deployment | No | `''` |
| `github_token` | GitHub token for repo access | No | `${{ github.token }}` |
| `allowed_tools` | Comma-separated list of allowed Claude Code tools | No | `''` |
| `disallowed_tools` | Comma-separated list of disallowed Claude Code tools | No | `''` |
| `max_turns` | Maximum number of agentic turns | No | `''` |
| `timeout_minutes` | Timeout for Claude Code execution | No | `30` |

#### Outputs

| Output | Description |
|--------|-------------|
| `result` | The result of the Sherpa deployment |
| `sherpa_changes` | Whether `.sherpa.sh` memories were committed to the repo (`true`/`false`) |

#### Required Permissions

Your workflow needs these permissions:

```yaml
permissions:
  contents: write      # To push .sherpa.sh memories
  pull-requests: write # To comment on PRs (optional)
  id-token: write      # For OIDC auth with cloud providers (optional)
```
### Passing Secrets

#### Github Action

To give Sherpa access to your cloud provider credentials and other secrets, you need to:

1. **Add secrets to your repository** - See [GitHub's documentation on using secrets](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets) for how to create and manage repository secrets.

2. **Pass secrets as environment variables** to the action:

```yaml
- uses: sherpa-sh/sherpa-action@v1.0.0-alpha.1
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
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

Any environment variables you pass will only be available to Sherpa.sh during workflow execution. Otherwise they are encrypted at rest inside of Github's secret management system.

#### Using a Secret File

For projects with many environment variables, you can store an entire `.env` file as a single GitHub secret:

1. Create a secret named `ENV_FILE` containing your environment variables:
   ```
   DATABASE_URL=postgres://...
   REDIS_URL=redis://...
   API_KEY=sk-...
   ```

2. Pass it to the action:
   ```yaml
   - uses: sherpa-sh/sherpa-action@v1.0.0-alpha.1
     with:
       anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
       prompt: "Deploy to AWS"
     env:
       ENV_FILE: ${{ secrets.ENV_FILE }}
   ```

#### Environment Files (.env)

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

### Provider Credentials Setup

#### Cloudflare

Create an API token at [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens) with the appropriate permissions for your deployment (e.g., Workers, Pages, DNS). Add these secrets:

```
CLOUDFLARE_API_TOKEN=...
CLOUDFLARE_ACCOUNT_ID=...
```

You can find your Account ID in the Cloudflare Dashboard under any domain's Overview page (right sidebar) or at the top of the Workers & Pages section.

#### AWS

Create an IAM user with programmatic access and the necessary permissions for your deployment. Add these secrets:

```
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

#### Digital Ocean

Generate a personal access token at [Digital Ocean API Settings](https://cloud.digitalocean.com/account/api/tokens). Add it as `DIGITALOCEAN_API_TOKEN` in your repository secrets.

#### Other providers

See roadmap below.

### SSH Key Setup

Backend deployments require SSH keys for VM access. Generate them once:

```bash
# Generate an ed25519 keypair (recommended)
ssh-keygen -t ed25519 -f sherpa-deploy -N "" -C "sherpa-deploy"

# Add to GitHub secrets:
# SSH_PRIVATE_KEY = contents of sherpa-deploy
# SSH_PUBLIC_KEY  = contents of sherpa-deploy.pub
```

Then add these as repository secrets in GitHub (Settings → Secrets → Actions).

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

## Contributing

Want to contribute or request a feature? [Open an issue](https://github.com/sherpa-sh/sherpa-action/issues) or [join the community](https://discord.com/invite/Pn7N2Wwbjy).

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

## Links

- [Sherpa Website](https://sherpa.sh)
- [Docs](https://github.com/sherpa-sh/sherpa-action)
- [GitHub](https://github.com/sherpa-sh/sherpa-action)
- [Discord](https://discord.com/invite/Pn7N2Wwbjy)
- [Youtube](https://www.youtube.com/@sherpa-sh)