---
name: Angular Deployment
description: Steps required to build and deploy Angular applications to cloud providers, including SPA deployments and Angular Universal SSR.
---

When building and deploying Angular applications, follow these steps. Angular apps are typically SPAs that compile to static files, but can also use Angular Universal for server-side rendering.

## Detection

Identify Angular applications by:
- `@angular/core` in `package.json` dependencies
- `angular.json` configuration file in root
- `@angular/cli` in devDependencies
- `.ts` files with Angular decorators (`@Component`, `@NgModule`)

## Determine Application Type

Check `angular.json` and dependencies:

1. **Standard SPA**: No `@angular/ssr` or `@nguniversal/*` packages
2. **Angular Universal (SSR)**: Has `@angular/ssr` (Angular 17+) or `@nguniversal/*` packages
3. **Prerendered (SSG)**: Uses prerender builder in `angular.json`

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with production configuration.

### Standard SPA Build

```bash
# Build command
npm run build
# or
npx ng build --configuration=production
# or (older projects)
npx ng build --prod
```

**Output directory:** `dist/[project-name]/` or `dist/[project-name]/browser/` (Angular 17+)

Check `angular.json` for custom output path:
```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "options": {
            "outputPath": "dist/my-app"
          }
        }
      }
    }
  }
}
```

### Angular 17+ Application Builder

Angular 17+ uses the new application builder with different output structure:

```
dist/[project-name]/
├── browser/     # Client-side files (deploy this for SPA)
└── server/      # Server files (for SSR only)
```

### Server-Side Rendering (SSR) Build

```bash
# Angular 17+ with @angular/ssr
npm run build
# Outputs both browser and server bundles

# Older Angular Universal
npm run build:ssr
```

## Deployment by Mode

### Static SPA Deployment

**When:** Standard Angular application without SSR

1. Identify the output directory:
   - Angular 17+: `dist/[project-name]/browser/`
   - Angular 16 and earlier: `dist/[project-name]/`

2. Upload the output folder to CDN using `cdn-uploading` skill.

3. Configure SPA fallback for client-side routing.

### Server-Side Rendering (SSR) Deployment

**When:** Using `@angular/ssr` or Angular Universal

1. Build the application: `npm run build`

2. Copy the `dist/` folder to the server via scp using `vm-access` skill

3. Run the Angular server:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  # Angular 17+ with @angular/ssr
  node dist/[project-name]/server/server.mjs

  # Or with PM2
  pm2 start dist/[project-name]/server/server.mjs --name "angular-app"

  # Older Angular Universal
  node dist/[project-name]/server/main.js
EOF
```

4. Optionally upload static assets to CDN for offloading.

### Prerendered (SSG) Deployment

**When:** Using prerender builder

```bash
# Build with prerendering
npm run prerender
# or
npx ng run [project-name]:prerender
```

Output will include pre-rendered HTML files. Deploy to CDN using `cdn-uploading` skill.

## Client-Side Routing Configuration

Angular apps using the Router need SPA fallback configuration.

### Important: SPA Fallback

Configure your CDN/server to serve `index.html` for all routes that don't match a static file.

**Cloudflare Pages:** Automatic SPA support
**AWS S3 + CloudFront:** Configure custom error response for 404s
**Nginx:**
```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Hash Location Strategy Alternative

If using `HashLocationStrategy`, no server configuration needed:
```typescript
// app.config.ts or app.module.ts
providers: [
  { provide: LocationStrategy, useClass: HashLocationStrategy }
]
```

## Environment Variables

Angular uses environment files for build-time configuration:

### File-based Environments

```typescript
// src/environments/environment.ts (development)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000'
};

// src/environments/environment.prod.ts (production)
export const environment = {
  production: true,
  apiUrl: 'https://api.example.com'
};
```

Configure file replacement in `angular.json`:
```json
{
  "configurations": {
    "production": {
      "fileReplacements": [{
        "replace": "src/environments/environment.ts",
        "with": "src/environments/environment.prod.ts"
      }]
    }
  }
}
```

### Runtime Environment Variables (SSR only)

For SSR applications, you can use Node.js environment variables:
```typescript
// In server code
const apiUrl = process.env['API_URL'] || 'https://api.example.com';
```

## Base Path Configuration

For deployment to a subdirectory:

### Build-time Configuration

```bash
npx ng build --base-href=/my-app/
```

Or in `angular.json`:
```json
{
  "architect": {
    "build": {
      "options": {
        "baseHref": "/my-app/"
      }
    }
  }
}
```

### Deploy URL (for assets on CDN)

```bash
npx ng build --deploy-url=https://cdn.example.com/assets/
```

## Monorepo Support (Nx)

For Nx workspaces:

1. Identify the app (e.g., `apps/my-app`)
2. Build with Nx:
   ```bash
   npx nx build my-app --configuration=production
   # or
   npx nx run my-app:build:production
   ```
3. Output: `dist/apps/my-app/`

## PM2 Configuration for SSR

For production SSR deployments:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app

  cat > ecosystem.config.js << 'PMEOF'
module.exports = {
  apps: [{
    name: 'angular-app',
    script: 'dist/[project-name]/server/server.mjs',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 4000
    }
  }]
}
PMEOF

  pm2 start ecosystem.config.js
  pm2 save
EOF
```

## Service Worker / PWA

If using `@angular/pwa`:

1. Service worker files are automatically included in build output
2. Ensure `ngsw-config.json` is configured correctly
3. All PWA assets will be in the output directory
4. Configure proper cache headers on CDN

## Common Issues

### Assets Not Loading
- Check `baseHref` configuration
- Verify `deployUrl` if using CDN for assets
- Ensure assets are in the `assets` array in `angular.json`

### Routes Returning 404
- Configure SPA fallback on CDN/server
- Verify `baseHref` matches deployment path

### Build Failing
- Clear cache: `rm -rf .angular/cache node_modules/.cache`
- Check for TypeScript errors: `npx ng build --configuration=production`
- Verify all dependencies are installed

### SSR Hydration Errors
- Ensure server and client render same content
- Use `isPlatformBrowser()` for browser-only code
- Check for direct DOM manipulation

### Lazy Loaded Modules Not Loading
- Verify chunk files are deployed
- Check `baseHref` and `deployUrl` configuration
- Ensure all JavaScript files from output are uploaded

## Output Structure Reference

### Angular 17+ (Application Builder)
```
dist/my-app/
├── browser/
│   ├── index.html
│   ├── main-[hash].js
│   ├── polyfills-[hash].js
│   ├── styles-[hash].css
│   └── assets/
└── server/           # Only with SSR
    └── server.mjs
```

### Angular 16 and Earlier
```
dist/my-app/
├── index.html
├── main.[hash].js
├── polyfills.[hash].js
├── runtime.[hash].js
├── styles.[hash].css
└── assets/
```

## When to Use This Skill

- Deploying Angular CLI applications
- Angular apps with or without server-side rendering
- Angular Universal applications
- Nx workspace Angular applications
