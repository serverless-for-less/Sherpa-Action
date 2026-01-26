---
name: Astro Deployment
description: Steps required to build and deploy Astro applications with support for static generation, SSR, and hybrid rendering modes.
---

When building and deploying Astro applications, follow these steps. Astro is a content-focused framework that supports static generation by default, with optional server-side rendering.

**Important:** Use the Node.js adapter (`@astrojs/node`) for SSR deployments to VMs. Do not use platform-specific adapters unless deploying to those platforms.

## Detection

Identify Astro applications by:
- `astro` in `package.json` dependencies
- `astro.config.mjs` or `astro.config.ts` in root
- `src/pages/` directory structure
- `.astro` component files

## Determine Output Mode

Check `astro.config.mjs` for the output configuration:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  // Static generation (default)
  output: 'static',

  // Server-side rendering
  output: 'server',

  // Hybrid (static by default, opt-in to SSR)
  output: 'hybrid'
});
```

### Page-Level Rendering Control

With `hybrid` or `server` output:
```astro
---
// Static page in server/hybrid mode
export const prerender = true;

// Dynamic page in hybrid mode
export const prerender = false;
---
```

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

```bash
# Build command (all modes)
npm run build
# or
npx astro build
```

## Deployment by Output Mode

### Static Generation (Default)

**When:** `output: 'static'` (default) or not specified

**Output directory:** `dist/`

1. Build: `npm run build`
2. Upload the `dist/` folder to CDN using `cdn-uploading` skill

```bash
# Build static site
npx astro build
# Output is in dist/
```

No server configuration needed - all pages are pre-rendered HTML.

### Server-Side Rendering (SSR)

**When:** `output: 'server'` with Node.js adapter

**Output directory:** `dist/`

1. Install and configure the Node.js adapter:
```bash
npx astro add node
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({
    mode: 'standalone'  // or 'middleware'
  })
});
```

2. Build: `npm run build`

3. Copy the `dist/` folder to the server via scp using `vm-access` skill

4. Run the Astro server:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app/dist

  # Run the server
  node server/entry.mjs

  # Or with PM2
  pm2 start server/entry.mjs --name "astro-app"
EOF
```

5. Upload static assets from `dist/client/` to CDN using `cdn-uploading` skill (optional)

### Hybrid Rendering

**When:** `output: 'hybrid'`

Deploy the same as SSR mode - server required for dynamic pages. Static pages are pre-rendered and served directly.

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'hybrid',
  adapter: node({
    mode: 'standalone'
  })
});
```

## Node.js Adapter Modes

### Standalone Mode (Recommended)
```javascript
adapter: node({
  mode: 'standalone'
})
```
- Self-contained server, run with `node server/entry.mjs`
- Handles both static assets and dynamic routes

### Middleware Mode
```javascript
adapter: node({
  mode: 'middleware'
})
```
- Exports a handler for use with Express/Fastify
- You provide the HTTP server

Example with Express:
```javascript
import express from 'express';
import { handler as ssrHandler } from './dist/server/entry.mjs';

const app = express();
app.use(express.static('dist/client/'));
app.use(ssrHandler);
app.listen(3000);
```

## Environment Variables

Astro uses different access patterns for client/server:

### Server-Side Only
```astro
---
// In .astro files or API routes
const apiKey = import.meta.env.API_KEY;
---
```

### Client-Side (Public)
Variables prefixed with `PUBLIC_` are available in client-side code:
```javascript
// Anywhere
const apiUrl = import.meta.env.PUBLIC_API_URL;
```

### Setting Variables

**Build Time:**
```bash
PUBLIC_API_URL=https://api.example.com npm run build
```

**Runtime (SSR only):**
```bash
API_KEY=secret123 node dist/server/entry.mjs
```

### .env Files
Astro supports `.env`, `.env.local`, `.env.production` files.

## Base Path Configuration

For deployment to a subdirectory:

```javascript
// astro.config.mjs
export default defineConfig({
  base: '/my-app/',
  // For assets on a different domain
  build: {
    assetsPrefix: 'https://cdn.example.com'
  }
});
```

## Framework Integrations

Astro supports multiple UI frameworks (Islands Architecture):

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import svelte from '@astrojs/svelte';

export default defineConfig({
  integrations: [react(), vue(), svelte()]
});
```

All framework components are included in the build output automatically.

## PM2 Configuration for SSR

For production SSR deployments:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  cat > ecosystem.config.js << 'PMEOF'
module.exports = {
  apps: [{
    name: 'astro-app',
    script: 'dist/server/entry.mjs',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      HOST: '0.0.0.0',
      PORT: 4321,
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

- `PORT` - Server port (default: 4321)
- `HOST` - Server host (default: localhost)

```bash
HOST=0.0.0.0 PORT=3000 node dist/server/entry.mjs
```

## API Routes (Endpoints)

Astro supports API routes in `src/pages/api/`:

```typescript
// src/pages/api/hello.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = ({ params, request }) => {
  return new Response(JSON.stringify({ message: 'Hello!' }), {
    headers: { 'Content-Type': 'application/json' }
  });
};
```

- API routes require SSR mode (`output: 'server'` or `'hybrid'`)
- With static output, API routes are not available at runtime

## Content Collections

Astro's content collections are processed at build time:

```
src/
├── content/
│   ├── config.ts
│   └── blog/
│       ├── post-1.md
│       └── post-2.md
└── pages/
    └── blog/
        └── [slug].astro
```

All content is compiled into the output - no runtime processing needed.

## Monorepo Support

For monorepos:

1. Identify the Astro app directory
2. Build from the app directory:
   ```bash
   cd apps/web && npm run build
   # or
   turbo run build --filter=web
   ```
3. Output will be in `apps/web/dist/`

## Common Issues

### Build Failing
- Check for missing environment variables referenced in code
- Verify all integrations are installed: `npx astro add [integration]`
- Clear cache: `rm -rf node_modules/.astro`

### 404 on Pages
- Static mode: Ensure all pages are in `src/pages/`
- SSR mode: Check adapter configuration
- Verify `base` path configuration

### Client-Side JavaScript Not Working
- Astro ships zero JS by default
- Add `client:*` directive to interactive components:
  ```astro
  <ReactComponent client:load />
  <VueComponent client:visible />
  ```

### Environment Variables Undefined
- Check `PUBLIC_` prefix for client-side variables
- Verify `.env` file is in project root
- Restart dev server after adding variables

### Assets Not Loading
- Check `base` configuration
- Verify `build.assetsPrefix` if using CDN
- Ensure asset paths are correct

### SSR API Routes Not Working
- API routes require `output: 'server'` or `'hybrid'`
- Check that Node.js adapter is configured
- Verify route file uses correct export format

## Output Structure

### Static Output
```
dist/
├── index.html
├── about/
│   └── index.html
├── _astro/              # Bundled assets
│   ├── [hash].css
│   └── [hash].js
└── [static assets]
```

### SSR Output (Standalone)
```
dist/
├── client/              # Static assets
│   ├── _astro/
│   └── [static files]
└── server/
    └── entry.mjs        # Server entry point
```

## When to Use This Skill

- Deploying Astro websites and applications
- Content-heavy sites with optional interactivity
- Multi-framework projects using Astro Islands
- Static sites with blog, documentation, or marketing content
- SSR applications requiring server-side data fetching
