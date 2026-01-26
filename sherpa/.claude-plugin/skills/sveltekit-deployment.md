---
name: SvelteKit Deployment
description: Steps required to build and deploy SvelteKit applications with support for SSR, SSG, SPA modes, and various adapters.
---

When building and deploying SvelteKit applications, follow these steps. SvelteKit is a full-stack framework that supports multiple deployment targets through adapters.

**Important:** Use the Node.js adapter (`@sveltejs/adapter-node`) for VM deployments. Do not use platform-specific adapters (Cloudflare, Vercel, Netlify) unless deploying to those specific platforms.

## Detection

Identify SvelteKit applications by:
- `@sveltejs/kit` in `package.json` dependencies
- `svelte.config.js` in root
- `src/routes/` directory structure
- `.svelte` files with `+page.svelte`, `+layout.svelte` naming

For plain Svelte (not SvelteKit), treat as a Vite SPA - use `react-deployment` skill patterns.

## Determine Rendering Strategy

Check `svelte.config.js` for the adapter and `+page.js`/`+page.server.js` files for page-level configuration:

### Adapter Configuration
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node';    // SSR (default for VMs)
import adapter from '@sveltejs/adapter-static';  // Full SSG
import adapter from '@sveltejs/adapter-auto';    // Auto-detect platform

export default {
  kit: {
    adapter: adapter()
  }
};
```

### Page-Level Prerendering
```javascript
// src/routes/+page.js or +layout.js
export const prerender = true;  // Prerender this page
export const ssr = false;       // Disable SSR (SPA mode)
```

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

```bash
# Build command (all modes)
npm run build
# or
npx vite build
```

## Deployment by Adapter

### Node.js Adapter (SSR) - Recommended for VMs

**When:** Using `@sveltejs/adapter-node`

**Output directory:** `build/`

1. Ensure `@sveltejs/adapter-node` is configured:
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter({
      out: 'build',
      precompress: true  // Optional: generate .gz and .br files
    })
  }
};
```

2. Build: `npm run build`

3. Copy the `build/` folder to the server via scp using `vm-access` skill

4. Run the SvelteKit server:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app/build

  # Install production dependencies if needed
  npm ci --production  # if package.json is in build folder

  # Run the server
  node index.js

  # Or with PM2
  pm2 start index.js --name "sveltekit-app"
EOF
```

5. Optionally upload static assets to CDN for offloading.

### Static Adapter (SSG)

**When:** Using `@sveltejs/adapter-static`

**Output directory:** `build/`

1. Configure the static adapter:
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-static';

export default {
  kit: {
    adapter: adapter({
      pages: 'build',
      assets: 'build',
      fallback: 'index.html',  // For SPA fallback
      precompress: true
    })
  }
};
```

2. Enable prerendering globally:
```javascript
// src/routes/+layout.js
export const prerender = true;
```

3. Build: `npm run build`

4. Upload `build/` folder to CDN using `cdn-uploading` skill

5. Configure SPA fallback if using client-side routing with `fallback: 'index.html'`

### SPA Mode (Client-Side Only)

**When:** Want a pure SPA without SSR/SSG

1. Configure static adapter with SPA fallback:
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-static';

export default {
  kit: {
    adapter: adapter({
      fallback: 'index.html'
    })
  }
};
```

2. Disable SSR globally:
```javascript
// src/routes/+layout.js
export const ssr = false;
```

3. Build and deploy to CDN

## Environment Variables

SvelteKit uses different variable prefixes for client/server:

### Public Variables (Client + Server)
- Prefix: `PUBLIC_`
- Access: `import { env } from '$env/dynamic/public'`
- Build-time: `import { PUBLIC_API_URL } from '$env/static/public'`

### Private Variables (Server Only)
- No prefix required
- Access: `import { env } from '$env/dynamic/private'`
- Build-time: `import { API_SECRET } from '$env/static/private'`

### Setting Variables

**Build Time:**
```bash
PUBLIC_API_URL=https://api.example.com npm run build
```

**Runtime (SSR only):**
```bash
PUBLIC_API_URL=https://api.example.com node build/index.js
```

### .env Files
SvelteKit supports `.env`, `.env.local`, `.env.production` files via Vite.

## Base Path Configuration

For deployment to a subdirectory:

```javascript
// svelte.config.js
export default {
  kit: {
    paths: {
      base: '/my-app',
      assets: 'https://cdn.example.com'  // Optional: CDN for assets
    }
  }
};
```

Update links in your app to use the base path:
```svelte
<script>
  import { base } from '$app/paths';
</script>

<a href="{base}/about">About</a>
```

## PM2 Configuration for SSR

For production SSR deployments:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  cat > ecosystem.config.js << 'PMEOF'
module.exports = {
  apps: [{
    name: 'sveltekit-app',
    script: 'build/index.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
      HOST: '0.0.0.0',
      PUBLIC_API_URL: 'https://api.example.com'
    }
  }]
}
PMEOF

  pm2 start ecosystem.config.js
  pm2 save
EOF
```

## Server Configuration

The Node.js adapter respects these environment variables:

- `PORT` - Server port (default: 3000)
- `HOST` - Server host (default: 0.0.0.0)
- `ORIGIN` - The origin URL for the app
- `PROTOCOL_HEADER` - Header for protocol (e.g., `X-Forwarded-Proto`)
- `HOST_HEADER` - Header for host (e.g., `X-Forwarded-Host`)

## Form Actions and API Routes

SvelteKit supports server-side form actions and API routes (`+server.js`):

- These require SSR mode (Node.js adapter)
- With static adapter, form actions and server routes are not available
- API routes become serverless functions with appropriate adapters

## Monorepo Support

For monorepos:

1. Identify the SvelteKit app directory
2. Build from the app directory:
   ```bash
   cd apps/web && npm run build
   # or
   turbo run build --filter=web
   ```
3. Output will be in `apps/web/build/`

## Common Issues

### Build Failing with Prerender Errors
- Ensure all prerendered pages have data available at build time
- Check for dynamic routes that need `entries` configuration:
```javascript
// src/routes/blog/[slug]/+page.js
export const entries = () => {
  return [
    { slug: 'post-1' },
    { slug: 'post-2' }
  ];
};
```

### 404 on Client-Side Navigation
- Configure SPA fallback with `fallback: 'index.html'` in adapter
- Ensure CDN serves `index.html` for unknown routes

### Environment Variables Undefined
- Check variable prefix (`PUBLIC_` for client access)
- Static imports require rebuild; dynamic imports work at runtime
- Verify `.env` file is in project root

### Assets Not Loading
- Check `paths.base` configuration
- Verify `paths.assets` if using CDN
- Ensure all asset imports are correct

### Form Actions Not Working
- Form actions require SSR (Node.js adapter)
- Cannot use form actions with static adapter
- Check if action is in `+page.server.js`

## Output Structure

### Node.js Adapter
```
build/
├── index.js           # Server entry point
├── handler.js         # Request handler
├── client/            # Client-side assets
│   ├── _app/
│   └── [static files]
├── server/            # Server-side code
└── prerendered/       # Prerendered pages (if any)
```

### Static Adapter
```
build/
├── index.html
├── _app/
│   ├── immutable/     # Hashed assets
│   └── version.json
└── [prerendered pages]
```

## When to Use This Skill

- Deploying SvelteKit applications (SSR, SSG, or SPA)
- Svelte apps built with SvelteKit
- For plain Svelte without SvelteKit (using Vite), treat as a standard Vite SPA
