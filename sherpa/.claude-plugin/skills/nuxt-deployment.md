---
name: Nuxt Deployment
description: Steps required to build and deploy Nuxt applications (Nuxt 3) with support for SSR, SSG, and hybrid rendering modes.
---

When building and deploying Nuxt applications, follow these steps. Nuxt supports multiple rendering modes including server-side rendering (SSR), static site generation (SSG), and hybrid rendering.

**Important:** Do not use third-party deployment adapters or wrappers. Deploy the actual Nuxt output directly.

## Detection

Identify Nuxt applications by:
- `nuxt` in `package.json` dependencies (Nuxt 3)
- `nuxt.config.ts` or `nuxt.config.js` in root
- `.nuxt/` directory after build

## Determine Rendering Mode

Check `nuxt.config.ts` for the rendering configuration:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Static generation (SSG)
  ssr: true,        // default
  nitro: {
    preset: 'static'  // or prerender all routes
  },

  // OR: Client-side only (SPA mode)
  ssr: false,

  // OR: Server-side rendering (SSR) - default
  ssr: true,  // with no static preset

  // OR: Hybrid rendering (per-route)
  routeRules: {
    '/': { prerender: true },
    '/api/**': { cors: true },
    '/admin/**': { ssr: false }
  }
})
```

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

```bash
# Build command (all modes)
npm run build
# or
npx nuxt build

# For static generation specifically
npm run generate
# or
npx nuxt generate
```

## Deployment by Mode

### Static Generation (SSG) / Prerendered

**When:** `nitro.preset: 'static'` or using `nuxt generate`

**Output directory:** `.output/public/`

1. Run `npm run generate` or `npm run build` with static preset
2. Upload the `.output/public/` folder to CDN using `cdn-uploading` skill

```bash
# Build static site
npx nuxt generate
# Output is in .output/public/
```

### Client-Side Only (SPA Mode)

**When:** `ssr: false` in config

**Output directory:** `.output/public/`

1. Build the application: `npm run build`
2. Upload `.output/public/` to CDN using `cdn-uploading` skill
3. Configure SPA fallback for client-side routing (serve `index.html` for all routes)

### Server-Side Rendering (SSR)

**When:** `ssr: true` (default) without static preset

**Output directory:** `.output/`

1. Build the application: `npm run build`
2. Copy the entire `.output/` folder to the server via scp using `vm-access` skill
3. Run the Nuxt server:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app/.output

  # Run with Node.js directly
  node server/index.mjs

  # Or with PM2 for process management
  pm2 start server/index.mjs --name "nuxt-app"
EOF
```

4. Upload static assets from `.output/public/_nuxt/` to CDN using `cdn-uploading` skill (optional, for asset offloading)

### Hybrid Rendering

**When:** Using `routeRules` with mixed prerender/SSR

Deploy as SSR (server required) but prerendered routes will be served as static files automatically.

## Environment Variables

Nuxt uses runtime config for environment variables:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Private keys (server-only)
    apiSecret: process.env.API_SECRET,

    // Public keys (exposed to client)
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE
    }
  }
})
```

### At Build Time
```bash
NUXT_PUBLIC_API_BASE=https://api.example.com npm run build
```

### At Runtime (SSR only)
```bash
NUXT_PUBLIC_API_BASE=https://api.example.com node .output/server/index.mjs
```

Runtime environment variables must be prefixed with `NUXT_` to override config.

## Base Path Configuration

For deployment to a subdirectory:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    baseURL: '/my-app/',
    cdnURL: 'https://cdn.example.com/' // optional: serve assets from CDN
  }
})
```

## Monorepo Support

For monorepos:

1. Identify the Nuxt app directory (e.g., `apps/web`)
2. Build from the app directory or use workspace commands:
   ```bash
   turbo run build --filter=web
   # or
   cd apps/web && npm run build
   ```
3. Output will be in `apps/web/.output/`

## Nitro Server Configuration

Nuxt 3 uses Nitro as its server engine. Check for custom server configuration:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'node-server', // default for SSR
    // preset: 'static',   // for SSG

    // Custom server options
    serveStatic: true,

    // Compression
    compressPublicAssets: true
  }
})
```

## PM2 Configuration for SSR

For production SSR deployments:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  # Create PM2 ecosystem file
  cat > ecosystem.config.js << 'PMEOF'
module.exports = {
  apps: [{
    name: 'nuxt-app',
    script: '.output/server/index.mjs',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      NUXT_PUBLIC_API_BASE: 'https://api.example.com'
    }
  }]
}
PMEOF

  pm2 start ecosystem.config.js
  pm2 save
EOF
```

## Common Issues

### Hydration Mismatch
- Ensure server and client render the same content
- Check for browser-only code not wrapped in `<ClientOnly>` or `process.client`

### API Routes Not Working (Static)
- API routes (`/api/**`) require SSR mode
- For static sites, use external API or serverless functions

### Assets Not Loading
- Check `app.baseURL` configuration
- Verify CDN URL if using `app.cdnURL`

### Environment Variables Not Available
- Use `runtimeConfig` instead of direct `process.env` access
- Prefix runtime overrides with `NUXT_`

### Build Failing
- Clear `.nuxt/` and `.output/` directories: `rm -rf .nuxt .output`
- Check for incompatible modules (Nuxt 2 vs Nuxt 3)

## When to Use This Skill

- Deploying Nuxt 3 applications
- Vue.js apps requiring SSR or SSG
- Hybrid Vue.js applications with mixed rendering strategies
- For simple Vue SPAs without SSR, use `vue-deployment` skill instead
