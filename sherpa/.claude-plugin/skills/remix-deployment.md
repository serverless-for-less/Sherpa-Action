---
name: Remix Deployment
description: Steps required to build and deploy Remix applications with SSR support using various deployment adapters and runtimes.
---

When building and deploying Remix applications, follow these steps. Remix is a full-stack React framework focused on web standards and server-side rendering.

**Important:** Remix v2 uses Vite as the build tool by default. For VM deployments, use the Express adapter or build for Node.js runtime. Do not use platform-specific templates unless deploying to those platforms.

## Detection

Identify Remix applications by:
- `@remix-run/react` in `package.json` dependencies
- `remix.config.js` (Remix v1) or `vite.config.ts` with Remix plugin (Remix v2)
- `app/routes/` directory structure
- `app/root.tsx` file

### Remix Version Detection
- **Remix v2 (Vite):** `@remix-run/dev` v2.x, `vite.config.ts` with `@remix-run/dev/vite-plugin`
- **Remix v1 (Classic):** `@remix-run/dev` v1.x, `remix.config.js`

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

### Remix v2 (Vite)

```bash
# Build command
npm run build
# or
npx remix vite:build
```

**Output directory:** `build/`
- `build/server/` - Server bundle
- `build/client/` - Client assets

### Remix v1 (Classic Compiler)

```bash
# Build command
npm run build
# or
npx remix build
```

**Output directory:**
- `build/` - Server bundle
- `public/build/` - Client assets

## Server Configuration

Remix requires a server to handle requests. Check for server entry point:

### Remix v2 with Express

```typescript
// server.ts or server.js
import { createRequestHandler } from '@remix-run/express';
import express from 'express';

const app = express();
app.use(express.static('build/client'));
app.all('*', createRequestHandler({ build: await import('./build/server/index.js') }));
app.listen(3000);
```

### Remix v1 with Express

```javascript
// server.js
const { createRequestHandler } = require('@remix-run/express');
const express = require('express');
const path = require('path');

const app = express();
app.use(express.static('public'));
app.all('*', createRequestHandler({ build: require('./build') }));
app.listen(3000);
```

## Deployment

Remix apps require a server runtime. Follow these steps for VM deployment:

1. Build: `npm run build`

2. Copy the following to the server via scp using `vm-access` skill:
   - Remix v2: `build/`, `package.json`, `package-lock.json`, `server.ts` (or server entry)
   - Remix v1: `build/`, `public/`, `package.json`, `package-lock.json`, `server.js`

3. Install dependencies on server:
```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app
  npm ci --production
EOF
```

4. Run the Remix server:
```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  # Remix v2
  node server.js
  # or if using tsx
  npx tsx server.ts

  # With PM2
  pm2 start server.js --name "remix-app"
EOF
```

5. Upload static assets to CDN using `cdn-uploading` skill (optional):
   - Remix v2: `build/client/`
   - Remix v1: `public/build/`

## Vite Configuration (Remix v2)

```typescript
// vite.config.ts
import { vitePlugin as remix } from '@remix-run/dev';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    remix({
      // Server bundle output
      serverBuildFile: 'index.js',
      // Enable server-side HMR in dev
      future: {
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
      },
    }),
  ],
});
```

## Environment Variables

Remix accesses environment variables through the loader context:

### Server-Side (Loaders/Actions)
```typescript
// app/routes/_index.tsx
export async function loader({ context }: LoaderFunctionArgs) {
  const apiKey = process.env.API_KEY;
  // fetch data...
}
```

### Client-Side
Use loader data to pass environment values to client:
```typescript
// app/root.tsx
export async function loader() {
  return json({
    ENV: {
      PUBLIC_API_URL: process.env.PUBLIC_API_URL,
    },
  });
}

export default function App() {
  const data = useLoaderData<typeof loader>();
  return (
    <html>
      <head>
        <script
          dangerouslySetInnerHTML={{
            __html: `window.ENV = ${JSON.stringify(data.ENV)}`,
          }}
        />
      </head>
      {/* ... */}
    </html>
  );
}
```

### Setting Variables

**Runtime (recommended for Remix):**
```bash
API_KEY=secret PUBLIC_API_URL=https://api.example.com node server.js
```

**Build Time:**
```bash
# Only for static values baked into client bundle
PUBLIC_API_URL=https://api.example.com npm run build
```

## Base Path Configuration

For deployment to a subdirectory:

### Remix v2 (Vite)
```typescript
// vite.config.ts
export default defineConfig({
  base: '/my-app/',
  plugins: [remix()],
});
```

### Remix v1
```javascript
// remix.config.js
module.exports = {
  publicPath: '/my-app/build/',
  assetsBuildDirectory: 'public/my-app/build',
};
```

## PM2 Configuration

For production deployments:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  cat > ecosystem.config.js << 'PMEOF'
module.exports = {
  apps: [{
    name: 'remix-app',
    script: 'server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
      API_KEY: process.env.API_KEY,
      PUBLIC_API_URL: 'https://api.example.com'
    }
  }]
}
PMEOF

  pm2 start ecosystem.config.js
  pm2 save
EOF
```

## Express Server Template

Complete Express server for Remix v2:

```typescript
// server.ts
import { createRequestHandler } from '@remix-run/express';
import compression from 'compression';
import express from 'express';
import morgan from 'morgan';

const app = express();

app.use(compression());

// Serve static assets
app.use(
  '/assets',
  express.static('build/client/assets', { immutable: true, maxAge: '1y' })
);
app.use(express.static('build/client', { maxAge: '1h' }));

app.use(morgan('tiny'));

// Handle Remix requests
app.all(
  '*',
  createRequestHandler({
    build: await import('./build/server/index.js'),
  })
);

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

## File Uploads and Forms

Remix handles forms natively. Ensure your server handles multipart data:

```typescript
// app/routes/upload.tsx
import { unstable_parseMultipartFormData } from '@remix-run/node';

export async function action({ request }: ActionFunctionArgs) {
  const formData = await unstable_parseMultipartFormData(
    request,
    uploadHandler
  );
  // Process upload...
}
```

## Monorepo Support

For monorepos (Turborepo, Nx):

1. Identify the Remix app directory
2. Build from root or app directory:
   ```bash
   turbo run build --filter=web
   # or
   cd apps/web && npm run build
   ```
3. Output will be in `apps/web/build/`

## Database Considerations

Remix apps often connect to databases:

- Set `DATABASE_URL` environment variable at runtime
- For Prisma: Run `npx prisma generate` during build, `npx prisma migrate deploy` on server
- Connection pooling recommended for serverless-like deployments

## Streaming and Deferred Data

Remix supports streaming responses. Ensure your server/CDN doesn't buffer responses:

```typescript
// In loader
import { defer } from '@remix-run/node';

export async function loader() {
  return defer({
    critical: await getCriticalData(),
    lazy: getLazyData(), // Promise, not awaited
  });
}
```

## Common Issues

### Build Failing
- Check for missing dependencies in production
- Verify Vite config for Remix v2
- Clear cache: `rm -rf node_modules/.vite build`

### Server Not Starting
- Check server entry file exists and is correct
- Verify `@remix-run/express` or appropriate adapter installed
- Check PORT environment variable

### Assets Not Loading
- Verify static file serving in Express configuration
- Check `base` path configuration
- Ensure `build/client/` directory is accessible

### Environment Variables Not Available
- Use `process.env` in loaders/actions (server-side)
- Pass values through loader for client access
- Set variables at runtime, not just build time

### Forms Not Submitting
- Ensure server handles POST requests
- Check for CORS issues if API is separate
- Verify action is exported from route

### Sessions Not Persisting
- Configure session storage properly
- For distributed deployments, use Redis or database sessions
- Check cookie settings (domain, secure flag)

## Output Structure

### Remix v2 (Vite)
```
build/
├── client/
│   ├── assets/           # Hashed static assets
│   │   ├── root-[hash].js
│   │   └── routes/
│   └── favicon.ico
└── server/
    └── index.js          # Server bundle
```

### Remix v1 (Classic)
```
build/
├── index.js              # Server bundle
└── [route modules]

public/
└── build/
    ├── manifest-[hash].js
    └── [client bundles]
```

## When to Use This Skill

- Deploying Remix applications (v1 or v2)
- Full-stack React apps with server-side rendering
- Applications with complex data loading requirements
- Apps using React Router v6+ patterns
- Projects requiring streaming SSR
